Group: QA-2 Go to File and workspace search. Pass: 3. Date: 2026-10-25.

### QA-2.1
1. Pressing `Ctrl+P` (`Cmd+P` on Mac, and also `Ctrl+E` on Windows and Linux), typing part of a file name, and pressing Enter on a row opens that file and closes the list.
2. Choosing Go to File... in the Go menu, or Open File... in the right-click menu of the empty editor area or of the tab bar, opens the same list.
3. Each row shows a file icon, the file name, and the folder path next to it in grey.
4. A row for a file that has unsaved changes is read out by screen readers with the words "unsaved changes" added.
5. When nothing matches, the list shows "No matching results".
6. Files that match `files.exclude` or `search.exclude` are left out of the results.
7. Settings `search.useIgnoreFiles`, `search.useGlobalIgnoreFiles` and `search.useParentIgnoreFiles` each turn on or off hiding the files named in ignore files, and `search.followSymlinks` turns on or off searching inside symlinked folders.
8. With `workbench.editor.customLabels.enabled` on, a file that has a custom label shows that label as its name and its full relative path as the description.
9. In Cursor the box placeholder reads "Search files, content, and symbols (append : to go to line or @ to go to symbol)" and the rows come from Cursor's own ranking. [cursor only]

### QA-2.2
1. With an empty box the list shows the editor history under a "recently opened" heading.
2. Once you type, the history rows that match stay above the "file results" heading, and those same files are left out of the file results below.
3. Setting `search.quickOpen.includeHistory` (on by default) chooses whether history rows are included in the results while you type, and an empty box still lists history.
4. Setting `search.quickOpen.history.filterSortOrder` chooses how history rows are ordered while you type, with values `default` (most relevant first) and `recency` (most recently opened first).
5. History rows only match on the file name unless the typed text contains a slash, in which case they also match on the folder path.
6. In Cursor, history rows for files under a `/.cursor/worktrees/` folder are left out of the list. [cursor only]

### QA-2.3
1. Typed letters match out of order across the file name and the folder path, and each row draws its matched letters highlighted.
2. Typing a space starts a second search word, and a row stays in the list only if it matches every word.
3. Wrapping the typed text in double quotes asks for an exact match instead of a fuzzy one.
4. Typing a slash makes the match include the folder path and not only the file name.

### QA-2.4
1. Typing a full path to a file shows that exact file as the only result row.
2. Typing a path that starts with `~` shows the file from the home folder as the only result row.
3. Typing a path relative to a workspace folder shows that file as the only result row.
4. The single result shows even when `search.exclude` or `files.exclude` would normally hide the file, and all of its text is drawn highlighted.
5. When the typed path uses different letter casing than the file on disk, the row shows the file name with the casing from disk. [stock only]

### QA-2.5
1. After a file name, typing `:12`, `#12` or `(12)` opens the file at line 12 when you pick it.
2. After a file name, typing `:12:5` or `:12,5` opens the file at line 12, column 5.
3. After a file name, typing a range such as `:20-40` or `:20:3-40:5` opens the file with that range selected. [stock only]
4. Changing only the suffix after the file name keeps the current result rows in place instead of searching again.
5. In Cursor the file opens with the cursor collapsed to the start of the target position. [cursor only]

### QA-2.6
1. Typing `@` after a partial file name lists the symbols of the highlighted file in the same list.
2. The highlighted file opens in a temporary tab behind the list without taking keyboard focus.
3. The active symbol row scrolls to the middle of the editor and is highlighted there.
4. Pressing Escape puts the earlier editor view back.
5. Picking a symbol row opens the file with the cursor at that symbol.
6. Deleting the `@` part of the text brings the earlier file list back.

### QA-2.7
1. Each file row has an "Open to the Side" button that opens the file in a split on the side and closes the list.
2. Setting `workbench.editor.openSideBySideDirection` chooses the side, with values `right` and `down`, and with `down` the button reads "Open to the Bottom" and the file opens below.
3. Pressing `Alt+Enter` on a row opens that file to the side.
4. Pressing `Ctrl+Enter` on a row opens that file to the side while `workbench.editor.enablePreviewFromQuickOpen` is off, which is the default.
5. The same Open to the Side button appears on the symbol rows of the `#` and `@` lists.

### QA-2.8
1. Pressing Right Arrow on a file row opens that file in the background and the list stays open.
2. The file opened this way becomes a pinned tab, not a preview tab.
3. Focus stays in the list, so moving down and pressing Right Arrow again opens several files in a row.
4. Right Arrow opens the row only when focus is in the list or the text cursor is at the end of the input, and otherwise it moves the text cursor.
5. Symbol rows and Quick Search match rows also open in the background with Right Arrow.

### QA-2.9
1. By default a file picked from the list opens as a pinned tab.
2. With `workbench.editor.enablePreviewFromQuickOpen` on, a picked file opens as a preview tab, drawn in italics and replaced by the next preview.
3. With that setting on, holding Ctrl while picking a file (`Ctrl+Enter`) opens it as a pinned tab instead.
4. With `workbench.editor.enablePreview` off, preview tabs are off everywhere, so a picked file is pinned even when `workbench.editor.enablePreviewFromQuickOpen` is on.

### QA-2.10
1. A row that comes from editor history has an X button with the tooltip "Remove from Recently Opened".
2. Clicking the X removes that row from the list and removes the file from the editor history.
3. A history row for a file with unsaved changes shows a filled dot in place of the X, and the dot is always visible instead of only on hover.
4. Rows that are file results and not history have no X button.

### QA-2.11
1. With `search.quickOpen.includeSymbols` on (it is off by default), symbols from language servers are scored together with files in the same list.
2. The heading over the combined results reads "file and symbol results" in place of "file results".
3. Only symbols of top-level kinds (classes, enums, interfaces, namespaces, modules, packages and files) or symbols with no container are added.
4. No symbols are added while a line suffix such as `:12` is typed.
5. Picking a symbol row opens its file with the cursor at that symbol.

### QA-2.12
1. Pressing `Ctrl+T` (`Cmd+T` on Mac) or typing `#` in the quick access box lists symbols from all language servers and narrows them as you type.
2. Choosing Go to Symbol in Workspace... in the Go menu opens the same list.
3. When opened with the command, the box starts filled with the one-line text selected in the focused editor, or with the word at its cursor when nothing is selected.
4. Each row shows a kind icon, the symbol name, and a grey description made of the container name, a bullet, and the relative path of its file.
5. Deprecated symbols are drawn struck through.
6. With a second search word, the first word matches the symbol name and the rest match the container and path text.
7. Pressing Enter on a row opens the file with the cursor at the start of the symbol.
8. A search with no hits shows "No matching workspace symbols".
9. In Cursor the list is an older copy where the kind icon is written into the label text of each row. [cursor only]

### QA-2.13
1. Setting `search.quickAccess.excludeSymbolProviders` takes a list of extension ids, empty by default, and symbols supplied by those extensions do not appear in the `#` list. [cursor only]
2. Extension ids in the list match without regard to upper or lower case. [cursor only]
3. Changing the setting takes effect on the next search without reopening the list or reloading the window. [cursor only]

### QA-2.14
1. Typing `%` in the quick access box lists text matches from the workspace, with the matched text highlighted in each line.
2. Matches are grouped under one heading row per file that shows the file name and folder, and the file of the active editor is listed first.
3. The list shows up to 30 files, followed by a "See More Files" row, and up to 10 matches per file, followed by a "More" row.
4. Pressing Enter on a match row opens its file with the match selected, and `Alt+Enter` opens it to the side.
5. Running Search: Quick Search from the Command Palette starts with the selected text of the focused editor already typed in when `editor.find.seedSearchStringFromSelection` is on.
6. With an empty box the list shows "Enter a term to search for across your files."
7. A search with no hits shows "No matching results".
8. Setting `search.maxResults` limits how many matches the search collects, and a value of `null` means no limit.
9. Setting `search.smartCase` makes an all-lowercase search ignore case and a search with capitals match case.
10. Setting `search.sortOrder` chooses the order of the file groups, with values `default`, `fileNames`, `type`, `modified`, `countDescending` and `countAscending`.

### QA-2.15
1. Moving to a match row opens that file in a temporary tab at the match, without taking keyboard focus.
2. The file opens only after a 75 ms pause in moving, so stepping quickly through rows does not open every file.
3. Pressing Escape puts the earlier editor view back.
4. While results are listed, the matching text is highlighted in the open editors, and accepting a match clears those highlights.

### QA-2.16
1. The "Open in Search View" button at the right of the input moves the search into the Search view and closes the list.
2. The "Open in Search View" button on a match row opens the Search view with that match focused and closes the list.
3. The "More" and "See More Files" rows open the Search view with the next hidden match or file focused.
4. The "Open File" button on a file heading opens that file and closes the list.
5. In Cursor the input button only opens the Search view when the result list is still empty, and does not move a search over. [cursor only]

### QA-2.17
1. Setting `workbench.quickOpen.preserveInput` (off by default) brings back the last text typed in Go to File when the list opens again.
2. Setting `search.quickAccess.preserveInput` (off by default) brings back the last text typed in Quick Search when it opens again.
3. Restored text is shown selected, so typing replaces it.
4. Go to File and Quick Search each keep their own last text, and text from one does not appear in the other.

### QA-2.18
1. Running `workbench.action.quickOpenPreviousEditor` opens Go to File with an empty box and the editor history listed.
2. The second history row is the active row, so pressing Enter reopens the file you were just in.

### QA-2.19
1. By default every quick list, including Go to File, closes when focus moves out of it.
2. With `workbench.quickOpen.closeOnFocusLost` off, Go to File and every other quick pick stay open until you press Escape or pick something.
3. Pressing `Escape` or `Shift+Escape` while a quick list is open closes it.

### QA-2.20
1. The first group, headed "file results" (or "file and symbol results" when symbols are included), holds at most 6 matches, those that score 100 or more. [cursor only]
2. Every other match goes under an "other results" heading, with at most 512 rows in the whole list. [cursor only]
3. With several search words, a high-scoring row whose matched runs of letters average 3 letters or fewer is dropped from the results instead of shown. [cursor only]
4. The two-group list applies only while `search.quickOpen.useVscodeSearch` is off, which is its default. [cursor only]

### QA-2.21
1. Setting `search.quickOpen.useVscodeSearch`, off by default, when turned on makes Go to File use VS Code's own search instead of Cursor's modified one. [cursor only]
2. With the setting on, the list has a single "file results" group in place of the "file results" and "other results" groups. [cursor only]
3. With the setting on, no Create File row appears when nothing matches. [cursor only]

### QA-2.22
1. When nothing matches and the typed text looks like a path, the list shows "No matching results" and a row "Create File '<path>'" with the note "Create a new file at this path". [cursor only]
2. Typed text counts as a path when it is absolute, contains a slash, or ends in an extension such as `.ts`, and it does not end in a slash. [cursor only]
3. Pressing Enter on the Create File row makes the empty file, makes any missing folders, and opens the new file. [cursor only]
4. A relative path is created under the first workspace folder. [cursor only]
5. When a file already exists at the typed path, pressing Enter on the Create File row opens it and leaves it unchanged. [cursor only]
6. Pressing `Alt+Enter` or `Ctrl+Enter` on the Create File row opens the new file to the side. [cursor only]
7. When the file cannot be made, a "Failed to create file" message shows. [cursor only]

### QA-2.23
1. In VS Code 1.144 and later (not in 1.128.0), typing or pasting a link such as `scheme://...` into Go to File shows a link row. [stock only]
2. The link row has an "Open to the Side" button, and `Alt+Enter` opens the link to the side. [stock only]
3. A link that is not valid shows an "Invalid Link" row with the note "Enter a valid link to open it from Quick Open." [stock only]
4. The Go to File placeholder gains the words "or paste a link". [stock only]

## Unclear

- QA-2.1: the feature text says a row shows a filled dot for unsaved changes, but the source only draws the dot on rows that come from editor history (QA-2.10), so QA-2.1 only has the screen reader wording for unsaved files.
- QA-2.16: the feature text says the "Open File" button on a file heading hands results to the Search view, but the source opens that file and closes the list, so the piece follows the source.
- QA-2.20: the score 100 and cap 6 rule and the 512 limit were rechecked in the Cursor bundle, but the drop rule for several search words (matched runs averaging 3 letters or fewer) was taken from the feature block and not read again.
- Behavior on screen was not watched. Every piece comes from source or bundle strings.
