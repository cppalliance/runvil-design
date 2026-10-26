Group: SB-7 Outline, Timeline and References. Pass: 3. Date: 2026-10-25.

### SB-7.1
1. The Outline pane sits under the Explorer tree, starts collapsed, and lists the symbols of the active text editor as a tree of classes, functions and other kinds.
2. Clicking a symbol row, or focusing it and pressing Enter, moves the editor to that symbol.
3. Running `outline.focus` from the Command Palette reveals the Outline pane and puts keyboard focus in it.
4. While symbols are loading the pane says "Loading document symbols for '<file>'...", and when none come back it says "No symbols found in document '<file>'".
5. When the active editor cannot provide symbols the pane says "The active editor cannot provide outline information."
6. The Outline title has a Collapse All button (`outline.collapse`) that folds every row, and an Expand All button (`outline.expand`) that opens every row.
7. Setting `outline.collapseItems` chooses whether rows start collapsed or expanded, with values `alwaysExpand` (default) and `alwaysCollapse`, and it can differ per language.

### SB-7.2
1. Choosing Follow Cursor from the Outline "..." menu turns on following, and the menu row shows a checked state while it is on.
2. With Follow Cursor on, moving the caret in the editor moves the selected Outline row to the symbol that holds the caret.
3. With Follow Cursor off, moving the caret leaves the Outline selection where it was.
4. The Follow Cursor choice is remembered the next time the Outline opens.

### SB-7.3
1. Typing while the Outline has focus opens a find box at the top of the tree.
2. With Filter on Type on, rows that do not match the typed text are hidden and the tree opens up so deep matches show.
3. With Filter on Type off, all rows stay visible and only the matching text is highlighted.
4. Filter on Type in the Outline "..." menu is a checked toggle that matches the current find mode, and flipping the find mode from the find box updates the menu row too.
5. The Filter on Type choice is remembered the next time the Outline opens.

### SB-7.4
1. The Outline "..." menu has three Sort By choices (`outline.sortByPosition`, `outline.sortByName`, `outline.sortByKind`) and only one is checked at a time.
2. Sort By: Position lists rows in the order the symbols appear in the file.
3. Sort By: Name lists rows alphabetically by symbol name.
4. Sort By: Category groups rows by symbol kind.
5. The chosen sort order is remembered the next time the Outline opens.

### SB-7.5
1. Setting `outline.icons` turns the symbol-kind icon on each Outline row on or off, and it is on by default.
2. Setting `outline.collapseItems` chooses whether the Outline opens with all rows collapsed or all expanded, with values `alwaysExpand` and `alwaysCollapse`, and it can be set per language.
3. Setting `outline.problems.enabled` turns error and warning markers on Outline rows on or off, and it is on by default.
4. Setting `outline.problems.colors` colors Outline rows red or yellow for errors and warnings, and `outline.problems.badges` adds a count badge to those rows.
5. Both `outline.problems.colors` and `outline.problems.badges` have no visible effect while `problems.visibility` is off.
6. Turning off a `outline.show<Kind>` setting hides rows of that kind from the Outline, all 26 kinds are on by default, and each can be set for one language only.
7. The 26 `outline.show<Kind>` settings cover files, modules, namespaces, packages, classes, methods, properties, fields, constructors, enums, interfaces, functions, variables, constants, strings, numbers, booleans, arrays, objects, keys, null, enum members, structs, events, operators and type parameters.
### SB-7.6
1. When a notebook is the active editor, the Outline lists the notebook's markdown headers and code cells instead of text symbols.
2. A Filter Entries button in the Outline title opens a menu of three checked options, Markdown Headers Only, Code Cells and Code Cell Symbols.
3. Setting `notebook.outline.showMarkdownHeadersOnly` hides markdown cells that have no header, and is the same switch as Markdown Headers Only in the Filter Entries menu.
4. Setting `notebook.outline.showCodeCells` shows or hides code cell rows, and is the same switch as Code Cells in the Filter Entries menu.
5. Setting `notebook.outline.showCodeCellSymbols` shows or hides symbols found inside code cells, needs `notebook.outline.showCodeCells` on, and is the same switch as Code Cell Symbols in the Filter Entries menu.
6. Each notebook Outline row has an action menu with Run Cell on a code cell row.
7. A markdown header row's action menu has Run Cells In Section, Fold Section and Expand Section, which run or fold or open the cells under that header in the notebook.

### SB-7.7
1. The Timeline pane sits under Outline in the Explorer container and lists the open file's earlier versions, newest first, with entries from every source that has some.
2. The Timeline header shows the file name beside the title.
3. A Refresh button in the Timeline title (`timeline.refresh`) reloads the list, and `timeline.focus` reveals the pane.
4. Scrolling to the end of the list loads the next page, and a "Load more" row at the end loads it on click.
5. Setting `timeline.pageOnScroll` turns the load-on-scroll behavior on or off, and when off only the "Load more" row loads more.
6. Setting `timeline.pageSize` chooses how many entries show at first and per page, and `null` sizes the page to the visible area.
7. The pane says "The active editor cannot provide timeline information." when the editor has nothing to show, and "All timeline sources have been filtered out." when every source is hidden.
8. When a source returns nothing the pane says "No timeline information was provided." or "No filtered timeline information was provided.", with a Local History hint or "Source Control has not been configured." where that applies.
9. While entries load the pane says "Loading timeline for <file>...".
10. The Timeline pane exists only when at least one timeline source is registered.

### SB-7.8
1. A pin button in the Timeline title (`timeline.toggleFollowActiveEditor`, "Pin the Current Timeline") locks the pane to the file it now shows.
2. While pinned, switching editor tabs does not change the Timeline list.
3. While pinned, the Timeline header reads "<file> (pinned)".
4. When pinned the same button changes to "Unpin the Current Timeline", and pressing it makes the pane follow the active editor again.

### SB-7.9
1. A Filter Timeline button in the Timeline title opens a list with one checked row per timeline source, such as Git History and Local History.
2. Unticking a source hides its entries from the Timeline and ticking it shows them again.
3. The ticked and unticked choice is kept in the profile (`timeline.excludeSources`) and applies the next time the window opens.
4. Hiding every source makes the pane say "All timeline sources have been filtered out."

### SB-7.10
1. Right-clicking a file in the Explorer shows an Open Timeline row (`files.openTimeline`).
2. Choosing Open Timeline reveals the Timeline pane with the history of that file, even when it is not the active editor.
3. After Open Timeline the pane is pinned to the chosen file, so its header reads "<file> (pinned)" and switching tabs does not change it until it is unpinned.
4. The Open Timeline row shows only on file rows, not folder rows, and only when some timeline source is registered.

### SB-7.11
1. A Pin Workspace Timeline button in the Timeline title (`timeline.showWorkspaceTimeline`) switches the pane from one file to a timeline for the whole workspace. [stock only]
2. While the workspace timeline shows, the Timeline header reads "Workspace" instead of a file name. [stock only]
3. While the workspace timeline loads, the pane says "Loading workspace timeline...". [stock only]
4. The Pin Workspace Timeline button shows only when some source can provide a workspace timeline and the pane is not already showing it. [stock only]

### SB-7.12
1. Every save of a file adds a Local History entry to the Timeline, timestamped and labelled with what caused the save.
2. Hovering a Local History entry shows a card with its details.
3. Saves that happen within a short window replace the last entry instead of adding a new one, and setting `workbench.localHistory.mergeWindow` chooses that window in seconds, default 10.
4. Clicking a Local History entry opens a comparison between that entry and the current file.
5. Setting `workbench.localHistory.enabled` turns Local History on or off, default on, and turning it off stops new entries without removing old ones.
6. Setting `workbench.localHistory.maxFileSize` chooses the largest file in KB that gets entries, default 256, and larger files get none.
7. Setting `workbench.localHistory.maxFileEntries` chooses how many entries a file keeps, default 50, and the oldest entries are dropped past that.
8. Setting `workbench.localHistory.exclude` lists paths or glob patterns whose files get no Local History entries.
9. When a file has no entries the pane says "Local History will track recent changes as you save them unless the file has been excluded or is too large."

### SB-7.13
1. Right-clicking a Local History entry and choosing Compare with File opens a comparison between that entry and the current file.
2. Choosing Compare with Previous opens a comparison between that entry and the entry before it.
3. Choosing Select for Compare marks that entry as the left side of a comparison.
4. After one entry is marked, right-clicking another Local History entry shows Compare with Selected, and choosing it opens a comparison of the two.
5. The comparison tab title names each side with the file name plus its save source and time in brackets, and a Compare with File tab ends with the file name again.

### SB-7.14
1. Right-clicking a Local History entry and choosing Show Contents opens that old copy in an editor.
2. Choosing Restore Contents on an entry first asks "Do you want to restore the contents of '<file>'?" with the note "Restoring will discard any unsaved changes." and a Restore button.
3. Confirming Restore replaces the file with that old copy, and if it fails an error dialog says "Unable to restore '<file>'."
4. An old copy open in an editor shows a Restore Contents button in the editor title (`workbench.action.localHistory.restoreViaEditor`).
5. Find Entry to Restore, in the Command Palette and in the Timeline "..." menu as "Local History: Find Entry to Restore...", opens a picker first for the file ("Select the file to show local history for") and then for the entry ("Select the local history entry to open").
6. The "Local History: Find Entry to Restore..." row in the Timeline "..." menu shows only while `workbench.localHistory.enabled` is on.

### SB-7.15
1. Right-clicking a Local History entry and choosing Rename opens an input titled "Rename Local History Entry" with the placeholder "Enter the new name of the local history entry".
2. Confirming a new name changes the label of that entry in the Timeline.
3. Right-clicking an entry and choosing Delete asks "Do you want to delete the local history entry of '<file>' from <date>?" with the note "This action is irreversible!" and a Delete button.
4. Confirming Delete removes that entry from the Timeline.
5. Delete All in the Command Palette asks "Do you want to delete all entries of all files in local history?" with the note "This action is irreversible!" and a Delete All button, and confirming clears the history of every file.
6. Create Entry in the Command Palette opens an input titled "Create Local History Entry" for naming a snapshot of the active file, and confirming adds that snapshot as a new entry.

### SB-7.16
1. The Git History source lists the commits that changed the open file, each row showing the commit message and a relative time.
2. Each Git History row shows the commit author, and setting `git.timeline.showAuthor` turns the author on or off, default on.
3. Hovering a Git History row shows a card with the full commit.
4. Setting `git.timeline.date` chooses whether row times use the committed date or the authored date, with values `committed` (default) and `authored`.
5. A Staged Changes row sits at the top of the Git History entries when the file has staged changes.
6. Setting `git.timeline.showUncommitted` adds an Uncommitted Changes row for the file, default off.

### SB-7.17
1. Clicking a Git History row opens the changes of that commit for the file in a diff editor.
2. Right-clicking a Git History row and choosing Open Changes (`git.timeline.openDiff`) does the same thing.
3. A View Commit button shows inline on commit rows, and View Commit in the right-click menu does the same, opening the whole commit in a multi-file diff.
4. Open Changes shows on every Git History row, but View Commit shows only on real commit rows and not on the Staged Changes or Uncommitted Changes rows.
5. None of these entries show when several Timeline rows are selected, or when `git.enabled` is off or Git is missing.

### SB-7.18
1. Right-clicking a Git History row and choosing Select for Compare marks that commit as the left side of a comparison.
2. After one row is marked, right-clicking a different Git History row shows Compare with Selected.
3. Choosing Compare with Selected opens a diff of the file between the two commits.

### SB-7.19
1. Right-clicking a Git History row and choosing Copy Commit Hash puts that commit's full hash on the clipboard.
2. Right-clicking a Git History row and choosing Copy Commit Message puts that commit's message on the clipboard.
3. The two Copy rows show only on real commit rows, not on the Staged Changes or Uncommitted Changes rows, and not when several rows are selected.

### SB-7.20
1. Running Find All References on a symbol fills the Reference Search Results view with every use of that symbol.
2. Find All References is reached by right-click in the editor, by the Command Palette, and by `Shift+Alt+F12`, and all three only work when the language has a reference provider.
3. The view lives in its own References container with a References icon, and it stays open and keeps its results while you edit and switch files.
4. Setting `references.preferredLocation` chooses what clicking a CodeLens reference count opens, with values `peek` (default, a peek editor) and `view` (this References view).

### SB-7.21
1. Running Find All Implementations on an interface or abstract member lists the implementing places in the Reference Search Results view.
2. The Find All Implementations row in the editor right-click menu shows only when the language has an implementation provider.

### SB-7.22
1. Results are grouped by file, with each matching line shown under its file row.
2. A line above the tree reads "N results in M files", with singular forms such as "1 result in 1 file".
3. Clicking a result row opens that file in the editor at the match.
4. While the view is open the matches are highlighted in the editor text and in its overview ruler.
5. Result rows can be dragged out of the view as links.
6. When a search finds nothing the view says "No results."
7. The Reference Search Results view appears only after a search has run (`reference-list.isActive`).

### SB-7.23
1. Pressing `F4` jumps to the next result, opening it in the editor and selecting it in the view.
2. Pressing `Shift+F4` jumps to the previous result, opening it in the editor and selecting it in the view.
3. Both keys work only while the view has results (`reference-list.hasResult`).
4. Go to Next Reference and Go to Previous Reference are hidden from the Command Palette.

### SB-7.24
1. Each file row and reference row has an inline Dismiss button that removes that row from the list.
2. Right-clicking a file row or reference row also offers Dismiss.
3. Right-clicking a file row or reference row offers Copy, which puts that row's text on the clipboard, and Copy All, which puts all results on the clipboard.
4. Right-clicking a file row also offers Copy Path, which puts the file path on the clipboard, and reference rows do not have it.
5. Dismiss, Copy, Copy All and Copy Path are hidden from the Command Palette.

### SB-7.25
1. A Refresh button in the view title reruns the search that made the current results.
2. A Clear button in the view title empties the results.
3. After Clear the view says "No results." or, when earlier searches exist, "No results. Try running a previous search again:" above the history list.
4. Refresh and Clear show in the view title and the Command Palette only while the view has results.

### SB-7.26
1. When the view has no results it lists earlier searches, each with a Rerun button.
2. Pressing Rerun on a history row, or choosing Rerun from its right-click menu, runs that search again.
3. Show History (`references-view.pickFromHistory`) opens a picker titled "Select previous reference search" and choosing one shows its results.
4. Clear History forgets every earlier search, and its button shows in the view title only while there is history and no results.
5. Show History and Clear History are in the Command Palette only while the References view is active, and Clear History also needs no current results.

### SB-7.27
1. Show Call Hierarchy lists the callers of the symbol as a tree in the Reference Search Results view.
2. Show Call Hierarchy is reached by right-click in the editor and by `Shift+Alt+H`, and both work only when the language has a call hierarchy provider.
3. The tree is headed Callers Of or Calls From, and a title button (Show Outgoing Calls while callers show, Show Incoming Calls while calls-from show) switches the whole tree to the other direction.
4. Right-clicking a call row offers Show Incoming Calls and Show Outgoing Calls for that row.
5. Each call row has an inline Dismiss button, and Dismiss in its right-click menu removes the row.
6. When the symbol has no calls the view says "No results."

### SB-7.28
1. Show Type Hierarchy lists the supertypes or subtypes of a type as a tree in the Reference Search Results view.
2. Show Type Hierarchy is reached by right-click in the editor, and only works when the language has a type hierarchy provider.
3. The tree is headed Supertypes Of or Subtypes Of, and a title button (Show Supertypes or Show Subtypes, whichever is not the current one) switches the whole tree to the other direction.
4. Right-clicking a type row offers Show Supertypes and Show Subtypes for that row.
5. Each type row has an inline Dismiss button, and Dismiss in its right-click menu removes the row.
6. When the type has no supertypes or subtypes the view says "No results."

## Unclear
- SB-7.5 and SB-7.1 both own the setting `outline.collapseItems`, so each has a piece for it. A tracker may count it twice.
- SB-7.11 exists only in the newer 1.144 tree and is not in Cursor, so all its pieces are marked `[stock only]` and its behavior was read from the newer tree, not from 1.128.0.
- SB-7.6 pieces on the row action menu (Run Cell, Run Cells In Section, Fold Section, Expand Section) were taken from the feature block and the command titles, and which action shows on which row type was not read in detail.
- The hover card contents for SB-7.12 and SB-7.16 entries were not read, so those pieces only say a card shows.
- SB-7.20 piece 3 says the view keeps its results while you edit, taken from the manifest description "separate, stable view", and was not run on screen.
- SB-7.9 piece 3 names the storage key `timeline.excludeSources`, taken from the feature block, and whether the choice survives a restart was not tested.
