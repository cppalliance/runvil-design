Group: AX-4 Tab strip and title buttons. Section: Auxiliary bar. Date: 2026-10-25. Pass: 2.
Features: 10.

### AX-4.1 Switch containers with the tab strip
- Tag: both
- Evidence: source
- Boundary: no
- What: The title row shows one tab per view container in the part. Click a tab to open that container, or move between tabs with the arrow keys and press Enter or Space. The active tab has an underline in the Panel title colors. Hovering a tab shows its name and key. The strip is a tab list that screen readers announce as "Active View Switcher".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (searched for the label "Active View Switcher", NLS 3321, and the `tablist` role; found. The Cursor strip is shown or hidden by the Agents sidebar state, not by a setting, and its tab chips are styled for chat tabs)
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (286-320); V\src\vs\workbench\browser\parts\compositeBarActions.ts (418); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (201-231, 263-284); C bundle (composite bar `create`, aux bar `shouldShowCompositeBar`)

### AX-4.2 Show tabs as text or icons
- Tag: both
- Evidence: source
- Boundary: no
- What: Tabs show as text labels by default. Turn the setting off, or right-click the strip and pick Show Icons, to show icons only. Pick Show Labels to switch back. The choice is ignored when the strip sits at the top or bottom of the part, which always uses icons. In Cursor the setting still works, but the menu row is gone and labels are always allowed.
- Commands: workbench.action.auxiliarybar.toggleShowLabels
- Menus: tab strip extra context actions (built in `auxiliaryBarPart.ts`, not a MenuId)
- Keys: none found
- Settings: workbench.secondarySideBar.showLabels
- Cursor: changed (the setting is registered, NLS 4220, and `canShowLabels` is always true; the `auxiliarybar.toggleShowLabels` id has 0 hits)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (163-170, 247-252); V\src\vs\workbench\browser\workbench.contribution.ts (618-623); C bundle (aux bar `resolveConfiguration`)

### AX-4.3 Put the strip at the top or bottom of the part, or hide it
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Set the Activity Bar location to `top`, `bottom` or `hidden` and the part's tab strip follows. With `default` the tabs sit in the title row. With `top` or `bottom` the tabs move to their own strip and the title row shows the container name. With `hidden` the tabs are gone. The right-click menu has an Activity Bar Position submenu with the same four choices. Cursor keeps the setting but this part pins the position to `default`.
- Commands: workbench.action.activityBarLocation.default; workbench.action.activityBarLocation.top; workbench.action.activityBarLocation.bottom; workbench.action.activityBarLocation.hide
- Menus: ActivityBarPositionMenu; ViewContainerTitleContext; CommandPalette
- Keys: none found
- Settings: workbench.activityBar.location
- Cursor: changed (the setting is registered and its text still names the Secondary Side Bar, NLS 4222-4226, but `resolveConfiguration` returns the fixed position `default`; the four command ids, `ActivityBarPositionMenu` and the label "Activity Bar Position" have 0 hits)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (163-179, 286-294); V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts (483-601); V\src\vs\workbench\browser\workbench.contribution.ts (650-662); C bundle (aux bar `resolveConfiguration`)

### AX-4.4 Hide the strip when only one container is left
- Tag: vscode
- Evidence: source
- Boundary: no
- What: With the strip at the top or bottom, turn on Auto Hide and the strip disappears when the part holds a single visible container, and returns when a second one appears. It has no effect with the strip in the title row.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.autoHide; workbench.activityBar.location
- Cursor: missing (`workbench.activityBar.autoHide` has 0 hits and the visible-containers tracker is not in the Cursor part)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (133-161, 268-281); V\src\vs\workbench\browser\workbench.contribution.ts (663-668); C bundle (0 hits)

### AX-4.5 Right-click menu on the tab strip
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click the strip or the title row to get a list of the part's containers with check marks to show or hide each tab (the last tab cannot be hidden). Stock adds a Views submenu (strip in the title row), Activity Bar Position, Move Secondary Side Bar Left or Right, Show Icons or Show Labels, and Hide Secondary Side Bar. Right-clicking a single tab adds the per-tab entries (Hide or Keep, Hide Badge, Configure Keybinding, Move To, Reset Location). Cursor keeps the check-mark list and the per-tab entries but drops all the stock extras.
- Commands: workbench.action.toggleSidebarPosition; workbench.action.toggleAuxiliaryBar
- Menus: tab strip extra context actions (built in `auxiliaryBarPart.ts`, not a MenuId); ActivityBarPositionMenu; ViewsSubMenu
- Keys: none found
- Settings: none found
- Cursor: changed (the strip menu builder and the labels "Hide '{0}'" and "Keep '{0}'", NLS 3325-3326, are present; the Cursor part passes an empty `fillExtraContextMenuActions:t=>{}`)
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (654-686); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (233-261); V\src\vs\workbench\browser\parts\compositeBarActions.ts (640-666); V\src\vs\workbench\browser\parts\paneCompositePart.ts (758-805); C bundle (composite bar `getContextMenuActions`, aux bar options)

### AX-4.6 Reorder tabs by dragging
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag a tab sideways and drop it before or after another tab to change the order. A colored marker shows where it will land. Dropping a container from another bar between tabs, or dropping a dragged view onto a tab, places the new tab at that spot and opens it. The part-wide drop target is AX-5.1.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (searched for `horizontallyBefore`, `getTargetIndex` and `canDrop` in the drag handler, all found; the drop marker color is set in the Cursor strip options. Whether it works in the running app is unverified)
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (36-138, 319-320); V\src\vs\workbench\browser\parts\paneCompositeBar.ts (114-118); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (227); C bundle (composite drag and drop class)

### AX-4.7 Reach hidden tabs from the Additional Views button
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When the tabs do not all fit, the strip shows an Additional Views button at the end. Click it for a list of the tabs that are cut off, and pick one to open it. The tab you have open is kept in the strip when it can fit.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (the label "Additional Views", NLS 3323, and the overflow code are present for the other bars, but this part passes `scrollable:!0`, so its width limit is unlimited and it never overflows; see AX-4.8)
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (530-652); V\src\vs\workbench\browser\parts\compositeBarActions.ts (449-513); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (217-218); C bundle (composite bar size computation)

### AX-4.8 Scroll the tab strip sideways
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor's strip for this part does not wrap or cut off tabs, it scrolls sideways with a thin 3 pixel bar that fades in on hover, it scrolls the active tab into view when you switch to it, and its tab chips are about 60 to 120 pixels wide with a fade on long text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only (searched for `scrollable:!0`, `composite-bar-scrollable` and `autoScrollActiveItemIntoView`, all found; the CSS hides the bar until the strip is hovered)
- Source: C bundle (aux bar `getCompositeBarOptions`, composite bar `create` and `autoScrollActiveItemIntoView`); C workbench.desktop.main.css (`.auxiliarybar .composite-bar-scrollable`)

### AX-4.9 Step to the previous or next view in the part
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Previous Secondary Side Bar View or Next Secondary Side Bar View from the Command Palette to open the neighboring container in the strip. It wraps around at the ends. There is no default key.
- Commands: workbench.action.previousAuxiliaryBarView; workbench.action.nextAuxiliaryBarView
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (both ids and the labels, NLS 3311 and 3312, found)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (194-214); V\src\vs\workbench\browser\parts\compositeBarActions.ts (760-791); C bundle (two classes extending the switch action)

### AX-4.10 Title row buttons fold into an overflow menu when the part is narrow
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. When the part is narrow, the buttons on the right of its title row collapse into a "..." menu, starting from the left-most button. The title label is also capped at 40 percent of the width.
- Commands: none found
- Menus: AuxiliaryBarTitle
- Keys: none found
- Settings: none found
- Cursor: missing (`getToolbarResponsiveBehavior` and `overflowFrom` have 0 hits in the bundle)
- Source: N\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (164-174); N\src\vs\workbench\browser\parts\auxiliarybar\media\auxiliaryBarPart.css (30-43); C bundle (0 hits)

## Handed off
- Title row buttons: Maximize or Restore is AX-3.3 and the Hide X button is AX-1.2. The other buttons in the row are each view's own toolbar and belong to that view's group.
- Per-tab entries on the single-tab right-click menu (Hide or Keep a tab, Hide Badge and Show Badge, Configure Keybinding, Move To, Reset Location): AB-2.1, AB-3.5, AB-2.9, AB-2.7 and AB-2.8 describe the same shared code on the rail. The Move To entry that sends a container into this part is AX-5.2.
- The Views submenu in the strip menu (show or hide the views of the open container): SB-1.9.
- Tab badges, their colors and the 1K+ shortening: AB-3.
- Cursor's chat title buttons (New Agent, Chat History, More Actions, Close, Show Agents Side Bar) and the agent tab styling: out of scope, chat.
- Previous and next view for the primary side bar and the panel: AB-1.10 and PN-1.15.

## Unclear
- Cursor adds a menu id `AuxiliaryBarHeader` for a `.auxiliary-bar-global-header` toolbar. I found no items registered into it, so I could not say what shows there.
- Cursor's tab right-click menu with its extras removed: the base list and the per-tab entries are read from code only. Whether the running app shows them was not checked.
- Right-clicking the title row when the strip is at the top or bottom shows the open container's own context menu instead of the strip menu (`onTitleAreaContextMenu`). I did not catalog that menu, because each container supplies its own entries.
- AX-4.7 and AX-4.8 are one stock behavior and one Cursor replacement for it. I split them so the tag stays honest.
