Group: AB-6 Pinned container tab row. Section: Activity bar. Date: 2026-10-25. Pass: 2.
Features: 14.

### AB-6.1 Choose between the tab row and the vertical rail
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Set `workbench.activityBar.orientation` to `horizontal` (the default: no rail, and the Primary Side Bar starts with the pinned tab row) or `vertical` (a stock style 48 px rail plus the older stock side bar). It sits in the Settings editor under Workbench > Appearance with the text "Controls the orientation of the activity bar in the workbench. Quit and reopen for changes to take effect." No command, menu item or key flips it, and the choice is read once at startup. In horizontal mode a container with one view also keeps that view's own header instead of merging it into the container title.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.orientation
- Cursor: cursor only
- Source: C bundle: setting definition `[D5e]:{type:"string",enum:["horizontal","vertical"],default:"horizontal"}` (NLS 4227); part chooser in class `UEa` (builds `JDt` for vertical and `ePt` for horizontal); layout members `activityBarDirection`, `isActivityBarHidden`, `getDummyActivityBarView`; Settings editor group `workbench/appearance` (NLS 8750); searched for a toggle command (`ctivityBarOrientation`, `ctivity Bar Orientation`): 0 hits

### AB-6.2 Restart prompt after changing the orientation
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: When the orientation value changes, a confirm dialog says "A setting has changed that requires a restart to take effect." with the detail "Press the restart button to restart {product name} and enable the setting." and a Restart button that restarts the app (the web build says reload instead). It shows only while the window has focus and only when the value really changed. If you decline, the saved value stays and the layout switches at the next start.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.orientation
- Cursor: cursor only
- Source: C bundle: settings change relauncher class `fri` (its `fri.SETTINGS` list includes the orientation setting, and `activitybarOrientation.handleChange` tracks it), NLS 8923, 8925, 8927 (native) and 8924, 8926, 8928 (web); stock counterpart V\src\vs\workbench\contrib\relauncher\browser\relauncher.contribution.ts (same prompt for other settings, no orientation)

### AB-6.3 See the pinned container tab row at the top of the side bar
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: In the default layout the Primary Side Bar starts with a 35 px row of icon-only tabs, one per pinned container, centered with a 2 px gap. Each tab is 26 px square with a 16 px icon and no text, tooltip or badge. The open container's tab gets the selected style (selected background, active panel title color, and a contrast outline in high contrast themes); hover uses the list hover background. A first run pins Explorer, Search, Source Control and Extensions. Run and Debug, Testing and every other container live only in the chevron list until pinned.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle: class `ePt` (part `workbench.parts.sidebar`, minimum size 214 x 77), `$yk` (the row), `b6m` (one tab), `pyk` (default pins `workbench.view.explorer`, `workbench.view.search`, `workbench.view.scm`, `workbench.view.extensions`); CSS classes `.sidebar2-item-icon-anysphere` and `.sidebar-list-item-selected` in `workbench.desktop.main.css`

### AB-6.4 Open a container by clicking its tab
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Click a tab to open that container in the side bar and move focus into it. Clicking the tab that is already open does nothing, because the row never reads `workbench.activityBar.iconClickBehavior` and has no double click guard. The tab, chevron and list code has no tab stops, arrow keys, ARIA roles or hover tooltips, so the row is mouse only. The Show Explorer, Show Search and similar commands still open the same containers.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle: `$yk` select handler (opens the container only when it differs from the active one, then closes the list), `ePt.openPaneComposite`; searched the whole row, chevron and list code for keydown, keyup, tabindex, role, aria and title attributes: 0 hits

### AB-6.5 Tabs that do not fit leave the row
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The row shows only as many pinned tabs as fit: the row width divided by 40 px, rounded down, minus one slot for the chevron, and minus one more when the open container is not pinned. At the 214 px minimum width that is 4 tabs, or 3 with an extra tab. A pinned tab that does not fit has no tab and can be reached only from the list. The width is watched live, so widening the side bar brings tabs back.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle: `$yk` (`Math.min(pinnedCount,Math.floor(width/40)-1-(extraTab?1:0))` and a resize observer on the row)

### AB-6.6 The open container always shows as a tab
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: When the open container is not pinned, the row adds one extra selected tab at its end for that container, so the current view is always visible. Switching to a pinned container removes the extra tab. Right-clicking the extra tab opens the same tab menu as any other tab.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle: `$yk` (finds the active container among all containers when it is not pinned and renders it as one more `b6m` tab with `selected` true)

### AB-6.7 Open the container list with the chevron
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A chevron button after the tabs (pointing down when closed, up and highlighted when open) opens a list under the row, lying over the view content with a soft shadow. It has one row per container in the side bar: icon, name, the container's keyboard shortcut at half opacity when it has one, and a pin icon. Pinned containers come first and the open container's row is highlighted. Click a row to open that container. The list closes after a pick, when the open container changes, or on a second click on the chevron; no click outside or Escape handling was found. Containers marked hide when empty that have no active views are not listed. The second title row of Run and Debug is hidden while the list is open.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle: `$yk` (chevron button, open and close state), `Hyk` (the list), `y6m` (one row), `ePt.getViewContainers` (the hide when empty filter, and a re-query 1 s and 11 s after containers register); CSS classes `.sidebar2-item-icon-anysphere-hover` and `.sidebar-list-item`

### AB-6.8 Pin or unpin a container
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: In the list, click a row's pin icon to add or remove that container's tab. The icon is hidden on unpinned rows until the row is hovered, and the click does not open the row. In the tab right-click menu, Hide '{name}' unpins and Keep '{name}' pins, and the checklist at the bottom of the menu toggles any container, with a check on pinned ones. The menu entries are disabled for the last pinned container, but the list pin icon has no such guard, so the list can unpin every container. The Agents container always counts as pinned. Every change is saved.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle: `y6m` (pin icon click handler), `$yk` tab menu builder (`togglePin` item and the checklist, both guarded by `pinnedViewContainerIDs.length>1`), `Wke` (pinned test, always true for `workbench.view.backgroundAgent`), `oQn` (save), NLS 3979-3980; CSS `.sidebar-2-pinned-icon-not-pinned` and `.sidebar-list-item:hover .sidebar-2-pinned-icon`

### AB-6.9 Reorder containers by dragging in the list
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Drag a list row up or down to change its place. A row moves only among rows of its own kind (pinned among pinned, unpinned among unpinned). The dragged row drops to half opacity and a copy follows the pointer. The new place is stored as a number halfway between its neighbors, and the tab order follows the pinned order. The list is ordered pinned first, then by saved order, then by the container's own order, then by a fixed tiebreak computed from the container id. The tabs in the row cannot be dragged, and no drop handling for a view or container dragged in from another place was found.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle: `$yk` (drag end handler, `viewContainerOrders` with `order` and `secondaryOrder`, and the sort), `Uyk` (draggable list row, opacity 0.5 while dragged), `Hyk`, `b6m` (tab, no drag code), `oQn` (save)

### AB-6.10 Right-click a tab for its menu
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Right-click a tab (the extra tab too) to open a menu with, in order: Configure Keybinding (when the container has a key id), Hide or Keep '{name}', then the container's title-context items (a show or hide toggle for each of its views, Reset Location when the container was moved from its home, and Hide Primary Side Bar), a separator, and a checklist of every container. Not offered, though the stock rail icon menu has them: Hide Badge and Show Badge, Move To, Activity Bar Position, Activity Bar Size, the Accounts and Menu toggles, and Move Primary Side Bar Right or Left. Right-click on list rows does nothing.
- Commands: workbench.action.toggleSidebarVisibility; `<viewId>.toggleVisibility` (pattern, one per view); `<containerId>.resetViewContainerLocation` (pattern)
- Menus: ViewContainerTitleContext
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle: `$yk` menu builder and its contextmenu handler, NLS module `vs/workbench/browser/parts/sidebar/sidebarMenu` (NLS 3977-3984), the `ViewContainerTitleContext` registrations in Cursor (Hide Primary Side Bar NLS 3101, Hide Secondary Side Bar NLS 3310, view toggles group `1_toggleVisibility`, Reset Location NLS 13433 group `1_viewActions`); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts 831-921; V\src\vs\workbench\browser\actions\layoutActions.ts 340-352; V\src\vs\workbench\browser\actions\layoutActions.ts 185-225 (the four Move Side Bar items that Cursor does not register on this menu)

### AB-6.11 Configure a container's keybinding from its tab
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The first item in the tab menu, Configure Keybinding, shows the container's current key after a tab character (or only the label when none is set) and opens the Keyboard Shortcuts editor filtered to that container's open command. It shows only for containers that have a keybinding id. The stock rail icon menu has the same item; the tab row has its own copy.
- Commands: workbench.action.openGlobalKeybindings
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle: `$yk` menu builder (`configureKeybinding` item), NLS 3977-3978; V\src\vs\workbench\browser\parts\compositeBarActions.ts 637-685 (stock counterpart on the rail)

### AB-6.12 Right-click empty row space to move the side bar
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Right-click on blank space in the row (not on a tab) to get one item, Move Primary Side Bar Right (or Left when the bar is already on the right), with a tooltip that repeats the same words. It flips the side bar to the other side.
- Commands: workbench.action.toggleSidebarPosition
- Menus: none found
- Keys: none found
- Settings: workbench.sideBar.location
- Cursor: cursor only
- Source: C bundle: `$yk` blank-space contextmenu handler (acts only when the click target is the row itself), NLS 3981-3984

### AB-6.13 Run and Debug gets a second title row
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: While Run and Debug is open, the title grows from 35 px to 70 px and a second centered row shows that container's title actions: the Start Debugging button with the launch configuration picker, the Open 'launch.json' gear, and an "Actions..." overflow menu for secondary ones. The row is hidden while the chevron list is open. Only the Run and Debug container asks for it (`wantsActionToolbar`, an option stock does not have). Other containers show no container-level toolbar in the tab row, because their views keep their own headers.
- Commands: workbench.action.debug.start; workbench.action.debug.configure
- Menus: ViewContainerTitle
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle: the `workbench.view.debug` container registration with `wantsActionToolbar:!0`, `$yk` (title height 35 or 70), `Xyk` (the `title-actions` toolbar with `toggleMenuTitle:"Actions..."`); V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts 171-224 (the stock title items); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (container registration; 0 hits for `wantsActionToolbar` anywhere under V\src)

### AB-6.14 Keep pins and order between sessions
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Pinning, unpinning and reordering save `{pinnedViewContainerIDs, viewContainerOrders}` as JSON under `sidebar2.sidebarData.memoized.v1` (application scope, machine target), so they apply to every window and workspace on the machine. The open container is saved per workspace under `workbench.sidebar.activeviewletid` (removed when it is Explorer, the default) and comes back at startup. A bad saved value is ignored with a console error. The vertical rail keeps its own state (`workbench.activity.pinnedViewlets2`, `workbench.activity.placeholderViewlets`, `workbench.activity.viewletsWorkspaceState`), and no code that copies between the two was found, so each orientation starts from its own pins.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle: `pyk` (load with the defaults), `oQn` (save), `ePt.openPaneComposite` and `ePt.ACTIVE_VIEWLET_SETTINGS_KEY`, class `ore` (the vertical rail's keys); V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts 62-64 (the stock keys)

## Handed off
- Previous and Next Primary Side Bar View and Focus Activity Bar go to AB-1. In Cursor the two cycle commands walk every container and ignore pins (`getPinnedPaneCompositeIds` returns all of them), and Focus Activity Bar is enabled only for the vertical layout.
- Badges go to AB-3. The tab row draws none: `showActivity` is a no-op and the tab has no badge element, so AB-3 should record Cursor as missing for the row.
- Reset Location and Move To for a container go to AB-2. The tab menu carries Reset Location only; Move To is missing there.
- Showing or hiding a single view, and a one-view container keeping its own header in horizontal mode, go to SB-1. So do Hide Primary Side Bar and Close Primary Side Bar (`workbench.action.closeSidebar`).
- The contents of the Run and Debug title actions go to SB-5.
- The update card that the same sidebar module draws ("Update to v...", Later, Install Now, Changelog, and the "New update available" and "New in" cards) goes to NT-6.

## Unclear
- The always-pinned Agents container tab (`workbench.view.backgroundAgent`) is out of scope as an agent surface. It affects this group in two places: it counts as pinned in the pin test, and it can change the top padding of the side bar in a no-title-bar layout. It was not cataloged.
- The pass 1 Cursor file says the tab menu includes Move Primary Side Bar Right or Left items from `ViewContainerTitleContext`. I found no such registration in Cursor: the only items on that menu are Hide Primary Side Bar, Hide Secondary Side Bar, the per-view toggles and Reset Location, and the V key `move side bar right` is not in `nls.keys.json`. Move Primary Side Bar Right or Left appears only in the blank space menu (AB-6.12).
- No tooltip was found in the tab code, and none of this was run, so hover behavior is unconfirmed.
- Whether the tab menu looks exactly as described in a running app is unconfirmed; the menu was read from code and never opened.
- Cursor Settings (Cursor's own settings screen) was not checked for an orientation entry. The Settings editor entry is derived from the setting id and group list.
- Pixel values (35, 70, 26, 16, 2, 40, 214) are read from code and CSS, not measured on screen.
