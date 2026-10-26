Group: AB-1 Rail icons and view switching. Pass: 3. Date: 2026-10-25.

### AB-1.1
1. A vertical strip of view container icons sits beside the side bar, 48 px wide with 24 px icons, showing Explorer, Search, Source Control, Run and Debug, Extensions, and any container an extension adds.
2. Each icon is a tab in a tablist that screen readers name Active View Switcher.
3. In Cursor's default horizontal setting the stock rail is hidden and a 26 px icon-only row of tabs shows at the top of the side bar instead. [cursor only]

### AB-1.2
1. The open view's icon is drawn in the active color with a 2 px bar on the outer edge of the rail, and the other icons are dimmed.
2. `activityBar.background`, `activityBar.border`, `activityBar.foreground`, and `activityBar.inactiveForeground` set the rail's background, its border, the open icon's color, and the dimmed icons' color.
3. `activityBar.activeBorder`, `activityBar.activeFocusBorder`, and `activityBar.activeBackground` set the bar's color, the bar's color when the icon has keyboard focus, and the fill behind the open icon.
4. `activityBar.dropBorder` sets the color of the drop line shown while an icon is dragged over the rail.
5. High contrast themes hide the bar unless the icon has keyboard focus.
6. Putting any of the `activityBar.*` ids in `workbench.colorCustomizations` overrides the theme's value for that color.
7. Cursor's horizontal row fills the selected tab with a background and draws no bar. [cursor only]

### AB-1.3
1. Clicking an icon opens that container in the primary side bar.
2. Clicking an icon while the side bar is hidden reveals the side bar with that container open.
3. A right click on an icon never switches the view.
4. A second click within 300 ms is ignored, so a double click does not toggle the view back.
5. In Cursor's horizontal row, a click on a tab also opens that container. [cursor only]

### AB-1.4
1. Clicking the icon of the view that is already open hides the primary side bar by default.
2. Setting `workbench.activityBar.iconClickBehavior` chooses what clicking the open icon does, with values `toggle` (hide the primary side bar, the default) and `focus` (move keyboard focus into the primary side bar).
3. The `workbench.activityBar.iconClickBehavior` value is ignored unless `workbench.activityBar.location` is `default`.
4. In Cursor's horizontal row the setting is ignored and a click on the open tab does nothing. [cursor only]

### AB-1.5
1. Hovering an icon shows a tooltip with its name followed by the shortcut of its open command in brackets, such as Explorer (Ctrl+Shift+E).
2. The tooltip appears on the side of the icon that faces the side bar.
3. An icon whose open command has no shortcut shows only its name, and changing the shortcut updates the tooltip text.
4. The same text is the icon's label for screen readers.
5. Cursor shows the tooltip at once on mouse over, where stock waits for the hover delay. [cursor only]

### AB-1.6
1. When the pinned icons do not fit the height of the rail, the extra ones move behind a "..." button at the bottom of the icon list.
2. The "..." button is named Additional Views and opens a menu that lists the icons that did not fit as radio items.
3. Choosing an item in the menu opens that view, and the item for the open view shows as checked.
4. An item whose icon has a number badge shows the number in brackets after its name.
5. Cursor's horizontal row has a chevron drop-down in place of the "..." button, with pin toggles and drag reorder. [cursor only]

### AB-1.7
1. The container that is open always has an icon on the rail, even if its icon was hidden or would not fit.
2. An unpinned icon that is on the rail only because its view is open goes away again once the person switches to another view.
3. Cursor's horizontal row shows the open unpinned container as one extra selected tab at the end of the row. [cursor only]

### AB-1.8
1. A container marked to hide when empty shows no icon on the rail while all its views are inactive, as Testing, Remote Explorer, and References do.
2. The icon appears as soon as one of the container's views becomes active, for example the References icon after a references command fills its list.
3. The icon goes away again when all of the container's views become inactive.

### AB-1.9
1. Pressing a container's shortcut opens that view in the primary side bar and puts keyboard focus in it.
2. Ctrl+Shift+E (`workbench.view.explorer`) opens Explorer, Ctrl+Shift+G (`workbench.view.scm`) opens Source Control, Ctrl+Shift+D (`workbench.view.debug`) opens Run and Debug, and Ctrl+Shift+X (`workbench.view.extensions`) opens Extensions.
3. `workbench.view.search` opens Search from the Command Palette but has no working key of its own, because Ctrl+Shift+F runs Find in Files instead.
4. `workbench.view.extension.test` opens Testing and has no default key.
5. Each Show command is listed in the Command Palette in the View category, and each container also has an entry in the View menu.
6. Pressing the shortcut again while the side bar already has focus sends focus back to the editor.

### AB-1.10
1. Previous Primary Side Bar View (`workbench.action.previousSideBarView`) moves the open view one step back through the icons on the rail.
2. Next Primary Side Bar View (`workbench.action.nextSideBarView`) moves the open view one step forward through the icons on the rail.
3. Both commands wrap around, so stepping back from the first icon lands on the last and stepping forward from the last lands on the first.
4. The steps count only the icons currently on the rail, which are the pinned icons plus the open one.
5. Both commands are listed in the Command Palette and have no default key.
6. In Cursor's horizontal row both commands run but cycle through every container, pinned or not. [cursor only]

### AB-1.11
1. Running Focus Activity Bar (`workbench.action.focusActivityBar`) puts keyboard focus on the rail.
2. If the activity bar location was set to hidden, the command brings the bar back first and then focuses it.
3. The command is listed in the Command Palette and has no default key.
4. In Cursor the command is enabled only when `workbench.activityBar.orientation` is `vertical`, so it is disabled in the default horizontal row. [cursor only]
5. In Cursor, running the command when the rail does not exist shows an info dialog telling the person to quit and reopen. [cursor only]

### AB-1.12
1. With focus on the rail, Down Arrow or Right Arrow moves to the next icon and Up Arrow or Left Arrow moves to the previous icon, stopping at the ends instead of looping.
2. Home jumps to the first icon and End jumps to the last icon.
3. Enter or Space opens the focused icon.
4. Pressing Down Arrow past the last icon moves focus into the Accounts and Manage icons.
5. Pressing Up Arrow before the first icon moves focus to the compact Menu button when it is shown.
6. These keys are fixed in the action bar and cannot be rebound.

## Unclear

- AB-1.12 has no Cursor piece. The feature file says Cursor's horizontal row has no key handling in its tab code, which is an absence and not a behavior a person can check, so it is left out.
- AB-1.5 has no piece for tooltips in Cursor's horizontal row, because the feature file could not tell whether it shows one (not run).
- AB-1.6 piece 3 says the open view's item shows checked, as the feature describes, but AB-1.7 keeps the open icon on the rail, so the checked item may rarely be visible. Not run.
- AB-1.9 could be split by view (one piece group per Show command), and the `workbench.view.*` commands may also be claimed by SB-2 to SB-6. They are kept together here.
