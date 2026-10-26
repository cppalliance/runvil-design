Group: PN-1 Panel frame and layout. Pass: 3. Date: 2026-10-25.

### PN-1.1
1. Choosing View > Appearance > Panel shows the panel when it is hidden and hides it when it is shown (`workbench.action.togglePanel`).
2. Clicking the Hide Panel button at the right end of the panel title closes the panel (`workbench.action.closePanel`).
3. The Command Palette has Toggle Panel Visibility and Hide Panel, and the layout controls menu has a panel toggle row.
4. Opening any panel view (Problems, Output, Terminal, Debug Console and so on) also shows the panel.
5. Showing the panel brings back the last view used, or the first container that still has views.
6. Hiding the panel closes the active view and moves focus to the editor.
7. The panel starts hidden in a new workspace, and it hides itself when its last view container goes away.
8. In Cursor both commands are disabled while an auxiliary window has focus, and both layout toggle icons use the `panelFrame` icon where stock uses `layoutPanel` and `layoutPanelOff`. [cursor only]
9. In Cursor the title bar panel toggle sits in menu group `2_pane_toggles`. [cursor only]

### PN-1.2
1. Pressing Ctrl+J (Cmd+J on macOS) hides the panel when it is open.
2. Pressing Ctrl+J (Cmd+J on macOS) shows the panel when it is closed.
3. Pressing Ctrl+J while the panel is maximized first restores the editor, then hides the panel.
4. In Cursor, pressing Ctrl+J while the maximized layout that hides the editor is active first leaves that layout before it shows the panel. [cursor only]
5. In Cursor, pressing Ctrl+J while the panel is open in the maximized layout that hides the editor keeps that layout instead of hiding the panel. [cursor only]
6. In Cursor, Ctrl+J and the Toggle Panel Visibility command are switched off while an auxiliary window has focus. [cursor only]

### PN-1.3
1. A panel at the bottom sits under the editor (the default), at the top it sits above the editor, and at the left or right it sits beside the editor.
2. Four Command Palette commands, Move Panel To Top, Move Panel Left, Move Panel Right and Move Panel To Bottom, move the panel (`workbench.action.positionPanelTop`, `workbench.action.positionPanelLeft`, `workbench.action.positionPanelRight`, `workbench.action.positionPanelBottom`).
3. View > Appearance > Panel Position is a submenu with the four positions.
4. The Panel Position submenu in the panel tab bar right-click menu lists the four positions with the current one ticked.
5. Moving a hidden panel to a new position also shows it.
6. The panel position is remembered per workspace.
7. Alignment choices apply only when the panel is at the top or bottom.

### PN-1.4
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

### PN-1.5
1. Clicking the Maximize button at the right of the panel title makes the panel fill everything under the title bar by hiding the editor area.
2. While maximized the button shows Restore, and clicking it brings the editor back at its earlier size.
3. Running Toggle Maximized Panel from the Command Palette (`workbench.action.toggleMaximizedPanel`) maximizes and restores the same way.
4. Running Toggle Maximized Panel while the panel is hidden shows the panel and maximizes it.
5. Maximize works only when the panel is center aligned or sits left or right, and for a top or bottom panel with another alignment the command is disabled with the warning "Maximizing the panel is only supported when it is center aligned."
6. In Cursor the Maximize button uses the `chevronUp` icon and Restore swaps to a new panel-restore icon (`chevronDown`), where stock keeps one `screenFull` icon. [cursor only]
7. In Cursor the tooltips read "Maximize Panel Size" and "Restore Panel Size" where stock reads "Maximize Panel" and "Restore Panel". [cursor only]
8. In Cursor, maximizing a left or right panel in unified mode keeps the secondary side bar width. [cursor only]

### PN-1.6
1. Setting `workbench.panel.opensMaximized` chooses whether the panel opens maximized each time it opens, with values `always`, `never` and `preserve` (the default).
2. With `always` the panel opens maximized every time.
3. With `never` the panel opens not maximized every time.
4. With `preserve` the panel opens in the state it had when it was closed.
5. The setting is ignored when the panel is at the top or bottom with a non-center alignment.

### PN-1.7
1. Running Toggle Editor Area Visibility from the Command Palette (`workbench.action.toggleEditorVisibility`) hides the editor area, and running it again brings the editor area back.
2. The command is disabled for a top or bottom panel with a non-center alignment.
3. In stock the command runs the same maximize routine as PN-1.5, so the panel fills the window. [stock only]
4. In Cursor the command hides the editor area through the part toggle instead of maximizing the panel. [cursor only]
5. In Cursor the command makes sure the agent chat is open (or creates it) before hiding the editor area. [cursor only]
6. In Cursor, bringing the editor back also hides a panel that had been forced open only by the hide. [cursor only]

### PN-1.8
1. Dragging the line between the editor and the panel changes the panel height (top or bottom) or width (left or right).
2. The panel will not shrink below 77 px high or 300 px wide, and dragging past that snaps it shut.
3. A closed panel can be dragged open only when it has view containers.
4. Double-clicking the line jumps to the preferred size, which is 40 percent of the window height for a top or bottom panel, or the active view's best width (at least 300) for a left or right panel.
5. The size is remembered, and the last non-maximized height and width are kept separately for each direction.
6. With focus in the panel, Increase Current View Size and Decrease Current View Size change it by 60 px (`workbench.action.increaseViewSize`, `workbench.action.decreaseViewSize`).

### PN-1.9
1. Setting `workbench.panel.defaultLocation` picks where the panel first appears in a workspace with no saved layout, with values `bottom` (the default), `top`, `left` and `right`.
2. A workspace that already has a saved panel position keeps it.
3. The Restore Defaults button in Customize Layout resets this setting.

### PN-1.10
1. The panel title row holds one tab for each view container in the panel (Problems, Output, Debug Console, Terminal and so on) in a fixed order.
2. Clicking a tab opens that container, and the active tab gets an underline and brighter text.
3. A second click on a tab within 300 ms is ignored.
4. A tab shows a count badge when its container reports one.
5. Hovering a tab shows its name with its shortcut.
6. With a tab focused, Left Arrow, Right Arrow, Home and End move between tabs and Enter or Space opens one.
7. Tabs for containers that hide when empty (Terminal, Test Results, Ports, Comments) appear only while the container is active or has views.

### PN-1.11
1. The panel tabs show text labels by default.
2. Setting `workbench.panel.showLabels` (default true) chooses between text labels and compact 16 px icons.
3. The tab bar right-click menu row reads Show Icons when labels are on and Show Labels when icons are on, and choosing it switches between them.
4. The Show Icons and Show Labels row is a menu-only action (`workbench.action.panel.toggleShowLabels`) and is not in the Command Palette.

### PN-1.12
1. When the tabs do not fit the row width after the toolbar, the extra ones collapse behind a "..." button called Additional Views.
2. Clicking Additional Views lists the hidden tabs.
3. The list marks the active tab, and shows a count after the name when a tab has a badge.
4. The active tab always stays visible, even if it is not kept.

### PN-1.13
1. Right-clicking the panel title row, or empty space in the tab strip, opens one menu with a ticked row for each tab.
2. Unticking a tab hides it from the strip, and the last kept tab cannot be unticked.
3. A Views submenu shows below the list when the active container has more than one view action.
4. Panel Position and Align Panel submenus show below the list with the current choice ticked.
5. A Show Icons or Show Labels row switches the tab display.
6. A Hide Panel row closes the panel.

### PN-1.14
1. Right-clicking one panel tab opens its own menu.
2. Configure Keybinding shows when the tab has a shortcut command.
3. Hide '<name>' or Keep '<name>' shows, and Hide is disabled for the last kept tab.
4. Hide Badge or Show Badge shows for the tab.
5. A Move To submenu offers Primary Side Bar and Secondary Side Bar (the panel is left out because the tab is already there), and choosing one sends the whole container to the end of that bar and opens it.
6. Reset Location shows when the container was moved from its home.
7. The tab bar rows from PN-1.13 follow after a separator.

### PN-1.15
1. Previous Panel View (`workbench.action.previousPanelView`) switches to the tab before the active one and wraps around at the start.
2. Next Panel View (`workbench.action.nextPanelView`) switches to the tab after the active one and wraps around at the end.
3. Both commands move keyboard focus into the opened view.
4. Both commands are in the Command Palette and have no default shortcut.

### PN-1.16
1. Running Focus into Panel from the Command Palette (`workbench.action.focusPanel`) moves keyboard focus into the active view of the panel.
2. Running Focus into Panel while the panel is hidden opens the panel first.
3. Focus into Panel has no default shortcut.
4. In Cursor, Focus into Panel is disabled while an auxiliary window has focus. [cursor only]

### PN-1.17
1. The right side of the panel title row shows the active view's own buttons, for example the terminal's New, Split and Kill.
2. A "..." menu holds the extra buttons that do not fit.
3. Maximize or Restore and Hide Panel sit at the far end of the title row.
4. The Maximize and terminal Split icons turn to match the panel when it is on the left, right or top.
5. In stock a separator sits between the view's buttons and the two panel buttons. [stock only]
6. In stock the Maximize button looks pressed while the panel is maximized. [stock only]
7. In Cursor the Hide Panel button uses the `xTwo` icon where stock uses close, and Maximize uses `chevronUp` with Restore using `chevronDown`. [cursor only]
8. In Cursor the "..." overflow tooltip reads "Close, Export, Settings and More..." where stock reads "Views and More Actions...". [cursor only]

### PN-1.18
1. Dragging a tab left or right along the panel tab bar reorders it, and a line in the `panel.dropBorder` color shows where it will land.
2. Hovering a tab for 800 ms while dragging switches to that tab.
3. Dragging a container tab from the side bar rail or the secondary side bar onto the panel tab bar moves that container into the panel.
4. Dragging one view header onto the panel tab bar moves that view into the panel, and a lone view gets its own new tab.
5. Dropping on the empty panel body also moves the container or view into the panel.
6. An empty panel reads "Drag a view here to display."
7. Only views that allow moving can be dragged.

### PN-1.19
1. Running Move Secondary Side Bar Views To Panel from the Command Palette (`workbench.action.moveSecondarySideBarToPanel`) sends every view container from the secondary side bar to the panel as new tabs.
2. The command shows the panel.
3. When the panel had no active container, the container that was active in the secondary side bar opens.
4. A hidden twin command, `workbench.action.moveSidePanelToPanel`, does the same.

### PN-1.20
1. A popped-out editor window has an editor, a status bar and a title bar, and never a panel.
2. In stock the layout controls of a popped-out window leave out the panel toggle. [stock only]
3. Panel commands run with a popped-out window focused act on the main window's panel.
4. Increase Current View Size, Decrease Current View Size and Toggle Editor Area Visibility are switched off while a popped-out window has focus.
5. In Cursor, Ctrl+J, Toggle Panel Visibility, Hide Panel and Focus into Panel are also switched off while a popped-out window has focus. [cursor only]
6. In Cursor the panel row in the layout controls menu has no popped-out window condition. [cursor only]

### PN-1.21
1. `panel.background` sets the fill of the panel body.
2. `panel.border` sets the line that separates the panel from the editor.
3. `panelTitle.border` sets the line under the panel title row.
4. `panelTitle.activeForeground` and `panelTitle.inactiveForeground` set the text color of the active tab and the other tabs.
5. `panelTitle.activeBorder` sets the underline of the active tab.
6. `panelTitleBadge.background` and `panelTitleBadge.foreground` set the colors of the tab count badge.
7. `panel.dropBorder` sets the color of the tab drag line.

### PN-1.22
1. The experimental setting `workbench.experimental.modernUI` is off by default and turns on the floating card look. [stock only]
2. With it on, the side bars and the bottom panel are drawn as floating cards with rounded corners and 4 px gaps. [stock only]
3. With it on, the panel is inset by the card margin and border. [stock only]
4. With it on, the outermost cards get a doubled gutter against the window edge. [stock only]

### PN-1.23
1. In Cursor's no-title-bar layout (body class `no-titlebar-layout`, context key `cursor.noTitlebarLayout.enabled`) the panel fills the full height. [cursor only]
2. In that layout the panel title area gives up the slot where layout buttons would go, and those buttons can sit in the editor tab strip instead. [cursor only]
3. In that layout the panel's right toolbar is marked as not draggable so clicks reach its buttons. [cursor only]
4. On macOS in that layout the panel gets 35 px of top padding when it is left, right, top or maximized, except when a right side bar is beside it. [cursor only]
5. The panel's own layout button container is hidden in every layout. [cursor only]

## Unclear
- PN-1.1: the Hide Panel title row being declared inside the toggle command is a code detail, so it has no piece. Only the menu group `2_pane_toggles` is a piece.
- PN-1.3: the setting `workbench.panel.defaultLocation` is listed in this feature, but its piece is in PN-1.9 so the same behavior is not written twice.
- PN-1.5: the `panelmaximized` class that Cursor adds on the main container has no visible effect traced, so it has no piece.
- PN-1.7: the Cursor pieces say the command opens or creates the agent chat. That is the boundary reason, and the chat itself is out of scope. Piece 2 says the disabled precondition applies, based on the Cursor field saying the same precondition is present. Not run.
- PN-1.17: the Cursor `xTwo` and chevron icon names are from code, and the "Close, Export, Settings and More..." tooltip text was read from a string and not seen on screen.
- PN-1.18: the Cursor field notes that the title label is made draggable unconditionally where stock ties it to `canMoveViews`. It is not written as a piece because the Cursor field says present and it is unclear whether piece 7 (only views that allow moving can be dragged) still holds in Cursor. The parent should confirm.
- PN-1.19: whether the move carries the agent chat container into the panel in Cursor was not checked, so the pieces say nothing about it.
- PN-1.20: piece 2 is marked `[stock only]` because the Cursor field says the panel row has no popped-out window condition. Whether Cursor draws that menu in a popped-out window was not checked.
- PN-1.21: the one-line meanings of `panel.border` and the title colors come from the stock color descriptions in `V\src\vs\workbench\common\theme.ts` and the feature block. Not seen on screen.
- Features that Cursor changes carry `[cursor only]` on the pieces that state the Cursor behavior. Pieces that only name stock behavior in a `changed` feature carry `[stock only]`.
