Group: AX-4 Tab strip and title buttons. Pass: 3. Date: 2026-10-25.

### AX-4.1
1. The part's title row shows one tab for each view container in the part, and the strip is a tab list that screen readers announce as "Active View Switcher".
2. Clicking a tab opens that container in the part.
3. The arrow keys move focus between tabs, and Enter or Space opens the focused tab.
4. The active tab has an underline drawn in the Panel title colors.
5. Hovering a tab shows its name and its key.
6. In Cursor the tab strip is shown or hidden by the Agents sidebar state instead of a setting. [cursor only]

### AX-4.2
1. Setting `workbench.secondarySideBar.showLabels` to true (the default) shows the tabs as text labels.
2. Setting it to false shows the tabs as icons only.
3. Right-clicking the strip and picking Show Icons switches the tabs to icons, and the row then reads Show Labels to switch back. [stock only]
4. The setting is ignored when the strip sits at the top or bottom of the part, where the tabs always show icons.
5. The setting still switches between labels and icons, the Show Icons and Show Labels row is not in the right-click menu, and labels are always allowed. [cursor only]

### AX-4.3
1. Setting `workbench.activityBar.location` to `default` puts the tabs in the part's title row.
2. Setting it to `top` moves the tabs into their own strip at the top of the part, and the title row shows the container name instead.
3. Setting it to `bottom` moves the tabs into their own strip at the bottom of the part, and the title row shows the container name instead.
4. Setting it to `hidden` removes the tab strip from the part.
5. Right-clicking the strip shows an Activity Bar Position submenu with Default, Top, Bottom, and Hidden, with a check mark on the current choice. [stock only]
6. The Command Palette lists Move Activity Bar to Side, Move Activity Bar to Top, Move Activity Bar to Bottom, and Hide Activity Bar (`workbench.action.activityBarLocation.default`, `.top`, `.bottom`, `.hide`), each only while it is not the current position. [stock only]
7. The setting is kept but this part pins the position to `default`, so the tabs never leave the title row or disappear. [cursor only]

### AX-4.4
1. With the strip at the top or bottom and `workbench.activityBar.autoHide` on, the strip disappears when the part holds a single visible container. [stock only]
2. The strip returns when a second container becomes visible in the part. [stock only]
3. Auto Hide has no effect while the strip sits in the title row. [stock only]

### AX-4.5
1. Right-clicking the strip or the title row lists the part's containers with check marks, and picking a row shows or hides that tab.
2. The last visible tab cannot be hidden from the list.
3. With the strip in the title row, stock adds a Views submenu to the list. [stock only]
4. Stock also adds Activity Bar Position, Move Secondary Side Bar Left or Right, Show Icons or Show Labels, and Hide Secondary Side Bar below a separator. [stock only]
5. Right-clicking a single tab adds the per-tab entries Hide or Keep, Hide Badge, Configure Keybinding, Move To, and Reset Location.

### AX-4.6
1. Dragging a tab sideways and dropping it before or after another tab changes the order of the tabs.
2. A colored marker shows where the dragged tab will land.
3. Dropping a container from another bar between two tabs places its tab at that spot and opens it.
4. Dropping a dragged view onto a tab places a new tab at that spot and opens it.

### AX-4.7
1. When the tabs do not all fit, the strip shows an Additional Views button at its end. [stock only]
2. Clicking the Additional Views button lists the tabs that are cut off, and picking one opens it. [stock only]
3. The tab that is open stays in the strip whenever it can fit. [stock only]

### AX-4.8
1. The strip never wraps or cuts off tabs, and scrolls sideways when the tabs are wider than the part. [cursor only]
2. A thin 3 px scroll bar is hidden until the strip is hovered and then fades in. [cursor only]
3. Switching to a tab scrolls that tab into view. [cursor only]
4. Tab chips are about 60 to 120 px wide, and long text fades out at the end of the chip. [cursor only]

### AX-4.9
1. Running Next Secondary Side Bar View (`workbench.action.nextAuxiliaryBarView`) from the Command Palette opens the next container in the strip.
2. Running Previous Secondary Side Bar View (`workbench.action.previousAuxiliaryBarView`) from the Command Palette opens the previous container in the strip.
3. Both commands wrap around from the last container to the first and from the first to the last.
4. Neither command has a default key.

### AX-4.10
1. Newer than 1.128: when the part is narrow, the buttons on the right of its title row fold into a "..." menu. [stock only]
2. Newer than 1.128: the buttons fold starting from the left-most one. [stock only]
3. Newer than 1.128: the title label is capped at 40 percent of the part's width. [stock only]

## Unclear

- AX-4.4 piece 3 is read from the stock code, where Auto Hide is only checked for the top and bottom positions. It was not run.
- AX-4.5 pieces 3 and 4 repeat rows that AX-4.2, AX-4.3, AX-2.2, and AX-1.2 own, and are listed here only because they are part of this menu.
- AX-4.10 is newer than 1.128, so it is absent from the 1.128 base as well as from Cursor.
