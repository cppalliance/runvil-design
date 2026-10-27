# 7. Panel (PN)

The pane usually along the bottom. Views dragged in from the side bar keep their own features.

7 groups, 219 features. Each feature lists its entry points and the pieces that finish it.

## PN-1 Panel frame and layout

(both). The container itself: show or hide, position, align, maximize, and the tab bar. Examples: show or hide (Ctrl+J); position and alignment; maximize and restore; tabs as labels or icons.

23 features.

#### PN-1.1 Show or hide the panel

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (both commands are present with the same menu rows, but both are disabled while an auxiliary window has focus; the two layout toggle icons both use codicon panelFrame where stock uses layoutPanel and layoutPanelOff; the title bar toggle sits in group 2_pane_toggles; the Hide Panel title row is declared inside the toggle command)
- What: Open or close the whole panel from View > Appearance > Panel, from the Hide Panel button at the right end of the panel title, or from the Command Palette (Toggle Panel Visibility, Hide Panel). Opening any panel view (Problems, Output, Terminal, Debug Console and so on) also shows the panel. Showing it brings back the last view used, or the first container that still has views. Hiding it closes the active view and moves focus to the editor. The panel starts hidden in a new workspace, and it hides itself when its last view container goes away.
- Commands: workbench.action.togglePanel; workbench.action.closePanel
- Menus: MenubarAppearanceMenu (group 2_workbench_layout, order 5); LayoutControlMenuSubmenu (group 0_workbench_layout, order 4); PanelTitle (group navigation, order 2); CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (25-28, 31-97); layout.ts (2050-2130, 2294-2309, 2828-2854); parts\paneCompositePart.ts (186-206, 510-537); C bundle (class hva with `Toggle Panel Visibility`, closePanel action with precondition on isAuxiliaryWindowFocusedContext, icons EhS and IhS bound to Oe.panelFrame); NLS 3932-3936, 3952-3953, 3974
- Pieces:
  1. Choosing View > Appearance > Panel shows the panel when it is hidden and hides it when it is shown (`workbench.action.togglePanel`).
  2. Clicking the Hide Panel button at the right end of the panel title closes the panel (`workbench.action.closePanel`).
  3. The Command Palette has Toggle Panel Visibility and Hide Panel, and the layout controls menu has a panel toggle row.
  4. Opening any panel view (Problems, Output, Terminal, Debug Console and so on) also shows the panel.
  5. Showing the panel brings back the last view used, or the first container that still has views.
  6. Hiding the panel closes the active view and moves focus to the editor.
  7. The panel starts hidden in a new workspace, and it hides itself when its last view container goes away.
  8. In Cursor both commands are disabled while an auxiliary window has focus, and both layout toggle icons use the `panelFrame` icon where stock uses `layoutPanel` and `layoutPanelOff`. [cursor only]
  9. In Cursor the title bar panel toggle sits in menu group `2_pane_toggles`. [cursor only]

#### PN-1.2 Toggle the panel with Ctrl+J

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (key and command present; Cursor adds `!isAuxiliaryWindowFocusedContext` to the key and the command, and a branch for the context key agentChatMaximized that calls setUnifiedMaximizeState)
- What: Press Ctrl+J (Cmd+J on macOS) to hide the panel when it is open and show it when it is closed; the same command is in the Command Palette as Toggle Panel Visibility, hiding a maximized panel first restores the editor, and in Cursor the key first leaves the maximized layout that hides the editor before it shows the panel, and keeps that layout instead of hiding the panel when the panel is open in it.
- Commands: workbench.action.togglePanel
- Menus: CommandPalette
- Keys: Ctrl+J (Cmd+J on macOS)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (31-72); layout.ts (2050-2130); C bundle (class hva: keybinding `Me.CtrlCmd|ve.KeyJ` with `when:FT.toNegated()`, run body reading context key Eme, method setUnifiedMaximizeState, context key `agentChatMaximized`)
- Pieces:
  1. Pressing Ctrl+J (Cmd+J on macOS) hides the panel when it is open.
  2. Pressing Ctrl+J (Cmd+J on macOS) shows the panel when it is closed.
  3. Pressing Ctrl+J while the panel is maximized first restores the editor, then hides the panel.
  4. In Cursor, pressing Ctrl+J while the maximized layout that hides the editor is active first leaves that layout before it shows the panel. [cursor only]
  5. In Cursor, pressing Ctrl+J while the panel is open in the maximized layout that hides the editor keeps that layout instead of hiding the panel. [cursor only]
  6. In Cursor, Ctrl+J and the Toggle Panel Visibility command are switched off while an auxiliary window has focus. [cursor only]

#### PN-1.3 Move the panel to the bottom, top, left or right

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all four ids, the labels Move Panel To Top, Move Panel Left, Move Panel Right, Move Panel To Bottom, and the Panel Position submenu registration were found)
- What: Put the panel under the editor (the default), above it, or beside it on the left or right. Use View > Appearance > Panel Position, the Panel Position submenu in the panel tab bar right-click menu (the current position is ticked), or four Command Palette commands. Moving a hidden panel shows it. The position is remembered per workspace, and alignment only applies to top and bottom.
- Commands: workbench.action.positionPanelTop; workbench.action.positionPanelLeft; workbench.action.positionPanelRight; workbench.action.positionPanelBottom
- Menus: MenubarAppearanceMenu (submenu Panel Position, group 3_workbench_layout_move, order 4); PanelPositionMenu; CommandPalette
- Keys: none found
- Settings: workbench.panel.defaultLocation
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (127-215); layout.ts (2362-2440, 2847); parts\panel\panelPart.ts (167-194); C bundle (position actions built from the list xhS, submenu Pe.PanelPositionMenu); NLS 3938-3941, 3946, 3955-3958
- Pieces:
  1. A panel at the bottom sits under the editor (the default), at the top it sits above the editor, and at the left or right it sits beside the editor.
  2. Four Command Palette commands, Move Panel To Top, Move Panel Left, Move Panel Right and Move Panel To Bottom, move the panel (`workbench.action.positionPanelTop`, `workbench.action.positionPanelLeft`, `workbench.action.positionPanelRight`, `workbench.action.positionPanelBottom`).
  3. View > Appearance > Panel Position is a submenu with the four positions.
  4. The Panel Position submenu in the panel tab bar right-click menu lists the four positions with the current one ticked.
  5. Moving a hidden panel to a new position also shows it.
  6. The panel position is remembered per workspace.
  7. Alignment choices apply only when the panel is at the top or bottom.

#### PN-1.4 Align the panel (center, left, right, justify)

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all four ids, the labels Set Panel Alignment to Left, Right, Center and Justify, the Align Panel submenu registration, and the four Panel Alignment rows of the Customize Layout picker (list oym, NLS 3141-3144 and header 3152) were found; the picker itself is TB-4.7)
- What: Choose how far a top or bottom panel stretches. Center keeps it under the editor only (the default). Justify stretches it across the full window width, under both side bars. Left and Right stretch it to that window edge, under whichever side bar sits on that edge. Pick it from View > Appearance > Align Panel, the Align Panel submenu in the tab bar right-click menu, the Panel Alignment rows in the Customize Layout picker, or four Command Palette commands. Choosing an alignment while the panel is on the left or right first moves it to the bottom, and a non-center alignment restores a maximized panel. The choice is remembered per profile.
- Commands: workbench.action.alignPanelLeft; workbench.action.alignPanelRight; workbench.action.alignPanelCenter; workbench.action.alignPanelJustify; workbench.action.customizeLayout
- Menus: MenubarAppearanceMenu (submenu Align Panel, group 3_workbench_layout_move, order 5); PanelAlignmentMenu; CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (135-251); layout.ts (1960-2048, 2323-2325, 2848); actions\layoutActions.ts (1378-1383, 1502-1504); C bundle (align actions built from the list AhS, submenu Pe.PanelAlignmentMenu, context key panelAlignment); NLS 3942-3945, 3947, 3959-3962
- Pieces:
  1. Center alignment keeps a top or bottom panel under the editor only, and is the default.
  2. Justify alignment stretches a top or bottom panel across the full window width, under both side bars.
  3. Left and Right alignment stretch a top or bottom panel to that window edge, under whichever side bar sits on that edge.
  4. Four Command Palette commands, Set Panel Alignment to Left, Right, Center and Justify, set the alignment (`workbench.action.alignPanelLeft`, `workbench.action.alignPanelRight`, `workbench.action.alignPanelCenter`, `workbench.action.alignPanelJustify`).
  5. View > Appearance > Align Panel is a submenu with the four alignments.
  6. The Align Panel submenu in the panel tab bar right-click menu lists the four alignments with the current one ticked.
  7. The Customize Layout picker (`workbench.action.customizeLayout`) has four Panel Alignment rows.
  8. Choosing an alignment while the panel is on the left or right first moves the panel to the bottom.
  9. Choosing a non-center alignment restores a maximized panel.
  10. The alignment is remembered per profile.

#### PN-1.5 Maximize and restore the panel

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (command, menu row and state present; the button icon is chevronUp and the restore state swaps to a new panel-restore icon (chevronDown) where stock keeps one screenFull icon; tooltips read "Maximize Panel Size" and "Restore Panel Size" where stock reads "Maximize Panel" and "Restore Panel"; a panelmaximized class is added on the main container; in unified mode with a left or right panel the secondary side bar width is kept)
- What: Click the Maximize button at the right of the panel title (or run Toggle Maximized Panel) to make the panel fill everything under the title bar by hiding the editor area. Click it again, now showing Restore, to bring the editor back at its earlier size. Running it while the panel is hidden shows the panel and maximizes it. It only works when the panel is center aligned or sits left or right. For a top or bottom panel with another alignment the command is disabled, and the code has a warning "Maximizing the panel is only supported when it is center aligned."
- Commands: workbench.action.toggleMaximizedPanel
- Menus: PanelTitle (group navigation, order 1); CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (24-25, 275-318); layout.ts (2209-2251, 1886-1893); parts\panel\media\panelpart.css (17-45, 77-100); C bundle (action with id toggleMaximizedPanel, icons ChS and ThS, method toggleMaximizedPanel with isUnifiedMode, method updatePanelMaximizedClass; C css rules for codicon-panel-maximize and codicon-panel-restore rotation); NLS 3929-3930, 3949-3951, 3965
- Pieces:
  1. Clicking the Maximize button at the right of the panel title makes the panel fill everything under the title bar by hiding the editor area.
  2. While maximized the button shows Restore, and clicking it brings the editor back at its earlier size.
  3. Running Toggle Maximized Panel from the Command Palette (`workbench.action.toggleMaximizedPanel`) maximizes and restores the same way.
  4. Running Toggle Maximized Panel while the panel is hidden shows the panel and maximizes it.
  5. Maximize works only when the panel is center aligned or sits left or right, and for a top or bottom panel with another alignment the command is disabled with the warning "Maximizing the panel is only supported when it is center aligned."
  6. In Cursor the Maximize button uses the `chevronUp` icon and Restore swaps to a new panel-restore icon (`chevronDown`), where stock keeps one `screenFull` icon. [cursor only]
  7. In Cursor the tooltips read "Maximize Panel Size" and "Restore Panel Size" where stock reads "Maximize Panel" and "Restore Panel". [cursor only]
  8. In Cursor, maximizing a left or right panel in unified mode keeps the secondary side bar width. [cursor only]

#### PN-1.6 Choose whether the panel opens maximized

- Tag: both. Evidence: source. Boundary: no. Cursor: present (setting found with the same three values and default preserve; the stored flag panel.wasLastMaximized is also there)
- What: A setting decides what happens each time the panel opens: always open it maximized, never open it maximized, or reopen it in the state it had when it was closed (the default). It is ignored when the panel is top or bottom with a non-center alignment, because maximize is not supported there.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.panel.opensMaximized
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (585-595); layout.ts (2112-2125, 2242-2251, 2833); C bundle (setting registration with enum always, never, preserve and enumDescriptions); NLS 4216-4219
- Pieces:
  1. Setting `workbench.panel.opensMaximized` chooses whether the panel opens maximized each time it opens, with values `always`, `never` and `preserve` (the default).
  2. With `always` the panel opens maximized every time.
  3. With `never` the panel opens not maximized every time.
  4. With `preserve` the panel opens in the state it had when it was closed.
  5. The setting is ignored when the panel is at the top or bottom with a non-center alignment.

#### PN-1.7 Hide the editor area so the panel fills the window

- Tag: both. Evidence: source. Boundary: yes. Cursor: changed (id, label Toggle Editor Area Visibility and the same precondition are present; the run body calls ensureChatVisibleOrCreate and setPartHidden on the editor part rather than toggleMaximizedPanel)
- What: Run Toggle Editor Area Visibility from the Command Palette to hide the editor area and bring it back. In stock this runs the same maximize routine as PN-1.5, so the panel fills the window; it is disabled for a top or bottom panel with a non-center alignment. In Cursor it hides the editor area through the part toggle instead, makes sure the agent chat is open first, and when it brings the editor back it hides a panel that had been forced open only by the hide. Boundary reason: in Cursor the command opens or creates the agent chat (`ensureChatVisibleOrCreate`).
- Commands: workbench.action.toggleEditorVisibility
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (251-274); layout.ts (1886-1893, 2216-2240); C bundle (action with id toggleEditorVisibility and context key mainEditorAreaVisible); NLS 3167
- Pieces:
  1. Running Toggle Editor Area Visibility from the Command Palette (`workbench.action.toggleEditorVisibility`) hides the editor area, and running it again brings the editor area back.
  2. The command is disabled for a top or bottom panel with a non-center alignment.
  3. In stock the command runs the same maximize routine as PN-1.5, so the panel fills the window. [stock only]
  4. In Cursor the command hides the editor area through the part toggle instead of maximizing the panel. [cursor only]
  5. In Cursor the command makes sure the agent chat is open (or creates it) before hiding the editor area. [cursor only]
  6. In Cursor, bringing the editor back also hides a panel that had been forced open only by the hide. [cursor only]

#### PN-1.8 Resize the panel

- Tag: both. Evidence: source. Boundary: no. Cursor: present (class has minimumWidth 300, minimumHeight 77, preferredHeight 0.4 of window height, preferredWidth with 300 floor, and a snap getter; the grid sash reset handler and the stored keys panel.size, panel.lastNonMaximizedHeight and panel.lastNonMaximizedWidth were found; both resize ids were found)
- What: Drag the line between the editor and the panel to change its height (top or bottom) or width (left or right). It will not shrink below 77 px high or 300 px wide, dragging past that snaps it shut, and it can be dragged open from closed only when it has view containers. Double-click the line to jump to the preferred size, which is 40 percent of the window height for a top or bottom panel, or the active view's best width (at least 300) for a left or right panel. The size is remembered, and the last non-maximized height and width are kept separately for each direction. With focus in the panel, Increase and Decrease Current View Size change it by 60 px.
- Commands: workbench.action.increaseViewSize; workbench.action.decreaseViewSize
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\panel\panelPart.ts (40-65); parts\paneCompositePart.ts (112-115); layout.ts (1799-1830, 2828-2833); actions\layoutActions.ts (1138-1277); V\src\vs\base\browser\ui\grid\grid.ts (710-741); C bundle (panel part class with minimumHeight 77, onDidSashReset in the grid class, state keys). The resize commands are also listed in SB-1.4.
- Pieces:
  1. Dragging the line between the editor and the panel changes the panel height (top or bottom) or width (left or right).
  2. The panel will not shrink below 77 px high or 300 px wide, and dragging past that snaps it shut.
  3. A closed panel can be dragged open only when it has view containers.
  4. Double-clicking the line jumps to the preferred size, which is 40 percent of the window height for a top or bottom panel, or the active view's best width (at least 300) for a left or right panel.
  5. The size is remembered, and the last non-maximized height and width are kept separately for each direction.
  6. With focus in the panel, Increase Current View Size and Decrease Current View Size change it by 60 px (`workbench.action.increaseViewSize`, `workbench.action.decreaseViewSize`).

#### PN-1.9 Set the default panel location for new workspaces

- Tag: both. Evidence: source. Boundary: no. Cursor: present (setting found with the same four values and default bottom; the stored key panel.position is there too)
- What: A setting picks where the panel first appears in a workspace that has no saved layout: bottom (the default), top, left or right. A workspace that already has a saved position keeps it. The Restore Defaults button in Customize Layout resets this setting (TB-4.8).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.panel.defaultLocation
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (578-584); layout.ts (2847, 3014-3015, 2869); C bundle (setting registration with enum left, bottom, top, right); NLS 4215
- Pieces:
  1. Setting `workbench.panel.defaultLocation` picks where the panel first appears in a workspace with no saved layout, with values `bottom` (the default), `top`, `left` and `right`.
  2. A workspace that already has a saved panel position keeps it.
  3. The Restore Defaults button in Customize Layout resets this setting.

#### PN-1.10 Switch panel views from the tab bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for the label Active View Switcher, the tablist role on the action bar, the same pinned and placeholder storage keys, and the tab action classes; all found)
- What: The panel title row holds one tab for each view container in the panel (Problems, Output, Debug Console, Terminal and so on, in a fixed order). Click a tab to open that container, and the active tab gets an underline and brighter text. A second click within 300 ms is ignored. Tabs show a count badge when the container reports one, and hovering shows the name with its shortcut. The strip is a tab list, so Left, Right, Home and End move between tabs and Enter or Space opens one. Tabs for containers that hide when empty (Terminal, Test Results, Ports, Comments) appear only while the container is active or has views.
- Commands: none found
- Menus: none found
- Keys: Left Arrow; Right Arrow; Home; End; Enter; Space (with a tab focused)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\paneCompositePart.ts (381-441, 723-731); parts\compositeBar.ts (283-322); parts\paneCompositeBar.ts (419-461, 765-835); parts\compositeBarActions.ts (155-440); parts\panel\panelPart.ts (135-165, 213-220); parts\panel\media\panelpart.css (60-76); C bundle (action bar with ariaRole tablist, label NLS 3321, keys workbench.panel.pinnedPanels and workbench.panel.placeholderPanels)
- Pieces:
  1. The panel title row holds one tab for each view container in the panel (Problems, Output, Debug Console, Terminal and so on) in a fixed order.
  2. Clicking a tab opens that container, and the active tab gets an underline and brighter text.
  3. A second click on a tab within 300 ms is ignored.
  4. A tab shows a count badge when its container reports one.
  5. Hovering a tab shows its name with its shortcut.
  6. With a tab focused, Left Arrow, Right Arrow, Home and End move between tabs and Enter or Space opens one.
  7. Tabs for containers that hide when empty (Terminal, Test Results, Ports, Comments) appear only while the container is active or has views.

#### PN-1.11 Show tabs as labels or icons

- Tag: both. Evidence: source. Boundary: no. Cursor: present (setting found with default true, labels Show Icons and Show Labels found, and the menu action id found)
- What: The panel tabs show text labels by default. Switch to compact 16 px icons from the tab bar right-click menu (it says Show Icons when labels are on, and Show Labels when icons are on) or from the setting. The menu row is a menu-only action with id `workbench.action.panel.toggleShowLabels`, not a registered command.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.panel.showLabels
- Source: V\src\vs\workbench\browser\parts\panel\panelPart.ts (111-119, 139-165, 180-193); workbench.contribution.ts (573-577); C bundle (setting registration, toggleShowLabels action in fillExtraContextMenuActions); NLS 3970-3971, 4214
- Pieces:
  1. The panel tabs show text labels by default.
  2. Setting `workbench.panel.showLabels` (default true) chooses between text labels and compact 16 px icons.
  3. The tab bar right-click menu row reads Show Icons when labels are on and Show Labels when icons are on, and choosing it switches between them.
  4. The Show Icons and Show Labels row is a menu-only action (`workbench.action.panel.toggleShowLabels`) and is not in the Command Palette.

#### PN-1.12 Reach hidden tabs from the overflow menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present (action id additionalComposites.action and the label Additional Views found; the chat variant of this button reads More Chats and belongs to the chat surface)
- What: When the tabs do not fit the row width (after the toolbar), the extra ones collapse behind a "..." button called Additional Views. Click it for a list of the hidden tabs with the active one marked, and a count after the name when the tab has a badge. The active tab always stays visible even if it is not kept.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (528-651); parts\compositeBarActions.ts (449-512); parts\panel\panelPart.ts (150-152); C bundle (overflow action class, NLS 3323)
- Pieces:
  1. When the tabs do not fit the row width after the toolbar, the extra ones collapse behind a "..." button called Additional Views.
  2. Clicking Additional Views lists the hidden tabs.
  3. The list marks the active tab, and shows a count after the name when a tab has a badge.
  4. The active tab always stays visible, even if it is not kept.

#### PN-1.13 Tab bar right-click menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the pinned-tab list code, the Views submenu hook, Pe.PanelPositionMenu and Pe.PanelAlignmentMenu reads, the toggleShowLabels action and the Hide Panel row were found in the panel part class; the stored list key is workbench.panel.pinnedPanels)
- What: Right-click the panel title row (or empty space in the tab strip) for one menu with a ticked row for each tab. Unticking a tab hides it from the strip, and the last kept tab cannot be unticked. Below the list are a Views submenu (shown when the active container has more than one view action), Panel Position and Align Panel submenus with the current choice ticked, Show Icons or Show Labels, and Hide Panel.
- Commands: none found
- Menus: PanelPositionMenu; PanelAlignmentMenu; Views (ViewsSubMenu)
- Keys: none found
- Settings: workbench.panel.showLabels
- Source: V\src\vs\workbench\browser\parts\panel\panelPart.ts (167-194); parts\compositeBar.ts (654-684); parts\paneCompositePart.ts (332-344, 758-803); C bundle (fillExtraContextMenuActions with submenu ids workbench.action.panel.position and workbench.action.panel.align; getContextMenuActions with checked and enabled rows); NLS 3928, 3970-3974
- Pieces:
  1. Right-clicking the panel title row, or empty space in the tab strip, opens one menu with a ticked row for each tab.
  2. Unticking a tab hides it from the strip, and the last kept tab cannot be unticked.
  3. A Views submenu shows below the list when the active container has more than one view action.
  4. Panel Position and Align Panel submenus show below the list with the current choice ticked.
  5. A Show Icons or Show Labels row switches the tab display.
  6. A Hide Panel row closes the panel.

#### PN-1.14 Single tab right-click menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present (action ids show.toggleCompositePinned and show.toggleCompositeBadge, submenu id moveToMenu, resetLocationAction, and the labels Hide '{0}', Keep '{0}', Hide Badge, Show Badge, Move To, Reset Location and Configure Keybinding were found; Cursor adds extra rows only for chat tabs, which are not panel tabs)
- What: Right-click one panel tab for its own menu: Configure Keybinding (when the tab has a shortcut command), Hide '<name>' or Keep '<name>' (Hide is disabled for the last kept tab), Hide Badge or Show Badge, Move To with Primary Side Bar and Secondary Side Bar (the panel is left out because the tab is already there), and Reset Location when the container was moved from its home. The tab bar rows from PN-1.13 follow after a separator. Move To sends the whole container to the end of that bar and opens it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts (515-686, 722-757); parts\paneCompositeBar.ts (157-214); C bundle (tab item class showContextMenu, getContextMenuActionsForComposite with moveToMenu); NLS 3920-3925, 3325-3330, 3977-3980
- Pieces:
  1. Right-clicking one panel tab opens its own menu.
  2. Configure Keybinding shows when the tab has a shortcut command.
  3. Hide '<name>' or Keep '<name>' shows, and Hide is disabled for the last kept tab.
  4. Hide Badge or Show Badge shows for the tab.
  5. A Move To submenu offers Primary Side Bar and Secondary Side Bar (the panel is left out because the tab is already there), and choosing one sends the whole container to the end of that bar and opens it.
  6. Reset Location shows when the container was moved from its home.
  7. The tab bar rows from PN-1.13 follow after a separator.

#### PN-1.15 Go to the previous or next panel view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids and labels Previous Panel View and Next Panel View found, built on the same shared switch action)
- What: Two Command Palette commands switch to the tab before or after the active one and wrap around at the ends. They move keyboard focus into the opened view. They have no default shortcut.
- Commands: workbench.action.previousPanelView; workbench.action.nextPanelView
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (252-273); parts\compositeBarActions.ts (760-791); C bundle (two actions extending the switch action with location 1 and offsets -1 and 1); NLS 3963-3964
- Pieces:
  1. Previous Panel View (`workbench.action.previousPanelView`) switches to the tab before the active one and wraps around at the start.
  2. Next Panel View (`workbench.action.nextPanelView`) switches to the tab after the active one and wraps around at the end.
  3. Both commands move keyboard focus into the opened view.
  4. Both commands are in the Command Palette and have no default shortcut.

#### PN-1.16 Focus into Panel

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (id, label Focus into Panel and behavior present; Cursor adds a precondition so it is disabled while an auxiliary window has focus)
- What: Run Focus into Panel from the Command Palette to jump keyboard focus into the active view of the panel. If the panel is hidden it opens first. It has no default shortcut.
- Commands: workbench.action.focusPanel
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (99-125); layout.ts (1287-1290); C bundle (action with precondition FT.toNegated and body that shows the panel then focuses the active composite); NLS 3937, 3954
- Pieces:
  1. Running Focus into Panel from the Command Palette (`workbench.action.focusPanel`) moves keyboard focus into the active view of the panel.
  2. Running Focus into Panel while the panel is hidden opens the panel first.
  3. Focus into Panel has no default shortcut.
  4. In Cursor, Focus into Panel is disabled while an auxiliary window has focus. [cursor only]

#### PN-1.17 Panel title toolbar

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (PanelTitle holds the same two rows; the Hide icon is codicon xTwo where stock uses close; the Maximize icon is chevronUp and Restore is chevronDown; the global toolbar is created without the highlightToggledItems option; the view toolbar has no trailing separator option, and its overflow tooltip reads "Close, Export, Settings and More..." where stock reads "Views and More Actions..."; the icon rotation rules for right, left and top panels are the same)
- What: The right side of the panel title row shows the active view's own buttons (for example the terminal's New, Split and Kill), a "..." menu for the extras, then Maximize or Restore and Hide Panel at the far end. In stock a separator sits between the view's buttons and the two panel buttons, and the Maximize button looks pressed while the panel is maximized. The Maximize and terminal Split icons turn to match the panel when it is on the left, right or top.
- Commands: workbench.action.toggleMaximizedPanel; workbench.action.togglePanel
- Menus: PanelTitle (Maximize at order 1, Hide Panel at order 2); ViewTitle (only as the source of each view's own buttons)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositePart.ts (405-433); parts\paneCompositePart.ts (332-364); V\src\vs\base\browser\ui\toolbar\toolbar.ts (266-268); parts\panel\panelActions.ts (24-28, 74-81, 275-318); parts\panel\media\panelpart.css (77-100); C bundle (title toolbar created with toggleMenuTitle NLS 3332 and no trailingSeparator anywhere in the bundle, global toolbar with hiddenItemStrategy -1 and no highlightToggledItems; C css rotation rules for basepanel right, left and top); NLS 3332, 3927, 3929-3931
- Pieces:
  1. The right side of the panel title row shows the active view's own buttons, for example the terminal's New, Split and Kill.
  2. A "..." menu holds the extra buttons that do not fit.
  3. Maximize or Restore and Hide Panel sit at the far end of the title row.
  4. The Maximize and terminal Split icons turn to match the panel when it is on the left, right or top.
  5. In stock a separator sits between the view's buttons and the two panel buttons. [stock only]
  6. In stock the Maximize button looks pressed while the panel is maximized. [stock only]
  7. In Cursor the Hide Panel button uses the `xTwo` icon where stock uses close, and Maximize uses `chevronUp` with Restore using `chevronDown`. [cursor only]
  8. In Cursor the "..." overflow tooltip reads "Close, Export, Settings and More..." where stock reads "Views and More Actions...". [cursor only]

#### PN-1.18 Drag tabs and views onto the panel

- Tag: both. Evidence: source. Boundary: no. Cursor: present (CompositeBarDndCallbacks with the dragged-over-head and dragged-over-tail classes, the empty pane message area and its text NLS 3926, and the drop handler calling moveViewContainerToLocation and moveViewToLocation with "dnd" were found; the title label is made draggable unconditionally where stock ties it to canMoveViews)
- What: Drag a tab left or right along the panel tab bar to reorder it, and a line shows where it will land. Hovering a tab for 800 ms while dragging switches to it. Drag a container tab from the side bar rail or the secondary side bar onto the panel tab bar, or drag one view header onto it, to move it into the panel (a lone view gets its own new tab). Dropping on the empty panel body also works, and an empty panel reads "Drag a view here to display." Only views that allow moving can be dragged. The drop line uses the colour panel.dropBorder.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (36-138, 163-232); parts\compositeBarActions.ts (551-594); parts\paneCompositePart.ts (252-330, 368-379); V\src\vs\workbench\browser\dnd.ts; C bundle (same classes and handlers); NLS 3926
- Pieces:
  1. Dragging a tab left or right along the panel tab bar reorders it, and a line in the `panel.dropBorder` color shows where it will land.
  2. Hovering a tab for 800 ms while dragging switches to that tab.
  3. Dragging a container tab from the side bar rail or the secondary side bar onto the panel tab bar moves that container into the panel.
  4. Dragging one view header onto the panel tab bar moves that view into the panel, and a lone view gets its own new tab.
  5. Dropping on the empty panel body also moves the container or view into the panel.
  6. An empty panel reads "Drag a view here to display."
  7. Only views that allow moving can be dragged.

#### PN-1.19 Move all secondary side bar views into the panel

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids and the label Move Secondary Side Bar Views To Panel found; in Cursor the secondary side bar holds the agent chat container, see Unclear)
- What: Run Move Secondary Side Bar Views To Panel from the Command Palette to send every view container from the secondary side bar to the panel as new tabs, show the panel, and open the container that was active in the side bar when the panel had none. A hidden twin id, `workbench.action.moveSidePanelToPanel`, does the same.
- Commands: workbench.action.moveSecondarySideBarToPanel; workbench.action.moveSidePanelToPanel
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (344-364, 399-425); C bundle (shared move class lEr with ids moveSidePanelToPanel and moveSecondarySideBarToPanel); NLS 3968-3969
- Pieces:
  1. Running Move Secondary Side Bar Views To Panel from the Command Palette (`workbench.action.moveSecondarySideBarToPanel`) sends every view container from the secondary side bar to the panel as new tabs.
  2. The command shows the panel.
  3. When the panel had no active container, the container that was active in the secondary side bar opens.
  4. A hidden twin command, `workbench.action.moveSidePanelToPanel`, does the same.

#### PN-1.20 The panel exists only in the main window

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the three panel commands and the Ctrl+J key carry `!isAuxiliaryWindowFocusedContext` in Cursor and not in stock; the other three already had it in stock; the panel row in LayoutControlMenu has no isAuxiliaryWindow condition in Cursor, and whether a popped-out window in Cursor draws that menu was not checked)
- What: A popped-out editor window has an editor, a status bar and a title bar and never a panel. In stock its layout controls leave out the panel toggle. Panel commands still act on the main window's panel when run with the popped-out window focused, except that Increase and Decrease Current View Size and Toggle Editor Area Visibility are switched off there. Cursor switches off Ctrl+J, Toggle Panel Visibility, Hide Panel and Focus into Panel in that case as well, and it drops the popped-out-window condition from the panel toggle's layout menu row.
- Commands: workbench.action.togglePanel; workbench.action.closePanel; workbench.action.focusPanel; workbench.action.toggleEditorVisibility; workbench.action.increaseViewSize; workbench.action.decreaseViewSize
- Menus: LayoutControlMenu
- Keys: Ctrl+J (Cmd+J on macOS)
- Settings: none found
- Source: V\src\vs\workbench\browser\layout.ts (1313-1336, 1342-1370); V\src\vs\workbench\services\layout\browser\layoutService.ts (293-300); parts\panel\panelActions.ts (31-125, 320-342); actions\layoutActions.ts (251-274, 1140-1260); C bundle (context key isAuxiliaryWindowFocusedContext, class FT, in the preconditions of hva, closePanel and focusPanel)
- Pieces:
  1. A popped-out editor window has an editor, a status bar and a title bar, and never a panel.
  2. In stock the layout controls of a popped-out window leave out the panel toggle. [stock only]
  3. Panel commands run with a popped-out window focused act on the main window's panel.
  4. Increase Current View Size, Decrease Current View Size and Toggle Editor Area Visibility are switched off while a popped-out window has focus.
  5. In Cursor, Ctrl+J, Toggle Panel Visibility, Hide Panel and Focus into Panel are also switched off while a popped-out window has focus. [cursor only]
  6. In Cursor the panel row in the layout controls menu has no popped-out window condition. [cursor only]

#### PN-1.21 Panel theme colors

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all nine color ids found in the bundle)
- What: A theme or `workbench.colorCustomizations` can restyle the panel frame: panel.background, panel.border, panelTitle.border, panelTitle.activeForeground, panelTitle.inactiveForeground, panelTitle.activeBorder, panelTitleBadge.background, panelTitleBadge.foreground and panel.dropBorder (the tab drag line). panelTitle.border is the line under the title row and panelTitle.activeBorder is the underline of the active tab. Input, section and sticky scroll colors that share the panel prefix are listed under Handed off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts (480-528); parts\panel\panelPart.ts (121-134, 155-164); C bundle (registerColor calls for the nine ids)
- Pieces:
  1. `panel.background` sets the fill of the panel body.
  2. `panel.border` sets the line that separates the panel from the editor.
  3. `panelTitle.border` sets the line under the panel title row.
  4. `panelTitle.activeForeground` and `panelTitle.inactiveForeground` set the text color of the active tab and the other tabs.
  5. `panelTitle.activeBorder` sets the underline of the active tab.
  6. `panelTitleBadge.background` and `panelTitleBadge.foreground` set the colors of the tab count badge.
  7. `panel.dropBorder` sets the color of the tab drag line.

#### PN-1.22 Float the panel as a rounded card (experimental Modern UI)

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched the bundle for modernUI, floating-panels and experimental.floatingPanels, zero hits)
- What: An experimental setting, off by default, draws the side bars and the bottom panel as floating cards with rounded corners, 4 px gaps and a refreshed style; the panel is inset by the card margin and border, and the outermost cards get a doubled gutter against the window edge.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.experimental.modernUI
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (810-815); layout.ts (621-632, 1905); parts\paneCompositePart.ts (596-722); V\src\vs\workbench\services\layout\browser\layoutService.ts (53-63, 380-384); V\src\vs\workbench\browser\media\floatingPanels.css
- Pieces:
  1. The experimental setting `workbench.experimental.modernUI` is off by default and turns on the floating card look. [stock only]
  2. With it on, the side bars and the bottom panel are drawn as floating cards with rounded corners and 4 px gaps. [stock only]
  3. With it on, the panel is inset by the card margin and border. [stock only]
  4. With it on, the outermost cards get a doubled gutter against the window edge. [stock only]

#### PN-1.23 No-title-bar layout adjustments for the panel

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor has a layout with no title bar (body class `no-titlebar-layout`, context key `cursor.noTitlebarLayout.enabled`). In it the panel fills the full height, its title area gives up the slot where layout buttons would go (those buttons can sit in the editor tab strip instead, TB-4.10), the panel's right toolbar is marked as not draggable so clicks reach it, and on macOS the panel gets 35 px of top padding when it is left, right, top or maximized so it clears the window buttons (with an exception when a right side bar is beside it). The panel's own layout button container is also hidden in every layout.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (panel part class overrides createTitleArea and getLayoutActionsContainer using isNoTitlebarLayoutActive, function klt reading context key `cursor.noTitlebarLayout.enabled` or the body class); C css (rules for `body.no-titlebar-layout .monaco-workbench .part.panel`, the 35px mac padding, `.part.panel>.title .global-actions` no-drag, and `.part.panel>.title>.layout-actions-container{display:none!important}`)
- Pieces:
  1. In Cursor's no-title-bar layout (body class `no-titlebar-layout`, context key `cursor.noTitlebarLayout.enabled`) the panel fills the full height. [cursor only]
  2. In that layout the panel title area gives up the slot where layout buttons would go, and those buttons can sit in the editor tab strip instead. [cursor only]
  3. In that layout the panel's right toolbar is marked as not draggable so clicks reach its buttons. [cursor only]
  4. On macOS in that layout the panel gets 35 px of top padding when it is left, right, top or maximized, except when a right side bar is beside it. [cursor only]
  5. The panel's own layout button container is hidden in every layout. [cursor only]


## PN-2 Terminal sessions

(both). Create, name, arrange and close terminals, including profiles, the tabs list, splits, and moving to the editor or a new window. Examples: new terminal and profile picker; the side tabs list; split and groups; move to editor or window.

50 features.

#### PN-2.1 Show or focus the Terminal view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+` or pick View > Terminal to open the panel on the Terminal tab (a first terminal is created if none exists yet), and run Focus Terminal to jump straight into the active terminal, creating one when there is none.
- Commands: workbench.action.terminal.toggleTerminal; workbench.action.terminal.focus
- Menus: MenubarViewMenu
- Keys: Ctrl+` on workbench.action.terminal.toggleTerminal (Mac Ctrl+`); Ctrl+Down on workbench.action.terminal.focus (only when accessibility mode is on and the accessible view for the terminal is open on its last line)
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminal.contribution.ts (view and open command, lines 106-135); terminalActions.ts (Focus, lines 497-514); terminalView.ts (_initializeTerminal, lines 144-186); V\src\vs\workbench\services\views\browser\viewsService.ts (open command descriptor adds the View menu row); C bundle (ids toggleTerminal and focus; NLS key miToggleIntegratedTerminal in module terminal.view.contribution)
- Pieces:
  1. Pressing Ctrl+` (Mac Ctrl+`) or picking View > Terminal opens the panel on the Terminal tab and creates a first terminal when none exists yet.
  2. Pressing Ctrl+` while the Terminal view is open and the panel has focus closes the panel instead.
  3. Running Focus Terminal (`workbench.action.terminal.focus`) moves keyboard focus into the active terminal and creates a terminal when there is none.
  4. With accessibility mode on and the terminal's accessible view open on its last line, pressing Ctrl+Down runs Focus Terminal and returns focus to the terminal.

#### PN-2.2 Create a new terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Click the plus button in the Terminal view, press Ctrl+Shift+` or pick Terminal > New Terminal to open a terminal with the default profile in the default location (the panel, or the editor area when the location setting says editor); Alt or Ctrl click on the plus button splits the active terminal instead, a multi-root window first asks which folder to start in (see PN-2.8), a secondary window opens the terminal in its own editor area, and two palette-only variants create one in the active workspace or a local one while in a remote window.
- Commands: workbench.action.terminal.new; workbench.action.terminal.newInActiveWorkspace; workbench.action.terminal.newLocal
- Menus: MenubarTerminalMenu; ViewTitle; TerminalEditorInstanceContext; TerminalTabEmptyAreaContext
- Keys: Ctrl+Shift+` on workbench.action.terminal.new (Mac Ctrl+Shift+`); Ctrl+Shift+C on workbench.action.terminal.new (web build only, when terminal is not focused)
- Settings: terminal.integrated.defaultLocation
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (New, lines 1213-1263; NewInActiveWorkspace, lines 297-309); V\src\vs\workbench\contrib\terminal\electron-browser\terminalRemote.ts (NewLocal); terminalMenus.ts (Terminal menu lines 41-105, plus button lines 464-483, tab empty area lines 307-330, editor instance menu lines 209-305); terminal.web.contribution.ts; terminalConfiguration.ts (defaultLocation, lines 118-127); C bundle (ids new, newInActiveWorkspace, newLocal; NLS keys miNewTerminal and workbench.action.terminal.new)
- Pieces:
  1. Clicking the plus button in the Terminal view title opens a new terminal with the default profile and makes it the active tab.
  2. Pressing Ctrl+Shift+` or picking Terminal > New Terminal opens a new terminal with the default profile in the default location.
  3. Right-clicking the empty area of the tabs list or inside a terminal editor shows a New Terminal row that opens a new terminal.
  4. Holding Alt or Ctrl while clicking the plus button splits the active terminal instead of creating a separate one.
  5. Setting `terminal.integrated.defaultLocation` chooses where new terminals open, with values `view` (the panel, default) and `editor` (an editor tab).
  6. In a secondary window, New Terminal opens the terminal in that window's own editor area.
  7. In a remote window, the Command Palette offers `workbench.action.terminal.newInActiveWorkspace` to create a terminal in the active workspace and `workbench.action.terminal.newLocal` to create a local one.
  8. In the web build, pressing Ctrl+Shift+C while no terminal has focus creates a new terminal.

#### PN-2.3 Create a terminal with a chosen profile

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run Create New Terminal (With Profile), or right-click the empty area of the tabs list and pick New Terminal With Profile, to get a pick list headed "Select the terminal profile to create" with sections for configured profiles, contributed profiles and detected profiles (the default one first); Alt held on accept splits the active terminal instead, a gear button on each profile row saves it to settings under a name you type, and a profile whose path another user can edit asks for a Yes or Cancel first.
- Commands: workbench.action.terminal.newWithProfile
- Menus: TerminalTabEmptyAreaContext
- Keys: none found
- Settings: terminal.integrated.profiles.windows; terminal.integrated.profiles.osx; terminal.integrated.profiles.linux; terminal.integrated.useWslProfiles
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (NewWithProfile, lines 1530-1642); terminalProfileQuickpick.ts (lines 42-175 and 232-310); terminalService.ts (showProfileQuickPick, lines 236-275); terminalMenus.ts (lines 307-330); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (profiles, lines 228-395); C bundle (id newWithProfile; NLS keys terminal.integrated.selectProfileToCreate, terminalProfiles, terminalProfiles.detected, enterTerminalProfileName, unsafePathWarning, createQuickLaunchProfile)
- Pieces:
  1. Running Create New Terminal (With Profile) opens a pick list headed "Select the terminal profile to create".
  2. The pick list has sections for configured profiles, contributed profiles and detected profiles, with the default profile first.
  3. Choosing a profile row creates a terminal with that profile in the current default location.
  4. Right-clicking the empty area of the tabs list and picking New Terminal With Profile opens the same pick list.
  5. Holding Alt while accepting a profile row splits the active terminal with that profile instead of creating a separate terminal.
  6. Clicking the gear button on a profile row asks for a name you type and saves that profile to the per-platform profiles setting.
  7. Choosing a profile whose path another user can edit first shows a warning with Yes and Cancel buttons.
  8. Settings `terminal.integrated.profiles.windows`, `terminal.integrated.profiles.osx` and `terminal.integrated.profiles.linux` hold the configured profiles shown in the list.
  9. Setting `terminal.integrated.useWslProfiles` chooses whether detected WSL distributions appear in the list, as on or off.

#### PN-2.4 Launch Profile drop-down beside the plus button

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (no New Terminal, New Terminal Window or Split Terminal rows at the top; the split submenu is titled "Split Terminal" and is not named "Split Terminal with Profile"; no grouping of AI profiles; default profile sorted first)
- What: The small chevron next to the plus button (tooltip "Launch Profile...") opens a list of profiles to create a terminal from, with the default profile marked "(Default)", a submenu to split with a chosen profile, then Select Default Profile, Configure Terminal Settings, Run Task... and Configure Tasks...; the same drop-down sits on the editor-area toolbar for terminal editors; stock also puts New Terminal, New Terminal Window and Split Terminal at its top.
- Commands: workbench.action.terminal.newWithProfile; workbench.action.terminal.split; workbench.action.terminal.selectDefaultShell; workbench.action.terminal.openSettings; workbench.action.tasks.runTask; workbench.action.tasks.configureTaskRunner
- Menus: TerminalNewDropdownContext; ViewTitle
- Keys: none found
- Settings: terminal.integrated.profiles.windows; terminal.integrated.profiles.osx; terminal.integrated.profiles.linux; terminal.integrated.defaultProfile.windows; terminal.integrated.defaultProfile.osx; terminal.integrated.defaultProfile.linux
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (getTerminalActionBarArgs, lines 783-930; TerminalNewDropdownContext, lines 332-379); terminalView.ts (createActionViewItem, lines 250-302); terminalEditor.ts (lines 155-190); C bundle (function that builds the drop-down starts straight at the profile loop, NLS keys launchProfile, defaultTerminalProfile and splitTerminal, no key split.profile; the js-debug extension adds a "JavaScript Debug Terminal" profile at C\extensions\ms-vscode.js-debug\package.json)
- Pieces:
  1. Clicking the small chevron next to the plus button (tooltip "Launch Profile...") opens a list of profiles to create a terminal from.
  2. The default profile row is marked "(Default)", and choosing any profile row creates a new terminal with that profile.
  3. A submenu in the list splits the active terminal with a chosen profile.
  4. The list ends with the rows Select Default Profile, Configure Terminal Settings, Run Task... and Configure Tasks...
  5. The same drop-down shows on the toolbar of a terminal editor tab.
  6. Settings `terminal.integrated.profiles.*` decide which profiles are listed and `terminal.integrated.defaultProfile.*` decide which one is marked "(Default)".
  7. Stock puts New Terminal, New Terminal Window and Split Terminal rows at the top of the list. [stock only]
  8. In Cursor the list starts straight at the profile rows with the default profile first, and its split submenu is titled "Split Terminal". [cursor only]

#### PN-2.5 Choose the default terminal profile

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run Terminal: Select Default Profile (or pick it at the bottom of the Launch Profile list) to get a pick list headed "Select your default terminal profile"; the choice is written to the per-platform default profile setting (and a detected profile is also added to the profiles setting), and from then on the plus button, Ctrl+Shift+` and splits use it.
- Commands: workbench.action.terminal.selectDefaultShell
- Menus: TerminalNewDropdownContext
- Keys: none found
- Settings: terminal.integrated.defaultProfile.windows; terminal.integrated.defaultProfile.osx; terminal.integrated.defaultProfile.linux; terminal.integrated.profiles.windows; terminal.integrated.profiles.osx; terminal.integrated.profiles.linux; terminal.integrated.useWslProfiles
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (SelectDefaultProfile, lines 1371-1375); terminalProfileQuickpick.ts (setDefault branch, lines 42-80); terminalService.ts (lines 236-246); terminalMenus.ts (lines 332-343); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (defaultProfile, lines 440-480); C bundle (id selectDefaultShell; NLS key terminal.integrated.chooseDefaultProfile; the three defaultProfile settings)
- Pieces:
  1. Running Terminal: Select Default Profile, or picking Select Default Profile at the bottom of the Launch Profile list, opens a pick list headed "Select your default terminal profile".
  2. Choosing a profile writes it to the setting `terminal.integrated.defaultProfile.windows`, `terminal.integrated.defaultProfile.osx` or `terminal.integrated.defaultProfile.linux` for the current platform.
  3. Choosing a detected profile also adds it to the per-platform `terminal.integrated.profiles.*` setting.
  4. After choosing, the plus button, Ctrl+Shift+` and splits create terminals with the new default profile.
  5. Setting `terminal.integrated.useWslProfiles` chooses whether detected WSL distributions appear in the pick list, as on or off.

#### PN-2.6 Open a new terminal in its own window

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched the bundle for the id and the NLS keys newInNewWindow and miNewInNewWindow: none)
- What: Pick Terminal > New Terminal Window or press Ctrl+Shift+Alt+` to open a terminal in a new compact window (an auxiliary window that holds only that terminal), and the same row shows in the Launch Profile list.
- Commands: workbench.action.terminal.newInNewWindow
- Menus: MenubarTerminalMenu; TerminalNewDropdownContext
- Keys: Ctrl+Shift+Alt+` on workbench.action.terminal.newInNewWindow (Mac Ctrl+Shift+Alt+`)
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (NewInNewWindow, lines 356-374); terminalMenus.ts (menu row lines 56-67; drop-down lines 802-807); V\src\vs\workbench\contrib\terminal\common\terminalStrings.ts (newInNewWindow); C bundle (no hit for newInNewWindow)
- Pieces:
  1. Picking Terminal > New Terminal Window opens a terminal in a new compact window that holds only that terminal. [stock only]
  2. Pressing Ctrl+Shift+Alt+` runs New Terminal Window (`workbench.action.terminal.newInNewWindow`) and opens the same compact window. [stock only]
  3. The Launch Profile list has a New Terminal Window row that does the same. [stock only]

#### PN-2.7 Create a terminal in the editor area

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the three commands and the toolbar plus button exist; no New Terminal row in the tab bar or empty-group menus, and no registration on the compact window title menu, searched the bundle and found none)
- What: Run Create New Terminal in Editor Area (or the version "to the Side") to open a terminal as an editor tab; the plus button on a terminal editor's toolbar adds another one in the same group, the tab bar and empty-group right-click menus carry a New Terminal row, and the default-location setting makes every new terminal open this way.
- Commands: workbench.action.createTerminalEditor; workbench.action.createTerminalEditorSide; workbench.action.createTerminalEditorSameGroup
- Menus: EditorTabsBarContext; EmptyEditorGroupContext; EditorTitle; CompactWindowEditorTitle
- Keys: none found
- Settings: terminal.integrated.defaultLocation
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 315-354); terminalMenus.ts (tab bar and empty group, lines 187-207; toolbar, lines 710-722); terminalEditor.ts (lines 155-190); terminalEditorService.ts (openEditor, lines 140-165); C bundle (the three ids; the only registration of createTerminalEditorSameGroup besides the command is the Pe.EditorTitle toolbar item; `Pe.CompactWindowEditorTitle` has 0 hits)
- Pieces:
  1. Running Create New Terminal in Editor Area opens a terminal as an editor tab.
  2. Running Create New Terminal in Editor Area to the Side opens the terminal as an editor tab in a group beside the current one.
  3. Clicking the plus button on a terminal editor's toolbar opens another terminal editor in the same group (`workbench.action.terminal.createTerminalEditorSameGroup`).
  4. Right-clicking the editor tab bar or an empty editor group shows a New Terminal row that opens a terminal in the editor area. [stock only]
  5. Setting `terminal.integrated.defaultLocation` set to `editor` makes every new terminal open as an editor tab.
  6. In stock the New Terminal button also shows on the title menu of a compact window. [stock only]

#### PN-2.8 Starting folder for new and split terminals

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (present, but the picker text reads "(Overriden)" with a typo that V has fixed)
- What: A new terminal in a multi-root window first shows a pick list "Select current working directory for new terminal" (a folder row says "(Overridden)" when the folder sets its own terminal start path), a single folder starts there, and a split follows the split setting (workspace root with a picker when there are several roots, the parent's initial folder, or the parent's current folder); a command also starts a terminal in a folder passed as an argument.
- Commands: workbench.action.terminal.newWithCwd
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.cwd; terminal.integrated.splitCwd
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (getCwdForSplit, lines 92-122; NewWithCwd, lines 984-1011; pickTerminalCwd, lines 1649-1692; New, lines 1240-1253); terminalService.ts (_resolveCwd, lines 1186-1200); terminalConfiguration.ts (cwd, lines 383-389; splitCwd, lines 490-500); C bundle (NLS keys workbench.action.terminal.newWorkspacePlaceholder and workbench.action.terminal.overriddenCwdDescription with the text "(Overriden) {0}"; id newWithCwd; both settings)
- Pieces:
  1. In a window with several workspace folders, creating a terminal first shows a pick list "Select current working directory for new terminal".
  2. A folder row says "(Overridden)" when that folder sets its own terminal start path. [stock only]
  3. In Cursor the same folder row says "(Overriden)" with a spelling mistake. [cursor only]
  4. With a single workspace folder, a new terminal starts in that folder without asking.
  5. Setting `terminal.integrated.cwd` sets an explicit start folder for new terminals.
  6. Setting `terminal.integrated.splitCwd` chooses the start folder of a split, with values `workspaceRoot` (with a picker when there are several roots), `initial` (the parent's initial folder) and `inherited` (the parent's current folder, default).
  7. Running `workbench.action.terminal.newWithCwd` with a folder passed as an argument starts a terminal in that folder.

#### PN-2.9 Workspace trust gate for new terminals

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the two messages exist; the bypass setting `terminal.integrated.allowInUntrustedWorkspace` is not in the bundle or the key list)
- What: In an untrusted workspace, starting a terminal asks for trust with the message "Creating a terminal process requires executing code", and if trust is refused the tab shows "Cannot launch a terminal process in an untrusted workspace"; a restricted setting can bypass the check.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.allowInUntrustedWorkspace
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (_createProcess, lines 1591-1606; _trust, lines 1948-1957); terminalConfiguration.ts (lines 674-680); C bundle (NLS keys workspaceNotTrustedCreateTerminal and terminal.requestTrust present; setting string absent)
- Pieces:
  1. Starting a terminal in an untrusted workspace asks for trust with the message "Creating a terminal process requires executing code".
  2. If trust is refused, the terminal tab shows "Cannot launch a terminal process in an untrusted workspace".
  3. Setting `terminal.integrated.allowInUntrustedWorkspace` lets terminals start in an untrusted workspace without asking, as on or off. [stock only]

#### PN-2.10 Run selected text or the active file in a terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick Terminal > Run Selected Text (or Run Active File) to send the selection, the current line or the file path to the active terminal and bring it forward, creating a terminal when none is open; a warning says "Only files on disk can be run in the terminal" for other file kinds, and the terminal and editor toolbars carry both as hidden-by-default buttons.
- Commands: workbench.action.terminal.runSelectedText; workbench.action.terminal.runActiveFile
- Menus: MenubarTerminalMenu; ViewTitle; EditorTitle
- Keys: none found
- Settings: terminal.integrated.focusAfterRun
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (RunSelectedText, lines 564-585; RunActiveFile, lines 587-612); terminalMenus.ts (menu rows lines 82-104; view toolbar lines 497-523; editor toolbar lines 736-757); terminalConfiguration.ts (focusAfterRun, lines 664-673); C bundle (ids runSelectedText and runActiveFile; NLS keys miRunSelectedText, miRunActiveFile and workbench.action.terminal.runActiveFile.noFile; setting focusAfterRun)
- Pieces:
  1. Picking Terminal > Run Selected Text sends the selected text to the active terminal and brings that terminal forward.
  2. With nothing selected, Run Selected Text sends the current line.
  3. Picking Terminal > Run Active File sends the file's path to the active terminal and brings that terminal forward.
  4. When no terminal is open, both commands create one first.
  5. Run Active File shows the warning "Only files on disk can be run in the terminal" for files that are not on disk.
  6. The terminal view toolbar and the editor toolbar carry Run Selected Text and Run Active File buttons that are hidden by default.
  7. Setting `terminal.integrated.focusAfterRun` chooses what gets focus after Run Selected Text, with values `terminal`, `accessible-buffer` and `none` (default).

#### PN-2.11 Name a terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (all commands and the first three settings present; `terminal.integrated.tabs.allowAgentCliTitle` is not in the bundle or the key list)
- What: Press F2 (Enter on Mac) on a tab, or pick Rename... from its right-click menu, to type a new name right in the tabs list (empty resets to the default title, Enter saves, Esc cancels), while Rename from the palette or from a terminal editor tab asks in an "Enter terminal name" box; the default title and the grey description next to it come from template settings such as process, task, working folder and separator.
- Commands: workbench.action.terminal.rename; workbench.action.terminal.renameActiveTab; workbench.action.terminal.renameWithArg
- Menus: TerminalTabContext; EditorTitleContext
- Keys: F2 on workbench.action.terminal.renameActiveTab (when terminalTabsFocus; Mac Enter)
- Settings: terminal.integrated.tabs.title; terminal.integrated.tabs.description; terminal.integrated.tabs.separator; terminal.integrated.tabs.allowAgentCliTitle
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (Rename and RenameActiveTab, lines 762-818; RenameWithArgs, lines 1013-1045; renameWithQuickPick, lines 1738-1754; validateTerminalName, lines 1504-1514); terminalTabsList.ts (inline input box, lines 438-512); terminalInstance.ts (rename, lines 2398-2420); terminalConfiguration.ts (title templates, lines 19-42; settings, lines 345-358; allowAgentCliTitle, lines 139-143); terminalMenus.ts (lines 591-600 and 674-681); C bundle (the ids; NLS keys emptyTerminalNameInfo, workbench.action.terminal.rename.prompt and terminalInputAriaLabel; the three template settings)
- Pieces:
  1. Pressing F2 (Enter on Mac) on a focused tab, or picking Rename... from its right-click menu, turns the title into a text box in the tabs list.
  2. In that box Enter saves the name, Esc cancels, and an empty name resets the title to its default.
  3. Running Rename from the Command Palette, or from the right-click menu of a terminal editor tab, asks for the name in an "Enter terminal name" input box.
  4. Running `workbench.action.terminal.renameWithArg` with a name passed as an argument renames the active terminal without asking.
  5. Setting `terminal.integrated.tabs.title` sets the template for the default title (default `${process}`), and `terminal.integrated.tabs.description` sets the template for the grey description next to it (default `${task}${separator}${local}${separator}${cwdFolder}`).
  6. Setting `terminal.integrated.tabs.separator` sets the separator text used by both templates (default " - ").
  7. Setting `terminal.integrated.tabs.allowAgentCliTitle` chooses whether agent command line tools may set the tab title, and when off the title template is used instead. [stock only]

#### PN-2.12 Change a terminal's icon or color

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick Change Icon... to open a floating grid of every icon with a filter box, and Change Color... to pick one of the terminal ANSI colors (or Reset to default) in a list; with several tabs selected the same icon or color is applied to all of them, and two settings set the default icon and color for every tab.
- Commands: workbench.action.terminal.changeIcon; workbench.action.terminal.changeIconActiveTab; workbench.action.terminal.changeColor; workbench.action.terminal.changeColorActiveTab
- Menus: TerminalTabContext; EditorTitleContext
- Keys: none found
- Settings: terminal.integrated.tabs.defaultIcon; terminal.integrated.tabs.defaultColor
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 710-760); terminalInstance.ts (changeIcon, lines 2422-2437; changeColor, lines 2439-2488); terminalIconPicker.ts; terminalIcon.ts; terminalMenus.ts (lines 601-620 and 683-700); terminalConfiguration.ts (lines 52-61); C bundle (the four ids; NLS keys workbench.action.terminal.changeIcon and workbench.action.terminal.changeColor; both settings; "Reset to default" string)
- Pieces:
  1. Picking Change Icon... on a tab's right-click menu opens a floating grid of every icon with a filter box.
  2. Picking Change Color... lists the terminal ANSI colors and a Reset to default row.
  3. With several tabs selected, the chosen icon or color is applied to all of them.
  4. Right-clicking a terminal editor tab also shows Change Icon... and Change Color...
  5. Setting `terminal.integrated.tabs.defaultIcon` sets the default icon for every terminal tab, as a codicon id.
  6. Setting `terminal.integrated.tabs.defaultColor` sets the default icon color for every terminal tab, as a theme color id.

#### PN-2.13 Status icons on a terminal tab

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A small icon with a color shows on the tab (as the icon when the list is narrow, as a badge letter when it is wide) for a bell, a lost connection to the shell ("Lost connection to process"), extensions that want the terminal relaunched (with a Relaunch Terminal action), extensions that changed its environment (with Show Environment Contributions), the shell integration level (Rich, Basic, No, Injection failed to activate) and a shell integration launch failure (with Learn more about shell integration and Open user settings); animated icons can be turned off.
- Commands: workbench.action.terminal.relaunch; workbench.action.terminal.showEnvironmentContributions; workbench.action.terminal.learnMore
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.tabs.enableAnimation; terminal.integrated.enableVisualBell; terminal.integrated.environmentChangesRelaunch
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalStatusList.ts; terminalInstance.ts (statuses at lines 862, 988, 1575 and 1788); environmentVariableInfo.ts (lines 28-95); terminalTabsList.ts (TabDecorationsProvider, lines 810-843; renderElement, lines 346-436); terminalConfiguration.ts (lines 66-71, 413-421 and 474-478); C bundle (status ids bell, disconnected, relaunch-needed, env-var-info-changes-active, shell-integration-info and shell-integration-attention-needed; NLS keys bellStatus, disconnectStatus, shellIntegration.learnMore, shellIntegration.openSettings, relaunchTerminalLabel; the shell integration level strings live in module terminalTooltip in Cursor and in terminalInstance in V)
- Pieces:
  1. When the shell rings the bell and the bell or `terminal.integrated.enableVisualBell` setting is on, the tab briefly shows a bell icon.
  2. When the connection to the shell is lost, the tab shows a status icon with the text "Lost connection to process".
  3. When extensions ask for the terminal to be relaunched, the tab shows a status with a Relaunch Terminal action (`workbench.action.terminal.relaunch`).
  4. When extensions have changed the terminal's environment, the tab shows a status with a Show Environment Contributions action (`workbench.action.terminal.showEnvironmentContributions`).
  5. The tab shows a status for the shell integration level, with the texts Rich, Basic, No and Injection failed to activate.
  6. When shell integration fails to launch, the tab shows a warning with Learn more about shell integration and Open user settings links (`workbench.action.terminal.learnMore`).
  7. A status shows as the tab's icon when the tabs list is narrow and as a colored badge letter when it is wide.
  8. Setting `terminal.integrated.tabs.enableAnimation` chooses whether animated status icons animate, as on or off.
  9. Setting `terminal.integrated.environmentChangesRelaunch` chooses whether the relaunch status shows when extensions change the environment, as on or off.

#### PN-2.14 Hover card on a terminal tab

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Hovering a tab (or the single tab in the view title) shows a card with the title, the process ID, the command line, the text of every status, and action links from those statuses, ending in a Show Details or Hide Details link that adds longer text and is remembered; Ctrl+K Ctrl+I puts focus into the card.
- Commands: workbench.action.terminal.focusHover
- Menus: none found
- Keys: Ctrl+K Ctrl+I on workbench.action.terminal.focusHover (when terminalTabsFocus or terminalFocus)
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalTooltip.ts (lines 1-76); terminalActions.ts (FocusHover, lines 1342-1352); terminalTabsList.ts (hoverInfo, lines 370-373); terminalTabbedView.ts (focusHover, lines 604-616); V\src\vs\workbench\contrib\terminal\common\terminalStorageKeys.ts (TabsShowDetailed); C bundle (id focusHover; NLS keys showDetails, hideDetails, shellProcessTooltip.processId, shellProcessTooltip.commandLine; storage key terminal.integrated.tabs.showDetailed)
- Pieces:
  1. Hovering a tab shows a card with the title, the process ID and the command line.
  2. The card lists the text of every status on the tab and shows action links from those statuses.
  3. The card ends with a Show Details or Hide Details link that adds longer text, and the choice is remembered.
  4. Hovering the single active-terminal button in the view title shows the same card.
  5. Pressing Ctrl+K Ctrl+I while the tabs list or a terminal has focus moves focus into the card (`workbench.action.terminal.focusHover`).

#### PN-2.15 Side tabs list and when it hides

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Beside the terminal, a list shows one row per terminal (icon, title, description, a split mark of ┌ ├ └ for panes in one group, and inline Split and Kill buttons when the list is wide), and by default it appears only once there is more than one terminal and stays open while the mouse is over it; the empty area of the list has a right-click menu with New Terminal With Profile, New Terminal and Hide Tabs, and turning tabs off replaces the list with a drop-down of numbered groups plus a Show Tabs entry.
- Commands: workbench.action.terminal.switchTerminal; workbench.action.terminal.splitActiveTab; workbench.action.terminal.killActiveTab
- Menus: TerminalTabEmptyAreaContext; ViewTitle
- Keys: none found
- Settings: terminal.integrated.tabs.enabled; terminal.integrated.tabs.hideCondition
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalTabbedView.ts (shouldShowTabs, lines 167-210; empty area menu, lines 432-455; getTabActions, lines 550-562); terminalTabsList.ts (renderElement, lines 346-436; fillActionBar, lines 525-548); terminalView.ts (SwitchTerminalActionViewItem, lines 359-402); terminalActions.ts (SwitchTerminal, lines 1403-1445; switchTerminalShowTabsTitle, line 68); terminalGroupService.ts (getGroupLabels, lines 500-504); terminalConfiguration.ts (lines 63-83); C bundle (ids switchTerminal, splitActiveTab, killActiveTab; NLS keys showTerminalTabs, hideTabs, terminals, terminal.tabs; both settings)
- Pieces:
  1. Beside the terminal a list shows one row per terminal with its icon, title and description.
  2. Panes in one split group show a split mark (┌ ├ └) in front of their rows.
  3. When the list is wide, each row has inline Split and Kill buttons.
  4. By default the list appears only once there is more than one terminal, and it stays open while the mouse is over it.
  5. Setting `terminal.integrated.tabs.hideCondition` chooses when the list hides, with values `never`, `singleTerminal` (default) and `singleGroup`.
  6. Right-clicking the empty area of the list shows New Terminal With Profile, New Terminal and Hide Tabs, and Hide Tabs turns the list off.
  7. Setting `terminal.integrated.tabs.enabled` off replaces the list with a drop-down of numbered groups plus a Show Tabs entry that turns the list back on.

#### PN-2.16 Place and size the tabs list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click the empty area of the tabs list and pick Move Tabs Left or Move Tabs Right to put it on the other side of the terminal, and drag the divider to resize it, where the list snaps to a narrow icon-only strip (46 px) or a wide text view (at least 80 px, at most 500 px) and a double click on the divider fits the width to the longest title or collapses it again; the width is remembered separately for a bottom or top panel and a left or right panel.
- Commands: none found
- Menus: TerminalTabEmptyAreaContext
- Keys: none found
- Settings: terminal.integrated.tabs.location
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalTabbedView.ts (lines 118-135, 217-338 and 550-562); terminalTabsList.ts (TerminalTabsListSizes, lines 62-70); V\src\vs\workbench\contrib\terminal\common\terminalStorageKeys.ts (TabsListWidthHorizontal, TabsListWidthVertical); terminalConfiguration.ts (lines 108-117); C bundle (storage keys tabs-list-width-vertical and tabs-list-width-horizontal; NLS keys moveTabsLeft and moveTabsRight; setting tabs.location)
- Pieces:
  1. Right-clicking the empty area of the tabs list and picking Move Tabs Left or Move Tabs Right puts the list on that side of the terminal.
  2. Setting `terminal.integrated.tabs.location` chooses the side of the list, with values `left` and `right` (default).
  3. Dragging the divider resizes the list, and it snaps between a narrow icon-only strip (46 px) and a wide text view (80 px to 500 px).
  4. Double-clicking the divider fits the list width to the longest title, or collapses it again.
  5. The width is remembered separately for a bottom or top panel and for a left or right panel.

#### PN-2.17 Compact toolbar when the tabs list is hidden

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With the tabs list hidden or in its narrow strip, the view title shows the active terminal as one button (icon, title and description in its color; click to focus, drag to move it, hover for the card) and, next to the plus button, Split and Kill buttons, with settings choosing when each shows (always, single terminal, single terminal or narrow, never); three more buttons (Clear Terminal, Run Active File, Run Selected Text) can be switched on from the toolbar menu.
- Commands: workbench.action.terminal.focus; workbench.action.terminal.split; workbench.action.terminal.kill; workbench.action.terminal.clear; workbench.action.terminal.switchTerminal
- Menus: ViewTitle
- Keys: none found
- Settings: terminal.integrated.tabs.showActiveTerminal; terminal.integrated.tabs.showActions; terminal.integrated.tabs.enabled; terminal.integrated.tabs.hideCondition
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (ViewTitle, lines 381-551); terminalView.ts (SingleTerminalTabActionViewItem, lines 405-565); V\src\vs\workbench\contrib\terminal\common\terminalContextKey.ts (shouldShowViewInlineActions, lines 147-170); terminalConfiguration.ts (lines 84-107); C bundle (ids focus, split, kill, clear; the two settings; the ViewTitle menu code that tests config.terminal.integrated.tabs.showActiveTerminal and terminalGroupCount; isHiddenByDefault on the toolbar items)
- Pieces:
  1. With the tabs list hidden or narrow, the view title shows the active terminal as one button with its icon, title and description in its color.
  2. Clicking that button focuses the terminal, dragging it moves the terminal, and hovering it shows the hover card.
  3. Next to the plus button the view title shows Split and Kill buttons.
  4. Setting `terminal.integrated.tabs.showActiveTerminal` chooses when the active terminal button shows, with values `always`, `singleTerminal`, `singleTerminalOrNarrow` (default) and `never`.
  5. Setting `terminal.integrated.tabs.showActions` chooses when the Split and Kill buttons show, with values `always`, `singleTerminal`, `singleTerminalOrNarrow` (default) and `never`.
  6. Clear Terminal, Run Active File and Run Selected Text buttons are hidden by default and can be switched on from the view toolbar's menu.

#### PN-2.18 Select and focus terminals in the tabs list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking a row selects it and, by default, a double click moves focus into that terminal (a setting makes a single click do it), Ctrl or Shift click selects several for one menu action, Alt click splits that terminal, a middle click kills it, a double click on empty space creates a terminal, and a keyboard shortcut jumps focus into the list.
- Commands: workbench.action.terminal.focusTabs
- Menus: TerminalTabContext
- Keys: Ctrl+Shift+\ on workbench.action.terminal.focusTabs (when terminalTabsFocus or terminalFocus)
- Settings: terminal.integrated.tabs.focusMode
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalTabsList.ts (mouse handlers, lines 158-221; middle click, lines 395-403); terminalTabbedView.ts (focusTabs, lines 573-582); terminalActions.ts (FocusTabs, lines 516-526; getSelectedViewInstances, lines 1471-1503); terminalConfiguration.ts (lines 129-138); C bundle (id focusTabs; NLS key focus.tabsView; setting tabs.focusMode)
- Pieces:
  1. Clicking a row selects it, and by default a double click moves focus into that terminal.
  2. Setting `terminal.integrated.tabs.focusMode` chooses how a row focuses its terminal, with values `singleClick` and `doubleClick` (default).
  3. Ctrl click or Shift click selects several rows so one menu action applies to all of them.
  4. Alt click on a row splits that terminal.
  5. Middle click on a row kills that terminal.
  6. Double click on empty space in the list creates a new terminal.
  7. Right-clicking a row opens the tab menu for the selected rows.
  8. Pressing Ctrl+Shift+\ moves focus into the tabs list (`workbench.action.terminal.focusTabs`) when a terminal or the list has focus.

#### PN-2.19 Reorder tabs and drop things onto them

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Drag a row to reorder terminals and groups (dropping on another row moves the group there, dropping on empty space sends it to the end), hold a drag over a row for half a second to switch to it, drop a file or folder from the Explorer or the desktop onto a row to paste its path into that terminal, and drop a terminal editor tab or a terminal from another window onto the list to bring it into the panel.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalTabsList.ts (drag and drop, lines 640-806); terminalTabbedView.ts (empty area drop, lines 370-410 and 477-547); V\src\vs\workbench\contrib\terminal\browser\terminal.contribution.ts (drag and drop contribution, lines 83-102); terminalUri.ts; C bundle (code for sendPath, moveGroupToEnd and requestDetachInstance present)
- Pieces:
  1. Dragging a row onto another row moves the dragged group to that row's position.
  2. Dropping a dragged row on empty space in the list sends it to the end.
  3. Holding a drag over a row for half a second switches to that terminal.
  4. Dropping a file or folder from the Explorer or the desktop onto a row pastes its path into that terminal.
  5. Dropping a terminal editor tab onto the list brings that terminal into the panel.
  6. Dropping a terminal from another window onto the list brings it into this window's panel.

#### PN-2.20 Switch between terminal groups

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+PageDown or Ctrl+PageUp (Cmd+Shift+] or Cmd+Shift+[ on Mac) in a terminal to make the next or previous group active and bring the panel forward, and nine unbound commands jump to the first through ninth group so you can assign your own keys.
- Commands: workbench.action.terminal.focusNext; workbench.action.terminal.focusPrevious; workbench.action.terminal.focusAtIndex1; workbench.action.terminal.focusAtIndex2; workbench.action.terminal.focusAtIndex3; workbench.action.terminal.focusAtIndex4; workbench.action.terminal.focusAtIndex5; workbench.action.terminal.focusAtIndex6; workbench.action.terminal.focusAtIndex7; workbench.action.terminal.focusAtIndex8; workbench.action.terminal.focusAtIndex9
- Menus: none found
- Keys: Ctrl+PageDown on workbench.action.terminal.focusNext and Ctrl+PageUp on workbench.action.terminal.focusPrevious (when terminalFocus and not terminalEditorFocus; Mac Cmd+Shift+] and Cmd+Shift+[)
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (FocusNext and FocusPrevious, lines 528-562); terminalCommands.ts (lines 1-28); C bundle (ids focusNext, focusPrevious and focusAtIndex1 to focusAtIndex9; the focusAtIndex registration loop is present)
- Pieces:
  1. Pressing Ctrl+PageDown in a terminal makes the next group active and brings the panel forward (Mac Cmd+Shift+]).
  2. Pressing Ctrl+PageUp in a terminal makes the previous group active and brings the panel forward (Mac Cmd+Shift+[).
  3. In a terminal editor tab, Ctrl+PageDown and Ctrl+PageUp are not taken by these commands.
  4. Nine commands `workbench.action.terminal.focusAtIndex1` to `workbench.action.terminal.focusAtIndex9` have no default key and jump to the first through ninth group once you assign keys.

#### PN-2.21 Split a terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+Shift+5 in a terminal, click the Split button, pick Terminal > Split Terminal, use Split on a tab's right-click menu, or Alt click the plus button to add a second terminal next to the active one inside the same group (side by side when the panel is at the bottom or top, stacked when it is on a side); in the editor area a split opens another terminal editor to the side.
- Commands: workbench.action.terminal.split; workbench.action.terminal.splitActiveTab; workbench.action.terminal.splitInActiveWorkspace
- Menus: MenubarTerminalMenu; ViewTitle; TerminalTabContext; TerminalEditorInstanceContext; EditorTitle
- Keys: Ctrl+Shift+5 on workbench.action.terminal.split (when terminalFocus; Mac Cmd+\ with Ctrl+Shift+5 as the second key); Ctrl+Shift+5 on workbench.action.terminal.splitActiveTab (when terminalTabsFocus; Mac Cmd+\)
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (Split, lines 1053-1083; SplitActiveTab, lines 1085-1111; SplitInActiveWorkspace, lines 1184-1192); terminalMenus.ts (lines 68-80, 440-450, 560-568, 210-219 and 715-722); terminalGroup.ts (orientation and split, lines 483-552); terminalService.ts (_splitTerminal, lines 1203-1229); terminalEditorService.ts (splitInstance, lines 215-245); terminalView.ts (panel-only split, lines 252-272); C bundle (ids split, splitActiveTab, splitInActiveWorkspace; NLS keys miSplitTerminal and splitTerminal)
- Pieces:
  1. Pressing Ctrl+Shift+5 in a terminal (Mac Cmd+\) adds a second terminal next to the active one inside the same group.
  2. Clicking the Split button in the view title or picking Terminal > Split Terminal does the same.
  3. Picking Split on a tab's right-click menu, or pressing Ctrl+Shift+5 with the tabs list focused, splits that tab's terminal.
  4. Alt clicking the plus button splits the active terminal.
  5. Panes sit side by side when the panel is at the bottom or top and stacked when it is on a side.
  6. In the editor area a split opens another terminal editor to the side.
  7. In a remote window, `workbench.action.terminal.splitInActiveWorkspace` splits the terminal using the active workspace.

#### PN-2.22 Move focus between split panes

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With a split group active, Alt plus the Left or Up arrow moves focus to the previous terminal in the group and Alt plus the Right or Down arrow to the next (Alt+Cmd plus arrows on Mac), and the panel is shown if it was hidden.
- Commands: workbench.action.terminal.focusPreviousPane; workbench.action.terminal.focusNextPane
- Menus: none found
- Keys: Alt+Left and Alt+Up on workbench.action.terminal.focusPreviousPane; Alt+Right and Alt+Down on workbench.action.terminal.focusNextPane (when terminalFocus and a split terminal is active; Mac Alt+Cmd with the arrow)
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 405-445); C bundle (ids focusPreviousPane and focusNextPane; NLS keys workbench.action.terminal.focusPreviousPane and workbench.action.terminal.focusNextPane)
- Pieces:
  1. With a split group active, Alt+Left or Alt+Up moves focus to the previous terminal in the group (Mac Alt+Cmd with the arrow).
  2. With a split group active, Alt+Right or Alt+Down moves focus to the next terminal in the group (Mac Alt+Cmd with the arrow).
  3. These commands show the panel when it was hidden.

#### PN-2.23 Resize split panes

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Drag the divider between panes, or press the resize shortcuts with a terminal focused, to make the active pane larger toward the chosen side (Linux has Ctrl+Shift with Left or Right, the Mac has Ctrl+Cmd with all four arrows, Windows has no default key).
- Commands: workbench.action.terminal.resizePaneLeft; workbench.action.terminal.resizePaneRight; workbench.action.terminal.resizePaneUp; workbench.action.terminal.resizePaneDown
- Menus: none found
- Keys: Ctrl+Shift+Left on workbench.action.terminal.resizePaneLeft and Ctrl+Shift+Right on workbench.action.terminal.resizePaneRight (Linux only, when terminalFocus); Ctrl+Cmd+Left, Right, Up and Down on the four resize commands (Mac only, when terminalFocus)
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 447-495); terminalGroup.ts (SplitPaneContainer.resizePane, lines 66-110; resizePane, lines 585-632); C bundle (the four ids; NLS keys workbench.action.terminal.resizePaneLeft to resizePaneDown)
- Pieces:
  1. Dragging the divider between two panes resizes them.
  2. On Linux, Ctrl+Shift+Left and Ctrl+Shift+Right with a terminal focused make the active pane larger toward that side.
  3. On Mac, Ctrl+Cmd with the Left, Right, Up or Down arrow makes the active pane larger toward that side.
  4. On Windows the four commands `workbench.action.terminal.resizePaneLeft`, `resizePaneRight`, `resizePaneUp` and `resizePaneDown` have no default key and can be given one.

#### PN-2.24 Unsplit a pane or join terminals into a group

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a pane's tab and pick Unsplit Terminal to move it into a group of its own, or select two or more tabs and pick Join Terminals (or run Join Terminals... from the palette and choose one) to merge separate groups into one split group, with messages when there are too few terminals or all are already joined.
- Commands: workbench.action.terminal.unsplit; workbench.action.terminal.join; workbench.action.terminal.joinActiveTab
- Menus: TerminalTabContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (Unsplit, lines 1113-1123; JoinActiveTab, lines 1125-1135; Join, lines 1137-1182); terminalMenus.ts (lines 632-651); terminalGroupService.ts (unsplitInstance, joinInstances); C bundle (ids unsplit, join, joinActiveTab; NLS keys unsplitTerminal, workbench.action.terminal.joinInstance, workbench.action.terminal.join.insufficientTerminals, workbench.action.terminal.join.onlySplits)
- Pieces:
  1. Picking Unsplit Terminal on a pane's tab right-click menu moves that pane into a group of its own.
  2. Selecting two or more tabs and picking Join Terminals on the right-click menu merges their groups into one split group.
  3. Running Join Terminals... from the Command Palette asks you to choose a terminal and joins it with the active one.
  4. With fewer than two terminals, a warning says "Insufficient terminals for the join action".
  5. When all terminals are already in one split group, a warning says "All terminals are joined already".

#### PN-2.25 Move a terminal into the editor area or a new window

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick Move Terminal into Editor Area on a tab's right-click menu (or from the palette) to turn the selected terminals into editor tabs, pick Move Terminal into New Window to put them in a window of their own, or drag a tab (or the single tab in the view title) into the editor area to do the same.
- Commands: workbench.action.terminal.moveToEditor; workbench.action.terminal.moveIntoNewWindow
- Menus: TerminalTabContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 376-391); terminalMenus.ts (lines 568-590); terminalService.ts (moveToEditor and moveIntoNewEditor, lines 790-815); terminal.contribution.ts (drag and drop data, lines 83-102); V\src\vs\workbench\contrib\terminal\common\terminalStrings.ts; C bundle (ids moveToEditor and moveIntoNewWindow; NLS keys moveToEditor and moveIntoNewWindow)
- Pieces:
  1. Picking Move Terminal into Editor Area on a tab's right-click menu turns the selected terminals into editor tabs.
  2. Running Move Terminal into Editor Area from the Command Palette moves the active terminal into an editor tab.
  3. Picking Move Terminal into New Window puts the selected terminals in a window of their own.
  4. Dragging a tab from the tabs list into the editor area moves that terminal into an editor tab.
  5. Dragging the single tab button in the view title into the editor area does the same.

#### PN-2.26 Move a terminal back into the panel

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a terminal editor tab and pick Move Terminal into Panel (or run it from the palette) to return the terminal to the Terminal view, or drag the editor tab onto the tabs list.
- Commands: workbench.action.terminal.moveToTerminalPanel
- Menus: EditorTitleContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 393-403); terminalMenus.ts (lines 665-673); terminalService.ts (moveToTerminalView, lines 817-850); C bundle (id moveToTerminalPanel; NLS key workbench.action.terminal.moveToTerminalPanel; the EditorTitleContext registration with the vscode-terminal scheme check)
- Pieces:
  1. Picking Move Terminal into Panel on a terminal editor tab's right-click menu returns the terminal to the Terminal view.
  2. Running Move Terminal into Panel from the Command Palette does the same for the active terminal editor.
  3. Dragging the terminal editor tab onto the tabs list moves it back into the panel.

#### PN-2.27 Terminal as an editor tab

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (present, but the toolbar buttons are not registered on the compact window title menu)
- What: A terminal in the editor area is an editor tab with the terminal's icon, color, status decorations, title and description, a toolbar with plus (Alt click splits to the side) and the profile drop-down, and a right-click menu with Split, New Terminal, Kill Terminal, Copy, Copy as HTML, Paste, Clear, Select All and Toggle Size to Content Width; Ctrl+W closes it, and terminal editors are saved with the window layout and come back after a reload.
- Commands: workbench.action.terminal.killEditor
- Menus: TerminalEditorInstanceContext; EditorTitle; EditorTitleContext
- Keys: Ctrl+W on workbench.action.terminal.killEditor (when terminalFocus and terminalEditorFocus; on Windows Ctrl+F4 with Ctrl+W as the second key)
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalEditor.ts; terminalEditorInput.ts (lines 30-170); terminalEditorService.ts; terminalEditorSerializer.ts; terminalActions.ts (KillEditor, lines 1305-1316); terminalMenus.ts (lines 209-305, 665-709 and 710-781); terminal.contribution.ts (editor pane and serializer, lines 71-82); C bundle (id killEditor; editor type id workbench.editors.terminal; menu id Pe.TerminalEditorInstanceContext; no Pe.CompactWindowEditorTitle)
- Pieces:
  1. A terminal in the editor area is an editor tab showing the terminal's icon, color, status decorations, title and description.
  2. The tab's toolbar has a plus button, where Alt click splits to the side, and the Launch Profile drop-down.
  3. Right-clicking inside a terminal editor shows Split, New Terminal, Kill Terminal, Copy, Copy as HTML, Paste, Clear, Select All and Toggle Size to Content Width.
  4. Pressing Ctrl+W in a focused terminal editor kills it (`workbench.action.terminal.killEditor`), and on Windows Ctrl+F4 is the primary key with Ctrl+W as the second.
  5. Terminal editors are saved with the window layout and come back after a reload.
  6. In stock the terminal editor toolbar buttons also show on the title bar of a compact window. [stock only]

#### PN-2.28 Kill terminals

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Click the trash button in the view title or on a tab row, press Delete on a focused tab, middle-click a tab, or pick Kill Terminal from a terminal's or tab's right-click menu to end a terminal and its shell (all selected tabs go together), and run Kill All Terminals from the palette to end every one.
- Commands: workbench.action.terminal.kill; workbench.action.terminal.killViewOrEditor; workbench.action.terminal.killActiveTab; workbench.action.terminal.killAll
- Menus: ViewTitle; TerminalInstanceContext; TerminalTabContext
- Keys: Delete on workbench.action.terminal.killActiveTab (when terminalTabsFocus; Mac Cmd+Backspace or Delete)
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 1265-1340); terminalMenus.ts (lines 110-119, 451-463 and 652-662); terminalTabsList.ts (inline Kill, lines 541-544; middle click, lines 395-403); terminalService.ts (safeDisposeTerminal, lines 430-444); C bundle (ids kill, killViewOrEditor, killActiveTab, killAll; NLS keys workbench.action.terminal.kill, workbench.action.terminal.killAll, killTerminal and killTerminal.short)
- Pieces:
  1. Clicking the trash button in the view title kills the active terminal and its shell.
  2. Clicking the trash button on a tab row kills that terminal.
  3. Pressing Delete on a focused tab kills the terminal (Mac Cmd+Backspace or Delete).
  4. Middle-clicking a tab kills its terminal.
  5. Picking Kill Terminal from a terminal's right-click menu or a tab's right-click menu kills the terminal.
  6. With several tabs selected, killing from a tab action ends all of the selected terminals.
  7. Running Kill All Terminals from the Command Palette ends every terminal.

#### PN-2.29 Confirm before killing a busy terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a terminal has a running child process, killing a panel terminal asks "Do you want to terminate the active terminal session?" with a Terminate button, while a terminal editor tab is marked as changed and asks "Do you want to terminate running processes?" when closed; the setting chooses where it asks (never, editor, panel, always; editor by default) and a second setting lists process names that do not count as running work.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.confirmOnKill; terminal.integrated.ignoreProcessNames
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalService.ts (safeDisposeTerminal, lines 430-444; _showTerminalCloseConfirmation, lines 935-948); terminalEditorInput.ts (showConfirm and confirm, lines 96-118); terminalConfiguration.ts (confirmOnKill, lines 401-411); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (ignoreProcessNames, lines 413-428); C bundle (NLS keys terminalService.terminalCloseConfirmationSingular, confirmDirtyTerminal.message and confirmDirtyTerminal.detail; both settings)
- Pieces:
  1. Killing a panel terminal that has a running child process asks "Do you want to terminate the active terminal session?" with a Terminate button.
  2. A terminal editor tab with a running child process is marked as changed and asks "Do you want to terminate running processes?" when closed.
  3. Setting `terminal.integrated.confirmOnKill` chooses where the confirmation shows, with values `never`, `editor` (default), `panel` and `always`.
  4. Setting `terminal.integrated.ignoreProcessNames` lists process names that do not count as running work.

#### PN-2.30 Confirm before closing the window with terminals open

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With the setting on, closing the window or quitting while terminals are running asks "Do you want to terminate the N active terminal sessions?" (or the single-session wording) with a Terminate button, either always or only when a terminal has child processes, and it is skipped when the sessions will be kept for a reload; off by default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.confirmOnExit
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalService.ts (shutdown handling, lines 630-668; _showTerminalCloseConfirmation, lines 935-948); terminalConfiguration.ts (confirmOnExit, lines 390-400); C bundle (NLS keys terminalService.terminalCloseConfirmationPlural and terminate; setting confirmOnExit)
- Pieces:
  1. With `terminal.integrated.confirmOnExit` on, closing the window or quitting while terminals are running asks "Do you want to terminate the N active terminal sessions?" with a Terminate button.
  2. With a single running session the question uses the single-session wording.
  3. Setting `terminal.integrated.confirmOnExit` chooses when it asks, with values `never` (default), `always` and `hasChildProcesses`.
  4. The question is skipped when the sessions will be kept for a reload.

#### PN-2.31 Hide the Terminal view when it is empty

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Closing the last terminal hides the Terminal view when it is the only view in its panel tab (a setting can keep it open), and at startup the view can be hidden always, hidden only when no sessions were restored, or never hidden, in which case a first terminal is created.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.hideOnLastClosed; terminal.integrated.hideOnStartup
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalService.ts (lines 205-215); terminalView.ts (_initializeTerminal, lines 144-186); terminalConfiguration.ts (lines 559-574); C bundle (both settings)
- Pieces:
  1. Closing the last terminal hides the Terminal view when it is the only view in its panel tab.
  2. Setting `terminal.integrated.hideOnLastClosed` off keeps the empty Terminal view open after the last terminal closes.
  3. At startup with the default `terminal.integrated.hideOnStartup` value of `never`, the Terminal view stays visible and a first terminal is created when there are no sessions to restore.
  4. Setting `terminal.integrated.hideOnStartup` chooses whether the view hides at startup, with values `never` (default), `whenEmpty` (hide only when no sessions were restored) and `always` (hide even when sessions were restored).

#### PN-2.32 Process exit and launch failure notices

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a shell exits with an error, an error notification says "The terminal process ... terminated with exit code: N" (skipped after Ctrl+D and when the alert setting is off), a shell that cannot start says "The terminal process failed to launch" with an Open Help action, a terminal set to wait on exit keeps its tab until you press a key, and Relaunch Active Terminal restarts the shell in the same tab.
- Commands: workbench.action.terminal.launchHelp; workbench.action.terminal.relaunch
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.showExitAlert
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (exit handling, lines 1735-1783; relaunch, lines 1913-1925; messages, lines 2845-2890); terminalActions.ts (TerminalLaunchHelpAction, lines 124-135; Relaunch, lines 1047-1051); terminalConfiguration.ts (lines 479-483); C bundle (ids launchHelp and relaunch; NLS keys terminalLaunchHelp, launchFailed.exitCodeOnly, terminated.exitCodeOnly and workbench.action.terminal.relaunch; setting showExitAlert)
- Pieces:
  1. When a shell exits with an error, an error notification says "The terminal process ... terminated with exit code: N".
  2. No exit notification shows after you end the shell yourself with Ctrl+D.
  3. Setting `terminal.integrated.showExitAlert` chooses whether the exit notification shows, as on (default) or off.
  4. A shell that cannot start shows "The terminal process failed to launch" with an Open Help action (`workbench.action.terminal.launchHelp`).
  5. A terminal set to wait on exit keeps its tab until you press a key.
  6. Running Relaunch Active Terminal restarts the shell in the same tab (`workbench.action.terminal.relaunch`).

#### PN-2.33 Restore terminals after a reload

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: After a window reload the tabs, groups, split layout, titles, icons, colors and a number of scrollback lines come back and the shells reattach to their running processes, after quitting or closing the window a setting decides whether content and processes are recreated on next open, a lost shell connection restarts the terminal with "Restarting the terminal because the connection to the shell process was lost...", and background tasks with problem matchers can reconnect too.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enablePersistentSessions; terminal.integrated.persistentSessionReviveProcess; terminal.integrated.persistentSessionScrollback; task.reconnection
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalService.ts (backend start, lines 277-330; reconnect and recreate groups, lines 452-560; shutdown and revive, lines 620-700; saved title and icon, lines 738-760); terminalProcessManager.ts (line 570); terminalEditorSerializer.ts; terminalMainContribution.ts; terminalConfiguration.ts (lines 542-558); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (lines 401-406); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (task.reconnection, line 551); C bundle (the three terminal settings and task.reconnection; NLS key ptyHostRelaunch; restoredGroupCount code)
- Pieces:
  1. After a window reload the tabs, groups, split layout, titles, icons and colors come back.
  2. After a window reload a number of scrollback lines come back and the shells reattach to their running processes.
  3. Setting `terminal.integrated.enablePersistentSessions` chooses whether terminal sessions persist across window reloads, as on (default) or off.
  4. Setting `terminal.integrated.persistentSessionReviveProcess` chooses when content and processes are recreated after quitting or closing the window, with values `onExit` (default), `onExitAndWindowClose` and `never`.
  5. A lost shell connection restarts the terminal with the message "Restarting the terminal because the connection to the shell process was lost...".
  6. Setting `task.reconnection` lets background tasks with problem matchers reconnect after a reload, as on or off.

#### PN-2.34 Detach and attach terminal sessions

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run Detach Session to close the active terminal's tab while its shell keeps running in the background terminal host, and Attach to Session to pick one of those unattached shells (shown with its title, folder and process ID) and open it in a new tab, with a message when there is none to attach.
- Commands: workbench.action.terminal.detachSession; workbench.action.terminal.attachToSession
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 820-869); terminalInstance.ts (detachProcessAndDispose, line 1353); terminalService.ts (isAttachedToTerminal); C bundle (ids detachSession and attachToSession; NLS keys workbench.action.terminal.detachSession, workbench.action.terminal.attachToSession and noUnattachedTerminals; code isAttachedToTerminal present)
- Pieces:
  1. Running Detach Session closes the active terminal's tab while its shell keeps running in the background terminal host.
  2. Running Attach to Session opens a pick list of unattached shells, each shown with its title, folder and process ID.
  3. Choosing a shell in that list opens it in a new tab.
  4. When there is nothing to attach, a message says "There are no unattached terminals to attach to".

#### PN-2.35 Task state on a terminal tab

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The tab of a task's terminal shows the task state as an icon (a spinner while it runs, a check on success, and error, warning or info icons when the problem matcher finds problems, with "and waiting..." text for idle background tasks), with hover text such as "Task is running" or "Task has errors", and a marker line notes where the detected errors for the run begin.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.tabs.enableAnimation
- Source: V\src\vs\workbench\contrib\tasks\browser\taskTerminalStatus.ts (lines 29-66); terminalTaskSystem.ts (status manager, line 238); terminalStatusList.ts; terminalConfiguration.ts (lines 68-71); C bundle (all ten NLS keys of module taskTerminalStatus, among them taskTerminalStatus.active, taskTerminalStatus.errors and task.watchFirstError)
- Pieces:
  1. The tab of a task's terminal shows a spinner while the task runs.
  2. The tab shows a check icon when the task succeeds.
  3. The tab shows an error, warning or info icon when the problem matcher finds problems.
  4. An idle background task shows text ending in "and waiting..." on its tab.
  5. Hovering the task state shows text such as "Task is running" or "Task has errors".
  6. A marker line in the terminal notes where the detected errors for the current run begin.
  7. With `terminal.integrated.tabs.enableAnimation` off, the running spinner does not animate.

#### PN-2.36 Task terminal reuse

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Running a task reuses an idle terminal according to the task's presentation options in tasks.json (panel shared, dedicated or new, plus group, clear, close and showReuseMessage), and a finished task terminal stays open with "Terminal will be reused by tasks, press any key to close it." or "Press any key to close the terminal." instead of vanishing; a reused terminal scrolls to the bottom and is cleared when the task asks for it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\terminalTaskSystem.ts (reuse, lines 1554-1606; wait on exit text, lines 2008-2025); V\src\vs\workbench\contrib\tasks\common\tasks.ts (PanelKind, presentation options); C bundle (NLS keys reuseTerminal and closeTerminal in module terminalTaskSystem; the strings showReuseMessage and "dedicated" in the task schema)
- Pieces:
  1. Running a task reuses an idle terminal according to the task's `presentation.panel` option in tasks.json, with values `shared`, `dedicated` and `new`.
  2. A finished task terminal stays open and prints "Terminal will be reused by tasks, press any key to close it." when `presentation.showReuseMessage` is on.
  3. With panel `new`, a finished task terminal stays open and prints "Press any key to close the terminal."
  4. Setting `presentation.close` to true closes the task's terminal when the task exits.
  5. Setting `presentation.clear` to true clears the terminal before the task runs, and a reused terminal scrolls to the bottom.
  6. Setting `presentation.group` runs the task in a named terminal group using split panes.

#### PN-2.37 Rerun a task from its terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a terminal that belongs to a task, pick Rerun Task from its right-click menu, or press Ctrl+Shift+R, to run that task again.
- Commands: workbench.action.tasks.rerunForActiveTerminal
- Menus: TerminalInstanceContext
- Keys: Ctrl+Shift+R on workbench.action.tasks.rerunForActiveTerminal (when terminalFocus; Mac Ctrl+Shift+R)
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (lines 584-603); V\src\vs\workbench\contrib\tasks\common\tasks.ts (RerunForActiveTerminalCommandId, line 1441); C bundle (id rerunForActiveTerminal; NLS key rerunTask in module terminalTaskSystem; context key for an active task terminal)
- Pieces:
  1. Picking Rerun Task on the right-click menu of a task's terminal runs that task again.
  2. Pressing Ctrl+Shift+R with a task's terminal focused runs that task again (`workbench.action.tasks.rerunForActiveTerminal`).
  3. The Rerun Task row and key apply only when the focused terminal belongs to a task.

#### PN-2.38 Shell integration for new terminals

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the enabled setting is present; the timeout setting is not in the bundle or the key list)
- What: By default each new terminal gets a startup script injected for bash, zsh, fish and PowerShell (and Git Bash on Windows) so the terminal learns about commands and folders; the setting only affects terminals created afterwards, a wait-time setting decides how long to wait before calling it absent, and a shell that fails to start with the script is relaunched without it with a warning on the tab and links to learn more or open the setting.
- Commands: workbench.action.terminal.learnMore
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.shellIntegration.enabled; terminal.integrated.shellIntegration.timeout
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 610-636); terminalInstance.ts (level text, lines 950-991; relaunch without shell integration, lines 1783-1806); terminalTooltip.ts (lines 44-48); C bundle (id learnMore; NLS keys shellIntegration.learnMore and shellIntegration.openSettings; setting enabled; timeout absent)
- Pieces:
  1. Each new terminal gets a startup script injected for bash, zsh, fish and PowerShell (and Git Bash on Windows) by default.
  2. Setting `terminal.integrated.shellIntegration.enabled` turns the injection on (default) or off, and only affects terminals created afterwards.
  3. Setting `terminal.integrated.shellIntegration.timeout` sets how long to wait before shell integration is treated as absent. [stock only]
  4. A shell that fails to start with the script is relaunched without it and the tab shows a warning.
  5. The warning offers Learn more about shell integration and Open user settings links (`workbench.action.terminal.learnMore`).

#### PN-2.39 Scrollback size

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: One setting sets how many lines each terminal keeps in its buffer (1000 by default, more lines use more memory) and a second sets how many of those lines come back when a session is restored (100 by default).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.scrollback; terminal.integrated.persistentSessionScrollback
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 318-321); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (lines 401-406); C bundle (both settings; NLS key terminal.integrated.persistentSessionScrollback)
- Pieces:
  1. Setting `terminal.integrated.scrollback` sets how many lines each terminal keeps in its buffer (1000 by default), and more lines use more memory.
  2. Setting `terminal.integrated.persistentSessionScrollback` sets how many of those lines come back when a session is restored (100 by default).

#### PN-2.40 Set the terminal width and height

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run Set Fixed Dimensions to type a column count and then a row count (empty means automatic) that give the terminal a fixed size with scrollbars, or press Alt+Z (Toggle Size to Content Width) to widen the terminal to its longest wrapped line and press it again to go back to fitting the panel.
- Commands: workbench.action.terminal.setDimensions; workbench.action.terminal.sizeToContentWidth
- Menus: TerminalTabContext; TerminalInstanceContext; TerminalEditorInstanceContext; EditorTitleContext
- Keys: Alt+Z on workbench.action.terminal.sizeToContentWidth (when terminalFocus)
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 1384-1401); terminalInstance.ts (setFixedDimensions, lines 2173-2209; toggleSizeToContentWidth, lines 2211-2236); terminalMenus.ts (lines 164-172, 292-301, 622-630 and 702-708); C bundle (ids setDimensions and sizeToContentWidth; NLS keys workbench.action.terminal.setFixedDimensions and workbench.action.terminal.sizeToContentWidthInstance; code setFixedDimensions and toggleSizeToContentWidth present)
- Pieces:
  1. Running Set Fixed Dimensions asks for a column count and then a row count in input boxes, and an empty answer means automatic.
  2. A terminal with fixed dimensions larger than the panel shows scrollbars.
  3. Pressing Alt+Z (Toggle Size to Content Width) widens the terminal to its longest wrapped line, and pressing it again goes back to fitting the panel.
  4. Set Fixed Dimensions and Toggle Size to Content Width are rows on the tab, terminal and terminal editor right-click menus.

#### PN-2.41 Open an external terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present (key rule in Cursor has only the not-terminal-focused condition, V also excludes the sessions window)
- What: Press Ctrl+Shift+C while no terminal has focus, or run Open New External Terminal from the Command Palette, to start the system terminal program (not the built-in one) in the workspace folder; with several folders open a list titled "Select workspace folder" asks which one, a remote window uses the folder's local path when it has one, and otherwise the folder of the last active file or the home folder is used; three per-platform settings choose the program, and debug launch requests for an external console use the same program setting. This is the desktop use of Ctrl+Shift+C, so PN-2.2 lists the same key for New Terminal in the web build only.
- Commands: workbench.action.terminal.openNativeConsole
- Menus: CommandPalette
- Keys: Ctrl+Shift+C on workbench.action.terminal.openNativeConsole (when terminal is not focused; Mac Cmd+Shift+C)
- Settings: terminal.external.windowsExec; terminal.external.osxExec; terminal.external.linuxExec
- Source: V\src\vs\workbench\contrib\externalTerminal\electron-browser\externalTerminal.contribution.ts (command and key, lines 29-101; palette entry, lines 104-108; settings, lines 137-165); V\src\vs\workbench\api\node\extHostDebugService.ts (external branch, lines 209-232); C bundle (id openNativeConsole with primary key CtrlCmd+Shift+C and the notFocus condition; NLS keys globalConsoleAction and selectWorkspace; the three terminal.external settings)
- Pieces:
  1. Pressing Ctrl+Shift+C (Mac Cmd+Shift+C) while no terminal has focus starts the system terminal program in the workspace folder.
  2. Running Open New External Terminal from the Command Palette starts the same system terminal program.
  3. With several folders open, a list titled "Select workspace folder" asks which one to start in.
  4. In a remote window the folder's local path is used when it has one, and otherwise the folder of the last active file or the home folder is used.
  5. Settings `terminal.external.windowsExec`, `terminal.external.osxExec` and `terminal.external.linuxExec` choose which system terminal program starts on each platform.
  6. A debug launch request for an external console starts the program chosen by the same per-platform setting.

#### PN-2.42 Environment variables for new terminals

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Edit one settings object per platform to add environment variables to every new terminal (a null value deletes a variable), choose whether new shells inherit the editor's own environment (on macOS and Linux this may source a login shell so PATH is set up; no effect on Windows), and choose whether the LANG variable is forced to a UTF-8 value (auto, off or on); a terminal profile can also carry its own env object.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.env.windows; terminal.integrated.env.osx; terminal.integrated.env.linux; terminal.integrated.inheritEnv; terminal.integrated.detectLocale
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (detectLocale, lines 323-333; env settings, lines 447-473); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (inheritEnv, lines 395-400; profile env property, lines 52-59); V\src\vs\workbench\contrib\terminal\browser\terminalProfileResolverService.ts (line 152); V\src\vs\workbench\contrib\terminal\electron-browser\localTerminalBackend.ts (line 375); V\src\vs\workbench\contrib\terminal\browser\remoteTerminalBackend.ts (line 177); C bundle (the five setting ids; NLS messages for env.osx, inheritEnv and detectLocale)
- Pieces:
  1. Settings `terminal.integrated.env.windows`, `terminal.integrated.env.osx` and `terminal.integrated.env.linux` each take an object that adds environment variables to every new terminal, and a null value deletes a variable.
  2. Setting `terminal.integrated.inheritEnv` chooses whether new shells inherit the editor's own environment, which on macOS and Linux may source a login shell, and has no effect on Windows.
  3. Setting `terminal.integrated.detectLocale` chooses whether the `LANG` variable is forced to a UTF-8 value, with values `auto` (default), `off` and `on`.
  4. A terminal profile can carry its own `env` object.

#### PN-2.43 Terminal profile for tasks and debug

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Set a profile object (shell path, arguments, icon, color and so on) per platform that tasks and debug sessions use for their terminals instead of the default profile; the setting is empty by default (so the default profile is used), it is a restricted setting that an untrusted workspace ignores, and on Windows it is ignored while the old automation shell setting is set.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.automationProfile.windows; terminal.integrated.automationProfile.osx; terminal.integrated.automationProfile.linux
- Source: V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (schema, lines 81-94; the three settings, lines 117-170); V\src\vs\workbench\contrib\terminal\browser\terminalProfileService.ts (lines 158-165); V\src\vs\workbench\contrib\terminal\browser\terminalProfileResolverService.ts (allowAutomationShell, lines 205-250); V\src\vs\workbench\contrib\tasks\browser\terminalTaskSystem.ts (line 1227); V\src\vs\workbench\api\browser\mainThreadTerminalService.ts (lines 128-129); C bundle (the three setting ids with restricted true; NLS key terminal.integrated.automationProfile.linux; allowAutomationShell in the task system and in the terminal service bridge)
- Pieces:
  1. Settings `terminal.integrated.automationProfile.windows`, `terminal.integrated.automationProfile.osx` and `terminal.integrated.automationProfile.linux` each take a profile object (shell path, arguments, icon, color and so on) that tasks and debug sessions use for their terminals.
  2. The setting is empty by default, so tasks and debug sessions use the default profile.
  3. The setting is restricted, so an untrusted workspace ignores it.
  4. On Windows the setting is ignored while the old automation shell setting is set.

#### PN-2.44 Debug session in an integrated terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the ext host bundle has the clear-before-reuse code and the forced shell integration option)
- What: When a launch configuration asks for the integrated terminal, the debug session's program runs in a terminal tab with the debug icon, named after the session title (or "Debug Process"), started with the automation profile and with shell integration forced on; the panel shows the tab without taking focus, a finished session's tab is reused by the next session with the same shell, folder and title once its program has ended, and a setting clears the screen before the reuse.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.terminal.clearBeforeReusing; terminal.integrated.automationProfile.windows; terminal.integrated.automationProfile.osx; terminal.integrated.automationProfile.linux
- Source: V\src\vs\workbench\api\node\extHostDebugService.ts (integrated branch, lines 79-207; DebugTerminalCollection, lines 234-289); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (clearBeforeReusing, lines 558-562); C bundle (setting debug.terminal.clearBeforeReusing in the main bundle; NLS key debug.terminal.title in module extHostDebugService; the reuse code in C\out\vs\workbench\api\node\extensionHostProcess.js)
- Pieces:
  1. When a launch configuration asks for the integrated terminal, the debug session's program runs in a terminal tab with the debug icon.
  2. The tab is named after the session title, or "Debug Process" when there is none.
  3. The session starts with the per-platform `terminal.integrated.automationProfile.*` profile.
  4. The panel shows the tab without taking keyboard focus.
  5. The next session with the same shell, folder and title reuses a finished session's tab once its program has ended.
  6. Setting `debug.terminal.clearBeforeReusing` clears the screen before a tab is reused, as on or off.

#### PN-2.45 Task terminal visibility and echo

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The presentation options in a task's tasks.json entry decide how its terminal shows up: reveal (always brings the terminal forward, silent only when the task exits non-zero or its problem matcher finds an error, never keeps it in the background), focus (the terminal also takes keyboard focus), revealProblems (always, onProblem or never opens the Problems view, and a choice of always opens Problems instead of the terminal) and echo (whether the "Executing task: ..." line is printed first); defaults are echo on, reveal always, focus off, revealProblems never.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\terminalTaskSystem.ts (background tasks, lines 904-916 and 985-996; other tasks, lines 1106-1118; immediate reveal and focus, lines 1155-1163; echo, lines 1330-1385 and 1536-1539); V\src\vs\workbench\contrib\tasks\common\tasks.ts (RevealKind, lines 130-163; RevealProblemKind, lines 165-196; defaults, lines 289-292); V\src\vs\workbench\contrib\tasks\common\jsonSchema_v2.ts (presentation schema); C bundle (schema keys echo, focus and revealProblems with NLS keys JsonSchema.tasks.presentation.echo and JsonSchema.tasks.presentation.revealProblems.onProblem; the defaults object with echo true, reveal 1, revealProblems 1, focus false; the Executing task string)
- Pieces:
  1. Setting `presentation.reveal` in a task's tasks.json entry chooses when its terminal comes forward, with values `always` (default), `silent` (only when the task exits non-zero or its problem matcher finds an error) and `never` (stays in the background).
  2. Setting `presentation.focus` to true also gives the task terminal keyboard focus (off by default).
  3. Setting `presentation.revealProblems` chooses when the Problems view opens, with values `never` (default), `onProblem` and `always`.
  4. With `presentation.revealProblems` set to `always`, the Problems view opens instead of the terminal.
  5. Setting `presentation.echo` chooses whether the "Executing task: ..." line is printed first, as on (default) or off.

#### PN-2.46 Task terminal name, icon and color

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (name, icon and color are present; the schema key preserveTerminalName is not in the bundle)
- What: A task's terminal tab is named after the task (with the folder name in brackets in a multi-root workspace) and shows the icon and color the task sets in tasks.json (icon.id is a codicon id and icon.color is one of the terminal ANSI color ids); a task started as a dependency inherits any icon id or color it does not set from the task that asked for it, and each task terminal also gets a Rerun Task button on its tab row (see PN-2.37). V also lists a presentation option preserveTerminalName in the schema, but I found no code that reads it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\terminalTaskSystem.ts (_createTerminalName, lines 1195-1198; icon and color, lines 1231-1247, 1358-1361 and 1541-1542; dependency inheritance, lines 653-660; tab actions, lines 177 and 1407); V\src\vs\workbench\contrib\terminal\browser\terminalTabsList.ts (tab action buttons, lines 534-540); V\src\vs\workbench\contrib\tasks\common\tasks.ts (getQualifiedLabel, lines 701-708); V\src\vs\workbench\contrib\tasks\common\jsonSchema_v2.ts (icon schema, lines 115-145; preserveTerminalName, lines 211-215); C bundle (NLS key JsonSchema.tasks.icon; the code that reads configurationProperties.icon id and color)
- Pieces:
  1. A task's terminal tab is named after the task.
  2. In a multi-root workspace the tab name also shows the folder name in brackets.
  3. The tab shows the icon set by `icon.id` in tasks.json, which is a codicon id.
  4. The tab shows the color set by `icon.color` in tasks.json, which is one of the terminal ANSI color ids.
  5. A task started as a dependency inherits any icon id or color it does not set from the task that asked for it.
  6. Each task terminal gets a Rerun Task button on its tab row.
  7. The tasks.json schema offers a `presentation.preserveTerminalName` option, though it changes nothing visible. [stock only]

#### PN-2.47 Windows terminal backend options

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (windowsEnableConpty exists only in Cursor, default true; windowsUseConptyDll defaults to false and is tagged preview, V defaults to true)
- What: Two Windows-only settings pick how the shell is hosted: one uses the conpty.dll shipped with the editor instead of the one inside Windows, and (Cursor only) one chooses ConPTY or the older Winpty; when ConPTY cannot start, the launch failure message tells you to try the DLL setting. In V 1.128.0 Winpty is gone, so only the DLL setting exists and it is on by default; in Cursor the DLL setting is off by default and tagged preview. The newer 1.144.0 tree adds terminal.integrated.windowsReparentConsoleWindow, which is not part of this feature in 1.128.0.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.windowsUseConptyDll; terminal.integrated.windowsEnableConpty
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (windowsUseConptyDll, lines 484-489); V\src\vs\platform\terminal\node\terminalProcess.ts (lines 159-168 and 250-257); C bundle (setting windowsUseConptyDll with tags preview and default false, NLS text "experimental conpty.dll (v1.22.250204002)"; setting windowsEnableConpty with NLS text "Winpty will be used if this is false"); N (windowsReparentConsoleWindow, terminalConfiguration.ts line 501, newer-than-1.128)
- Pieces:
  1. The Windows-only setting `terminal.integrated.windowsUseConptyDll` makes the terminal use the conpty.dll shipped with the editor instead of the one inside Windows.
  2. In stock `terminal.integrated.windowsUseConptyDll` is on by default. [stock only]
  3. In Cursor `terminal.integrated.windowsUseConptyDll` is off by default and tagged preview. [cursor only]
  4. The Windows-only setting `terminal.integrated.windowsEnableConpty` chooses between ConPTY (default) and the older Winpty. [cursor only]
  5. When ConPTY cannot start, the launch failure message tells you to try the conpty.dll setting.

#### PN-2.48 Return to the terminal after the editor's wait flag

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Running the editor's command line tool with its wait flag from a terminal inside the editor (for example as the commit message editor for Git) keeps that command waiting until you close the opened file, and then moves keyboard focus back to the active terminal; it only applies to requests that come from the editor's own terminal (TERM_PROGRAM is vscode).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\electron-browser\terminalNativeContribution.ts (lines 29 and 48-75); C bundle (class with _onOpenFileRequest that tests termProgram equal to "vscode" and filesToWait, then focuses the active terminal)
- Pieces:
  1. Running the editor's command line tool with its wait flag from the editor's own terminal keeps that command waiting until you close the opened file.
  2. After you close the file, keyboard focus moves back to the active terminal.
  3. This applies only to requests from the editor's own terminal (TERM_PROGRAM is vscode), so requests from other terminals do not move focus.

#### PN-2.49 Terminal tip at the first prompt

- Tag: cursor. Evidence: bundle. Boundary: yes - the tip advertises the Cursor agent command line tool, and its text and action come from a remote experiment named terminal_tip.. Cursor: cursor only
- What: At the first prompt of an ordinary new terminal (not an extension terminal or a task or other feature terminal), a faint italic tip can appear after the cursor when the cursor is in the first 30 columns; it is only offered to a person whose machine has the Claude or Codex command line tool but not the Cursor one, the experiment parameters include a show count and an hours interval between shows, any key press removes it, and when the tip has an action then Shift+Tab as the very first key types that action into the prompt without running it.
- Commands: none found
- Menus: none found
- Keys: Shift+Tab (only as the first key press while the tip shows, when the tip has an action)
- Settings: none found
- Source: C bundle (contribution id terminal.cursorCliPrompt; experiment name terminal_tip with parameters enabled, message, action, show_count and show_every_hours; stored state terminalTipState; CLI detection status values claudeStatus, codexStatus and agentStatus; NLS module terminalContrib/cursorCliPrompt)
- Pieces:
  1. At the first prompt of an ordinary new terminal a faint italic tip can appear after the cursor. [cursor only]
  2. The tip only appears when the cursor is within the first 30 columns. [cursor only]
  3. The tip is not shown in extension terminals, task terminals or other feature terminals. [cursor only]
  4. The tip is only offered when the machine has the Claude or Codex command line tool but not the Cursor one. [cursor only]
  5. A show count and an hours interval limit how often the tip appears. [cursor only]
  6. Pressing any key removes the tip. [cursor only]
  7. When the tip has an action, pressing Shift+Tab as the very first key types that action into the prompt without running it. [cursor only]

#### PN-2.50 Install Cursor CLI command

- Tag: cursor. Evidence: bundle. Boundary: yes - the command installs the Cursor agent command line tool, so its purpose is an agent surface even though the UI is a plain terminal.. Cursor: cursor only
- What: Run Install Cursor CLI to open a new terminal in the panel, focus it and run the installer line (curl https://cursor.com/install -fsS piped to bash), or get the message "Cursor CLI is already installed." when the app has already detected the tool; the palette flag f1 is set to the negation of a constant I could not resolve (it may mean Windows, since the line is a bash script, not verified).
- Commands: cursor.installCli
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (command id cursor.installCli; NLS keys cursor.installCli and cursor.cliAlreadyInstalled in module terminalContrib/cursorCliPrompt)
- Pieces:
  1. Install Cursor CLI appears in the Command Palette. [cursor only]
  2. Running Install Cursor CLI opens a new terminal in the panel, focuses it and runs the installer line (curl https://cursor.com/install -fsS piped to bash). [cursor only]
  3. When the app has already detected the tool, running Install Cursor CLI shows the message "Cursor CLI is already installed." instead. [cursor only]


## PN-3 Terminal tooling

(both). What you use inside one terminal: find, links, shell-integration marks, copy and paste. Examples: the find widget; command decorations and sticky scroll; links and quick fixes. Terminal suggest is AC-5.

47 features.

#### PN-3.1 Find in the terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+F in a terminal to open a find box over it, type to highlight matches in the output and scrollback, step with Enter and Shift+Enter, flip match case, whole word and regular expression, read the match count, and close it with Esc.
- Commands: workbench.action.terminal.focusFind; workbench.action.terminal.hideFind; workbench.action.terminal.findNext; workbench.action.terminal.findPrevious; workbench.action.terminal.toggleFindRegex; workbench.action.terminal.toggleFindWholeWord; workbench.action.terminal.toggleFindCaseSensitive
- Menus: none found
- Keys: Ctrl+F (Cmd+F on macOS) on focusFind when a terminal or the find box has focus; Esc on hideFind while the find box is visible; Alt+R (Cmd+Alt+R on macOS) on toggleFindRegex; Alt+W (Cmd+Alt+W on macOS) on toggleFindWholeWord; Alt+C (Cmd+Alt+C on macOS) on toggleFindCaseSensitive; F3 (Cmd+G on macOS) on findNext; Shift+F3 (Cmd+Shift+G on macOS) on findPrevious; Enter and Shift+Enter inside the find input
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\find\browser\terminal.find.contribution.ts (lines 104-240); V\src\vs\workbench\contrib\terminalContrib\find\browser\terminalFindWidget.ts; V\src\vs\workbench\contrib\terminalContrib\find\common\terminal.find.ts; C bundle (all seven command ids; module id terminal.find)
- Pieces:
  1. Pressing `Ctrl+F` (`Cmd+F` on macOS) with a terminal focused opens a find box over the terminal, filled with the selected text when the selection is on one line.
  2. Typing in the find box highlights the matches in the visible output and the scrollback, and the box shows a match count next to previous match, next match and close buttons.
  3. `Enter` in the find input steps to the next match and `Shift+Enter` to the previous one, and `F3` and `Shift+F3` (`Cmd+G` and `Cmd+Shift+G` on macOS) step the same way through `workbench.action.terminal.findNext` and `workbench.action.terminal.findPrevious`.
  4. The match case, whole word and regular expression toggles in the find box change which text matches, and `Alt+C`, `Alt+W` and `Alt+R` (`Cmd+Alt+C`, `Cmd+Alt+W` and `Cmd+Alt+R` on macOS) flip them from the keyboard.
  5. Pressing `Esc` or `Shift+Esc` while the find box is visible closes it.
  6. Dragging the left edge of the find box resizes it from its starting width.

#### PN-3.2 Search the workspace for selected terminal text

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With text selected in the terminal, press Ctrl+Shift+F to open the Search view filled with that text, so a path or error string can be searched across the workspace.
- Commands: workbench.action.terminal.searchWorkspace
- Menus: none found
- Keys: Ctrl+Shift+F (Cmd+Shift+F on macOS) when the terminal has focus, a process is supported and text is selected
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\find\browser\terminal.find.contribution.ts (lines 241-257); C bundle (id searchWorkspace)
- Pieces:
  1. With text selected in a terminal, pressing `Ctrl+Shift+F` (`Cmd+Shift+F` on macOS) opens the Search view with the selected text already in the search box.
  2. The `workbench.action.terminal.searchWorkspace` shortcut only applies while the terminal has focus, a process is supported and text is selected.

#### PN-3.3 Click a link in terminal output

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The terminal finds file paths (with line and column), folders, URLs and plain words in its output and underlines them on hover; Ctrl+click (Cmd+click on macOS, Alt+click when the multi-cursor modifier is Ctrl or Cmd) opens a file in the editor, focuses a workspace folder in the Explorer, opens an outside folder in a new window, opens a URL, or runs a workspace search for a word; a URL with a scheme that is not on the allowed list first asks "Allow <scheme>?" in a warning notification.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enableFileLinks; terminal.integrated.allowedLinkSchemes; editor.multiCursorModifier
- Source: V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkManager.ts (lines 115-155, 455-480); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkDetectorAdapter.ts (lines 128-140); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLocalLinkDetector.ts; terminalUriLinkDetector.ts; terminalWordLinkDetector.ts; terminalMultiLineLinkDetector.ts; terminalLinkOpeners.ts; V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 507-531); C bundle (settings enableFileLinks and allowedLinkSchemes; module id terminal.link)
- Pieces:
  1. Hovering a file path, folder, URL or plain word in terminal output underlines it as a link.
  2. Following a file path link with `Ctrl+click` (`Cmd+click` on macOS) opens that file in the editor, at the line and column when the path has them.
  3. Following a folder link focuses it in the Explorer when it is inside the workspace, and opens it in a new window when it is outside.
  4. Following a URL link opens the URL, and following a plain word link runs a workspace search for that word.
  5. A path that wraps across several terminal lines is still found as one link.
  6. Following a URL whose scheme is not on the allowed list first shows a warning notification that asks whether to allow opening links with that scheme, with an `Allow <scheme>` button.
  7. Setting `editor.multiCursorModifier` changes the click modifier, so the link click becomes `Alt+click` when the multi-cursor modifier is Ctrl or Cmd.
  8. Setting `terminal.integrated.enableFileLinks` chooses whether file and folder links are detected, with values `off`, `on` and `notRemote`.
  9. Setting `terminal.integrated.allowedLinkSchemes` lists the URL schemes that open without the prompt, by default `file`, `http`, `https`, `mailto`, `vscode` and `vscode-insiders`.

#### PN-3.4 Hover a terminal link

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pointing at a detected link shows a hover that names the action and the click modifier, such as "Follow link (ctrl + click)", "Open file in editor", "Focus folder in explorer", "Open folder in new window", "Search workspace", or "Follow link using forwarded port"; the hover can be turned off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.showLinkHover
- Source: V\src\vs\workbench\contrib\terminal\browser\widgets\terminalHoverWidget.ts (lines 27-60); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkManager.ts (lines 460-510); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkDetectorAdapter.ts (lines 128-140); C bundle (setting showLinkHover)
- Pieces:
  1. Pointing at a detected link shows a hover that names the action and the click modifier, such as `Follow link (ctrl + click)`.
  2. The modifier named in the hover follows the platform and `editor.multiCursorModifier`, reading `ctrl + click`, `cmd + click`, `alt + click` or `option + click`.
  3. The hover wording changes with the link type, reading `Open file in editor`, `Focus folder in explorer`, `Open folder in new window` or `Search workspace`.
  4. A link that goes through a forwarded port shows `Follow link using forwarded port` in its hover.
  5. Setting `terminal.integrated.showLinkHover` turns the link hover on or off.

#### PN-3.5 Open a link from the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+Shift+O to get a quick pick of the links in the terminal, grouped as Url, File, Folder and Workspace Search, with type-to-filter over all links (first the visible area, then the whole buffer); the Command Palette also has Open Last URL Link and Open Last Local File Link, and the same quick pick opens from the accessible terminal view with Ctrl+Shift+G.
- Commands: workbench.action.terminal.openDetectedLink; workbench.action.terminal.openUrlLink; workbench.action.terminal.openFileLink
- Menus: none found
- Keys: Ctrl+Shift+O (Cmd+Shift+O on macOS) on openDetectedLink when the terminal has focus; Ctrl+Shift+G (Cmd+Shift+G on macOS) on openDetectedLink when the accessible view shows the terminal
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\links\browser\terminal.links.contribution.ts (lines 117-160); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkQuickpick.ts (lines 55-125); V\src\vs\workbench\contrib\terminalContrib\links\common\terminal.links.ts; C bundle (ids openDetectedLink, openUrlLink, openFileLink)
- Pieces:
  1. Pressing `Ctrl+Shift+O` (`Cmd+Shift+O` on macOS) with a terminal focused opens a quick pick with the placeholder `Select the link to open, type to filter all links`.
  2. The quick pick groups links under separators named Url, File, Folder and Workspace Search.
  3. The list starts with links from the visible area, and typing a filter searches links from the whole buffer.
  4. Choosing a row opens that link the same way a click would.
  5. `Open Last URL Link` and `Open Last Local File Link` in the Command Palette open the newest URL or file link in the terminal without showing a pick.
  6. Pressing `Ctrl+Shift+G` (`Cmd+Shift+G` on macOS) in the accessible view that shows the terminal opens the same quick pick.

#### PN-3.6 Command decorations

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When shell integration is active, each command gets a dot in the left gutter and a mark in the right overview ruler that shows success or failure; hovering a dot says when the command ran, how long it took, and its exit code; right-clicking a dot shows Toggle Visibility, which opens a two-checkbox pick for gutter decorations and overview ruler decorations.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.shellIntegration.decorationsEnabled
- Source: V\src\vs\workbench\contrib\terminal\browser\xterm\decorationAddon.ts (lines 128-140, 396-424, 565-606); V\src\vs\workbench\contrib\terminal\browser\xterm\decorationStyles.ts (lines 32-135); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 616-628); C bundle (setting decorationsEnabled; action id terminal.toggleVisibility)
- Pieces:
  1. With shell integration active, each command gets a dot in the terminal's left gutter that shows whether it succeeded or failed.
  2. Each command also gets a mark in the overview ruler on the right of the terminal.
  3. A command that has not finished or has no known exit code shows a neutral default dot instead of a success or failure one.
  4. Hovering a dot shows when the command ran, how long it took, and its exit code on failure, for example `Command executed 5 seconds ago, took 2 seconds and failed (Exit Code 1)`.
  5. Right-clicking a dot shows a `Toggle Visibility` entry that opens a pick with two checkboxes, one for gutter decorations and one for overview ruler decorations.
  6. Setting `terminal.integrated.shellIntegration.decorationsEnabled` chooses where decorations show, with values `both`, `gutter`, `overviewRuler` and `never`.

#### PN-3.7 Rerun and copy a command or its output

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking a command's dot opens a menu with Rerun Command (it asks "Do you want to run the command" first when shell integration did not mark the command as trusted), Copy Command, Copy Command and Output, Copy Output, Copy Output as HTML, Run Recent Command, Go To Recent Directory and Learn About Shell Integration; the Command Palette has Copy Last Command, Copy Last Command Output and Copy Last Command and Output for the newest command.
- Commands: workbench.action.terminal.copyLastCommand; workbench.action.terminal.copyLastCommandOutput; workbench.action.terminal.copyLastCommandAndLastCommandOutput
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\xterm\decorationAddon.ts (lines 396-520, menu actions built in code with ids terminal.rerunCommand, terminal.copyCommand, terminal.copyCommandAndOutput, terminal.copyOutput, terminal.copyOutputAsHtml, terminal.learnShellIntegration); V\src\vs\workbench\contrib\terminalContrib\clipboard\browser\terminal.clipboard.contribution.ts (lines 188-246); C bundle (the three copyLast ids; action ids terminal.rerunCommand, terminal.copyCommandAndOutput, terminal.copyOutputAsHtml and terminal.learnShellIntegration)
- Pieces:
  1. Clicking a command's dot opens a menu of actions for that command.
  2. Choosing `Rerun Command` runs the command again, and when shell integration did not mark the command as trusted it first asks `Do you want to run the command`.
  3. `Copy Command`, `Copy Command and Output` and `Copy Output` copy those parts of that command to the clipboard, and `Copy Output as HTML` copies the output with its colors and styles.
  4. The menu also has `Run Recent Command` and `Go To Recent Directory`, which open their quick picks, and `Learn About Shell Integration`, which opens help about it.
  5. `Copy Last Command`, `Copy Last Command Output` and `Copy Last Command and Output` in the Command Palette copy the text, the output or both for the newest command.

#### PN-3.8 Jump between commands

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Ctrl+Up and Ctrl+Down scroll the terminal to the previous or next command mark so old commands can be skimmed; the same two actions can be added as arrow buttons on the terminal view toolbar or a terminal editor toolbar (hidden by default).
- Commands: workbench.action.terminal.scrollToPreviousCommand; workbench.action.terminal.scrollToNextCommand
- Menus: ViewTitle; EditorTitle; CompactWindowEditorTitle
- Keys: Ctrl+Up on scrollToPreviousCommand and Ctrl+Down on scrollToNextCommand (Cmd on macOS), when the terminal has focus and screen reader mode is off
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 868-930); V\src\vs\workbench\contrib\terminal\browser\xterm\markNavigationAddon.ts; C bundle (ids scrollToPreviousCommand and scrollToNextCommand)
- Pieces:
  1. Pressing `Ctrl+Up` (`Cmd+Up` on macOS) scrolls the terminal to the previous command, and `Ctrl+Down` scrolls to the next, when the terminal has focus and screen reader mode is off.
  2. The command that was jumped to is briefly highlighted with an outline so it is easy to spot.
  3. `Scroll To Previous Command` and `Scroll To Next Command` can be shown as up and down arrow buttons on the terminal view toolbar, where they are hidden by default.
  4. The same two arrow buttons can be shown on the toolbar of a terminal editor tab, where they are hidden by default.

#### PN-3.9 Select command output

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Ctrl+Shift+Up and Ctrl+Shift+Down extend a selection from the current spot to the previous or next command mark, which selects whole commands and their output; Select to Previous Line and Select to Next Line in the Command Palette extend the selection one line at a time.
- Commands: workbench.action.terminal.selectToPreviousCommand; workbench.action.terminal.selectToNextCommand; workbench.action.terminal.selectToPreviousLine; workbench.action.terminal.selectToNextLine
- Menus: none found
- Keys: Ctrl+Shift+Up on selectToPreviousCommand and Ctrl+Shift+Down on selectToNextCommand (Cmd+Shift on macOS), when the terminal has focus; none for the two line commands
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 932-982); V\src\vs\workbench\contrib\terminal\browser\xterm\markNavigationAddon.ts (lines 470-520); C bundle (all four ids)
- Pieces:
  1. Pressing `Ctrl+Shift+Up` (`Cmd+Shift+Up` on macOS) extends the selection back to the previous command mark, and `Ctrl+Shift+Down` extends it forward to the next one.
  2. Pressing the same key again keeps extending the selection across more commands and their output.
  3. `Select To Previous Line` and `Select To Next Line` in the Command Palette extend the selection by one line, and they have no default key.

#### PN-3.10 Command guide

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Hovering over the text of a command draws a thin vertical guide bar along that command's prompt and output, so you can see where one command starts and ends.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.shellIntegration.showCommandGuide
- Source: V\src\vs\workbench\contrib\terminalContrib\commandGuide\browser\terminal.commandGuide.contribution.ts (lines 20-90); V\src\vs\workbench\contrib\terminalContrib\commandGuide\common\terminalCommandGuideConfiguration.ts; V\src\vs\workbench\contrib\terminal\browser\xterm\markNavigationAddon.ts (showCommandGuide, line 290); C bundle (setting showCommandGuide; css class terminal-command-guide)
- Pieces:
  1. With shell integration active, hovering over the text of a command draws a thin vertical guide bar along that command's prompt and output.
  2. Moving the pointer off the command removes the guide bar.
  3. Setting `terminal.integrated.shellIntegration.showCommandGuide` turns the command guide on or off, on by default.

#### PN-3.11 Sticky scroll

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While you scroll up through a long command's output, that command's prompt line stays pinned at the top of the terminal (up to a set number of lines); hovering it says "Navigate to Command" with the jump keys, clicking it jumps back to the command, right-clicking it offers Toggle Sticky Scroll, and some commands can be excluded; Toggle Sticky Scroll is also in the Command Palette.
- Commands: workbench.action.terminal.toggleStickyScroll
- Menus: TerminalStickyScrollContext
- Keys: none found
- Settings: terminal.integrated.stickyScroll.enabled; terminal.integrated.stickyScroll.maxLineCount; terminal.integrated.stickyScroll.ignoredCommands
- Source: V\src\vs\workbench\contrib\terminalContrib\stickyScroll\browser\terminal.stickyScroll.contribution.ts (lines 20-45); V\src\vs\workbench\contrib\terminalContrib\stickyScroll\browser\terminalStickyScrollOverlay.ts (lines 86, 399-462); V\src\vs\workbench\contrib\terminalContrib\stickyScroll\common\terminalStickyScrollConfiguration.ts; V\src\vs\platform\actions\common\actions.ts (line 249); C bundle (id toggleStickyScroll; menu id TerminalStickyScrollContext; three settings)
- Pieces:
  1. With shell integration active, scrolling up through a long command's output keeps that command's prompt line pinned at the top of the terminal.
  2. Hovering the pinned line shows `Navigate to Command` with the keys for scrolling to the previous and next command.
  3. Clicking the pinned line jumps the terminal back to that command.
  4. Right-clicking the pinned line shows a menu with `Toggle Sticky Scroll`, and the same command in the Command Palette turns sticky scroll off and on.
  5. Setting `terminal.integrated.stickyScroll.enabled` turns sticky scroll on or off, on by default.
  6. Setting `terminal.integrated.stickyScroll.maxLineCount` limits how many lines are pinned, from 1 to 10 with a default of 5, and the pinned area never takes more than 40 percent of the terminal height.
  7. Setting `terminal.integrated.stickyScroll.ignoredCommands` lists commands that do not pin their prompt, by default `clear`, `cls`, `clear-host` and several coding agent command names.

#### PN-3.12 Run recent command

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+Alt+R to open a quick pick of recent commands from this terminal session and the shell's history file; moving through the rows scrolls the terminal to that command, Enter runs the chosen one, holding Alt types it on the prompt without running so it can be edited, each row has buttons to view its output in an editor or remove it from history, rows can be pinned, and a fuzzy search toggle sits in the input; Clear Previous Session History in the Command Palette wipes the stored history.
- Commands: workbench.action.terminal.runRecentCommand; workbench.action.terminal.clearPreviousSessionHistory
- Menus: ViewTitle; EditorTitle; CompactWindowEditorTitle
- Keys: Ctrl+Alt+R (Ctrl+Option+R on macOS) on runRecentCommand when the terminal has focus and screen reader mode is off; Ctrl+R on runRecentCommand when screen reader mode is on and the terminal or accessible terminal view has focus
- Settings: terminal.integrated.shellIntegration.history
- Source: V\src\vs\workbench\contrib\terminalContrib\history\browser\terminal.history.contribution.ts (lines 100-215); V\src\vs\workbench\contrib\terminalContrib\history\browser\terminalRunRecentQuickPick.ts (lines 54-275); V\src\vs\workbench\contrib\terminalContrib\history\common\terminal.history.ts; C bundle (ids runRecentCommand and clearPreviousSessionHistory; setting shellIntegration.history)
- Pieces:
  1. Pressing `Ctrl+Alt+R` (`Ctrl+Option+R` on macOS), or `Ctrl+R` when screen reader mode is on, opens a quick pick of recent commands with the placeholder `Select a command to run (hold Alt-key to edit the command)`.
  2. The rows sit under separators for the current session, the previous session and the shell's own history file.
  3. Moving through the rows scrolls the terminal to that command when it is in the current session.
  4. Pressing `Enter` on a row runs that command, and holding `Alt` (`Option` on macOS) while choosing it types the command at the prompt without running it.
  5. Each row has a `View Command Output` button that opens its output in an editor and a `Remove from Command History` button, and the shell history group has an `Open File` button.
  6. A row can be pinned, and pinned commands are remembered for that shell type.
  7. A fuzzy search toggle in the input switches the typed filter between contiguous and fuzzy matching.
  8. `Clear Previous Session History` in the Command Palette wipes the stored command history, so older commands no longer appear in the pick.
  9. `Run Recent Command` can be shown as a button on the terminal view toolbar or a terminal editor toolbar, where it is hidden by default.
  10. Setting `terminal.integrated.shellIntegration.history` sets how many recent commands are kept, with a default of 100 and 0 turning the history off.

#### PN-3.13 Go to recent directory

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+G to open a quick pick of folders visited in this terminal and listed in the shell's history; choosing one runs a cd command for that folder (hold Alt to type it on the prompt without running), and the pick has the same remove button and fuzzy search toggle as Run Recent Command.
- Commands: workbench.action.terminal.goToRecentDirectory
- Menus: ViewTitle; EditorTitle; CompactWindowEditorTitle
- Keys: Ctrl+G (Cmd+G on macOS) when the terminal has focus
- Settings: terminal.integrated.shellIntegration.history
- Source: V\src\vs\workbench\contrib\terminalContrib\history\browser\terminal.history.contribution.ts (lines 114-156); V\src\vs\workbench\contrib\terminalContrib\history\browser\terminalRunRecentQuickPick.ts (lines 196-250); C bundle (id goToRecentDirectory)
- Pieces:
  1. Pressing `Ctrl+G` (`Cmd+G` on macOS) with a terminal focused opens a quick pick of recent folders with the placeholder `Select a directory to go to (hold Alt-key to edit the command)`.
  2. The folders sit under separators for the current session and the previous session.
  3. Choosing a folder runs a `cd` command for it in the terminal.
  4. Holding `Alt` (`Option` on macOS) while choosing a folder types the `cd` command at the prompt without running it.
  5. The pick has the same remove button on each row and the same fuzzy search toggle as `Run Recent Command`.
  6. `Go to Recent Directory` can be shown as a button on the terminal view toolbar or a terminal editor toolbar, where it is hidden by default.
  7. Setting `terminal.integrated.shellIntegration.history` also decides how many recent entries are kept for this list, and 0 turns the history off.

#### PN-3.14 Quick fixes after a command

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the command and the built-in fix ids are present; the quickFixEnabled setting has no hit in the bundle)
- What: After a command finishes with a matching result, a lightbulb or sparkle icon appears to the left of the prompt; clicking it or pressing Ctrl+. lists fixes that either run a command ("Run: ...") or open a link ("Open: ..."), such as git suggestions for a mistyped subcommand, a fast-forward pull, push with set-upstream, create a pull request, retry with two dashes, free a busy port, PowerShell's "most similar commands" and command-not-found suggestions, and the npm "Did you mean" fix; the same fixes are listed in the command's dot menu.
- Commands: workbench.action.terminal.showQuickFixes
- Menus: none found
- Keys: Ctrl+. (Cmd+. on macOS) on showQuickFixes when the terminal has focus
- Settings: terminal.integrated.shellIntegration.quickFixEnabled
- Source: V\src\vs\workbench\contrib\terminalContrib\quickFix\browser\terminal.quickFix.contribution.ts (lines 30-100); V\src\vs\workbench\contrib\terminalContrib\quickFix\browser\terminalQuickFixBuiltinActions.ts; V\src\vs\workbench\contrib\terminalContrib\quickFix\browser\quickFixAddon.ts (lines 300-430); V\extensions\npm\package.json (contributes.terminalQuickFixes, id ms-vscode.npm-command); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 637-641); C\extensions\npm\package.json; C bundle (id showQuickFixes; fix ids Git Similar, Git Fast Forward Pull, Git Two Dashes, Free Port, Git Push Set Upstream, Git Create Pr, Pwsh General Error, Unix Command Not Found)
- Pieces:
  1. After a command finishes with a result that has a known fix, a lightbulb or sparkle icon appears to the left of the prompt.
  2. Clicking the icon or pressing `Ctrl+.` (`Cmd+.` on macOS) lists the available fixes.
  3. Each fix is labeled `Run: <command>` when it runs a command or `Open: <link>` when it opens a link.
  4. Git fixes cover a mistyped subcommand, a fast-forward pull, push with set-upstream, creating a pull request, and retrying with two dashes.
  5. Other fixes cover freeing a busy port, PowerShell's most similar commands and command-not-found suggestions, and the npm `Did you mean` fix.
  6. The same fixes are also listed in the command's dot menu.
  7. Setting `terminal.integrated.shellIntegration.quickFixEnabled` turns the quick fix icon on or off, on by default, and this setting is not found in Cursor. [stock only]

#### PN-3.15 Copy selected text

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Copy the selection with Ctrl+Shift+C (Cmd+C on macOS); on Windows Ctrl+C copies and clears the selection when text is selected and otherwise goes to the shell; Copy as HTML keeps colors and styles; a setting copies automatically as soon as text is selected.
- Commands: workbench.action.terminal.copySelection; workbench.action.terminal.copyAndClearSelection; workbench.action.terminal.copySelectionAsHtml
- Menus: TerminalInstanceContext; TerminalEditorInstanceContext
- Keys: Ctrl+Shift+C (Cmd+C on macOS) on copySelection and Ctrl+C on Windows on copyAndClearSelection, when text is selected and the terminal has focus
- Settings: terminal.integrated.copyOnSelection
- Source: V\src\vs\workbench\contrib\terminalContrib\clipboard\browser\terminal.clipboard.contribution.ts (lines 60-72, 248-292); V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (lines 120-140); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 159-163); C bundle (the three ids; setting copyOnSelection)
- Pieces:
  1. Pressing `Ctrl+Shift+C` (`Cmd+C` on macOS) with text selected and the terminal focused copies the selection to the clipboard.
  2. On Windows, `Ctrl+C` copies the selection and clears it when text is selected, and otherwise goes to the shell.
  3. `Copy as HTML` copies the selection with its colors and styles.
  4. The terminal right-click menu has `Copy` and `Copy as HTML` entries, in the panel and in a terminal editor tab.
  5. Setting `terminal.integrated.copyOnSelection` copies text to the clipboard as soon as it is selected, off by default.

#### PN-3.16 Paste into the terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (paste and pasteSelection are present; the pastePwsh id has no hit, and the PowerShell Ctrl+V rule is a plain send-sequence rule with no file-clipboard fallback)
- What: Paste the clipboard with Ctrl+V (Cmd+V on macOS; Ctrl+Shift+V also works on Windows and is the key on Linux), paste the Linux selection clipboard with Shift+Insert, or paste on middle click when that is chosen; if the clipboard holds a copied file and no text, its path is pasted; on Windows with PowerShell Ctrl+V is passed straight to the shell so PSReadLine handles it unless screen reader mode is on.
- Commands: workbench.action.terminal.paste; workbench.action.terminal.pasteSelection; workbench.action.terminal.pastePwsh
- Menus: TerminalInstanceContext; TerminalEditorInstanceContext
- Keys: Ctrl+V (Cmd+V on macOS) on paste; Ctrl+Shift+V on paste (secondary on Windows, primary on Linux); Shift+Insert on pasteSelection (Linux only); Ctrl+V on pastePwsh (Windows, PowerShell, screen reader mode off)
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\clipboard\browser\terminal.clipboard.contribution.ts (lines 80-124, 294-322); V\src\vs\workbench\contrib\terminalContrib\sendSequence\browser\terminal.sendSequence.contribution.ts (lines 124-151); C bundle (ids paste and pasteSelection; a send-sequence rule for Ctrl+V on terminalShellType pwsh)
- Pieces:
  1. Pressing `Ctrl+V` (`Cmd+V` on macOS) with a terminal focused pastes the clipboard text at the prompt, and the right-click menu has a `Paste` entry.
  2. `Ctrl+Shift+V` also pastes on Windows and is the paste key on Linux.
  3. On Linux, `Shift+Insert` pastes the selection clipboard through `workbench.action.terminal.pasteSelection`.
  4. Middle-clicking the terminal pastes when `terminal.integrated.middleClickBehavior` is `paste`.
  5. When the clipboard holds a copied file and no text, pasting inserts the file's path.
  6. On Windows in PowerShell with screen reader mode off, `Ctrl+V` is sent straight to the shell so PSReadLine handles the paste.
  7. In PowerShell on Windows, `Ctrl+V` with a copied file and no text on the clipboard still pastes the file's path through `workbench.action.terminal.pastePwsh`. [stock only]

#### PN-3.17 Multi-line paste warning

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pasting text with more than one line opens a warning dialog that asks whether to paste N lines, shows a preview of the first three lines, and offers Paste, Paste as one line and Cancel plus a "Do not ask me again" checkbox that sets the setting to never; in the default auto mode it is skipped when the shell is in bracketed paste mode and a single command with a trailing newline is pasted without the newline; a second setting makes the terminal ignore bracketed paste mode.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enableMultiLinePasteWarning; terminal.integrated.ignoreBracketedPasteMode
- Source: V\src\vs\workbench\contrib\terminalContrib\clipboard\browser\terminalClipboard.ts (lines 30-110); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 164-175, 653-657); C bundle (setting enableMultiLinePasteWarning with values auto, always and never; setting ignoreBracketedPasteMode)
- Pieces:
  1. Pasting text with more than one line opens a dialog that asks whether to paste that many lines and previews the first three.
  2. The dialog offers `Paste`, `Paste as one line` and `Cancel`.
  3. A `Do not ask me again` checkbox in the dialog sets the setting to `never`.
  4. In the default `auto` mode the dialog is skipped when the shell is in bracketed paste mode, and a single command with a trailing newline is pasted without the newline.
  5. Setting `terminal.integrated.enableMultiLinePasteWarning` chooses when the dialog shows, with values `auto`, `always` and `never`.
  6. Setting `terminal.integrated.ignoreBracketedPasteMode` makes the terminal ignore bracketed paste mode so the bracket sequences are left out of pasted text, off by default.

#### PN-3.18 Right-click menu and mouse click behavior

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the menu and all four settings are present; Cursor's TerminalInstanceContext also holds Split and New Terminal rows in group 1_create, which V lists only in TerminalEditorInstanceContext)
- What: Right-clicking the terminal shows a menu with Copy, Copy as HTML, Paste, Select All, Clear, Size to Content Width and Kill Terminal (plus Rerun Task in a task terminal); a setting changes right-click to copy-or-paste, paste, select the word under the pointer, or do nothing (Shift+right-click always shows the menu), middle-click can paste, Alt+click moves the prompt cursor to the click, and on macOS Option+click can force a normal line selection.
- Commands: workbench.action.terminal.copySelection; workbench.action.terminal.copySelectionAsHtml; workbench.action.terminal.paste; workbench.action.terminal.selectAll; workbench.action.terminal.clear; workbench.action.terminal.sizeToContentWidth; workbench.action.terminal.killViewOrEditor
- Menus: TerminalInstanceContext; TerminalEditorInstanceContext
- Keys: none found
- Settings: terminal.integrated.rightClickBehavior; terminal.integrated.middleClickBehavior; terminal.integrated.altClickMovesCursor; terminal.integrated.macOptionClickForcesSelection
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (lines 108-185 and 205-300); V\src\vs\workbench\contrib\terminalContrib\clipboard\browser\terminal.clipboard.contribution.ts (lines 124-163); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 149-158, 360-381); V\src\vs\workbench\contrib\terminal\browser\terminalTabbedView.ts (line 100); C bundle (TerminalInstanceContext menu items for split, new, killViewOrEditor, copySelection, copySelectionAsHtml, paste, clear, sizeToContentWidth and selectAll; the four settings)
- Pieces:
  1. Right-clicking the terminal shows a menu with `Copy`, `Copy as HTML`, `Paste`, `Select All`, `Clear`, `Size to Content Width` and `Kill Terminal`.
  2. In a task terminal the menu also has `Rerun Task`.
  3. Holding `Shift` while right-clicking always shows the menu, whatever the right-click setting says.
  4. Setting `terminal.integrated.rightClickBehavior` chooses what right-click does, with values `default` for the menu, `copyPaste`, `paste`, `selectWord` and `nothing`.
  5. Setting `terminal.integrated.middleClickBehavior` chooses what middle-click does, with values `default` for the platform default and `paste`.
  6. Setting `terminal.integrated.altClickMovesCursor` lets `Alt+click` move the prompt cursor to the click, on by default and only when `editor.multiCursorModifier` is `alt`.
  7. Setting `terminal.integrated.macOptionClickForcesSelection` makes `Option+click` on macOS force a regular line selection, off by default.
  8. In Cursor, the terminal right-click menu also has `Split` and `New Terminal` rows in a create group. [cursor only]

#### PN-3.19 Select text and word separators

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Select All selects the whole buffer (Cmd+A on macOS, or Select All in the right-click menu), Esc clears the selection, and double-clicking a word selects it up to the nearest word separator character (spaces, brackets, quotes, comma and similar by default).
- Commands: workbench.action.terminal.selectAll; workbench.action.terminal.clearSelection
- Menus: TerminalInstanceContext; TerminalEditorInstanceContext
- Keys: Cmd+A on selectAll (macOS only, when any terminal has focus); Esc on clearSelection when text is selected and the find box is not visible
- Settings: terminal.integrated.wordSeparators
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 695-705, 1195-1210); V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (lines 172-182); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 501-506); C bundle (ids selectAll and clearSelection; setting wordSeparators)
- Pieces:
  1. `Select All` in the right-click menu, or `Cmd+A` on macOS with any terminal focused, selects the whole buffer.
  2. Pressing `Esc` clears the selection when text is selected and the find box is not visible.
  3. Double-clicking a word selects it up to the nearest word separator character.
  4. Setting `terminal.integrated.wordSeparators` sets the separator characters, by default spaces, brackets, quotes, comma and a few others, and it also controls plain word link detection.

#### PN-3.20 Bell

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a program rings the terminal bell, the tab shows a warning bell icon with the tooltip "Bell" for a set time and the terminal bell signal can play a sound or announcement; the visual bell is off by default and the old enableBell setting is deprecated and migrated to the two newer settings.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enableVisualBell; terminal.integrated.bellDuration; terminal.integrated.enableBell; accessibility.signals.terminalBell
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 855-870); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 250-254, 413-421, 780-795); V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (setting accessibility.signals.terminalBell); C bundle (settings enableBell, enableVisualBell, bellDuration; signal setting accessibility.signals.terminalBell with its legacy key accessibility.alert.terminalBell)
- Pieces:
  1. When a program rings the terminal bell and the visual bell is on, the terminal's tab shows a warning bell icon with the tooltip `Bell`.
  2. Setting `terminal.integrated.enableVisualBell` turns the tab bell icon on or off, off by default.
  3. Setting `terminal.integrated.bellDuration` sets how many milliseconds the bell icon stays on the tab, 1000 by default.
  4. Setting `accessibility.signals.terminalBell` chooses a sound with `auto`, `on` or `off` and an announcement with `auto` or `off` when the bell rings.
  5. The old setting `terminal.integrated.enableBell` is deprecated and its settings entry points to the two newer settings.

#### PN-3.21 Terminal notifications

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A program in the terminal can send the OSC 99 escape sequence to raise a notification; it shows as a workbench notification from the source Terminal with the program's title, body and any buttons it asked for, plus Dismiss and Disable Terminal Notifications buttons, can focus the terminal when clicked, and can be limited to when the terminal is unfocused or hidden; turning the setting off ignores the sequence.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enableNotifications
- Source: V\src\vs\workbench\contrib\terminalContrib\notification\browser\terminalNotificationHandler.ts (lines 60-110, 300-370); V\src\vs\workbench\contrib\terminalContrib\notification\browser\terminal.notification.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\notification\common\terminalNotificationConfiguration.ts; C bundle (setting enableNotifications; handler action ids terminal.osc99.dismiss and terminal.osc99.disable)
- Pieces:
  1. When a program in the terminal sends the OSC 99 escape sequence, a workbench notification appears with the source Terminal and the program's title and body.
  2. The notification shows any buttons the program asked for, plus `Dismiss` and `Disable Terminal Notifications` buttons.
  3. Choosing `Disable Terminal Notifications` turns the notification setting off, so later sequences are ignored.
  4. A notification can focus the terminal when clicked, if the program asked for that.
  5. A program can limit its notification so it only shows while the terminal is unfocused or hidden.
  6. Setting `terminal.integrated.enableNotifications` turns OSC 99 notifications on or off, on by default.

#### PN-3.22 Accessible terminal view and help

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the five terminal commands and the terminal-help provider are present; the terminal find help provider, id terminalFindHelp, has no hit)
- What: With screen reader mode on, Alt+F2 in the terminal opens a read-only text view of the terminal buffer where Alt+Up and Alt+Down jump between commands and Ctrl+Home and Ctrl+End jump to the top or bottom; Alt+F1 opens an accessibility help page for the terminal (and a separate one while the find box has focus); two settings control whether the cursor position is kept and whether this view takes focus when a command runs.
- Commands: workbench.action.terminal.focusAccessibleBuffer; workbench.action.terminal.accessibleBufferGoToNextCommand; workbench.action.terminal.accessibleBufferGoToPreviousCommand; workbench.action.terminal.scrollToBottomAccessibleView; workbench.action.terminal.scrollToTopAccessibleView; editor.action.accessibilityHelp
- Menus: none found
- Keys: Alt+F2 (Alt+Shift+F2 on Linux) on focusAccessibleBuffer when screen reader mode is on and the terminal has focus; Alt+Down and Alt+Up on the two go-to-command commands while the accessible view shows the terminal; Ctrl+End and Ctrl+Home (Shift+End and Shift+Home on Linux) on the scroll commands in that view; Alt+F1 (Alt+Shift+F1 on Linux) on editor.action.accessibilityHelp
- Settings: terminal.integrated.accessibleViewPreserveCursorPosition; terminal.integrated.accessibleViewFocusOnCommandExecution; accessibility.verbosity.terminal
- Source: V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminal.accessibility.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminalAccessibilityHelp.ts; V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminalAccessibleBufferProvider.ts; V\src\vs\workbench\contrib\terminalContrib\find\browser\terminalFindAccessibilityHelp.ts; V\src\vs\workbench\contrib\terminalContrib\accessibility\common\terminalAccessibilityConfiguration.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 160-200); C bundle (the five ids; provider id terminal-help; settings accessibleViewPreserveCursorPosition and accessibleViewFocusOnCommandExecution; setting accessibility.verbosity.terminal)
- Pieces:
  1. With screen reader mode on, pressing `Alt+F2` (`Alt+Shift+F2` on Linux) in the terminal opens a read-only text view of the terminal buffer.
  2. In that view `Alt+Up` and `Alt+Down` jump to the previous and next command.
  3. In that view `Ctrl+Home` and `Ctrl+End` (`Shift+Home` and `Shift+End` on Linux) jump to the top or bottom of the buffer.
  4. Pressing `Alt+F1` (`Alt+Shift+F1` on Linux) in the terminal opens an accessibility help page for the terminal.
  5. Pressing `Alt+F1` while the terminal find box has focus opens a separate accessibility help page for terminal find. [stock only]
  6. Setting `terminal.integrated.accessibleViewPreserveCursorPosition` keeps the cursor position when the view is reopened instead of moving it to the bottom of the buffer, off by default.
  7. Setting `terminal.integrated.accessibleViewFocusOnCommandExecution` moves focus to this view when a command runs, off by default.
  8. Setting `accessibility.verbosity.terminal` chooses whether the terminal announces how to open its accessibility help.

#### PN-3.23 Command result sounds and announcements

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With shell integration active, a finished command can play a sound or screen reader announcement for success or for failure (also when you move to such a command in the accessible view), and a separate signal fires when terminal quick fixes become available.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: accessibility.signals.terminalCommandFailed; accessibility.signals.terminalCommandSucceeded; accessibility.signals.terminalQuickFix
- Source: V\src\vs\workbench\contrib\terminal\browser\xterm\decorationAddon.ts (lines 248-256); V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminal.accessibility.contribution.ts (lines 190-198); V\src\vs\workbench\contrib\terminalContrib\quickFix\browser\quickFixAddon.ts (line 310); V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (lines 475-515); C bundle (the three signal settings)
- Pieces:
  1. With shell integration active, a command that finishes with a zero exit code can play a sound or an announcement for success.
  2. A command that finishes with a non-zero exit code can play a sound or an announcement for failure, and moving to such a command in the accessible view does the same.
  3. A separate sound or announcement fires when terminal quick fixes become available.
  4. Settings `accessibility.signals.terminalCommandSucceeded`, `accessibility.signals.terminalCommandFailed` and `accessibility.signals.terminalQuickFix` each choose a sound with `auto`, `on` or `off` and an announcement with `auto` or `off`.

#### PN-3.24 Screen reader mode in the terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a screen reader is detected or accessibility support is on, the terminal switches xterm to screen reader mode, mirrors the prompt text into a hidden text area, tells the shell process about it (PowerShell does not run PSReadLine under a screen reader, so the PowerShell-only Ctrl+V rule is switched off), turns off the Ctrl+Up and Ctrl+Down command jumps, uses Ctrl+R for Run Recent Command, and labels its input box with the terminal number, the title and the accessibility help key (or a hint to turn screen reader mode on when it is off).
- Commands: none found
- Menus: none found
- Keys: Ctrl+R on runRecentCommand in screen reader mode; Ctrl+Alt+R sends Ctrl+R to the shell in screen reader mode
- Settings: editor.accessibilitySupport; accessibility.verbosity.terminal
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 1985-1988, 2098-2114); V\src\vs\workbench\contrib\terminal\browser\terminalProcessManager.ts (lines 313, 525); V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\textAreaSyncAddon.ts (lines 15-70); V\src\vs\workbench\contrib\terminalContrib\sendSequence\browser\terminal.sendSequence.contribution.ts (lines 124-136, 200-205); V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 872-909); C bundle (xterm option screenReaderMode set from the accessibility service; module id terminal.textAreaSync)
- Pieces:
  1. When a screen reader is detected or `editor.accessibilitySupport` is on, the terminal switches to screen reader mode, mirrors the prompt text into a hidden text area, and tells the shell process a screen reader is in use.
  2. In PowerShell on Windows under a screen reader, `Ctrl+V` uses the normal paste instead of being sent straight to the shell.
  3. `Ctrl+Up` and `Ctrl+Down` no longer jump between commands in screen reader mode.
  4. `Ctrl+R` opens `Run Recent Command` in screen reader mode, and `Ctrl+Alt+R` sends `Ctrl+R` to the shell instead.
  5. The terminal's input box is labeled with the terminal number, the title and the accessibility help key, or with a hint to turn screen reader mode on when it is off.
  6. Setting `editor.accessibilitySupport` chooses whether screen reader mode is used, with values `auto`, `on` and `off`.
  7. Setting `accessibility.verbosity.terminal` chooses whether the help key hint is included in the input box label.

#### PN-3.25 GPU acceleration

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Choose whether the terminal draws with the GPU (auto, on or off; off renders much slower but works everywhere), plus two related options that draw box and block characters directly and that squeeze wide glyphs back into their cell.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.gpuAcceleration; terminal.integrated.customGlyphs; terminal.integrated.rescaleOverlappingGlyphs
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 334-343, 575-592); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts; C bundle (the three settings)
- Pieces:
  1. Setting `terminal.integrated.gpuAcceleration` chooses whether the terminal draws with the GPU, with values `auto`, `on` and `off`, where `off` renders much slower but works on all systems.
  2. Setting `terminal.integrated.customGlyphs` draws box, block and similar characters directly instead of with the font, on by default and only when GPU acceleration is on.
  3. Setting `terminal.integrated.rescaleOverlappingGlyphs` squeezes wide single-cell glyphs back into their cell, on by default.

#### PN-3.26 Cursor style

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick the terminal cursor shape (block, line or underline), the shape it takes when the terminal is not focused (outline, block, line, underline or none), the width of the line cursor, and whether the cursor and blinking text blink.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.cursorStyle; terminal.integrated.cursorStyleInactive; terminal.integrated.cursorWidth; terminal.integrated.cursorBlinking; terminal.integrated.textBlinking
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 293-318); C bundle (all five settings)
- Pieces:
  1. Setting `terminal.integrated.cursorStyle` chooses the cursor shape while the terminal is focused, with values `block`, `line` and `underline`.
  2. Setting `terminal.integrated.cursorStyleInactive` chooses the cursor shape while the terminal is not focused, with values `outline`, `block`, `line`, `underline` and `none`.
  3. Setting `terminal.integrated.cursorWidth` sets the width of the cursor when the style is `line`.
  4. Setting `terminal.integrated.cursorBlinking` makes the cursor blink, off by default.
  5. Setting `terminal.integrated.textBlinking` lets blinking text blink, off by default.

#### PN-3.27 Font and text appearance

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Set the terminal font family, size, weight and bold weight, line height, letter spacing, tab stop width and Unicode width table, set a minimum contrast ratio (default 4.5) that nudges text colors, and choose whether bold text uses the bright color variants.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.fontFamily; terminal.integrated.fontSize; terminal.integrated.fontWeight; terminal.integrated.fontWeightBold; terminal.integrated.lineHeight; terminal.integrated.letterSpacing; terminal.integrated.tabStopWidth; terminal.integrated.unicodeVersion; terminal.integrated.minimumContrastRatio; terminal.integrated.drawBoldTextInBrightColors
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 172-182, 228-238, 255-292, 533-541); C bundle (all ten settings)
- Pieces:
  1. Settings `terminal.integrated.fontFamily` and `terminal.integrated.fontSize` set the font and size the terminal draws with.
  2. Settings `terminal.integrated.fontWeight` and `terminal.integrated.fontWeightBold` set normal and bold text weight, as `normal`, `bold` or a number from 1 to 1000.
  3. Settings `terminal.integrated.lineHeight` and `terminal.integrated.letterSpacing` change the spacing between lines and between characters.
  4. Setting `terminal.integrated.tabStopWidth` sets how many cells a tab stop spans.
  5. Setting `terminal.integrated.unicodeVersion` chooses the Unicode width table, with values `6` and `11`.
  6. Setting `terminal.integrated.minimumContrastRatio` nudges text colors to meet a contrast ratio, with a default of 4.5 and a value of 1 turning it off.
  7. Setting `terminal.integrated.drawBoldTextInBrightColors` chooses whether bold text always uses the bright color variants.

#### PN-3.28 Font ligatures

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Turn on font ligatures (off by default) so the configured terminal font draws sequences such as arrows as one glyph, tune the CSS font-feature-settings used with them, and give a fallback list of character sequences to join when the font cannot be parsed.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.fontLigatures.enabled; terminal.integrated.fontLigatures.featureSettings; terminal.integrated.fontLigatures.fallbackLigatures
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 184-206); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts; C bundle (the three settings; the xterm ligatures addon is loaded)
- Pieces:
  1. With ligatures turned on and a font that supports them, character sequences such as arrows draw as one glyph in the terminal.
  2. Setting `terminal.integrated.fontLigatures.enabled` turns ligatures on or off, off by default.
  3. Setting `terminal.integrated.fontLigatures.featureSettings` takes CSS `font-feature-settings` text for the ligatures, `"calt" on` by default.
  4. Setting `terminal.integrated.fontLigatures.fallbackLigatures` lists character sequences that are always drawn together when the font cannot be parsed and GPU acceleration is on.

#### PN-3.29 Inline images

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When turned on (off by default) and GPU rendering is on, images that a program sends with Sixel, the iTerm inline image protocol (Linux and macOS) or the kitty graphics protocol (all platforms) are drawn in the terminal, and transparency mode is switched on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enableImages
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 658-664); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts; C bundle (setting enableImages; the xterm image addon is loaded and refreshed)
- Pieces:
  1. With the setting on and GPU rendering on, images a program sends with the Sixel protocol are drawn in the terminal.
  2. Images sent with the iTerm inline image protocol are drawn on Linux and macOS, and images sent with the kitty graphics protocol are drawn on all platforms.
  3. Turning the setting on also turns on transparency mode in the terminal.
  4. Setting `terminal.integrated.enableImages` turns image support on or off, off by default.
  5. Inline images are not restored after a window reload or reconnect.

#### PN-3.30 Zoom the terminal font

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Increase Font Size, Decrease Font Size and Reset Font Size in the Command Palette change the terminal font size one step at a time (6 to 100), and holding Ctrl (Cmd on macOS) while scrolling the mouse wheel over a terminal zooms it when that setting is on.
- Commands: workbench.action.terminal.fontZoomIn; workbench.action.terminal.fontZoomOut; workbench.action.terminal.fontZoomReset
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.mouseWheelZoom; terminal.integrated.fontSize
- Source: V\src\vs\workbench\contrib\terminalContrib\zoom\browser\terminal.zoom.contribution.ts (lines 40-170); V\src\vs\workbench\contrib\terminalContrib\zoom\common\terminal.zoom.ts; C bundle (the three ids; setting mouseWheelZoom)
- Pieces:
  1. `Increase Font Size` in the Command Palette makes the terminal font one step larger, up to 100.
  2. `Decrease Font Size` in the Command Palette makes the terminal font one step smaller, down to 6.
  3. `Reset Font Size` in the Command Palette puts the terminal font back to its default size.
  4. Setting `terminal.integrated.mouseWheelZoom` lets `Ctrl` (`Cmd` on macOS) plus the mouse wheel over a terminal zoom its font, off by default.
  5. Each zoom step is saved into setting `terminal.integrated.fontSize`.

#### PN-3.31 Text antialiasing

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: newer-than-1.128. An experimental, advanced setting that is listed only on macOS lets the terminal follow the workbench font aliasing or force grayscale antialiasing on high-DPI displays; with GPU rendering it may need a window reload.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.fontRendering
- Source: N\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 183-194); N\src\vs\workbench\contrib\terminal\browser\xterm\terminalFontRendering.ts; C bundle (zero hits for fontRendering)
- Pieces:
  1. On macOS, setting `terminal.integrated.fontRendering` set to `inherit` makes terminal text follow the workbench font aliasing setting. [stock only]
  2. Setting `terminal.integrated.fontRendering` set to `grayscale` draws terminal text with grayscale antialiasing on high-DPI displays, which can look sharper. [stock only]
  3. With GPU rendering on, changing this setting to `grayscale` may need `Developer: Reload Window` before it takes effect. [stock only]
  4. The setting is tagged experimental and advanced and is listed only on macOS. [stock only]

#### PN-3.32 Size overlay while resizing

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While the terminal is being resized, a small box over it shows the new size in columns and rows (for example "80 x 24") and fades out half a second after the last change; it is announced politely to screen readers.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\resizeDimensionsOverlay\browser\terminalResizeDimensionsOverlay.ts (lines 14-62); V\src\vs\workbench\contrib\terminal\terminal.all.ts (line 33); C bundle (module id terminal.resizeDimensionsOverlay)
- Pieces:
  1. While the terminal is being resized, a small box over it shows the new size in columns and rows, for example `80 x 24`.
  2. The box fades out half a second after the last size change.
  3. The box is announced politely to screen readers each time its text changes.

#### PN-3.33 Scroll the terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Scroll by one line, by one page, or to the top or bottom from the keyboard while a program is not using the alternate screen, and tune smooth scrolling, the mouse wheel speed and the Alt-key fast scroll multiplier; the size of the scrollback buffer is described in PN-2.39.
- Commands: workbench.action.terminal.scrollDown; workbench.action.terminal.scrollUp; workbench.action.terminal.scrollDownPage; workbench.action.terminal.scrollUpPage; workbench.action.terminal.scrollToBottom; workbench.action.terminal.scrollToTop
- Menus: none found
- Keys: Ctrl+Alt+PageDown and Ctrl+Alt+PageUp on the line commands (Ctrl+Shift+Down and Ctrl+Shift+Up on Linux); Shift+PageDown and Shift+PageUp on the page commands (plain PageDown and PageUp on macOS); Ctrl+End and Ctrl+Home on bottom and top (Shift+End and Shift+Home on Linux); all with a terminal in focus in the normal buffer
- Settings: terminal.integrated.smoothScrolling; terminal.integrated.mouseWheelScrollSensitivity; terminal.integrated.fastScrollSensitivity
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 612-693); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 240-249, 648-652); C bundle (all six ids; the three settings)
- Pieces:
  1. Pressing `Ctrl+Alt+PageDown` or `Ctrl+Alt+PageUp` (`Ctrl+Shift+Down` or `Ctrl+Shift+Up` on Linux) scrolls the terminal down or up one line.
  2. Pressing `Shift+PageDown` or `Shift+PageUp` (plain `PageDown` or `PageUp` on macOS) scrolls the terminal down or up one page.
  3. Pressing `Ctrl+End` or `Ctrl+Home` (`Shift+End` or `Shift+Home` on Linux) scrolls the terminal to the bottom or the top.
  4. These scroll keys only apply while the terminal shows its normal buffer and not a program's alternate screen.
  5. Setting `terminal.integrated.smoothScrolling` makes the terminal scroll with an animation, off by default.
  6. Setting `terminal.integrated.mouseWheelScrollSensitivity` multiplies the mouse wheel scroll distance, 1 by default.
  7. Setting `terminal.integrated.fastScrollSensitivity` multiplies the scroll speed while `Alt` is held, 5 by default.

#### PN-3.34 Clear the terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clear empties the terminal output buffer (Cmd+K on macOS, the Clear row in the right-click menu, or the Command Palette) and also works from the accessible terminal view; no separate Reset Terminal command exists in 1.128.0.
- Commands: workbench.action.terminal.clear
- Menus: TerminalInstanceContext; TerminalEditorInstanceContext
- Keys: Cmd+K on macOS when the terminal has focus and screen reader mode is off, or when the accessible view shows the terminal; none on Windows and Linux
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 1354-1370); V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (lines 152-162); C bundle (id clear)
- Pieces:
  1. `Clear` in the Command Palette empties the terminal output buffer.
  2. The `Clear` row in the terminal right-click menu does the same, in the panel and in a terminal editor tab.
  3. Pressing `Cmd+K` on macOS clears the terminal when it has focus with screen reader mode off, or when the accessible view shows the terminal, and Windows and Linux have no default key.

#### PN-3.35 Send a sequence or a signal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Send Sequence asks for text (it understands \n, \r and \xAB escapes, and resolves variables) or takes a text argument from a keybinding and types it into the active terminal; Send Signal, in the Command Palette on Linux and macOS only, lists SIGINT, SIGTERM, SIGKILL, SIGSTOP, SIGCONT, SIGHUP, SIGQUIT, SIGUSR1 and SIGUSR2 or lets you type a signal name and sends it to the terminal's process.
- Commands: workbench.action.terminal.sendSequence; workbench.action.terminal.sendSignal
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\sendSequence\browser\terminal.sendSequence.contribution.ts (lines 30-120); V\src\vs\workbench\contrib\terminalContrib\sendSignal\browser\terminal.sendSignal.contribution.ts (lines 1-95); C bundle (ids sendSequence and sendSignal)
- Pieces:
  1. `Send Sequence` in the Command Palette asks `Enter sequence to send to the terminal` and types the answer into the active terminal.
  2. The typed text understands `\n`, `\r` and `\xAB` escapes and resolves variables before it is sent.
  3. A keybinding can give `Send Sequence` a text argument, so one key types fixed text into the terminal without asking.
  4. `Send Signal` in the Command Palette, on Linux and macOS only, lists SIGINT, SIGTERM, SIGKILL, SIGSTOP, SIGCONT, SIGHUP, SIGQUIT, SIGUSR1 and SIGUSR2 with a short description each and sends the chosen signal to the terminal's process.
  5. Typing a signal name that is not in the list into the `Send Signal` pick sends that signal.

#### PN-3.36 Keys that go to the shell

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Most keys in a focused terminal go to the shell, and settings decide which workbench keybindings still win (a list of commands that always skip the shell, a switch to send nearly everything to the shell, whether chords and Alt menu mnemonics are allowed, Option as Meta on macOS, the kitty keyboard protocol and win32 input mode); built-in rules also rewrite some keys for the shell, such as Alt+arrows to word movement, Ctrl+Backspace and Ctrl+Delete to delete a word, Cmd+Left, Cmd+Right and Cmd+Backspace on macOS to line start, line end and delete to line start, and a set of PowerShell-only shortcuts.
- Commands: workbench.action.terminal.sendSequence
- Menus: none found
- Keys: Alt+Up and Alt+Down send Ctrl+Up and Ctrl+Down; Alt+Left and Alt+Right send word movement; Ctrl+Backspace (Option+Backspace on macOS) and Ctrl+Delete (Option+Delete on macOS) delete a word; Cmd+Left, Cmd+Right and Cmd+Backspace on macOS; Ctrl+Shift+2 sends NUL; Ctrl+Shift+6 sends RS; Ctrl+Alt+G sends Ctrl+G; Ctrl+Space, Alt+Space and Shift+Enter in PowerShell with shell integration and win32 input mode; all with a terminal in focus
- Settings: terminal.integrated.commandsToSkipShell; terminal.integrated.sendKeybindingsToShell; terminal.integrated.allowChords; terminal.integrated.allowMnemonics; terminal.integrated.macOptionIsMeta; terminal.integrated.enableKittyKeyboardProtocol; terminal.integrated.enableWin32InputMode
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 1148-1192); V\src\vs\workbench\contrib\terminal\browser\terminalConfigurationService.ts (lines 25-75); V\src\vs\workbench\contrib\terminal\common\terminal.ts (line 502, default skip list); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 47-52, 144-148, 423-446, 593-606); V\src\vs\workbench\contrib\terminalContrib\sendSequence\browser\terminal.sendSequence.contribution.ts (lines 108-260); C bundle (the seven settings; send-sequence key rules such as Alt+Up with the Ctrl+Up sequence)
- Pieces:
  1. Most keys pressed in a focused terminal go to the shell instead of triggering workbench keybindings.
  2. Setting `terminal.integrated.commandsToSkipShell` lists command ids whose keys always stay with the workbench, and an id with a leading `-` hands a default skipped command back to the shell.
  3. Setting `terminal.integrated.sendKeybindingsToShell` sends nearly every key to the shell and overrides the skip list.
  4. Settings `terminal.integrated.allowChords` and `terminal.integrated.allowMnemonics` choose whether keybinding chords and `Alt` menu mnemonics still work while the terminal has focus.
  5. Setting `terminal.integrated.macOptionIsMeta` makes `Option` act as the Meta key on macOS.
  6. Settings `terminal.integrated.enableKittyKeyboardProtocol` and `terminal.integrated.enableWin32InputMode` turn on the kitty keyboard protocol and win32 input mode, so programs can read more detailed key input.
  7. `Alt+Up` and `Alt+Down` send `Ctrl+Up` and `Ctrl+Down`, `Alt+Left` and `Alt+Right` send word movement, and `Ctrl+Backspace` and `Ctrl+Delete` (`Option+Backspace` and `Option+Delete` on macOS) delete a word.
  8. On macOS, `Cmd+Left` and `Cmd+Right` move to the start and end of the line and `Cmd+Backspace` deletes to the start of the line.
  9. `Ctrl+Shift+2` sends NUL, `Ctrl+Shift+6` sends RS and `Ctrl+Alt+G` sends `Ctrl+G` to the shell.
  10. In PowerShell with shell integration and win32 input mode on, `Ctrl+Space`, `Alt+Space` and `Shift+Enter` are mapped to keys that its line editor handles.

#### PN-3.37 Environment variable collection indicator

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the commands, the statuses and the relaunch setting are present; Cursor also keeps a terminal.integrated.environmentChangesIndicator setting with off, on and warnonly, default warnonly, which V 1.128.0 does not define)
- What: When extensions change a terminal's environment, its tab gets a status whose detail lists which extensions contributed, with a Show Environment Contributions link that opens a read-only "Terminal Environment Changes" document grouped by extension and workspace folder; if the change needs a restart the status is a warning that lists the extensions and offers Relaunch Terminal, and a terminal you have not typed into relaunches by itself when the relaunch setting is on; the input box label also says the environment is stale.
- Commands: workbench.action.terminal.showEnvironmentContributions; workbench.action.terminal.relaunch
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.environmentChangesRelaunch
- Source: V\src\vs\workbench\contrib\terminal\browser\environmentVariableInfo.ts (lines 28-95); V\src\vs\workbench\contrib\terminalContrib\environmentChanges\browser\terminal.environmentChanges.contribution.ts (lines 20-95); V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 2304-2345); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 474-478); C bundle (ids showEnvironmentContributions and relaunch; status ids relaunch-needed and env-var-info-changes-active; setting environmentChangesIndicator at its definition with enum off, on and warnonly)
- Pieces:
  1. When extensions change a terminal's environment, the terminal's tab gets a status whose detail lists the extensions that contributed.
  2. The status has a `Show Environment Contributions` link that opens a read-only `Terminal Environment Changes` document grouped by extension and workspace folder.
  3. When a change needs a restart, the status becomes a warning that lists the extensions and offers `Relaunch Terminal`.
  4. A terminal you have not typed into relaunches by itself when setting `terminal.integrated.environmentChangesRelaunch` is on.
  5. The terminal's input box label also says the environment is stale when a relaunch is needed.
  6. In Cursor, setting `terminal.integrated.environmentChangesIndicator` chooses when the tab shows the environment status, with values `off`, `on` and `warnonly`, and `warnonly` by default. [cursor only]

#### PN-3.38 Local echo

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: On a slow connection, typed characters can be echoed locally in a dim style before the shell echoes them; it can be on, off or on only for remote workspaces (off by default), starts after a latency threshold, skips listed programs, and the echo style can be a font style or a color.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.localEchoEnabled; terminal.integrated.localEchoLatencyThreshold; terminal.integrated.localEchoExcludePrograms; terminal.integrated.localEchoStyle
- Source: V\src\vs\workbench\contrib\terminalContrib\typeAhead\common\terminalTypeAheadConfiguration.ts; V\src\vs\workbench\contrib\terminalContrib\typeAhead\browser\terminalTypeAheadAddon.ts; V\src\vs\workbench\contrib\terminal\terminal.all.ts (line 32); C bundle (zero hits for localEcho, typeAhead and TypeAhead)
- Pieces:
  1. On a slow connection, typed characters can be echoed locally in a dim style before the shell echoes them. [stock only]
  2. Setting `terminal.integrated.localEchoEnabled` chooses when local echo is on, with values `on`, `off` and `auto` for remote workspaces only, and `off` by default. [stock only]
  3. Setting `terminal.integrated.localEchoLatencyThreshold` sets the delay in milliseconds after which local echo starts, 30 by default, where 0 means always on and -1 means disabled. [stock only]
  4. Setting `terminal.integrated.localEchoExcludePrograms` lists program names that turn local echo off when they appear in the terminal title. [stock only]
  5. Setting `terminal.integrated.localEchoStyle` chooses the echo style as `bold`, `dim`, `italic`, `underlined`, `inverted` or a hex color, `dim` by default. [stock only]

#### PN-3.39 Drop a file to type its path

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Dragging a file or folder from the Explorer or the operating system onto a terminal shows a drop overlay and, on drop, types the item's path at the prompt, quoted for the shell, without running it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 1259-1268, 1405-1415, 2549-2652); C bundle (css class terminal-drop-overlay and the drop controller code that reads the drop side)
- Pieces:
  1. Dragging a file or folder from the Explorer or from the operating system over a terminal shows a drop overlay on it.
  2. Dropping the item types its path at the prompt without running it.
  3. The typed path is quoted for the shell the terminal is running.

#### PN-3.40 Terminal developer diagnostics

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Command Palette has Developer entries for the terminal: Show Terminal Texture Atlas (saves textureAtlas.png in the first workspace folder and opens it), Write Data to Terminal (prints raw data to the terminal bypassing the process), Record Terminal Session (shows "Recording terminal session..." in the status bar and copies the recorded events to the clipboard as JSON when it ends), and Restart Pty Host; a developer mode setting adds prompt-input debugging to each terminal.
- Commands: workbench.action.terminal.showTextureAtlas; workbench.action.terminal.writeDataToTerminal; workbench.action.terminal.recordSession; workbench.action.terminal.restartPtyHost
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.developer.devMode
- Source: V\src\vs\workbench\contrib\terminalContrib\developer\browser\terminal.developer.contribution.ts (lines 31-215 and 217-362); V\src\vs\workbench\contrib\terminalContrib\developer\common\terminal.developer.ts; C bundle (the four ids; setting developer.devMode)
- Pieces:
  1. `Developer: Show Terminal Texture Atlas` in the Command Palette saves `textureAtlas.png` in the first workspace folder and opens it.
  2. `Developer: Write Data to Terminal` asks for data and prints it directly in the terminal without sending it to the process.
  3. `Developer: Record Terminal Session` shows `Recording terminal session...` in the status bar and copies the recorded events to the clipboard as JSON when recording ends.
  4. `Developer: Restart Pty Host` restarts the process host that runs the terminals.
  5. Setting `terminal.integrated.developer.devMode` adds prompt-input debugging to each terminal.

#### PN-3.41 Terminal text, cursor and selection colors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A color theme or `workbench.colorCustomizations` sets the terminal's own background (terminal.background, which falls back to the editor background for a terminal in an editor tab, the panel background in the panel, or the side bar background in the side bar), the default text color (terminal.foreground), the cursor color (terminalCursor.foreground, which falls back to the text color), the color of the character under a block cursor (terminalCursor.background), the selection background (terminal.selectionBackground, which follows the editor selection color), a dimmer selection background for when the terminal is not focused (terminal.inactiveSelectionBackground), and an optional selection text color (terminal.selectionForeground); there is no settings page for them.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Source: V\src\vs\workbench\contrib\terminal\common\terminalColorRegistry.ts (lines 18-38); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts (getXtermTheme, lines 1009-1035); V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (getBackgroundColor, lines 2901-2914); C bundle (all seven color ids found, with the same registry text and defaults)
- Pieces:
  1. Theme color `terminal.background` sets the terminal's own background, and without a value it falls back to the editor background in an editor tab, the panel background in the panel and the side bar background in the side bar.
  2. Theme color `terminal.foreground` sets the default text color.
  3. Theme colors `terminalCursor.foreground` and `terminalCursor.background` set the cursor color, which falls back to the text color, and the color of the character under a block cursor.
  4. Theme color `terminal.selectionBackground` sets the selection background and follows the editor selection color by default.
  5. Theme color `terminal.inactiveSelectionBackground` sets a dimmer selection background for when the terminal is not focused.
  6. Theme color `terminal.selectionForeground` sets an optional color for selected text.
  7. Setting `workbench.colorCustomizations` overrides these colors, and there is no settings page for them.

#### PN-3.42 ANSI color palette

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A theme or `workbench.colorCustomizations` sets the 16 colors that programs use for colored output - black, red, green, yellow, blue, magenta, cyan and white, plus a bright version of each - as terminal.ansiBlack through terminal.ansiBrightWhite; each has its own default for light, dark and high contrast themes, and the minimum contrast ratio in PN-3.27 can still nudge a color for readability.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Source: V\src\vs\workbench\contrib\terminal\common\terminalColorRegistry.ts (ansiColorMap and its registration loop, lines 102-253); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts (lines 1037-1052); C bundle (all 16 ids found; the registry copy has the same defaults, for example ansiBlack index 0 with #000000 in light and dark)
- Pieces:
  1. Theme colors `terminal.ansiBlack`, `terminal.ansiRed`, `terminal.ansiGreen`, `terminal.ansiYellow`, `terminal.ansiBlue`, `terminal.ansiMagenta`, `terminal.ansiCyan` and `terminal.ansiWhite` color the normal output colors programs use.
  2. Theme colors `terminal.ansiBrightBlack` through `terminal.ansiBrightWhite` color the bright version of each of the eight colors.
  3. Each of the 16 colors has its own default for light, dark and high contrast themes.
  4. Setting `workbench.colorCustomizations` overrides any of the 16 colors.
  5. The minimum contrast ratio setting can still nudge a color for readability.

#### PN-3.43 Highlight and marker colors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A theme or `workbench.colorCustomizations` also colors the terminal's helper marks: the current find match and the other matches (fill and border, terminal.findMatchBackground, terminal.findMatchBorder, terminal.findMatchHighlightBackground, terminal.findMatchHighlightBorder), their marker in the overview ruler (terminalOverviewRuler.findMatchForeground), the ruler marker for the active match or for the line you just jumped to (terminalOverviewRuler.cursorForeground), the left border of the ruler (terminalOverviewRuler.border), the highlight under a word whose link hover is showing (terminal.hoverHighlightBackground), the three command dots (terminalCommandDecoration.defaultBackground, successBackground and errorBackground), and the fill shown over the terminal while something is dragged onto it (terminal.dropBackground).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Source: V\src\vs\workbench\contrib\terminal\common\terminalColorRegistry.ts (lines 40-93); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts (_updateFindColors, lines 621-642); V\src\vs\workbench\contrib\terminal\browser\xterm\decorationAddon.ts (line 612); V\src\vs\workbench\contrib\terminal\browser\xterm\markNavigationAddon.ts (line 389); V\src\vs\workbench\contrib\terminal\browser\media\terminal.css (lines 44 and 616); C bundle (all 12 ids found)
- Pieces:
  1. Theme colors `terminal.findMatchBackground` and `terminal.findMatchBorder` color the current find match.
  2. Theme colors `terminal.findMatchHighlightBackground` and `terminal.findMatchHighlightBorder` color the other find matches.
  3. Theme color `terminalOverviewRuler.findMatchForeground` colors the find match markers in the overview ruler.
  4. Theme color `terminalOverviewRuler.cursorForeground` colors the ruler marker for the active match or for the line you just jumped to.
  5. Theme color `terminalOverviewRuler.border` colors the left border of the overview ruler.
  6. Theme color `terminal.hoverHighlightBackground` colors the highlight under a word whose link hover is showing.
  7. Theme colors `terminalCommandDecoration.defaultBackground`, `terminalCommandDecoration.successBackground` and `terminalCommandDecoration.errorBackground` color the three kinds of command dot.
  8. Theme color `terminal.dropBackground` colors the fill shown over the terminal while something is dragged onto it.
  9. Setting `workbench.colorCustomizations` overrides any of these colors.

#### PN-3.44 Auto replies

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A setting holds a list of messages the terminal should watch for and the reply to type when one shows up, for example "Terminate batch job (Y/N)" answered with Y and Enter (written as \r), so routine prompts are answered without you; a reply can fire at most once a second, the match runs on the raw output so styled text may not match, a null value switches off a default, the setting is empty by default, and it is restricted so a workspace in Restricted Mode cannot use it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.autoReplies
- Source: V\src\vs\workbench\contrib\terminalContrib\autoReplies\common\terminalAutoRepliesConfiguration.ts (lines 7-33); V\src\vs\workbench\contrib\terminalContrib\autoReplies\browser\terminal.autoReplies.contribution.ts (lines 14-55); C bundle (module terminalAutoRepliesConfiguration.js with default {} and restricted true; the config-change listener that calls uninstallAllAutoReplies)
- Pieces:
  1. When the terminal output contains a message listed in setting `terminal.integrated.autoReplies`, the terminal types the configured reply, for example answering `Terminate batch job (Y/N)` with `Y`.
  2. A reply written with `\r` presses Enter after the text.
  3. A reply fires at most once a second.
  4. The match runs on the raw output, so text that is styled or split by escape codes may not match.
  5. Setting a message to `null` switches off a default reply for it.
  6. The setting is empty by default and is restricted, so a workspace in Restricted Mode cannot use it.

#### PN-3.45 Tab key in the terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a focused terminal the Tab key and Shift+Tab go to the shell, so shell completion works and focus does not jump away; Toggle Tab Key Moves Focus (Ctrl+M, or Ctrl+Shift+M on macOS) is on the terminal's list of keys that skip the shell, so it still works there, and once Tab Moves Focus is on, Tab leaves the terminal and moves focus to the next control instead; the command and its status bar item are described in ED-5 and SS-3.
- Commands: editor.action.toggleTabFocusMode
- Menus: none found
- Keys: Ctrl+M (Ctrl+Shift+M on macOS) on editor.action.toggleTabFocusMode
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 1168-1180); V\src\vs\workbench\contrib\terminal\common\terminal.ts (line 560, default skip-shell list); V\src\vs\editor\contrib\toggleTabFocusMode\browser\toggleTabFocusMode.ts (lines 15-25); C bundle (the id is in the skip-shell list; the toggle command class carries the same id)
- Pieces:
  1. In a focused terminal, `Tab` and `Shift+Tab` go to the shell, so shell completion works and focus does not jump away.
  2. Pressing `Ctrl+M` (`Ctrl+Shift+M` on macOS) toggles Tab Moves Focus and still works inside the terminal because it is on the list of keys that skip the shell.
  3. Once Tab Moves Focus is on, `Tab` leaves the terminal and moves focus to the next control.

#### PN-3.46 Localhost links open in the Cursor Browser

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: In Cursor, following an http or https link in the terminal that points at this machine (localhost, a loopback address or a .localhost name) opens it in a Cursor Browser tab, with no "Opening ... in Cursor Browser" toast, and holding Shift while following it sends it to the system browser instead; the link hover then reads like "Follow link (ctrl + click, hold shift for your browser)"; it does not apply in a remote window or when the Cursor Settings Browser toggle "Open Local Links in Cursor Browser" is off, where the link goes to the system browser as in stock VS Code. Boundary reason: the link lands in the Cursor Browser, which is an agent-driven surface.
- Commands: workbench.action.openBrowserEditor
- Menus: none found
- Keys: Shift held while following the link forces the system browser; the follow modifier itself is the one in PN-3.3
- Settings: autoOpenLocalhostUrls (a Cursor application storage key shown on the Cursor Settings Browser page, default on; it is not a settings.json id)
- Source: C bundle (the terminal URL link opener near the string "Tried to open a url without a resolved URI", which sets openExternal from the localhost check and Shift key; the link hover builder that adds NLS message 10847 ", hold shift for your browser"; the opener service branch that runs workbench.action.openBrowserEditor; the storage default autoOpenLocalhostUrls true; the Cursor Settings entry labelled "Open Local Links in Cursor Browser"); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkOpeners.ts (TerminalUrlLinkOpener, which always passes openExternal true in stock). I did not open Cursor Settings or click a link on screen.
- Pieces:
  1. Following an http or https link in the terminal that points at this machine, such as localhost, a loopback address or a `.localhost` name, opens it in a Cursor Browser tab. [cursor only]
  2. No `Opening ... in Cursor Browser` toast is shown when such a link opens. [cursor only]
  3. Holding `Shift` while following such a link sends it to the system browser instead. [cursor only]
  4. The hover on such a link reads like `Follow link (ctrl + click, hold shift for your browser)`. [cursor only]
  5. In a remote window, or with the Cursor Settings Browser toggle `Open Local Links in Cursor Browser` off, local links go to the system browser as in stock VS Code. [cursor only]
  6. The Cursor Settings Browser toggle `Open Local Links in Cursor Browser` is on by default and is stored as `autoOpenLocalhostUrls`, not as a settings.json setting. [cursor only]

#### PN-3.47 Console dialogs stay in front on Windows

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: newer-than-1.128. A Windows-only advanced setting that needs the bundled conpty.dll option (PN-2.47) makes the terminal's window the owner of its console window, so dialogs that console programs open, such as sign-in prompts, appear in front of the editor instead of behind it; it is on by default in 1.144.0, restricted in untrusted workspaces, and skipped in remote windows.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.windowsReparentConsoleWindow
- Source: N\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 501-511); N\src\vs\workbench\contrib\terminal\browser\terminalProcessManager.ts (lines 373 and 782-832); N\src\vs\platform\terminal\common\terminal.ts (line 102); C bundle (zero hits for windowsReparentConsoleWindow)
- Pieces:
  1. On Windows with the bundled conpty.dll option on, setting `terminal.integrated.windowsReparentConsoleWindow` makes the terminal's window the owner of its console window, so dialogs from console programs such as sign-in prompts appear in front of the editor instead of behind it. [stock only]
  2. The setting is tagged advanced and is on by default in 1.144.0. [stock only]
  3. The setting is restricted, so it has no effect in untrusted workspaces. [stock only]
  4. The setting is skipped in remote windows. [stock only]


## PN-4 Problems

(both) [boundary for two Cursor menu entries]. The errors, warnings and infos list as tree or table, with filters and quick fix. Cursor adds Add to Chat and Fix with Agent to the row menu.

20 features.

#### PN-4.1 Open, toggle and focus the Problems view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all four ids found by string; the generated focus command comes from the same registerFocusViewAction code that Cursor still has; labels Toggle Problems (Errors, Warnings, Infos) and Focus Problems (Errors, Warnings, Infos) found in NLS)
- What: Show the Problems tab, toggle it, or move focus into it from the View menu (Problems), from Ctrl+Shift+M, from the Command Palette, or by clicking the Problems item in the status bar. The command title reads Toggle Problems because the view lives in the panel. The tab hides itself when the view has been removed from its container.
- Commands: workbench.actions.view.problems; workbench.actions.view.toggleProblems; workbench.action.problems.focus; workbench.action.showErrorsWarnings; workbench.panel.markers.view.focus; workbench.action.tasks.toggleProblems
- Menus: MenubarViewMenu (group 5_panel, order 0, generated from the view's open command); CommandPalette
- Keys: Ctrl+Shift+M (workbench.actions.view.problems)
- Settings: none found
- Source: contrib\markers\browser\markers.contribution.ts (43-90, 132-157, 340-570); contrib\markers\common\markers.ts (8-30); V\src\vs\workbench\services\views\browser\viewsService.ts (471-575); V\src\vs\workbench\contrib\markers\browser\messages.ts (12-13); research entry-points 01-title-bar (View menu rows, 5_panel order 0); C bundle (markers view registration with `workbench.actions.view.problems` and keybinding Ctrl+Shift+M, registerFocusViewAction)
- Pieces:
  1. Choosing Problems from the View menu opens the panel on the Problems tab and moves focus into the problem list.
  2. Pressing `Ctrl+Shift+M` (`workbench.actions.view.problems`, titled Toggle Problems) opens the Problems tab and focuses it, and pressing it again while the Problems view has focus hides the panel.
  3. Toggle Problems (Errors, Warnings, Infos) in the Command Palette (`workbench.actions.view.toggleProblems`) shows the Problems view when it is hidden and closes it when it is already visible.
  4. Focus Problems (Errors, Warnings, Infos) in the Command Palette (`workbench.action.problems.focus`) opens the Problems tab and moves focus into it without ever closing it.
  5. Clicking the Problems item in the status bar shows the Problems view when it is hidden and closes it when it is visible.
  6. When the Problems view has been removed from its container, no Problems tab shows in the panel.

#### PN-4.2 See problems as a tree grouped by file

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the markers tree classes, the collapseAll template id, the row CSS class names marker-code, marker-source and related-info-resource, and the NLS label Collapse All were found)
- What: The default view is a tree. Each file that has problems is one row with a count badge, and its problems sit under it. A problem row shows the severity icon, the message (all lines of it), the source and code beside the message, and [Ln, Col] at the end. A code that has a link target is a clickable link. Files expand and collapse, and a toolbar button collapses all files (that button only shows in tree mode).
- Commands: workbench.actions.treeView.workbench.panel.markers.view.collapseAll; workbench.actions.table.workbench.panel.markers.view.viewAsTree
- Menus: ViewTitle (Collapse All, group navigation, order 2, tree mode only)
- Keys: none found
- Settings: problems.defaultViewMode
- Source: contrib\markers\browser\markersTreeViewer.ts (152-226 file row and count, 279-415 marker row, 416-455 related rows); markersView.ts (487-520); markers.contribution.ts (161-184, 534-552); markersModel.ts (38-140); C bundle (markers tree viewer, collapseAll registration with menu Pe.ViewTitle order 2); NLS 7654
- Pieces:
  1. In tree mode, which is the default look, every file that has problems is one row and its problems sit nested under it.
  2. Each file row shows a count badge with how many problems that file has.
  3. A problem row shows the severity icon, the message with all of its lines, the source and code beside the message, and `[Ln, Col]` at the end.
  4. A code that has a link target shows as a clickable link that opens its target.
  5. File rows expand and collapse to show or hide their problems.
  6. The Collapse All button in the Problems header (`workbench.actions.treeView.workbench.panel.markers.view.collapseAll`) collapses every file row, and it is only shown in tree mode.
  7. Hovering over the message of a problem row shows its full message in a hover.

#### PN-4.3 See problems as a flat table

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the markers-table-container class name and the View as Table label were found; the table column labels use the same NLS strings as stock)
- What: Table mode lists every problem as its own row with no file grouping. The columns are an icon column (severity, with the quick fix light bulb), Code, Message, File and Source. The Code column shows the source name and the code, and the code is a link when the diagnostic gives a target. The File column shows the workspace-relative path and [Ln, Col]. Matches from the filter box are highlighted in the message, file, code and source cells.
- Commands: workbench.actions.table.workbench.panel.markers.view.viewAsTable
- Menus: ViewTitle (View as Table, group navigation, order 3, shown only in tree mode)
- Keys: none found
- Settings: problems.defaultViewMode
- Source: contrib\markers\browser\markersTable.ts (63-335); markersView.ts (464-485); markers.contribution.ts (186-208); C bundle (markers table container, viewAsTable registration); NLS 7637
- Pieces:
  1. In table mode every problem is its own row with no grouping by file.
  2. The table has five columns, in this order: an icon column, Code, Message, File and Source.
  3. The icon column shows the severity icon of each problem.
  4. The Code column shows the source name and the code, and the code is a link when the diagnostic gives a link target.
  5. The File column shows the workspace-relative path of the file followed by `[Ln, Col]`.
  6. Text typed in the filter box is highlighted in the matching Message, File, Code and Source cells.

#### PN-4.4 Switch between tree and table

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids and the setting found; the setting default is tree in both)
- What: One toolbar button in the Problems header flips between the two views. It reads View as Table in tree mode and View as Tree in table mode. The choice is saved per workspace. The setting only decides the mode the first time, before any choice is saved.
- Commands: workbench.actions.table.workbench.panel.markers.view.viewAsTree; workbench.actions.table.workbench.panel.markers.view.viewAsTable
- Menus: ViewTitle (group navigation, order 3, one entry for each direction)
- Keys: none found
- Settings: problems.defaultViewMode
- Source: markers.contribution.ts (104-109, 161-208); markersView.ts (172-173, 384-392, 531-533, 907-908); C bundle (viewAsTree and viewAsTable registrations, setting problems.defaultViewMode); NLS 7636-7637
- Pieces:
  1. In tree mode the Problems header has a button with the View as Table label that switches the list to the flat table.
  2. In table mode the same spot has a button with the View as Tree label that switches the list back to the tree.
  3. The mode a person picks is saved for the workspace and is used again the next time the workspace opens.
  4. Setting `problems.defaultViewMode` chooses the mode used before any choice has been saved, with values `tree` (default) and `table`.
  5. Switching between tree and table keeps the problems that were selected still selected in the new view.

#### PN-4.5 Filter problems by text and file pattern

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the commands, keys and glob and ! filtering are present; the placeholder reads "Filter (e.g. text, **/*.ts, !**/node_modules/**)" with no "@source:ts" hint, because the source filter code is missing, see PN-4.6)
- What: Type in the filter box in the Problems header to hide rows that do not match. Plain text matches the message, file name, source and code. Comma separated glob patterns such as **/*.ts show only matching files, and a leading ! excludes. Past filters are kept in a history. Ctrl+F in the list jumps to the filter box, Ctrl+Down moves back into the list, and Escape clears the text. Typing a letter anywhere in the list also moves focus to the filter box. The text is saved per workspace.
- Commands: problems.action.focusFilter; problems.action.focusProblemsFromFilter; problems.action.clearFilterText
- Menus: none found
- Keys: Ctrl+F (when the Problems view is focused); Ctrl+Down (when the filter box is focused); Escape (when the filter box is focused)
- Settings: none found
- Source: markersView.ts (156-167, 222-236, 277-291, 579-581); markersFilterOptions.ts (14-130); markers.contribution.ts (443-475, 514-532); V\src\vs\workbench\browser\parts\views\viewFilter.ts (60-235); messages.ts (40); C bundle (markers filter options and commands; placeholder is NLS 7706); NLS 7705-7706
- Pieces:
  1. Typing plain text in the filter box in the Problems header hides every problem whose message, file name, source and code do not match it.
  2. Comma separated glob patterns such as `**/*.ts` show only problems from files that match.
  3. A pattern with a leading `!` hides the files that match it, for example `!**/node_modules/**`.
  4. A pattern that starts with a dot, such as `.js`, is treated as a file extension pattern and shows only problems from files with that extension.
  5. Past filter texts are kept in a history that can be recalled from the filter box.
  6. Pressing `Ctrl+F` while the Problems list has focus moves focus into the filter box (`problems.action.focusFilter`), and pressing `Ctrl+Down` in the filter box moves focus back into the list (`problems.action.focusProblemsFromFilter`).
  7. Pressing `Escape` in the filter box clears the filter text (`problems.action.clearFilterText`).
  8. Typing a letter while the list has focus moves focus to the filter box.
  9. The filter text is saved for the workspace and is still there when the workspace is reopened.
  10. In Cursor the filter box placeholder reads "Filter (e.g. text, **/*.ts, !**/node_modules/**)" with no `@source:ts` tip. [cursor only]

#### PN-4.6 Filter problems by source with @source

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched the C bundle for @source:, includeSourceFilters and matchesSourceFilters, and NLS for the stock placeholder text; the only @source hits are the Keyboard Shortcuts and Settings editors, and the Problems placeholder has no source tip)
- What: Type @source:ts in the filter box to keep only problems reported by one source, such as ts or eslint. Put ! in front (!@source:eslint) to hide a source. Quote names that have spaces. The tip is part of the placeholder text in stock.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: markersFilterOptions.ts (14, 56-60, 86-106, 130-155); markersTreeViewer.ts (509-512); messages.ts (40); C bundle (no source filter in the markers filter options; the @source: regexes found belong to the settings and keybindings search); NLS 7706
- Pieces:
  1. Typing `@source:ts` in the filter box keeps only problems reported by the source `ts`, and the match ignores upper and lower case. [stock only]
  2. Using more than one `@source:` term shows a problem when its source matches any one of them. [stock only]
  3. Putting `!` in front, as in `!@source:eslint`, hides problems from that source. [stock only]
  4. A source name with spaces is wrapped in quotes, as in `@source:"my source"`. [stock only]
  5. The `@source:` term is taken out of the filter text, so any other text or glob in the box still filters as usual alongside it. [stock only]
  6. The stock filter box placeholder ends with `@source:ts` as a tip. [stock only]

#### PN-4.7 Show or hide errors, warnings and infos

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all three template ids found with the toggled context keys problems.filter.errors, problems.filter.warnings and problems.filter.info; labels found in NLS)
- What: Click the More Filters button (filter icon) next to the filter box and tick or untick Show Errors, Show Warnings and Show Infos. The button gets a highlight whenever any filter is active, and the choices are saved per workspace. All three are on by default.
- Commands: workbench.actions.workbench.panel.markers.view.toggleErrors; workbench.actions.workbench.panel.markers.view.toggleWarnings; workbench.actions.workbench.panel.markers.view.toggleInfos
- Menus: submenu.view.filter (More Filters, group 1_filter, orders 1 to 3); menu.view.filter (the filter icon button)
- Keys: none found
- Settings: none found
- Source: markers.contribution.ts (210-283); markersViewActions.ts (14-120); markersView.ts (185-192, 284-287, 901-905); V\src\vs\workbench\browser\parts\views\viewFilter.ts (32-60); contrib\markers\common\markers.ts (36-40); C bundle (toggleErrors, toggleWarnings and toggleInfos registrations); NLS 7638-7643
- Pieces:
  1. A More Filters button with a filter icon sits next to the filter box and opens a menu with Show Errors, Show Warnings and Show Infos.
  2. Each of the three entries shows a check when it is on, and unticking one hides the problems of that severity.
  3. All three are on by default.
  4. The More Filters button is highlighted whenever any filter is active.
  5. The tick choices are saved for the workspace.

#### PN-4.8 Show only the active file, and hide excluded files

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both template ids found, with the toggled context keys problems.filter.activeFile and problems.filter.excludedFiles; the labels Show Active File Only and Show Excluded Files are in NLS)
- What: Two more entries in the More Filters menu. Show Active File Only limits the list to problems in the file of the active editor (the list follows you as you switch tabs, and auto reveal is skipped while it is on). Show Excluded Files is ticked by default, and unticking it hides files that match the files.exclude patterns of their workspace folder.
- Commands: workbench.actions.workbench.panel.markers.view.toggleActiveFile; workbench.actions.workbench.panel.markers.view.toggleExcludedFiles
- Menus: submenu.view.filter (More Filters, group 2_filter, orders 1 and 2)
- Keys: none found
- Settings: files.exclude
- Source: markers.contribution.ts (285-333); markersView.ts (193-200, 395-423, 770-780); markersViewActions.ts (14-120); contrib\markers\common\markers.ts (40-42); messages.ts (33-36); C bundle (toggleActiveFile and toggleExcludedFiles registrations); NLS 7644-7647
- Pieces:
  1. The More Filters menu has a Show Active File Only entry, in a group below the three severity entries.
  2. Ticking Show Active File Only limits the list to problems in the file of the active editor.
  3. While Show Active File Only is on, the list changes to the new file each time the active editor changes.
  4. The More Filters menu has a Show Excluded Files entry that is ticked by default, which shows problems from every file including those that match `files.exclude`.
  5. Unticking Show Excluded Files hides problems in files that match the `files.exclude` patterns of their workspace folder.
  6. Both choices are saved for the workspace.

#### PN-4.9 See how many problems match, and what an empty list means

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all four strings, Clear Filters and the Showing {0} of {1} badge text found in NLS)
- What: When a filter hides some rows, the filter box shows a badge such as "Showing 3 of 12". When nothing is left, the body shows a message instead of an empty list: "No problems have been detected in the workspace.", "No problems have been detected in the current file." (Active File Only on), or "No results found with provided filter criteria." followed by a Clear Filters link that resets the text and the severity and exclude toggles. A screen reader hears the same counts.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: markersView.ts (279-291, 685-768); messages.ts (24-27); V\src\vs\workbench\browser\parts\views\viewFilter.ts (updateBadge and checkMoreFilters, 60-235); C bundle (markers view message box code, class name messageAction); NLS 7705 region
- Pieces:
  1. When filters hide some problems, the filter box shows a badge such as "Showing 3 of 12" with the visible count and the total.
  2. The badge is not shown when no problems are hidden or when the workspace has no problems.
  3. When there are no problems at all, the body of the view shows "No problems have been detected in the workspace." instead of an empty list.
  4. With Show Active File Only on and no problems in the current file, the body shows "No problems have been detected in the current file."
  5. When filters hide every problem, the body shows "No results found with provided filter criteria." followed by a Clear Filters link.
  6. Clicking Clear Filters, or pressing `Enter` or `Space` on it, empties the filter text and turns Show Errors, Show Warnings and Show Infos and Show Excluded Files back on.
  7. A screen reader hears "Showing N problems" or "Showing N of M problems" when rows are listed, and hears the same text as the body message when the list is empty.

#### PN-4.10 Open a problem in the editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present (problems.action.open found, with the range highlight decoration code)
- What: Click, double-click or press Enter on a problem row to open its file with the cursor on the problem. A single click or arrow-key move opens it without taking focus from the list and highlights the problem range in the editor. Double-click or Enter pins the tab and moves focus into the editor. Related-information rows open their own location the same way. Opening a file row does nothing, only problem rows jump.
- Commands: problems.action.open
- Menus: none found
- Keys: Enter (when a problem row is focused); Cmd+Down on macOS (secondary)
- Settings: none found
- Source: markers.contribution.ts (44-56); markersView.ts (300-326, 447-449, 782-808); contrib\markers\common\markers.ts (23); C bundle (problems.action.open keybinding rule, rangeHighlightDecorations)
- Pieces:
  1. Clicking a problem row opens its file in the editor with the cursor on the problem, and focus stays in the Problems list.
  2. Moving the selection with the arrow keys opens each problem the same way and highlights the problem's range in the editor while focus stays in the list.
  3. Pressing `Enter` (`problems.action.open`) or double-clicking a problem row opens its file with the cursor on the problem, pins the tab and moves focus into the editor.
  4. On macOS, `Cmd+Down` on a focused problem row does the same as `Enter`.
  5. Pressing `Enter` or double-clicking a file row does not jump to the editor, only problem rows do.

#### PN-4.11 Open a problem beside the current editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present (problems.action.openToSide found)
- What: Press Ctrl+Enter on a focused problem row (Ctrl+Enter on macOS too) to open its file in the side editor group, with the cursor on the problem. A mouse click with Ctrl, Cmd or Alt held reaches the same code path through the list's open event (listService.ts 732 and 753).
- Commands: problems.action.openToSide
- Menus: none found
- Keys: Ctrl+Enter (when a problem row is focused); Ctrl+Enter on macOS (WinCtrl)
- Settings: none found
- Source: markers.contribution.ts (58-70); markersView.ts (300-326, 447-449); contrib\markers\common\markers.ts (24); V\src\vs\platform\list\browser\listService.ts (732, 753); C bundle (problems.action.openToSide keybinding rule)
- Pieces:
  1. Pressing `Ctrl+Enter` on a focused problem row (`problems.action.openToSide`) opens its file in the side editor group with the cursor on the problem.
  2. On macOS the key for opening to the side is `Ctrl+Enter`, not `Cmd+Enter`.
  3. Clicking a problem row with `Ctrl`, `Cmd` or `Alt` held opens its file in the side editor group.

#### PN-4.12 Keep the active file's problems in view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (setting found)
- What: With the setting on (default), the list scrolls to and reveals the problems of the file in the active editor whenever the active editor changes, and also when the current file gets its first problem. It is skipped while Show Active File Only is on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: problems.autoReveal
- Source: markers.contribution.ts (100-104); markersView.ts (586-608, 656-663, 770-780); messages.ts (15); C bundle (setting problems.autoReveal)
- Pieces:
  1. Setting `problems.autoReveal` turns on automatic reveal of the active file's problems in the list, as a true or false value that is true by default.
  2. With it on, switching the active editor scrolls the Problems list to the problems of the new file and reveals them.
  3. With it on, the list also reveals the current file's problems when that file gets its first problem.
  4. Automatic reveal does not happen while Show Active File Only is on.

#### PN-4.13 Quick fix from a problem row

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids found, plus the markers-panel-action-quickfix class name and the "Show fixes" label; no Cursor-specific fix entry was found in this path)
- What: The selected or focused problem row (tree or table) shows a light bulb when a language provider offers a quick fix for it. Click the light bulb, press Ctrl+. on the row, or right-click the row to see the fixes (quick-fix kind only), then pick one to apply it. The tooltip reads "Show fixes". A row with an auto-fixable fix gets a stronger bulb. The view waits for the file's text model to load before it can offer fixes. Picking one opens the file at the problem without taking focus, then applies the fix.
- Commands: problems.action.showQuickFixes; workbench.actions.problems.quickfix (action id of the light bulb button, not a registered command)
- Menus: ProblemsPanelContext (quick fix entries are added at the top by the view, not by a menu contribution)
- Keys: Ctrl+. (when a problem row is focused)
- Settings: none found
- Source: markers.contribution.ts (80-93); markersViewActions.ts (122-185); markersTreeViewer.ts (279-345, 574-700); markersTable.ts (63-120); markersView.ts (293-298, 451-460, 838-852); messages.ts (35); C bundle (QuickFixAction and quick fix view item, problems.action.showQuickFixes); NLS 7705 region
- Pieces:
  1. A selected or focused problem row, in tree or table mode, shows a light bulb when a language provider offers a quick fix for it, and a row with no fix shows none.
  2. Clicking the light bulb (tooltip "Show fixes") opens a menu of the quick fixes for that problem, quick-fix kind only.
  3. Pressing `Ctrl+.` on a focused problem row opens the same list of fixes (`problems.action.showQuickFixes`).
  4. Right-clicking a problem row puts the same fixes at the top of its context menu.
  5. A row whose fix can be applied automatically shows a stronger light bulb icon.
  6. The light bulb and the fixes only appear once the file's text has loaded, so there can be a short delay after the row is selected.
  7. Picking a fix opens the file at the problem without taking focus and then applies the fix.

#### PN-4.14 Right-click menu on a problem row

- Tag: both. Evidence: source. Boundary: yes - Cursor adds Add to Chat and Fix with Agent to this menu in a separate group; those two chat hand-offs are not listed, only the stock entries are.. Cursor: changed (the stock entries are present; Cursor adds workbench.action.problems.addToChat "Add to Chat" and workbench.action.problems.fixInChat "Fix with Agent" in group z_chat, orders 1 and 2, when a problem row is focused)
- What: Right-click a problem (tree or table) to get a menu with the quick fixes first, then Copy and Copy Message. On a related-information row only Copy Message shows. The entries show their keyboard shortcut next to them when they have one.
- Commands: problems.action.copy; problems.action.copyMessage; problems.action.copyRelatedInformationMessage
- Menus: ProblemsPanelContext (group navigation)
- Keys: Ctrl+C (Copy, when the Problems view is focused)
- Settings: none found
- Source: markers.contribution.ts (372-442); markersView.ts (809-852); V\src\vs\platform\actions\common\actions.ts (148); C bundle (six Pe.ProblemsPanelContext registrations: Copy, Copy Message, Copy related message, quick fix path, and the two chat entries); NLS 7662-7664, 7667-7668
- Pieces:
  1. Right-clicking a problem row, in tree or table mode, opens a context menu with the quick fixes listed first.
  2. Under the fixes the menu has Copy and Copy Message.
  3. Menu entries that have a keyboard shortcut show it next to their name, for example `Ctrl+C` next to Copy.
  4. Right-clicking a related-information row shows a menu with only Copy Message.
  5. In Cursor the menu also has Add to Chat and Fix with Agent in a separate group after the stock entries, when a problem row is focused. [cursor only]

#### PN-4.15 Copy a problem or its message

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all three ids found, with the Ctrl+C keybinding rule)
- What: Select one or more problem rows (or whole file rows) and use Copy, or press Ctrl+C in the list, to put the full problem data on the clipboard as JSON text in square brackets (severity, message, range, source, code, file path and related information). Copy Message puts only the message text there, one line per problem. On a related-information row, Copy Message copies that related message.
- Commands: problems.action.copy; problems.action.copyMessage; problems.action.copyRelatedInformationMessage
- Menus: ProblemsPanelContext (group navigation)
- Keys: Ctrl+C (Copy, when the Problems view is focused, the list is visible and no related-information row is focused)
- Settings: none found
- Source: markers.contribution.ts (350-442); markersModel.ts (112-118); contrib\markers\common\markers.ts (17-19); C bundle (Copy, Copy Message and related message registrations); NLS 7662-7664
- Pieces:
  1. Choosing Copy from the context menu with one or more problem rows selected puts those problems on the clipboard as JSON text wrapped in square brackets.
  2. Each copied problem holds its severity, message, range, source, code, file path and any related information.
  3. Copying with a file row selected includes every problem under that file.
  4. Pressing `Ctrl+C` while the Problems list has focus does the same as Copy (`problems.action.copy`), unless a related-information row is focused.
  5. Copy Message (`problems.action.copyMessage`) puts only the message text of the selected problems on the clipboard, one line per problem.
  6. Copy Message on a related-information row copies that related message (`problems.action.copyRelatedInformationMessage`).

#### PN-4.16 Show problem messages on one line or several

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both command ids, the toggleMultiline action id, the two chevron icons markers-view-multi-line-expanded and markers-view-multi-line-collapsed, and both labels found)
- What: A problem whose message has more than one line shows a small chevron at the end of its row. Click it to switch between all lines and the first line only (tooltips Show message in single line and Show message in multiple lines). Two Command Palette entries do the same for every row at once. The choice is saved per workspace, and multiple lines is the default. Messages longer than 1000 characters per line are cut with an ellipsis.
- Commands: problems.action.showMultilineMessage; problems.action.showSinglelineMessage; problems.action.toggleMultiline (action id of the row chevron, not a registered command)
- Menus: CommandPalette (shown when the Problems view is visible)
- Keys: none found
- Settings: none found
- Source: markers.contribution.ts (480-513); markersTreeViewer.ts (100-115, 256-277, 343-385, 600-615); markersView.ts (527-529, 906); contrib\markers\common\markers.ts (21-22); C bundle (the two icons and Usg="problems.action.toggleMultiline")
- Pieces:
  1. A problem whose message has more than one line shows a small chevron at the end of the first line of its row, and a one-line message has no active chevron.
  2. By default every line of a long message is shown, with the chevron pointing up and the tooltip "Show message in single line".
  3. Clicking the chevron cuts that row down to its first line, points the chevron down and changes the tooltip to "Show message in multiple lines", and clicking again shows all lines.
  4. The Command Palette entries "Show message in multiple lines" and "Show message in single line" (Problems category) change every row at once, and they are listed only while the Problems view is visible.
  5. The choice is saved for the workspace, and showing multiple lines is the default.
  6. A message line longer than 1000 characters is cut with an ellipsis.

#### PN-4.17 See the related locations of a problem

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the related-information renderer class names and the copy command id found)
- What: When a diagnostic points to other places (for example "first declared here"), its row gets an expand arrow, and its screen reader label says it has references to N locations. Expanding with the arrow shows one child row per location with the file name, the message and [Ln, Col]. Clicking a child opens that spot in the editor. A plain click on the problem row does not expand it, only the arrow does.
- Commands: problems.action.copyRelatedInformationMessage
- Menus: ProblemsPanelContext (Copy Message, when a related-information row is focused)
- Keys: Enter (the list's own open key; problems.action.open needs a problem row focused)
- Settings: none found
- Source: markersTreeViewer.ts (416-455, 510-512); markers.contribution.ts (422-442); markersModel.ts (133-140); messages.ts (57, 64-74); C bundle (RelatedInformationRenderer, class related-info-resource, RELATED_INFORMATION_COPY_MESSAGE_ACTION_ID)
- Pieces:
  1. A problem whose diagnostic points to other locations shows an expand arrow on its row.
  2. Expanding the row shows one child row per location, each with the file name, the message and `[Ln, Col]`.
  3. Clicking a child row, or pressing `Enter` on it, opens that location in the editor.
  4. Clicking the problem row itself does not expand it, only the arrow does.
  5. A screen reader hears the problem's label followed by "This problem has references to N locations."

#### PN-4.18 Problems count badge on the tab

- Tag: both. Evidence: source. Boundary: no. Cursor: present (showViewActivity called with the markers view id and a number badge, and the label Total {0} Problems found; registered with disableInGlass so the Agents window skips it)
- What: A number badge with the total of errors, warnings and infos sits on the Problems tab in the panel (tooltip "Total N Problems"). It disappears when the count is zero. If the Problems view is dragged into the side bar or auxiliary bar, the same badge shows on that container's icon instead. The tab itself can be icon only or labelled depending on the panel label setting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.panel.showLabels
- Source: markers.contribution.ts (696-721); V\src\vs\workbench\browser\workbench.contribution.ts (573-593 for workbench.panel.showLabels); C bundle (ActivityUpdater class e6a with showViewActivity); NLS 7633
- Pieces:
  1. A number badge sits on the Problems tab in the panel with the total of errors, warnings and infos, and its tooltip reads "Total N Problems".
  2. The badge disappears when the total is zero.
  3. The number follows the problems as they are added and removed, and it does not change when filters hide rows in the list.
  4. When the Problems view is moved into the side bar or the auxiliary bar, the same badge shows on that container's icon instead.
  5. Setting `workbench.panel.showLabels` chooses whether the Problems tab and the other panel tabs show a text label or only an icon, with values `true` (label, default) and `false` (icon).

#### PN-4.19 Turn problem markers off everywhere

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the setting string, the status.problemsVisibility item with the whole-word icon and the settings link @id:problems.visibility, and the NLS text were found)
- What: The setting problems.visibility (on by default) switches problem markers off across the workbench: squiggles in editors, and error and warning marks in the Explorer and the Outline. When it is off, a warning item with the whole-word icon is added in the status bar next to the Problems counts ("Problems are turned off. Click to open settings.") and opens this setting. I found no read of the setting in the Problems list code or in the count item, so neither is hidden by it. It is listed here because the setting lives in the Problems settings group.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: problems.visibility
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (958-969); markers.contribution.ts (578-640); V\src\vs\workbench\browser\parts\editor\textEditor.ts (41-43, 111-112, 171); markersFileDecorations.ts (69-85); V\src\vs\workbench\contrib\outline\browser\outline.contribution.ts (68-78); C bundle (MarkersStatusBarContributions, getMarkersItemTurnedOff); NLS 7626-7627
- Pieces:
  1. Setting `problems.visibility`, on by default, switches problem markers on or off across the workbench, and turning it off removes the squiggles in editors.
  2. Turning `problems.visibility` off also removes the error and warning marks on files and folders in the Explorer and on elements in the Outline.
  3. While the setting is off, a warning item with the whole-word icon appears in the status bar next to the Problems counts, with the tooltip "Problems are turned off. Click to open settings."
  4. Clicking that status bar item opens the Settings editor filtered to `@id:problems.visibility`.
  5. The item disappears as soon as the setting is turned back on, without a reload.
  6. Turning the setting off does not hide the Problems list or the Problems counts in the status bar.

#### PN-4.20 Open accessibility help for the Problems filter

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched NLS for "Accessibility Help: Problems Panel Filter" and "You are in the Problems panel", and the bundle for ProblemsFilterHelp; the only filter help page in Cursor is "Accessibility Help: Terminal Find". The generic command editor.action.accessibilityHelp is present)
- What: With focus in the Problems filter box, Open Accessibility Help shows a text page for screen reader users that explains what the filter does, the filter syntax, the severity and scope toggles, keyboard navigation and the problems settings. The page is chosen by the context key problemsFilterFocus.
- Commands: editor.action.accessibilityHelp
- Menus: CommandPalette (Open Accessibility Help)
- Keys: Alt+F1 (Linux: Alt+Shift+F1, with Alt+F1 as secondary)
- Settings: accessibility.verbosity.find
- Source: markersAccessibilityHelp.ts (1-110); markers.contribution.ts (723-724); V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (164-182); V\src\vs\workbench\contrib\accessibility\common\accessibilityCommands.ts (7-8); C bundle (editor.action.accessibilityHelp found, no Problems help provider); NLS (page title absent)
- Pieces:
  1. With focus in the Problems filter box, Open Accessibility Help (`editor.action.accessibilityHelp`, `Alt+F1`, and on Linux `Alt+Shift+F1` with `Alt+F1` as the second key) opens a page titled "Accessibility Help: Problems Panel Filter". [stock only]
  2. The page explains that the filter is not a navigating search, that it hides problems that do not match, and that focus stays in the box while the list updates. [stock only]
  3. The page describes the filter syntax, where plain text matches the message, file path or code and `!text` hides matches. [stock only]
  4. The page names the Errors, Warnings, Info and Active File Only toggles and says they work together with the text filter. [stock only]
  5. The page lists the keyboard moves: Down Arrow into the list, Tab to the toggles, Enter on a problem, the next and previous problem keys, and Escape to clear the filter. [stock only]
  6. The page lists the settings `accessibility.verbosity.find`, `problems.autoReveal`, `problems.defaultViewMode`, `problems.sortOrder` and `problems.showCurrentInStatus`, with the key that opens Settings. [stock only]
  7. Setting `accessibility.verbosity.find` controls whether the filter box announces the hint that Accessibility Help is available. [stock only]


## PN-5 Output and logs

(both). A read-only channel view with a channel picker and log tools. Cursor adds about 30 channels from its own extensions. Examples: channel picker; log level and filter; auto-scroll; open in editor, save, import and export.

20 features.

#### PN-5.1 Open, toggle and focus the Output view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (toggleOutput with the same key and the Linux chord found; the generated focus command comes from the registerFocusViewAction code that Cursor still has)
- What: Show the Output tab, toggle it, or move focus into it from the View menu (Output), from Ctrl+Shift+U (Linux: Ctrl+K Ctrl+H), or from the Command Palette. The view shows the channel last used in this workspace, and the first registered channel when none was chosen yet.
- Commands: workbench.action.output.toggleOutput; workbench.panel.output.view.focus
- Menus: MenubarViewMenu (group 5_panel, order 1, generated from the view's open command); CommandPalette
- Keys: Ctrl+Shift+U (Linux: Ctrl+K Ctrl+H)
- Settings: none found
- Source: output.contribution.ts (72-102); outputServices.ts (307-395); V\src\vs\workbench\services\views\browser\viewsService.ts (471-575); research entry-points 01-title-bar (View menu row Output, 5_panel order 1); C bundle (Output container with storageId and order 1, view with openCommandActionDescriptor and keybindings); NLS 8533-8535
- Pieces:
  1. Choosing Output from the View menu opens the panel on the Output tab and moves focus into the Output text.
  2. Pressing `Ctrl+Shift+U` (on Linux the chord `Ctrl+K Ctrl+H`, `workbench.action.output.toggleOutput`) opens Output and focuses it, and pressing it again while Output has focus hides the panel.
  3. Toggle Output in the Command Palette does the same as the key.
  4. Focus on Output View in the Command Palette (`workbench.panel.output.view.focus`) opens the Output tab and moves focus into it.
  5. When the workspace has no channel chosen yet, the Output view opens on the first registered channel.

#### PN-5.2 Switch the channel from the drop-down

- Tag: both. Evidence: source. Boundary: no. Cursor: present (switchBetweenOutputs, the submenu id, the three group names and the per-channel show id template all found; the channel descriptors also have a workspaceId in Cursor)
- What: The Output title row has a drop-down that shows the current channel name. Open it to pick another channel, with a check on the active one. Channels are grouped: extension channels first, then built-in ones, then user channels (imported or combined logs), with separators. New channels appear in the list while Output is open, and removed ones disappear. The chosen channel is remembered for the workspace.
- Commands: workbench.output.action.switchBetweenOutputs; workbench.action.output.show.<channel id>
- Menus: ViewTitle (submenu workbench.output.menu.switchOutput, group navigation, order 1, shows the selection); workbench.output.menu.switchOutput (groups 0_ext_outputchannels, 1_core_outputchannels, 2_user_outputchannels)
- Keys: none found
- Settings: none found
- Source: output.contribution.ts (133-191); outputServices.ts (307-345, 404-415); outputView.ts (128-136); V\src\vs\workbench\services\output\common\output.ts (descriptor types); C bundle (registerSwitchOutputAction); NLS 8493-8494
- Pieces:
  1. The Output title row has a drop-down that shows the name of the current channel.
  2. Opening the drop-down lists the channels with a check beside the active one, and picking a channel switches the Output text to it.
  3. The list has three groups with separators between them: extension channels first, then built-in channels, then user channels such as imported or combined logs.
  4. Channels that are added while Output is open show up in the list, and channels that are removed disappear from it.
  5. The channel picked in the drop-down is remembered for the workspace and shown again the next time Output opens.
  6. Running `workbench.action.output.show.<channel id>` opens Output on that channel.

#### PN-5.3 Pick a channel from a list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all three ids found; labels Show Output Channels..., Show Logs..., Show Window Log and the placeholders Select Output Channel and Select Log found in NLS)
- What: Three Command Palette commands open a quick pick of channels. Show Output Channels lists every channel (extension channels first, then built-in ones). Show Logs lists only log channels, with extension logs under a separator. Show Window Log jumps straight to the Window log. Choosing an entry opens Output on that channel. Show Logs and Show Window Log are in the Developer category.
- Commands: workbench.action.showOutputChannels; workbench.action.showLogs; workbench.action.showWindowLog
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: output.contribution.ts (272-308, 532-573); V\src\vs\workbench\contrib\logs\common\logs.contribution.ts (182-198); V\src\vs\workbench\services\log\common\logConstants.ts (9-11); C bundle (registerShowOutputChannelsAction, registerShowLogsAction, ShowWindowLogAction); NLS (labels matched by text)
- Pieces:
  1. Show Output Channels... in the Command Palette (Output category) opens a list titled "Select Output Channel" with every channel, extension channels first, then a separator, then built-in channels.
  2. Show Logs... in the Command Palette (Developer category) opens a list titled "Select Log" with only log channels, built-in logs first and then an Extension Logs separator with the extension logs.
  3. Show Window Log in the Command Palette (Developer category) opens Output straight on the Window log with no list.
  4. Choosing an entry in either list opens Output on that channel.

#### PN-5.4 Set the log level

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the gear submenu id, both level id prefixes, setLogLevel and setDefaultLogLevel found; labels Set Log Level..., Set As Default and Set Default Log Level found in NLS)
- What: For a log channel the Output title has a gear menu (Set Log Level...) with Trace, Debug, Info, Warning, Error and Off, a check on the current level, and Set As Default, which is enabled when the level differs from the default. The Command Palette command Set Log Level... (Developer) opens a picker with an All section that sets the level for everything, then every log channel, with a button on each level to make it the default. The telemetry log cannot be set.
- Commands: workbench.action.output.activeOutputLogLevel.<level>; workbench.action.output.activeOutputLogLevelDefault; workbench.action.setLogLevel; workbench.action.setDefaultLogLevel
- Menus: ViewTitle (submenu workbench.output.menu.logLevel, group navigation, order 6, gear icon, shown only when the active channel's level can be set); workbench.output.menu.logLevel (groups 0_level and 1_default); CommandPalette
- Keys: none found
- Settings: none found
- Source: output.contribution.ts (455-530); outputServices.ts (433-462); V\src\vs\workbench\contrib\logs\common\logsActions.ts (24-158); logs\common\logs.contribution.ts (27-51); C bundle (registerConfigureActiveOutputLogLevelAction, SetLogLevelAction); NLS (labels matched by text)
- Pieces:
  1. For a log channel whose level can be set, the Output title row shows a gear button (Set Log Level...) that opens a menu.
  2. The gear menu lists Trace, Debug, Info, Warning, Error and Off with a check on the current level, and picking one sets the level for that channel.
  3. A Set As Default entry below the levels is enabled only when the channel's level differs from the default, and picking it saves that level as the default.
  4. The gear does not show for plain channels, and the telemetry log cannot have its level set.
  5. Set Log Level... in the Command Palette (Developer category) opens a picker that starts with an All section listing the six levels, with a check on the current one and the word Default beside the default one, and picking a level sets it for everything.
  6. Below the All section the picker lists extension logs and then the other logs, and picking a channel opens a second picker of the six levels for just that channel.
  7. In each picker a button on every level that is not the default, with the tooltip "Set as Default Log Level", makes that level the default and closes the picker.

#### PN-5.5 Filter the output text

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the box, the clear command and Escape are present, but the compiled filter matches one plain text only, with no comma list or ! exclusion, and the placeholder is just "Filter" where stock says "Filter (e.g. text, !excludeText, text1,text2)")
- What: Type in the filter box in the Output header to hide lines that do not match, with the matches highlighted. Stock also takes several comma separated terms (a line shows if any matches), a leading ! to exclude, and quoted terms. Escape in the box clears it. For log channels the filter works on whole log entries, and for plain channels on single lines. The text is saved per workspace.
- Commands: workbench.actions.workbench.panel.output.clearFilterText; output.editor.contrib.filterController
- Menus: none found
- Keys: Escape (when the filter box is focused)
- Settings: none found
- Source: outputView.ts (88-135, 357-560); outputServices.ts (134-182); output.contribution.ts (693-711); V\src\vs\workbench\browser\parts\views\viewFilter.ts (60-235); C bundle (output filter controller with a single findMatches call on the filter text; the placeholder is NLS 8537); NLS 8537
- Pieces:
  1. Typing in the filter box in the Output header hides the lines that do not contain the text and highlights the matches in the lines that stay, without caring about upper and lower case.
  2. In a log channel the filter hides or keeps whole log entries, and in a plain channel it hides or keeps single lines.
  3. Typing several terms separated by commas shows a line when any one term matches. [stock only]
  4. A term with a leading `!` hides the lines that contain it. [stock only]
  5. A term in double quotes can contain a comma without being split. [stock only]
  6. Pressing `Escape` in the filter box clears the filter text (`workbench.actions.workbench.panel.output.clearFilterText`).
  7. The filter text is saved for the workspace.
  8. The stock filter box placeholder reads "Filter (e.g. text, !excludeText, text1,text2)". [stock only]
  9. In Cursor the filter takes one plain text with no comma list and no `!` exclusion, and its placeholder reads just "Filter". [cursor only]

#### PN-5.6 Filter log lines by level

- Tag: both. Evidence: source. Boundary: no. Cursor: present (group 2_log_filter found, the five filter context keys output.filter.trace to output.filter.error found, and the label "Show or hide {0} messages in the output" found in NLS)
- What: When a log channel is open, the More Filters button (filter icon) next to the filter box lists Trace, Debug, Info, Warning and Error as toggles. Untick one to hide the log entries of that level. The button gets a highlight when any level is hidden, and the choices are saved per workspace. The entries do not show for plain channels.
- Commands: workbench.actions.workbench.panel.output.toggle.trace; workbench.actions.workbench.panel.output.toggle.debug; workbench.actions.workbench.panel.output.toggle.info; workbench.actions.workbench.panel.output.toggle.warning; workbench.actions.workbench.panel.output.toggle.error
- Menus: submenu.view.filter (More Filters, group 2_log_filter, only while the active channel is a log file)
- Keys: none found
- Settings: none found
- Source: output.contribution.ts (641-691); outputView.ts (196-200, 480-552); V\src\vs\workbench\services\output\common\output.ts (filter context keys); C bundle (registerLogLevelFilterActions, shouldShowLogLevel)
- Pieces:
  1. While a log channel is open, the More Filters button (filter icon) next to the filter box lists Trace, Debug, Info, Warning and Error as toggles, all ticked by default.
  2. Unticking a level hides the log entries of that level.
  3. The More Filters button is highlighted while any level is hidden.
  4. The ticked and unticked levels are saved for the workspace.
  5. The level toggles do not appear while a plain channel is open.

#### PN-5.7 Filter log lines by category

- Tag: both. Evidence: source. Boundary: no. Cursor: present (group 1_category_filter, toggleCategory and the category filter context key found)
- What: Some log entries have a category (a sub-source name). When the open log has categories, each one shows up as a toggle in the More Filters menu, above the level toggles. Untick a category to hide its entries. The list is built from the open log, so it changes when you change channel.
- Commands: workbench.actions.workbench.panel.output.toggle.<channel id>:<category>
- Menus: submenu.view.filter (More Filters, group 1_category_filter)
- Keys: none found
- Settings: none found
- Source: outputView.ts (357-446, 498-504, 553-559); outputServices.ts (270-300); V\src\vs\workbench\services\output\common\output.ts (HIDE_CATEGORY_FILTER_CONTEXT); C bundle (output filter controller, shouldShowCategory, toggleCategory)
- Pieces:
  1. When the open log has entries with a category, each category appears as a toggle in the More Filters menu, in a group above the level toggles.
  2. Unticking a category hides the log entries of that category.
  3. The list of categories is built from the open log, so it changes when another channel is opened.
  4. The More Filters button is highlighted while a category of the open channel is hidden.
  5. The hidden categories are saved for the workspace.

#### PN-5.8 Auto scroll and scroll lock

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the command, the setting and the context key outputView.scrollLock found; both tooltips and the setting description found in NLS)
- What: A lock button in the Output title row switches between following new output and staying put (tooltips Turn Auto Scrolling Off and Turn Auto Scrolling On, lock and unlock icons). While following, the view jumps to the last line as text arrives. With smart scroll on (the default), clicking in the text above the last line locks it, and clicking on the last line unlocks it. This is the only setting in the Output group of the Settings editor.
- Commands: workbench.output.action.toggleAutoScroll
- Menus: ViewTitle (group navigation, order 3)
- Keys: none found
- Settings: output.smartScroll.enabled
- Source: output.contribution.ts (344-370, 839-851); outputView.ts (148-172); V\src\vs\workbench\services\output\common\output.ts (CONTEXT_OUTPUT_SCROLL_LOCK); C bundle (registerToggleAutoScrollAction, setting output.smartScroll.enabled); NLS (labels matched by text)
- Pieces:
  1. The Output title row has a lock button next to Clear Output that switches between following new output and staying put.
  2. While following, the button shows a lock icon with the tooltip "Turn Auto Scrolling Off", and the view jumps to the last line as new text arrives.
  3. Clicking the button stops following and shows an unlock icon with the tooltip "Turn Auto Scrolling On", and new text then no longer moves the view, until the button is clicked again.
  4. Setting `output.smartScroll.enabled`, on by default, lets a click in the text above the last line stop following and a click on the last line resume it.
  5. Opening a channel in the Output view scrolls it to its last line.

#### PN-5.9 Clear the output

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the command and the inOutput context key found; label Clear Output found in NLS)
- What: Clear the text of the open channel with the clear-all button in the Output title row, with Clear Output in the Command Palette (View category), or from the right-click menu inside the Output text. It plays the clear sound signal for accessibility users.
- Commands: workbench.output.action.clearOutput
- Menus: ViewTitle (group navigation, order 2); CommandPalette; EditorContext (when focus is in the Output text)
- Keys: none found
- Settings: none found
- Source: output.contribution.ts (310-342); outputView.ts (333-340); C bundle (registerClearOutputAction)
- Pieces:
  1. The clear-all button in the Output title row empties the text of the open channel.
  2. Clear Output in the Command Palette (View category) empties the open channel the same way.
  3. Right-clicking inside the Output text offers Clear Output in the context menu, and it empties the open channel.
  4. Clearing the output plays the clear sound signal for accessibility users.

#### PN-5.10 Open the output in an editor tab or a new window

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids found with the same hidden-by-default menu rows; both labels found in NLS)
- What: Open Output in Editor opens the open channel as a pinned read-only editor tab. Open Output in New Window does the same in a separate window. Both are title row actions declared with isHiddenByDefault, so a fresh window does not show them as buttons. I could not tell from code alone where a user finds them (see Unclear).
- Commands: workbench.action.openActiveLogOutputFile; workbench.action.openActiveLogOutputFileInNewWindow
- Menus: ViewTitle (group navigation, orders 4 and 5, both hidden by default)
- Keys: none found
- Settings: none found
- Source: output.contribution.ts (372-415, 444-453); C bundle (registerOpenActiveOutputFileAction, registerOpenActiveOutputFileInAuxWindowAction); NLS 8524-8525
- Pieces:
  1. Open Output in Editor (`workbench.action.openActiveLogOutputFile`, go-to-file icon) opens the open channel as a pinned read-only editor tab.
  2. Open Output in New Window (`workbench.action.openActiveLogOutputFileInNewWindow`, empty-window icon) opens the open channel in a separate window.
  3. Both are Output title row actions that are hidden by default, so a fresh window does not show them as buttons.

#### PN-5.11 Save the output as a file

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, menu group 1_export and label found; the dialog title Save Output As found in NLS)
- What: Save Output As... in the Output title row's more actions menu writes the open channel's text to a .log file. A save dialog opens with the channel name as the file name in the default folder.
- Commands: workbench.action.saveActiveLogOutputAs
- Menus: ViewTitle (group 1_export, order 1, in the more actions menu)
- Keys: none found
- Settings: none found
- Source: output.contribution.ts (417-442); outputServices.ts (492-540); C bundle (registerSaveActiveOutputAsAction, saveOutputAs); NLS 8526, 8536
- Pieces:
  1. Save Output As... in the more actions menu of the Output title row opens a save dialog titled "Save Output As".
  2. The dialog starts in the default folder with the channel name plus `.log` as the file name, and its file type filter is limited to `log`.
  3. Confirming the dialog writes the text of the open channel to the chosen file, and cancelling writes nothing.

#### PN-5.12 Export logs

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the command, menu row and lists are present, but the compiled run method takes no arguments, so the outputPath and outputChannelIds arguments of stock are missing)
- What: Export Logs... shows a multi-select list of log channels (built-in logs, then Extension Logs, then User Logs) and saves the ones you tick into one .log file through a save dialog. It is in the Output title row's more actions menu and in the Command Palette (Developer). In stock the command can also be called with an output path and a list of channel ids to skip the lists.
- Commands: workbench.action.exportLogs
- Menus: ViewTitle (group 1_export, order 2, in the more actions menu); CommandPalette
- Keys: none found
- Settings: none found
- Source: output.contribution.ts (713-783); outputServices.ts (492-540); C bundle (registerExportLogsAction with run(e) reading no arguments, saveOutputAs(...channels)); NLS 8530
- Pieces:
  1. Export Logs... in the more actions menu of the Output title row, and in the Command Palette (Developer category), opens a list of log channels where several can be ticked.
  2. The list shows built-in logs first, then an Extension Logs separator with the extension logs, then a User Logs separator with the user logs.
  3. Confirming the ticked logs opens a save dialog and saves them into one `.log` file, named `output.log` by default when more than one is ticked and after the channel when only one is.
  4. In stock the command can be called with an output path and a list of channel ids, which skips the list, the save dialog or both. [stock only]
  5. In Cursor the command always shows the list and the save dialog, because it ignores any output path or channel ids passed to it. [cursor only]

#### PN-5.13 Import a log file

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id and menu row found; the descriptor also has a workspaceId and a workspace-specific id)
- What: Import Log... opens a file picker for .log files (one or several). The chosen file becomes a new user channel named after the file, and Output switches to it. Picking several files makes one channel with a section per file. Imported channels sit in the user group of the channel drop-down and can be removed (see PN-5.14).
- Commands: workbench.action.importLog
- Menus: ViewTitle (group 2_add, order 2, in the more actions menu); CommandPalette
- Keys: none found
- Settings: none found
- Source: output.contribution.ts (785-833); C bundle (registerImportLogAction, registerChannel with user:!0 and workspaceId); NLS 8531
- Pieces:
  1. Import Log... in the more actions menu of the Output title row, and in the Command Palette (Developer category), opens a file picker titled "Import Log File".
  2. The picker accepts one or several files and shows only files of the type Log Files, which is `.log`.
  3. Picking one file creates a new user channel named after the file and switches Output to it.
  4. Picking several files creates one channel named after the first file, with a section for each file.
  5. Imported channels show in the user group of the channel drop-down.

#### PN-5.14 Combine logs into one channel, and remove custom channels

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids, group 2_add and the notification text found)
- What: Add Compound Log... lists the log channels (built-in first, then extension logs) as a multi-select and merges the ticked ones into one new channel, shown at once, with the names joined by commas as its label. It is in the Output title row's more actions menu and in the Command Palette (Output category). Remove Output... (Command Palette only) lists the user channels, meaning imported and combined ones, and removes the ticked ones. With none it says "No custom outputs to remove."
- Commands: workbench.action.output.addCompoundLog; workbench.action.output.remove
- Menus: ViewTitle (Add Compound Log, group 2_add, in the more actions menu); CommandPalette
- Keys: none found
- Settings: none found
- Source: output.contribution.ts (193-270); outputServices.ts (464-490); C bundle (registerAddCompoundLogAction, registerRemoveLogAction); NLS 8516-8517
- Pieces:
  1. Add Compound Log... in the more actions menu of the Output title row, and in the Command Palette (Output category), opens a list titled "Select Log" where several log channels can be ticked, built-in logs first and then an Extension Logs separator with the extension logs.
  2. Confirming the list merges the ticked logs into one new channel that Output shows at once, with the names of the logs joined by commas as its label.
  3. Remove Output... in the Command Palette (Output category) opens a list titled "Select Log" of the user channels, meaning the imported and combined ones, where several can be ticked.
  4. Confirming the Remove Output list removes the ticked channels from the channel list.
  5. When there are no user channels, Remove Output shows the notification "No custom outputs to remove."

#### PN-5.15 Open a log file or the logs folder

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all three ids and labels found; workbench.action.openSessionLogFile is absent, which matches stock desktop because the session command lives in the web-only logs contribution)
- What: Open Log... (Command Palette, Developer) lists the log channels, extension logs first, and opens the chosen log file in a pinned editor tab. It can also be called with a log id to skip the list. Open Logs Folder reveals the main.log file in the operating system file manager, and Open Extension Logs Folder reveals the first file in the extension logs folder. Both folder commands are desktop only. Stock also has Open Window Log File (Session)..., which picks a session folder and then a log file, but only in the web build.
- Commands: workbench.action.openLogFile; workbench.action.openLogsFolder; workbench.action.openExtensionLogsFolder
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: output.contribution.ts (575-639); V\src\vs\workbench\contrib\logs\electron-browser\logs.contribution.ts (1-35); V\src\vs\workbench\contrib\logs\electron-browser\logsActions.ts (14-50); V\src\vs\workbench\contrib\logs\browser\logs.contribution.ts (24-45); V\src\vs\workbench\workbench.web.main.ts (138); V\src\vs\workbench\workbench.desktop.main.ts (113); C bundle (registerOpenLogFileAction, openLogsFolder, openExtensionLogsFolder); NLS (labels matched by text)
- Pieces:
  1. Open Log... in the Command Palette (Developer category) opens a list titled "Select Log File" of log channels, extension logs first and then a separator and the built-in logs.
  2. Picking a log in that list opens its log file in a pinned editor tab.
  3. Running Open Log... with a log id such as `"window"` opens that log file straight away with no list.
  4. Open Logs Folder reveals the `main.log` file in the operating system file manager, and it only exists in the desktop app.
  5. Open Extension Logs Folder reveals the first file in the extension logs folder in the operating system file manager, and it only exists in the desktop app.
  6. In the web build, Open Window Log File (Session)... first asks for a session folder and then for a log file to open. [stock only]

#### PN-5.16 Read output and logs in the read-only text area

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the compiled Output editor reads the [Log] override for minimap and word wrap, the Log language ids and mime types are registered, and the built-in log extension ships with the same manifest as stock)
- What: The body of the Output view is a read-only text editor. Lines always wrap, and the line numbers, folding, minimap and current-line highlight are off. Log channels use the Log language, so the built-in log grammar colors timestamps, levels and similar parts. You can select and copy text and use the editor's find widget and its right-click menu there. The editor options can be changed for just this view with a [Log] language override (editor.minimap.enabled and editor.wordWrap are read from it).
- Commands: none found
- Menus: EditorContext (shared entries; Clear Output is added when focus is in the Output text)
- Keys: none found
- Settings: [Log]
- Source: outputView.ts (228-351); output.contribution.ts (53-70); outputChannelModel.ts (248, 393, 405); V\src\vs\workbench\services\output\common\output.ts (18-33); V\extensions\log\package.json; C ext log\package.json; C bundle (OutputEditor getConfigurationOverrides reading "[Log]", ids text/x-code-output and text/x-code-log-output)
- Pieces:
  1. The body of the Output view is a read-only text editor where text can be selected and copied but not changed.
  2. Lines always wrap, and line numbers, folding, the minimap and the current-line highlight are off.
  3. Log channels use the Log language, so the built-in log grammar colors parts such as timestamps and levels.
  4. Right-clicking in the Output text opens the shared editor context menu.
  5. Setting `[Log]` with `editor.minimap.enabled` set to true turns the minimap on in the Output view only.
  6. Setting `[Log]` with `editor.wordWrap` changes how lines wrap in the Output view only, with the values `off`, `on`, `wordWrapColumn` and `bounded`.

#### PN-5.17 Click file paths in the output

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the output link computer module and its workspace folders call found)
- What: When a folder is open, file paths inside a workspace folder that appear in Output text become links. Ctrl+click (Cmd+click on macOS) opens the file, at a line and column when the path is followed by one in the usual forms (path:8:13, path(8,13), "on line 8, column 13" or ":line 8"). With no folder open there are no links.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: outputLinkProvider.ts (40-115); V\src\vs\workbench\contrib\output\common\outputLinkComputer.ts (25-155); outputLinkComputerMain.ts; C bundle (outputLinkComputer, $setWorkspaceFolders)
- Pieces:
  1. When a folder is open, file paths inside a workspace folder that appear in the Output text become links.
  2. `Ctrl+click` on a link (`Cmd+click` on macOS) opens that file in the editor.
  3. When a line follows the path in a form such as `path:8:13`, `path(8,13)`, "on line 8, column 13" or ":line 8", the file opens at that line, and at that column when one is given.
  4. When no folder is open, no links appear in the Output text.

#### PN-5.18 Quiet Output indicator in the status bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the setting, the status item id status.view.showQuietly and the _showChannelQuietly method found; labels Show Output and Show {0} Output Channel found in NLS)
- What: With workbench.view.showQuietly set for the Output view, an extension that asks to show a channel while Output is closed gets a small prominent output icon in the status bar instead of the panel opening. The tooltip is "Show {channel} Output Channel", and clicking it opens that channel. The item is part of the extension API path, so you only see it when an extension asks to reveal a channel.
- Commands: workbench.action.output.show.<channel id>
- Menus: none found
- Keys: none found
- Settings: workbench.view.showQuietly
- Source: V\src\vs\workbench\api\browser\mainThreadOutputService.ts (80-112); V\src\vs\workbench\browser\workbench.contribution.ts (691-702); C bundle (reveal method calling _showChannelQuietly, setting workbench.view.showQuietly); NLS 2830
- Pieces:
  1. With `workbench.view.showQuietly` turned on for Output, an extension that asks to show a channel while Output is closed makes a small prominent output icon appear in the status bar instead of opening the panel.
  2. The icon's tooltip reads "Show {channel} Output Channel", and clicking it opens Output on that channel.
  3. Setting `workbench.view.showQuietly` is an object whose `workbench.panel.output` key is true or false, and it is not set by default.
  4. When that key is off, or Output is already open, an extension's request to show a channel opens Output on that channel as usual.

#### PN-5.19 Open accessibility help for the Output filter

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched NLS for "Accessibility Help: Output Panel Filter"; the only filter help page in Cursor is "Accessibility Help: Terminal Find". The generic command editor.action.accessibilityHelp is present)
- What: With focus in the Output filter box, Open Accessibility Help shows a text page for screen reader users that explains the filter, its syntax (text, !text, comma lists, escaping), keyboard navigation and the Output settings. The page is chosen by the context key outputFilterFocus.
- Commands: editor.action.accessibilityHelp
- Menus: CommandPalette (Open Accessibility Help)
- Keys: Alt+F1 (Linux: Alt+Shift+F1, with Alt+F1 as secondary)
- Settings: accessibility.verbosity.find
- Source: outputAccessibilityHelp.ts (1-105); output.contribution.ts (49-52); V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (164-182); C bundle (editor.action.accessibilityHelp found, no Output help provider); NLS (page title absent)
- Pieces:
  1. With focus in the Output filter box, Open Accessibility Help (`editor.action.accessibilityHelp`, `Alt+F1`, and on Linux `Alt+Shift+F1` with `Alt+F1` as the second key) opens a page titled "Accessibility Help: Output Panel Filter". [stock only]
  2. The page explains that the filter is not a navigating search, that it hides lines that do not match, that focus stays in the box while the output updates, and that new output is filtered as it arrives. [stock only]
  3. The page describes the filter syntax: plain text that ignores case, `!text` to hide, an escape for a literal exclamation mark, and comma separated terms where any one may match. [stock only]
  4. The page lists the keyboard moves: Down Arrow into the output, Tab to the log level buttons, and Escape to clear the filter. [stock only]
  5. The page lists the settings `accessibility.verbosity.find` and `output.smartScroll.enabled`, with the key that opens Settings. [stock only]
  6. Setting `accessibility.verbosity.find` controls whether the filter box announces the hint that Accessibility Help is available. [stock only]

#### PN-5.20 Cursor's extra Output channels

- Tag: cursor. Evidence: bundle. Boundary: yes - most of these channels log agent, MCP, indexing and account plumbing, so only the rows they add to the channel lists are in scope, not their contents. Only Cursor Tab logs an in-scope feature.. Cursor: cursor only
- What: Cursor adds about 34 named channels to the Output drop-down, Show Output Channels and (for the ones created as log channels) Show Logs, plus one "MCP: <server name>" channel per MCP server. From its built-in extensions: Cursor Tab, Cursor Always Local, Cursor Structured Logs, Cursor-always-local debug logs, Filesync, Cursor Indexing & Retrieval, Cursor Git Graph, Cursor Grep Service, Cursor Snapshot Service, Cursor-retrieval debug logs, Cursor Agent Review, Cursor-commits debug logs, Cursor Agent Host, Cursor Agent Host Network, Cursor Agent Host Storage, Cursor Agent Host Structured Logs, Cursor Plugins, Mcp FileSystem Writer, Cursor Agent Exec, Cursor Agent Worker, Local Agent Runtime, MCP Logs, Cursor Computer Use, Cursor IDE Browser Automation, Shadow Workspace, Cursor Resolver, Cursor Resolver Helper, Cursor Socket and NDJSON Ingest. From the main bundle: Cursor Request Traces, Hooks, Analytics Events, Cursor Auth Debug and Utility Host RPC Trace. Cursor Computer Use, Cursor IDE Browser Automation, Cursor Resolver, Cursor Resolver Helper, Hooks, Cursor Auth Debug and Utility Host RPC Trace are plain output channels, and the rest are log channels, so only those get the level filters, the category filters and the log level menu.
- Commands: workbench.action.output.show.<channel id>
- Menus: workbench.output.menu.switchOutput (extension channels in group 0_ext_outputchannels, bundle channels in 1_core_outputchannels); CommandPalette (Show Output Channels, Show Logs, Set Log Level, Open Log, Export Logs)
- Keys: none found
- Settings: none found
- Source: C ext cursor-always-local, cursor-retrieval, cursor-commits, cursor-agent-host, cursor-agent-exec, cursor-agent-worker, cursor-local-agent-runtime, cursor-mcp, cursor-computer-use, cursor-browser-automation, cursor-shadow-workspace, cursor-resolver, cursor-resolver-helper, cursor-socket and cursor-ndjson-ingest (createOutputChannel calls in each dist script); C bundle (registerChannel calls for Cursor Request Traces, Hooks, Analytics Events and Cursor Auth Debug; the utilityHostRpcTrace contribution); NLS 11661
- Pieces:
  1. The channel drop-down and Show Output Channels list the channels of Cursor's built-in extensions: Cursor Tab, Cursor Always Local, Cursor Structured Logs, Cursor-always-local debug logs, Filesync, Cursor Indexing & Retrieval, Cursor Git Graph, Cursor Grep Service, Cursor Snapshot Service, Cursor-retrieval debug logs, Cursor Agent Review, Cursor-commits debug logs, Cursor Agent Host, Cursor Agent Host Network, Cursor Agent Host Storage, Cursor Agent Host Structured Logs, Cursor Plugins, Mcp FileSystem Writer, Cursor Agent Exec, Cursor Agent Worker, Local Agent Runtime, MCP Logs, Cursor Computer Use, Cursor IDE Browser Automation, Shadow Workspace, Cursor Resolver, Cursor Resolver Helper, Cursor Socket and NDJSON Ingest. [cursor only]
  2. The same lists also show the channels made by Cursor's own code: Cursor Request Traces, Hooks, Analytics Events, Cursor Auth Debug and Utility Host RPC Trace. [cursor only]
  3. Each MCP server adds one more channel named "MCP: <server name>". [cursor only]
  4. In the channel drop-down the channels from Cursor's built-in extensions sit in the extension group, and the channels from Cursor's own code sit in the built-in group. [cursor only]
  5. Cursor channels that are log channels also show in Show Logs and get the level toggles, the category toggles and the log level gear menu. [cursor only]
  6. Cursor Computer Use, Cursor IDE Browser Automation, Cursor Resolver, Cursor Resolver Helper, Hooks, Cursor Auth Debug and Utility Host RPC Trace are plain channels, so they get the text filter but none of the log level or category toggles or the log level gear menu. [cursor only]
  7. Choosing Cursor Tab opens Output on the log channel where Cursor Tab writes its log. [cursor only]


## PN-6 Debug Console and Test Results

(both). The two views that show the results of running code. Examples: the Debug Console REPL; console filter and find; the test results tree; test message and output.

32 features.

#### PN-6.1 Show or hide the Debug Console

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (toggle key is Ctrl+Shift+Alt+Y, stock is Ctrl+Shift+Y)
- What: Open, toggle or focus the Debug Console tab from a key, the View menu, the Command Palette or the Run and Debug view menu. The tab only exists while a debugger extension is active.
- Commands: workbench.debug.action.toggleRepl; workbench.debug.action.focusRepl; workbench.panel.repl.view.focus; debug.toggleReplIgnoreFocus; workbench.panel.repl; workbench.panel.repl.view
- Menus: MenubarViewMenu; Views (the "..." menu of the Run and Debug view, entry "Debug Console"); CommandPalette
- Keys: Ctrl+Shift+Y (workbench.debug.action.toggleRepl)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts; V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts; V\src\vs\workbench\services\views\browser\viewsService.ts; C bundle (workbench.debug.action.toggleRepl with keybindings CtrlCmd|Shift|Alt|KeyY)
- Pieces:
  1. Pressing Ctrl+Shift+Y (`workbench.debug.action.toggleRepl`) opens the Debug Console tab in the panel when it is hidden and closes it when it is showing. [stock only]
  2. In Cursor, pressing Ctrl+Shift+Alt+Y toggles the Debug Console tab, and Ctrl+Shift+Y is not the toggle key. [cursor only]
  3. Choosing Debug Console in the View menu opens the Debug Console tab.
  4. Choosing Debug Console in the "..." menu of the Run and Debug view opens the Debug Console tab.
  5. Running the Command Palette entries for `workbench.debug.action.focusRepl` or `workbench.panel.repl.view.focus` brings up the Debug Console with keyboard focus inside it.
  6. The Debug Console tab only appears in the panel tab bar while a debugger extension is active, and is absent otherwise.

#### PN-6.2 Open and close the console automatically with a session

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Choose whether the Debug Console opens when a debug session starts (never, every session, or first session only) and whether it closes when the session ends. A launch configuration can override the open choice.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.internalConsoleOptions; debug.console.closeOnEnd
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; V\src\vs\workbench\contrib\debug\browser\debugService.ts; V\src\vs\workbench\contrib\debug\common\debug.ts; C bundle (debug.internalConsoleOptions, debug.console.closeOnEnd)
- Pieces:
  1. Setting `debug.internalConsoleOptions` chooses when the Debug Console opens by itself, with values `neverOpen`, `openOnSessionStart` and `openOnFirstSessionStart`, default `openOnFirstSessionStart`.
  2. With `neverOpen`, starting a debug session leaves the Debug Console closed.
  3. With `openOnSessionStart`, the Debug Console opens every time a debug session starts.
  4. With `openOnFirstSessionStart`, the Debug Console opens when the first session starts and not for later sessions.
  5. Setting `debug.console.closeOnEnd`, off by default, makes the Debug Console close by itself when the debug session ends.
  6. A launch configuration that sets its own `internalConsoleOptions` overrides the setting for that session.

#### PN-6.3 Evaluate an expression in the console

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Type an expression in the input at the bottom of the console and press Enter to run it in the debugged program. The result lands in the console list and objects expand with a twisty. The input grows to several lines, and is read only with the hint "Please start a debug session to evaluate expressions" when no session is active.
- Commands: repl.action.acceptInput
- Menus: none found
- Keys: Enter (when text input focus and in the debug console)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\workbench\contrib\debug\browser\replViewer.ts; C bundle (repl.action.acceptInput, repl-input-wrapper, replEvaluationResult)
- Pieces:
  1. Typing an expression in the input at the bottom of the console and pressing Enter (`repl.action.acceptInput`) runs it in the debugged program and adds the result to the console list.
  2. A result that is an object shows a twisty that expands it into its child values.
  3. The input grows taller to show several lines as the typed text gets longer.
  4. With no active debug session the input is read only and shows the hint "Please start a debug session to evaluate expressions".

#### PN-6.4 Step through earlier input

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Up or Down in an empty console input to bring back earlier expressions. History holds up to 100 entries and is saved per workspace.
- Commands: history.showPrevious; history.showNext
- Menus: none found
- Keys: Up Arrow; Down Arrow; Alt+Up Arrow; Alt+Down Arrow (when the history input has focus and history navigation is enabled)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\platform\history\browser\contextScopedHistoryWidget.ts; C bundle (history.showPrevious, history.showNext, debug.repl.history)
- Pieces:
  1. Pressing Up Arrow in an empty console input fills it with the previous expression, and pressing it again goes further back.
  2. Pressing Down Arrow steps forward through the history toward the newest entry.
  3. Pressing Alt+Up Arrow or Alt+Down Arrow moves through the history the same way as Up and Down.
  4. While the suggest list is showing, Up and Down move through the suggestions and do not change the history.
  5. The history holds up to 100 entries and drops the oldest first.
  6. The history is saved per workspace, so earlier expressions come back after a reload.

#### PN-6.5 Get completions while typing in the console

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The input shows the editor suggest list with names from the debug adapter (after a dot or the adapter's trigger characters), plus earlier input at the bottom of the list. Enter only accepts a suggestion when the setting is on, because Enter also runs the line.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.console.historySuggestions; debug.console.acceptSuggestionOnEnter
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; C bundle (supportsCompletionsRequest, completionTriggerCharacters, debug.console.historySuggestions, debug.console.acceptSuggestionOnEnter)
- Pieces:
  1. Typing in the console input opens the editor suggest list with names from the debug adapter, after a dot or the adapter's trigger characters.
  2. Earlier console input shows at the bottom of the suggest list.
  3. Setting `debug.console.historySuggestions`, on by default, chooses whether earlier input is offered in the suggest list.
  4. Setting `debug.console.acceptSuggestionOnEnter` chooses whether Enter accepts the highlighted suggestion, with values `off` and `on`, default `off`.
  5. With `acceptSuggestionOnEnter` set to `off`, pressing Enter with the suggest list open runs the typed line instead of accepting a suggestion.

#### PN-6.6 Read program output with severity colors and links

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Output, warnings and errors from the debugged program appear as lines in the console list, colored by severity, with ANSI colors and clickable links in the text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\replViewer.ts; V\src\vs\workbench\contrib\debug\common\replModel.ts; C bundle (outputReplElement, wasANSI)
- Pieces:
  1. Output, warnings and errors from the debugged program appear as lines in the console list.
  2. Each line is colored by its severity, so errors and warnings look different from plain output.
  3. ANSI color codes in the program's text show as colors in the line.
  4. Links inside a line, such as file paths and URLs, are clickable.

#### PN-6.7 Open the source location of an output line

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Output lines that know where they came from show a small file name and line label at the right. Click it to open that file at that line, hover it for the full path.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\replViewer.ts (SourceWidget); C bundle (value-and-source, sourceData)
- Pieces:
  1. An output line that knows where it came from shows a small file name and line number label at its right.
  2. Clicking the label opens that file at that line in the editor.
  3. Hovering the label shows the full path of the file.
  4. An output line that has no known origin shows no label.

#### PN-6.8 Collapse identical lines with a count badge

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Repeated identical output lines fold into one row with a number badge showing how many times it occurred.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.console.collapseIdenticalLines
- Source: V\src\vs\workbench\contrib\debug\browser\replViewer.ts; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; C bundle (debug.console.collapseIdenticalLines, count-badge-wrapper)
- Pieces:
  1. Repeated identical output lines fold into one row with a number badge showing how many times the line occurred.
  2. The badge count goes up as more identical lines arrive.
  3. Setting `debug.console.collapseIdenticalLines`, on by default, chooses whether identical lines are folded, and when off each repeat shows as its own row.

#### PN-6.9 Expand and collapse output groups

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Grouped output from the program (console.group style) shows as rows you expand or collapse, and some groups open by themselves. Collapse All folds every group at once.
- Commands: debug.collapseRepl
- Menus: DebugConsoleContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\workbench\contrib\debug\common\replModel.ts; C bundle (debug.collapseRepl, startGroup)
- Pieces:
  1. Grouped output from the program, such as `console.group`, shows as a row with a twisty that expands and collapses the lines inside it.
  2. Some groups open by themselves when they start, and others start collapsed.
  3. Choosing Collapse All (`debug.collapseRepl`) in the console right-click menu folds every output group at once.

#### PN-6.10 Filter the console output

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A filter box in the Debug Console header hides output lines that do not match. Separate terms with commas, start a term with ! to exclude it, or with a backslash to match it literally. A badge says "Showing X of Y", and the filter text and its history are kept per workspace. Typed input and results are never hidden.
- Commands: repl.action.filter
- Menus: none found
- Keys: Ctrl+F (when text input focus in the console input)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\workbench\contrib\debug\browser\replFilter.ts; C bundle (repl.action.filter, debug.repl.filterValue, debug.repl.filterHistory)
- Pieces:
  1. A filter box in the Debug Console header with the placeholder "Filter (e.g. text, !exclude, \escape)" hides output lines that do not match the typed text.
  2. Separating terms with commas matches lines that contain any of the terms.
  3. Starting a term with `!` excludes lines that match it.
  4. Starting a term with a backslash matches that term literally.
  5. A badge in the filter box says "Showing X of Y" while the filter hides some lines.
  6. Typed input and evaluation results are never hidden by the filter.
  7. The filter text and its history are kept per workspace.
  8. Pressing Ctrl+F (`repl.action.filter`) in the console input moves focus to the filter box.

#### PN-6.11 Find text in the console

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Open the list's find box to search the console output, from the search icon in the header, the right-click menu or a key.
- Commands: repl.action.find; list.find.replInputFocus
- Menus: ViewTitle; DebugConsoleContext
- Keys: Ctrl+Alt+F (when in the debug console or the console view is focused)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; C bundle (repl.action.find)
- Pieces:
  1. Clicking the search icon in the Debug Console header (`repl.action.find`) opens the find box for the console output.
  2. Choosing the find entry in the console right-click menu opens the find box.
  3. Pressing Ctrl+Alt+F while the console or its input is focused opens the find box.
  4. Typing in the find box searches the console output list.

#### PN-6.12 Wrap long lines

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Turn wrapping of long output lines on or off. Changing it rebuilds the list, and when it is off the list scrolls sideways.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.console.wordWrap
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; C bundle (debug.console.wordWrap)
- Pieces:
  1. Setting `debug.console.wordWrap`, on by default, makes long output lines wrap onto several rows.
  2. With `debug.console.wordWrap` off, long lines stay on one row and the list scrolls sideways.
  3. Changing `debug.console.wordWrap` rebuilds the console list right away with the new wrapping.

#### PN-6.13 Set the console font and line height

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Choose the font size, font family and line height used in the console list and input. A line height of 0 works it out from the font size.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.console.fontSize; debug.console.fontFamily; debug.console.lineHeight
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts (ReplOptions); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; C bundle (debug.console.fontSize, debug.console.fontFamily, debug.console.lineHeight)
- Pieces:
  1. Setting `debug.console.fontSize` sets the font size in pixels for the console list and input, default 14 (12 on macOS).
  2. Setting `debug.console.fontFamily` sets the font family for the console list and input, default `default`.
  3. Setting `debug.console.lineHeight` sets the line height in pixels for the console list and input, default 0.
  4. A `debug.console.lineHeight` of 0 makes the line height follow from the font size.

#### PN-6.14 Clear the console

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Remove all output from the current console, from the header icon, the right-click menu or the Command Palette. Clearing an ended session also drops it from the console list.
- Commands: workbench.debug.panel.action.clearReplAction
- Menus: ViewTitle; DebugConsoleContext; CommandPalette
- Keys: Cmd+K on macOS only (when the console view is focused)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; C bundle (workbench.debug.panel.action.clearReplAction, Clear Console)
- Pieces:
  1. Clicking the Clear Console icon in the Debug Console header removes all output from the current console.
  2. Choosing Clear Console in the console right-click menu removes all output from the current console.
  3. Running Clear Console (`workbench.debug.panel.action.clearReplAction`) from the Command Palette removes all output from the current console.
  4. On macOS, pressing Cmd+K while the console view is focused clears the console.
  5. Clearing the console of an ended session also removes that session from the console list.

#### PN-6.15 Copy console text and paste into the input

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The console right-click menu has Copy (the selection, or the value of the clicked result), Copy All (all visible lines) and Paste (adds clipboard text to the input, only while a session is active). Copy All also exists as an editor command in the input.
- Commands: debug.replCopy; workbench.debug.action.copyAll; repl.action.copyAll; debug.replPaste
- Menus: DebugConsoleContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; C bundle (debug.replCopy, workbench.debug.action.copyAll, repl.action.copyAll, debug.replPaste)
- Pieces:
  1. Choosing Copy in the console right-click menu (`debug.replCopy`) copies the selected text, or the value of the clicked result when nothing is selected.
  2. Choosing Copy All in the console right-click menu (`workbench.debug.action.copyAll`) copies all visible lines.
  3. Choosing Paste in the console right-click menu (`debug.replPaste`) adds the clipboard text to the input, and only works while a debug session is active.
  4. The command "Debug: Console Copy All" (`repl.action.copyAll`) copies all console lines while the input has focus.

#### PN-6.16 Switch the console between debug sessions

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When more than one session has its own console, a drop-down in the header picks which session's console you see. Keys and commands move to the next or previous console, and a Command Palette entry opens a list of all consoles.
- Commands: workbench.action.debug.selectRepl; workbench.action.debug.nextConsole; workbench.action.debug.prevConsole; workbench.action.debug.selectDebugConsole
- Menus: ViewTitle; CommandPalette
- Keys: Ctrl+PageDown (next); Ctrl+PageUp (previous); Shift+Cmd+] and Shift+Cmd+[ on macOS (when in the debug console)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts (SelectReplActionViewItem); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; C bundle (workbench.action.debug.selectRepl, workbench.action.debug.nextConsole, workbench.action.debug.prevConsole, workbench.action.debug.selectDebugConsole, multiSessionRepl)
- Pieces:
  1. When more than one debug session has its own console, a drop-down in the Debug Console header (`workbench.action.debug.selectRepl`) lists those consoles and picking one shows that session's console.
  2. With only one console, the header shows no session drop-down.
  3. Pressing Ctrl+PageDown (`workbench.action.debug.nextConsole`) shows the next session's console.
  4. Pressing Ctrl+PageUp (`workbench.action.debug.prevConsole`) shows the previous session's console.
  5. On macOS, pressing Shift+Cmd+] and Shift+Cmd+[ in the debug console shows the next and previous console.
  6. Running Select Debug Console (`workbench.action.debug.selectDebugConsole`) from the Command Palette opens a list of all consoles, and choosing one shows it.

#### PN-6.17 Limit how many console lines are kept

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: Cap the number of output entries the console keeps (10000 by default). The oldest entries are dropped first.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.console.maximumLines
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; V\src\vs\workbench\contrib\debug\common\replModel.ts; C bundle (zero hits for debug.console.maximumLines and maximumLines)
- Pieces:
  1. Setting `debug.console.maximumLines`, default 10000, caps how many output entries the console keeps. [stock only]
  2. Once the console holds more entries than the cap, the oldest entries are dropped first as new output arrives. [stock only]
  3. Lowering the value below the number of entries already shown trims the list down to the new cap when the next output arrives. [stock only]

#### PN-6.18 Open the Test Results view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Test Results tab shows up once any test run has produced results. Open it with Show Output (also a button in the Test Explorer header) or the Focus command, and it jumps to the latest run.
- Commands: testing.showMostRecentOutput; workbench.panel.testResults.view.focus; workbench.panel.testResults
- Menus: ViewTitle (Test Explorer header, navigation group); CommandPalette
- Keys: Ctrl+; then Ctrl+O (testing.showMostRecentOutput)
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts; V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts; V\src\vs\workbench\contrib\testing\browser\testingOutputPeek.ts (TestResultsView); V\src\vs\workbench\services\views\browser\viewsService.ts; C bundle (testing.showMostRecentOutput, workbench.panel.testResults.view, testing.hasAnyResults)
- Pieces:
  1. The Test Results tab appears in the panel tab bar only once a test run has produced results.
  2. Running Show Output (`testing.showMostRecentOutput`) from the Command Palette opens the Test Results view on the latest run.
  3. Clicking the Show Output button in the Test Explorer header opens the Test Results view on the latest run.
  4. Pressing Ctrl+; and then Ctrl+O opens the Test Results view on the latest run.
  5. Running the focus command `workbench.panel.testResults.view.focus` brings up the Test Results view with keyboard focus in it.

#### PN-6.19 Open Test Results automatically

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Choose whether starting tests opens the Test Results view, the Test Explorer, or nothing, or whether it opens only on the first failure. Default opens Test Results when tests start.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: testing.automaticallyOpenTestResults; testing.openTesting
- Source: V\src\vs\workbench\contrib\testing\browser\testingProgressUiService.ts; V\src\vs\workbench\contrib\testing\common\configuration.ts; C bundle (testing.automaticallyOpenTestResults with openOnTestStart default, testing.openTesting migration)
- Pieces:
  1. Setting `testing.automaticallyOpenTestResults` chooses what opens when tests start, with values `neverOpen`, `openOnTestStart`, `openOnTestFailure` and `openExplorerOnTestStart`, default `openOnTestStart`.
  2. With `openOnTestStart`, starting tests opens the Test Results view.
  3. With `openOnTestFailure`, the Test Results view opens on any test failure and not when tests start.
  4. With `openExplorerOnTestStart`, starting tests opens the Test Explorer instead of Test Results.
  5. With `neverOpen`, starting tests opens neither testing view.
  6. A value in the older setting `testing.openTesting` is carried over into `testing.automaticallyOpenTestResults`.

#### PN-6.20 See the latest run as a tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A tree pane beside the details pane lists the newest run: each task with its tests under it, a status icon on every row, the parent test names shown dimmed after the label, tests sorted by state, and failure messages as child rows ("+ N more lines"). Chains with a single child fold into one row.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts; V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsViewContent.ts; C bundle (test-output-peek-tree, testing-stdtree, Test Output Peek)
- Pieces:
  1. A tree pane beside the details pane lists the newest run, with each task as a row and its tests under it.
  2. Every row has a status icon showing the state of that task, test or message.
  3. The names of a test's parent tests show dimmed after the test's own label.
  4. Tests are sorted by their state.
  5. A test's failure messages show as child rows, and a long message is cut off with a "+ N more lines" label.
  6. A chain of rows that each have a single child folds into one row.

#### PN-6.21 Browse older runs

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Earlier runs sit in a collapsed "N older results" row under the newest run. Expand it to open any past run, with its own tasks and tests. Stock keeps up to 128 results.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts (OlderResultsElement); V\src\vs\workbench\contrib\testing\common\testResultStorage.ts; C bundle (older- element id in the results tree)
- Pieces:
  1. Earlier runs sit in a collapsed "N older results" row under the newest run, reading "1 older result" when there is one.
  2. Expanding the older results row lists the past runs.
  3. Opening a past run shows its own tasks and tests in the tree.
  4. The list keeps up to 128 results and drops the oldest beyond that.

#### PN-6.22 Read a failed test message

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick a message row (or a test with a message) and the details pane shows the failure text, as colored plain text or rendered markdown, under a header with the test name.
- Commands: none found
- Menus: TestMessageContent
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsOutput.ts (PlainTextMessagePeek, MarkdownTestMessagePeek); V\src\vs\workbench\contrib\testing\browser\testMessageColorizer.ts; V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsSubject.ts; C bundle (messageUri, markdown.showPreview)
- Pieces:
  1. Picking a message row, or a test that has a message, shows the failure text in the details pane under a header with the test name.
  2. A message sent as plain text shows as colored plain text.
  3. A message sent as markdown shows as rendered markdown.

#### PN-6.23 Compare expected and actual values in a diff

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a failure carries an expected and an actual value, the details pane shows them in a side by side diff labeled "Expected result" and "Actual result", with line numbers when either value spans several lines.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsOutput.ts (DiffContentProvider); C bundle (expectedUri, actualUri, isDiffable)
- Pieces:
  1. When a failure carries an expected and an actual value, the details pane shows them in a side by side diff.
  2. The two sides are labeled "Expected result" and "Actual result".
  3. Line numbers show in the diff when either value spans several lines.

#### PN-6.24 Read test or run output in a terminal pane

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Choose Show Result Output on a run or a test and the details pane streams its output in a read-only terminal view, with colors and links that resolve from the test's folder. Empty cases say "The test case did not report any output." or "The test run did not record any output." and old runs say output is only kept for new runs.
- Commands: none found
- Menus: TestPeekElement
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsOutput.ts (TerminalMessagePeek); V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts (the Show Result Output row action, built in code); C bundle (testing.outputPeek.showResultOutput, createDetachedTerminal, selectMarkedRange)
- Pieces:
  1. Choosing Show Result Output on a run or a test row streams its output into a read-only terminal view in the details pane.
  2. The streamed output shows its colors, and links in it resolve from the test's folder.
  3. A test with no output shows "The test case did not report any output."
  4. A run with no output shows "The test run did not record any output."
  5. An old run shows "Test output is only available for new test runs."

#### PN-6.25 Read the stack trace under a failure

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a message has stack frames, the details pane lists them under the message with a file and line for each. Click a frame to open that spot, and fold or unfold frames. The Collapse Stack Frames button exists only on the editor peek toolbar, not on the panel view.
- Commands: testing.collapsePeekStack; callStackWidget.goToFile
- Menus: TestPeekTitle; DebugCallStackToolbar
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsViewContent.ts; V\src\vs\workbench\contrib\testing\browser\testingOutputPeek.ts; V\src\vs\workbench\contrib\debug\browser\callStackWidget.ts; C bundle (testing.collapsePeekStack, callStackWidget, test-output-call-stack-container)
- Pieces:
  1. When a failure message has stack frames, the details pane lists them under the message with a file and line for each frame.
  2. Clicking a stack frame opens that file at that line in the editor.
  3. Stack frames can be folded and unfolded in the list.
  4. The Collapse Stack Frames button (`testing.collapsePeekStack`) shows on the toolbar of the editor peek and folds every frame, and the Test Results panel header has no such button.

#### PN-6.26 Cancel, rerun or debug a whole run

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A run row has hover buttons: Cancel Test Run while it is running, otherwise Rerun Last Run and Debug Last Run. The same actions are chords in the Command Palette, plus a Rerun Last Run with Coverage entry.
- Commands: testing.cancelRun; testing.reRunLastRun; testing.debugLastRun; testing.coverageLastRun
- Menus: TestPeekElement; CommandPalette; ViewTitle (Test Explorer header, cancel only)
- Keys: Ctrl+; then Ctrl+X (cancel); Ctrl+; then L (rerun last); Ctrl+; then Ctrl+L (debug last); Ctrl+; then Ctrl+Shift+L (coverage last)
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts (TreeActionsProvider, row actions built in code); V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts; C bundle (testing.cancelRun, testing.reRunLastRun, testing.debugLastRun, testing.coverageLastRun, testing.outputPeek.rerun)
- Pieces:
  1. While a run is still going, hovering its row in the results tree shows a Cancel Test Run button.
  2. When a run is not running, hovering its row shows Rerun Last Run and Debug Last Run buttons.
  3. Clicking Rerun Last Run runs the same tests again, and clicking Debug Last Run runs them again under the debugger.
  4. The Test Explorer header also shows Cancel Test Run while a run is going, with no rerun or debug buttons.
  5. The Command Palette lists Cancel Test Run (`testing.cancelRun`), Rerun Last Run (`testing.reRunLastRun`), Debug Last Run (`testing.debugLastRun`) and Rerun Last Run with Coverage (`testing.coverageLastRun`).
  6. Pressing Ctrl+; then Ctrl+X cancels, Ctrl+; then L reruns the last run, Ctrl+; then Ctrl+L debugs the last run, and Ctrl+; then Ctrl+Shift+L reruns it with coverage.

#### PN-6.27 Rerun or debug only the failed tests of a run

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: When a run has failed tests, its row also offers Rerun Failed Tests and Debug Failed Tests. The Command Palette has the matching "from Last Run" commands. The Test Explorer's own Rerun Failed Tests command (testing.reRunFailTests) is present in both products and belongs to SB-6.
- Commands: testing.reRunFailedFromLastRun; testing.debugFailedFromLastRun
- Menus: TestPeekElement; CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts; V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts; C bundle (zero hits for testing.reRunFailedFromLastRun, testing.debugFailedFromLastRun and the failed-test row actions)
- Pieces:
  1. When a run has failed tests, its row also shows Rerun Failed Tests and Debug Failed Tests buttons on hover. [stock only]
  2. A run with no failed tests shows neither of those buttons. [stock only]
  3. Clicking Rerun Failed Tests runs only the failed tests of that run again, and Debug Failed Tests runs them under the debugger. [stock only]
  4. The Command Palette lists Rerun Failed Tests from Last Run (`testing.reRunFailedFromLastRun`) and Debug Failed Tests from Last Run (`testing.debugFailedFromLastRun`). [stock only]

#### PN-6.28 Rerun or debug one test from its row or the details header

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Test rows show Run Test and Debug Test buttons, and the details pane header has Rerun Test and Debug Test for the test being shown (or the whole run when a run is shown).
- Commands: testing.callStack.run; testing.callStack.debug; vscode.runTestsById
- Menus: TestCallStack; TestPeekElement
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsViewContent.ts; V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts; C bundle (testing.callStack.run, testing.callStack.debug, testing.outputPeek.runTest, testing.outputPeek.debugTest, vscode.runTestsById)
- Pieces:
  1. A test row shows a Run Test button, when the test can be run, that runs just that test.
  2. A test row shows a Debug Test button, when the test can be debugged, that runs just that test under the debugger.
  3. The details pane header shows a Rerun Test button (`testing.callStack.run`) that runs the test being shown again.
  4. The details pane header shows a Debug Test button (`testing.callStack.debug`) that runs the test being shown again under the debugger.
  5. When a whole run is shown in the details pane instead of a test, the header Rerun and Debug buttons act on the whole run.

#### PN-6.29 Jump from a result to the test, the error line or the Test Explorer

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (Go to Test appears on message rows only, stock also shows it on test rows)
- What: Row buttons open the test's source (Go to Test) or the failing line (Go to Error), and the right-click menu has Reveal in Test Explorer. Selecting a test row also selects it in the Test Explorer.
- Commands: vscode.revealTest; _revealTestInExplorer
- Menus: TestPeekElement; TestMessageContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts; C bundle (testing.outputPeek.goToTest, testing.outputPeek.goToError, testing.outputPeek.revealInExplorer, vscode.revealTest, _revealTestInExplorer)
- Pieces:
  1. The Go to Test button on a message row opens the test's source file at the test.
  2. In stock VS Code, the Go to Test button also shows on test rows. [stock only]
  3. In Cursor, the Go to Test button shows on message rows only. [cursor only]
  4. The Go to Error button on a message row that has a location opens the failing line in the editor without moving keyboard focus.
  5. Right-clicking a test or message row offers Reveal in Test Explorer, which shows that test in the Test Explorer.
  6. Selecting a test row in the results tree also selects that test in the Test Explorer.

#### PN-6.30 Clear all results

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The header icon, or the Command Palette entry, throws away every stored run. The Test Results tab then hides itself because there is nothing left to show.
- Commands: testing.clearTestResults
- Menus: ViewTitle (Test Results header, navigation group); TestPeekTitle; CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (ClearTestResultsAction); V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts; C bundle (testing.clearTestResults on the view title when view is workbench.panel.testResults.view)
- Pieces:
  1. Clicking the Clear All Results icon in the Test Results header throws away every stored run.
  2. Running Clear All Results (`testing.clearTestResults`) from the Command Palette throws away every stored run.
  3. After results are cleared, the Test Results tab hides itself because there is nothing left to show.
  4. Clear All Results is also a button on the title bar of the editor peek.

#### PN-6.31 Swap the tree and details sides

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: A header button, or a setting, moves the run tree between the right and left of the details pane.
- Commands: testing.toggleResultsViewLayout
- Menus: ViewTitle (Test Results header, navigation group)
- Keys: none found
- Settings: testing.resultsView.layout
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (ToggleResultsViewLayoutAction); V\src\vs\workbench\contrib\testing\common\configuration.ts; V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsViewContent.ts; C bundle (zero hits for testing.toggleResultsViewLayout, testing.resultsView.layout, treeLeft)
- Pieces:
  1. A Toggle Tree Position button in the Test Results header (`testing.toggleResultsViewLayout`) moves the run tree to the other side of the details pane. [stock only]
  2. Setting `testing.resultsView.layout` chooses where the run tree sits, with values `treeRight` and `treeLeft`, default `treeRight`. [stock only]
  3. Changing the layout, from the button or the setting, rearranges an open Test Results view right away. [stock only]

#### PN-6.32 Run a follow-up action offered under a failure

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A test extension can offer follow-up links under a failed message. The first shows as a link, any others sit behind "+N More..." which opens a pick list. The links come from the extension, nothing built in supplies them.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsViewContent.ts (FollowupActionWidget); C bundle (provideTestFollowups, testing-followup-action)
- Pieces:
  1. When a test extension offers follow-up actions for a failed message, the first one shows as a link under the message.
  2. When there is more than one follow-up, the rest sit behind a "+N More..." link.
  3. Clicking "+N More..." opens a pick list of the remaining follow-up actions, and choosing one runs it.
  4. A failed message with no follow-up actions shows no links, because nothing built in supplies them.


## PN-7 Context-only panel views

(both). Tabs that appear only when something feeds them. Examples: Comments; Ports; Refactor Preview. Ports is a remote feature, so it is marked for your call.

27 features.

#### PN-7.1 See all comment threads in the Comments view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Comments tab lists every comment thread in the workspace under its file. Each row shows a state icon (comment, unresolved or draft), the author, the body, the line range, the time, the reply count, and an "Outdated" tag when it applies. The tab only exists once an extension supplies comments, and an empty list says "There are no comments in this workspace yet."
- Commands: workbench.action.focusCommentsPanel
- Menus: none found
- Keys: none found
- Settings: comments.useRelativeTime
- Source: V\src\vs\workbench\contrib\comments\browser\commentsView.ts; V\src\vs\workbench\contrib\comments\browser\commentsTreeViewer.ts; V\src\vs\workbench\contrib\comments\browser\commentsModel.ts; V\src\vs\workbench\api\browser\mainThreadComments.ts; C bundle (workbench.action.focusCommentsPanel, comments.useRelativeTime, commentsView.hasComments)
- Pieces:
  1. The Comments tab lists every comment thread in the workspace as rows nested under a row for the file they belong to.
  2. Each thread row starts with a state icon that shows whether the thread is a plain comment, unresolved or a draft.
  3. Each thread row shows the author, the comment body, the line range such as `[Ln 3-5]` and the time.
  4. A thread with replies shows a second line with the reply count such as "2 replies", "Last reply from" the latest author, and the time of that reply.
  5. A thread whose code has changed shows an "Outdated" tag on its row.
  6. Setting `comments.useRelativeTime`, on by default, chooses whether comment times read as relative text such as "1 day ago" or as a full local date and time.
  7. With no comments in the workspace the view shows "There are no comments in this workspace yet."
  8. The Comments tab only appears in the panel once an extension supplies comments.
  9. Running `workbench.action.focusCommentsPanel` brings up the Comments view with keyboard focus in it.

#### PN-7.2 Open a comment from the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Click or press Enter on a file row or a thread row to open that file and reveal the thread in the editor. The usual open-to-the-side and keep-preview gestures apply.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\comments\browser\commentsView.ts (openFile); V\src\vs\workbench\contrib\comments\browser\commentsController.ts (revealCommentThread); C bundle (CommentsViewThreadActions, workbench.panel.comments)
- Pieces:
  1. Clicking a thread row opens its file in the editor and reveals the thread there.
  2. Pressing Enter on a thread row opens its file and reveals the thread.
  3. Clicking or pressing Enter on a file row opens that file in the editor.
  4. Using the usual open-to-the-side gesture on a row opens the file in an editor beside the current one.
  5. Using the keep-preview gesture on a row makes the opened editor tab permanent instead of a preview tab.

#### PN-7.3 Filter comments by text or author

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A filter box in the Comments header hides threads that do not match. It matches the file name, comment text, author and replies, and a leading ! excludes matches. A badge says "Showing X of Y", and the filter text and history are kept per workspace.
- Commands: commentsFocusFilter; commentsFocusViewFromFilter; commentsClearFilterText
- Menus: none found
- Keys: Ctrl+F (when the Comments view is focused); Ctrl+Down Arrow (when the filter has focus, moves to the list); Escape (when the filter has focus, clears it)
- Settings: none found
- Source: V\src\vs\workbench\contrib\comments\browser\commentsView.ts; V\src\vs\workbench\contrib\comments\browser\commentsViewActions.ts; V\src\vs\workbench\contrib\comments\browser\commentsFilterOptions.ts; V\src\vs\workbench\contrib\comments\browser\commentsTreeViewer.ts (Filter); C bundle (commentsFocusFilter, commentsFocusViewFromFilter, commentsClearFilterText)
- Pieces:
  1. A filter box in the Comments header with the placeholder "Filter (e.g. text, author)" hides threads that do not match the typed text.
  2. The filter matches the file name, the comment text, the author and the replies.
  3. Starting the filter text with `!` hides the threads that match instead of keeping them.
  4. A badge in the filter box says "Showing X of Y" while the filter hides some threads.
  5. The filter text and its history are kept per workspace.
  6. Pressing Ctrl+F while the Comments view is focused moves focus into the filter box (`commentsFocusFilter`).
  7. Pressing Ctrl+Down Arrow in the filter box moves focus to the list (`commentsFocusViewFromFilter`).
  8. Pressing Escape in the filter box clears its text (`commentsClearFilterText`).

#### PN-7.4 Show or hide resolved and unresolved threads

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The filter drop-down next to the box has two checked items, Show Resolved and Show Unresolved, to hide either kind of thread. The choice is saved per workspace.
- Commands: workbench.actions.workbench.panel.comments.toggleResolvedComments; workbench.actions.workbench.panel.comments.toggleUnResolvedComments
- Menus: submenu.view.filter
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\comments\browser\commentsViewActions.ts; V\src\vs\workbench\browser\parts\views\viewFilter.ts; C bundle (toggleResolvedComments, toggleUnResolvedComments, commentsView.showResolvedFilter, commentsView.showUnResolvedFilter)
- Pieces:
  1. The filter drop-down next to the filter box has Show Resolved and Show Unresolved items, both checked at first.
  2. Unchecking Show Resolved (`workbench.actions.workbench.panel.comments.toggleResolvedComments`) hides resolved threads.
  3. Unchecking Show Unresolved (`workbench.actions.workbench.panel.comments.toggleUnResolvedComments`) hides unresolved threads.
  4. The two choices are saved per workspace.

#### PN-7.5 Sort comments by file position or last update

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A Sort By submenu in the filter drop-down orders the list by position in the file (default) or by most recently updated thread.
- Commands: workbench.actions.workbench.panel.comments.toggleSortByResource; workbench.actions.workbench.panel.comments.toggleSortByUpdatedAt
- Menus: submenu.filter.commentSort (inside submenu.view.filter)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\comments\browser\commentsViewActions.ts; V\src\vs\workbench\contrib\comments\browser\commentsView.ts; C bundle (toggleSortByResource, toggleSortByUpdatedAt, submenu.filter.commentSort, commentsView.sortBy)
- Pieces:
  1. A Sort By submenu in the filter drop-down offers Position in File and Updated Time.
  2. Position in File, the default, orders threads by where they sit in their files.
  3. Updated Time orders threads with the most recently updated first.
  4. The active sort shows a check mark in the Sort By submenu.

#### PN-7.6 Expand or collapse every thread

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: One header icon expands all threads, or collapses them when any is open. Only one of the two shows at a time.
- Commands: comments.expand; comments.collapse
- Menus: ViewTitle
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\comments\browser\comments.contribution.ts; V\src\vs\workbench\contrib\comments\browser\commentsView.ts; C bundle (comments.expand, comments.collapse)
- Pieces:
  1. Clicking the Expand All icon in the Comments header (`comments.expand`) opens every thread row to show its replies.
  2. Clicking the Collapse All icon in the Comments header (`comments.collapse`) closes every thread row.
  3. Only one of the two icons shows at a time, with Collapse All showing while any thread is expanded.
  4. Neither icon shows while the view has no comments.

#### PN-7.7 Reply to a thread from the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A Reply button on a thread row opens the thread in the editor with the reply box focused. It is only enabled when the thread allows replies.
- Commands: comments.reply
- Menus: CommentsViewThreadActions; AccessibleView
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\comments\browser\comments.contribution.ts; V\src\vs\workbench\contrib\comments\browser\commentsTreeViewer.ts (CommentsMenus); C bundle (comments.reply, CommentsViewThreadActions)
- Pieces:
  1. A Reply button on a thread row opens the thread in the editor with the reply box focused (`comments.reply`).
  2. The Reply button is enabled only when the thread allows replies.
  3. The accessible view of a comment thread also offers the Reply action.

#### PN-7.8 See the unresolved count on the Comments tab

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Comments tab carries a number badge with how many threads are unresolved, with the tooltip "N Unresolved Comments".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\comments\browser\comments.contribution.ts (UnresolvedCommentsBadge); C bundle (showViewActivity with the comments view id and a number badge)
- Pieces:
  1. The Comments tab carries a number badge with how many threads are unresolved.
  2. Hovering the badge shows the tooltip "N Unresolved Comments".
  3. The badge number changes as threads are added, resolved or deleted.

#### PN-7.9 Open the Comments view automatically

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Choose whether the Comments view opens by itself when you open a file that has comments: never, every such file, the first such file in a session, or the first with an unresolved thread. An older setting with similar choices is deprecated.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: comments.openView; comments.openPanel
- Source: V\src\vs\workbench\contrib\comments\browser\comments.contribution.ts; V\src\vs\workbench\contrib\comments\browser\commentsController.ts (openCommentsView); C bundle (comments.openView, comments.openPanel, firstFileUnresolved)
- Pieces:
  1. Setting `comments.openView` chooses when the Comments view opens by itself, with values `never`, `file`, `firstFile` and `firstFileUnresolved`, default `firstFile`.
  2. With `never`, the Comments view is never opened by itself.
  3. With `file`, the Comments view opens whenever a file with comments becomes the active file.
  4. With `firstFile`, the Comments view opens the first time in a session that a file with comments is active, and not again.
  5. With `firstFileUnresolved`, the Comments view opens the first time in a session that a file with an unresolved comment is active.
  6. The older setting `comments.openPanel`, with values `neverOpen`, `openOnSessionStart` and `openOnSessionStartWithComments`, is marked deprecated in favor of `comments.openView`.

#### PN-7.10 See forwarded ports in the Ports view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Ports tab lists forwarded and detected ports in a table with Port, Forwarded Address, Running Process and Origin columns, plus a Visibility column when the tunnel provider can change visibility. When nothing is forwarded it shows "No forwarded ports" text with a Forward a Port link. The tab appears only when port forwarding is switched on (a remote window, or after a local port is forwarded). This is a remote feature.
- Commands: ~remote.forwardedPorts.focus
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts; V\src\vs\workbench\contrib\remote\browser\remoteExplorer.ts; V\src\vs\workbench\services\remote\common\remoteExplorerService.ts; C bundle (~remote.forwardedPortsContainer, ~remote.forwardedPorts, tunnelViewFocus, forwardedPortsViewEnabled)
- Pieces:
  1. The Ports tab lists forwarded and detected ports in a table with the columns Port, Forwarded Address, Running Process and Origin.
  2. When the tunnel provider can change visibility, a Visibility column shows between Running Process and Origin.
  3. With nothing forwarded in a remote window, the view shows "No forwarded ports. Forward a port to access your running services locally." and a Forward a Port link.
  4. With nothing forwarded in a local window, the view shows "No forwarded ports. Forward a port to access your locally running services over the internet." and a Forward a Port link.
  5. Hovering a port row shows a tooltip saying which remote port is forwarded to which local address, or that the remote port is not forwarded.
  6. Hovering a column header shows a tooltip describing that column.
  7. The Ports tab appears only when port forwarding is switched on, which is in a remote window or after a local port has been forwarded.
  8. Running `~remote.forwardedPorts.focus` brings up the Ports view with keyboard focus in it.

#### PN-7.11 Forward a port

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Forward a port by number or address (for example 3000 or 10.10.10.10:2000) from the add row, the empty-state link, a "Forward Port" button on a detected port, the right-click menu, or the Command Palette.
- Commands: remote.tunnel.forwardInline; remote.tunnel.forwardCommandPalette
- Menus: TunnelContext; TunnelPortInline; CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (ForwardPortAction); V\src\vs\workbench\contrib\remote\browser\remoteExplorer.ts; C bundle (remote.tunnel.forwardInline, remote.tunnel.forwardCommandPalette, Forward a Port)
- Pieces:
  1. An Add Port button in the add row of the Ports table prompts for a port number or address with the hint "Port number or address (eg. 3000 or 10.10.10.10:2000)." and forwards it (`remote.tunnel.forwardInline`).
  2. The Forward a Port link in the empty Ports view starts the same prompt.
  3. A detected port that is not yet forwarded shows a Forward Port button on its row.
  4. The right-click menu in the Ports view has a Forward a Port entry.
  5. Running Forward a Port (`remote.tunnel.forwardCommandPalette`) from the Command Palette starts the same prompt.
  6. Typing text that is not a number or a host and port shows "Forwarded port should be a number or a host:port.", and typing a port that is already forwarded shows "Port is already forwarded".
  7. When forwarding fails, a warning says "Unable to forward" the host and port, naming that the host may not be available or the port may already be forwarded.

#### PN-7.12 Stop forwarding a port

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Remove a forwarded port from its row button, the right-click menu, the Delete key, or a pick list from the Command Palette.
- Commands: remote.tunnel.closeInline; remote.tunnel.closeCommandPalette
- Menus: TunnelContext; TunnelPortInline; CommandPalette
- Keys: Delete (Cmd+Backspace on macOS; when the Ports view is focused and the port can be closed)
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (ClosePortAction); C bundle (remote.tunnel.closeInline, remote.tunnel.closeCommandPalette, Stop Forwarding Port)
- Pieces:
  1. A Stop Forwarding Port button on a forwarded port's row removes that port (`remote.tunnel.closeInline`).
  2. The right-click menu on a forwarded port has a Stop Forwarding Port entry.
  3. Pressing Delete, or Cmd+Backspace on macOS, while the Ports view is focused removes the selected port when it can be closed.
  4. Running Stop Forwarding Port (`remote.tunnel.closeCommandPalette`) from the Command Palette opens a list titled "Choose a port to stop forwarding" and removes the chosen port.
  5. With no forwarded ports, the Command Palette list says "No ports currently forwarded. Try running the Forward a Port command".

#### PN-7.13 Open a forwarded port in the browser or in an editor tab

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Open the forwarded address in your browser, or choose Preview in Editor to show it in an editor tab. Both are inline buttons on the address, entries in the right-click menu, and Command Palette lists.
- Commands: remote.tunnel.open; remote.tunnel.openPreview; remote.tunnel.openCommandPalette
- Menus: TunnelContext; TunnelLocalAddressInline; CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (OpenPortInBrowserAction, OpenPortInPreviewAction); C bundle (remote.tunnel.open, remote.tunnel.openPreview, remote.tunnel.openCommandPalette, openPreviewEnabled)
- Pieces:
  1. An Open in Browser button on a forwarded address opens that address in your default browser (`remote.tunnel.open`).
  2. A Preview in Editor button on a forwarded address shows that address in an editor tab (`remote.tunnel.openPreview`).
  3. The right-click menu on a forwarded port has Open in Browser and Preview in Editor entries.
  4. Running Open Port in Browser (`remote.tunnel.openCommandPalette`) from the Command Palette opens a list titled "Choose the port to open" and opens the chosen port.
  5. With no forwarded ports, the Open Port in Browser list says "No ports currently forwarded. Open the Ports view to get started." and offers "Open the Ports view...".
  6. The forwarded address is a link whose hover text names the click needed to follow it, such as "ctrl + click".

#### PN-7.14 Copy a port's local address

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Copy the forwarded address from an inline button, the right-click menu, Ctrl+C on the row, or a pick list in the Command Palette.
- Commands: remote.tunnel.copyAddressInline; remote.tunnel.copyAddressCommandPalette
- Menus: TunnelContext; TunnelLocalAddressInline; CommandPalette
- Keys: Ctrl+C (when the Ports view is focused and one forwarded or detected port is selected)
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (CopyAddressAction); C bundle (remote.tunnel.copyAddressInline, remote.tunnel.copyAddressCommandPalette)
- Pieces:
  1. A Copy Local Address button on a forwarded address copies that address (`remote.tunnel.copyAddressInline`).
  2. The right-click menu on a forwarded port has a Copy Local Address entry.
  3. Pressing Ctrl+C while the Ports view is focused and one forwarded or detected port is selected copies its address.
  4. Running Copy Forwarded Port Address (`remote.tunnel.copyAddressCommandPalette`) from the Command Palette opens a list titled "Choose a forwarded port" and copies the chosen port's address.

#### PN-7.15 Label a port

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Give a forwarded port a name that shows in the Port column, from an inline button, the right-click menu or F2.
- Commands: remote.tunnel.label
- Menus: TunnelContext; TunnelPortInline
- Keys: F2 (Enter on macOS; when a single forwarded port is focused)
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (LabelTunnelAction); C bundle (remote.tunnel.label)
- Pieces:
  1. A Set Port Label button on a port's row opens an inline text box in the Port column with the placeholder "Port label" (`remote.tunnel.label`).
  2. The right-click menu on a forwarded port has a Set Port Label entry that opens the same text box.
  3. Pressing F2 (Enter on macOS) while a single forwarded port is focused opens the same text box.
  4. Pressing Enter in the text box saves the name and shows it in the Port column, and pressing Escape cancels.

#### PN-7.16 Change the local port number

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick a different local port for a forwarded port from the right-click menu. It warns when the port is taken and another was used.
- Commands: remote.tunnel.changeLocalPort
- Menus: TunnelContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (ChangeLocalPortAction); C bundle (remote.tunnel.changeLocalPort)
- Pieces:
  1. Choosing Change Local Address Port in the right-click menu of a forwarded port opens an inline text box with the placeholder "New local port" (`remote.tunnel.changeLocalPort`).
  2. Typing something that is not a number shows "Local port should be a number."
  3. When the chosen local port is taken, a warning says that port is not available and names the port number that was used instead.
  4. The Change Local Address Port entry only shows for a forwarded port whose local port can be changed.

#### PN-7.17 Set a port's visibility

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A Port Visibility submenu in the right-click menu switches a forwarded port between the options the tunnel provider supports, such as Private and Public, with a check next to the current one. It only shows when the provider allows it. The command ids are built per option at run time.
- Commands: remote.tunnel.privacy<id> (one per visibility option)
- Menus: TunnelContext; TunnelPrivacy
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (registerPrivacyActions); C bundle (remote.tunnel.privacy, TunnelPrivacy)
- Pieces:
  1. A Port Visibility submenu in the right-click menu of a forwarded port lists the visibility options the tunnel provider supports, such as Private and Public.
  2. The current visibility option shows a check mark in the submenu.
  3. Choosing another option changes the port's visibility (`remote.tunnel.privacy<id>`).
  4. The Port Visibility submenu only shows when the tunnel provider allows changing visibility.

#### PN-7.18 Change a port's protocol

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A Change Port Protocol submenu in the right-click menu sets a forwarded port to HTTP or HTTPS.
- Commands: remote.tunnel.setProtocolHttp; remote.tunnel.setProtocolHttps
- Menus: TunnelContext; TunnelProtocol
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (SetTunnelProtocolAction); C bundle (remote.tunnel.setProtocolHttp, remote.tunnel.setProtocolHttps, TunnelProtocol)
- Pieces:
  1. A Change Port Protocol submenu in the right-click menu of a forwarded port offers HTTP and HTTPS.
  2. Choosing HTTP (`remote.tunnel.setProtocolHttp`) or HTTPS (`remote.tunnel.setProtocolHttps`) sets the forwarded port to that protocol.
  3. The current protocol shows a check mark in the submenu.
  4. The Change Port Protocol submenu only shows when the tunnel provider allows changing the protocol.

#### PN-7.19 Forward ports automatically

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Turn on automatic forwarding of ports that running programs open, choose how they are detected (process watching, terminal and debug output, or a hybrid of the two), set when it falls back to the hybrid mode, forward local URLs opened from the terminal, restore forwarded ports in the workspace, and pick the local host name used.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: remote.autoForwardPorts; remote.autoForwardPortsSource; remote.autoForwardPortsFallback; remote.forwardOnOpen; remote.restoreForwardedPorts; remote.localPortHost
- Source: V\src\vs\workbench\contrib\remote\common\remote.contribution.ts; V\src\vs\workbench\contrib\remote\browser\remoteExplorer.ts; V\src\vs\workbench\contrib\remote\browser\urlFinder.ts; C bundle (remote.autoForwardPorts, remote.autoForwardPortsSource, remote.autoForwardPortsFallback, remote.forwardOnOpen, remote.restoreForwardedPorts, remote.localPortHost)
- Pieces:
  1. Setting `remote.autoForwardPorts`, on by default, chooses whether ports that new running processes listen on are forwarded automatically.
  2. Setting `remote.autoForwardPortsSource` chooses how ports are detected, with values `process` (watch started processes), `output` (read terminal and debug output) and `hybrid` (read output and un-forward by watching processes), default `process`.
  3. Setting `remote.autoForwardPortsFallback`, default 20, sets how many auto forwarded ports switch detection from `process` to `hybrid`, and 0 turns the switch off.
  4. Setting `remote.forwardOnOpen`, on by default, chooses whether local URLs with a port are forwarded when opened from the terminal or the debug console.
  5. Setting `remote.restoreForwardedPorts`, on by default, chooses whether ports you forwarded in a workspace come back when it reopens.
  6. Setting `remote.localPortHost` chooses the local host name used for forwarding, with values `localhost` and `allInterfaces`, default `localhost`.

#### PN-7.20 Set rules for specific ports

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Write per-port rules in settings, by number, range, host and port, or process pattern: a label, what happens on automatic forwarding (notify, open browser, open preview, stay silent, ignore), whether to ask for elevation, whether a modal dialog shows when the local port differs, and the protocol. A second setting gives defaults for all other ports.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: remote.portsAttributes; remote.otherPortsAttributes
- Source: V\src\vs\workbench\contrib\remote\common\remote.contribution.ts; C bundle (remote.portsAttributes, remote.otherPortsAttributes)
- Pieces:
  1. Setting `remote.portsAttributes` holds per-port rules whose keys are a port number, a range such as `40000-55000`, a host and port such as `db:1234`, or a regular expression matched against the process command line.
  2. A rule's `label` sets the name shown for that port in the Ports view.
  3. A rule's `onAutoForward` chooses what happens on automatic forwarding, with values `notify`, `openBrowser`, `openBrowserOnce`, `openPreview`, `silent` and `ignore`, default `notify`.
  4. A rule's `elevateIfNeeded` asks for elevation when the local port needs it, and its `requireLocalPort` shows a modal dialog when the chosen local port is not used, and both are off by default.
  5. A rule's `protocol` sets the protocol, with values `http` and `https`.
  6. The default value of `remote.portsAttributes` sets ports 443 and 8443 to the `https` protocol.
  7. Setting `remote.otherPortsAttributes` gives the same kind of defaults for all ports no rule matches, with `onAutoForward` values `notify`, `openBrowser`, `openPreview`, `silent` and `ignore`.

#### PN-7.21 See the forwarded port count on the Ports tab

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Ports tab carries a number badge with how many ports are forwarded, with the tooltip "1 forwarded port" or "N forwarded ports".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\remoteExplorer.ts (updateActivityBadge); C bundle (showViewActivity with the tunnel view container and tunnelModel.forwarded.size)
- Pieces:
  1. The Ports tab carries a number badge with how many ports are forwarded.
  2. Hovering the badge shows the tooltip "1 forwarded port" or "N forwarded ports".
  3. The badge changes as ports are forwarded or stopped, and is gone while no port is forwarded.

#### PN-7.22 Review pending edits in Refactor Preview

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a rename or another action asks for a preview, the Refactor Preview tab opens with a tree of the pending edits: files (with renames, creations and deletions marked), each edit with the removed text struck and the added text highlighted, and categories when edits come from several sources. When you finish, the panel goes back to the tab you had before. Idle, it says "Invoke a code action, like rename, to see a preview of its changes here."
- Commands: refactorPreview.focus
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditTree.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts; C bundle (refactorPreview.enabled, workbench.contrib.bulkEditPreview, Refactor Preview, setPreviewHandler)
- Pieces:
  1. When a rename or another action asks for a preview, the Refactor Preview tab opens in the panel with a tree of the pending edits.
  2. File rows in the tree mark renames as "old path → new path" with "(renaming)", creations with "(creating)" and deletions with "(deleting)".
  3. Each edit row shows the removed text struck through and the added text highlighted.
  4. When edits come from several sources, the tree also has category rows, and edits with no named source go under a category called "Other".
  5. When you apply or discard the preview, the panel goes back to the tab you had open before.
  6. With no preview pending, the view says "Invoke a code action, like rename, to see a preview of its changes here."
  7. Running `refactorPreview.focus` brings up the Refactor Preview view with keyboard focus in it.

#### PN-7.23 Check or uncheck edits

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Each file, edit and category has a checkbox so you can leave out parts of the refactoring. Space toggles the focused row. With nothing checked, the Apply Refactoring command is unavailable.
- Commands: refactorPreview.toggleCheckedState
- Menus: BulkEditContext
- Keys: Space (when a list has focus and a preview is active)
- Settings: none found
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts; C bundle (refactorPreview.toggleCheckedState, refactorPreview.hasCategories)
- Pieces:
  1. Each file, edit and category row has a checkbox.
  2. Unchecking a row leaves that file, edit or category out of the refactoring.
  3. Pressing Space on the focused row toggles its checkbox (`refactorPreview.toggleCheckedState`).
  4. The right-click menu in the tree has a Toggle Change entry for the selected row.
  5. With nothing checked, the Apply Refactoring command is unavailable.

#### PN-7.24 Apply the refactoring

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Apply the checked edits from the Apply button, the right-click menu or Ctrl+Enter. If a file changed in the meantime a warning says the refactoring cannot be applied.
- Commands: refactorPreview.apply
- Menus: BulkEditContext
- Keys: Ctrl+Enter (when a preview is active and the Refactor Preview view is focused)
- Settings: none found
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts; C bundle (refactorPreview.apply)
- Pieces:
  1. Clicking the Apply button in the Refactor Preview view applies the checked edits to the files.
  2. Choosing Apply Refactoring (`refactorPreview.apply`) in the right-click menu of the tree applies the checked edits.
  3. Pressing Ctrl+Enter while a preview is active and the Refactor Preview view is focused applies the checked edits.
  4. If a file changed since the preview was made, a warning says "Cannot apply refactoring because" that file "has changed in the meantime", or names how many other files changed.

#### PN-7.25 Discard the refactoring

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Throw the preview away without changing any file, from the Discard button or the right-click menu.
- Commands: refactorPreview.discard
- Menus: BulkEditContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts; C bundle (refactorPreview.discard)
- Pieces:
  1. Clicking the Discard button in the Refactor Preview view throws the preview away without changing any file.
  2. Choosing Discard Refactoring (`refactorPreview.discard`) in the right-click menu of the tree throws the preview away without changing any file.
  3. After the preview is discarded, the view goes back to its idle message.

#### PN-7.26 Group changes by file or by type

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When edits come from more than one category, a header icon or the right-click menu switches the tree between grouping by file and grouping by type. The choice is remembered across sessions.
- Commands: refactorPreview.groupByFile; refactorPreview.groupByType; refactorPreview.toggleGrouping
- Menus: BulkEditTitle; BulkEditContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts; C bundle (refactorPreview.groupByFile, refactorPreview.groupByType, refactorPreview.toggleGrouping, BulkEditTitle)
- Pieces:
  1. Clicking the grouping icon in the Refactor Preview header toggles the tree between grouping by file and grouping by type (`refactorPreview.toggleGrouping`).
  2. The right-click menu in the tree has Group Changes By File and Group Changes By Type entries (`refactorPreview.groupByFile`, `refactorPreview.groupByType`).
  3. Grouping by file lists files with their edits under them, and grouping by type lists categories with their files under them.
  4. The grouping controls only show when the edits come from more than one category.
  5. The chosen grouping is remembered across sessions.

#### PN-7.27 Open a change in the preview diff editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Opening a file or edit row shows a multi-file diff tab titled "Refactor Preview" with before and after for every file in the refactoring, scrolled to the edit you picked. It is a temporary tab that closes when the preview ends.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts (_openElementInMultiDiffEditor); V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPreview.ts; C bundle (vscode-bulkeditpreview-multieditor)
- Pieces:
  1. Opening a file or edit row in the Refactor Preview tree shows a multi-file diff tab titled "Refactor Preview".
  2. The diff tab shows the before and after of every file in the refactoring.
  3. The diff tab is scrolled to the edit you picked.
  4. The diff tab is temporary and closes when the preview ends.
