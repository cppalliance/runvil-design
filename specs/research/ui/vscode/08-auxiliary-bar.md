# Auxiliary Bar (AX) - VS Code 1.128.0 findings

Section: Auxiliary bar (AX). Date: 2026-10-25. Pass: 1.

- Scope of this file: stock VS Code 1.128.0 only (root V). Cursor findings are in the sibling file under `cursor\`.
- Level: minor groups only. No full feature lists, no pieces. Up to 4 example features per group.
- Short names: V = `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0`. N = `C:\Users\Vinnie\src\vscode` (1.144.0, used only to spot newer things). C = the Cursor install `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`. All proof paths below start under `V\src\vs\workbench\` unless they say otherwise.
- Line numbers are from the files as read. They can drift by a line or two, so search the quoted id or label if one does not match.
- Tags say `both` when I also searched the Cursor bundle for a command id or label of that item. The Cursor evidence itself is in the sibling file.
- The user-facing name in V is "Secondary Side Bar". The code name is "auxiliary bar". Both mean the same part.

## What this section is

The second side bar, on the opposite side of the editor from the primary side bar. In stock VS Code its usual occupant is Chat, which is out of scope. The container is in scope: showing and hiding it, where it sits, its tab strip, moving views into it, maximizing it, and its title buttons.

Key facts about the part in V:

- It is one `AuxiliaryBarPart` that extends the shared pane-composite part (`browser\parts\auxiliarybar\auxiliaryBarPart.ts`). It hosts view containers whose location is `ViewContainerLocation.AuxiliaryBar` (enum value 2, `common\views.ts` line 42).
- It has no activity bar column. Its tabs (the "composite bar") sit in its own title row, or in a strip at the top or bottom of the part if `workbench.activityBar.location` says so (`auxiliaryBarPart.ts` lines 163-170 and 286-293).
- No built-in extension under `V\extensions` contributes to the secondary side bar (searched every `package.json` for `secondarySidebar`, no hits). Inside core, the only occupant is Chat under `contrib\chat`, which I did not catalog.
- Its position is not independent. It always sits on the opposite side from the primary side bar, which is set by `workbench.sideBar.location` (`browser\workbench.contribution.ts` line 570 says so in its description).

## Proposed minor groups

Count: 5. Reason: the part is small, but V gives it five separable behaviors that a person would test one at a time. Fewer than 5 would hide the maximize feature (own commands, own setting, own context key, own restore key) or the view-moving feature (the only way anything other than Chat gets into this part). Confidence: medium - AX-4 could be merged with AX-1 and it would still read fine.

### AX-1 Show, hide and focus the secondary side bar

- Plain name: Show, hide and focus the secondary side bar
- One sentence: Open or close the part, move keyboard focus into it, and choose whether it starts open in a new workspace.
- Tag: both (cursor: searched, toggle, focus and hide ids found)
- Evidence: source
- Boundary: no (in V; Cursor flags this group, see the Cursor file)
- Example features:
  - Toggle Secondary Side Bar Visibility (`workbench.action.toggleAuxiliaryBar`, Ctrl+Alt+B, View menu under Appearance, Configure Layout submenu, screen reader announcement "Secondary Side Bar shown" or "hidden")
  - Hide Secondary Side Bar (`workbench.action.closeAuxiliaryBar`, Command Palette only; also Hide in the tab strip right-click menu and the X button in the title row)
  - Focus into Secondary Side Bar (`workbench.action.focusAuxiliaryBar`, shows the part first if hidden, then focuses the active view)
  - Starting visibility: `workbench.secondarySideBar.defaultVisibility` with five choices (hidden, visibleInWorkspace, visible, maximizedInWorkspace, maximized)
- Proof paths:
  - `browser\parts\auxiliarybar\auxiliaryBarActions.ts` (toggle lines 30-83 with keybinding at lines 51-54 and menus at lines 55-65, X button in title lines 86-93, close lines 95-110, focus lines 112-139, title bar layout toggle menus lines 141-179, tab strip context Hide line 181-191)
  - `browser\layout.ts` (`setAuxiliaryBarHidden` lines 2253-2291, `setPartHidden` line 2294, default visibility logic lines 2970-2995, state key `auxiliaryBar.hidden` line 2855, zen mode hide and restore lines 1472-1477 and 1556-1558, startup restore lines 819-825 and 1165-1173)
  - `browser\workbench.contribution.ts` lines 597-610 (`workbench.secondarySideBar.defaultVisibility`)
  - `browser\actions\layoutActions.ts` line 1368 (toggle shown in the Customize Layout picker)
  - `browser\parts\editor\editorActions.ts` lines 1071-1091 and 1130-1152 (Expand Editor Group and Hide Side Bars, Maximize Editor Group and Hide Side Bars, both also hide this part)
  - `common\contextkeys.ts` lines 157-162 (`auxiliaryBarVisible`, `auxiliaryBarFocus`, `activeAuxiliary`)
- Notes: the Hide X in the title row only shows when `workbench.activityBar.location` is `default` (line 94 in `auxiliaryBarActions.ts`). Closing the only view of the active container hides that container (`services\views\browser\viewsService.ts` lines 336-338); I did not verify that this also hides the whole part.

### AX-2 Where it sits and how wide it is

- Plain name: Where it sits and how wide it is
- One sentence: The part follows the primary side bar to the opposite side, can be flipped with it, and resizes with a drag handle within width limits.
- Tag: both (cursor: searched, flip commands found, width numbers differ)
- Evidence: source
- Boundary: no
- Example features:
  - Opposite side from the primary side bar (follows `workbench.sideBar.location`; the grid moves the part when the setting flips)
  - Flip sides (`workbench.action.toggleSidebarPosition`; tab strip right-click menu shows "Move Secondary Side Bar Left" or "Move Secondary Side Bar Right" and runs the same command)
  - Width limits and drag resize (minimum 170, preferred width is the active view's own optimal width but never under 300, stored size key `auxiliaryBar.size`, default 300 or a quarter of the window, whichever is less)
  - Border and background follow Side Bar theme colors; border sits on the editor-facing edge
- Proof paths:
  - `browser\parts\auxiliarybar\auxiliaryBarPart.ts` (size limits lines 52-77, priority line 79, `updateStyles` lines 181-199)
  - `browser\layout.ts` (position class swap lines 636-660, grid move when side bar position changes lines 1965-2020, `resizePart` case line 1823, state keys lines 2827-2843, size defaults lines 2970-2975)
  - `browser\actions\layoutActions.ts` (`ToggleSidebarPositionAction` lines 141-171, secondary side bar left and right context items lines 209-230, `MoveSidebarRight` and `MoveSidebarLeft` lines 100-135, resize view actions lines 1140-1195 which include this part when focused)
  - `browser\actions\navigationActions.ts` lines 112-128 and 270-315 (focus the neighbor part left, right, up, down including this part, and the cycle of parts)
  - `browser\media\floatingPanels.css` (experimental Modern UI floating card look for this part, see the setting `workbench.experimental.modernUI`, `services\layout\browser\layoutService.ts` line 53, registered at `workbench.contribution.ts` lines 810-815)
  - `browser\parts\auxiliarybar\media\auxiliaryBarPart.css`
- Notes: there is no setting that places this part on its own side. If someone asks for "secondary bar on the left while primary is also on the left", V cannot do it.

### AX-3 Maximize and restore

- Plain name: Maximize and restore
- One sentence: Blow the secondary side bar up to fill the whole window (editor, panel and primary side bar get hidden) and put everything back later.
- Tag: vscode (cursor: searched, none of the maximize ids, the setting, or the context key found)
- Evidence: source
- Boundary: no
- Example features:
  - Maximize Secondary Side Bar (`workbench.action.maximizeAuxiliaryBar`, Command Palette only, disabled when already maximized)
  - Restore Secondary Side Bar (`workbench.action.restoreAuxiliaryBar`, Ctrl+W, and also Ctrl+F4 on Windows, only while maximized)
  - Maximize or restore button in the title row (`workbench.action.toggleMaximizedAuxiliaryBar`, icon `auxiliarybar-maximize`, first button in `MenuId.AuxiliaryBarTitle`, tooltip flips to "Restore Secondary Side Bar")
  - Startup and automatic maximize: `workbench.secondarySideBar.defaultVisibility` values `maximized` and `maximizedInWorkspace`, and the experimental `workbench.secondarySideBar.forceMaximized` (maximize on startup and whenever no editor is open)
- Proof paths:
  - `browser\parts\auxiliarybar\auxiliaryBarActions.ts` lines 214-299 (all three commands, the keybinding at lines 253-257, title button menu at lines 284-289)
  - `browser\layout.ts` (`isAuxiliaryBarMaximized` and `setAuxiliaryBarMaximized` lines 2132-2205, which save and restore sizes and which parts were visible; auto un-maximize when the editor, side bar or panel is shown lines 1873, 1912, 2055; editor opening un-maximizes only on explicit user action lines 346-362; force-maximize listeners lines 365-387 and 447-454; state keys lines 2835-2841; first-run defaults lines 2970-2975)
  - `browser\workbench.contribution.ts` lines 597-617 (`defaultVisibility`, `forceMaximized`)
  - `common\contextkeys.ts` line 162 and `browser\contextkeys.ts` lines 202-203 and 258-260 (`auxiliaryBarMaximized`)
  - `contrib\splash\browser\partsSplash.ts` line 98 (splash screen remembers the maximized state)
- Notes: panel maximize and this are mutually exclusive in the code (`isPanelMaximized` returns false while this is maximized, `layout.ts` line 2213). The Welcome page is skipped at startup while this part is maximized (`contrib\welcomeGettingStarted\browser\startupPage.ts` lines 118-121).

### AX-4 Tab strip and title buttons

- Plain name: Tab strip and title buttons
- One sentence: The row at the top of the part that holds one tab per view container, plus the little buttons next to it and the right-click menu on it.
- Tag: both (cursor: searched, prev and next view ids and the labels setting found; position-based pieces not found)
- Evidence: source
- Boundary: no
- Example features:
  - Tabs as text or icons (`workbench.secondarySideBar.showLabels`, default true, ignored when the activity bar is at the top)
  - Where the tabs go (`workbench.activityBar.location`: default puts them in the title row, top or bottom puts them in a strip, hidden hides them; `workbench.activityBar.autoHide` hides the strip when only one container exists)
  - Tab right-click menu (Activity Bar Position submenu, Move Secondary Side Bar Left or Right, Show Icons or Show Labels, Hide Secondary Side Bar, plus Move To and Reset Location for the clicked tab)
  - Previous and next view (`workbench.action.previousAuxiliaryBarView`, `workbench.action.nextAuxiliaryBarView`, Command Palette only)
- Proof paths:
  - `browser\parts\auxiliarybar\auxiliaryBarPart.ts` (config resolve lines 163-170, composite bar options lines 201-231, extra context menu lines 233-261, show or hide the strip lines 263-283, strip position lines 286-293, auto hide tracker lines 133-161)
  - `browser\parts\auxiliarybar\auxiliaryBarActions.ts` lines 193-212 (previous and next), lines 86-93 and 281-290 (title buttons)
  - `browser\parts\activitybar\activitybarPart.ts` lines 484-601 (Activity Bar Position submenu items, and the context item that appears for this part at lines 592-601)
  - `browser\parts\paneCompositeBar.ts` lines 157-213 (per-tab menu: Move To, Reset Location)
  - `browser\workbench.contribution.ts` lines 618-622 (`showLabels`) and lines 650-668 (`activityBar.location`, `autoHide`)
  - `browser\parts\auxiliarybar\media\auxiliaryBarPart.css` (title row and tab styles), `browser\parts\views\media\views.css` lines 286-290 (filter box width in this title row)
- Notes: the title row also shows the active view's own toolbar on the right. That toolbar is owned by the view, not by this part.

### AX-5 Moving views and containers into it

- Plain name: Moving views and containers into it
- One sentence: The ways to put a view or a whole view container into the secondary side bar (drag, menus, commands, or an extension) and to send it back.
- Tag: both (cursor: searched, all menu and command ids found; extension contribution is blocked, see the Cursor file)
- Evidence: source
- Boundary: no
- Example features:
  - Drag and drop (drag a view header, a tab, or the container title label onto the part; shows a drop color; empty part says "Drag a view here to display."; dropping on a tab reorders)
  - Move Focused View (`workbench.action.moveFocusedView`, Command Palette; offers "New Secondary Side Bar Entry" and any existing container in this part)
  - Move To submenu on a tab right-click (Primary Side Bar, Panel, Secondary Side Bar) and Reset Location
  - Move whole sets: Move Panel Views To Secondary Side Bar (`workbench.action.movePanelToSecondarySideBar`) and Move Secondary Side Bar Views To Panel (`workbench.action.moveSecondarySideBarToPanel`)
- Proof paths:
  - `browser\parts\paneCompositePart.ts` (drop target, drop feedback color and empty message lines 255-329, draggable title label lines 369-378)
  - `browser\parts\compositeBar.ts` lines 36-137 (`CompositeDragAndDrop`: what can be dropped and what happens)
  - `browser\parts\paneCompositeBar.ts` lines 157-213 (Move To, Reset Location)
  - `browser\actions\layoutActions.ts` (view picker lists this part lines 880-902, Move Focused View lines 938-1091 with this part at lines 995-1001 and 1045-1064, Reset Focused View Location lines 1096-1135, Reset View Locations at line 773)
  - `browser\parts\panel\panelActions.ts` lines 344-427 (move panel views to and from this part)
  - `api\browser\viewsExtensionPoint.ts` lines 74-76 and 310-324 (extension point `viewsContainers.secondarySidebar`)
  - `contrib\quickaccess\browser\viewQuickAccess.ts` lines 154 and 169 (Open View lists a "Secondary Side Bar" group)
  - `services\views\browser\viewsService.ts` lines 434-462 and 548 (View menu group `4_auxbar` for containers that default to this part)
  - `services\views\browser\viewDescriptorService.ts` lines 354-360 (`moveViewContainerToLocation`), lines 898-922 (Reset Location item on the container title menu)
- Notes: this group is the only way anything other than Chat gets into this part in V, since no built-in extension targets it. Chat itself is left out (see below).

## Left out and why

- Chat and everything under `contrib\chat`. Out of scope per the brief. The part's default occupant, its title buttons when Chat is showing, and the agent session projection code that reads `AuxiliaryBar` (`contrib\chat\browser\agentSessions\experiments\agentSessionProjectionService.ts`, `chatViewPane.ts`, `chatInputPart.ts`) were not read.
- `contrib\chat\browser\actions\chatActions.ts` lines 1804-1822: a Chat-menu toggle that writes `workbench.secondarySideBar.defaultVisibility`. It is Chat UI. The setting itself is covered in AX-1.
- `contrib\welcomeAgentSessions`: it references this part for agent sessions startup. Agent flow, left out.
- Test fixtures under `workbench\test\browser\componentFixtures` that mention the part. Not UI.
- The Agents window and sessions window variants (`IsSessionsWindowContext`, the `agentsWindow:` markers on the settings). Out of scope per the brief.
- Telemetry for the setting (`contrib\telemetry\browser\telemetry.contribution.ts` lines 387-393). Not UI.

## Newer than 1.128

Found by diffing the three part files and searching N. Mark all as `newer-than-1.128`.

- A new context key `secondarySideBarVisible` ("Whether the layout surface representing the secondary side bar is visible"), N `src\vs\workbench\common\contextkeys.ts` line 179. The toggle command now shows its checked state from it (N `auxiliaryBarActions.ts` line 39).
- A new layout service pair `isSecondarySideBarVisible` and `toggleSecondarySideBar`, N `src\vs\workbench\browser\layout.ts` lines 2348-2358. The toggle command now calls it and the screen reader announcement moved into it (N `auxiliaryBarActions.ts` line 71).
- The View menu Appearance entry for the toggle is now hidden in the sessions window (`when: IsSessionsWindowContext.negate()`), N `auxiliaryBarActions.ts` lines 62-66.
- The title row toolbar now collapses into an overflow menu when narrow (`getToolbarResponsiveBehavior`, `overflowFrom: 'start'`), N `auxiliaryBarPart.ts` lines 164-174. Related CSS: the title label gets `max-width: 40%` and the title actions area is `flex: 1`, N `media\auxiliaryBarPart.css`.
- No new settings under `workbench.secondarySideBar.*` and no new commands with `AuxiliaryBar` in the id in N. The command id list is the same as V.

## Unverified

- I did not run VS Code. Everything here is from reading source. For example, I did not confirm by clicking that the Hide X vanishes when the activity bar location is top.
- `workbench.secondarySideBar.forceMaximized` is tagged experimental in V. I did not check how it behaves when the primary side bar is on the right.
- I did not read how the shared base class (`browser\parts\paneCompositePart.ts` beyond lines 255-380) lays out the title row, so the exact pixel layout is not recorded.
- Whether closing the only view in this part also hides the part itself (see AX-1 notes).

## Pointers to other sections

- Title bar layout toggle buttons for the secondary side bar (`MenuId.LayoutControlMenu`, shown when `workbench.layoutControl.type` is `toggles` or `both`) live in the title bar section. Menu entries at `auxiliaryBarActions.ts` lines 141-179.
- Customize Layout picker (`layoutActions.ts` line 1368) belongs to the quick pick and layout controls, not to this part.
- The shared pane composite and tab strip code is also used by the Side Bar and Panel. Group AX-4 and AX-5 behaviors repeat there (see the side bar and panel files). Only this part's labels and settings are listed here.

*2026-10-25 - claude-sonnet-5-5*
