Group: SB-1 Side bar frame and shared view behavior. Pass: 3. Date: 2026-10-25.

### SB-1.1
1. Pressing `Ctrl+B` (`Cmd+B` on Mac) hides the primary side bar when it is showing and shows it when it is hidden, and the editor area takes the freed width.
2. Hiding the side bar closes its active container, and showing it again brings back the container used last.
3. View > Appearance > Primary Side Bar, the layout control buttons in the title bar, and Hide Primary Side Bar in the container right-click menu all toggle the side bar the same way.
4. Running Close Primary Side Bar (`workbench.action.closeSidebar`) from the Command Palette hides the side bar, and the entry is only enabled while the side bar is showing.
5. After the toggle, a screen reader announces "Primary Side Bar hidden" or "Primary Side Bar shown". [stock only]
6. In Cursor the toggle command and `Ctrl+B` do nothing while focus is in an auxiliary window, and the layout toggle button sits in the `2_pane_toggles` group of the layout menu instead of the navigation group. [cursor only]

### SB-1.2
1. Setting `workbench.sideBar.location` puts the primary side bar and the activity bar on the `left` (default) or `right` edge, and the secondary side bar goes to the opposite edge.
2. Running Toggle Primary Side Bar Position (`workbench.action.toggleSidebarPosition`) from the Command Palette flips the side bar and activity bar to the other edge.
3. View > Appearance shows Move Primary Side Bar Right while the side bar is on the left and Move Primary Side Bar Left while it is on the right, and choosing it moves the side bar.
4. The container right-click menu has the same Move Primary Side Bar Right or Left entry, shown only for a container that lives in the primary side bar.
5. In Cursor the toggle command first saves the choice under a Cursor layout override key and then updates the setting, and the horizontal tab row has its own right-click move entry (see AB-6). [cursor only]

### SB-1.3
1. Pressing `Ctrl+0` (`Cmd+0` on Mac) puts keyboard focus inside the active view of the primary side bar.
2. When the side bar is hidden, the same command opens it first and then moves focus into it.
3. Focus into Primary Side Bar (`workbench.action.focusSideBar`) is listed in the Command Palette.
4. In Cursor the key and the command do nothing while focus is in an auxiliary window. [cursor only]

### SB-1.4
1. Dragging the edge between the side bar and the editor changes the side bar width.
2. Double-clicking that edge jumps the side bar to a best-fit width of at least 300 pixels.
3. Dragging the edge below the minimum width snaps the side bar shut.
4. With focus in the side bar, Increase Current View Size, Decrease Current View Size, Increase Current View Width and Decrease Current View Width in the Command Palette (`workbench.action.increaseViewSize`, `decreaseViewSize`, `increaseViewWidth`, `decreaseViewWidth`) change its width by 60 pixels.
5. In Cursor the side bar cannot be narrower than 214 pixels (stock stops at 170), the horizontal side bar has a minimum height of 77 pixels, and the four resize commands do nothing while focus is in an auxiliary window. [cursor only]

### SB-1.5
1. The strip above the views shows the container's name, with the container's own buttons at its right edge.
2. A "..." button in that strip opens a menu whose first entry is a Views submenu listing the container's views.
3. Container actions in the navigation group (from the `SidebarTitle` and `ViewContainerTitle` menus) show as icon buttons in the strip, and the other container actions are listed in the "..." menu under the Views submenu.
4. In Cursor's default horizontal side bar the tab row replaces the title label and the container's actions go to a Cursor-made toolbar, while the stock title row remains in the vertical side bar. [cursor only]

### SB-1.6
1. Right-clicking the container title shows a menu that lists each of the container's views with a check mark on the visible ones, and choosing an entry shows or hides that view.
2. Right-clicking the empty space under the views shows the same menu.
3. Reset Location (`<containerId>.resetViewContainerLocation`) appears in the menu only when the container has been moved away from its default place, and choosing it sends the container home.
4. The menu has Move Primary Side Bar Right (or Left when the side bar is on the right) and Hide Primary Side Bar entries.
5. The menu has an Activity Bar Position submenu for choosing where the activity bar sits. [stock only]
6. In Cursor's default horizontal side bar the tab row has its own right-click menu (see AB-6) instead of this one. [cursor only]

### SB-1.7
1. Clicking a view header folds an open view and unfolds a folded one.
2. With a view header focused, Enter or Space toggles the view open or closed.
3. With a view header focused, Left Arrow folds the view and Right Arrow unfolds it.
4. Up Arrow and Down Arrow move focus from one view header to the previous or next one.
5. Each view's folded or open state is remembered.

### SB-1.8
1. Each view header shows its own buttons at the right when the pointer is over the header or focus is inside the view, and hides them otherwise.
2. A "..." menu on the view header holds the extra view title actions (the `ViewTitle` menu) that do not show as buttons.
3. Setting `workbench.view.alwaysShowHeaderActions` keeps the view header buttons visible all the time when on, and shows them only on hover or focus when off (the default).

### SB-1.9
1. The Views submenu in the container "..." menu lists every view of the container with a check mark on the visible ones, and choosing one shows or hides it (`<viewId>.toggleVisibility`).
2. The container right-click menu lists the same show or hide toggles for each view.
3. Right-clicking a view header shows the same toggles and a Hide '<view>' entry that hides that view (`<viewId>.removeView`).
4. The last visible view cannot be hidden, because its toggle and its Hide entry are disabled.

### SB-1.10
1. Dragging the line between two view headers gives one view more height and the other less.
2. Double-clicking that line splits the space between the two nearest open views evenly.
3. The sizes of the views are remembered.

### SB-1.11
1. Dragging a view header onto another view in the same container moves the dragged view to that spot.
2. While dragging, a highlight on the target view shows whether the view will land above or below it.
3. The new order of the views is saved.

### SB-1.12
1. Dragging a view header over another container, the panel or the secondary side bar shows a drop overlay on that area.
2. Dropping on the top, bottom, left or right part of the overlay places the view above, below, left or right of the views already there.
3. After the drop, the view appears in the target area and is gone from the container it came from.

### SB-1.13
1. Dragging a container title (for example EXPLORER) onto the panel or the secondary side bar moves all of that container's views together.
2. Dropping a container title onto another container adds all of its views to that container.
3. In Cursor's default horizontal side bar there is no title label to drag, and its tabs have their own reorder drag (see AB-2 and AB-6). [cursor only]

### SB-1.14
1. Running Move View (`workbench.action.moveView`) from the Command Palette opens a picker titled "Select a View to Move" that lists the movable views.
2. After a view is picked, a second picker titled "Select a Destination for the View" offers New Side Bar Entry, New Panel Entry, New Secondary Side Bar Entry and every existing container.
3. Move Focused View (`workbench.action.moveFocusedView`) skips the first picker and moves the view that has focus.
4. Choosing a destination moves the view there, and a New Entry destination creates a new container in that area to hold it.

### SB-1.15
1. With a view focused, `Ctrl+K` then Up Arrow or Left Arrow moves it one place earlier in its container (`views.moveViewUp`, `views.moveViewLeft`).
2. With a view focused, `Ctrl+K` then Down Arrow or Right Arrow moves it one place later in its container (`views.moveViewDown`, `views.moveViewRight`).
3. These chords work only while a view has focus, and they use `Cmd+K` on Mac.

### SB-1.16
1. Running Reset View Locations (`workbench.action.resetViewLocations`) from the Command Palette puts every moved view back in the container it started in.
2. The same command puts every moved container back in the area it started in.

### SB-1.17
1. Right-clicking a view header and choosing Reset Location sends that view back to the container it started in and opens it there.
2. Reset Location shows in the view header menu only while the view is not in its starting container.
3. Running Reset Focused View Location (`workbench.action.resetFocusedViewLocation`) from the Command Palette does the same for the view that has focus.
4. Running it with no view focused shows an error dialog saying "There is no view currently focused."

### SB-1.18
1. A view with nothing to show displays its welcome text instead of a blank list, for example the Explorer when no folder is open.
2. A welcome line that holds only a link shows as a button that runs its command, and links inside text open their target.
3. Welcome lines appear and disappear as their conditions change.
4. A welcome button or command link is greyed out and disabled while its precondition is not met.
5. When the view is wider than 640 pixels the welcome content switches to a wider layout.

### SB-1.19
1. The Command Palette lists a "Focus on <name> View" command for each view, and running it opens the view's container and puts focus inside the view (for example `outline.focus` and `timeline.focus`).
2. A keybinding can pass `preserveFocus` to the command to open the view without moving focus into it.
3. A view can declare its own shortcut for its focus command.
4. A view's focus command is listed only while that view is available.

### SB-1.20
1. When a container shows only one view, the view's own header is hidden.
2. The container title then shows the view's buttons.
3. The container title reads "Container: View" when the two names differ, and just the container name when they match.
4. When a second view becomes visible, the combined title goes away and each view gets its own header again.
5. In Cursor's default horizontal side bar the merged title text has no place because the tab row replaces the title label, and a single view that is not merged gets the lone-view state (see SB-1.21). [cursor only]

### SB-1.21
1. A view that is alone in its container is forced open and cannot be folded. [cursor only]
2. In the default horizontal side bar the collapse arrow of a lone view is hidden. [cursor only]
3. In the vertical side bar the collapse arrow of a lone view stays but clicking it does nothing. [cursor only]
4. Setting `workbench.activityBar.orientation` chooses `horizontal` (default) or `vertical`, which decides which of the two lone-view looks you see after a restart. [cursor only]

### SB-1.22
1. In the horizontal side bar a container can ask for an extra action row, and its title area grows from 35 to 70 pixels to hold it. [cursor only]
2. The Run and Debug container uses the two-row title, with the Start button and the launch configuration picker in the second row. [cursor only]
3. Containers that do not ask for the extra row keep the 35 pixel title. [cursor only]
4. Setting `workbench.activityBar.orientation` set to `vertical` removes the two-row title and brings back the stock title row. [cursor only]

### SB-1.23
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

### SB-1.24
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

## Unclear
- SB-1.1 piece 5 is tagged `[stock only]` because the two announcement strings were not found in the Cursor message file. Nothing was run, so the behavior is not seen on screen.
- SB-1.5 piece 4 and SB-1.6 piece 6 rest on the pass 2 note that the stock title right-click and the title "..." menu have no matching wiring in Cursor's horizontal side bar. Whether the "..." button shows there was not seen.
- SB-1.13 piece 2 and SB-1.12 piece 3 both end with views landing in another container, but one starts from a view header and the other from a container title, so they are kept as different experiences.
- SB-1.20 piece 4 comes from reading how the single-view merge is recomputed when the number of visible views changes, not from a stated line in the feature block.
- SB-1.23 is workbench-wide list and tree behavior, and SB-1.24 piece 8 is a theme color, not a control. The parent can move or drop them.
