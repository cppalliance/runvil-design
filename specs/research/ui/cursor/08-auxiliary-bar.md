# Auxiliary Bar (AX) - Cursor findings

Section: Auxiliary bar (AX). Date: 2026-10-25. Pass: 1.

- Scope of this file: Cursor 3.27.12 (VS Code base 1.128.0) only. The stock VS Code side is in the sibling file under `vscode\`. Group ids match that file.
- Level: minor groups only. Up to 4 example features per group.
- Short names: C = `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`. MAIN = `C\out\vs\workbench\workbench.desktop.main.js`. CSS = `C\out\vs\workbench\workbench.desktop.main.css`. MSG = `C\out\nls.messages.json` (the label text table; the bundle refers to labels by index, so a label like "Hide Secondary Side Bar" shows up as `k(3310,null)` in MAIN and as entry 3310 in MSG). V and N are as in the sibling file.
- Evidence level for everything in this file is `bundle`. I read strings and minified code in MAIN, MSG and CSS. I did not read source and I did not run Cursor.
- I never opened `workbench.glass.main.js` (the Agents window), per the brief.

## Plain summary

The Cursor auxiliary bar is V's part rewritten as a chat host. Its `super()` call passes the same arguments as V's (part id `workbench.parts.auxiliarybar`, the active view key, the `activeAuxiliary` and `auxiliaryBarFocus` context keys, the title menu `AuxiliaryBarTitle`), so it is the same kind of pane-composite part. It uses the same storage keys (`workbench.auxiliarybar.activepanelid`, `pinnedPanels`, `placeholderPanels`, `viewContainersWorkspaceState`) plus one new one (`workbench.auxiliarybar.initialViewContainers`). Search for `workbench.auxiliarybar.activepanelid` in MAIN to land on it.

After I remove everything that is agent or chat (the New Agent, Chat History, More Actions and Close buttons, the Show Agents Side Bar button, the agent tab styling, the embedded chat editor group, Hide Chat, Maximize Chat Size), very little that is Cursor-only is left. Three things:

- A block on extensions: extensions cannot contribute view containers to this part.
- Different width limits: minimum 300 (V is 170), preferred width never under 540 (V is 300).
- Tabs in the strip scroll instead of overflowing (`scrollable: true`).

What is left otherwise is V's shell, with several V features cut. The cuts matter more than the additions. See "V items missing from Cursor" below.

## Groups

Group count in Cursor stays at the five from V. AX-3 exists only as a gap (nothing to catalog, listed so ids stay stable). Confidence: medium - Cursor's own side is thin, but dropping the group ids would break the match with the V file.

### AX-1 Show, hide and focus the secondary side bar

- Plain name: Show, hide and focus the secondary side bar
- One sentence: Same command ids as V for opening, hiding and focusing, but opening now brings up the agent chat and can create a new one.
- Tag: both (searched MAIN and MSG, ids and labels found)
- Evidence: bundle
- Boundary: yes - the toggle command opens the agent chat pane and creates a new chat if none is open, so what appears is agent output.
- Example features:
  - Toggle Secondary Side Bar Visibility (`workbench.action.toggleAuxiliaryBar`, Ctrl+Alt+B, View menu under Appearance, Configure Layout submenu). Differences from V: it does nothing in a separate auxiliary window (precondition and keybinding `when` are `!isAuxiliaryWindowFocusedContext`); in the Agents window it runs `glass.togglePanel` instead; when it opens the part and no container is active it calls `createComposer({view:"pane",unifiedMode:"agent",openInNewTab:true})`; it sends `ai_pane.opened` and `ai_pane.closed` events
  - Focus into Secondary Side Bar (`workbench.action.focusAuxiliaryBar`, MSG 3309; shows the part first, focuses the active container; does nothing in the Agents window)
  - Hide Secondary Side Bar in the tab strip right-click menu (container title context menu, MSG 3310, only when the right-clicked container is in this part)
  - Toggle shown in the Customize Layout picker (MSG 3136, same checked state as the part being visible)
- Proof paths:
  - MAIN, search `workbench.action.toggleAuxiliaryBar` (3 hits; the class with `Toggle Secondary Side Bar Visibility`, a `Toggle Chat Pane` menu item for chat views, and the shared layout list), then `workbench.action.focusAuxiliaryBar`
  - MSG entries 3305-3312 (the label block for this module)
  - MSG 3136 and MAIN search `uYn(Uin.ID` (Customize Layout picker row)
  - MAIN, search `AUXILIARYBAR_HIDDEN:new` (state key `auxiliaryBar.hidden`, default set from the layout preset `chatVisible`, not from a setting)
- Notes:
  - Starting visibility is not a setting any more. `workbench.secondarySideBar.defaultVisibility` has 0 hits. The default comes from the layout presets `default-agent` and `default-editor`, which both have `chatVisible:true` (search `default-agent` in MAIN). The presets are agent layout, left out.
  - `Expand Editor Group and Hide Side Bars` and `Maximize Editor Group and Hide Side Bars` still exist (MSG 3626 and 3629). I did not confirm that they still hide this part.
  - The title bar draws its own layout toggles and treats this part's toggle id as the chat toggle (MAIN, search `h.id===Uin.ID`). That belongs to the title bar section.

### AX-2 Where it sits and how wide it is

- Plain name: Where it sits and how wide it is
- One sentence: Still opposite the primary side bar, still flipped by the same commands, but with higher width limits and a chat-first look.
- Tag: both (searched MAIN and MSG)
- Evidence: bundle
- Boundary: no
- Example features:
  - Opposite side from the primary side bar (setting `workbench.sideBar.location`, description in MSG 4213 still says "The secondary side bar will show on the opposite side")
  - Flip sides (`workbench.action.toggleSidebarPosition` and `moveSideBarRight` still exist, MSG 3085-3087 and 3159-3161; the labels say Primary Side Bar only)
  - Width limits (minimum 300, preferred never under 540, snap off, priority High when no editor is visible and Low otherwise, saved size key `auxiliaryBar.size` with a static default of 200 in the key table, plus a Cursor-only width key `cursor/layout/editor.auxiliaryBar.width`)
  - Looks (background is the editor background, not the Side Bar color; the border is drawn on the editor-facing edge, with conditions tied to chat layout flags)
- Proof paths:
  - MAIN, search `get minimumWidth(){return 300}` and the same class for `preferredWidth` and `get priority()`
  - MAIN, search `Pa={MAIN_EDITOR_CENTERED` (state key table: `AUXILIARYBAR_SIZE:new vFr("auxiliaryBar.size",0,1,200)`)
  - MAIN, search `cursor/layout/editor.auxiliaryBar.width`
  - MAIN, search `updateStyles(){super.updateStyles();const e=kl(this.getContainer())`
  - MSG 4213, 3085-3087, 3159-3161
- Notes:
  - V's tab strip menu items "Move Secondary Side Bar Left" and "Move Secondary Side Bar Right" are gone (no such label in MSG, 0 hits).
  - V's experimental Modern UI floating cards are not in Cursor: `workbench.experimental.modernUI` and `floating-panels` have 0 hits in MAIN and CSS.
  - The part has extra layout code for an "unified sidebar" (the Agents sidebar, `workbench.parts.unifiedsidebar`, 78 hits) and a no-titlebar layout. That is agent window layout, left out.

### AX-3 Maximize and restore

- Plain name: Maximize and restore
- One sentence: Not in Cursor. V's maximize feature was removed from this part.
- Tag: vscode (cursor: searched, 0 hits for all of these)
- Evidence: bundle
- Boundary: no
- Example features: none to list. Searched and not found in MAIN or MSG: `workbench.action.maximizeAuxiliaryBar`, `workbench.action.restoreAuxiliaryBar`, `workbench.action.toggleMaximizedAuxiliaryBar`, `workbench.secondarySideBar.forceMaximized`, `workbench.secondarySideBar.defaultVisibility`, the context key `auxiliaryBarMaximized`, the icon `auxiliarybar-maximize`, and the layout state keys `auxiliaryBar.wasLastMaximized` and `auxiliaryBar.lastNonMaximizedSize`.
- Proof paths:
  - MAIN, the state key table (search `Pa={MAIN_EDITOR_CENTERED`): it has `PANEL_WAS_LAST_MAXIMIZED` but no auxiliary bar maximize keys
  - MAIN, the context key block (search `auxiliaryBarFocus`): it has `auxiliaryBarFocus` and `auxiliaryBarVisible` but no `auxiliaryBarMaximized`
  - MSG entries 3305-3312: no maximize or restore label
- Notes: Cursor does have its own maximize, but it is chat's: `workbench.action.maximizeChatSize` (Ctrl+Alt+E, MSG 3089 says "hides editor, panel, and primary sidebar to focus on AI agent") and a double-click on the chat tab that can `toggleExpandAuxiliaryBar`. Both are chat features and are left out. I did not read how they size the part.

### AX-4 Tab strip and title buttons

- Plain name: Tab strip and title buttons
- One sentence: The tab strip is kept but cut down: labels or icons still work, previous and next view still work, but position settings and the right-click extras are gone, and the title buttons are agent buttons.
- Tag: both (searched; some pieces not found)
- Evidence: bundle
- Boundary: no (the agent buttons are left out, not cataloged)
- Example features:
  - Tabs as text or icons (`workbench.secondarySideBar.showLabels`, the only `workbench.secondarySideBar.*` setting left; MSG 4220 says "secondary sidebar title")
  - Previous and next view (`workbench.action.previousAuxiliaryBarView`, `workbench.action.nextAuxiliaryBarView`, MSG 3311 and 3312, Command Palette only)
  - Per-tab Move To and Reset Location (shared base code, MSG 3920-3925; the Move To list still offers Secondary Side Bar)
  - Scrollable tabs (Cursor addition: `scrollable:true` in the composite bar options, scrollbar hidden until hover; the tab chip CSS, 60 to 120 px wide with a fade on cut-off text, looks built for chat tabs)
- Proof paths:
  - MAIN, search `resolveConfiguration(){const e="default"` (tab position is fixed to `default`; `canShowLabels` is always true)
  - MAIN, search `getCompositeBarOptions(){const e=this;return{partContainerClass:"auxiliarybar"` (note `fillExtraContextMenuActions:t=>{}` is empty and `scrollable:!0`)
  - MAIN, search `shouldShowCompositeBar(){return this.isNoTitlebarLayoutActive()` (tab strip shows or hides based on unified sidebar visibility, not on a setting)
  - MAIN, search `workbench.action.previousAuxiliaryBarView`
  - CSS, search `.auxiliarybar .composite-bar-action-tab`
  - MSG 3920-3925
- Notes:
  - `workbench.activityBar.location` still exists (29 hits in MAIN) but this part ignores it, because the position is hard-coded to `default`. The setting text still mentions the Secondary Side Bar (MSG 4222-4226). The settings page would therefore promise an effect that does not happen here. I did not confirm that in the running app.
  - The tab right-click menu has no extras. V adds Activity Bar Position, Move Left or Right, Show Icons or Labels, Hide (V `auxiliaryBarPart.ts` lines 233-261). Cursor's callback is empty. The Move To and Reset Location items come from the base class and do remain.
  - The title buttons are: New Agent (MSG 3315), Chat History (3316), More Actions (3319), Close (3318) and Show Agents Side Bar (3313). All agent or chat, left out. V's two title buttons (maximize, hide X) are not there.
  - Cursor adds a new menu id `AuxiliaryBarHeader` for a header toolbar (`.auxiliary-bar-global-header`). I found no items registered into it in MAIN. Unverified what, if anything, shows there.
  - The V auto-hide setting `workbench.activityBar.autoHide` has 0 hits in MAIN, and the tracker class `VisibleViewContainersTracker` is not in the Cursor part constructor.

### AX-5 Moving views and containers into it

- Plain name: Moving views and containers into it
- One sentence: The same menus and commands as V still list the secondary side bar as a destination, but extensions are blocked from adding their own containers to it.
- Tag: both (searched; ids and labels found)
- Evidence: bundle
- Boundary: no
- Example features:
  - Drag and drop target with the empty message "Drag a view here to display." (MSG 3926; drag code lives in the shared base class)
  - Move Focused View destinations "New Secondary Side Bar Entry" and any container in this part (`workbench.action.moveFocusedView`, MSG 3126 and 3129, id `_.auxiliarybar.newcontainer`)
  - Move To submenu and Reset Location on a tab (MSG 3920-3925, ids `moveViewContainerTo<n>`, 15 hits)
  - Move Panel Views To Secondary Side Bar and Move Secondary Side Bar Views To Panel (`workbench.action.movePanelToSecondarySideBar`, `workbench.action.moveSecondarySideBarToPanel`, MSG 3966-3969)
- Proof paths:
  - MAIN, search `_.auxiliarybar.newcontainer` (2 hits)
  - MAIN, search `createMoveAction(e,t,n){return fh({id:` and `3925` (Move To label for this part)
  - MAIN, search `workbench.action.movePanelToSecondarySideBar` and `workbench.action.moveSecondarySideBarToPanel`
  - MAIN, search `case"secondarySidebar":o.error(k(2890,null))` (the block) with MSG 2890
  - MSG 8916 and MAIN `4_auxbar` (Open View group, View menu group)
- Notes:
  - The block text, MSG 2890: "View containers cannot be contributed to the Secondary Side Bar in Cursor. It is reserved for Cursor's agent UI. Contribute to 'activitybar' or 'panel' instead." It is raised when an extension uses `viewsContainers.secondarySidebar`. Only `activitybar` and `panel` are registered; the `secondarySidebar` key is still recognized and then rejected with this error.
  - Whether a view dragged in by hand actually renders in Cursor is unverified. The part's title row and tab strip depend on chat layout flags (`shouldRenderTitleArea`, `shouldShowCompositeBar`), so a manually moved view may behave differently from V.

## Cursor additions

Everything below is agent or chat related, except the first three. None is cataloged as a feature.

- Extension block on `viewsContainers.secondarySidebar` (non-agent, AX-5).
- Width limits 300 and 540 (non-agent, AX-2).
- Scrollable tab strip (non-agent, AX-4).
- Agent title buttons: New Agent, Chat History, More Actions, Close, Show Agents Side Bar. Left out.
- `aichat.close-sidebar` "Hide Chat" and a `Toggle Chat Pane` item in the view title menu of chat views, both using the toggle id. Left out.
- `workbench.action.maximizeChatSize` and `toggleExpandAuxiliaryBar`. Left out (chat).
- Embedded chat editor group inside the part (`embedded-aux-bar-editor`, setting key `cursor/chatEditorGroupEnabled`). Left out.
- New menu id `AuxiliaryBarHeader`. Contents unverified.
- `Reset Editor Group Sizes (Excluding Agent)` (`workbench.action.evenEditorWidthsExcludingAgent`, MSG 3568, Command Palette only, MAIN search `evenEditorWidthsExcludingAgent`). It is an editor group command, not an auxiliary bar one. It evens out editor group widths but skips groups whose active editor is an agent editor (typeIds `workbench.editor.composer.input` and `workbench.editors.backgroundComposerPeekEditorInput`). It belongs with the editor section (ED). Boundary: yes (output depends on agent editors). V's plain `workbench.action.evenEditorWidths` is still present.

## Left out and why

- Everything in the Cursor chat and agent UI that lives in this part: buttons, tabs styled as chat tabs (CSS classes `inline-agent-tabs`, `background-composer-tab`, and the agent-mode title class), the workspace header, chat title text, the history flyout. Out of scope per the brief.
- Agents sidebar and unified sidebar (`workbench.action.toggleUnifiedSidebar`, `workbench.parts.unifiedsidebar`, `cursor.toggleAgentWindowIDEUnification`). Out of scope.
- Layout presets `default-agent` and `default-editor`. Agent layout.
- `workbench.glass.main.js`. The Agents window, never read.
- Title bar layout buttons that include this part's toggle. Belongs to the title bar section.

## V items missing from Cursor

Each searched in MAIN, MSG or CSS with 0 hits unless noted.

- Maximize, Restore and Toggle Maximized commands, the title button, the Ctrl+W restore key, `auxiliaryBarMaximized` context key (all of AX-3).
- Settings `workbench.secondarySideBar.defaultVisibility` and `workbench.secondarySideBar.forceMaximized`.
- `workbench.action.closeAuxiliaryBar` (Hide from the Command Palette). Cursor has `aichat.close-sidebar` "Hide Chat" instead.
- The Hide X button in the title row for non-agent use.
- Title bar layout toggle menu items "Toggle Secondary Side Bar" for `MenuId.LayoutControlMenu` (label not in MSG). Cursor's title bar draws its own.
- Right-click extras on the tab strip: Activity Bar Position submenu, Move Secondary Side Bar Left or Right, Show Icons or Show Labels (the V id `workbench.action.auxiliarybar.toggleShowLabels` is absent; `workbench.action.panel.toggleShowLabels` exists for the panel).
- The container title menu items "Move Secondary Side Bar Left" and "Move Secondary Side Bar Right".
- Tab position under `workbench.activityBar.location` (ignored by this part) and the `workbench.activityBar.autoHide` setting (0 hits anywhere in MAIN).
- Extension contribution to `viewsContainers.secondarySidebar` (blocked, MSG 2890).
- Experimental Modern UI floating cards (`workbench.experimental.modernUI`).
- The `auxiliaryBar.empty` state key and the first-run rule that kept the part closed when it was empty before.
- Screen reader announcement on toggle ("Secondary Side Bar shown" or "hidden", V `auxiliaryBarActions.ts` lines 71-82). Not found in MSG. Unverified whether Cursor announces another way.

## Newer than 1.128

None of the newer-than-1.128 items in the V file exist in Cursor: `secondarySideBarVisible`, `toggleSecondarySideBar`, `isSecondarySideBarVisible` and `getToolbarResponsiveBehavior` all have 0 hits in MAIN, which fits a 1.128 base. Nothing in Cursor is newer than 1.128 for this part.

## Unverified

- I did not run Cursor. Every behavior note is from reading code strings, so "does nothing in the Agents window" or "setting has no effect" are readings of the code, not observations.
- Whether the Move To, Reset Location and drag and drop paths work on this part at runtime. The code is present, but the part's tab strip is tied to chat layout flags.
- What the `AuxiliaryBarHeader` menu holds.
- Whether the editor group commands "Expand Editor Group and Hide Side Bars" and "Maximize Editor Group and Hide Side Bars" still hide this part.
- Whether the screen reader announcement on toggle exists in Cursor.
- Size of the part in pixels at first run. The key `auxiliaryBar.size` defaults to 200, but the minimum is 300, so I do not know which wins on a fresh profile.

*2026-10-25 - claude-sonnet-5-5*
