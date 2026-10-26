Group: SB-3 Search. Pass: 3. Date: 2026-10-25.

### SB-3.1
1. Pressing `Ctrl+Shift+F` (`workbench.action.findInFiles`) opens the Search view with the cursor in the search box.
2. Edit > Find in Files and View > Search open the Search view too.
3. The search box is pre-filled with the text selected in the editor.
4. Setting `search.seedWithNearestWord` (off by default) fills the box with the word nearest the cursor when the editor has no selection.
5. Setting `search.seedOnFocus` (off by default) updates the query to the editor selection whenever the Search view gets focus by a click or by the `workbench.views.search.focus` command.
6. A keybinding can pass `workbench.action.findInFiles` a JSON argument (`query`, `replace`, `preserveCase`, `triggerSearch`, `filesToInclude`, `filesToExclude`, `isRegex`, `isCaseSensitive`, `matchWholeWord`, `useExcludeSettingsAndIgnoreFiles`, `onlyOpenEditors`, `showIncludesExcludes`) to open a prefilled search.
7. Setting `search.globalFindClipboard` (macOS only, off by default) lets the Search view read and write the shared find clipboard.

### SB-3.2
1. Typing in the search box runs the search after a short pause, and results fill in as you type.
2. Pressing Enter in the box runs the search at once.
3. Pressing Esc in the box stops a running search.
4. Pressing `Ctrl+Enter` (`Control+Enter` on macOS) adds a new line so the box can hold a multi-line query.
5. In the box, Page Up and Page Down jump to the start or end of the text, and Tab moves to the next control.
6. Toggle Search on Type (`workbench.action.toggleSearchOnType`) turns searching as you type on or off.
7. Setting `search.searchOnType` chooses whether results update as you type, on by default.
8. Setting `search.searchOnTypeDebouncePeriod` chooses the pause in milliseconds between a typed character and the search starting, with a default of 300.
9. Setting `search.ripgrep.maxThreads` chooses how many threads the search uses, with 0 (the default) letting the engine decide.

### SB-3.3
1. The Match Case toggle at the right edge of the search box (`toggleSearchCaseSensitive`, `Alt+C`) makes the search case sensitive.
2. The Match Whole Word toggle (`toggleSearchWholeWord`, `Alt+W`) makes the search match whole words only.
3. The Use Regular Expression toggle (`toggleSearchRegex`, `Alt+R`) makes the search text a regular expression.
4. A broken regular expression shows its error text under the box and the search does not run.
5. The three keys work while the Search view is focused, and on macOS they are `Cmd+Alt+C`, `Cmd+Alt+W` and `Cmd+Alt+R`, where the case key also needs a file or folder row not to be focused.
6. Setting `search.smartCase` (off by default) searches case-insensitively when the pattern is all lowercase and case-sensitively otherwise.
7. Setting `search.usePCRE2` is deprecated, and advanced regular expression features are now used automatically when the pattern needs them.

### SB-3.4
1. While a notebook is open, a filter button (Notebook Find Filters) joins the toggles in the search box.
2. The filter button's drop-down has four choices - Markdown Source, Rendered Markdown, Code Cell Source and Code Cell Output - that pick which cell parts are searched.
3. A match inside a notebook cell opens in the notebook editor.
4. Setting `search.experimental.closedNotebookRichContentResults` (off by default) shows rich content results for closed notebooks after the search is refreshed.

### SB-3.5
1. Up Arrow in the search, replace, include or exclude box shows the previous entry, and Down Arrow shows the next one (`history.showPrevious`, `history.showNext`).
2. `Alt+Up Arrow` and `Alt+Down Arrow` step through the same entries.
3. Each of the four boxes keeps its own list, saved for the workspace.
4. The arrow keys step through history only while a history box is focused and there is an entry to move to.
5. Clear Search History (`search.action.clearHistory`) in the Command Palette empties all four lists.

### SB-3.6
1. The chevron at the left of the search box (Toggle Replace) opens a Replace box under the search box and closes it again.
2. Pressing `Ctrl+Shift+H` (`workbench.action.replaceInFiles`) or choosing Edit > Replace in Files opens the Search view with the Replace box already open.
3. The Replace box has a Preserve Case toggle (`toggleSearchPreserveCase`, `Alt+P`, `Cmd+Alt+P` on macOS) that keeps the capital letters of the matched text in the replacement.
4. Pressing Esc in the Replace box closes it (`closeReplaceInFilesWidget`).

### SB-3.7
1. With a replace term typed and a search run, a Replace button on a match row replaces that one match (`search.action.replace`).
2. A Replace All button on a file row replaces every match in that file (`search.action.replaceAllInFile`).
3. A Replace All button on a folder row replaces every match in that folder (`search.action.replaceAllInFolder`).
4. The right-click menu on a result row has the matching Replace or Replace All entry.
5. `Ctrl+Shift+1` runs the replace for the focused row kind, and `Ctrl+Shift+Enter` runs Replace All for a focused file or folder row, both only while the Replace row is active.
6. The files that were changed are saved after the replacement.

### SB-3.8
1. The Replace All button at the end of the Replace row is disabled, with the tooltip "Replace All (Submit Search to Enable)", until a search has run, and it goes back to disabled when the search text changes.
2. Clicking Replace All, or pressing `Ctrl+Alt+Enter` (`search.action.replaceAll`) while the Replace row is active and the editor find widget is not showing, opens a confirm box titled Replace All that asks "Replace N occurrences across M files?" with a Replace button.
3. Nothing changes until the confirm box is accepted, and declining leaves every file as it was.
4. After accepting, a progress indicator shows while the files are changed.
5. When it finishes, the view reports "Replaced N occurrences across M files." with the singular wording when the count is 1.

### SB-3.9
1. With a replace term typed, clicking a match opens a side-by-side diff titled "<file> ↔ <file> (Replace Preview)" that shows what the replacement would do, instead of opening the plain file.
2. A match inside a notebook still opens in the notebook editor.
3. Setting `search.useReplacePreview` chooses whether selecting or replacing a match opens Replace Preview, on by default.

### SB-3.10
1. The "..." button under the search box (Toggle Search Details, `workbench.action.search.toggleQueryDetails`) shows or hides the include and exclude boxes.
2. The "files to include" and "files to exclude" boxes take comma-separated globs (for example `*.ts, src/**/include`) that limit where the search looks.
3. Each of the two boxes keeps its own history.
4. Pressing `Ctrl+Shift+J` while the Search view is focused toggles the details, and it also works in the Search Editor.

### SB-3.11
1. A toggle in the exclude box (Use Exclude Settings and Ignore Files) is on by default and makes the search skip files matched by `search.exclude`, `files.exclude` and the `.gitignore` and `.ignore` files.
2. Turning the toggle off finds those files too, and the result count line then says "exclude settings and ignore files are disabled" with an "enable" link that turns the toggle back on.
3. Setting `search.exclude` lists glob patterns the search skips, with defaults of `**/node_modules`, `**/bower_components` and `**/*.code-search`, and it also takes in the patterns of `files.exclude`.
4. Settings `search.useIgnoreFiles` (on by default), `search.useGlobalIgnoreFiles` (off) and `search.useParentIgnoreFiles` (off) choose whether `.gitignore` and `.ignore` files, the global gitignore file, and ignore files in parent folders are used, where the last two need the first one on.
5. Setting `search.followSymlinks` chooses whether the search follows symlinks, on by default.

### SB-3.12
1. A book-icon toggle in the include box (Search only in Open Editors) limits the search to files that are open in editors.
2. While it is on, the result count line says "searching only in open files" with a "disable" link that searches the whole workspace again.
3. Clicking that link shows the search details area if it was hidden and turns the toggle off.

### SB-3.13
1. An edit-icon toggle in the include box (Search only in Changed Files (Source Control)) limits the search to the files changed in Source Control. [stock only]
2. The toggle is enabled only while Source Control has changes. [stock only]
3. Turning this toggle on switches the open editors toggle off, and turning the open editors toggle on switches this one off. [stock only]

### SB-3.14
1. Right-clicking a folder in the Explorer and choosing Find in Folder... (`filesExplorer.findInFolder`) opens Search with that folder in the include box.
2. Find in Workspace... (`filesExplorer.findInWorkspace`) is the variant shown when the right-clicked root is not a folder, and it opens Search with the include box cleared.
3. Pressing `Shift+Alt+F` while the Explorer is focused on a folder runs Find in Folder.

### SB-3.15
1. Right-clicking a folder row in the results and choosing Restrict Search to Folder (`search.action.restrictSearchToFolder`) puts that folder into the include box and reruns the search.
2. Choosing Exclude Folder from Search (`search.action.excludeFromSearch`) puts that folder into the exclude box and reruns the search.
3. Pressing `Shift+Alt+F` while a folder row that has a path is focused runs Restrict Search to Folder.

### SB-3.16
1. Right-clicking a file row and choosing Exclude File Type from Search (`search.action.excludeFileTypeFromSearch`) adds the file's extension pattern, for example `*.test.ts`, to the exclude box and reruns the search. [stock only]
2. Choosing Include File Type from Search (`search.action.includeFileTypeInSearch`, spelled that way in the source) adds the pattern to the include box and reruns the search. [stock only]

### SB-3.17
1. Matches appear under file rows that show a file icon, the file name, the folder path and a match count badge whose hover text reads "N matches found".
2. Each match line shows its text with the matched part highlighted.
3. An "Other files" row holds matches in files outside the workspace folders, and a multi-root workspace gets one row per root folder.
4. Setting `search.showLineNumbers` (off by default) shows the line number and a colon before each match.
5. A match that spans several lines shows "+N" in the line number column, and hovering it reads "From line N" and "N more lines".
6. With a replace term typed, each match shows the replacement text next to the matched text.
7. Settings `search.decorations.colors` and `search.decorations.badges` (both on by default) choose whether file rows use colors and badges from file decorations.
8. Setting `search.actionsPosition` chooses where the row buttons sit, with values `right` (default, always at the right edge) and `auto` (at the right edge when the view is narrow and right after the content when it is wide).
9. Hovering over, focusing or selecting a result row shows its small action buttons, such as Dismiss and Replace.

### SB-3.18
1. A header button flips the results between a flat list of files and a folder tree, showing View as Tree while the results are a list and View as List while they are a tree (`search.action.viewAsTree`, `search.action.viewAsList`).
2. In the tree layout, file rows hide the folder path because the folder rows above them show it.
3. Setting `search.defaultViewMode` chooses the layout a new search starts in, with values `list` (default) and `tree`.

### SB-3.19
1. The Collapse All header button (`search.action.collapseSearchResults`) folds every file row.
2. When every row is already folded, the same button becomes Expand All (`search.action.expandSearchResults`) and unfolds everything.
3. Right-clicking a folder row adds Expand Recursively (`search.action.expandRecursively`), which unfolds everything under it.
4. Setting `search.collapseResults` chooses how a new search starts, with values `auto` (files with fewer than 10 results open, others folded), `alwaysCollapse` and `alwaysExpand` (default).
5. When a search finds results in only one file, that file's matches open by themselves.

### SB-3.20
1. By default the files in the results are listed alphabetically by folder and then file name.
2. Setting `search.sortOrder` chooses the order, with values `default` (folder and file name), `fileNames` (file name only), `type` (extension), `modified` (last modified time, newest first), `countDescending` and `countAscending` (match count per file).
3. With `modified` or a count order, the list re-sorts by itself when a result file is edited or a match count changes.

### SB-3.21
1. Clicking a match opens its file in the editor with the match selected and highlighted.
2. Double-clicking a match, or pressing Enter on a focused file or match row, keeps the editor tab open (`search.action.openResult`, also `Cmd+Down` on macOS).
3. Pressing `Ctrl+Enter` (`Control+Enter` on macOS) on a focused file or match row opens it beside the current editor (`search.action.openResultToSide`).
4. Setting `workbench.list.openMode` chooses whether a `singleClick` (default) or a `doubleClick` opens a match with the mouse.

### SB-3.22
1. With a file or match row focused, pressing `Ctrl+Shift+L` (`addCursorsAtSearchResults`) opens that file and puts a cursor on each match the search found in it.
2. The key is active only while the Search view is visible and a file or match row is focused.

### SB-3.23
1. The X button on a result row removes that match, file or folder from the results without touching the file.
2. Pressing Delete (`Cmd+Backspace` on macOS) on a focused result row dismisses it (`search.action.remove`).
3. Dismiss in the right-click menu of a result row does the same.
4. Dismissing a file or folder row removes every match under it.

### SB-3.24
1. Right-clicking a match or file row and choosing Copy copies its text, and `Ctrl+C` does the same while a file or match row is focused (`search.action.copyMatch`).
2. Copy Path on a file or folder row copies its path, and `Ctrl+Alt+C` (`Shift+Alt+C` on Windows) does the same while such a row is focused (`search.action.copyPath`).
3. Copy All copies every result as text (`search.action.copyAll`).

### SB-3.25
1. Right-clicking a file row and choosing Reveal in Explorer View (`search.action.revealInSideBar`) opens the Explorer and selects that file.
2. The entry shows on a file row only when there are results, and it works only for files inside the workspace.

### SB-3.26
1. Pressing `Ctrl+Down` in an input box moves focus to the next box and then to the results (`search.focus.nextInputBox`).
2. Pressing `Ctrl+Up` in an input box other than the search box moves focus to the previous box (`search.focus.previousInputBox`).
3. Pressing `Ctrl+Up` while the first match is focused, or while accessibility mode is on, returns focus to the search box (`search.action.focusSearchFromResults`).
4. Pressing `F4` selects the next match and `Shift+F4` selects the previous match, from anywhere (`search.action.focusNextSearchResult`, `search.action.focusPreviousSearchResult`).
5. Focus List (`search.action.focusSearchList`) moves focus into the results list.

### SB-3.27
1. The Refresh header button (`search.action.refreshSearchResults`) reruns the current search while the search is not slow.
2. While a search is slow the Refresh button is replaced by a Cancel Search button, and pressing Esc while the Search view is visible and a list has focus also stops it (`search.action.cancel`).
3. The Clear Search Results header button (`search.action.clearSearchResults`) empties the boxes and the results.

### SB-3.28
1. The Open New Search Editor header button (`search.action.openNewEditorFromView`) opens a Search Editor that keeps the file filters and toggles from the Search view.
2. Open Results in Editor (`search.action.openInEditor`, `Alt+Enter`, `Cmd+Enter` on macOS) copies the current results into a Search Editor while there are results and the Search view is focused.
3. The "Open in editor" link in the result count line does the same, with the hover text "Copy current search results to an editor".
4. An Open New Search Editor button in the title bar of the Search view (`search.action.openNewEditorFromView`) opens a Search Editor tab.
5. Open Results in Editor (`search.action.openInEditor`) from the Command Palette, or `Alt+Enter` (`Cmd+Enter` on macOS) while the Search view is focused and has results, opens a Search Editor tab holding the current query, include and exclude patterns and results.
6. When `search.searchEditor.defaultNumberOfContextLines` is above 0, the new Search Editor runs the search again so each match shows its surrounding lines, and when it is 0 the results are copied as they are.

### SB-3.29
1. By default, Find in Files, Find in Folder and Find in Workspace show their results in the Search view.
2. Setting `search.mode` chooses where they show, with values `view` (default), `reuseEditor` and `newEditor`.
3. With `reuseEditor` the search goes to an existing Search Editor when one is open and to a new one otherwise, while `newEditor` always opens a new Search Editor.

### SB-3.30
1. A line under the boxes reports "N results in M files" after a search, using "result" and "file" when a count is 1.
2. When nothing is found, the line explains why and names the include and exclude patterns in use, for example "No results found in '<include>' excluding '<exclude>' - ".
3. After a no-results message, a link follows it - "Search again in all files" when an include or exclude pattern is set, and "Open Settings" when neither is set.
4. When the open editors toggle is on and nothing is found, the message starts "No results found in open editors" instead.
5. When a search was canceled before any results were found, the message reads "Search was canceled before any results could be found - " followed by a "Search again" link.
6. Setting `search.maxResults` (default 20000, empty for unlimited) caps the list, and when the cap cuts it short a warning reads "The result set only contains a subset of all matches. Be more specific in your search to narrow down the results."
7. A window with no folder shows "You have not opened or specified a folder. Only open files are currently searched - " with an Open Folder link.
8. The no-results text without patterns reads "Review your settings for configured exclusions and check your gitignore files" where stock reads "Review your configured exclusions and check your gitignore files", and there is no Search with AI link or keyword suggestion. [cursor only]

## Unclear
- SB-3.30 does not repeat the count line reminders "exclude settings and ignore files are disabled" and "searching only in open files". They are pieces 2 of SB-3.11 and SB-3.12, and the "Open in editor" link in that line is piece 3 of SB-3.28.
- SB-3.13 and SB-3.16 are tagged `[stock only]` on every piece because pass 2 found no trace of their ids, labels or members in the Cursor bundle. Nothing was run, so this is strings and ids, not observed behavior.
- SB-3.30 piece 8 combines the older no-results wording with the missing AI link in one piece because the feature has one Cursor field. The AI search link itself is not a feature of this group.
- SB-3.10 piece 4 and Cursor: Ctrl+Shift+J is also bound to Cursor Settings (see pass 2 for SB-3). Which one wins while the Search view is focused was not determined, so the piece states the stock behavior only.
- SB-3.21 has no right-click piece because 1.128.0 has no right-click item for Open Match or Open to the Side.
