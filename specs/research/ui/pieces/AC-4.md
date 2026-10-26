Group: AC-4 Snippets. Pass: 3. Date: 2026-10-25.

### AC-4.1
1. After a snippet is inserted, pressing Tab moves to the next tab stop and selects its placeholder text, or puts a bare cursor there when the stop is empty.
2. Pressing Shift+Tab moves back to the previous tab stop in the same way.
3. Tab stops that share a number are linked, so they are selected together and typing into one types into all of them.
4. Reaching the last stop (`$0`, or the highest number when there is no `$0`) ends snippet mode, and Tab goes back to its normal job.
5. Tab and Shift+Tab only jump while snippet mode is active and a next or previous stop exists (`inSnippetMode`, `hasNextTabstop`, `hasPrevTabstop`), so on the first stop Shift+Tab does not jump.

### AC-4.2
1. While a snippet is active, each of its tab stops is outlined and tinted so you can see where Tab will go.
2. The final `$0` stop is marked with its own background and border, different from the other stops.
3. All the marks disappear when snippet mode ends.
4. Typing at the edge of the current stop grows only that stop's mark, and the marks of the other stops keep their size.
5. The theme colors `editor.snippetTabstopHighlightBackground` and `editor.snippetTabstopHighlightBorder` set the look of normal stops, and `editor.snippetFinalTabstopHighlightBackground` and `editor.snippetFinalTabstopHighlightBorder` set the look of the final stop.

### AC-4.3
1. A placeholder such as `${1:name}` inserts its default text already selected, so typing replaces it.
2. A placeholder can contain other placeholders, and the outer one stays marked as active while an inner one is selected.
3. If you delete a placeholder's default text, the next Tab or Shift+Tab jump skips that stop.
4. A placeholder that has a transform gets its text rewritten when you move off it to another stop.

### AC-4.4
1. A choice placeholder such as `${1|one,two,three|}` opens a suggest-style list of its values as soon as that stop becomes active.
2. The values are listed in the order they are written in the snippet.
3. Picking a value inserts it and jumps straight to the next tab stop.
4. Pressing Shift+Tab while the choice list is open goes back to the previous tab stop instead of acting on the list (`inSnippetChoice`). [stock only]
5. In Cursor, Shift+Tab with the choice list open is handled by the suggest list's own keys instead of going back to the previous tab stop (inferred from the missing `inSnippetChoice` rule, not run). [cursor only]

### AC-4.5
1. A variable such as `$TM_FILENAME` or `${TM_FILENAME:default}` in a snippet is replaced with its value at the moment the snippet is inserted.
2. File and workspace variables insert the file name, the name without its extension, the folder, the full path, the path relative to the workspace, the workspace name and the workspace folder (`TM_FILENAME`, `TM_FILENAME_BASE`, `TM_DIRECTORY`, `TM_FILEPATH`, `RELATIVE_FILEPATH`, `WORKSPACE_NAME`, `WORKSPACE_FOLDER`).
3. Editor variables insert the selected text, the current line, the current word, the line index or number, the cursor index or number and the clipboard text (`TM_SELECTED_TEXT`, `SELECTION`, `TM_CURRENT_LINE`, `TM_CURRENT_WORD`, `TM_LINE_INDEX`, `TM_LINE_NUMBER`, `CURSOR_INDEX`, `CURSOR_NUMBER`, `CLIPBOARD`).
4. Date and time variables insert the current year, month, date, hour, minute, second, day name, month name, Unix seconds and time zone offset (`CURRENT_YEAR`, `CURRENT_YEAR_SHORT`, `CURRENT_MONTH`, `CURRENT_DATE`, `CURRENT_HOUR`, `CURRENT_MINUTE`, `CURRENT_SECOND`, `CURRENT_DAY_NAME`, `CURRENT_DAY_NAME_SHORT`, `CURRENT_MONTH_NAME`, `CURRENT_MONTH_NAME_SHORT`, `CURRENT_SECONDS_UNIX`, `CURRENT_TIMEZONE_OFFSET`).
5. Comment variables insert the comment markers of the language at the cursor (`LINE_COMMENT`, `BLOCK_COMMENT_START`, `BLOCK_COMMENT_END`).
6. Random variables insert a random number, a random hex string or a new UUID each time (`RANDOM`, `RANDOM_HEX`, `UUID`).
7. A transform on a variable, such as `${TM_FILENAME/(.*)\..+$/$1/}`, reshapes the value before it is inserted.
8. A name that is not a known variable and has no default, such as `${foo}`, is inserted as plain text and turned into a placeholder you can Tab to.
9. Cursor's list of known variable names lacks `CURRENT_MILLISECOND`, `CURRENT_MILLISECONDS_UNIX`, `CURRENT_TIMEZONE_NAME` and `TM_DIRECTORY_BASE`, which stock VS Code knows. [cursor only]

### AC-4.6
1. Inserting a snippet while another snippet is active places the new snippet's tab stops inside the current stop instead of ending the first snippet.
2. This happens when you pick a snippet from the suggest list or from the Insert Snippet picker while a placeholder is selected.
3. Pressing Tab walks through the inner snippet's stops first, then continues with the outer snippet's remaining stops.

### AC-4.7
1. Pressing Esc or Shift+Esc while snippet mode is active ends it on the spot and collapses the selection to the main cursor (`leaveSnippet`).
2. Snippet mode ends by itself when the cursor moves out of every tab stop.
3. Snippet mode ends by itself when undo goes back to before the snippet was inserted.
4. The command `acceptSnippet` runs through all the remaining stops and ends snippet mode at the end of the snippet, and it has no default key and no Command Palette entry.

### AC-4.8
1. With several cursors, inserting a snippet puts it at every cursor.
2. Linked tab stops are selected at all cursors together when you jump with Tab.
3. Every extra line of the snippet takes the indentation of the line it lands on, at each cursor.
4. The indentation of the inserted lines is converted to the spaces or tabs the file uses.

### AC-4.9
1. With `editor.tabCompletion` set to `onlySnippets`, typing a snippet prefix and pressing Tab inserts that snippet with no list.
2. When several snippets share the typed prefix, pressing Tab opens a small list of them to pick from.
3. Selected text that equals a snippet prefix also counts, so pressing Tab on that selection inserts the snippet.
4. Setting `editor.tabCompletion` chooses how Tab completes, with values `on`, `off` and `onlySnippets`, and the default is `off`.
5. When the word before the cursor matches no snippet prefix, or a snippet is already active, Tab does its usual job instead.

### AC-4.10
1. A snippet whose prefix matches what you type shows as a row in the suggest list with the snippet icon and the prefix as its label.
2. The row shows the snippet name as its description and "{description or name} ({source})" as its detail text.
3. The details pane shows a code preview of the snippet body for the focused snippet row.
4. A snippet matches from any word start or symbol in what you type, not only from the start of the line.
5. Rows that share the same prefix get ", {name}" added to the label so they can be told apart.
6. Your own snippets are sorted before snippets from extensions.
7. Setting `editor.snippetSuggestions` chooses where snippet rows sit in the list, with values `top`, `bottom`, `inline` and `none`, and the default is `inline`.
8. Setting `editor.suggest.showSnippets` turns snippet rows in the suggest list on or off.

### AC-4.11
1. Running Insert Snippet from the Command Palette (also reachable as `editor.action.showSnippets`) opens a picker with the placeholder "Select a snippet" that lists the snippets for the language at the cursor.
2. The rows are grouped under separators named "User Snippets", "Workspace Snippets" and the name of each extension that contributes snippets.
3. Each row shows the prefix as its label, or the snippet name when there is no prefix, with the description beneath it, or the body when there is no description.
4. Typing in the picker filters on both the label and the text beneath it.
5. Within a group, recently used snippets are listed first.
6. Picking a row inserts the snippet at the cursor and starts snippet mode.
7. When the language has no snippets, the picker shows the message "No snippet available".
8. A keybinding that passes `snippet` to `editor.action.insertSnippet` inserts that text as a snippet, and one that passes `name` inserts the snippet with that name, both without opening the picker.
9. A keybinding that passes `langId` to `editor.action.insertSnippet` opens the picker for that language instead of the language at the cursor.
10. Insert Snippet is unavailable in a read-only editor.

### AC-4.12
1. In the Insert Snippet and Surround with Snippet pickers, every snippet from an extension has an eye-closed button with the tooltip "Hide from IntelliSense".
2. Clicking that button removes the snippet from the suggest list, and the picker refreshes in place.
3. A hidden snippet stays in the picker with "(hidden from IntelliSense)" on its row, and its button becomes an eye with the tooltip "Show in IntelliSense".
4. Clicking the eye button brings the snippet back into the suggest list.
5. Rows for your own and workspace snippets have no such button.
6. The hidden or shown choice is kept per profile, so it is still there after a restart.

### AC-4.13
1. With text selected, running Surround with Snippet... from the Command Palette opens a picker with the placeholder "Select a snippet".
2. The picker lists only snippets whose body uses the selection (`$TM_SELECTED_TEXT` or `$SELECTION`).
3. Picking a snippet wraps the selected text with it and starts snippet mode.
4. When no snippet qualifies, the command does nothing and no picker opens.
5. The command is unavailable in a read-only editor and when nothing is selected.

### AC-4.14
1. Running Fill File with Snippet from the Command Palette opens a picker with the placeholder "Select a snippet" that lists the file-template snippets (marked `isFileTemplate`) that match the open file.
2. The rows are grouped under a separator for each language, and each row shows the prefix or name with the description beneath it.
3. Picking a snippet switches the file to that snippet's language and replaces the whole file with the snippet.
4. When no file-template snippet matches the open file, the command does nothing and no picker opens.

### AC-4.15
1. With text selected, the light bulb menu lists snippets that can wrap the selection, each row showing the snippet name.
2. At most four snippet rows show, and a "More..." row follows them when more than four match.
3. Choosing the "More..." row opens the Surround with Snippet... picker.
4. The rows sit under a "Surround With" group header when the menu shows group headers.
5. Choosing a snippet row wraps the selection with that snippet straight away, without opening a picker.
6. Setting `editor.snippets.codeActions.enabled` set to false removes these rows from the light bulb menu, and its default is true.

### AC-4.16
1. In an empty file, the light bulb offers a "Start with: {snippet name}" row for each file-template snippet of the file's language.
2. At most four such rows show, and a "Start with Snippet" row follows them when more match.
3. Choosing the "Start with Snippet" row opens the Fill File with Snippet picker.
4. The rows sit under the same "Surround With" group header as the surround rows.
5. Choosing a "Start with: ..." row fills the file with that snippet.
6. Setting `editor.snippets.codeActions.enabled` set to false removes these rows from the light bulb menu.
7. In a file that already has content, these rows never show.

### AC-4.17
1. Configure Snippets opens a picker with the placeholder "Select Snippets File or Create Snippets".
2. The picker opens from File > Preferences > Snippets, from the Snippets row in the Manage gear menu, and from Configure Snippets in the Command Palette.
3. The "Existing Snippets" group lists your own snippet files, with `.code-snippets` files first and the rest sorted by name.
4. Each existing row shows the file name with a description such as "(global)", up to four language names with "..." after them, or "(TypeScript)" for a language file.
5. The "New Snippets" group lists "New Global Snippets file...", one "New Snippets file for '{folder}'..." row for each workspace folder, and every language that has no file yet.
6. Picking an existing file opens it in an editor tab.
7. Typing in the picker filters on both the file name and its description.
8. When you have no snippet files yet, the picker shows only the "New Snippets" group.

### AC-4.18
1. Picking "New Global Snippets file..." asks for a file name in an input with the placeholder "Type snippet file name", and the new file goes in the user snippets folder.
2. Picking "New Snippets file for '{folder}'..." asks for a name the same way, and the new file goes in that folder's `.vscode` folder.
3. The name input rejects an empty name with "Invalid file name", an invalid name with "'{name}' is not a valid file name" and an existing name with "'{name}' already exists".
4. After a valid name, a `.code-snippets` file (the extension is added when missing) is created with a commented template that explains scope, prefix, body and description and shows an example, and it opens in an editor tab.
5. Picking a language that has no file yet writes `<language>.json` in the user snippets folder with a commented template for that language and opens it.
6. The template has no paragraph or example about restricting snippets with `include` and `exclude` patterns, where stock VS Code includes one. [cursor only]

### AC-4.19
1. Snippet files (`.code-snippets` files and the JSON files in the user snippets folder) open as normal editor tabs in the "Code Snippets" language, which is JSON with comments.
2. Typing in a snippet file offers property completions with descriptions from a schema (`prefix`, `body`, `description`, `isFileTemplate`, and `scope` in `.code-snippets` files).
3. A snippet entry that breaks the schema, such as one with no `body` or with an unknown property, shows an error squiggle.
4. The completion list includes an "Empty snippet" skeleton that inserts a whole snippet entry with tab stops for its name, prefix, body and description.
5. Setting `[snippets]` `editor.quickSuggestions` is `{ "strings": true }`, so suggestions also open while you type inside strings.
6. Setting `[snippets]` `editor.suggest.insertMode` is `replace`, so accepting a suggestion replaces the word to the right of the cursor.
7. Saving a snippet file reloads its snippets at once, so edits show in the suggest list and the pickers without a restart.
8. In Cursor, the snippet file schema offers `prefix`, `isFileTemplate`, `body` and `description` but not `include` or `exclude`. [cursor only]

### AC-4.20
1. A snippet in a language file such as `typescript.json` in the user snippets folder applies to that language only.
2. A snippet in a `.code-snippets` file applies to the languages named in its `scope` field, and to every language when `scope` is empty or missing.
3. A snippet in a workspace folder's `.vscode` folder shows only in that workspace, under "Workspace Snippets" in the pickers.
4. `prefix` can be one string or a list of strings, and each prefix in the list brings up the snippet.
5. A snippet with no `prefix` never appears in the suggest list and is reached only from the pickers and the light bulb.

### AC-4.21
1. A snippet with an `include` list of globs shows only for files that match at least one of the globs. [stock only]
2. A snippet with an `exclude` list of globs is hidden for files that match any of the globs, and exclude wins when a file matches both lists. [stock only]
3. A pattern that contains a slash is matched against the absolute path of the file. [stock only]
4. A pattern with no slash is matched against the file name only. [stock only]
5. Matching ignores letter case. [stock only]
6. Snippet files offer `include` and `exclude` as properties in completion, with their descriptions, and do not mark them as errors. [stock only]

## Unclear
- AC-4.10 pieces 7 and 8 name `editor.snippetSuggestions` and `editor.suggest.showSnippets`, which AC-1.12 also lists, so a counting tool should attach each setting piece to one owner.
- AC-4.4 piece 5 and the stock-only Shift+Tab piece 4 rest on an inference from the missing `inSnippetChoice` rule, not on a run of Cursor.
- AC-4.1 has no piece for Cursor Tab winning the Tab key over the tab stop jump, because the feature calls it inferred and not run, and AC-6 owns the Cursor Tab accept behavior.
- AC-4.17 has no piece for the Manage gear not being drawn in Cursor's default layout, because AB-4.18 owns that.
