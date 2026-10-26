Group: AB-2 Customizing the rail. Section: Activity bar. Date: 2026-10-25. Pass: 2.
Features: 11.

Reading note: the Cursor field records the stock vertical rail, which Cursor builds only when `workbench.activityBar.orientation` is `vertical`. Cursor's default is `horizontal`, which hides the stock rail and draws its own tab row (AB-6, owned elsewhere). Where the default row behaves differently, the What line says so. None of these features has a setting of its own. Their state is saved per profile in storage keys `workbench.activity.pinnedViewlets2` and `views.customizations`, not in settings.json.

### AB-2.1 Hide or keep an icon from its menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click an icon and choose Hide 'Name' to take it off the rail or Keep 'Name' to put it back, where Hide is greyed out for the last icon left, and hiding the open view opens the default view (Explorer) or the next icon instead; Cursor's horizontal tab menu has the same Hide and Keep entries.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\workbench\browser\parts\compositeBar.ts; C bundle (action id `show.toggleCompositePinned`, labels NLS 3325 and 3326, enabled only while more than one icon is pinned)

### AB-2.2 Show or hide icons from the rail's background menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-clicking empty space on the rail opens a checklist of every available container where a tick means it is on the rail, ticking one adds it and opens it, unticking removes it, and the last ticked item cannot be unticked; in Cursor's horizontal row the checklist sits at the bottom of the tab menu, and empty row space offers only Move Primary Side Bar Right or Left.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts; C bundle (`getContextMenuActions` building the checked list with pin and unpin, identical to stock)

### AB-2.3 Reorder icons by dragging
- Tag: both
- Evidence: source
- Boundary: no
- What: Dragging an icon up or down the rail shows a drop line above or below the target depending on which half you hover, dropping on empty rail space puts it first or last, and the new order is remembered; Cursor's horizontal tabs are not draggable and only the rows of its chevron list reorder.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\workbench\browser\parts\compositeBar.ts; C bundle (icon registered as a draggable with `type:"composite"`, `dragged-over-head` and `dragged-over-tail` classes, same drop handler)

### AB-2.4 Hold a dragged item over an icon to open it
- Tag: both
- Evidence: source
- Boundary: no
- What: While dragging a container or view, resting over a rail icon for 0.8 seconds opens that icon's view so you can see where the drop will land; Cursor's horizontal row has no such handler in its tab code.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\base\browser\dnd.ts (`DelayedDragHandler`, 800 ms); C bundle (the same drag-over timer class with `},800)`)

### AB-2.5 Drag a container from another bar onto the rail
- Tag: both
- Evidence: source
- Boundary: no
- What: Dragging a view container's tab from the Panel or Secondary Side Bar onto the rail and dropping it at the wanted spot moves the whole container into the primary side bar and opens it; no drop target for dragged containers was found in Cursor's horizontal row code (not run).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (`CompositeDragAndDrop.drop`, `CompositeBarDndCallbacks`); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (`moveViewContainerToLocation`); C bundle (`moveViewContainerToLocation(...,"dnd")` in the same drop handler)

### AB-2.6 Drag a single view onto the rail to give it its own icon
- Tag: both
- Evidence: source
- Boundary: no
- What: Dragging one view by its header onto the rail creates a new container that holds only that view, places its icon where you dropped it and opens it, and only views that allow moving can be dragged; no drop target for views was found in Cursor's horizontal row code (not run).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts; V\src\vs\workbench\browser\parts\views\viewPaneContainer.ts (view header is draggable); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (`moveViewToLocation`); C bundle (`moveViewToLocation(s,this.targetContainerLocation,"dnd")` with the `canMoveView` check)

### AB-2.7 Move a container with Move To
- Tag: both
- Evidence: source
- Boundary: no
- What: In an icon's menu, Move To lists Primary Side Bar, Secondary Side Bar and Panel (leaving out where it already is) and sends the whole container to the end of that bar (or back to its default spot when that bar is its home) and opens it; Cursor's horizontal tab menu has no Move To entry.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts; C bundle (`moveToMenu` submenu over locations 0, 2, 1, labels NLS 3920 and 3923 to 3925, `createMoveAction`)

### AB-2.8 Reset Location
- Tag: both
- Evidence: source
- Boundary: no
- What: Reset Location shows in an icon's menu only when its container, or the single view it holds, is away from its default home, and it sends it back and opens it; Cursor's horizontal tab menu reaches the same action through the container title menu.
- Commands: `<container id>.resetViewContainerLocation` (one per container, title menu variant)
- Menus: ViewContainerTitleContext (group 1_viewActions)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts; V\src\vs\workbench\services\views\browser\viewDescriptorService.ts; C bundle (`resetLocationAction` in the rail menu code, `${e.id}.resetViewContainerLocation` with `Pe.ViewContainerTitleContext`, labels NLS 3921, 3922 and 13433)

### AB-2.9 Configure Keybinding for a container
- Tag: both
- Evidence: source
- Boundary: no
- What: The first entry of an icon's menu, shown only when the container has an open command, opens the Keyboard Shortcuts editor filtered to that command so you can assign or change its key; Cursor's horizontal tab menu has the same entry.
- Commands: workbench.action.openGlobalKeybindings
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\platform\actions\common\menuService.ts (`createConfigureKeybindingAction`); C bundle (helper `configureKeybinding/` with label NLS 1785 calling `workbench.action.openGlobalKeybindings` with `@command:`)

### AB-2.10 New containers appear on the rail by themselves
- Tag: both
- Evidence: source
- Boundary: no
- What: A view container that has not been seen before (for example one added by an extension, or a new one made by dragging a view out) is pinned automatically, so its icon appears on the rail without any setup.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts (`onDidRegisterViewContainers`); C bundle (`onDidRegisterViewContainers` pins any container missing from the cache)

### AB-2.11 The rail is restored from a saved copy at startup
- Tag: both
- Evidence: source
- Boundary: no
- What: Which icons are pinned, their order, and their names and icons are saved per profile and drawn at once when the window opens, before extensions finish loading, with placeholder actions standing in until the real containers register.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts (cached view containers, `PlaceHolderViewContainerActivityAction`); V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts (storage key names); C bundle (storage keys `workbench.activity.pinnedViewlets2`, `workbench.activity.placeholderViewlets`, `workbench.activity.viewletsWorkspaceState`, placeholder action classes)

## Handed off

- Hide Badge and Show Badge in the same icon menu: AB-3.6.
- Accounts toggle in the rail menu: AB-4.
- Activity Bar Position and Activity Bar Size in the rail menu: AB-5.
- Menu and Hide Menu entries in the rail menu: TB-1.
- Move Primary Side Bar Right or Left and Hide Primary Side Bar in the rail menu: SB-1.
- View: Reset View Locations, Move View, and the side bar header menu: SB-1.
- Dropping a rail icon onto another container's view area, onto the Panel tab bar, or onto the Secondary Side Bar: SB-1, PN-1, AX-5.
- Cursor's horizontal tab row menu, pin icons in the chevron list, and reorder inside that list: AB-6.

## Unclear

- Cursor's rail icon menu adds chat-only entries (Rename Chat, Open as Editor, Close Chat, Close Other Chats) for chat containers and filters out `workbench.panel.aichat.*` items from the shared part of the menu. That is agent and chat surface, so it is not cataloged here.
- Cursor's Move To still lists Secondary Side Bar in code, but Cursor blocks contributing view containers to that bar (NLS 2890, "reserved for Cursor's agent UI"). What happens when you pick it was not run.
- Cursor's horizontal row keeps its own order and pin state under `sidebar2.sidebarData.memoized.v1` (application scope), apart from the stock keys. Switching orientation therefore shows two different saved layouts. Not checked on screen.
- Stock 1.144.0 adds only an optional `transformContextMenuActionsForComposite` hook in `paneCompositeBar.ts` and no new customizing feature.
