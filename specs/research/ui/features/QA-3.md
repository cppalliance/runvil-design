Group: QA-3 Go to a spot in this file. Section: Quick access. Date: 2026-10-25. Pass: 2.
Features: 15.

### QA-3.1 Go to Line/Column
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+G, or typing a colon, opens a one-row list that tells you what Enter will do. Type a line number, optionally followed by a column (12:5, 12,5 or 12#5), and Enter moves the cursor there. In stock the column counts visible columns, so a tab counts as its tab width. With no text editor open the row reads "Open a text editor first to go to a line or an offset."
- Commands: workbench.action.gotoLine
- Menus: MenubarGoMenu (Go to Line/Column..., group 5_infile_nav, order 1)
- Keys: Ctrl+G; Control+G (Mac)
- Settings: none found
- Cursor: changed (same command, key, menu item and prefix; older copy with other row text such as "Go to line {0} and character {1}." and "Current Line: {0}, Character: {1}. Type a line number between 1 and {2} to navigate to.", the column is a plain character column, and the no-editor row reads "Open a text editor first to go to a line.")
- Source: V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts; C bundle (class EOr, base class dig, action rlg, messages 1476 to 1480, 5039 to 5041, 8907)

### QA-3.2 Count back from the end with negative numbers
- Tag: both
- Evidence: source
- Boundary: no
- What: A negative line number counts from the last line (-1 is the last line), and in stock a negative column counts from the end of the line. Stock also clips a number that is too big to the nearest valid line or column, so Enter always lands somewhere. In Cursor, only the line may be negative, and a number out of range shows a hint row and Enter does nothing.
- Commands: workbench.action.gotoLine
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (negative line works, negative column and clipping do not)
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; C bundle (parsePosition, isValidLineNumber, isValidColumn in class dig)

### QA-3.3 Go to Offset
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Typing two colons (::1024), or running the command, goes to a character position counted from the start of the file. A negative number counts from the end. The row says which line and column the offset lands on, or asks for a position "from 1 to {max}".
- Commands: workbench.action.gotoOffset
- Menus: CommandPalette (Go to Offset..., f1 only)
- Keys: none found
- Settings: none found
- Cursor: missing (no command id, no label, no :: prefix in the bundle)
- Source: V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; C bundle searched for workbench.action.gotoOffset, no hits

### QA-3.4 Switch between 1-based and 0-based offsets
- Tag: vscode
- Evidence: source
- Boundary: no
- What: While you are in offset mode (the text starts with ::) a toggle button "Toggle Zero-Based Offset" appears at the right of the input. It flips between counting from 1 and from 0, and the choice is remembered for all windows.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing (no gotoLine.useZeroBasedOffset key and no toggle label in the bundle)
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts (storage key gotoLine.useZeroBasedOffset, application scope); C bundle searched, no hits

### QA-3.5 Preview the target line as you type
- Tag: both
- Evidence: source
- Boundary: no
- What: As the typed number changes, the editor scrolls that line to the middle with a smooth scroll and paints the whole line with a range highlight plus a mark in the overview ruler. Pressing Escape puts back the view you had before. A number that gives no line clears the highlight.
- Commands: workbench.action.gotoLine
- Menus: none found
- Keys: Escape
- Settings: none found
- Cursor: present (theme colors editor.rangeHighlightBackground and editorOverviewRuler.rangeHighlightForeground and the quick-access-range-highlight decoration are in the bundle)
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\editorNavigationQuickAccess.ts; V\src\vs\editor\common\core\editorColorRegistry.ts; C bundle (base class K4o, addDecorations, revealRangeInCenter)

### QA-3.6 See absolute line numbers while Go to Line is open
- Tag: both
- Evidence: source
- Boundary: no
- What: If the editor shows relative line numbers, they switch to normal numbers while the Go to Line list is open, so the number you type matches what you see. They go back to relative when the list closes.
- Commands: workbench.action.gotoLine
- Menus: none found
- Keys: none found
- Settings: editor.lineNumbers
- Cursor: present
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; C bundle (provideWithTextEditor in class dig, lineNumbers on and relative)

### QA-3.7 Land on the spot centered, with focus back in the editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Enter selects the target, scrolls it to the middle of the editor with a smooth scroll, and returns focus to the editor. The jump counts as a navigation jump, so Go Back returns to where you were. Stock tells screen readers "Line {0}, column {1}: {text}", and Cursor reads out only the line text.
- Commands: workbench.action.gotoLine; workbench.action.gotoSymbol
- Menus: none found
- Keys: Enter
- Settings: none found
- Cursor: changed (same jump, scroll and focus; shorter screen reader message)
- Source: V\src\vs\editor\contrib\quickAccess\browser\editorNavigationQuickAccess.ts; V\src\vs\workbench\browser\parts\editor\textEditor.ts (JUMP selection source); V\src\vs\workbench\services\history\browser\historyService.ts (JUMP adds a navigation stop); C bundle (gotoLocation with code.jump)

### QA-3.8 Open the jump in a side editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Alt+Enter, or Ctrl+Enter while previews from Quick Open are off (the default), opens the same file in the side group at the target. Symbol rows also have an "Open to the Side" button ("Open to the Bottom" when the side direction is down). The view in the first editor is restored.
- Commands: workbench.action.gotoLine; workbench.action.gotoSymbol
- Menus: none found
- Keys: Alt+Enter (modifier read by the picker, no keybinding rule); Ctrl+Enter (same)
- Settings: workbench.editor.openSideBySideDirection; workbench.editor.enablePreviewFromQuickOpen
- Cursor: present
- Source: V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts; V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; C bundle (gotoLocation overrides in classes EOr and bce, messages 1485 and 1486)

### QA-3.9 Jump without closing the list
- Tag: both
- Evidence: source
- Boundary: no
- What: Right Arrow on a row moves the editor to that line or symbol while the list stays open and focus stays in the list, so you can hop between spots.
- Commands: quickInput.acceptInBackground
- Menus: none found
- Keys: Right Arrow (when in quick pick and focus is in the list or the cursor is at the end of the input box)
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; C bundle (quickInput.acceptInBackground rule, canAcceptInBackground in both providers)

### QA-3.10 Go to Symbol in Editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+Shift+O, or typing @, lists the symbols of the open file under a "symbols (N)" heading. Each row has a kind icon, the name, and its container, with the matched letters highlighted, and deprecated symbols are struck through. With an empty box the symbol that contains the cursor starts as the active row. In a diff editor the list uses the modified side.
- Commands: workbench.action.gotoSymbol
- Menus: MenubarGoMenu (Go to Symbol in Editor..., group 4_symbol_nav, order 1)
- Keys: Ctrl+Shift+O (when the accessible view and accessibility help are not shown); Cmd+Shift+O (Mac)
- Settings: none found
- Cursor: present (same command, key, menu item and placeholder "Type the name of a symbol to go to."; the rows have no shift-to-attach callback)
- Source: V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; C bundle (class bce, base class gie, action r5o, messages 5043 to 5047, 1487 to 1500)

### QA-3.11 Group symbols by category
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing @: (a colon right after the @) groups the symbols by kind and puts a heading with a count over each group, such as "methods (4)", "functions (2)" or "classes (1)". The groups are ordered by the kind's heading text, and inside a group rows are sorted by match score, then by position in the file. Anything after the colon filters as usual.
- Commands: workbench.action.gotoSymbol
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the by-category help text 5046 and the kind headings 1487 to 1500 are in the table; prefix constant SCOPE_PREFIX in the bundle)
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts (help entry "Go to Symbol in Editor by Category"); C bundle (doGetSymbolPicks with SCOPE_PREFIX, messages 5046 and 1487 to 1500)

### QA-3.12 Narrow symbols by container with a second word
- Tag: both
- Evidence: source
- Boundary: no
- What: With two search words the first matches the symbol name and the rest match its container (for example the class name), so "run Server" finds run inside Server. The container text is highlighted in the grey description.
- Commands: workbench.action.gotoSymbol
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; C bundle (doGetSymbolPicks, container query split)

### QA-3.13 Preview the symbol as you move through the list
- Tag: both
- Evidence: source
- Boundary: no
- What: The active row scrolls its symbol to the middle of the editor with a smooth scroll and highlights the symbol's whole range, plus a mark in the overview ruler. Escape puts the old view back, and the cursor does not move until you accept.
- Commands: workbench.action.gotoSymbol
- Menus: none found
- Keys: Up Arrow; Down Arrow; Escape
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\editorNavigationQuickAccess.ts; C bundle (onDidChangeActive handler with revealRangeInCenter and addDecorations)

### QA-3.14 Go to Symbol in notebooks and other non-text editors
- Tag: both
- Evidence: source
- Boundary: no
- What: When the active editor is not a plain text editor but can make an outline (a notebook, for example), the same list shows the outline entries with their own icons. Moving through the rows previews the entry, Enter reveals it, and an empty filter result shows "No matching entries". For notebooks a setting chooses whether full code symbols are listed or only Markdown headers.
- Commands: workbench.action.gotoSymbol
- Menus: none found
- Keys: Ctrl+Shift+O
- Settings: notebook.gotoSymbols.showAllSymbols
- Cursor: present
- Source: V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\outline\notebookOutline.ts; C bundle (doGetOutlinePicks, getQuickPickElements, canCreateOutline, message 5042 and 8034)

### QA-3.15 See why the symbol list is empty
- Tag: both
- Evidence: source
- Boundary: no
- What: With no editor open the list says "To go to a symbol, first open a text editor with symbol information." With an editor that has no symbol provider it says "The active text editor does not provide symbol information." and switches to the real list on its own if language support arrives late. A search with no hits says "No matching editor symbols" and an empty file says "No editor symbols".
- Commands: workbench.action.gotoSymbol
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; C bundle (provideWithoutTextEditor, waitForLanguageSymbolRegistry, messages 1481 to 1484)

## Handed off

- Status bar line and column item (status.editor.selection, click runs Go to Line/Column): SS-3
- Go menu and its rows (Go to Line/Column, Go to Symbol in Editor): TB-1 (commands and menu ids are listed in the features above)
- Command center row "Go to Symbol in Editor" (order 40) and the `?` help list entries for the colon, double colon, @ and @: prefixes: QA-5
- Go Back and Go Forward, which walk the stops that a jump records: TB-2 (back and forward arrows) and TB-1 (Go menu)
- Outline view in the side bar and the breadcrumbs symbol picker, which show the same symbols in other places: SB-7 and ED-5
- Go to Definition, Peek and the other language-help jumps: ED-4
- The shared list, input and button chrome (including the in-input toggle button style used for Toggle Zero-Based Offset): QA-6
- Go to File with a :line:col or @symbol suffix: QA-2

## Unclear

- Cursor's bundle carries older copies of the Go to Line files (older prompts, no offset mode, no negative column, no clipping, shorter screen reader message). The bundle says base 1.128.0, so I could not tell whether Cursor reverted these files or never took the newer ones. Any build of this group should be checked against Cursor feature by feature.
- Default key tables were not pulled from the Cursor bundle. Ctrl+G, Ctrl+Shift+O and Ctrl+P were confirmed only from the action registrations, not from the keybinding resolver, so a changed default would not show here.
- Shift+Enter on a symbol row attaches it to the focused chat in stock. It is chat, so out of scope and not listed.
- The standalone editor has its own Go to Line and Go to Symbol commands (editor.action.gotoLine and editor.action.quickOutline). They apply only to the web-component editor, not the workbench, so they were left out.
- Whether Ctrl+Enter or Alt+Enter overlap with other quick input keys in the Cursor build was not traced.
- Behavior on screen was not watched. Every statement comes from reading source or bundle strings.

