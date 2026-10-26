Group: AX-5 Moving views and containers in. Section: Auxiliary bar. Date: 2026-10-25. Pass: 2.
Features: 3.

### AX-5.1 Drop a view or container onto the part
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag a view header, a container tab, or the container title label over the part. It shows a drop color (`editorGroup.dropBackground`) when the drop is allowed. An empty part says "Drag a view here to display." Dropping a container moves all its views in, and dropping a view creates a new tab for it and opens it. Cursor does not register this part-wide drop target for this part and hides the empty message. Dropping onto a tab in its strip is AX-4.6, and the drag side of this is SB-1.12 and SB-1.13.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (the label "Drag a view here to display.", NLS 3926, is present, but `createEmptyPaneMessage` hides the message and returns before the drop target is registered when the part is the auxiliary bar)
- Source: V\src\vs\workbench\browser\parts\paneCompositePart.ts (252-330, 368-380); V\src\vs\workbench\browser\parts\compositeBar.ts (36-138); C bundle (`createEmptyPaneMessage`); C nls.messages.json (3926)

### AX-5.2 Send a container into the part with Move To
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click the tab of a container in the activity bar, the panel or the strip, choose Move To, then Secondary Side Bar. The container goes to the end of this part's tabs and opens. The Move To list leaves out the place the container is already in. Reset Location sends it back to its default home. Cursor still lists Secondary Side Bar in Move To, but what happens when you pick it was not run.
- Commands: none found
- Menus: tab right-click menu (built in `paneCompositeBar.ts`, not a MenuId); ViewContainerTitleContext
- Keys: none found
- Settings: none found
- Cursor: present (searched for the labels "Move To", "Reset Location" and "Secondary Side Bar", NLS 3920-3925, and the ids `moveViewContainerTo`, 15 hits; all found)
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts (157-216); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (354-360, 898-922); C bundle (Move To builder); C nls.messages.json (3920-3925)

### AX-5.3 Move all panel views to the secondary side bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Move Panel Views To Secondary Side Bar to move every container in the panel into this part in one step. The part is shown, and the container that was open in the panel opens here if the part had none. A hidden twin command, `workbench.action.movePanelToSidePanel`, does the same.
- Commands: workbench.action.movePanelToSecondarySideBar; workbench.action.movePanelToSidePanel
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (both ids and the label "Move Panel Views To Secondary Side Bar", NLS 3966-3967, found; the reverse move is PN-1.19)
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (344-397); C bundle (shared move class with the two ids); C nls.messages.json (3966-3967)

## Handed off
- Move View and Move Focused View (the pickers offer New Secondary Side Bar Entry, id `_.auxiliarybar.newcontainer` in Cursor): SB-1.14.
- Reset View Locations and Reset Focused View Location: SB-1.16 and SB-1.17.
- Reset Location and Move To on a rail icon: AB-2.7 and AB-2.8. AX-5.2 is the same code seen from this part, kept here because the destination is the part.
- Move Secondary Side Bar Views To Panel: PN-1.19.
- Open View picker, which lists a "Secondary Side Bar" group (`workbench.action.openView`): QA-4.
- View menu group `4_auxbar`, for containers whose default home is this part. No stock built-in extension has one: TB-1.
- Secondary Side Bar row in the Move Focused View picker and in the Customize Layout picker: SB-1.14 and TB-4.7.

## Unclear
- AX-5.4 is reachable only through an extension, which the scope puts out. The catalog text for AX-5 names the extension contribution point, so I listed it. The parent can cut it.
- Whether a view dragged in by hand renders in Cursor is unverified. The part's title row and tab strip depend on chat layout flags (`shouldRenderTitleArea`, `shouldShowCompositeBar`), so a moved container may behave differently from stock.
- In Cursor the part-wide drop target is skipped (AX-5.1). I did not check that the strip's own drop handler still accepts a drop from another bar.
- No built-in extension in V declares `secondarySidebar` (searched every `package.json`, per the pass 1 notes), so in stock the part holds only Chat until you move something in or an extension adds one.
