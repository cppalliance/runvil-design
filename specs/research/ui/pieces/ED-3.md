Group: ED-3 Find and replace. Pass: 3. Date: 2026-10-25.

### ED-3.1
1. Pressing `Ctrl+F` (`Cmd+F` on macOS) in a focused editor opens a small find box at the top right of the editor with the cursor in the search field.
2. Choosing Edit > Find from the menu bar, or running Find (`actions.find`) from the Command Palette, opens the same find box.
3. Pressing the shortcut while the find box is already open puts the cursor back in the search field with its text selected.

### ED-3.2
1. Running `editor.actions.findWithArgs` from a keybinding opens the find box with the search text, regex, whole word and case flags from its JSON argument already set (`searchString`, `isRegex`, `matchWholeWord`, `isCaseSensitive`).
2. When the argument includes `replaceString`, the find box opens with the replace row showing and that replacement text filled in, and `preserveCase` sets the Preserve Case toggle.
3. When the argument sets `findInSelection` to true, the find box opens already limited to the current selection.
4. Find with Arguments is listed in the Command Palette and has no default key, so run with no argument it opens a normal find box.

### ED-3.3
1. Opening find with a single-line selection puts the selected text in the find box, and with an empty selection it puts in the word at the cursor.
2. Find with Selection (`actions.findWithSelection`, `Cmd+E` on macOS only) puts the selected text in the find box, including a multi-line selection, and leaves focus in the editor.
3. Setting `editor.find.seedSearchStringFromSelection` chooses what fills the find box, with values `never` (nothing), `always` (the selection, or the word at the cursor, the default), and `selection` (only a non-empty selection).

### ED-3.4
1. Typing in the find box highlights matches as you type, and the editor selection jumps to the nearest match without leaving the box.
2. Setting `editor.find.cursorMoveOnType` (on by default) chooses whether the cursor jumps to a match while typing, and when it is off the matches are only highlighted.
3. Setting `editor.find.findOnType` (on by default) chooses whether the search runs as you type, and when it is off the search runs only when Enter is pressed in the find box. [stock only]

### ED-3.5
1. Next to the arrow buttons the find box shows the position of the current match as "3 of 12".
2. When nothing matches, the box shows "No results" and the find box gets a red outline.
3. When there are more than 19999 matches the count reads "19999+", and hovering it shows "Only the first 19999 results are highlighted, but all find operations work on the entire text."
4. The Previous Match and Next Match arrow buttons are greyed out while the find box is empty or has no matches.

### ED-3.6
1. Pressing `F3` (`Cmd+G` on macOS, `F3` also) in the editor moves to the next match, and `Shift+F3` (`Cmd+Shift+G` on macOS, `Shift+F3` also) moves to the previous one.
2. Pressing `Enter` in the find box moves to the next match and `Shift+Enter` to the previous one, and focus stays in the box.
3. The down arrow and up arrow buttons in the find box do the same, and Find Next (`editor.action.nextMatchFindAction`) and Find Previous (`editor.action.previousMatchFindAction`) are in the Command Palette.
4. At the last match, Find Next wraps to the first match, and at the first match, Find Previous wraps to the last.
5. Setting `editor.find.loop` (on by default) chooses whether the search wraps, and when it is off the arrow buttons are greyed out at the first or last match.
6. Pressing `F3` with the find box closed and no earlier search opens the find box without moving focus into it, filled from the selection or word at the cursor, and goes to the next match.

### ED-3.7
1. With the find box open, Go to Match (`editor.action.goToMatchFindAction`) from the Command Palette opens an input that says "Type a number to go to a specific match (between 1 and N)".
2. Typing a number moves the editor to that match as you type and highlights the whole line of the target match, and pressing Enter keeps that match.
3. A negative number counts from the end, so -1 goes to the last match.
4. A number outside the range shows the message "Please type a number between 1 and N" in the input.
5. With no matches, Go to Match shows a warning "No matches. Try searching for something else." instead of the input.
6. Go to Match is available in the Command Palette only while the find box is open.

### ED-3.8
1. Find Next Selection (`editor.action.nextSelectionMatchFindAction`, `Ctrl+F3` with the editor focused) moves to the next place where the selected text appears, without typing it in the find box.
2. Find Previous Selection (`editor.action.previousSelectionMatchFindAction`, `Ctrl+Shift+F3`) moves to the previous place where the selected text appears.
3. Using either one with the find box closed still finds the next or previous place, and focus stays in the editor instead of moving into the find box.

### ED-3.9
1. The "Aa" Match Case toggle at the right end of the find box makes the search case sensitive when on, and shows as highlighted while on.
2. Pressing `Alt+C` (`Cmd+Alt+C` on macOS) with the editor focused, or running `toggleFindCaseSensitive`, flips the same toggle and re-runs the search.
3. Hovering the toggle shows its name with the keyboard shortcut.

### ED-3.10
1. The "ab" Match Whole Word toggle in the find box limits matches to whole words when on, and shows as highlighted while on.
2. Pressing `Alt+W` (`Cmd+Alt+W` on macOS) with the editor focused, or running `toggleFindWholeWord`, flips the toggle and re-runs the search.
3. Hovering the toggle shows its name with the keyboard shortcut.

### ED-3.11
1. The ".*" Use Regular Expression toggle in the find box treats the search text as a regular expression when on, and shows as highlighted while on.
2. Pressing `Alt+R` (`Cmd+Alt+R` on macOS) with the editor focused, or running `toggleFindRegex`, flips the toggle and re-runs the search.
3. Hovering the toggle shows its name with the keyboard shortcut.

### ED-3.12
1. The selection-icon Find in Selection button in the find box limits the search to the selected range, shades that range in the editor, and shows as highlighted while on.
2. Pressing `Alt+L` (`Cmd+Alt+L` on macOS) with the editor focused, or running `toggleFindInSelection`, flips the button.
3. Setting `editor.find.autoFindInSelection` chooses when the button switches on by itself when find opens, with values `never` (the default), `always`, and `multiline` (only when the selection covers more than one line).
4. The button is greyed out when the editor has no selection and it is not already on.

### ED-3.13
1. Flipping Match Case, Whole Word or Regex with its key while the find box is closed shows small toggle badges at the top right of the editor, showing the new state of each option.
2. The badges hide by themselves after about two seconds, and stay while the mouse is over them.
3. Hovering a badge shows its name and shortcut, and clicking a badge flips that option.

### ED-3.14
1. Pressing `Ctrl+H` (`Cmd+Alt+F` on macOS), choosing Edit > Replace, or running Replace (`editor.action.startFindReplaceAction`) opens the find box with a second row for the replacement text.
2. A chevron at the left edge of the find box (Toggle Replace) shows or hides the replace row, and shows a right-pointing chevron when hidden and a down-pointing one when shown.
3. When the editor has a selected single line, Replace fills the find box with it and puts the cursor in the replace field, and when it has none the cursor starts in the find field.
4. Pressing `Tab` in the find box moves the cursor to the replace box.
5. In a read-only editor, Replace does nothing and the chevron is greyed out.

### ED-3.15
1. The Replace button (replace icon) at the right of the replace box replaces the current match and moves to the next one.
2. Pressing `Enter` in the replace box does the same, and `Ctrl+Shift+1` does it while the find box is visible.
3. When the editor selection is not on a match, the first Replace only moves to the next match, and the following Replace changes it.
4. The Replace button is greyed out while the find box is empty.

### ED-3.16
1. The Replace All button (replace-all icon) at the right of the replace box replaces every match in the file in one step.
2. Pressing `Ctrl+Alt+Enter` while the find box is visible runs Replace All, and so does `Cmd+Enter` on macOS with the replace box focused.
3. The whole replace is one undo step.
4. On a file that is too large for Replace All, a warning says "The file is too large to perform a replace all operation."
5. The Replace All button is greyed out while the find box is empty.

### ED-3.17
1. The "AB" Preserve Case toggle at the right end of the replace box, when on, makes each replacement copy the capital letters of the match it replaces (all caps, first letter capital, or lower case).
2. Pressing `Alt+P` (`Cmd+Alt+P` on macOS) with the editor focused, or running `togglePreserveCase`, flips the toggle.
3. Hovering the toggle shows its name with the keyboard shortcut.

### ED-3.18
1. With Use Regular Expression on, `$1` to `$99` in the replacement insert the text matched by that capture group, and `$&` or `$0` insert the whole match.
2. `$$` in the replacement inserts a single dollar sign.
3. `\n` in the replacement inserts a line break, `\t` inserts a tab, and `\\` inserts one backslash.
4. `\u` or `\l` placed before a `$n` makes the first letter of the inserted text upper or lower case, and `\U` or `\L` makes all of it upper or lower case.
5. With Use Regular Expression off, the replacement is inserted exactly as typed, and none of these codes apply.

### ED-3.19
1. Pressing `Alt+Enter` while the find box is visible (`editor.action.selectAllMatches`) puts a selection on every match in the file, so typing changes all of them at once.
2. Focus moves back from the find box to the editor text after the matches are selected.
3. With no matches, the command does nothing.

### ED-3.20
1. Pressing `Ctrl+Enter` in the find box inserts a line break when no keybinding handles it, and on macOS it is the real Ctrl key, not Cmd.
2. The find box grows taller as lines are added, up to 118 px, and then scrolls.
3. Pressing `Ctrl+Enter` in the replace box inserts a line break in the replacement text in the same way when no keybinding handles it.
4. A find text with several lines matches those lines across line breaks in the file.

### ED-3.21
1. Pressing `Up Arrow` in the find box fills it with the previous search string, and `Down Arrow` moves to the next one (`history.showPrevious`, `history.showNext`).
2. The replace box does the same with earlier replacement strings.
3. Earlier search and replace strings are remembered for the workspace and are still there after reopening it.
4. Setting `editor.find.history` chooses whether search strings are remembered, with values `never` and `workspace` (the default).
5. Setting `editor.find.replaceHistory` chooses whether replacement strings are remembered, with values `never` and `workspace` (the default).
6. While `Up Arrow` and `Down Arrow` are the history keys, each box shows a short hint that they step through history.

### ED-3.22
1. Pressing `Escape` (`closeFindWidget`) while the find box is open closes it and returns focus to the editor text.
2. The Close button at the right end of the find box does the same, and hovering it shows "Close" with its key.
3. `Shift+Escape` closes the find box when it is visible and the editor has focus.
4. Closing the find box turns Find in Selection off.

### ED-3.23
1. With `editor.find.closeOnResult` on, pressing Find Next or Find Previous from the find box (`F3`, `Shift+F3`, `Enter`) and landing on a match closes the find box. [stock only]
2. Setting `editor.find.closeOnResult` is off by default, and when it is off the find box stays open after landing on a match. [stock only]
3. Pressing Find Next from the editor with the find box open does not close it, and only a step taken from the find box does. [stock only]

### ED-3.24
1. Dragging the left edge of the find box makes it wider than its starting width of 419 px.
2. Double-clicking the left edge resets the find box to its starting width.
3. In a narrow editor the find box first hides the match count, then shrinks to a narrow layout, and in a very narrow editor it collapses and hides the arrow buttons, replace buttons and option toggles.

### ED-3.25
1. While the find box is open, a blank band appears at the top of the editor so the first line can be scrolled out from under the box.
2. Setting `editor.find.addExtraSpaceOnTop` (on by default) chooses whether that band is added, and when it is off the first line stays under the find box.

### ED-3.26
1. Every match of the search text gets a highlight in the editor, and the current match gets a stronger one.
2. Each match draws a mark in the scrollbar strip and in the minimap.
3. With more than 1000 matches, nearby matches merge into broader marks in the scrollbar strip, while the text highlights stay on every match.

### ED-3.27
1. On macOS with `editor.find.globalFindClipboard` on, opening find fills the find box from the system find pasteboard, so text searched for in another app shows up here.
2. A search typed in the editor is written back to the system find pasteboard, so other apps can pick it up.
3. Setting `editor.find.globalFindClipboard` turns this on or off for the editor, and is off by default and only meaningful on macOS.
4. Setting `search.globalFindClipboard` does the same for the Search view, so a search typed in the editor appears in the Search view and the other way round, and is off by default and only meaningful on macOS.

### ED-3.28
1. Pressing `Alt+F1` with focus anywhere in the find box opens an accessibility help page for find. [stock only]
2. The help page gives the current search term, the match position as "Match N of M", the replacement text when the replace row is showing, and what you hear when moving to a match. [stock only]
3. The help page also lists the keys for the find box, replace box and editor, explains the find options, names the related settings, and tells you how to close find. [stock only]
4. Closing the help page returns focus to the part of the find box that had it. [stock only]
5. With screen reader hints on, the find box and replace box labels add "Press the shortcut for accessibility help". [stock only]
6. Setting `accessibility.verbosity.find` chooses whether the find box announces that help hint. [stock only]

### ED-3.29
1. New Search Editor (`search.action.openNewEditor`) in the Command Palette opens a Search Editor as a new tab with the cursor in its query box.
2. Open New Search Editor to the Side (`search.action.openNewEditorToSide`) opens the Search Editor in the editor group to the side.
3. Open Search Editor (`search.action.openEditor`) goes back to the most recently used Search Editor and puts the selected text in its query box, and opens a new one when none is open.
4. A new Search Editor's query starts with the text selected in the editor, and with an empty selection it starts with the word at the cursor when `search.seedWithNearestWord` is on.
5. A Search Editor opened with a seeded query runs the search at once when `search.searchOnType` is on.
6. Setting `search.searchEditor.reusePriorSearchConfiguration` (off by default) makes a new Search Editor start with the include, exclude and flag settings of the previous one.
7. A keybinding can pass these commands a JSON argument (`query`, `filesToInclude`, `filesToExclude`, `contextLines`, `matchWholeWord`, `isCaseSensitive`, `isRegexp`, `useExcludeSettingsAndIgnoreFiles`, `showIncludesExcludes`, `onlyOpenEditors`, `triggerSearch`, `focusResults`) to open a prefilled search, with the cursor going to the results instead of the query box when `focusResults` is true.

### ED-3.31
1. The top of a Search Editor has a query box, and pressing `Enter` in it runs the search.
2. Match Case, Match Whole Word and Use Regular Expression toggles sit at the right end of the query box, and changing one runs the search again.
3. A context-lines toggle beside them shows or hides the lines around each match and runs the search again.
4. With the query box focused, `Alt+C`, `Alt+W` and `Alt+R` (`Cmd+Alt+C`, `Cmd+Alt+W`, `Cmd+Alt+R` on macOS) flip the three toggles (`toggleSearchEditorCaseSensitive`, `toggleSearchEditorWholeWord`, `toggleSearchEditorRegex`), and each is also in the Command Palette under Search Editor.
5. Focus Search Editor Input (`search.action.focusQueryEditorWidget`, `Escape` in the Search Editor) moves the cursor from the results back to the query box.

### ED-3.32
1. A Toggle Search Details button under the query box shows or hides the "files to include" and "files to exclude" boxes.
2. Typing a pattern in either box filters the search and runs it again.
3. A switch in the include box limits the search to open editors, and runs it again.
4. A switch in the exclude box chooses whether the exclude settings and ignore files apply, and runs the search again.
5. Focus Files to Include and Focus Files to Exclude (`search.action.focusFilesToInclude`, `search.action.focusFilesToExclude`) in the Command Palette move the cursor to each box.
6. A Search Editor loaded with include or exclude patterns, or with ignore files turned off, opens with the details area already shown.

### ED-3.33
1. The results text starts with header lines for the query (`# Query:`) and, when set, the flags (`# Flags:` with `CaseSensitive`, `WordMatch`, `RegExp`, `OpenEditors`, `IgnoreExcludeSettings`), the included files, the excluded files and the context lines count.
2. Below the header a count line reads "12 results - 4 files" (or "1 result - 1 file"), and "No Results" when nothing matched.
3. Each file with matches appears as a path heading ending in a colon, with its match lines underneath, each starting with its line number and a colon.
4. Context lines show with their line number and no colon, and a blank line separates groups of lines that are not next to each other.
5. When the result set was capped, a note below the count line says "The result set only contains a subset of all matches. Be more specific in your search to narrow down the results."

### ED-3.34
1. Toggle Context Lines (`toggleSearchEditorContextLines`, `Alt+L`, `Cmd+Alt+L` on macOS) turns the lines above and below each match on or off and runs the search again.
2. Increase Context Lines (`increaseSearchEditorContextLines`, `Alt+=`) adds one more line around each match, and Decrease Context Lines (`decreaseSearchEditorContextLines`, `Alt+-`) takes one away.
3. The three commands are also in the Command Palette under Search Editor.
4. Setting `search.searchEditor.defaultNumberOfContextLines` chooses how many context lines a new Search Editor starts with, 1 by default, and an empty value reuses the previous Search Editor's number when `search.searchEditor.reusePriorSearchConfiguration` is on.

### ED-3.35
1. While a Search Editor is the active tab, a refresh button in the editor title bar runs the search again with the current query and options.
2. Search Again (`rerunSearchEditorSearch`) from the Command Palette, or `Ctrl+Shift+R` (`Cmd+Shift+R` on macOS) in a Search Editor, does the same.
3. Editing the header lines at the top of the results text, such as `# Query:`, changes the query and options that the next search uses.
4. A Search Editor opened with a query but no results shows a "Run Search" link, and clicking it runs the search.
5. Typing in the "files to include" or "files to exclude" box can run the search after the pause set by `search.searchOnTypeDebouncePeriod`.

### ED-3.36
1. Double-clicking a match line opens that file at that line in the active editor group by default.
2. Setting `search.searchEditor.doubleClickBehaviour` chooses what double-click does, with values `selectWord`, `goToLocation` (the default) and `openLocationToSide`.
3. Setting `search.searchEditor.singleClickBehaviour` chooses what single-click does, with values `default` (nothing, the default) and `peekDefinition` (opens a peek window showing the match in its file).
4. Pressing `F12` on a result line, or `Ctrl+click` on a result line, opens the file at that match, and `Ctrl+click` on a file heading opens the file.
5. Setting `search.searchEditor.focusResultsOnSearch` (off by default) chooses whether the cursor moves to the results or stays in the query box when a search runs.

### ED-3.37
1. Delete File Results (`search.searchEditor.action.deleteFileResults`, `Ctrl+Shift+Backspace` in a Search Editor) removes the file heading, match lines and context lines of the file under the cursor, and leaves the other files.
2. Delete File Results is in the Command Palette under Search Editor while a Search Editor is active.

### ED-3.38
1. Select All Matches (`selectAllSearchEditorMatches`, `Ctrl+Shift+L` in a Search Editor) puts a selection on every match in the results, so typing changes them together.
2. Select All Matches is in the Command Palette under Search Editor while a Search Editor is active.

### ED-3.39
1. A new Search Editor opens as an untitled tab, and saving it writes a `.code-search` file.
2. Opening a saved `.code-search` file shows it in the Search Editor with its query, options and results.
3. An unsaved Search Editor comes back after restarting the window, still marked as unsaved.
4. Running the search again in a saved `.code-search` tab marks the tab as modified, and the internal command `cleanSearchEditorState` clears that mark.

### ED-3.40
1. Match lines in the results show in bold and context lines show dimmed.
2. The search-result language colors the results, such as the header lines, file headings and line numbers.
3. The Outline view lists each file in the results as a file entry.
4. A file heading shows as a link when you hold `Ctrl` over it.
5. Line numbers in the editor gutter are hidden by default for the Search Editor.
6. Typing at the start of one of the first lines, or after `#`, suggests the header lines `# Query:`, `# Flags:`, `# Including:`, `# Excluding:` and `# ContextLines:`, and on the flags line it suggests `RegExp`, `CaseSensitive`, `IgnoreExcludeSettings` and `WordMatch`.

### ED-3.41
1. With Use Regular Expression on, typing a pattern that does not compile, such as an open bracket, shows a red error box under the find box with the engine's own error text.
2. The error box goes away when the pattern is fixed.
3. The error box goes away when Use Regular Expression is turned off, and an empty find box never shows one.

### ED-3.42
1. Match Case, Whole Word and Use Regular Expression stay as you left them when you open find in another file.
2. Preserve Case stays as you left it in the same way.
3. These four options are still as you left them after closing and reopening the workspace.
4. Find in Selection is switched off each time you switch to another file.

### ED-3.43
1. Each time the match count or the current match changes, the find box speaks the full text of the matched line, then a message such as "3 of 12 found for 'foo', at 14:5".
2. When nothing matches, it speaks "No results found for 'foo'", or just "No results found" while the find box is empty.
3. Pressing Find Next or Find Previous with the find box open and nothing found speaks the no-results message again.

### ED-3.44
1. Pressing `Tab` in the find box moves to the replace box when the replace row is showing, and to the Match Case toggle when it is not.
2. Pressing `Tab` in the replace box moves to the Match Case toggle, and `Shift+Tab` in the replace box moves back to the find box.
3. With the replace row showing, `Tab` from the Use Regular Expression toggle moves to the Preserve Case toggle, and `Shift+Tab` from the Match Case toggle moves back to the replace box.
4. `Tab` then continues through the Previous Match, Next Match, Find in Selection and Close buttons and the Replace and Replace All buttons, and `Space` or `Enter` on a focused button or toggle activates it.
5. Pressing `Ctrl+Down` (`Cmd+Down` on macOS) in the find box or replace box puts the cursor back in the editor and leaves the find box open.

### ED-3.45
1. A Search Editor tab shows a search icon.
2. The tab title reads "Search: " followed by the query, with a query longer than 12 characters cut to 12 characters plus "...".
3. The tab title is just "Search" while the query is empty.
4. A saved `.code-search` file shows the title "Search: " followed by the file name without its extension.
5. The tab title changes as you change the query.

### ED-3.46
1. In a Search Editor, `F4` (`search.action.focusNextSearchResult`) moves the cursor to the next highlighted match in the results, and `Shift+F4` (`search.action.focusPreviousSearchResult`) moves to the previous one.
2. Past the last match `F4` wraps to the first, and before the first match `Shift+F4` wraps to the last.
3. The match is scrolled into view when the cursor moves to it.
4. A screen reader announces "Matched <text> at <line> in file <path>" for each match reached.
5. Focus Next Search Result and Focus Previous Search Result are in the Command Palette under Search, and can be run when the Search view has results or the active editor is a Search Editor.
6. In Cursor, those two commands pass the move to the Search Editor's own `searchEditor.action.focusNextResult` and `searchEditor.action.focusPreviousResult`, and the person sees the same move to the next or previous match. [cursor only]

### ED-3.47
1. In a Search Editor, `Ctrl+Down` (`Cmd+Down` on macOS, `search.focus.nextInputBox`) in the query box moves the cursor to "files to include", then "files to exclude", then the results.
2. `Ctrl+Up` (`Cmd+Up` on macOS, `search.focus.previousInputBox`) moves back the other way, and from the query box it wraps to the results.
3. The "files to include" and "files to exclude" boxes are skipped while Toggle Search Details has them hidden.
4. In Cursor, those two commands pass the move to the Search Editor's own `searchEditor.action.focusNextInput` and `searchEditor.action.focusPreviousInput`, and the person sees the same hop between boxes. [cursor only]

### ED-3.48
1. While a notebook is open, a filter button appears with the toggles in the Search Editor's query box.
2. The button opens four choices, Markdown Source, Rendered Markdown, Code Cell Source and Code Cell Output, that can each be checked or unchecked.
3. The checked choices decide which parts of notebook cells the Search Editor's search looks in.
4. A new Search Editor starts with Markdown Source, Code Cell Source and Code Cell Output checked, and Rendered Markdown unchecked.

### ED-3.49
1. The matched text on each result line in a Search Editor is shaded.
2. The shading follows the theme colors `searchEditor.findMatchBackground` and `searchEditor.findMatchBorder`.
3. The shaded ranges are kept with an unsaved Search Editor, so they show again when it comes back after a restart.

### ED-3.50
1. On Windows desktop, the first time `Ctrl+Enter` is pressed in the replace box and no keybinding handles it, an information notification says "Ctrl+Enter now inserts line break instead of replacing all. You can modify the keybinding for editor.action.replaceAll to override this behavior." [cursor only]
2. The notification shows only once, and does not come back after it has been shown. [cursor only]
3. The notification does not appear on macOS or Linux, or when a keybinding handles `Ctrl+Enter`. [cursor only]

## Unclear

- ED-3.46 and ED-3.47 are marked changed in Cursor only because Cursor routes the keys through its own Search Editor commands, so the last piece of each states that routing and no visible difference was found.
- ED-3.48 piece 4 reads the starting choices from the Search Editor default configuration; whether the query box widget shows the same choices at startup was not confirmed.
