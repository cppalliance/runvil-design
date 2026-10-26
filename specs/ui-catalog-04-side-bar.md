# 4. Side bar (SB)

The primary side pane. Its views can also be dragged into the panel or the auxiliary bar.

7 groups, 244 features. Each feature lists its entry points and the pieces that finish it.

## SB-1 Side bar frame and shared view behavior

(both). The pane itself and what every view in it shares: show and hide, side, headers, the "..." menu, welcome text, dragging a view elsewhere. Examples: Ctrl+B and left or right side; the view header menu; Move View and Reset View Locations. Cursor's tab row is AB-6.

24 features.

#### SB-1.1 Show or hide the primary side bar

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the command, Ctrl+B and Close Primary Side Bar are all present; Cursor adds a "not focused in an auxiliary window" condition to the command and to the key; the screen reader messages "Primary Side Bar hidden" and "Primary Side Bar shown" were not found in NLS; the layout toggle button sits in group 2_pane_toggles instead of navigation)
- What: Press Ctrl+B or use View > Appearance > Primary Side Bar to open or close the whole side bar. Hiding it also closes the active container, and showing it brings back the last container used.
- Commands: workbench.action.toggleSidebarVisibility; workbench.action.closeSidebar
- Menus: MenubarAppearanceMenu; LayoutControlMenuSubmenu; LayoutControlMenu; ViewContainerTitleContext
- Keys: Ctrl+B (Cmd+B on Mac)
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (286-391); V\src\vs\workbench\browser\layout.ts (1911-1944); V\src\vs\workbench\browser\parts\sidebar\sidebarActions.ts (18-33); C bundle (toggleSidebarVisibility class, closeSidebar action)
- Pieces:
  1. Pressing `Ctrl+B` (`Cmd+B` on Mac) hides the primary side bar when it is showing and shows it when it is hidden, and the editor area takes the freed width.
  2. Hiding the side bar closes its active container, and showing it again brings back the container used last.
  3. View > Appearance > Primary Side Bar, the layout control buttons in the title bar, and Hide Primary Side Bar in the container right-click menu all toggle the side bar the same way.
  4. Running Close Primary Side Bar (`workbench.action.closeSidebar`) from the Command Palette hides the side bar, and the entry is only enabled while the side bar is showing.
  5. After the toggle, a screen reader announces "Primary Side Bar hidden" or "Primary Side Bar shown". [stock only]
  6. In Cursor the toggle command and `Ctrl+B` do nothing while focus is in an auxiliary window, and the layout toggle button sits in the `2_pane_toggles` group of the layout menu instead of the navigation group. [cursor only]

#### SB-1.2 Move the side bar to the left or right

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (all three ids and the setting are present, the setting has 39 hits; Cursor's toggle command also stores the choice under an agent-layout override key before it updates the setting; the horizontal tab row has its own right-click entry, see AB-6)
- What: Switch the primary side bar (and the activity bar with it) between the left and right edge. The secondary side bar goes to the opposite side.
- Commands: workbench.action.toggleSidebarPosition; workbench.action.moveSideBarRight; workbench.action.moveSideBarLeft
- Menus: ViewContainerTitleContext; MenubarAppearanceMenu
- Keys: none found
- Settings: workbench.sideBar.location
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (97-171, 186-249); V\src\vs\workbench\browser\workbench.contribution.ts (566-572); C bundle (toggleSidebarPosition class, NLS "Toggle Primary Side Bar Position", "Move Primary Side Bar Right", "Move Primary Side Bar Left")
- Pieces:
  1. Setting `workbench.sideBar.location` puts the primary side bar and the activity bar on the `left` (default) or `right` edge, and the secondary side bar goes to the opposite edge.
  2. Running Toggle Primary Side Bar Position (`workbench.action.toggleSidebarPosition`) from the Command Palette flips the side bar and activity bar to the other edge.
  3. View > Appearance shows Move Primary Side Bar Right while the side bar is on the left and Move Primary Side Bar Left while it is on the right, and choosing it moves the side bar.
  4. The container right-click menu has the same Move Primary Side Bar Right or Left entry, shown only for a container that lives in the primary side bar.
  5. In Cursor the toggle command first saves the choice under a Cursor layout override key and then updates the setting, and the horizontal tab row has its own right-click move entry (see AB-6). [cursor only]

#### SB-1.3 Focus the side bar

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (id, label "Focus into Primary Side Bar" and Ctrl+0 are present; Cursor adds the "not focused in an auxiliary window" condition to the key and the command)
- What: Press Ctrl+0 to jump keyboard focus into the open side bar view. If the side bar is hidden it opens first.
- Commands: workbench.action.focusSideBar
- Menus: CommandPalette
- Keys: Ctrl+0 (Cmd+0 on Mac)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\sidebar\sidebarActions.ts (35-66); C bundle (focusSideBar class)
- Pieces:
  1. Pressing `Ctrl+0` (`Cmd+0` on Mac) puts keyboard focus inside the active view of the primary side bar.
  2. When the side bar is hidden, the same command opens it first and then moves focus into it.
  3. Focus into Primary Side Bar (`workbench.action.focusSideBar`) is listed in the Command Palette.
  4. In Cursor the key and the command do nothing while focus is in an auxiliary window. [cursor only]

#### SB-1.4 Resize the side bar

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the four commands are present with the same auxiliary window condition; the minimum width is 214 pixels in both Cursor side bar classes against 170 in stock, and the horizontal one has a minimum height of 77)
- What: Drag the edge to change the width, double-click the edge to jump to a best-fit width (at least 300 pixels), or drag it below the minimum to snap it shut. Increase and Decrease Current View Size change the width by 60 pixels while focus is in the side bar.
- Commands: workbench.action.increaseViewSize; workbench.action.decreaseViewSize; workbench.action.increaseViewWidth; workbench.action.decreaseViewWidth
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts (44-65); V\src\vs\workbench\browser\actions\layoutActions.ts (1138-1260); V\src\vs\workbench\browser\layout.ts (1799-1813); V\src\vs\base\browser\ui\grid\grid.ts (710-735); C bundle (side bar classes with `minimumWidth=214`, increaseViewSize action)
- Pieces:
  1. Dragging the edge between the side bar and the editor changes the side bar width.
  2. Double-clicking that edge jumps the side bar to a best-fit width of at least 300 pixels.
  3. Dragging the edge below the minimum width snaps the side bar shut.
  4. With focus in the side bar, Increase Current View Size, Decrease Current View Size, Increase Current View Width and Decrease Current View Width in the Command Palette (`workbench.action.increaseViewSize`, `decreaseViewSize`, `increaseViewWidth`, `decreaseViewWidth`) change its width by 60 pixels.
  5. In Cursor the side bar cannot be narrower than 214 pixels (stock stops at 170), the horizontal side bar has a minimum height of 77 pixels, and the four resize commands do nothing while focus is in an auxiliary window. [cursor only]

#### SB-1.5 Container title row and its "..." menu

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the Views submenu and the ViewContainerTitle menu are registered; in the default horizontal side bar the title label is replaced by the tab row, and the composite's primary and secondary actions are pushed to a Cursor-made toolbar; the stock title row remains in the vertical side bar)
- What: The strip above the views shows the container name, or "Container: View" when it holds a single view, with its own buttons on the right. Its "..." menu starts with a Views submenu and lists the container's other actions.
- Commands: none found
- Menus: SidebarTitle; ViewContainerTitle; Views
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\paneCompositePart.ts (332-366, 793-800); V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (47-51, 544-562); V\src\vs\workbench\browser\parts\views\viewMenuActions.ts (90); V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts (97-99); C bundle (Views submenu registration, composite toolbar `setActions`)
- Pieces:
  1. The strip above the views shows the container's name, with the container's own buttons at its right edge.
  2. A "..." button in that strip opens a menu whose first entry is a Views submenu listing the container's views.
  3. Container actions in the navigation group (from the `SidebarTitle` and `ViewContainerTitle` menus) show as icon buttons in the strip, and the other container actions are listed in the "..." menu under the Views submenu.
  4. In Cursor's default horizontal side bar the tab row replaces the title label and the container's actions go to a Cursor-made toolbar, while the stock title row remains in the vertical side bar. [cursor only]

#### SB-1.6 Container right-click menu

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (ViewContainerTitleContext, "Hide Primary Side Bar", "Move Primary Side Bar Right" and "Reset Location" are present; the Activity Bar Position submenu (ActivityBarPositionMenu) was not found; the horizontal tab row has its own right-click menu, see AB-6)
- What: Right-click the container title or the empty space under the views to show or hide its views, reset the container's location, move the side bar to the other side, hide it, or pick an activity bar position.
- Commands: <containerId>.resetViewContainerLocation; workbench.action.toggleSidebarPosition; workbench.action.toggleSidebarVisibility
- Menus: ViewContainerTitleContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (846-860, 899-923); V\src\vs\workbench\browser\actions\layoutActions.ts (186-229, 340-350); V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts (592-601); V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (395-397, 564-579); V\src\vs\workbench\browser\parts\paneCompositePart.ts (758-791); C bundle (ViewContainerTitleContext items, NLS "Hide Primary Side Bar")
- Pieces:
  1. Right-clicking the container title shows a menu that lists each of the container's views with a check mark on the visible ones, and choosing an entry shows or hides that view.
  2. Right-clicking the empty space under the views shows the same menu.
  3. Reset Location (`<containerId>.resetViewContainerLocation`) appears in the menu only when the container has been moved away from its default place, and choosing it sends the container home.
  4. The menu has Move Primary Side Bar Right (or Left when the side bar is on the right) and Hide Primary Side Bar entries.
  5. The menu has an Activity Bar Position submenu for choosing where the activity bar sits. [stock only]
  6. In Cursor's default horizontal side bar the tab row has its own right-click menu (see AB-6) instead of this one. [cursor only]

#### SB-1.7 Collapse and expand a view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched the bundle for `setExpanded` and the header class; the Cursor ViewPane adds one rule, see SB-1.21)
- What: Click a view header, or press Enter, Space, Left or Right on it, to fold or unfold the view. Up and Down move between headers, and each view's folded state is remembered.
- Commands: none found
- Menus: none found
- Keys: Enter; Space; Left Arrow; Right Arrow; Up Arrow; Down Arrow (with a view header focused)
- Settings: none found
- Source: V\src\vs\base\browser\ui\splitview\paneview.ts (296-320, 574-580); V\src\vs\workbench\browser\parts\views\viewPane.ts (437-444, 503-512); C bundle (ViewPane `setExpanded`)
- Pieces:
  1. Clicking a view header folds an open view and unfolds a folded one.
  2. With a view header focused, Enter or Space toggles the view open or closed.
  3. With a view header focused, Left Arrow folds the view and Right Arrow unfolds it.
  4. Up Arrow and Down Arrow move focus from one view header to the previous or next one.
  5. Each view's folded or open state is remembered.

#### SB-1.8 View header buttons and "..." menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present (ViewTitle has 110 uses in the bundle; the setting id has 3 hits and its description string is in NLS; the `actions-always-visible` class is there)
- What: Each view header shows its own buttons when you hover or focus it, plus a "..." menu for the extra actions. A setting keeps the buttons visible all the time.
- Commands: none found
- Menus: ViewTitle
- Keys: none found
- Settings: workbench.view.alwaysShowHeaderActions
- Source: V\src\vs\workbench\browser\parts\views\viewPane.ts (59-68, 455-496, 698-704); V\src\vs\workbench\browser\parts\views\viewMenuActions.ts; V\src\vs\workbench\browser\parts\views\treeView.ts (575-612); V\src\vs\workbench\browser\workbench.contribution.ts (686-690); C bundle (ViewPane header toolbar)
- Pieces:
  1. Each view header shows its own buttons at the right when the pointer is over the header or focus is inside the view, and hides them otherwise.
  2. A "..." menu on the view header holds the extra view title actions (the `ViewTitle` menu) that do not show as buttons.
  3. Setting `workbench.view.alwaysShowHeaderActions` keeps the view header buttons visible all the time when on, and shows them only on hover or focus when off (the default).

#### SB-1.9 Show or hide views in a container

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for `.toggleVisibility`, `.removeView`, the NLS label "Hide '{0}'" and ViewTitleContext, all found)
- What: Turn single views on or off from the Views submenu of the "..." menu, the container right-click menu, or a view header's right-click menu (which also has Hide '<view>'). The last visible view cannot be hidden.
- Commands: <viewId>.toggleVisibility; <viewId>.removeView
- Menus: Views; ViewContainerTitleContext; ViewTitleContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (831-897); V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (47-51, 751-761, 842-848); C bundle (toggleVisibility and removeView registrations)
- Pieces:
  1. The Views submenu in the container "..." menu lists every view of the container with a check mark on the visible ones, and choosing one shows or hides it (`<viewId>.toggleVisibility`).
  2. The container right-click menu lists the same show or hide toggles for each view.
  3. Right-clicking a view header shows the same toggles and a Hide '<view>' entry that hides that view (`<viewId>.removeView`).
  4. The last visible view cannot be hidden, because its toggle and its Hide entry are disabled.

#### SB-1.10 Resize views in the stack

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for `onDidSashReset` and `saveViewSizes`, both found)
- What: Drag the line between two view headers to give one view more height. Double-click the line to split the space of the two nearest open views evenly. The sizes are remembered.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (394, 515, 701-744, 1130-1168); V\src\vs\base\browser\ui\splitview\splitview.ts (1180-1198); C bundle (ViewPaneContainer sash reset handler)
- Pieces:
  1. Dragging the line between two view headers gives one view more height and the other less.
  2. Double-clicking that line splits the space between the two nearest open views evenly.
  3. The sizes of the views are remembered.

#### SB-1.11 Reorder views by dragging a header

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for `movePane(` and the view drag registration, both found)
- What: Drag a view header onto another view in the same container. A highlight shows whether it will land above or below, and the order is saved.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (71-182, 886-1015); C bundle (`movePane(`, view drag registration)
- Pieces:
  1. Dragging a view header onto another view in the same container moves the dragged view to that spot.
  2. While dragging, a highlight on the target view shows whether the view will land above or below it.
  3. The new order of the views is saved.

#### SB-1.12 Drag a view to another container, the panel or the secondary side bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for the `monaco-pane-drop-overlay` id and the drop target registration, both found)
- What: Drag a view header over another container, the panel or the secondary side bar. A drop overlay appears, and dropping splits that area above, below, left or right of the existing views. Dropping a whole container's title there merges all its views in.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.location
- Source: V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (71-182, 402-513, 886-1015); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (350-352, 381); C bundle (drop overlay class)
- Pieces:
  1. Dragging a view header over another container, the panel or the secondary side bar shows a drop overlay on that area.
  2. Dropping on the top, bottom, left or right part of the overlay places the view above, below, left or right of the views already there.
  3. After the drop, the view appears in the target area and is gone from the container it came from.

#### SB-1.13 Drag a whole container by its title

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the title label drag is registered once in the stock pane part class used by the vertical side bar; the default horizontal side bar has no title label, and its tabs have their own reorder drag, see AB-2 and AB-6)
- What: Drag the container title (for example EXPLORER) onto the panel, the secondary side bar or another container to move all its views together.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\paneCompositePart.ts (368-380); C bundle (title label drag registration)
- Pieces:
  1. Dragging a container title (for example EXPLORER) onto the panel or the secondary side bar moves all of that container's views together.
  2. Dropping a container title onto another container adds all of its views to that container.
  3. In Cursor's default horizontal side bar there is no title label to drag, and its tabs have their own reorder drag (see AB-2 and AB-6). [cursor only]

#### SB-1.14 Move a view with the picker

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids and the strings "Select a View to Move", "Select a Destination for the View", "New Panel Entry", "New Side Bar Entry", "New Secondary Side Bar Entry" are in the bundle)
- What: Run Move View to pick any movable view, then pick a destination: New Side Bar Entry, New Panel Entry, New Secondary Side Bar Entry, or an existing container. Move Focused View skips the first step.
- Commands: workbench.action.moveView; workbench.action.moveFocusedView
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (791-1094); C bundle (moveView and moveFocusedView actions)
- Pieces:
  1. Running Move View (`workbench.action.moveView`) from the Command Palette opens a picker titled "Select a View to Move" that lists the movable views.
  2. After a view is picked, a second picker titled "Select a Destination for the View" offers New Side Bar Entry, New Panel Entry, New Secondary Side Bar Entry and every existing container.
  3. Move Focused View (`workbench.action.moveFocusedView`) skips the first picker and moves the view that has focus.
  4. Choosing a destination moves the view there, and a New Entry destination creates a new container in that area to hold it.

#### SB-1.15 Move the focused view with the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the four ids and their key chords are in the bundle with the same weight, with a Mac variant)
- What: With a view focused, press Ctrl+K then Up or Left to move it one place earlier in its container, or Down or Right to move it one place later.
- Commands: views.moveViewUp; views.moveViewDown; views.moveViewLeft; views.moveViewRight
- Menus: none found
- Keys: Ctrl+K Up Arrow; Ctrl+K Down Arrow; Ctrl+K Left Arrow; Ctrl+K Right Arrow (when a view has focus; Cmd+K on Mac)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (1197-1288); C bundle (views.moveViewUp registration)
- Pieces:
  1. With a view focused, `Ctrl+K` then Up Arrow or Left Arrow moves it one place earlier in its container (`views.moveViewUp`, `views.moveViewLeft`).
  2. With a view focused, `Ctrl+K` then Down Arrow or Right Arrow moves it one place later in its container (`views.moveViewDown`, `views.moveViewRight`).
  3. These chords work only while a view has focus, and they use `Cmd+K` on Mac.

#### SB-1.16 Reset all view locations

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id and the string "Reset View Locations" found)
- What: Run Reset View Locations to put every moved view and container back where it started.
- Commands: workbench.action.resetViewLocations
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (773-789); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (reset at 408); C bundle (resetViewLocations action)
- Pieces:
  1. Running Reset View Locations (`workbench.action.resetViewLocations`) from the Command Palette puts every moved view back in the container it started in.
  2. The same command puts every moved container back in the area it started in.

#### SB-1.17 Reset one view to its home

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for `workbench.action.resetFocusedViewLocation`, `.resetViewLocation` and the strings "Reset Location" and "Reset Focused View Location", all found)
- What: Right-click a view header and choose Reset Location, or run Reset Focused View Location, to send that one view back to the container it started in.
- Commands: workbench.action.resetFocusedViewLocation; <viewId>.resetViewLocation
- Menus: ViewTitleContext; CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (1096-1136); V\src\vs\workbench\services\views\browser\viewsService.ts (604-638); C bundle (resetFocusedViewLocation action, per-view reset registration)
- Pieces:
  1. Right-clicking a view header and choosing Reset Location sends that view back to the container it started in and opens it there.
  2. Reset Location shows in the view header menu only while the view is not in its starting container.
  3. Running Reset Focused View Location (`workbench.action.resetFocusedViewLocation`) from the Command Palette does the same for the view that has focus.
  4. Running it with no view focused shows an error dialog saying "There is no view currently focused."

#### SB-1.18 Welcome content in an empty view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for the `welcome-view` classes and the `viewsWelcome` contribution point, both found)
- What: A view with nothing to show displays text, links and buttons instead of a blank list, for example the Explorer with no folder open. Lines appear and disappear as conditions change, a button can be greyed out by a precondition, and the layout widens above 640 pixels.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\views\viewPane.ts (101-309); V\src\vs\workbench\common\views.ts (395-435, 515-540); V\src\vs\workbench\contrib\welcomeViews\common\viewsWelcomeContribution.ts; V\src\vs\workbench\contrib\welcomeViews\common\viewsWelcomeExtensionPoint.ts; C bundle (welcome view controller)
- Pieces:
  1. A view with nothing to show displays its welcome text instead of a blank list, for example the Explorer when no folder is open.
  2. A welcome line that holds only a link shows as a button that runs its command, and links inside text open their target.
  3. Welcome lines appear and disappear as their conditions change.
  4. A welcome button or command link is greyed out and disabled while its precondition is not met.
  5. When the view is wider than 640 pixels the welcome content switches to a wider layout.

#### SB-1.19 Focus a view by command

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the string "Focus on {0} View" is in NLS, and `outline.focus` and `timeline.focus` were found in the bundle)
- What: Every view gets a "Focus on <name> View" command in the Command Palette that opens its container and puts focus inside, for example `outline.focus` and `timeline.focus`. A view can also declare its own shortcut for it.
- Commands: <viewId>.focus
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\views\browser\viewsService.ts (557-602); C bundle (FocusViewAction registration)
- Pieces:
  1. The Command Palette lists a "Focus on <name> View" command for each view, and running it opens the view's container and puts focus inside the view (for example `outline.focus` and `timeline.focus`).
  2. A keybinding can pass `preserveFocus` to the command to open the view without moving focus into it.
  3. A view can declare its own shortcut for its focus command.
  4. A view's focus command is listed only while that view is available.

#### SB-1.20 A one-view container shows one combined title

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the merge logic is present, with 14 hits for `isViewMergedWithContainer`; Cursor adds an "alone" state for a single view that is not merged, see SB-1.21; the merged title text has no place in the horizontal side bar because the tab row replaces the title label)
- What: When a container shows only one view, such as the Explorer with just Folders, the view header is hidden and the container title carries the view's buttons, reading "Container: View" when the names differ.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (544-562, 1080-1122); V\src\vs\workbench\browser\parts\views\viewPane.ts (410-417); C bundle (ViewPaneContainer `updateViewHeaders`)
- Pieces:
  1. When a container shows only one view, the view's own header is hidden.
  2. The container title then shows the view's buttons.
  3. The container title reads "Container: View" when the two names differ, and just the container name when they match.
  4. When a second view becomes visible, the combined title goes away and each view gets its own header again.
  5. In Cursor's default horizontal side bar the merged title text has no place because the tab row replaces the title label, and a single view that is not merged gets the lone-view state (see SB-1.21). [cursor only]

#### SB-1.21 A lone view cannot be folded

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: In Cursor a view that is alone in its container is forced open. In the default horizontal side bar its collapse arrow is also hidden, while in the vertical side bar the arrow stays but does nothing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.orientation
- Source: C bundle (ViewPane `setIsAlone`, `isAlone`, `updateTwistyIcon` with the orientation check; ViewPaneContainer `updateViewHeaders` calling `setIsAlone`)
- Pieces:
  1. A view that is alone in its container is forced open and cannot be folded. [cursor only]
  2. In the default horizontal side bar the collapse arrow of a lone view is hidden. [cursor only]
  3. In the vertical side bar the collapse arrow of a lone view stays but clicking it does nothing. [cursor only]
  4. Setting `workbench.activityBar.orientation` chooses `horizontal` (default) or `vertical`, which decides which of the two lone-view looks you see after a restart. [cursor only]

#### SB-1.22 Two-row title for a container with a toolbar

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: In the horizontal side bar a container can ask for an extra action row. Run and Debug does, so its title grows from 35 to 70 pixels to hold the Start button and the launch configuration picker.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.orientation
- Source: C bundle (`wantsActionToolbar: true` on the Run and Debug container, `heightOfTitle` set to 35 or 70; 0 hits for `wantsActionToolbar` in V)
- Pieces:
  1. In the horizontal side bar a container can ask for an extra action row, and its title area grows from 35 to 70 pixels to hold it. [cursor only]
  2. The Run and Debug container uses the two-row title, with the Start button and the launch configuration picker in the second row. [cursor only]
  3. Containers that do not ask for the extra row keep the 35 pixel title. [cursor only]
  4. Setting `workbench.activityBar.orientation` set to `vertical` removes the two-row title and brings back the stock title row. [cursor only]

#### SB-1.23 Move through and find inside a view's tree or list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for `list.find`, `list.toggleFindMode`, `list.focusDown`, `list.collapseAll` and the six settings, all found)
- What: In any view tree or list, arrow keys, Home, End, Page Up and Page Down move focus, Right and Left unfold and fold, Enter opens, and Space toggles. Ctrl+Alt+F opens a find box that either highlights or filters rows, and typing letters jumps to a match.
- Commands: list.focusDown; list.focusUp; list.expand; list.collapse; list.collapseAll; list.select; list.toggleExpand; list.find; list.closeFind; list.toggleFindMode; list.toggleFindMatchType; list.triggerTypeNavigation; list.collapseAllToFocus; list.focusAnyDown; list.focusAnyFirst; list.focusAnyLast; list.focusAnyUp; list.focusFirst; list.focusLast; list.focusPageDown; list.focusPageUp; list.focusParent; list.selectAndPreserveFocus
- Menus: none found
- Keys: Up Arrow; Down Arrow; Left Arrow; Right Arrow; Enter; Space; Home; End; Ctrl+Alt+F; F3; Esc (all with a list focused)
- Settings: workbench.list.defaultFindMode; workbench.list.typeNavigationMode; workbench.list.defaultFindMatchType; workbench.list.openMode; workbench.list.multiSelectModifier; workbench.list.horizontalScrolling
- Source: V\src\vs\workbench\browser\actions\listCommands.ts (76-900); V\src\vs\platform\list\browser\listService.ts (170-186); C bundle (list command registrations)
- Pieces:
  1. In any view tree or list, Up Arrow and Down Arrow move focus one row, Home and End jump to the first and last row, and Page Up and Page Down move by a page (`list.focusUp`, `list.focusDown`).
  2. Right Arrow unfolds a focused folded row, and Left Arrow folds an open row or moves focus to its parent row (`list.expand`, `list.collapse`).
  3. `Ctrl+Left Arrow` folds every row in the tree (`list.collapseAll`).
  4. Enter opens the focused row (`list.select`) and Space toggles it open or closed (`list.toggleExpand`).
  5. `Ctrl+Alt+F` or `F3` opens a find box in the list (`list.find`), and Esc closes it (`list.closeFind`).
  6. Setting `workbench.list.defaultFindMode` chooses what the find box does, with values `highlight` (default) and `filter`, and `list.toggleFindMode` flips it.
  7. Setting `workbench.list.defaultFindMatchType` chooses how the find box matches, with values `fuzzy` (default) and `contiguous`, and `list.toggleFindMatchType` flips it.
  8. Typing letters in a focused list jumps to a match, and setting `workbench.list.typeNavigationMode` chooses `automatic` (default) or `trigger`, where typing starts a jump only after `list.triggerTypeNavigation` runs.
  9. Setting `workbench.list.openMode` chooses `singleClick` (default) or `doubleClick` for opening items with the mouse, and `workbench.list.multiSelectModifier` chooses `ctrlCmd` (default) or `alt` for adding to a mouse multi-selection.
  10. Setting `workbench.list.horizontalScrolling` lets lists and trees scroll sideways when on, and keeps them from scrolling sideways when off (the default).

#### SB-1.24 Sticky scroll and indent look of view trees

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for `tree.toggleStickyScroll`, the five settings and the colour `sideBarStickyScroll.background`, all found)
- What: Trees such as the Explorer pin the parent rows at the top while you scroll. Toggle it from the Command Palette or the setting, choose how many rows stick, and change indent size, indent guides, and whether a single or double click expands.
- Commands: tree.toggleStickyScroll; list.stickyScroll.collapse; list.stickyScrollselect; list.stickyScrolltoggleExpand
- Menus: CommandPalette
- Keys: Left (list.stickyScroll.collapse, pinned row focused; macOS also Cmd+Up); Enter (list.stickyScrollselect; macOS also Cmd+Down); Space (list.stickyScrolltoggleExpand)
- Settings: workbench.tree.enableStickyScroll; workbench.tree.stickyScrollMaxItemCount; workbench.tree.indent; workbench.tree.renderIndentGuides; workbench.tree.expandMode
- Source: V\src\vs\workbench\browser\actions\listCommands.ts (967-986); V\src\vs\platform\list\browser\listService.ts (179-186); V\src\vs\workbench\common\theme.ts (650-654); C bundle (ToggleStickyScroll action); V\src\vs\workbench\browser\actions\listCommands.ts; V\src\vs\platform\list\browser\listService.ts (defaults true and 7); V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (getContext reads the pinned focus); C bundle (tree.toggleStickyScroll, label Toggle Tree Sticky Scroll, list.stickyScroll.collapse, list.stickyScrollselect, list.stickyScrolltoggleExpand, same defaults)
- Pieces:
  1. While you scroll a tree such as the Explorer, the parent rows of the first visible row stay pinned at the top.
  2. Running Toggle Tree Sticky Scroll (`tree.toggleStickyScroll`) from the Command Palette turns the pinned rows on or off.
  3. Setting `workbench.tree.enableStickyScroll` turns pinned rows on (the default) or off for trees.
  4. Setting `workbench.tree.stickyScrollMaxItemCount` chooses how many rows can be pinned, with a default of 7 and a minimum of 1.
  5. Setting `workbench.tree.indent` chooses the indent of each tree level in pixels, from 4 to 40, with a default of 8.
  6. Setting `workbench.tree.renderIndentGuides` chooses when indent guides show, with values `none`, `onHover` (default) and `always`.
  7. Setting `workbench.tree.expandMode` chooses whether a `singleClick` (default) or a `doubleClick` on a folder name expands it.
  8. Pinned rows in the side bar draw on the `sideBarStickyScroll.background` color, which a theme can change.
  9. Scrolling deep into a folder keeps its parent folder rows pinned at the top of the tree.
  10. With a pinned row focused, Left collapses that folder (`Cmd+Up` also does on macOS).


## SB-2 Explorer

(both) [boundary for Cursor-added items only]. The file tree, file right-click operations, Open Editors, and the empty-folder state. Cursor adds right-click items for chat, browser, and Cursor Blame.

48 features.

#### SB-2.1 Show and focus the Explorer

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+Shift+E or pick Explorer in the View menu to show the Explorer container with its Folders tree (Open Editors starts hidden), or run Focus on Files Explorer to put the keyboard in the tree. In Cursor the Explorer is the first pinned tab of the Cursor tab row, which AB-6 owns.
- Commands: workbench.view.explorer; workbench.files.action.focusFilesExplorer; workbench.explorer.fileView.focus
- Menus: MenubarViewMenu; CommandPalette
- Keys: Ctrl+Shift+E (workbench.view.explorer)
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\explorerViewlet.ts; V\src\vs\workbench\contrib\files\browser\fileActions.ts (FocusFilesExplorer); V\src\vs\workbench\services\views\browser\viewsService.ts; C bundle (container id workbench.view.explorer, FocusFilesExplorer id, focusCommand workbench.explorer.fileView.focus)
- Pieces:
  1. Pressing `Ctrl+Shift+E` (`workbench.view.explorer`) or picking Explorer in the View menu opens the side bar on the Explorer container and shows its Folders tree.
  2. The Open Editors pane starts hidden, so a fresh Explorer shows only the Folders pane.
  3. Running Focus on Files Explorer (`workbench.files.action.focusFilesExplorer`) from the Command Palette puts keyboard focus in the Folders tree so the arrow keys move between rows.
  4. Running `workbench.explorer.fileView.focus` moves focus to the Folders pane directly.

#### SB-2.2 Expand and collapse folders

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Click a folder twisty or name, or press Right and Left, to open and close folders; Ctrl+Left collapses the whole tree, and the Collapse Folders in Explorer header button folds every folder (in a multi-root workspace it folds the children and keeps the roots open). Folders with a single child open on their own.
- Commands: workbench.files.action.collapseExplorerFolders; list.expand; list.collapse; list.collapseAll; list.toggleExpand
- Menus: ViewTitle (view == workbench.explorer.fileView); CommandPalette
- Keys: Right (list.expand); Left (list.collapse); Ctrl+Left (list.collapseAll); Space (list.toggleExpand)
- Settings: workbench.tree.expandMode
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (collapseAll, header button); V\src\vs\workbench\browser\actions\listCommands.ts; V\src\vs\platform\list\browser\listService.ts; C bundle (workbench.files.action.collapseExplorerFolders, list.collapseAll, workbench.tree.expandMode)
- Pieces:
  1. Clicking the twisty arrow on a folder row opens or closes that folder.
  2. Setting `workbench.tree.expandMode` chooses whether clicking a folder name opens it on one click (`singleClick`, the default) or on a double click (`doubleClick`).
  3. With a folder row focused, Right opens it and Left closes it (`list.expand`, `list.collapse`).
  4. Pressing Space on a focused folder row opens or closes it (`list.toggleExpand`).
  5. Pressing `Ctrl+Left` closes every open folder in the tree at once (`list.collapseAll`).
  6. The Collapse Folders in Explorer button in the Explorer header (also `workbench.files.action.collapseExplorerFolders` in the Command Palette) folds every folder, and in a multi-root workspace it folds the children while keeping each root open.
  7. Opening a folder that holds only one child folder also opens that child.

#### SB-2.3 Open a file from the tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Click a file to open it in the editor as a preview (italic tab), double-click to keep it open, press Enter to open it and move focus to the editor (Cmd+Down on macOS), or press Space to open it and keep focus in the tree.
- Commands: explorer.openAndPassFocus; filesExplorer.openFilePreserveFocus; list.select
- Menus: none found
- Keys: Enter (explorer.openAndPassFocus, when a non-folder is focused; macOS Cmd+Down); Space (filesExplorer.openFilePreserveFocus, when a non-folder is focused)
- Settings: workbench.editor.enablePreview; workbench.list.openMode; workbench.tree.expandMode
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (tree.onDidOpen); V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\platform\list\browser\listService.ts; C bundle (explorer.openAndPassFocus, filesExplorer.openFilePreserveFocus, workbench.list.openMode)
- Pieces:
  1. Clicking a file in the tree opens it in the editor as a preview, shown with an italic tab title.
  2. Double-clicking a file opens it as a kept-open editor, and the tab title is no longer italic.
  3. Pressing Enter on a focused file opens it and moves keyboard focus to the editor (`explorer.openAndPassFocus`, `Cmd+Down` on macOS).
  4. Pressing Space on a focused file opens it and keeps keyboard focus in the tree (`filesExplorer.openFilePreserveFocus`).
  5. Setting `workbench.editor.enablePreview` chooses whether a file opened from the tree takes the preview tab or a normal tab, with values on and off.
  6. Setting `workbench.list.openMode` chooses whether a file opens on a single click (`singleClick`, the default) or only on a double click (`doubleClick`).

#### SB-2.4 Open a file to the side

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Open the selected file or files in a new editor group beside the current one from Open to the Side in the right-click menu or with Ctrl+Enter (Control+Enter on macOS). The same entry is in the Open Editors menu.
- Commands: explorer.openToSide
- Menus: ExplorerContext; OpenEditorsContext
- Keys: Ctrl+Enter (when the Explorer viewlet has focus; macOS Control+Enter)
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; C bundle (explorer.openToSide, label Open to the Side)
- Pieces:
  1. Right-clicking a file and picking Open to the Side opens it in a new editor group beside the current one.
  2. Pressing `Ctrl+Enter` (`Control+Enter` on macOS) while the Explorer has focus opens the selected file beside the current editor (`explorer.openToSide`).
  3. With several files selected, Open to the Side opens each of them in the side group.
  4. The right-click menu of an Open Editors row has the same Open to the Side entry.

#### SB-2.5 Open a file with another editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick Open With... in the right-click menu on a file to choose which editor opens it (for example the text editor or a custom viewer). The entry shows for files that have editors listed in the available editors context key, and never for folders.
- Commands: explorer.openWith
- Menus: ExplorerContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\common\files.ts (ExplorerResourceAvailableEditorIdsContext); C bundle (explorer.openWith, label Open With...)
- Pieces:
  1. Right-clicking a file and picking Open With... lets the person choose which editor opens it, such as the text editor or a custom viewer (`explorer.openWith`).
  2. The Open With... entry shows only for files whose available editors context key is not empty.
  3. The Open With... entry never shows on a folder.

#### SB-2.6 Create a file or folder

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Make a new file or folder with a name box typed straight into the tree. Use the New File and New Folder buttons in the Explorer header, New File... and New Folder... in the right-click menu on a folder, or double-click empty space for a new file. The box checks the name as you type (empty, leading slash, invalid characters, name already used).
- Commands: explorer.newFile; explorer.newFolder; workbench.files.action.createFileFromExplorer; workbench.files.action.createFolderFromExplorer
- Menus: ExplorerContext; ViewTitle (view == workbench.explorer.fileView); CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (openExplorerAndCreate, validateFileName); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (header buttons, double-click on empty area); C bundle (explorer.newFile, explorer.newFolder, createFileFromExplorer, createFolderFromExplorer)
- Pieces:
  1. Clicking the New File button in the Explorer header puts a name box in the tree, and typing a name and pressing Enter creates the file and opens it as a pinned tab.
  2. Clicking the New Folder button in the Explorer header puts a name box in the tree, and pressing Enter creates the folder and selects it.
  3. Right-clicking a folder and picking New File... or New Folder... shows the name box inside that folder, and with a file selected the box goes in that file's folder.
  4. Double-clicking empty space in the tree starts a new file name box.
  5. The name box shows an error as you type when the name is empty, starts with a slash, uses characters that are not valid, or is already used in that folder, and a warning for leading or trailing spaces.
  6. Pressing Escape in the name box cancels without creating anything.
  7. Typing a name that ends in a slash makes a folder instead of a file.
  8. Running New File or New Folder from the Command Palette (`workbench.files.action.createFileFromExplorer`, `workbench.files.action.createFolderFromExplorer`) opens the Explorer and shows the same name box.
  9. Creating inside a read-only folder fails with the error "Parent folder is readonly.".

#### SB-2.7 Rename a file or folder

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press F2 (Enter on macOS) or pick Rename... in the right-click menu to edit the name in place; the box shows the same name checks as creating. Not offered on workspace root folders or read-only items.
- Commands: renameFile
- Menus: ExplorerContext
- Keys: F2 (renameFile, files Explorer focused, item not a root and writable; macOS Enter)
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (renameHandler, validateFileName); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; C bundle (renameFile)
- Pieces:
  1. Pressing F2 (Enter on macOS) on the selected row, or picking Rename... in the right-click menu, turns its name into an editable box in place with the file name selected but not its extension (`renameFile`).
  2. The rename box shows the same name errors and warnings as the create box, and Enter confirms while Escape cancels.
  3. Pressing F2 again inside the rename box moves the selection between the name, the whole text and the extension.
  4. Rename... is not offered on workspace root folders or read-only items.

#### SB-2.8 Delete a file or folder

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Delete (Cmd+Backspace on macOS) or pick Delete in the right-click menu to send items to the Recycle Bin or Trash, or press Shift+Delete or hold Alt on the menu to delete permanently. With the OS trash turned off, Delete removes for good. The confirm box is owned by DL-4.
- Commands: moveFileToTrash; deleteFile
- Menus: ExplorerContext
- Keys: Delete (moveFileToTrash; macOS Cmd+Backspace or Delete); Shift+Delete (deleteFile; macOS Cmd+Alt+Backspace); Delete (deleteFile, when trash is unavailable; macOS Cmd+Backspace)
- Settings: files.enableTrash; explorer.confirmDelete
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (moveFileToTrashHandler, deleteFileHandler); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (moveFileToTrash, deleteFile, files.enableTrash, explorer.confirmDelete, label Delete Permanently)
- Pieces:
  1. Pressing Delete (`Cmd+Backspace` on macOS) on the selected rows, or picking Delete in the right-click menu, sends them to the Recycle Bin or Trash and removes them from the tree (`moveFileToTrash`).
  2. Pressing `Shift+Delete` (`Cmd+Alt+Backspace` on macOS) deletes the rows permanently (`deleteFile`).
  3. Holding Alt while the right-click menu is open changes Delete into Delete Permanently.
  4. Setting `files.enableTrash` chooses whether Delete uses the OS trash, and when it is off the Delete key removes items for good.
  5. Setting `explorer.confirmDelete` chooses whether a confirmation box appears before deleting, with values on and off and on as the default (the box is DL-4).

#### SB-2.9 Cut, copy and paste files

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Cut, Copy and Paste files and folders with Ctrl+X, Ctrl+C and Ctrl+V or the right-click menu. Cut items look dimmed until pasted, and Esc cancels the cut. Pasting beside the original makes a copy with a new name, and files copied from the operating system paste in too.
- Commands: filesExplorer.cut; filesExplorer.copy; filesExplorer.paste; filesExplorer.cancelCut
- Menus: ExplorerContext
- Keys: Ctrl+X (filesExplorer.cut); Ctrl+C (filesExplorer.copy); Ctrl+V (filesExplorer.paste); Escape (filesExplorer.cancelCut, when something is cut)
- Settings: explorer.incrementalNaming; explorer.confirmPasteNative
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (cutFileHandler, copyFileHandler, pasteFileHandler, findValidPasteFileTarget); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (paste event, itemsCopied); C bundle (filesExplorer.cut, copy, paste, cancelCut, explorer.incrementalNaming, explorer.confirmPasteNative)
- Pieces:
  1. Pressing `Ctrl+X` or picking Cut in the right-click menu marks the selected rows as cut, and they look dimmed until they are pasted.
  2. Pressing Escape while rows are cut cancels the cut and the dimming goes away (`filesExplorer.cancelCut`).
  3. Pressing `Ctrl+C` or picking Copy in the right-click menu copies the selected rows.
  4. Pressing `Ctrl+V` or picking Paste with a folder selected puts copies of the copied items in that folder.
  5. Pasting cut items into a folder moves them there.
  6. Pasting copied items beside the originals makes a copy with a new name.
  7. Setting `explorer.incrementalNaming` chooses the new name of a pasted copy, with values `simple` (adds the word copy and maybe a number), `smart` (adds or raises a number) and `disabled` (asks to overwrite instead).
  8. Files copied from the operating system file manager paste into the tree with `Ctrl+V`.
  9. Setting `explorer.confirmPasteNative` chooses whether pasting files from the operating system asks first, with values on and off and on as the default (the box is DL-4).

#### SB-2.10 Undo and redo file operations

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+Z in the Explorer to undo the last create, rename, move, copy or delete, and Ctrl+Y to redo. Some undo steps ask first, depending on the confirm level.
- Commands: undo; redo
- Menus: MenubarEditMenu
- Keys: Ctrl+Z (undo); Ctrl+Y or Ctrl+Shift+Z (redo; macOS Cmd+Shift+Z)
- Settings: explorer.enableUndo; explorer.confirmUndo
- Source: V\src\vs\workbench\contrib\files\browser\files.contribution.ts (UndoCommand and RedoCommand implementations named explorer); V\src\vs\editor\browser\editorExtensions.ts; C bundle (addImplementation(110,"explorer") for undo, explorer.enableUndo, explorer.confirmUndo)
- Pieces:
  1. Pressing `Ctrl+Z` with the Explorer focused undoes the last create, rename, move, copy or delete, and the tree shows the item back as before.
  2. Pressing `Ctrl+Y` or `Ctrl+Shift+Z` (`Cmd+Shift+Z` on macOS) with the Explorer focused redoes the undone file operation.
  3. The Undo and Redo entries in the Edit menu do the same while the Explorer has focus.
  4. Setting `explorer.enableUndo` turns Explorer undo on or off, with on as the default.
  5. Setting `explorer.confirmUndo` chooses when an undo asks first, with values `verbose` (before every undo), `default` (before destructive undos) and `light` (no prompt when the Explorer is focused).

#### SB-2.11 Drag and drop to move or copy in the tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Drag files and folders onto a folder to move them, or hold Ctrl (Alt on macOS) while dropping to copy. Hovering over a folder opens it, dropping on a file uses its parent folder, and in a multi-root workspace dragging a root folder reorders the roots. A confirm box asks before a move unless turned off (DL-4 owns the box).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.enableDragAndDrop; explorer.confirmDragAndDrop; explorer.incrementalNaming
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FileDragAndDrop); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (explorer.enableDragAndDrop, explorer.confirmDragAndDrop, explorer.incrementalNaming)
- Pieces:
  1. Dragging files or folders onto a folder in the tree moves them into that folder.
  2. Holding `Ctrl` (`Alt` on macOS) while dropping copies the dragged items instead of moving them.
  3. Holding a dragged item over a closed folder opens that folder after a short wait.
  4. Dropping onto a file row puts the dragged items in that file's parent folder.
  5. In a multi-root workspace, dragging a root folder onto another root reorders the roots.
  6. A confirm box asks before a move unless `explorer.confirmDragAndDrop` is off, with on as the default (the box is DL-4).
  7. Setting `explorer.enableDragAndDrop` turns moving by drag and drop inside the Explorer on or off, with on as the default.
  8. Setting `explorer.incrementalNaming` chooses the name of a dropped copy when the target folder already holds that name, with values `simple`, `smart` and `disabled`.

#### SB-2.12 Drop files in from the OS or drag them out

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Drop files or folders from the operating system onto the tree to copy them into the folder under the drop (a dropped file opens by default), and drag tree items out to the editor area or outside the app. Dragging between two windows' Explorers also works.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.autoOpenDroppedFile
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FileDragAndDrop.drop and onDragStart); V\src\vs\workbench\contrib\files\browser\fileImportExport.ts (ExternalFileImport); C bundle (explorer.autoOpenDroppedFile)
- Pieces:
  1. Dropping files or folders from the operating system onto the tree copies them into the folder under the drop.
  2. A file dropped in from the operating system opens in the editor by default, and setting `explorer.autoOpenDroppedFile` turns that on or off.
  3. Dragging rows from the tree into the editor area opens those files in the editor.
  4. Dragging rows from the tree to a place outside the app hands the files to the operating system.
  5. Dragging rows from one window's Explorer onto another window's Explorer copies them there.

#### SB-2.13 Download and upload files

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick Download... on a remote file or folder to save a copy to the local machine, and Upload... on a writable folder to add local files (Upload is web only; Download on a local desktop folder does not show). Rarely visible in a local desktop session.
- Commands: explorer.download; explorer.upload
- Menus: ExplorerContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (DOWNLOAD_COMMAND_ID, UPLOAD_COMMAND_ID); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\fileImportExport.ts (FileDownload, BrowserFileUpload); C bundle (explorer.download, explorer.upload, labels Download... and Upload...)
- Pieces:
  1. Right-clicking a remote file or folder and picking Download... saves a copy of it to the local machine (`explorer.download`).
  2. The Download... entry does not show on a folder or file in a local desktop session.
  3. Right-clicking a writable folder and picking Upload... lets the person pick local files and adds them to that folder (`explorer.upload`).
  4. The Upload... entry shows only in the web version and not on read-only folders.

#### SB-2.14 Reveal the active file in the Explorer

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The tree selects and scrolls to the file in the active editor by itself (auto reveal), or you run Reveal Active File in Explorer View, or Reveal in Explorer View on an editor tab menu, when you want it. Paths matching the auto reveal exclude list (node_modules and bower_components by default) are skipped.
- Commands: workbench.files.action.showActiveFileInExplorer; revealInExplorer
- Menus: EditorTitleContext; CommandPalette
- Keys: none found
- Settings: explorer.autoReveal; explorer.autoRevealExclude
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (ShowActiveFileInExplorer); V\src\vs\workbench\contrib\files\browser\fileCommands.ts (REVEAL_IN_EXPLORER_COMMAND_ID); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (selectActiveFile, selectResource); V\src\vs\workbench\contrib\files\browser\explorerService.ts; C bundle (showActiveFileInExplorer, revealInExplorer, explorer.autoReveal, explorer.autoRevealExclude)
- Pieces:
  1. With `explorer.autoReveal` on, switching to another editor selects and scrolls to that file in the tree, and opens the folders above it if they were closed.
  2. Setting `explorer.autoReveal` chooses what happens when a file opens, with values `true` (reveal and select, the default), `false` (do nothing) and `focusNoScroll` (focus the row without scrolling to it).
  3. Running Reveal Active File in Explorer View from the Command Palette (`workbench.files.action.showActiveFileInExplorer`) selects and scrolls to the active file on demand.
  4. Picking Reveal in Explorer View from an editor tab's right-click menu (`revealInExplorer`) selects that tab's file in the tree.
  5. Files matching `explorer.autoRevealExclude` are skipped by auto reveal, with `**/node_modules` and `**/bower_components` as the default entries.

#### SB-2.15 Reveal a file in the OS file manager

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick Reveal in File Explorer (Reveal in Finder on macOS, Open Containing Folder on Linux) on a file or folder, or press Ctrl+Alt+R (Shift+Alt+R on Windows), to show it in the operating system's file manager. Ctrl+K R does the same for the active editor. Desktop only.
- Commands: revealFileInOS; workbench.action.files.revealActiveFileInWindows
- Menus: ExplorerContext; OpenEditorsContext; EditorTitleContext; CommandPalette
- Keys: Ctrl+Alt+R (revealFileInOS, when no editor text has focus; Windows Shift+Alt+R); Ctrl+K R (workbench.action.files.revealActiveFileInWindows)
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\electron-browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\electron-browser\fileCommands.ts; C bundle (revealFileInOS, workbench.action.files.revealActiveFileInWindows)
- Pieces:
  1. Right-clicking a file or folder and picking Reveal in File Explorer shows it in the operating system's file manager (`revealFileInOS`), and the label reads Reveal in Finder on macOS and Open Containing Folder on Linux.
  2. The Open Editors menu and the editor tab menu have the same Reveal in File Explorer entry.
  3. Pressing `Ctrl+Alt+R` (`Shift+Alt+R` on Windows) while no editor text has focus reveals the selected item in the file manager.
  4. Pressing `Ctrl+K R` reveals the active editor's file in the file manager (`workbench.action.files.revealActiveFileInWindows`).
  5. These entries and keys exist only in the desktop app and not in a browser session.

#### SB-2.16 Copy a file path

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick Copy Path or Copy Relative Path on a file or folder (several items copy one path per line) to put the path on the clipboard. Keys and Command Palette entries do the same for the active file, and two settings choose the slash style.
- Commands: copyFilePath; copyRelativeFilePath; workbench.action.files.copyPathOfActiveFile
- Menus: ExplorerContext; OpenEditorsContext; EditorTitleContext; CommandPalette
- Keys: Ctrl+Alt+C (copyFilePath, no editor focus; Windows Shift+Alt+C); Ctrl+Shift+Alt+C (copyRelativeFilePath, no editor focus; Windows Ctrl+K Ctrl+Shift+C); Ctrl+K P (workbench.action.files.copyPathOfActiveFile)
- Settings: explorer.copyPathSeparator; explorer.copyRelativePathSeparator
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (copyFilePath, copyRelativeFilePath, explorer.copyPathSeparator, explorer.copyRelativePathSeparator)
- Pieces:
  1. Right-clicking a file or folder and picking Copy Path puts its full path on the clipboard (`copyFilePath`).
  2. Picking Copy Relative Path puts the path relative to the workspace folder on the clipboard (`copyRelativeFilePath`).
  3. With several rows selected, each copy entry puts one path per line on the clipboard.
  4. The Open Editors menu and the editor tab menu have the same Copy Path and Copy Relative Path entries.
  5. Pressing `Ctrl+Alt+C` (`Shift+Alt+C` on Windows) copies the path and `Ctrl+Shift+Alt+C` (`Ctrl+K Ctrl+Shift+C` on Windows) copies the relative path while no editor text has focus, and `Ctrl+K P` copies the path of the active file.
  6. Copy Path of Active File (`workbench.action.files.copyPathOfActiveFile`) and Copy Relative Path of Active File in the Command Palette copy the path of the active file.
  7. Settings `explorer.copyPathSeparator` and `explorer.copyRelativePathSeparator` choose the slash style of the copied path, with values `/`, `\` and `auto` (the operating system's own, the default).

#### SB-2.17 Compare two files from the tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a file and pick Select for Compare, then right-click another file and pick Compare with Selected to open a diff, or select exactly two files and pick Compare Selected. Folders are not offered. The diff editor itself is ED-6.
- Commands: selectForCompare; compareFiles; compareSelected
- Menus: ExplorerContext; OpenEditorsContext; EditorTitleContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\fileConstants.ts; C bundle (selectForCompare, compareFiles, compareSelected, labels Select for Compare and Compare with Selected)
- Pieces:
  1. Right-clicking a file and picking Select for Compare marks it as the left side of a comparison.
  2. Right-clicking another file and picking Compare with Selected opens a diff of the two files, and that entry shows only after a file has been marked.
  3. Selecting exactly two files and picking Compare Selected opens a diff of the two.
  4. The compare entries are not offered on folders.
  5. The Open Editors menu and the editor tab menu have the same compare entries.

#### SB-2.18 Open a folder in a terminal

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick Open in Integrated Terminal to start a terminal in the chosen folder (or a file's folder), or Open in External Terminal to start the system terminal there. A setting decides which of the two entries show. The terminal itself is PN-2.
- Commands: openInIntegratedTerminal; openInTerminal
- Menus: ExplorerContext
- Keys: none found
- Settings: terminal.explorerKind
- Source: V\src\vs\workbench\contrib\externalTerminal\browser\externalTerminal.contribution.ts; V\src\vs\workbench\contrib\externalTerminal\electron-browser\externalTerminal.contribution.ts (terminal.explorerKind); C bundle (openInIntegratedTerminal, openInTerminal, terminal.explorerKind)
- Pieces:
  1. Right-clicking a folder and picking Open in Integrated Terminal opens a terminal whose working folder is that folder (`openInIntegratedTerminal`).
  2. Picking Open in Integrated Terminal on a file opens a terminal in that file's folder.
  3. Picking Open in External Terminal starts the system terminal in the chosen folder (`openInTerminal`).
  4. Setting `terminal.explorerKind` chooses which entries show for local files, with values `integrated` (the default), `external` and `both`.
  5. In a remote window the integrated terminal opens even when the external entry is chosen.

#### SB-2.19 Manage workspace root folders

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a root folder and pick Add Folder to Workspace... or Remove Folder from Workspace (shown when multi-root workspaces are supported or a workspace file is open). A multi-root workspace lists each root at the top level, and a setting expands a workspace that holds only one folder when the window starts. Open Folder Settings on a root is handed off.
- Commands: addRootFolder; removeRootFolder; workbench.action.removeRootFolder
- Menus: ExplorerContext
- Keys: none found
- Settings: explorer.expandSingleFolderWorkspaces
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\workbench\contrib\files\browser\fileCommands.ts (REMOVE_ROOT_FOLDER_COMMAND_ID); V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (setTreeInput); C bundle (addRootFolder, removeRootFolder, labels Add Folder to Workspace and Remove Folder from Workspace, explorer.expandSingleFolderWorkspaces)
- Pieces:
  1. Right-clicking a root folder and picking Add Folder to Workspace... opens a folder picker and adds the chosen folder as another root (`addRootFolder`).
  2. Right-clicking a root folder and picking Remove Folder from Workspace takes that root out of the tree (`removeRootFolder`).
  3. The add and remove entries show only when multi-root workspaces are supported or a workspace file is open.
  4. A multi-root workspace lists each root folder as its own top level row in the Folders tree.
  5. Setting `explorer.expandSingleFolderWorkspaces` chooses whether a workspace holding only one folder starts expanded when the window opens, with on as the default.

#### SB-2.20 Refresh the Explorer

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press the Refresh Explorer button in the Explorer header, or run the command, to reload the tree from disk. It is disabled while the tree find box is filtering.
- Commands: workbench.files.action.refreshFilesExplorer
- Menus: ViewTitle (view == workbench.explorer.fileView); CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (refreshFilesExplorer action); C bundle (workbench.files.action.refreshFilesExplorer)
- Pieces:
  1. Clicking the Refresh Explorer button in the Explorer header reloads the tree from disk.
  2. Running Refresh Explorer from the Command Palette (`workbench.files.action.refreshFilesExplorer`) does the same.
  3. The refresh is disabled while the tree find box is filtering.

#### SB-2.21 Compact folders

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A folder that holds only one subfolder is shown on one row as a joined path (for example src/main/java). With such a row focused, Left, Right, Home and End move between the parts of the path. The setting is ignored when a screen reader is in use.
- Commands: previousCompressedFolder; nextCompressedFolder; firstCompressedFolder; lastCompressedFolder
- Menus: none found
- Keys: Left (previousCompressedFolder, compact row focused); Right (nextCompressedFolder); Home (firstCompressedFolder); End (lastCompressedFolder)
- Settings: explorer.compactFolders
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (isCompressionEnabled, compressed navigation); V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (CompressedNavigationController); V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (previousCompressedFolder, nextCompressedFolder, firstCompressedFolder, lastCompressedFolder, explorer.compactFolders)
- Pieces:
  1. A folder that holds only one subfolder shows on a single row as a joined path such as `src/main/java`.
  2. With a joined row focused, Left and Right move focus to the previous or next part of the path (`previousCompressedFolder`, `nextCompressedFolder`).
  3. With a joined row focused, Home and End jump to the first or last part of the path (`firstCompressedFolder`, `lastCompressedFolder`).
  4. Setting `explorer.compactFolders` turns joining on or off, with on as the default, and with it off every folder gets its own row.
  5. The setting is ignored when a screen reader is in use, so folders show on separate rows.

#### SB-2.22 File nesting

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Related files group under one parent file, for example a .ts file with its .js and .d.ts files, and the parent gets a twisty. A setting turns nesting on (it is off by default), another decides whether nests start open, and a pattern list says which files nest under which.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.fileNesting.enabled; explorer.fileNesting.expand; explorer.fileNesting.patterns
- Source: V\src\vs\workbench\contrib\files\common\explorerFileNestingTrie.ts; V\src\vs\workbench\contrib\files\common\explorerModel.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts; V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (explorer.fileNesting.enabled, explorer.fileNesting.expand, explorer.fileNesting.patterns with the same default patterns)
- Pieces:
  1. With file nesting on, related files group under one parent file, for example `package.json` with `package-lock.json` and `yarn.lock` beneath it.
  2. A parent file row gets a twisty, and its nested files show indented under it when it is open.
  3. Clicking a nesting parent's name opens the file, and only its twisty opens or closes the nest.
  4. Setting `explorer.fileNesting.enabled` turns nesting on or off, with off as the default.
  5. Setting `explorer.fileNesting.expand` chooses whether nests start open, with on as the default.
  6. Setting `explorer.fileNesting.patterns` lists which files nest under which parent, as parent patterns with comma separated child patterns.

#### SB-2.23 Sort order in the tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Choose how files and folders sort: by name with folders first (default), mixed, files first, by file type, by last modified, or folders first with nesting parents ahead of other files. Also reverse the order and pick how upper and lower case names sort.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.sortOrder; explorer.sortOrderLexicographicOptions; explorer.sortOrderReverse
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FileSorter); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (explorer.sortOrder, explorer.sortOrderLexicographicOptions, explorer.sortOrderReverse)
- Pieces:
  1. By default the tree sorts items by name with folders listed before files.
  2. Setting `explorer.sortOrder` chooses the order, with values `default`, `mixed`, `filesFirst`, `type`, `modified` and `foldersNestsFiles`.
  3. With `mixed` files and folders sort together by name, and with `filesFirst` files are listed before folders.
  4. With `type` items group by extension and then by name with folders first, and with `modified` they sort by last modified date, newest first, with folders first.
  5. With `foldersNestsFiles` folders come first and files that have nested children come before other files.
  6. Setting `explorer.sortOrderLexicographicOptions` chooses how upper and lower case names sort, with values `default` (mixed together), `upper` (uppercase first), `lower` (lowercase first) and `unicode` (Unicode order).
  7. Setting `explorer.sortOrderReverse` reverses the sort order, with off as the default.

#### SB-2.24 Hide files from the tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Files and folders that match the files.exclude patterns (by default .git, .svn, .hg, .DS_Store and Thumbs.db) do not show in the tree, and a setting also hides what .gitignore lists. A hidden file you open in an editor shows up again, dimmed. Both are edited in the Settings editor (ED-8).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.exclude; explorer.excludeGitIgnore
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FilesFilter); V\src\vs\workbench\contrib\files\browser\views\explorerDecorationsProvider.ts (excluded items dimmed); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (files.exclude with the same default patterns, explorer.excludeGitIgnore)
- Pieces:
  1. Files and folders that match the `files.exclude` patterns do not show in the tree, and the default patterns hide `.git`, `.svn`, `.hg`, `.DS_Store` and `Thumbs.db`.
  2. Setting `explorer.excludeGitIgnore` also hides everything that `.gitignore` lists, with off as the default.
  3. A hidden file that is open in an editor shows in the tree again with a dimmed name.

#### SB-2.25 File decorations in the tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Rows take on colors and letter badges from other features: Git status such as M and U, and error or warning marks from Problems. The Explorer adds its own: a bent arrow on symbolic links, a question mark on unknown file types, a red exclamation on a root folder that cannot be read, and dimmed names for hidden files that are open. Two settings turn colors and badges off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.decorations.colors; explorer.decorations.badges; git.decorations.enabled; problems.decorations.enabled
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerDecorationsProvider.ts; V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (fileDecorations); V\src\vs\workbench\contrib\markers\browser\markersFileDecorations.ts; V\src\vs\workbench\services\decorations\browser\decorationsService.ts; V\extensions\git\src\decorationProvider.ts; V\extensions\git\package.json; C bundle (explorer.decorations.colors, explorer.decorations.badges, problems.decorations.enabled, strings Symbolic Link, Unknown File Type, Unable to resolve workspace folder); C\extensions\git\package.json (git.decorations.enabled)
- Pieces:
  1. Rows take on a color and a letter badge from Git status, such as `M` for modified and `U` for untracked.
  2. Rows take on error and warning colors and marks from Problems.
  3. A symbolic link shows a bent arrow badge with the status text Symbolic Link.
  4. A file of an unknown type shows a question mark badge.
  5. A root folder that cannot be read shows a red exclamation mark.
  6. A file hidden by the exclude patterns but open in an editor shows with a dimmed name.
  7. Settings `explorer.decorations.colors` and `explorer.decorations.badges` turn decoration colors and badges off, with both on as the default.
  8. Settings `git.decorations.enabled` and `problems.decorations.enabled` turn the Git marks and the Problems marks on or off.

#### SB-2.26 Find in the tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+Alt+F or F3 with the tree focused to open the find box, type to highlight matches or to filter the tree down to them, switch between fuzzy and contiguous matching, and press Esc to close it. The search also reaches folders that were never opened, and it reads the search excludes.
- Commands: list.find; list.closeFind; list.toggleFindMode; list.toggleFindMatchType
- Menus: none found
- Keys: Ctrl+Alt+F or F3 (list.find, list that supports find is focused); Escape (list.closeFind, tree find box open)
- Settings: workbench.list.defaultFindMode; workbench.list.defaultFindMatchType; search.exclude; explorer.excludeGitIgnore
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (ExplorerFindProvider, searchInWorkspace); V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (findProvider); V\src\vs\workbench\contrib\files\common\files.ts (ExplorerFindProviderActive); V\src\vs\workbench\browser\actions\listCommands.ts; V\src\vs\platform\list\browser\listService.ts; C bundle (list.find, list.closeFind, list.toggleFindMode, list.toggleFindMatchType, workbench.list.defaultFindMode, explorerFindProviderActive, ExplorerFindProvider)
- Pieces:
  1. Pressing `Ctrl+Alt+F` or F3 with the tree focused opens the find box over the tree (`list.find`).
  2. In highlight mode, typing in the find box highlights the matching rows and the up and down keys visit only those matches.
  3. In filter mode, typing in the find box hides every row that does not match, and `list.toggleFindMode` switches between highlight and filter.
  4. Setting `workbench.list.defaultFindMode` chooses the starting mode, with values `highlight` (the default) and `filter`.
  5. Setting `workbench.list.defaultFindMatchType` chooses fuzzy or contiguous matching, with `fuzzy` as the default, and `list.toggleFindMatchType` switches between them.
  6. The search also finds items inside folders that were never opened, and it skips what `search.exclude` and `explorer.excludeGitIgnore` hide.
  7. A folder that holds matches shows a count badge with the hover text "Directory contains N matches".
  8. When there are too many matches the find box warns that the result set only contains a subset of all matches.
  9. Pressing Escape closes the find box (`list.closeFind`).

#### SB-2.27 Type to jump in the tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Start typing a name with the tree focused to jump to the matching row. A setting picks whether typing starts the jump at any time (automatic) or only after you run the trigger command.
- Commands: list.triggerTypeNavigation
- Menus: none found
- Keys: none found
- Settings: workbench.list.typeNavigationMode
- Source: V\src\vs\workbench\browser\actions\listCommands.ts; V\src\vs\platform\list\browser\listService.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (keyboardNavigationLabelProvider); C bundle (list.triggerTypeNavigation, workbench.list.typeNavigationMode with default automatic)
- Pieces:
  1. Typing a name with the tree focused jumps focus to the row whose name matches what was typed.
  2. Setting `workbench.list.typeNavigationMode` chooses when typing jumps, with values `automatic` (the default) and `trigger`.
  3. With `trigger`, typing jumps only after the `list.triggerTypeNavigation` command has been run.

#### SB-2.29 Tree indent and indent guides

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Set how far each folder level is indented (4 to 40 pixels, default 8) and when the vertical indent guide lines show (never, on hover, or always; default on hover). The same two settings apply to every tree in the workbench.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.tree.indent; workbench.tree.renderIndentGuides
- Source: V\src\vs\platform\list\browser\listService.ts; V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FilesRenderer indent offset); C bundle (workbench.tree.indent, workbench.tree.renderIndentGuides with the same defaults)
- Pieces:
  1. Setting `workbench.tree.indent` sets how many pixels each folder level is indented, from 4 to 40, with 8 as the default.
  2. Setting `workbench.tree.renderIndentGuides` chooses when vertical indent guide lines show, with values `none`, `onHover` and `always`.
  3. With the default `onHover`, the guide lines show while the pointer is over the tree.
  4. Both settings apply to every tree in the workbench and not only the Explorer.

#### SB-2.30 Open Editors list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Open Editors pane lists every open editor, with group headings when there is more than one editor group; each heading has Save All in Group and Close Group buttons. Rows show the file icon, an italic name for a preview editor, a pin or close button that turns into a dot for unsaved changes, and click switches while middle-click closes. The pane starts hidden (show it from the Explorer Views menu, SB-1) and Ctrl+K E focuses it.
- Commands: workbench.files.action.focusOpenEditorsView; workbench.files.action.saveAllInGroup; workbench.files.action.closeGroup; workbench.action.closeActiveEditor; workbench.action.unpinActiveEditor
- Menus: none found
- Keys: Ctrl+K E (workbench.files.action.focusOpenEditorsView; macOS Cmd+K E)
- Settings: explorer.openEditors.visible; explorer.openEditors.minVisible
- Source: V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts; V\src\vs\workbench\contrib\files\browser\explorerViewlet.ts (createOpenEditorsViewDescriptor, hideByDefault); V\src\vs\workbench\contrib\files\common\files.ts (OpenEditor); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (descriptor for workbench.explorer.openEditorsView with hideByDefault true, workbench.files.action.focusOpenEditorsView, explorer.openEditors.visible, explorer.openEditors.minVisible)
- Pieces:
  1. The Open Editors pane lists every open editor as one row.
  2. When there is more than one editor group, each group gets a heading row with the editors listed beneath it.
  3. Each group heading has a Save All in Group button and a Close Group button.
  4. Each row shows the file icon and name, and the name is italic for a preview editor.
  5. Each row has a close button, a pinned editor shows an unpin button in its place, and a row with unsaved changes shows a dot in place of the button.
  6. Clicking a row switches to that editor, and middle-clicking a row closes it.
  7. The pane starts hidden and is shown from the Explorer Views menu (SB-1), and `Ctrl+K E` (`Cmd+K E` on macOS) focuses it.
  8. Setting `explorer.openEditors.visible` sets the most editors shown before a scroll bar appears, with 9 as the default.
  9. Setting `explorer.openEditors.minVisible` sets how many editor slots the pane keeps free, with 0 as the default, which lets the pane resize to the number of editors.

#### SB-2.31 Open Editors header buttons

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Buttons in the Open Editors title bar: Save All, Close All Editors, New Untitled Text File, and Toggle Vertical/Horizontal Editor Layout (the last shows only when there is more than one editor group).
- Commands: workbench.action.files.saveAll; openEditors.closeAll; openEditors.newUntitledFile; workbench.action.toggleEditorGroupLayout
- Menus: ViewTitle (view == workbench.explorer.openEditorsView); MenubarLayoutMenu
- Keys: Shift+Alt+0 (workbench.action.toggleEditorGroupLayout; macOS Cmd+Alt+0)
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (registerAction2 blocks); C bundle (workbench.action.files.saveAll, openEditors.closeAll, openEditors.newUntitledFile, workbench.action.toggleEditorGroupLayout, label Toggle Vertical/Horizontal Editor Layout)
- Pieces:
  1. Clicking the Save All button in the Open Editors title bar saves every editor with unsaved changes (`workbench.action.files.saveAll`).
  2. Clicking the Close All Editors button closes every open editor (`openEditors.closeAll`).
  3. Clicking the New Untitled Text File button opens a new untitled text file (`openEditors.newUntitledFile`).
  4. The Toggle Vertical/Horizontal Editor Layout button shows only when there is more than one editor group and switches the groups between side by side and stacked.
  5. Pressing `Shift+Alt+0` (`Cmd+Alt+0` on macOS) or picking the entry in the Layout menu does the same as the layout button (`workbench.action.toggleEditorGroupLayout`).

#### SB-2.32 Open Editors right-click menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click an editor row for Open to the Side, Reveal in File Explorer, Reopen Editor With..., Copy Path, Copy Relative Path, Save, Revert File, Save All (on a group heading), Compare with Saved, the three compare entries, and Close, Close Others, Close Saved, Close All. Entries show or hide by editor type, unsaved state, read-only state and auto save. Cursor adds nothing to this menu.
- Commands: explorer.openToSide; revealFileInOS; workbench.action.reopenWithEditor; copyFilePath; copyRelativeFilePath; workbench.action.files.save; workbench.action.files.revert; workbench.files.action.saveAllInGroup; workbench.files.action.compareWithSaved; selectForCompare; compareFiles; compareSelected; workbench.action.closeActiveEditor; workbench.action.closeOtherEditors; workbench.action.closeUnmodifiedEditors; workbench.action.closeEditorsInGroup
- Menus: OpenEditorsContext
- Keys: none found
- Settings: files.autoSave
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\electron-browser\fileActions.contribution.ts (Reveal entry); V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (onListContextMenu); C bundle (OpenEditorsContext items in groups 1_open, 2_save, 3_compare, 4_close, and the ids above)
- Pieces:
  1. Right-clicking an editor row shows Open to the Side and Reveal in File Explorer, and Open to the Side is offered only for file and untitled editors.
  2. Reopen Editor With... shows for editors that have other editors to choose from, and not on a group heading.
  3. Copy Path and Copy Relative Path show for editors whose resource is a file on disk.
  4. Save and Revert File are greyed out unless the editor has unsaved changes, and Revert File is hidden for read-only editors.
  5. Compare with Saved shows for file editors with unsaved changes and is hidden while `files.autoSave` saves after a short delay.
  6. Select for Compare and Compare with Selected show for file and untitled editors, and Compare Selected shows when two rows are selected.
  7. Close, Close Others, Close Saved and Close All end the menu, and Close and Close Others are not on a group heading.
  8. Save All shows only when the right-click lands on a group heading, and it is greyed out when nothing is unsaved.

#### SB-2.33 Sort the Open Editors list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: List editors in tab order (default), alphabetically by tab name inside each group, or alphabetically by full path. Dragging rows to reorder is switched off unless the order is tab order.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.openEditors.sortOrder
- Source: V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (getElements, OpenEditorsDragAndDrop); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (explorer.openEditors.sortOrder)
- Pieces:
  1. Setting `explorer.openEditors.sortOrder` chooses how the Open Editors list is ordered, with values `editorOrder`, `alphabetical` and `fullPath`.
  2. With `editorOrder`, the default, rows follow the order of the editor tabs.
  3. With `alphabetical`, rows are sorted by tab name inside each editor group.
  4. With `fullPath`, rows are sorted by full path inside each editor group.
  5. Dragging rows to reorder them is turned off unless the order is `editorOrder`.

#### SB-2.34 Unsaved count badge in Open Editors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A badge in the Open Editors title shows how many files have unsaved changes (for example 2 unsaved) and hides itself when there are none.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.autoSave
- Source: V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (updateDirtyIndicator); C bundle (string {0} unsaved)
- Pieces:
  1. A badge in the Open Editors title shows how many files have unsaved changes, with text such as "2 unsaved".
  2. The badge hides itself when no file has unsaved changes.
  3. With `files.autoSave` set to save after a short delay, files that will be saved on their own are not counted in the badge.

#### SB-2.35 Reorder editors by dragging in Open Editors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Drag rows to reorder editors or move them into another editor group (group headings are drop targets), drop files from the operating system to open them, and drag a row out to the editor area. Reordering works only when the list is in tab order.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.openEditors.sortOrder
- Source: V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (OpenEditorsDragAndDrop); C bundle (the same drag over logic: sort order check for editorOrder, feedback index, native file drop check)
- Pieces:
  1. Dragging an editor row up or down inside a group reorders the editors when the list is in tab order.
  2. Dragging an editor row onto another group, or onto its heading, moves that editor into that group.
  3. Dropping files from the operating system onto the list opens them in editors.
  4. Dragging a row out to the editor area opens that editor there.
  5. Dragging a row to reorder is refused when `explorer.openEditors.sortOrder` is not `editorOrder`.

#### SB-2.36 No Folder Opened welcome

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With no folder or workspace open, the Explorer shows a No Folder Opened view with welcome text and buttons: Open Folder with Open Recent, or an add-a-folder link when a workspace file is open, or a remote variant. Five wordings are chosen by state, and when editors are open the text warns that opening a folder closes them and offers adding a folder instead. Dropping a folder or file onto the view opens it.
- Commands: workbench.action.files.openFolder; workbench.action.files.openFolderViaWorkspace; workbench.action.addRootFolder; workbench.action.openRecent
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\explorerViewlet.ts (registerViewWelcomeContent for EmptyView.ID); V\src\vs\workbench\contrib\files\browser\views\emptyView.ts (drop handler); V\src\vs\workbench\browser\actions\workspaceActions.ts; C bundle (view id workbench.explorer.emptyView, title No Folder Opened, strings You have not yet opened a folder and Opening a folder will close all currently open editors, and the four command ids)
- Pieces:
  1. With no folder or workspace open, the Explorer shows a No Folder Opened view with the text "You have not yet opened a folder." and an Open Folder button.
  2. Clicking the Open Folder button opens the folder picker (`workbench.action.files.openFolder`).
  3. When editors are open, the text adds "Opening a folder will close all currently open editors. To keep them open, add a folder instead." and the add a folder link runs `workbench.action.addRootFolder`.
  4. When a workspace file with no folders is open, the text reads "You have not yet added a folder to the workspace." with a button that adds a folder.
  5. In a remote window with no folder open, the text reads "Connected to remote." with an Open Folder button.
  6. When a workspace file is open and entering workspaces is not supported, the text offers Open Folder (`workbench.action.files.openFolderViaWorkspace`) and Open Recent (`workbench.action.openRecent`) buttons.
  7. Dropping a folder or file onto the view opens it.

#### SB-2.37 Clone Repository text in the empty Explorer

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When Git is on and no source control provider is registered, the empty Explorer adds the line "You can clone a repository locally" with a Clone Repository button, then a Learn more line about Git and source control, below the Open Folder text. The wording is the same in Cursor.
- Commands: git.clone
- Menus: none found
- Keys: none found
- Settings: git.enabled
- Source: V\extensions\git\package.json (viewsWelcome entries for view explorer); V\extensions\git\package.nls.json; C\extensions\git\package.json (same two entries); C\extensions\git\package.nls.json (same text); C bundle (git.clone)
- Pieces:
  1. Below the Open Folder text, the empty Explorer shows the line "You can clone a repository locally" with a Clone Repository button.
  2. Clicking Clone Repository starts the clone flow (`git.clone`).
  3. A Learn more line about Git and source control follows the clone button.
  4. The clone text shows only when `git.enabled` is on and no source control provider is registered.

#### SB-2.38 Open a file in the app's browser

- Tag: both. Evidence: source. Boundary: yes. Cursor: changed (own id filesExplorer.openFileInBrowser, wider file list, Explorer menu only)
- What: Right-click a local file and pick Open in Browser (Cursor, for html, svg, pdf, text, Markdown, JSON, XML, YAML, CSV, log, CSS, script and image files) or Open in Integrated Browser (stock, for .html and .htm only) to open it in a browser tab inside the app. Boundary yes: it opens the Cursor Browser tab, which the catalog treats as agent-driven (ED-10).
- Commands: workbench.action.browser.openFile; filesExplorer.openFileInBrowser; workbench.action.openBrowserEditor
- Menus: ExplorerContext; EditorTitleContext; EditorTitle
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\browserView\electron-browser\features\browserTabManagementFeatures.ts (OpenFileInIntegratedBrowserAction); V\src\vs\platform\browserView\common\browserView.ts (OpenFile id); C bundle (filesExplorer.openFileInBrowser, NLS 7047 Open in Browser, the extension regex, the call to workbench.action.openBrowserEditor with trackingId context_menu_new_tab)
- Pieces:
  1. Right-clicking a local file and picking Open in Browser opens it in a Browser tab inside the app, and the entry is offered for html, svg, pdf, text, Markdown, JSON, XML, YAML, CSV, log, CSS, script and image files (`filesExplorer.openFileInBrowser`). [cursor only]
  2. The Open in Browser entry is in the Explorer right-click menu only, with no entry in the editor tab menu or the editor title bar. [cursor only]
  3. Right-clicking a local `.html` or `.htm` file and picking Open in Integrated Browser opens it in a browser tab inside the app (`workbench.action.browser.openFile`). [stock only]
  4. The Open in Integrated Browser entry also shows in the editor tab right-click menu and as a globe button in the editor title bar for local `.html` and `.htm` files. [stock only]

#### SB-2.39 Cursor Blame for a file

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: Right-click a file and pick Cursor Blame to open that file's blame view (command title Open File Blame). The entry is not shown for folders or workspace roots, and it appears only when the cursor_blame feature gate and a team admin setting are both on. Boundary yes: the blame data comes from Cursor conversation tracking, not from Git alone.
- Commands: cursorBlame.openFileBlame
- Menus: ExplorerContext; CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (cursorBlame.openFileBlame, NLS 5375 Open File Blame, NLS 5376 Cursor Blame, menu label NLS 5384, ExplorerContext group 12_cursor_blame order 1, gate cursor_blame with isCursorBlameEnabledForTeam, classifyConversations)
- Pieces:
  1. Right-clicking a file and picking Cursor Blame opens that file's blame view (`cursorBlame.openFileBlame`). [cursor only]
  2. The Command Palette lists Open File Blame (`cursorBlame.openFileBlame`). [cursor only]
  3. The Cursor Blame entry is not shown on folders or workspace roots. [cursor only]
  4. The Cursor Blame entry shows only when the `cursor_blame` feature gate and a team admin setting are both on. [cursor only]

#### SB-2.40 Open in Images Preview

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: Right-click a folder, or an image or video file, and pick Open in Images Preview to browse the images of that folder in the Images Preview viewer. It is marked experimental in stock and on by default. Cursor has none of its ids, setting or label. The viewer itself is ED-10.
- Commands: workbench.action.openImagesInCarousel
- Menus: ExplorerContext
- Keys: none found
- Settings: imageCarousel.explorerContextMenu.enabled
- Source: V\src\vs\workbench\contrib\imageCarousel\browser\imageCarousel.contribution.ts; V\src\vs\workbench\workbench.common.main.ts; C bundle (searched the command id, the setting id, the label Open in Images Preview and the module name imageCarousel, 0 hits each)
- Pieces:
  1. Right-clicking a folder, or an image or video file, and picking Open in Images Preview opens the images of that folder in the Images Preview viewer (`workbench.action.openImagesInCarousel`). [stock only]
  2. Setting `imageCarousel.explorerContextMenu.enabled` turns the Open in Images Preview entry on or off, with on as the default, and is tagged experimental. [stock only]
  3. The Explorer right-click menu in Cursor has no Open in Images Preview entry and no setting for it. [stock only]

#### SB-2.41 Select several files and folders at once

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Hold Ctrl (Cmd on macOS) and click to add or remove one row, hold Shift and click to select a range, or press Shift+Up and Shift+Down to extend the selection from the keyboard. Ctrl+A selects the visible rows under the focused row's parent folder, Ctrl+Shift+Enter adds or removes the focused row, and Esc clears the selection. Delete, Cut, Copy, Compare Selected, Open to the Side and dragging then act on every selected row, while Rename and the New File entries need one row. A setting swaps the click modifier to Alt (Option on macOS), and its text says the Open to the Side mouse gestures adapt so they do not clash.
- Commands: list.expandSelectionDown; list.expandSelectionUp; list.selectAll; list.toggleSelection; list.clear
- Menus: none found
- Keys: Shift+Down (list.expandSelectionDown, list supports multi-select); Shift+Up (list.expandSelectionUp); Ctrl+A (list.selectAll, list supports multi-select); Ctrl+Shift+Enter (list.toggleSelection); Escape (list.clear, when something is selected or focused)
- Settings: workbench.list.multiSelectModifier
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (multipleSelectionSupport true, getContext respects multi selection); V\src\vs\workbench\browser\actions\listCommands.ts (list.expandSelectionDown, list.expandSelectionUp, list.selectAll, list.toggleSelection, list.clear); V\src\vs\platform\list\browser\listService.ts (multiSelectModifier setting and the Alt switch); C bundle (all five list command ids and workbench.list.multiSelectModifier with the same description text, NLS 2127)
- Pieces:
  1. Holding `Ctrl` (`Cmd` on macOS) and clicking a row adds it to or removes it from the selection.
  2. Holding Shift and clicking a row selects the whole range from the last selected row.
  3. Pressing `Shift+Down` or `Shift+Up` extends the selection by one row (`list.expandSelectionDown`, `list.expandSelectionUp`).
  4. Pressing `Ctrl+A` selects the visible rows under the focused row's parent folder (`list.selectAll`).
  5. Pressing `Ctrl+Shift+Enter` adds the focused row to the selection or removes it (`list.toggleSelection`).
  6. Pressing Escape clears the selection (`list.clear`).
  7. Delete, Cut, Copy, Compare Selected, Open to the Side and dragging act on every selected row.
  8. Rename and the New File entries need exactly one row and are not offered when several are selected.
  9. Setting `workbench.list.multiSelectModifier` chooses the click modifier for adding rows, with values `ctrlCmd` (the default) and `alt`, and with `alt` the Open to the Side mouse gestures change so they do not clash.

#### SB-2.42 Scrolling options for the tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Five settings shape how the tree scrolls: sideways scrolling for long names (off by default, and the Explorer switches it off while a name box is open), smooth scrolling (off), clicking the scrollbar to scroll a page at a time (off; with it on, double-clicking blank space no longer makes a new file), the mouse wheel speed (1) and the Alt-held fast scroll speed (5). Ctrl+Up and Ctrl+Down nudge the view by 10 pixels without moving the focused row (the key only fires when the view is not already at the top or bottom). Scroll Left and Scroll Right have no default keys. The settings apply to every tree and list, and SB-1.23 also names the sideways scrolling setting.
- Commands: list.scrollUp; list.scrollDown; list.scrollLeft; list.scrollRight
- Menus: none found
- Keys: Ctrl+Up (list.scrollUp, list focused and not scrolled to the top); Ctrl+Down (list.scrollDown, list focused and not scrolled to the bottom)
- Settings: workbench.list.horizontalScrolling; workbench.list.smoothScrolling; workbench.list.scrollByPage; workbench.list.mouseWheelScrollSensitivity; workbench.list.fastScrollSensitivity
- Source: V\src\vs\platform\list\browser\listService.ts (the five setting definitions); V\src\vs\workbench\browser\actions\listCommands.ts (list.scrollUp, list.scrollDown, list.scrollLeft, list.scrollRight); V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (setEditable turns horizontalScrolling off, onMouseDblClick checks scrollByPage); C bundle (the five setting ids, list.scrollUp with the same top-of-list condition, list.scrollDown, list.scrollLeft, list.scrollRight)
- Pieces:
  1. Setting `workbench.list.horizontalScrolling` lets long names scroll sideways, with off as the default, and the Explorer turns it off while a name box is open.
  2. Setting `workbench.list.smoothScrolling` makes the tree scroll smoothly, with off as the default.
  3. Setting `workbench.list.scrollByPage` makes a click in the scroll bar scroll a page at a time, with off as the default, and with it on a double click on blank space no longer starts a new file.
  4. Setting `workbench.list.mouseWheelScrollSensitivity` sets the mouse wheel speed multiplier, with 1 as the default.
  5. Setting `workbench.list.fastScrollSensitivity` sets the speed multiplier while Alt is held, with 5 as the default.
  6. Pressing `Ctrl+Up` or `Ctrl+Down` nudges the view by 10 pixels without moving the focused row, and only when the view is not already at the top or bottom (`list.scrollUp`, `list.scrollDown`).
  7. Scroll Left and Scroll Right move the view sideways and have no default keys (`list.scrollLeft`, `list.scrollRight`).
  8. The five scrolling settings apply to every tree and list in the workbench and not only the Explorer.

#### SB-2.43 Hover a row for its path and status

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Rest the pointer on a row in the tree to see a tooltip with the full path, followed by a dot and the status text from any decoration on it (for example a Git state or Symbolic Link). Press Ctrl+K Ctrl+I with a row focused to show the same tooltip from the keyboard. A setting sets the delay before it appears.
- Commands: list.showHover
- Menus: none found
- Keys: Ctrl+K Ctrl+I (list.showHover, list focused)
- Settings: workbench.hover.delay
- Source: V\src\vs\workbench\browser\labels.ts (title built from the path label, decoration tooltip appended after a dot); V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FilesRenderer.renderStat passes the decorations and no title); V\src\vs\workbench\browser\actions\listCommands.ts (list.showHover); V\src\vs\workbench\browser\workbench.contribution.ts (workbench.hover.delay); C bundle (list.showHover, workbench.hover.delay). Nothing was run on screen, so the tooltip wording comes from source.
- Pieces:
  1. Resting the pointer on a row shows a tooltip with the full path of that file or folder.
  2. The tooltip adds a dot and then the status text of any decoration on the row, such as a Git state or Symbolic Link.
  3. Pressing `Ctrl+K Ctrl+I` with a row focused shows the same tooltip from the keyboard (`list.showHover`).
  4. Setting `workbench.hover.delay` sets the delay in milliseconds before the tooltip appears, with 500 as the default (1500 on macOS).

#### SB-2.44 Folders header shows the workspace name

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The header of the Folders pane reads the opened folder's name, or the workspace file's name followed by (Workspace), or Untitled (Workspace) for an unsaved workspace, and a remote suffix when the folder is remote. It updates when the workspace is renamed, and a screen reader hears it as Explorer Section followed by the name. Dragging something over the header while the pane is folded opens it after a short wait. When Folders is the only pane showing, the container title carries it instead (SB-1.20).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (name getter, renderHeader with setHeader and DelayedDragHandler); V\src\vs\workbench\services\label\common\labelService.ts (getWorkspaceLabel, Untitled (Workspace), {0} (Workspace)); C bundle (the same renderHeader code with NLS 7353 Explorer Section: {0} and the drag handler; NLS 2751 and 2752 for the two workspace labels)
- Pieces:
  1. The Folders pane header reads the name of the opened folder.
  2. For an open workspace file, the header reads the workspace file's name followed by (Workspace).
  3. For an unsaved workspace, the header reads Untitled (Workspace).
  4. For a remote folder, the header adds a remote suffix to the name.
  5. The header updates when the workspace is renamed.
  6. A screen reader announces the header as Explorer Section followed by the name.
  7. Dragging something over the header while the pane is folded opens the pane after a short wait.
  8. When Folders is the only pane showing, the container title carries the name instead (SB-1.20).

#### SB-2.45 The tree remembers which folders were open

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Close the window and open the same workspace again and the Explorer shows the same folders unfolded, with the same row selected and the same scroll position (auto reveal of the active file, SB-2.14, can then move it). In a multi-root workspace with no saved state, up to five roots open on their own, and a root you add opens when it appears. There is no setting and no button for it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (TREE_VIEW_STATE_STORAGE_KEY workbench.explorer.treeViewState, storeTreeViewState, setTreeInput view state and root expansion); C bundle (TREE_VIEW_STATE_STORAGE_KEY with the same key, storeTreeViewState, getViewState)
- Pieces:
  1. Closing the window and opening the same workspace again shows the same folders unfolded as before.
  2. The reopened tree has the same row selected and the same scroll position.
  3. In a multi-root workspace with no saved tree state, up to five roots open on their own.
  4. A root folder added to the workspace opens when it appears.

#### SB-2.46 The tree follows changes on disk

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Files that are created, deleted or renamed outside the app show up in the tree on their own, after a short delay, and the tree also refreshes when the window gets focus again. It holds off while a name box is open. Every workspace folder is watched except the paths the exclude setting names, and an include setting adds extra paths such as symbolic-link folders. Changing either setting restarts watching. If the system cannot watch, or the watcher dies, a warning notification appears (NT-2 is the closest owner for the notification itself).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.watcherExclude; files.watcherInclude
- Source: V\src\vs\workbench\contrib\files\browser\explorerService.ts (onDidFilesChange with the reaction delay, refresh on window focus, no refresh while editable); V\src\vs\workbench\contrib\files\browser\workspaceWatcher.ts (watchWorkspace, setting listeners, the two warning prompts); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (the two settings and the default exclude list); C bundle (files.watcherExclude, files.watcherInclude, NLS 7384 and 7386 for the two warnings)
- Pieces:
  1. Files created, deleted or renamed outside the app show up in the tree on their own after a short delay.
  2. The tree also refreshes when the window gets focus again.
  3. The tree holds off refreshing while a name box is open.
  4. Every workspace folder is watched except the paths named in `files.watcherExclude`.
  5. Setting `files.watcherInclude` adds extra paths to watch, such as symbolic link folders.
  6. Changing `files.watcherExclude` or `files.watcherInclude` restarts watching without reloading the window.
  7. When the system cannot watch, a warning notification says "Unable to watch for file changes" and has an Instructions button (NT-2 owns the notification).
  8. When the watcher dies, a warning notification says "File changes watcher stopped unexpectedly" and has a Reload button (NT-2 owns the notification).

#### SB-2.47 Paste text to make a new file

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: With a row focused in the tree and plain text on the clipboard, Ctrl+V makes a new file named Untitled (Untitled 1, Untitled 2 and so on when the name is taken) holding that text, in the focused folder, the folder of the focused file, or the first root when nothing is focused. The file opens as a pinned tab without taking focus, and its name goes straight into the rename box so you can type the real name. It does nothing when the clipboard is blank or holds files (those paste as in SB-2.9), when a name box is already open, or when the focused row is read-only, and a read-only folder gives the error "Parent folder is readonly." Stock VS Code has no such command.
- Commands: explorer.createFileFromPaste
- Menus: none found
- Keys: Ctrl+V (browser paste event while the tree has focus, no command key rule)
- Settings: none found
- Source: C bundle (command registration explorer.createFileFromPaste and its handler writing the Untitled file, the paste listener on the Explorer view that calls it when clipboardData has text and no files, NLS 7135 Parent folder is readonly.); V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (stock paste listener only forwards files to filesExplorer.paste; 0 hits for createFileFromPaste in V and in N)
- Pieces:
  1. With a row focused and plain text on the clipboard, pressing `Ctrl+V` makes a new file named Untitled holding that text. [cursor only]
  2. The new file goes in the focused folder, in the folder of the focused file, or in the first root when no row is focused. [cursor only]
  3. When the name is taken the new file is named Untitled 1, Untitled 2 and so on. [cursor only]
  4. The new file opens as a pinned tab without taking focus, and its name goes straight into the rename box so the person can type the real name. [cursor only]
  5. Nothing happens when the clipboard is blank or holds files (files paste as in SB-2.9), or when a name box is already open. [cursor only]
  6. Nothing happens when the focused row is read-only, and a read-only folder gives the error "Parent folder is readonly.". [cursor only]

#### SB-2.48 Drag an editor tab into the tree to copy its file

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Drag an editor tab onto a folder in the tree to copy that file into the folder (the tree treats the drop as coming from outside, so it copies instead of moving). A name clash asks before it replaces (DL-4 owns the box). The drop is refused when the folder is read-only or when drag and drop is turned off. Rows dragged from the Open Editors list are refused, because the tree turns away drags that start in another list. The reverse direction, tree rows to the editor area, is SB-2.12. Read from the accepted drag types in source, not run.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.enableDragAndDrop
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FileDragAndDrop.handleDragOver accepts Files, code files and ResourceURLs data, drop calls ExternalFileImport); V\src\vs\workbench\contrib\files\browser\fileImportExport.ts (ExternalFileImport.doImport uses extractEditorsAndFilesDropData, importResources); V\src\vs\platform\dnd\browser\dnd.ts (extractEditorsDropData reads editor and resource drag data); V\src\vs\workbench\browser\dnd.ts (fillEditorsDragData sets ResourceURLs and editors data); V\src\vs\base\browser\ui\list\listView.ts (drag data from another list becomes ExternalElementsDragAndDropData, drag data from outside any list becomes NativeDragAndDropData); C bundle (handleDragOver with the same accepted types FILES, code files and RESOURCES)
- Pieces:
  1. Dragging an editor tab onto a folder in the tree copies that file into the folder, and does not move it.
  2. When the folder already holds a file of that name, a box asks before replacing it (DL-4 owns the box).
  3. The drop is refused when the target folder is read-only.
  4. The drop is refused when `explorer.enableDragAndDrop` is off.
  5. Rows dragged from the Open Editors list are refused by the tree.

#### SB-2.49 Run or debug tests under a file or folder

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a file or folder and pick Run Tests, Debug Tests or Run Tests with Coverage to run every test the test providers found under it; results show in the Test Explorer (SB-6). Each entry appears only while some test provider offers that kind of run, so a plain install with no test extension shows none of them. When nothing is found under the item, an info notification says "No tests found in the selected file or folder". SB-6 handed these three entries to this group, and the first pass here had handed them to SB-6, so they are listed here.
- Commands: testing.run.uri; testing.debug.uri; testing.coverage.uri
- Menus: ExplorerContext (group 6.5_testing; each entry needs the matching run capability context key)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (ExecuteTestsUnderUriAction, RunTestsUnderUri, DebugTestsUnderUri, CoverageTestsUnderUri, labels at lines 85 to 87); V\src\vs\workbench\contrib\testing\common\constants.ts (the three ids); C bundle (testing.run.uri, testing.debug.uri, testing.coverage.uri, ExplorerContext group 6.5_testing with the capability key, NLS 11059 and 11066 to 11068)
- Pieces:
  1. Right-clicking a file or folder and picking Run Tests runs every test found under it, with results in the Test Explorer (SB-6) (`testing.run.uri`).
  2. Picking Debug Tests runs the tests under the item in debug mode (`testing.debug.uri`).
  3. Picking Run Tests with Coverage runs the tests under the item and collects coverage (`testing.coverage.uri`).
  4. Each of the three entries shows only while some test provider offers that kind of run, so an install with no test extension shows none of them.
  5. When no tests are found under the item, an info notification says "No tests found in the selected file or folder".


## SB-3 Search

(both). Find and replace across files with include and exclude filters and a results tree. Examples: the search box with Match Case, Whole Word and Regex toggles; the replace row; Open in Editor (the Search Editor itself is ED-3).

30 features.

#### SB-3.1 Open the Search view and fill the box from the editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+Shift+F, choose Find in Files from the Edit menu, or pick Search from the View menu to open the Search view with the cursor in the search box, pre-filled with the editor selection; the command also takes a JSON argument (query, replace, preserveCase, triggerSearch, filesToInclude, filesToExclude, isRegex, isCaseSensitive, matchWholeWord, useExcludeSettingsAndIgnoreFiles, onlyOpenEditors, showIncludesExcludes) so a keybinding can open a prefilled search.
- Commands: workbench.action.findInFiles; workbench.view.search
- Menus: MenubarEditMenu (group 4_find_global, order 1); MenubarViewMenu (group 3_sidebar, order 1, generated from the container's openCommandActionDescriptor)
- Keys: Ctrl+Shift+F (workbench.action.findInFiles); the same chord on workbench.view.search sits behind a never-matching when clause, so it does nothing
- Settings: search.seedWithNearestWord; search.seedOnFocus; search.globalFindClipboard (macOS only)
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:202-258; V\src\vs\workbench\contrib\search\browser\searchActionsBase.ts:88-150; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:54-82; V\src\vs\workbench\contrib\search\browser\searchView.ts:1224-1262; C bundle (SearchCommandIds enum entries FindInFilesActionId and workbench.view.search container id; Edit menu row listed in the title bar entry point inventory)
- Pieces:
  1. Pressing `Ctrl+Shift+F` (`workbench.action.findInFiles`) opens the Search view with the cursor in the search box.
  2. Edit > Find in Files and View > Search open the Search view too.
  3. The search box is pre-filled with the text selected in the editor.
  4. Setting `search.seedWithNearestWord` (off by default) fills the box with the word nearest the cursor when the editor has no selection.
  5. Setting `search.seedOnFocus` (off by default) updates the query to the editor selection whenever the Search view gets focus by a click or by the `workbench.views.search.focus` command.
  6. A keybinding can pass `workbench.action.findInFiles` a JSON argument (`query`, `replace`, `preserveCase`, `triggerSearch`, `filesToInclude`, `filesToExclude`, `isRegex`, `isCaseSensitive`, `matchWholeWord`, `useExcludeSettingsAndIgnoreFiles`, `onlyOpenEditors`, `showIncludesExcludes`) to open a prefilled search.
  7. Setting `search.globalFindClipboard` (macOS only, off by default) lets the Search view read and write the shared find clipboard.

#### SB-3.2 Type a query and let the search run

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Type in the Search box and results fill in as you type after a short pause, Enter runs the search at once, Esc stops it, and Ctrl+Enter adds a new line so the box can hold a multi-line query.
- Commands: workbench.action.toggleSearchOnType
- Menus: none found
- Keys: Enter (run now); Escape (stop a running search, from the box); Ctrl+Enter (new line in the box, Control+Enter on macOS); PageUp and PageDown (jump to the start or end of the text); Tab (next control)
- Settings: search.searchOnType; search.searchOnTypeDebouncePeriod; search.ripgrep.maxThreads
- Source: V\src\vs\workbench\contrib\search\browser\searchWidget.ts:636-740, 814-826; V\src\vs\workbench\contrib\search\browser\searchView.ts:1589-1601; V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:318-337; V\src\vs\workbench\contrib\search\browser\search.common.contribution.ts:22-34; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:177-181; C bundle (workbench.action.toggleSearchOnType, settings search.searchOnType and search.searchOnTypeDebouncePeriod and search.ripgrep.maxThreads, label "Search: Type Search Term and press Enter to search", label "Toggle Search on Type")
- Pieces:
  1. Typing in the search box runs the search after a short pause, and results fill in as you type.
  2. Pressing Enter in the box runs the search at once.
  3. Pressing Esc in the box stops a running search.
  4. Pressing `Ctrl+Enter` (`Control+Enter` on macOS) adds a new line so the box can hold a multi-line query.
  5. In the box, Page Up and Page Down jump to the start or end of the text, and Tab moves to the next control.
  6. Toggle Search on Type (`workbench.action.toggleSearchOnType`) turns searching as you type on or off.
  7. Setting `search.searchOnType` chooses whether results update as you type, on by default.
  8. Setting `search.searchOnTypeDebouncePeriod` chooses the pause in milliseconds between a typed character and the search starting, with a default of 300.
  9. Setting `search.ripgrep.maxThreads` chooses how many threads the search uses, with 0 (the default) letting the engine decide.

#### SB-3.3 Match Case, Whole Word and Regex toggles

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Three icon toggles at the right edge of the search box make the search case sensitive, whole word only, or a regular expression, and a broken regular expression shows its error under the box instead of running.
- Commands: toggleSearchCaseSensitive; toggleSearchWholeWord; toggleSearchRegex
- Menus: none found
- Keys: Alt+C; Alt+W; Alt+R, all when the Search view is focused (Cmd+Alt+C, Cmd+Alt+W and Cmd+Alt+R on macOS, where the case toggle also needs a file or folder row not to be focused)
- Settings: search.smartCase; search.usePCRE2 (deprecated); search.useRipgrep
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:80-135; V\src\vs\editor\contrib\find\browser\findModel.ts:39-58; V\src\vs\workbench\contrib\search\browser\searchWidget.ts:441-480, 636-650; V\src\vs\workbench\contrib\search\browser\searchFindInput.ts; C bundle (the three toggle ids in the SearchCommandIds enum, labels "Toggle Case Sensitive", "Toggle Whole Word", "Toggle Regex", settings search.smartCase and search.usePCRE2)
- Pieces:
  1. The Match Case toggle at the right edge of the search box (`toggleSearchCaseSensitive`, `Alt+C`) makes the search case sensitive.
  2. The Match Whole Word toggle (`toggleSearchWholeWord`, `Alt+W`) makes the search match whole words only.
  3. The Use Regular Expression toggle (`toggleSearchRegex`, `Alt+R`) makes the search text a regular expression.
  4. A broken regular expression shows its error text under the box and the search does not run.
  5. The three keys work while the Search view is focused, and on macOS they are `Cmd+Alt+C`, `Cmd+Alt+W` and `Cmd+Alt+R`, where the case key also needs a file or folder row not to be focused.
  6. Setting `search.smartCase` (off by default) searches case-insensitively when the pattern is all lowercase and case-sensitively otherwise.
  7. Setting `search.usePCRE2` is deprecated, and advanced regular expression features are now used automatically when the pattern needs them.

#### SB-3.4 Search inside notebooks and pick which cell parts

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While a notebook is open, a filter button joins the toggles in the search box with four choices (Markdown Source, Rendered Markdown, Code Cell Source, Code Cell Output), and matches inside notebook cells open in the notebook editor.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.experimental.closedNotebookRichContentResults
- Source: V\src\vs\workbench\contrib\search\browser\searchFindInput.ts:12-84; V\src\vs\workbench\contrib\search\browser\searchWidget.ts:182-252; V\src\vs\workbench\contrib\notebook\browser\contrib\find\notebookFindReplaceWidget.ts:67-72; V\src\vs\workbench\contrib\search\browser\searchView.ts:2223-2300; V\src\vs\workbench\contrib\search\browser\notebookSearch\notebookSearchContributions.ts; C bundle (labels "Notebook Find Filters", "Markdown Source", "Rendered Markdown", "Code Cell Source", "Code Cell Output", setting search.experimental.closedNotebookRichContentResults)
- Pieces:
  1. While a notebook is open, a filter button (Notebook Find Filters) joins the toggles in the search box.
  2. The filter button's drop-down has four choices - Markdown Source, Rendered Markdown, Code Cell Source and Code Cell Output - that pick which cell parts are searched.
  3. A match inside a notebook cell opens in the notebook editor.
  4. Setting `search.experimental.closedNotebookRichContentResults` (off by default) shows rich content results for closed notebooks after the search is refreshed.

#### SB-3.5 Recall earlier searches and clear the history

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Up and Down in the search, replace, include or exclude box step through earlier entries saved for this workspace, and Clear Search History in the Command Palette wipes all four lists.
- Commands: history.showPrevious; history.showNext; search.action.clearHistory
- Menus: none found
- Keys: Up Arrow and Alt+Up Arrow (history.showPrevious); Down Arrow and Alt+Down Arrow (history.showNext); both only while a history box is focused and there is an entry to move to
- Settings: none found
- Source: V\src\vs\platform\history\browser\contextScopedHistoryWidget.ts:110-140; V\src\vs\workbench\contrib\search\common\searchHistoryService.ts:22-65 (workspace storage key workbench.search.history); V\src\vs\workbench\contrib\search\browser\searchActionsTopBar.ts:25-40; V\src\vs\workbench\contrib\search\browser\searchView.ts:490-500, 545-575; C bundle (history.showPrevious, history.showNext, search.action.clearHistory, storage key workbench.search.history, label "Clear Search History")
- Pieces:
  1. Up Arrow in the search, replace, include or exclude box shows the previous entry, and Down Arrow shows the next one (`history.showPrevious`, `history.showNext`).
  2. `Alt+Up Arrow` and `Alt+Down Arrow` step through the same entries.
  3. Each of the four boxes keeps its own list, saved for the workspace.
  4. The arrow keys step through history only while a history box is focused and there is an entry to move to.
  5. Clear Search History (`search.action.clearHistory`) in the Command Palette empties all four lists.

#### SB-3.6 Replace row

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The chevron at the left of the search box opens a Replace box under it with a Preserve Case toggle, Ctrl+Shift+H opens the Search view with the Replace box already open, and Esc in the Replace box closes it.
- Commands: workbench.action.replaceInFiles; closeReplaceInFilesWidget; toggleSearchPreserveCase
- Menus: MenubarEditMenu (group 4_find_global, order 2)
- Keys: Ctrl+Shift+H (workbench.action.replaceInFiles); Escape when the Replace box is focused (closeReplaceInFilesWidget); Alt+P for Preserve Case (Cmd+Alt+P on macOS)
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:56-77, 137-155, 394-420; V\src\vs\workbench\contrib\search\browser\searchWidget.ts:421-436, 553-612, 774-803; C bundle (the three ids in the SearchCommandIds enum, labels "Toggle Replace", "Toggle Preserve Case", "Replace in Files", "Close Replace Widget")
- Pieces:
  1. The chevron at the left of the search box (Toggle Replace) opens a Replace box under the search box and closes it again.
  2. Pressing `Ctrl+Shift+H` (`workbench.action.replaceInFiles`) or choosing Edit > Replace in Files opens the Search view with the Replace box already open.
  3. The Replace box has a Preserve Case toggle (`toggleSearchPreserveCase`, `Alt+P`, `Cmd+Alt+P` on macOS) that keeps the capital letters of the matched text in the replacement.
  4. Pressing Esc in the Replace box closes it (`closeReplaceInFilesWidget`).

#### SB-3.7 Replace one match, or all matches in a file or folder

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With a replace term typed and a search run, you replace a single match, every match in one file, or every match in one folder from row buttons, the right-click menu or Ctrl+Shift+1, and the changed files are saved.
- Commands: search.action.replace; search.action.replaceAllInFile; search.action.replaceAllInFolder
- Menus: SearchContext (group search, order 1); SearchActionMenu (group inline, order 1)
- Keys: Ctrl+Shift+1 (all three, when the Replace row is active and the matching row kind is focused); Ctrl+Shift+Enter (Replace All in a file or folder)
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsRemoveReplace.ts:150-255; V\src\vs\workbench\contrib\search\browser\replaceService.ts:112-137; C bundle (search.action.replace, search.action.replaceAllInFile, search.action.replaceAllInFolder, labels "Replace" and "Replace All")
- Pieces:
  1. With a replace term typed and a search run, a Replace button on a match row replaces that one match (`search.action.replace`).
  2. A Replace All button on a file row replaces every match in that file (`search.action.replaceAllInFile`).
  3. A Replace All button on a folder row replaces every match in that folder (`search.action.replaceAllInFolder`).
  4. The right-click menu on a result row has the matching Replace or Replace All entry.
  5. `Ctrl+Shift+1` runs the replace for the focused row kind, and `Ctrl+Shift+Enter` runs Replace All for a focused file or folder row, both only while the Replace row is active.
  6. The files that were changed are saved after the replacement.

#### SB-3.8 Replace All with a confirmation

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Replace All button at the end of the Replace row (Ctrl+Alt+Enter) asks "Replace N occurrences across M files?" in a standard confirm box before it changes anything, shows progress, then reports "Replaced N occurrences across M files" in the view.
- Commands: search.action.replaceAll
- Menus: none found
- Keys: Ctrl+Alt+Enter, when the Replace row is active and the editor find widget is not showing
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchWidget.ts:75-125, 580-585, 853-870; V\src\vs\workbench\contrib\search\browser\searchView.ts:812-931; C bundle (search.action.replaceAll, labels "Replace All (Submit Search to Enable)", "Replace {0} occurrences across {1} files?", "Replaced {0} occurrences across {1} files.")
- Pieces:
  1. The Replace All button at the end of the Replace row is disabled, with the tooltip "Replace All (Submit Search to Enable)", until a search has run, and it goes back to disabled when the search text changes.
  2. Clicking Replace All, or pressing `Ctrl+Alt+Enter` (`search.action.replaceAll`) while the Replace row is active and the editor find widget is not showing, opens a confirm box titled Replace All that asks "Replace N occurrences across M files?" with a Replace button.
  3. Nothing changes until the confirm box is accepted, and declining leaves every file as it was.
  4. After accepting, a progress indicator shows while the files are changed.
  5. When it finishes, the view reports "Replaced N occurrences across M files." with the singular wording when the count is 1.

#### SB-3.9 Preview a replacement as a diff

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With a replace term typed, clicking a match opens a side-by-side "file ↔ file (Replace Preview)" diff showing what the replacement would do, instead of opening the plain file (notebook matches still open in the notebook editor).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.useReplacePreview
- Source: V\src\vs\workbench\contrib\search\browser\searchView.ts:2223-2237; V\src\vs\workbench\contrib\search\browser\replaceService.ts:140-200; V\src\vs\workbench\contrib\search\browser\replaceContributions.ts; C bundle (openReplacePreview, label "{0} ↔ {1} (Replace Preview)", setting search.useReplacePreview)
- Pieces:
  1. With a replace term typed, clicking a match opens a side-by-side diff titled "<file> ↔ <file> (Replace Preview)" that shows what the replacement would do, instead of opening the plain file.
  2. A match inside a notebook still opens in the notebook editor.
  3. Setting `search.useReplacePreview` chooses whether selecting or replacing a match opens Replace Preview, on by default.

#### SB-3.10 Filter by files to include and files to exclude

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The "..." button under the search box (Toggle Search Details) shows two comma-separated glob boxes, files to include and files to exclude, that limit where the search looks, and each box keeps its own history.
- Commands: workbench.action.search.toggleQueryDetails
- Menus: none found
- Keys: Ctrl+Shift+J when the Search view is focused (also works in the Search Editor)
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:31-54; V\src\vs\workbench\contrib\search\browser\searchView.ts:500-600, 1533-1563; V\src\vs\workbench\contrib\search\browser\patternInputWidget.ts:33-184; C bundle (workbench.action.search.toggleQueryDetails, labels "Toggle Search Details", "files to include", "files to exclude", placeholders "e.g. *.ts, src/**/include" and "e.g. *.ts, src/**/exclude")
- Pieces:
  1. The "..." button under the search box (Toggle Search Details, `workbench.action.search.toggleQueryDetails`) shows or hides the include and exclude boxes.
  2. The "files to include" and "files to exclude" boxes take comma-separated globs (for example `*.ts, src/**/include`) that limit where the search looks.
  3. Each of the two boxes keeps its own history.
  4. Pressing `Ctrl+Shift+J` while the Search view is focused toggles the details, and it also works in the Search Editor.

#### SB-3.11 Use exclude settings and ignore files

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A toggle in the exclude box, on by default, makes the search skip files matched by search.exclude, files.exclude and the .gitignore and .ignore files; turning it off finds those files too, and the view then says so with a link to turn it back on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.exclude; files.exclude; search.useIgnoreFiles; search.useGlobalIgnoreFiles; search.useParentIgnoreFiles; search.followSymlinks
- Source: V\src\vs\workbench\contrib\search\browser\patternInputWidget.ts:281-331; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:104-171, 198-202; V\src\vs\workbench\contrib\search\browser\searchView.ts:2047-2050, 2072-2078; C bundle (labels "Use Exclude Settings and Ignore Files", "exclude settings and ignore files are disabled", settings search.exclude, search.useIgnoreFiles, search.useGlobalIgnoreFiles, search.useParentIgnoreFiles, search.followSymlinks)
- Pieces:
  1. A toggle in the exclude box (Use Exclude Settings and Ignore Files) is on by default and makes the search skip files matched by `search.exclude`, `files.exclude` and the `.gitignore` and `.ignore` files.
  2. Turning the toggle off finds those files too, and the result count line then says "exclude settings and ignore files are disabled" with an "enable" link that turns the toggle back on.
  3. Setting `search.exclude` lists glob patterns the search skips, with defaults of `**/node_modules`, `**/bower_components` and `**/*.code-search`, and it also takes in the patterns of `files.exclude`.
  4. Settings `search.useIgnoreFiles` (on by default), `search.useGlobalIgnoreFiles` (off) and `search.useParentIgnoreFiles` (off) choose whether `.gitignore` and `.ignore` files, the global gitignore file, and ignore files in parent folders are used, where the last two need the first one on.
  5. Setting `search.followSymlinks` chooses whether the search follows symlinks, on by default.

#### SB-3.12 Search only in open editors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A book-icon toggle in the include box limits the search to files open in editors, and the view then shows "searching only in open files" with a link to search the whole workspace again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\patternInputWidget.ts:185-278; V\src\vs\workbench\contrib\search\browser\searchView.ts:545-558, 2052-2056, 2079-2083; C bundle (label "Search only in Open Editors", label "searching only in open files", members useSearchInEditorsBox and onChangeSearchInEditorsBox)
- Pieces:
  1. A book-icon toggle in the include box (Search only in Open Editors) limits the search to files that are open in editors.
  2. While it is on, the result count line says "searching only in open files" with a "disable" link that searches the whole workspace again.
  3. Clicking that link shows the search details area if it was hidden and turns the toggle off.

#### SB-3.13 Search only in changed files

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: An edit-icon toggle in the include box, enabled only when Source Control has changes, limits the search to the changed files, and it and the open editors toggle switch each other off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\patternInputWidget.ts:185-278; V\src\vs\workbench\contrib\search\browser\searchView.ts:551-556, 1625-1629; C bundle (searched the label "Search only in Changed Files (Source Control)" in the message file and the members useSearchInChangedFilesBox, onChangeSearchInChangedFilesBox, onlySearchInChangedFiles, setOnlySearchInChangedFilesEnabled in the bundle, no hits; the open editors toggle next to it is present)
- Pieces:
  1. An edit-icon toggle in the include box (Search only in Changed Files (Source Control)) limits the search to the files changed in Source Control. [stock only]
  2. The toggle is enabled only while Source Control has changes. [stock only]
  3. Turning this toggle on switches the open editors toggle off, and turning the open editors toggle on switches this one off. [stock only]

#### SB-3.14 Find in Folder from the Explorer

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a folder in the Explorer and choose Find in Folder... to open Search with that folder in the include box; Find in Workspace... is the root-level variant shown when the right-clicked root is not a folder and opens Search with the include box cleared.
- Commands: filesExplorer.findInFolder; filesExplorer.findInWorkspace
- Menus: ExplorerContext (group 4_search, order 10, both items)
- Keys: Shift+Alt+F when the Explorer is focused on a folder (filesExplorer.findInFolder)
- Settings: search.mode
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:260-320, 358-413; V\src\vs\workbench\contrib\search\browser\searchView.ts:1565-1587; C bundle (filesExplorer.findInFolder, filesExplorer.findInWorkspace, labels "Find in Folder..." and "Find in Workspace...")
- Pieces:
  1. Right-clicking a folder in the Explorer and choosing Find in Folder... (`filesExplorer.findInFolder`) opens Search with that folder in the include box.
  2. Find in Workspace... (`filesExplorer.findInWorkspace`) is the variant shown when the right-clicked root is not a folder, and it opens Search with the include box cleared.
  3. Pressing `Shift+Alt+F` while the Explorer is focused on a folder runs Find in Folder.

#### SB-3.15 Restrict the search to a folder or exclude a folder from the results

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a folder row in the results to put that folder into the include box (Restrict Search to Folder) or the exclude box (Exclude Folder from Search) and rerun the search.
- Commands: search.action.restrictSearchToFolder; search.action.excludeFromSearch
- Menus: SearchContext (group search, order 3 for restrict and order 4 for exclude)
- Keys: Shift+Alt+F when a folder row with a path is focused (search.action.restrictSearchToFolder)
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:36-61, 85-105, 358-413; V\src\vs\workbench\contrib\search\browser\searchView.ts:1565-1587; C bundle (both ids in the SearchCommandIds enum, labels "Restrict Search to Folder" and "Exclude Folder from Search")
- Pieces:
  1. Right-clicking a folder row in the results and choosing Restrict Search to Folder (`search.action.restrictSearchToFolder`) puts that folder into the include box and reruns the search.
  2. Choosing Exclude Folder from Search (`search.action.excludeFromSearch`) puts that folder into the exclude box and reruns the search.
  3. Pressing `Shift+Alt+F` while a folder row that has a path is focused runs Restrict Search to Folder.

#### SB-3.16 Exclude or include a file type from a result row

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: Right-click a file row to add its extension pattern (for example *.test.ts) to the exclude box (Exclude File Type from Search) or to the include box (Include File Type from Search, spelled that way in the source) and rerun the search.
- Commands: search.action.excludeFileTypeFromSearch; search.action.includeFileTypeInSearch
- Menus: SearchContext (group search, order 5 for exclude and order 6 for include)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:107-148, 331-343, 422-443; V\src\vs\workbench\contrib\search\common\constants.ts:51-52; C bundle (searched both ids and both labels, no hits; the SearchCommandIds enum goes from ViewAsListActionId straight to ToggleQueryDetailsActionId and has no entry for either)
- Pieces:
  1. Right-clicking a file row and choosing Exclude File Type from Search (`search.action.excludeFileTypeFromSearch`) adds the file's extension pattern, for example `*.test.ts`, to the exclude box and reruns the search. [stock only]
  2. Choosing Include File Type from Search (`search.action.includeFileTypeInSearch`, spelled that way in the source) adds the pattern to the include box and reruns the search. [stock only]

#### SB-3.17 Read the results tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Matches appear under file rows that show a file icon, name, folder path and a match count badge, with the matched text highlighted on each line, plus an "Other files" row for matches outside the workspace and one row per root in a multi-root workspace.
- Commands: none found
- Menus: SearchActionMenu (inline row buttons)
- Keys: none found
- Settings: search.showLineNumbers; search.decorations.colors; search.decorations.badges; search.actionsPosition
- Source: V\src\vs\workbench\contrib\search\browser\searchResultsView.ts:100-215, 280-290, 355-365, 490-500; V\src\vs\workbench\contrib\search\browser\searchView.ts:960-1030; V\src\vs\workbench\contrib\search\browser\media\searchview.css; C bundle (labels "Text Results", "Other files", "{0} matches found", "{0} files found", "From line {0}", "{0} more lines", settings search.showLineNumbers and search.decorations.colors and search.decorations.badges and search.actionsPosition; stylesheet classes .search-view .filematch, .foldermatch, .linematch, .actionBarContainer)
- Pieces:
  1. Matches appear under file rows that show a file icon, the file name, the folder path and a match count badge whose hover text reads "N matches found".
  2. Each match line shows its text with the matched part highlighted.
  3. An "Other files" row holds matches in files outside the workspace folders, and a multi-root workspace gets one row per root folder.
  4. Setting `search.showLineNumbers` (off by default) shows the line number and a colon before each match.
  5. A match that spans several lines shows "+N" in the line number column, and hovering it reads "From line N" and "N more lines".
  6. With a replace term typed, each match shows the replacement text next to the matched text.
  7. Settings `search.decorations.colors` and `search.decorations.badges` (both on by default) choose whether file rows use colors and badges from file decorations.
  8. Setting `search.actionsPosition` chooses where the row buttons sit, with values `right` (default, always at the right edge) and `auto` (at the right edge when the view is narrow and right after the content when it is wide).
  9. Hovering over, focusing or selecting a result row shows its small action buttons, such as Dismiss and Replace.

#### SB-3.18 View as list or tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A header button flips the results between a flat list of files and a folder tree, and search.defaultViewMode sets which one a new search starts in.
- Commands: search.action.viewAsTree; search.action.viewAsList
- Menus: ViewTitle (group navigation, order 2, when the view is Search)
- Keys: none found
- Settings: search.defaultViewMode
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsTopBar.ts:158-204; V\src\vs\workbench\contrib\search\browser\searchView.ts:386-400; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:297-306; C bundle (both ids, labels "View as Tree" and "View as List", setting search.defaultViewMode)
- Pieces:
  1. A header button flips the results between a flat list of files and a folder tree, showing View as Tree while the results are a list and View as List while they are a tree (`search.action.viewAsTree`, `search.action.viewAsList`).
  2. In the tree layout, file rows hide the folder path because the folder rows above them show it.
  3. Setting `search.defaultViewMode` chooses the layout a new search starts in, with values `list` (default) and `tree`.

#### SB-3.19 Expand and collapse results

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A header button collapses every file row or, when all are collapsed, expands everything; right-clicking a folder row adds Expand Recursively; search.collapseResults decides what a new search starts with, and a single result file opens by itself.
- Commands: search.action.collapseSearchResults; search.action.expandSearchResults; search.action.expandRecursively
- Menus: ViewTitle (group navigation, order 4); SearchContext (group search, order 4, Expand Recursively on a folder row)
- Keys: none found
- Settings: search.collapseResults
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsTopBar.ts:91-133, 250-273, 294-399; V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:64-83; V\src\vs\workbench\contrib\search\browser\searchView.ts:797-810, 1770-1781; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:226-235; C bundle (the three ids, labels "Collapse All", "Expand All", "Expand Recursively", setting search.collapseResults)
- Pieces:
  1. The Collapse All header button (`search.action.collapseSearchResults`) folds every file row.
  2. When every row is already folded, the same button becomes Expand All (`search.action.expandSearchResults`) and unfolds everything.
  3. Right-clicking a folder row adds Expand Recursively (`search.action.expandRecursively`), which unfolds everything under it.
  4. Setting `search.collapseResults` chooses how a new search starts, with values `auto` (files with fewer than 10 results open, others folded), `alwaysCollapse` and `alwaysExpand` (default).
  5. When a search finds results in only one file, that file's matches open by themselves.

#### SB-3.20 Sort the results

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A setting orders the files in the results by folder and file name, file name only, extension, last modified time, or match count in either direction.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.sortOrder
- Source: V\src\vs\workbench\contrib\search\browser\search.contribution.ts:273-286; V\src\vs\workbench\contrib\search\browser\searchCompare.ts:21-60; V\src\vs\workbench\contrib\search\browser\searchView.ts:286-292, 2650, 2755-2770; C bundle (setting search.sortOrder)
- Pieces:
  1. By default the files in the results are listed alphabetically by folder and then file name.
  2. Setting `search.sortOrder` chooses the order, with values `default` (folder and file name), `fileNames` (file name only), `type` (extension), `modified` (last modified time, newest first), `countDescending` and `countAscending` (match count per file).
  3. With `modified` or a count order, the list re-sorts by itself when a result file is edited or a match count changes.

#### SB-3.21 Open a match in the editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking a match opens its file in the editor with the match selected and highlighted, double-click or Enter keeps the tab open, and Ctrl+Enter opens it beside the current editor.
- Commands: search.action.openResult; search.action.openResultToSide
- Menus: none found
- Keys: Enter (also Cmd+Down on macOS); Ctrl+Enter (Control+Enter on macOS); both when a file or match row is focused
- Settings: workbench.list.openMode
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:158-213; V\src\vs\workbench\contrib\search\browser\searchView.ts:1020-1030, 2239-2300; V\src\vs\platform\list\browser\listService.ts:171; C bundle (search.action.openResult, search.action.openResultToSide, label "Open Match")
- Pieces:
  1. Clicking a match opens its file in the editor with the match selected and highlighted.
  2. Double-clicking a match, or pressing Enter on a focused file or match row, keeps the editor tab open (`search.action.openResult`, also `Cmd+Down` on macOS).
  3. Pressing `Ctrl+Enter` (`Control+Enter` on macOS) on a focused file or match row opens it beside the current editor (`search.action.openResultToSide`).
  4. Setting `workbench.list.openMode` chooses whether a `singleClick` (default) or a `doubleClick` opens a match with the mouse.

#### SB-3.22 Add cursors at every match in a file

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With a file or match row focused, Ctrl+Shift+L opens that file and puts a cursor on each match the search found in it.
- Commands: addCursorsAtSearchResults
- Menus: none found
- Keys: Ctrl+Shift+L when the Search view is visible and a file or match row is focused
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:215-238; V\src\vs\workbench\contrib\search\browser\searchView.ts:2302-2331; C bundle (addCursorsAtSearchResults, label "Add Cursors at Search Results")
- Pieces:
  1. With a file or match row focused, pressing `Ctrl+Shift+L` (`addCursorsAtSearchResults`) opens that file and puts a cursor on each match the search found in it.
  2. The key is active only while the Search view is visible and a file or match row is focused.

#### SB-3.23 Dismiss a result

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Remove a match, a file or a folder from the results without touching the file, using the X button on the row, the Delete key, or Dismiss in the right-click menu.
- Commands: search.action.remove
- Menus: SearchContext (group search, order 2); SearchActionMenu (group inline, order 2)
- Keys: Delete (Cmd+Backspace on macOS) when the Search view is visible and a result row is focused
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsRemoveReplace.ts:54-148; V\src\vs\workbench\contrib\search\browser\searchActionsBase.ts:34-70; C bundle (search.action.remove, label "Dismiss")
- Pieces:
  1. The X button on a result row removes that match, file or folder from the results without touching the file.
  2. Pressing Delete (`Cmd+Backspace` on macOS) on a focused result row dismisses it (`search.action.remove`).
  3. Dismiss in the right-click menu of a result row does the same.
  4. Dismissing a file or folder row removes every match under it.

#### SB-3.24 Copy results and paths

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a result to Copy the match or file text (Ctrl+C), Copy Path for a file or folder row (Ctrl+Alt+C, Shift+Alt+C on Windows), or Copy All results as text.
- Commands: search.action.copyMatch; search.action.copyPath; search.action.copyAll
- Menus: SearchContext (group search_2, orders 1, 2 and 3)
- Keys: Ctrl+C (search.action.copyMatch, file or match row); Ctrl+Alt+C, Shift+Alt+C on Windows (search.action.copyPath, file or folder row with a path)
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsCopy.ts:20-100, 133-150; C bundle (the three ids, labels "Copy", "Copy Path", "Copy All")
- Pieces:
  1. Right-clicking a match or file row and choosing Copy copies its text, and `Ctrl+C` does the same while a file or match row is focused (`search.action.copyMatch`).
  2. Copy Path on a file or folder row copies its path, and `Ctrl+Alt+C` (`Shift+Alt+C` on Windows) does the same while such a row is focused (`search.action.copyPath`).
  3. Copy All copies every result as text (`search.action.copyAll`).

#### SB-3.25 Reveal a result file in the Explorer

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a file row and choose Reveal in Explorer View to open the Explorer and select that file, which only works for files inside the workspace.
- Commands: search.action.revealInSideBar
- Menus: SearchContext (group search_3, order 1, on a file row when there are results)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:150-200; C bundle (search.action.revealInSideBar, label "Reveal in Explorer View")
- Pieces:
  1. Right-clicking a file row and choosing Reveal in Explorer View (`search.action.revealInSideBar`) opens the Explorer and selects that file.
  2. The entry shows on a file row only when there are results, and it works only for files inside the workspace.

#### SB-3.26 Move around with the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Ctrl+Down and Ctrl+Up hop between the boxes and the results, Ctrl+Up from the first result returns to the search box, and F4 and Shift+F4 select the next and previous match from anywhere.
- Commands: search.focus.nextInputBox; search.focus.previousInputBox; search.action.focusSearchFromResults; search.action.focusSearchList; search.action.focusNextSearchResult; search.action.focusPreviousSearchResult
- Menus: none found
- Keys: Ctrl+Down (search.focus.nextInputBox, in an input box); Ctrl+Up (search.focus.previousInputBox, in an input box other than the search box); Ctrl+Up (search.action.focusSearchFromResults, when the first match is focused or accessibility mode is on); F4 (search.action.focusNextSearchResult); Shift+F4 (search.action.focusPreviousSearchResult)
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:241-316, 339-392; V\src\vs\workbench\contrib\search\browser\searchView.ts:1128-1230, 1285-1365; C bundle (the six ids in the SearchCommandIds enum, labels "Focus Next Input", "Focus Previous Input", "Focus Search From Results", "Focus List", "Focus Next Search Result", "Focus Previous Search Result")
- Pieces:
  1. Pressing `Ctrl+Down` in an input box moves focus to the next box and then to the results (`search.focus.nextInputBox`).
  2. Pressing `Ctrl+Up` in an input box other than the search box moves focus to the previous box (`search.focus.previousInputBox`).
  3. Pressing `Ctrl+Up` while the first match is focused, or while accessibility mode is on, returns focus to the search box (`search.action.focusSearchFromResults`).
  4. Pressing `F4` selects the next match and `Shift+F4` selects the previous match, from anywhere (`search.action.focusNextSearchResult`, `search.action.focusPreviousSearchResult`).
  5. Focus List (`search.action.focusSearchList`) moves focus into the results list.

#### SB-3.27 Refresh, stop and clear a search

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Header buttons rerun the current search (Refresh), swap to a Stop button while a search is slow (Esc also stops it), and Clear Search Results empties the boxes and the results.
- Commands: search.action.refreshSearchResults; search.action.cancel; search.action.clearSearchResults
- Menus: ViewTitle (group navigation: Refresh order 0 when not slow, Cancel order 0 while slow, Clear order 1)
- Keys: Escape (search.action.cancel, when the Search view is visible and a list has focus)
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsTopBar.ts:42-152, 276-292; C bundle (the three ids, labels "Refresh", "Cancel Search", "Clear Search Results")
- Pieces:
  1. The Refresh header button (`search.action.refreshSearchResults`) reruns the current search while the search is not slow.
  2. While a search is slow the Refresh button is replaced by a Cancel Search button, and pressing Esc while the Search view is visible and a list has focus also stops it (`search.action.cancel`).
  3. The Clear Search Results header button (`search.action.clearSearchResults`) empties the boxes and the results.

#### SB-3.28 Open results in a Search Editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Open New Search Editor header button opens a Search Editor that carries over the file filters and toggles, and Open Results in Editor (Alt+Enter) or the "Open in editor" link in the result count line copies the current results into one; the Search Editor page itself is ED-3.
- Commands: search.action.openNewEditorFromView; search.action.openInEditor
- Menus: ViewTitle (group navigation, order 2, Open New Search Editor); ViewTitle (group navigation, order 2, when view is the Search view); CommandPalette
- Keys: Alt+Enter (Cmd+Enter on macOS) when there are results and the Search view is focused (search.action.openInEditor); Alt+Enter (Cmd+Enter on macOS) when the Search view is focused and has results
- Settings: none found
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts:350-374, 598-613; V\src\vs\workbench\contrib\searchEditor\browser\searchEditorActions.ts:78-96; V\src\vs\workbench\contrib\search\browser\searchView.ts:2085-2094; C bundle (search.action.openInEditor, search.action.openNewEditorFromView, labels "Open Results in Editor", "Open New Search Editor", "Open in editor", "Copy current search results to an editor"); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts (OpenSearchEditorAction, OpenInEditorCommandId); V\src\vs\workbench\contrib\searchEditor\browser\searchEditorActions.ts (createEditorFromSearchResult); C bundle (ids search.action.openNewEditorFromView, search.action.openInEditor, labels "Open New Search Editor", "Open Results in Editor")
- Pieces:
  1. The Open New Search Editor header button (`search.action.openNewEditorFromView`) opens a Search Editor that keeps the file filters and toggles from the Search view.
  2. Open Results in Editor (`search.action.openInEditor`, `Alt+Enter`, `Cmd+Enter` on macOS) copies the current results into a Search Editor while there are results and the Search view is focused.
  3. The "Open in editor" link in the result count line does the same, with the hover text "Copy current search results to an editor".
  4. An Open New Search Editor button in the title bar of the Search view (`search.action.openNewEditorFromView`) opens a Search Editor tab.
  5. Open Results in Editor (`search.action.openInEditor`) from the Command Palette, or `Alt+Enter` (`Cmd+Enter` on macOS) while the Search view is focused and has results, opens a Search Editor tab holding the current query, include and exclude patterns and results.
  6. When `search.searchEditor.defaultNumberOfContextLines` is above 0, the new Search Editor runs the search again so each match shows its surrounding lines, and when it is 0 the results are copied as they are.

#### SB-3.29 Choose whether Find in Files opens the view or a Search Editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A setting makes Find in Files, Find in Folder and Find in Workspace open in the Search view (the default), in an existing Search Editor, or in a new one.
- Commands: workbench.action.findInFiles; filesExplorer.findInFolder; filesExplorer.findInWorkspace
- Menus: none found
- Keys: none found
- Settings: search.mode
- Source: V\src\vs\workbench\contrib\search\browser\search.contribution.ts:130-139; V\src\vs\workbench\contrib\search\browser\searchActionsBase.ts:114-150; V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:300-320, 358-413; C bundle (setting search.mode with its description in the message file)
- Pieces:
  1. By default, Find in Files, Find in Folder and Find in Workspace show their results in the Search view.
  2. Setting `search.mode` chooses where they show, with values `view` (default), `reuseEditor` and `newEditor`.
  3. With `reuseEditor` the search goes to an existing Search Editor when one is open and to a new one otherwise, while `newEditor` always opens a new Search Editor.

#### SB-3.30 Result count line, messages and empty states

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the no-results text is older: "Review your settings for configured exclusions and check your gitignore files" where stock says "Review your configured exclusions"; the Search with AI link and keyword suggestions are absent)
- What: A line under the boxes reports "N results in M files" with the Open in editor link and reminders when excludes or open-editors scope are changing the result; when nothing is found it explains why with links (Search again in all files, Open Settings, Search again after a cancel), it warns when search.maxResults cuts the list short, and a window with no folder says only open files are searched with an Open Folder link.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.maxResults
- Source: V\src\vs\workbench\contrib\search\browser\searchView.ts:1797-1905, 2056-2105, 2183-2208; V\src\vs\workbench\contrib\search\browser\searchMessage.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:221-225; C bundle (message file strings "{0} results in {1} files", "Search again", "Search again in all files", "Open Settings", "The result set only contains a subset of all matches. Be more specific in your search to narrow down the results.", "You have not opened or specified a folder. Only open files are currently searched - ", "Search was canceled before any results could be found - ", the older no-results sentence)
- Pieces:
  1. A line under the boxes reports "N results in M files" after a search, using "result" and "file" when a count is 1.
  2. When nothing is found, the line explains why and names the include and exclude patterns in use, for example "No results found in '<include>' excluding '<exclude>' - ".
  3. After a no-results message, a link follows it - "Search again in all files" when an include or exclude pattern is set, and "Open Settings" when neither is set.
  4. When the open editors toggle is on and nothing is found, the message starts "No results found in open editors" instead.
  5. When a search was canceled before any results were found, the message reads "Search was canceled before any results could be found - " followed by a "Search again" link.
  6. Setting `search.maxResults` (default 20000, empty for unlimited) caps the list, and when the cap cuts it short a warning reads "The result set only contains a subset of all matches. Be more specific in your search to narrow down the results."
  7. A window with no folder shows "You have not opened or specified a folder. Only open files are currently searched - " with an Open Folder link.
  8. The no-results text without patterns reads "Review your settings for configured exclusions and check your gitignore files" where stock reads "Review your configured exclusions and check your gitignore files", and there is no Search with AI link or keyword suggestion. [cursor only]


## SB-4 Source Control

(both) [boundary for Cursor-added items only]. The Git panel for changes, staging, commit, repositories and the commit graph. Cursor adds an Agent Review view (id `bugbot`), a Generate Commit Message button, and a worktrees setting, and has 154 Git commands against stock's 183.

53 features.

#### SB-4.1 View changed files in groups

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (adds scm.maxRenderedResources, default 5000 rendered files per group, not in stock)
- What: Open the Source Control view (Changes) and see changed files sorted into Merge Changes, Staged Changes, Changes and Untracked Changes groups, each with a count, status letters and inline buttons.
- Commands: workbench.view.scm
- Menus: scm/resourceGroup/context
- Keys: Ctrl+Shift+G (Control+Shift+G on Mac) opens the view
- Settings: git.untrackedChanges; git.alwaysShowStagedChangesResourceGroup; git.statusLimit; git.ignoreLimitWarning; git.ignoreSubmodules; git.autorefresh; scm.alwaysShowActions; scm.maxRenderedResources; git.similarityThreshold
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts; V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts; V\extensions\git\src\repository.ts (resource groups merge, index, workingTree, untracked); V\extensions\git\package.json; C bundle (view id workbench.scm, key Ctrl+Shift+G in openCommandActionDescriptor, setting scm.maxRenderedResources with limitRenderedChildren)
- Pieces:
  1. Opening the Source Control view lists the repository's changed files under group headings, each heading showing a count of its files.
  2. Pressing Ctrl+Shift+G (Control+Shift+G on Mac) runs `workbench.view.scm` and opens the Source Control view.
  3. Changed files are sorted into Merge Changes, Staged Changes, Changes and Untracked Changes groups, and the Merge Changes group only shows while there are conflicts.
  4. Each changed file row shows the file name, its folder path in list view, and a status letter such as M, D or U on the right.
  5. Hovering a group header or a file row shows inline buttons, and `scm.alwaysShowActions` keeps them visible all the time.
  6. Setting `git.untrackedChanges` chooses where new files go, with values `mixed` (inside Changes), `separate` (their own Untracked Changes group) and `hidden`.
  7. Setting `git.alwaysShowStagedChangesResourceGroup` keeps the Staged Changes group on screen even when nothing is staged.
  8. Setting `git.statusLimit` caps how many changes are listed, and `git.ignoreLimitWarning` hides the warning shown when the cap is hit.
  9. Settings `git.autorefresh` and `git.ignoreSubmodules` decide whether the list updates itself when files change and whether submodule changes are listed.
  10. Setting `scm.maxRenderedResources` limits how many files are drawn in each group, with a default of 5000. [cursor only]

#### SB-4.2 Stage or unstage changes

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Move one file, a folder, or a whole group between Changes and Staged Changes with the plus and minus buttons or the right-click menu.
- Commands: git.stage; git.unstage; git.stageAll; git.unstageAll; git.stageAllTracked; git.stageAllUntracked; git.stageAllMerge; git.stageFile; git.unstageFile
- Menus: scm/resourceState/context; scm/resourceFolder/context; scm/resourceGroup/context; git.changes; multiDiffEditor/resource/title; SCMResourceContext; SCMResourceFolderContext
- Keys: none found
- Settings: git.untrackedChanges; git.optimisticUpdate
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json (same 46 scm/resourceState/context items); C\extensions\git\dist\main.js (git.stage, git.unstage, git.stageAll found)
- Pieces:
  1. Clicking the plus button on a file row under Changes moves that file into Staged Changes and the group counts update.
  2. Clicking the minus button on a file row under Staged Changes moves that file back into Changes.
  3. Right-clicking a file row offers Stage Changes or Unstage Changes (`git.stage`, `git.unstage`) with the same result as the inline button.
  4. In tree view a folder row has the same plus or minus button and right-click entry, and it stages or unstages every file under that folder.
  5. The Changes header has Stage All Changes (`git.stageAll`) and the Staged Changes header has Unstage All Changes (`git.unstageAll`), as inline buttons and right-click entries.
  6. The Merge Changes header has Stage All Merge Changes (`git.stageAllMerge`), which stages every conflicted file in one step.
  7. Setting `git.untrackedChanges` changes the header buttons: with `mixed` the Changes header stages everything, and otherwise it shows Stage All Tracked Changes (`git.stageAllTracked`) while the Untracked Changes header shows Stage All Untracked Changes (`git.stageAllUntracked`).
  8. In a multi-file diff editor each file's title has Stage and Unstage buttons (`git.stageFile`, `git.unstageFile`).

#### SB-4.3 Discard changes

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Throw away edits to a file, a folder or a whole group from the right-click menu or the inline undo button, after a confirmation; untracked files can go to the Recycle Bin.
- Commands: git.clean; git.cleanAll; git.cleanAllTracked; git.cleanAllUntracked
- Menus: scm/resourceState/context; scm/resourceFolder/context; scm/resourceGroup/context; git.changes
- Keys: none found
- Settings: git.discardUntrackedChangesToTrash; git.untrackedChanges
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.clean, git.cleanAll, git.cleanAllUntracked found)
- Pieces:
  1. Clicking the Discard Changes button on a file row, or choosing it from the right-click menu, asks for confirmation and then restores the file to its last committed content (`git.clean`).
  2. The confirmation for a tracked file is a modal that names the file and has a Discard File button, and a file that was deleted shows Restore File instead.
  3. In tree view a folder row has Discard Changes in its right-click menu and as an inline button, and it applies to every file under that folder.
  4. The Changes header has Discard All Changes (`git.cleanAll`) when `git.untrackedChanges` is `mixed` and Discard All Tracked Changes (`git.cleanAllTracked`) otherwise, and the modal states the file count and that the loss is irreversible.
  5. The Untracked Changes header has Discard All Untracked Changes (`git.cleanAllUntracked`), which asks before deleting the new files.
  6. Discarding untracked files shows a modal saying the files will be deleted, with a Move to Recycle Bin button on Windows (Move to Trash on other systems) and a note that they can be restored from there.
  7. Setting `git.discardUntrackedChangesToTrash` set to false changes that modal into a permanent delete warning with a Delete File button.
  8. When a selection holds both tracked and untracked files, the modal offers Discard All Tracked Files or Discard All Files.

#### SB-4.4 Open a diff from the list

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (group View Changes buttons are gated on config.multiDiffEditor.experimental.enabled, which defaults to true in the C bundle)
- What: Click a changed file to open its diff, use the inline Open File button, Open File (HEAD) or Compare with Workspace, or open a whole group in one multi-file diff with the group's View Changes button.
- Commands: git.openChange; git.openFile; git.openFile2; git.openHEADFile; git.compareWithWorkspace; git.openAllChanges; git.viewChanges; git.viewStagedChanges; git.viewUntrackedChanges
- Menus: scm/resourceState/context; scm/resourceGroup/context
- Keys: none found
- Settings: git.openDiffOnClick; git.showInlineOpenFileAction; scm.autoReveal; git.closeDiffOnOperation
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts; C\extensions\git\package.json; C bundle (setting multiDiffEditor.experimental.enabled, default true)
- Pieces:
  1. Clicking a changed file row opens a diff editor of that file against its Git version.
  2. Setting `git.openDiffOnClick` turned off makes a click open the file itself instead of a diff.
  3. An inline Open File button on the row opens the file in the regular editor (`git.openFile`), and `git.showInlineOpenFileAction` hides it.
  4. The right-click menu has Open File (HEAD), which shows the committed version, and Compare with Workspace (`git.openHEADFile`, `git.compareWithWorkspace`).
  5. Each group header has an inline View Changes button that opens every file of that group in one multi-file diff editor (`git.viewChanges`, `git.viewStagedChanges`, `git.viewUntrackedChanges`).
  6. Setting `scm.autoReveal` on makes the list scroll to and select a file's row when that file is opened.
  7. Setting `git.closeDiffOnOperation` on closes open diff tabs when changes are staged, unstaged, discarded, committed or stashed.
  8. The group View Changes buttons show only while `multiDiffEditor.experimental.enabled` is on, and it is on by default. [cursor only]

#### SB-4.5 Arrange the list: list or tree, and sort

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Switch the changes list between a flat list and a folder tree, pick the sort order (name, path or status) from the View and Sort menu, and collapse or compact folders.
- Commands: workbench.scm.action.setListViewMode; workbench.scm.action.setTreeViewMode; workbench.scm.action.setListViewModeNavigation; workbench.scm.action.setTreeViewModeNavigation; workbench.scm.action.setSortKey.name; workbench.scm.action.setSortKey.path; workbench.scm.action.setSortKey.status; workbench.scm.action.collapseAll
- Menus: scm/title (View and Sort submenu and the navigation toggle); SCMResourceGroupContext
- Keys: none found
- Settings: scm.defaultViewMode; scm.defaultViewSortKey; scm.compactFolders
- Source: V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (lines 953-1360); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts; C bundle (workbench.scm.action.setListViewMode, setTreeViewMode, setSortKey, collapseAll; settings scm.defaultViewMode, scm.defaultViewSortKey, scm.compactFolders)
- Pieces:
  1. The View and Sort submenu in the "..." title menu switches between View as List and View as Tree, with the current choice checked (`workbench.scm.action.setListViewMode`, `workbench.scm.action.setTreeViewMode`).
  2. A button in the view title bar flips the view mode in one click, showing a tree icon while the list is shown and a list icon while the tree is shown (`workbench.scm.action.setTreeViewModeNavigation`, `workbench.scm.action.setListViewModeNavigation`).
  3. The View and Sort submenu has Sort Changes by Name, Sort Changes by Path and Sort Changes by Status, with the active one checked (`workbench.scm.action.setSortKey.name`, `.path`, `.status`).
  4. In tree view a Collapse All button closes every folder (`workbench.scm.action.collapseAll`).
  5. Setting `scm.defaultViewMode` chooses the starting view mode, with values `tree` and `list` and a default of `list`.
  6. Setting `scm.defaultViewSortKey` chooses the starting sort order, with values `name`, `path` and `status` and a default of `path`.
  7. Setting `scm.compactFolders` on, the default, joins folders that hold a single child folder into one combined row.

#### SB-4.6 Reveal a changed file or add it to .gitignore

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a changed file to show it in the Explorer view or the operating system file manager, or to add it to .gitignore.
- Commands: git.revealInExplorer; git.revealFileInOS.windows; git.revealFileInOS.mac; git.revealFileInOS.linux; git.ignore
- Menus: scm/resourceState/context; scm/resourceFolder/context
- Keys: none found
- Settings: none found
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.revealInExplorer, git.ignore found)
- Pieces:
  1. Right-clicking a changed file row and choosing Reveal in Explorer View selects that file in the Explorer view (`git.revealInExplorer`).
  2. Right-clicking a changed file row and choosing Reveal in File Explorer on Windows, Reveal in Finder on Mac or Open Containing Folder on Linux shows the file in the system file manager (`git.revealFileInOS.windows`, `git.revealFileInOS.mac`, `git.revealFileInOS.linux`).
  3. Right-clicking a changed file row and choosing Add to .gitignore adds that file to the repository's `.gitignore` (`git.ignore`).
  4. In tree view a folder row has the same reveal and ignore entries in its right-click menu, and they act on the folder.

#### SB-4.7 Type a message and commit

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (scm.clearValidation not found; the diagnostics hook settings are spelled git.diagnosticsCommitHook.Enabled and .Sources with capitals)
- What: Type a message in the box at the top of the Changes view and press Ctrl+Enter or the Commit button; the box can grow, warn about long lines, ask about smart commit when nothing is staged, and prompt before committing on a protected branch.
- Commands: scm.acceptInput; git.commit; scm.clearValidation; scm.clearInput
- Menus: scm/title; SCMInputBox
- Keys: Ctrl+Enter (scm.acceptInput, when scmRepository); Escape (scm.clearInput, when no suggest widget and no validation message); Escape (scm.clearValidation, when a validation message is shown)
- Settings: git.showCommitInput; git.enableSmartCommit; git.smartCommitChanges; git.suggestSmartCommit; git.requireGitUserConfig; git.promptToSaveFilesBeforeCommit; git.inputValidation; git.inputValidationLength; git.inputValidationSubjectLength; git.branchProtection; git.branchProtectionPrompt; git.diagnosticsCommitHook.enabled; git.diagnosticsCommitHook.sources; scm.inputFontFamily; scm.inputFontSize; scm.inputMaxLineCount; scm.inputMinLineCount
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 420-475); V\src\vs\workbench\contrib\scm\browser\scmInput.ts; V\extensions\git\src\commands.ts (commitWithAnyInput, lines 2372-2576); V\extensions\git\src\repository.ts (acceptInputCommand line 998, placeholder line 3284); V\extensions\git\src\diagnostics.ts; V\extensions\git\src\postCommitCommands.ts; C bundle (scm.acceptInput, scm.clearInput, settings scm.inputFontFamily and the rest); C\extensions\git\dist\main.js (diagnosticsCommitHook.Enabled)
- Pieces:
  1. The message box at the top of the Changes view, hidden when `git.showCommitInput` is off, shows the placeholder `Message (Ctrl+Enter to commit on "branch")` and grows from `scm.inputMinLineCount` to `scm.inputMaxLineCount` lines using `scm.inputFontFamily` and `scm.inputFontSize`.
  2. Pressing Ctrl+Enter in the box (`scm.acceptInput`) or clicking the Commit button in the view title (`git.commit`) commits with the typed message and empties the box.
  3. Pressing Escape clears the typed message when no suggestion list or warning is open (`scm.clearInput`).
  4. Pressing Escape while a warning shows under the box dismisses the warning (`scm.clearValidation`). [stock only]
  5. Setting `git.inputValidation` on shows a warning under the box when the message or its first line runs past `git.inputValidationLength` (72) or `git.inputValidationSubjectLength` (50).
  6. Committing with nothing staged asks whether to stage all changes and commit them, with Yes, Always and Never, and `git.enableSmartCommit`, `git.smartCommitChanges` (values `all` and `tracked`) and `git.suggestSmartCommit` decide whether that happens silently or is offered.
  7. Committing on a branch listed in `git.branchProtection` asks Commit Anyway or Commit to a New Branch, and `git.branchProtectionPrompt` chooses `alwaysCommit`, `alwaysCommitToNewBranch` or `alwaysPrompt`.
  8. Setting `git.promptToSaveFilesBeforeCommit`, with values `always`, `staged` and `never`, decides whether unsaved files raise a Save All & Commit Changes prompt.
  9. Setting `git.diagnosticsCommitHook.enabled` on makes a commit with unresolved problems stop and ask Commit Anyway or View Problems, and `git.diagnosticsCommitHook.sources` lists which sources and severities count.
  10. In Cursor the two diagnostics hook settings are spelled `git.diagnosticsCommitHook.Enabled` and `git.diagnosticsCommitHook.Sources` with capital letters. [cursor only]

#### SB-4.8 Use the action button under the message box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A wide button below the message box reads Commit, Continue (during a merge or rebase), Sync Changes with ahead and behind counts, or Publish Branch depending on the state, and the Commit button has a drop-down for Commit and Push, Commit and Sync, or Commit (Amend).
- Commands: git.commit; git.sync; git.publish
- Menus: none found
- Keys: none found
- Settings: git.showActionButton; scm.showActionButton; git.postCommitCommand; git.rememberPostCommitCommand
- Source: V\extensions\git\src\actionButton.ts; V\extensions\git\src\postCommitCommands.ts; V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts; C\extensions\git\dist\main.js (strings Sync Changes and Publish Branch found); C\extensions\git\package.json (git.postCommitCommand, git.showActionButton)
- Pieces:
  1. A wide button under the message box reads Commit with a check icon when there are changes ready to commit (`git.commit`).
  2. The wide button reads Continue during a merge or rebase and finishes that operation.
  3. The wide button reads Sync Changes with the number of commits to pull and to push when the branch is ahead or behind its remote (`git.sync`).
  4. The wide button reads Publish Branch when the local branch has no remote branch yet (`git.publish`).
  5. The Commit button has a drop-down arrow listing Commit and Push, Commit and Sync, and Commit (Amend).
  6. Setting `git.postCommitCommand` chooses what runs after a commit, with values `none`, `push` and `sync`, and `git.rememberPostCommitCommand` makes the button remember the last choice.
  7. Settings `git.showActionButton` (with `commit`, `publish` and `sync` switches) and `scm.showActionButton` show or hide the wide button.

#### SB-4.9 Choose how to commit

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: From the Commit submenu in the "..." menu, commit only staged changes, commit all changes, sign off, skip the hooks (No Verify) or make an empty commit.
- Commands: git.commitStaged; git.commitAll; git.commitSigned; git.commitStagedSigned; git.commitAllSigned; git.commitNoVerify; git.commitStagedNoVerify; git.commitAllNoVerify; git.commitEmpty; git.commitAllSignedNoVerify; git.commitEmptyNoVerify; git.commitSignedNoVerify; git.commitStagedSignedNoVerify
- Menus: git.commit; scm/title
- Keys: none found
- Settings: git.alwaysSignOff; git.enableCommitSigning; git.allowNoVerifyCommit; git.confirmNoVerifyCommit; git.confirmEmptyCommits
- Source: V\extensions\git\package.json (git.commit submenu, 20 items); V\extensions\git\src\commands.ts; C\extensions\git\package.json (same 20 items); C\extensions\git\dist\main.js (git.commitStaged, git.commitAll, git.commitSigned, git.commitNoVerify, git.commitEmpty found)
- Pieces:
  1. The Commit submenu in the "..." title menu lists Commit, Commit Staged and Commit All (`git.commit`, `git.commitStaged`, `git.commitAll`).
  2. Commit Staged commits only the staged files, and Commit All stages every change and commits them (`git.commitStaged`, `git.commitAll`).
  3. The Signed Off entries add a sign-off line to the message (`git.commitSigned`, `git.commitStagedSigned`, `git.commitAllSigned`), `git.alwaysSignOff` does it for every commit, and `git.enableCommitSigning` signs commits with GPG, X.509 or SSH.
  4. The No Verify entries skip the commit hooks and only show when `git.allowNoVerifyCommit` is on (`git.commitNoVerify`, `git.commitStagedNoVerify`, `git.commitAllNoVerify`).
  5. Setting `git.confirmNoVerifyCommit` makes a No Verify commit ask for confirmation with OK and OK, Don't Ask Again.
  6. Commit Empty makes a commit with no changes after asking for confirmation, and `git.confirmEmptyCommits` turns the question off (`git.commitEmpty`).
  7. Committing when there are no changes shows "There are no changes to commit." with a Create Empty Commit button.

#### SB-4.10 Amend the last commit

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Rewrite the previous commit with the current message and changes using Commit (Amend), Commit Staged (Amend) or Commit All (Amend).
- Commands: git.commitAmend; git.commitStagedAmend; git.commitAllAmend; git.commitAmendNoVerify; git.commitStagedAmendNoVerify; git.commitAllAmendNoVerify
- Menus: git.commit; action button drop-down (SourceControlActionButton secondary commands)
- Keys: none found
- Settings: git.allowNoVerifyCommit
- Source: V\extensions\git\package.json (git.commit submenu groups 3_amend and 4_amend_noverify); V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.commitAmend found)
- Pieces:
  1. Commit (Amend) in the Commit submenu rewrites the last commit with the staged changes and the text in the message box (`git.commitAmend`).
  2. Commit Staged (Amend) uses only the staged changes, and Commit All (Amend) stages everything first (`git.commitStagedAmend`, `git.commitAllAmend`).
  3. Amending with an empty message box keeps the previous commit message and does not ask for a new one.
  4. Commit (Amend) is also in the drop-down of the Commit button under the message box.
  5. The Amend No Verify entries skip the commit hooks and only show when `git.allowNoVerifyCommit` is on (`git.commitAmendNoVerify`, `git.commitStagedAmendNoVerify`, `git.commitAllAmendNoVerify`).

#### SB-4.11 Undo the last commit

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Undo Last Commit takes the newest commit back into the working tree and puts its message in the message box.
- Commands: git.undoCommit
- Menus: git.commit
- Keys: none found
- Settings: none found
- Source: V\extensions\git\package.json (git.commit submenu, group 1_commit at 4); V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.undoCommit found)
- Pieces:
  1. Choosing Undo Last Commit in the Commit submenu removes the newest commit and puts its changes back into the working tree (`git.undoCommit`).
  2. The undone commit's message appears in the message box, ready to edit and commit again.
  3. When the newest commit is a merge commit, a modal asks "The last commit was a merge commit. Are you sure you want to undo it?" with an Undo merge commit button.
  4. In a repository with no commits, Undo Last Commit shows the warning "Can't undo because HEAD doesn't point to any commit."

#### SB-4.12 Write the commit message in a full editor

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the Accept and Cancel buttons are declared under editor/title, not editor/content)
- What: With the setting on (the default), committing with an empty message box opens git's commit message in a full editor tab with Accept and Cancel buttons, and the message template can be restored.
- Commands: git.commitMessageAccept; git.commitMessageDiscard; git.restoreCommitTemplate
- Menus: editor/content (stock); editor/title (Cursor)
- Keys: none found
- Settings: git.useEditorAsCommitInput; git.verboseCommit; git.terminalGitEditor
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts (line 2461); V\extensions\git\src\gitEditor.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.commitMessageAccept, git.restoreCommitTemplate found)
- Pieces:
  1. With `git.useEditorAsCommitInput` on, the default, committing with an empty message box opens git's commit message in a full editor tab instead of asking in a small input.
  2. The commit message tab has Commit and Cancel buttons, where Commit finishes the commit with the text and Cancel drops it (`git.commitMessageAccept`, `git.commitMessageDiscard`).
  3. Running Restore Commit Template puts the commit message template back into the open message editor (`git.restoreCommitTemplate`).
  4. Setting `git.verboseCommit` on adds the verbose diff of what is being committed to the commit message editor.
  5. Setting `git.terminalGitEditor` on makes the app the Git editor for Git commands run in the integrated terminal, after the terminal is restarted.
  6. With `git.useEditorAsCommitInput` off, committing with an empty message box shows an input box that asks "Please provide a commit message".
  7. In Cursor the Commit and Cancel buttons sit in the editor title bar instead of the editor content area. [cursor only]

#### SB-4.13 Browse earlier commit messages

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In the message box, press Up or Down at the first or last line, or Alt+Up and Alt+Down anywhere, to step through messages you typed before.
- Commands: scm.viewPreviousCommit; scm.viewNextCommit; scm.forceViewPreviousCommit; scm.forceViewNextCommit
- Menus: none found
- Keys: Up (scm.viewPreviousCommit, when scmRepository and scmInputIsInFirstPosition and no suggest widget); Down (scm.viewNextCommit, when scmInputIsInLastPosition); Alt+Up (scm.forceViewPreviousCommit); Alt+Down (scm.forceViewNextCommit)
- Settings: none found
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 483-537); V\src\vs\workbench\contrib\scm\browser\scmInput.ts (showNextHistoryValue); C bundle (all four ids found, context key scmInputIsInLastPosition found, labels Source Control: View Next Commit and View Previous Commit in the NLS file)
- Pieces:
  1. Pressing Up while the cursor is on the first line of the message box replaces the text with the previous message from the history (`scm.viewPreviousCommit`).
  2. Pressing Down while the cursor is on the last line steps to the next message in the history (`scm.viewNextCommit`).
  3. Pressing Alt+Up or Alt+Down anywhere in the box steps back or forward through the history whatever line the cursor is on (`scm.forceViewPreviousCommit`, `scm.forceViewNextCommit`).
  4. While the suggestion list is open, Up and Down move through the suggestions instead of the history.
  5. The history is kept separately for each repository and holds up to 100 messages.

#### SB-4.14 Generate a commit message

- Tag: cursor. Evidence: bundle. Boundary: yes (a model reads the staged diff and recent commit messages and writes the text, so the result comes from the agent side). Cursor: cursor only
- What: A sparkle button in the message box fills it with a generated commit message, and the same action is in the Command Palette.
- Commands: cursor.generateGitCommitMessage; scm.input.cancelAction
- Menus: scm/inputBox; commandPalette
- Keys: none found
- Settings: scm.showInputActionButton
- Source: C\extensions\cursor-always-local\package.json (contributes.menus scm/inputBox with when scmProvider == git, enabled API proposal contribSourceControlInputBoxMenu); C bundle (command class with id cursor.generateGitCommitMessage, title Generate Commit Message, icon sparkle, f1 true; menu id SCMInputBox registered); V\src\vs\workbench\contrib\scm\browser\scmInput.ts (the input toolbar that hosts the button, shown only when scm.showInputActionButton is on)
- Pieces:
  1. A sparkle button labeled Generate Commit Message appears in the message box toolbar of a Git repository. [cursor only]
  2. Clicking the sparkle button fills the message box with a generated commit message. [cursor only]
  3. Running Generate Commit Message from the Command Palette fills the message box the same way (`cursor.generateGitCommitMessage`). [cursor only]
  4. Setting `scm.showInputActionButton` on, the default, lets the message box toolbar with this button show, and turning it off hides it. [cursor only]

#### SB-4.15 Switch branches

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick Checkout to... from the title bar menu to switch to a local branch, remote branch or tag, or check out a detached commit.
- Commands: git.checkout; git.checkoutDetached
- Menus: scm/title (group 1_header)
- Keys: none found
- Settings: git.checkoutType; git.branchSortOrder; git.showReferenceDetails; git.pullBeforeCheckout
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.checkout, git.checkoutDetached found)
- Pieces:
  1. Choosing Checkout to... from the "..." title menu opens a pick list of branches, remote branches and tags (`git.checkout`).
  2. Picking a local branch, a remote branch or a tag in the list switches the repository to that ref.
  3. The list includes Create new branch..., Create new branch from... and Checkout detached... entries along with the refs.
  4. Checkout to (Detached)... opens a pick list for checking out a commit without a branch (`git.checkoutDetached`).
  5. When local changes would be overwritten by a checkout, a modal offers Stash & Checkout, Migrate Changes or Force Checkout.
  6. Setting `git.checkoutType` chooses which kinds of refs the list shows, with values `local`, `remote` and `tags`.
  7. Setting `git.branchSortOrder` chooses how branches are ordered in the list, with values `committerdate` and `alphabetically`.
  8. Settings `git.showReferenceDetails` and `git.pullBeforeCheckout` turn on or off the last commit details beside each ref and the fast-forward of a branch with no outgoing commits before it is checked out.

#### SB-4.16 Create, rename or delete a branch

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Make a new branch from the current commit or from a chosen ref, rename the current branch, or delete a local or remote branch, with naming rules from settings.
- Commands: git.branch; git.branchFrom; git.renameBranch; git.deleteBranch; git.deleteRemoteBranch
- Menus: git.branch; scm/title
- Keys: none found
- Settings: git.branchPrefix; git.branchValidationRegex; git.branchWhitespaceChar; git.branchRandomName.enable; git.branchRandomName.dictionary; git.defaultBranchName
- Source: V\extensions\git\package.json (git.branch submenu, 8 items); V\extensions\git\src\commands.ts; C\extensions\git\package.json (same 8 items); C\extensions\git\dist\main.js (git.branch, git.branchFrom, git.deleteBranch, git.renameBranch found)
- Pieces:
  1. Create Branch... in the Branch submenu asks for a name in an input box that says "Please provide a new branch name" and then creates and switches to the new branch (`git.branch`).
  2. Create Branch From... first asks which ref to start from ("Select a ref to create the branch from") and then asks for the name (`git.branchFrom`).
  3. Typing a name that already exists shows the message `Branch "name" already exists` and the name is not accepted.
  4. Rename Branch... asks for a new name for the current branch and shows an error for an invalid name or one that already exists (`git.renameBranch`).
  5. Delete Branch... lists local branches to delete, refuses the active branch with "The active branch cannot be deleted.", and for an unmerged branch asks "not fully merged. Delete anyway?" with a Delete Branch button (`git.deleteBranch`).
  6. Delete Remote Branch... lists remote branches to delete and refuses the remote branch of the active branch (`git.deleteRemoteBranch`).
  7. Setting `git.branchPrefix` adds a prefix to new branch names and `git.branchWhitespaceChar` replaces spaces in typed names, with a default of `-`.
  8. Setting `git.branchValidationRegex` rejects a new branch name that does not match with the message "Branch name needs to match regex".
  9. Settings `git.branchRandomName.enable` and `git.branchRandomName.dictionary` pre-fill a random name built from `adjectives`, `animals`, `colors` and `numbers`, and the input has a Regenerate Branch Name button.
  10. Setting `git.defaultBranchName` names the first branch of a repository made with Initialize Repository.

#### SB-4.17 Merge a branch

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Merge another branch into the current one from the Branch submenu, and abort a merge that is in progress.
- Commands: git.merge; git.mergeAbort
- Menus: git.branch (group 1_merge)
- Keys: none found
- Settings: none found
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.merge, git.mergeAbort found)
- Pieces:
  1. Merge... in the Branch submenu opens a pick list that says "Select a branch or tag to merge from" and merges the pick into the current branch (`git.merge`).
  2. A merge that stops on conflicts shows an error saying there are merge conflicts to resolve, with a Show Changes button, and the files appear in the Merge Changes group.
  3. Abort Merge in the Branch submenu cancels the merge that is in progress (`git.mergeAbort`).

#### SB-4.18 Rebase a branch

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Rebase the current branch onto another from the Branch submenu, and abort a rebase that is in progress with Abort Rebase in the Commit submenu.
- Commands: git.rebase; git.rebaseAbort
- Menus: git.branch (group 1_merge); git.commit (group 1_commit)
- Keys: none found
- Settings: none found
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.rebase, git.rebaseAbort found)
- Pieces:
  1. Rebase Branch... in the Branch submenu opens a pick list that says "Select a branch to rebase onto" and rebases the current branch onto the pick (`git.rebase`).
  2. Abort Rebase in the Commit submenu abandons the rebase that is in progress (`git.rebaseAbort`).
  3. Running Abort Rebase when no rebase is running shows the message "No rebase in progress."

#### SB-4.19 Work with tags

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the Tags submenu has 3 items against 4, Push Tags is missing from the menu, but the command id is declared)
- What: Create a tag, delete a local or remote tag, and push tags, from the Tags submenu or the Command Palette.
- Commands: git.createTag; git.deleteTag; git.deleteRemoteTag; git.pushTags; git.pushWithTags; git.pushWithTagsForce
- Menus: git.tags; scm/title
- Keys: none found
- Settings: git.pullTags; git.replaceTagsWhenPull; git.followTagsWhenSync
- Source: V\extensions\git\package.json (git.tags submenu); V\extensions\git\src\commands.ts; C\extensions\git\package.json (git.tags has 3 items); C\extensions\git\dist\main.js (git.createTag, git.deleteTag, git.pushTags found)
- Pieces:
  1. Create Tag... asks for a tag name and then an optional message, and a message makes an annotated tag (`git.createTag`).
  2. Delete Tag... lists local tags to delete, and a repository with none shows "This repository has no tags." (`git.deleteTag`).
  3. Delete Remote Tag... asks which remote and then which remote tag to delete (`git.deleteRemoteTag`).
  4. Push Tags sends all local tags to the remote, and Push (Follow Tags) and Push (Follow Tags, Force) push the branch along with its tags (`git.pushTags`, `git.pushWithTags`, `git.pushWithTagsForce`).
  5. Settings `git.pullTags`, `git.replaceTagsWhenPull` and `git.followTagsWhenSync` turn on or off fetching tags on pull, replacing conflicting local tags on pull, and pushing annotated tags on sync.
  6. A pull blocked by conflicting tags shows a modal asking whether to replace the local tags, with a Replace Local Tag(s) button and an Always Replace Local Tag(s) button.
  7. The Tags submenu in the "..." title menu lists Push Tags as a fourth entry. [stock only]

#### SB-4.20 Stash changes

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the View Stash item needs config.multiDiffEditor.experimental.enabled, which defaults to true in the C bundle)
- What: Stash changes (all, including untracked, or staged only), then apply, pop, view or drop a stash, or drop them all.
- Commands: git.stash; git.stashIncludeUntracked; git.stashStaged; git.stashApply; git.stashApplyLatest; git.stashPop; git.stashPopLatest; git.stashDrop; git.stashDropAll; git.stashView
- Menus: git.stash; scm/title
- Keys: none found
- Settings: git.promptToSaveFilesBeforeStash; git.useCommitInputAsStashMessage; git.closeDiffOnOperation
- Source: V\extensions\git\package.json (git.stash submenu, 10 items); V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.stash, git.stashPop, git.stashApply, git.stashDrop, git.stashView found); C bundle (git.stash, git.stashIncludeUntracked found)
- Pieces:
  1. Stash saves the tracked changes away and cleans the working tree, after asking for an optional message in an input box that says "Stash message" (`git.stash`).
  2. Stash (Include Untracked) also stashes new files and Stash Staged stashes only the staged changes (`git.stashIncludeUntracked`, `git.stashStaged`).
  3. Stashing with nothing to stash shows "There are no changes to stash." or "There are no staged changes to stash."
  4. Apply Stash... and Pop Stash... ask which stash to use, and Apply Latest Stash and Pop Latest Stash use the newest one without asking, where pop also removes the stash (`git.stashApply`, `git.stashPop`, `git.stashApplyLatest`, `git.stashPopLatest`).
  5. Drop Stash... asks "Are you sure you want to drop the stash" before removing one, and Drop All Stashes... asks with the count and warns it may be impossible to recover (`git.stashDrop`, `git.stashDropAll`).
  6. View Stash... asks which stash to look at and opens its changes in a multi-file diff (`git.stashView`).
  7. Applying, popping, dropping or viewing when the repository has no stashes shows "There are no stashes in the repository."
  8. Setting `git.promptToSaveFilesBeforeStash`, with values `always`, `staged` and `never`, decides whether unsaved files raise a Save All & Stash or Stash Anyway prompt.
  9. Setting `git.useCommitInputAsStashMessage` on uses the text in the commit message box as the default stash message.
  10. The View Stash item only shows while `multiDiffEditor.experimental.enabled` is on, and it is on by default. [cursor only]

#### SB-4.21 Resolve merge conflicts from the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Conflicted files appear in a Merge Changes group where you can open the file with its conflict markers, accept all current or all incoming changes from the right-click menu, and stage them once resolved.
- Commands: git.stageAllMerge; git.stage; git.openFile; merge-conflict.accept.all-current; merge-conflict.accept.all-incoming
- Menus: scm/resourceGroup/context; scm/resourceState/context
- Keys: none found
- Settings: none found
- Source: V\extensions\git\package.json; V\extensions\merge-conflict\package.json (scm/resourceState/context items for the merge group); V\extensions\git\src\repository.ts (merge group id merge); C\extensions\git\package.json; C\extensions\merge-conflict\package.json (same two scm/resourceState/context items); C\extensions\git\dist\main.js (git.stageAllMerge found); C\extensions\merge-conflict\dist\mergeConflictMain.js (accept.all-current and accept.all-incoming found)
- Pieces:
  1. Conflicted files appear in the Merge Changes group, each marked with a conflict status letter.
  2. Opening a conflicted file shows it in the editor with its conflict markers (`git.openFile`).
  3. Right-clicking a conflicted file offers Accept All Current and Accept All Incoming, which resolve every conflict in that file one way (`merge-conflict.accept.all-current`, `merge-conflict.accept.all-incoming`).
  4. Stage Changes on a conflicted file, or Stage All Merge Changes on the group header, moves the file into Staged Changes as resolved (`git.stage`, `git.stageAllMerge`).
  5. Staging a file that still has conflicts asks whether you are sure you want to stage it with merge conflicts, with a Yes button.
  6. A file deleted on one side and changed on the other shows a modal that says who deleted it, with a Keep Our Version or Keep Their Version button and a Delete File button.

#### SB-4.22 Fetch

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Fetch from the default remote, fetch with prune, or fetch from all remotes, and optionally let Git fetch on a timer.
- Commands: git.fetch; git.fetchPrune; git.fetchAll
- Menus: scm/title (group 1_header); git.pullpush; scm/history/title
- Keys: none found
- Settings: git.autofetch; git.autofetchPeriod; git.pruneOnFetch; git.fetchOnPull
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; V\extensions\git\src\autofetch.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.fetch, git.fetchPrune, git.fetchAll found)
- Pieces:
  1. Fetch in the title menu fetches from the default remote (`git.fetch`).
  2. Fetch (Prune) fetches and removes remote branches that no longer exist on the remote (`git.fetchPrune`).
  3. Fetch From All Remotes fetches every remote, and a repository with no remotes shows "This repository has no remotes configured to fetch from." (`git.fetchAll`).
  4. Fetch is also available from the Pull, Push submenu and from the Graph title bar.
  5. Setting `git.autofetch` chooses whether Git fetches by itself on a timer, with values `true`, `false` and `all` (all remotes).
  6. Setting `git.autofetchPeriod` sets the number of seconds between automatic fetches.
  7. Setting `git.pruneOnFetch` on makes every fetch prune.

#### SB-4.23 Pull

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pull the current branch, pull with rebase, or pull from a chosen remote and branch, optionally stashing changes first.
- Commands: git.pull; git.pullRebase; git.pullFrom
- Menus: scm/title (group 1_header); git.pullpush
- Keys: none found
- Settings: git.autoStash; git.pullTags; git.fetchOnPull; git.ignoreRebaseWarning; git.replaceTagsWhenPull
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.pull, git.pullRebase, git.pullFrom found)
- Pieces:
  1. Pull in the title menu pulls the current branch from its remote (`git.pull`).
  2. Pull (Rebase) pulls and rebases the branch (`git.pullRebase`).
  3. Pull from... asks "Pick a remote to pull the branch from" and then "Pick a branch to pull from" (`git.pullFrom`).
  4. A repository with no remote shows "Your repository has no remotes configured to pull from."
  5. When a pull looks like the branch was rebased, a warning asks first and offers Always Pull, and `git.ignoreRebaseWarning` hides that warning.
  6. Setting `git.autoStash` on stashes changes before the pull and restores them after a successful pull.
  7. Setting `git.fetchOnPull` on makes a pull fetch all branches instead of only the current one.

#### SB-4.24 Push

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Push the current branch, push to a chosen remote, or force push (shown only when force push is allowed), with a confirmation for force pushes.
- Commands: git.push; git.pushTo; git.pushForce; git.pushToForce
- Menus: scm/title (group 1_header); git.pullpush
- Keys: none found
- Settings: git.allowForcePush; git.useForcePushWithLease; git.useForcePushIfIncludes; git.confirmForcePush; git.showPushSuccessNotification
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.push, git.pushTo, git.pushForce found); C bundle (git.push found)
- Pieces:
  1. Push in the title menu pushes the current branch (`git.push`).
  2. Push to... asks which remote to push to (`git.pushTo`).
  3. A repository with no remote shows "Your repository has no remotes configured to push to." with an Add Remote button.
  4. A branch with no remote branch asks "Would you like to publish this branch?" with OK and OK, Don't Ask Again buttons.
  5. Push (Force) and Push to... (Force) show in the Pull, Push submenu only when `git.allowForcePush` is on (`git.pushForce`, `git.pushToForce`).
  6. A force push asks for confirmation with a message that it can overwrite others' changes, and `git.confirmForcePush` turns that question off.
  7. Settings `git.useForcePushWithLease` and `git.useForcePushIfIncludes` choose the safer force push variants.
  8. Setting `git.showPushSuccessNotification` on shows a notification after a push succeeds.
  9. A push the remote rejects shows "Can't push refs to remote. Try running Pull first to integrate your changes."

#### SB-4.25 Sync changes

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Sync pulls then pushes in one step, from the Sync Changes action button, the Pull, Push submenu, or the Command Palette, with an optional rebase variant and a confirmation.
- Commands: git.sync; git.syncRebase
- Menus: git.pullpush (group 1_sync)
- Keys: none found
- Settings: git.confirmSync; git.rebaseWhenSync; git.followTagsWhenSync; git.supportCancellation
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; V\extensions\git\src\actionButton.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.sync, git.syncRebase found)
- Pieces:
  1. The Sync Changes action button, the Sync entry in the Pull, Push submenu and the Command Palette each pull and then push in one step (`git.sync`).
  2. Sync (Rebase) pulls with rebase and then pushes (`git.syncRebase`).
  3. Sync first asks for confirmation with the message "This action will pull and push commits from and to" the remote branch, with OK and OK, Don't Show Again buttons, and `git.confirmSync` turns it off.
  4. Setting `git.rebaseWhenSync` on makes plain Sync rebase instead of merge.
  5. Setting `git.supportCancellation` on shows a notification while Sync runs that lets you cancel it.

#### SB-4.26 Publish a branch or manage remotes

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Publish Branch pushes a local branch for the first time (and offers a provider such as GitHub when there is no remote), and Add Remote and Remove Remote manage the remote list.
- Commands: git.publish; git.addRemote; git.removeRemote; github.publish
- Menus: git.branch (group 4_publish); git.remotes; scm/history/title
- Keys: none found
- Settings: git.showActionButton
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts (publish, line 4349); V\extensions\github\package.json (github.publish and its scm welcome text); C\extensions\git\package.json; C\extensions\github\package.json (github.publish and two scm welcome entries); C\extensions\git\dist\main.js (git.publish, git.addRemote, git.removeRemote found)
- Pieces:
  1. Publish Branch... pushes a local branch with no remote branch for the first time and asks which remote when there are several (`git.publish`).
  2. With no remote set up, Publish Branch lists providers such as Publish to GitHub (`github.publish`), and shows "Your repository has no remotes configured to publish to." when no provider is available.
  3. Add Remote... in the Remote submenu asks for a URL or a provider and then a remote name, and rejects a name that is badly formed or already used (`git.addRemote`).
  4. Remove Remote in the Remote submenu asks "Pick a remote to remove", and a repository with no remotes shows "Your repository has no remotes." (`git.removeRemote`).
  5. Publish Branch is reachable from the Branch submenu, from the Graph title bar and from the action button under the message box.

#### SB-4.27 Clone a repository

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clone a repository from a URL from the title bar menu, the Command Palette, or the empty Source Control and Explorer views, including a recursive clone and a prompt to open it afterwards.
- Commands: git.clone; git.cloneRecursive; git.continueInLocalClone
- Menus: scm/title (group 1_header); viewsWelcome (scm and explorer)
- Keys: none found
- Settings: git.defaultCloneDirectory; git.openAfterClone
- Source: V\extensions\git\package.json; V\extensions\git\src\cloneManager.ts; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.clone, git.cloneRecursive found); C bundle (git.clone found)
- Pieces:
  1. Clone from the title menu, the Command Palette, or the empty Source Control and Explorer views asks for a source, Clone from URL or a provider, and then for the repository URL (`git.clone`).
  2. After the URL, a folder picker titled "Choose a folder to clone" with a Select as Repository Destination button asks where to put the clone.
  3. A progress notification reads `Cloning git repository "url"...` while the clone runs.
  4. Clone (Recursive) also clones the submodules (`git.cloneRecursive`).
  5. When the clone finishes, a prompt asks "Would you like to open the repository?" with Open, Open in New Window and Add to Workspace buttons.
  6. Setting `git.openAfterClone` chooses what happens after a clone, with values `always`, `alwaysNewWindow`, `whenNoFolderOpen` and `prompt`.
  7. Setting `git.defaultCloneDirectory` sets the default folder that clones go into.
  8. Cloning a repository that is already cloned offers a pick list titled "Open Existing Repository Clone" with a Clone again choice.
  9. Clone Repository Locally and Open on Desktop... clones the current remote repository to this machine (`git.continueInLocalClone`).

#### SB-4.28 See repositories and choose which ones show

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Repositories view (hidden by default) lists every repository with its branch, and you tick which ones the Changes view shows, sort them, collapse or expand them, show count badges, and pick the active repository from a list.
- Commands: scm.setActiveProvider; workbench.scm.action.toggleRepositoryVisibility.<provider id>; workbench.scm.action.repositories.setSortKey.discoveryTime; workbench.scm.action.repositories.setSortKey.name; workbench.scm.action.repositories.setSortKey.path; workbench.scm.action.collapseAllRepositories; workbench.scm.action.expandAllRepositories
- Menus: SCMTitle (View and Sort, Repositories submenu); SCMSourceControlTitle; SCMSourceControlInline
- Keys: none found
- Settings: scm.alwaysShowRepositories; scm.repositories.visible; scm.repositories.sortOrder; scm.providerCountBadge
- Source: V\src\vs\workbench\contrib\scm\browser\scmRepositoriesViewPane.ts; V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (lines 953-1332); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (view descriptors lines 90-105, settings lines 318-338); C bundle (view id workbench.scm.repositories, the sort, collapse and toggle ids, scm.setActiveProvider, settings found)
- Pieces:
  1. The Repositories view, hidden by default, lists every repository with its name and current branch.
  2. A Repositories submenu in the View and Sort menu has a checkable entry for each repository that decides whether the Changes view shows it (`workbench.scm.action.toggleRepositoryVisibility.<provider id>`).
  3. The View and Sort menu has Sort by Discovery Time, Sort by Name and Sort by Path for the repository list, with the active one checked (`workbench.scm.action.repositories.setSortKey.discoveryTime`, `.name`, `.path`).
  4. Setting `scm.repositories.sortOrder` chooses the starting repository order, with values `discovery time`, `name` and `path`.
  5. With several repositories shown, the Changes view has a header for each one, and Collapse All Repositories and Expand All Repositories buttons appear in its title (`workbench.scm.action.collapseAllRepositories`, `workbench.scm.action.expandAllRepositories`).
  6. Setting `scm.alwaysShowRepositories` on shows the repository header in the Changes view even when there is only one repository.
  7. Setting `scm.providerCountBadge` chooses the count badge on repository headers, with values `hidden`, `auto` and `visible`.
  8. Setting `scm.repositories.visible` sets how many repositories the Repositories view shows at once, with a default of 10, and 0 lets you resize it by hand.
  9. A pick list of repositories chooses the active repository (`scm.setActiveProvider`).

#### SB-4.29 Open, close and reopen repositories

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (adds git.rescan; Reopen Closed Repositories sits under scm/sourceControl/title, not scm/repositories/title)
- What: Open a repository by path, close one or all the others from the repository header right-click menu, reopen closed ones, open a repository in a terminal, and tune how Git finds repositories (scan depth, parent folders, ignored paths, submodules); Cursor adds a Rescan Repositories command.
- Commands: git.openRepository; git.close; git.closeOtherRepositories; git.reopenClosedRepositories; git.rescan; git.openRepositoriesInParentFolders; scm.openInTerminal; scm.openInIntegratedTerminal
- Menus: scm/sourceControl; scm/repositories/title (stock); scm/sourceControl/title (Cursor); commandPalette
- Keys: none found
- Settings: git.autoRepositoryDetection; git.scanRepositories; git.ignoredRepositories; git.repositoryScanMaxDepth; git.repositoryScanIgnoredFolders; git.detectSubmodules; git.detectSubmodulesLimit; terminal.sourceControlRepositoriesKind
- Source: V\extensions\git\package.json; V\extensions\git\src\model.ts; V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 538-618, terminal items); C\extensions\git\package.json (git.rescan, scm/sourceControl/title); C\extensions\git\dist\main.js (git.openRepository, git.close, git.closeOtherRepositories, git.reopenClosedRepositories, git.rescan found); C bundle (scm.openInTerminal, scm.openInIntegratedTerminal, terminal.sourceControlRepositoriesKind found)
- Pieces:
  1. Open Repository asks you to pick a folder and adds the repository in it to the list (`git.openRepository`).
  2. Right-clicking a repository header offers Close Repository and Close Other Repositories (`git.close`, `git.closeOtherRepositories`).
  3. Reopen Closed Repositories... opens a pick list that says "Pick a repository to reopen" with an All Repositories choice (`git.reopenClosedRepositories`).
  4. Right-clicking a repository header offers Open in External Terminal or Open in Integrated Terminal, and `terminal.sourceControlRepositoriesKind` chooses which show, with values `external`, `integrated` and `both` (`scm.openInTerminal`, `scm.openInIntegratedTerminal`).
  5. Setting `git.autoRepositoryDetection` chooses when repositories are found by themselves, with values `true`, `false`, `subFolders` and `openEditors`.
  6. Settings `git.scanRepositories` and `git.ignoredRepositories` add folders to search and repositories to skip.
  7. Settings `git.repositoryScanMaxDepth` and `git.repositoryScanIgnoredFolders` set how deep the scan goes and which folders it skips, with `node_modules` skipped by default.
  8. Settings `git.detectSubmodules` and `git.detectSubmodulesLimit` choose whether submodules are listed as repositories and how many.
  9. Rescan Repositories searches the folders again for repositories, and Reopen Closed Repositories is a button on the repository header title. [cursor only]

#### SB-4.30 Browse branches, tags, stashes and worktrees in the Repositories view

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: An experimental mode of the Repositories view shows Branches, Stashes, Tags and Worktrees under the selected repository, each group with its own create button and right-click menu; it works only when repository selection is set to single.
- Commands: git.repositories.checkout; git.repositories.checkoutDetached; git.repositories.createBranch; git.repositories.createTag; git.repositories.merge; git.repositories.rebase; git.repositories.deleteBranch; git.repositories.stashApply; git.repositories.stashPop; git.repositories.stashDrop; git.repositories.createWorktree; git.repositories.openWorktree; git.repositories.deleteWorktree; git.repositories.compareRef; git.repositories.copyBranchName; git.repositories.copyCommitHash; git.repositories.copyStashName; git.repositories.copyTagName; git.repositories.createFrom; git.repositories.deleteTag; git.repositories.openWorktreeInNewWindow; git.repositories.stashCopyBranchName; git.repositories.stashView; git.repositories.worktreeCopyBranchName; git.repositories.worktreeCopyCommitHash; git.repositories.worktreeCopyPath
- Menus: scm/artifactGroup/context; scm/artifact/context; git.repositories.stash; SCMArtifactGroupContext; SCMArtifactContext
- Keys: none found
- Settings: scm.repositories.explorer; scm.repositories.selectionMode
- Source: V\extensions\git\package.json (26 git.repositories.* commands, scm/artifact/context 26 items, scm/artifactGroup/context 4 items); V\extensions\git\src\artifactProvider.ts (groups branches, stashes, tags, worktrees); V\src\vs\workbench\contrib\scm\browser\scmRepositoriesViewPane.ts; V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 339-354); C\extensions\git\package.json (none of these menus or commands); C bundle and C\extensions\git\dist\main.js (scm.repositories.explorer, scm.repositories.selectionMode, scmArtifact, git.repositories. all 0 hits)
- Pieces:
  1. Setting `scm.repositories.explorer` turns on an experimental Repositories view that shows Branches, Stashes, Tags and Worktrees groups under the selected repository. [stock only]
  2. Setting `scm.repositories.selectionMode`, with values `multiple` and `single`, must be `single` for those groups to show. [stock only]
  3. Each group header has its own create button, such as Create Branch, Create Tag or Create Worktree (`git.repositories.createBranch`, `git.repositories.createTag`, `git.repositories.createWorktree`). [stock only]
  4. Right-clicking a branch offers Checkout, Checkout (Detached), Merge, Rebase, Delete Branch and Compare with... (`git.repositories.checkout`, `git.repositories.checkoutDetached`, `git.repositories.merge`, `git.repositories.rebase`, `git.repositories.deleteBranch`, `git.repositories.compareRef`). [stock only]
  5. Right-clicking a stash offers Apply, Pop and Drop (`git.repositories.stashApply`, `git.repositories.stashPop`, `git.repositories.stashDrop`). [stock only]
  6. Right-clicking a worktree offers Open and Delete (`git.repositories.openWorktree`, `git.repositories.deleteWorktree`). [stock only]

#### SB-4.31 Remember open editors per branch

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With the setting on, switching branches saves the editors you had open and brings back the set saved for the branch you switch to; a second setting decides what a branch with no saved set starts with.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: scm.workingSets.enabled; scm.workingSets.default
- Source: V\src\vs\workbench\contrib\scm\browser\workingSet.ts; V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 365-379); C bundle (both settings found)
- Pieces:
  1. With `scm.workingSets.enabled` on, switching branches saves the editors that were open on the branch you leave.
  2. Switching to a branch that has a saved set brings back the editors saved for it.
  3. Setting `scm.workingSets.default` chooses what a branch with no saved set starts with, with values `empty` (no editors) and `current` (keep the editors that are open now), and a default of `current`.

#### SB-4.32 Create, open and delete worktrees

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (adds git.deleteWorktreeFromPalette; git.deleteWorktree2 and git.worktreeIncludeFiles not found)
- What: Create a Git worktree, open one in the current or a new window, delete one, or move changes into a new worktree, from the Worktrees submenu, the repository right-click menu, or the Command Palette.
- Commands: git.createWorktree; git.openWorktree; git.openWorktreeInNewWindow; git.deleteWorktree; git.deleteWorktree2; git.deleteWorktreeFromPalette; git.migrateWorktreeChanges
- Menus: git.worktrees; scm/sourceControl; commandPalette
- Keys: none found
- Settings: git.detectWorktrees; git.detectWorktreesLimit; git.worktreeIncludeFiles
- Source: V\extensions\git\package.json (git.worktrees submenu, 4 items); V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.createWorktree, git.openWorktree, git.deleteWorktree, git.migrateWorktreeChanges, deleteWorktreeFromPalette found; git.deleteWorktree2 not found)
- Pieces:
  1. Create Worktree... asks which branch or tag to start from ("Select a branch or tag to create the new worktree from") and then asks for the worktree path in an input box with a Select Worktree Destination button (`git.createWorktree`).
  2. Picking a branch that is already checked out offers a Create New Branch button instead.
  3. After a worktree is created, a prompt offers Open Worktree in Current Window or Open Worktree in New Window (`git.openWorktree`, `git.openWorktreeInNewWindow`).
  4. Delete Worktree... asks which worktree to delete, and deleting the one you are in shows "You cannot delete the worktree you are currently in" (`git.deleteWorktree`).
  5. Migrate Worktree Changes... asks which worktree to take changes from and brings them into the current one (`git.migrateWorktreeChanges`).
  6. The worktree commands are reachable from the Worktrees submenu, the repository header right-click menu and the Command Palette.
  7. Settings `git.detectWorktrees` and `git.detectWorktreesLimit` choose whether existing worktrees are listed as repositories and how many, with a default of off and 50.
  8. Setting `git.worktreeIncludeFiles` lists glob patterns for ignored files and folders that are copied into a new worktree. [stock only]
  9. Delete Worktree is also available from the Command Palette (`git.deleteWorktreeFromPalette`). [cursor only]

#### SB-4.33 Show Cursor worktrees as repositories

- Tag: cursor. Evidence: bundle. Boundary: yes (the worktrees under the Cursor worktrees folder come from Cursor's agent worktree flow, which is inferred from bundle strings and not confirmed). Cursor: cursor only
- What: By default the git extension hides repositories that live under the Cursor worktrees folder in your home directory unless one is the open workspace, and turning the setting on lists them with your other repositories; the manifest describes it as the Source Control Repositories view.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.showCursorWorktrees
- Source: C\extensions\git\package.json (git.showCursorWorktrees, boolean, scope window, default false); C\extensions\git\dist\main.js (shouldHideCursorWorktreePath, isPathUnderCursorWorktrees, path .cursor/worktrees, shouldRepositoryBeIgnored)
- Pieces:
  1. Repositories that live under the Cursor worktrees folder in your home folder are hidden from the repository lists unless one of them is the open workspace. [cursor only]
  2. Setting `git.showCursorWorktrees` on lists those repositories along with your others. [cursor only]

#### SB-4.34 Browse commit history in the Graph

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (no list or tree switch; the Graph view sits at order 3 behind Agent Review; the hover gets a quick action row with copy hash plus items from the scm/historyItem/hover menu)
- What: The Graph view draws the commit history of the current branch, all branches or chosen refs with branch and tag badges, loads more as you scroll, has title buttons (repository picker, ref picker, go to current commit, refresh, fetch, pull, push or publish), and shows a hover card for each commit.
- Commands: workbench.scm.action.graph.pickRepository; workbench.scm.action.graph.pickHistoryItemRefs; workbench.scm.action.graph.revealCurrentHistoryItem; workbench.scm.action.graph.refresh; workbench.scm.action.graph.setListViewMode; workbench.scm.action.graph.setTreeViewMode; git.fetchAll; git.pullRef; git.pushRef; git.publish; workbench.scm.action.graph.copyHistoryItemId; workbench.scm.history
- Menus: scm/history/title; scm/historyItem/hover (Cursor); SCMHistoryTitle
- Keys: none found
- Settings: scm.graph.pageOnScroll; scm.graph.pageSize; scm.graph.badges
- Source: V\src\vs\workbench\contrib\scm\browser\scmHistoryViewPane.ts (lines 79-290, 483, 1497-1504); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 130-143, 385-405); V\extensions\git\package.json (scm/history/title, 4 items); V\extensions\git\src\historyProvider.ts; C bundle (pickRepository, pickHistoryItemRefs, revealCurrentHistoryItem, refresh found; graph setListViewMode and setTreeViewMode not found; menu id SCMHistoryItemHover and _getHoverActions with copyHistoryItemId; Graph order 3); C\extensions\github\package.json (scm/historyItem/hover)
- Pieces:
  1. The Graph view draws the commit history as a list with graph lines and a row per commit, with branch and tag badges on the right.
  2. Title bar buttons pick the repository, when there are several, and the refs shown, which are the current branch, all branches or chosen refs (`workbench.scm.action.graph.pickRepository`, `workbench.scm.action.graph.pickHistoryItemRefs`).
  3. A Go to Current History Item button scrolls the Graph to the commit you have checked out (`workbench.scm.action.graph.revealCurrentHistoryItem`).
  4. Title bar buttons refresh the Graph, fetch, pull, and push or publish the branch (`workbench.scm.action.graph.refresh`, `git.fetchAll`, `git.pullRef`, `git.pushRef`, `git.publish`).
  5. Scrolling to the end of the list loads the next page when `scm.graph.pageOnScroll` is on, and `scm.graph.pageSize` sets the page size, with a default of 50.
  6. Setting `scm.graph.badges` chooses which badges show, with values `all` and `filter`, and a default of `filter`.
  7. Hovering a commit row shows a hover card with that commit's details.
  8. In Cursor the commit hover card also has a quick action row with copy hash and items from the commit hover menu. [cursor only]
  9. The Graph title bar has View as List and View as Tree buttons (`workbench.scm.action.graph.setListViewMode`, `workbench.scm.action.graph.setTreeViewMode`). [stock only]
  10. In Cursor the Graph view sits in third place in the Source Control container, behind Agent Review. [cursor only]

#### SB-4.35 See incoming and outgoing changes in the Graph

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: The Graph also shows commits you can pull (Incoming Changes) and commits you can push (Outgoing Changes) next to the history, and two settings hide them.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: scm.graph.showIncomingChanges; scm.graph.showOutgoingChanges
- Source: V\src\vs\workbench\contrib\scm\browser\scmHistoryViewPane.ts (lines 398-400, 2051-2053); V\src\vs\workbench\contrib\scm\common\history.ts (SCMIncomingHistoryItemId); V\src\vs\workbench\contrib\scm\browser\scmViewService.ts (lines 248-249); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 407-416); C bundle (both settings, scm-graph-incoming-changes and the labels Incoming Changes and Outgoing Changes: 0 hits)
- Pieces:
  1. Commits you could pull show as an Incoming Changes entry next to the history in the Graph. [stock only]
  2. Commits you could push show as an Outgoing Changes entry next to the history in the Graph. [stock only]
  3. Settings `scm.graph.showIncomingChanges` and `scm.graph.showOutgoingChanges` hide those entries when turned off, and both default to on. [stock only]

#### SB-4.36 Right-click a commit or ref in the Graph

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (6 of 9 git commit items, the 3 Compare items are missing; adds Open in Cursor Blame behind the cursor_blame feature gate; Open Changes is gated on config.multiDiffEditor.experimental.enabled)
- What: Right-click a commit to open its changes, check it out detached, create a branch or tag, cherry-pick it, compare it, open it on GitHub, or copy its hash or message; right-click a branch or tag badge to check it out or delete it.
- Commands: workbench.scm.action.graph.viewChanges; git.graph.checkoutDetached; git.branch; git.createTag; git.graph.cherryPick; git.cherryPick; git.cherryPickAbort; git.graph.compareRef; git.graph.compareWithRemote; git.graph.compareWithMergeBase; git.copyCommitId; git.copyCommitMessage; git.graph.checkout; git.graph.deleteBranch; git.graph.deleteTag; github.graph.openOnGitHub; workbench.scm.action.graph.openInCursorBlame
- Menus: scm/historyItem/context; scm/historyItemRef/context; SCMHistoryItemRefContext
- Keys: none found
- Settings: none found
- Source: V\extensions\git\package.json (scm/historyItem/context 9 items, scm/historyItemRef/context 3 items); V\extensions\github\package.json (github.graph.openOnGitHub); V\src\vs\workbench\contrib\scm\browser\scmHistoryViewPane.ts (lines 288-390); C\extensions\git\package.json (6 and 3 items); C\extensions\git\dist\main.js (git.graph.checkout, git.graph.cherryPick found; git.graph.compareRef, git.graph.compareWithRemote, git.graph.compareWithMergeBase 0 hits); C bundle (workbench.scm.action.graph.viewChanges, workbench.scm.action.graph.openInCursorBlame with checkFeatureGate cursor_blame; NLS labels Open Changes 9351 and Open in Cursor Blame 9353)
- Pieces:
  1. Right-clicking a commit row in the Graph shows a menu of actions for that commit.
  2. Open Changes opens the commit's changed files in one multi-file diff (`workbench.scm.action.graph.viewChanges`).
  3. Checkout (Detached) switches to that commit without a branch (`git.graph.checkoutDetached`).
  4. Create Branch... and Create Tag... start a new branch or tag at that commit (`git.branch`, `git.createTag`).
  5. Cherry Pick applies the commit to the current branch, shows a message when the changes are already present or when conflicts stop it, and Abort Cherry Pick ends a stopped one (`git.graph.cherryPick`, `git.cherryPickAbort`).
  6. Copy Commit Hash and Copy Commit Message put the hash or the message on the clipboard (`git.copyCommitId`, `git.copyCommitMessage`).
  7. Open on GitHub opens the commit's page on GitHub (`github.graph.openOnGitHub`).
  8. Right-clicking a branch or tag badge offers Checkout and Delete Branch or Delete Tag, and delete asks for confirmation first (`git.graph.checkout`, `git.graph.deleteBranch`, `git.graph.deleteTag`).
  9. Compare with..., Compare with Remote and Compare with Merge Base open a comparison of the commit with another ref (`git.graph.compareRef`, `git.graph.compareWithRemote`, `git.graph.compareWithMergeBase`). [stock only]
  10. In Cursor Open Changes needs `multiDiffEditor.experimental.enabled` on, and the menu adds Open in Cursor Blame when the `cursor_blame` feature gate allows it (`workbench.scm.action.graph.openInCursorBlame`). [cursor only]

#### SB-4.37 Refresh the list and read the Git log

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The title bar of the Changes view has Commit and Refresh buttons and a "..." menu (Pull, Push, Clone, Checkout to, Fetch, eight submenus) whose footer has Show Git Output, which opens the output channel listing the Git commands that ran.
- Commands: git.commit; git.refresh; git.showOutput
- Menus: scm/title
- Keys: none found
- Settings: git.autorefresh; git.showProgress; git.commandsToLog
- Source: V\extensions\git\package.json (scm/title, 16 items, and the 8 submenus); V\extensions\git\src\commands.ts; C\extensions\git\package.json (scm/title, same 16 items); C\extensions\git\dist\main.js (git.refresh, git.showOutput found)
- Pieces:
  1. The Changes view title bar has a Commit check button and a Refresh button beside the "..." menu (`git.commit`, `git.refresh`).
  2. Refresh reads the repository status again and updates the changed file list.
  3. The "..." menu lists Pull, Push, Clone, Checkout to... and Fetch at the top, followed by eight submenus for commit, changes, pull and push, branch, remote, stash, tags and worktrees.
  4. Show Git Output at the bottom of the "..." menu opens the Git output channel that lists the Git commands that ran (`git.showOutput`).
  5. Setting `git.showProgress` on shows progress while Git actions run.

#### SB-4.38 Review the branch from the title bar

- Tag: cursor. Evidence: bundle. Boundary: yes (it opens Cursor's Review Changes editor, which the brief treats as agent adjacent). Cursor: cursor only
- What: A diff icon in the Changes view title bar, named Review Branch, opens a Review Changes editor tab that shows your branch against the default branch (the label reads Diff with Default Branch or Diff with the branch name).
- Commands: workbench.scm.action.openReviewChangesPane
- Menus: SCMTitle (navigation group, order -900, when the view is workbench.scm and a repository exists)
- Keys: none found
- Settings: none found
- Source: C bundle (action class with id workbench.scm.action.openReviewChangesPane, title Review Branch from NLS 9381, icon diff, gitMode diffWithMain, labels from NLS 9382 and 9383)
- Pieces:
  1. A diff icon button named Review Branch appears in the Changes view title bar when a repository exists (`workbench.scm.action.openReviewChangesPane`). [cursor only]
  2. Clicking it opens a Review Changes editor tab that shows the branch against the default branch. [cursor only]
  3. The wording reads Diff with Default Branch, or Diff with the branch name. [cursor only]

#### SB-4.39 Run an Agent Review

- Tag: cursor. Evidence: bundle. Boundary: yes (the review and its found issues come from the agent, and Fix All Issues hands them to chat). Cursor: cursor only
- What: The Agent Review view in the Source Control container has a split Start Agent Review button that turns into Stop review while it runs, a status line, a list of issues, a Fix All Issues button and a gear for Agent Review settings; a review can also start from the Command Palette or automatically after you make a local commit that is not yet pushed.
- Commands: workbench.scm.action.triggerAgentReview; workbench.scm.action.openBugbotSettings; cursor.runEditorBugbot; cursor.openBugbotPane; cursor.enableAutoReviewOnCommit
- Menus: ViewTitle (gear, when the view id starts with bugbot); commandPalette
- Keys: none found
- Settings: autoRunBugbotOnCommit
- Source: C bundle (view id bugbot, name Agent Review NLS 9305, order 2, registration class editorBugbotViewGatedRegistration, NLS 9290 Agent Review Settings, NLS 9291 Start Agent Review, CSS classes scm-review-container and scm-review-run-split, autoRunBugbotOnCommit read by name); C\extensions\cursor-commits\dist\main.js (calls cursor.runEditorBugbot after a recent unpushed commit authored by you)
- Pieces:
  1. The Agent Review view in the Source Control container has a split Start Agent Review button. [cursor only]
  2. While a review runs the button reads Stop review. [cursor only]
  3. A status line under the button tells how the review is going. [cursor only]
  4. A list in the view shows the issues the review found. [cursor only]
  5. A Fix All Issues button sends the found issues to chat. [cursor only]
  6. A gear in the view title opens the Agent Review settings (`workbench.scm.action.openBugbotSettings`). [cursor only]
  7. Start Agent Review can also be run from the Command Palette, and `cursor.openBugbotPane` opens the Agent Review view (`workbench.scm.action.triggerAgentReview`, `cursor.runEditorBugbot`). [cursor only]
  8. Setting `autoRunBugbotOnCommit` on starts a review by itself after you make a local commit that is not yet pushed. [cursor only]

#### SB-4.40 Start from an empty Source Control view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With no repository the view shows welcome text with buttons: Open Folder, Clone Repository, Initialize Repository, Add Folder to Workspace, Open Repository from a parent folder, Manage Unsafe Repositories and Reopen Closed Repositories, plus messages for scanning, missing Git, disabled Git and Restricted Mode.
- Commands: git.init; git.cloneRecursive; git.openRepositoriesInParentFolders; git.manageUnsafeRepositories; git.reopenClosedRepositories; vscode.openFolder; workbench.action.addRootFolder; workbench.trust.manage
- Menus: viewsWelcome (scm view)
- Keys: none found
- Settings: git.enabled; git.path; git.ignoreMissingGitWarning
- Source: V\extensions\git\package.json (viewsWelcome, 17 entries for the scm view); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 70-88); C\extensions\git\package.json (19 viewsWelcome entries, same as stock); C bundle (NLS 9294 No source control providers registered, 9295 Restricted Mode text, workbench.trust.manage); C\extensions\git\dist\main.js (git.init, git.manageUnsafeRepositories, git.openRepositoriesInParentFolders found)
- Pieces:
  1. With no folder open, the view shows welcome text with Open Folder and Clone Repository buttons (`vscode.openFolder`, `git.cloneRecursive`).
  2. With a workspace that has no folders, the welcome text offers Add Folder to Workspace and Clone Repository buttons (`workbench.action.addRootFolder`).
  3. With a folder open that is not a repository, the welcome text offers an Initialize Repository button (`git.init`).
  4. Until Git has finished starting, the view shows "Scanning folder for Git repositories..." or "Scanning workspace for Git repositories...".
  5. When a repository is found in a parent folder, the welcome text offers an Open Repository button, and with several it lets you choose (`git.openRepositoriesInParentFolders`).
  6. When Git refuses a repository as unsafe, the welcome text offers a Manage Unsafe Repositories button (`git.manageUnsafeRepositories`).
  7. When repositories were closed, the welcome text offers a Reopen Closed Repositories button (`git.reopenClosedRepositories`).
  8. When Git is not installed the view shows text for the platform with a Download Git link and reload and troubleshoot links.
  9. With setting `git.enabled` off the view asks you to enable Git in your settings.
  10. In Restricted Mode the view says none of the source control providers work there, with a Manage Workspace Trust button (`workbench.trust.manage`), and with nothing else to show it reads "No source control providers registered."

#### SB-4.41 Rename a file with Git

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run Git: Rename from the Command Palette on the file in the active editor, type the new path in an input box (the file name part is pre-selected), and Git moves the file with git mv, closes the old tab and opens the renamed file.
- Commands: git.rename
- Menus: commandPalette (only when the file is in an open repository)
- Keys: none found
- Settings: none found
- Source: V\extensions\git\package.json (command line 323, commandPalette entry line 1349); V\extensions\git\src\commands.ts (rename, lines 1452-1477); V\extensions\git\src\git.ts (args mv, line 2195); C\extensions\git\package.json (git.rename declared); C\extensions\git\dist\main.js (git.rename found)
- Pieces:
  1. Running Git: Rename from the Command Palette opens an input box holding the active file's path with the file name part selected (`git.rename`).
  2. Confirming the new path moves the file with `git mv`, closes the old tab and opens the renamed file.
  3. The command only appears in the Command Palette when the active file is inside an open repository.

#### SB-4.42 Delete a committed file with Git

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (neither the command nor the setting is in the Cursor git manifest, and dist\main.js has no git.delete string)
- What: Run Git: Delete from the Command Palette on the file in the active editor to remove it with git rm, which only works when the file has no uncommitted changes (otherwise an information message says so), and the tab closes if it has no unsaved edits; the confirm setting is declared but nothing in the 1.128.0 source reads it, so no confirmation was found.
- Commands: git.delete
- Menus: commandPalette (only when the file is in an open repository and has the file scheme)
- Keys: none found
- Settings: git.confirmCommittedDelete
- Source: V\extensions\git\package.json (command line 330, commandPalette entry line 1353, setting line 3451); V\extensions\git\src\commands.ts (delete, lines 1479-1512); V\extensions\git\src\repository.ts (rm, line 1303); V\extensions\git\src\git.ts (args rm, line 1979); C\extensions\git\package.json (154 commands, no git.delete, no git.confirmCommittedDelete); C\extensions\git\dist\main.js (0 hits for git.delete and confirmCommittedDelete)
- Pieces:
  1. Running Git: Delete from the Command Palette removes the file in the active editor with `git rm` when it has no uncommitted changes (`git.delete`). [stock only]
  2. A file with uncommitted changes is not removed, and an information message says Git: Delete can only be performed on committed files without uncommitted changes. [stock only]
  3. The tab of the deleted file closes when it has no unsaved edits. [stock only]
  4. The command only appears in the Command Palette when the active file is inside an open repository and uses the file scheme. [stock only]
  5. Setting `git.confirmCommittedDelete` is declared, with a default of on, but nothing reads it, so no confirmation shows. [stock only]

#### SB-4.43 Close diff editors or unmodified editors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: From the Command Palette, Git: Close All Diff Editors closes every open diff tab that compares a file against its Git version (even when git.closeDiffOnOperation is off), and Git: Close All Unmodified Editors closes every open text or notebook tab whose file has no Git changes and no unsaved edits.
- Commands: git.closeAllDiffEditors; git.closeAllUnmodifiedEditors
- Menus: commandPalette
- Keys: none found
- Settings: git.closeDiffOnOperation (the Close All Diff Editors command ignores it)
- Source: V\extensions\git\package.json (commands lines 910-918, commandPalette entries line 1705 and after); V\extensions\git\src\commands.ts (closeDiffEditors and closeUnmodifiedEditors, lines 4940-4972); V\extensions\git\src\repository.ts (closeDiffEditors, lines 1667-1690); C\extensions\git\package.json (both ids declared); C\extensions\git\dist\main.js (both ids found)
- Pieces:
  1. Running Git: Close All Diff Editors closes every open diff tab that compares a file with its Git version, even when `git.closeDiffOnOperation` is off (`git.closeAllDiffEditors`).
  2. Running Git: Close All Unmodified Editors closes every open text or notebook tab whose file has no Git changes and no unsaved edits (`git.closeAllUnmodifiedEditors`).
  3. Both commands are listed in the Command Palette.

#### SB-4.44 Answer Git credential prompts in the window

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a Git command that talks to a remote needs a secret, a prompt opens at the top of the window instead of the terminal: a text box for the username and a masked box for the password (titled Git: host), a masked box for an SSH key passphrase, and a yes or no pick for an unknown SSH host fingerprint; for github.com with the GitHub setting on, the signed-in GitHub account supplies the username and token so no prompt shows (the GitHub sign-in itself is not cataloged here), and the same prompt serves Git run in the integrated terminal.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.useIntegratedAskPass; git.terminalAuthentication; github.gitAuthentication; git.githubAuthentication
- Source: V\extensions\git\src\askpass.ts (lines 60-170); V\extensions\git\src\main.ts (Askpass wiring, lines 75-83); V\extensions\git\package.json (settings useIntegratedAskPass and terminalAuthentication); V\extensions\github\src\credentialProvider.ts (github.com only); V\extensions\github\package.json (github.gitAuthentication); C\extensions\git\package.json (both git settings); C\extensions\github\package.json (github.gitAuthentication); C\extensions\git\dist\main.js (strings Passphrase and Are you sure you want to continue connecting? found)
- Pieces:
  1. A Git command that needs a username opens a prompt at the top of the window with a text box for the username and then a masked box for the password, titled with the host.
  2. A command that needs an SSH key passphrase opens a prompt with a masked box.
  3. A connection to an unknown SSH host opens a yes or no pick that asks "Are you sure you want to continue connecting?"
  4. For github.com with `github.gitAuthentication` on, the signed-in GitHub account supplies the username and token, so no prompt shows.
  5. Setting `git.useIntegratedAskPass` on makes Git ask in the window instead of the terminal.
  6. Setting `git.terminalAuthentication` on makes the same window prompt serve Git commands run in the integrated terminal, after the terminal is restarted.

#### SB-4.45 Clone from a link

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Opening a link of the form scheme://vscode.git/clone?url=REPO_URL (optionally with &ref=BRANCH) makes the app start the Clone flow for that URL, ignores a link with a bad URL, scheme or ref, and shows a modal with a Download Git button if Git is missing; the scheme for Cursor is cursor (product.json urlProtocol), and no link was tried on screen.
- Commands: git.clone
- Menus: none found
- Keys: none found
- Settings: git.defaultCloneDirectory; git.openAfterClone
- Source: V\extensions\git\src\protocolHandler.ts (whole file, handleUri at line 25); V\extensions\git\src\commands.ts (git.clone, line 1032); C\extensions\git\dist\main.js (registerUriHandler, Could not clone your repository as Git is not installed, Invalid reference. found); C\product.json (urlProtocol cursor)
- Pieces:
  1. Opening a link of the form `scheme://vscode.git/clone?url=REPO_URL` starts the Clone flow for that URL (`git.clone`).
  2. Adding `&ref=BRANCH` to the link clones the repository at that ref.
  3. A link with a bad URL, scheme or ref is ignored, and a bad ref shows "Invalid reference."
  4. When Git is missing, the link shows a modal that says it could not clone because Git is not installed, with a Download Git button.
  5. The destination folder and the prompt after cloning follow `git.defaultCloneDirectory` and `git.openAfterClone`.
  6. In Cursor the link scheme is `cursor`, so the link starts with `cursor://`. [cursor only]

#### SB-4.46 Get asked to open a repository from a parent folder

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a Git repository is found in a folder above the workspace or an open file, an information message asks whether to open it with Yes, Always and Never buttons; Yes runs Open Repositories In Parent Folders (a picker with an All Repositories choice when there are several), and Always or Never saves the choice in the setting.
- Commands: git.openRepositoriesInParentFolders
- Menus: none found
- Keys: none found
- Settings: git.openRepositoryInParentFolders
- Source: V\extensions\git\src\model.ts (showParentRepositoryNotification, lines 1166-1190; triggers at lines 319-337 and 632-640); V\extensions\git\src\commands.ts (openRepositoriesInParentFolders, line 4976); V\extensions\git\package.json (setting openRepositoryInParentFolders, values prompt, always, never, default prompt); C\extensions\git\package.json (same setting); C\extensions\git\dist\main.js (the parent folders message strings found)
- Pieces:
  1. When a Git repository is found in a folder above the workspace or an open file, an information message asks whether to open it, with Yes, Always and Never buttons.
  2. Yes runs Open Repositories In Parent Folders, which opens a pick list titled "Pick a repository to open" with an All Repositories choice when there are several (`git.openRepositoriesInParentFolders`).
  3. Always or Never saves the choice in setting `git.openRepositoryInParentFolders`, which has values `prompt`, `always` and `never`, with a default of `prompt`.

#### SB-4.47 Get warned about a missing or old Git

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A warning message appears when Git cannot be found while a workspace folder is a Git repository ("Git not found. Install it or configure it using the git.path setting", with Download Git), when the installed Git is older than 2 ("Code works best with git >= 2", with Update Git), and on Windows when Git 2.25 or 2.26 is installed ("known issues ... update to Git >= 2.27", with Update Git); each has a Don't Show Again button that sets its ignore setting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.ignoreMissingGitWarning; git.ignoreLegacyWarning; git.ignoreWindowsGit27Warning; git.path
- Source: V\extensions\git\src\main.ts (warnAboutMissingGit, lines 159-190; checkGitv1, lines 266-292; checkGitWindows, lines 294-319); V\extensions\git\package.json (the three ignore settings and git.path); C\extensions\git\package.json (same settings); C\extensions\git\dist\main.js (strings Git not found. Install it, Download Git, Update Git, Don't Show Again, Code works best with git >= 2 and known issues with the installed Git found)
- Pieces:
  1. When Git cannot be found while a workspace folder is a Git repository, a warning reads "Git not found. Install it or configure it using the git.path setting" with a Download Git button.
  2. When the installed Git is older than 2, a warning reads "Code works best with git >= 2" with an Update Git button.
  3. On Windows with Git 2.25 or 2.26, a warning says there are known issues and to update to Git 2.27 or later, with an Update Git button.
  4. Each warning has a Don't Show Again button that sets its ignore setting (`git.ignoreMissingGitWarning`, `git.ignoreLegacyWarning`, `git.ignoreWindowsGit27Warning`).
  5. Setting `git.path` gives the path to the Git program, either as one path or as a list of paths to try.

#### SB-4.48 Get warned about an unsafe repository

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When Git refuses a repository because its folder is owned by another user and other repositories are already open, an error message says the repository is potentially unsafe, with Manage Unsafe Repositories (a picker, with an All Repositories choice when there are several, that marks the pick as a safe directory and opens it) and Learn More; with no repository open the empty view's welcome text is used instead (SB-4.40).
- Commands: git.manageUnsafeRepositories
- Menus: commandPalette (only while an unsafe repository is known)
- Keys: none found
- Settings: none found
- Source: V\extensions\git\src\model.ts (showUnsafeRepositoryNotification, lines 1192-1212; triggers at lines 342 and 657); V\extensions\git\src\commands.ts (manageUnsafeRepositories, lines 5009-5053); V\extensions\git\package.json (palette when git.unsafeRepositoryCount != 0); C\extensions\git\package.json; C\extensions\git\dist\main.js (the potentially unsafe message strings and git.manageUnsafeRepositories found)
- Pieces:
  1. When Git refuses a repository because its folder belongs to another user and other repositories are open, an error message says the repository is potentially unsafe, with Manage Unsafe Repositories and Learn More buttons.
  2. Manage Unsafe Repositories opens a pick list titled "Pick a repository to mark as safe and open" with an All Repositories choice when there are several, and the pick is marked safe and opened (`git.manageUnsafeRepositories`).
  3. The Manage Unsafe Repositories command appears in the Command Palette only while an unsafe repository is known.

#### SB-4.49 Expand a commit in the Graph to see its changed files

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (the bundle has no SCMHistoryItemChangeContext menu id and no workbench.scm.action.graph.openFile; it calls provideHistoryItemChanges only from the Open Changes command and the commit click handler, never to fill child rows)
- What: Each commit row in the Graph can be expanded (collapsed by default) to list the files it changed, as a flat list or a folder tree following the Graph's View as List or View as Tree choice; clicking a file opens its diff against the parent commit (an added file opens the file, a deleted file opens the old version), and an inline Open File button opens the file as it was at that commit; the Incoming Changes and Outgoing Changes rows expand the same way; the arrow icon is hidden on commit rows (force-no-twistie), so how a person expands a row was inferred from the tree defaults and not observed.
- Commands: workbench.scm.action.graph.openFile; workbench.scm.action.graph.setListViewMode; workbench.scm.action.graph.setTreeViewMode
- Menus: SCMHistoryItemChangeContext (inline and 0_view groups); scm/history/title (9_viewmode group)
- Keys: none found
- Settings: scm.compactFolders
- Source: V\src\vs\workbench\contrib\scm\browser\scmHistoryViewPane.ts (graph.openFile action, lines 368-410; HistoryItemChangeRenderer, lines 649-750; data source getChildren, lines 906-1018; tree options with collapseByDefault and force-no-twistie, lines 1999-2025; _onDidOpen change branch, lines 2044-2086); V\extensions\git\src\historyProvider.ts (provideHistoryItemChanges, line 338); C bundle (SCMHistory menu ids found are Item, ItemCount, ItemHover, ItemRef and Title only; graph.openFile 0 hits)
- Pieces:
  1. Each commit row in the Graph can be expanded to list the files that commit changed, and rows start collapsed. [stock only]
  2. The files of an expanded commit show as a flat list or as a folder tree, following the Graph's View as List or View as Tree choice (`workbench.scm.action.graph.setListViewMode`, `workbench.scm.action.graph.setTreeViewMode`). [stock only]
  3. Clicking a file of an expanded commit opens its diff against the parent commit, and an added file opens the file while a deleted file opens the old version. [stock only]
  4. An inline Open File button on a file row opens the file as it was at that commit (`workbench.scm.action.graph.openFile`). [stock only]
  5. The Incoming Changes and Outgoing Changes entries expand the same way to list their files. [stock only]
  6. Setting `scm.compactFolders` on joins single child folders into one row in the expanded folder tree. [stock only]

#### SB-4.50 Open a commit's changes by clicking it in the Graph

- Tag: cursor. Evidence: bundle. Boundary: no (when the cursor_blame feature gate and the team setting are on, the click opens a Cursor Blame editor instead; whether Cursor Blame output is agent-driven was not decided, as for SB-4.36). Cursor: cursor only
- What: Clicking a commit row in the Graph opens that commit's changed files in one multi-file diff editor titled with the commit, or in a Cursor Blame editor for the commit when the gate and team setting allow it; stock VS Code 1.128.0 has no open handler for commit rows, only for the changed-file rows and the Load More row.
- Commands: _workbench.openMultiDiffEditor
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (the Graph pane's _onDidOpen with a commit branch: checkFeatureGate cursor_blame, isCursorBlameEnabledForTeam, CursorBlameEditorInput, provideHistoryItemChanges, command _workbench.openMultiDiffEditor, scheme scm-history-item); V\src\vs\workbench\contrib\scm\browser\scmHistoryViewPane.ts (_onDidOpen, lines 2044-2092, no commit branch)
- Pieces:
  1. Clicking a commit row in the Graph opens that commit's changed files in one multi-file diff editor titled with the commit (`_workbench.openMultiDiffEditor`). [cursor only]
  2. When the `cursor_blame` feature gate and the team setting are on, clicking a commit row opens a Cursor Blame editor for that commit instead. [cursor only]

#### SB-4.51 Drag changed files out of the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Drag one or more changed files from the Changes view and the drag carries them as editor and file data, so dropping onto an editor area opens them; the drag label shows the file name or the count, and only file rows drag, not group or folder rows; other drop targets such as the terminal were not tried.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (SCMTreeDragAndDrop, lines 200-250; dnd option, line 1634); C bundle (the same drag class with getDragURI, onDragStart, getDragLabel and getResourcesFromDragAndDropData found)
- Pieces:
  1. Dragging a changed file from the Changes view onto an editor area opens that file.
  2. Dragging several selected files carries all of them together.
  3. The drag label shows the file name for one file and the number of files for several.
  4. Group header and folder rows cannot be dragged, only file rows.

#### SB-4.52 Move focus between commit boxes and file groups with the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Four commands move keyboard focus around the Changes view, between the commit message boxes of the visible repositories and between the file groups; they have no default key, so a person has to bind them.
- Commands: workbench.scm.action.focusPreviousInput; workbench.scm.action.focusNextInput; workbench.scm.action.focusPreviousResourceGroup; workbench.scm.action.focusNextResourceGroup
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 620-670); V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (focusInput, focusResourceGroup, lines 2060-2145); C bundle (all four ids found, each opening the Source Control view and calling the matching focus method)
- Pieces:
  1. Running Focus Next Input or Focus Previous Input opens the Changes view if needed and puts the cursor in the commit message box of the active repository.
  2. When the box already has focus and several repositories are visible, the same command moves to the next or previous repository's box and wraps around at the ends.
  3. With one visible repository and focus already in its box, nothing happens.
  4. Focus Next Resource Group or Focus Previous Resource Group moves focus into the file list and selects the first visible group, such as Staged Changes or Changes, then steps to the next or previous visible group and wraps around.
  5. Groups that are not shown are skipped.
  6. None of the four does anything when no repository is visible, and none has a default key or a menu row, so they show up only once a key is assigned.

#### SB-4.53 Git status colors

- Tag: both. Evidence: source. Boundary: no. Cursor: present - all ten color ids (gitDecoration.addedResourceForeground, modifiedResourceForeground, deletedResourceForeground, renamedResourceForeground, untrackedResourceForeground, ignoredResourceForeground, stageModifiedResourceForeground, stageDeletedResourceForeground, conflictingResourceForeground, submoduleResourceForeground) and their ten labels were found in the Cursor copy of the Git extension manifest and its nls file; how the Source Control list draws them was not checked in the Cursor bundle
- What: Git paints each changed file in a color for its status (modified, added, deleted, renamed or copied, untracked, ignored, conflict, submodule, and the staged forms of modified and deleted), and a theme or the person can change every one of these colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.decorations.enabled; explorer.decorations.colors; workbench.editor.decorations.colors; workbench.colorCustomizations
- Source: V\extensions\git\package.json (contributes.colors lines 4149-4260; setting git.decorations.enabled lines 3578-3582); V\extensions\git\package.nls.json (colors.* lines 338-347); V\extensions\git\src\repository.ts (getStatusLetter line 58, getStatusColor lines 118-151, resourceDecoration line 308, useIcons line 3002); V\extensions\git\src\decorationProvider.ts (ignore provider line 27, submodule badge lines 103-106, enable and disable lines 276-300); V\src\vs\workbench\services\decorations\browser\decorationsService.ts (lines 79-120); V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (lines 591 and 636-642); V\src\vs\workbench\browser\workbench.contribution.ts (lines 109-113); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (lines 551-555); C\extensions\git\package.json and package.nls.json
- Pieces:
  1. A modified or type-changed file is drawn in `gitDecoration.modifiedResourceForeground` (dark #E2C08D, light #895503), and a file modified in the index uses `gitDecoration.stageModifiedResourceForeground`, which starts with the same values so the two can be recolored apart.
  2. An added file (or one marked intent to add) uses `gitDecoration.addedResourceForeground` (dark #81b88b, light #587c0c), while a renamed or copied file uses `gitDecoration.renamedResourceForeground` and an untracked file uses `gitDecoration.untrackedResourceForeground`, and those last two both start at dark #73C991 and light #007100.
  3. A file deleted in the working tree uses `gitDecoration.deletedResourceForeground` (dark #c74e39, light #ad0707), and a file deleted in the index uses `gitDecoration.stageDeletedResourceForeground` with the same starting values.
  4. Every conflict state (both deleted, added or deleted by us or by them, both added, both modified) uses `gitDecoration.conflictingResourceForeground` (dark #e4676b, light #ad0707) and carries a `!` badge.
  5. A file that .gitignore covers is greyed with `gitDecoration.ignoredResourceForeground` (dark #8C8C8C, light #8E8E90), and this grey is still applied when `git.decorations.enabled` is off.
  6. A submodule folder gets an `S` badge with the tooltip Submodule and the color `gitDecoration.submoduleResourceForeground` (dark #8db9e2, light #1258a7).
  7. The color shows on the name and the badge letter of rows in the Explorer and the Open Editors view and on the title of editor tabs, and a changed file also colors the folders above it and puts a small dot on them, except when the change is a deletion.
  8. Setting `git.decorations.enabled` (default on) removes Git colors and badges from the Explorer and the Open Editors view when off, and the settings `explorer.decorations.colors` and `workbench.editor.decorations.colors` (both default on) turn off name colors in those views and in editor tab titles.
  9. In the Source Control list each file shows its status letter in the status color while the file name keeps the normal text color, and with `git.decorations.enabled` off the list shows status icons instead of letters.
  10. There is no settings page for these colors, so a person changes them with `workbench.colorCustomizations` or a theme, and each token has its own defaults for light, dark, high contrast and high contrast light themes.


## SB-5 Run and Debug

(both). Start button and launch picker, then Variables, Watch, Call Stack and Breakpoints while debugging. The Debug Console is PN-6.

31 features.

#### SB-5.1 Open the Run and Debug view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Ctrl+Shift+D, pick Run from the View menu, or click the Run and Debug icon to open the side bar that stacks Variables, Watch, Call Stack, Breakpoints and Loaded Scripts (a Run welcome view stands in before the first run, see SB-5.8).
- Commands: workbench.view.debug
- Menus: MenubarViewMenu (item generated from the container's openCommandActionDescriptor, mnemonic title "&&Run")
- Keys: Ctrl+Shift+D (Cmd+Shift+D on macOS)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:453-475; V\src\vs\workbench\services\views\browser\viewsService.ts:457; C bundle (container id workbench.view.debug with the same Ctrl+Shift+D chord, order 3)
- Pieces:
  1. Opening Run and Debug shows a side bar with the Variables, Watch, Call Stack and Breakpoints views stacked, plus Loaded Scripts when the debugger reports scripts.
  2. Pressing `Ctrl+Shift+D` (`Cmd+Shift+D` on macOS) runs `workbench.view.debug` and shows the Run and Debug side bar.
  3. Choosing Run from the View menu shows the Run and Debug side bar.
  4. Clicking the Run and Debug icon in the activity bar shows the Run and Debug side bar.

#### SB-5.2 Let the view open itself when debugging starts or hits a breakpoint

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Run and Debug view opens on its own - by default when a breakpoint is hit, or on every session start, or only the first one, or never - and the Explorer can be brought back when the session ends.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.openDebug; debug.openExplorerOnEnd
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:514-518, 563-567; V\src\vs\workbench\contrib\debug\browser\debugService.ts:643-650, 791; V\src\vs\workbench\contrib\debug\browser\debugSession.ts:1392; C bundle (debug.openDebug enum with openOnDebugBreak as default, debug.openExplorerOnEnd)
- Pieces:
  1. When a breakpoint is hit during a debug session, the Run and Debug view opens on its own by default.
  2. Setting `debug.openDebug` chooses when the view opens, with values `neverOpen`, `openOnSessionStart`, `openOnFirstSessionStart` and `openOnDebugBreak` (default).
  3. A session started with Run Without Debugging never opens the view when it starts, whatever `debug.openDebug` says.
  4. Setting `debug.openExplorerOnEnd` turned on brings the Explorer back when the last debug session ends and the side bar is visible, and it is off by default.

#### SB-5.3 Start Debugging button in the view title

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the Run and Debug container asks for a second title row, wantsActionToolbar, 70 px tall, to hold the button and picker; the button code and ids are otherwise the same)
- What: A play button (tooltip shows the key) in the view title starts the selected launch configuration (F5 starts it, Ctrl+F5 runs it without the debugger); the button and the gear next to it are hidden in the simple layout (before a configuration exists) and when no folder is open, the button can be hidden while debugging, and "Start Additional Session" appears in the "..." menu once the toolbar is docked or the button is hidden.
- Commands: workbench.action.debug.start; workbench.action.debug.run; workbench.action.debug.selectandstart; debug.openView
- Menus: ViewContainerTitle (group navigation, order 10, for the Run and Debug container); ViewContainerTitle (Start Additional Session, order 10, no group); MenubarDebugMenu (group 1_debug, Start Debugging and Run Without Debugging)
- Keys: F5 (when debuggersAvailable and debugState is inactive); Ctrl+F5 (Ctrl+F5 on macOS, using the Control key); F5 and Ctrl+F5 when no debugger extension exists (opens this view)
- Settings: debug.hideLauncherWhileDebugging; debug.saveBeforeStart; debug.onTaskErrors; debug.hideSlowPreLaunchWarning
- Source: V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts:171-192, 290-303; V\src\vs\workbench\contrib\debug\browser\debugActionViewItems.ts:82-122; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:764-789, 1161-1176; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:272-282, 651-700; C bundle (start-debug-action-item render code, workbench.action.debug.start and run ids, wantsActionToolbar on the container registration and sidebar heightOfTitle 70)
- Pieces:
  1. A play button in the Run and Debug view title starts the selected launch configuration, and its tooltip shows the keyboard shortcut.
  2. Pressing `F5` starts debugging the selected configuration when a debugger is available and no session is running, `Ctrl+F5` runs it without the debugger, and the Start Debugging and Run Without Debugging rows in the Run menu do the same.
  3. When no debugger extension is installed, `F5` and `Ctrl+F5` open the Run and Debug view instead of starting a session.
  4. Before any launch configuration exists, and when no folder is open, the play button and the gear next to it are hidden.
  5. Setting `debug.hideLauncherWhileDebugging` hides the play button while a session is active, and only matters when `debug.toolBarLocation` is not `docked`.
  6. When the debug toolbar is docked or the play button is hidden, the "..." menu in the view title offers Start Additional Session.
  7. Setting `debug.saveBeforeStart` chooses which editors are saved before a session starts, with values `allEditorsInActiveGroup` (default), `nonUntitledEditorsInActiveGroup` and `none`.
  8. Setting `debug.onTaskErrors` chooses what happens when a `preLaunchTask` fails, with values `debugAnyway`, `showErrors`, `prompt` (default) and `abort`.
  9. Setting `debug.hideSlowPreLaunchWarning` turned on hides the warning shown when a `preLaunchTask` has been running for a while, and it is off by default.
  10. In Cursor the Run and Debug container has a taller second title row, 70 px high, that holds the play button and the configuration picker. [cursor only]

#### SB-5.4 Pick a launch configuration from the dropdown

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (an older native-style select box with separator rows, no group categories and no "Search configurations" filter box; stock 1.128.0 uses an action widget dropdown with a filter)
- What: The dropdown beside the play button shows the selected configuration and lists every configuration and compound from all launch scopes (grouped and ordered by launch.json presentation group and order, hidden ones left out), up to three recent dynamic configurations, one "Provider..." row per dynamic provider, and an "Add Configuration..." row per folder, and choosing a row only selects it (the button starts it).
- Commands: workbench.action.debug.start
- Menus: ViewContainerTitle (the dropdown is rendered by the Start Debugging action view item)
- Keys: Right Arrow (button to dropdown); Left Arrow (dropdown to button)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debugActionViewItems.ts:36-353; V\src\vs\workbench\contrib\debug\common\debugSchemas.ts:163-187; V\src\vs\workbench\contrib\debug\browser\debugConfigurationManager.ts:314-326; C bundle (StartDebugActionViewItem code with this.selectBox.setOptions and separator options; NLS has "Debug Launch Configurations" and "No Configurations" but not "Search configurations")
- Pieces:
  1. The dropdown beside the play button shows the name of the selected launch configuration, or "No Configurations" when there are none.
  2. Opening the dropdown lists every launch configuration and compound from all launch scopes in launch.json presentation order, with hidden ones left out.
  3. The list also shows up to three recently used dynamic configurations and one "Provider..." row for each dynamic configuration provider.
  4. The last rows of the list are "Add Configuration..." (or "Add Config (folder)..." in a multi-folder workspace), one per folder.
  5. Choosing a row only selects that configuration, and the play button is what starts it.
  6. Pressing Right Arrow on the play button moves focus to the dropdown, and Left Arrow moves it back.
  7. In Cursor the dropdown is an older plain select box with separator rows, with no group headings and no "Search configurations" filter box. [cursor only]

#### SB-5.5 Open or create launch.json with the gear button

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The gear beside the picker opens the launch.json of the selected configuration's folder (asking which folder or the workspace file when none is selected, or opening the workspace file or the `launch` entry in user settings when the selected configuration lives there), scrolling to the chosen entry; if no file exists, a "Select debugger" list (Suggested, "More X options...", "Install an extension for X...") builds a starter file.
- Commands: workbench.action.debug.configure
- Menus: ViewContainerTitle (group navigation, order 20; a second entry at order 20 shows while debugging with a docked toolbar); MenubarDebugMenu (group 2_configuration, order 1, "Open Configurations"); CommandPalette (f1); DebugCreateConfiguration
- Keys: none found
- Settings: launch
- Source: V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts:194-262; V\src\vs\workbench\contrib\debug\browser\debugConfigurationManager.ts:630-800; V\src\vs\workbench\contrib\debug\browser\debugAdapterManager.ts:382-456; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:611-619; C bundle (workbench.action.debug.configure, NLS "Select debugger", "Suggested", "More {0} options...", "Install an extension for {0}...", "Select a workspace folder to create a launch.json file in or add it to the workspace config file", "user settings")
- Pieces:
  1. Clicking the gear opens the launch.json of the selected configuration's folder, scrolled to the selected entry.
  2. With no configuration selected, the gear asks which workspace folder to create the launch.json in, or whether to add it to the workspace config file.
  3. When the selected configuration lives in the workspace file or in the `launch` setting of user settings, the gear opens that location instead of a launch.json.
  4. When no launch.json exists yet, the gear shows a "Select debugger" list with Suggested debuggers, "More X options..." and "Install an extension for X...", and picking a debugger builds a starter file.
  5. The Run menu row Open Configurations and the Command Palette entry for `workbench.action.debug.configure` open the same launch.json.
  6. While a session runs with a docked toolbar, the view title shows a second gear entry.

#### SB-5.6 Add a configuration to launch.json

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: "Add Configuration..." (from the launch.json editor button, the picker's last row, the Run menu or the palette) inserts a new entry at the top of the `configurations` array and opens the snippet suggestions list so the person can pick a template.
- Commands: debug.addConfiguration
- Menus: EditorContent (only for .vscode/launch.json in a text editor); MenubarDebugMenu (group 2_configuration, order 2); CommandPalette (f1, category Debug)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:985-1015; V\src\vs\workbench\contrib\debug\browser\debugEditorContribution.ts:678-740; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:313-322; V\src\vs\workbench\contrib\debug\browser\debugActionViewItems.ts:325-333; C bundle (debug.addConfiguration id, regex for .vscode launch.json, NLS "Add Configuration..." and "Add Config ({0})...")
- Pieces:
  1. Choosing Add Configuration... inserts a new entry at the top of the `configurations` array in launch.json.
  2. Right after the new entry is inserted, the snippet suggestions list opens so the person can pick a configuration template.
  3. An Add Configuration... button shows inside the editor only when a text editor has `.vscode/launch.json` open.
  4. The last row of the configuration dropdown, the Run menu row Add Configuration... and the Debug entry in the Command Palette all start the same flow.
  5. In a multi-folder workspace the dropdown has one "Add Config (folder)..." row for each folder, and each adds to that folder's launch.json.

#### SB-5.7 Run several configurations at once with a compound

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A `compounds` entry in launch.json (name, list of configuration names or name-and-folder pairs, stopAll, preLaunchTask, presentation) shows up in the picker like a normal configuration and starts all its sessions together, with an error if a named configuration is missing or ambiguous.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: launch
- Source: V\src\vs\workbench\contrib\debug\common\debugSchemas.ts:189-263; V\src\vs\workbench\contrib\debug\browser\debugService.ts:376-435; V\src\vs\workbench\contrib\debug\browser\debugConfigurationManager.ts:372-380, 531-560; C bundle (NLS "List of compounds. Each compound references multiple configurations which will get launched together.", "Controls whether manually terminating one session will stop all of the compound sessions.", "Task to run before any of the compound configurations start.", "Could not find launch configuration")
- Pieces:
  1. A `compounds` entry in launch.json shows up in the configuration dropdown like any other configuration.
  2. Selecting a compound and pressing play starts all the sessions it lists together.
  3. A compound lists its sessions by configuration name, or by name and folder pair in a multi-folder workspace.
  4. The `stopAll` option on a compound makes manually stopping one session stop all of them.
  5. The `preLaunchTask` option on a compound runs a task before any of its sessions start.
  6. The `presentation` option on a compound sets its group, order and hidden state in the dropdown.
  7. Starting a compound shows an error when a named configuration cannot be found or the name is ambiguous.
  8. Setting `launch` in user settings or the workspace file can hold compounds, and they appear in the dropdown too.

#### SB-5.8 Run welcome view before the first run

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Until a configuration is selected or a session has run, the container shows a "Run" welcome view instead of Variables, Watch, Call Stack and Loaded Scripts (Breakpoints stays only if some exist), with an "Open a file which can be debugged or run" link, a "Run and Debug" button, a "create a launch.json file" link (or "open a folder" when none is open) and, when every debug extension is off, a plain-text notice.
- Commands: workbench.action.debug.start; workbench.action.debug.configure; workbench.action.files.openFile; workbench.action.files.openFolder
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\welcomeView.ts:31-167; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:470-476; V\src\vs\workbench\contrib\debug\browser\debugService.ts:311-313; C bundle (view id workbench.debug.welcome with label "Run", NLS "[Open a file](command:{0}) which can be debugged or run.", "To customize Run and Debug [create a launch.json file]({0}).", "All debug extensions are disabled.")
- Pieces:
  1. Until a configuration is selected or a session has run, the Run and Debug container shows a "Run" welcome view instead of the Variables, Watch, Call Stack and Loaded Scripts views.
  2. The Breakpoints view stays beside the welcome view only when some breakpoints exist.
  3. The "Open a file" link in the welcome view, which says "which can be debugged or run", opens the file picker.
  4. The "Run and Debug" button in the welcome view starts debugging.
  5. The "create a launch.json file" link opens the configure flow, and it reads "open a folder" and opens the folder picker when no folder is open.
  6. When every debug extension is disabled, the welcome view shows the plain-text notice "All debug extensions are disabled."

#### SB-5.9 Debug toolbar docked into the view title

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With debug.toolBarLocation set to docked, the Run and Debug title swaps the play button for the debug toolbar while a session is active - Continue or Pause, Step Over, Step Into, Step Out, Restart, Stop with a dropdown (Disconnect, Disconnect and Suspend) and, for adapters that support them, Step Back and Reverse; the default location is a floating bar (ED-11).
- Commands: workbench.action.debug.continue; workbench.action.debug.pause; workbench.action.debug.stepOver; workbench.action.debug.stepInto; workbench.action.debug.stepOut; workbench.action.debug.restart; workbench.action.debug.stop; workbench.action.debug.disconnect; workbench.action.debug.disconnectAndSuspend; workbench.action.debug.stepBack; workbench.action.debug.reverseContinue
- Menus: DebugToolBar (group navigation, orders 10 to 100, mirrored into ViewContainerTitle while docked); DebugToolBarStop (items of the stop dropdown)
- Keys: F5 (Continue, when stopped); F6 (Pause, when running); F10 (Step Over, when stopped); F11 (Step Into, when a session is active; Alt+F11 on Windows web); Shift+F11 (Step Out, when stopped); Shift+F5 (Stop, or Disconnect for attach sessions, in debug mode); Ctrl+Shift+F5 (Restart, in debug mode)
- Settings: debug.toolBarLocation
- Source: V\src\vs\workbench\contrib\debug\browser\debugToolBar.ts:390-491; V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts:100-117; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:471-692; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:535-547; C bundle (menu ids DebugToolBar and DebugToolBarStop, debug.toolBarLocation enum floating, docked, commandCenter, hidden, the same command ids and F-key rules)
- Pieces:
  1. With `debug.toolBarLocation` set to `docked`, the Run and Debug title replaces the play button with the debug toolbar while a session is active, and the play button returns when no session is active.
  2. Setting `debug.toolBarLocation` chooses where the debug toolbar sits, with values `floating` (default), `docked`, `commandCenter` (experimental) and `hidden`.
  3. The docked toolbar shows Continue (or Pause while the program runs), Step Over, Step Into, Step Out, Restart and Stop buttons in that order.
  4. The Stop button has a dropdown with Disconnect and Disconnect and Suspend.
  5. For debuggers that support them, Step Back and Reverse buttons also appear in the docked toolbar.
  6. Pressing `F5` continues when stopped, `F6` pauses when running, `F10` steps over, `F11` steps into, `Shift+F11` steps out, `Shift+F5` stops (or disconnects an attach session) and `Ctrl+Shift+F5` restarts, while debugging.

#### SB-5.10 Switch the focused debug session

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When more than one session runs, a "Debug Session" dropdown in the docked toolbar lists them (child sessions indented, or hidden unless sub-sessions are enabled) and picking one changes which session the Variables, Watch, Call Stack and Breakpoints views show.
- Commands: workbench.action.debug.focusProcess
- Menus: DebugToolBar (group navigation, order 100, only with several sessions and not in the command center); ViewContainerTitle (mirror while docked)
- Keys: none found
- Settings: debug.showSubSessionsInToolBar
- Source: V\src\vs\workbench\contrib\debug\browser\debugActionViewItems.ts:355-430; V\src\vs\workbench\contrib\debug\browser\debugToolBar.ts:456; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:709-721; V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts:100-107; C bundle (workbench.action.debug.focusProcess id, NLS "Debug Session" and "Focus Session", setting debug.showSubSessionsInToolBar)
- Pieces:
  1. When more than one debug session runs, a "Debug Session" dropdown appears in the docked toolbar in the view title, and it is absent with a single session.
  2. Picking a session from the dropdown changes which session the Variables, Watch, Call Stack and Breakpoints views show.
  3. Child sessions are listed indented under their parent in the dropdown.
  4. Setting `debug.showSubSessionsInToolBar` chooses whether sub-sessions appear in the toolbar, with values on and off (default off), and with it off, Stop on a sub-session also stops its parent.

#### SB-5.11 Browse variables by scope

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Variables view shows the scopes (Locals, Globals and so on) of the focused stack frame as a tree, opens the first cheap scope by itself, remembers expansion per frame, resolves lazy values on a click of the eye icon ("Click to expand"), can show each value's type, and has a Collapse All button.
- Commands: workbench.debug.action.focusVariablesView; variables.collapse
- Menus: ViewTitle (Collapse All, view Variables); DebugScopesContext
- Keys: none found
- Settings: debug.showVariableTypes; debug.autoExpandLazyVariables
- Source: V\src\vs\workbench\contrib\debug\browser\variablesView.ts:75-215, 833-851; V\src\vs\workbench\contrib\debug\browser\baseDebugView.ts:154-176; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:470, 491-495, 677-686; C bundle (view id workbench.debug.variablesView, focus command with 3 hits, variables.collapse, NLS "Debug Variables" and "Click to expand")
- Pieces:
  1. The Variables view shows the scopes of the focused stack frame, such as Locals and Globals, as an expandable tree of variables.
  2. When the program stops, the first scope that is cheap to compute opens by itself.
  3. Each stack frame remembers which variable rows were expanded when you come back to it.
  4. A lazy value, such as a getter, shows an eye icon with the tooltip "Click to expand", and clicking it resolves the value.
  5. Setting `debug.autoExpandLazyVariables` chooses whether lazy values resolve by themselves, with values `auto` (default, only in screen reader optimized mode), `on` and `off`.
  6. Setting `debug.showVariableTypes` turned on shows each variable's type next to its value, and it is off by default.
  7. A Collapse All button in the Variables title collapses every open node.
  8. The Command Palette command `workbench.debug.action.focusVariablesView` moves keyboard focus into the Variables view.

#### SB-5.12 Edit a variable's value

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Double-click a value, press F2 (Enter on macOS), or choose Set Value to type a new value in place, when the debugger supports setting variables and the value is not read-only or lazy.
- Commands: debug.setVariable
- Menus: DebugVariablesContext (group 3_modification, order 10)
- Keys: F2 (Enter on macOS, when the Variables view is focused)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\variablesView.ts:211-238, 495-503, 670-681; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:866-886; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:177-181; C bundle (debug.setVariable, NLS "Set Value" and "Type new variable value")
- Pieces:
  1. Double-clicking a variable's value turns it into an input where a new value can be typed in place.
  2. Pressing `F2` (`Enter` on macOS) on a focused variable row opens the same input.
  3. Choosing Set Value in a variable's right-click menu opens the same input.
  4. Set Value is missing for a row when the debugger cannot set variables or the value is read-only or lazy.

#### SB-5.13 Copy a value, an expression or the watch list

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (Copy All is missing, no workbench.debug.viewlet.action.copyAllWatchExpressions id; the other three commands are present)
- What: Right-click a variable or watch row to copy its value or a path that can be pasted as an expression, copy a watch expression's text, or copy every watch as "name: value" lines, and use Ctrl+C on a focused row.
- Commands: workbench.debug.viewlet.action.copyValue; debug.copyEvaluatePath; debug.copyWatchExpression; workbench.debug.viewlet.action.copyAllWatchExpressions
- Menus: DebugVariablesContext (group 5_cutcopypaste, orders 10 and 20); DebugWatchContext (group 3_modification, orders 20 to 50, Copy All at 45)
- Keys: Ctrl+C (Cmd+C on macOS, Variables or Watch focused and no input open); Ctrl+Alt+C (Cmd+Alt+C on macOS, copy watch expression, Watch focused)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:176-224; V\src\vs\workbench\contrib\debug\browser\variablesView.ts:683-732, 810-823; V\src\vs\workbench\contrib\debug\browser\watchExpressionsView.ts:660-720; C bundle (copyValue, copyEvaluatePath and copyWatchExpression ids, NLS "Copy as Expression" and "Copy Expression", no copyAllWatchExpressions id)
- Pieces:
  1. Choosing Copy Value in a variable or watch row's right-click menu copies the value to the clipboard.
  2. Choosing Copy as Expression in a variable's right-click menu copies a path that can be pasted into code as an expression.
  3. Choosing Copy Expression in a watch row's right-click menu copies the watch expression text.
  4. Choosing Copy All in the watch right-click menu copies every watch as lines of "name: value". [stock only]
  5. Pressing `Ctrl+C` (`Cmd+C` on macOS) on a focused Variables or Watch row copies it when no input box is open.
  6. Pressing `Ctrl+Alt+C` (`Cmd+Alt+C` on macOS) on a focused watch row copies the watch expression.

#### SB-5.14 Break when a variable's value changes or is read

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a variable or watch row and choose Break on Value Change, Read or Access (only the ones the debugger offers) to add a data breakpoint that stops when that value is written or read.
- Commands: debug.breakWhenValueChanges; debug.breakWhenValueIsAccessed; debug.breakWhenValueIsRead
- Menus: DebugVariablesContext (group z_commands, orders 200 to 220); DebugWatchContext (group z_commands, orders 200 to 220)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:917-946; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:187-211; V\src\vs\workbench\contrib\debug\browser\watchExpressionsView.ts:410-440; C bundle (the three ids with 1 hit each, NLS "Break on Value Change", "Break on Value Access", "Break on Value Read")
- Pieces:
  1. Choosing Break on Value Change in a variable's right-click menu adds a data breakpoint that stops the program when that value is written.
  2. Choosing Break on Value Read or Break on Value Access adds a data breakpoint that stops when the value is read or accessed.
  3. Each Break on Value row appears only when the debugger offers that kind of data breakpoint for that value.
  4. The same Break on Value rows appear in a watch row's right-click menu.
  5. A data breakpoint added this way shows up in the Breakpoints view.

#### SB-5.15 View a variable's binary data

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A small inline icon on a variable or watch row that points to memory opens its bytes in a hex view, and the first use asks to install the Hex Editor extension (the install itself is the extension system, left out).
- Commands: workbench.debug.viewlet.action.viewMemory
- Menus: DebugVariablesContext (group inline, order 15); DebugWatchContext (group inline, order 10)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\variablesView.ts:734-808; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:183, 205; C bundle (workbench.debug.viewlet.action.viewMemory, ms-vscode.hexeditor, NLS "View Binary Data")
- Pieces:
  1. A small inline icon that points to memory appears on a variable or watch row only while debugging and only when the debugger can show that value's bytes.
  2. Clicking the icon opens the value's bytes in a hex view in the editor area, and it reuses an already open view of the same bytes.
  3. The first click asks to install the Hex Editor extension, with the message "Inspecting binary data requires this extension", before the bytes open.

#### SB-5.16 Notebook Variables view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: An experimental "Notebook Variables" view joins the Run and Debug container when the setting is on (or the REPL editor is active) and the notebook's kernel can list variables, and a right-click on a row offers Copy Value.
- Commands: notebook.openVariablesView; workbench.debug.viewlet.action.copyWorkspaceVariableValue
- Menus: NotebookVariablesContext
- Keys: none found
- Settings: notebook.variablesView
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\notebookVariables\notebookVariables.ts:30-100; V\src\vs\workbench\contrib\notebook\browser\contrib\notebookVariables\notebookVariablesView.ts:35-135; V\src\vs\workbench\contrib\notebook\browser\contrib\notebookVariables\notebookVariableCommands.ts:17-45; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts:1276-1280; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:213; C bundle (view id workbench.notebook.variables, setting notebook.variablesView, notebook.openVariablesView, copyWorkspaceVariableValue, NLS "Notebook Variables")
- Pieces:
  1. An experimental "Notebook Variables" view joins the Run and Debug container when a notebook is open whose selected kernel can list variables.
  2. Setting `notebook.variablesView` turns the view on or off, and it is off by default.
  3. The view also appears when the REPL editor is the active editor, whatever `notebook.variablesView` says.
  4. Right-clicking a variable row in the view offers Copy Value, which copies that variable's value.

#### SB-5.17 Add a watch expression

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Click the + (Add Expression) button in the Watch title, double-click empty space in the view, or right-click a variable and choose Add to Watch, then type an expression in place; every watch is evaluated again at each stop.
- Commands: workbench.debug.viewlet.action.addWatchExpression; debug.addToWatchExpressions; workbench.debug.action.focusWatchView; workbench.debug.watchExpressionsView
- Menus: ViewTitle (group navigation, view Watch); DebugWatchContext (group 3_modification, order 10); DebugVariablesContext (group z_commands, order 100)
- Keys: none found
- Settings: accessibility.debugWatchVariableAnnouncements
- Source: V\src\vs\workbench\contrib\debug\browser\watchExpressionsView.ts:200-216, 611-634; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:185, 201, 471; C bundle (workbench.debug.viewlet.action.addWatchExpression, debug.addToWatchExpressions, focus command with 3 hits, NLS "Add Expression", "Add to Watch", "Expression to watch")
- Pieces:
  1. Clicking the + (Add Expression) button in the Watch title opens an input in the list where an expression can be typed to add it.
  2. Double-clicking empty space in the Watch view opens the same input.
  3. Choosing Add Expression in a watch row's right-click menu opens the same input.
  4. Choosing Add to Watch in a variable's right-click menu adds that variable's expression without typing, and only shows when the variable has an expression.
  5. Every watch expression is evaluated again each time the program stops.
  6. The Command Palette command `workbench.debug.action.focusWatchView` moves keyboard focus into the Watch view.

#### SB-5.18 Edit, remove or reorder watch expressions

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In Watch, press F2 (Enter on macOS) or choose Edit Expression to rename a watch, choose Set Value to change a result, remove one with its X or the Delete key, remove all from the title, drag rows to reorder them, and collapse all.
- Commands: debug.renameWatchExpression; debug.setWatchExpression; debug.removeWatchExpression; workbench.debug.viewlet.action.removeAllWatchExpressions; watch.collapse
- Menus: DebugWatchContext (Edit Expression order 20, Set Value order 30, Remove Expression inline order 20, Remove All Expressions group z_commands order 20); ViewTitle (Remove All Expressions order 20, Collapse All order 30, group navigation, view Watch)
- Keys: F2 (Enter on macOS, when the Watch view is focused); Delete (Cmd+Backspace on macOS, Watch focused and no input open)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\watchExpressionsView.ts:200-216, 520-584, 586-657; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:833-913; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:200-211; C bundle (rename, set and remove watch ids with 1 hit each, watch.collapse, DebugModel moveWatchExpression for drag reorder, NLS "Edit Expression", "Remove Expression", "Remove All Expressions")
- Pieces:
  1. Pressing `F2` (`Enter` on macOS) on a watch row, or choosing Edit Expression in its right-click menu, turns the expression into an input for renaming.
  2. Choosing Set Value in a watch row's right-click menu lets the person type a new value for that watch's result.
  3. Clicking the X on a watch row, or pressing `Delete` (`Cmd+Backspace` on macOS) on a focused row with no input open, removes that watch.
  4. Remove All Expressions in the Watch title or right-click menu removes every watch.
  5. Dragging a watch row up or down changes the order of the watch list.
  6. A Collapse All button in the Watch title collapses every open node.

#### SB-5.19 Browse sessions, threads and stack frames

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Call Stack view lists each debug session with its state, its threads (Running or "Paused on <reason>") and the frames of a paused thread with file and line; clicking a frame opens its source and points Variables and Watch at it, library frames fold into "Show N More" rows, long stacks end in "Load More Stack Frames", and a Collapse All button sits in the title.
- Commands: workbench.debug.action.focusCallStackView; callStack.collapse; workbench.action.debug.callStackTop; workbench.action.debug.callStackBottom; workbench.action.debug.callStackUp; workbench.action.debug.callStackDown
- Menus: ViewTitle (Collapse All, group navigation, order 10, view Call Stack); CommandPalette (the four Navigate commands, shown when stopped)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\callStackView.ts:150-300, 830-940, 1015-1045, 1106-1124; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:406-436; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:147-152, 471; C bundle (view id workbench.debug.callStackView, callStack.collapse, the four callStack navigate ids, NLS "Paused on {0}", "Show {0} More: {1}", "Show {0} More Stack Frames", "Load More Stack Frames", "Debug Call Stack")
- Pieces:
  1. The Call Stack view lists each debug session as a top-level row that shows the session's state.
  2. Each thread under a session shows Running, or "Paused on" and the reason such as a breakpoint or an exception.
  3. A paused thread lists its stack frames, each with the function name, file and line.
  4. Clicking a stack frame opens its source file at that line and points the Variables and Watch views at that frame.
  5. Library frames fold into a single "Show N More" row, and clicking it reveals them.
  6. A long stack ends in a "Load More Stack Frames" row that fetches more frames when clicked.
  7. A Collapse All button in the Call Stack title collapses every open node.
  8. While stopped, the Command Palette offers commands that move the selected frame to the top, bottom, up or down of the call stack, and `workbench.debug.action.focusCallStackView` moves keyboard focus into the view.

#### SB-5.20 Control a session or thread from its row

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Hover a session row, or a thread row (or a session that has one thread), to get inline Pause or Continue, Step Over, Step Into, Step Out, Restart and Stop or Disconnect buttons, and right-click for Disconnect and Suspend and Terminate Thread.
- Commands: workbench.action.debug.pause; workbench.action.debug.continue; workbench.action.debug.stepOver; workbench.action.debug.stepInto; workbench.action.debug.stepOut; workbench.action.debug.restart; workbench.action.debug.stop; workbench.action.debug.disconnect; workbench.action.debug.disconnectAndSuspend; workbench.action.debug.terminateThread
- Menus: DebugCallStackContext (group inline, orders 10 to 60; menu groups 3_modification and termination)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\callStackView.ts:1128-1146; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:164-182; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:360-366, 648-667; C bundle (menu id DebugCallStackContext with 20 hits, the ten command ids, NLS "Disconnect and Suspend", "Terminate Thread")
- Pieces:
  1. Hovering a thread row, or a session row that has only one thread, shows inline buttons for Pause or Continue, Step Over, Step Into, Step Out, Restart and Stop or Disconnect.
  2. A running thread offers Pause and a stopped thread offers Continue, and the three step buttons are disabled unless the thread is stopped.
  3. Right-clicking a session row offers Restart, Disconnect, Stop and, when the debugger supports it, Disconnect and Suspend.
  4. Right-clicking a thread row offers Terminate Thread when the debugger supports it.

#### SB-5.21 Restart a stack frame

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a stack frame, or use the small hover button on it, and choose Restart Frame to re-run the function from its start, when the debugger supports it for that frame.
- Commands: workbench.action.debug.restartFrame
- Menus: DebugCallStackContext (order 10, shown for stack frames when restart is supported); the hover button is built in the row renderer, not a menu item
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:181; V\src\vs\workbench\contrib\debug\browser\callStackView.ts:770-790; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:668-683; C bundle (workbench.action.debug.restartFrame, debug.callStack.restartFrame action, NLS "Restart Frame")
- Pieces:
  1. Choosing Restart Frame in a stack frame's right-click menu re-runs that function from its start.
  2. Hovering a stack frame shows a small button that does the same thing as Restart Frame.
  3. Restart Frame and the hover button are available only when the debugger supports restarting that frame.

#### SB-5.22 Copy the call stack

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click a stack frame and choose Copy Call Stack to put the whole stack of that thread on the clipboard as text.
- Commands: debug.copyStackTrace
- Menus: DebugCallStackContext (group 3_modification, order 20, for stack frames)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:182; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:327-340; C bundle (debug.copyStackTrace, NLS "Copy Call Stack")
- Pieces:
  1. Choosing Copy Call Stack in a stack frame's right-click menu puts every frame of that frame's thread on the clipboard as text, one frame per line.
  2. Copy Call Stack appears only on stack frame rows, not on session or thread rows.

#### SB-5.23 See and open breakpoints

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Breakpoints view lists exception filters first, then function, data, instruction and line breakpoints (file name, folder and a line badge), hover text gives condition, hit count, log message or why one is unverified, the breakpoint that just stopped the program is highlighted, a header warning appears when the debugger left some unverified, and a click opens the code (Ctrl+Enter opens it to the side).
- Commands: workbench.debug.action.focusBreakpointsView; debug.openBreakpointToSide
- Menus: none found
- Keys: Ctrl+Enter (Cmd+Enter on macOS) and Alt+Enter (when the Breakpoints view is focused)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:178-300, 380-500, 506-560, 895-950, 1800-1950; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:1060-1078; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:472; C bundle (view id workbench.debug.breakPointsView, debug.openBreakpointToSide, NLS "Unverified Breakpoint", "Hit after breakpoint", "Instruction breakpoint at address", "Function breakpoints are not supported by this debug type")
- Pieces:
  1. The Breakpoints view lists exception filters first, then function breakpoints, data breakpoints, instruction breakpoints and line breakpoints.
  2. A line breakpoint row shows its file name, its folder and a badge with the line number.
  3. Hovering a breakpoint row shows its condition, hit count or log message, or the reason the debugger left it unverified.
  4. The breakpoint that just stopped the program is highlighted in the list.
  5. When the debugger left some breakpoints unverified, a warning appears in the view header.
  6. Clicking a breakpoint row opens its code, and pressing `Ctrl+Enter` (`Cmd+Enter` on macOS) or `Alt+Enter` on a focused row opens it beside the current editor.
  7. The Command Palette command `workbench.debug.action.focusBreakpointsView` moves keyboard focus into the Breakpoints view.

#### SB-5.24 Enable, disable or skip breakpoints

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Tick a breakpoint's checkbox or press Space to turn it on or off, use Enable All and Disable All, or press the title button Toggle Activate Breakpoints to skip every breakpoint without deleting any; Reapply All Breakpoints re-sends them to a running session.
- Commands: workbench.debug.viewlet.action.enableAllBreakpoints; workbench.debug.viewlet.action.disableAllBreakpoints; workbench.debug.viewlet.action.toggleBreakpointsActivatedAction; workbench.debug.viewlet.action.reapplyBreakpointsAction; debug.toggleBreakpoint
- Menus: ViewTitle (Toggle Activate Breakpoints, group navigation, order 20, view Breakpoints); DebugBreakpointsContext (group z_commands: Enable All order 10, Disable All order 20, Reapply All order 30); MenubarDebugMenu (group 5_breakpoints, Enable All order 1, Disable All order 2)
- Keys: Space (when the Breakpoints view is focused and no input has focus)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:280-292, 2149-2170, 2243-2322; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:790-807; C bundle (the four viewlet action ids with 1 hit each, debug.toggleBreakpoint, NLS "Toggle Activate Breakpoints", "Reapply All Breakpoints", "Enable All Breakpoints", "Disable All Breakpoints")
- Pieces:
  1. Ticking or clearing the checkbox on a breakpoint row turns that breakpoint on or off without removing it.
  2. Pressing `Space` on a focused breakpoint row, with no input box focused, toggles that breakpoint.
  3. Enable All Breakpoints and Disable All Breakpoints, in the Breakpoints right-click menu and the Run menu, switch every breakpoint on or off at once.
  4. The Toggle Activate Breakpoints button in the Breakpoints title makes the debugger skip every breakpoint without deleting any, and pressing it again turns them back on.
  5. Reapply All Breakpoints in the Breakpoints right-click menu sends every breakpoint again to a running session.

#### SB-5.25 Remove breakpoints

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Remove a breakpoint with the X on its row, the Delete key, a middle-click or Remove Breakpoint in its menu, or clear every kind at once with Remove All Breakpoints.
- Commands: workbench.debug.viewlet.action.removeBreakpoint; debug.removeBreakpoint; workbench.debug.viewlet.action.removeAllBreakpoints
- Menus: DebugBreakpointsContext (Remove Breakpoint in group 3_modification and inline; Remove All Breakpoints in group 3_modification, order 20); ViewTitle (Remove All Breakpoints, group navigation, order 30, view Breakpoints); MenubarDebugMenu (group 5_breakpoints, order 3)
- Keys: Delete (Cmd+Backspace on macOS, when the Breakpoints view is focused and no condition box has focus)
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:242-255, 2171-2241; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:947-970; C bundle (workbench.debug.viewlet.action.removeBreakpoint and removeAllBreakpoints, debug.removeBreakpoint, NLS "Remove Breakpoint", "Remove All Breakpoints")
- Pieces:
  1. Clicking the X on a breakpoint row removes that breakpoint.
  2. Pressing `Delete` (`Cmd+Backspace` on macOS) on a focused breakpoint row removes it, unless a condition box has focus.
  3. Middle-clicking a breakpoint row removes it.
  4. Choosing Remove Breakpoint in a breakpoint's right-click menu removes it.
  5. Remove All Breakpoints, from the Breakpoints title, its right-click menu or the Run menu, clears every kind of breakpoint at once.

#### SB-5.26 Edit a breakpoint's condition, hit count or mode

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the mode command is titled "Manual Mode..." where stock says "Edit Mode..."; the other three commands and their labels are present)
- What: Right-click a breakpoint, or press the pencil on its row, to edit its condition, hit count or mode in an inline box (a line breakpoint opens the editor's breakpoint box instead, see ED-11); function breakpoints get Edit Function Condition and Edit Hit Count, data breakpoints get Edit Hit Count, and debuggers that define modes add a mode pick.
- Commands: debug.editBreakpoint; debug.editFunctionBreakpoint; debug.editFunctionBreakpointHitCount; debug.editBreakpointMode
- Menus: DebugBreakpointsContext (group navigation, orders 10 to 20; Edit Condition also inline)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:2349-2486; C bundle (debug.editBreakpointMode registration with title NLS entry "Manual Mode...", "Select Breakpoint Mode", "Edit Condition...", "Edit Hit Count...", "Edit Function Condition...")
- Pieces:
  1. Choosing Edit Condition... in a breakpoint's right-click menu opens an inline box on the row to type the condition, hit count or mode.
  2. A pencil button on a breakpoint row opens the same inline Edit Condition box.
  3. Editing a line breakpoint opens the editor's breakpoint box instead of an inline box in the list.
  4. Function breakpoints offer Edit Function Condition... and Edit Hit Count... in their right-click menu.
  5. Data breakpoints offer Edit Hit Count... in their right-click menu.
  6. When the debugger defines breakpoint modes, the right-click menu adds a mode row that opens a "Select Breakpoint Mode" pick.
  7. In Cursor the breakpoint mode row is titled "Manual Mode..." where stock VS Code says "Edit Mode...". [cursor only]

#### SB-5.27 Add a function breakpoint

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Click the function-breakpoint button in the Breakpoints title (or use New Breakpoint then Function Breakpoint in the Run menu) and type a function name in the inline box that says "Function to break on"; double-click a function row to rename it.
- Commands: workbench.debug.viewlet.action.addFunctionBreakpointAction
- Menus: ViewTitle (group navigation, order 10, view Breakpoints); MenubarNewBreakpointMenu (group 1_breakpoints, order 3); CommandPalette (f1)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:270-280, 1480-1500, 1953-1983; C bundle (workbench.debug.viewlet.action.addFunctionBreakpointAction, NLS "Add Function Breakpoint", "Function to break on")
- Pieces:
  1. Clicking the function breakpoint button in the Breakpoints title opens an inline box with the placeholder "Function to break on", and typing a name adds the function breakpoint.
  2. Choosing New Breakpoint then Function Breakpoint in the Run menu, or Add Function Breakpoint in the Command Palette, opens the same inline box.
  3. Double-clicking a function breakpoint row turns its name into an input for renaming.

#### SB-5.28 Add a data breakpoint at a memory address

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the debugger can break on bytes, a title button (or New Breakpoint then Data Breakpoint in the Run menu) asks for a range such as 0x1234 + 0xff and then an access type (read, write or access), and Edit Address on the row changes the range later.
- Commands: workbench.debug.viewlet.action.addDataBreakpointOnAddress; workbench.debug.viewlet.action.editDataBreakpointOnAddress
- Menus: ViewTitle (group navigation, order 11, view Breakpoints); MenubarNewBreakpointMenu (group 1_breakpoints, order 4); DebugBreakpointsContext (Edit Address, group navigation, order 15)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:1984-2147; C bundle (both ids with 1 hit, NLS "Add Data Breakpoint at Address", "Edit Address...", "Enter a memory range in which to break", "Select the access type to monitor")
- Pieces:
  1. When the debugger can break on bytes, a button in the Breakpoints title, or New Breakpoint then Data Breakpoint in the Run menu, asks for a memory range to watch, written like `0x1234 + 0xff`.
  2. After the range is entered, a pick asks which access type to watch, with read, write or access.
  3. The new data breakpoint appears in the Breakpoints list as a data breakpoint row.
  4. Choosing Edit Address... in the data breakpoint's right-click menu lets the person change the range later.

#### SB-5.29 Exception breakpoints

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (exception rows, the condition box and its labels are present; the Toggle Exception Breakpoints command and its label are missing)
- What: Exception filters offered by the debugger (for example all or uncaught exceptions) sit at the top of the Breakpoints list with checkboxes, some take a condition typed in place, and a palette command toggles several at once.
- Commands: debug.toggleExceptionBreakpoints; debug.editBreakpoint
- Menus: CommandPalette (f1, category Debug); DebugBreakpointsContext (Edit Condition on exception rows)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:508-520, 990-1060, 1660-1730; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:1080-1160; C bundle (NLS "Unverified Exception Breakpoint", "Expression condition: {0}", "Type exception breakpoint condition"; no debug.toggleExceptionBreakpoints id or "Toggle Exception Breakpoints" label)
- Pieces:
  1. Exception filters offered by the debugger, such as all exceptions or uncaught exceptions, sit at the top of the Breakpoints list, each with a checkbox.
  2. An exception filter that supports a condition shows Edit Condition... in its right-click menu and opens an inline box for typing it, with the placeholder "Break when expression evaluates to true" unless the debugger gives its own hint.
  3. A condition set on an exception filter shows in its row beside the name, and hovering it reads "Expression condition:" followed by the condition.
  4. A Toggle Exception Breakpoints command in the Command Palette turns several exception filters on or off in one pick. [stock only]

#### SB-5.30 Show breakpoints as a list or as a tree by file

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: A title button flips the Breakpoints view between a flat list (file name, folder, line badge) and a tree grouped by file, where each file row has a tri-state checkbox and a "Remove Breakpoints in File" button and each line row shows that line's code.
- Commands: workbench.debug.viewlet.action.toggleBreakpointsPresentation
- Menus: ViewTitle (group navigation, order 10, view Breakpoints)
- Keys: none found
- Settings: debug.breakpointsView.presentation
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:135-139, 500-560, 740-830, 895-975, 2325-2347; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:641-646; C bundle (searched for the command id, the setting key and the label "Toggle Breakpoints View Presentation" and the setting text, none found)
- Pieces:
  1. A button in the Breakpoints title flips the view between a flat list and a tree grouped by file. [stock only]
  2. In list mode each row shows the file name, the folder and a line badge. [stock only]
  3. In tree mode each file row has a checkbox that is checked, unchecked or partly checked depending on its breakpoints. [stock only]
  4. In tree mode each file row shows a "Remove Breakpoints in File" button on hover. [stock only]
  5. In tree mode each line row shows the code on that line. [stock only]
  6. Setting `debug.breakpointsView.presentation` chooses how breakpoints are shown, with values `list` (default) and `tree`. [stock only]

#### SB-5.31 Browse and open loaded scripts

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Loaded Scripts view (collapsed by default and shown only for debuggers that report scripts) lists every script the program loaded, grouped by session and folder, opens one read-only on click, narrows by name as you type, and has a Collapse All button; a palette command shows the same list as a quick pick.
- Commands: loadedScripts.collapse; workbench.action.debug.showLoadedScripts
- Menus: ViewTitle (Collapse All, group navigation, order 30, view Loaded Scripts); CommandPalette (Open Loaded Script, when in debug mode and the debugger supports it)
- Keys: none found
- Settings: debug.closeReadonlyTabsOnEnd
- Source: V\src\vs\workbench\contrib\debug\browser\loadedScriptsView.ts:396-410, 506-520, 601-625, 763-805; V\src\vs\workbench\contrib\debug\common\loadedScriptsPicker.ts:1-108; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:694-699; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:147, 476; C bundle (view id workbench.debug.loadedScriptsView, loadedScripts.collapse, workbench.action.debug.showLoadedScripts, NLS "Loaded Scripts", "Open Loaded Script...", "Debug Loaded Scripts")
- Pieces:
  1. The Loaded Scripts view lists every script the program has loaded, grouped by session and then by folder.
  2. The view is collapsed by default and shows only for debuggers that report loaded scripts.
  3. Clicking a script opens it read-only in the editor.
  4. Typing while the Loaded Scripts tree is focused narrows the list to scripts whose names match.
  5. A Collapse All button in the Loaded Scripts title collapses every open node.
  6. The Open Loaded Script... command in the Command Palette, available while debugging when the debugger supports it, shows the same scripts as a quick pick.


## SB-6 Testing

(both). The Test Explorer that lists, runs and debugs tests, plus the coverage view. Test Results is PN-6.

30 features.

#### SB-6.1 Open the Testing view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Open the Testing side bar from View > Testing or the "Show Testing" command. The container stays hidden until a test extension registers a provider, then it shows the Test Explorer, normally as its only view (so the container title "Testing" doubles as the view header; the Test Coverage view joins it during a coverage run), with a preferred width of 400 px.
- Commands: workbench.view.extension.test; workbench.view.testing.focus (built from the view id at run time)
- Menus: MenubarViewMenu (group 3_sidebar, label "T&&esting")
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts (container 62-79, Test Explorer view 112-131); V\src\vs\workbench\contrib\testing\browser\testingViewPaneContainer.ts; V\src\vs\workbench\services\views\browser\viewsService.ts (open action 401-462, focus action 557-562); C bundle (container id workbench.view.extension.test, hideIfEmpty, view id workbench.view.testing); C out\nls.messages.json ("Testing", "T&&esting", "Test Explorer")
- Pieces:
  1. The Testing container stays hidden in the side bar until a test extension registers a provider, and then it shows the Test Explorer view.
  2. Choosing Testing in the View menu opens the Testing side bar.
  3. The Show Testing command in the Command Palette (`workbench.view.extension.test`) opens the Testing side bar.
  4. The command `workbench.view.testing.focus` moves keyboard focus into the Test Explorer view.
  5. While the Test Explorer is the only view in the container, the container title "Testing" doubles as the view header.
  6. The Testing side bar opens at a preferred width of 400 px.

#### SB-6.2 Switch between tree and list, and collapse all

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Show the tests as a nested tree (controller, file, suite, test) or as a flat list of tests, and collapse every open node with one action. The mode is remembered per workspace and defaults to tree.
- Commands: testing.viewAsTree; testing.viewAsList; testing.collapseAll
- Menus: ViewTitle (group viewAs, only for view workbench.view.testing); ViewTitle (group displayAction, same view)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (751-798, 903-924); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (view mode 719-731, projection switch 1130-1148); V\src\vs\workbench\contrib\testing\browser\explorerProjections\treeProjection.ts; V\src\vs\workbench\contrib\testing\browser\explorerProjections\listProjection.ts; C bundle (ids testing.viewAsTree, testing.viewAsList, testing.collapseAll; storage key testing.viewMode); C out\nls.messages.json ("View as Tree", "View as List", "Collapse All Tests")
- Pieces:
  1. Choosing View as Tree shows the tests as a nested tree of controller, file, suite and test.
  2. Choosing View as List shows the tests as a flat list of tests.
  3. Collapse All Tests collapses every open node in the tree with one action.
  4. The View as Tree, View as List and Collapse All Tests rows are in the view's title menu, the tree mode is the default, and the chosen mode is remembered per workspace.

#### SB-6.3 Sort tests

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Order the tests by Status, by Location, or by Duration. The choice is a checked row in the view's menu, is remembered per workspace, and defaults to Location.
- Commands: testing.sortByStatus; testing.sortByLocation; testing.sortByDuration
- Menus: ViewTitle (group sortBy, only for view workbench.view.testing)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (800-870); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (sorting 733-747, TreeSorter 1311-1349); V\src\vs\workbench\contrib\testing\common\constants.ts (30-34); C bundle (ids testing.sortByStatus, testing.sortByLocation, testing.sortByDuration; storage key testing.viewSorting); C out\nls.messages.json ("Sort by Status", "Sort by Location", "Sort by Duration")
- Pieces:
  1. Choosing Sort by Status orders the tests by their result status.
  2. Choosing Sort by Location orders the tests by where they sit in the files.
  3. Choosing Sort by Duration orders the tests by how long they took to run.
  4. The three sort choices are rows in the view's title menu with a check on the active one, and Sort by Location is the default.
  5. The chosen sort order is remembered per workspace.

#### SB-6.4 Filter tests by text and tag

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Type in the filter box above the tree to show only matching tests. A comma separates terms, a leading ! excludes, and @controller:tag filters by a test tag. Typing @ opens a suggestion list of the filter terms and known tags, the box keeps a history, and typing a character while the tree is focused jumps into the box. Placeholder text is "Filter (e.g. text, !exclude, @tag)".
- Commands: workbench.actions.treeView.testExplorer.filter
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerFilter.ts (35-160); V\src\vs\workbench\contrib\testing\common\testExplorerFilterState.ts (95-210); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (836-855, 880-886, 1202-1308); C bundle (id workbench.actions.treeView.testExplorer.filter, element id testing.explorer.filter, storage key testing.filterHistory2); C out\nls.messages.json ("Filter (e.g. text, !exclude, @tag)", "Filter text for tests in the explorer")
- Pieces:
  1. Typing in the filter box above the tree shows only the tests that match the text.
  2. A comma separates filter terms, and a leading `!` on a term excludes matching tests.
  3. A term written as `@controller:tag` shows only tests that carry that test tag.
  4. Typing `@` in the filter box opens a suggestion list of the filter terms and the known tags.
  5. The filter box keeps a history of earlier filters that can be recalled.
  6. Typing a character while the tree is focused jumps into the filter box.
  7. The empty filter box shows the placeholder "Filter (e.g. text, !exclude, @tag)".

#### SB-6.5 Narrow the tree with the More Filters menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The funnel button beside the filter box opens a menu with Show Only Failed Tests, Show Only Executed Tests, Show in Active File Only, Show in Opened Files Only, Fuzzy Match, Show Hidden Tests and Unhide All Tests. Each toggle writes its term (@failed, @executed, @doc, @openedFiles, @hidden) into the filter box, and the button looks checked while any test is hidden. Fuzzy Match changes how the text in the box matches and is remembered across workspaces.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerFilter.ts (27-33, 175-250); V\src\vs\workbench\contrib\testing\common\testExplorerFilterState.ts (110-120, 227-241); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (TestsFilter 1202-1309); C bundle (storage key testHistoryFuzzy); C out\nls.messages.json ("More Filters...", "Fuzzy Match", "Show Hidden Tests", "Show Only Failed Tests", "Show Only Executed Tests", "Show in Active File Only", "Show in Opened Files Only", "Unhide All Tests")
- Pieces:
  1. A funnel button beside the filter box opens a More Filters menu with Show Only Failed Tests, Show Only Executed Tests, Show in Active File Only, Show in Opened Files Only, Fuzzy Match, Show Hidden Tests and Unhide All Tests.
  2. Show Only Failed Tests and Show Only Executed Tests are toggles that write `@failed` and `@executed` into the filter box.
  3. Show in Active File Only and Show in Opened Files Only are toggles that write `@doc` and `@openedFiles` into the filter box.
  4. Fuzzy Match changes how the text in the filter box matches test names, and the choice is remembered across workspaces.
  5. Show Hidden Tests is a toggle that writes `@hidden` into the filter box, and Unhide All Tests makes every hidden test visible again.
  6. The funnel button looks checked while any test is hidden.

#### SB-6.6 Run all tests

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run every test with the default Run profile from the Command Palette or the key chord, or press the Run Tests button in the view title to run just the tests currently visible after filtering. While a run is going the button is replaced by Cancel (see SB-6.13). If no tests exist, an info message says a test provider extension may be needed. Dirty editors are saved first when the setting is on.
- Commands: testing.runAll; testing.runSelected
- Menus: ViewTitle (group navigation, hidden while running); CommandPalette
- Keys: Ctrl/Cmd+; then A (testing.runAll)
- Settings: testing.saveBeforeTest
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (513-543, 586-593, 626-669); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (getTreeIncludeExclude 156-268, title button 311-329); V\src\vs\workbench\contrib\testing\common\testServiceImpl.ts (save before run 443-444); C bundle (ids testing.runAll, testing.runSelected, keybinding chord Ctrl/Cmd+; A); C out\nls.messages.json ("Run All Tests", "Run Tests")
- Pieces:
  1. Choosing Run All Tests in the Command Palette, or pressing `Ctrl/Cmd+;` then `A`, runs every test with the default Run profiles.
  2. The Run Tests button in the view title runs only the tests that are currently visible after filtering.
  3. When no tests exist, running all tests shows an info message that says a test provider extension may need to be installed.
  4. Setting `testing.saveBeforeTest` saves all dirty editors before a run starts, and it is on by default.

#### SB-6.7 Run one test or a group from its row

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Hover a test, suite or file row to get a Run Test button, or pick Run Test from its right-click menu. Pressing Enter in the tree runs the focused row, or all selected rows when the focused row is part of the selection. Selecting several rows and using a row action runs them all.
- Commands: testing.run
- Menus: TestItem (group inline, and group builtin@1)
- Keys: Enter in the focused tree (hard-coded key handler, not a keybinding rule)
- Settings: testing.saveBeforeTest
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (150-183, 243-252); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (880-886, 1092-1111, action runner 1372-1388, row actions 1572-1580); C bundle (id testing.run, TestItem menu rows); C out\nls.messages.json ("Run Test")
- Pieces:
  1. Hovering a test, suite or file row shows a Run Test button that runs that test, or every test under that suite or file.
  2. Choosing Run Test in a row's right-click menu runs the same tests.
  3. Pressing `Enter` in the tree runs the focused row, or every selected row when the focused row is part of the selection.
  4. Selecting several rows and using a row action runs all of the selected rows.
  5. Setting `testing.saveBeforeTest` saves all dirty editors before a row run starts.

#### SB-6.8 Debug tests

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Start a debug session instead of a plain run. Use the Debug Tests button in the view title for the visible tests, the Debug Test button on a row or in its right-click menu for one test, or Debug All Tests from the Command Palette. These only appear when a test extension has registered a debug profile.
- Commands: testing.debugAll; testing.debugSelected; testing.debug
- Menus: ViewTitle (group navigation, hidden while running); TestItem (group inline, and group builtin@1); CommandPalette
- Keys: Ctrl/Cmd+; then Ctrl/Cmd+A (testing.debugAll)
- Settings: testing.saveBeforeTest
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (185-194, 596-603, 626-632, 671-687); V\src\vs\workbench\contrib\testing\common\testingContextKeys.ts (hasDebuggableTests); C bundle (ids testing.debugAll, testing.debugSelected, testing.debug); C out\nls.messages.json ("Debug All Tests", "Debug Tests", "Debug Test")
- Pieces:
  1. The Debug Tests button in the view title starts a debug session for the tests that are currently visible.
  2. The Debug Test button on a row, or Debug Test in its right-click menu, debugs that one test.
  3. Choosing Debug All Tests in the Command Palette, or pressing `Ctrl/Cmd+;` then `Ctrl/Cmd+A`, debugs every test.
  4. The Debug buttons and menu rows appear only when a test extension has registered a debug profile.
  5. Setting `testing.saveBeforeTest` saves all dirty editors before a debug run starts.

#### SB-6.9 Run tests with coverage

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run tests and collect coverage. Use the Run Tests with Coverage button in the view title for the visible tests, Run Test with Coverage on a row or in its menu for one test, or Run All Tests with Coverage from the Command Palette. A finished run that produced coverage opens the Test Coverage view (SB-6.24).
- Commands: testing.coverageAll; testing.coverageSelected; testing.coverage
- Menus: ViewTitle (group navigation, hidden while running); TestItem (group inline, and group builtin@1); CommandPalette
- Keys: Ctrl/Cmd+; then Ctrl/Cmd+Shift+A (testing.coverageAll)
- Settings: testing.saveBeforeTest
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (196-205, 605-615, 689-705); V\src\vs\workbench\contrib\testing\common\testCoverageService.ts (100-145); C bundle (ids testing.coverageAll, testing.coverageSelected, testing.coverage); C out\nls.messages.json ("Run All Tests with Coverage", "Run Tests with Coverage", "Run Test with Coverage")
- Pieces:
  1. The Run Tests with Coverage button in the view title runs the visible tests and collects coverage.
  2. The Run Test with Coverage button on a row, or the same entry in its right-click menu, runs one test with coverage.
  3. Choosing Run All Tests with Coverage in the Command Palette, or pressing `Ctrl/Cmd+;` then `Ctrl/Cmd+Shift+A`, runs every test with coverage.
  4. When a coverage run finishes with coverage data, the Test Coverage view opens in the Testing container.
  5. Setting `testing.saveBeforeTest` saves all dirty editors before a coverage run starts.

#### SB-6.10 Run with a chosen profile

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a test extension offers more than one profile for Run, Debug or Coverage, the title button becomes a split button. Its drop-down lists each controller's profiles (the default is marked "(Default)") and runs the visible tests with the one you pick. A row's right-click menu has Execute Using Profile... to pick a profile for that test. Extensions can add rows to the drop-down through the testing/profiles/context menu.
- Commands: testing.runUsing; vscode.pickTestProfile
- Menus: TestItem (group builtin@2, when testing.hasNonDefaultProfile); TestProfilesContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (title drop-down 311-451); V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (207-241); V\src\vs\workbench\contrib\testing\browser\testingConfigurationUi.ts (33-90, 133-160); V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts (361); C bundle (ids testing.runUsing, vscode.pickTestProfile, menu TestProfilesContext, context key testing.profile.context.group); C out\nls.messages.json ("Execute Using Profile...", "Select Configuration...", "{0} (Default)", "Pick a test profile to use")
- Pieces:
  1. When a test extension offers more than one profile for Run, Debug or Coverage, the title button becomes a split button with a drop-down arrow.
  2. The drop-down lists each controller's profiles, and the default profile is marked "(Default)".
  3. Picking a profile in the drop-down runs the visible tests with that profile.
  4. Choosing Execute Using Profile... in a row's right-click menu opens a "Pick a test profile to use" list and runs that test with the pick.
  5. Extensions can add their own rows to the drop-down through the `testing/profiles/context` menu.

#### SB-6.11 Set the default profile and configure profiles

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Choose which profiles count as the default for Run, Debug and Coverage (a multi-select picker), and open a profile's own configuration step. Both are rows at the bottom of the title-button drop-down (Select Default Profile when more than one profile exists, Configure Test Profiles when one can be configured). Configure Test Profiles is also in the Command Palette. Select Default Profile has no palette entry of its own.
- Commands: testing.selectDefaultTestProfiles; testing.configureProfile; vscode.pickMultipleTestProfiles
- Menus: CommandPalette (testing.configureProfile, when testing.hasConfigurableProfile)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (254-277, 349-376); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (390-410); V\src\vs\workbench\contrib\testing\browser\testingConfigurationUi.ts (98-132); C bundle (ids testing.selectDefaultTestProfiles, testing.configureProfile, vscode.pickMultipleTestProfiles); C out\nls.messages.json ("Select Default Profile", "Configure Test Profiles", "Update Test Configuration", "Select a profile to update")
- Pieces:
  1. Select Default Profile, at the bottom of the title-button drop-down when more than one profile exists, opens a multi-select pick of the default profiles for Run, Debug and Coverage.
  2. Configure Test Profiles, at the bottom of the drop-down when a profile can be configured, opens a pick titled "Select a profile to update" and then that profile's own configuration step.
  3. Configure Test Profiles is also listed in the Command Palette while a configurable profile exists.
  4. Select Default Profile has no Command Palette entry and is reached only from the drop-down.

#### SB-6.12 Turn continuous run on or off

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Make tests re-run by themselves when files change. A row has a Turn on Continuous Run toggle (it shows Turn off Continuous Run when active, and is disabled when a parent is already running continuously), and the row's menu has Start Continous Run Using... to pick which profiles watch. The view title has Start Continuous Run and Stop Continuous Run buttons, which become a split button with a check per profile when several profiles support it. A row that is continuous, or has a continuous child, keeps its continuous-run icon visible even when not hovered.
- Commands: testing.toggleContinuousRunForTest; testing.continuousRunUsingForTest; testing.startContinuousRun; testing.stopContinuousRun
- Menus: TestItem (group inline, group builtin@1, and group builtin@2 for the "Using" row); ViewTitle (group navigation); CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (279-347, 380-411, 487-511); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (continuous drop-down 453-511, row style 1572-1580); V\src\vs\workbench\contrib\testing\browser\media\testing.css (94-112); V\src\vs\workbench\contrib\testing\common\testingContinuousRunService.ts; C bundle (ids testing.toggleContinuousRunForTest, testing.continuousRunUsingForTest, testing.startContinuousRun, testing.stopContinuousRun); C out\nls.messages.json ("Turn on Continuous Run", "Turn off Continuous Run", "Start Continous Run Using...", "Start Continuous Run", "Stop Continuous Run")
- Pieces:
  1. A test row has a Turn on Continuous Run toggle that makes the test re-run by itself when files change, and it reads Turn off Continuous Run while it is active.
  2. The Turn on Continuous Run toggle is disabled on a row whose parent already runs continuously.
  3. Choosing Start Continous Run Using... in a row's right-click menu picks which profiles watch that test.
  4. The view title shows a Start Continuous Run button and a Stop Continuous Run button.
  5. When several profiles support continuous run, the title button becomes a split button whose drop-down has a check beside each profile.
  6. A row that is running continuously, or has a continuous child, keeps its continuous-run icon visible even when it is not hovered.
  7. Start Continuous Run and Stop Continuous Run are also listed in the Command Palette.

#### SB-6.13 Cancel a running test run

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While tests are running, the Run, Debug and Coverage buttons in the view title are replaced by a Cancel Test Run button. The command also stops every unfinished run when called without a target.
- Commands: testing.cancelRun
- Menus: ViewTitle (group navigation, only while testing.isRunning); CommandPalette (while running)
- Keys: Ctrl/Cmd+; then Ctrl/Cmd+X
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (707-749); C bundle (id testing.cancelRun, keybinding chord Ctrl/Cmd+; X); C out\nls.messages.json ("Cancel Test Run")
- Pieces:
  1. While tests are running, the Run, Debug and Coverage buttons in the view title are replaced by a Cancel Test Run button.
  2. Clicking Cancel Test Run stops the tests that are running.
  3. Cancel Test Run appears in the Command Palette only while tests are running, and using it without a target stops every unfinished run.
  4. Pressing `Ctrl/Cmd+;` then `Ctrl/Cmd+X` cancels the test run.

#### SB-6.14 Refresh tests

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Ask test extensions to re-discover tests, from the Refresh Tests button in the view title, the inline button on a root row, or the Command Palette. While a refresh runs the button turns into Cancel Test Refresh, and the view shows a discovery progress bar. It only appears when a test controller supports refresh.
- Commands: testing.refreshTests; testing.cancelTestRefresh
- Menus: ViewTitle (group navigation, when testing.canRefresh); TestItem (group inline, root rows only); CommandPalette
- Keys: Ctrl/Cmd+; then Ctrl/Cmd+R (when testing.canRefresh)
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (1722-1795); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (progress 523-531, root-row refresh context 1656-1659); C bundle (ids testing.refreshTests, testing.cancelTestRefresh, keybinding chord Ctrl/Cmd+; R); C out\nls.messages.json ("Refresh Tests", "Cancel Test Refresh", "Discovering Tests")
- Pieces:
  1. The Refresh Tests button in the view title asks the test extensions to find tests again.
  2. An inline Refresh Tests button on a root row refreshes the tests of that controller.
  3. Choosing Refresh Tests in the Command Palette, or pressing `Ctrl/Cmd+;` then `Ctrl/Cmd+R`, refreshes the tests.
  4. While a refresh is running, the Refresh Tests button turns into Cancel Test Refresh.
  5. While a refresh is running, the view shows a "Discovering Tests" progress bar.
  6. The refresh buttons and the key chord appear only when a test controller supports refresh.

#### SB-6.15 Right-click a test for its menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-clicking a test, suite or file row opens its context menu: Run Test, Debug Test, Run Test with Coverage, Execute Using Profile..., Start Continous Run Using..., Turn on or off Continuous Run, Go to Test, and Hide Test or Unhide Test (Refresh Tests is an inline button only, not a menu row). If the clicked row is part of the selection, the action applies to all selected rows. Extensions can add rows through the testing/item/context menu.
- Commands: testing.run; testing.debug; testing.coverage; testing.runUsing; testing.continuousRunUsingForTest; testing.toggleContinuousRunForTest; testing.editFocusedTest; testing.hideTest; testing.unhideTest
- Menus: TestItem (testing/item/context; the menu shows every group except inline)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (onContextMenu 1076-1090, getActionableElementActions 1642-1687); V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (89-162, 207-241, 279-347); V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts (351); C bundle (TestItem menu rows for the same ids with the same groups and orders; no Cursor-added rows found)
- Pieces:
  1. Right-clicking a test, suite or file row opens a menu with Run Test, Debug Test and Run Test with Coverage.
  2. The same menu has Execute Using Profile..., Start Continous Run Using... and Turn on Continuous Run or Turn off Continuous Run.
  3. The same menu has Go to Test and Hide Test, which reads Unhide Test on a hidden row.
  4. Refresh Tests is an inline button only and is not a row in the right-click menu.
  5. When the right-clicked row is part of the selection, the chosen action applies to every selected row.
  6. Extensions can add their own rows to the menu through the `testing/item/context` menu.

#### SB-6.16 Open a test's source

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Opening a test that has no child rows (a single click by default, a double click if the list open mode says so) opens its file at the test's line, or peeks the first error instead when the test has failed. Open to the side puts it beside the current editor. Go to Test in the row's menu, or Alt+Enter on the focused row, opens the file without the failure peek.
- Commands: testing.editFocusedTest; vscode.revealTest
- Menus: TestItem (group builtin@1, when testing.testItemHasUri)
- Keys: Alt+Enter (when the focused view is workbench.view.testing)
- Settings: workbench.list.openMode
- Source: V\src\vs\platform\list\browser\listService.ts (openMode 171, 693-701); V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (961-991); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (open handler 857-872, peek first error 1070-1076); V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts (vscode.revealTest 210-258); C bundle (ids testing.editFocusedTest, vscode.revealTest, keybinding Alt+Enter for view workbench.view.testing); C out\nls.messages.json ("Go to Test")
- Pieces:
  1. Opening a test that has no child rows opens its file at the test's line.
  2. Opening a test that has failed shows a peek of its first error instead of just opening the file.
  3. Setting `workbench.list.openMode` chooses whether a single click or a double click opens a test, with values `singleClick` (default) and `doubleClick`.
  4. Opening a test to the side puts its file in an editor beside the current one.
  5. Choosing Go to Test in a row's right-click menu opens the test's file without the failure peek.
  6. Pressing `Alt+Enter` on a focused row in the Testing view does the same as Go to Test.

#### SB-6.17 Hide and unhide tests

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Hide a test, suite or file from the tree with Hide Test in its menu. Hidden rows disappear until you turn on Show Hidden Tests (they then show dimmed with a hidden icon and an Unhide Test entry), or clear everything with Unhide All Tests. Revealing a hidden test from elsewhere turns Show Hidden Tests on automatically.
- Commands: testing.hideTest; testing.unhideTest; testing.unhideAllTests
- Menus: TestItem (group builtin@2 for Hide Test; Unhide Test with order 21)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (89-148); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (revealById 988-1057, hidden row style 1591-1597, hidden filter 1222-1228); V\src\vs\workbench\contrib\testing\browser\testingExplorerFilter.ts (228-243); C bundle (ids testing.hideTest, testing.unhideTest, testing.unhideAllTests); C out\nls.messages.json ("Hide Test", "Unhide Test", "Unhide All Tests", "Show Hidden Tests")
- Pieces:
  1. Choosing Hide Test in a row's right-click menu removes that test, suite or file from the tree.
  2. With Show Hidden Tests on, hidden rows show again, dimmed and with a hidden icon.
  3. A hidden row's right-click menu has Unhide Test, which brings that row back to normal.
  4. Unhide All Tests makes every hidden test visible again.
  5. Revealing a hidden test from somewhere else, such as from the editor, turns Show Hidden Tests on by itself.

#### SB-6.18 Follow the running test in the tree

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With follow turned on, the tree scrolls to and selects the test that is currently running (it does nothing while several rows are selected). With "always reveal" off, only tests that fail are revealed.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: testing.followRunningTest; testing.alwaysRevealTestOnStateChange
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (895-933); V\src\vs\workbench\contrib\testing\common\configuration.ts (114-117, 162-166); C bundle (settings testing.followRunningTest, testing.alwaysRevealTestOnStateChange); C out\nls.messages.json ("Controls whether the running test should be followed in the Test Explorer view.")
- Pieces:
  1. With `testing.followRunningTest` on (off by default), the tree scrolls to and selects the test that is currently running.
  2. The tree does not follow the running test while several rows are selected.
  3. Setting `testing.alwaysRevealTestOnStateChange` chooses whether every executed test is revealed when following is on, and with it off (default) only tests that fail are revealed.

#### SB-6.19 Read a test's status in its row

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Each row shows a state icon (passed, failed, errored, queued, running with a spinner, skipped, not yet run), a lighter icon color (70 percent opacity, theme colors testing.icon*.retired) when the result is outdated, an optional description, and the last duration (for example 12ms or 1.2s). Hovering reads the full label such as "Test name (Passed), in 12ms" or ", outdated result".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (label text 1391-1409, TestItemRenderer 1500-1626, duration format 1628-1638); V\src\vs\workbench\contrib\testing\common\constants.ts (36-48); V\src\vs\workbench\contrib\testing\browser\icons.ts; V\src\vs\workbench\contrib\testing\browser\theme.ts (167-173); V\src\vs\workbench\contrib\testing\browser\media\testing.css (94-117, 156); C out\nls.messages.json ("{0}, in {1}", "{0}, outdated result", "Not yet run", "Passed", "Failed", "Errored", "Queued", "Running", "Skipped")
- Pieces:
  1. Each test row shows a state icon for passed, failed, errored, queued, running with a spinner, skipped or not yet run.
  2. When a result is outdated, the state icon shows in a lighter color through the `testing.icon*.retired` theme colors at 70 percent opacity.
  3. A row can show a short description beside its name.
  4. A row shows how long the last run took, for example `12ms` or `1.2s`.
  5. Hovering a row reads out the full state, such as "Test name (Passed), in 12ms", with ", outdated result" added when the result is outdated.

#### SB-6.20 See the run summary and Testing icon badge

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (no spinner badge while tests run, the rest is present)
- What: A row above the tree shows the last run as a status icon, passed over total count (hover gives the detail), and the duration, with Rerun Last Run and Debug Last Run buttons once the run is done. With no runs it says "No test results yet." The same code sends a badge for the Testing icon: the number of failed, passed or skipped tests (or none), and a watch icon while continuous run is on. Stock also shows a spinner badge while tests run. The icon itself is drawn by the rail (AB-3).
- Commands: testing.reRunLastRun; testing.debugLastRun
- Menus: none found
- Keys: none found
- Settings: testing.countBadge
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (ResultSummaryView 545-681); V\src\vs\workbench\contrib\testing\common\configuration.ts (100-113); C bundle (renderActivityBadge has no running branch, only count and continuous badges); C out\nls.messages.json ("No test results yet.", "{0} failed tests", "{0} passed tests", "{0} skipped tests", "Tests are being watched for changes", "Controls the count badge on the Testing icon on the Activity Bar."; the string "Tests are running" is not present)
- Pieces:
  1. A summary row above the tree shows the last run as a status icon, the passed count over the total count, and the duration.
  2. Hovering the passed over total count gives the detail of the run.
  3. Rerun Last Run and Debug Last Run buttons appear in the summary row once the run is done.
  4. With no runs yet, the summary row says "No test results yet."
  5. Setting `testing.countBadge` chooses what the badge on the Testing icon counts, with values `failed` (default), `passed`, `skipped` and `off`.
  6. The badge on the Testing icon shows a watch icon while continuous run is on.
  7. A spinner badge shows on the Testing icon while tests are running. [stock only]

#### SB-6.21 Re-run failed tests or the last run

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the two "from last run" commands and their labels are not found, the other five are present)
- What: Command Palette commands that repeat earlier work: Rerun Failed Tests and Debug Failed Tests (a test counts as failed until a later run passes it), Rerun Last Run, Debug Last Run and Rerun Last Run with Coverage (same profiles as before when they still exist), and Rerun or Debug Failed Tests from Last Run (only the failures of the newest run). They show up when a test provider exists, and the last-run ones also need results. Rerun Last Run and Debug Last Run are also the two buttons in the summary row (SB-6.20).
- Commands: testing.reRunFailTests; testing.debugFailTests; testing.reRunLastRun; testing.debugLastRun; testing.coverageLastRun; testing.reRunFailedFromLastRun; testing.debugFailedFromLastRun
- Menus: CommandPalette
- Keys: Ctrl/Cmd+; then E (reRunFailTests); Ctrl/Cmd+; then Ctrl/Cmd+E (debugFailTests); Ctrl/Cmd+; then L (reRunLastRun); Ctrl/Cmd+; then Ctrl/Cmd+L (debugLastRun); Ctrl/Cmd+; then Ctrl/Cmd+Shift+L (coverageLastRun)
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (1386-1663); C bundle (ids testing.reRunFailTests, testing.debugFailTests, testing.reRunLastRun, testing.debugLastRun, testing.coverageLastRun; ids testing.reRunFailedFromLastRun and testing.debugFailedFromLastRun not found); C out\nls.messages.json ("Rerun Failed Tests", "Debug Failed Tests", "Rerun Last Run", "Debug Last Run", "Rerun Last Run with Coverage")
- Pieces:
  1. Rerun Failed Tests and Debug Failed Tests in the Command Palette run again every test that has failed, and a test counts as failed until a later run passes it.
  2. Rerun Last Run, Debug Last Run and Rerun Last Run with Coverage repeat the previous run with the same profiles when those profiles still exist.
  3. Rerun Failed Tests from Last Run and Debug Failed Tests from Last Run repeat only the failures of the newest run. [stock only]
  4. These commands show in the Command Palette only when a test provider exists, and the last-run commands also need earlier results.
  5. The chords are `Ctrl/Cmd+;` then `E` (rerun failed), `Ctrl/Cmd+E` (debug failed), `L` (rerun last run), `Ctrl/Cmd+L` (debug last run) and `Ctrl/Cmd+Shift+L` (rerun last run with coverage).

#### SB-6.22 See the empty states

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Once discovery has finished and no tests exist, the view shows "No tests have been found in this workspace yet." with an "Install Additional Test Extensions..." link (it opens an Extensions view search for the testing category, and that view is out of scope here). If the filter is set to the active file and that file has no tests, the tree is replaced by "No tests were found in this file." with a Show Workspace Tests button that switches the file filter off.
- Commands: testing.searchForTestExtension
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts (welcome content 103-110); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (welcome state 146-148, 1113-1123, NoTestsForDocumentWidget 1351-1370); V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (1665-1676); C bundle (welcome content registered for workbench.view.testing with link command:testing.searchForTestExtension); C out\nls.messages.json ("No tests have been found in this workspace yet.", "Install Additional Test Extensions...", "No tests were found in this file.", "Show Workspace Tests", "Search for Test Extension")
- Pieces:
  1. When discovery has finished and no tests exist, the view shows "No tests have been found in this workspace yet."
  2. The "Install Additional Test Extensions..." link under that text opens an Extensions view search for the testing category.
  3. When the filter is set to the active file and that file has no tests, the tree is replaced by "No tests were found in this file."
  4. The Show Workspace Tests button under that text switches the active-file filter off and shows the whole workspace again.

#### SB-6.23 Open the Testing views automatically

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Choose whether starting a run opens a view for you: never, open Test Results when tests start (the default), open Test Results on the first failure, or open the Test Explorer when tests start. A run that asks to keep focus where it is never opens anything. The same choice also governs the Test Results view (PN-6). It replaces the older setting testing.openTesting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: testing.automaticallyOpenTestResults
- Source: V\src\vs\workbench\contrib\testing\browser\testingProgressUiService.ts (17-83); V\src\vs\workbench\contrib\testing\common\configuration.ts (33-38, 145-161, 222-232); C bundle (setting testing.automaticallyOpenTestResults, migration from testing.openTesting and testing.automaticallyOpenResults); C out\nls.messages.json ("Controls when the testing view should open.")
- Pieces:
  1. Setting `testing.automaticallyOpenTestResults` chooses whether starting a run opens a testing view, with values `neverOpen`, `openOnTestStart` (default), `openOnTestFailure` and `openExplorerOnTestStart`.
  2. With `openOnTestStart`, the Test Results view opens when tests start.
  3. With `openOnTestFailure`, the Test Results view opens at the first test failure.
  4. With `openExplorerOnTestStart`, the Test Explorer opens when tests start.
  5. A run that asks to keep focus where it is never opens any view.

#### SB-6.24 Browse the Test Coverage view

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (view, tree, bars and both settings are present; the Collapse All Coverage command and label are not found)
- What: While a coverage report is open, a Test Coverage view joins the Testing container. It is a tree of folders and files (single-child folder chains are merged), each row showing the overall percent and compact bars for statements, functions and branches, with a hover that gives covered over total counts. Expanding a file loads its declarations with their own bars. Opening a row opens the file, at the function for a declaration, and open to the side is supported. A Collapse All Coverage button sits in the view title. Settings pick which percent is the headline (total, statement or minimum) and the red, yellow and green cut-offs.
- Commands: testing.coverageViewCollapseAll
- Menus: ViewTitle (only for view workbench.view.testCoverage)
- Keys: none found
- Settings: testing.displayedCoveragePercent; testing.coverageBarThresholds
- Source: V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts (view 122-131); V\src\vs\workbench\contrib\testing\browser\testCoverageView.ts (view 62-120, tree 232-335, renderers 440-700, collapse all 805-822); V\src\vs\workbench\contrib\testing\browser\testCoverageBars.ts (43-135); V\src\vs\workbench\contrib\testing\common\testCoverageService.ts (100-145); V\src\vs\workbench\contrib\testing\common\configuration.ts (168-193); C bundle (view id workbench.view.testCoverage, settings testing.displayedCoveragePercent and testing.coverageBarThresholds; id testing.coverageViewCollapseAll not found); C out\nls.messages.json ("Test Coverage", "Test Coverage Explorer"; "Collapse All Coverage" not found)
- Pieces:
  1. While a coverage report is open, a Test Coverage view joins the Testing container.
  2. The Test Coverage view is a tree of folders and files, and a chain of folders with a single child is merged into one row.
  3. Each row shows its overall percent and compact bars for statements, functions and branches.
  4. Hovering a row's bars shows covered over total counts.
  5. Expanding a file loads its declarations, each with its own bars.
  6. Opening a row opens the file, jumping to the function for a declaration, and opening to the side is supported.
  7. A Collapse All Coverage button in the Test Coverage title collapses every open node. [stock only]
  8. Setting `testing.displayedCoveragePercent` chooses which percent is the headline, with values `totalCoverage` (default), `statement` and `minimum`.
  9. Setting `testing.coverageBarThresholds` sets the red, yellow and green cut-offs for coverage percents, with defaults of 0, 60 and 90.

#### SB-6.25 Sort the coverage view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Change Sort Order button in the Test Coverage title opens a picker with Sort by Location (files alphabetical, declarations by position), Sort by Coverage (best covered first), and Sort by Name. Stock remembers the choice per workspace.
- Commands: testing.coverageViewChangeSorting
- Menus: ViewTitle (group navigation, order 1, only for view workbench.view.testCoverage)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testCoverageView.ts (62-100, 764-803); C bundle (id testing.coverageViewChangeSorting, view id workbench.view.testCoverage); C out\nls.messages.json ("Change Sort Order", "Sort by Coverage", "Sort by Name", "Sort the Test Coverage view...")
- Pieces:
  1. The Change Sort Order button in the Test Coverage title opens a picker of sort orders.
  2. Sort by Location lists files alphabetically and declarations by their position in the file.
  3. Sort by Coverage lists the best covered rows first.
  4. Sort by Name lists rows by name.
  5. The chosen coverage sort order is remembered per workspace.

#### SB-6.26 Filter coverage to one test

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the coverage has per-test data, Filter Coverage by Test opens a picker with All Tests and each test, and the view previews the pick as you move through it. The view then shows a first row "Showing coverage for ..." with a filter button that reopens the picker. The button in the view title looks filled while a filter is active.
- Commands: testing.coverageFilterToTest
- Menus: ViewTitle (group navigation, when testing.hasPerTestCoverage, only for view workbench.view.testCoverage); CommandPalette (when testing.hasPerTestCoverage); TestCoverageFilterItem (group inline)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testCoverageView.ts (707-761, filter row 202-216 and 501-538); V\src\vs\workbench\contrib\testing\common\testCoverageService.ts; C bundle (id testing.coverageFilterToTest, menu TestCoverageFilterItem); C out\nls.messages.json ("Filter Coverage by Test")
- Pieces:
  1. When the coverage has per-test data, the Filter Coverage by Test button in the Test Coverage title opens a picker with All Tests and each test.
  2. Filter Coverage by Test is also in the Command Palette while the coverage has per-test data.
  3. Moving through the picker previews the coverage of the highlighted test in the view before it is chosen.
  4. After a test is chosen, the view's first row reads "Showing coverage for" and the test, with a filter button that reopens the picker.
  5. The filter button in the view title looks filled while a test filter is active.

#### SB-6.27 Close or reopen the coverage report

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clear Coverage (the X in the Test Coverage title, or the Command Palette while coverage is open) closes the report and hides the Test Coverage view. Open Coverage in the Command Palette reopens the report from the newest run, or says "No coverage information available on the last test run." A new run that finishes without coverage closes the old report, and one that has coverage opens it without taking focus.
- Commands: testing.coverage.close; testing.openCoverage
- Menus: ViewTitle (group navigation, only for view workbench.view.testCoverage); CommandPalette (testing.coverage.close when testing.isTestCoverageOpen; testing.openCoverage when testing.hasAnyResults)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (1796-1843); V\src\vs\workbench\contrib\testing\common\testCoverageService.ts (100-145); C bundle (ids testing.coverage.close, testing.openCoverage); C out\nls.messages.json ("Clear Coverage", "Open Coverage", "No coverage information available on the last test run.")
- Pieces:
  1. The X (Clear Coverage) in the Test Coverage title closes the report and hides the Test Coverage view.
  2. Clear Coverage is also in the Command Palette while a coverage report is open.
  3. Open Coverage in the Command Palette reopens the report from the newest run when any results exist.
  4. When the newest run has no coverage, Open Coverage says "No coverage information available on the last test run."
  5. A new run that finishes without coverage closes the old coverage report.
  6. A new run that finishes with coverage opens its report without taking focus.

#### SB-6.28 Show coverage bars in the file Explorer

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the setting and the Explorer bar code are present; the toggle command and its labels are not found)
- What: While a coverage report is open, files and folders in the Explorer file tree (SB-2) show compact coverage bars, with a hover giving the counts. Toggle Coverage in Explorer in the Command Palette (only while coverage is open) flips the setting, and its menu title changes to Hide Coverage in Explorer when on.
- Commands: testing.toggleCoverageInExplorer
- Menus: CommandPalette (when testing.isTestCoverageOpen)
- Keys: none found
- Settings: testing.showCoverageInExplorer
- Source: V\src\vs\workbench\contrib\testing\browser\codeCoverageDecorations.ts (997-1019); V\src\vs\workbench\contrib\testing\browser\testCoverageBars.ts (183-236); V\src\vs\workbench\contrib\testing\browser\testingProgressUiService.ts (34-45); V\src\vs\workbench\contrib\testing\common\configuration.ts (168-171); C bundle (setting testing.showCoverageInExplorer and the Explorer bar class that reads it; id testing.toggleCoverageInExplorer not found); C out\nls.messages.json ("Whether test coverage should be down in the File Explorer view."; "Toggle Coverage in Explorer" and "Hide Coverage in Explorer" not found)
- Pieces:
  1. While a coverage report is open, files and folders in the Explorer file tree show compact coverage bars.
  2. Hovering the bars on an Explorer row gives the covered counts.
  3. Setting `testing.showCoverageInExplorer` chooses whether those bars show in the Explorer, and it is on by default.
  4. Toggle Coverage in Explorer in the Command Palette, listed only while a coverage report is open, flips that setting, and its title reads Hide Coverage in Explorer when the bars are on. [stock only]

#### SB-6.29 Run, debug or cover the tests in the current file

- Tag: both. Evidence: source. Boundary: no. Cursor: present - ids, labels (nls 11100 to 11102) and the chords are in the bundle
- What: Run all tests of the file you are editing, debug them, or run them with coverage, from the Command Palette, the editor right-click menu or a key chord.
- Commands: testing.runCurrentFile; testing.debugCurrentFile; testing.coverageCurrentFile
- Menus: CommandPalette (Test category, when the matching run profile exists); EditorContext (group testing, when the active editor has tests and the profile exists)
- Keys: Ctrl+; then F (run); Ctrl+; then Ctrl+F (debug); Ctrl+; then Ctrl+Shift+F (coverage), all when the editor has text focus
- Settings: testing.saveBeforeTest
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts:1239-1372; C bundle (classes with ids testing.runCurrentFile, testing.debugCurrentFile and testing.coverageCurrentFile and their key chords)
- Pieces:
  1. "Test: Run Tests in Current File" runs every test found in the active editor's file with the default run profile, "Test: Debug Tests in Current File" starts them as a debug session, and "Test: Run Tests with Coverage in Current File" runs them with coverage.
  2. Each entry shows in the palette, and in the editor's right-click menu under the testing group, only when a test extension has registered a matching run profile, and the editor menu also needs the file to contain tests.
  3. With text focus in the editor, `Ctrl+;` then `F` runs, `Ctrl+;` then `Ctrl+F` debugs, and `Ctrl+;` then `Ctrl+Shift+F` runs with coverage.
  4. When the file has no tests, the message "No tests found in this file" appears at the cursor in the editor.
  5. Setting `testing.saveBeforeTest` (default on) saves unsaved files before the run starts.

#### SB-6.30 Run, debug or cover the test at the cursor

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the three ids, the labels (nls 11097 to 11099), the run and debug chords and "No tests found here" are in the bundle. ED-11 handed these commands to SB-6 and SB-6 has no feature for them; the parent may merge this with SB-6.29, its current-file sibling.
- What: With the text cursor in a test file, run, debug or run with coverage the test that contains the cursor, from the Command Palette, the editor right-click menu or a key chord.
- Commands: testing.runAtCursor; testing.debugAtCursor; testing.coverageAtCursor
- Menus: CommandPalette (Test category, when a test provider exists); EditorContext (group testing, Run Test at Cursor first and Debug Test at Cursor and the coverage entry in the next slot, when the active editor has tests and a profile of that kind exists)
- Keys: Ctrl+; then C (run); Ctrl+; then Ctrl+C (debug); Ctrl+; then Ctrl+Shift+C (coverage), all when the editor has text focus (Cmd on macOS)
- Settings: testing.saveBeforeTest
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (ExecuteTestAtCursor lines 1048-1131, RunAtCursor, DebugAtCursor and CoverageAtCursor lines 1133-1176); V\src\vs\workbench\contrib\testing\common\constants.ts (lines 80 and 105); C bundle (classes with ids testing.runAtCursor, testing.debugAtCursor and testing.coverageAtCursor); C out\nls.messages.json (labels, "No tests found here")
- Pieces:
  1. The Command Palette lists Test: Run Test at Cursor, Test: Debug Test at Cursor and Test: Run Test at Cursor with Coverage whenever a test provider is registered.
  2. With text focus in the editor, `Ctrl+;` then `C` runs the test at the cursor, `Ctrl+;` then `Ctrl+C` debugs it, and `Ctrl+;` then `Ctrl+Shift+C` runs it with coverage.
  3. The editor right-click menu shows the matching entries in the testing group only when the active file has tests and a run, debug or coverage profile exists.
  4. The tests picked are the most specific ones whose range contains the cursor, and when none contains it, the closest test above the cursor; tests with an equal range all run, and only tests that have a profile of the chosen kind are used.
  5. When no test is found at the cursor but tests are known to relate to the code there, those related tests run instead.
  6. When nothing is found at all, a small message "No tests found here" appears at the cursor.
  7. A progress indicator shows while the tests of the file are being discovered.
  8. Setting `testing.saveBeforeTest` on makes the editor save the file and refresh the test list before the run starts.


## SB-7 Outline, Timeline and References

(both). The helper views for the open file's symbols, its saved history, and Find All References plus call and type hierarchies.

28 features.

#### SB-7.1 Outline view of the open file's symbols

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the view registration with id `outline`, order 2, collapsed, `outline.focus`, `outline.collapse` and `outline.expand` are all in the bundle, and the NLS strings "Outline", "The active editor cannot provide outline information." and "No symbols found in document" are found)
- What: The Outline pane under the Explorer tree lists the classes, functions and other symbols of the active editor as a tree. Click a row, or press Enter, to jump to it. The pane says when the editor cannot provide symbols or none were found, and it has Collapse All and Expand All buttons in its title.
- Commands: outline.focus; outline.collapse; outline.expand
- Menus: ViewTitle
- Keys: none found
- Settings: outline.collapseItems
- Source: V\src\vs\workbench\contrib\outline\browser\outline.contribution.ts (24-38); V\src\vs\workbench\contrib\outline\browser\outlinePane.ts (133-393); V\src\vs\workbench\contrib\outline\browser\outlineActions.ts (18-60); V\src\vs\workbench\contrib\codeEditor\browser\outline\documentSymbolsOutline.ts; C bundle (Outline view registration)
- Pieces:
  1. The Outline pane sits under the Explorer tree, starts collapsed, and lists the symbols of the active text editor as a tree of classes, functions and other kinds.
  2. Clicking a symbol row, or focusing it and pressing Enter, moves the editor to that symbol.
  3. Running `outline.focus` from the Command Palette reveals the Outline pane and puts keyboard focus in it.
  4. While symbols are loading the pane says "Loading document symbols for '<file>'...", and when none come back it says "No symbols found in document '<file>'".
  5. When the active editor cannot provide symbols the pane says "The active editor cannot provide outline information."
  6. The Outline title has a Collapse All button (`outline.collapse`) that folds every row, and an Expand All button (`outline.expand`) that opens every row.
  7. Setting `outline.collapseItems` chooses whether rows start collapsed or expanded, with values `alwaysExpand` (default) and `alwaysCollapse`, and it can differ per language.

#### SB-7.2 Follow Cursor in Outline

- Tag: both. Evidence: source. Boundary: no. Cursor: present (`outline.followCursor` and the NLS string "Follow Cursor" found)
- What: A toggle in the Outline "..." menu. When on, the selected Outline row moves to match the caret in the editor. The choice is remembered.
- Commands: outline.followCursor
- Menus: ViewTitle
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\outline\browser\outlineActions.ts (58-78); V\src\vs\workbench\contrib\outline\browser\outlinePane.ts (324-346); V\src\vs\workbench\contrib\outline\browser\outlineViewState.ts; C bundle (outline.followCursor action)
- Pieces:
  1. Choosing Follow Cursor from the Outline "..." menu turns on following, and the menu row shows a checked state while it is on.
  2. With Follow Cursor on, moving the caret in the editor moves the selected Outline row to the symbol that holds the caret.
  3. With Follow Cursor off, moving the caret leaves the Outline selection where it was.
  4. The Follow Cursor choice is remembered the next time the Outline opens.

#### SB-7.3 Filter on Type in Outline

- Tag: both. Evidence: source. Boundary: no. Cursor: present (`outline.filterOnType` and the NLS string "Filter on Type" found)
- What: Typing in the Outline opens a find box. With Filter on Type on, rows that do not match are hidden and the tree expands while you filter. With it off, matches are only highlighted.
- Commands: outline.filterOnType
- Menus: ViewTitle
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\outline\browser\outlineActions.ts (79-100); V\src\vs\workbench\contrib\outline\browser\outlinePane.ts (266-272, 306-307, 363-376); C bundle (outline.filterOnType action)
- Pieces:
  1. Typing while the Outline has focus opens a find box at the top of the tree.
  2. With Filter on Type on, rows that do not match the typed text are hidden and the tree opens up so deep matches show.
  3. With Filter on Type off, all rows stay visible and only the matching text is highlighted.
  4. Filter on Type in the Outline "..." menu is a checked toggle that matches the current find mode, and flipping the find mode from the find box updates the menu row too.
  5. The Filter on Type choice is remembered the next time the Outline opens.

#### SB-7.4 Sort the Outline

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the three ids and the strings "Sort By: Position", "Sort By: Name" and "Sort By: Category" found)
- What: Choose Sort By Position, Name or Category from the Outline "..." menu. The choice is remembered.
- Commands: outline.sortByPosition; outline.sortByName; outline.sortByKind
- Menus: ViewTitle
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\outline\browser\outlineActions.ts (101-161); V\src\vs\workbench\contrib\outline\browser\outlinePane.ts (253, 357-360); C bundle (outline.sortBy actions)
- Pieces:
  1. The Outline "..." menu has three Sort By choices (`outline.sortByPosition`, `outline.sortByName`, `outline.sortByKind`) and only one is checked at a time.
  2. Sort By: Position lists rows in the order the symbols appear in the file.
  3. Sort By: Name lists rows alphabetically by symbol name.
  4. Sort By: Category groups rows by symbol kind.
  5. The chosen sort order is remembered the next time the Outline opens.

#### SB-7.5 Choose what the Outline shows

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for `outline.icons`, `outline.collapseItems`, the three `outline.problems.*` ids, and five of the kind settings: `outline.showFiles`, `outline.showClasses`, `outline.showVariables`, `outline.showModules`, `outline.showFunctions`, all found; the other kinds were not searched one by one)
- What: Settings decide which symbol kinds appear (26 kinds from files and modules to type parameters), whether rows have icons, whether errors and warnings show as colors and badges, and whether items start collapsed. Some can differ per language.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: outline.icons; outline.collapseItems; outline.problems.enabled; outline.problems.colors; outline.problems.badges; outline.showFiles; outline.showModules; outline.showNamespaces; outline.showPackages; outline.showClasses; outline.showMethods; outline.showProperties; outline.showFields; outline.showConstructors; outline.showEnums; outline.showInterfaces; outline.showFunctions; outline.showVariables; outline.showConstants; outline.showStrings; outline.showNumbers; outline.showBooleans; outline.showArrays; outline.showObjects; outline.showKeys; outline.showNull; outline.showEnumMembers; outline.showStructs; outline.showEvents; outline.showOperators; outline.showTypeParameters
- Source: V\src\vs\workbench\contrib\outline\browser\outline.contribution.ts (42-239); V\src\vs\workbench\services\outline\browser\outline.ts (99-103); V\src\vs\workbench\contrib\codeEditor\browser\outline\documentSymbolsOutline.ts (163, 331-384); C bundle (outline configuration block with `outline.icons` and `outline.collapseItems`)
- Pieces:
  1. Setting `outline.icons` turns the symbol-kind icon on each Outline row on or off, and it is on by default.
  2. Setting `outline.collapseItems` chooses whether the Outline opens with all rows collapsed or all expanded, with values `alwaysExpand` and `alwaysCollapse`, and it can be set per language.
  3. Setting `outline.problems.enabled` turns error and warning markers on Outline rows on or off, and it is on by default.
  4. Setting `outline.problems.colors` colors Outline rows red or yellow for errors and warnings, and `outline.problems.badges` adds a count badge to those rows.
  5. Both `outline.problems.colors` and `outline.problems.badges` have no visible effect while `problems.visibility` is off.
  6. Turning off a `outline.show<Kind>` setting hides rows of that kind from the Outline, all 26 kinds are on by default, and each can be set for one language only.
  7. The 26 `outline.show<Kind>` settings cover files, modules, namespaces, packages, classes, methods, properties, fields, constructors, enums, interfaces, functions, variables, constants, strings, numbers, booleans, arrays, objects, keys, null, enum members, structs, events, operators and type parameters.

#### SB-7.6 Outline of a notebook

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all seven command ids and the three settings found in the bundle, and the strings "Filter Entries" and "Markdown Headers Only" are in NLS)
- What: When a notebook is active, the Outline lists markdown headers and code cells instead of text symbols. A Filter Entries button picks Markdown Headers Only, Code Cells and Code Cell Symbols, and each row has a menu to run the cell, run the cells in a section, or fold and expand a section.
- Commands: notebook.outline.toggleShowMarkdownHeadersOnly; notebook.outline.toggleCodeCells; notebook.outline.toggleCodeCellSymbols; notebook.section.runSingleCell; notebook.section.runCells; notebook.section.foldSection; notebook.section.expandSection
- Menus: ViewTitle; NotebookOutlineFilter; NotebookOutlineActionMenu
- Keys: none found
- Settings: notebook.outline.showMarkdownHeadersOnly; notebook.outline.showCodeCells; notebook.outline.showCodeCellSymbols
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\outline\notebookOutline.ts (861-975); V\src\vs\workbench\contrib\notebook\browser\controller\sectionActions.ts (33-200); V\src\vs\platform\actions\common\actions.ts (230-231); C bundle (notebook.outline toggles and section actions)
- Pieces:
  1. When a notebook is the active editor, the Outline lists the notebook's markdown headers and code cells instead of text symbols.
  2. A Filter Entries button in the Outline title opens a menu of three checked options, Markdown Headers Only, Code Cells and Code Cell Symbols.
  3. Setting `notebook.outline.showMarkdownHeadersOnly` hides markdown cells that have no header, and is the same switch as Markdown Headers Only in the Filter Entries menu.
  4. Setting `notebook.outline.showCodeCells` shows or hides code cell rows, and is the same switch as Code Cells in the Filter Entries menu.
  5. Setting `notebook.outline.showCodeCellSymbols` shows or hides symbols found inside code cells, needs `notebook.outline.showCodeCells` on, and is the same switch as Code Cell Symbols in the Filter Entries menu.
  6. Each notebook Outline row has an action menu with Run Cell on a code cell row.
  7. A markdown header row's action menu has Run Cells In Section, Fold Section and Expand Section, which run or fold or open the cells under that header in the notebook.

#### SB-7.7 Timeline view for the open file

- Tag: both. Evidence: source. Boundary: no. Cursor: present (view registration with `timeline.focus`, `timeline.refresh`, `timeline.pageSize`, `timeline.pageOnScroll` and the strings "Load more" and "The active editor cannot provide timeline information." found)
- What: The Timeline pane under Outline lists earlier versions of the open file, newest first, from every source that has some, with the file name beside the title. A Refresh button is in its title. It loads the next page when you scroll to the end or click Load more, and it shows a message when the editor cannot provide a timeline or every source is filtered out. The pane exists only when some timeline source is registered.
- Commands: timeline.focus; timeline.refresh
- Menus: TimelineTitle
- Keys: none found
- Settings: timeline.pageSize; timeline.pageOnScroll
- Source: V\src\vs\workbench\contrib\timeline\browser\timeline.contribution.ts (28-65); V\src\vs\workbench\contrib\timeline\browser\timelinePane.ts (194-232, 241-300, 440-446, 819-860, 925-997, 1280-1297); C bundle (Timeline view registration)
- Pieces:
  1. The Timeline pane sits under Outline in the Explorer container and lists the open file's earlier versions, newest first, with entries from every source that has some.
  2. The Timeline header shows the file name beside the title.
  3. A Refresh button in the Timeline title (`timeline.refresh`) reloads the list, and `timeline.focus` reveals the pane.
  4. Scrolling to the end of the list loads the next page, and a "Load more" row at the end loads it on click.
  5. Setting `timeline.pageOnScroll` turns the load-on-scroll behavior on or off, and when off only the "Load more" row loads more.
  6. Setting `timeline.pageSize` chooses how many entries show at first and per page, and `null` sizes the page to the visible area.
  7. The pane says "The active editor cannot provide timeline information." when the editor has nothing to show, and "All timeline sources have been filtered out." when every source is hidden.
  8. When a source returns nothing the pane says "No timeline information was provided." or "No filtered timeline information was provided.", with a Local History hint or "Source Control has not been configured." where that applies.
  9. While entries load the pane says "Loading timeline for <file>...".
  10. The Timeline pane exists only when at least one timeline source is registered.

#### SB-7.8 Pin or unpin the current timeline

- Tag: both. Evidence: source. Boundary: no. Cursor: present (`timeline.toggleFollowActiveEditor` and the strings "Pin the Current Timeline" and "Unpin the Current Timeline" found)
- What: A pin button in the Timeline title locks the view to the file it shows, so it stops changing when you switch tabs, and the header reads "<file> (pinned)". The same button unpins it.
- Commands: timeline.toggleFollowActiveEditor
- Menus: TimelineTitle
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\timeline\browser\timelinePane.ts (299-310, 440-446, 1299-1325); C bundle (toggleFollowActiveEditor command)
- Pieces:
  1. A pin button in the Timeline title (`timeline.toggleFollowActiveEditor`, "Pin the Current Timeline") locks the pane to the file it now shows.
  2. While pinned, switching editor tabs does not change the Timeline list.
  3. While pinned, the Timeline header reads "<file> (pinned)".
  4. When pinned the same button changes to "Unpin the Current Timeline", and pressing it makes the pane follow the active editor again.

#### SB-7.9 Filter timeline sources

- Tag: both. Evidence: source. Boundary: no. Cursor: present (`timeline.toggleExcludeSource:`, `TimelineFilterSubMenu`, the storage key `timeline.excludeSources` and the string "Filter Timeline" found)
- What: A Filter Timeline button in the title opens a list of sources, one per timeline provider such as Git History and Local History. Tick or untick each one to show or hide its entries. The choice is kept in the profile.
- Commands: timeline.toggleExcludeSource:<sourceId>
- Menus: TimelineTitle; TimelineFilterSubMenu
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\timeline\browser\timeline.contribution.ts (96-104); V\src\vs\workbench\contrib\timeline\browser\timelinePane.ts (1349-1376); C bundle (timeline source filter registration)
- Pieces:
  1. A Filter Timeline button in the Timeline title opens a list with one checked row per timeline source, such as Git History and Local History.
  2. Unticking a source hides its entries from the Timeline and ticking it shows them again.
  3. The ticked and unticked choice is kept in the profile (`timeline.excludeSources`) and applies the next time the window opens.
  4. Hiding every source makes the pane say "All timeline sources have been filtered out."

#### SB-7.10 Open the timeline for a file from the Explorer

- Tag: both. Evidence: source. Boundary: no. Cursor: present (`files.openTimeline` found in the bundle)
- What: Right-click a file in the Explorer and choose Open Timeline to show that file's history in the Timeline pane, even if it is not the active editor.
- Commands: files.openTimeline
- Menus: ExplorerContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\timeline\browser\timeline.contribution.ts (67-94); C bundle (files.openTimeline command)
- Pieces:
  1. Right-clicking a file in the Explorer shows an Open Timeline row (`files.openTimeline`).
  2. Choosing Open Timeline reveals the Timeline pane with the history of that file, even when it is not the active editor.
  3. After Open Timeline the pane is pinned to the chosen file, so its header reads "<file> (pinned)" and switching tabs does not change it until it is unpinned.
  4. The Open Timeline row shows only on file rows, not folder rows, and only when some timeline source is registered.

#### SB-7.11 Pin the workspace timeline

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched the bundle and NLS for `timeline.showWorkspaceTimeline` and "Pin Workspace Timeline", 0 hits)
- What: newer-than-1.128. A Pin Workspace Timeline button in the Timeline title switches the pane from one file to a timeline for the whole workspace, with a "Workspace" header and a "Loading workspace timeline..." message.
- Commands: timeline.showWorkspaceTimeline
- Menus: TimelineTitle
- Keys: none found
- Settings: none found
- Source: N\src\vs\workbench\contrib\timeline\browser\timelinePane.ts (the `timeline.showWorkspaceTimeline` action and the TimelineForWorkspaceContext key)
- Pieces:
  1. A Pin Workspace Timeline button in the Timeline title (`timeline.showWorkspaceTimeline`) switches the pane from one file to a timeline for the whole workspace. [stock only]
  2. While the workspace timeline shows, the Timeline header reads "Workspace" instead of a file name. [stock only]
  3. While the workspace timeline loads, the pane says "Loading workspace timeline...". [stock only]
  4. The Pin Workspace Timeline button shows only when some source can provide a workspace timeline and the pane is not already showing it. [stock only]

#### SB-7.12 Local History entries in the Timeline

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for the source id `timeline.localHistory` and the five settings, all found; the string "Local History" has 6 hits in NLS)
- What: Every save of a file adds a Local History entry to the Timeline, labelled with what caused the save and timestamped, with a hover card. Saves within a short window merge into one entry. Settings cap the file size and the number of entries per file, exclude paths, or turn the source off. Clicking an entry opens a comparison with the current file.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.localHistory.enabled; workbench.localHistory.maxFileSize; workbench.localHistory.maxFileEntries; workbench.localHistory.exclude; workbench.localHistory.mergeWindow
- Source: V\src\vs\workbench\contrib\localHistory\browser\localHistoryTimeline.ts (30-165); V\src\vs\workbench\browser\workbench.contribution.ts (461-497); C bundle (Local History timeline source)
- Pieces:
  1. Every save of a file adds a Local History entry to the Timeline, timestamped and labelled with what caused the save.
  2. Hovering a Local History entry shows a card with its details.
  3. Saves that happen within a short window replace the last entry instead of adding a new one, and setting `workbench.localHistory.mergeWindow` chooses that window in seconds, default 10.
  4. Clicking a Local History entry opens a comparison between that entry and the current file.
  5. Setting `workbench.localHistory.enabled` turns Local History on or off, default on, and turning it off stops new entries without removing old ones.
  6. Setting `workbench.localHistory.maxFileSize` chooses the largest file in KB that gets entries, default 256, and larger files get none.
  7. Setting `workbench.localHistory.maxFileEntries` chooses how many entries a file keeps, default 50, and the oldest entries are dropped past that.
  8. Setting `workbench.localHistory.exclude` lists paths or glob patterns whose files get no Local History entries.
  9. When a file has no entries the pane says "Local History will track recent changes as you save them unless the file has been excluded or is too large."

#### SB-7.13 Compare a Local History entry

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all four ids found in the bundle)
- What: Right-click an entry to compare it with the current file, compare it with the entry before it, or mark it with Select for Compare and later pick Compare with Selected on another entry.
- Commands: workbench.action.localHistory.compareWithFile; workbench.action.localHistory.compareWithPrevious; workbench.action.localHistory.selectForCompare; workbench.action.localHistory.compareWithSelected
- Menus: TimelineItemContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\localHistory\browser\localHistoryCommands.ts (49-180); C bundle (localHistory compare commands)
- Pieces:
  1. Right-clicking a Local History entry and choosing Compare with File opens a comparison between that entry and the current file.
  2. Choosing Compare with Previous opens a comparison between that entry and the entry before it.
  3. Choosing Select for Compare marks that entry as the left side of a comparison.
  4. After one entry is marked, right-clicking another Local History entry shows Compare with Selected, and choosing it opens a comparison of the two.
  5. The comparison tab title names each side with the file name plus its save source and time in brackets, and a Compare with File tab ends with the file name again.

#### SB-7.14 Show or restore a Local History entry

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all four ids found, and the string "Find Entry to Restore" is in NLS)
- What: Right-click an entry and choose Show Contents to open that old copy, or Restore Contents to put it back after a warning that unsaved changes are lost. An old copy open in an editor has a Restore Contents button in its title. Find Entry to Restore opens a picker over every file's history, from the Command Palette or the Timeline "..." menu.
- Commands: workbench.action.localHistory.open; workbench.action.localHistory.restore; workbench.action.localHistory.restoreViaEditor; workbench.action.localHistory.restoreViaPicker; workbench.action.localHistory.revealInOS
- Menus: TimelineItemContext; EditorTitle; TimelineTitle; CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\localHistory\browser\localHistoryCommands.ts (184-440); C bundle (localHistory open, restore and picker commands)
- Pieces:
  1. Right-clicking a Local History entry and choosing Show Contents opens that old copy in an editor.
  2. Choosing Restore Contents on an entry first asks "Do you want to restore the contents of '<file>'?" with the note "Restoring will discard any unsaved changes." and a Restore button.
  3. Confirming Restore replaces the file with that old copy, and if it fails an error dialog says "Unable to restore '<file>'."
  4. An old copy open in an editor shows a Restore Contents button in the editor title (`workbench.action.localHistory.restoreViaEditor`).
  5. Find Entry to Restore, in the Command Palette and in the Timeline "..." menu as "Local History: Find Entry to Restore...", opens a picker first for the file ("Select the file to show local history for") and then for the entry ("Select the local history entry to open").
  6. The "Local History: Find Entry to Restore..." row in the Timeline "..." menu shows only while `workbench.localHistory.enabled` is on.

#### SB-7.15 Rename, delete or create Local History entries

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all four ids found in the bundle)
- What: Right-click an entry to rename it or delete it (after a warning that it cannot be undone). Delete All clears the history of every file, and Create Entry saves a named snapshot of the active file by hand.
- Commands: workbench.action.localHistory.rename; workbench.action.localHistory.delete; workbench.action.localHistory.deleteAll; workbench.action.localHistory.create
- Menus: TimelineItemContext; CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\localHistory\browser\localHistoryCommands.ts (440-600); C bundle (localHistory rename, delete and create commands)
- Pieces:
  1. Right-clicking a Local History entry and choosing Rename opens an input titled "Rename Local History Entry" with the placeholder "Enter the new name of the local history entry".
  2. Confirming a new name changes the label of that entry in the Timeline.
  3. Right-clicking an entry and choosing Delete asks "Do you want to delete the local history entry of '<file>' from <date>?" with the note "This action is irreversible!" and a Delete button.
  4. Confirming Delete removes that entry from the Timeline.
  5. Delete All in the Command Palette asks "Do you want to delete all entries of all files in local history?" with the note "This action is irreversible!" and a Delete All button, and confirming clears the history of every file.
  6. Create Entry in the Command Palette opens an input titled "Create Local History Entry" for naming a snapshot of the active file, and confirming adds that snapshot as a new entry.

#### SB-7.16 Git History entries in the Timeline

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the three settings and the six `git.timeline.*` commands are in the Cursor git manifest, and `git-history` and "Git History" are in the Cursor git extension code)
- What: The Git History source lists the commits that changed the open file with message, author and relative time. A hover card shows the full commit, and a Staged Changes row sits at the top, plus an Uncommitted Changes row if turned on. Settings pick the authored or committed date, hide the author, and show uncommitted changes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.timeline.date; git.timeline.showAuthor; git.timeline.showUncommitted
- Source: V\extensions\git\src\timelineProvider.ts (18-280); V\extensions\git\package.json (settings `git.timeline.*`); C extensions (git\package.json, git\dist\main.js)
- Pieces:
  1. The Git History source lists the commits that changed the open file, each row showing the commit message and a relative time.
  2. Each Git History row shows the commit author, and setting `git.timeline.showAuthor` turns the author on or off, default on.
  3. Hovering a Git History row shows a card with the full commit.
  4. Setting `git.timeline.date` chooses whether row times use the committed date or the authored date, with values `committed` (default) and `authored`.
  5. A Staged Changes row sits at the top of the Git History entries when the file has staged changes.
  6. Setting `git.timeline.showUncommitted` adds an Uncommitted Changes row for the file, default off.

#### SB-7.17 Open a Git history entry's changes or commit

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids and their menu entries are in the Cursor git manifest)
- What: Click a Git History row to open its changes in a diff, or use the inline button or right-click View Commit to open the whole commit in a multi-file diff.
- Commands: git.timeline.openDiff; git.timeline.viewCommit
- Menus: timeline/item/context
- Keys: none found
- Settings: none found
- Source: V\extensions\git\package.json (timeline/item/context); V\extensions\git\src\commands.ts (4779-4866); C extensions (git\package.json)
- Pieces:
  1. Clicking a Git History row opens the changes of that commit for the file in a diff editor.
  2. Right-clicking a Git History row and choosing Open Changes (`git.timeline.openDiff`) does the same thing.
  3. A View Commit button shows inline on commit rows, and View Commit in the right-click menu does the same, opening the whole commit in a multi-file diff.
  4. Open Changes shows on every Git History row, but View Commit shows only on real commit rows and not on the Staged Changes or Uncommitted Changes rows.
  5. None of these entries show when several Timeline rows are selected, or when `git.enabled` is off or Git is missing.

#### SB-7.18 Compare two Git history entries

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids and their menu entries are in the Cursor git manifest)
- What: Right-click a Git History row, choose Select for Compare, then right-click another row and choose Compare with Selected.
- Commands: git.timeline.selectForCompare; git.timeline.compareWithSelected
- Menus: timeline/item/context
- Keys: none found
- Settings: none found
- Source: V\extensions\git\package.json (timeline/item/context); V\extensions\git\src\commands.ts (4887-4929); C extensions (git\package.json)
- Pieces:
  1. Right-clicking a Git History row and choosing Select for Compare marks that commit as the left side of a comparison.
  2. After one row is marked, right-clicking a different Git History row shows Compare with Selected.
  3. Choosing Compare with Selected opens a diff of the file between the two commits.

#### SB-7.19 Copy a commit hash or message from the Timeline

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids and their menu entries are in the Cursor git manifest)
- What: Right-click a Git History row and choose Copy Commit Hash or Copy Commit Message.
- Commands: git.timeline.copyCommitId; git.timeline.copyCommitMessage
- Menus: timeline/item/context
- Keys: none found
- Settings: none found
- Source: V\extensions\git\package.json (timeline/item/context); V\extensions\git\src\commands.ts (4867-4886); C extensions (git\package.json)
- Pieces:
  1. Right-clicking a Git History row and choosing Copy Commit Hash puts that commit's full hash on the clipboard.
  2. Right-clicking a Git History row and choosing Copy Commit Message puts that commit's message on the clipboard.
  3. The two Copy rows show only on real commit rows, not on the Staged Changes or Uncommitted Changes rows, and not when several rows are selected.

#### SB-7.20 Find All References in the References view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the Cursor references-view manifest has the same command, key, menu entry and setting, and `references-view` has 5 hits in the bundle)
- What: Run Find All References on a symbol to list every use in the Reference Search Results view, which has its own References icon and stays open while you edit. A setting makes CodeLens reference counts open this view instead of a peek.
- Commands: references-view.findReferences
- Menus: editor/context; commandPalette
- Keys: Shift+Alt+F12 (when editorHasReferenceProvider)
- Settings: references.preferredLocation
- Source: V\extensions\references-view\package.json (32-66, 67-177, 178-373, 374); V\extensions\references-view\src\references\index.ts (30-70); C extensions (references-view\package.json)
- Pieces:
  1. Running Find All References on a symbol fills the Reference Search Results view with every use of that symbol.
  2. Find All References is reached by right-click in the editor, by the Command Palette, and by `Shift+Alt+F12`, and all three only work when the language has a reference provider.
  3. The view lives in its own References container with a References icon, and it stays open and keeps its results while you edit and switch files.
  4. Setting `references.preferredLocation` chooses what clicking a CodeLens reference count opens, with values `peek` (default, a peek editor) and `view` (this References view).

#### SB-7.21 Find All Implementations in the References view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (command and editor/context entry are in the Cursor references-view manifest)
- What: Run Find All Implementations on an interface or abstract member to list the implementing places in the same References view.
- Commands: references-view.findImplementations
- Menus: editor/context
- Keys: none found
- Settings: none found
- Source: V\extensions\references-view\package.json (commands, menus editor/context); V\extensions\references-view\src\references\index.ts; C extensions (references-view\package.json)
- Pieces:
  1. Running Find All Implementations on an interface or abstract member lists the implementing places in the Reference Search Results view.
  2. The Find All Implementations row in the editor right-click menu shows only when the language has an implementation provider.

#### SB-7.22 Browse the reference results

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the view `references-view.tree` with its `when` clause `reference-list.isActive` is in the Cursor references-view manifest, same as stock)
- What: Results are grouped by file with each matching line shown. A line above the tree reads "N results in M files". Clicking a row opens the file at that match, the matches are highlighted in the editor and its overview ruler while the view is open, and rows can be dragged out as links.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\extensions\references-view\src\tree.ts (60-130, 189-222); V\extensions\references-view\src\references\model.ts (105-126, 295-305); V\extensions\references-view\src\highlights.ts (11-15); V\extensions\references-view\src\navigation.ts (47-52); C extensions (references-view\package.json)
- Pieces:
  1. Results are grouped by file, with each matching line shown under its file row.
  2. A line above the tree reads "N results in M files", with singular forms such as "1 result in 1 file".
  3. Clicking a result row opens that file in the editor at the match.
  4. While the view is open the matches are highlighted in the editor text and in its overview ruler.
  5. Result rows can be dragged out of the view as links.
  6. When a search finds nothing the view says "No results."
  7. The Reference Search Results view appears only after a search has run (`reference-list.isActive`).

#### SB-7.23 Step to the next or previous reference

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids and both keys are in the Cursor references-view manifest)
- What: Press F4 or Shift+F4 to jump to the next or previous result, opening it in the editor and selecting it in the view.
- Commands: references-view.next; references-view.prev
- Menus: commandPalette
- Keys: F4 (when reference-list.hasResult); Shift+F4 (when reference-list.hasResult)
- Settings: none found
- Source: V\extensions\references-view\package.json (374); V\extensions\references-view\src\navigation.ts (19-85); C extensions (references-view\package.json)
- Pieces:
  1. Pressing `F4` jumps to the next result, opening it in the editor and selecting it in the view.
  2. Pressing `Shift+F4` jumps to the previous result, opening it in the editor and selecting it in the view.
  3. Both keys work only while the view has results (`reference-list.hasResult`).
  4. Go to Next Reference and Go to Previous Reference are hidden from the Command Palette.

#### SB-7.24 Dismiss or copy results

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all four ids and their menu entries are in the Cursor references-view manifest)
- What: Each result row has a Dismiss button that removes it from the list. Right-click offers Copy, Copy All and, on a file row, Copy Path.
- Commands: references-view.removeReferenceItem; references-view.copy; references-view.copyAll; references-view.copyPath
- Menus: view/item/context; commandPalette; ViewItemContext
- Keys: none found
- Settings: none found
- Source: V\extensions\references-view\package.json (menus view/item/context); C extensions (references-view\package.json)
- Pieces:
  1. Each file row and reference row has an inline Dismiss button that removes that row from the list.
  2. Right-clicking a file row or reference row also offers Dismiss.
  3. Right-clicking a file row or reference row offers Copy, which puts that row's text on the clipboard, and Copy All, which puts all results on the clipboard.
  4. Right-clicking a file row also offers Copy Path, which puts the file path on the clipboard, and reference rows do not have it.
  5. Dismiss, Copy, Copy All and Copy Path are hidden from the Command Palette.

#### SB-7.25 Refresh or clear the results

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids and their view/title entries are in the Cursor references-view manifest)
- What: The view title has a Refresh button that reruns the search and a Clear button that empties the results.
- Commands: references-view.refresh; references-view.clear
- Menus: view/title; commandPalette
- Keys: none found
- Settings: none found
- Source: V\extensions\references-view\package.json (menus view/title); C extensions (references-view\package.json)
- Pieces:
  1. A Refresh button in the view title reruns the search that made the current results.
  2. A Clear button in the view title empties the results.
  3. After Clear the view says "No results." or, when earlier searches exist, "No results. Try running a previous search again:" above the history list.
  4. Refresh and Clear show in the view title and the Command Palette only while the view has results.

#### SB-7.26 Search history in the References view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all three ids and their menu entries are in the Cursor references-view manifest)
- What: When the view has no results it lists earlier searches with a Rerun button on each. Show History picks one from a list, and Clear History forgets them all.
- Commands: references-view.pickFromHistory; references-view.clearHistory; references-view.refind
- Menus: view/title; view/item/context; commandPalette
- Keys: none found
- Settings: none found
- Source: V\extensions\references-view\package.json (menus view/title, view/item/context, commandPalette); V\extensions\references-view\src\tree.ts (119-129, 224-260); C extensions (references-view\package.json)
- Pieces:
  1. When the view has no results it lists earlier searches, each with a Rerun button.
  2. Pressing Rerun on a history row, or choosing Rerun from its right-click menu, runs that search again.
  3. Show History (`references-view.pickFromHistory`) opens a picker titled "Select previous reference search" and choosing one shows its results.
  4. Clear History forgets every earlier search, and its button shows in the view title only while there is history and no results.
  5. Show History and Clear History are in the Command Palette only while the References view is active, and Clear History also needs no current results.

#### SB-7.27 Call hierarchy in the References view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all four ids, the key and the menu entries are in the Cursor references-view manifest)
- What: Show Call Hierarchy lists the callers of the symbol as a tree in the References view. A title button switches between Callers Of and Calls From, and a row's right-click menu shows incoming or outgoing calls for that row. Dismiss removes a row.
- Commands: references-view.showCallHierarchy; references-view.showIncomingCalls; references-view.showOutgoingCalls; references-view.removeCallItem
- Menus: editor/context; view/title; view/item/context
- Keys: Shift+Alt+H (when editorHasCallHierarchyProvider)
- Settings: none found
- Source: V\extensions\references-view\package.json (67-177, 178-373, 374); V\extensions\references-view\src\calls\index.ts; V\extensions\references-view\src\calls\model.ts (15-40, 200-215); C extensions (references-view\package.json)
- Pieces:
  1. Show Call Hierarchy lists the callers of the symbol as a tree in the Reference Search Results view.
  2. Show Call Hierarchy is reached by right-click in the editor and by `Shift+Alt+H`, and both work only when the language has a call hierarchy provider.
  3. The tree is headed Callers Of or Calls From, and a title button (Show Outgoing Calls while callers show, Show Incoming Calls while calls-from show) switches the whole tree to the other direction.
  4. Right-clicking a call row offers Show Incoming Calls and Show Outgoing Calls for that row.
  5. Each call row has an inline Dismiss button, and Dismiss in its right-click menu removes the row.
  6. When the symbol has no calls the view says "No results."

#### SB-7.28 Type hierarchy in the References view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all four ids and the menu entries are in the Cursor references-view manifest)
- What: Show Type Hierarchy lists the supertypes or subtypes of a type in the References view. A title button switches between Supertypes Of and Subtypes Of, a row's right-click menu shows supertypes or subtypes for that row, and Dismiss removes a row.
- Commands: references-view.showTypeHierarchy; references-view.showSupertypes; references-view.showSubtypes; references-view.removeTypeItem
- Menus: editor/context; view/title; view/item/context
- Keys: none found
- Settings: none found
- Source: V\extensions\references-view\package.json (67-177, 178-373); V\extensions\references-view\src\types\index.ts; V\extensions\references-view\src\types\model.ts (15-40, 170-180); C extensions (references-view\package.json)
- Pieces:
  1. Show Type Hierarchy lists the supertypes or subtypes of a type as a tree in the Reference Search Results view.
  2. Show Type Hierarchy is reached by right-click in the editor, and only works when the language has a type hierarchy provider.
  3. The tree is headed Supertypes Of or Subtypes Of, and a title button (Show Supertypes or Show Subtypes, whichever is not the current one) switches the whole tree to the other direction.
  4. Right-clicking a type row offers Show Supertypes and Show Subtypes for that row.
  5. Each type row has an inline Dismiss button, and Dismiss in its right-click menu removes the row.
  6. When the type has no supertypes or subtypes the view says "No results."
