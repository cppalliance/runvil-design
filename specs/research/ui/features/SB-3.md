Group: SB-3 Search. Section: Side bar. Date: 2026-10-25. Pass: 2.
Features: 30.

### SB-3.1 Open the Search view and fill the box from the editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Shift+F, choose Find in Files from the Edit menu, or pick Search from the View menu to open the Search view with the cursor in the search box, pre-filled with the editor selection; the command also takes a JSON argument (query, replace, preserveCase, triggerSearch, filesToInclude, filesToExclude, isRegex, isCaseSensitive, matchWholeWord, useExcludeSettingsAndIgnoreFiles, onlyOpenEditors, showIncludesExcludes) so a keybinding can open a prefilled search.
- Commands: workbench.action.findInFiles; workbench.view.search
- Menus: MenubarEditMenu (group 4_find_global, order 1); MenubarViewMenu (group 3_sidebar, order 1, generated from the container's openCommandActionDescriptor)
- Keys: Ctrl+Shift+F (workbench.action.findInFiles); the same chord on workbench.view.search sits behind a never-matching when clause, so it does nothing
- Settings: search.seedWithNearestWord; search.seedOnFocus; search.globalFindClipboard (macOS only)
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:202-258; V\src\vs\workbench\contrib\search\browser\searchActionsBase.ts:88-150; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:54-82; V\src\vs\workbench\contrib\search\browser\searchView.ts:1224-1262; C bundle (SearchCommandIds enum entries FindInFilesActionId and workbench.view.search container id; Edit menu row listed in the title bar entry point inventory)

### SB-3.2 Type a query and let the search run
- Tag: both
- Evidence: source
- Boundary: no
- What: Type in the Search box and results fill in as you type after a short pause, Enter runs the search at once, Esc stops it, and Ctrl+Enter adds a new line so the box can hold a multi-line query.
- Commands: workbench.action.toggleSearchOnType
- Menus: none found
- Keys: Enter (run now); Escape (stop a running search, from the box); Ctrl+Enter (new line in the box, Control+Enter on macOS); PageUp and PageDown (jump to the start or end of the text); Tab (next control)
- Settings: search.searchOnType; search.searchOnTypeDebouncePeriod; search.ripgrep.maxThreads
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchWidget.ts:636-740, 814-826; V\src\vs\workbench\contrib\search\browser\searchView.ts:1589-1601; V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:318-337; V\src\vs\workbench\contrib\search\browser\search.common.contribution.ts:22-34; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:177-181; C bundle (workbench.action.toggleSearchOnType, settings search.searchOnType and search.searchOnTypeDebouncePeriod and search.ripgrep.maxThreads, label "Search: Type Search Term and press Enter to search", label "Toggle Search on Type")

### SB-3.3 Match Case, Whole Word and Regex toggles
- Tag: both
- Evidence: source
- Boundary: no
- What: Three icon toggles at the right edge of the search box make the search case sensitive, whole word only, or a regular expression, and a broken regular expression shows its error under the box instead of running.
- Commands: toggleSearchCaseSensitive; toggleSearchWholeWord; toggleSearchRegex
- Menus: none found
- Keys: Alt+C; Alt+W; Alt+R, all when the Search view is focused (Cmd+Alt+C, Cmd+Alt+W and Cmd+Alt+R on macOS, where the case toggle also needs a file or folder row not to be focused)
- Settings: search.smartCase; search.usePCRE2 (deprecated); search.useRipgrep
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:80-135; V\src\vs\editor\contrib\find\browser\findModel.ts:39-58; V\src\vs\workbench\contrib\search\browser\searchWidget.ts:441-480, 636-650; V\src\vs\workbench\contrib\search\browser\searchFindInput.ts; C bundle (the three toggle ids in the SearchCommandIds enum, labels "Toggle Case Sensitive", "Toggle Whole Word", "Toggle Regex", settings search.smartCase and search.usePCRE2)

### SB-3.4 Search inside notebooks and pick which cell parts
- Tag: both
- Evidence: source
- Boundary: no
- What: While a notebook is open, a filter button joins the toggles in the search box with four choices (Markdown Source, Rendered Markdown, Code Cell Source, Code Cell Output), and matches inside notebook cells open in the notebook editor.
- Commands: none found
- Menus: none found (the filter button opens its own drop-down)
- Keys: none found
- Settings: search.experimental.closedNotebookRichContentResults
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchFindInput.ts:12-84; V\src\vs\workbench\contrib\search\browser\searchWidget.ts:182-252; V\src\vs\workbench\contrib\notebook\browser\contrib\find\notebookFindReplaceWidget.ts:67-72; V\src\vs\workbench\contrib\search\browser\searchView.ts:2223-2300; V\src\vs\workbench\contrib\search\browser\notebookSearch\notebookSearchContributions.ts; C bundle (labels "Notebook Find Filters", "Markdown Source", "Rendered Markdown", "Code Cell Source", "Code Cell Output", setting search.experimental.closedNotebookRichContentResults)

### SB-3.5 Recall earlier searches and clear the history
- Tag: both
- Evidence: source
- Boundary: no
- What: Up and Down in the search, replace, include or exclude box step through earlier entries saved for this workspace, and Clear Search History in the Command Palette wipes all four lists.
- Commands: history.showPrevious; history.showNext; search.action.clearHistory
- Menus: none found (Command Palette only)
- Keys: Up Arrow and Alt+Up Arrow (history.showPrevious); Down Arrow and Alt+Down Arrow (history.showNext); both only while a history box is focused and there is an entry to move to
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\history\browser\contextScopedHistoryWidget.ts:110-140; V\src\vs\workbench\contrib\search\common\searchHistoryService.ts:22-65 (workspace storage key workbench.search.history); V\src\vs\workbench\contrib\search\browser\searchActionsTopBar.ts:25-40; V\src\vs\workbench\contrib\search\browser\searchView.ts:490-500, 545-575; C bundle (history.showPrevious, history.showNext, search.action.clearHistory, storage key workbench.search.history, label "Clear Search History")

### SB-3.6 Replace row
- Tag: both
- Evidence: source
- Boundary: no
- What: The chevron at the left of the search box opens a Replace box under it with a Preserve Case toggle, Ctrl+Shift+H opens the Search view with the Replace box already open, and Esc in the Replace box closes it.
- Commands: workbench.action.replaceInFiles; closeReplaceInFilesWidget; toggleSearchPreserveCase
- Menus: MenubarEditMenu (group 4_find_global, order 2)
- Keys: Ctrl+Shift+H (workbench.action.replaceInFiles); Escape when the Replace box is focused (closeReplaceInFilesWidget); Alt+P for Preserve Case (Cmd+Alt+P on macOS)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:56-77, 137-155, 394-420; V\src\vs\workbench\contrib\search\browser\searchWidget.ts:421-436, 553-612, 774-803; C bundle (the three ids in the SearchCommandIds enum, labels "Toggle Replace", "Toggle Preserve Case", "Replace in Files", "Close Replace Widget")

### SB-3.7 Replace one match, or all matches in a file or folder
- Tag: both
- Evidence: source
- Boundary: no
- What: With a replace term typed and a search run, you replace a single match, every match in one file, or every match in one folder from row buttons, the right-click menu or Ctrl+Shift+1, and the changed files are saved.
- Commands: search.action.replace; search.action.replaceAllInFile; search.action.replaceAllInFolder
- Menus: SearchContext (group search, order 1); SearchActionMenu (group inline, order 1)
- Keys: Ctrl+Shift+1 (all three, when the Replace row is active and the matching row kind is focused); Ctrl+Shift+Enter (Replace All in a file or folder)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsRemoveReplace.ts:150-255; V\src\vs\workbench\contrib\search\browser\replaceService.ts:112-137; C bundle (search.action.replace, search.action.replaceAllInFile, search.action.replaceAllInFolder, labels "Replace" and "Replace All")

### SB-3.8 Replace All with a confirmation
- Tag: both
- Evidence: source
- Boundary: no
- What: The Replace All button at the end of the Replace row (Ctrl+Alt+Enter) asks "Replace N occurrences across M files?" in a standard confirm box before it changes anything, shows progress, then reports "Replaced N occurrences across M files" in the view.
- Commands: search.action.replaceAll
- Menus: none found
- Keys: Ctrl+Alt+Enter, when the Replace row is active and the editor find widget is not showing
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchWidget.ts:75-125, 580-585, 853-870; V\src\vs\workbench\contrib\search\browser\searchView.ts:812-931; C bundle (search.action.replaceAll, labels "Replace All (Submit Search to Enable)", "Replace {0} occurrences across {1} files?", "Replaced {0} occurrences across {1} files.")

### SB-3.9 Preview a replacement as a diff
- Tag: both
- Evidence: source
- Boundary: no
- What: With a replace term typed, clicking a match opens a side-by-side "file ↔ file (Replace Preview)" diff showing what the replacement would do, instead of opening the plain file (notebook matches still open in the notebook editor).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.useReplacePreview
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchView.ts:2223-2237; V\src\vs\workbench\contrib\search\browser\replaceService.ts:140-200; V\src\vs\workbench\contrib\search\browser\replaceContributions.ts; C bundle (openReplacePreview, label "{0} ↔ {1} (Replace Preview)", setting search.useReplacePreview)

### SB-3.10 Filter by files to include and files to exclude
- Tag: both
- Evidence: source
- Boundary: no
- What: The "..." button under the search box (Toggle Search Details) shows two comma-separated glob boxes, files to include and files to exclude, that limit where the search looks, and each box keeps its own history.
- Commands: workbench.action.search.toggleQueryDetails
- Menus: none found
- Keys: Ctrl+Shift+J when the Search view is focused (also works in the Search Editor)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:31-54; V\src\vs\workbench\contrib\search\browser\searchView.ts:500-600, 1533-1563; V\src\vs\workbench\contrib\search\browser\patternInputWidget.ts:33-184; C bundle (workbench.action.search.toggleQueryDetails, labels "Toggle Search Details", "files to include", "files to exclude", placeholders "e.g. *.ts, src/**/include" and "e.g. *.ts, src/**/exclude")

### SB-3.11 Use exclude settings and ignore files
- Tag: both
- Evidence: source
- Boundary: no
- What: A toggle in the exclude box, on by default, makes the search skip files matched by search.exclude, files.exclude and the .gitignore and .ignore files; turning it off finds those files too, and the view then says so with a link to turn it back on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.exclude; files.exclude; search.useIgnoreFiles; search.useGlobalIgnoreFiles; search.useParentIgnoreFiles; search.followSymlinks
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\patternInputWidget.ts:281-331; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:104-171, 198-202; V\src\vs\workbench\contrib\search\browser\searchView.ts:2047-2050, 2072-2078; C bundle (labels "Use Exclude Settings and Ignore Files", "exclude settings and ignore files are disabled", settings search.exclude, search.useIgnoreFiles, search.useGlobalIgnoreFiles, search.useParentIgnoreFiles, search.followSymlinks)

### SB-3.12 Search only in open editors
- Tag: both
- Evidence: source
- Boundary: no
- What: A book-icon toggle in the include box limits the search to files open in editors, and the view then shows "searching only in open files" with a link to search the whole workspace again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\patternInputWidget.ts:185-278; V\src\vs\workbench\contrib\search\browser\searchView.ts:545-558, 2052-2056, 2079-2083; C bundle (label "Search only in Open Editors", label "searching only in open files", members useSearchInEditorsBox and onChangeSearchInEditorsBox)

### SB-3.13 Search only in changed files
- Tag: vscode
- Evidence: source
- Boundary: no
- What: An edit-icon toggle in the include box, enabled only when Source Control has changes, limits the search to the changed files, and it and the open editors toggle switch each other off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\workbench\contrib\search\browser\patternInputWidget.ts:185-278; V\src\vs\workbench\contrib\search\browser\searchView.ts:551-556, 1625-1629; C bundle (searched the label "Search only in Changed Files (Source Control)" in the message file and the members useSearchInChangedFilesBox, onChangeSearchInChangedFilesBox, onlySearchInChangedFiles, setOnlySearchInChangedFilesEnabled in the bundle, no hits; the open editors toggle next to it is present)

### SB-3.14 Find in Folder from the Explorer
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a folder in the Explorer and choose Find in Folder... to open Search with that folder in the include box; Find in Workspace... is the root-level variant shown when the right-clicked root is not a folder and opens Search with the include box cleared.
- Commands: filesExplorer.findInFolder; filesExplorer.findInWorkspace
- Menus: ExplorerContext (group 4_search, order 10, both items)
- Keys: Shift+Alt+F when the Explorer is focused on a folder (filesExplorer.findInFolder)
- Settings: search.mode
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:260-320, 358-413; V\src\vs\workbench\contrib\search\browser\searchView.ts:1565-1587; C bundle (filesExplorer.findInFolder, filesExplorer.findInWorkspace, labels "Find in Folder..." and "Find in Workspace...")

### SB-3.15 Restrict the search to a folder or exclude a folder from the results
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a folder row in the results to put that folder into the include box (Restrict Search to Folder) or the exclude box (Exclude Folder from Search) and rerun the search.
- Commands: search.action.restrictSearchToFolder; search.action.excludeFromSearch
- Menus: SearchContext (group search, order 3 for restrict and order 4 for exclude)
- Keys: Shift+Alt+F when a folder row with a path is focused (search.action.restrictSearchToFolder)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:36-61, 85-105, 358-413; V\src\vs\workbench\contrib\search\browser\searchView.ts:1565-1587; C bundle (both ids in the SearchCommandIds enum, labels "Restrict Search to Folder" and "Exclude Folder from Search")

### SB-3.16 Exclude or include a file type from a result row
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Right-click a file row to add its extension pattern (for example *.test.ts) to the exclude box (Exclude File Type from Search) or to the include box (Include File Type from Search, spelled that way in the source) and rerun the search.
- Commands: search.action.excludeFileTypeFromSearch; search.action.includeFileTypeInSearch
- Menus: SearchContext (group search, order 5 for exclude and order 6 for include)
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:107-148, 331-343, 422-443; V\src\vs\workbench\contrib\search\common\constants.ts:51-52; C bundle (searched both ids and both labels, no hits; the SearchCommandIds enum goes from ViewAsListActionId straight to ToggleQueryDetailsActionId and has no entry for either)

### SB-3.17 Read the results tree
- Tag: both
- Evidence: source
- Boundary: no
- What: Matches appear under file rows that show a file icon, name, folder path and a match count badge, with the matched text highlighted on each line, plus an "Other files" row for matches outside the workspace and one row per root in a multi-root workspace.
- Commands: none found
- Menus: SearchActionMenu (inline row buttons)
- Keys: none found
- Settings: search.showLineNumbers; search.decorations.colors; search.decorations.badges; search.actionsPosition
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchResultsView.ts:100-215, 280-290, 355-365, 490-500; V\src\vs\workbench\contrib\search\browser\searchView.ts:960-1030; V\src\vs\workbench\contrib\search\browser\media\searchview.css; C bundle (labels "Text Results", "Other files", "{0} matches found", "{0} files found", "From line {0}", "{0} more lines", settings search.showLineNumbers and search.decorations.colors and search.decorations.badges and search.actionsPosition; stylesheet classes .search-view .filematch, .foldermatch, .linematch, .actionBarContainer)

### SB-3.18 View as list or tree
- Tag: both
- Evidence: source
- Boundary: no
- What: A header button flips the results between a flat list of files and a folder tree, and search.defaultViewMode sets which one a new search starts in.
- Commands: search.action.viewAsTree; search.action.viewAsList
- Menus: ViewTitle (group navigation, order 2, when the view is Search)
- Keys: none found
- Settings: search.defaultViewMode
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsTopBar.ts:158-204; V\src\vs\workbench\contrib\search\browser\searchView.ts:386-400; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:297-306; C bundle (both ids, labels "View as Tree" and "View as List", setting search.defaultViewMode)

### SB-3.19 Expand and collapse results
- Tag: both
- Evidence: source
- Boundary: no
- What: A header button collapses every file row or, when all are collapsed, expands everything; right-clicking a folder row adds Expand Recursively; search.collapseResults decides what a new search starts with, and a single result file opens by itself.
- Commands: search.action.collapseSearchResults; search.action.expandSearchResults; search.action.expandRecursively
- Menus: ViewTitle (group navigation, order 4); SearchContext (group search, order 4, Expand Recursively on a folder row)
- Keys: none found
- Settings: search.collapseResults
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsTopBar.ts:91-133, 250-273, 294-399; V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:64-83; V\src\vs\workbench\contrib\search\browser\searchView.ts:797-810, 1770-1781; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:226-235; C bundle (the three ids, labels "Collapse All", "Expand All", "Expand Recursively", setting search.collapseResults)

### SB-3.20 Sort the results
- Tag: both
- Evidence: source
- Boundary: no
- What: A setting orders the files in the results by folder and file name, file name only, extension, last modified time, or match count in either direction.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.sortOrder
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\search.contribution.ts:273-286; V\src\vs\workbench\contrib\search\browser\searchCompare.ts:21-60; V\src\vs\workbench\contrib\search\browser\searchView.ts:286-292, 2650, 2755-2770; C bundle (setting search.sortOrder)

### SB-3.21 Open a match in the editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking a match opens its file in the editor with the match selected and highlighted, double-click or Enter keeps the tab open, and Ctrl+Enter opens it beside the current editor.
- Commands: search.action.openResult; search.action.openResultToSide
- Menus: none found (1.128.0 has no right-click item for either)
- Keys: Enter (also Cmd+Down on macOS); Ctrl+Enter (Control+Enter on macOS); both when a file or match row is focused
- Settings: workbench.list.openMode
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:158-213; V\src\vs\workbench\contrib\search\browser\searchView.ts:1020-1030, 2239-2300; V\src\vs\platform\list\browser\listService.ts:171; C bundle (search.action.openResult, search.action.openResultToSide, label "Open Match")

### SB-3.22 Add cursors at every match in a file
- Tag: both
- Evidence: source
- Boundary: no
- What: With a file or match row focused, Ctrl+Shift+L opens that file and puts a cursor on each match the search found in it.
- Commands: addCursorsAtSearchResults
- Menus: none found
- Keys: Ctrl+Shift+L when the Search view is visible and a file or match row is focused
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:215-238; V\src\vs\workbench\contrib\search\browser\searchView.ts:2302-2331; C bundle (addCursorsAtSearchResults, label "Add Cursors at Search Results")

### SB-3.23 Dismiss a result
- Tag: both
- Evidence: source
- Boundary: no
- What: Remove a match, a file or a folder from the results without touching the file, using the X button on the row, the Delete key, or Dismiss in the right-click menu.
- Commands: search.action.remove
- Menus: SearchContext (group search, order 2); SearchActionMenu (group inline, order 2)
- Keys: Delete (Cmd+Backspace on macOS) when the Search view is visible and a result row is focused
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsRemoveReplace.ts:54-148; V\src\vs\workbench\contrib\search\browser\searchActionsBase.ts:34-70; C bundle (search.action.remove, label "Dismiss")

### SB-3.24 Copy results and paths
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a result to Copy the match or file text (Ctrl+C), Copy Path for a file or folder row (Ctrl+Alt+C, Shift+Alt+C on Windows), or Copy All results as text.
- Commands: search.action.copyMatch; search.action.copyPath; search.action.copyAll
- Menus: SearchContext (group search_2, orders 1, 2 and 3)
- Keys: Ctrl+C (search.action.copyMatch, file or match row); Ctrl+Alt+C, Shift+Alt+C on Windows (search.action.copyPath, file or folder row with a path)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsCopy.ts:20-100, 133-150; C bundle (the three ids, labels "Copy", "Copy Path", "Copy All")

### SB-3.25 Reveal a result file in the Explorer
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a file row and choose Reveal in Explorer View to open the Explorer and select that file, which only works for files inside the workspace.
- Commands: search.action.revealInSideBar
- Menus: SearchContext (group search_3, order 1, on a file row when there are results)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:150-200; C bundle (search.action.revealInSideBar, label "Reveal in Explorer View")

### SB-3.26 Move around with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+Down and Ctrl+Up hop between the boxes and the results, Ctrl+Up from the first result returns to the search box, and F4 and Shift+F4 select the next and previous match from anywhere.
- Commands: search.focus.nextInputBox; search.focus.previousInputBox; search.action.focusSearchFromResults; search.action.focusSearchList; search.action.focusNextSearchResult; search.action.focusPreviousSearchResult
- Menus: none found
- Keys: Ctrl+Down (search.focus.nextInputBox, in an input box); Ctrl+Up (search.focus.previousInputBox, in an input box other than the search box); Ctrl+Up (search.action.focusSearchFromResults, when the first match is focused or accessibility mode is on); F4 (search.action.focusNextSearchResult); Shift+F4 (search.action.focusPreviousSearchResult)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts:241-316, 339-392; V\src\vs\workbench\contrib\search\browser\searchView.ts:1128-1230, 1285-1365; C bundle (the six ids in the SearchCommandIds enum, labels "Focus Next Input", "Focus Previous Input", "Focus Search From Results", "Focus List", "Focus Next Search Result", "Focus Previous Search Result")

### SB-3.27 Refresh, stop and clear a search
- Tag: both
- Evidence: source
- Boundary: no
- What: Header buttons rerun the current search (Refresh), swap to a Stop button while a search is slow (Esc also stops it), and Clear Search Results empties the boxes and the results.
- Commands: search.action.refreshSearchResults; search.action.cancel; search.action.clearSearchResults
- Menus: ViewTitle (group navigation: Refresh order 0 when not slow, Cancel order 0 while slow, Clear order 1)
- Keys: Escape (search.action.cancel, when the Search view is visible and a list has focus)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsTopBar.ts:42-152, 276-292; C bundle (the three ids, labels "Refresh", "Cancel Search", "Clear Search Results")

### SB-3.28 Open results in a Search Editor
- Tag: both
- Evidence: source
- Boundary: no
- What: The Open New Search Editor header button opens a Search Editor that carries over the file filters and toggles, and Open Results in Editor (Alt+Enter) or the "Open in editor" link in the result count line copies the current results into one; the Search Editor page itself is ED-3.
- Commands: search.action.openNewEditorFromView; search.action.openInEditor
- Menus: ViewTitle (group navigation, order 2, Open New Search Editor); ViewTitle (group navigation, order 2, when view is the Search view); CommandPalette
- Keys: Alt+Enter (Cmd+Enter on macOS) when there are results and the Search view is focused (search.action.openInEditor); Alt+Enter (Cmd+Enter on macOS) when the Search view is focused and has results
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts:350-374, 598-613; V\src\vs\workbench\contrib\searchEditor\browser\searchEditorActions.ts:78-96; V\src\vs\workbench\contrib\search\browser\searchView.ts:2085-2094; C bundle (search.action.openInEditor, search.action.openNewEditorFromView, labels "Open Results in Editor", "Open New Search Editor", "Open in editor", "Copy current search results to an editor"); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts (OpenSearchEditorAction, OpenInEditorCommandId); V\src\vs\workbench\contrib\searchEditor\browser\searchEditorActions.ts (createEditorFromSearchResult); C bundle (ids search.action.openNewEditorFromView, search.action.openInEditor, labels "Open New Search Editor", "Open Results in Editor")

### SB-3.29 Choose whether Find in Files opens the view or a Search Editor
- Tag: both
- Evidence: source
- Boundary: no
- What: A setting makes Find in Files, Find in Folder and Find in Workspace open in the Search view (the default), in an existing Search Editor, or in a new one.
- Commands: workbench.action.findInFiles; filesExplorer.findInFolder; filesExplorer.findInWorkspace
- Menus: none found
- Keys: none found
- Settings: search.mode
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\search.contribution.ts:130-139; V\src\vs\workbench\contrib\search\browser\searchActionsBase.ts:114-150; V\src\vs\workbench\contrib\search\browser\searchActionsFind.ts:300-320, 358-413; C bundle (setting search.mode with its description in the message file)

### SB-3.30 Result count line, messages and empty states
- Tag: both
- Evidence: source
- Boundary: no
- What: A line under the boxes reports "N results in M files" with the Open in editor link and reminders when excludes or open-editors scope are changing the result; when nothing is found it explains why with links (Search again in all files, Open Settings, Search again after a cancel), it warns when search.maxResults cuts the list short, and a window with no folder says only open files are searched with an Open Folder link.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.maxResults
- Cursor: changed (the no-results text is older: "Review your settings for configured exclusions and check your gitignore files" where stock says "Review your configured exclusions"; the Search with AI link and keyword suggestions are absent)
- Source: V\src\vs\workbench\contrib\search\browser\searchView.ts:1797-1905, 2056-2105, 2183-2208; V\src\vs\workbench\contrib\search\browser\searchMessage.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts:221-225; C bundle (message file strings "{0} results in {1} files", "Search again", "Search again in all files", "Open Settings", "The result set only contains a subset of all matches. Be more specific in your search to narrow down the results.", "You have not opened or specified a folder. Only open files are currently searched - ", "Search was canceled before any results could be found - ", the older no-results sentence)

## Handed off
- The Search Editor page (a results editor tab, ids search.action.openNewEditor, search.action.openNewEditorToSide, search.action.openEditor, searchEditor.action.*, settings search.searchEditor.*, context lines): ED-3.
- Go to Symbol in Workspace (workbench.action.showAllSymbols, the # prefix) and Quick Search (workbench.action.quickTextSearch, the % prefix) plus the Quick Open settings search.quickOpen.*, search.quickAccess.preserveInput, and the Cursor-only search.quickOpen.useVscodeSearch and search.quickAccess.excludeSymbolProviders: QA-2.
- Search accessibility help (Alt+F1 from the search box, searchAccessibilityHelp.ts, priority 105): no group owns the accessible view, the catalog lists it as unowned. Cursor has none of its text (the heading "Accessibility Help: Search Across Files" has 0 hits in the Cursor message file).
- Dragging the Search view to the Panel or Secondary Side Bar, Move View, Reset View Locations, and the deprecated search.location setting: SB-1.
- The Search icon in the activity rail and the Search tab in Cursor's pinned tab row: AB-1 and AB-6.

## Unclear
- Cursor lacks the stock AI search (a pass 1 finding, confirmed). The ids search.action.searchWithAI, search.action.showAIResults and search.action.hideAIResults, the settings search.searchView.semanticSearchBehavior and search.searchView.keywordSuggestions, and the labels "Search with AI" and "Search instead for: " have 0 hits in the Cursor bundle and message file, and the SearchCommandIds enum in the bundle has no AI entries. Stock only turns this on when an extension registers an AI text search provider (context key hasAIResultProviderKey; the copilot extension in V\extensions\copilot calls registerAITextSearchProvider at src\extension\conversation\vscode-node\conversationFeature.ts:216, behind the aiTextSearchProvider proposed API), so I did not list it as a feature.
- Cursor's Search view is older than 1.128.0 in more places than the AI part: no changed-files toggle (SB-3.13), no file type items (SB-3.16), no search accessibility help, and older no-results wording (SB-3.30). Why was not investigated. Nothing was run, so all Cursor findings are strings and ids, not observed behavior.
- Pass 1 said the header button that opens a Search Editor is search.action.openInEditor. In 1.128.0 that id is Open Results in Editor (Alt+Enter and the "Open in editor" link); the header button is search.action.openNewEditorFromView (Open New Search Editor). Both are in SB-3.28. Pass 1 also listed Open to the Side in the result right-click menu, but search.action.openResultToSide has keys only in 1.128.0 (SB-3.21).
- Ctrl+Shift+J is the stock key for Toggle Search Details (SB-3.10) and Cursor binds the same chord to Cursor Settings (aiSettings.action.open, TB-5). Which one wins while the Search view is focused was not determined. Cursor's other default keys for this group were not compared, only the command ids.
- Cursor's descriptions for search.mode and search.actionsPosition say "search view" in lower case where stock says "Search view", another sign of older text.
- Stock disables Find in Files and Replace in Files in a Sessions window (IsSessionsWindowContext in searchActionsFind.ts:250 and searchActionsNav.ts:405); that window is out of scope here and Cursor's Agents window was not read.
- No UI found for these, so they are not features: search.action.getSearchResults (f1 false, returns the results as text), search.action.focusActiveEditor (declared in constants.ts but no registered action found), the deprecated settings search.useRipgrep and search.maintainFileSearchCache (both present in Cursor), and search.experimental.useIgnoreFilesInFindFiles (extension API behavior, missing in Cursor).
- Search results as chat context (searchChatContext.ts, attach results to a chat) was skipped as chat, which is out of scope.
- newer-than-1.128, refinements and not features: N adds the keybinding to the hover on the Toggle Search Details button (searchView.ts) and a narrow layout switch for the search box options (searchWidget.ts, hideSearchOptions); N also moves several settings into search.common.contribution.ts and drops search.location, search.useRipgrep and search.maintainFileSearchCache. None were looked for in Cursor.
- The count of 30 is above the 20 guide because the group has many separately nameable controls (box, toggles, replace, filters, tree, row actions, header buttons) and I kept unlike things apart so each tag and Cursor status stays exact.
