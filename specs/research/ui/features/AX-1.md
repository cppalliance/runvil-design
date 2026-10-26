Group: AX-1 Show, hide and focus. Section: Auxiliary bar. Date: 2026-10-25. Pass: 2.
Features: 4.

### AX-1.1 Toggle the secondary side bar open or closed
- Tag: both
- Evidence: source
- Boundary: yes
- What: Press Ctrl+Alt+B, or use View > Appearance > Secondary Side Bar, to show or hide the part. Showing it reopens the container you used last (or the first container that has views), and hiding it also closes the active container and moves focus to the panel or the editor. Stock tells screen readers "Secondary Side Bar shown" or "hidden". Boundary: in Cursor, when the part opens with no container active, the toggle creates a new agent chat in it, and it sends `ai_pane.opened` and `ai_pane.closed` events.
- Commands: workbench.action.toggleAuxiliaryBar
- Menus: MenubarAppearanceMenu; LayoutControlMenuSubmenu
- Keys: Ctrl+Alt+B (Cmd+Alt+B on Mac); in Cursor the key and the command are off while a separate auxiliary window has focus
- Settings: none found
- Cursor: changed (id, label, key and both menu rows found; the action opens or creates the agent chat, is disabled in an auxiliary window, and no "Secondary Side Bar shown" or "hidden" alert string exists in the bundle)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (30-84); V\src\vs\workbench\browser\layout.ts (2253-2292); C bundle (class with `Toggle Secondary Side Bar Visibility`); C nls.messages.json (3305-3308)

### AX-1.2 Hide the part without toggling
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Hide Secondary Side Bar from the Command Palette, click the X button in the part's title row, or right-click the container title and pick Hide Secondary Side Bar. All three close the part and never open it. The X button only shows when `workbench.activityBar.location` is `default`. Hiding while the part is maximized first leaves maximized mode.
- Commands: workbench.action.closeAuxiliaryBar; workbench.action.toggleAuxiliaryBar
- Menus: AuxiliaryBarTitle; ViewContainerTitleContext; CommandPalette
- Keys: none found
- Settings: workbench.activityBar.location
- Cursor: changed (the container title menu row "Hide Secondary Side Bar" is present, NLS 3310; `workbench.action.closeAuxiliaryBar`, the X button and the `auxiliarybar-close` icon have 0 hits)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (86-110, 181-191); V\src\vs\workbench\browser\layout.ts (2253-2256); C bundle (`ViewContainerTitleContext` item with the toggle id); C nls.messages.json (3310)

### AX-1.3 Move keyboard focus into the part
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Focus into Secondary Side Bar to show the part if it is hidden and put the cursor in the active container. There is no default key.
- Commands: workbench.action.focusAuxiliaryBar
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (id and label "Focus into Secondary Side Bar" found; the Cursor action is off while an auxiliary window has focus and does nothing in the Agents window)
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (112-139); C bundle (class with `workbench.action.focusAuxiliaryBar`); C nls.messages.json (3309)

### AX-1.4 Choose whether a new workspace starts with the part open
- Tag: vscode
- Evidence: source
- Boundary: no
- What: In Settings, pick `hidden`, `visibleInWorkspace` (the default), or `visible` to decide if a workspace or window opened for the first time shows the part. The two `maximized` values belong to AX-3.4. The setting is skipped when the startup editor is the agent sessions page.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.secondarySideBar.defaultVisibility
- Cursor: missing (the setting id has 0 hits; Cursor decides first-run visibility from its layout presets, which belong to the agent layout)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (597-610); V\src\vs\workbench\browser\layout.ts (2970-3000, 3045-3059); C bundle (0 hits for the id)

## Handed off
- Layout toggle button for the part in the title bar, and the Configure Layout drop-down row: TB-4.5 and TB-4.6.
- Secondary Side Bar row in the Customize Layout picker: TB-4.7.
- Zen Mode hides the part and brings it back on exit: ED-1.34.
- Expand Editor Group and Hide Side Bars, and Maximize Editor Group and Hide Side Bars (both also hide this part): ED-1.19 and ED-1.20.
- Welcome page skipped on startup while the part is maximized: ED-9.
- Chat toggles (`Toggle Chat Pane`, `aichat.close-sidebar` Hide Chat): out of scope, chat.

## Unclear
- Whether closing the only view of the active container also hides the whole part. The stock code hides that container (`viewsService.ts` 336-338) but I did not trace what happens to the part.
- Cursor's Expand and Maximize Editor Group commands exist (NLS 3626 and 3629). I did not confirm that they still hide this part.
- Nothing on the Cursor side was run. "Does nothing in the Agents window" and "off in an auxiliary window" are readings of the code, not observations.
