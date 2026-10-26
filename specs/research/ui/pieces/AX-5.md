Group: AX-5 Moving views and containers in. Pass: 3. Date: 2026-10-25.

### AX-5.1
1. Dragging a view header, a container tab, or a container title label over the part shows a drop color (`editorGroup.dropBackground`) over the part when the drop is allowed. [stock only]
2. An empty part shows the message "Drag a view here to display." [stock only]
3. Dropping a whole container on the part moves all of its views into the part. [stock only]
4. Dropping a single view on the part creates a new tab for it and opens it. [stock only]

### AX-5.2
1. Right-clicking a container tab in the activity bar, the panel, or this part's strip shows a Move To submenu with the Primary Side Bar, Secondary Side Bar, and Panel rows, leaving out the place the container is in now.
2. Picking Secondary Side Bar in Move To moves the container to the end of this part's tabs and opens it.
3. Reset Location on the tab menu sends a moved container back to its default home and opens it, and the row shows only while the container is away from its default home.
4. Right-clicking a container title also shows Reset Location while the container is away from its default home.

### AX-5.3
1. Running Move Panel Views To Secondary Side Bar (`workbench.action.movePanelToSecondarySideBar`) from the Command Palette moves every container in the panel into the part in one step.
2. The part is shown if it was hidden, and the container that was open in the panel opens here if the part held no containers before the move.
3. The hidden command `workbench.action.movePanelToSidePanel` does the same thing and does not appear in the Command Palette.

## Unclear

- AX-5.2 in Cursor: Secondary Side Bar is still listed in Move To, but what happens after you pick it was not run, so the pieces carry no Cursor tag.
- AX-5.4 is reachable only through an extension, which is out of scope. The parent can cut it.
