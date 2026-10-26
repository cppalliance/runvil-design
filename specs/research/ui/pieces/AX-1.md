Group: AX-1 Show, hide and focus. Pass: 3. Date: 2026-10-25.

### AX-1.1
1. Pressing `Ctrl+Alt+B` (`Cmd+Alt+B` on Mac) while the secondary side bar is hidden shows it and reopens the container used last, or the first container that has views.
2. Pressing the same key while the part is shown hides it, closes its active container, and moves focus to the panel or the editor.
3. View > Appearance > Secondary Side Bar and the row in the Layout Control drop-down run `workbench.action.toggleAuxiliaryBar`, and the Appearance row shows a check mark while the part is open.
4. Toggling the part makes screen readers announce "Secondary Side Bar shown" or "Secondary Side Bar hidden". [stock only]
5. When the part opens with no container active, the toggle creates a new agent chat in it, and the key and command are turned off while a separate auxiliary window has focus. [cursor only]

### AX-1.2
1. Running Hide Secondary Side Bar (`workbench.action.closeAuxiliaryBar`) from the Command Palette closes the part, and the command is disabled while the part is already hidden. [stock only]
2. Clicking the X button in the part's title row closes the part, and the button shows only when `workbench.activityBar.location` is `default`. [stock only]
3. Right-clicking the container title and picking Hide Secondary Side Bar closes the part and never opens it.
4. Hiding the part while it is maximized first leaves maximized mode and then closes the part.

### AX-1.3
1. Running Focus into Secondary Side Bar (`workbench.action.focusAuxiliaryBar`) from the Command Palette puts the keyboard cursor in the active container of the part.
2. Running it while the part is hidden shows the part first and then moves focus into it.
3. The command is listed in the Command Palette under the View category and has no default key.
4. The command is turned off while a separate auxiliary window has focus. [cursor only]

### AX-1.4
1. Setting `workbench.secondarySideBar.defaultVisibility` set to `visibleInWorkspace` (the default) shows the part the first time a folder or workspace opens, and leaves it hidden in an empty window. [stock only]
2. Setting it to `visible` shows the part the first time any window opens, with or without a folder. [stock only]
3. Setting it to `hidden` keeps the part hidden the first time a workspace or window opens. [stock only]
4. The setting is ignored and the part starts hidden when `workbench.startupEditor` is the agent sessions page. [stock only]
5. The setting only affects a workspace or window opened for the first time, and later opens use the saved layout. [stock only]

## Unclear

- AX-1.4 piece 5 follows the setting description ("opened for the first time") and the `isNew` check in the layout code. How an empty window counts as first-time was not traced.
