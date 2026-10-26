Group: SB-1 Side bar frame and shared view behavior. Section: Side bar. Date: 2026-10-25. Pass: 2.
Features: 24.

Short names used in Source: V is the stock 1.128.0 tree, N is the 1.144.0 clone, C bundle is `workbench.desktop.main.js` in the Cursor install, and NLS is `out\nls.messages.json` next to it. Line numbers are from the files as read and can drift by a line or two. Cursor's default side bar is a Cursor-made horizontal one, so some stock title-area wiring is marked `changed` below. Nothing was run on screen.

### SB-1.1 Show or hide the primary side bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+B or use View > Appearance > Primary Side Bar to open or close the whole side bar. Hiding it also closes the active container, and showing it brings back the last container used.
- Commands: workbench.action.toggleSidebarVisibility; workbench.action.closeSidebar
- Menus: MenubarAppearanceMenu; LayoutControlMenuSubmenu; LayoutControlMenu; ViewContainerTitleContext
- Keys: Ctrl+B (Cmd+B on Mac)
- Settings: none found
- Cursor: changed (the command, Ctrl+B and Close Primary Side Bar are all present; Cursor adds a "not focused in an auxiliary window" condition to the command and to the key; the screen reader messages "Primary Side Bar hidden" and "Primary Side Bar shown" were not found in NLS; the layout toggle button sits in group 2_pane_toggles instead of navigation)
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (286-391); V\src\vs\workbench\browser\layout.ts (1911-1944); V\src\vs\workbench\browser\parts\sidebar\sidebarActions.ts (18-33); C bundle (toggleSidebarVisibility class, closeSidebar action)

### SB-1.2 Move the side bar to the left or right
- Tag: both
- Evidence: source
- Boundary: no
- What: Switch the primary side bar (and the activity bar with it) between the left and right edge. The secondary side bar goes to the opposite side.
- Commands: workbench.action.toggleSidebarPosition; workbench.action.moveSideBarRight; workbench.action.moveSideBarLeft
- Menus: ViewContainerTitleContext; MenubarAppearanceMenu
- Keys: none found
- Settings: workbench.sideBar.location
- Cursor: changed (all three ids and the setting are present, the setting has 39 hits; Cursor's toggle command also stores the choice under an agent-layout override key before it updates the setting; the horizontal tab row has its own right-click entry, see AB-6)
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (97-171, 186-249); V\src\vs\workbench\browser\workbench.contribution.ts (566-572); C bundle (toggleSidebarPosition class, NLS "Toggle Primary Side Bar Position", "Move Primary Side Bar Right", "Move Primary Side Bar Left")

### SB-1.3 Focus the side bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+0 to jump keyboard focus into the open side bar view. If the side bar is hidden it opens first.
- Commands: workbench.action.focusSideBar
- Menus: CommandPalette
- Keys: Ctrl+0 (Cmd+0 on Mac)
- Settings: none found
- Cursor: changed (id, label "Focus into Primary Side Bar" and Ctrl+0 are present; Cursor adds the "not focused in an auxiliary window" condition to the key and the command)
- Source: V\src\vs\workbench\browser\parts\sidebar\sidebarActions.ts (35-66); C bundle (focusSideBar class)

### SB-1.4 Resize the side bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag the edge to change the width, double-click the edge to jump to a best-fit width (at least 300 pixels), or drag it below the minimum to snap it shut. Increase and Decrease Current View Size change the width by 60 pixels while focus is in the side bar.
- Commands: workbench.action.increaseViewSize; workbench.action.decreaseViewSize; workbench.action.increaseViewWidth; workbench.action.decreaseViewWidth
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (the four commands are present with the same auxiliary window condition; the minimum width is 214 pixels in both Cursor side bar classes against 170 in stock, and the horizontal one has a minimum height of 77)
- Source: V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts (44-65); V\src\vs\workbench\browser\actions\layoutActions.ts (1138-1260); V\src\vs\workbench\browser\layout.ts (1799-1813); V\src\vs\base\browser\ui\grid\grid.ts (710-735); C bundle (side bar classes with `minimumWidth=214`, increaseViewSize action)

### SB-1.5 Container title row and its "..." menu
- Tag: both
- Evidence: source
- Boundary: no
- What: The strip above the views shows the container name, or "Container: View" when it holds a single view, with its own buttons on the right. Its "..." menu starts with a Views submenu and lists the container's other actions.
- Commands: none found
- Menus: SidebarTitle; ViewContainerTitle; Views
- Keys: none found
- Settings: none found
- Cursor: changed (the Views submenu and the ViewContainerTitle menu are registered; in the default horizontal side bar the title label is replaced by the tab row, and the composite's primary and secondary actions are pushed to a Cursor-made toolbar; the stock title row remains in the vertical side bar)
- Source: V\src\vs\workbench\browser\parts\paneCompositePart.ts (332-366, 793-800); V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (47-51, 544-562); V\src\vs\workbench\browser\parts\views\viewMenuActions.ts (90); V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts (97-99); C bundle (Views submenu registration, composite toolbar `setActions`)

### SB-1.6 Container right-click menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click the container title or the empty space under the views to show or hide its views, reset the container's location, move the side bar to the other side, hide it, or pick an activity bar position.
- Commands: <containerId>.resetViewContainerLocation; workbench.action.toggleSidebarPosition; workbench.action.toggleSidebarVisibility
- Menus: ViewContainerTitleContext
- Keys: none found
- Settings: none found
- Cursor: changed (ViewContainerTitleContext, "Hide Primary Side Bar", "Move Primary Side Bar Right" and "Reset Location" are present; the Activity Bar Position submenu (ActivityBarPositionMenu) was not found; the horizontal tab row has its own right-click menu, see AB-6)
- Source: V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (846-860, 899-923); V\src\vs\workbench\browser\actions\layoutActions.ts (186-229, 340-350); V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts (592-601); V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (395-397, 564-579); V\src\vs\workbench\browser\parts\paneCompositePart.ts (758-791); C bundle (ViewContainerTitleContext items, NLS "Hide Primary Side Bar")

### SB-1.7 Collapse and expand a view
- Tag: both
- Evidence: source
- Boundary: no
- What: Click a view header, or press Enter, Space, Left or Right on it, to fold or unfold the view. Up and Down move between headers, and each view's folded state is remembered.
- Commands: none found
- Menus: none found
- Keys: Enter; Space; Left Arrow; Right Arrow; Up Arrow; Down Arrow (with a view header focused)
- Settings: none found
- Cursor: present (searched the bundle for `setExpanded` and the header class; the Cursor ViewPane adds one rule, see SB-1.21)
- Source: V\src\vs\base\browser\ui\splitview\paneview.ts (296-320, 574-580); V\src\vs\workbench\browser\parts\views\viewPane.ts (437-444, 503-512); C bundle (ViewPane `setExpanded`)

### SB-1.8 View header buttons and "..." menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Each view header shows its own buttons when you hover or focus it, plus a "..." menu for the extra actions. A setting keeps the buttons visible all the time.
- Commands: none found
- Menus: ViewTitle
- Keys: none found
- Settings: workbench.view.alwaysShowHeaderActions
- Cursor: present (ViewTitle has 110 uses in the bundle; the setting id has 3 hits and its description string is in NLS; the `actions-always-visible` class is there)
- Source: V\src\vs\workbench\browser\parts\views\viewPane.ts (59-68, 455-496, 698-704); V\src\vs\workbench\browser\parts\views\viewMenuActions.ts; V\src\vs\workbench\browser\parts\views\treeView.ts (575-612); V\src\vs\workbench\browser\workbench.contribution.ts (686-690); C bundle (ViewPane header toolbar)

### SB-1.9 Show or hide views in a container
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn single views on or off from the Views submenu of the "..." menu, the container right-click menu, or a view header's right-click menu (which also has Hide '<view>'). The last visible view cannot be hidden.
- Commands: <viewId>.toggleVisibility; <viewId>.removeView
- Menus: Views; ViewContainerTitleContext; ViewTitleContext
- Keys: none found
- Settings: none found
- Cursor: present (searched for `.toggleVisibility`, `.removeView`, the NLS label "Hide '{0}'" and ViewTitleContext, all found)
- Source: V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (831-897); V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (47-51, 751-761, 842-848); C bundle (toggleVisibility and removeView registrations)

### SB-1.10 Resize views in the stack
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag the line between two view headers to give one view more height. Double-click the line to split the space of the two nearest open views evenly. The sizes are remembered.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (searched for `onDidSashReset` and `saveViewSizes`, both found)
- Source: V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (394, 515, 701-744, 1130-1168); V\src\vs\base\browser\ui\splitview\splitview.ts (1180-1198); C bundle (ViewPaneContainer sash reset handler)

### SB-1.11 Reorder views by dragging a header
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag a view header onto another view in the same container. A highlight shows whether it will land above or below, and the order is saved.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (searched for `movePane(` and the view drag registration, both found)
- Source: V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (71-182, 886-1015); C bundle (`movePane(`, view drag registration)

### SB-1.12 Drag a view to another container, the panel or the secondary side bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag a view header over another container, the panel or the secondary side bar. A drop overlay appears, and dropping splits that area above, below, left or right of the existing views. Dropping a whole container's title there merges all its views in.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.location
- Cursor: present (searched for the `monaco-pane-drop-overlay` id and the drop target registration, both found)
- Source: V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (71-182, 402-513, 886-1015); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (350-352, 381); C bundle (drop overlay class)

### SB-1.13 Drag a whole container by its title
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag the container title (for example EXPLORER) onto the panel, the secondary side bar or another container to move all its views together.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (the title label drag is registered once in the stock pane part class used by the vertical side bar; the default horizontal side bar has no title label, and its tabs have their own reorder drag, see AB-2 and AB-6)
- Source: V\src\vs\workbench\browser\parts\paneCompositePart.ts (368-380); C bundle (title label drag registration)

### SB-1.14 Move a view with the picker
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Move View to pick any movable view, then pick a destination: New Side Bar Entry, New Panel Entry, New Secondary Side Bar Entry, or an existing container. Move Focused View skips the first step.
- Commands: workbench.action.moveView; workbench.action.moveFocusedView
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (both ids and the strings "Select a View to Move", "Select a Destination for the View", "New Panel Entry", "New Side Bar Entry", "New Secondary Side Bar Entry" are in the bundle)
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (791-1094); C bundle (moveView and moveFocusedView actions)

### SB-1.15 Move the focused view with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: With a view focused, press Ctrl+K then Up or Left to move it one place earlier in its container, or Down or Right to move it one place later.
- Commands: views.moveViewUp; views.moveViewDown; views.moveViewLeft; views.moveViewRight
- Menus: none found
- Keys: Ctrl+K Up Arrow; Ctrl+K Down Arrow; Ctrl+K Left Arrow; Ctrl+K Right Arrow (when a view has focus; Cmd+K on Mac)
- Settings: none found
- Cursor: present (the four ids and their key chords are in the bundle with the same weight, with a Mac variant)
- Source: V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (1197-1288); C bundle (views.moveViewUp registration)

### SB-1.16 Reset all view locations
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Reset View Locations to put every moved view and container back where it started.
- Commands: workbench.action.resetViewLocations
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (id and the string "Reset View Locations" found)
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (773-789); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (reset at 408); C bundle (resetViewLocations action)

### SB-1.17 Reset one view to its home
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a view header and choose Reset Location, or run Reset Focused View Location, to send that one view back to the container it started in.
- Commands: workbench.action.resetFocusedViewLocation; <viewId>.resetViewLocation
- Menus: ViewTitleContext; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (searched for `workbench.action.resetFocusedViewLocation`, `.resetViewLocation` and the strings "Reset Location" and "Reset Focused View Location", all found)
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (1096-1136); V\src\vs\workbench\services\views\browser\viewsService.ts (604-638); C bundle (resetFocusedViewLocation action, per-view reset registration)

### SB-1.18 Welcome content in an empty view
- Tag: both
- Evidence: source
- Boundary: no
- What: A view with nothing to show displays text, links and buttons instead of a blank list, for example the Explorer with no folder open. Lines appear and disappear as conditions change, a button can be greyed out by a precondition, and the layout widens above 640 pixels.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (searched for the `welcome-view` classes and the `viewsWelcome` contribution point, both found)
- Source: V\src\vs\workbench\browser\parts\views\viewPane.ts (101-309); V\src\vs\workbench\common\views.ts (395-435, 515-540); V\src\vs\workbench\contrib\welcomeViews\common\viewsWelcomeContribution.ts; V\src\vs\workbench\contrib\welcomeViews\common\viewsWelcomeExtensionPoint.ts; C bundle (welcome view controller)

### SB-1.19 Focus a view by command
- Tag: both
- Evidence: source
- Boundary: no
- What: Every view gets a "Focus on <name> View" command in the Command Palette that opens its container and puts focus inside, for example `outline.focus` and `timeline.focus`. A view can also declare its own shortcut for it.
- Commands: <viewId>.focus
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (the string "Focus on {0} View" is in NLS, and `outline.focus` and `timeline.focus` were found in the bundle)
- Source: V\src\vs\workbench\services\views\browser\viewsService.ts (557-602); C bundle (FocusViewAction registration)

### SB-1.20 A one-view container shows one combined title
- Tag: both
- Evidence: source
- Boundary: no
- What: When a container shows only one view, such as the Explorer with just Folders, the view header is hidden and the container title carries the view's buttons, reading "Container: View" when the names differ.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (the merge logic is present, with 14 hits for `isViewMergedWithContainer`; Cursor adds an "alone" state for a single view that is not merged, see SB-1.21; the merged title text has no place in the horizontal side bar because the tab row replaces the title label)
- Source: V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (544-562, 1080-1122); V\src\vs\workbench\browser\parts\views\viewPane.ts (410-417); C bundle (ViewPaneContainer `updateViewHeaders`)

### SB-1.21 A lone view cannot be folded
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: In Cursor a view that is alone in its container is forced open. In the default horizontal side bar its collapse arrow is also hidden, while in the vertical side bar the arrow stays but does nothing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.orientation
- Cursor: cursor only
- Source: C bundle (ViewPane `setIsAlone`, `isAlone`, `updateTwistyIcon` with the orientation check; ViewPaneContainer `updateViewHeaders` calling `setIsAlone`)

### SB-1.22 Two-row title for a container with a toolbar
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: In the horizontal side bar a container can ask for an extra action row. Run and Debug does, so its title grows from 35 to 70 pixels to hold the Start button and the launch configuration picker.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.orientation
- Cursor: cursor only
- Source: C bundle (`wantsActionToolbar: true` on the Run and Debug container, `heightOfTitle` set to 35 or 70; 0 hits for `wantsActionToolbar` in V)

### SB-1.23 Move through and find inside a view's tree or list
- Tag: both
- Evidence: source
- Boundary: no
- What: In any view tree or list, arrow keys, Home, End, Page Up and Page Down move focus, Right and Left unfold and fold, Enter opens, and Space toggles. Ctrl+Alt+F opens a find box that either highlights or filters rows, and typing letters jumps to a match.
- Commands: list.focusDown; list.focusUp; list.expand; list.collapse; list.collapseAll; list.select; list.toggleExpand; list.find; list.closeFind; list.toggleFindMode; list.toggleFindMatchType; list.triggerTypeNavigation; list.collapseAllToFocus; list.focusAnyDown; list.focusAnyFirst; list.focusAnyLast; list.focusAnyUp; list.focusFirst; list.focusLast; list.focusPageDown; list.focusPageUp; list.focusParent; list.selectAndPreserveFocus
- Menus: none found
- Keys: Up Arrow; Down Arrow; Left Arrow; Right Arrow; Enter; Space; Home; End; Ctrl+Alt+F; F3; Esc (all with a list focused)
- Settings: workbench.list.defaultFindMode; workbench.list.typeNavigationMode; workbench.list.defaultFindMatchType; workbench.list.openMode; workbench.list.multiSelectModifier; workbench.list.horizontalScrolling
- Cursor: present (searched for `list.find`, `list.toggleFindMode`, `list.focusDown`, `list.collapseAll` and the six settings, all found)
- Source: V\src\vs\workbench\browser\actions\listCommands.ts (76-900); V\src\vs\platform\list\browser\listService.ts (170-186); C bundle (list command registrations)

### SB-1.24 Sticky scroll and indent look of view trees
- Tag: both
- Evidence: source
- Boundary: no
- What: Trees such as the Explorer pin the parent rows at the top while you scroll. Toggle it from the Command Palette or the setting, choose how many rows stick, and change indent size, indent guides, and whether a single or double click expands.
- Commands: tree.toggleStickyScroll; list.stickyScroll.collapse; list.stickyScrollselect; list.stickyScrolltoggleExpand
- Menus: CommandPalette
- Keys: Left (list.stickyScroll.collapse, pinned row focused; macOS also Cmd+Up); Enter (list.stickyScrollselect; macOS also Cmd+Down); Space (list.stickyScrolltoggleExpand)
- Settings: workbench.tree.enableStickyScroll; workbench.tree.stickyScrollMaxItemCount; workbench.tree.indent; workbench.tree.renderIndentGuides; workbench.tree.expandMode
- Cursor: present (searched for `tree.toggleStickyScroll`, the five settings and the colour `sideBarStickyScroll.background`, all found)
- Source: V\src\vs\workbench\browser\actions\listCommands.ts (967-986); V\src\vs\platform\list\browser\listService.ts (179-186); V\src\vs\workbench\common\theme.ts (650-654); C bundle (ToggleStickyScroll action); V\src\vs\workbench\browser\actions\listCommands.ts; V\src\vs\platform\list\browser\listService.ts (defaults true and 7); V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (getContext reads the pinned focus); C bundle (tree.toggleStickyScroll, label Toggle Tree Sticky Scroll, list.stickyScroll.collapse, list.stickyScrollselect, list.stickyScrolltoggleExpand, same defaults)

## Handed off
- Previous and Next Primary Side Bar View (`workbench.action.previousSideBarView`, `nextSideBarView`) and clicking the open icon to hide or focus the side bar: AB-1.
- Per-container Move To, drag onto the rail and rail-side Reset Location: AB-2 (SB-1.6 only shows the title right-click entry for Reset Location).
- Activity Bar Position submenu on the container right-click menu (MenuId `ActivityBarPositionMenu`, absent in Cursor): AB-5.
- Cursor's horizontal tab row, its right-click Hide and Keep, overflow list and orientation setting: AB-6.
- Side bar toggle button, Configure Layout menu and title bar right-click Move entries: TB-4.
- Open View picker (`workbench.action.openView`, Ctrl+Q): QA-4.
- Header filter box in a view pane (`viewpane.action.filter`, used by Problems, Output, Comments and Debug Console, none in the side bar): PN-4 (also PN-5 and PN-6).

## Unclear
- Right-click on the title and drag of the title label in Cursor's horizontal side bar: the stock code for both lives in the pane part class used by the vertical side bar, and the Cursor-made component showed no matching wiring. Read from strings only.
- Whether the container "..." menu shows in the horizontal side bar: the component sends the composite's actions and secondary actions to its own toolbar, but nothing was run to see the "..." button.
- SB-1.4 resize commands act on whichever part has focus (side bar, panel, editor, secondary side bar), so PN-1, AX-2 and the editor groups may list the same ids.
- SB-1.23 and SB-1.24 are workbench-wide list and tree behavior, not side bar only. No other catalog group looked like an owner, so they sit here. The parent can move them.
- SB-1.22 overlaps AB-6 because it belongs to Cursor's horizontal side bar frame.
- Stock labels differ from pass 1 in one place: the focus command reads "Focus into Primary Side Bar", not "Focus Side Bar".
- Moving views is switched off in the Agents window (`canMoveViews` returns false there), which is out of scope.
- Holding Alt while dragging a view sash changes the drag mode (`resetSashDragState` with `altKey` in `splitview.ts`). Not read in detail.
- Theme colors for the frame (`sideBar.background`, `sideBarTitle.foreground`, `sideBarSectionHeader.*`, `sideBar.dropBackground`) exist in both and are not listed as features. `sideBar.dropBackground`, `sideBarSectionHeader.background` and `sideBar.background` were found in the Cursor bundle.
