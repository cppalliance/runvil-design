Group: AB-2 Customizing the rail. Pass: 3. Date: 2026-10-25.

### AB-2.1
1. Right-clicking an icon and choosing Hide 'Name' takes that icon off the rail.
2. An icon that is not pinned shows Keep 'Name' in its menu instead of Hide, and choosing it puts the icon back on the rail to stay.
3. Hide is greyed out when the icon is the last one left on the rail.
4. Hiding the icon of the open view opens the default view (Explorer) or the next icon's view instead.
5. Cursor's horizontal tab menu has the same Hide and Keep entries. [cursor only]

### AB-2.2
1. Right-clicking empty space on the rail opens a checklist of every available container, with a tick on each one that is on the rail.
2. Ticking an item adds its icon to the rail and opens it.
3. Unticking an item removes its icon from the rail.
4. The last ticked item cannot be unticked.
5. In Cursor's horizontal row the checklist sits at the bottom of the tab menu, and right-clicking empty row space offers only Move Primary Side Bar Right or Left. [cursor only]

### AB-2.3
1. Dragging an icon up or down the rail shows a drop line above or below the icon under the pointer, depending on which half of it the pointer is over.
2. Dropping an icon on empty rail space puts it first or last.
3. The new order is remembered the next time the window opens.
4. Cursor's horizontal tabs are not draggable, and only the rows of its chevron list reorder. [cursor only]

### AB-2.4
1. While a container or view is being dragged, resting the pointer over a rail icon for 0.8 seconds opens that icon's view, so the person can see where the drop will land.
2. Leaving the icon, dropping, or ending the drag before 0.8 seconds cancels it, and the view does not open.
3. An icon whose view is already open is left alone.
4. In Cursor's horizontal row, resting a dragged item over a tab does not open that tab's view. [cursor only]

### AB-2.5
1. Dropping a view container's tab from the Panel or the Secondary Side Bar onto the rail moves the whole container into the primary side bar and opens it.
2. The container's icon lands at the spot on the rail where it was dropped.
3. In Cursor's horizontal row, the rail drop for dragged containers is not available. [cursor only]

### AB-2.6
1. Dragging one view by its header onto the rail creates a new container that holds only that view.
2. The new container's icon is placed where it was dropped, and the container opens.
3. Only views that allow moving can be dragged this way.
4. In Cursor's horizontal row, the rail drop for dragged views is not available. [cursor only]

### AB-2.7
1. An icon's menu has a Move To submenu that lists Primary Side Bar, Secondary Side Bar, and Panel, leaving out the place the container already is.
2. Choosing a place sends the whole container to the end of that bar and opens it.
3. When the chosen bar is the container's home, the container goes back to its default spot instead of the end.
4. Cursor's horizontal tab menu has no Move To entry. [cursor only]

### AB-2.8
1. Reset Location shows in an icon's menu only when its container, or the single view it holds, is away from its default home.
2. Choosing Reset Location sends the container back to its default location and opens it.
3. For a container that holds a single moved view, Reset Location sends that view back to its default container.
4. Each container also has a Reset Location command (`<container id>.resetViewContainerLocation`) in its title menu.
5. Cursor's horizontal tab menu reaches Reset Location through the container title menu. [cursor only]

### AB-2.9
1. The first entry of an icon's menu is Configure Keybinding, and it shows only when the container has an open command.
2. Choosing it opens the Keyboard Shortcuts editor filtered to that container's command (`workbench.action.openGlobalKeybindings` with an `@command:` filter), so the person can assign or change its key.
3. Cursor's horizontal tab menu has the same Configure Keybinding entry. [cursor only]

### AB-2.10
1. A view container that is new to the rail, such as one added by an extension, is pinned automatically so its icon appears without any setup.
2. A container the window already knows keeps its saved pinned or hidden state instead of being pinned again.

### AB-2.11
1. Which icons are pinned, their order, and their names and icons are saved per profile.
2. When the window opens, the rail is drawn at once from the saved copy, before extensions finish loading.
3. Placeholder icons stand in for containers that have not registered yet, and the real containers replace them as they register.

## Unclear

- AB-2.4 piece 4, AB-2.5 piece 3, and AB-2.6 piece 4 come from the feature files saying Cursor's horizontal row code has no drop handler. Not run, so the results on screen are unconfirmed.
- AB-2.7 piece 1 lists Secondary Side Bar because stock and Cursor's code both do, but Cursor blocks contributing view containers to that bar, so what Cursor does when it is chosen was not run.
- Cursor's horizontal row keeps its own order and pin state in a separate saved key, so AB-2.3 piece 3 and AB-2.11 piece 1 describe the stock rail only.
- AB-2.1 piece 2 is read from the menu code, where the label switches between Hide and Keep by pinned state. Whether an icon that is not pinned can be right-clicked in any state other than being the open view was not checked.
