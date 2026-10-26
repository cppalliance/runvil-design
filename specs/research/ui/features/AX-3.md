Group: AX-3 Maximize and restore. Section: Auxiliary bar. Date: 2026-10-25. Pass: 2.
Features: 6.

### AX-3.1 Maximize the secondary side bar
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Run Maximize Secondary Side Bar to make the part fill the window. The editor, the panel and the primary side bar are hidden, and the part shows first if it was hidden. The code remembers which parts were visible and the part's width so they can come back. It is disabled while the part is already maximized. Maximizing the panel and maximizing this part cannot be on together.
- Commands: workbench.action.maximizeAuxiliaryBar; workbench.action.toggleMaximizedAuxiliaryBar
- Menus: CommandPalette; AuxiliaryBarTitle
- Keys: none found
- Settings: none found
- Cursor: missing (searched the bundle and the NLS table for both ids, the labels, the `auxiliaryBarMaximized` context key and the `auxiliaryBar.wasLastMaximized` state key; 0 hits)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (216-239, 269-299); V\src\vs\workbench\browser\layout.ts (2132-2207, 2209-2213); V\src\vs\workbench\common\contextkeys.ts (162); C bundle (0 hits)

### AX-3.2 Restore the layout after maximizing
- Tag: vscode
- Evidence: source
- Boundary: no
- What: While the part is maximized, run Restore Secondary Side Bar or press Ctrl+W (Ctrl+F4 or Ctrl+W on Windows) to bring back the editor, panel and primary side bar that were visible before, at their earlier sizes, and put focus back in the part.
- Commands: workbench.action.restoreAuxiliaryBar
- Menus: CommandPalette
- Keys: Ctrl+W (Cmd+W on Mac); Ctrl+F4 and Ctrl+W on Windows; only while the part is maximized
- Settings: none found
- Cursor: missing (id, label and the `auxiliaryBarMaximized` key have 0 hits)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (241-267); V\src\vs\workbench\browser\layout.ts (2182-2205); C bundle (0 hits)

### AX-3.3 Maximize or restore with the button in the title row
- Tag: vscode
- Evidence: source
- Boundary: no
- What: The first button in the part's title row, with the `auxiliarybar-maximize` screen-full icon, maximizes the part. While maximized its tooltip changes to "Restore Secondary Side Bar" and the same click restores.
- Commands: workbench.action.toggleMaximizedAuxiliaryBar
- Menus: AuxiliaryBarTitle
- Keys: none found
- Settings: none found
- Cursor: missing (command id, icon id and menu row have 0 hits; Cursor's own title buttons are agent buttons, which are left out)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (22, 269-299); C bundle (0 hits)

### AX-3.4 Start a new workspace with the part maximized
- Tag: vscode
- Evidence: source
- Boundary: no
- What: In Settings, set the starting visibility to `maximized` (always) or `maximizedInWorkspace` (only when a folder or workspace is open). The first time a workspace opens, the part is shown and maximized, and the Welcome page is skipped at startup while it is maximized. The other three values of the setting are AX-1.4.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.secondarySideBar.defaultVisibility
- Cursor: missing (the setting id has 0 hits)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (597-610); V\src\vs\workbench\browser\layout.ts (3045-3059, 3086-3092); V\src\vs\workbench\contrib\welcomeGettingStarted\browser\startupPage.ts (118-121); C bundle (0 hits)

### AX-3.5 Keep the part maximized whenever no editor is open
- Tag: vscode
- Evidence: source
- Boundary: no
- What: An experimental setting forces the part to maximize at startup and again each time the last editor closes. Turning it on while no editor is open maximizes at once, and turning it off while maximized restores. With it on, the first-run width is half the window or 300, whichever is larger.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.secondarySideBar.forceMaximized
- Cursor: missing (the setting id has 0 hits)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (611-617); V\src\vs\workbench\browser\layout.ts (365-389, 447-455, 2970-2975); C bundle (0 hits)

### AX-3.6 Maximized mode ends when you bring back another part
- Tag: vscode
- Evidence: source
- Boundary: no
- What: While the part is maximized, showing the editor, the primary side bar or the panel, or opening an editor yourself, restores the saved layout first, an editor opened automatically instead of by your own action does not end maximized mode, and hiding the part also restores first.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing (no maximize state exists to end)
- Source: V\src\vs\workbench\browser\layout.ts (344-363, 1872-1875, 1911-1914, 2050-2057, 2253-2256); C bundle (state key table has `PANEL_WAS_LAST_MAXIMIZED` and no aux bar maximize keys)

## Handed off
- Welcome page skipped on startup while the part is maximized: ED-9.
- Maximize the panel and its title button: PN-1.5.
- Maximize an editor group and Maximize Editor Group and Hide Side Bars: ED-1.19.
- Cursor's own maximize for chat (`workbench.action.maximizeChatSize`, Ctrl+Alt+E, and `toggleExpandAuxiliaryBar`): out of scope, chat.

## Unclear
- AX-3.6 is a reading of the layout code. I did not run stock VS Code to watch each path.
- The stock splash screen remembers the maximized state (`partsSplash.ts`, line 98). It has no menu, key or setting, so it is not listed as a feature.
- The `forceMaximized` setting says it applies "in layouts that support a maximized secondary side bar". I did not find which layouts those are, and I did not check how it behaves when the primary side bar is on the right.
