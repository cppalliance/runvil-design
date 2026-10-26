Group: SB-7 Outline, Timeline and References. Section: Side bar. Date: 2026-10-25. Pass: 2.
Features: 28.

Short names used in Source: V is the stock 1.128.0 tree, N is the 1.144.0 clone, C bundle is `workbench.desktop.main.js` in the Cursor install, C extensions is the Cursor `extensions` folder, and NLS is `out\nls.messages.json` next to the bundle. Line numbers are from the files as read and can drift by a line or two. Outline and Timeline are panes inside the Explorer container. References has its own container. Nothing was run on screen, so every Cursor finding comes from strings and manifests.

### SB-7.1 Outline view of the open file's symbols
- Tag: both
- Evidence: source
- Boundary: no
- What: The Outline pane under the Explorer tree lists the classes, functions and other symbols of the active editor as a tree. Click a row, or press Enter, to jump to it. The pane says when the editor cannot provide symbols or none were found, and it has Collapse All and Expand All buttons in its title.
- Commands: outline.focus; outline.collapse; outline.expand
- Menus: ViewTitle
- Keys: none found
- Settings: outline.collapseItems
- Cursor: present (the view registration with id `outline`, order 2, collapsed, `outline.focus`, `outline.collapse` and `outline.expand` are all in the bundle, and the NLS strings "Outline", "The active editor cannot provide outline information." and "No symbols found in document" are found)
- Source: V\src\vs\workbench\contrib\outline\browser\outline.contribution.ts (24-38); V\src\vs\workbench\contrib\outline\browser\outlinePane.ts (133-393); V\src\vs\workbench\contrib\outline\browser\outlineActions.ts (18-60); V\src\vs\workbench\contrib\codeEditor\browser\outline\documentSymbolsOutline.ts; C bundle (Outline view registration)

### SB-7.2 Follow Cursor in Outline
- Tag: both
- Evidence: source
- Boundary: no
- What: A toggle in the Outline "..." menu. When on, the selected Outline row moves to match the caret in the editor. The choice is remembered.
- Commands: outline.followCursor
- Menus: ViewTitle
- Keys: none found
- Settings: none found
- Cursor: present (`outline.followCursor` and the NLS string "Follow Cursor" found)
- Source: V\src\vs\workbench\contrib\outline\browser\outlineActions.ts (58-78); V\src\vs\workbench\contrib\outline\browser\outlinePane.ts (324-346); V\src\vs\workbench\contrib\outline\browser\outlineViewState.ts; C bundle (outline.followCursor action)

### SB-7.3 Filter on Type in Outline
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing in the Outline opens a find box. With Filter on Type on, rows that do not match are hidden and the tree expands while you filter. With it off, matches are only highlighted.
- Commands: outline.filterOnType
- Menus: ViewTitle
- Keys: none found
- Settings: none found
- Cursor: present (`outline.filterOnType` and the NLS string "Filter on Type" found)
- Source: V\src\vs\workbench\contrib\outline\browser\outlineActions.ts (79-100); V\src\vs\workbench\contrib\outline\browser\outlinePane.ts (266-272, 306-307, 363-376); C bundle (outline.filterOnType action)

### SB-7.4 Sort the Outline
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose Sort By Position, Name or Category from the Outline "..." menu. The choice is remembered.
- Commands: outline.sortByPosition; outline.sortByName; outline.sortByKind
- Menus: ViewTitle
- Keys: none found
- Settings: none found
- Cursor: present (the three ids and the strings "Sort By: Position", "Sort By: Name" and "Sort By: Category" found)
- Source: V\src\vs\workbench\contrib\outline\browser\outlineActions.ts (101-161); V\src\vs\workbench\contrib\outline\browser\outlinePane.ts (253, 357-360); C bundle (outline.sortBy actions)

### SB-7.5 Choose what the Outline shows
- Tag: both
- Evidence: source
- Boundary: no
- What: Settings decide which symbol kinds appear (26 kinds from files and modules to type parameters), whether rows have icons, whether errors and warnings show as colors and badges, and whether items start collapsed. Some can differ per language.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: outline.icons; outline.collapseItems; outline.problems.enabled; outline.problems.colors; outline.problems.badges; outline.showFiles; outline.showModules; outline.showNamespaces; outline.showPackages; outline.showClasses; outline.showMethods; outline.showProperties; outline.showFields; outline.showConstructors; outline.showEnums; outline.showInterfaces; outline.showFunctions; outline.showVariables; outline.showConstants; outline.showStrings; outline.showNumbers; outline.showBooleans; outline.showArrays; outline.showObjects; outline.showKeys; outline.showNull; outline.showEnumMembers; outline.showStructs; outline.showEvents; outline.showOperators; outline.showTypeParameters
- Cursor: present (searched for `outline.icons`, `outline.collapseItems`, the three `outline.problems.*` ids, and five of the kind settings: `outline.showFiles`, `outline.showClasses`, `outline.showVariables`, `outline.showModules`, `outline.showFunctions`, all found; the other kinds were not searched one by one)
- Source: V\src\vs\workbench\contrib\outline\browser\outline.contribution.ts (42-239); V\src\vs\workbench\services\outline\browser\outline.ts (99-103); V\src\vs\workbench\contrib\codeEditor\browser\outline\documentSymbolsOutline.ts (163, 331-384); C bundle (outline configuration block with `outline.icons` and `outline.collapseItems`)

### SB-7.6 Outline of a notebook
- Tag: both
- Evidence: source
- Boundary: no
- What: When a notebook is active, the Outline lists markdown headers and code cells instead of text symbols. A Filter Entries button picks Markdown Headers Only, Code Cells and Code Cell Symbols, and each row has a menu to run the cell, run the cells in a section, or fold and expand a section.
- Commands: notebook.outline.toggleShowMarkdownHeadersOnly; notebook.outline.toggleCodeCells; notebook.outline.toggleCodeCellSymbols; notebook.section.runSingleCell; notebook.section.runCells; notebook.section.foldSection; notebook.section.expandSection
- Menus: ViewTitle; NotebookOutlineFilter; NotebookOutlineActionMenu
- Keys: none found
- Settings: notebook.outline.showMarkdownHeadersOnly; notebook.outline.showCodeCells; notebook.outline.showCodeCellSymbols
- Cursor: present (all seven command ids and the three settings found in the bundle, and the strings "Filter Entries" and "Markdown Headers Only" are in NLS)
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\outline\notebookOutline.ts (861-975); V\src\vs\workbench\contrib\notebook\browser\controller\sectionActions.ts (33-200); V\src\vs\platform\actions\common\actions.ts (230-231); C bundle (notebook.outline toggles and section actions)

### SB-7.7 Timeline view for the open file
- Tag: both
- Evidence: source
- Boundary: no
- What: The Timeline pane under Outline lists earlier versions of the open file, newest first, from every source that has some, with the file name beside the title. A Refresh button is in its title. It loads the next page when you scroll to the end or click Load more, and it shows a message when the editor cannot provide a timeline or every source is filtered out. The pane exists only when some timeline source is registered.
- Commands: timeline.focus; timeline.refresh
- Menus: TimelineTitle
- Keys: none found
- Settings: timeline.pageSize; timeline.pageOnScroll
- Cursor: present (view registration with `timeline.focus`, `timeline.refresh`, `timeline.pageSize`, `timeline.pageOnScroll` and the strings "Load more" and "The active editor cannot provide timeline information." found)
- Source: V\src\vs\workbench\contrib\timeline\browser\timeline.contribution.ts (28-65); V\src\vs\workbench\contrib\timeline\browser\timelinePane.ts (194-232, 241-300, 440-446, 819-860, 925-997, 1280-1297); C bundle (Timeline view registration)

### SB-7.8 Pin or unpin the current timeline
- Tag: both
- Evidence: source
- Boundary: no
- What: A pin button in the Timeline title locks the view to the file it shows, so it stops changing when you switch tabs, and the header reads "<file> (pinned)". The same button unpins it.
- Commands: timeline.toggleFollowActiveEditor
- Menus: TimelineTitle
- Keys: none found
- Settings: none found
- Cursor: present (`timeline.toggleFollowActiveEditor` and the strings "Pin the Current Timeline" and "Unpin the Current Timeline" found)
- Source: V\src\vs\workbench\contrib\timeline\browser\timelinePane.ts (299-310, 440-446, 1299-1325); C bundle (toggleFollowActiveEditor command)

### SB-7.9 Filter timeline sources
- Tag: both
- Evidence: source
- Boundary: no
- What: A Filter Timeline button in the title opens a list of sources, one per timeline provider such as Git History and Local History. Tick or untick each one to show or hide its entries. The choice is kept in the profile.
- Commands: timeline.toggleExcludeSource:<sourceId>
- Menus: TimelineTitle; TimelineFilterSubMenu
- Keys: none found
- Settings: none found
- Cursor: present (`timeline.toggleExcludeSource:`, `TimelineFilterSubMenu`, the storage key `timeline.excludeSources` and the string "Filter Timeline" found)
- Source: V\src\vs\workbench\contrib\timeline\browser\timeline.contribution.ts (96-104); V\src\vs\workbench\contrib\timeline\browser\timelinePane.ts (1349-1376); C bundle (timeline source filter registration)

### SB-7.10 Open the timeline for a file from the Explorer
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a file in the Explorer and choose Open Timeline to show that file's history in the Timeline pane, even if it is not the active editor.
- Commands: files.openTimeline
- Menus: ExplorerContext
- Keys: none found
- Settings: none found
- Cursor: present (`files.openTimeline` found in the bundle)
- Source: V\src\vs\workbench\contrib\timeline\browser\timeline.contribution.ts (67-94); C bundle (files.openTimeline command)

### SB-7.11 Pin the workspace timeline
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. A Pin Workspace Timeline button in the Timeline title switches the pane from one file to a timeline for the whole workspace, with a "Workspace" header and a "Loading workspace timeline..." message.
- Commands: timeline.showWorkspaceTimeline
- Menus: TimelineTitle
- Keys: none found
- Settings: none found
- Cursor: missing (searched the bundle and NLS for `timeline.showWorkspaceTimeline` and "Pin Workspace Timeline", 0 hits)
- Source: N\src\vs\workbench\contrib\timeline\browser\timelinePane.ts (the `timeline.showWorkspaceTimeline` action and the TimelineForWorkspaceContext key)

### SB-7.12 Local History entries in the Timeline
- Tag: both
- Evidence: source
- Boundary: no
- What: Every save of a file adds a Local History entry to the Timeline, labelled with what caused the save and timestamped, with a hover card. Saves within a short window merge into one entry. Settings cap the file size and the number of entries per file, exclude paths, or turn the source off. Clicking an entry opens a comparison with the current file.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.localHistory.enabled; workbench.localHistory.maxFileSize; workbench.localHistory.maxFileEntries; workbench.localHistory.exclude; workbench.localHistory.mergeWindow
- Cursor: present (searched for the source id `timeline.localHistory` and the five settings, all found; the string "Local History" has 6 hits in NLS)
- Source: V\src\vs\workbench\contrib\localHistory\browser\localHistoryTimeline.ts (30-165); V\src\vs\workbench\browser\workbench.contribution.ts (461-497); C bundle (Local History timeline source)

### SB-7.13 Compare a Local History entry
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click an entry to compare it with the current file, compare it with the entry before it, or mark it with Select for Compare and later pick Compare with Selected on another entry.
- Commands: workbench.action.localHistory.compareWithFile; workbench.action.localHistory.compareWithPrevious; workbench.action.localHistory.selectForCompare; workbench.action.localHistory.compareWithSelected
- Menus: TimelineItemContext
- Keys: none found
- Settings: none found
- Cursor: present (all four ids found in the bundle)
- Source: V\src\vs\workbench\contrib\localHistory\browser\localHistoryCommands.ts (49-180); C bundle (localHistory compare commands)

### SB-7.14 Show or restore a Local History entry
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click an entry and choose Show Contents to open that old copy, or Restore Contents to put it back after a warning that unsaved changes are lost. An old copy open in an editor has a Restore Contents button in its title. Find Entry to Restore opens a picker over every file's history, from the Command Palette or the Timeline "..." menu.
- Commands: workbench.action.localHistory.open; workbench.action.localHistory.restore; workbench.action.localHistory.restoreViaEditor; workbench.action.localHistory.restoreViaPicker; workbench.action.localHistory.revealInOS
- Menus: TimelineItemContext; EditorTitle; TimelineTitle; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (all four ids found, and the string "Find Entry to Restore" is in NLS)
- Source: V\src\vs\workbench\contrib\localHistory\browser\localHistoryCommands.ts (184-440); C bundle (localHistory open, restore and picker commands)

### SB-7.15 Rename, delete or create Local History entries
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click an entry to rename it or delete it (after a warning that it cannot be undone). Delete All clears the history of every file, and Create Entry saves a named snapshot of the active file by hand.
- Commands: workbench.action.localHistory.rename; workbench.action.localHistory.delete; workbench.action.localHistory.deleteAll; workbench.action.localHistory.create
- Menus: TimelineItemContext; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (all four ids found in the bundle)
- Source: V\src\vs\workbench\contrib\localHistory\browser\localHistoryCommands.ts (440-600); C bundle (localHistory rename, delete and create commands)

### SB-7.16 Git History entries in the Timeline
- Tag: both
- Evidence: source
- Boundary: no
- What: The Git History source lists the commits that changed the open file with message, author and relative time. A hover card shows the full commit, and a Staged Changes row sits at the top, plus an Uncommitted Changes row if turned on. Settings pick the authored or committed date, hide the author, and show uncommitted changes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.timeline.date; git.timeline.showAuthor; git.timeline.showUncommitted
- Cursor: present (the three settings and the six `git.timeline.*` commands are in the Cursor git manifest, and `git-history` and "Git History" are in the Cursor git extension code)
- Source: V\extensions\git\src\timelineProvider.ts (18-280); V\extensions\git\package.json (settings `git.timeline.*`); C extensions (git\package.json, git\dist\main.js)

### SB-7.17 Open a Git history entry's changes or commit
- Tag: both
- Evidence: source
- Boundary: no
- What: Click a Git History row to open its changes in a diff, or use the inline button or right-click View Commit to open the whole commit in a multi-file diff.
- Commands: git.timeline.openDiff; git.timeline.viewCommit
- Menus: timeline/item/context
- Keys: none found
- Settings: none found
- Cursor: present (both ids and their menu entries are in the Cursor git manifest)
- Source: V\extensions\git\package.json (timeline/item/context); V\extensions\git\src\commands.ts (4779-4866); C extensions (git\package.json)

### SB-7.18 Compare two Git history entries
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a Git History row, choose Select for Compare, then right-click another row and choose Compare with Selected.
- Commands: git.timeline.selectForCompare; git.timeline.compareWithSelected
- Menus: timeline/item/context
- Keys: none found
- Settings: none found
- Cursor: present (both ids and their menu entries are in the Cursor git manifest)
- Source: V\extensions\git\package.json (timeline/item/context); V\extensions\git\src\commands.ts (4887-4929); C extensions (git\package.json)

### SB-7.19 Copy a commit hash or message from the Timeline
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a Git History row and choose Copy Commit Hash or Copy Commit Message.
- Commands: git.timeline.copyCommitId; git.timeline.copyCommitMessage
- Menus: timeline/item/context
- Keys: none found
- Settings: none found
- Cursor: present (both ids and their menu entries are in the Cursor git manifest)
- Source: V\extensions\git\package.json (timeline/item/context); V\extensions\git\src\commands.ts (4867-4886); C extensions (git\package.json)

### SB-7.20 Find All References in the References view
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Find All References on a symbol to list every use in the Reference Search Results view, which has its own References icon and stays open while you edit. A setting makes CodeLens reference counts open this view instead of a peek.
- Commands: references-view.findReferences
- Menus: editor/context; commandPalette
- Keys: Shift+Alt+F12 (when editorHasReferenceProvider)
- Settings: references.preferredLocation
- Cursor: present (the Cursor references-view manifest has the same command, key, menu entry and setting, and `references-view` has 5 hits in the bundle)
- Source: V\extensions\references-view\package.json (32-66, 67-177, 178-373, 374); V\extensions\references-view\src\references\index.ts (30-70); C extensions (references-view\package.json)

### SB-7.21 Find All Implementations in the References view
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Find All Implementations on an interface or abstract member to list the implementing places in the same References view.
- Commands: references-view.findImplementations
- Menus: editor/context
- Keys: none found
- Settings: none found
- Cursor: present (command and editor/context entry are in the Cursor references-view manifest)
- Source: V\extensions\references-view\package.json (commands, menus editor/context); V\extensions\references-view\src\references\index.ts; C extensions (references-view\package.json)

### SB-7.22 Browse the reference results
- Tag: both
- Evidence: source
- Boundary: no
- What: Results are grouped by file with each matching line shown. A line above the tree reads "N results in M files". Clicking a row opens the file at that match, the matches are highlighted in the editor and its overview ruler while the view is open, and rows can be dragged out as links.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the view `references-view.tree` with its `when` clause `reference-list.isActive` is in the Cursor references-view manifest, same as stock)
- Source: V\extensions\references-view\src\tree.ts (60-130, 189-222); V\extensions\references-view\src\references\model.ts (105-126, 295-305); V\extensions\references-view\src\highlights.ts (11-15); V\extensions\references-view\src\navigation.ts (47-52); C extensions (references-view\package.json)

### SB-7.23 Step to the next or previous reference
- Tag: both
- Evidence: source
- Boundary: no
- What: Press F4 or Shift+F4 to jump to the next or previous result, opening it in the editor and selecting it in the view.
- Commands: references-view.next; references-view.prev
- Menus: commandPalette
- Keys: F4 (when reference-list.hasResult); Shift+F4 (when reference-list.hasResult)
- Settings: none found
- Cursor: present (both ids and both keys are in the Cursor references-view manifest)
- Source: V\extensions\references-view\package.json (374); V\extensions\references-view\src\navigation.ts (19-85); C extensions (references-view\package.json)

### SB-7.24 Dismiss or copy results
- Tag: both
- Evidence: source
- Boundary: no
- What: Each result row has a Dismiss button that removes it from the list. Right-click offers Copy, Copy All and, on a file row, Copy Path.
- Commands: references-view.removeReferenceItem; references-view.copy; references-view.copyAll; references-view.copyPath
- Menus: view/item/context; commandPalette; ViewItemContext
- Keys: none found
- Settings: none found
- Cursor: present (all four ids and their menu entries are in the Cursor references-view manifest)
- Source: V\extensions\references-view\package.json (menus view/item/context); C extensions (references-view\package.json)

### SB-7.25 Refresh or clear the results
- Tag: both
- Evidence: source
- Boundary: no
- What: The view title has a Refresh button that reruns the search and a Clear button that empties the results.
- Commands: references-view.refresh; references-view.clear
- Menus: view/title; commandPalette
- Keys: none found
- Settings: none found
- Cursor: present (both ids and their view/title entries are in the Cursor references-view manifest)
- Source: V\extensions\references-view\package.json (menus view/title); C extensions (references-view\package.json)

### SB-7.26 Search history in the References view
- Tag: both
- Evidence: source
- Boundary: no
- What: When the view has no results it lists earlier searches with a Rerun button on each. Show History picks one from a list, and Clear History forgets them all.
- Commands: references-view.pickFromHistory; references-view.clearHistory; references-view.refind
- Menus: view/title; view/item/context; commandPalette
- Keys: none found
- Settings: none found
- Cursor: present (all three ids and their menu entries are in the Cursor references-view manifest)
- Source: V\extensions\references-view\package.json (menus view/title, view/item/context, commandPalette); V\extensions\references-view\src\tree.ts (119-129, 224-260); C extensions (references-view\package.json)

### SB-7.27 Call hierarchy in the References view
- Tag: both
- Evidence: source
- Boundary: no
- What: Show Call Hierarchy lists the callers of the symbol as a tree in the References view. A title button switches between Callers Of and Calls From, and a row's right-click menu shows incoming or outgoing calls for that row. Dismiss removes a row.
- Commands: references-view.showCallHierarchy; references-view.showIncomingCalls; references-view.showOutgoingCalls; references-view.removeCallItem
- Menus: editor/context; view/title; view/item/context
- Keys: Shift+Alt+H (when editorHasCallHierarchyProvider)
- Settings: none found
- Cursor: present (all four ids, the key and the menu entries are in the Cursor references-view manifest)
- Source: V\extensions\references-view\package.json (67-177, 178-373, 374); V\extensions\references-view\src\calls\index.ts; V\extensions\references-view\src\calls\model.ts (15-40, 200-215); C extensions (references-view\package.json)

### SB-7.28 Type hierarchy in the References view
- Tag: both
- Evidence: source
- Boundary: no
- What: Show Type Hierarchy lists the supertypes or subtypes of a type in the References view. A title button switches between Supertypes Of and Subtypes Of, a row's right-click menu shows supertypes or subtypes for that row, and Dismiss removes a row.
- Commands: references-view.showTypeHierarchy; references-view.showSupertypes; references-view.showSubtypes; references-view.removeTypeItem
- Menus: editor/context; view/title; view/item/context
- Keys: none found
- Settings: none found
- Cursor: present (all four ids and the menu entries are in the Cursor references-view manifest)
- Source: V\extensions\references-view\package.json (67-177, 178-373); V\extensions\references-view\src\types\index.ts; V\extensions\references-view\src\types\model.ts (15-40, 170-180); C extensions (references-view\package.json)

## Handed off
- Peek Call Hierarchy and Peek Type Hierarchy inside the editor (`editor.showCallHierarchy`, `editor.showIncomingCalls`, `editor.showOutgoingCalls`, `editor.refocusCallHierarchy`, `editor.showTypeHierarchy`, and the Peek submenu entries): ED-4. All of these ids were found in the Cursor bundle.
- Peek References, Go to References and the other editor context menu entries that sit beside Find All References: ED-4.
- Breadcrumbs symbol path and Go to Symbol in Editor, which read the same document symbols as Outline: ED-5 for breadcrumbs, QA-3 for Go to Symbol.
- The multi-file diff editor that View Commit opens and the diff editor that Local History comparisons open: ED-6.
- The Explorer folder and file right-click menu itself, where Open Timeline sits: SB-2 (the Open Timeline feature stays here as SB-7.10).
- Notebook editor features other than the Outline pane (cells, toolbars, run and fold on the cell itself): ED-7.

## Unclear
- Pass 1 called one Timeline toggle "Follow Active Editor". The command id is `timeline.toggleFollowActiveEditor`, but the labels a person sees are "Pin the Current Timeline" and "Unpin the Current Timeline".
- Other Timeline changes exist only in N and are not listed as features: selecting several Timeline rows with a Compare Selected action (`timeline.compareSelected`), renaming a Local History entry inline in the view, Open Timeline working on folders and from a keybinding. None were found in Cursor. Only the workspace timeline (SB-7.11) is listed on its own.
- Shift+Alt+H is bound twice: by `references-view.showCallHierarchy` here and by the editor peek command `editor.showCallHierarchy`. Which one wins in the editor was not tested.
- Whether the References container icon stays visible when the view has no results was not observed. The view's `when` clause is `reference-list.isActive`.
- In Cursor's default horizontal side bar, References sits in the overflow list rather than as a default tab (pass 1 finding). Nothing was run to confirm it.
- SB-7.5 lists all 26 kind settings from V, but only five of them were searched in the Cursor bundle one by one.
- Cursor's own timeline sources: searched C extensions for `registerTimelineProvider` and only the git extension registers one. No Cursor-added source was found.
- ED-7 may also list the notebook Outline filters (SB-7.6), since the notebook editor owns the section actions. They are listed here because the pane is the Outline view.
