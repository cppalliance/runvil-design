# Activity bar (AB) - Cursor 3.27.12 findings

Section: Activity bar (AB). Date: 2026-10-25. Pass: 1.

- Scope of this file: what Cursor 3.27.12 (VS Code base 1.128.0) does for the activity bar, compared with the stock file `vscode\03-activity-bar.md`.
- Level: minor groups only. Each group lists up to 4 example features. No pieces yet.
- Evidence is `bundle` unless stated. Bundle means a search of `C\out\vs\workbench\workbench.desktop.main.js` (C = `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`). Minified names such as `ePt` and `D5e` are specific to this build and will change in other builds. NLS numbers are indexes into `C\out\nls.messages.json`; module names come from `C\out\nls.keys.json`.

## The big finding

Cursor ships two layouts and picks one at startup from a new setting, `workbench.activityBar.orientation`.

- `horizontal` (the default): the stock activity bar is not built at all. The side bar draws a row of pinned container icon tabs across its top (this section owns that row, group AB-6).
- `vertical`: Cursor builds a stock-style rail (class `ore`, id `workbench.parts.activitybar`, 48 px wide) and a stock-style side bar (class `JDt`). So yes, the vertical activity bar still exists in Cursor. It is an older copy of the stock code (see "V items missing from Cursor").

Proof:
- `D5e="workbench.activityBar.orientation"` and the definition `[D5e]:{type:"string",enum:["horizontal","vertical"],default:"horizontal",description:k(4227,null)}`.
- Part chooser: `c=i==="vertical"?{case:"vertical",v:s.createInstance(JDt)}:{case:"horizontal",v:s.createInstance(ePt)}` (class `UEa`, the pane composite service).
- Hidden rail: `isActivityBarHidden(){if(this.configurationService.getValue(D5e)==="horizontal")return!0;...}`.
- The DOM part list builds `workbench.parts.activitybar` only for vertical: `...g==="vertical"?[{id:"workbench.parts.activitybar",role:"none",classes:["activitybar",v?"left":"right"]}]:[]`, with `g=i.getValue(D5e)`. In horizontal mode layout code uses `getDummyActivityBarView()` as a stand-in.

## Proposed minor groups (Cursor view)

- AB-1 Rail icons and view switching - tag `both`, evidence `source` (V) and `bundle` (Cursor), boundary no
  - Cursor: stock rail only when orientation is `vertical`. Same click, overflow ("Additional Views", NLS 3323), arrow key navigation, Previous and Next Primary Side Bar View (NLS 3302, 3303), Focus Activity Bar (NLS 3304). `workbench.action.focusActivityBar` is enabled only for vertical (`precondition:ge.equals("config."+D5e,"vertical")`). In horizontal mode the tab row has no keyboard focus code (see AB-6).
  - Click behavior: `workbench.activityBar.iconClickBehavior` (`toggle` | `focus`, default `toggle`, NLS 4228-4230) is still defined. Only the vertical rail reads it. A click on the active tab in the horizontal row does nothing: `E=N=>{N!==t.sidebarData.activeViewContainerID&&t.openPaneComposite(N,!0),p(!1)}`.
- AB-2 Customizing the rail - tag `both`, evidence `source` and `bundle`, boundary no
  - Cursor vertical rail: same as V (Hide and Keep NLS 3325-3326, Move To NLS 3920, Reset Location NLS 3921-3922, drag and drop). The horizontal row has its own, smaller version: pin and unpin, drag to reorder in the dropdown, a right-click menu (covered in AB-6, not repeated here).
- AB-3 Badges on rail icons - tag `both`, evidence `source` and `bundle`, boundary no
  - Cursor vertical rail: same as V (Hide Badge NLS 3327, Show Badge NLS 3328, `activityBarBadge.*` colors NLS 4491-4492). The horizontal tab row draws no badges. Its `showActivity(e,t,n,i)` is a no-op (`showActivity(e,t,n,i){return Jt(()=>{})}`) and the tab component has no badge element. So Source Control change counts never show on the Cursor tabs.
- AB-4 Accounts and Manage entries - tag `both`, evidence `source` and `bundle`, boundary no
  - Cursor draws Accounts and Manage only in the vertical rail (GlobalCompositeBar, class `uVe`, NLS 3838-3852). The horizontal row never draws them and the Cursor title bar never draws them (`get activityActionsEnabled(){return!1}`). So in the default layout the Manage menu (Settings, Keyboard Shortcuts, Themes, update items) has no button anywhere. Details in "Accounts and Manage in Cursor" below.
- AB-5 Placement, size, and visibility - tag `both`, evidence `source` and `bundle`, boundary no
  - Cursor keeps `workbench.activityBar.location` (default, top, bottom, hidden; NLS 4222-4226) and `workbench.activityBar.iconClickBehavior`. In horizontal mode `location` has no effect (the horizontal part never reads it). It does not have `workbench.activityBar.autoHide`, `workbench.activityBar.compact`, the Activity Bar Size submenu, or the four `workbench.action.activityBarLocation.*` commands with the Activity Bar Position submenu. `Toggle Activity Bar Visibility` (`workbench.action.toggleActivityBarVisibility`, NLS 3985) exists but in horizontal mode it resets `location` and shows an info dialog (below).
- AB-6 Pinned container tab row (Cursor's replacement) - tag `cursor`, evidence `bundle`, boundary no
  - One sentence: the Cursor-only row of icon tabs at the top of the side bar, its overflow drop-down, pin and reorder controls, menus, and the `orientation` setting that switches it on or off.
  - Examples: default tabs Explorer, Search, Source Control, Extensions; chevron drop-down listing every container with a pin toggle; right-click menu with Hide or Keep and a checklist of all containers; `workbench.activityBar.orientation` (`horizontal` or `vertical`) with a restart prompt.
  - Boundary note: the row also hosts an always-pinned Agents container (`workbench.view.backgroundAgent`). That container is out of scope (agent surface) and listed under left out. It is a scope seam, not agent output, so the flag stays no.

Confidence that this grouping is the right shape: high - the horizontal row is a separate Cursor-built UI with its own code, menus, and storage, and every stock group still has a Cursor counterpart (vertical only), with AB-4 the one that mostly disappears.

## The orientation setting (AB-6)

- Id: `workbench.activityBar.orientation`. Type string. Values: `horizontal`, `vertical`. Default: `horizontal`.
- Label text: the only description is NLS 4227, "Controls the orientation of the activity bar in the workbench. Quit and reopen for changes to take effect." There are no per-value descriptions (no `enumDescriptions`).
- Where it shows in the Settings editor: the group `workbench/appearance` (NLS 8750) matches `workbench.activityBar.*`, so it sits under Workbench > Appearance next to `location` and `iconClickBehavior`. Proof: `{id:"workbench/appearance",label:k(8750,null),settings:["workbench.activityBar.*","workbench.*color*",...]}`.
- Change behavior: the setting is in the restart-required list (`fri.SETTINGS` includes `D5e`, and the watcher calls `this.activitybarOrientation.handleChange(r.workbench?.activityBar?.orientation)`). Changing it shows a dialog "A setting has changed that requires a restart to take effect." with detail "Press the restart button to restart {0} and enable the setting." and a Restart button (NLS 8923, 8925, 8927).
- Other effects of `vertical` versus `horizontal`: layout tab order and focus order include the rail only for vertical (`activityBarDirection` checks in layout code); view pane headers keep their collapse twisty when they are alone only in vertical (`updateTwistyIcon`: `!this.isAlone()||this.configurationService.getValue(D5e)==="vertical"`); a single view in a side bar container is not merged into the container title in horizontal mode, so it keeps its own header (`isViewMergedWithContainer` returns false when the container location is the side bar and `activityBarDirection!=="vertical"`). The last two belong to SB, listed here for the seam.
- `Toggle Activity Bar Visibility` (`workbench.action.toggleActivityBarVisibility`, NLS 3985) in horizontal mode: it cannot find `workbench.parts.activitybar`, so it clears `workbench.activityBar.location` and shows an info dialog "Please quit and reopen for the activity bar to be displayed vertically." Proof: `run(e){...try{if(!t.getContainer(rn,"workbench.parts.activitybar"))throw new Error}catch{return n.updateValue("workbench.activityBar.location",void 0),e.get(Wc).confirm({message:"Please quit and reopen for the activity bar to be displayed vertically.",type:"info"}),...}`. Same message in NLS 3301 for `Focus Activity Bar` when the rail is missing.

## The horizontal tab row (AB-6 detail)

Code: class `ePt` (part id `workbench.parts.sidebar`, min width 214, min height 77), built with Solid.js. Root component `Xyk` holds the tab row `$yk` and the container host `vyk`. The old stock side bar code is in class `JDt`. Proof for the overall shape: `ePt=class extends qJt{...super("workbench.parts.sidebar",e,t)...this.minimumWidth=214...this.minimumHeight=77...[this.sidebarData,this.setSidebarData]=pyk(...)`; `create(e,t){...Jyk(this.element,this.sidebarData,...)}`.

What a person sees:

- A title row, 35 px tall, icon tabs centered with a 2 px gap, then the active container below. When the active container is Run and Debug the title becomes 70 px: a second row holds that container's title actions toolbar (centered, `toggleMenuTitle:"Actions..."`). Only the Debug container sets `wantsActionToolbar:!0`; V has no such option. Proof: `sn(()=>{v()?t.setSidebarData("heightOfTitle",70):t.setSidebarData("heightOfTitle",35)})` and `...,alwaysUseContainerInfo:!0,order:3,wantsActionToolbar:!0}`.
- Tabs: icon only, 26 x 26 px, 16 px icon, no text, no tooltip, no badge. Hover uses the list hover background; the selected tab gets class `sidebar-list-item-selected` (selected background, panel title active color, 1 px contrast outline in high contrast themes). Proof: `b6m` renders only an icon div; style object `mMr={height:"26px",...width:"26px",...}`; CSS classes `.sidebar2-item-icon-anysphere` (radius 4px, margin 4px, inactive color is the panel title inactive foreground variable) and `.sidebar-list-item-selected` (background is the sidebar list item selected background variable) in `C\out\vs\workbench\workbench.desktop.main.css`.
- No keyboard support found in the tab component: no `tabindex`, no `role`, no key handlers in the tab, chevron, or dropdown code (searched the whole `ePt` component region). Arrow key movement, tooltips with keybindings, and ARIA tab roles that the stock rail has are absent.
- Default tabs: Explorer (`workbench.view.explorer`), Search (`workbench.view.search`), Source Control (`workbench.view.scm`), Extensions (`workbench.view.extensions`). Proof: `pinnedViewContainerIDs:[uyk,Rcr,dyk,hyk]`. Run and Debug, Testing, References and any other container are not pinned by default; they live in the drop-down.
- How many tabs fit: `Math.min(pinnedCount, Math.floor(rowWidth/40)-1-(extraActiveTab?1:0))`. It measures the width with a ResizeObserver and reserves one 40 px slot for the chevron. At the minimum width of 214 px that is 4 tabs. Pinned tabs that do not fit are reachable only from the drop-down.
- Active tab that is not pinned: shown as one extra selected tab at the end of the row (`f(t.sidebarData.viewContainers.elements.find(...))`), so the current view is always visible.
- Chevron (overflow) button: sits after the tabs, same look as a tab, icon chevron down when closed and chevron up when open. Click opens or closes a drop-down panel below the row (absolute, z-index 100, soft shadow). The panel closes after any selection or whenever the active container changes.
- Drop-down rows (one per container, pinned ones first): icon, label text, the container keybinding in half opacity if there is one, and a pin icon on the right. The pin icon is hidden (opacity 0) for unpinned rows until the row is hovered; clicking it pins or unpins without selecting the row. Clicking the row opens that container. Proof: `y6m` (row), `Hyk` (list), CSS `.sidebar-2-pinned-icon-not-pinned{opacity:0}` and `.sidebar-list-item:hover .sidebar-2-pinned-icon{opacity:1!important}`.
- Drag to reorder: only inside the drop-down, and only inside the same group (pinned rows with pinned rows, unpinned with unpinned). The dragged row goes to half opacity and a drag overlay follows the pointer. The new position is stored as a number halfway between its neighbors. The tabs in the row themselves are not draggable (the tab component `b6m` has no drag code; only the `Uyk` drop-down rows do). Proof: handler `A=({draggable:N,droppable:F})=>{...U=U.filter(q=>Wke(q.id,t.sidebarData)===j)...X=(q+K)/2...t.setSidebarData("viewContainerOrders",U[B].id,{order:X,secondaryOrder:Y})`.
- Order rule: pinned first; then the saved order number; then the container's own `order`; then a fixed pseudo-random tiebreak from the container id (`Math.abs(Math.sin(parseInt(id.replace(/\./g,""),36)))`).
- Pin rules: Hide is disabled for the last pinned container (the row never goes empty). The Agents container `workbench.view.backgroundAgent` always counts as pinned (`Wke(e,t){return e===r4t?!0:...}`).
- Persistence: key `sidebar2.sidebarData.memoized.v1`, application scope, stores `{pinnedViewContainerIDs, viewContainerOrders}` only. The active container is stored separately under `workbench.sidebar.activeviewletid` (workspace scope).
- Right-click on a tab (`M` builds the menu, module `vs/workbench/browser/parts/sidebar/sidebarMenu`):
  - Configure Keybinding (only if the container has a keybinding id; with the current key shown after a tab character) - NLS 3977, 3978; runs `workbench.action.openGlobalKeybindings` with the keybinding id
  - Hide '{name}' or Keep '{name}' (NLS 3979, 3980); disabled when it would unpin the only pinned container
  - every item from the `ViewContainerTitleContext` menu: the container's view show or hide toggles, Reset Location (NLS 13433), Hide Primary Side Bar (NLS 3101), Move Primary Side Bar Right or Left (NLS 3086, 3087)
  - a separator, then a checklist of every container (check mark = pinned, keybinding shown, disabled for the only pinned one)
  - not present, but present in the stock rail menu: Hide Badge or Show Badge, Move To submenu, Activity Bar Position, Activity Bar Size, Accounts toggle, Menu toggle
- Right-click on empty space in the row (only when the click lands on the row itself): one item, Move Primary Side Bar Right or Left (NLS 3981-3984, command `workbench.action.toggleSidebarPosition`), with a tooltip. Proof: `L=N=>{if(N.target!==N.currentTarget)return;...id:"workbench.action.toggleSidebarPosition",label:k(U?3981:3982,null),tooltip:k(U?3983:3984,null)...}`.
- Side bar on the right: works the same; position only changes `getSideBarPosition()` checks.
- Keyboard commands still run but see all containers: `getPinnedPaneCompositeIds`, `getVisiblePaneCompositeIds`, `getPaneCompositeIds` all return every registered container sorted by order, ignoring the pin state. So Previous and Next Primary Side Bar View cycle through unpinned containers too. Proof: `getPinnedPaneCompositeIds(){return this.getPaneComposites().map(e=>e.id)}getVisiblePaneCompositeIds(){return this.getPaneComposites().map(e=>e.id)}`.

## Accounts and Manage in Cursor (AB-4 detail)

- Vertical rail: drawn at the bottom of the rail by the stock global bar. The rail is built with the global items switched on (`...,"workbench.parts.activitybar",this.paneCompositePart,!0)`, class `aMr` creates `uVe`). The Accounts icon, Manage gear, Hide Accounts, the Accounts toggle in the rail menu, the profile text badge, and the "Manage {0} (Profile)" hover text are all present (NLS 3838-3852, 4517-4518, 11585). Same behavior as V for those pieces.
- Horizontal row: not drawn. The tab component has no global items.
- Title bar: the stock code that draws Accounts and Manage there is still in the bundle (NLS 4022-4025, module `vs/workbench/browser/parts/titlebar/titlebarActions`) but disabled: `get activityActionsEnabled(){return!1}`. So setting `workbench.activityBar.location` to `top` or `bottom` in Cursor does not move Accounts and Manage to the title bar either; with the vertical rail and location `top` or `bottom` they are drawn nowhere (derived from the code, not run).
- Net result in the default Cursor layout: Settings, Keyboard Shortcuts, Themes, Snippets, Tasks, Profile, and the update items are not reachable from any button. They remain reachable from the Command Palette and the application menu. Cursor's own settings entry is a title bar control (NLS 4038-4039), owned by TB.
- Manage menu contents in Cursor (every `Pe.GlobalActivity` registration, 18 hits): Command Palette... (group `1_command`), Profile ({name}) submenu or Profile entry, Settings, Extensions, Keyboard Shortcuts, Configure Snippets, Tasks, Themes (group `2_configuration`, orders 1 to 7 as in V), and the update items in group `7_update`: Check for Updates..., checking, download now, downloading, install, updating, Show Update Release Notes, Restart to Update. Proof strings: `Pe.GlobalActivity,{group:"1_command",command:{id:mcn.ID,title:k(8908,null)},order:1}`, `Pe.GlobalActivity,{group:"7_update",command:{id:"update.check",...},when:Wle.isEqualTo("idle")}`. No Cursor-specific item was found in the Manage menu.
- Accounts menu contents in Cursor (every `Pe.AccountsContext` registration, 8 hits): one submenu per signed-in account (Manage Trusted Extensions, Sign Out), sign-in request items (group `2_signInRequests`, NLS 12306), access request items (group `3_accessRequests`, NLS 12305), remote tunnel items (group `2_remoteTunnel`), a disabled placeholder "No accounts requested yet..." (NLS 4887), "Loading..." (NLS 4888), and Manage Extension Account Preferences... (`_manageAccountPreferencesForExtension`, NLS 4859, 4866). No Cursor-specific item (no Cursor sign-in) was found in this menu.

## The vertical rail when orientation is `vertical`

- Part class `ore`: fixed 48 px wide (`this.minimumWidth=48,this.maximumWidth=48`), 48 px items, 24 px icons, item size 52. Colors, drag and drop border, badge colors, and hover side (right of the rail when the side bar is left) match V. Same storage keys `workbench.activity.pinnedViewlets2`, `workbench.activity.placeholderViewlets`, `workbench.activity.viewletsWorkspaceState`.
- Rail right-click menu: Menu or Hide Menu (when `window.menuBarVisibility` is `compact`, `hidden`, or `toggle`), the Accounts toggle, Move Primary Side Bar Right or Left, Hide Primary Side Bar. No Activity Bar Position submenu and no Activity Bar Size submenu. Proof: `getActivityBarContextMenuActions(){const e=[fh({id:WMt.ID,label:WMt.getLabel(this.layoutService),...})];return this.part==="workbench.parts.sidebar"&&e.push(fh({id:uV.ID,label:uV.LABEL,...})),e}`.
- Side bar class `JDt`: still honors `workbench.activityBar.location` for top and bottom (`shouldShowCompositeBar` returns true for `top` or `bottom`), then draws the icon strip in the side bar title. It has no auto-hide check.
- The compact hamburger Menu button works the same (`installMenubar`, `window.menuBarVisibility`).

## Cursor additions (for the parent)

- Setting `workbench.activityBar.orientation` and the restart prompt.
- The whole horizontal tab row (class `ePt`, Solid.js): icon tabs, chevron drop-down with pin toggles, drag reorder in the drop-down, its own tab menu module `vs/workbench/browser/parts/sidebar/sidebarMenu`, storage key `sidebar2.sidebarData.memoized.v1`.
- Two-row title for Run and Debug (`wantsActionToolbar`).
- Always-pinned Agents container tab (`workbench.view.backgroundAgent`), out of scope.
- A sidebar-module update card ("Update to v...", Later, Install Now, Changelog; class `update-notification-container`) lives next to the tab row code (function `Zyk`, `jyk`, `Gyk`). Where it appears was not checked; likely TB or NT territory. Listed under left out.

## Built-in extension contributions (Cursor)

- None new. `C\extensions\references-view\package.json` declares the same one container `references-view` (icon `$(references)`) as V. No `cursor-*` extension declares `viewsContainers`: a search for `viewsContainers` across `C\extensions\*\package.json` matched only `references-view`. The V `copilot` extension (two rail containers, Chat Debug and Language Context Inspector) is not in Cursor.

## V items missing from Cursor (another agent suspected Cursor's stock UI copy is older than 1.128)

Each was searched in the bundle with zero hits unless noted. This matches the idea that Cursor's activity bar code predates several 1.128 features, but the bundle cannot say which older version.

- `workbench.activityBar.autoHide` setting (V `workbench.contribution.ts` 663-668): 0 hits. Cursor's `shouldShowCompositeBar` only checks `top` or `bottom`.
- `workbench.activityBar.compact` setting and the Activity Bar Size submenu with commands `workbench.action.activityBar.size.default` and `.compact`: 0 hits. Cursor's rail has no compact class.
- Four commands `workbench.action.activityBarLocation.default`, `.top`, `.bottom`, `.hide` with their "Move Activity Bar to ..." titles, the `ActivityBarPositionMenu` submenu in View > Appearance and in the icon menu: 0 hits for the ids, `ActivityBarPositionMenu`, and the title strings in NLS. The `location` setting alone remains.
- `workbench.experimental.modernUI` (floating panels, rail gutter): 0 hits.
- Accounts menu items: Backup and Sync Settings... (`workbench.userDataSync.actions.*`, 0 hits), Turn on Cloud Changes... (`workbench.editSessions.actions.*`, 0 hits), Sign in to use GitHub Copilot... (`workbench.action.chat.triggerSetupFromAccounts`, 0 hits), Manage Language Model Access... (`workbench.action.chat.manageLanguageModelAuthentication`, 0 hits), Sign in to access Extensions Marketplace (`workbench.extensions.actions.gallery.signIn`, 0 hits), Manage Dynamic Authentication Providers... (`workbench.action.removeDynamicAuthenticationProviders`, 0 hits). Manage menu: no Settings Sync items.
- Horizontal row lacks what the stock rail has: badges, tooltips, keyboard focus and arrow keys, ARIA tab roles, Hide Badge or Show Badge, Move To, drag from tab to reorder, `iconClickBehavior` on the active tab, overflow "Additional Views" menu (replaced by the chevron drop-down).

## Left out

- Agents sidebar, background agents, chat containers: `workbench.view.backgroundAgent` tab and the unified sidebar part (`workbench.parts.unifiedsidebar`) are agent surface.
- Extensions icon contents: Extensions is only a default tab here; its view is not cataloged.
- `workbench.view.alwaysShowHeaderActions`: defined in Cursor (NLS 4231, `viewPane` class `fx`, `AlwaysShowActionsConfig`), same as V. It controls view header actions in the side bar. Belongs to SB.
- Secondary side bar icon switcher (`workbench.secondarySideBar.showLabels`, NLS 4220): AX. Cursor also blocks contributing view containers to the Secondary Side Bar (NLS 2890, "reserved for Cursor's agent UI").
- The update notification card in the sidebar module: not the tab row. Seam with TB (Update control) and NT.
- Cursor Settings button in the title bar (NLS 4038-4039): TB.
- Side bar frame, view headers, and the Run and Debug toolbar content: SB.

## Newer than 1.128

- Nothing in the Cursor bundle is newer than 1.128 for this section. The things newer than 1.128 that exist only in N (account avatar, Codex account entries, menu animation, Modern UI density) are listed in the V file. Cursor has none of them (`workbench.accounts.showAvatar`: 0 hits).

## Could not verify

- Nothing was run. All Cursor behavior comes from reading the bundle, the CSS, and the NLS files.
- Whether the tab row shows tooltips on hover through a browser `title` attribute set after render: the tab template is `<div><div>` and no `title` was set in its code, but a runtime check was not done.
- Where the update card appears on screen.
- Where Accounts and Manage appear for vertical rail with location `top` or `bottom` (derived: nowhere).
- Exact on-screen text of the orientation setting title in the Settings editor (the editor derives it from the id, "Workbench > Appearance: Activity Bar: Orientation"; not seen).
- Whether `Hide Primary Side Bar` and `Move Primary Side Bar Right` entries in the tab menu come through `ViewContainerTitleContext` for the horizontal row exactly as in V: the strings and menu registrations exist (`Pe.ViewContainerTitleContext` appears with ids `uV.ID` and a Secondary Side Bar entry), and the tab menu reads that menu, but the menu was not opened.

## Notes for the side bar file

The side bar agent's file (`cursor\04-side-bar.md`) calls the overflow menu one that "lists every container with a check mark". Two separate things exist: the chevron drop-down (rows with pin icons, reorder by drag) and the right-click menu (checklist with check marks). Also, class `ore` is the stock rail (`workbench.parts.activitybar`), not a third side bar. Both are cataloged here under AB-6 and the vertical rail section.

