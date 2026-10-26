Group: AX-2 Where it sits and how wide. Pass: 3. Date: 2026-10-25.

### AX-2.1
1. With the primary side bar on the left, the secondary side bar shows on the right side of the editor.
2. With the primary side bar on the right, the secondary side bar shows on the left side of the editor.
3. Changing `workbench.sideBar.location` between `left` and `right` moves the secondary side bar to the opposite side at once.
4. Running Toggle Primary Side Bar Position (`workbench.action.toggleSidebarPosition`) from the Command Palette, or the Move Primary Side Bar Right and Move Primary Side Bar Left commands (`workbench.action.moveSideBarRight`, `workbench.action.moveSideBarLeft`) from their menu rows, moves the primary side bar and so moves the secondary side bar the other way.
5. No setting or command puts both side bars on the same side of the editor.

### AX-2.2
1. Right-clicking the part's tab strip shows a row that reads Move Secondary Side Bar Left or Move Secondary Side Bar Right, and the label names the side the part would move to. [stock only]
2. Right-clicking the container title shows the same Move Secondary Side Bar row, and the label flips with the current side. [stock only]
3. Picking the row runs `workbench.action.toggleSidebarPosition`, so the primary and secondary side bars trade places. [stock only]

### AX-2.3
1. Dragging the part's edge toward or away from the editor changes the part's width.
2. The width is saved, and a fresh layout starts at 300 px or a quarter of the window width, whichever is smaller.
3. Dragging the edge below 170 px closes the part by snapping it shut. [stock only]
4. Double-clicking the edge sets a best-fit width for the open container that is never under 300 px. [stock only]
5. The narrowest width is 300 px, a double-click on the edge sets a best-fit width that is never under 540 px, and dragging smaller never snaps the part shut. [cursor only]

### AX-2.4
1. The part is painted with `sideBar.background` and its text uses `sideBar.foreground`. [stock only]
2. A 1 px border in `sideBar.border`, or in the contrast border color when the theme sets no border, shows only on the edge of the part that faces the editor. [stock only]
3. In the title row the tab strip uses the Panel title colors, and in a strip at the top or bottom of the part it uses the Activity Bar top colors.
4. The part is painted with the editor background, its border takes the Cursor stroke color `cursor-stroke-primary`, and the border shows only under certain layout conditions. [cursor only]

## Unclear

- AX-2.4 piece 3 is read from the stock part code only. I did not check whether the Cursor tab strip keeps the same color wiring, since Cursor restyles the strip for chat tabs.
- AX-2.1 piece 3 follows the setting text and the layout code. I did not run a live switch.
