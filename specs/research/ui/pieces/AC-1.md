Group: AC-1 Suggest list. Pass: 3. Date: 2026-10-25.

### AC-1.1
1. Pressing `Ctrl+Space` or `Ctrl+I` in a writable editor opens the suggest list at the cursor.
2. On macOS the keys that open the list are `Ctrl+Space`, `Alt+Esc` and `Cmd+I`.
3. Running `Trigger Suggest` (`editor.action.triggerSuggest`) from the Command Palette opens the suggest list at the cursor.
4. The keys and the command open the list only while it is hidden, and do nothing in a read-only editor or in a language that has no completion provider.

### AC-1.2
1. While you type a word in an editor, the suggest list opens on its own after a short delay.
2. Setting `editor.quickSuggestionsDelay` sets how many milliseconds the list waits before it opens on its own, with a default of 10.
3. Setting `editor.quickSuggestions` chooses whether typing opens the list, with values `on`, `inline` (the suggestion is drawn as ghost text and not as a list) and `off`.
4. Setting `editor.suggest.snippetsPreventQuickSuggestions` chooses whether the list stays closed while typing inside an active snippet, with values on and off, default off.
5. In Cursor the `editor.quickSuggestions` entry for ordinary code defaults to `on`, so typing in code opens the list on its own out of the box. [cursor only]

### AC-1.3
1. Setting `editor.quickSuggestions` has three separate entries, `other`, `comments` and `strings`, so typing in ordinary code, inside a comment and inside a string can each open the list on their own or not.
2. By default the list opens on its own while typing in ordinary code and stays closed while typing inside a comment or inside a string.
3. Setting the `comments` entry to `on` makes the list open on its own while you type inside a comment.
4. Setting the `strings` entry to `on` makes the list open on its own while you type inside a string.
5. Setting the `other` entry to `off` stops the list from opening on its own in ordinary code while the comment and string entries keep their own values.

### AC-1.4
1. Typing a trigger character such as a dot opens the list at once, without waiting for the quick suggestion delay.
2. Typing a trigger character while the list is showing rows adds the rows from the sources that registered that character and keeps the rows that came from other sources.
3. Setting `editor.suggestOnTriggerCharacters` chooses whether trigger characters open the list, with values on and off, default on.

### AC-1.5
1. While ghost text from an inline suggestion is on screen, typing does not open the list on its own and trigger characters do not open it either, when inline suggestions are set to suppress it.
2. Setting `editor.inlineSuggest.suppressSuggestions` chooses whether an inline suggestion holds the list back, with values on and off.
3. Stock VS Code 1.128 adds the value `offWhenInlineCompletions` to `editor.quickSuggestions`, which turns opening on its own off only while inline completions are present, and Cursor does not have it. [stock only]
4. Stock VS Code 1.128 adds the value `offWithInlineSuggestions` to `editor.wordBasedSuggestions`, which turns word rows off only while inline suggestions are present, and Cursor does not have it. [stock only]

### AC-1.6
1. After a manual open, a small "Loading..." box appears at the cursor if the sources have not answered after a short delay.
2. When the answers arrive, the "Loading..." box is replaced by the rows.
3. When a manual open finds nothing, the box reads "No suggestions."
4. When an automatic open finds nothing, the list stays hidden and no message is shown.

### AC-1.7
1. Each row starts with a small icon for the kind of its item, such as method, function, class or snippet.
2. A row for a color item shows a small color swatch in place of the kind icon.
3. Rows for file and folder items show the icon from the active file icon theme.
4. Setting `editor.suggest.showIcons` chooses whether rows show icons, with values on and off, default on.
5. Stock VS Code 1.128 has a 29th kind, Tool, drawn with a tools icon, and Cursor has 28 kinds and no Tool icon. [stock only]

### AC-1.8
1. After the icon, each row shows the label of its item as the main text.
2. When an item has a structured label, a dimmed signature follows the label on the same row.
3. Rows with a structured label always show a smaller description or detail text at the right end.
4. Rows whose item has only plain detail text show that detail at the right end only on the highlighted row.
5. Setting `editor.suggest.showInlineDetails` chooses whether detail text shows inline with the label or only in the details pane, with values on and off, default on.

### AC-1.9
1. The letters you typed are drawn bold in a highlight color inside each row's label wherever they match.
2. The selected row draws its matched letters in a second highlight color that differs from the one on other rows.
3. While the list is frozen on its last results, the matched letters are not bold.

### AC-1.10
1. A row for an item the source marks as deprecated is dimmed and its label has a line through it.
2. A deprecated row gets no match highlighting on its label.
3. Setting `editor.suggest.showDeprecated` chooses whether deprecated rows are listed at all, with values on and off, default on.

### AC-1.11
1. Typing while the list is open narrows it to the rows whose labels fuzzy match what you typed.
2. The remaining rows are ordered by match quality and then by the sort text the source gave.
3. Setting `editor.suggest.filterGraceful` chooses whether filtering and sorting forgive small typos, with values on and off, default on.
4. Setting `editor.suggest.localityBonus` chooses whether words that appear close to the cursor sort higher, with values on and off, default off.
5. Setting `editor.suggest.matchOnWordStartOnly` chooses whether the first typed letter must match at the start of a word, with values on and off, default on.
6. After a manual open, if nothing matches any more the list freezes on its last results instead of closing, until you type a character that is not part of a word.

### AC-1.12
1. Snippets from the user, the workspace and extensions appear as rows with the snippet icon and the snippet prefix as the label.
2. A snippet row shows the snippet name as its description, and its detail text is the snippet description followed by the snippet source in brackets.
3. Setting `editor.snippetSuggestions` chooses where snippet rows sit, with values `top`, `bottom`, `inline` (mixed in by sort text) and `none` (left out).
4. Setting `editor.suggest.showSnippets` chooses whether snippet rows are listed, with values on and off, default on.

### AC-1.13
1. Plain words found in documents appear as rows with the text icon.
2. Setting `editor.wordBasedSuggestions` chooses which documents the words come from, with values `off`, `currentDocument`, `matchingDocuments` and `allDocuments`.
3. Setting `editor.suggest.showWords` chooses whether word rows are listed, with values on and off, default on.
4. In Cursor the setting `editor.wordBasedSuggestions` defaults to `matchingDocuments`. [cursor only]

### AC-1.14
1. Turning off one of the kind settings removes the rows of that kind from the list, and every kind setting is on by default.
2. Settings `editor.suggest.showMethods`, `editor.suggest.showFunctions`, `editor.suggest.showConstructors`, `editor.suggest.showFields`, `editor.suggest.showVariables`, `editor.suggest.showProperties` and `editor.suggest.showEvents` each choose whether rows of that member kind are listed, with values on and off.
3. Settings `editor.suggest.showClasses`, `editor.suggest.showStructs`, `editor.suggest.showInterfaces`, `editor.suggest.showModules`, `editor.suggest.showEnums` and `editor.suggest.showTypeParameters` each choose whether rows of that type kind are listed, with values on and off.
4. Settings `editor.suggest.showOperators`, `editor.suggest.showUnits`, `editor.suggest.showValues`, `editor.suggest.showConstants` and `editor.suggest.showEnumMembers` each choose whether rows of that value kind are listed, with values on and off.
5. Setting `editor.suggest.showKeywords` chooses whether keyword rows are listed, with values on and off.
6. Settings `editor.suggest.showColors`, `editor.suggest.showCustomcolors`, `editor.suggest.showFiles`, `editor.suggest.showFolders` and `editor.suggest.showReferences` each choose whether rows of that kind are listed, with values on and off.
7. Settings `editor.suggest.showUsers` and `editor.suggest.showIssues` each choose whether user and issue rows are listed, with values on and off.
8. Setting `editor.suggest.filteredTypes` is shown as deprecated with a message that points to the separate show settings, and a kind set to false in it turns the matching show setting off when that one has no value of its own.

### AC-1.15
1. With `editor.suggestSelection` set to `first`, the first row is highlighted whenever the list opens.
2. With `editor.suggestSelection` set to `recentlyUsed`, the row you last accepted in a similar place is highlighted when the list opens, if it is in the list.
3. With `editor.suggestSelection` set to `recentlyUsedByPrefix`, the row you accepted the last time you typed the same prefix is highlighted when the list opens.
4. Setting `editor.suggest.shareSuggestSelections` chooses whether the remembered choices are shared between workspaces and windows, with values on and off, default off.

### AC-1.16
1. A manual open always highlights a row, whatever `editor.suggest.selectionMode` says.
2. Setting `editor.suggest.selectionMode` chooses whether an automatic open highlights a row, with values `always`, `never`, `whenTriggerCharacter` and `whenQuickSuggestion`, default `always`.
3. With `never`, an automatic open shows the list with no row highlighted, so Tab and Enter insert nothing until you move the highlight.
4. With `whenTriggerCharacter` a row is highlighted only when a trigger character opened the list, and with `whenQuickSuggestion` only when typing opened it.

### AC-1.17
1. With `editor.suggest.preview` on, the text that the highlighted row would insert is drawn as ghost text in the editor behind the list.
2. Moving the highlight to another row redraws the ghost text for that row.
3. Closing the list removes the ghost text.
4. Setting `editor.suggest.preview` chooses whether the highlighted row is previewed in the editor, with values on and off, default off.

### AC-1.18
1. Setting `editor.suggestFontSize` sets the text size of the list, and a value of 0 uses the editor font size.
2. Setting `editor.suggestLineHeight` sets the row height of the list, and a value of 0 uses the editor line height, with a minimum of 8.
3. The status bar row and the details pane text follow the same list font size.

### AC-1.19
1. The list opens below the cursor, or above it when there is no room below.
2. The list opens 430 pixels wide and 12 rows high.
3. Dragging an edge of the list resizes it, and the next list opens at the size you left it.
4. Running `Reset Suggest Widget Size` (`editor.action.resetSuggestSize`) from the Command Palette clears the remembered size.
5. Setting `editor.suggest.maxVisibleSuggestions` is shown as deprecated with a message that the list can now be resized.

### AC-1.20
1. The list background, border and text take their colors from the theme colors `editorSuggestWidget.background`, `editorSuggestWidget.border` and `editorSuggestWidget.foreground`.
2. The selected row takes its colors from `editorSuggestWidget.selectedBackground`, `editorSuggestWidget.selectedForeground` and `editorSuggestWidget.selectedIconForeground`.
3. Match highlights and the status bar text take their colors from `editorSuggestWidget.highlightForeground`, `editorSuggestWidget.focusHighlightForeground` and `editorSuggestWidgetStatus.foreground`.
4. Setting `workbench.colorCustomizations` lets a person override any of these list colors.
5. The list frame is drawn with a rounded corner and a shadow.
6. In Cursor the frame has a fixed corner radius of 3px and its shadow does not use the large shadow token. [cursor only]

### AC-1.21
1. With a row highlighted, pressing `Ctrl+Space` or `Ctrl+I` opens the details pane beside the list.
2. Pressing the same keys again closes the details pane, and on macOS the keys are `Ctrl+Space` and `Cmd+I`.
3. Clicking the small X in the corner of the details pane closes it.
4. Whether the details pane is open or closed is remembered, so the next list opens the same way.

### AC-1.22
1. When the highlighted item has docs to show, a small chevron titled "Read More" appears at the right end of that row while the pointer is over it.
2. Clicking the "Read More" chevron opens the details pane.
3. The "Read More" chevron stays hidden while the status bar is shown.

### AC-1.23
1. The details pane shows the detail text of the item, such as its type or signature, at the top.
2. The details pane shows the docs of the item below the detail text, as plain text or as Markdown with links and code.
3. Long content scrolls inside the details pane.
4. While the item is still being resolved, the details pane shows "Loading...".
5. An item with no docs or no detail text leaves out the matching section of the pane.

### AC-1.24
1. The details pane sits to the right of the list by default.
2. The details pane moves to the left of the list or below it when there is no room on the right.
3. Dragging an edge of the details pane resizes it.
4. The details pane keeps the size you dragged it to until the list closes.
5. Stock VS Code 1.128 puts the details pane above the list when the list is drawn above the cursor, and Cursor has no placement above. [stock only]

### AC-1.25
1. Pressing `Ctrl+Alt+Space` while the list is open moves focus into the details pane, opening the pane first if it is closed, and pressing it again returns focus to the list.
2. While the details pane has focus, the page commands scroll it by a page and the first and last commands scroll it to the top and to the bottom.
3. Pressing `Ctrl+C` while the details pane has focus copies the selected text.
4. Clicking inside the details pane moves focus into it so its text can be selected.
5. Stock VS Code 1.128 runs the copy through the command `suggestWidgetCopy`, bound to `Ctrl+C` and, on Windows, `Ctrl+Insert` while the pane has focus, and Cursor has neither the command nor that focus condition. [stock only]
6. In Cursor, while the details pane has focus, any key other than `Ctrl+C` (`Cmd+C` on macOS) and the modifier keys sends focus back to the editor. [cursor only]

### AC-1.26
1. Pressing `Ctrl+/` while a row is highlighted turns the details pane into a debug view, opening the pane first if it is closed.
2. The debug view shows the match score, the typed prefix, the word that was matched, the locality distance, the sort index with the sort text it is based on, and the commit characters of the highlighted item.
3. The debug view shows the name of the provider that gave the highlighted item in place of the detail text.
4. Pressing `Ctrl+/` again turns the pane back into the normal details view.
5. With no row highlighted, `Ctrl+/` does nothing.

### AC-1.27
1. In stock VS Code main (added after 1.128.0), with the `fitWidthToDetails` suggest option on, the list opens wide enough to fit its longest label and that row's inline detail. [stock only]
2. The widened list never grows wider than the editor. [stock only]
3. When a size you dragged is remembered, the list keeps that size and is not widened. [stock only]

### AC-1.28
1. A thin status bar under the rows shows the accept action for the highlighted row, as Insert or as Replace depending on `editor.suggest.insertMode`.
2. When the item supports both, the status bar also shows the opposite action next to the first one.
3. Clicking an Insert or Replace action in the status bar accepts the highlighted row in that mode.
4. Setting `editor.suggest.showStatusBar` chooses whether the status bar is shown, with values on and off, default off.
5. In Cursor the Insert and Replace actions in the status bar are not limited to the moments when a row is highlighted. [cursor only]

### AC-1.29
1. At the right of the status bar a text action reads "Show More" while the details pane is closed, and clicking it opens the pane.
2. The same action reads "Show Less" while the details pane is open, and clicking it closes the pane.
3. The action appears only when the highlighted item has docs, a detail text that differs from its label, or can still be resolved.
4. The action is visible only while `editor.suggest.showStatusBar` is on.

### AC-1.30
1. In stock VS Code 1.128, when the list is open with no row highlighted and the status bar is shown, the status bar offers "Select" at the left. [stock only]
2. Clicking "Select" highlights the next row. [stock only]
3. Once a row is highlighted, "Select" gives way to the Insert and Replace actions. [stock only]
4. When `editor.suggest.selectionMode` leaves no row highlighted, for example with the value `never`, the status bar offers "Select" instead of Insert or Replace. [stock only]

### AC-1.31
1. With a row highlighted, pressing `Tab` inserts that row and closes the list.
2. Accepting a row also applies any extra edits it carries, such as an added import, and runs any follow-up command it names.
3. With the list open and no row highlighted, `Tab` does not insert a row.
4. In Cursor, `Tab` is also bound to `editor.action.acceptCursorTabSuggestion` with a higher priority, so while a Cursor Tab suggestion is waiting `Tab` accepts that suggestion and not the highlighted row. [cursor only]

### AC-1.32
1. With accept on enter on, pressing `Enter` with a row highlighted inserts that row and closes the list.
2. Setting `editor.acceptSuggestionOnEnter` chooses whether `Enter` accepts a row, with values `on`, `smart` and `off`.
3. With `smart`, `Enter` inserts the highlighted row only when doing so would change the text, so after you have typed the whole word `Enter` makes a new line.
4. With `off`, `Enter` never accepts a row.
5. A keybinding that uses the old command id `acceptSelectedSuggestionOnEnter` still accepts the highlighted row.

### AC-1.33
1. Typing a commit character of the highlighted item, such as a semicolon or a dot, inserts that item first and then types the character.
2. Setting `editor.acceptSuggestionOnCommitCharacter` chooses whether commit characters accept a row, with values on and off, default on.
3. Only the commit characters of the highlighted item count, so any other character is simply typed and the list narrows.

### AC-1.34
1. Accepting a row in insert mode leaves the text to the right of the cursor in place.
2. Accepting a row in replace mode overwrites the text to the right of the cursor that the item covers.
3. Setting `editor.suggest.insertMode` chooses which mode a normal accept uses, with values `insert` and `replace`, default `insert`.
4. Pressing `Shift+Enter` or `Shift+Tab` with a row highlighted accepts it in the opposite mode when the item supports both.

### AC-1.35
1. Clicking or tapping a row inserts that row and closes the list.
2. After a row is inserted by click, keyboard focus is in the editor.
3. Pressing the mouse down inside the list does not take focus away from the text.

### AC-1.36
1. Pressing `Down` or `Ctrl+Down` moves the highlight to the next row, and on macOS `Ctrl+N` does the same.
2. Pressing `Up` or `Ctrl+Up` moves the highlight to the previous row, and on macOS `Ctrl+P` does the same.
3. The highlight wraps from the last row to the first row and from the first row to the last.
4. Pressing `PageDown` or `Ctrl+PageDown` moves the highlight down one page, and `PageUp` or `Ctrl+PageUp` moves it up one page.
5. Commands `selectFirstSuggestion` and `selectLastSuggestion` move the highlight to the first and last row, and they have no default keys.
6. These keys move the highlight only when the list has more than one row or no row is highlighted yet.

### AC-1.37
1. With the list open and no row highlighted, pressing `Ctrl+Space` or `Ctrl+I` highlights the first row, and on macOS the keys are `Ctrl+Space` and `Cmd+I`.
2. Command `focusAndAcceptSuggestion` highlights the first row and accepts it in one step, and it has no default key.

### AC-1.38
1. Pressing `Escape` or `Shift+Escape` while the list is open closes the list and the details pane.
2. The list closes when the editor loses focus.
3. The list closes when the cursor moves to another line.
4. A list that opened by itself closes when what you type no longer matches any row.

### AC-1.39
1. With `editor.tabCompletion` on, pressing `Tab` at the end of a word with the list hidden inserts the best match straight away, with no list shown.
2. Pressing `Tab` again steps to the next match when there are others, and `Shift+Tab` steps back to the previous one.
3. Setting `editor.tabCompletion` chooses whether `Tab` completes a word, with values `on`, `off` and `onlySnippets`, default `off`.
4. `Tab` completes this way only at the end of a word, while the list is hidden and no snippet is active.

### AC-1.40
1. The list is exposed to screen readers as a listbox, and the text area points at the highlighted row.
2. Each row is read with its label, signature, description and kind.
3. While the details pane is open, a row is also read with its detail text and docs.
4. "Loading..." and "No suggestions." are announced when they appear.
5. Accepting an item that has extra edits announces how many edits were made.

### AC-1.41
1. Pressing `Backspace` while the list is open re-filters the list for the new cursor position.
2. Moving the cursor left with the arrow keys onto another word refreshes the list with rows for that word.
3. Starting a new word to the right of the cursor shows rows for the new word and reuses the rows the list already has.
4. Moving the cursor with the mouse, or making a selection, closes the list instead of re-filtering it.
5. The list closes when the leading whitespace of the line changes, for example when the word is outdented.

### AC-1.42
1. Each kind icon takes its own color from a per-kind theme color, such as `symbolIcon.methodForeground`, `symbolIcon.functionForeground` and `symbolIcon.classForeground`.
2. There are 33 kind colors, and the same colors are used for the matching icons in the Outline and in the breadcrumbs.
3. Setting `workbench.colorCustomizations` lets a person override each kind color.

### AC-1.43
1. When ghost text from an inline suggestion is on screen and the list opens, the row whose text is the start of the ghost text is highlighted first.
2. When several rows start the ghost text, the longest one is highlighted.
3. The row that matches the ghost text wins over the remembered choice from `editor.suggestSelection`.
4. When no row starts the ghost text, the usual highlight choice applies.

### AC-1.44
1. With `editor.suggest.preview` on, holding `Shift` redraws the ghost text of the highlighted row as it would look if accepted in the opposite insert or replace mode.
2. Letting go of `Shift` draws the normal preview again.
3. Pressing `Shift+Enter` or `Shift+Tab` while the opposite-mode preview is showing inserts that previewed result.
4. With `editor.suggest.preview` off, holding `Shift` draws no ghost text, and `editor.suggest.insertMode` decides which mode counts as the normal one.

### AC-1.45
1. With `editor.suggest.preview` on, when the previewed text of the highlighted row shows as ghost text on two or more lines, the list is drawn above the cursor so it does not cover the ghost lines.
2. The list moves above the cursor only when there are at least 150 pixels of room above it.
3. The list returns to its normal placement once the preview is gone.

### AC-1.46
1. In a Markdown file, typing inside a link target lists files and folders as rows in the suggest list.
2. Header rows also appear in the same list for link targets.
3. Setting `markdown.suggest.paths.enabled` chooses whether path rows are offered, with values on and off, default on.
4. Setting `markdown.suggest.paths.includeWorkspaceHeaderCompletions` chooses when headers from other workspace files appear, with values `never`, `onDoubleHash` (after `##`) and `onSingleOrDoubleHash` (after `#` or `##`), default `onDoubleHash`.
5. Markdown files default `editor.quickSuggestions` to `off` for other code, comments and strings, so the list does not open on its own while typing in Markdown.

## Unclear
- AC-1.14 holds 27 settings and was grouped into kind families to stay within 10 pieces. It could be split into one feature per family if per-setting tracking is wanted. `editor.suggest.showCustomcolors` looks inert in 1.128.0 because the filter reads `showColors` for that kind.
- AC-1.17 has no piece for `editor.suggest.previewMode` because it has no settings key.
- AC-1.25 piece 2 is read from stock source. In Cursor the details key handler sends every key except `Ctrl+C` back to the editor, so whether the page, first and last scroll keys still scroll the pane there was not confirmed.
- AC-1.27 describes an option that has no settings key in newer VS Code, so a person cannot switch it on from the Settings editor.
- AC-1.28 piece 5 is read from the missing has-focused-suggestion condition in the Cursor menu entries. What a person sees when no row is highlighted was not run.
- AC-1.43 was not checked for whether any Cursor source actually feeds ghost text to the list selector, so the piece may only apply when ghost text is present.
- AC-1.46 pieces 1 and 2 follow the feature's What sentence. The exact trigger positions for header rows inside a link target were not read from the language server.
