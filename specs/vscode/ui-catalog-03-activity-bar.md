# 3. Activity bar (AB)

The icon rail that switches the side bar, and Cursor's replacement for it.

6 groups, 79 features. Each feature lists its entry points and the pieces that finish it.

## AB-1 Rail icons and view switching

(both). The icon strip, how a click or key switches the side bar view, and overflow. Examples: click with `iconClickBehavior`; active marker and hover text with the keybinding; the Additional Views menu; Previous and Next Primary Side Bar View.

12 features.

#### AB-1.1 Icon strip of view containers

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A vertical strip of view container icons (Explorer, Search, Source Control, Run and Debug, Extensions, plus any extension container) sits beside the side bar, 48 px wide with 24 px icons, and each icon is a tab in a tablist named Active View Switcher; in Cursor's default horizontal row this is a different 26 px icon-only tab row at the top of the side bar (AB-6) with no tablist or tab roles found.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; V\src\vs\workbench\browser\parts\paneCompositeBar.ts; V\src\vs\workbench\browser\parts\compositeBar.ts; V\src\vs\workbench\browser\parts\activitybar\media\activityaction.css; C bundle (rail part class `ore`, tablist label NLS 3321, `ariaRole:"tablist"`)
- Pieces:
  1. A vertical strip of view container icons sits beside the side bar, 48 px wide with 24 px icons, showing Explorer, Search, Source Control, Run and Debug, Extensions, and any container an extension adds.
  2. Each icon is a tab in a tablist that screen readers name Active View Switcher.
  3. In Cursor's default horizontal setting the stock rail is hidden and a 26 px icon-only row of tabs shows at the top of the side bar instead. [cursor only]

#### AB-1.2 Active icon marker and rail colors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The open view's icon is drawn in the active color with a 2 px bar on the outer edge of the rail and the other icons are dimmed, with colors from the activityBar.* theme colors (background, border, foreground, inactiveForeground, activeBorder, activeFocusBorder, activeBackground, dropBorder), and high contrast themes hide the bar unless the icon has focus; Cursor's horizontal row instead fills the selected tab with a background and has no bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Source: V\src\vs\workbench\browser\parts\activitybar\media\activityaction.css; V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; V\src\vs\workbench\common\theme.ts; C bundle (color ids activityBar.activeBorder, activityBar.activeFocusBorder, activityBar.activeBackground, activityBar.inactiveForeground, activityBar.dropBorder; CSS rule for `.activitybar.left` active-item-indicator)
- Pieces:
  1. The open view's icon is drawn in the active color with a 2 px bar on the outer edge of the rail, and the other icons are dimmed.
  2. `activityBar.background`, `activityBar.border`, `activityBar.foreground`, and `activityBar.inactiveForeground` set the rail's background, its border, the open icon's color, and the dimmed icons' color.
  3. `activityBar.activeBorder`, `activityBar.activeFocusBorder`, and `activityBar.activeBackground` set the bar's color, the bar's color when the icon has keyboard focus, and the fill behind the open icon.
  4. `activityBar.dropBorder` sets the color of the drop line shown while an icon is dragged over the rail.
  5. High contrast themes hide the bar unless the icon has keyboard focus.
  6. Putting any of the `activityBar.*` ids in `workbench.colorCustomizations` overrides the theme's value for that color.
  7. Cursor's horizontal row fills the selected tab with a background and draws no bar. [cursor only]

#### AB-1.3 Switch the side bar view by clicking an icon

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking an icon opens that container in the primary side bar and reveals the side bar if it was hidden, a right click never switches, and a second click within 300 ms is ignored so a double click does not toggle it back; in Cursor's horizontal row a click on a tab also opens that container.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts; V\src\vs\workbench\browser\parts\compositeBarActions.ts; C bundle (action class with `preventDoubleClickDelay=300`, right click guard `e.button===2`)
- Pieces:
  1. Clicking an icon opens that container in the primary side bar.
  2. Clicking an icon while the side bar is hidden reveals the side bar with that container open.
  3. A right click on an icon never switches the view.
  4. A second click within 300 ms is ignored, so a double click does not toggle the view back.
  5. In Cursor's horizontal row, a click on a tab also opens that container. [cursor only]

#### AB-1.4 Click the open icon again to hide or focus the side bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking the icon of the view that is already open either hides the primary side bar (`toggle`, the default) or moves focus into it (`focus`); in Cursor's horizontal row the setting is ignored and a click on the open tab does nothing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.iconClickBehavior
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (setting definition with NLS 4228 to 4230, the same switch in the rail action class)
- Pieces:
  1. Clicking the icon of the view that is already open hides the primary side bar by default.
  2. Setting `workbench.activityBar.iconClickBehavior` chooses what clicking the open icon does, with values `toggle` (hide the primary side bar, the default) and `focus` (move keyboard focus into the primary side bar).
  3. The `workbench.activityBar.iconClickBehavior` value is ignored unless `workbench.activityBar.location` is `default`.
  4. In Cursor's horizontal row the setting is ignored and a click on the open tab does nothing. [cursor only]

#### AB-1.5 Hover text with name and shortcut

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (hover appears at once, stock waits for the delay)
- What: Hovering an icon shows its name followed by the shortcut of its open command in brackets, such as Explorer (Ctrl+Shift+E), on the side facing the side bar, and the same text is the icon's screen reader label; Cursor shows the hover at once on mouse over where stock waits for the hover delay, and its horizontal row set no tooltip in code (not run).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; C bundle (composite bar item class using `showInstantHover` on mouse over, title format NLS 3322)
- Pieces:
  1. Hovering an icon shows a tooltip with its name followed by the shortcut of its open command in brackets, such as Explorer (Ctrl+Shift+E).
  2. The tooltip appears on the side of the icon that faces the side bar.
  3. An icon whose open command has no shortcut shows only its name, and changing the shortcut updates the tooltip text.
  4. The same text is the icon's label for screen readers.
  5. Cursor shows the tooltip at once on mouse over, where stock waits for the hover delay. [cursor only]

#### AB-1.6 Additional Views overflow button

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the pinned icons do not fit the height of the rail, the extra ones go behind a "..." button at the bottom of the icon list whose menu is titled Additional Views and lists them as radio items with the open one checked; Cursor's horizontal row has a chevron drop-down instead, with pin toggles and drag reorder.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts; V\src\vs\workbench\browser\parts\compositeBarActions.ts (action id `additionalComposites.action`); C bundle (`additionalComposites.action`, label NLS 3323, `getOverflowingComposites`, radio checked representation)
- Pieces:
  1. When the pinned icons do not fit the height of the rail, the extra ones move behind a "..." button at the bottom of the icon list.
  2. The "..." button is named Additional Views and opens a menu that lists the icons that did not fit as radio items.
  3. Choosing an item in the menu opens that view, and the item for the open view shows as checked.
  4. An item whose icon has a number badge shows the number in brackets after its name.
  5. Cursor's horizontal row has a chevron drop-down in place of the "..." button, with pin toggles and drag reorder. [cursor only]

#### AB-1.7 The open icon always stays on the rail

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The container that is open always has an icon on the rail, even if you hid its icon or it would not fit, and an unpinned icon goes away again once you switch to another view; Cursor's horizontal row shows the open unpinned container as one extra selected tab at the end of the row.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts; V\src\vs\workbench\browser\parts\paneCompositeBar.ts; C bundle (`getOverflowingComposites` and the active item rule, identical to stock)
- Pieces:
  1. The container that is open always has an icon on the rail, even if its icon was hidden or would not fit.
  2. An unpinned icon that is on the rail only because its view is open goes away again once the person switches to another view.
  3. Cursor's horizontal row shows the open unpinned container as one extra selected tab at the end of the row. [cursor only]

#### AB-1.8 Icons that appear only when they have content

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A container whose views are all inactive has no icon (Testing, Remote Explorer and References are examples), and its icon appears once one of its views turns active, such as the References icon after a references command fills the list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts (`shouldBeHidden`); V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts; V\extensions\references-view\package.json (activitybar container `references-view`, view `references-view.tree` shown when `reference-list.isActive`); C bundle (`shouldBeHidden` with `hideIfEmpty`, Testing container registered with `hideIfEmpty:!0`)
- Pieces:
  1. A container marked to hide when empty shows no icon on the rail while all its views are inactive, as Testing, Remote Explorer, and References do.
  2. The icon appears as soon as one of the container's views becomes active, for example the References icon after a references command fills its list.
  3. The icon goes away again when all of the container's views become inactive.

#### AB-1.9 Open a view with its own shortcut or Show command

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pressing a container's shortcut, or running its Show command from the Command Palette or the View menu, opens that view in the side bar, and pressing it again while the side bar already has focus sends focus back to the editor.
- Commands: workbench.view.explorer; workbench.view.search; workbench.view.scm; workbench.view.debug; workbench.view.extensions; workbench.view.extension.test
- Menus: MenubarViewMenu (group 3_sidebar); CommandPalette (f1)
- Keys: Ctrl+Shift+E (workbench.view.explorer); Ctrl+Shift+G (workbench.view.scm); Ctrl+Shift+D (workbench.view.debug); Ctrl+Shift+X (workbench.view.extensions); workbench.view.search is bound to Ctrl+Shift+F only behind a never-matching condition, the real Ctrl+Shift+F belongs to Find in Files; workbench.view.extension.test has no key
- Settings: none found
- Source: V\src\vs\workbench\services\views\browser\viewsService.ts; V\src\vs\workbench\contrib\files\browser\explorerViewlet.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts; V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; V\src\vs\workbench\contrib\extensions\browser\extensions.contribution.ts; V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts; C bundle (the same container descriptors with the same keys, Show and Toggle labels NLS 13435 and 13436)
- Pieces:
  1. Pressing a container's shortcut opens that view in the primary side bar and puts keyboard focus in it.
  2. Ctrl+Shift+E (`workbench.view.explorer`) opens Explorer, Ctrl+Shift+G (`workbench.view.scm`) opens Source Control, Ctrl+Shift+D (`workbench.view.debug`) opens Run and Debug, and Ctrl+Shift+X (`workbench.view.extensions`) opens Extensions.
  3. `workbench.view.search` opens Search from the Command Palette but has no working key of its own, because Ctrl+Shift+F runs Find in Files instead.
  4. `workbench.view.extension.test` opens Testing and has no default key.
  5. Each Show command is listed in the Command Palette in the View category, and each container also has an entry in the View menu.
  6. Pressing the shortcut again while the side bar already has focus sends focus back to the editor.

#### AB-1.10 Step to the previous or next side bar view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Previous Primary Side Bar View and Next Primary Side Bar View move the open view one step back or forward through the icons currently on the rail (pinned icons plus the open one) and wrap around at the ends; in Cursor's horizontal row both commands run but cycle through every container, pinned or not.
- Commands: workbench.action.previousSideBarView; workbench.action.nextSideBarView
- Menus: CommandPalette (f1)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; V\src\vs\workbench\browser\parts\compositeBarActions.ts (`SwitchCompositeViewAction`); C bundle (both command ids, titles NLS 3302 and 3303, `getVisiblePaneCompositeIds` returns every container in the horizontal part)
- Pieces:
  1. Previous Primary Side Bar View (`workbench.action.previousSideBarView`) moves the open view one step back through the icons on the rail.
  2. Next Primary Side Bar View (`workbench.action.nextSideBarView`) moves the open view one step forward through the icons on the rail.
  3. Both commands wrap around, so stepping back from the first icon lands on the last and stepping forward from the last lands on the first.
  4. The steps count only the icons currently on the rail, which are the pinned icons plus the open one.
  5. Both commands are listed in the Command Palette and have no default key.
  6. In Cursor's horizontal row both commands run but cycle through every container, pinned or not. [cursor only]

#### AB-1.11 Focus the rail

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Focus Activity Bar puts keyboard focus on the rail, first bringing the bar back if its location was set to hidden; in Cursor the command is enabled only when orientation is `vertical`, so it is disabled in the default horizontal row, and it shows a "quit and reopen" info dialog if the rail does not exist.
- Commands: workbench.action.focusActivityBar
- Menus: CommandPalette (f1)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts; V\src\vs\workbench\browser\layout.ts; C bundle (command id with precondition `config.workbench.activityBar.orientation == vertical`, label NLS 3304, dialog text NLS 3301)
- Pieces:
  1. Running Focus Activity Bar (`workbench.action.focusActivityBar`) puts keyboard focus on the rail.
  2. If the activity bar location was set to hidden, the command brings the bar back first and then focuses it.
  3. The command is listed in the Command Palette and has no default key.
  4. In Cursor the command is enabled only when `workbench.activityBar.orientation` is `vertical`, so it is disabled in the default horizontal row. [cursor only]
  5. In Cursor, running the command when the rail does not exist shows an info dialog telling the person to quit and reopen. [cursor only]

#### AB-1.12 Move between rail icons with the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With focus on the rail, Up and Down (or Left and Right) move between icons without looping, Home and End jump to the first and last icon, and Enter or Space opens the focused icon; Down past the last icon moves into the Accounts and Manage icons, and Up before the first moves to the compact Menu button when it is shown; Cursor's horizontal row has no key handling in its tab code.
- Commands: none found
- Menus: none found
- Keys: Up and Down arrows (rail focused); Left and Right arrows (rail focused); Home; End; Enter; Space - hard-coded in the action bar, not rebindable
- Settings: none found
- Source: V\src\vs\base\browser\ui\actionbar\actionbar.ts; V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; V\src\vs\workbench\browser\parts\compositeBar.ts; C bundle (`registerKeyboardNavigationListeners` with the same arrow handling, `triggerKeys:{keyDown:!0}`)
- Pieces:
  1. With focus on the rail, Down Arrow or Right Arrow moves to the next icon and Up Arrow or Left Arrow moves to the previous icon, stopping at the ends instead of looping.
  2. Home jumps to the first icon and End jumps to the last icon.
  3. Enter or Space opens the focused icon.
  4. Pressing Down Arrow past the last icon moves focus into the Accounts and Manage icons.
  5. Pressing Up Arrow before the first icon moves focus to the compact Menu button when it is shown.
  6. These keys are fixed in the action bar and cannot be rebound.


## AB-2 Customizing the rail

(both). Choosing which containers show, their order and where they live, through menus and drag. Examples: Hide or Keep an icon; drag to reorder; drag a container onto the rail; Move To and Reset Location.

11 features.

#### AB-2.1 Hide or keep an icon from its menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click an icon and choose Hide 'Name' to take it off the rail or Keep 'Name' to put it back, where Hide is greyed out for the last icon left, and hiding the open view opens the default view (Explorer) or the next icon instead; Cursor's horizontal tab menu has the same Hide and Keep entries.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\workbench\browser\parts\compositeBar.ts; C bundle (action id `show.toggleCompositePinned`, labels NLS 3325 and 3326, enabled only while more than one icon is pinned)
- Pieces:
  1. Right-clicking an icon and choosing Hide 'Name' takes that icon off the rail.
  2. An icon that is not pinned shows Keep 'Name' in its menu instead of Hide, and choosing it puts the icon back on the rail to stay.
  3. Hide is greyed out when the icon is the last one left on the rail.
  4. Hiding the icon of the open view opens the default view (Explorer) or the next icon's view instead.
  5. Cursor's horizontal tab menu has the same Hide and Keep entries. [cursor only]

#### AB-2.2 Show or hide icons from the rail's background menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-clicking empty space on the rail opens a checklist of every available container where a tick means it is on the rail, ticking one adds it and opens it, unticking removes it, and the last ticked item cannot be unticked; in Cursor's horizontal row the checklist sits at the bottom of the tab menu, and empty row space offers only Move Primary Side Bar Right or Left.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts; C bundle (`getContextMenuActions` building the checked list with pin and unpin, identical to stock)
- Pieces:
  1. Right-clicking empty space on the rail opens a checklist of every available container, with a tick on each one that is on the rail.
  2. Ticking an item adds its icon to the rail and opens it.
  3. Unticking an item removes its icon from the rail.
  4. The last ticked item cannot be unticked.
  5. In Cursor's horizontal row the checklist sits at the bottom of the tab menu, and right-clicking empty row space offers only Move Primary Side Bar Right or Left. [cursor only]

#### AB-2.3 Reorder icons by dragging

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Dragging an icon up or down the rail shows a drop line above or below the target depending on which half you hover, dropping on empty rail space puts it first or last, and the new order is remembered; Cursor's horizontal tabs are not draggable and only the rows of its chevron list reorder.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\workbench\browser\parts\compositeBar.ts; C bundle (icon registered as a draggable with `type:"composite"`, `dragged-over-head` and `dragged-over-tail` classes, same drop handler)
- Pieces:
  1. Dragging an icon up or down the rail shows a drop line above or below the icon under the pointer, depending on which half of it the pointer is over.
  2. Dropping an icon on empty rail space puts it first or last.
  3. The new order is remembered the next time the window opens.
  4. Cursor's horizontal tabs are not draggable, and only the rows of its chevron list reorder. [cursor only]

#### AB-2.4 Hold a dragged item over an icon to open it

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While dragging a container or view, resting over a rail icon for 0.8 seconds opens that icon's view so you can see where the drop will land; Cursor's horizontal row has no such handler in its tab code.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\base\browser\dnd.ts (`DelayedDragHandler`, 800 ms); C bundle (the same drag-over timer class with `},800)`)
- Pieces:
  1. While a container or view is being dragged, resting the pointer over a rail icon for 0.8 seconds opens that icon's view, so the person can see where the drop will land.
  2. Leaving the icon, dropping, or ending the drag before 0.8 seconds cancels it, and the view does not open.
  3. An icon whose view is already open is left alone.
  4. In Cursor's horizontal row, resting a dragged item over a tab does not open that tab's view. [cursor only]

#### AB-2.5 Drag a container from another bar onto the rail

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Dragging a view container's tab from the Panel or Secondary Side Bar onto the rail and dropping it at the wanted spot moves the whole container into the primary side bar and opens it; no drop target for dragged containers was found in Cursor's horizontal row code (not run).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (`CompositeDragAndDrop.drop`, `CompositeBarDndCallbacks`); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (`moveViewContainerToLocation`); C bundle (`moveViewContainerToLocation(...,"dnd")` in the same drop handler)
- Pieces:
  1. Dropping a view container's tab from the Panel or the Secondary Side Bar onto the rail moves the whole container into the primary side bar and opens it.
  2. The container's icon lands at the spot on the rail where it was dropped.
  3. In Cursor's horizontal row, the rail drop for dragged containers is not available. [cursor only]

#### AB-2.6 Drag a single view onto the rail to give it its own icon

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Dragging one view by its header onto the rail creates a new container that holds only that view, places its icon where you dropped it and opens it, and only views that allow moving can be dragged; no drop target for views was found in Cursor's horizontal row code (not run).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts; V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (view header is draggable); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (`moveViewToLocation`); C bundle (`moveViewToLocation(s,this.targetContainerLocation,"dnd")` with the `canMoveView` check)
- Pieces:
  1. Dragging one view by its header onto the rail creates a new container that holds only that view.
  2. The new container's icon is placed where it was dropped, and the container opens.
  3. Only views that allow moving can be dragged this way.
  4. In Cursor's horizontal row, the rail drop for dragged views is not available. [cursor only]

#### AB-2.7 Move a container with Move To

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In an icon's menu, Move To lists Primary Side Bar, Secondary Side Bar and Panel (leaving out where it already is) and sends the whole container to the end of that bar (or back to its default spot when that bar is its home) and opens it; Cursor's horizontal tab menu has no Move To entry.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts; C bundle (`moveToMenu` submenu over locations 0, 2, 1, labels NLS 3920 and 3923 to 3925, `createMoveAction`)
- Pieces:
  1. An icon's menu has a Move To submenu that lists Primary Side Bar, Secondary Side Bar, and Panel, leaving out the place the container already is.
  2. Choosing a place sends the whole container to the end of that bar and opens it.
  3. When the chosen bar is the container's home, the container goes back to its default spot instead of the end.
  4. Cursor's horizontal tab menu has no Move To entry. [cursor only]

#### AB-2.8 Reset Location

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Reset Location shows in an icon's menu only when its container, or the single view it holds, is away from its default home, and it sends it back and opens it; Cursor's horizontal tab menu reaches the same action through the container title menu.
- Commands: `<container id>.resetViewContainerLocation` (one per container, title menu variant)
- Menus: ViewContainerTitleContext (group 1_viewActions)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts; V\src\vs\workbench\services\views\browser\viewDescriptorService.ts; C bundle (`resetLocationAction` in the rail menu code, `${e.id}.resetViewContainerLocation` with `Pe.ViewContainerTitleContext`, labels NLS 3921, 3922 and 13433)
- Pieces:
  1. Reset Location shows in an icon's menu only when its container, or the single view it holds, is away from its default home.
  2. Choosing Reset Location sends the container back to its default location and opens it.
  3. For a container that holds a single moved view, Reset Location sends that view back to its default container.
  4. Each container also has a Reset Location command (`<container id>.resetViewContainerLocation`) in its title menu.
  5. Cursor's horizontal tab menu reaches Reset Location through the container title menu. [cursor only]

#### AB-2.9 Configure Keybinding for a container

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The first entry of an icon's menu, shown only when the container has an open command, opens the Keyboard Shortcuts editor filtered to that command so you can assign or change its key; Cursor's horizontal tab menu has the same entry.
- Commands: workbench.action.openGlobalKeybindings
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\platform\actions\common\menuService.ts (`createConfigureKeybindingAction`); C bundle (helper `configureKeybinding/` with label NLS 1785 calling `workbench.action.openGlobalKeybindings` with `@command:`)
- Pieces:
  1. The first entry of an icon's menu is Configure Keybinding, and it shows only when the container has an open command.
  2. Choosing it opens the Keyboard Shortcuts editor filtered to that container's command (`workbench.action.openGlobalKeybindings` with an `@command:` filter), so the person can assign or change its key.
  3. Cursor's horizontal tab menu has the same Configure Keybinding entry. [cursor only]

#### AB-2.10 New containers appear on the rail by themselves

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A view container that has not been seen before (for example one added by an extension, or a new one made by dragging a view out) is pinned automatically, so its icon appears on the rail without any setup.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts (`onDidRegisterViewContainers`); C bundle (`onDidRegisterViewContainers` pins any container missing from the cache)
- Pieces:
  1. A view container that is new to the rail, such as one added by an extension, is pinned automatically so its icon appears without any setup.
  2. A container the window already knows keeps its saved pinned or hidden state instead of being pinned again.

#### AB-2.11 The rail is restored from a saved copy at startup

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Which icons are pinned, their order, and their names and icons are saved per profile and drawn at once when the window opens, before extensions finish loading, with placeholder actions standing in until the real containers register.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts (cached view containers, `PlaceHolderViewContainerActivityAction`); V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts (storage key names); C bundle (storage keys `workbench.activity.pinnedViewlets2`, `workbench.activity.placeholderViewlets`, `workbench.activity.viewletsWorkspaceState`, placeholder action classes)
- Pieces:
  1. Which icons are pinned, their order, and their names and icons are saved per profile.
  2. When the window opens, the rail is drawn at once from the saved copy, before extensions finish loading.
  3. Placeholder icons stand in for containers that have not registered yet, and the real containers replace them as they register.


## AB-3 Badges on rail icons

(both). The count, progress or icon marker on a rail icon and the per-icon switch to hide it. Examples: number badge with `1K+` shortening; progress badge; icon badge; Hide Badge.

8 features.

#### AB-3.1 Number badge with 1K+ shortening

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A small pill with a count sits at the lower right of an icon (Source Control pending changes, unsaved files on Explorer, active debug sessions on Run and Debug, test counts on Testing, extension updates or restarts on Extensions), it adds up every count reported for that container, hides at zero, and shows 1000 as 1K and 1500 as 1K+; Cursor's horizontal row never shows it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\workbench\browser\parts\activitybar\media\activityaction.css; V\src\vs\workbench\services\activity\common\activity.ts; V\src\vs\workbench\contrib\scm\browser\activity.ts; V\src\vs\workbench\contrib\files\common\dirtyFilesIndicator.ts; V\src\vs\workbench\contrib\debug\browser\debugService.ts; V\src\vs\workbench\contrib\extensions\browser\extensionsViewlet.ts; C bundle (badge code with `${a}K+`, count strings NLS 9199, 7388, 7389, 5835, 5836, 6743 to 6746, 11164 to 11166)
- Pieces:
  1. A small count pill appears at the lower right of a rail icon when its container reports a count, for example pending changes on Source Control, unsaved files on Explorer, active sessions on Run and Debug, test counts on Testing, and extension updates or restarts on Extensions.
  2. When more than one view in the same container reports a count, the pill shows the sum of all of them.
  3. The pill is hidden when the total is zero.
  4. A total above 999 is shortened to thousands: an exact multiple such as 1000 shows `1K`, and anything between shows the whole thousands with a plus, so 1500 shows `1K+`.

#### AB-3.2 Progress badge

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While a long task tied to a container runs (file work on Explorer, repository work on Source Control, installs on Extensions), the icon shows a small progress glyph instead of a count; it appears after 300 ms, stays at least 300 ms, has no hover text of its own, and wins over icon and number badges; Cursor's horizontal row never shows it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts; V\src\vs\workbench\services\progress\browser\media\progressService.css; V\src\vs\workbench\browser\parts\compositeBarActions.ts; C bundle (`showOnActivityBar` code with the 300 ms delay and minimum, `progress-badge` CSS rule)
- Pieces:
  1. While a long task tied to a container is running, such as file work on Explorer, repository work on Source Control or an install on Extensions, the icon shows a small progress glyph in the pill position instead of a count.
  2. The glyph appears only after the task has run for 300 ms and, once shown, stays for at least 300 ms, so short tasks cause no flicker.
  3. While the glyph is showing it replaces any icon badge or count pill on the same icon, and the pill comes back when the task ends.
  4. The progress glyph that shows for a running container task adds no text of its own to the icon's hover or screen reader label.

#### AB-3.3 Icon badge

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (Extensions warning and Testing watch glyphs exist, the Tests are running spinner is missing)
- What: An icon glyph can replace the count in the pill, such as a warning triangle on Extensions when there is an extensions warning, a watch glyph on Testing when tests re-run on change, or a spinner while tests run; it beats the number badge, and warning and error glyphs use their own colors; Cursor lacks the Tests are running spinner, and its horizontal row never shows any badge.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\workbench\services\activity\common\activity.ts (`IconBadge`, `WarningBadge`, `ErrorBadge`); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts; V\src\vs\workbench\contrib\extensions\browser\extensionsViewlet.ts; C bundle (`icon-badge` and `icon-overlay` classes, Testing watch string NLS 11163, key `testingRunningBadge` absent from `nls.keys.json`)
- Pieces:
  1. An icon glyph can take the place of the count in the pill, and while one is present it is drawn instead of the count on that icon.
  2. Extensions shows a warning triangle in the pill when there is an extensions warning.
  3. Testing shows a watch glyph in the pill when tests are set to re-run on change.
  4. Testing shows a spinner in the pill while tests are running, with the description "Tests are running". [stock only]

#### AB-3.4 Badge description in the hover and screen reader label

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The badge's own description is added after the icon name in the hover and the screen reader label, as in Source Control (Ctrl+Shift+G) - 3 pending changes, and by code reading this text stays even after Hide Badge has hidden the pill; Cursor's horizontal row has no badge text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts (`computeTitle`, `updateTitle`); C bundle (`computeTitle` reads `getVisibleBadges` and adds each description)
- Pieces:
  1. Hovering a rail icon that has a badge shows the icon name, then its key in parentheses when it has one, then a dash and the badge's description, as in `Source Control (Ctrl+Shift+G) - 3 pending changes`.
  2. A screen reader reads that same text as the icon's label.
  3. When several badges of the shown kind are on one icon, each badge's description is added after the name in turn.
  4. By code reading, the badge description stays in the hover and label even after Hide Badge has hidden the pill.

#### AB-3.5 Hide or show a badge per icon

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: An icon's right-click menu has Hide Badge, which removes every kind of badge from that icon, and the entry then reads Show Badge to bring it back; the choice is saved per profile only while badges are off, and Cursor's horizontal tab menu has no such entry.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts (`ToggleCompositeBadgeAction`, `showContextMenu`); V\src\vs\workbench\browser\parts\compositeBar.ts (`areBadgesEnabled`, `toggleBadgeEnablement`); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (storage key `views.customizations`, `viewContainerBadgeEnablementStates`); C bundle (action id `show.toggleCompositeBadge`, labels NLS 3327 and 3328, `viewContainerBadgeEnablementStates`)
- Pieces:
  1. Right-clicking a rail icon shows a Hide Badge entry, and picking it removes every kind of badge (count, progress and icon glyph) from that icon.
  2. After a badge is hidden, the same entry reads Show Badge, and picking it brings the badge back.
  3. Hiding a badge affects only the icon you right-clicked, and other icons keep their badges.
  4. The choice is saved per profile, so a hidden badge stays hidden after a restart.

#### AB-3.6 A view's badge follows the view to its new container

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Badges that a view reports, such as the Source Control change count, are drawn on whichever container currently holds that view, so if you drag the view into another container or give it its own icon, the badge moves with it; Cursor's horizontal row never shows the result.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\activity\browser\activityService.ts (`ViewContainerActivityByView`); V\src\vs\workbench\contrib\scm\browser\activity.ts; C bundle (class tracking `viewId` with `onDidChangeContainer` and `onDidChangeLocation`, `showViewActivity` calls)
- Pieces:
  1. A badge that a view reports, such as the Source Control change count, is drawn on the icon of whichever container currently holds that view.
  2. Dragging the view into a different container moves its badge to that container's icon and takes it off the old one.
  3. Giving the view its own icon, so it becomes a container of its own, moves its badge to the new icon.

#### AB-3.7 Badge colors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The count pill is drawn with the activityBarBadge.background and activityBarBadge.foreground theme colors, warning and error icon badges use the activityWarningBadge.* and activityErrorBadge.* colors, and themes with a contrast border get a thin border around the pill.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Source: V\src\vs\workbench\common\theme.ts; V\src\vs\platform\theme\common\colors\miscColors.ts; V\src\vs\workbench\browser\parts\compositeBarActions.ts (`updateStyles`); C bundle (color ids `activityBarBadge.background`, `activityWarningBadge.background`, `activityErrorBadge.background`, NLS 4491 and 4492)
- Pieces:
  1. The count pill is filled with the `activityBarBadge.background` color and its number is drawn in the `activityBarBadge.foreground` color.
  2. Warning icon badges use the `activityWarningBadge.*` colors, and error icon badges use the `activityErrorBadge.*` colors.
  3. Themes that define a contrast border draw a thin border around the pill.
  4. Setting `workbench.colorCustomizations` can override these badge colors, and the pill is redrawn with the new colors when they change.

#### AB-3.8 Source Control and Testing count badge settings

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: `scm.countBadge` sets the Source Control count to the sum of all repositories, the focused repository only, or off, and `testing.countBadge` sets the Testing count to failed (default), passed, skipped tests, or off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: scm.countBadge; testing.countBadge; git.countBadge
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts; V\src\vs\workbench\contrib\scm\browser\activity.ts; V\src\vs\workbench\contrib\testing\common\configuration.ts; V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts; C bundle (`scm.countBadge` definition with values all, focused, off; `testing.countBadge` with NLS 11256 to 11259)
- Pieces:
  1. Setting `scm.countBadge` chooses what the Source Control icon counts, with values `all` (the default, the sum of every visible repository), `focused` (only the focused repository) and `off` (no count).
  2. With `scm.countBadge` set to `focused`, switching to another repository in Source Control changes the pill to that repository's count.
  3. Setting `testing.countBadge` chooses which tests the Testing icon counts, with values `failed` (the default), `passed`, `skipped` and `off`.
  4. The Testing icon's badge description follows the counted kind, reading "{n} failed tests", "{n} passed tests" or "{n} skipped tests".


## AB-4 Accounts and Manage entries

(both). The two global icons under the rail, their menus, hiding Accounts, and their title bar placement when the rail is on top or bottom. Cursor draws them only in the vertical rail, never in the title bar, and drops the Settings Sync entries. Examples: the Accounts menu with per-account Sign Out; the Manage menu with Command Palette, Settings, Keyboard Shortcuts, Themes; Hide Accounts; the profile badge on the gear.

25 features.

#### AB-4.1 Open the Accounts menu and see signed-in accounts

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - drawn only in the vertical rail, and it runs an older copy of the menu code: a disabled "You are not signed in to any accounts" row when providers exist but no account does, and a disabled "No accounts requested yet..." placeholder (`noAuthenticationProviders`) when no provider exists. V has neither row.
- What: Click the Accounts icon (a person glyph under the rail, hover text "Accounts") to open a menu with one submenu per signed-in account, named `{account} ({provider})`, then a separator and the rows that other features add to the Accounts menu. Click, Enter or Space opens it, beside the rail (to the right when the side bar is on the left). A disabled "Loading..." row shows until accounts resolve, and a provider that fails shows a disabled "{0} is currently unavailable" and is retried in the background. The action id `workbench.actions.accounts` is a view item id, not a registered command.
- Commands: workbench.actions.accounts
- Menus: AccountsContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (lines 47-151, 258-500); V\src\vs\workbench\common\activity.ts (line 7); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3838-3847); C bundle: vs/workbench/contrib/authentication/browser/authentication.contribution (NLS 4887-4888)
- Pieces:
  1. Clicking the Accounts icon, a person glyph under the rail with the hover text "Accounts", opens the Accounts menu beside the rail, to the right when the side bar is on the left.
  2. Pressing Enter or Space on the focused Accounts icon opens the same menu.
  3. The menu lists one submenu per signed-in account, named `{account} ({provider})`.
  4. A separator follows the account submenus, and below it come the rows that other features add to the Accounts menu.
  5. A disabled "Loading..." row shows until the accounts have resolved.
  6. A provider that fails shows a disabled "{0} is currently unavailable" row and is retried in the background.
  7. In Cursor the menu shows a disabled "You are not signed in to any accounts" row when providers exist but no account is signed in, and a disabled "No accounts requested yet..." row when there is no provider. [cursor only]

#### AB-4.2 Sign out of an account

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the Sign Out row, `_signOutOfAccount` and the confirm strings are present (vertical rail only). The Command Palette route `workbench.action.manageAccounts` is missing (0 hits for the id and its strings).
- What: Open an account's submenu in the Accounts menu and pick Sign Out. A confirm box asks "Sign out of '{0}'?", or if extensions have used the account it lists them first ("The account '{0}' has been used by: ... Sign out from these extensions?"), with a Sign Out button. On confirm it removes the account's sessions and its saved extension usage and allowed-extension records. The Sign Out row is left out for an account the host says cannot sign out (an embedder session). The same action is reachable from the Command Palette entry "Accounts: Manage Accounts", which picks an account and then offers Manage Trusted Extensions, Manage Trusted MCP Servers and Sign Out. The submenu rows are built in code, not registered on a menu.
- Commands: _signOutOfAccount; workbench.action.manageAccounts; authentication.command.anysphere.signOut
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (389-420, 470-482); V\src\vs\workbench\contrib\authentication\browser\actions\signOutOfAccountAction.ts; V\src\vs\workbench\contrib\authentication\browser\actions\manageAccountsAction.ts; V\src\vs\workbench\contrib\authentication\browser\authentication.contribution.ts (92-98); C bundle: vs/workbench/contrib/authentication/browser/actions/signOutOfAccountAction (NLS 4880-4883); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3846)
- Pieces:
  1. Picking Sign Out in an account's submenu opens a confirm box reading "Sign out of '{0}'?" with a Sign Out button.
  2. When extensions have used the account, the confirm box lists them first and reads "The account '{0}' has been used by: ... Sign out from these extensions?".
  3. Confirming removes the account's sessions, its saved extension usage and its allowed-extension records.
  4. The Sign Out row is left out for an account that the host reports as unable to sign out.
  5. Running "Accounts: Manage Accounts" from the Command Palette asks you to pick an account, then lists actions for it including Manage Trusted Extensions and Sign Out. [stock only]
  6. With no signed-in account, "Accounts: Manage Accounts" shows a single row reading "There are no active accounts.". [stock only]

#### AB-4.3 Choose which extensions can use an account

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - an older copy of the action with the same command id, title and "Choose which extensions can access this account" text. The `workbench.action.manageAccounts` route is missing.
- What: In an account's submenu pick Manage Trusted Extensions to open a checklist of extensions that have used or been allowed on that account, newest use first, each with "Last used this account {time}". Untick to revoke or tick to allow, then accept. Extensions trusted by Microsoft sit under a separator and are locked on. Each row has buttons to view the extension and to open its account preferences. This is account access control in the authentication code, not the Extensions view (see Unclear). Also reachable from the Command Palette, which first asks which account.
- Commands: _manageTrustedExtensionsForAccount; workbench.action.manageAccounts
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (389-400); V\src\vs\workbench\contrib\authentication\browser\actions\manageTrustedExtensionsForAccountAction.ts (21-220); C bundle: vs/workbench/contrib/authentication/browser/actions/manageTrustedExtensionsForAccountAction (NLS 4876-4879); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3845)
- Pieces:
  1. Picking Manage Trusted Extensions in an account's submenu opens a checklist titled "Manage Trusted Extensions" with the placeholder "Choose which extensions can access this account".
  2. The checklist holds the extensions that have used or been allowed on that account, newest use first, each with "Last used this account {time}" or "Has not used this account" when it never has.
  3. Unticking an extension revokes its access and ticking it allows access, and the change is applied when you accept.
  4. Extensions trusted by Microsoft sit under a "Trusted by Microsoft" separator, are locked on, and show a tooltip saying they always have access to the account.
  5. Each row has two buttons, one to view the extension's details and one to manage account preferences for that extension.
  6. When no extension has used the account, an info box says "This account has not been used by any extensions.".
  7. Choosing Manage Trusted Extensions after picking an account in "Accounts: Manage Accounts" opens the same checklist for that account. [stock only]

#### AB-4.4 Choose which account an extension uses

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the command and its strings are present, but it has no Accounts menu row and is hidden from the Command Palette (`f1:!1`, no `menu`). Only the Extensions view context menu calls it, so AB-4 cannot reach it in Cursor.
- What: Pick "Manage Extension Account Preferences..." at the bottom of the Accounts menu, choose an extension that has used an account (then a provider if it used several), and pick the account it should use for this workspace from the provider's list. Providers that allow several accounts also offer "Use a new account...". If the extension has used no account it says "This extension has not used any accounts yet."
- Commands: _manageAccountPreferencesForExtension
- Menus: AccountsContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\authentication\browser\actions\manageAccountPreferencesForExtensionAction.ts (19-135); V\src\vs\workbench\contrib\extensions\browser\extensions.contribution.ts (1784); C bundle: vs/workbench/contrib/authentication/browser/actions/manageAccountPreferencesForExtensionAction (NLS 4859-4867)
- Pieces:
  1. Picking "Manage Extension Account Preferences..." at the bottom of the Accounts menu opens a picker of the extensions that have used an account. [stock only]
  2. When the chosen extension has used more than one provider, a second picker asks which provider to manage.
  3. The provider's account list is titled "'{extension}' Account Preferences For This Workspace", marks the account in use as "Current account", and picking another account sets it for this workspace.
  4. For a provider that allows several accounts, the list also offers "Use a new account...".
  5. When the extension has used no account, an info box says "This extension has not used any accounts yet.".
  6. In Cursor the Accounts menu has no row for this picker and the Command Palette does not list it. [cursor only]

#### AB-4.5 Answer an extension's sign-in or access request

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - extension sign-in and access rows and the number badge are kept. Stock's MCP variants, Settings Sync badge, Cloud Changes badge and Marketplace sign-in badge are not in the bundle (only one `showAccountsActivity` call site found).
- What: When an extension asks to use an account, the Accounts menu gains a row "Sign in with {provider} to use {extension} (1)" or "Grant access to {provider} for {extension}... (1)", and the Accounts icon shows a number badge (hover "Sign in requested") counting open requests. A sign-in row starts the provider's sign-in. An access row opens either an Allow or Deny box ("The extension '{0}' wants to access the {1} account '{2}'.") or, when several accounts exist, an account picker with "Sign in to another account". The row and the badge clear once answered. Stock also feeds the Accounts badge from MCP server requests, Settings Sync sign-in, Cloud Changes and Marketplace sign-in (see AB-4.7 and AB-4.9).
- Commands: {providerId}:{extensionId}:signIn{n} (dynamic); {providerId}{extensionId}Access (dynamic)
- Menus: AccountsContext (groups 2_signInRequests and 3_accessRequests)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\authentication\browser\authenticationExtensionsService.ts (131-150, 253-283, 393-505); V\src\vs\workbench\services\authentication\browser\authenticationMcpService.ts (206, 453, 513); C bundle: vs/workbench/services/authentication/browser/authenticationExtensionsService (NLS 12298-12306)
- Pieces:
  1. When an extension asks to sign in, the Accounts menu gains a row reading "Sign in with {provider} to use {extension} (1)", and picking it starts the provider's sign-in.
  2. When an extension asks for access to a signed-in account, the Accounts menu gains a row reading "Grant access to {provider} for {extension}... (1)".
  3. Picking an access row opens an Allow or Deny box reading "The extension '{0}' wants to access the {1} account '{2}'.".
  4. When several accounts exist, the access row opens an account picker instead, which includes "Sign in to another account".
  5. The Accounts icon shows a number badge with the hover text "Sign in requested" that counts the open requests.
  6. Once a request is answered, its row disappears from the menu and its share of the badge clears.

#### AB-4.6 Turn on or manage Remote Tunnel Access

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - the three registrations and ids are in the bundle. Whether the feature activates depends on product config that was not checked (see Unclear).
- What: The Accounts menu shows "Turn on Remote Tunnel Access..." when the tunnel is off. While connecting it shows "Remote Tunnel Access is Connecting", and once on it shows "Remote Tunnel Access is On". The last two open a list: Learn More, Copy Browser URL, Show Log, Change Tunnel Name, and Turn Off. Desktop build only.
- Commands: workbench.remoteTunnel.actions.turnOn; workbench.remoteTunnel.actions.manage; workbench.remoteTunnel.actions.connecting
- Menus: AccountsContext (group 2_remoteTunnel)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remoteTunnel\electron-browser\remoteTunnel.contribution.ts (489-630, 756-790); C bundle: vs/workbench/contrib/remoteTunnel/electron-sandbox/remoteTunnel.contribution (NLS 9164-9165)
- Pieces:
  1. When the tunnel is off, the Accounts menu shows "Turn on Remote Tunnel Access...", and picking it starts turning the tunnel on.
  2. While the tunnel is connecting, the menu shows "Remote Tunnel Access is Connecting".
  3. Once the tunnel is running, the menu shows "Remote Tunnel Access is On".
  4. Picking the connecting or on row opens a list with Learn More, Copy Browser URL, Show Log, Change Tunnel Name and Turn Off.
  5. These rows are offered only in the desktop build.

#### AB-4.7 Turn on and manage Settings Sync

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - 0 hits for `workbench.userDataSync.actions.*`, `workbench.userData.actions.*` and the labels "Backup and Sync Settings", "Settings Sync is On" and "Sign in to Sync Settings". The three `settingsSync.*` setting ids are still registered in the bundle.
- What: Both menus offer "Backup and Sync Settings..." to start Settings Sync (Manage group 3_configuration, Accounts group 1_settings). While it starts a disabled "Turning on Settings Sync..." row shows. Once on, a ticked "Settings Sync is On" row opens a list of sync commands. If the sync account has signed out, Manage shows "Sign in to Sync Settings" and Accounts shows "Sign in to Sync Settings (1)" with a count badge on the Accounts icon. Cursor drops all of these.
- Commands: workbench.userDataSync.actions.turnOn; workbench.userData.actions.turningOn; workbench.userData.actions.signin; workbench.userDataSync.actions.manage
- Menus: AccountsContext; GlobalActivity
- Keys: none found
- Settings: settingsSync.ignoredSettings; settingsSync.ignoredExtensions; settingsSync.keybindingsPerPlatform
- Source: V\src\vs\workbench\contrib\userDataSync\browser\userDataSync.ts (765-875, 912-960, 467-483); V\src\vs\platform\userDataSync\common\userDataSync.ts (79-115)
- Pieces:
  1. Picking "Backup and Sync Settings..." in the Accounts menu or the Manage menu starts Settings Sync. [stock only]
  2. While Settings Sync is starting, both menus show a disabled "Turning on Settings Sync..." row. [stock only]
  3. Once Settings Sync is on, a ticked "Settings Sync is On" row opens a list with Configure..., Show Settings, Show Synced Data, Sync Now and Turn Off. [stock only]
  4. In that list, Sync Now reads "syncing" or "synced {time}" beside it, and Turn Off shows the account name and provider. [stock only]
  5. When the sync account has signed out, Manage shows "Sign in to Sync Settings" and Accounts shows "Sign in to Sync Settings (1)" with a count badge on the Accounts icon. [stock only]
  6. Setting `settingsSync.ignoredSettings` lists the settings that Settings Sync leaves out.
  7. Setting `settingsSync.ignoredExtensions` lists the extensions, as `{publisher}.{name}`, that Settings Sync leaves out.
  8. Setting `settingsSync.keybindingsPerPlatform`, on by default, chooses whether keybindings sync separately for each platform.

#### AB-4.8 Resolve Settings Sync conflicts from Manage

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - 0 hits for `workbench.userDataSync.actions.showConflicts` and the "Show Conflicts" label.
- What: When sync finds conflicts the Manage gear gets a number badge (hover "{0}: Conflicts Detected") and the Manage menu shows "Show Conflicts ({n})", which opens the conflict resolver. While Settings Sync is turning on, the gear shows a progress badge instead.
- Commands: workbench.userDataSync.actions.showConflicts
- Menus: GlobalActivity (group 3_configuration)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\userDataSync\browser\userDataSync.ts (68, 454-470, 877-910)
- Pieces:
  1. When Settings Sync finds conflicts, the Manage gear shows a number badge with the conflict count and the hover text "{0}: Conflicts Detected". [stock only]
  2. The Manage menu then shows "Show Conflicts ({n})", and picking it opens the conflict resolver. [stock only]
  3. While Settings Sync is turning on, the gear shows a progress badge instead. [stock only]

#### AB-4.9 Turn on or off Cloud Changes

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - 0 hits for both command ids and the "Turn on Cloud Changes" label.
- What: The Accounts menu offers "Turn on Cloud Changes..." (shown as "Turn on Cloud Changes... (1)" with a badge while a request is pending) to sign in for storing edit sessions in the cloud. Once signed in, "Turn off Cloud Changes..." signs out. Only these two menu rows are listed here, not the edit sessions feature itself (catalog decision 9).
- Commands: workbench.editSessions.actions.signIn; workbench.editSessions.actions.resetAuth
- Menus: AccountsContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\editSessions\browser\editSessionsStorageService.ts (462-515); V\src\vs\workbench\contrib\editSessions\browser\editSessions.contribution.ts (256)
- Pieces:
  1. While signed out, the Accounts menu shows "Turn on Cloud Changes...", and picking it signs in to store working changes in the cloud. [stock only]
  2. While a sign-in request is pending, the row reads "Turn on Cloud Changes... (1)" and the Accounts icon shows a badge. [stock only]
  3. Once signed in, the row becomes "Turn off Cloud Changes...", and picking it asks "Do you want to disable storing working changes in the cloud?" with a checkbox "Delete all stored data from the cloud." and signs out on confirm. [stock only]
  4. Both actions are also registered for the Command Palette, so "Turn on Cloud Changes..." and "Turn off Cloud Changes..." can be run from it. [stock only]

#### AB-4.10 Hide or show the Accounts icon

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - Hide Accounts, the "Accounts" toggle and the storage key are all kept.
- What: Right-click the Accounts icon and pick Hide Accounts, or untick "Accounts" in the rail's right-click menu, to remove the icon. Tick it again to bring it back. The Manage gear stays and cannot be hidden. The choice is stored per profile in the storage key `workbench.activity.showAccounts` (default shown), not as a setting, and the same key drives both the rail and the title bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (89-153, 258-260, 729-750); V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts (339-343); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3839, 3841, 3850-3852)
- Pieces:
  1. Right-clicking the Accounts icon shows a Hide Accounts entry, and picking it removes the Accounts icon from the rail.
  2. The rail's right-click menu has an "Accounts" check item, which is ticked while the icon shows, and unticking it hides the icon and ticking it again brings the icon back.
  3. The Manage gear stays in place when Accounts is hidden, and nothing hides the gear.
  4. The choice is stored per profile under the storage key `workbench.activity.showAccounts`, shown by default, and is not a setting.

#### AB-4.11 Show the account picture on the Accounts icon

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - 0 hits for `workbench.accounts.showAvatar`.
- What: newer-than-1.128. The Accounts icon can show the signed-in account's profile picture in place of the person glyph. `workbench.accounts.showAvatar` (default on) controls it, and the glyph returns if the image fails to load. Evidence is from N only.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.accounts.showAvatar
- Source: N\src\vs\workbench\browser\workbench.contribution.ts (669-673); N\src\vs\workbench\browser\parts\globalCompositeBar.ts (284, 350, 398-443); N\src\vs\workbench\services\authentication\common\authentication.ts (22)
- Pieces:
  1. The Accounts icon shows the signed-in account's profile picture in place of the person glyph, which VS Code adds after 1.128 (newer-than-1.128). [stock only]
  2. If the picture fails to load, the person glyph is shown again. [stock only]
  3. Setting `workbench.accounts.showAvatar`, on by default, chooses whether the picture is shown, and changing it updates the icon (newer-than-1.128). [stock only]

#### AB-4.12 Open the Manage menu from the gear

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - same gear and groups in the vertical rail, without the Settings Sync rows. No Cursor-only row was found. Cursor's own settings button is a title bar control (TB-5).
- What: Click the gear under the rail (hover text "Manage") to open a menu in four stacked groups: Command Palette; then Profile, Settings, Extensions, Keyboard Shortcuts, Snippets, Tasks, Themes; then Settings Sync rows; then update rows. The rows use their short titles. It opens the same way as the Accounts menu (click, Enter, Space). The gear is always drawn. The Extensions row is the extension system and is not listed as a feature. Stock web builds also add "Switch to Insiders Version..." (web only, not listed). The action id `workbench.actions.manage` is a view item id, not a registered command.
- Commands: workbench.actions.manage
- Menus: GlobalActivity
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (47-151, 583-660); V\src\vs\workbench\common\activity.ts (line 6); V\src\vs\workbench\contrib\update\browser\update.ts (371-376); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3840, 3848); C bundle: `Pe.GlobalActivity` registrations (18 matches, including the view item class)
- Pieces:
  1. Clicking the gear under the rail, with the hover text "Manage", opens the Manage menu, and pressing Enter or Space on the focused gear opens it too.
  2. The menu stacks its rows in groups: Command Palette first, then Profile, Settings, Extensions, Keyboard Shortcuts, Snippets, Tasks and Themes, then the update rows.
  3. A group of Settings Sync rows sits between the configuration rows and the update rows. [stock only]
  4. The rows use their short titles, so the menu reads "Keyboard Shortcuts" and "Snippets" rather than the full command titles.

#### AB-4.13 See the current profile on the gear

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - the `profile-text-overlay` badge code and the "Manage {0} (Profile)" string are in the bundle.
- What: When a non-default profile is active the gear's hover text becomes "Manage {name} (Profile)". If the profile has its own icon, that icon replaces the gear. If it has no icon, the gear gets a two-letter uppercase badge made from the profile name, hidden while a count or progress badge (updates, sync) is showing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (583-660); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3849)
- Pieces:
  1. While a non-default profile is active, hovering the gear shows "Manage {name} (Profile)".
  2. When that profile has its own icon, the icon replaces the gear glyph.
  3. When the profile has no icon, the gear gets a two-letter uppercase badge made from the profile name.
  4. The two-letter badge is hidden while a count or progress badge, such as an update or sync badge, is showing on the gear.

#### AB-4.14 Open the Command Palette from Manage

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - the `GlobalActivity` registration with group `1_command` and order 1 is in the bundle.
- What: The first row of the Manage menu, "Command Palette...", opens the Command Palette. Only this menu row is listed here. The palette itself is QA-1.
- Commands: workbench.action.showCommands
- Menus: GlobalActivity (group 1_command, order 1)
- Keys: F1; Ctrl+Shift+P
- Settings: none found
- Source: V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (90-98); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (273-289); C bundle: vs/workbench/contrib/quickaccess/browser/quickAccess.contribution (NLS 8908)
- Pieces:
  1. The first row of the Manage menu, "Command Palette...", opens the Command Palette.
  2. The row sits alone in the first group of the menu, above the configuration rows.

#### AB-4.15 Open Settings from Manage

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - the Manage row has no condition. Cursor's Preferences menu row for the same command carries a condition that Manage lacks (see the menu bar notes for TB-1), so Manage is where plain Settings still shows in the IDE window.
- What: The "Settings" row opens the Settings editor. Only this menu row is listed here. The editor is ED-8.
- Commands: workbench.action.openSettings
- Menus: GlobalActivity (group 2_configuration, order 2)
- Keys: Ctrl+,
- Settings: none found
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts (229-250); C bundle: `Pe.GlobalActivity` registration with `Ctrl+Comma` and order 2
- Pieces:
  1. The "Settings" row in the Manage menu opens the Settings editor.
  2. The row sits in the configuration group right after Profile and before Extensions.
  3. Pressing `Ctrl+,` opens the same Settings editor.

#### AB-4.16 Open Keyboard Shortcuts from Manage

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - the command and the `GlobalActivity` registration at order 4 are in the bundle.
- What: The "Keyboard Shortcuts" row (the command is titled "Open Keyboard Shortcuts", the Manage menu shows its short title) opens the Keyboard Shortcuts editor. Only this menu row is listed here. The editor is ED-8.
- Commands: workbench.action.openGlobalKeybindings
- Menus: GlobalActivity (group 2_configuration, order 4)
- Keys: Ctrl+K Ctrl+S
- Settings: none found
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts (880-915); C bundle: `Pe.GlobalActivity` registration with `openGlobalKeybindingSettings` and order 4
- Pieces:
  1. The "Keyboard Shortcuts" row in the Manage menu opens the Keyboard Shortcuts editor.
  2. The menu row shows the short title "Keyboard Shortcuts" while the command itself is titled "Open Keyboard Shortcuts".
  3. Pressing `Ctrl+K Ctrl+S` opens the same editor.

#### AB-4.17 Switch or manage profiles from Manage

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - both Manage registrations (submenu when `hasProfiles`, plain row when not), the manage command and `switchProfile` are in the bundle.
- What: With more than one profile, Manage shows "Profile ({current name})", a submenu that lists every profile with a tick on the active one (pick to switch) and a "Profiles" row that opens the Profiles editor. With only the default profile, Manage shows a plain "Profiles" row that opens the Profiles editor. The editor and the Command Palette routes (Switch Profile..., New Window with Profile...) are not cataloged here (see Unclear).
- Commands: workbench.profiles.actions.manageProfiles; workbench.profiles.actions.switchProfile; workbench.profiles.actions.profileEntry.{profile id} (dynamic)
- Menus: GlobalActivity (group 2_configuration, order 1); ProfilesMenu
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\userDataProfile\browser\userDataProfile.ts (73-75, 176-198, 221-243, 322-392); V\src\vs\workbench\services\userDataProfile\common\userDataProfile.ts (155); C bundle: vs/workbench/contrib/userDataProfile/browser/userDataProfile (NLS 11535)
- Pieces:
  1. When only the default profile exists, the Manage menu shows a plain "Profiles" row that opens the Profiles editor.
  2. When more than one profile exists, the Manage menu shows a "Profile ({current name})" submenu instead.
  3. The submenu lists every profile with a tick on the active one, and picking another profile switches to it.
  4. The submenu also has a "Profiles" row that opens the Profiles editor.

#### AB-4.18 Configure snippets from Manage

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - the command and the `GlobalActivity` registration at order 5 are in the bundle.
- What: The "Snippets" row (the command is titled "Configure Snippets", the Manage menu shows its short title) opens the snippet file picker. Only this menu row is listed here. The picker belongs with AC-4.
- Commands: workbench.action.openSnippets
- Menus: GlobalActivity (group 2_configuration, order 5)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\configureSnippets.ts (244-262); C bundle: `Pe.GlobalActivity` registration with order 5 beside `Pe.MenubarPreferencesMenu`
- Pieces:
  1. The "Snippets" row in the Manage menu opens the snippet file picker.
  2. The menu row shows the short title "Snippets" while the command itself is titled "Configure Snippets".

#### AB-4.19 Open user tasks from Manage

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - the command and the `GlobalActivity` registration at order 6 are in the bundle.
- What: The "Tasks" row opens the user-level `tasks.json` for editing. It shows only where task execution is supported.
- Commands: workbench.action.tasks.openUserTasks
- Menus: GlobalActivity (group 2_configuration, order 6)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (343-375); V\src\vs\workbench\contrib\tasks\common\taskService.ts (26)
- Pieces:
  1. The "Tasks" row in the Manage menu opens the user-level `tasks.json` for editing.
  2. The row shows only where task execution is supported.

#### AB-4.20 Pick a theme from Manage

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - same three rows. The submenu order values are 1, 3 and 4 in the bundle against 1, 2 and 3 in V, so the visible order is unchanged.
- What: The "Themes" row opens a submenu with Color Theme, File Icon Theme and Product Icon Theme. Each opens its own picker. Only the submenu is listed here. The three pickers have no owning group (see Unclear).
- Commands: workbench.action.selectTheme; workbench.action.selectIconTheme; workbench.action.selectProductIconTheme
- Menus: GlobalActivity (group 2_configuration, order 7); ThemesSubMenu
- Keys: Ctrl+K Ctrl+T (Color Theme only)
- Settings: workbench.colorTheme; workbench.iconTheme; workbench.productIconTheme
- Source: V\src\vs\workbench\contrib\themes\browser\themes.contribution.ts (405-418, 490, 527, 902-937); V\src\vs\workbench\services\themes\common\workbenchThemeService.ts (24-26); C bundle: vs/workbench/contrib/themes/browser/themes.contribution (NLS 11362-11366)
- Pieces:
  1. The "Themes" row in the Manage menu opens a submenu with Color Theme, File Icon Theme and Product Icon Theme.
  2. Color Theme opens the color theme picker, and pressing `Ctrl+K Ctrl+T` opens the same picker.
  3. File Icon Theme opens the file icon theme picker.
  4. Product Icon Theme opens the product icon theme picker.

#### AB-4.21 Check for, download and install an update from Manage

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the same rows and the gear badge call are in the bundle (vertical rail only). The "Cancelling Update..." row and the "no updates available" info box are not (0 hits).
- What: The bottom group of Manage changes with the updater state: "Check for Updates..." when idle, a disabled "Checking for Updates...", "Download Update (1)", a disabled "Downloading Update...", "Install Update... (1)", a disabled "Installing Update...", and "Restart to Update (1)". The gear shows a count badge of 1 when an update is available or ready ("New {0} update available.") and a progress badge while checking, downloading or installing. A check that finds nothing opens an info box, "There are currently no updates available." Stock also has a disabled "Cancelling Update..." row. The title bar Update button is TB-5 and the update cards are NT-6.
- Commands: update.check; update.checking; update.downloadNow; update.downloading; update.install; update.updating; update.cancelling; update.restart
- Menus: GlobalActivity (group 7_update)
- Keys: none found
- Settings: update.mode; update.enableWindowsBackgroundUpdates
- Source: V\src\vs\workbench\contrib\update\browser\update.ts (80-157, 260-318); V\src\vs\platform\update\common\update.config.contribution.ts (19-75); C bundle: vs/workbench/contrib/update/browser/update (NLS 11511-11514)
- Pieces:
  1. When no update work is under way, the last group of the Manage menu shows "Check for Updates...", and picking it checks for an update.
  2. When an update can be downloaded, the row reads "Download Update (1)", and picking it starts the download.
  3. When an update has been downloaded and waits for install, the row reads "Install Update... (1)", and picking it starts the install.
  4. When an update is ready, the row reads "Restart to Update (1)", and picking it quits and installs the update.
  5. While the updater is busy, the last group shows one disabled row that names the step: "Checking for Updates...", "Downloading Update..." or "Installing Update...".
  6. The gear shows a count badge of 1 with the hover text "New {0} update available." when an update is available, downloaded or ready, and a progress badge while the updater is checking, downloading or installing.
  7. A check that finds no update opens an info box reading "There are currently no updates available.". [stock only]
  8. While an update is being cancelled, a disabled "Cancelling Update..." row shows and the gear shows a progress badge. [stock only]
  9. Setting `update.mode` chooses how updates are found, with values `none`, `manual`, `start` and `default`.
  10. In Cursor, `update.mode` also accepts the value `silentlyApplyOnQuit`. [cursor only]

#### AB-4.22 Read the update release notes from Manage

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - the command and the row (condition: state is ready and a major or minor update) are in the bundle. `update.showReleaseNotes` is not.
- What: When an update is ready, it is a major or minor version step and the build is stable, Manage shows "Show Update Release Notes" above "Restart to Update". It opens the release notes for the new version. The release notes page itself is not cataloged here (see Unclear).
- Commands: update.showUpdateReleaseNotes
- Menus: GlobalActivity (group 7_update, order 1)
- Keys: none found
- Settings: update.showReleaseNotes
- Source: V\src\vs\workbench\contrib\update\browser\update.ts (270-280, 320-341); V\src\vs\platform\update\common\update.config.contribution.ts (74-80); C bundle: vs/workbench/contrib/update/browser/update (NLS 11513)
- Pieces:
  1. When an update is ready, it is a major or minor version step and the build is stable, the Manage menu shows "Show Update Release Notes" above "Restart to Update".
  2. Picking "Show Update Release Notes" opens the release notes for the new version.
  3. For a patch-level update, or on a build that is not stable, the row is not shown.
  4. Setting `update.showReleaseNotes`, on by default, chooses whether release notes are shown after an update. [stock only]

#### AB-4.23 Reach Accounts and Manage with the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present (vertical rail only) - the arrow key listeners that call `globalCompositeBar.focus()` are in the bundle. The focus command is enabled only when `workbench.activityBar.orientation` is `vertical`. The default horizontal tab row has no keyboard handling at all (AB-6).
- What: In the rail, Down or Right arrow on the view container icons moves focus into the Accounts and Manage icons, and Up or Left from them moves back to the last view container icon. Enter or Space on a focused icon opens its menu. Focus does not wrap round (`preventLoopNavigation`), and the two icons are a vertical action bar labelled "Manage" for screen readers. `workbench.action.focusActivityBar` focuses the rail and has no default key.
- Commands: workbench.action.focusActivityBar
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts (373-408, 625-640); V\src\vs\workbench\browser\parts\globalCompositeBar.ts (95-103, 218-228); C bundle: the activity bar part's `registerKeyboardNavigationListeners` code (`this.globalCompositeBar?.focus()`)
- Pieces:
  1. In the rail, pressing Down or Right on the view container icons moves focus into the Accounts and Manage icons.
  2. Pressing Up or Left on those two icons moves focus back to the last view container icon.
  3. Focus does not wrap round when it reaches either end of the two icons.
  4. A screen reader announces the Accounts and Manage icons as one vertical action bar labelled "Manage".
  5. Running `workbench.action.focusActivityBar` puts keyboard focus on the rail, and it has no default key.
  6. In Cursor, `workbench.action.focusActivityBar` is enabled only when `workbench.activityBar.orientation` is `vertical`, and the default horizontal tab row has no keyboard handling at all. [cursor only]

#### AB-4.24 Draw Accounts and Manage in the title bar when the activity bar is top or bottom

- Tag: vscode. Evidence: source. Boundary: no. Cursor: changed - the code, strings and right-toolbar entries (Accounts at order 900, Manage at order 910) are in the bundle, but the title bar's `activityActionsEnabled` getter always returns false, so they are never drawn there. With the vertical rail and location `top` or `bottom` they appear nowhere (derived from code, not run).
- What: When `workbench.activityBar.location` is `top` or `bottom`, the rail is not drawn, so the title bar's right toolbar ends with the Accounts icon (if shown) and the Manage gear, after the layout and global buttons. They never fold into the toolbar's "..." overflow (exempt from its 12-item limit), the hover opens below them, and they use the same menus, profile badge and update badge as the rail versions. They are not drawn in auxiliary windows or compact title bars. With location `default` they live in the rail. With `hidden` stock draws them nowhere.
- Commands: workbench.actions.accounts; workbench.actions.manage
- Menus: AccountsContext; GlobalActivity
- Keys: none found
- Settings: workbench.activityBar.location
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts (37, 631-640, 671-678, 740-751, 800-805, 884-887); V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts (277-292); V\src\vs\workbench\browser\workbench.contribution.ts (650-661); C bundle: vs/workbench/browser/parts/titlebar/titlebarActions (NLS 4022-4025)
- Pieces:
  1. With `workbench.activityBar.location` set to `top` or `bottom`, the rail is not drawn and the right end of the title bar toolbar shows the Accounts icon, if it is shown, and the Manage gear, after the layout and global buttons. [stock only]
  2. The two icons never fold into the toolbar's "..." overflow menu, even when the toolbar is over its 12 item limit. [stock only]
  3. Clicking either icon opens the same Accounts or Manage menu as on the rail, with the menu opening below the icon, and the gear keeps its profile badge and update badge. [stock only]
  4. The two icons are not drawn in auxiliary windows or in compact title bars. [stock only]
  5. With location `default` the two icons live in the rail, and with `hidden` they are drawn nowhere.
  6. In Cursor, with location `top` or `bottom` the Accounts icon and the Manage gear appear nowhere, because the title bar never draws them. [cursor only]

#### AB-4.25 Right-click the title bar Accounts or Manage icon

- Tag: vscode. Evidence: source. Boundary: no. Cursor: changed - the menu code (ids `hideAccounts`, `toggle.hideAccounts`, `toggle.hideManage`) is in the bundle, but it is never reached because the title bar never draws these icons.
- What: Right-clicking the title bar Accounts icon shows "Hide Accounts", a separator, an "Accounts" check item, and a "Manage" check item that is ticked and disabled (Manage cannot be hidden, and the action throws if forced). Right-clicking the Manage gear shows only the last two. This replaces the rail's right-click menu, which would show the activity bar position and size items. Hide Accounts removes the icon from the title bar and the rail, because both read one storage key.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (662-750); V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts (631-640); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3850-3852)
- Pieces:
  1. Right-clicking the title bar Accounts icon shows "Hide Accounts", a separator, an "Accounts" check item and a "Manage" check item. [stock only]
  2. The "Manage" check item is ticked and disabled, because Manage cannot be hidden. [stock only]
  3. Right-clicking the title bar Manage gear shows only the "Accounts" and "Manage" check items. [stock only]
  4. Picking Hide Accounts in this menu removes the Accounts icon from both the title bar and the rail, because both read the same storage key. [stock only]
  5. In Cursor this menu is never reached, because the title bar does not draw the Accounts or Manage icons. [cursor only]


## AB-5 Placement, size and visibility

(vscode, since Cursor lacks the commands). Where the bar lives (side, top, bottom, hidden), how big it is, and the settings that change that. Examples: `workbench.activityBar.location`; the Move Activity Bar commands; `compact` and `autoHide`; Toggle Activity Bar Visibility.

9 features.

#### AB-5.1 Choose where the activity bar lives in Settings

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the setting is kept with the same four values and the same descriptions, but the side bar and rail that read it load only in the vertical layout; the default horizontal tab row never reads it, and only small side effects still follow it there: the title bar empty check, a view header refresh, and the hover side of the compact menu button)
- What: Set `workbench.activityBar.location` to default (rail beside the Primary Side Bar), top, bottom or hidden, from the Settings editor or settings.json. Old values migrate: `workbench.activityBar.visible` set to false becomes hidden, and `side` becomes default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.location
- Source: V\src\vs\workbench\browser\workbench.contribution.ts 650-662 (setting), 974-995 (migrations); V\src\vs\workbench\services\layout\browser\layoutService.ts 45-47; V\src\vs\workbench\browser\layout.ts 3154-3160 (when the rail counts as hidden); C bundle: setting definition (NLS 4222-4226), migrations for `workbench.activityBar.visible` and `side`, class `ePt` (horizontal side bar, no read of the setting), class `JDt` (vertical side bar, reads it), `isActivityBarHidden` (returns true when orientation is horizontal)
- Pieces:
  1. Setting `workbench.activityBar.location` chooses where the activity bar sits, with values `default` (a rail beside the Primary Side Bar and a strip on top of the Secondary Side Bar), `top`, `bottom` and `hidden`, and it can be changed in the Settings editor or in settings.json.
  2. A saved `workbench.activityBar.visible` of false is migrated to `workbench.activityBar.location` set to `hidden`, and a saved location of `side` is migrated to `default`.
  3. In Cursor the setting keeps the same four values and descriptions, but the side bar and rail that read it load only when `workbench.activityBar.orientation` is `vertical`, so in the default horizontal layout it has almost no visible effect. [cursor only]

#### AB-5.2 Move the activity bar from a menu or the Command Palette

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched the bundle and both NLS files for the four command ids, `ActivityBarPositionMenu`, `Activity Bar Position`, `Move Activity Bar` and `Hide Activity Bar`: 0 hits; the Cursor module `vs/workbench/browser/parts/activitybar/activitybarPart` keeps only the keys menu, hideMenu, reloadForActivityBar, previousSideBarView, nextSideBarView, focusActivityBar)
- What: Run Move Activity Bar to Side, Move Activity Bar to Top, Move Activity Bar to Bottom or Hide Activity Bar from the Command Palette, or pick Default, Top, Bottom or Hidden in the Activity Bar Position submenu. The submenu appears in View > Appearance, in the rail icon right-click menu, in the view container context menu and in the Secondary Side Bar title menu, and the current choice shows a check. Each palette command is offered only when the bar is not already there.
- Commands: workbench.action.activityBarLocation.default; workbench.action.activityBarLocation.top; workbench.action.activityBarLocation.bottom; workbench.action.activityBarLocation.hide
- Menus: ActivityBarPositionMenu; MenubarAppearanceMenu (submenu entry, group 3_workbench_layout_move, order 2); ViewContainerTitleContext (submenu entry, group 3_workbench_layout_move, order 1); CommandPalette
- Keys: none found
- Settings: workbench.activityBar.location
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts 451-478 (rail menu submenu), 480-601 (four commands and both submenu entries); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts 244-258; V\src\vs\platform\actions\common\actions.ts 134 (menu id); C bundle: NLS module `vs/workbench/browser/parts/activitybar/activitybarPart` in `C\out\nls.keys.json`
- Pieces:
  1. Running "Move Activity Bar to Side", "Move Activity Bar to Top", "Move Activity Bar to Bottom" or "Hide Activity Bar" from the Command Palette sets `workbench.activityBar.location` to `default`, `top`, `bottom` or `hidden`. [stock only]
  2. Each of those four palette commands is offered only when the bar is not already in that place. [stock only]
  3. View > Appearance has an "Activity Bar Position" submenu with Default, Top, Bottom and Hidden, and the current choice shows a check. [stock only]
  4. The same "Activity Bar Position" submenu also appears in the rail icon right-click menu, in the view container context menu and in the Secondary Side Bar title menu. [stock only]

#### AB-5.3 See the activity bar as a strip at the top or bottom of the side bar

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the strip is drawn only by the old stock side bar class that loads when orientation is `vertical`; the default horizontal layout never draws it; Accounts and Manage do not move to the title bar because the title bar's activity actions are switched off)
- What: With location top or bottom the 48 px rail is not drawn. The same container icons (16 px, 30 px overflow slot) show as a horizontal strip in the Primary Side Bar title (top) or along its bottom edge (bottom), using the `activityBarTop.*` colors, and a right-click on the strip adds a Views submenu. In stock, Accounts and Manage also move to the title bar (owned by AB-4) and the custom title bar is never treated as empty.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.location
- Source: V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts 197-232 (strip options), 234-273 (when it shows and where); V\src\vs\workbench\browser\parts\paneCompositePart.ts 793-805 (Views submenu); V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts 884-887; V\src\vs\workbench\services\layout\browser\layoutService.ts 590-612 (title bar never empty); C bundle: class `JDt` methods `shouldShowCompositeBar` and `getCompositeBarPosition`, title bar `get activityActionsEnabled(){return!1}`, `getDummyActivityBarView` (zero-size stand-in in horizontal mode)
- Pieces:
  1. With location `top`, the rail is not drawn and the same container icons show as a horizontal strip in the title of the Primary Side Bar.
  2. With location `bottom`, the same strip runs along the bottom edge of the Primary Side Bar.
  3. The strip's icons are 16 px with a 30 px overflow slot, and the strip is colored with the `activityBarTop.*` theme colors.
  4. Right-clicking the strip adds a Views submenu to its context menu.
  5. In Cursor the strip is drawn only by the older side bar that loads when orientation is `vertical`, the default horizontal layout never draws it, and Accounts and Manage do not move to the title bar. [cursor only]

#### AB-5.4 Hide the activity bar

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (hiding by the setting works in the vertical layout; the Hide Activity Bar command and the Hidden menu entry are missing; in the default horizontal layout there is no rail, so nothing is left to hide)
- What: Choose Hidden and no rail or strip is drawn for the Primary or Secondary Side Bar, and the Accounts and Manage icons go with it. Reach it from the setting, the Hide Activity Bar command or the Hidden entry in the Activity Bar Position submenu. The layout state key `activityBar.hidden` follows the setting, and in stock a change to it writes `hidden` back to the setting.
- Commands: workbench.action.activityBarLocation.hide
- Menus: ActivityBarPositionMenu; CommandPalette
- Keys: none found
- Settings: workbench.activityBar.location; workbench.activityBar.visible (legacy, migrated to location)
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts 558-582; V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts 255-262; V\src\vs\workbench\browser\layout.ts 1361-1362, 2296-2297, 2851, 2940-2941, 3154-3160; C bundle: `setActivityBarHidden`, `isActivityBarHidden`, `updateLegacySettingsFromState` (writes `hidden` back to the setting)
- Pieces:
  1. Setting `workbench.activityBar.location` to `hidden` removes the rail or strip for both the Primary and Secondary Side Bars, and the Accounts and Manage icons go with it.
  2. Running "Hide Activity Bar" from the Command Palette, or picking Hidden in the Activity Bar Position submenu, hides the bar the same way. [stock only]
  3. When the bar is hidden by any route other than editing the setting, `workbench.activityBar.location` is updated to `hidden`.
  4. In Cursor, hiding by the setting works only in the vertical layout, and the default horizontal layout has no rail to hide. [cursor only]

#### AB-5.5 Toggle Activity Bar Visibility

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the command id and title are kept and work as stock in the vertical layout; the Customize Layout picker has no Activity Bar item, its list is Menu Bar, Primary Side Bar, Secondary Side Bar, Panel and Status Bar; in the default horizontal layout the command clears `workbench.activityBar.location` and shows an info dialog "Please quit and reopen for the activity bar to be displayed vertically.")
- What: Run Toggle Activity Bar Visibility to hide the bar, and run it again to bring it back where it was last visible (side, top or bottom, remembered in profile storage). The Customize Layout picker has an Activity Bar eye toggle that runs the same command. The command has no Command Palette entry, no menu item and no default key in stock.
- Commands: workbench.action.toggleActivityBarVisibility
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.location
- Source: V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts 275-290 (remember last visible position), 321-335 (command); V\src\vs\workbench\browser\actions\layoutActions.ts 62 (id), 1360-1370 (picker items); C bundle: command `workbench.action.toggleActivityBarVisibility` (NLS 3985, module `vs/workbench/browser/parts/sidebar/sidebarPart` key toggleActivityBar), dialog text (NLS 3301, key reloadForActivityBar), picker list built next to `config.workbench.sideBar.location` (no activity bar entry)
- Pieces:
  1. Running "Toggle Activity Bar Visibility" (`workbench.action.toggleActivityBarVisibility`) hides the activity bar.
  2. Running it again brings the bar back where it was last visible, side, top or bottom, which is remembered in profile storage.
  3. The Customize Layout picker has an Activity Bar eye toggle that runs the same command. [stock only]
  4. The command can be run only from the Customize Layout picker or from a key you assign, because it has no Command Palette entry, menu item or default key.
  5. In Cursor the command keeps its id and title, the Customize Layout picker has no Activity Bar item, and in the default horizontal layout the command clears `workbench.activityBar.location` and shows an info dialog reading "Please quit and reopen for the activity bar to be displayed vertically.". [cursor only]

#### AB-5.6 Switch the activity bar between default and compact size

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched for the setting id, `workbench.action.activityBar.size`, `Activity Bar Size`, `activityBarCompact` and a compact class: 0 hits; the Cursor rail is fixed at 48 px wide, 48 px items, 24 px icons, item size 52)
- What: Turn on `workbench.activityBar.compact`, or pick Compact in the Activity Bar Size submenu of the rail right-click menu, for a narrower rail (36 px wide, 32 px items, 16 px icons) instead of 48 px, 48 px and 24 px. The submenu shows only when location is default, and the rail is rebuilt at once. The two size entries are built in code and are not registered commands.
- Commands: workbench.action.activityBar.size.default; workbench.action.activityBar.size.compact
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.compact; workbench.activityBar.location
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts 44-52 (sizes), 95-120 (live switch), 456-466 (submenu); V\src\vs\workbench\browser\workbench.contribution.ts 669-674; V\src\vs\workbench\test\browser\parts\activitybar\activitybarPart.test.ts; C bundle: class `ore` (vertical rail, fixed sizes)
- Pieces:
  1. Setting `workbench.activityBar.compact`, off by default, switches the rail to a narrower layout, 36 px wide with 32 px items and 16 px icons instead of 48 px, 48 px and 24 px. [stock only]
  2. The "Activity Bar Size" submenu in the rail right-click menu offers Default and Compact, with a check on the current size. [stock only]
  3. The "Activity Bar Size" submenu shows only when location is `default`. [stock only]
  4. The rail is rebuilt at once when the size changes. [stock only]

#### AB-5.7 Auto-hide the activity bar when only one container shows

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched for `activityBar.autoHide`, `activityBarAutoHide` and the visible-containers tracker class: 0 hits; Cursor's `shouldShowCompositeBar` only tests for top or bottom)
- What: With `workbench.activityBar.autoHide` on and location top or bottom, the strip disappears when the side bar has a single visible view container, and comes back when a second one appears. It applies to both the Primary and Secondary Side Bars.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.autoHide; workbench.activityBar.location
- Source: V\src\vs\workbench\browser\workbench.contribution.ts 663-668; V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts 131-143, 234-253; V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts 130-161, 262-283; C bundle: class `JDt` method `shouldShowCompositeBar`
- Pieces:
  1. With `workbench.activityBar.autoHide` on, off by default, and location set to `top` or `bottom`, the strip disappears when the side bar has only one visible view container. [stock only]
  2. When a second visible view container appears, the strip comes back. [stock only]
  3. The hiding applies to both the Primary and Secondary Side Bars. [stock only]

#### AB-5.8 Hide the activity bar when Zen Mode turns on

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (setting kept with default true and the same description; its change handler is an older copy that does not keep the strip hidden for top or bottom; with the default horizontal layout there is no rail, so it has no visible effect)
- What: With `zenMode.hideActivityBar` on (the default), entering Zen Mode hides the activity bar, changing the setting while in Zen Mode shows or hides it at once, and leaving Zen Mode brings it back unless it was already hidden.
- Commands: workbench.action.toggleZenMode
- Menus: none found
- Keys: none found
- Settings: zenMode.hideActivityBar
- Source: V\src\vs\workbench\browser\workbench.zenMode.contribution.ts 56-59, 102; V\src\vs\workbench\browser\layout.ts 1480-1481, 1509-1512, 1564-1565; V\src\vs\workbench\services\layout\browser\layoutService.ts 37; C bundle: `zenMode.hideActivityBar` definition (NLS 4316) and the Zen Mode configuration listener that calls `setActivityBarHidden`
- Pieces:
  1. Running `workbench.action.toggleZenMode` to enter Zen Mode hides the activity bar while `zenMode.hideActivityBar` is on.
  2. Setting `zenMode.hideActivityBar`, on by default, chooses whether Zen Mode hides the activity bar, and with it off the bar stays visible.
  3. Changing the setting while in Zen Mode shows or hides the activity bar at once.
  4. Leaving Zen Mode brings the activity bar back, unless it was already hidden before.
  5. In Cursor the setting keeps its default of true, but its change handler is an older copy that does not keep the strip hidden for `top` or `bottom`, and with the default horizontal layout there is no rail, so it has no visible effect. [cursor only]

#### AB-5.9 See smaller badges in the compact layouts

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the strip badge styles are kept and used by the vertical layout; the compact rail styles are missing along with the setting, 0 hits for `activitybar.compact` in the Cursor CSS; the default horizontal row draws no badges)
- What: Badges shrink where the bar is compact. On a compact rail (`workbench.activityBar.compact` on) the number pill is 13 px tall and the profile text on the gear is 8 px, instead of 16 px and 9 px. On the top or bottom strip the badge uses the `compact` class, and a number of three or more characters adds `compact-content` with an 8 px font. Progress badges on the strip use a 13 px mask.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.compact
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts 325-350; V\src\vs\workbench\browser\parts\activitybar\media\activityaction.css 158-205; V\src\vs\workbench\browser\parts\media\paneCompositePart.css 235-262; C bundle: `this.options.compact&&s.length>=3&&i.push("compact-content")`, CSS `badge.compact` rules in `workbench.desktop.main.css`, class `JDt` option `compact:!0`
- Pieces:
  1. On a compact rail the number pill is 13 px tall instead of 16 px. [stock only]
  2. On a compact rail the profile letters on the gear use an 8 px font instead of 9 px. [stock only]
  3. On the top or bottom strip the number pill is drawn in a smaller style, 13 px tall with a 9 px font.
  4. On the strip, a number of three or more characters, such as `100` or `1K+`, drops to an 8 px font.
  5. On the strip, the progress glyph in a progress badge is drawn at 13 px.
  6. In Cursor the strip badge styles are kept for the vertical layout, the compact rail styles are missing along with the setting, and the default horizontal row draws no badges. [cursor only]


## AB-6 Pinned container tab row

(cursor). Cursor's own icon tab row at the top of the side bar, its overflow list, pin and reorder controls, menus, and the `orientation` setting. Examples: default tabs Explorer, Search, Source Control, Extensions; the chevron list with pin toggles; right-click Hide or Keep; `workbench.activityBar.orientation` with a restart prompt. It has no badges, tooltips, or keyboard handling.

14 features.

#### AB-6.1 Choose between the tab row and the vertical rail

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Set `workbench.activityBar.orientation` to `horizontal` (the default: no rail, and the Primary Side Bar starts with the pinned tab row) or `vertical` (a stock style 48 px rail plus the older stock side bar). It sits in the Settings editor under Workbench > Appearance with the text "Controls the orientation of the activity bar in the workbench. Quit and reopen for changes to take effect." No command, menu item or key flips it, and the choice is read once at startup. In horizontal mode a container with one view also keeps that view's own header instead of merging it into the container title.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.orientation
- Source: C bundle: setting definition `[D5e]:{type:"string",enum:["horizontal","vertical"],default:"horizontal"}` (NLS 4227); part chooser in class `UEa` (builds `JDt` for vertical and `ePt` for horizontal); layout members `activityBarDirection`, `isActivityBarHidden`, `getDummyActivityBarView`; Settings editor group `workbench/appearance` (NLS 8750); searched for a toggle command (`ctivityBarOrientation`, `ctivity Bar Orientation`): 0 hits
- Pieces:
  1. Setting `workbench.activityBar.orientation` chooses the layout, with `horizontal` (the default, no rail, and the Primary Side Bar starts with the pinned tab row) and `vertical` (a stock style 48 px rail plus the older stock side bar).
  2. In the Settings editor the setting sits under Workbench > Appearance with the text "Controls the orientation of the activity bar in the workbench. Quit and reopen for changes to take effect.".
  3. No command, menu item or key flips the orientation, and the choice is read once at startup.
  4. In horizontal mode a container with one view keeps that view's own header instead of merging it into the container title.

#### AB-6.2 Restart prompt after changing the orientation

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: When the orientation value changes, a confirm dialog says "A setting has changed that requires a restart to take effect." with the detail "Press the restart button to restart {product name} and enable the setting." and a Restart button that restarts the app (the web build says reload instead). It shows only while the window has focus and only when the value really changed. If you decline, the saved value stays and the layout switches at the next start.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.orientation
- Source: C bundle: settings change relauncher class `fri` (its `fri.SETTINGS` list includes the orientation setting, and `activitybarOrientation.handleChange` tracks it), NLS 8923, 8925, 8927 (native) and 8924, 8926, 8928 (web); stock counterpart V\src\vs\workbench\contrib\relauncher\browser\relauncher.contribution.ts (same prompt for other settings, no orientation)
- Pieces:
  1. When the orientation value changes, a confirm dialog says "A setting has changed that requires a restart to take effect." with the detail "Press the restart button to restart {product name} and enable the setting.".
  2. The dialog's Restart button restarts the app, and the web build says reload instead.
  3. The dialog shows only while the window has focus and only when the value really changed.
  4. If you decline the dialog, the saved value stays and the layout switches at the next start.

#### AB-6.3 See the pinned container tab row at the top of the side bar

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: In the default layout the Primary Side Bar starts with a 35 px row of icon-only tabs, one per pinned container, centered with a 2 px gap. Each tab is 26 px square with a 16 px icon and no text, tooltip or badge. The open container's tab gets the selected style (selected background, active panel title color, and a contrast outline in high contrast themes); hover uses the list hover background. A first run pins Explorer, Search, Source Control and Extensions. Run and Debug, Testing and every other container live only in the chevron list until pinned.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle: class `ePt` (part `workbench.parts.sidebar`, minimum size 214 x 77), `$yk` (the row), `b6m` (one tab), `pyk` (default pins `workbench.view.explorer`, `workbench.view.search`, `workbench.view.scm`, `workbench.view.extensions`); CSS classes `.sidebar2-item-icon-anysphere` and `.sidebar-list-item-selected` in `workbench.desktop.main.css`
- Pieces:
  1. In the default layout the Primary Side Bar starts with a 35 px row of icon-only tabs, one per pinned container, centered with a 2 px gap.
  2. Each tab is 26 px square with a 16 px icon and shows no text, tooltip or badge.
  3. The tab of the open container gets the selected style, with a selected background and the active panel title color, plus a contrast outline in high contrast themes.
  4. Hovering a tab uses the list hover background.
  5. A first run pins Explorer, Search, Source Control and Extensions.
  6. Run and Debug, Testing and every other container show only in the chevron list until they are pinned.

#### AB-6.4 Open a container by clicking its tab

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Click a tab to open that container in the side bar and move focus into it. Clicking the tab that is already open does nothing, because the row never reads `workbench.activityBar.iconClickBehavior` and has no double click guard. The tab, chevron and list code has no tab stops, arrow keys, ARIA roles or hover tooltips, so the row is mouse only. The Show Explorer, Show Search and similar commands still open the same containers.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle: `$yk` select handler (opens the container only when it differs from the active one, then closes the list), `ePt.openPaneComposite`; searched the whole row, chevron and list code for keydown, keyup, tabindex, role, aria and title attributes: 0 hits
- Pieces:
  1. Clicking a tab opens that container in the side bar and moves focus into it.
  2. Clicking the tab of the container that is already open does nothing, whatever `workbench.activityBar.iconClickBehavior` says.
  3. By code reading, the tabs, the chevron and the list work only with the mouse, with no tab stops, arrow keys, ARIA roles or hover tooltips.
  4. Commands such as Show Explorer and Show Search still open the same containers, and the selected tab follows the open container.

#### AB-6.5 Tabs that do not fit leave the row

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The row shows only as many pinned tabs as fit: the row width divided by 40 px, rounded down, minus one slot for the chevron, and minus one more when the open container is not pinned. At the 214 px minimum width that is 4 tabs, or 3 with an extra tab. A pinned tab that does not fit has no tab and can be reached only from the list. The width is watched live, so widening the side bar brings tabs back.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle: `$yk` (`Math.min(pinnedCount,Math.floor(width/40)-1-(extraTab?1:0))` and a resize observer on the row)
- Pieces:
  1. The row shows only as many pinned tabs as fit, which is the row width divided by 40 px rounded down, minus one slot for the chevron, and minus one more when the open container is not pinned.
  2. At the 214 px minimum side bar width the row shows 4 tabs, or 3 when an extra tab is showing.
  3. A pinned tab that does not fit has no tab and can be reached only from the list.
  4. The count of tabs follows the side bar width as it changes, so widening the side bar brings tabs back.

#### AB-6.6 The open container always shows as a tab

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: When the open container is not pinned, the row adds one extra selected tab at its end for that container, so the current view is always visible. Switching to a pinned container removes the extra tab. Right-clicking the extra tab opens the same tab menu as any other tab.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle: `$yk` (finds the active container among all containers when it is not pinned and renders it as one more `b6m` tab with `selected` true)
- Pieces:
  1. When the open container is not pinned, the row adds one extra selected tab at its end for that container.
  2. Switching to a pinned container removes the extra tab.
  3. Right-clicking the extra tab opens the same tab menu as any other tab.

#### AB-6.7 Open the container list with the chevron

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A chevron button after the tabs (pointing down when closed, up and highlighted when open) opens a list under the row, lying over the view content with a soft shadow. It has one row per container in the side bar: icon, name, the container's keyboard shortcut at half opacity when it has one, and a pin icon. Pinned containers come first and the open container's row is highlighted. Click a row to open that container. The list closes after a pick, when the open container changes, or on a second click on the chevron; no click outside or Escape handling was found. Containers marked hide when empty that have no active views are not listed. The second title row of Run and Debug is hidden while the list is open.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle: `$yk` (chevron button, open and close state), `Hyk` (the list), `y6m` (one row), `ePt.getViewContainers` (the hide when empty filter, and a re-query 1 s and 11 s after containers register); CSS classes `.sidebar2-item-icon-anysphere-hover` and `.sidebar-list-item`
- Pieces:
  1. A chevron button after the tabs opens a list under the row, lying over the view content with a soft shadow.
  2. The chevron points down while the list is closed, and points up and is highlighted while the list is open.
  3. The list has one row per container in the side bar, showing the icon, the name, the container's keyboard shortcut at half opacity when it has one, and a pin icon.
  4. Pinned containers come first in the list, and the row of the open container is highlighted.
  5. Clicking a row opens that container.
  6. The list closes after a pick, when the open container changes, and on a second click on the chevron, and by code reading clicking outside the list or pressing Escape does not close it.
  7. Containers marked hide when empty that have no active views are not listed.

#### AB-6.8 Pin or unpin a container

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: In the list, click a row's pin icon to add or remove that container's tab. The icon is hidden on unpinned rows until the row is hovered, and the click does not open the row. In the tab right-click menu, Hide '{name}' unpins and Keep '{name}' pins, and the checklist at the bottom of the menu toggles any container, with a check on pinned ones. The menu entries are disabled for the last pinned container, but the list pin icon has no such guard, so the list can unpin every container. The Agents container always counts as pinned. Every change is saved.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle: `y6m` (pin icon click handler), `$yk` tab menu builder (`togglePin` item and the checklist, both guarded by `pinnedViewContainerIDs.length>1`), `Wke` (pinned test, always true for `workbench.view.backgroundAgent`), `oQn` (save), NLS 3979-3980; CSS `.sidebar-2-pinned-icon-not-pinned` and `.sidebar-list-item:hover .sidebar-2-pinned-icon`
- Pieces:
  1. Clicking the pin icon on a list row adds or removes that container's tab, and the click does not open the row.
  2. The pin icon is hidden on unpinned rows until the row is hovered.
  3. In a tab's right-click menu, Hide '{name}' unpins that container and Keep '{name}' pins it.
  4. The checklist at the bottom of the tab menu toggles any container, with a check on the pinned ones.
  5. The tab menu entries are disabled for the last pinned container, but the pin icon in the list has no such guard, so the list can unpin every container.

#### AB-6.9 Reorder containers by dragging in the list

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Drag a list row up or down to change its place. A row moves only among rows of its own kind (pinned among pinned, unpinned among unpinned). The dragged row drops to half opacity and a copy follows the pointer. The new place is stored as a number halfway between its neighbors, and the tab order follows the pinned order. The list is ordered pinned first, then by saved order, then by the container's own order, then by a fixed tiebreak computed from the container id. The tabs in the row cannot be dragged, and no drop handling for a view or container dragged in from another place was found.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle: `$yk` (drag end handler, `viewContainerOrders` with `order` and `secondaryOrder`, and the sort), `Uyk` (draggable list row, opacity 0.5 while dragged), `Hyk`, `b6m` (tab, no drag code), `oQn` (save)
- Pieces:
  1. Dragging a list row up or down changes its place in the list.
  2. A row moves only among rows of its own kind, pinned among pinned and unpinned among unpinned.
  3. While a row is dragged it drops to half opacity and a copy follows the pointer.
  4. The order of the tabs in the row follows the pinned order in the list.
  5. The list is ordered pinned first, then by the saved order, then by the container's own order, then by a fixed tiebreak computed from the container id.
  6. The tabs in the row cannot be dragged.

#### AB-6.10 Right-click a tab for its menu

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Right-click a tab (the extra tab too) to open a menu with, in order: Configure Keybinding (when the container has a key id), Hide or Keep '{name}', then the container's title-context items (a show or hide toggle for each of its views, Reset Location when the container was moved from its home, and Hide Primary Side Bar), a separator, and a checklist of every container. Not offered, though the stock rail icon menu has them: Hide Badge and Show Badge, Move To, Activity Bar Position, Activity Bar Size, the Accounts and Menu toggles, and Move Primary Side Bar Right or Left. Right-click on list rows does nothing.
- Commands: workbench.action.toggleSidebarVisibility; `<viewId>.toggleVisibility` (pattern, one per view); `<containerId>.resetViewContainerLocation` (pattern)
- Menus: ViewContainerTitleContext
- Keys: none found
- Settings: none found
- Source: C bundle: `$yk` menu builder and its contextmenu handler, NLS module `vs/workbench/browser/parts/sidebar/sidebarMenu` (NLS 3977-3984), the `ViewContainerTitleContext` registrations in Cursor (Hide Primary Side Bar NLS 3101, Hide Secondary Side Bar NLS 3310, view toggles group `1_toggleVisibility`, Reset Location NLS 13433 group `1_viewActions`); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts 831-921; V\src\vs\workbench\browser\actions\layoutActions.ts 340-352; V\src\vs\workbench\browser\actions\layoutActions.ts 185-225 (the four Move Side Bar items that Cursor does not register on this menu)
- Pieces:
  1. Right-clicking a tab, including the extra tab, opens a menu.
  2. The menu lists, in order, Configure Keybinding when the container has a key id, then Hide or Keep '{name}', then the container's title-context items, then a separator, then a checklist of every container.
  3. The title-context items are a show or hide toggle for each of the container's views, Reset Location when the container was moved from its home, and Hide Primary Side Bar.
  4. The menu does not offer Hide Badge or Show Badge, Move To, Activity Bar Position, Activity Bar Size, the Accounts and Menu toggles, or Move Primary Side Bar Right or Left, which the stock rail icon menu has.
  5. Right-clicking a row in the chevron list does nothing.

#### AB-6.11 Configure a container's keybinding from its tab

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The first item in the tab menu, Configure Keybinding, shows the container's current key after a tab character (or only the label when none is set) and opens the Keyboard Shortcuts editor filtered to that container's open command. It shows only for containers that have a keybinding id. The stock rail icon menu has the same item; the tab row has its own copy.
- Commands: workbench.action.openGlobalKeybindings
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle: `$yk` menu builder (`configureKeybinding` item), NLS 3977-3978; V\src\vs\workbench\browser\parts\compositeBarActions.ts 637-685 (stock counterpart on the rail)
- Pieces:
  1. The first item in the tab menu, Configure Keybinding, shows the container's current key after a tab character, or only the label when no key is set.
  2. Picking Configure Keybinding opens the Keyboard Shortcuts editor filtered to that container's open command.
  3. The item shows only for containers that have a keybinding id.

#### AB-6.12 Right-click empty row space to move the side bar

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Right-click on blank space in the row (not on a tab) to get one item, Move Primary Side Bar Right (or Left when the bar is already on the right), with a tooltip that repeats the same words. It flips the side bar to the other side.
- Commands: workbench.action.toggleSidebarPosition
- Menus: none found
- Keys: none found
- Settings: workbench.sideBar.location
- Source: C bundle: `$yk` blank-space contextmenu handler (acts only when the click target is the row itself), NLS 3981-3984
- Pieces:
  1. Right-clicking blank space in the row, not on a tab, opens a menu with one item, Move Primary Side Bar Right, or Move Primary Side Bar Left when the bar is already on the right.
  2. The item has a tooltip that repeats the same words.
  3. Picking the item flips the Primary Side Bar to the other side, changing `workbench.sideBar.location`.

#### AB-6.13 Run and Debug gets a second title row

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: While Run and Debug is open, the title grows from 35 px to 70 px and a second centered row shows that container's title actions: the Start Debugging button with the launch configuration picker, the Open 'launch.json' gear, and an "Actions..." overflow menu for secondary ones. The row is hidden while the chevron list is open. Only the Run and Debug container asks for it (`wantsActionToolbar`, an option stock does not have). Other containers show no container-level toolbar in the tab row, because their views keep their own headers.
- Commands: workbench.action.debug.start; workbench.action.debug.configure
- Menus: ViewContainerTitle
- Keys: none found
- Settings: none found
- Source: C bundle: the `workbench.view.debug` container registration with `wantsActionToolbar:!0`, `$yk` (title height 35 or 70), `Xyk` (the `title-actions` toolbar with `toggleMenuTitle:"Actions..."`); V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts 171-224 (the stock title items); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (container registration; 0 hits for `wantsActionToolbar` anywhere under V\src)
- Pieces:
  1. While Run and Debug is open, the title grows from 35 px to 70 px and a second centered row shows that container's title actions.
  2. The second row has the Start Debugging button with the launch configuration picker, the Open 'launch.json' gear, and an "Actions..." overflow menu for the secondary actions.
  3. The second row is hidden while the chevron list is open.
  4. Only Run and Debug gets this second row, and other containers show no container-level toolbar in the tab row because their views keep their own headers.

#### AB-6.14 Keep pins and order between sessions

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Pinning, unpinning and reordering save `{pinnedViewContainerIDs, viewContainerOrders}` as JSON under `sidebar2.sidebarData.memoized.v1` (application scope, machine target), so they apply to every window and workspace on the machine. The open container is saved per workspace under `workbench.sidebar.activeviewletid` (removed when it is Explorer, the default) and comes back at startup. A bad saved value is ignored with a console error. The vertical rail keeps its own state (`workbench.activity.pinnedViewlets2`, `workbench.activity.placeholderViewlets`, `workbench.activity.viewletsWorkspaceState`), and no code that copies between the two was found, so each orientation starts from its own pins.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle: `pyk` (load with the defaults), `oQn` (save), `ePt.openPaneComposite` and `ePt.ACTIVE_VIEWLET_SETTINGS_KEY`, class `ore` (the vertical rail's keys); V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts 62-64 (the stock keys)
- Pieces:
  1. Pinning, unpinning and reordering are remembered across restarts and apply to every window and workspace on the machine, because they are saved under `sidebar2.sidebarData.memoized.v1` in application scope.
  2. The open container is saved per workspace and comes back at startup, with Explorer, the default, saved as no value.
  3. A bad saved value is ignored with an error in the console.
  4. The vertical rail keeps its own pins, so each orientation starts from its own pins and order.
