Group: AX-2 Where it sits and how wide. Section: Auxiliary bar. Date: 2026-10-25. Pass: 2.
Features: 4.

### AX-2.1 The part sits opposite the primary side bar and flips with it
- Tag: both
- Evidence: source
- Boundary: no
- What: The part has no side setting of its own. It always shows on the opposite side of the editor from the primary side bar, and when the side bar moves left or right the part moves the other way. There is no way to put both on the same side.
- Commands: workbench.action.toggleSidebarPosition; workbench.action.moveSideBarRight; workbench.action.moveSideBarLeft
- Menus: none found
- Keys: none found
- Settings: workbench.sideBar.location
- Cursor: present (setting registered with the same text, "The secondary side bar will show on the opposite side of the workbench", NLS 4213; the three commands found with Primary Side Bar labels, NLS 3085-3087)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (566-572); V\src\vs\workbench\browser\layout.ts (636-660, 1965-2020); V\src\vs\workbench\browser\actions\layoutActions.ts (97-171); C bundle (`workbench.sideBar.location` registration, `toggleSidebarPosition` class)

### AX-2.2 Swap the two side bars from the part's own menus
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Right-click the part's tab strip or its container title and pick Move Secondary Side Bar Left or Move Secondary Side Bar Right. The label flips with the current side. It runs the same command as the primary side bar swap, so both bars trade places.
- Commands: workbench.action.toggleSidebarPosition
- Menus: ViewContainerTitleContext; tab strip extra context actions (built in `auxiliaryBarPart.ts`, not a MenuId)
- Keys: none found
- Settings: workbench.sideBar.location
- Cursor: missing (neither label is in the NLS table or the bundle; the primary side bar rows "Move Primary Side Bar Right" and "Move Primary Side Bar Left" are present)
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (209-230); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (233-261); C bundle (container title context items for the primary side bar only)

### AX-2.3 Resize the part by dragging its edge
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag the edge toward the editor to change the width. Stock allows 170 pixels at the narrowest, drags shut if you pull below that, and a double-click on the edge sets a best-fit width that is never under 300. The size is saved and starts at 300 or a quarter of the window, whichever is smaller. Cursor raises the narrowest width to 300, the best-fit floor to 540, and turns snapping shut off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (`get minimumWidth(){return 300}`, `Math.max(t,540)` for the preferred width, `get snap(){return!1}`; saved-size key `auxiliaryBar.size` with a default of 200 in its key table; an extra Cursor width key `cursor/layout/editor.auxiliaryBar.width`)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (52-79); V\src\vs\workbench\browser\parts\paneCompositePart.ts (113-117); V\src\vs\base\browser\ui\grid\grid.ts (719); V\src\vs\workbench\browser\layout.ts (1823, 2827, 2970-2975); C bundle (aux bar part class, state key table)

### AX-2.4 The part takes the Side Bar colors, with a border toward the editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Stock paints the part with the Side Bar background, foreground and border theme colors (`sideBar.background`, `sideBar.foreground`, `sideBar.border`, with the contrast border as a fallback) and draws a 1 pixel border only on the edge facing the editor, with the tab strip using the Panel title colors in the title row and the Activity Bar top colors in a top or bottom strip, while Cursor paints the background with the editor background, takes the border color from a Cursor stroke color, and draws the border only under certain layout conditions.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (`updateStyles` sets the editor background color variable, falls back to a Cursor stroke color variable (`cursor-stroke-primary`) for the border, and ties the border to the editor and chat layout flags)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (181-199, 219-228); C bundle (aux bar part `updateStyles`)

## Handed off
- Increase and Decrease Current View Size, and the width and height variants (`workbench.action.increaseViewSize`, `increaseViewWidth` and others). They resize this part when it has focus: SB-1.4.
- Focus the neighbor part (`workbench.action.navigateLeft`, `navigateRight`, `navigateUp`, `navigateDown`) and Focus Next or Previous Part. They list this part among the parts. No group owns part-to-part navigation, so the closest is SB-1.3.
- Experimental Modern UI floating cards (`workbench.experimental.modernUI`) cover the side bars and the panel together: PN-1.22.
- Side bar swap commands and title bar swap rows (primary side bar labels, title bar right-click menu): SB-1.2 and TB-4.9.

## Unclear
- Cursor's saved size key has a static default of 200 but its minimum width is 300. I did not find which one wins on a fresh profile.
- Cursor has extra layout code for the Agents sidebar (`workbench.parts.unifiedsidebar`) and a no-title-bar layout that touches this part's width and border. That is agent window layout and was not read.
- The experimental stock setting `workbench.secondarySideBar.forceMaximized` also changes the first-run width (half the window, at least 300). It is listed under AX-3.5.
