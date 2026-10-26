# Side bar (SB) - VS Code 1.128.0 findings

Section: Side bar (SB). Date: 2026-10-25. Pass: 1.

- Scope of this file: stock VS Code 1.128.0 only (root V). The Cursor delta is in the sibling file under `cursor\`.
- Level: minor groups only. No full feature lists, no pieces.
- Short names: V = `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0`. N = `C:\Users\Vinnie\src\vscode` (1.144.0, used only to spot newer things). C = the Cursor install `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`.
- Proof paths start under `V\src\vs\workbench\` unless they begin with `ext:` (meaning `V\extensions\`) or `platform:` (meaning `V\src\vs\platform\`).
- Line numbers are from the files as read. They can drift a line or two, so search the quoted id or label if one does not match.
- Tags in this file were set by checking each group's command ids, view ids or settings against the Cursor bundle and the Cursor `extensions` folder. The details of what differs are in the Cursor file.

## What this section is

The primary side pane, left by default and movable to the right. It shows one view container at a time, and a container holds one or more stacked, collapsible view panes. The container is chosen from the activity bar (or from tabs in the side bar title when the activity bar is moved to top or bottom).

Containers registered in the Sidebar location in V, with their views:

- Explorer (`workbench.view.explorer`, order 0)
  - Views: Open Editors (`workbench.explorer.openEditorsView`, hidden by default); Folders (`workbench.explorer.fileView`, cannot be hidden); No Folder Opened (`workbench.explorer.emptyView`, only with no workspace); Outline (`outline`, collapsed); Timeline (`timeline`, collapsed, only when a timeline provider exists)
  - Declared in: `contrib\files\browser\explorerViewlet.ts` lines 108-153, 257-275; `contrib\outline\browser\outline.contribution.ts` lines 26-40; `contrib\timeline\browser\timeline.contribution.ts` lines 28-62
- Search (`workbench.view.search`, order 1)
  - Views: Search (`workbench.view.search`)
  - Declared in: `contrib\search\browser\search.contribution.ts` lines 54-82
- Source Control (`workbench.view.scm`, order 2)
  - Views: Repositories (`workbench.scm.repositories`, hidden by default); Changes (`workbench.scm`); Graph (`workbench.scm.history`, only when a history provider exists)
  - Declared in: `contrib\scm\browser\scm.contribution.ts` lines 56-68, 90-143; ids in `contrib\scm\common\scm.ts` lines 21-24
- Run and Debug (`workbench.view.debug`, order 3)
  - Views: Variables, Watch, Call Stack, Breakpoints, Loaded Scripts (collapsed), plus a Run welcome view when the debug UX is `simple`
  - Declared in: `contrib\debug\browser\debug.contribution.ts` lines 453-475; ids in `contrib\debug\common\debug.ts` lines 34-43
- Testing (`workbench.view.extension.test`, order 6)
  - Views: Test Explorer (`workbench.view.testing`); Test Coverage (`workbench.view.testCoverage`, only while a coverage run is open)
  - Declared in: `contrib\testing\browser\testing.contribution.ts` lines 62-135; ids in `contrib\testing\common\constants.ts` lines 10-24
- References (`references-view`, no order set)
  - Views: Reference Search Results (`references-view.tree`, only when `reference-list.isActive`)
  - Declared in: `ext:references-view\package.json` lines 49-65 (the container is declared under `activitybar`, which lands in the Sidebar location)

Every view above sets `canMoveView: true` in source except the Explorer empty view, the Debug welcome view and the References view (declared by an extension manifest, where the default was not read). So the main views can be dragged to the Panel or the Secondary Side Bar and back. This section catalogs them here regardless of where a user parks them. Evidence: `workbench\browser\parts\views\viewPaneContainer.ts` (drag and drop, lines 62-72, 797-897), `workbench\browser\actions\layoutActions.ts` (Move View, line 797).

Other containers exist in the Sidebar location in V but are hidden when empty or out of scope. They are listed under left out: Extensions, Remote Explorer, Settings Sync data, Edit Sessions, Voice Transcripts, Voice Event Stream.

## Proposed minor groups

Count: 7. Reason: the section is one shared frame plus six things a person opens separately by clicking a different icon (Explorer, Search, Source Control, Run and Debug, Testing, and the small helper views). Outline and Timeline live inside the Explorer container in V, but a person thinks of them with References as "tell me about this file or symbol", so they are grouped together instead of being buried in Explorer. Debug and Testing are kept apart because they are separate containers with separate icons and separate command families. Six would force merging Debug with Testing, which hides the fact that Testing has its own filters, profiles, and coverage. Eight would split the frame or References for no gain. This is above the usual 3 to 6 but within the allowed 8. Confidence: high - the split follows the container boundaries in the source.

### SB-1 Side bar frame and shared view behavior

- Plain-words name: Side bar frame and shared view behavior
- One sentence: The pane itself (its title, width, side, show and hide) and the behaviors every view inside it shares, such as collapsible headers, the "..." menu, welcome text, and dragging a view to another place.
- Tag: both (the stock ids are all in the Cursor bundle, but Cursor rebuilds the frame by default, see the Cursor file)
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Toggle Primary Side Bar Visibility (Ctrl+B; id `workbench.action.toggleSidebarVisibility`), plus the side choice `workbench.sideBar.location` (left or right; ids `workbench.action.moveSideBarRight`, `moveSideBarLeft`, `toggleSidebarPosition`)
  - View pane header: collapse twisty, title, inline actions, and the "..." menu (inline actions show on hover or focus unless `workbench.view.alwaysShowHeaderActions` is on)
  - Move View (`workbench.action.moveView`), drag a view header to another container or to a panel, Reset View Locations (`workbench.action.resetViewLocations`), and per-view Move Up, Down, Left, Right (`views.moveViewUp` and the three others)
  - Welcome content shown in an empty view, for example the Explorer "You have not yet opened a folder" text with Open Folder and Open Recent buttons
- Also seen in this group, not counted as examples: Focus Side Bar (`workbench.action.focusSideBar`), Close Side Bar (`workbench.action.closeSidebar`), the "Views" submenu on the container title (lists every view in the container so each can be shown or hidden), the activity bar tabs shown inside the side bar title when `workbench.activityBar.location` is `top` or `bottom`, and the minimum width of 170 pixels with snap-to-hide.
- Where it is reachable from: View menu, activity bar (right-click), the "..." on any view header, drag and drop.
- Proof paths:
  - `browser\parts\sidebar\sidebarPart.ts` lines 40 (active container key), 44-65 (min width 170, snap, preferred width at least 300), 97-99 (Sidebar location and `MenuId.SidebarTitle`), 116-126 (auto-hide tracking, activity bar location listeners)
  - `browser\parts\sidebar\sidebarActions.ts` lines 22, 39 (close and focus)
  - `browser\actions\layoutActions.ts` lines 98-143 (side position actions), 288-307 (toggle visibility, Ctrl+B), 779-800 (reset view locations, move view)
  - `browser\workbench.contribution.ts` lines 566 (`workbench.sideBar.location`), 656-679 (`workbench.activityBar.location` and related), 686-693 (`workbench.view.alwaysShowHeaderActions`, `workbench.view.showQuietly`)
  - `browser\parts\views\viewPane.ts` lines 59-64 (when header actions show), 101 (welcome controller), 311-313 (pane class, always-show setting), 406 (view title menu)
  - `browser\parts\views\viewPaneContainer.ts` lines 47-51 (Views submenu), 62-72 (drop directions), 1230-1294 (move view actions)
  - `browser\parts\views\viewMenuActions.ts` line 90 (container title menu); `treeView.ts` (extension-provided tree views); `viewFilter.ts` (filter box inside a pane header)
  - `contrib\files\browser\explorerViewlet.ts` lines 286-350 (welcome content for the empty Explorer)

### SB-2 Explorer (files and open editors)

- Plain-words name: Explorer
- One sentence: The file tree for the open folder or workspace, with the right-click file operations, the Open Editors list, and the empty "no folder opened" state.
- Tag: both
- Evidence: source
- Boundary: no (Cursor adds a few right-click items that hand a file to the agent or Browser, see the Cursor file)
- Examples (up to 4):
  - Folders tree with four inline buttons in its header: New File, New Folder, Refresh Explorer, Collapse Folders in Explorer (ids `workbench.files.action.createFileFromExplorer`, `createFolderFromExplorer`, `refreshFilesExplorer`, `collapseExplorerFolders`)
  - Right-click file menu: Open to the Side (`explorer.openToSide`), Open With (`explorer.openWith`), Select for Compare and Compare Selected (`selectForCompare`, `compareSelected`), Cut, Copy, Paste, Copy Path and Copy Relative Path (`copyFilePath`, `copyRelativeFilePath`), Rename (`renameFile`), Delete (`moveFileToTrash`, `deleteFile`)
  - Open Editors pane (hidden by default; Ctrl+K E focuses it; header buttons Save All, Close All, New Untitled Text File, and a Toggle Vertical/Horizontal Editor Layout button when there are several editor groups)
  - Tree behaviors set in settings: drag and drop to move or copy files, auto reveal the active file, compact folders, file nesting, sort order, file decoration colors and badges (`explorer.enableDragAndDrop`, `explorer.autoReveal`, `explorer.compactFolders`, `explorer.fileNesting.enabled`, `explorer.sortOrder`, `explorer.decorations.colors`)
- Also seen in this group, not counted as examples: the "No Folder Opened" welcome view with Open Folder, Open Recent, Add Folder buttons (5 variants by workspace state); Add Folder to Workspace and Remove Folder from Workspace on root folders; Reveal in File Explorer and Open in Integrated Terminal (desktop only); Find in Folder (hands off to the Search view); Open Timeline on a file (hands off to the Timeline view); Download (remote or web) and Upload (web only); inline new-file and rename text box in the tree.
- Where it is reachable from: Ctrl+Shift+E (id `workbench.view.explorer`), View menu "Explorer", activity bar icon.
- Proof paths:
  - `contrib\files\browser\explorerViewlet.ts` lines 63-101 (which views register by workspace state), 108-153 (Open Editors, empty view, Folders descriptors), 257-275 (container, Ctrl+Shift+E at 270), 286-350 (welcome content)
  - `contrib\files\browser\views\explorerView.ts` lines 154 (view class), 1047-1128 (four header buttons); `explorerViewer.ts` (tree, drag and drop, rename); `emptyView.ts` lines 26-29; `openEditorsView.ts` lines 62-67, 428, 845-945 (header buttons)
  - `contrib\files\browser\fileActions.contribution.ts` lines 294-470 (Open Editors context menu), 476-668 (Explorer context menu groups `navigation`, `3_compare`, `5_cutcopypaste`, `6_copypath`, `2_workspace`, `7_modification`)
  - `contrib\files\electron-browser\fileActions.contribution.ts` (three Explorer context items, desktop only)
  - Settings: `contrib\files\browser\files.contribution.ts` lines 421-640 (`explorer.*`)
  - Menu ids: `platform\actions\common\actions.ts` (`ExplorerContext`, `OpenEditorsContext`)

### SB-3 Search

- Plain-words name: Search
- One sentence: Find and replace text across all files in the workspace, with include and exclude filters, a results tree, and a way to open the results in an editor tab.
- Tag: both for the view and its core actions. The "Search with AI" row is vscode only (ids not found in the Cursor bundle).
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Search box with Match Case, Match Whole Word and Use Regular Expression toggles (Ctrl+Shift+F, id `workbench.action.findInFiles`; toggles `toggleSearchCaseSensitive`, `toggleSearchWholeWord`, `toggleSearchRegex`)
  - Replace row with Preserve Case, Replace, and Replace All (Ctrl+Shift+H, id `workbench.action.replaceInFiles`; `search.action.replaceAllInFile`, `togglePreserveCase`)
  - Search details panel with "files to include" and "files to exclude" boxes and the "use exclude settings and ignore files" toggle (id `workbench.action.search.toggleQueryDetails`)
  - Results tree with header buttons Refresh, Clear Search Results, Collapse All or Expand All, View as Tree or View as List, and Open in Editor (`search.action.openInEditor`, which opens a Search Editor)
- Also seen in this group, not counted as examples: Search with AI (`search.action.searchWithAI`) and AI results section; right-click on a result (Dismiss, Copy, Open to the Side, Replace); Cancel Search; Clear Search History; Find in Folder and Find in Workspace entries from the Explorer menu (`filesExplorer.findInFolder`, `filesExplorer.findInWorkspace`); notebook results (`contrib\search\browser\notebookSearch`); search-on-type toggle (`workbench.action.toggleSearchOnType`).
- Overlap note: the Search Editor itself (a results page that opens in the editor area, ids `search.action.openNewEditor`, `searchEditor.action.*`) is an editor, so the editor section may catalog it. Here only the hand-off from the Search view is counted. Source is still listed in the proof paths because the brief asks for it.
- Where it is reachable from: Ctrl+Shift+F, Edit menu "Find in Files" and "Replace in Files", activity bar icon, Explorer right-click Find in Folder.
- Proof paths:
  - `contrib\search\browser\search.contribution.ts` lines 54-82 (container, view, Ctrl+Shift+F at 73), 106-340 (`search.*` settings including `search.defaultViewMode`, `search.sortOrder`, `search.collapseResults`, `search.actionsPosition`, `search.searchView.semanticSearchBehavior`, `search.searchView.keywordSuggestions`)
  - `contrib\search\common\constants.ts` lines 8-56 (command ids)
  - `contrib\search\browser\searchView.ts` (the view; `MenuId.SearchContext` at 1106), `searchWidget.ts` (input and replace row), `searchFindInput.ts`, `patternInputWidget.ts` (include and exclude boxes), `searchResultsView.ts` (tree), `replace.ts`, `replaceService.ts`
  - `contrib\search\browser\searchActionsTopBar.ts` lines 30-212 (header buttons incl. Search with AI at 212); `searchActionsFind.ts` lines 39-116 and 241-270 (Find in Files, Find in Folder); `searchActionsNav.ts` line 402 (Ctrl+Shift+H); `searchActionsRemoveReplace.ts`; `searchActionsCopy.ts`
  - `contrib\search\browser\AISearch\`, `notebookSearch\` (sub-features)
  - `contrib\searchEditor\browser\searchEditor.contribution.ts` lines 66-90 (settings), 284, 511-603 (editor actions, `search.action.openNewEditorFromView` at 602); `searchEditor.ts`, `searchEditorActions.ts`
  - `ext:search-result\package.json` (syntax coloring for Search Editor output)

### SB-4 Source Control

- Plain-words name: Source Control
- One sentence: The git panel where you see changed files, stage them, type a commit message and commit, switch between repositories, and browse the commit graph.
- Tag: both (the git extension and the SCM view containers are in Cursor; a few pieces are vscode only, see Also seen)
- Evidence: source
- Boundary: no in stock VS Code (Cursor adds agent-driven items here, see the Cursor file)
- Examples (up to 4):
  - Changes view: message box with a Commit button (Ctrl+Enter, id `scm.acceptInput`), groups for Staged Changes, Changes, Merge Changes and Untracked Changes, list or tree layout (`scm.defaultViewMode`) and sort (`scm.defaultViewSortKey`)
  - Title bar of the view: Commit, Refresh, and the "..." menu with Pull, Push, Clone, Checkout to..., Fetch and submenus Commit, Changes, Pull/Push, Branch, Remote, Stash, Tags, Worktrees (all from the git extension)
  - Repositories view (hidden by default; appears when there is a repository) listing each repo and its branch, with Close Repository and Open Worktree actions
  - Graph view: history of the current branch with incoming and outgoing changes, branch and tag badges, and a right-click menu per commit (Checkout (Detached), Create Branch, Create Tag, Cherry Pick, three Compare entries, Copy Commit Hash, Copy Commit Message)
- Also seen in this group, not counted as examples: Sync Changes and Publish Branch action buttons below the message box (`scm.showActionButton`); right-click on a changed file (Open Changes, Stage, Discard, Open File and more; 46 entries in `scm/resourceState/context`); the "No source control providers registered" and Restricted Mode welcome texts; Source Control Repositories view stash, branch, tag and worktree tree (`scm.repositories.explorer`, experimental, vscode only); repository selection mode (`scm.repositories.selectionMode`, vscode only); Open in Terminal on a repository (`scm.openInTerminal`).
- Cross-section notes: the editor gutter change bars (`scm.diffDecorations`) and the status bar branch item belong to the Editor and Status bar sections. The Git Blame editor decoration is the Editor section.
- Where it is reachable from: Ctrl+Shift+G (Control+Shift+G on Mac), View menu "Source Control", activity bar icon with a change-count badge (`scm.countBadge`).
- Proof paths:
  - `contrib\scm\browser\scm.contribution.ts` lines 56-68 (container), 70-88 (welcome texts), 90-143 (Repositories, Changes, Graph views), 172-420 (all `scm.*` settings), 421-425 (Ctrl+Enter), 570-597 (open in terminal)
  - `contrib\scm\browser\scmViewPane.ts` (Changes view, list or tree), `scmViewPaneContainer.ts`, `scmInput.ts` (message box, `MenuId.SCMInputBox`), `scmRepositoriesViewPane.ts` (Repositories view, artifact rows at `scm-artifact-group`), `scmHistoryViewPane.ts` (Graph), `scmRepositoryRenderer.ts`, `workingSet.ts`, `menus.ts`
  - `contrib\scm\common\scm.ts` lines 21-24 (view ids)
  - Menu ids: `platform\actions\common\actions.ts` lines 149-165 (SCM menu ids)
  - `ext:git\package.json` line 76 (183 commands), 1222 (menus), 1925-2006 (`scm/title`), 2007 (`scm/repositories/title`), 2086 (`scm/sourceControl`), 2135 (`scm/artifact/context`), 2268 (`scm/resourceGroup/context`), 2437 (`scm/resourceState/context`), 2669 (`scm/history/title`), 2691 (`scm/historyItem/context`), 3244-3283 (8 submenus), 3283 (104 settings under `configuration`), 4274 (`viewsWelcome`)
  - `ext:git\src\historyProvider.ts`, `artifactProvider.ts` (feed Graph and Repositories view)

### SB-5 Run and Debug

- Plain-words name: Run and Debug
- One sentence: The debugging side bar that holds the start button and launch configuration picker, then shows variables, watches, the call stack and breakpoints while a program is stopped.
- Tag: both (some V-only pieces, see Also seen)
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Start Debugging button with a launch configuration dropdown, plus the gear that opens `launch.json` (F5; ids `workbench.action.debug.start`, `workbench.action.debug.configure`, `workbench.action.debug.selectandstart`)
  - Variables and Watch views (focus ids `workbench.debug.action.focusVariablesView`, `focusWatchView`; Watch has Add Expression, Remove All, Copy All)
  - Call Stack view with sessions, threads and frames, and inline row buttons for Pause, Continue, Restart and Stop (menu `DebugCallStackToolbar`)
  - Breakpoints view with line, function, data, instruction and exception breakpoints, plus Add Function Breakpoint, Toggle Activate Breakpoints, Remove All (ids `workbench.debug.viewlet.action.addFunctionBreakpointAction`, `toggleBreakpointsActivatedAction`, `removeAllBreakpoints`)
- Also seen in this group, not counted as examples: Loaded Scripts view (collapsed, only for adapters that support it); the Run welcome view with a Run and Debug button and a "create a launch.json file" link (only in the `simple` debug UX before a launch config exists, plus an "all debug extensions are disabled" text); the open-the-view-on-start setting `debug.openDebug`; the Breakpoints list-or-tree presentation toggle (`debug.breakpointsView.presentation`, vscode only); "Add to Chat" rows on variables, scopes and watches (vscode only, chat integration, out of scope).
- Left to other sections (do not count here): Debug Console (Panel), floating debug toolbar, Disassembly view (editor), hover, inline breakpoint widgets and the exception widget (editor), and the debug status bar color (status bar).
- Where it is reachable from: Ctrl+Shift+D (id `workbench.view.debug`), Run menu, activity bar icon.
- Proof paths:
  - `contrib\debug\browser\debug.contribution.ts` lines 453-475 (container and five views plus welcome view), 514-664 (`debug.*` settings including `debug.openDebug` 563, `debug.toolBarLocation` 535, `debug.breakpointsView.presentation` 641)
  - `contrib\debug\browser\debugViewlet.ts` lines 40-120 (container class, start button view item), 189-197 (start and configure actions)
  - `contrib\debug\browser\variablesView.ts`, `watchExpressionsView.ts`, `callStackView.ts`, `breakpointsView.ts`, `loadedScriptsView.ts`, `welcomeView.ts`, `debugActionViewItems.ts`
  - `contrib\debug\browser\debugCommands.ts` (start, run, configure ids); `contrib\debug\common\debug.ts` lines 34-43 (view ids)
  - Menu ids: `platform\actions\common\actions.ts` lines 70-82 (`DebugBreakpointsContext`, `DebugCallStackContext`, `DebugVariablesContext`, `DebugWatchContext`, `DebugScopesContext`)

### SB-6 Testing

- Plain-words name: Testing
- One sentence: The test side bar that lists tests from test extensions, runs or debugs them, and shows code coverage for the last run.
- Tag: both
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Test Explorer tree with Run All Tests, Debug All Tests and Run All Tests with Coverage (ids `testing.runAll`, `testing.debugAll`, `testing.coverageAll`)
  - Filter box and state filter, View as Tree or View as List, Sort by Status, Location or Duration (ids `testing.viewAsTree`, `testing.viewAsList`, `testing.sortByStatus`, `testing.sortByLocation`, `testing.sortByDuration`)
  - Per-test row buttons and menu: run, debug, run with coverage, hide test, go to test, Toggle Continuous Run (ids `testing.run`, `testing.debug`, `testing.hideTest`, `testing.toggleContinuousRunForTest`)
  - Test Coverage view (appears while a coverage result is open) with a toggle to show coverage in the Explorer and a coverage bar per file (ids `workbench.view.testCoverage`, `testing.toggleCoverageInExplorer`)
- Also seen in this group, not counted as examples: Refresh Tests, Cancel Test Run, Clear All Results, Configure Test Profiles and Select Default Profile (`testing.configureProfile`, `testing.selectDefaultTestProfiles`); the welcome text with a "Install Additional Test Extensions" link (`testing.searchForTestExtension`); Unhide All Tests.
- Left to other sections: Test Results (Panel, `workbench.panel.testResults`), inline test decorations and gutter run buttons, peek output, and coverage line decorations (editor).
- Where it is reachable from: View menu "Testing" (no default key), activity bar icon (shows only when a test provider exists).
- Proof paths:
  - `contrib\testing\browser\testing.contribution.ts` lines 62-78 (container), 93-135 (views and welcome content)
  - `contrib\testing\browser\testingViewPaneContainer.ts`, `testingExplorerView.ts` lines 92 (view), 547 (result summary), 689 (view model), `testExplorerActions.ts`, `testingExplorerFilter.ts`, `testCoverageView.ts`, `explorerProjections\`
  - `contrib\testing\common\constants.ts` lines 10-24 (view ids), 25-35 (view mode and sorting), 57-120 (command ids)
  - Menu ids: `platform\actions\common\actions.ts` lines 171-175 (`TestItem`, `TestItemGutter`, `TestProfilesContext`)

### SB-7 Outline, Timeline and References

- Plain-words name: Outline, Timeline and References
- One sentence: Small helper views that show the symbols of the open file, the saved history of the open file, and the results of Find All References and the call and type hierarchies.
- Tag: both
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Outline view (a pane under the Explorer tree) with Follow Cursor, Filter on Type, Sort By Position, Name or Category, Collapse All, Expand All (focus id `outline.focus`; settings `outline.icons`, `outline.collapseItems`, `outline.problems.*`, and 26 `outline.show*` kind filters)
  - Timeline view (a pane under Outline) listing Git History commits and Local History saves for the open file, with Refresh, a Filter Timeline submenu to hide sources, Follow Active Editor, and Load More on scroll (focus id `timeline.focus`; `timeline.pageSize`, `timeline.pageOnScroll`)
  - Reference Search Results view (own container with a References icon; appears after Find All References or Find All Implementations) with Clear, Refresh, Rerun, Copy, Dismiss, and a history picker (`references-view.findReferences`, `references-view.clear`, `references-view.pickFromHistory`)
  - Call Hierarchy and Type Hierarchy shown in the same References view (Show Incoming Calls, Show Outgoing Calls, Show Supertypes, Show Subtypes; ids `references-view.showCallHierarchy`, `references-view.showTypeHierarchy`)
- Also seen in this group, not counted as examples: Open Timeline on a file from the Explorer menu (`files.openTimeline`); `references.preferredLocation` (peek or view for CodeLens references); next and previous reference keys (`references-view.next`, `references-view.prev`); Timeline item right-click entries from the git extension (7 items under `timeline/item/context`).
- Where it is reachable from: Outline and Timeline are panes in the Explorer, shown or hidden from the Explorer "..." menu; References appears from Find All References (Shift+Alt+F12, id `references-view.findReferences`) and from Show Call Hierarchy (Shift+Alt+H); F4 and Shift+F4 step through results.
- Proof paths:
  - `contrib\outline\browser\outline.contribution.ts` lines 22-40 (view descriptor, id `outline` in `outline.ts`, order 2, collapsed), 43-80 (settings); `outlineActions.ts` lines 22-160 (view title actions); `outlinePane.ts`
  - `contrib\timeline\browser\timeline.contribution.ts` lines 28-62 (descriptor, settings), 66-105 (Open Timeline, Filter Timeline submenu); `timelinePane.ts` lines 1283-1360 (refresh, follow active editor, per-source exclude actions); `contrib\timeline\common\timeline.ts` line 12 (pane id `timeline`)
  - `contrib\localHistory\browser\localHistoryTimeline.ts` (Local History source, id `timeline.localHistory`); `ext:git\src\timelineProvider.ts` (Git History source, id `git-history`); `ext:git\package.json` line 2906 (`timeline/item/context`)
  - `ext:references-view\package.json` lines 32-48 (setting), 49-66 (container and view), 67-177 (21 commands), 178-373 (menus), 374 (keybindings)

## Built-in extension contributions that belong in this section

Counts are entries in the V (1.128.0) manifests. Labels are in the `package.nls.json` next to each manifest.

- git (`ext:git\package.json`): 183 commands (line 76), 3 keybindings (line 1202), 8 submenus plus 1 extra stash submenu (lines 3244-3283), 104 settings (line 3283 on), 19 welcome texts (line 4274; 17 for the `scm` view, 2 for the `explorer` view). It declares no view or view container of its own. It fills the core Source Control containers (SB-4) through these menu ids: `scm/title` 16 items, `scm/repositories/title` 1, `scm/sourceControl` 5, `scm/artifactGroup/context` 4, `scm/artifact/context` 26, `scm/resourceGroup/context` 19, `scm/resourceFolder/context` 14, `scm/resourceState/context` 46, `scm/history/title` 4, `scm/historyItem/context` 9, `scm/historyItemRef/context` 3, and the eight `git.*` submenus (commit, changes, pull/push, branch, remotes, stash, tags, worktrees). Also `timeline/item/context` 7 items (SB-7) and `viewsWelcome` for the Explorer (Clone Repository text in the empty Explorer, SB-2). Menus on `editor/*`, `scm/change/title`, `multiDiffEditor/*` and `diffEditor/*` belong to the Editor section.
- github (`ext:github\package.json`): only two `viewsWelcome` entries on the `scm` view (publish to GitHub texts) are in this section. Its 9 commands (copy vscode.dev link, open pull request and so on) are not side bar UI.
- references-view (`ext:references-view\package.json`): the only built-in extension with its own side bar container. 1 container (`references-view`), 1 view (`references-view.tree`), 21 commands (line 67), menus (line 178), 4 keybindings (line 374: Find All References Shift+Alt+F12, Next F4, Previous Shift+F4, Call Hierarchy Shift+Alt+H), 1 setting `references.preferredLocation` (line 32). Covers SB-7.
- search-result (`ext:search-result\package.json`): no side bar UI. It colors Search Editor output. Listed under SB-3 for cross-reference only.
- merge-conflict, simple-browser, markdown-language-features: no side bar UI. They belong to the Editor section.

## Left out and why

- Extensions view and container (`workbench.view.extensions`, `contrib\extensions\browser\extensionsViewlet.ts`): out of scope by the brief.
- Remote Explorer (`workbench.view.remote`, `contrib\remote\browser\remote.ts`) and the Ports view: shown only when a remote extension is present; not in the brief's list. The parent can add a group later if wanted. Confidence it should stay out: medium - it is side bar UI, but it only appears with extra extensions installed.
- Settings Sync data views (`contrib\userDataSync\browser\userDataSync.ts`, container `workbench.view.sync`) and Edit Sessions (`contrib\editSessions\browser\editSessions.contribution.ts`, `workbench.view.editSessions`): both are hidden when empty and appear only for signed-in sync features. Left out for the same reason as Remote Explorer.
- Voice Transcripts and Voice Event Stream containers (`contrib\agentsVoice\browser\transcriptsView\`): voice and agent features, out of scope.
- NPM Scripts pane (`ext:npm\package.json` view `npm` in the `explorer` container, hidden by default, gated by `npm:showScriptExplorer`): it is a real Explorer pane, but the npm extension is not in the brief's built-in extension list. Left out; the parent can decide to fold it into SB-2.
- Chat hooks inside side bar views: debug "Add to Chat" actions (`debugChatIntegration.ts`), Source Control Graph "Add to Chat" and "Summarize History Item" (`scmHistoryChatContext.ts`), and search chat context (`searchChatContext.ts`). Chat is out of scope.
- Other sections: Debug Console and Test Results (Panel), floating debug toolbar, Disassembly view, debug hover and breakpoint widgets (Editor), editor gutter change bars (Editor), status bar branch and debug items (Status bar), the Search Editor page itself (Editor).

## Newer than 1.128

Checked by comparing the same source folders, menu ids and manifests in N (1.144.0) against V. All items here are tagged `newer-than-1.128` and must not be assumed to exist in Cursor.

- `newer-than-1.128` Timeline "Pin Workspace Timeline" (`timeline.showWorkspaceTimeline`) and "Compare Selected" (`timeline.compareSelected`), plus the strings "Workspace" and "Loading workspace timeline...". Source: `N\src\vs\workbench\contrib\timeline\browser\timelinePane.ts`. Not present in V.
- `newer-than-1.128` git setting `git.worktreeSymlinkFolders`. Source: `N\extensions\git\package.json` (105 settings against 104 in V).
- `newer-than-1.128` cosmetic: `SidebarPart.setBoundarySashes` adds a CSS class `primary-sidebar-sash` to the side bar resize handle. Source: `N\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts` (355 lines against 342 in V).
- Nothing else new in the Explorer, Search, Source Control, Debug, Testing or Outline folders: same file lists, same command ids and same `explorer.*`, `search.*`, `scm.*`, `debug.*`, `testing.*`, `outline.*` setting keys apart from the items above. The three new menu ids in N (`AgentSessionsExternalFilterSubMenu`, `DiffEditorViewSubmenu`, `TitleBarUpdate`) are not side bar menus.
- Gone from N (not newer, listed so nobody is surprised): the settings `search.location`, `search.maintainFileSearchCache`, `search.useRipgrep`, `testing.automaticallyOpenResults`, and the npm setting `npm.enableScriptExplorer` were not found in N. They exist in V.

## Could not verify

- Nothing was run. Default visibility of a view depends on workspace state and `when` clauses. This file reports the `when` clause, not an observed screen.
- Whether the References container icon stays visible in the activity bar when it has no results: the manifest declares it under `activitybar`, which the extension point maps to the Sidebar location (`api\browser\viewsExtensionPoint.ts`, the `case 'activitybar'` branch), and its only view has the `when` clause `reference-list.isActive`. The visible behavior was not observed.
- Exact keybindings on macOS were not read for every group. Windows and Linux keys are given.
- The `canMoveView` default for extension-declared views (References) was not read.
- Counts of `scm/*` menu items are item counts in the manifest. Several entries repeat the same command under a different `when`, so they overstate the number of distinct buttons.
