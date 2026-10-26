Section: Title bar (TB). Date: 2026-10-25. Pass: 1.

# Title bar in Cursor 3.27.12 (C)

This file records what Cursor does for each title bar group (TB-1 to TB-6, defined in the V file for this section), what Cursor adds, what it removes, and what I left out. The menu bar item list with Cursor status per row is in the entry-points file.

## How I searched

- Bundle: `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app\out\vs\workbench\workbench.desktop.main.js`. On disk it is 41,558,918 bytes (the step-zero notes say 39.6 MB, so the install was probably updated since). `package.json` says version 3.27.12. I did not run the app. Everything here is strings and minified code, so the evidence level for Cursor facts is `bundle`.
- Strings: the bundle calls `k(1234, null)` for text, where the number is a position in `out\nls.messages.json`. `out\nls.keys.json` lists the module and key for each position, in the same order. I joined the two to get the module name and English text for every call (13,625 keys over 1,169 modules). This is how I found module names such as `vs/workbench/contrib/ui/browser/buttons/IdeTitlebarUpdateButton` and `vs/workbench/browser/actions/agentLayoutQuickMenu/agentLayoutQuickMenuTypes`. Those names do not appear as text in the bundle itself, only in `nls.keys.json`.
- Menus: I parsed every `.MenubarXxx` and title bar menu reference in the bundle with the same scratch parser used on V, then compared by label, group and order. Counts: Cursor has V's count plus 2 for most menus (one for the `new MenuId(...)` definition and one for the property), so most menus match V item for item.
- Context key `isGlass` (`new RawContextKey("isGlass", ...)`) is true in the Agents window. Several menu items are hidden there. The title bar I describe is the IDE window (`isGlass` false).
- `isAgent` in the title bar state is the constant `true` in `computeToolbarState` (so the "agent layout" rules always apply in this build), which is why many title bar buttons below are agent features.

## TB-1 Menu bar (Cursor)

- Tag: both. Evidence: bundle.
- Same eight top-level menus plus Preferences. Strings `mFile`, `mEdit`, `mSelection`, `mView`, `mGoto`, `mTerminal`, `mHelp` are in `vs/workbench/browser/parts/titlebar/menubarControl` (idx 3996-4002). `Run` is registered by the debug module with the title "&&Run" and, like Selection and Go, only shows when `isGlass` is not true.
- Top-level Preferences is titled "Preferences" without a mnemonic (V: "&&Preferences").
- Menu changes: see the entry-points file. In short, Help loses 12 stock rows (including Welcome, Open Walkthrough... and Editor Playground), File loses New File..., View loses Chat and Browser, Appearance loses Custom Title Bar and Activity Bar Position, Terminal loses New Terminal Window, and the whole Settings Sync set is gone. Cursor adds Cursor Settings, VS Code Settings, Give Feedback..., Open Extension Monitor, Open Browser (Appearance), two "Add Symbol to ... Chat" rows in Go, and New Agents Window in File.
- Settings: `window.menuBarVisibility` (20 hits), `window.enableMenuBarMnemonics` (2), `window.customMenuBarAltFocus` (1) are present. `window.menuStyle` has 0 hits (V has it).
- Custom menu bar update rows are the same as V except `update.cancelling` (string `cancellingUpdate` is not in `menubarControl` strings). `Focus Application Menu` (`workbench.actions.menubar.focus`) is present.
- Native menu strings (`vs/platform/menubar/electron-main/menubar`, 35 keys) match V's structure: app menu, Window menu, update rows. Not run on macOS.

## TB-2 Command center and window title (Cursor)

- Tag: both. Evidence: bundle.
- Present: the command center (`command-center` appears 44 times in the bundle), the strings "Search", "Search {0} ({1}) ..." and "Command Center" (module `commandCenterControl`, idx 3990-3995, same as V), Back (order 1) and Forward (order 2) on `MenuId.CommandCenter`, Quick Open (`workbench.action.quickOpenWithModes`) on `CommandCenterCenter`, and the debug toolbar submenu.
- Settings present: `window.commandCenter`, `window.titleSeparator`, `window.title` (the default template code is the same as V: `${activeEditorShort}${separator}${rootName}${separator}${profileName}` on native macOS, the longer form with `${dirty}` and `${appName}` elsewhere, separator " - " or an em dash on macOS), `workbench.navigationControl.enabled` (7 hits).
- Not found: the `Share...` row on the command center, and V's `Agents` control entry.
- Title bar right-click menu (ids `toggle.*`, strings in module `titlebarActions`, idx 4014-4021): Navigation Controls, Layout Controls, Agents Window Button (new, setting `workbench.agentsWindowButton.enabled`, order 5), Editor Actions (order 3), Menu Bar, and Restore Native Title Bar (new, see TB-3). Missing against V: Command Center toggle, Notifications toggle, Integrated Browser toggle, Hide Custom Title Bar and Hide Custom Title Bar In Full Screen.
- When the command center is hidden, Cursor shows Go Back and Go Forward as buttons in its own left toolbar instead (see TB-5).

## TB-3 Window frame and controls (Cursor)

- Tag: both. Evidence: bundle.
- Present: custom window controls (`window-minimize`, `window-max-restore`, `window-close`, `window-controls-container`, `window-appicon`, `titlebar-drag-region` each found), the web overlay class `wco-enabled`, `window.titleBarStyle` (11 hits), `window.controlsStyle`, `window.doubleClickIconToClose`, `window.dialogStyle`, `window.nativeTabs`, `window.nativeFullScreen`, `window.clickThroughInactive`.
- The Cursor text for `window.titleBarStyle` (module `electron-sandbox/desktop.contribution`, idx 12061) says: "Adjust the appearance of the window title bar to be native by the OS or custom. On Linux and Windows, this setting also affects the application and context menu appearances. Changes require a full restart to apply." The V text is shorter and the menu part moved to `window.menuStyle`. This points to Cursor's desktop settings block being older than V's.
- Not found (0 hits): `window.customTitleBarVisibility`, `window.menuStyle`, `window.border`, `update.titleBar`, and the commands `toggle.toggleCustomTitleBar`, `showCustomTitleBar`, `hideCustomTitleBar`, `hideCustomTitleBarInFullScreen`.
- Added: command `toggle.window.titleBarStyle`, title "Restore Native Title Bar", on `TitleBarContext` and `TitleBarTitleContext`, group `4_restore_native_title`, shown when the title bar style is `custom` (the context key is `NJt`, value compared to `"custom"`). Running it sets `window.titleBarStyle` to `native`. The registration is guarded by two platform flags (`Ef&&Q_`), probably "desktop and not web". I did not decode them.
- "No title bar layout" mode: the page body gets the class `no-titlebar-layout` when `isNoTitlebarLayoutActive()` is true (`klt(contextKeyService)`, not decoded). In that mode the CSS makes the title bar a transparent 34 px strip fixed to the top (`position:fixed; height:34px; pointer-events:none; z-index:10000`), hides the window title text (`.titlebar-center>.window-title{display:none}`), gives the left and right toolbars back their clicks, and adds 35 px of top padding to side bar, panel and auxiliary bar. It is part of Cursor's unified layout. I could not determine from the bundle when it is active.
- "Window in window" classes (`.window-in-window`, `.wiw-show-title-bar`) hide the app icon and window controls unless the second class is set. Purpose not verified.

## TB-4 Layout controls (Cursor)

- Tag: both for the stock controls, cursor for the Switch Layout overlay. Evidence: bundle. Boundary: yes, because the layout presets switch agent and chat panes on and off.
- Settings: `workbench.layoutControl.enabled` (9 hits), `workbench.layoutControl.type` (`menu`, `toggles`, `both`, 18 hits). The default value was not read.
- `LayoutControlMenu` in Cursor holds the toggles in group `2_pane_toggles` (V uses `navigation`): Toggle Primary Side Bar (order 0 with the side bar on the left, 2 on the right), Toggle Panel (order 1), plus agent items Toggle Agents Side Bar (`workbench.action.toggleUnifiedSidebar`) and Maximize Chat Size (`workbench.action.maximizeChatSize`), and group `6_workbench_toggles` order 7 Open Layout Settings Menu (`workbench.action.openLayoutSettingsMenu`, a no-op). The Secondary Side Bar toggle is not on this menu. Customize Layout... (`workbench.action.customizeLayout`) is only in the Configure Layout drop-down (`z_end`).
- The title bar does not render this menu as plain buttons. It builds two toolbars from fixed lists with an order number and a visibility rule: a left list (`S6w`) and a right list (`y6w`), then pulls the Primary Side Bar and Panel toggle items out of the menu and slots them in (`T6w`, `E6w`). Orders, right toolbar: Upgrade 0, New Agent 100, Chat History 110, More Actions 120, Panel and Primary Side Bar 200 or 210 (swap with the side bar position), Toggle Agents 220, Cursor Settings 225, Open Agents Window 226, Accounts 900, Manage 910.
- The layout buttons can live in the title bar (`.action-toolbar-container.in-titlebar`) or inside the editor tab strip (`.in-editor-tabs`, inside `.layout-actions-container`), decided by `getLayoutActionsPositioningMode()` from the layout state, experiments and editor groups. The rule was not decoded.
- Right-click on the left toolbar offers Move Side Bar Right or Left (runs `workbench.action.toggleSidebarPosition`; strings `movePrimarySideBarRight` and `movePrimarySideBarLeft`, idx 4029-4030).
- Switch Layout overlay: command `workbench.action.openLayoutSwitcher`, title "Switch Layout", category View, in the Command Palette, key Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS) (module `vs/workbench/browser/actions/layoutActions`, idx 3166). It opens a full-screen overlay (`.layout-switcher-overlay` with `__backdrop`, `__container`, `__tile`, `__label`, `__icon-wrapper`, `__hint`, `__shortcut`, `__separator`; strings `layoutSwitcher.agent` "Agent", `layoutSwitcher.editor` "Editor", `layoutSwitcher.untitled` "Untitled"). Tiles: other open windows first (from `nativeHostService.getWindows`), then layouts. Keys: Tab, Left and Right move, Enter applies, Escape closes, letting go of the modifier applies. Preview icons are drawn with `layout-preview-icon__*` CSS.
- Layout presets (`hEr`, strings in `agentLayoutQuickMenuTypes`, idx 2998-3001):

| Id | Label | Agents | Chat | Editors | Panel | Side bar | Side bar side |
|-|-|-|-|-|-|-|-|
| `default-agent` | Layout • Left | on | on | on | off | off | left |
| `default-editor` | Layout • Right | off | on | on | off | on | right |
| `default-zen` | Layout • Zen | off | off | on | off | off | left |
| `default-browser` | Browser | off | on | on | off | off | left |

  Custom layouts are saved under the storage key `cursor/agentLayout.quickMenu.customLayouts`, and the last pick under `cursor/agentLayout.quickMenu.lastSelectedLayoutId`. Other storage keys for the same feature: `cursor/agentLayout.sidebarVisible`, `sidebarLocation`, `sidebarWidth`, `editorVisible`, `editorWidth`, `panelVisible`, `panelWidth`, `panelHeight`, `auxiliaryBarVisible`, `auxiliaryBarWidth`, `statusBarVisible`, and `workbench.agentLayoutControl.visibilityState`. "Layout • Left" is the agent layout and is out of scope. Right, Zen and Browser are in scope. Custom layouts are not offered when no folder is open.
- The command id `workbench.action.openAgentLayoutQuickMenu` still exists but is now titled "Open Cursor Settings" (gear icon, runs `aiSettings.action.open`). So there is no separate "layout quick menu" in the title bar in this build. The "layout menu" is the Switch Layout overlay above.
- Also in the layout module: `workbench.action.toggleInlineDiffs` ("Toggle Inline Diffs", Command Palette). Inline diff is an in-scope feature owned by the Editor section. Noted here because it is registered next to the layout actions.
- Agent only: `Swap Agent Sidebar Location` (Ctrl or Cmd+E when not the Agents window), Maximize Chat Size (Ctrl or Cmd+Alt+E), `workbench.action.toggleUnifiedSidebar`, `workbench.action.toggleAgents`. Left out.

## TB-5 Title bar buttons and update control (Cursor)

- Tag: both for "an update control exists", cursor for the buttons below. Evidence: bundle. Boundary: yes for Cursor Settings and Upgrade (account and agent dependent), no for Update.
- Not in Cursor: V's `workbench.actions.updateIndicator` (0 hits), the `update.titleBar` setting (0), `MenuId.TitleBarAdjacentCenter` (0), the notification bell and its settings `workbench.notifications.showInTitleBar` and `workbench.notifications.position` (0), the Integrated Browser button and its `workbench.action.browser.*` and `workbench.browser.*` commands and settings (0). Cursor draws its title bar buttons from the two fixed toolbar lists described in TB-4 instead.

### Update control (`IdeTitlebarUpdateButton`)

- Module `vs/workbench/contrib/ui/browser/buttons/IdeTitlebarUpdateButton`, strings idx 11480-11484: label "Update", label while installing "Installing", tooltip "Restart Cursor to install the pending update.", tooltip with version "Restart Cursor to install the pending update to {0}.", and a tooltip while installing that says Cursor will close and reopen automatically.
- Toolbar action: id `cursor.titlebar.installUpdate`, label "Restart to Update" (module `titlebarPart`, idx 4034). It sits in the left toolbar at order 130. A custom view item draws it as a text link button (`button.titlebar-text-link.ide-titlebar-update-button` with `.ide-titlebar-update-button-label` and an SVG spinner `.ide-titlebar-update-button-spinner`).
- When it shows: the update service state is `ready` AND the experiment `ide_update_ux_exp` has the variant `non_sidebar` AND the window is not the Agents window (`ideUpdateAvailable && !isGlassWindow`). So it is gated by a server-side experiment. With other variants Cursor shows the update elsewhere (not checked).
- What it does: on click it stops the event, starts a 300 ms timer, sets the installing state, then runs the command `update.restartToUpdate` with the source `"titlebar"`. If the result is `restartSkipped`, or the shutdown is vetoed (`onShutdownVeto`, `onBeforeShutdownError`), or the command throws, it goes back to "Update". While installing, the button shows the spinner, `data-installing="true"` and `aria-busy="true"`.
- Hover: a compact hover card below the button (`hoverPosition: BELOW`, pointer shown) with the tooltip text.
- Other update entry points kept from V: the Manage menu rows (`update.check`, `update.downloadNow`, `update.install`, `update.restart`, Show Update Release Notes), the custom menu bar Help rows and the macOS native menu rows.

### Other buttons Cursor draws

| Button | Action id | Where and when | Notes |
|-|-|-|-|
| Upgrade | `cursor.titlebar.upgradeToPro` | right toolbar, order 0, shown when the plan is Free and no refresh is pending (`shouldPromptUpgrade`) | label "Upgrade", class `.upgrade-pro-button`, hover "Upgrade for extended limits and full model access", click starts the upgrade checkout flow and logs `titlebar.upgrade_pro_clicked` |
| Cursor Settings (gear) | `workbench.action.openAgentLayoutQuickMenu` (title "Open Cursor Settings"), which runs `aiSettings.action.open` | right toolbar, order 225, always shown | opens the Cursor Settings popup. Keys Ctrl or Cmd+Shift+J, and Ctrl or Cmd+, when not the Agents window. Also Preferences > Cursor Settings |
| Go Back and Go Forward | `workbench.action.navigateBack`, `workbench.action.navigateForward` | left toolbar, orders 200 and 210, shown when `workbench.navigationControl.enabled` is on and the command center is hidden | arrows left and right, enabled from the context keys `canNavigateBack` and `canNavigateForward` |
| Toggle Primary Side Bar | `workbench.action.toggleSidebarVisibility` | left toolbar order 110 when the Agents side bar is on the right, otherwise right toolbar 200 or 210 | icon changes with side and visibility |
| Toggle Panel | `workbench.action.togglePanel` | right toolbar 200 or 210 | uses the menu item icon |

- Agent buttons also in these lists (left out): Toggle Agents, New Agent, Chat History, More Actions, Open Agents Window, and the "Agents Window Button" setting (`workbench.agentsWindowButton.enabled`).
- The title bar action toolbar collapses after 10 items (V: 12) (`overflowBehavior.maxItems: 10`) with the `...` placed at the end.

## TB-6 Accounts and Manage entries (Cursor)

- Tag: vscode for title bar placement. Evidence: bundle. Boundary: no.
- Cursor keeps both menus (`MenuId.AccountsContext`: 5 registrations, `MenuId.GlobalActivity`: 16) and the view-item class for them, the tile actions "Accounts" and "Manage" (module `titlebarActions`, idx 4022-4025), and the visibility storage key `workbench.activity.showAccounts`.
- The Cursor title bar part defines `get activityActionsEnabled(){return!1}`. The right toolbar rule for Accounts (order 900) and Manage (order 910) is `activityActionsEnabled && ...`, so neither is ever drawn in the Cursor title bar. In V they appear when the activity bar is at the top or bottom.
- Accounts menu differences: V has Settings Sync, Cloud Changes, Copilot sign-in, language model access, marketplace sign-in rows. Cursor has Remote Tunnel, extension access and sign-in requests, and a `noAuthenticationProviders` placeholder. See the entry-points file.
- Where Cursor does show them (likely the activity bar bottom, section 03) was not verified.

## Cursor additions in this section

1. Update control `IdeTitlebarUpdateButton` in the left toolbar, experiment gated (TB-5).
2. Upgrade button in the right toolbar for Free plans (TB-5).
3. Cursor Settings gear button and the Preferences > Cursor Settings menu row (TB-5, TB-1).
4. Left and right title bar toolbars built from fixed lists, including Go Back and Go Forward, side bar and panel toggles, with layout buttons that can move into the editor tab strip (TB-4, TB-5).
5. Switch Layout overlay with presets Layout • Right, Layout • Zen, Browser (and Layout • Left, out of scope), custom layouts and saved layout state (TB-4).
6. Restore Native Title Bar command in the title bar right-click menu (TB-3).
7. The `no-titlebar-layout` transparent title bar mode (TB-3).
8. Menu bar additions: Cursor Settings, VS Code Settings, Give Feedback..., Open Extension Monitor, Open Browser (Appearance), and Run, Selection and Go hidden in the Agents window (TB-1).

## Left out (and why)

- New Agent button (id held in a minified constant, label "New Agent"), Chat History (`composer.showComposerHistory`), More Actions (`cursor.titlebar.chatMaximizedMoreActions`), Open Agents Window (`cursor.openOrFocusGlassWindow`, label "Open Agents Window"), Toggle Agents (`workbench.action.toggleAgents`, `workbench.action.toggleUnifiedSidebar`), the "Agents Window Button" setting and context menu toggle: agent window features, out of scope.
- Layout • Left (`default-agent`), Swap Agent Sidebar Location, Maximize Chat Size: agent layout, out of scope.
- File > New Agents Window and Switch to Agents Window (`cursor.openOrFocusGlassWindow`), Go > Add Symbol to Current Chat and Add Symbol to New Chat (`editor.action.addSymbolToChat`, `editor.action.addSymbolToNewChat`): agent and chat features.
- Private Inference Mode (`privateInference.turnOn`, `privateInference.turnOff`, lock icon on `MenuId.TitleBar`, hidden in the Agents window): an inference mode switch, out of scope. It is a real title bar button, so the parent may want to decide.
- Help > Open Extension Monitor and the `application.experimental.extensionMonitor.enabled` setting: extension tooling, out of scope.
- View > Extensions and Preferences > Extensions rows: extension system, out of scope (kept in the inventory for completeness).
- Anything in `workbench.glass.main.js` (the Agents window): not read.

## Newer than 1.128

Nothing in Cursor is newer than 1.128. The stock items that N adds (`MenuId.TitleBarUpdate`, update toolbar, menu motion, 120 character recent labels) are not in the Cursor bundle (`TitleBarUpdate` 0 hits).

## Could not verify

- When `no-titlebar-layout` is active (`klt` was not decoded), and what "window in window" means.
- The rule that moves layout buttons between the title bar and the editor tab strip (`getLayoutActionsPositioningMode`, `B_a`, `r_S` not decoded).
- What the other `ide_update_ux_exp` variants show, and whether the experiment is on for any given user.
- Default values of `workbench.layoutControl.type` and `workbench.navigationControl.enabled` in Cursor (strings only).
- Command ids for Open Extension Monitor and VS Code Settings (minified constants). The Cursor Settings id is `aiSettings.action.open`.
- Whether Accounts and Manage appear in the Cursor activity bar.
- The macOS native menu and the macOS title bar: I only have the strings and the source side, not a running Mac.
- The Agents window title bar (`workbench.glass.main.js`) was deliberately not read.
- Visual details (sizes, colours, icons) were not captured. This pass is about what exists and where it is declared.

