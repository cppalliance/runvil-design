# Side bar (SB) - Cursor delta

Section: Side bar (SB). Date: 2026-10-25. Pass: 1.

- Scope of this file: what Cursor 3.27.12 changes, adds or lacks in the side bar compared with stock VS Code 1.128.0. The stock groups SB-1 to SB-7 are defined in the sibling file under `vscode\`. Group ids here point at those.
- Level: minor groups only. Cursor additions are listed by group and plain name. No feature ids are assigned yet, so later passes can number them.
- Short names: V = `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0`. C = `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`. Bundle = `C\out\vs\workbench\workbench.desktop.main.js`. CSS = `C\out\vs\workbench\workbench.desktop.main.css`. NLS = `C\out\nls.messages.json` (the bundle calls `k(N, ...)` or `De(N, "...")` with N as an index into this file, so `k(9305)` means entry 9305).
- Cursor has no source here. Everything below is a string found in the bundle, the stylesheet, or a built-in extension manifest, so the evidence level is `bundle` unless stated.
- Minified names in quotes (for example `Clt`, `OLt`) are only there so a later reader can find the same spot again. They change every build.

## Result in one paragraph

Cursor keeps the same six side bar containers as VS Code (Explorer, Search, Source Control, Run and Debug, Testing, References) with the same view ids, orders and default states. Its biggest change is the frame: by default Cursor replaces the stock activity bar plus side bar with its own side bar that has a row of pinned container tabs across the top (setting `workbench.activityBar.orientation`, default `horizontal`). It adds one new view, "Agent Review", inside the Source Control container. It adds a commit message generator button to the Source Control input box. It adds six right-click items to the Explorer (four "Add ... to Cursor Chat" items, one "Open in Browser", one Cursor Blame item), and one git setting that lists Cursor worktrees in the Repositories view. No Explorer decorations, indexing or docs views, codebase status panes, or checkout views were found in the side bar. Cursor's core is also behind V in a few places (Repositories artifacts, Graph incoming and outgoing, Breakpoints tree presentation, AI search), listed below.

## Cursor additions for this section

Each item says which stock group it joins. `boundary` follows the brief: in scope, but the result comes from the agent or a Cursor feature the brief names.

### Added to SB-1 Side bar frame and shared view behavior

- Horizontal side bar with a row of pinned view container tabs (the Cursor default)
  - Tag: cursor. Evidence: bundle. Boundary: no.
  - What it is: Cursor ships two side bar implementations and picks one at startup from a new setting `workbench.activityBar.orientation` (values `horizontal` or `vertical`, default `horizontal`, description "Controls the orientation of the activity bar in the workbench. Quit and reopen for changes to take effect.", NLS 4227). `vertical` builds a stock-style side bar with the activity bar down the side. `horizontal` builds a Cursor-made side bar (a Solid.js component, CSS class `sidebar2-compositeComponent`) and hides the stock activity bar (`isActivityBarHidden` returns true for `horizontal`).
  - What the horizontal side bar shows: a title row of icon tabs, one per pinned view container, then the active container below. Default pinned tabs are Explorer, Search, Source Control, Extensions; every other container (Run and Debug, Testing, References and so on) goes into an overflow list. Tabs can be dragged to reorder (order is saved). Right-click on a tab gives "Configure Keybinding" (NLS 3977, 3978) and "Hide '{name}'" or "Keep '{name}'" (NLS 3979, 3980; the last pinned tab cannot be hidden). Right-click on empty space in the row gives "Move Primary Side Bar Right" or "Move Primary Side Bar Left" (NLS 3981 to 3984, command `workbench.action.toggleSidebarPosition`). An overflow menu lists every container with a check mark for the pinned ones. The number of tabs shown depends on the row width (about 40 pixels per tab). The pinned list and order persist under the storage key `sidebar2.sidebarData.memoized.v1`.
  - Two-row title: a container can ask for an extra action row (`wantsActionToolbar`); Run and Debug does, so its title grows from 35 to 70 pixels to fit the start button and configuration picker. V has no such option.
  - Size and header differences from V: minimum width 214 pixels in both Cursor implementations (V: 170); minimum height 77 in the horizontal one. In `horizontal` mode a view pane header hides its collapse twisty when it is the only view in the container.
  - Overlap: this tab row is the Cursor replacement for the Activity Bar, so the activity bar section (AB) will also see it. The two sections should agree on which one catalogs it. Here it is listed because it sits inside the side bar frame.
  - Proof strings:
    - Bundle: `D5e="workbench.activityBar.orientation"`, definition `[D5e]:{type:"string",enum:["horizontal","vertical"],default:"horizontal",description:k(4227,null)}`
    - Bundle: part chooser `i==="vertical"?{case:"vertical",v:s.createInstance(JDt)}:{case:"horizontal",v:s.createInstance(ePt)}` and `isActivityBarHidden(){if(this.configurationService.getValue(D5e)==="horizontal")return!0;...}`
    - Bundle: class `ePt` (`this.minimumWidth=214`, `this.minimumHeight=77`, `partId="workbench.parts.sidebar"`), class `JDt` (`this.minimumWidth=214`), storage key `p6m="sidebar2.sidebarData.memoized.v1"`, default `pinnedViewContainerIDs:[uyk,Rcr,dyk,hyk]` with `uyk="workbench.view.explorer"`, `Rcr="workbench.view.search"`, `dyk="workbench.view.scm"`, `hyk="workbench.view.extensions"`, `heightOfTitle:35` and the switch to 70 when `wantsActionToolbar===!0`
    - Bundle: Run and Debug container `...title:De(5659,"Run and Debug"),...alwaysUseContainerInfo:!0,order:3,wantsActionToolbar:!0`; `wantsActionToolbar` appears 2 times in the bundle and 0 times in V source
    - CSS: `.sidebar2-compositeComponent`, `.sidebar2-item-icon-anysphere`, `.sidebar2-item-icon-anysphere-hover`
    - Header twisty rule: `updateTwistyIcon(){this.twistiesContainer&&(!this.isAlone()||this.configurationService.getValue(D5e)==="vertical"?...`

### Added to SB-4 Source Control

- Agent Review view
  - Tag: cursor. Evidence: bundle. Boundary: yes (review results come from the agent, same product as "Review Changes").
  - What it is: a fourth view in the Source Control container. View name "Agent Review" (NLS 9305), pane title "Review" (NLS 9204), view id `bugbot`, registered at order 2 so it sits between Changes (order 1) and Graph. Graph moves from V's order 2 to order 3 in Cursor.
  - What the body shows (from the Solid-rendered pane, CSS class `scm-review-container`): a split Run button with a chevron ("Start Agent Review", turns into "Stop review" while running, "Reviewing" label), a live status line "Reviewing your changes...", a results list with a status header, "No issues found" text, and a footer button "Fix All Issues" (tooltip "Fix All Issues in Chat"). When other views share the container the pane sizes itself to its content (minimum body 68 pixels).
  - Gating: the view is only registered when a feature gate is on. Registration class id `workbench.contrib.editorBugbotViewGatedRegistration`. The default value of the gate was not found, so the view may not show for every user.
  - Related ids: command `cursor.runEditorBugbot` (also called by `cursor-commits` right after a git commit, under a condition not read). Messages: "Agent Review complete - Found 1 potential issue", "Agent Review cancelled".
  - Proof strings:
    - Bundle: `registerViews([{id:Clt,containerTitle:Hcn,name:De(9305,"Agent Review"),...order:2,containerIcon:wii}],OLt)` where `Clt="bugbot"` and `OLt` is the Source Control container (`id:t$n,title:De(9301,"Source Control")`)
    - Bundle: `pdt.TITLE=De(9204,"Review")`, `pdt.DEFAULT_MINIMUM_BODY_SIZE=68`, `Cii.ID="workbench.contrib.editorBugbotViewGatedRegistration"`
    - Bundle: the label strings "Start Agent Review", "Stop review", "Reviewing your changes...", "Fix All Issues in Chat", "No issues found"
    - CSS: `.scm-review-root`, `.scm-review-content`, `.scm-review-run-split`, `.scm-review-run-split-main`, `.scm-review-run-split-chevron-wrapper`, `.scm-review-run-wide`, `.scm-review-actions-container`
    - Extension: `C\extensions\cursor-commits\dist\main.js` calls `commands.executeCommand("cursor.runEditorBugbot", rootUri, HEAD.commit)`

- Generate Commit Message button in the message box
  - Tag: cursor. Evidence: bundle plus extension manifest. Boundary: yes (a model writes the message).
  - What it is: a sparkle-icon button in the Source Control message box toolbar, shown when the provider is git. Command id `cursor.generateGitCommitMessage`, title "Generate Commit Message", also in the Command Palette. It reads the staged diff (or the working tree diff when nothing is staged) plus recent commit messages, then fills the message box. Errors show as "Failed to generate commit message: ...".
  - Proof strings:
    - Manifest: `C\extensions\cursor-always-local\package.json` (single minified line 1), `contributes.menus["scm/inputBox"]` = `{"command":"cursor.generateGitCommitMessage","when":"scmProvider == git"}`; this extension enables the API proposal `contribSourceControlInputBoxMenu`
    - Bundle: `super({id:"cursor.generateGitCommitMessage",title:{value:"Generate Commit Message",...},f1:!0,icon:Oe.sparkle})`

- Cursor worktrees in the Repositories view
  - Tag: cursor. Evidence: extension manifest. Boundary: yes (likely; the worktrees come from Cursor's agent worktree flow, inferred from bundle strings `composer.openTerminalInWorktree`, `isCreatingWorktree`, `isApplyingWorktree`, not confirmed).
  - What it is: a git setting `git.showCursorWorktrees`, default `false`, scope `window`, description "Show Cursor worktrees in the Source Control Repositories view." When on, worktrees Cursor created appear as rows in the Repositories view. Worktree submenu commands (Create, Open, Delete) are stock and are in V too.
  - Proof: `C\extensions\git\package.json` (single line 1), `contributes.configuration` property `git.showCursorWorktrees`. The bundle has no string for it (0 hits), so the behavior lives in the extension code, not read.

### Added to SB-2 Explorer

- Add to Cursor Chat items in the file right-click menu
  - Tag: cursor. Evidence: bundle. Boundary: yes (hands a file or folder to the chat).
  - What it is: four items in menu group `3_composer`: "Add File to Cursor Chat" (order 20, for files) and "Add Directory to Cursor Chat" (order 21, for folders), both command `composer.addfilestocomposer`; "Add File to New Cursor Chat" (order 25) and "Add Directory to New Cursor Chat" (order 26), both command `composer.addfilestonnewcomposer`. They show for file system resources only.
  - Proof strings: `{id:m0t,title:"Add File to Cursor Chat"}` with `m0t="composer.addfilestocomposer"`, `{id:m0t,title:"Add Directory to Cursor Chat"}`, `{id:JOn,title:"Add File to New Cursor Chat"}` with `JOn="composer.addfilestonnewcomposer"`, `{id:JOn,title:"Add Directory to New Cursor Chat"}`, each appended with `Pe.ExplorerContext` (menu id `ExplorerContext`) and group `"3_composer"`.

- Open in Browser item in the file right-click menu
  - Tag: cursor. Evidence: bundle. Boundary: yes (opens the Cursor Browser, a feature the brief names).
  - What it is: "Open in Browser" (NLS 7047), command `filesExplorer.openFileInBrowser`, menu group `navigation` order 15. Shows for one non-folder `file:` resource whose extension is one of html, htm, xhtml, svg, pdf, txt, md, markdown, json, xml, yaml, yml, csv, log, css, js, cjs, mjs, ts, tsx, jsx, png, jpg, jpeg, gif, webp, bmp, ico, avif. It runs `workbench.action.openBrowserEditor` with the file URL and tracking id `context_menu_new_tab`.
  - Proof strings: `var mnw={id:Vng,title:k(7047,null)}` with `Vng="filesExplorer.openFileInBrowser"`; `qng=/^\.(?:html?|xhtml|svg|pdf|txt|md|markdown|json|xml|ya?ml|csv|log|css|[cm]?js|tsx?|jsx|png|jpe?g|gif|webp|bmp|ico|avif)$/i`; `executeCommand("workbench.action.openBrowserEditor",{url:...,trackingId:"context_menu_new_tab"})`. Stock V has no `openBrowserEditor` string.

- Cursor Blame item in the file right-click menu
  - Tag: cursor. Evidence: bundle. Boundary: not decided here (Cursor Blame is in scope under its own catalog area, likely the Editor section; only the Explorer hand-off is listed here).
  - What it is: "Open File Blame" (NLS 5375) under category "Cursor Blame" (NLS 5376), command `cursorBlame.openFileBlame`, menu group `12_cursor_blame` order 1. Shows for a file that is not a folder and not a root. Also in the Command Palette. Sibling ids in the bundle: `cursorBlame.openCommit`, `cursorBlame.viewCommit`.
  - Proof strings: `var VAa="cursorBlame.openFileBlame"`, `ExplorerContext,{group:"12_cursor_blame",order:1,command:{id:VAa,title:k(5384,null)}`; NLS 5384 "Cursor Blame".

## Status of each stock group in Cursor

Method: each group's view ids, command ids and settings were searched in the bundle (and the extension manifests for git and references-view). Counts of hits are in the proof lines.

- SB-1 Side bar frame and shared view behavior: tag both, but Cursor's frame is rebuilt (see "Added to SB-1": horizontal tab row, minimum width 214, two-row title for Run and Debug). The stock frame ids are still in the bundle: `workbench.action.toggleSidebarVisibility` (3 hits), `workbench.action.focusSideBar`, `workbench.action.moveSideBarRight`, `workbench.action.resetViewLocations`, `workbench.action.moveView`, `workbench.view.alwaysShowHeaderActions` (3), `workbench.sideBar.location` (13), `workbench.activityBar.location` (9). Cursor registers its chat containers in the Auxiliary Bar location (registration call passes location `2`, container id prefix `workbench.panel.composerChatViewPane`), so they are not side bar content. Cursor also has an "agent IDE unification" layout with a unified sidebar (see left out).
- SB-2 Explorer: tag both, plus the three Cursor right-click items above. The container, the Folders view, the empty view and the Open Editors view (still `hideByDefault: true`, focus chord Ctrl+K E) match V. Ids found: `explorer.newFile`, `explorer.newFolder`, `renameFile`, `revealInExplorer`, `compareFiles`, `selectForCompare`, `copyRelativeFilePath`, `explorer.openToSide`, `explorer.openWith`, `workbench.files.action.refreshFilesExplorer`, `workbench.files.action.collapseExplorerFolders`, `workbench.files.action.focusOpenEditorsView`, `filesExplorer.findInFolder`. Settings found: `explorer.fileNesting.enabled`, `explorer.compactFolders`, `explorer.excludeGitIgnore`.
- SB-3 Search: tag both for the view and its core actions (`workbench.action.findInFiles`, `workbench.action.replaceInFiles`, `search.action.openInEditor`, `search.action.viewAsTree`, `search.action.viewAsList`, `search.action.collapseSearchResults`, `toggleSearchCaseSensitive`, `workbench.action.search.toggleQueryDetails`, `search.action.clearHistory`, all found). The Search Editor ids `search.action.openNewEditor`, `openNewEditorToSide`, `openNewEditorFromView`, `openEditor`, `focusQueryEditorWidget` are found. The "Search with AI" and "Show AI results" actions (`search.action.searchWithAI`, `search.action.showAIResults`) have 0 hits, and so do the settings `search.searchView.semanticSearchBehavior` and `search.searchView.keywordSuggestions`. Treat the AI results part of V's Search as `vscode` only.
- SB-4 Source Control: tag both for the containers, the Repositories, Changes and Graph views, and most git commands. Differences are in the next section. Cursor adds Agent Review, Generate Commit Message and Cursor worktrees.
- SB-5 Run and Debug: tag both. All five view ids and the four focus commands are in the bundle (`workbench.debug.variablesView`, `watchExpressionsView`, `callStackView`, `breakPointsView`, `loadedScriptsView`; `workbench.debug.action.focusVariablesView` and the three siblings, 3 hits each), plus `workbench.action.debug.start`, `workbench.action.debug.configure`, `workbench.action.debug.selectandstart`, `debug.openDebug`, `debug.toolBarLocation`. Cursor's Run and Debug container sets `wantsActionToolbar: true`, an option that does not exist in V (0 hits in V source); it makes the horizontal side bar title two rows tall (see "Added to SB-1"). The Breakpoints list-or-tree toggle is missing (see next section). The JavaScript debugger extension adds three more views to this container in Cursor (see built-in extension contributions).
- SB-6 Testing: tag both. `workbench.view.testing` (33 hits), `workbench.view.testCoverage` (6), `testing.runAll`, `testing.debugAll`, `testing.coverageAll`, `testing.viewAsTree`, `testing.collapseAll`, `testing.toggleContinuousRunForTest`, and the container id `workbench.view.extension.test` with title "Testing" (NLS 11122) and view title "Test Explorer" (NLS 11125) are present. The container registers with `hideIfEmpty: true`, same as V.
- SB-7 Outline, Timeline and References: tag both. `outline.focus`, `timeline.focus`, `files.openTimeline`, `timeline.toggleFollowActiveEditor`, `timeline.pageSize`, `timeline.pageOnScroll`, `outline.collapseItems`, `outline.problems.badges`, `outline.showFiles` all found. The Local History timeline source is there too (`timeline.localHistory` 2 hits, `workbench.action.localHistory.` 19 hits). The Outline view registers at order 2 and collapsed, as in V. `references-view` in `C\extensions\references-view` has the same 21 commands and the same settings as V.

## Where Cursor is behind V (tagged vscode only)

Found by listing every `explorer`, `files`, `search`, `scm`, `debug`, `testing`, `outline` and `timeline` setting key (175 keys) and every literal command id (126 ids) in the V source folders of this section, then searching each in the bundle. Not covered: ids built by code at run time, and extension-contributed ids.

- Settings in V and not in the bundle (9): `scm.repositories.selectionMode`, `scm.repositories.explorer`, `scm.graph.showIncomingChanges`, `scm.graph.showOutgoingChanges`, `debug.breakpointsView.presentation`, `debug.console.maximumLines` (Panel, not this section), `search.searchView.semanticSearchBehavior`, `search.searchView.keywordSuggestions`, `search.experimental.useIgnoreFilesInFindFiles`.
- Command ids in V and not in the bundle (13): `workbench.debug.viewlet.action.toggleBreakpointsPresentation`, `workbench.debug.action.addVariableToChat`, `workbench.debug.action.addScopeToChat`, `workbench.debug.action.addWatchExpressionToChat`, `workbench.scm.action.graph.addHistoryItemToChat`, `workbench.scm.action.graph.addHistoryItemChangeToChat`, `workbench.scm.action.graph.summarizeHistoryItem`, `workbench.scm.action.graph.setListViewMode`, `workbench.scm.action.graph.setTreeViewMode`, `workbench.scm.action.graph.openFile`, `scm.clearValidation`, `scm.editor.triggerSetup`, `workbench.files.openFile`.
- Same finding by direct search: `search.action.searchWithAI` and `search.action.showAIResults` have 0 hits.
- What this means for the stock groups: in V the Repositories view can show branches, tags, stashes and worktrees as a tree (`scm.repositories.explorer`), the Graph can hide incoming and outgoing changes and switch between list and tree, the Breakpoints view can switch between list and tree, and the Search view can offer AI results. None of those four were found in Cursor. They stay in the V file as `vscode` features. Cursor's `product.json` says `vscodeVersion` 1.128.0, so these look like features the Cursor build does not include, not features added later. Why they are missing was not investigated.

## Built-in extension contributions that belong in this section (Cursor build)

- git (`C\extensions\git\package.json`, version string 1.0.0, one minified line). 154 commands against 183 in V; 102 settings against 104; 19 welcome texts (same as V); `scm/title` 16 items and `scm/resourceState/context` 46 items (same as V); `scm/historyItem/context` 6 items against 9. Differences:
  - Only in Cursor: commands `git.rescan` ("Rescan Repositories", command palette), `git.deleteWorktreeFromPalette` ("Delete Worktree...", command palette), `git.resolveAllConflictsInChat` ("Resolve in Chat", an `editor/content` button on files with merge conflicts, so Editor section); settings `git.showCursorWorktrees` and `git.blame.editorDecoration.forceDisabled` (the second is the Editor section).
  - Only in V (32 commands): `git.repositories.*` (26 commands behind the Repositories view artifact tree: checkout, create branch, create tag, merge, rebase, stash apply and pop, worktree open and delete, copy names), `git.graph.compareRef`, `git.graph.compareWithRemote`, `git.graph.compareWithMergeBase`, `git.unstageChange`, `git.delete`, `git.deleteWorktree2`. Menu keys only in V: `scm/repositories/title`, `scm/repository`, `scm/artifactGroup/context`, `scm/artifact/context`, `git.repositories.stash`. Cursor puts "Reopen Closed Repositories..." under `scm/sourceControl/title`; V moved it to `scm/repositories/title`. Settings only in V: `git.confirmCommittedDelete`, `git.worktreeIncludeFiles`, `git.addAICoAuthor`, `git.blame.editorDecoration.disableHover`.
  - So Cursor's git extension looks older than V's. The version string 1.0.0 against V's 10.0.0 is a packaging difference, not a feature version.
- cursor-always-local (`C\extensions\cursor-always-local\package.json`): the one Cursor extension that puts UI in the side bar, the `scm/inputBox` button for Generate Commit Message. Its other contributions are JSON schemas for `.cursor/environment.json` and `.cursor/permissions.json`, not UI.
- cursor-explorer, cursor-checkout, cursor-commits: no side bar UI. `cursor-explorer` registers `cursor.registerExplorerProvider`, which scans the workspace for git repositories and returns path, display path, kind and remotes (code only, no view). `cursor-checkout` registers `cursor.registerCheckoutProvider` (snapshot the working tree and push a branch for a cloud agent; code only, no view). `cursor-commits` tracks requests and commits (uses `scm.gitCommitWasRun`, `scm.registerGitIgnoreProvider`, an output channel; command `cursor.action.startTrackingRequest`); its manifest has empty command, keybinding and menu lists.
- references-view: identical to V (21 commands, 1 container, 1 view, 1 setting).
- search-result, merge-conflict, simple-browser, markdown-language-features: no side bar UI. merge-conflict has one extra Cursor command `merge-conflict.resolve-in-chat` (Editor section, chat).
- cursor-retrieval: commands `cursor.grepClient.debug`, `cursor.codebaseTelemetry.triggerSnapshot`, `cursor.codebaseTelemetry.reportComposerRequest` (command palette only, no side bar view). Checked because the brief asked about codebase or index status in the sidebar. Nothing is shown in the side bar.

## Left out and why

- Agent chat containers and agent lists: chat view containers are registered in the Auxiliary Bar location; `workbench.view.backgroundAgent` and `workbench.view.backgroundAgent.content`; the Agents sidebar CSS (`.agent-sidebar`, `.agent-sidebar-cell`); and the unified sidebar of the agent IDE unification layout (context keys `cursor.agentIdeUnification.sidebarLocation` default `right`, `cursor.agentIdeUnification.unifiedSidebarVisible`, storage keys `workbench.unifiedSidebar.hidden` and `workbench.unifiedSidebar.size`). All agent or chat surfaces, out of scope. The Maximize Chat item on view titles is chat as well.
- Explorer view add-ons that touch only the agent: the cursor-explorer and cursor-checkout providers above (no UI).
- `.cursorignore` handling (56 string hits, `.cursorindexingignore`, `.cursorrules`): it is a file watcher and a settings page option. No Explorer decoration or badge was found for ignored files, so nothing is cataloged.
- Indexing and docs: 79 hits for "Indexing" are in the Cursor Settings page and in an indexing provider. No side bar view. No string for "Codebase Index", "Index Status", "Indexed Files", "Resync Index", "Docs view" or "Manage Docs" (0 hits each).
- JavaScript debugger views (`C\extensions\ms-vscode.js-debug\package.json`, container `debug`, views `jsBrowserBreakpoints`, `jsExcludedCallers`, `jsDebugNetworkTree`): added into the Run and Debug container in Cursor. That extension is not in the V source tree and not in the brief's built-in list, so it is left out. The parent can fold it into SB-5.
- NPM Scripts pane (`C\extensions\npm\package.json`, same as V, hidden by default): left out as in the V file.
- Remote Explorer (`workbench.view.remote`, 1 string hit "Remote Explorer"): present in Cursor, left out as in the V file. Voice Transcripts and Voice Event Stream have 0 hits in the bundle. Edit Sessions has 0 hits. Settings Sync strings have 4 hits.
- Extensions view: out of scope by the brief. It is one of the four default pinned tabs in the horizontal side bar, so the tab exists, but its contents are not cataloged.
- The Agent (background agent) container `workbench.view.backgroundAgent` is always treated as pinned by the horizontal side bar code (`Wke(e,t)` returns true for it). Agent surface, out of scope.
- Update cards: the same Solid module that builds the side bar also draws floating cards "Update to v...", "New update available" and "New in {version}" (CSS `.update-notification-outer-container`, `position: fixed`, `-left` and `-right` variants). They are not side bar content. They belong to the notifications section or the title-bar Update control. Listed so the owner of that section can find them.

## Newer than 1.128

- Nothing from N applies to Cursor here. The newer items found in N (Timeline "Pin Workspace Timeline" and "Compare Selected", the git setting `git.worktreeSymlinkFolders`, and a CSS class on the side bar sash) are in the V file under newer than 1.128. None were found in Cursor: searched the bundle for `timeline.showWorkspaceTimeline` and `git.worktreeSymlinkFolders`, 0 hits.

## Could not verify

- Whether Agent Review shows by default: the feature gate (`workbench.contrib.editorBugbotViewGatedRegistration`) depends on a function call whose value was not read. Nothing was run in the app.
- Where exactly the Generate Commit Message button draws in the message box (inline or in an overflow menu): the manifest only gives the menu id `scm/inputBox`.
- What the Cursor worktrees setting does at run time: the logic is in the extension code and was not read.
- Why Cursor lacks some V 1.128 features (Repositories artifacts, Graph incoming and outgoing, Breakpoints tree, AI search): only the absence was established.
- The condition under which `cursor-commits` calls `cursor.runEditorBugbot` after a commit (a boolean named `a` in minified code): not read.
- Cursor Blame's boundary status: not decided.
- Where the stock Accounts and Manage (gear) buttons live when the activity bar is hidden in `horizontal` mode: not checked. That is the title bar or activity bar sections' question.
- Where the stock Accounts and Manage (gear) buttons live when the activity bar is hidden in `horizontal` mode: not checked. That is the title bar or activity bar sections' question.
- Behavior of the vertical side bar class `JDt` against stock `SidebarPart`: only its constructor was read (minimum width 214, title enabled). Differences beyond that were not checked.
- A third class in the bundle (`ore`) looks like another stock-style side bar with `onDidChangeActivityBarLocation`; which of the three the app creates for each setting value was read only for `horizontal` and `vertical` in the part chooser. Not exercised.
- Whether a user who sets `workbench.activityBar.orientation` to `vertical` still sees the Cursor-only items (Agent Review, Generate Commit Message, the Explorer items): they live in the views and menus, not the frame, so they should, but it was not run.
