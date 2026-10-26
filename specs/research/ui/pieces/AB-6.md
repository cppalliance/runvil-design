Group: AB-6 Pinned container tab row. Pass: 3. Date: 2026-10-25.

### AB-6.1
1. Setting `workbench.activityBar.orientation` chooses the layout, with `horizontal` (the default, no rail, and the Primary Side Bar starts with the pinned tab row) and `vertical` (a stock style 48 px rail plus the older stock side bar).
2. In the Settings editor the setting sits under Workbench > Appearance with the text "Controls the orientation of the activity bar in the workbench. Quit and reopen for changes to take effect.".
3. No command, menu item or key flips the orientation, and the choice is read once at startup.
4. In horizontal mode a container with one view keeps that view's own header instead of merging it into the container title.

### AB-6.2
1. When the orientation value changes, a confirm dialog says "A setting has changed that requires a restart to take effect." with the detail "Press the restart button to restart {product name} and enable the setting.".
2. The dialog's Restart button restarts the app, and the web build says reload instead.
3. The dialog shows only while the window has focus and only when the value really changed.
4. If you decline the dialog, the saved value stays and the layout switches at the next start.

### AB-6.3
1. In the default layout the Primary Side Bar starts with a 35 px row of icon-only tabs, one per pinned container, centered with a 2 px gap.
2. Each tab is 26 px square with a 16 px icon and shows no text, tooltip or badge.
3. The tab of the open container gets the selected style, with a selected background and the active panel title color, plus a contrast outline in high contrast themes.
4. Hovering a tab uses the list hover background.
5. A first run pins Explorer, Search, Source Control and Extensions.
6. Run and Debug, Testing and every other container show only in the chevron list until they are pinned.

### AB-6.4
1. Clicking a tab opens that container in the side bar and moves focus into it.
2. Clicking the tab of the container that is already open does nothing, whatever `workbench.activityBar.iconClickBehavior` says.
3. By code reading, the tabs, the chevron and the list work only with the mouse, with no tab stops, arrow keys, ARIA roles or hover tooltips.
4. Commands such as Show Explorer and Show Search still open the same containers, and the selected tab follows the open container.

### AB-6.5
1. The row shows only as many pinned tabs as fit, which is the row width divided by 40 px rounded down, minus one slot for the chevron, and minus one more when the open container is not pinned.
2. At the 214 px minimum side bar width the row shows 4 tabs, or 3 when an extra tab is showing.
3. A pinned tab that does not fit has no tab and can be reached only from the list.
4. The count of tabs follows the side bar width as it changes, so widening the side bar brings tabs back.

### AB-6.6
1. When the open container is not pinned, the row adds one extra selected tab at its end for that container.
2. Switching to a pinned container removes the extra tab.
3. Right-clicking the extra tab opens the same tab menu as any other tab.

### AB-6.7
1. A chevron button after the tabs opens a list under the row, lying over the view content with a soft shadow.
2. The chevron points down while the list is closed, and points up and is highlighted while the list is open.
3. The list has one row per container in the side bar, showing the icon, the name, the container's keyboard shortcut at half opacity when it has one, and a pin icon.
4. Pinned containers come first in the list, and the row of the open container is highlighted.
5. Clicking a row opens that container.
6. The list closes after a pick, when the open container changes, and on a second click on the chevron, and by code reading clicking outside the list or pressing Escape does not close it.
7. Containers marked hide when empty that have no active views are not listed.

### AB-6.8
1. Clicking the pin icon on a list row adds or removes that container's tab, and the click does not open the row.
2. The pin icon is hidden on unpinned rows until the row is hovered.
3. In a tab's right-click menu, Hide '{name}' unpins that container and Keep '{name}' pins it.
4. The checklist at the bottom of the tab menu toggles any container, with a check on the pinned ones.
5. The tab menu entries are disabled for the last pinned container, but the pin icon in the list has no such guard, so the list can unpin every container.

### AB-6.9
1. Dragging a list row up or down changes its place in the list.
2. A row moves only among rows of its own kind, pinned among pinned and unpinned among unpinned.
3. While a row is dragged it drops to half opacity and a copy follows the pointer.
4. The order of the tabs in the row follows the pinned order in the list.
5. The list is ordered pinned first, then by the saved order, then by the container's own order, then by a fixed tiebreak computed from the container id.
6. The tabs in the row cannot be dragged.

### AB-6.10
1. Right-clicking a tab, including the extra tab, opens a menu.
2. The menu lists, in order, Configure Keybinding when the container has a key id, then Hide or Keep '{name}', then the container's title-context items, then a separator, then a checklist of every container.
3. The title-context items are a show or hide toggle for each of the container's views, Reset Location when the container was moved from its home, and Hide Primary Side Bar.
4. The menu does not offer Hide Badge or Show Badge, Move To, Activity Bar Position, Activity Bar Size, the Accounts and Menu toggles, or Move Primary Side Bar Right or Left, which the stock rail icon menu has.
5. Right-clicking a row in the chevron list does nothing.

### AB-6.11
1. The first item in the tab menu, Configure Keybinding, shows the container's current key after a tab character, or only the label when no key is set.
2. Picking Configure Keybinding opens the Keyboard Shortcuts editor filtered to that container's open command.
3. The item shows only for containers that have a keybinding id.

### AB-6.12
1. Right-clicking blank space in the row, not on a tab, opens a menu with one item, Move Primary Side Bar Right, or Move Primary Side Bar Left when the bar is already on the right.
2. The item has a tooltip that repeats the same words.
3. Picking the item flips the Primary Side Bar to the other side, changing `workbench.sideBar.location`.

### AB-6.13
1. While Run and Debug is open, the title grows from 35 px to 70 px and a second centered row shows that container's title actions.
2. The second row has the Start Debugging button with the launch configuration picker, the Open 'launch.json' gear, and an "Actions..." overflow menu for the secondary actions.
3. The second row is hidden while the chevron list is open.
4. Only Run and Debug gets this second row, and other containers show no container-level toolbar in the tab row because their views keep their own headers.

### AB-6.14
1. Pinning, unpinning and reordering are remembered across restarts and apply to every window and workspace on the machine, because they are saved under `sidebar2.sidebarData.memoized.v1` in application scope.
2. The open container is saved per workspace and comes back at startup, with Explorer, the default, saved as no value.
3. A bad saved value is ignored with an error in the console.
4. The vertical rail keeps its own pins, so each orientation starts from its own pins and order.

## Unclear
- Every AB-6 piece is a Cursor only behavior by nature, so the `[cursor only]` suffix is left off inside this file as the assignment allows.
- The always-pinned Agents container tab (`workbench.view.backgroundAgent`) is an agent surface and is out of scope, so AB-6.8 has no piece for the Agents container always counting as pinned.
- Nothing was run. Pixel values (35, 26, 16, 2, 40, 214, 70) come from the feature blocks, which read them from code and CSS, and hover or tooltip absence is by code reading.
- AB-6.7 piece 6 and AB-6.4 piece 3 state absences found by searching the bundle code (no outside click or Escape handling, no keyboard or tooltip code). They could be wrong in a running app.
- AB-6.9 leaves out how the new place is stored (a number halfway between neighbors) because it is not something a person sees.
- AB-6.10 piece 3 gives Hide Primary Side Bar as an item only. What it does belongs to SB-1.
- AB-6.1 piece 2 was derived from the setting id and the group list, as the feature block says. Cursor Settings was not checked for an orientation entry.
