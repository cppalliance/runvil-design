Group: QA-3 Go to a spot in this file. Pass: 3. Date: 2026-10-25.

### QA-3.1
1. Pressing `Ctrl+G` (`Control+G` on Mac) or typing `:` in the quick access box opens a one-row list that tells you what Enter will do.
2. Choosing Go to Line/Column... in the Go menu opens the same list.
3. Typing a line number and pressing Enter moves the cursor to that line.
4. Typing a line number followed by a column (`12:5`, `12,5` or `12#5`) and pressing Enter moves the cursor to that line and column.
5. The single row changes as you type, with prompts such as "Type a line number to go to (from 1 to {max})." and "Press 'Enter' to go to line {n} at column {c}." [stock only]
6. The column counts visible columns, so a tab counts as its tab width. [stock only]
7. With no text editor open, the row tells you to open a text editor first.
8. The list uses an older copy with rows such as "Go to line {0} and character {1}." and "Current Line: {0}, Character: {1}. Type a line number between 1 and {2} to navigate to.", the column is a plain character column, and the no-editor row reads "Open a text editor first to go to a line." [cursor only]

### QA-3.2
1. Typing a negative line number counts from the last line, so `-1` goes to the last line.
2. A negative column counts from the end of the line. [stock only]
3. A number that is too big is clipped to the nearest valid line or column, so pressing Enter always lands somewhere. [stock only]
4. A number out of range shows a hint row and pressing Enter does nothing. [cursor only]

### QA-3.3
1. Typing `::1024` in the quick access box and pressing Enter goes to the character position 1024 counted from the start of the file. [stock only]
2. Running Go to Offset... from the Command Palette opens the list ready for an offset. [stock only]
3. A negative number counts from the end of the file. [stock only]
4. The row says which line and column the offset lands on, or asks for a position "from 1 to {max}". [stock only]

### QA-3.4
1. A toggle button "Toggle Zero-Based Offset" shows at the right of the input only while the text starts with `::`. [stock only]
2. Clicking the button flips the count between starting at 1 and starting at 0, and the prompt changes between "from 1 to {max}" and "from 0 to {max-1}". [stock only]
3. The choice is remembered across windows. [stock only]

### QA-3.5
1. As the typed number changes, the editor scrolls that line to the middle with a smooth scroll.
2. The whole target line is painted with a range highlight and a mark in the overview ruler.
3. A typed number that gives no line clears the highlight.
4. Pressing Escape puts back the view you had before.
5. The highlight and the ruler mark take their colors from the theme colors `editor.rangeHighlightBackground` and `editorOverviewRuler.rangeHighlightForeground`.

### QA-3.6
1. While the Go to Line list is open, line numbers that were drawn relative switch to normal numbers, so the number you type matches what you see.
2. When the list closes, the editor goes back to relative line numbers.
3. Only `editor.lineNumbers` set to `relative` is changed while the list is open, and the values `on`, `off` and `interval` stay as they are.

### QA-3.7
1. Pressing Enter in the Go to Line or Go to Symbol list moves the cursor to the target and scrolls it to the middle of the editor with a smooth scroll.
2. Focus returns to the editor after the jump, so typing goes straight into the file.
3. The jump counts as a navigation jump, so Go Back returns to where you were before.
4. Screen readers are told "Line {0}, column {1}: {text}" after the jump. [stock only]
5. Screen readers hear only the text of the target line after the jump. [cursor only]

### QA-3.8
1. Pressing `Alt+Enter` in the Go to Line or Go to Symbol list opens the same file in the side group with the cursor at the target.
2. Pressing `Ctrl+Enter` does the same while `workbench.editor.enablePreviewFromQuickOpen` is off, which is the default.
3. Symbol rows have an "Open to the Side" button that does the same, and it reads "Open to the Bottom" when `workbench.editor.openSideBySideDirection` is `down`.
4. The view in the first editor is restored, so only the side editor moves.

### QA-3.9
1. Pressing Right Arrow on a row moves the editor to that line or symbol while the list stays open.
2. Focus stays in the list, so you can hop between spots.
3. Right Arrow acts this way only when focus is in the list or the text cursor is at the end of the input.

### QA-3.10
1. Pressing `Ctrl+Shift+O` (`Cmd+Shift+O` on Mac) or typing `@` lists the symbols of the open file.
2. Choosing Go to Symbol in Editor... in the Go menu opens the same list.
3. The list sits under a "symbols (N)" heading that shows how many symbols match.
4. Each row shows a kind icon, the symbol name and its container, with the matched letters highlighted.
5. Deprecated symbols are drawn struck through.
6. With an empty box, the symbol that contains the cursor starts as the active row.
7. In a diff editor the list uses the symbols of the modified side.
8. The box placeholder reads "Type the name of a symbol to go to."
9. `Ctrl+Shift+O` does nothing while the accessible view or accessibility help is shown.
10. A progress indicator shows on the input while the symbols are being collected.

### QA-3.11
1. Typing `@:` (a colon right after the `@`) groups the symbols by kind, with a heading and a count over each group, such as "methods (4)", "functions (2)" or "classes (1)".
2. The groups are ordered by the text of the kind heading.
3. Inside a group, rows are sorted by match score and then by position in the file.
4. Text typed after the colon filters the symbols as usual, and each group heading shows its filtered count.

### QA-3.12
1. With two search words, the first matches the symbol name and the rest match its container, so `run Server` finds `run` inside `Server`.
2. A symbol with no container is left out when the extra words do not also match its own name.
3. The container text in the grey description is highlighted where it matched.

### QA-3.13
1. Moving to a row scrolls its symbol to the middle of the editor with a smooth scroll.
2. The whole range of the symbol is highlighted, with a mark in the overview ruler.
3. The cursor and selection do not move until you accept a row.
4. Pressing Escape puts the old view back.
5. Up Arrow and Down Arrow change the active row, and the preview follows.

### QA-3.14
1. When the active editor is not a plain text editor but can make an outline (a notebook, for example), the list shows the outline entries with their own icons.
2. Moving through the rows previews the entry in the editor.
3. Pressing Enter on a row reveals that entry.
4. A filter with no result shows "No matching entries".
5. Setting `notebook.gotoSymbols.showAllSymbols` chooses whether notebooks list full code symbols as well as Markdown headers (on, the default) or only Markdown headers (off).

### QA-3.15
1. With no editor open, the list says "To go to a symbol, first open a text editor with symbol information."
2. With an editor that has no symbol provider, the list says "The active text editor does not provide symbol information."
3. When language support arrives late, that message is replaced by the real symbol list without reopening it.
4. A search with no hits says "No matching editor symbols".
5. A file with no symbols says "No editor symbols".

## Unclear

- QA-3.1: pieces 5 and 8 give the stock wording and the Cursor wording of the row text separately because the two builds use different prompts, so a build should be checked against Cursor for the exact strings.
- QA-3.4 and QA-3.3: the zero-based prompt text "from 0 to {max-1}" is taken from the stock source and was not confirmed on screen.
- QA-3.10: the feature text did not mention the progress indicator, so piece 10 comes straight from the source and was not checked in the Cursor bundle.
- Behavior on screen was not watched. Every piece comes from source or bundle strings.
