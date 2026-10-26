Group: PN-1 Panel frame and layout. Section: Panel. Date: 2026-10-25. Pass: 2.
Features: 23.

Short names used in Source: V is the stock 1.128.0 tree, N is the 1.144.0 clone, C bundle is `workbench.desktop.main.js` in the Cursor install, C css is `workbench.desktop.main.css` next to it, and NLS is `out\nls.messages.json`. Line numbers are from the files as read and can drift by a line or two. Cursor findings come from strings and compiled code only. Nothing was run on screen. Unless a path says otherwise, V paths below start at `V\src\vs\workbench\browser\`.

### PN-1.1 Show or hide the panel
- Tag: both
- Evidence: source
- Boundary: no
- What: Open or close the whole panel from View > Appearance > Panel, from the Hide Panel button at the right end of the panel title, or from the Command Palette (Toggle Panel Visibility, Hide Panel). Opening any panel view (Problems, Output, Terminal, Debug Console and so on) also shows the panel. Showing it brings back the last view used, or the first container that still has views. Hiding it closes the active view and moves focus to the editor. The panel starts hidden in a new workspace, and it hides itself when its last view container goes away.
- Commands: workbench.action.togglePanel; workbench.action.closePanel
- Menus: MenubarAppearanceMenu (group 2_workbench_layout, order 5); LayoutControlMenuSubmenu (group 0_workbench_layout, order 4); PanelTitle (group navigation, order 2); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (both commands are present with the same menu rows, but both are disabled while an auxiliary window has focus; the two layout toggle icons both use codicon panelFrame where stock uses layoutPanel and layoutPanelOff; the title bar toggle sits in group 2_pane_toggles; the Hide Panel title row is declared inside the toggle command)
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (25-28, 31-97); layout.ts (2050-2130, 2294-2309, 2828-2854); parts\paneCompositePart.ts (186-206, 510-537); C bundle (class hva with `Toggle Panel Visibility`, closePanel action with precondition on isAuxiliaryWindowFocusedContext, icons EhS and IhS bound to Oe.panelFrame); NLS 3932-3936, 3952-3953, 3974

### PN-1.2 Toggle the panel with Ctrl+J
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+J (Cmd+J on macOS) to hide the panel when it is open and show it when it is closed; the same command is in the Command Palette as Toggle Panel Visibility, hiding a maximized panel first restores the editor, and in Cursor the key first leaves the maximized layout that hides the editor before it shows the panel, and keeps that layout instead of hiding the panel when the panel is open in it.
- Commands: workbench.action.togglePanel
- Menus: CommandPalette
- Keys: Ctrl+J (Cmd+J on macOS)
- Settings: none found
- Cursor: changed (key and command present; Cursor adds `!isAuxiliaryWindowFocusedContext` to the key and the command, and a branch for the context key agentChatMaximized that calls setUnifiedMaximizeState)
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (31-72); layout.ts (2050-2130); C bundle (class hva: keybinding `Me.CtrlCmd|ve.KeyJ` with `when:FT.toNegated()`, run body reading context key Eme, method setUnifiedMaximizeState, context key `agentChatMaximized`)

### PN-1.3 Move the panel to the bottom, top, left or right
- Tag: both
- Evidence: source
- Boundary: no
- What: Put the panel under the editor (the default), above it, or beside it on the left or right. Use View > Appearance > Panel Position, the Panel Position submenu in the panel tab bar right-click menu (the current position is ticked), or four Command Palette commands. Moving a hidden panel shows it. The position is remembered per workspace, and alignment only applies to top and bottom.
- Commands: workbench.action.positionPanelTop; workbench.action.positionPanelLeft; workbench.action.positionPanelRight; workbench.action.positionPanelBottom
- Menus: MenubarAppearanceMenu (submenu Panel Position, group 3_workbench_layout_move, order 4); PanelPositionMenu; CommandPalette
- Keys: none found
- Settings: workbench.panel.defaultLocation
- Cursor: present (all four ids, the labels Move Panel To Top, Move Panel Left, Move Panel Right, Move Panel To Bottom, and the Panel Position submenu registration were found)
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (127-215); layout.ts (2362-2440, 2847); parts\panel\panelPart.ts (167-194); C bundle (position actions built from the list xhS, submenu Pe.PanelPositionMenu); NLS 3938-3941, 3946, 3955-3958

### PN-1.4 Align the panel (center, left, right, justify)
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose how far a top or bottom panel stretches. Center keeps it under the editor only (the default). Justify stretches it across the full window width, under both side bars. Left and Right stretch it to that window edge, under whichever side bar sits on that edge. Pick it from View > Appearance > Align Panel, the Align Panel submenu in the tab bar right-click menu, the Panel Alignment rows in the Customize Layout picker, or four Command Palette commands. Choosing an alignment while the panel is on the left or right first moves it to the bottom, and a non-center alignment restores a maximized panel. The choice is remembered per profile.
- Commands: workbench.action.alignPanelLeft; workbench.action.alignPanelRight; workbench.action.alignPanelCenter; workbench.action.alignPanelJustify; workbench.action.customizeLayout
- Menus: MenubarAppearanceMenu (submenu Align Panel, group 3_workbench_layout_move, order 5); PanelAlignmentMenu; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (all four ids, the labels Set Panel Alignment to Left, Right, Center and Justify, the Align Panel submenu registration, and the four Panel Alignment rows of the Customize Layout picker (list oym, NLS 3141-3144 and header 3152) were found; the picker itself is TB-4.7)
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (135-251); layout.ts (1960-2048, 2323-2325, 2848); actions\layoutActions.ts (1378-1383, 1502-1504); C bundle (align actions built from the list AhS, submenu Pe.PanelAlignmentMenu, context key panelAlignment); NLS 3942-3945, 3947, 3959-3962

### PN-1.5 Maximize and restore the panel
- Tag: both
- Evidence: source
- Boundary: no
- What: Click the Maximize button at the right of the panel title (or run Toggle Maximized Panel) to make the panel fill everything under the title bar by hiding the editor area. Click it again, now showing Restore, to bring the editor back at its earlier size. Running it while the panel is hidden shows the panel and maximizes it. It only works when the panel is center aligned or sits left or right. For a top or bottom panel with another alignment the command is disabled, and the code has a warning "Maximizing the panel is only supported when it is center aligned."
- Commands: workbench.action.toggleMaximizedPanel
- Menus: PanelTitle (group navigation, order 1); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (command, menu row and state present; the button icon is chevronUp and the restore state swaps to a new panel-restore icon (chevronDown) where stock keeps one screenFull icon; tooltips read "Maximize Panel Size" and "Restore Panel Size" where stock reads "Maximize Panel" and "Restore Panel"; a panelmaximized class is added on the main container; in unified mode with a left or right panel the secondary side bar width is kept)
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (24-25, 275-318); layout.ts (2209-2251, 1886-1893); parts\panel\media\panelpart.css (17-45, 77-100); C bundle (action with id toggleMaximizedPanel, icons ChS and ThS, method toggleMaximizedPanel with isUnifiedMode, method updatePanelMaximizedClass; C css rules for codicon-panel-maximize and codicon-panel-restore rotation); NLS 3929-3930, 3949-3951, 3965

### PN-1.6 Choose whether the panel opens maximized
- Tag: both
- Evidence: source
- Boundary: no
- What: A setting decides what happens each time the panel opens: always open it maximized, never open it maximized, or reopen it in the state it had when it was closed (the default). It is ignored when the panel is top or bottom with a non-center alignment, because maximize is not supported there.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.panel.opensMaximized
- Cursor: present (setting found with the same three values and default preserve; the stored flag panel.wasLastMaximized is also there)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (585-595); layout.ts (2112-2125, 2242-2251, 2833); C bundle (setting registration with enum always, never, preserve and enumDescriptions); NLS 4216-4219

### PN-1.7 Hide the editor area so the panel fills the window
- Tag: both
- Evidence: source
- Boundary: yes
- What: Run Toggle Editor Area Visibility from the Command Palette to hide the editor area and bring it back. In stock this runs the same maximize routine as PN-1.5, so the panel fills the window; it is disabled for a top or bottom panel with a non-center alignment. In Cursor it hides the editor area through the part toggle instead, makes sure the agent chat is open first, and when it brings the editor back it hides a panel that had been forced open only by the hide. Boundary reason: in Cursor the command opens or creates the agent chat (`ensureChatVisibleOrCreate`).
- Commands: workbench.action.toggleEditorVisibility
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (id, label Toggle Editor Area Visibility and the same precondition are present; the run body calls ensureChatVisibleOrCreate and setPartHidden on the editor part rather than toggleMaximizedPanel)
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (251-274); layout.ts (1886-1893, 2216-2240); C bundle (action with id toggleEditorVisibility and context key mainEditorAreaVisible); NLS 3167

### PN-1.8 Resize the panel
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag the line between the editor and the panel to change its height (top or bottom) or width (left or right). It will not shrink below 77 px high or 300 px wide, dragging past that snaps it shut, and it can be dragged open from closed only when it has view containers. Double-click the line to jump to the preferred size, which is 40 percent of the window height for a top or bottom panel, or the active view's best width (at least 300) for a left or right panel. The size is remembered, and the last non-maximized height and width are kept separately for each direction. With focus in the panel, Increase and Decrease Current View Size change it by 60 px.
- Commands: workbench.action.increaseViewSize; workbench.action.decreaseViewSize
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (class has minimumWidth 300, minimumHeight 77, preferredHeight 0.4 of window height, preferredWidth with 300 floor, and a snap getter; the grid sash reset handler and the stored keys panel.size, panel.lastNonMaximizedHeight and panel.lastNonMaximizedWidth were found; both resize ids were found)
- Source: V\src\vs\workbench\browser\parts\panel\panelPart.ts (40-65); parts\paneCompositePart.ts (112-115); layout.ts (1799-1830, 2828-2833); actions\layoutActions.ts (1138-1277); V\src\vs\base\browser\ui\grid\grid.ts (710-741); C bundle (panel part class with minimumHeight 77, onDidSashReset in the grid class, state keys). The resize commands are also listed in SB-1.4.

### PN-1.9 Set the default panel location for new workspaces
- Tag: both
- Evidence: source
- Boundary: no
- What: A setting picks where the panel first appears in a workspace that has no saved layout: bottom (the default), top, left or right. A workspace that already has a saved position keeps it. The Restore Defaults button in Customize Layout resets this setting (TB-4.8).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.panel.defaultLocation
- Cursor: present (setting found with the same four values and default bottom; the stored key panel.position is there too)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (578-584); layout.ts (2847, 3014-3015, 2869); C bundle (setting registration with enum left, bottom, top, right); NLS 4215

### PN-1.10 Switch panel views from the tab bar
- Tag: both
- Evidence: source
- Boundary: no
- What: The panel title row holds one tab for each view container in the panel (Problems, Output, Debug Console, Terminal and so on, in a fixed order). Click a tab to open that container, and the active tab gets an underline and brighter text. A second click within 300 ms is ignored. Tabs show a count badge when the container reports one, and hovering shows the name with its shortcut. The strip is a tab list, so Left, Right, Home and End move between tabs and Enter or Space opens one. Tabs for containers that hide when empty (Terminal, Test Results, Ports, Comments) appear only while the container is active or has views.
- Commands: none found
- Menus: none found
- Keys: Left Arrow; Right Arrow; Home; End; Enter; Space (with a tab focused)
- Settings: none found
- Cursor: present (searched for the label Active View Switcher, the tablist role on the action bar, the same pinned and placeholder storage keys, and the tab action classes; all found)
- Source: V\src\vs\workbench\browser\parts\paneCompositePart.ts (381-441, 723-731); parts\compositeBar.ts (283-322); parts\paneCompositeBar.ts (419-461, 765-835); parts\compositeBarActions.ts (155-440); parts\panel\panelPart.ts (135-165, 213-220); parts\panel\media\panelpart.css (60-76); C bundle (action bar with ariaRole tablist, label NLS 3321, keys workbench.panel.pinnedPanels and workbench.panel.placeholderPanels)

### PN-1.11 Show tabs as labels or icons
- Tag: both
- Evidence: source
- Boundary: no
- What: The panel tabs show text labels by default. Switch to compact 16 px icons from the tab bar right-click menu (it says Show Icons when labels are on, and Show Labels when icons are on) or from the setting. The menu row is a menu-only action with id `workbench.action.panel.toggleShowLabels`, not a registered command.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.panel.showLabels
- Cursor: present (setting found with default true, labels Show Icons and Show Labels found, and the menu action id found)
- Source: V\src\vs\workbench\browser\parts\panel\panelPart.ts (111-119, 139-165, 180-193); workbench.contribution.ts (573-577); C bundle (setting registration, toggleShowLabels action in fillExtraContextMenuActions); NLS 3970-3971, 4214

### PN-1.12 Reach hidden tabs from the overflow menu
- Tag: both
- Evidence: source
- Boundary: no
- What: When the tabs do not fit the row width (after the toolbar), the extra ones collapse behind a "..." button called Additional Views. Click it for a list of the hidden tabs with the active one marked, and a count after the name when the tab has a badge. The active tab always stays visible even if it is not kept.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (action id additionalComposites.action and the label Additional Views found; the chat variant of this button reads More Chats and belongs to the chat surface)
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (528-651); parts\compositeBarActions.ts (449-512); parts\panel\panelPart.ts (150-152); C bundle (overflow action class, NLS 3323)

### PN-1.13 Tab bar right-click menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click the panel title row (or empty space in the tab strip) for one menu with a ticked row for each tab. Unticking a tab hides it from the strip, and the last kept tab cannot be unticked. Below the list are a Views submenu (shown when the active container has more than one view action), Panel Position and Align Panel submenus with the current choice ticked, Show Icons or Show Labels, and Hide Panel.
- Commands: none found
- Menus: PanelPositionMenu; PanelAlignmentMenu; Views (ViewsSubMenu)
- Keys: none found
- Settings: workbench.panel.showLabels
- Cursor: present (the pinned-tab list code, the Views submenu hook, Pe.PanelPositionMenu and Pe.PanelAlignmentMenu reads, the toggleShowLabels action and the Hide Panel row were found in the panel part class; the stored list key is workbench.panel.pinnedPanels)
- Source: V\src\vs\workbench\browser\parts\panel\panelPart.ts (167-194); parts\compositeBar.ts (654-684); parts\paneCompositePart.ts (332-344, 758-803); C bundle (fillExtraContextMenuActions with submenu ids workbench.action.panel.position and workbench.action.panel.align; getContextMenuActions with checked and enabled rows); NLS 3928, 3970-3974

### PN-1.14 Single tab right-click menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click one panel tab for its own menu: Configure Keybinding (when the tab has a shortcut command), Hide '<name>' or Keep '<name>' (Hide is disabled for the last kept tab), Hide Badge or Show Badge, Move To with Primary Side Bar and Secondary Side Bar (the panel is left out because the tab is already there), and Reset Location when the container was moved from its home. The tab bar rows from PN-1.13 follow after a separator. Move To sends the whole container to the end of that bar and opens it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (action ids show.toggleCompositePinned and show.toggleCompositeBadge, submenu id moveToMenu, resetLocationAction, and the labels Hide '{0}', Keep '{0}', Hide Badge, Show Badge, Move To, Reset Location and Configure Keybinding were found; Cursor adds extra rows only for chat tabs, which are not panel tabs)
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts (515-686, 722-757); parts\paneCompositeBar.ts (157-214); C bundle (tab item class showContextMenu, getContextMenuActionsForComposite with moveToMenu); NLS 3920-3925, 3325-3330, 3977-3980

### PN-1.15 Go to the previous or next panel view
- Tag: both
- Evidence: source
- Boundary: no
- What: Two Command Palette commands switch to the tab before or after the active one and wrap around at the ends. They move keyboard focus into the opened view. They have no default shortcut.
- Commands: workbench.action.previousPanelView; workbench.action.nextPanelView
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (both ids and labels Previous Panel View and Next Panel View found, built on the same shared switch action)
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (252-273); parts\compositeBarActions.ts (760-791); C bundle (two actions extending the switch action with location 1 and offsets -1 and 1); NLS 3963-3964

### PN-1.16 Focus into Panel
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Focus into Panel from the Command Palette to jump keyboard focus into the active view of the panel. If the panel is hidden it opens first. It has no default shortcut.
- Commands: workbench.action.focusPanel
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (id, label Focus into Panel and behavior present; Cursor adds a precondition so it is disabled while an auxiliary window has focus)
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (99-125); layout.ts (1287-1290); C bundle (action with precondition FT.toNegated and body that shows the panel then focuses the active composite); NLS 3937, 3954

### PN-1.17 Panel title toolbar
- Tag: both
- Evidence: source
- Boundary: no
- What: The right side of the panel title row shows the active view's own buttons (for example the terminal's New, Split and Kill), a "..." menu for the extras, then Maximize or Restore and Hide Panel at the far end. In stock a separator sits between the view's buttons and the two panel buttons, and the Maximize button looks pressed while the panel is maximized. The Maximize and terminal Split icons turn to match the panel when it is on the left, right or top.
- Commands: workbench.action.toggleMaximizedPanel; workbench.action.togglePanel
- Menus: PanelTitle (Maximize at order 1, Hide Panel at order 2); ViewTitle (only as the source of each view's own buttons)
- Keys: none found
- Settings: none found
- Cursor: changed (PanelTitle holds the same two rows; the Hide icon is codicon xTwo where stock uses close; the Maximize icon is chevronUp and Restore is chevronDown; the global toolbar is created without the highlightToggledItems option; the view toolbar has no trailing separator option, and its overflow tooltip reads "Close, Export, Settings and More..." where stock reads "Views and More Actions..."; the icon rotation rules for right, left and top panels are the same)
- Source: V\src\vs\workbench\browser\parts\compositePart.ts (405-433); parts\paneCompositePart.ts (332-364); V\src\vs\base\browser\ui\toolbar\toolbar.ts (266-268); parts\panel\panelActions.ts (24-28, 74-81, 275-318); parts\panel\media\panelpart.css (77-100); C bundle (title toolbar created with toggleMenuTitle NLS 3332 and no trailingSeparator anywhere in the bundle, global toolbar with hiddenItemStrategy -1 and no highlightToggledItems; C css rotation rules for basepanel right, left and top); NLS 3332, 3927, 3929-3931

### PN-1.18 Drag tabs and views onto the panel
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag a tab left or right along the panel tab bar to reorder it, and a line shows where it will land. Hovering a tab for 800 ms while dragging switches to it. Drag a container tab from the side bar rail or the secondary side bar onto the panel tab bar, or drag one view header onto it, to move it into the panel (a lone view gets its own new tab). Dropping on the empty panel body also works, and an empty panel reads "Drag a view here to display." Only views that allow moving can be dragged. The drop line uses the colour panel.dropBorder.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (CompositeBarDndCallbacks with the dragged-over-head and dragged-over-tail classes, the empty pane message area and its text NLS 3926, and the drop handler calling moveViewContainerToLocation and moveViewToLocation with "dnd" were found; the title label is made draggable unconditionally where stock ties it to canMoveViews)
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (36-138, 163-232); parts\compositeBarActions.ts (551-594); parts\paneCompositePart.ts (252-330, 368-379); V\src\vs\workbench\browser\dnd.ts; C bundle (same classes and handlers); NLS 3926

### PN-1.19 Move all secondary side bar views into the panel
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Move Secondary Side Bar Views To Panel from the Command Palette to send every view container from the secondary side bar to the panel as new tabs, show the panel, and open the container that was active in the side bar when the panel had none. A hidden twin id, `workbench.action.moveSidePanelToPanel`, does the same.
- Commands: workbench.action.moveSecondarySideBarToPanel; workbench.action.moveSidePanelToPanel
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (both ids and the label Move Secondary Side Bar Views To Panel found; in Cursor the secondary side bar holds the agent chat container, see Unclear)
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (344-364, 399-425); C bundle (shared move class lEr with ids moveSidePanelToPanel and moveSecondarySideBarToPanel); NLS 3968-3969

### PN-1.20 The panel exists only in the main window
- Tag: both
- Evidence: source
- Boundary: no
- What: A popped-out editor window has an editor, a status bar and a title bar and never a panel. In stock its layout controls leave out the panel toggle. Panel commands still act on the main window's panel when run with the popped-out window focused, except that Increase and Decrease Current View Size and Toggle Editor Area Visibility are switched off there. Cursor switches off Ctrl+J, Toggle Panel Visibility, Hide Panel and Focus into Panel in that case as well, and it drops the popped-out-window condition from the panel toggle's layout menu row.
- Commands: workbench.action.togglePanel; workbench.action.closePanel; workbench.action.focusPanel; workbench.action.toggleEditorVisibility; workbench.action.increaseViewSize; workbench.action.decreaseViewSize
- Menus: LayoutControlMenu
- Keys: Ctrl+J (Cmd+J on macOS)
- Settings: none found
- Cursor: changed (the three panel commands and the Ctrl+J key carry `!isAuxiliaryWindowFocusedContext` in Cursor and not in stock; the other three already had it in stock; the panel row in LayoutControlMenu has no isAuxiliaryWindow condition in Cursor, and whether a popped-out window in Cursor draws that menu was not checked)
- Source: V\src\vs\workbench\browser\layout.ts (1313-1336, 1342-1370); V\src\vs\workbench\services\layout\browser\layoutService.ts (293-300); parts\panel\panelActions.ts (31-125, 320-342); actions\layoutActions.ts (251-274, 1140-1260); C bundle (context key isAuxiliaryWindowFocusedContext, class FT, in the preconditions of hva, closePanel and focusPanel)

### PN-1.21 Panel theme colors
- Tag: both
- Evidence: source
- Boundary: no
- What: A theme or `workbench.colorCustomizations` can restyle the panel frame: panel.background, panel.border, panelTitle.border, panelTitle.activeForeground, panelTitle.inactiveForeground, panelTitle.activeBorder, panelTitleBadge.background, panelTitleBadge.foreground and panel.dropBorder (the tab drag line). panelTitle.border is the line under the title row and panelTitle.activeBorder is the underline of the active tab. Input, section and sticky scroll colors that share the panel prefix are listed under Handed off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (all nine color ids found in the bundle)
- Source: V\src\vs\workbench\common\theme.ts (480-528); parts\panel\panelPart.ts (121-134, 155-164); C bundle (registerColor calls for the nine ids)

### PN-1.22 Float the panel as a rounded card (experimental Modern UI)
- Tag: vscode
- Evidence: source
- Boundary: no
- What: An experimental setting, off by default, draws the side bars and the bottom panel as floating cards with rounded corners, 4 px gaps and a refreshed style; the panel is inset by the card margin and border, and the outermost cards get a doubled gutter against the window edge.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.experimental.modernUI
- Cursor: missing (searched the bundle for modernUI, floating-panels and experimental.floatingPanels, zero hits)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (810-815); layout.ts (621-632, 1905); parts\paneCompositePart.ts (596-722); V\src\vs\workbench\services\layout\browser\layoutService.ts (53-63, 380-384); V\src\vs\workbench\browser\media\floatingPanels.css

### PN-1.23 No-title-bar layout adjustments for the panel
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor has a layout with no title bar (body class `no-titlebar-layout`, context key `cursor.noTitlebarLayout.enabled`). In it the panel fills the full height, its title area gives up the slot where layout buttons would go (those buttons can sit in the editor tab strip instead, TB-4.10), the panel's right toolbar is marked as not draggable so clicks reach it, and on macOS the panel gets 35 px of top padding when it is left, right, top or maximized so it clears the window buttons (with an exception when a right side bar is beside it). The panel's own layout button container is also hidden in every layout.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (panel part class overrides createTitleArea and getLayoutActionsContainer using isNoTitlebarLayoutActive, function klt reading context key `cursor.noTitlebarLayout.enabled` or the body class); C css (rules for `body.no-titlebar-layout .monaco-workbench .part.panel`, the 35px mac padding, `.part.panel>.title .global-actions` no-drag, and `.part.panel>.title>.layout-actions-container{display:none!important}`)

## Handed off
- Title bar panel toggle button and the Configure Layout drop-down panel row: TB-4.4 and TB-4.6.
- Customize Layout picker (its Panel visibility row, Restore Defaults): TB-4.7 and TB-4.8. The four Panel Alignment rows of the picker are listed as an entry point of PN-1.4.
- View > Appearance menu as a whole: TB-1. Its Panel, Panel Position and Align Panel rows are entry points of PN-1.1, PN-1.3 and PN-1.4.
- Increase and Decrease Current View Size commands: SB-1.4. They are also in the Commands list of PN-1.8 because they resize the panel when it has focus.
- Move View, Move Focused View, Reset View Locations, Reset Focused View Location and the Ctrl+K move chords: SB-1.14 to SB-1.17.
- Dragging a view header over a container body to split it, and dragging a container title: SB-1.12 and SB-1.13.
- Dragging a container tab from the panel onto the activity bar rail, and the rail's Move To menu: AB-2.5 to AB-2.7.
- Move Panel Views To Secondary Side Bar (`workbench.action.movePanelToSecondarySideBar` and the hidden twin `workbench.action.movePanelToSidePanel`): AX-5.
- Zen Mode hiding and restoring the panel: ED-1.34.
- View headers, each view's own toolbar and "..." menu, collapse and expand, and stacked views inside a panel tab: SB-1.
- Panel section header and section border colors (`panelSectionHeader.*`, `panelSection.*`): SB-1. Sticky scroll colors (`panelStickyScroll.*`): PN-3. Input border (`panelInput.border`, the filter boxes in panel views): PN-4.
- Tab count badges and each view's own show and focus commands (Toggle Terminal, Problems, Output, Debug Console and so on): PN-2, PN-4, PN-5 and PN-6.
- The first-open layout Cursor applies to a new workspace (including whether the panel starts shown): TB-4.17.

## Unclear
- Toggle Editor Area Visibility: ED-1 handed it to TB-4 and TB-4 did not list it, so PN-1.7 takes it because the stock code is the panel maximize routine. The parent should confirm the owner.
- PN-1.23: the panel part overrides `createTitleArea` and `getLayoutActionsContainer` for the no-title-bar layout, but no code was found that creates a layout actions container in the panel, and the CSS hides that container in every layout. The visible effect of the override is not confirmed. What switches on the no-title-bar layout was not decoded, and the macOS padding rules were read from CSS only (this machine is Windows).
- PN-1.2: the chat-maximized branch was read from `setUnifiedMaximizeState` (it ensures the chat is open and hides the panel, editor and, unless skipped, the side bar). Only the first part of the restore branch was read. No run on screen.
- PN-1.19: in Cursor the secondary side bar holds the agent chat container (location 2). Whether the move command carries that container into the panel was not checked.
- Cursor's main container no longer gets the `panel-position-<side>` and `panel-alignment-<value>` classes that stock adds (stock `layout.ts` 1906-1907 and 2042-2043). Zero hits for both in the bundle and the CSS. What stock CSS or code used them was not traced.
- Cursor's layout class has unified side bar parts (`workbench.parts.unifiedsidebar`) and an `isUnifiedMode` check that stock lacks. Beyond the quoted fragments, how these change panel placement was not decoded.
- Focus Next Part and Focus Previous Part (F6 and Shift+F6), and Navigate Left, Right, Up and Down (`workbench.action.focusNextPart`, `focusPreviousPart`, `navigateLeft`, `navigateRight`, `navigateUp`, `navigateDown`) move focus into and out of the panel. All six ids are in the Cursor bundle. No catalog group owns them. A workbench-wide navigation owner is needed.
- The panel context keys `panelVisible`, `panelPosition`, `panelAlignment`, `panelMaximized`, `panelFocus` and `activePanel` (and `mainEditorAreaVisible`) exist in stock and in Cursor and drive `when` clauses. They are not listed as features.
- Where drag-reordered tab order is stored was not traced. The hide and keep list is stored under `workbench.panel.pinnedPanels`, with `workbench.panel.placeholderPanels` and `workbench.panel.viewContainersWorkspaceState` beside it, and the active tab under `workbench.panelpart.activepanelid`. All four keys are in the Cursor bundle.
- Newer than 1.128: N was compared for the panel commands and the three `workbench.panel.*` settings and nothing new was found for this group. N's `panelActions.ts` hides Left and Right alignment inside the Agents window, which is out of scope. The Agents window panel part in `V\src\vs\sessions\browser\parts\panelPart.ts` was not read.
- Nothing here was run on screen. Behavior comes from source and from strings in the Cursor bundle.
