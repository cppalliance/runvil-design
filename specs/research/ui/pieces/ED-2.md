Group: ED-2 Editing text. Pass: 3. Date: 2026-10-25.

### ED-2.1
1. Pressing `Ctrl+Alt+Up` or `Ctrl+Alt+Down` with the caret in the text adds another caret on the line above or below at the same column, and typing then edits every caret at once (`Shift+Alt+Up` and `Shift+Alt+Down` on Linux).
2. The Selection menu items Add Cursor Above and Add Cursor Below add a caret on the line above or below, the same as the keys.
3. With several lines selected, pressing `Shift+Alt+I` or choosing Add Cursors to Line Ends from the Selection menu puts one caret at the end of each selected line.
4. Running Add Cursors to Bottom or Add Cursors to Top from the Command Palette adds a caret on every line from the current line down to the last line, or up to the first line, at the starting column.
5. Pressing `Escape` while there is more than one caret drops the extra carets and keeps one (`removeSecondaryCursors`).
6. Running Focus Next Cursor or Focus Previous Cursor from the Command Palette moves focus to the next or previous caret, and does nothing in a read-only editor.
7. After a caret is added with the keys, a screen reader announces "Cursor added" or "Cursors added" with the line and column of each new caret.
8. Setting `editor.multiCursorLimit` sets the most carets one editor can have, default 10000, from 1 to 100000.
9. Setting `editor.multiCursorMergeOverlapping` chooses whether carets that land on the same spot merge into one, on or off.

### ED-2.2
1. Alt+clicking in the text drops an extra caret at the click point without moving the existing ones.
2. Alt+double-clicking adds the word at the click as another selection, and Alt+triple-clicking adds the whole line as another selection (`lastCursorWordSelect`, `lastCursorLineSelect`).
3. Choosing Switch to Ctrl+Click for Multi-Cursor in the Selection menu makes Ctrl+click (Cmd+click on macOS) add carets instead of Alt+click, and the menu item then reads Switch to Alt+Click for Multi-Cursor (`workbench.action.toggleMultiCursorModifier`).
4. Setting `editor.multiCursorModifier` chooses which modifier adds carets, with values `alt` and `ctrlCmd`, default `alt`.

### ED-2.3
1. Pressing `Ctrl+D` with nothing selected selects the word under the caret, and pressing it again adds the next match as another selection.
2. The Selection menu items Add Next Occurrence and Add Previous Occurrence add the next or previous match of the selected text as another selection.
3. Pressing `Ctrl+K Ctrl+D` moves the last selection to the next match instead of adding one, and Move Last Selection to Previous Find Match in the Command Palette moves it backward.
4. While text is selected, other copies of the selected text light up in the editor, and setting `editor.selectionHighlight` turns that highlight on or off.
5. Settings `editor.selectionHighlightMultiline` and `editor.selectionHighlightMaxLength` choose whether a multi-line selection is highlighted and how long a selection can be and still be highlighted, but Cursor does not have them, so its highlight has no such options. [stock only]

### ED-2.4
1. Pressing `Ctrl+Shift+L` selects every occurrence of the current selection, or of the word under the caret when nothing is selected, as separate selections.
2. Choosing Select All Occurrences from the Selection menu does the same as `Ctrl+Shift+L`.
3. Typing after selecting all occurrences replaces every one of them with the typed text at once.
4. Pressing `Ctrl+F2` or choosing Change All Occurrences from the right-click menu selects every occurrence of the word under the caret so one edit changes all of them.

### ED-2.5
1. Holding `Ctrl+Shift+Alt` and pressing Left, Right, Up, Down, PageUp or PageDown grows a rectangular block selection from the caret (these keys are unbound on Linux).
2. Dragging with `Shift+Alt` held selects a rectangular block of text.
3. Dragging with the middle mouse button selects a rectangular block of text.
4. The Selection menu item Column Selection Mode shows a check mark while on, and while on plain mouse drags and Shift+arrow keys select a block (`editor.action.toggleColumnSelection`).
5. Setting `editor.columnSelection` turns column selection mode on or off, and the menu item flips the same setting.

### ED-2.6
1. Pressing `Ctrl+A` selects the whole file, and the Selection menu item Select All and the right-click menu of simple text inputs do the same.
2. Pressing `Ctrl+L` selects the whole current line, and pressing it again adds the next line to the selection (`expandLineSelection`).
3. Double-clicking a word selects it, and double-clicking next to a bracket or quote selects what is inside it.
4. Triple-clicking selects the whole line, and clicking four times selects all the text.
5. Clicking a line number in the gutter selects that whole line.
6. Setting `editor.wordSeparators` lists the characters that count as word breaks when a word is selected by double-click.

### ED-2.7
1. Pressing `Shift+Alt+Right` grows the selection outward one syntax level at a time, such as word, then expression, then block (`Ctrl+Cmd+Shift+Right` on macOS).
2. Pressing `Shift+Alt+Left` steps the selection back in to the previous smaller one (`Ctrl+Cmd+Shift+Left` on macOS).
3. The Selection menu items Expand Selection and Shrink Selection do the same as the keys.
4. Setting `editor.smartSelect.selectLeadingAndTrailingWhitespace` chooses whether growing the selection also takes in the whitespace around it, on or off.
5. Setting `editor.smartSelect.selectSubwords` chooses whether the first growth step can select part of a word, such as one piece of a camelCase name, on or off.

### ED-2.8
1. Pressing `Ctrl+K Ctrl+B` drops a selection anchor at the caret, shows a mark that reads "Selection Anchor" on hover, and a screen reader announces "Anchor set at" the line and column.
2. After moving the caret, pressing `Ctrl+K Ctrl+K` selects all the text from the anchor to the caret.
3. Running Go to Selection Anchor from the Command Palette moves the caret back to the anchor.
4. Pressing `Escape` while an anchor is set removes the anchor.

### ED-2.9
1. The arrow keys move the caret one character left or right and one line up or down.
2. `Ctrl+Left` and `Ctrl+Right` (`Alt` on macOS) move the caret one word at a time.
3. The word-part commands `cursorWordPartLeft` and `cursorWordPartRight` move the caret one part at a time inside names such as camelCase or snake_case, with no default key.
4. `Home` and `End` move the caret to the start and end of the line.
5. `Ctrl+Home` and `Ctrl+End` move the caret to the start and end of the file.
6. `PageUp` and `PageDown` move the caret up and down by one screen.
7. Holding `Shift` with any of these keys extends the selection instead of moving the caret, such as `cursorLeftSelect`.
8. Setting `editor.wordSeparators` chooses which characters stop a word-wise move.
9. Setting `editor.wordSegmenterLocales` chooses the languages that use language-aware word breaks, such as for Japanese or Chinese text.

### ED-2.10
1. Pressing `Ctrl+Shift+\` jumps the caret to the bracket that matches the one next to it.
2. The Go menu item Go to Bracket does the same as the key.
3. Running Select to Bracket from the Command Palette selects a bracket pair and everything between it.
4. Pressing `Ctrl+Alt+Backspace` inside a bracket pair deletes both brackets and keeps what was between them.
5. Setting `editor.matchBrackets` chooses when matching brackets are highlighted, with values `always`, `near` and `never`.

### ED-2.11
1. Pressing `Alt+Up` or `Alt+Down` moves the current line, or every selected line, up or down by one line.
2. Pressing `Shift+Alt+Up` or `Shift+Alt+Down` copies the current line, or every selected line, above or below itself (`Ctrl+Shift+Alt+Up` and `Ctrl+Shift+Alt+Down` on Linux).
3. The Selection menu has Move Line Up, Move Line Down, Copy Line Up, Copy Line Down and Duplicate Selection items.
4. Running Duplicate Selection inserts a copy of the selected text right after it and selects the copy, and with nothing selected it copies the current line downward, with no default key.

### ED-2.12
1. Pressing `Backspace` or `Delete` removes the character before or after the caret, or the selected text.
2. Pressing `Ctrl+Backspace` or `Ctrl+Delete` (`Alt` on macOS) removes the word to the left or right of the caret.
3. The commands `deleteWordPartLeft` and `deleteWordPartRight` remove one part of a name such as camelCase or snake_case to the left or right of the caret.
4. The command `deleteInsideWord` removes the whole word around the caret.
5. Pressing `Ctrl+Shift+K` removes the whole current line, or every line that has a caret or selection.
6. On macOS, `Cmd+Backspace` removes everything from the caret back to the start of the line and `Ctrl+K` removes everything from the caret to the end of the line (`deleteAllLeft`, `deleteAllRight`), and on other systems these two have no default key.

### ED-2.13
1. Pressing `Ctrl+Enter` opens a blank line below the current one and moves the caret onto it, even when the caret is in the middle of the line.
2. Pressing `Ctrl+Shift+Enter` opens a blank line above the current one and moves the caret onto it.
3. On macOS, pressing `Ctrl+O` splits the line at the caret and leaves the caret where it was (`lineBreakInsert`).
4. Running Join Lines turns the selected lines, or the current line and the next one, into a single line, with `Ctrl+J` on macOS and no default key elsewhere.

### ED-2.14
1. Running Sort Lines Ascending from the Command Palette sorts the selected lines from first to last in alphabetical order.
2. Running Sort Lines Descending sorts the selected lines in reverse alphabetical order.
3. Running Delete Duplicate Lines removes repeated lines from the selected lines and keeps the first copy of each.
4. When nothing is selected, or the selection sits on one line, the sort and delete-duplicates commands work on the whole file.
5. Reverse lines puts the selected lines in the opposite order, but Cursor does not have this command. [stock only]

### ED-2.15
1. Pressing `Ctrl+K Ctrl+X` removes the spaces and tabs at the end of every line.
2. Running Insert Final New Line from the Command Palette adds a newline at the end of the file when it is missing.
3. Setting `files.trimTrailingWhitespace` chooses whether trailing spaces and tabs are removed from every line on each save, on or off.
4. Setting `files.trimTrailingWhitespaceInRegexAndStrings` chooses whether that save-time trim also reaches whitespace inside regex literals and strings, on or off.
5. Setting `files.insertFinalNewline` chooses whether a final newline is added on each save, on or off.
6. Setting `files.trimFinalNewlines` chooses whether extra blank lines at the end of the file are removed on each save, on or off.

### ED-2.16
1. Pressing `Ctrl+/` comments out the current line, or every selected line, with the language's line comment, and removes the comment marks when the lines are already commented.
2. The Edit menu items Toggle Line Comment and Toggle Block Comment do the same as their keys.
3. Pressing `Ctrl+K Ctrl+C` always adds a line comment and pressing `Ctrl+K Ctrl+U` always removes one.
4. Pressing `Shift+Alt+A` wraps the selection in the language's block comment, and pressing it again removes the wrap (`Ctrl+Shift+A` on Linux).
5. Setting `editor.comments.insertSpace` chooses whether a space is put after the comment marker, on or off.
6. Setting `editor.comments.ignoreEmptyLines` chooses whether empty lines are skipped when toggling line comments, on or off.

### ED-2.17
1. Pressing `Tab` with nothing selected puts an indent at the caret, either a tab character or spaces.
2. Pressing `Tab` with several lines selected indents every selected line.
3. Pressing `Shift+Tab` takes one indent away from the current line or every selected line.
4. Pressing `Ctrl+]` or `Ctrl+[` indents or outdents the current line from any caret position in it (Indent Line and Outdent Line).
5. `Tab` and `Shift+Tab` only change indentation while the editor is not set to let the Tab key move focus out of it.

### ED-2.18
1. Running Indent Using Spaces or Indent Using Tabs from the Command Palette opens a list titled Select Tab Size for Current File, and picking a size sets the file to that style and width.
2. Running Change Tab Display Size opens the same kind of size list and changes how wide a tab character looks without editing the file.
3. Running Detect Indentation from Content looks at the file and sets tabs or spaces and the width to match what it finds.
4. Setting `editor.tabSize` sets how many spaces one tab is worth.
5. Setting `editor.indentSize` sets how many spaces one indent step is, either a number or the value `tabSize`.
6. Setting `editor.insertSpaces` chooses whether pressing Tab and auto indent insert spaces or tab characters, on or off.
7. Setting `editor.detectIndentation` chooses whether tab size and spaces or tabs are guessed from the file when it opens, on or off.

### ED-2.19
1. Running Convert Indentation to Spaces turns the leading tabs of every line in the file into spaces.
2. Running Convert Indentation to Tabs turns the leading spaces of every line in the file into tabs.
3. Running Reindent Lines resets the indentation of every line using the language's indent rules.
4. Running Reindent Selected Lines resets the indentation of only the selected lines using the language's indent rules.

### ED-2.20
1. Running Transform to Uppercase or Transform to Lowercase from the Command Palette changes the selected text to all capitals or all small letters.
2. Running Transform to Title Case capitalizes the first letter of each word in the selection.
3. Running Transform to Snake Case joins the words of the selection with underscores, and Transform to Kebab Case joins them with hyphens.
4. Running Transform to Camel Case joins the words with the first word in lower case, and Transform to Pascal Case joins them with every word capitalized.
5. With nothing selected, each case command changes the word under the caret.

### ED-2.21
1. Running Transpose Letters or Transpose Characters around the Cursor swaps the two letters around the caret, with `Ctrl+T` for Transpose Letters on macOS.
2. Running Move Selected Text Left or Move Selected Text Right slides the selected text one character left or right.
3. Pressing `Ctrl+Shift+,` replaces the selected value with the previous one in its set, such as false for true or a number one lower.
4. Pressing `Ctrl+Shift+.` replaces the selected value with the next one in its set, such as true for false or a number one higher.

### ED-2.22
1. Typing an opening bracket or quote puts the matching closing one after the caret, with the caret left between the pair.
2. Typing the closing bracket or quote when the same character is already next to the caret types over it instead of adding a second one.
3. Pressing `Backspace` between an auto-inserted pair removes both characters.
4. Selecting text and typing a bracket or quote wraps the selection in that pair instead of replacing it.
5. Settings `editor.autoClosingBrackets` and `editor.autoClosingQuotes` choose when a closing bracket or quote is added, with values `always`, `languageDefined`, `beforeWhitespace` and `never`.
6. Setting `editor.autoClosingDelete` chooses whether `Backspace` removes the closing character next to the caret, with values `always`, `auto` and `never`.
7. Setting `editor.autoClosingOvertype` chooses whether typing the closing character types over the existing one, with values `always`, `auto` and `never`.
8. Setting `editor.autoClosingComments` chooses when brackets and quotes close inside comments, with values `always`, `languageDefined`, `beforeWhitespace` and `never`.
9. Setting `editor.autoSurround` chooses what typing a bracket or quote over a selection wraps, with values `languageDefined`, `quotes`, `brackets` and `never`.

### ED-2.23
1. Pressing `Enter` starts the new line at the same indent as the line above, and adds one more indent after an opening brace.
2. Typing a closing bracket on an indented line pulls that line back one indent level.
3. Pasting lines shifts their indentation so they fit the place where they land.
4. Setting `editor.autoIndent` chooses how far auto indent goes, with values `none`, `keep`, `brackets`, `advanced` and `full`.
5. Setting `editor.autoIndentOnPaste` chooses whether pasted lines are re-indented, on or off.
6. Setting `editor.autoIndentOnPasteWithinString` chooses whether pasted lines are re-indented when they are pasted inside a string, but Cursor does not have it. [stock only]

### ED-2.24
1. With `editor.formatOnType` on, typing a trigger character such as a semicolon or closing brace reformats the current line with the file's formatter, and the setting is off by default.
2. With `editor.formatOnPaste` on, pasted text is reformatted by the file's formatter right after it lands, and the setting is off by default.
3. Neither format on type nor format on paste changes anything when the file's language has no formatter that can format a range.

### ED-2.25
1. With `editor.useTabStops` on, pressing `Tab` or `Backspace` in leading spaces moves the caret to the next or previous tab stop instead of one space.
2. With `editor.stickyTabStops` on, moving the caret or selection through leading spaces with the arrow keys jumps a whole indent at a time, like a tab character.
3. With `editor.trimAutoWhitespace` on, indentation that the editor added on a line left empty is removed when the caret moves away.
4. With `editor.trimWhitespaceOnDelete` on, deleting a line break also removes the leading whitespace of the next line, but Cursor does not have this setting. [stock only]

### ED-2.26
1. Pressing `Insert` (`Alt+Cmd+O` on macOS) or running Toggle Overtype/Insert Mode switches typing between inserting and overwriting, and while overwriting each typed character replaces the one after the caret.
2. Setting `editor.overtypeCursorStyle` chooses the caret shape while overwriting, with values `line`, `block`, `underline`, `line-thin`, `block-outline` and `underline-thin`, default `block`.
3. Setting `editor.overtypeOnPaste` chooses whether pasted text overwrites the text after the caret while in overtype mode, on or off.

### ED-2.27
1. Pressing `Ctrl+Shift+F2` with the caret on a paired item such as an HTML tag name starts linked editing, and renaming one half updates the other half as you type.
2. Pressing `Escape` while linked editing is active ends it, so the two halves stop updating together (`cancelLinkedEditingInput`).
3. Setting `editor.linkedEditing` chooses whether linked editing starts by itself when the caret is on a paired item, on or off.
4. Setting `editor.renameOnType` is the older name for the same choice and is shown as deprecated in favor of `editor.linkedEditing`.

### ED-2.28
1. Pressing `Ctrl+X`, `Ctrl+C` and `Ctrl+V` cuts, copies and pastes the selected text.
2. The Edit menu and the editor's right-click menu list Cut, Copy and Paste, and the Command Palette lists them too.
3. With nothing selected, cut and copy take the whole current line, and setting `editor.emptySelectionClipboard` turns that on or off.
4. On Windows, `Shift+Delete` cuts, `Ctrl+Insert` copies and `Shift+Insert` pastes, and on Linux `Shift+Insert` also pastes.
5. Pasting text that has as many lines as there are carets puts one line at each caret, and setting `editor.multiCursorPaste` chooses between `spread` for one line each and `full` for the whole text at every caret.
6. Choosing Copy with Syntax Highlighting from the Copy As submenu of the Edit menu or the right-click menu copies the selection together with its colors, with no default key.
7. Setting `editor.copyWithSyntaxHighlighting` chooses whether an ordinary copy also carries the syntax colors, on or off.
8. On Linux, selecting text also places it on the primary clipboard for middle-click paste, and setting `editor.selectionClipboard` turns that on or off.

### ED-2.29
1. After a paste that can land in more than one way, a small widget appears at the pasted text, and pressing `Ctrl+.` opens its list of choices such as plain text or a link, and picking one changes what was pasted.
2. Running Paste As... from the Command Palette opens a list of ways to paste the clipboard, and Paste as Text pastes it as plain text.
3. After dropping text or a file into the editor, a drop widget appears, and pressing `Ctrl+.` opens its list of ways the drop can land.
4. Dropping a file while holding `Shift` inserts it into the text instead of opening it as a tab.
5. Pressing `Escape` hides the paste or drop widget.
6. Settings `editor.pasteAs.enabled` and `editor.dropIntoEditor.enabled` turn the paste-as and drop-into-editor features on or off.
7. Settings `editor.pasteAs.showPasteSelector` and `editor.dropIntoEditor.showDropSelector` choose whether the widget appears, with values `afterPaste` or `afterDrop` and `never`.
8. Settings `editor.pasteAs.preferences` and `editor.dropIntoEditor.preferences` list the kinds of paste or drop that are tried first.

### ED-2.30
1. Dragging selected text with the mouse and releasing it elsewhere in the editor moves the text to that spot.
2. Holding `Ctrl` (`Alt` on macOS) while releasing the drag copies the text to the new spot instead of moving it.
3. Setting `editor.dragAndDrop` chooses whether selected text can be moved by dragging, on or off.

### ED-2.31
1. Pressing `Ctrl+Z` undoes the last edit.
2. Pressing `Ctrl+Y` or `Ctrl+Shift+Z` redoes the edit that was undone (`Cmd+Shift+Z` on macOS).
3. The Edit menu items Undo and Redo do the same, and the Command Palette and the right-click menu of simple text inputs list them too.
4. Pressing `Ctrl+U` steps the caret and selection back to where they were before the last cursor move without changing the text, and Cursor Redo steps forward again.

### ED-2.32
1. Typing an abbreviation such as `ul>li*3` and pressing `Tab` expands it into markup when `emmet.triggerExpansionOnTab` is on.
2. Running Emmet: Expand Abbreviation from the Edit menu or the Command Palette expands the abbreviation before the caret.
3. Selecting text and running Wrap with Abbreviation asks for an abbreviation and wraps the selection in what it expands to.
4. Running Show Emmet Commands lists the Emmet commands to pick from.
5. Setting `emmet.triggerExpansionOnTab` chooses whether `Tab` expands an abbreviation even when no completion shows, on or off.
6. Settings `emmet.includeLanguages` and `emmet.excludeLanguages` choose the languages where Emmet abbreviations expand.
7. Setting `emmet.syntaxProfiles` defines the output profile for a syntax.
8. Setting `emmet.variables` defines the variables that Emmet snippets can use.
9. Setting `emmet.preferences` changes how some Emmet actions and expansions behave.
10. Setting `emmet.extensionsPath` lists folders that hold Emmet profiles and snippet files.

### ED-2.33
1. Running Remove Tag from the Command Palette deletes the opening and closing tag around the caret and keeps what was between them.
2. Running Update Tag asks for a new tag name and renames the opening and closing tag together.
3. Running Go to Matching Pair jumps the caret between a tag's opening and closing halves.
4. Running Balance (outward) selects the tag or block that encloses the caret or selection, and Balance (inward) selects the part inside it.
5. Running Split/Join Tag turns a paired tag into a self-closing one, and a self-closing tag back into a pair.
6. Running Merge Lines joins the selected lines into one line.
7. Running Go to Previous Edit Point or Go to Next Edit Point moves the caret to the previous or next spot in the markup that is likely to need typing.
8. Running Select Previous Item or Select Next Item selects the previous or next item in the markup or stylesheet, such as a tag, attribute or property.
9. Running Toggle Comment comments out the markup or rule around the caret, or removes the comment when it is already commented.

### ED-2.34
1. Running Evaluate Math Expression replaces an expression such as `2*3+1` with its result.
2. Running Increment by 1 or Decrement by 1 raises or lowers the number at the caret by 1.
3. Running Increment by 0.1 or Decrement by 0.1 raises or lowers the number at the caret by one tenth.
4. Running Increment by 10 or Decrement by 10 raises or lowers the number at the caret by 10.
5. Running Update Image Size sets the size attributes of an image tag, or the size in CSS, to the real size of the image.
6. Running Reflect CSS Value copies the value of a vendor-prefixed CSS property across its prefixed variants.

### ED-2.35
1. Pressing `Alt+Z` turns word wrap on or off for the current editor, so long lines either fold at the edge or run off to the right.
2. The View menu item Word Wrap does the same as the key and shows a check mark while wrapping is on.
3. For a file made mostly of very long lines, such as minified code, an icon button in the editor title bar reads Enable wrapping for this file, and reads Disable wrapping for this file when wrapping is on for it.
4. Setting `editor.wordWrap` chooses how lines wrap, with values `off`, `on`, `wordWrapColumn` and `bounded`.
5. Setting `editor.wordWrapColumn` sets the column where lines wrap when `editor.wordWrap` is `wordWrapColumn` or `bounded`.
6. Setting `editor.wrappingIndent` chooses how wrapped lines are indented, with values `none`, `same`, `indent` and `deepIndent`.
7. Setting `editor.wrappingStrategy` chooses how wrap points are worked out, with values `simple` and `advanced`.
8. Setting `editor.wordBreak` chooses the word break rules for wrapping, with values `normal` and `keepAll`.

### ED-2.36
1. Running Toggle Render Whitespace shows dots for spaces and arrows for tabs, or hides them, by flipping `editor.renderWhitespace` between `none` and `all`.
2. In stock, the View > Appearance > Render Whitespace submenu lists None, Boundary, Selection, Trailing and All with a check mark on the current one, and picking an entry sets that mode. [stock only]
3. Setting `editor.renderWhitespace` chooses what is drawn, with values `none`, `boundary`, `selection`, `trailing` and `all`.
4. In Cursor, View > Appearance has one Render Whitespace item that flips between `none` and `all`, and there is no submenu for the other three modes. [cursor only]

### ED-2.37
1. Running Set Active Editor Read-only in Session locks the current editor so typing does nothing until it is unlocked or the session ends.
2. Running Set Active Editor Writeable in Session makes a read-only editor editable for the session.
3. Running Toggle Active Editor Read-only in Session flips the current editor between read-only and editable.
4. Running Reset Active Editor Read-only in Session puts the editor back to the state that the settings and the file's permissions give it.
5. Setting `files.readonlyInclude` lists the file patterns that always open read-only.
6. Setting `files.readonlyExclude` lists the file patterns that are left out of the read-only patterns.
7. Setting `files.readonlyFromPermissions` chooses whether files without write permission open read-only, on or off.
8. Trying to type in a read-only editor shows the message "Cannot edit in read-only editor", but Cursor was not found to show it. [stock only]

### ED-2.38
1. Holding `Ctrl+Alt+V` (on Windows also `Alt+Backquote`) starts dictation, and spoken words are typed into the text at the caret.
2. Releasing the key, or pressing `Escape`, stops dictation.
3. Running Start Dictation in Editor from the Command Palette starts dictation without holding a key, and Stop Dictation in Editor ends it.
4. While dictation runs, a filled microphone button labelled Stop Dictation, with its key, shows in the editor, and clicking it stops dictation.
5. Setting `accessibility.voice.speechLanguage` chooses the language that dictation listens for.
6. The dictation commands only work when a speech provider is installed.

### ED-2.40
1. Clicking in the text puts the caret at the click point.
2. Shift+clicking stretches the selection from the caret to the click point.
3. Dragging with the left mouse button selects the text between where the press began and where the pointer is.
4. Dragging past the top, bottom or side edge of the editor scrolls the text along while the selection keeps growing.
5. Dragging after a double click grows the selection by whole words, and dragging after a triple click grows it by whole lines.

### ED-2.41
1. Pressing `Escape` with text selected collapses the selection to one caret at the end the caret was on, and the text stays as it was.
2. Pressing `Shift+Escape` with text selected does the same.
3. With more than one caret, `Escape` drops the extra carets before it clears any selection.

### ED-2.42
1. While typing with the operating system's input method, the unfinished text shows in place at the caret.
2. The input method's candidate window opens at the caret.
3. Confirming the composition replaces the unfinished text with the final text.
4. Typing a dead key followed by a letter produces the accented letter.
5. On macOS, holding a letter shows the accent menu, and picking an accented letter replaces the typed one (`replacePreviousChar`).
6. Setting `editor.editContext` chooses whether the editor takes input through the browser's EditContext or through a hidden text area, on or off. [stock only]
7. In Cursor the same choice is the setting `editor.experimentalEditContextEnabled`, which is on by default only outside the stable build. [cursor only]

### ED-2.43
1. After closing a file's tab and opening the file again, Undo steps back through the edits made before it was closed, and Redo steps forward through any that were undone.
2. The history is restored only when the file's content is unchanged since it was closed.
3. A file whose undo history is over 20 MB, or that is too large to check, opens again with an empty history.
4. Setting `files.restoreUndoStack` chooses whether the undo history is restored when a file is reopened, on or off, default on.

### ED-2.44
1. Cursor Vim: Remove Vim Mode shows in the Command Palette for everyone, with no category prefix and no condition. [cursor only]
2. Running it uninstalls the Vim extension (vscodevim.vim) when it is installed, with no confirmation box.
3. The window reloads straight after the uninstall is requested, so the Vim mode indicator and Vim key handling are gone once it comes back.
4. When the Vim extension is not installed nothing is uninstalled, and the window still reloads.

## Unclear

- ED-2.21 piece 1 joins `editor.action.transposeLetters` and `editor.action.transpose` into one piece because the source does not show how the two differ on screen. If they behave differently, split it.
- ED-2.9 piece 4 only says Home and End go to the line edges. Whether the first Home press stops at the first non-blank character was not checked.
- ED-2.20 and ED-2.14 list their commands under the Command Palette only, since Menus is `none found` for both. Cursor's palette entries were not checked on screen.
- ED-2.36 piece 4 repeats the pass 2 finding that Cursor has one toggle item. Whether Cursor still lists Toggle Render Whitespace in the Command Palette was not checked.
- ED-2.39 repeats ED-1.28, as pass 2 noted. The parent should keep one copy of the pieces.
- ED-2.35 and ED-2.36 may also be listed by ED-5. The parent should keep one copy.
- ED-2.32 has exactly 10 pieces and covers 7 of the 12 Emmet settings. Four suggestion settings belong to AC-1, and `emmet.optimizeStylesheetParsing` is not named in any feature, so no piece covers it.
- Platform and Cursor key differences were taken from the feature blocks (V defaults). No piece was run on screen.
