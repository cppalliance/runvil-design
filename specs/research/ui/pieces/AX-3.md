Group: AX-3 Maximize and restore. Pass: 3. Date: 2026-10-25.

### AX-3.1
1. Running Maximize Secondary Side Bar (`workbench.action.maximizeAuxiliaryBar`) from the Command Palette makes the part fill the window and hides the editor, the panel, and the primary side bar. [stock only]
2. Maximizing while the part is hidden shows the part first, and keyboard focus moves into it. [stock only]
3. The command is disabled while the part is already maximized. [stock only]
4. Running Toggle Maximized Secondary Side Bar (`workbench.action.toggleMaximizedAuxiliaryBar`) from the Command Palette maximizes the part, and running it again restores the layout. [stock only]
5. The panel and this part are never maximized at the same time. [stock only]

### AX-3.2
1. Running Restore Secondary Side Bar (`workbench.action.restoreAuxiliaryBar`) while the part is maximized brings back the editor, the panel, and the primary side bar that were visible before maximizing. [stock only]
2. The restored parts and the secondary side bar return at their earlier sizes, and keyboard focus moves back into the part. [stock only]
3. Pressing `Ctrl+W` (`Cmd+W` on Mac), or `Ctrl+F4` on Windows, restores the layout while the part is maximized. [stock only]
4. The command is disabled and the keys are not bound while the part is not maximized. [stock only]

### AX-3.3
1. The first button in the part's title row shows a screen-full icon with the tooltip Maximize Secondary Side Bar, and clicking it maximizes the part. [stock only]
2. While the part is maximized, the same button's tooltip reads Restore Secondary Side Bar, and clicking it restores the layout. [stock only]
3. The button sits before the Hide X button in the title row. [stock only]

### AX-3.4
1. Setting `workbench.secondarySideBar.defaultVisibility` to `maximized` shows the part maximized, with the editor, panel, and primary side bar hidden, the first time any workspace or window opens. [stock only]
2. Setting it to `maximizedInWorkspace` does the same only when a folder or workspace is open, and an empty window follows the normal visibility rules. [stock only]
3. The Welcome page is skipped at startup while the part is maximized. [stock only]

### AX-3.5
1. Setting `workbench.secondarySideBar.forceMaximized` (experimental, off by default) to true makes the part start maximized. [stock only]
2. With the setting on, the part maximizes again each time the last open editor closes. [stock only]
3. Turning the setting on while no editor is open maximizes the part at once, and turning it off while the part is maximized restores the layout. [stock only]
4. With the setting on, a first-run part is half the window wide or 300 px, whichever is larger. [stock only]

### AX-3.6
1. While the part is maximized, showing the editor, the primary side bar, or the panel restores the saved layout first. [stock only]
2. While the part is maximized, opening an editor yourself restores the saved layout first. [stock only]
3. An editor opened automatically, not by your own action, does not end maximized mode. [stock only]
4. Hiding the part while it is maximized restores the saved layout first and then hides the part. [stock only]

## Unclear

none
