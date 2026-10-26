Group: TB-4 Layout controls. Section: Title bar. Date: 2026-10-25. Pass: 2.
Features: 19.

### TB-4.1 Show or hide the layout controls
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn the whole layout control area in the title bar on or off, from the setting or from the "Layout Controls" row in the title bar right-click menu.
- Commands: toggle.workbench.layoutControl.enabled
- Menus: TitleBarContext (group 2_config, order 4); TitleBarTitleContext (group 2_config, order 4)
- Keys: none found
- Settings: workbench.layoutControl.enabled
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts:75-79; V\src\vs\workbench\browser\workbench.contribution.ts:777-784; C bundle (module titlebarActions strings 4016-4017, setting text 4249, getter `layoutControlEnabled` in the title bar part)

### TB-4.2 Choose the layout control style (menu, toggles, or both)
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick whether the title bar shows one Configure Layout drop-down (`menu`), separate toggle buttons (`toggles`), or both (`both`). Default is `both` in stock and in Cursor.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.layoutControl.type
- Cursor: present
- Source: V\src\vs\workbench\browser\workbench.contribution.ts:785-796; C bundle (module workbench.contribution, strings 4250-4253, default `both` read from the registration)

### TB-4.3 Toggle the primary side bar from a title bar button
- Tag: both
- Evidence: source
- Boundary: no
- What: Click a panel-shaped button to hide or show the primary side bar. The icon follows the side bar position and its on or off state. In Cursor the button sits in the left toolbar when the Agents side bar is on the right (the left-toolbar copy is hidden during onboarding), otherwise in the right toolbar.
- Commands: workbench.action.toggleSidebarVisibility
- Menus: LayoutControlMenu (V group navigation, order 0 or 2; Cursor group 2_pane_toggles, order 0 or 2); LayoutControlMenuSubmenu (group 0_workbench_layout, order 0)
- Keys: Ctrl+B (Cmd+B on macOS)
- Settings: workbench.sideBar.location; workbench.layoutControl.type
- Cursor: changed (button placement comes from fixed left and right toolbar lists; key and command are disabled while an auxiliary window has focus; no screen reader announcement on toggle)
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:282-391; C bundle (class for workbench.action.toggleSidebarVisibility with keybinding and precondition on isAuxiliaryWindowFocusedContext; toolbar lists S6w and y6w; string 4036)

### TB-4.4 Toggle the panel from a title bar button
- Tag: both
- Evidence: source
- Boundary: no
- What: Click a panel-shaped button to hide or show the bottom panel; in Cursor the button is always in the right toolbar, and the command first leaves the maximized layout that hides the editor if that is active.
- Commands: workbench.action.togglePanel
- Menus: LayoutControlMenu (V group navigation, order 1; Cursor group 2_pane_toggles, order 1); LayoutControlMenuSubmenu (group 0_workbench_layout, order 4)
- Keys: Ctrl+J (Cmd+J on macOS)
- Settings: workbench.layoutControl.type
- Cursor: changed (extra maximized-chat handling and same auxiliary-window key guard as the side bar toggle)
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts:34-70,320-342; C bundle (class with keybinding Ctrl+J and unified maximize handling; string 3948)

### TB-4.5 Toggle the secondary side bar from a title bar button
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Stock shows a second panel-shaped button that hides or shows the secondary side bar, and Cursor puts this button neither on the layout menu nor in the right toolbar.
- Commands: workbench.action.toggleAuxiliaryBar
- Menus: LayoutControlMenu (group navigation, order 0 or 2); LayoutControlMenuSubmenu (group 0_workbench_layout, order 1)
- Keys: Ctrl+Alt+B (Cmd+Alt+B on macOS)
- Settings: workbench.layoutControl.type
- Cursor: missing
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts:30-80,140-180; C bundle (LayoutControlMenu appendMenuItems list, toolbar builder T6w)

### TB-4.6 Open the Configure Layout drop-down
- Tag: both
- Evidence: source
- Boundary: yes
- What: When the style is `menu`, a single layout button opens a list with Primary Side Bar, Secondary Side Bar and Panel toggles (checked when shown) and Customize Layout... at the end. Boundary reason: one row of the drop-down (Secondary Side Bar) opens or creates the agent chat pane in Cursor.
- Commands: workbench.action.toggleSidebarVisibility; workbench.action.toggleAuxiliaryBar; workbench.action.togglePanel; workbench.action.customizeLayout
- Menus: LayoutControlMenu (group 1_workbench_layout, submenu); LayoutControlMenuSubmenu
- Keys: none found
- Settings: workbench.layoutControl.type
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:172-182; V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts:50-66; C bundle (submenu item with `config.workbench.layoutControl.type` equals `menu`; toggleAuxiliaryBar class calls ensureChatVisibleOrCreate)

### TB-4.7 Open Customize Layout
- Tag: both
- Evidence: source
- Boundary: yes
- What: A picker titled Customize Layout with sections for Visibility (Menu Bar, Primary Side Bar, Secondary Side Bar, Panel, Status Bar, and in stock also Activity Bar), side bar position, panel alignment, quick input position, and modes (Full Screen, Zen Mode, Centered Layout). In stock it also has its own title bar button when the style is `both`. In Cursor it is only in the Configure Layout drop-down and the Command Palette, and the Activity Bar row is gone. Boundary reason: the Secondary Side Bar row in Cursor shows or hides the agent chat pane.
- Commands: workbench.action.customizeLayout
- Menus: LayoutControlMenu (group navigation, or group 1_layout when editor actions live in the title bar); LayoutControlMenuSubmenu (group z_end); CommandPalette
- Keys: none found
- Settings: workbench.layoutControl.type
- Cursor: changed (no standalone button; no Activity Bar row)
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:1315-1375,1417-1450,1517-1560; C bundle (customizeLayout class with only the z_end submenu entry; item list vEr; strings 3134-3140, 3155, 3208)

### TB-4.8 Restore layout defaults from Customize Layout
- Tag: both
- Evidence: source
- Boundary: no
- What: The Restore Defaults button at the top of the Customize Layout picker puts the activity bar location, side bar location, status bar, panel location and menu bar visibility back to their defaults, and resets panel alignment to center and quick input to top.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.location; workbench.sideBar.location; workbench.statusBar.visible; workbench.panel.defaultLocation; window.menuBarVisibility
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:1538-1606; C bundle (nls key `restore defaults` in layoutActions; reset helper reads `inspect(id).defaultValue`; exact list of reset settings not decoded)

### TB-4.9 Move the side bar from the title bar right-click menu
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: Right-click empty space in the left title bar toolbar to get one row, Move Side Bar Right or Move Side Bar Left, that swaps the primary side bar to the other side. Boundary reason: moving the primary side bar also moves the Agents side bar to the opposite side (the command stores an agent location override).
- Commands: workbench.action.toggleSidebarPosition
- Menus: none found
- Keys: none found
- Settings: workbench.sideBar.location
- Cursor: cursor only
- Source: C bundle (title bar part method `showLeftActionsContextMenu`, strings 4029-4030; command class for toggleSidebarPosition with the `SIDEBAR_LOCATION_AGENT_OVERRIDE` store); V has the same command only in View > Appearance, not in the title bar

### TB-4.10 Layout buttons move into the editor tab strip
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: In Cursor's no-title-bar layout the side bar and panel buttons can render at the right end of the top-right editor tab strip instead of in the title bar. They go there when the right edge has no visible side bar and the editor area or panel is the right-most part, and they stay in the title bar otherwise.
- Commands: none found
- Menus: LayoutControlMenu
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (functions B_a and r_S, method getLayoutActionsPositioningMode, CSS classes `in-titlebar`, `in-editor-tabs`, `layout-actions-container`; inputs include the chat editor group storage key `cursor/chatEditorGroupEnabled`)

### TB-4.11 Open the Switch Layout overlay
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: Press Ctrl+Alt+Tab to open a full-screen overlay of tiles. Tab, Left and Right move the selection, Enter or a click applies it, Escape closes it, and letting go of the modifier keys applies the selected tile. The tile that matches the current layout starts selected. Boundary reason: the tiles are agent layouts and apply agent and chat pane visibility.
- Commands: workbench.action.openLayoutSwitcher
- Menus: CommandPalette (category View)
- Keys: Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module layoutSwitcherOverlay, strings 3213-3217 and layoutActions string 3166; component tpS; CSS classes `layout-switcher-overlay`, `__backdrop`, `__container`, `__tile`, `__icon-wrapper`, `__label`, `__separator`; the `__hint` and `__shortcut` classes are in the CSS only and the component does not create them)

### TB-4.12 Apply the Right layout (the Editor tile)
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: Pick the tile named Editor in the overlay (the preset is named Layout - Right elsewhere in the code). It hides the Agents side bar and the panel, shows the chat pane, the editor area and the primary side bar, and puts the Agents side bar location on the right. Boundary reason: the preset hides the Agents side bar and shows the chat pane.
- Commands: workbench.action.openLayoutSwitcher; workbench.action.toggleSidebarVisibility; workbench.action.togglePanel; workbench.action.toggleAuxiliaryBar
- Menus: none found
- Keys: Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (preset id `default-editor` in lists pEr and hEr with state in mva; apply function dpS; strings 2999 and 3214; storage key `cursor/agentLayout.quickMenu.lastSelectedLayoutId`)

### TB-4.13 Switch to another open window from the overlay
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Other open Cursor windows (not auxiliary windows, not the current one) show first in the overlay as tiles named after their workspace folder, then a separator, then the layout tiles. Choosing one focuses that window. The number of tiles is capped by the screen width.
- Commands: workbench.action.openLayoutSwitcher
- Menus: none found
- Keys: Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component tpS reads nativeHostService.getWindows with includeAuxiliaryWindows false and calls focusWindow; helper ipS names the tile; string 3215-3217 for the Untitled fallback)

### TB-4.14 Apply a saved custom layout from the overlay
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: Layouts saved in storage show as extra tiles after Agent and Editor, with a small preview icon drawn from the saved pane visibility. They are skipped when no folder or workspace is open, and any whose id or name clashes with a default tile is dropped. No control that creates, renames or deletes a layout was found in the IDE window code. Boundary reason: a saved layout stores agent, chat, editor, panel and side bar visibility, and part widths.
- Commands: workbench.action.openLayoutSwitcher
- Menus: none found
- Keys: Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (functions kbm and rpS read storage key `cursor/agentLayout.quickMenu.customLayouts`; tpS drops customs when the workbench state is empty)

### TB-4.15 Zen layout preset
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: A preset named Layout - Zen that shows only the editor area (Agents side bar, chat pane, panel and primary side bar all hidden). It is defined in the preset list, but no tile or button in the IDE window applies it (see Unclear). Boundary reason: the preset hides the Agents side bar and the chat pane.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (preset id `default-zen` in list hEr, one reference only; string 3000)

### TB-4.16 Browser layout preset
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: A preset named Browser with the editor area and chat pane shown and the Agents side bar, panel and primary side bar hidden. It is defined in the preset list, but no tile or button in the IDE window applies it (see Unclear). The preset has no field that opens a browser. Boundary reason: the preset shows the chat pane and hides the Agents side bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (preset id `default-browser` in list hEr, one reference only; string 3001)

### TB-4.17 New workspaces open in the last chosen layout
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: When a workspace has no saved side bar or panel visibility, Cursor applies the layout last picked in the overlay (the Editor layout if none) as its first-open layout. It always starts with the Agents side bar hidden and the chat pane shown, and the editor area starts hidden only when the auto-hide-editor-when-empty flag is on. Boundary reason: it decides whether the Agents side bar and chat pane start visible.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (method applyUnifiedWorkspaceLayout, functions hbm and LhS, storage keys `cursor/agentLayout.quickMenu.lastSelectedLayoutId`, `workbench.sideBar.hidden`, `workbench.panel.hidden`)

### TB-4.19 Keep an auxiliary window always on top
- Tag: vscode
- Evidence: source
- Boundary: no
- What: In a popped-out editor window the layout area holds a pin button that turns always-on-top on or off for that window.
- Commands: workbench.action.enableWindowAlwaysOnTop; workbench.action.disableWindowAlwaysOnTop; workbench.action.toggleWindowAlwaysOnTop
- Menus: LayoutControlMenu (group navigation, order 1, auxiliary windows only); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts:490-575; C bundle (searched the three ids and the labels Turn On Always on Top and Turn Off Always on Top, no hits)

### TB-4.20 Choose layout density in Customize Layout
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. When the experimental Modern UI is on, Customize Layout gets a Layout Density section with Default and Compact, and Restore Defaults also resets it.
- Commands: workbench.action.setLayoutDensity.default; workbench.action.setLayoutDensity.compact
- Menus: LayoutDensityMenu
- Keys: none found
- Settings: window.density.layout; workbench.experimental.modernUI
- Cursor: missing
- Source: N\src\vs\workbench\browser\actions\layoutDensityActions.ts; N\src\vs\workbench\browser\actions\layoutActions.ts (LayoutDensityActions); C bundle (searched `window.density.layout` and `setLayoutDensity`, no hits)

## Handed off

- Panel alignment rows (Left, Right, Center, Justify) in Customize Layout: PN-1
- Primary side bar position rows (Left, Right) in Customize Layout and the commands moveSideBarLeft and moveSideBarRight: SB-1
- Quick input position rows (Top, Center) in Customize Layout: QA-6
- Mode rows in Customize Layout, Zen Mode and Centered Layout: ED-1 (judgment call, no group owns them clearly); Full Screen: TB-3
- Visibility rows for Menu Bar, Activity Bar and Status Bar in Customize Layout: TB-1, AB-5, SS-1
- The show and hide commands themselves (Ctrl+B, Ctrl+J, Ctrl+Alt+B and the View > Appearance rows): SB-1, PN-1, AX-1. Only their title bar buttons are in TB-4.
- The title bar right-click menu as a whole (Command Center, Navigation Controls, Editor Actions, Menu Bar rows): TB-2. Only the Layout Controls row is in TB-4.1.
- Focus Title Bar (`workbench.action.focusTitleBar`, string 4042 in Cursor): TB-3

## Unclear

- Zen and Browser presets (TB-4.15, TB-4.16): the only list the Switch Layout overlay uses (`pEr`) holds Agent and Editor, and the apply function only looks in that list plus saved custom layouts. `default-zen` and `default-browser` appear once each, in the list used to match the current layout and to pick the first-open layout. No visible control for them was found in the IDE window. The Agents window bundle was not read, so it may apply them. Parent should decide whether to keep them in TB-4.
- Preset side field: pass 1 read `sidebarLocation` in the presets as the primary side bar side. The apply code passes it to `setUnifiedSidebarLocation` and the current-state reader returns the opposite of the primary side bar position, so it is probably the Agents side bar side. TB-4.12 is written that way. Medium confidence.
- Compact mode and Always on Top (TB-4.18, TB-4.19): pass 1 described them under TB-3. They are registered on LayoutControlMenu, so they are listed here. TB-3 may list them too. Dedupe when merging.
- Whether `workbench.layoutControl.enabled` set to false also hides the left-toolbar side bar button in Cursor. That button is built from a fixed list, not from LayoutControlMenu. Not verified.
- Cursor keeps a no-op row, Open Layout Settings Menu (`workbench.action.openLayoutSettingsMenu`, gear icon, group 6_workbench_toggles). It only swallows the click and no visible effect was found, so it is not listed.
- The Modern UI experiment (`workbench.experimental.modernUI`, floating side bars and panel) is a whole-workbench look and not a title bar control, so it is not a feature here. Cursor has 0 hits for it.
- When the no-title-bar layout is active (it decides whether TB-4.10 can happen) was not decoded.
- Screen reader announcements on toggle: only the side bar toggle body was read in the Cursor bundle (no announcement). The panel toggle body was read only in part.
- Skipped as out of scope, as instructed: Toggle Agents Side Bar, Maximize Chat Size, Swap Agent Sidebar Location, Layout - Left (the Agent tile), Agents Window Button.
