Group: ED-2 Editing text. Section: Editor (ED). Date: 2026-10-25. Pass: 2.
Features: 43.

### ED-2.1 Add cursors above, below and at line ends
- Tag: both
- Evidence: source
- Boundary: no
- What: The person adds a cursor on the line above or below, or at the end of every selected line, so one typing action edits many lines, and presses Escape to drop back to one cursor.
- Commands: editor.action.insertCursorAbove; editor.action.insertCursorBelow; editor.action.insertCursorAtEndOfEachLineSelected; editor.action.addCursorsToBottom; editor.action.addCursorsToTop; editor.action.focusNextCursor; editor.action.focusPreviousCursor; removeSecondaryCursors
- Menus: MenubarSelectionMenu
- Keys: Ctrl+Alt+Up and Ctrl+Alt+Down (editorTextFocus; Linux Shift+Alt+Up and Shift+Alt+Down); Shift+Alt+I (cursors to line ends); Escape (removeSecondaryCursors, when there is more than one cursor)
- Settings: editor.multiCursorLimit; editor.multiCursorMergeOverlapping
- Cursor: present
- Source: V\src\vs\editor\contrib\multicursor\browser\multicursor.ts; V\src\vs\editor\browser\coreCommands.ts; C bundle (nls module vs/editor/contrib/multicursor/browser/multicursor, ids found as quoted strings)

### ED-2.2 Add a cursor with the mouse and pick the modifier key
- Tag: both
- Evidence: source
- Boundary: no
- What: The person Alt+clicks to drop extra cursors, and can switch the modifier to Ctrl+click (Cmd+click on macOS) through a Selection menu item so Alt+click is free for other uses.
- Commands: workbench.action.toggleMultiCursorModifier; createCursor; lastCursorWordSelect; lastCursorLineSelect
- Menus: MenubarSelectionMenu
- Keys: none found (the gesture is a mouse click)
- Settings: editor.multiCursorModifier
- Cursor: present
- Source: V\src\vs\workbench\contrib\codeEditor\browser\toggleMultiCursorModifier.ts; V\src\vs\editor\browser\view\viewController.ts; V\src\vs\editor\browser\coreCommands.ts; C bundle (nls module vs/workbench/contrib/codeEditor/browser/toggleMultiCursorModifier, menu labels "Switch to Alt+Click for Multi-Cursor" and "Switch to Cmd+Click for Multi-Cursor")

### ED-2.3 Select the next or previous occurrence
- Tag: both
- Evidence: source
- Boundary: no
- What: The person presses Ctrl+D to select the word under the cursor, then again to add the next match as another selection, and other copies of the selected text light up as they go.
- Commands: editor.action.addSelectionToNextFindMatch; editor.action.addSelectionToPreviousFindMatch; editor.action.moveSelectionToNextFindMatch; editor.action.moveSelectionToPreviousFindMatch
- Menus: MenubarSelectionMenu
- Keys: Ctrl+D (editor focus); Ctrl+K Ctrl+D (move last selection to next match)
- Settings: editor.selectionHighlight; editor.selectionHighlightMultiline; editor.selectionHighlightMaxLength
- Cursor: changed - the four commands and editor.selectionHighlight are present, but the options selectionHighlightMultiline and selectionHighlightMaxLength were not found
- Source: V\src\vs\editor\contrib\multicursor\browser\multicursor.ts (MultiCursorSelectionController, SelectionHighlighter); C bundle (nls module vs/editor/contrib/multicursor/browser/multicursor)

### ED-2.4 Select all occurrences and edit them together
- Tag: both
- Evidence: source
- Boundary: no
- What: The person selects every match of the current selection or word at once, or uses Change All Occurrences, and then types one edit that lands everywhere.
- Commands: editor.action.selectHighlights; editor.action.changeAll
- Menus: MenubarSelectionMenu; EditorContext
- Keys: Ctrl+Shift+L (select all occurrences); Ctrl+F2 (change all occurrences)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\multicursor\browser\multicursor.ts (SelectHighlightsAction, CompatChangeAll); C bundle (labels "Select All &&Occurrences" and "Change All Occurrences" in nls messages)

### ED-2.5 Column selection
- Tag: both
- Evidence: source
- Boundary: no
- What: The person selects a rectangular block of text by keyboard, by Shift+Alt dragging or by dragging with the middle mouse button, or turns on Column Selection Mode so plain drags select a block.
- Commands: columnSelect; cursorColumnSelectLeft; cursorColumnSelectRight; cursorColumnSelectUp; cursorColumnSelectDown; cursorColumnSelectPageUp; cursorColumnSelectPageDown; editor.action.toggleColumnSelection
- Menus: MenubarSelectionMenu
- Keys: Ctrl+Shift+Alt+Left, Right, Up, Down, PageUp, PageDown (textInputFocus; unbound on Linux)
- Settings: editor.columnSelection
- Cursor: present
- Source: V\src\vs\editor\browser\coreCommands.ts; V\src\vs\editor\browser\view\viewController.ts; V\src\vs\workbench\contrib\codeEditor\browser\toggleColumnSelection.ts; C bundle (nls module vs/workbench/contrib/codeEditor/browser/toggleColumnSelection, label "Toggle Column Selection Mode")

### ED-2.6 Select all, a word or a line
- Tag: both
- Evidence: source
- Boundary: no
- What: The person selects the whole file, grows the selection to the full current line, double-clicks a word (or next to a bracket or quote to select what is inside it), triple-clicks a line, clicks a line number to select that line, or clicks four times to select everything.
- Commands: editor.action.selectAll; expandLineSelection; _wordSelect; _lineSelect
- Menus: MenubarSelectionMenu; CommandPalette; SimpleEditorContext
- Keys: Ctrl+A (select all); Ctrl+L (expandLineSelection, textInputFocus)
- Settings: editor.wordSeparators; editor.doubleClickSelectsBlock
- Cursor: present
- Source: V\src\vs\editor\browser\editorExtensions.ts (SelectAllCommand); V\src\vs\editor\contrib\lineSelection\browser\lineSelection.ts; V\src\vs\editor\browser\coreCommands.ts; V\src\vs\editor\browser\view\viewController.ts; C bundle (nls module vs/editor/contrib/lineSelection/browser/lineSelection)

### ED-2.7 Expand or shrink the selection
- Tag: both
- Evidence: source
- Boundary: no
- What: The person grows the selection outward one syntax level at a time (word, expression, block) and steps back in with Shrink Selection.
- Commands: editor.action.smartSelect.expand; editor.action.smartSelect.shrink; editor.action.smartSelect.grow
- Menus: MenubarSelectionMenu
- Keys: Shift+Alt+Right (expand) and Shift+Alt+Left (shrink); on macOS Ctrl+Cmd+Shift+Right and Left
- Settings: editor.smartSelect.selectLeadingAndTrailingWhitespace; editor.smartSelect.selectSubwords; editor.smartSelect
- Cursor: present
- Source: V\src\vs\editor\contrib\smartSelect\browser\smartSelect.ts; C bundle (nls module vs/editor/contrib/smartSelect/browser/smartSelect, labels "&&Expand Selection" and "&&Shrink Selection")

### ED-2.8 Selection anchor
- Tag: both
- Evidence: source
- Boundary: no
- What: The person drops an anchor at one spot, scrolls or moves elsewhere, then selects everything from the anchor to the cursor, jumps back to the anchor, or cancels it.
- Commands: editor.action.setSelectionAnchor; editor.action.goToSelectionAnchor; editor.action.selectFromAnchorToCursor; editor.action.cancelSelectionAnchor
- Menus: none found
- Keys: Ctrl+K Ctrl+B (set anchor); Ctrl+K Ctrl+K (select from anchor to cursor); Escape (cancel anchor)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\anchorSelect\browser\anchorSelect.ts; C bundle (nls module vs/editor/contrib/anchorSelect/browser/anchorSelect)

### ED-2.9 Move the cursor with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: The person moves the caret by character, word, word part, line, page, line edge or file edge, and holds Shift to extend the selection with the same keys.
- Commands: cursorLeft; cursorRight; cursorUp; cursorDown; cursorHome; cursorEnd; cursorTop; cursorBottom; cursorPageUp; cursorPageDown; cursorLineStart; cursorLineEnd; cursorWordLeft; cursorWordRight; cursorWordPartLeft; cursorWordPartRight; cursorMove
- Menus: none found
- Keys: Arrow keys; Ctrl+Left and Ctrl+Right (by word; Alt on macOS); Home and End; Ctrl+Home and Ctrl+End; PageUp and PageDown; add Shift to extend (all when textInputFocus; every command has a Select twin such as cursorLeftSelect)
- Settings: editor.wordSeparators; editor.wordSegmenterLocales
- Cursor: present
- Source: V\src\vs\editor\browser\coreCommands.ts; V\src\vs\editor\contrib\wordOperations\browser\wordOperations.ts; V\src\vs\editor\contrib\wordPartOperations\browser\wordPartOperations.ts; C bundle (nls module vs/editor/contrib/wordOperations/browser/wordOperations, nls module vs/editor/browser/coreCommands, ids found as quoted strings)

### ED-2.10 Jump to, select or remove matching brackets
- Tag: both
- Evidence: source
- Boundary: no
- What: The person jumps the cursor to the matching bracket, selects everything inside and including a bracket pair, or deletes a bracket pair while keeping the contents.
- Commands: editor.action.jumpToBracket; editor.action.selectToBracket; editor.action.removeBrackets
- Menus: MenubarGoMenu
- Keys: Ctrl+Shift+\ (go to bracket); Ctrl+Alt+Backspace (remove brackets)
- Settings: editor.matchBrackets
- Cursor: present
- Source: V\src\vs\editor\contrib\bracketMatching\browser\bracketMatching.ts; C bundle (nls module vs/editor/contrib/bracketMatching/browser/bracketMatching, labels "Go to Bracket", "Select to Bracket", "Remove Brackets")

### ED-2.11 Move, copy or duplicate lines
- Tag: both
- Evidence: source
- Boundary: no
- What: The person moves the current line or selected lines up or down, copies them above or below, or duplicates the selection in place.
- Commands: editor.action.moveLinesUpAction; editor.action.moveLinesDownAction; editor.action.copyLinesUpAction; editor.action.copyLinesDownAction; editor.action.duplicateSelection
- Menus: MenubarSelectionMenu
- Keys: Alt+Up and Alt+Down (move line); Shift+Alt+Up and Shift+Alt+Down (copy line; Linux Ctrl+Shift+Alt+Up and Down); duplicateSelection has no default key
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\linesOperations\browser\linesOperations.ts; C bundle (nls module vs/editor/contrib/linesOperations/browser/linesOperations, labels "Move Line Up", "Copy Line Up", "Duplicate Selection")

### ED-2.12 Delete a character, word or line
- Tag: both
- Evidence: source
- Boundary: no
- What: The person deletes one character with Backspace or Delete, a whole word to the left or right, the whole current line, or everything from the cursor to the start or end of the line.
- Commands: deleteLeft; deleteRight; deleteWordLeft; deleteWordRight; deleteWordPartLeft; deleteWordPartRight; deleteInsideWord; editor.action.deleteLines; deleteAllLeft; deleteAllRight
- Menus: none found
- Keys: Backspace and Delete (textInputFocus); Ctrl+Backspace and Ctrl+Delete (delete word; Alt on macOS); Ctrl+Shift+K (delete line); macOS Cmd+Backspace (delete all left) and Ctrl+K (delete all right)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\browser\coreCommands.ts; V\src\vs\editor\contrib\wordOperations\browser\wordOperations.ts; V\src\vs\editor\contrib\wordPartOperations\browser\wordPartOperations.ts; V\src\vs\editor\contrib\linesOperations\browser\linesOperations.ts; C bundle (nls modules vs/editor/contrib/linesOperations/browser/linesOperations and wordOperations, ids found as quoted strings)

### ED-2.13 Insert, split or join lines
- Tag: both
- Evidence: source
- Boundary: no
- What: The person opens a blank line above or below the current one without moving the caret to the line end, splits a line and keeps the caret in place, or joins the selected lines into one.
- Commands: editor.action.insertLineBefore; editor.action.insertLineAfter; lineBreakInsert; editor.action.joinLines
- Menus: none found
- Keys: Ctrl+Shift+Enter (insert line above); Ctrl+Enter (insert line below); macOS Ctrl+J (join lines); macOS Ctrl+O (lineBreakInsert)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\linesOperations\browser\linesOperations.ts; V\src\vs\editor\browser\coreCommands.ts; C bundle (nls module vs/editor/contrib/linesOperations/browser/linesOperations, label "Join Lines")

### ED-2.14 Sort, reverse or de-duplicate lines
- Tag: both
- Evidence: source
- Boundary: no
- What: The person sorts the selected lines ascending or descending, reverses their order, or deletes duplicate lines.
- Commands: editor.action.sortLinesAscending; editor.action.sortLinesDescending; editor.action.reverseLines; editor.action.removeDuplicateLines
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - Reverse lines (editor.action.reverseLines) not found in the bundle, the other three are present
- Source: V\src\vs\editor\contrib\linesOperations\browser\linesOperations.ts; C bundle (nls module vs/editor/contrib/linesOperations/browser/linesOperations, label "Delete Duplicate Lines")

### ED-2.15 Trim whitespace and fix the final newline
- Tag: both
- Evidence: source
- Boundary: no
- What: The person trims trailing spaces from every line or adds a final newline on demand, and can have trailing whitespace, a final newline and extra blank lines at the end fixed automatically on every save.
- Commands: editor.action.trimTrailingWhitespace; editor.action.insertFinalNewLine
- Menus: none found
- Keys: Ctrl+K Ctrl+X (trim trailing whitespace)
- Settings: files.trimTrailingWhitespace; files.trimTrailingWhitespaceInRegexAndStrings; files.insertFinalNewline; files.trimFinalNewlines
- Cursor: present
- Source: V\src\vs\editor\contrib\linesOperations\browser\linesOperations.ts; V\src\vs\editor\contrib\insertFinalNewLine\browser\insertFinalNewLine.ts; V\src\vs\workbench\contrib\codeEditor\browser\saveParticipants.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (nls modules vs/editor/contrib/insertFinalNewLine/browser/insertFinalNewLine and vs/workbench/contrib/codeEditor/browser/saveParticipants, setting names found)

### ED-2.16 Toggle line and block comments
- Tag: both
- Evidence: source
- Boundary: no
- What: The person comments or uncomments the current or selected lines with the language's line comment, or wraps the selection in the language's block comment.
- Commands: editor.action.commentLine; editor.action.addCommentLine; editor.action.removeCommentLine; editor.action.blockComment
- Menus: MenubarEditMenu
- Keys: Ctrl+/ (toggle line comment); Ctrl+K Ctrl+C (add line comment); Ctrl+K Ctrl+U (remove line comment); Shift+Alt+A (toggle block comment; Linux Ctrl+Shift+A)
- Settings: editor.comments.insertSpace; editor.comments.ignoreEmptyLines; editor.comments
- Cursor: present
- Source: V\src\vs\editor\contrib\comment\browser\comment.ts; C bundle (nls module vs/editor/contrib/comment/browser/comment, labels "Toggle Line Comment" and "Toggle &&Block Comment")

### ED-2.17 Indent and outdent lines
- Tag: both
- Evidence: source
- Boundary: no
- What: The person presses Tab to insert an indent or indent the selected lines, Shift+Tab to take one indent away, or uses the Indent Line and Outdent Line commands from anywhere on the line.
- Commands: tab; outdent; editor.action.indentLines; editor.action.outdentLines
- Menus: none found
- Keys: Tab and Shift+Tab (editorTextFocus and tabDoesNotMoveFocus); Ctrl+] (indent line); Ctrl+[ (outdent line)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\browser\coreCommands.ts; V\src\vs\editor\contrib\linesOperations\browser\linesOperations.ts; C bundle (nls module vs/editor/contrib/linesOperations/browser/linesOperations, ids tab and outdent found as quoted strings)

### ED-2.18 Set or detect the indent style and size
- Tag: both
- Evidence: source
- Boundary: no
- What: The person chooses tabs or spaces and the indent width for the current file, changes how wide a tab looks, or asks the editor to detect the style from the file's content.
- Commands: editor.action.indentUsingTabs; editor.action.indentUsingSpaces; editor.action.changeTabDisplaySize; editor.action.detectIndentation
- Menus: none found
- Keys: none found
- Settings: editor.tabSize; editor.indentSize; editor.insertSpaces; editor.detectIndentation
- Cursor: present
- Source: V\src\vs\editor\contrib\indentation\browser\indentation.ts; V\src\vs\editor\common\config\editorConfigurationSchema.ts; C bundle (nls module vs/editor/contrib/indentation/browser/indentation, label "Detect Indentation from Content", settings editor.indentSize, editor.insertSpaces, editor.detectIndentation, editor.trimAutoWhitespace found in the schema)

### ED-2.19 Convert or reindent existing lines
- Tag: both
- Evidence: source
- Boundary: no
- What: The person converts a file's leading tabs to spaces or spaces to tabs, or reindents all lines or the selected lines using the language's indent rules.
- Commands: editor.action.indentationToSpaces; editor.action.indentationToTabs; editor.action.reindentlines; editor.action.reindentselectedlines
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\indentation\browser\indentation.ts; C bundle (nls module vs/editor/contrib/indentation/browser/indentation, labels "Convert Indentation to Spaces", "Convert Indentation to Tabs", "Reindent Lines")

### ED-2.20 Change case
- Tag: both
- Evidence: source
- Boundary: no
- What: The person turns the selected text into uppercase, lowercase, title case, snake case, camel case, pascal case or kebab case.
- Commands: editor.action.transformToUppercase; editor.action.transformToLowercase; editor.action.transformToTitlecase; editor.action.transformToSnakecase; editor.action.transformToCamelcase; editor.action.transformToPascalcase; editor.action.transformToKebabcase
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\linesOperations\browser\linesOperations.ts; C bundle (nls module vs/editor/contrib/linesOperations/browser/linesOperations, label "Transform to Kebab Case")

### ED-2.21 Transpose, nudge or step text in place
- Tag: both
- Evidence: source
- Boundary: no
- What: The person swaps the two letters around the cursor, slides the selected text one character left or right, or replaces the selected value with the previous or next one in its set (such as true and false or the next number).
- Commands: editor.action.transposeLetters; editor.action.transpose; editor.action.moveCarretLeftAction; editor.action.moveCarretRightAction; editor.action.inPlaceReplace.up; editor.action.inPlaceReplace.down
- Menus: none found
- Keys: Ctrl+T (macOS, transposeLetters); Ctrl+Shift+, (replace with previous value); Ctrl+Shift+. (replace with next value)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\caretOperations\browser\transpose.ts; V\src\vs\editor\contrib\caretOperations\browser\caretOperations.ts; V\src\vs\editor\contrib\linesOperations\browser\linesOperations.ts (TransposeAction); V\src\vs\editor\contrib\inPlaceReplace\browser\inPlaceReplace.ts; C bundle (nls modules vs/editor/contrib/caretOperations/browser/transpose, caretOperations and vs/editor/contrib/inPlaceReplace/browser/inPlaceReplace)
### ED-2.22 Auto close and auto surround brackets and quotes
- Tag: both
- Evidence: source
- Boundary: no
- What: When the person types an opening bracket or quote the editor adds the closing one, types over the closing one when it is typed, deletes the pair on Backspace, and wraps the selection in a pair when a bracket or quote is typed over selected text.
- Commands: type; deleteLeft
- Menus: none found
- Keys: none found
- Settings: editor.autoClosingBrackets; editor.autoClosingQuotes; editor.autoClosingDelete; editor.autoClosingOvertype; editor.autoClosingComments; editor.autoSurround
- Cursor: present
- Source: V\src\vs\editor\common\cursor\cursorTypeEditOperations.ts; V\src\vs\editor\common\cursor\cursorDeleteOperations.ts; V\src\vs\editor\common\commands\surroundSelectionCommand.ts; V\src\vs\editor\common\config\editorOptions.ts; C bundle (editor option names autoClosingBrackets, autoClosingQuotes, autoClosingDelete, autoClosingOvertype, autoClosingComments, autoSurround found in the options table)

### ED-2.23 Auto indent
- Tag: both
- Evidence: source
- Boundary: no
- What: When the person presses Enter, types a closing bracket or pastes lines, the editor keeps or adjusts the indentation from the language's rules, such as indenting after an opening brace and outdenting on a closing one.
- Commands: type
- Menus: none found
- Keys: none found
- Settings: editor.autoIndent; editor.autoIndentOnPaste; editor.autoIndentOnPasteWithinString
- Cursor: changed - autoIndent and autoIndentOnPaste are present, autoIndentOnPasteWithinString was not found
- Source: V\src\vs\editor\common\cursor\cursorTypeEditOperations.ts (EnterOperation); V\src\vs\editor\common\languages\autoIndent.ts; V\src\vs\editor\common\languages\enterAction.ts; V\src\vs\editor\contrib\indentation\browser\indentation.ts (AutoIndentOnPaste); C bundle (nls module vs/editor/contrib/indentation/browser/indentation, contribution id editor.contrib.autoIndentOnPaste)

### ED-2.24 Format on type and on paste
- Tag: both
- Evidence: source
- Boundary: no
- What: With the settings on, the editor reformats the line when a trigger character such as a semicolon or closing brace is typed, and reformats text right after it is pasted, using the file's formatter.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.formatOnType; editor.formatOnPaste
- Cursor: present
- Source: V\src\vs\editor\contrib\format\browser\formatActions.ts (FormatOnType, FormatOnPaste); V\src\vs\editor\common\config\editorOptions.ts; C bundle (nls module vs/editor/contrib/format/browser/formatActions, option names formatOnType and formatOnPaste found)

### ED-2.25 Tab stops and automatic whitespace cleanup
- Tag: both
- Evidence: source
- Boundary: no
- What: Backspace and typing treat space indents like tab stops, the selection can stick to tab stops over leading spaces, auto-inserted indentation is removed from lines left empty, and deleting a line break can also remove the next line's indent.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.useTabStops; editor.stickyTabStops; editor.trimAutoWhitespace; editor.trimWhitespaceOnDelete
- Cursor: changed - useTabStops, stickyTabStops and trimAutoWhitespace are present, trimWhitespaceOnDelete was not found
- Source: V\src\vs\editor\common\config\editorOptions.ts; V\src\vs\editor\common\config\editorConfigurationSchema.ts; V\src\vs\editor\common\cursor\cursorDeleteOperations.ts; C bundle (option names useTabStops, stickyTabStops and editor.trimAutoWhitespace found)

### ED-2.26 Overtype mode
- Tag: both
- Evidence: source
- Boundary: no
- What: The person presses Insert to switch typing between inserting and overwriting characters, with a different cursor shape while overwriting and a setting for whether paste overwrites.
- Commands: editor.action.toggleOvertypeInsertMode
- Menus: none found
- Keys: Insert; macOS Alt+Cmd+O
- Settings: editor.overtypeOnPaste; editor.overtypeCursorStyle
- Cursor: present
- Source: V\src\vs\workbench\contrib\codeEditor\browser\toggleOvertype.ts; V\src\vs\editor\common\config\editorOptions.ts; C bundle (nls module vs/workbench/contrib/codeEditor/browser/toggleOvertype, label "Toggle Overtype/Insert Mode")

### ED-2.27 Linked editing
- Tag: both
- Evidence: source
- Boundary: no
- What: The person starts linked editing on a paired item such as an HTML opening tag, and renaming one half updates the other half as they type, or turns on the setting so it starts by itself.
- Commands: editor.action.linkedEditing; cancelLinkedEditingInput
- Menus: none found
- Keys: Ctrl+Shift+F2 (start linked editing); Escape (cancel, while the linked input is active)
- Settings: editor.linkedEditing; editor.renameOnType
- Cursor: present
- Source: V\src\vs\editor\contrib\linkedEditing\browser\linkedEditing.ts; C bundle (nls module vs/editor/contrib/linkedEditing/browser/linkedEditing, label "Start Linked Editing")

### ED-2.28 Cut, copy and paste
- Tag: both
- Evidence: source
- Boundary: no
- What: The person cuts, copies and pastes through keys, the Edit menu or the right-click menu, where nothing selected means the whole line, pasting one line per cursor fills several cursors, and Copy with Syntax Highlighting copies colored text.
- Commands: editor.action.clipboardCutAction; editor.action.clipboardCopyAction; editor.action.clipboardPasteAction; editor.action.clipboardCopyWithSyntaxHighlightingAction; cut; paste; editor.action.selectionClipboardPaste
- Menus: MenubarEditMenu; EditorContext; CommandPalette; SimpleEditorContext; MenubarCopy; EditorContextCopy
- Keys: Ctrl+X, Ctrl+C, Ctrl+V; Windows also Shift+Delete (cut), Ctrl+Insert (copy), Shift+Insert (paste); Linux also Shift+Insert (paste); Copy with Syntax Highlighting has no default key
- Settings: editor.emptySelectionClipboard; editor.copyWithSyntaxHighlighting; editor.multiCursorPaste; editor.selectionClipboard
- Cursor: present
- Source: V\src\vs\editor\contrib\clipboard\browser\clipboard.ts; V\src\vs\editor\browser\coreCommands.ts; V\src\vs\workbench\contrib\codeEditor\browser\selectionClipboard.ts (Linux selection clipboard); C bundle (nls modules vs/editor/contrib/clipboard/browser/clipboard and vs/workbench/contrib/codeEditor/electron-sandbox/selectionClipboard, label "Copy with Syntax Highlighting")

### ED-2.29 Paste As and drop into the editor
- Tag: both
- Evidence: source
- Boundary: no
- What: After a paste or a drop the person chooses how it lands, such as plain text or a link, from a small widget opened with Ctrl+. or from Paste As and Paste as Text, and Shift+dropping a file inserts it as text instead of opening it.
- Commands: editor.action.pasteAs; editor.action.pasteAsText; editor.changePasteType; editor.hidePasteWidget; editor.changeDropType; editor.hideDropWidget; workbench.action.configurePreferredDropAction; workbench.action.configurePreferredPasteAction; editor.pasteAs.default
- Menus: none found
- Keys: Ctrl+. (change paste type, when the paste widget shows; change drop type, when the drop widget shows); Escape (hide the widget)
- Settings: editor.pasteAs.enabled; editor.pasteAs.showPasteSelector; editor.pasteAs.preferences; editor.dropIntoEditor.enabled; editor.dropIntoEditor.showDropSelector; editor.dropIntoEditor.preferences; editor.dropIntoEditor; editor.pasteAs
- Cursor: present
- Source: V\src\vs\editor\contrib\dropOrPasteInto\browser\copyPasteContribution.ts; V\src\vs\editor\contrib\dropOrPasteInto\browser\copyPasteController.ts; V\src\vs\editor\contrib\dropOrPasteInto\browser\dropIntoEditorContribution.ts; V\src\vs\editor\contrib\dropOrPasteInto\browser\dropIntoEditorController.ts; V\src\vs\editor\contrib\dropOrPasteInto\browser\postEditWidget.ts; C bundle (nls modules vs/editor/contrib/dropOrPasteInto/browser/copyPasteContribution, dropIntoEditorController, postEditWidget; labels "Paste As..." and "Paste as Text")

### ED-2.30 Drag and drop selected text
- Tag: both
- Evidence: source
- Boundary: no
- What: The person drags selected text with the mouse to move it to a new spot in the editor, or holds Ctrl (Alt on macOS) while dropping to copy it.
- Commands: none found
- Menus: none found
- Keys: none found (the gesture is a mouse drag; Ctrl or Alt held means copy)
- Settings: editor.dragAndDrop
- Cursor: present
- Source: V\src\vs\editor\contrib\dnd\browser\dnd.ts; V\src\vs\editor\contrib\dnd\browser\dragAndDropCommand.ts; C bundle (contribution id editor.contrib.dragAndDrop and option name dragAndDrop found)

### ED-2.31 Undo and redo
- Tag: both
- Evidence: source
- Boundary: no
- What: The person undoes and redoes edits from the Edit menu or keys, and can separately undo or redo cursor moves without touching the text.
- Commands: undo; redo; cursorUndo; cursorRedo
- Menus: MenubarEditMenu; CommandPalette; SimpleEditorContext
- Keys: Ctrl+Z (undo); Ctrl+Y and Ctrl+Shift+Z (redo; macOS Cmd+Shift+Z); Ctrl+U (cursor undo)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\browser\editorExtensions.ts (UndoCommand, RedoCommand); V\src\vs\editor\contrib\cursorUndo\browser\cursorUndo.ts; C bundle (nls module vs/editor/contrib/cursorUndo/browser/cursorUndo, labels "&&Undo", "&&Redo", "Cursor Undo", "Cursor Redo")
### ED-2.32 Expand or wrap with an Emmet abbreviation
- Tag: both
- Evidence: source
- Boundary: no
- What: The person types an abbreviation such as ul>li*3 and presses Tab, or runs Emmet: Expand Abbreviation, to expand it into markup, or selects text and wraps it with an abbreviation.
- Commands: editor.emmet.action.expandAbbreviation; editor.emmet.action.wrapWithAbbreviation; workbench.action.showEmmetCommands
- Menus: MenubarEditMenu; commandPalette
- Keys: Tab (when config.emmet.triggerExpansionOnTab is on, with editorTextFocus and tabDoesNotMoveFocus)
- Settings: emmet.triggerExpansionOnTab; emmet.includeLanguages; emmet.excludeLanguages; emmet.syntaxProfiles; emmet.variables; emmet.preferences; emmet.extensionsPath
- Cursor: present
- Source: V\src\vs\workbench\contrib\emmet\browser\actions\expandAbbreviation.ts; V\src\vs\workbench\contrib\emmet\browser\emmetActions.ts; V\extensions\emmet\package.json; C\extensions\emmet\package.json (same 23 commands and 12 settings as stock); C bundle (label "Emmet: E&&xpand Abbreviation", id editor.emmet.action.expandAbbreviation)

### ED-2.33 Emmet tag and edit-point commands
- Tag: both
- Evidence: source
- Boundary: no
- What: The person removes, updates, matches, balances, splits or joins an HTML or XML tag, merges lines, jumps between edit points, selects the previous or next item, and toggles a comment, all from Emmet commands.
- Commands: editor.emmet.action.removeTag; editor.emmet.action.updateTag; editor.emmet.action.matchTag; editor.emmet.action.balanceIn; editor.emmet.action.balanceOut; editor.emmet.action.splitJoinTag; editor.emmet.action.mergeLines; editor.emmet.action.prevEditPoint; editor.emmet.action.nextEditPoint; editor.emmet.action.selectPrevItem; editor.emmet.action.selectNextItem; editor.emmet.action.toggleComment
- Menus: commandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\emmet\package.json; V\extensions\emmet\package.nls.json; C\extensions\emmet\package.json (same commands as stock)

### ED-2.34 Emmet number and CSS helpers
- Tag: both
- Evidence: source
- Boundary: no
- What: The person evaluates a math expression in place, bumps a number up or down by 0.1, 1 or 10, updates an image's size attributes, or reflects a CSS value across vendor-prefixed properties.
- Commands: editor.emmet.action.evaluateMathExpression; editor.emmet.action.incrementNumberByOneTenth; editor.emmet.action.incrementNumberByOne; editor.emmet.action.incrementNumberByTen; editor.emmet.action.decrementNumberByOneTenth; editor.emmet.action.decrementNumberByOne; editor.emmet.action.decrementNumberByTen; editor.emmet.action.updateImageSize; editor.emmet.action.reflectCSSValue
- Menus: commandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\emmet\package.json; V\extensions\emmet\package.nls.json; C\extensions\emmet\package.json (same commands as stock)

### ED-2.35 Word wrap
- Tag: both
- Evidence: source
- Boundary: no
- What: The person turns word wrap on or off for the current editor with Alt+Z, the View menu or an editor title action, and where and how lines wrap follows the wrap settings.
- Commands: editor.action.toggleWordWrap
- Menus: MenubarViewMenu; EditorTitle
- Keys: Alt+Z
- Settings: editor.wordWrap; editor.wordWrapColumn; editor.wrappingIndent; editor.wrappingStrategy; editor.wordBreak
- Cursor: present
- Source: V\src\vs\workbench\contrib\codeEditor\browser\toggleWordWrap.ts; V\src\vs\editor\common\config\editorOptions.ts; C bundle (nls module vs/workbench/contrib/codeEditor/browser/toggleWordWrap, id and Alt+Z binding, labels "&&Word Wrap", "Disable wrapping for this file", "Enable wrapping for this file")

### ED-2.36 Render whitespace
- Tag: both
- Evidence: source
- Boundary: no
- What: The person shows or hides dots and arrows for spaces and tabs, and in stock picks None, Boundary, Selection, Trailing or All from a View > Appearance submenu.
- Commands: editor.action.toggleRenderWhitespace; editor.action.renderWhitespace.none; editor.action.renderWhitespace.boundary; editor.action.renderWhitespace.selection; editor.action.renderWhitespace.trailing; editor.action.renderWhitespace.all
- Menus: MenubarAppearanceMenu; EditorRenderWhitespaceSubmenu
- Keys: none found
- Settings: editor.renderWhitespace
- Cursor: changed - Cursor has one Toggle Render Whitespace item in View > Appearance that flips between none and all; the five mode commands and the submenu were not found
- Source: V\src\vs\workbench\contrib\codeEditor\browser\toggleRenderWhitespace.ts; C bundle (nls module vs/workbench/contrib/codeEditor/browser/toggleRenderWhitespace, menu label "&&Render Whitespace", toggle code writes editor.renderWhitespace as none or all)

### ED-2.37 Read-only for this session
- Tag: both
- Evidence: source
- Boundary: no
- What: The person flips the active editor between read-only and editable for the current session, or resets it to the configured state, and a message explains the lock when they try to type in a read-only editor.
- Commands: workbench.action.files.setActiveEditorReadonlyInSession; workbench.action.files.setActiveEditorWriteableInSession; workbench.action.files.toggleActiveEditorReadonlyInSession; workbench.action.files.resetActiveEditorReadonlyInSession
- Menus: none found
- Keys: none found
- Settings: files.readonlyInclude; files.readonlyExclude; files.readonlyFromPermissions; editor.readOnly; editor.readOnlyMessage; editor.simple
- Cursor: changed - the four commands and the three settings are present, the read-only message controller (editor.contrib.readOnlyMessageController, text "Cannot edit in read-only editor") was not found
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\editor\contrib\readOnlyMessage\browser\contribution.ts; V\src\vs\workbench\services\filesConfiguration\common\filesConfigurationService.ts; C bundle (nls module vs/workbench/contrib/files/browser/fileActions, label "Set Active Editor Read-only in Session")

### ED-2.38 Dictate into the editor
- Tag: both
- Evidence: source
- Boundary: no
- What: With a speech provider installed, the person holds Ctrl+Alt+V to speak text into the editor at the cursor and stops with Escape or by releasing the key.
- Commands: workbench.action.editorDictation.start; workbench.action.editorDictation.stop
- Menus: none found
- Keys: Ctrl+Alt+V (Windows also Alt+Backquote; held key mode); Escape (stop, while dictation is running)
- Settings: accessibility.voice.speechLanguage
- Cursor: present
- Source: V\src\vs\workbench\contrib\codeEditor\browser\dictation\editorDictation.ts; V\src\vs\workbench\contrib\speech\common\speechService.ts; C bundle (nls module vs/workbench/contrib/codeEditor/browser/dictation/editorDictation, label "Start Dictation in Editor", context key hasSpeechProvider)

### ED-2.40 Place the cursor and drag out a selection with the mouse
- Tag: both
- Evidence: source
- Boundary: no
- What: The person clicks to put the caret where they want it, Shift+clicks to stretch the selection from the caret to the click, and drags to select; dragging past the top, bottom or side edge scrolls the text along, and dragging after a double click or triple click grows the selection by whole words or whole lines.
- Commands: _moveTo; _moveToSelect; _wordSelectDrag; _lineSelectDrag
- Menus: none found
- Keys: none found (mouse gestures; Shift+click extends the selection)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\browser\view\viewController.ts (dispatchMouse); V\src\vs\editor\browser\controller\mouseHandler.ts (MouseDownOperation, Shift on mouse down, drag scrolling at the editor edge); V\src\vs\editor\browser\controller\dragScrolling.ts; V\src\vs\editor\browser\coreCommands.ts (BaseMoveToCommand, WordSelectDrag, LineSelectDrag); C bundle (ids _moveTo, _moveToSelect, _wordSelectDrag, _lineSelectDrag found as quoted strings)

### ED-2.41 Clear a selection with Escape
- Tag: both
- Evidence: source
- Boundary: no
- What: With text selected, the person presses Escape (or Shift+Escape) and the selection collapses to a single caret at the end the caret was on, with the text left as it was.
- Commands: cancelSelection
- Menus: none found
- Keys: Escape and Shift+Escape (textInputFocus, when the selection is not empty; with several cursors Escape first drops the extra cursors, see ED-2.1)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\browser\coreCommands.ts (CancelSelection); V\src\vs\editor\common\cursor\cursorMoveCommands.ts (cancelSelection); C bundle (command id cancelSelection with precondition hasNonEmptySelection and primary key Escape found in the core commands code)

### ED-2.42 Type with an input method, dead keys and the macOS accent menu
- Tag: both
- Evidence: source
- Boundary: no
- What: The person types Chinese, Japanese, Korean or other composed text through the operating system's input method, uses dead keys for accents, or holds a letter on macOS to pick an accented one; the unfinished text shows in place while it is composed, the input method window opens at the caret, and the final text replaces it. A setting picks whether the editor takes input through the browser's EditContext or through a hidden text area.
- Commands: compositionStart; compositionType; compositionEnd; replacePreviousChar
- Menus: none found
- Keys: none found (the keys belong to the operating system's input method)
- Settings: editor.editContext; editor.renderRichScreenReaderContent
- Cursor: changed - the four composition commands are present; the setting is there under its older name editor.experimentalEditContextEnabled ("Sets whether the new experimental edit context should be used instead of the text area", on by default only outside the stable build), and the name editor.editContext was not found
- Source: V\src\vs\editor\browser\coreCommands.ts (registerOverwritableCommand for Handler.CompositionStart, CompositionType, CompositionEnd, ReplacePreviousChar); V\src\vs\editor\browser\controller\editContext\native\nativeEditContext.ts; V\src\vs\editor\browser\controller\editContext\textArea\textAreaEditContextInput.ts (macOS long press case); V\src\vs\editor\common\config\editorOptions.ts (editContext); V\src\vs\editor\browser\config\migrateOptions.ts (migration from experimentalEditContextEnabled); C bundle (command ids compositionStart, compositionType, compositionEnd, replacePreviousChar as quoted strings; option experimentalEditContextEnabled with description nls 598; class name native-edit-context)

### ED-2.43 Keep undo history when a file is reopened
- Tag: both
- Evidence: source
- Boundary: no
- What: The person closes a file's tab, opens the file again, and Undo and Redo still step through the earlier edits, because the editor keeps the undo history of a closed file on disk (up to a memory limit) and restores it when the file is unchanged; a setting turns it off.
- Commands: undo; redo
- Menus: none found
- Keys: none found (same keys as ED-2.31)
- Settings: files.restoreUndoStack
- Cursor: present
- Source: V\src\vs\editor\common\services\modelService.ts (_shouldRestoreUndoStack, _onWillDispose, MAX_MEMORY_FOR_CLOSED_FILES_UNDO_STACK of 20 MB, SHA1 check on reopen); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (files.restoreUndoStack, default true); C bundle (setting files.restoreUndoStack with description "Restore the undo stack when a file is reopened." nls 7262, and a shouldRestoreUndoStack reader of that setting)

### ED-2.44 Remove Vim mode
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Run Cursor Vim: Remove Vim Mode from the Command Palette to uninstall the Vim extension (vscodevim.vim) and reload the window.
- Commands: cursor.removevim
- Menus: commandPalette (no category, no condition)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (action cursor.removevim, title "Cursor Vim: Remove Vim Mode", f1 true with no precondition; it lists installed extensions, uninstalls the one whose id is vscodevim.vim, then reloads the window; the id is in the tracked command list under vim_mode)

## Handed off

- Format Document, Format Selection, Format Document With..., Format Modified Lines and format on save (`editor.action.formatDocument`, `editor.action.formatSelection`, `editor.formatOnSave`) - ED-4. Only the on-type and on-paste behavior stays here as ED-2.24.
- Bracket pair colorization and bracket pair guides (`editor.bracketPairColorization.enabled`, `editor.guides.bracketPairs`, `editor.guides.highlightActiveBracketPair`) - ED-5. They change how text is drawn, not how it is edited.
- Toggle Control Characters (`editor.action.toggleRenderControlCharacter`, `editor.renderControlCharacters`) - ED-5.
- Toggle Tab Key Moves Focus (`editor.action.toggleTabFocusMode`, Ctrl+M) - ED-5.
- Announce Cursor Position (`editor.action.announceCursorPosition`) and the screen reader mode toggle - ED-5.
- Scroll commands (`scrollLineUp`, `scrollLineDown`, `scrollPageUp`, `scrollPageDown`, `editorScroll`, Ctrl+Up and Ctrl+Down) - ED-5. They sit in `coreCommands.ts` but move the view, not the text.
- Go to Line and Go to Offset (`workbench.action.gotoLine`, `workbench.action.gotoOffset`) - ED-4 for the commands, QA for the picker.
- Occurrence highlighting by language provider and Go to Next or Previous Symbol Highlight (`editor.occurrencesHighlight`, `editor.action.wordHighlight.next`) - ED-4.
- Insert Snippet and Surround With Snippet (`editor.action.insertSnippet`, `editor.action.surroundWithSnippet`) - AC-4.
- Emmet abbreviation suggestions and the inline expanded preview (`emmet.showAbbreviationSuggestions`, `emmet.showExpandedAbbreviation`, `emmet.useInlineCompletions`, `emmet.showSuggestionsAsSnippets`) - AC-1.
- Tab completion and word-based suggestions (`editor.tabCompletion`, `editor.wordBasedSuggestions`) - AC-1.
- Status bar items that call ED-2 commands: the Spaces and Tab Size picker, the OVR overtype indicator, line and selection counts - SS-3.
- Save, Save As, Save All and Revert File (`workbench.action.files.save`, `workbench.action.files.saveAs`, `workbench.action.files.saveAll`, `workbench.action.files.revert`) - ED-1.
- The Cursor Blame submenu on the editor context menu (menu `EditorContextCursorBlame`) and the inline diff accept and reject actions (`editor.action.inlineDiffs.acceptAll`, `rejectAll`) - ED-6.
- Selection highlight and multi-cursor inside notebook cells (`V\src\vs\workbench\contrib\notebook\browser\contrib\multicursor\notebookSelectionHighlight.ts`) - ED-7.
- The editor right-click menu as a whole (`MenuId.EditorContext`, `editor.action.showContextMenu` on Shift+F10, the `editor.contextmenu` option, the Share submenu `EditorContextShare`) - no group owns it. The catalog lists it under items nobody owns, so the parent should place it. The line-number gutter menu (`EditorLineNumberContext`) was sent to ED-11 by the ED-5 pass. Not added here (confidence medium - the clipboard and multi-cursor entries in it are ED-2 features, but the menu itself is a container, not an editing feature).
- Toggle Tab Key Moves Focus (`editor.action.toggleTabFocusMode`, Ctrl+M, `editor.tabFocusMode`) - SS-3, already listed as SS-3.24 with the command, key and setting. The first ED-2 pass sent it to ED-5 and the ED-5 pass sent it back to ED-2, so neither lists it. Keep SS-3.24 as the one copy (confidence medium - the toggle is an editor action, but a second copy would repeat SS-3.24).
- Markdown source-editing features of the markdown-language-features extension (`markdown.editor.insertLinkFromWorkspace`, `markdown.editor.insertImageFromWorkspace`, paste a URL as a formatted link, paste and drop of files, update links on paste; settings `markdown.editor.drop.*`, `markdown.editor.filePaste.*`, `markdown.editor.pasteUrlAsFormattedLink.enabled`, `markdown.editor.updateLinksOnPaste.enabled`) - no group. The ED-10 pass handed them to ED-2, but the catalog scope names only the Markdown preview for that extension, so I did not add them (confidence medium - the scope text does not say source editing is excluded, so the user could widen it). All of these ids are in the Cursor copy of the extension manifest.

## Unclear

- Word wrap (ED-2.35), render whitespace (ED-2.36) and selection highlight (inside ED-2.3) were placed in ED-5 by the pass 1 research (`toggleWordWrap.ts`, `toggleRenderWhitespace.ts`). They are listed here because the prompt named them for ED-2. If the ED-5 pass lists them too, the parent should keep one copy.
- No group owns the right-click container of the text editor: `editor.action.showContextMenu` (Shift+F10), the `editor.contextmenu` option, the Share submenu (`EditorContextShare`, `workbench.action.share`), and the line-number gutter menu (`EditorLineNumberContext`, fed by Git quick diff, testing, breakpoints and Cursor's link item). ED-2.39 only uses the gutter and Share menus. The parent should place them; ED-2 is the closest fit (confidence medium - the menu is the home of the ED-2 clipboard and multi-cursor items).
- Newer than 1.128, seen only in N and not listed: `editor.selectedTextMatchMode` (changes how ED-2.3 and ED-2.4 match text) and `editor.wordWrapIndicator` (ED-2.35). No ED-2 command id was new in N. I compared the `id:` and `ID =` constants in every ED-2 source folder and the Emmet manifest (still 23 commands and 12 settings).
- Four option names exist in V and were not found in the Cursor bundle even as bare words: `autoIndentOnPasteWithinString`, `trimWhitespaceOnDelete`, `selectionHighlightMultiline`, `selectionHighlightMaxLength`. They are marked `changed` on ED-2.3, ED-2.23 and ED-2.25. Cursor's code looks older than 1.128.0 here, as pass 1 noted for other sections.
- Menus lists explicit menu registrations only. Editor actions made with `registerEditorAction` show in the Command Palette without an item of their own, so `none found` under Menus does not mean the Palette lacks them.
- Keys are the V defaults. Cursor's own keybindings for these items were not read, except Alt+Z for word wrap, which is visible in the bundle. All Cursor evidence is ids, labels and module names. Nothing was run on screen.
- Mouse gestures (Alt+click, Shift+Alt+drag, middle-button column drag, double-click next to brackets, text drag and drop) were read from `viewController.ts`, `mouseHandler.ts` and `dnd.ts`, not run. What happens with the modifier switched was not traced for every gesture.
- Dictation (ED-2.38) needs a speech provider extension. Stock ships none built in and the Cursor install has no speech extension folder, so the command is probably disabled. Not confirmed on screen.
- Cursor-only ids in the editor action space that I did not list: `editor.action.undoLexical` and `editor.action.redoLexical` (they sit beside composer constants, so they look like the chat prompt box, not the text editor), `editor.action.startAiInstantSearch` (inside the find id table, for the ED-3 owner to check), and the editor context entries that hand text to chat (out of scope).
- The core `type`, `cut` and `paste` handlers (`registerOverwritableCommand` in `coreCommands.ts`) run all typing and clipboard behavior, so the typing helpers ED-2.22 to ED-2.25 have no command id of their own. I list `type` where it is the entry point.
- ED-2.39 (Copy a cursor.com link) repeats ED-1.28, which lists the same two commands on the same four menus. The parent should keep one copy.
- The setting `editor.doubleClickSelectsBlock` (the double-click-next-to-a-bracket behavior described in ED-2.6) is not named in any feature and was not found in the Cursor bundle. The Linux command `editor.action.selectionClipboardPaste` ("Paste Selection Clipboard", present in Cursor, nls 5136) is not named in ED-2.28 either. I did not reword existing features to add them.
- What the overflow check compared: every command id in the ED-2 source folders, `coreCommands.ts`, `codeEditor\browser` and the Emmet manifest against the feature list; every editor option name and every `files.*` and `editor.*` setting against all feature files; the Edit and Selection menus; the Cursor bundle for `editor.action.` ids that stock lacks (none belong to ED-2) and for Cursor-only editing strings; and N for new ids and options (`selectedTextMatchMode`, `wordWrapIndicator`, `font`, `fullwidthCharacterWidth`, `occurrencesHighlightFromAllProviders`, none a new ED-2 feature). Word wrap and Render whitespace were not touched.
- Overflow check on 2026-10-25 added 4 features (ED-2.40 to ED-2.43).
