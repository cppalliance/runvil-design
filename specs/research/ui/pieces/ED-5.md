Group: ED-5 Editor view and orientation aids. Pass: 3. Date: 2026-10-25.

### ED-5.1
1. With the minimap on, a narrow scaled-down picture of the file's text appears along the right edge of the editor.
2. Choosing View > Appearance > Minimap hides the minimap when it is showing and shows it when it is hidden, and the menu row shows a check mark while the minimap is on.
3. Running Toggle Minimap (`editor.action.toggleMinimap`) from the Command Palette flips the minimap the same way.
4. Right-clicking the scrollbar or the minimap opens a menu whose Minimap item is checked while the minimap is on, and choosing it flips the minimap.
5. Setting `editor.minimap.enabled` turns the minimap on or off, default on, and the menu, Command Palette and right-click entries all change this same setting.

### ED-5.2
1. Right-clicking the scrollbar or the minimap shows minimap choices for Render Characters, Vertical size, Slider and Side, and each choice takes effect at once.
2. Render Characters, checked by default, draws the real letters of each line in the minimap, and unchecked draws plain color blocks (`editor.minimap.renderCharacters`).
3. Vertical size Proportional, the default, keeps the minimap the same size as the file contents so it may scroll, Fill stretches or shrinks it to fill the editor height, and Fit only shrinks it so it is never taller than the editor (`editor.minimap.size`).
4. Slider Mouse Over, the default, shows the box marking the visible part only while the pointer is over the minimap, and Always shows it all the time (`editor.minimap.showSlider`).
5. Side Right, the default, or Left puts the minimap on that edge of the editor (`editor.minimap.side`), and the stock menu names this submenu Side.
6. Setting `editor.minimap.scale` draws minimap text at scale 1, 2 or 3, default 1.
7. Setting `editor.minimap.maxColumn`, default 120, limits how many columns of each line are drawn so the minimap stays that wide at most.
8. Setting `editor.minimap.autohide` chooses `none` (always shown, the default), `mouseover` (shown only while the pointer is over the minimap) or `scroll` (shown only while the editor is being scrolled).

### ED-5.3
1. A comment containing `MARK:` followed by text is drawn in the minimap as a larger label showing that text, and setting `editor.minimap.showMarkSectionHeaders` turns this on or off, default on.
2. A `MARK:` comment with a dash after the colon also draws a separator line with its label in the minimap.
3. Named region markers are drawn in the minimap as titled labels showing the region name, and setting `editor.minimap.showRegionSectionHeaders` turns this on or off, default on.
4. Setting `editor.minimap.markSectionHeaderRegex` replaces the pattern that finds section headers in comments, and the pattern needs a named group `label` and may add a group `separator`.
5. Setting `editor.minimap.sectionHeaderFontSize` sets the size of the header labels in the minimap, default 9.
6. Setting `editor.minimap.sectionHeaderLetterSpacing` sets the space in pixels between the letters of a header label, default 1.

### ED-5.4
1. With sticky scroll on, scrolling down inside nested scopes such as a class, function or block keeps the header line of each enclosing scope pinned at the top of the editor.
2. Clicking a pinned line scrolls the editor to the start of that scope.
3. Shift-clicking a pinned line scrolls the editor to the end of that scope.
4. Clicking the fold arrow on a pinned line collapses or expands that scope.
5. Choosing View > Appearance > Sticky Scroll turns pinned lines on or off, and the menu row shows a check mark while it is on.
6. Right-clicking the pinned lines opens a menu with the sticky scroll toggle and the breadcrumbs toggle.
7. Running Toggle Editor Sticky Scroll (`editor.action.toggleStickyScroll`) from the Command Palette flips sticky scroll.
8. Setting `editor.stickyScroll.enabled` turns sticky scroll on or off, default on.

### ED-5.5
1. Setting `editor.stickyScroll.maxLineCount` sets the most lines that can be pinned at once, default 5, allowed from 1 to 20.
2. Setting `editor.stickyScroll.defaultModel` chooses what decides which lines stick, `outlineModel` (default), `foldingProviderModel` or `indentationModel`, and each falls back to the next one when it is not available.
3. Setting `editor.stickyScroll.scrollWithEditor`, default on, makes the pinned lines move sideways with the editor's horizontal scrollbar, and when off they stay in place.

### ED-5.6
1. Running Focus Editor Sticky Scroll (`editor.action.focusStickyScroll`) from the Command Palette moves keyboard focus onto the pinned lines, and it has no default key.
2. With the pinned lines focused, DownArrow selects the next pinned line and UpArrow selects the previous one.
3. With the pinned lines focused, Enter moves the editor to the selected pinned line.
4. With the pinned lines focused, Escape returns focus to the text.
5. These four keys act only while the pinned lines have focus.

### ED-5.7
1. With breadcrumbs on, a bar above the text lists the folders, the file and the symbols around the cursor, separated by small arrows.
2. Choosing View > Appearance > Breadcrumbs shows or hides the bar, and the menu row shows a check mark while breadcrumbs are on.
3. Running Toggle Breadcrumbs (`breadcrumbs.toggle`) from the Command Palette flips the bar.
4. Right-clicking the pinned sticky scroll lines offers a breadcrumbs toggle.
5. While breadcrumbs are off, pressing Ctrl+Shift+. runs `breadcrumbs.toggleToOn` and turns them back on.
6. Setting `breadcrumbs.enabled` turns the bar on or off, default on.
7. In Cursor the View > Appearance row for breadcrumbs is labeled Toggle Breadcrumbs, where stock labels it Breadcrumbs. [cursor only]

### ED-5.8
1. Setting `breadcrumbs.filePath` chooses `on` (show the whole path, the default), `off` (hide the path) or `last` (show only the last path element).
2. Setting `breadcrumbs.symbolPath` chooses `on` (show all symbols, the default), `off` (hide symbols) or `last` (show only the current symbol).
3. Setting `breadcrumbs.symbolSortOrder` orders the symbols in the symbol picker by `position` (the default), `name` or `type`.
4. Setting `breadcrumbs.icons`, default on, shows or hides the icon beside each breadcrumb item.
5. Each of the `breadcrumbs.show*` settings, such as `breadcrumbs.showFiles`, `breadcrumbs.showClasses`, `breadcrumbs.showMethods` and `breadcrumbs.showVariables`, turns one kind of symbol on or off in the bar and its picker.

### ED-5.9
1. Clicking a file or folder item in the breadcrumbs opens a picker listing the sibling files and folders at that level, and clicking a symbol item opens a picker listing the symbols in the file.
2. Choosing an entry in the picker with a click or Enter opens that file or jumps to that symbol.
3. Ctrl+Shift+. (`breadcrumbs.focusAndSelect`) focuses the last breadcrumb item and opens its picker, and Ctrl+Shift+; (`breadcrumbs.focus`) focuses the items without opening a picker.
4. With an item focused, RightArrow and LeftArrow move to the next or previous item, and these item keys work only while the breadcrumbs are visible and active.
5. With the picker open, Ctrl+RightArrow and Ctrl+LeftArrow (Alt on macOS) move to the next or previous item and open its picker.
6. Enter or DownArrow on a focused item (`breadcrumbs.selectFocused`) opens that item's picker.
7. In the picker, Space shows the highlighted entry in the editor without leaving the picker, and Ctrl+Enter opens it in a side editor.
8. Escape (`breadcrumbs.selectEditor`) leaves the breadcrumbs and returns focus to the text.
9. Setting `breadcrumbs.useQuickPick`, when on, shows the choices in a Quick Pick instead of the dropdown picker.

### ED-5.10
1. Right-clicking an editor tab shows Copy Breadcrumbs Path in the cut, copy and paste group when breadcrumbs are available and the file has symbols. [stock only]
2. Choosing Copy Breadcrumbs Path copies the symbol path at the cursor to the clipboard. [stock only]
3. Running Copy Breadcrumbs Path (`breadcrumbs.copyPath`) from the Command Palette does the same thing. [stock only]
4. Setting `breadcrumbs.symbolPathSeparator`, default a period, sets the text placed between the symbol names in the copied path. [stock only]

### ED-5.11
1. Fold (Ctrl+Shift+[, Cmd+Alt+[ on macOS) collapses the region the cursor is in, hiding its lines after the first.
2. Unfold (Ctrl+Shift+], Cmd+Alt+] on macOS) expands the collapsed region at the cursor.
3. Toggle Fold (Ctrl+K Ctrl+L) collapses the region at the cursor when it is open and expands it when it is collapsed.
4. Fold Recursively (Ctrl+K Ctrl+[) collapses the region at the cursor and every region nested inside it.
5. Unfold Recursively (Ctrl+K Ctrl+]) expands the region at the cursor and every region nested inside it.
6. Toggle Fold Recursively (Ctrl+K Ctrl+Shift+L) flips the region at the cursor and everything nested inside it.
7. All six commands are in the Command Palette, and the keys work only while the editor text has focus and folding is on (`editor.folding`).

### ED-5.12
1. Fold All (Ctrl+K Ctrl+0) collapses every region in the file.
2. Unfold All (Ctrl+K Ctrl+J) expands every region in the file.
3. Fold All Except Selected (Ctrl+K Ctrl+-) collapses every region except the ones that hold the selection.
4. Unfold All Except Selected (Ctrl+K Ctrl+=) expands every region except the ones that hold the selection.
5. All four commands are in the Command Palette, and the keys work only while the editor text has focus.

### ED-5.13
1. Fold Level 1 (Ctrl+K Ctrl+1) collapses every region that sits at the first nesting level.
2. Fold Level 2 through Fold Level 7 (Ctrl+K Ctrl+2 to Ctrl+K Ctrl+7) do the same for regions nested two to seven levels deep.
3. A region that holds a selected line stays open when folding to a level.
4. The Command Palette lists Fold Level 1 to Fold Level 7, and the keys work only while the editor text has focus.

### ED-5.14
1. Fold All Block Comments (Ctrl+K Ctrl+/) collapses every block comment in the file.
2. Fold All Regions (Ctrl+K Ctrl+8) collapses every region marked with region start and end comments.
3. Unfold All Regions (Ctrl+K Ctrl+9) expands every region marked with region start and end comments.
4. All three commands are in the Command Palette, and the keys work only while the editor text has focus.

### ED-5.15
1. Create Folding Range from Selection (Ctrl+K Ctrl+,) with more than one line selected turns those lines into a collapsed folding range and puts the cursor at the start of the first line.
2. A folding range made by hand shows its own manual fold arrow in the gutter, different from the arrow of an automatic range.
3. Remove Manual Folding Ranges (Ctrl+K Ctrl+.) removes the hand-made folding ranges that touch the selected lines.
4. Both commands are in the Command Palette, and the keys work only while the editor text has focus.

### ED-5.16
1. Go to Next Folding Range (`editor.gotoNextFold`) moves the cursor to the first line of the next folding range below it.
2. Go to Previous Folding Range (`editor.gotoPreviousFold`) moves the cursor to the first line of the previous folding range above it.
3. Go to Parent Fold (`editor.gotoParentFold`) moves the cursor to the first line of the range that contains the current one.
4. All three commands are in the Command Palette and have no default key.

### ED-5.17
1. Toggle Import Fold (`editor.toggleImportFold`) collapses the block of import lines at the top of the file when it is open and expands it when it is collapsed.
2. Setting `editor.foldingImportsByDefault`, default off, makes files open with their import block already collapsed.
3. Toggle Import Fold is in the Command Palette and has no default key.

### ED-5.18
1. In the gutter next to the line numbers, clicking the arrow on the first line of an open region collapses it, and clicking the arrow on a collapsed region expands it.
2. A collapsed line shows a "..." marker after its text, and clicking that marker expands the region.
3. Setting `editor.showFoldingControls` chooses `always` (arrows always shown), `mouseover` (arrows shown only while the pointer is over the gutter, the default) or `never`.
4. Setting `editor.foldingHighlight`, default on, highlights collapsed ranges.
5. Setting `editor.unfoldOnClickAfterEndOfLine`, default off, makes a click on the empty space after a collapsed line expand it.
6. Shift-click or middle-click on the arrow acts on the regions nested inside first, and acts on the region itself when there are none or they are already in that state.
7. Alt-click on the arrow expands the unrelated regions that are collapsed, or collapses all unrelated regions when none are collapsed.

### ED-5.19
1. Setting `editor.folding`, default on, turns folding on or off, and when off the fold arrows disappear and the fold commands do nothing.
2. Setting `editor.foldingStrategy` chooses `auto` (language folding when available, otherwise indentation, the default) or `indentation`.
3. Setting `editor.foldingMaximumRegions` caps how many foldable regions one file gets, default 5000, allowed from 10 to 65000.
4. Setting `editor.defaultFoldingRangeProvider` picks which installed language extension supplies the folding ranges.

### ED-5.20
1. With `editor.lineNumbers` set to `on`, the default, each line shows its own number in the gutter to the left of the text.
2. Setting `editor.lineNumbers` to `relative` shows the cursor line's own number and, on every other line, its distance from the cursor line.
3. Setting `editor.lineNumbers` to `interval` shows numbers only on the cursor line, every tenth line and the last line.
4. Setting `editor.lineNumbers` to `off` hides the line numbers.
5. Setting `editor.lineNumbersMinChars` sets the narrowest width of the number column in digits, default 5, allowed from 1 to 300.
6. Setting `editor.renderFinalNewline` chooses whether the empty last line after a final newline gets a number, `on`, `off` or `dimmed`, where dimmed is the default on Linux and on is the default elsewhere.
7. Setting `editor.glyphMargin`, default on, keeps or removes the icon margin beside the line numbers.

### ED-5.21
1. Setting `editor.rulers` to a list of column numbers draws a thin vertical line behind the text at each of those columns, and the default list is empty so no rulers show.
2. An entry in `editor.rulers` can be an object with a `column` and a `color` to draw that one ruler in its own color.

### ED-5.22
1. With `editor.renderLineHighlight` set to `line`, the default, the line the cursor is on is tinted across the editor.
2. Setting `editor.renderLineHighlight` to `gutter` tints only the gutter beside the cursor line, `all` tints both the line and the gutter, and `none` tints neither.
3. Setting `editor.renderLineHighlightOnlyWhenFocus`, default off, draws the tint only while the editor has focus when it is on.

### ED-5.23
1. Setting `editor.guides.indentation`, default on, draws a thin vertical line at each indent level.
2. Setting `editor.guides.highlightActiveIndentation` chooses `true` (emphasize the guide of the cursor's indent level, the default), `always` (emphasize it even when bracket guides are emphasized) or `false`.
3. Setting `editor.guides.bracketPairs` chooses `false` (the default), `true` (vertical guides join the two ends of every bracket pair) or `active` (only for the pair around the cursor).
4. Setting `editor.guides.bracketPairsHorizontal` chooses `active` (the default, horizontal guides only for the active pair), `true` (for every pair) or `false`.
5. Setting `editor.guides.highlightActiveBracketPair`, default on, emphasizes the guides of the bracket pair around the cursor.

### ED-5.24
1. When the cursor is next to a bracket, that bracket and its partner each get a box so the end of the pair is easy to see.
2. Setting `editor.matchBrackets` chooses `always` (the default), `near` (only when the cursor is touching the bracket) or `never`.
3. The matching bracket also gets a small mark in the strip beside the vertical scrollbar.

### ED-5.25
1. Matching brackets are colored by how deeply they are nested, with a set of colors that repeats at deeper levels.
2. A bracket with no partner is drawn in its own warning color.
3. Setting `editor.bracketPairColorization.enabled`, default on, turns the colors on or off.
4. Setting `editor.bracketPairColorization.independentColorPoolPerBracketType`, default off, makes each kind of bracket cycle through the colors on its own instead of sharing one cycle.

### ED-5.26
1. Setting `editor.smoothScrolling`, default off, animates scrolling when it is on.
2. Setting `editor.mouseWheelScrollSensitivity`, default 1, multiplies how far one mouse wheel notch moves the text.
3. Setting `editor.fastScrollSensitivity`, default 5, multiplies the scroll speed while Alt is held.
4. Setting `editor.scrollPredominantAxis`, default on, locks a diagonal scroll to the main direction so the text does not drift sideways.
5. Setting `editor.inertialScroll`, default off, makes touchpad scrolling keep gliding after you let go. [stock only]

### ED-5.27
1. With `editor.scrollOnMiddleClick` on, pressing the middle mouse button in the editor puts a dot at the pointer, and moving the pointer away from the dot scrolls the text in that direction. [stock only]
2. Releasing the middle button after scrolling, or pressing any key, ends the auto-scroll and removes the dot. [stock only]
3. Setting `editor.scrollOnMiddleClick` turns this auto-scroll on or off, default off. [stock only]
4. Setting `editor.mouseMiddleClickAction` chooses what a middle click does, `default`, `openLink` (open the link under the pointer) or `ctrlLeftClick` (act like Ctrl+click). [stock only]

### ED-5.28
1. With `editor.scrollBeyondLastLine` on, the default, the last line can be scrolled up toward the top of the editor, leaving empty space below it.
2. With `editor.scrollBeyondLastLine` off, the last line stops at the bottom edge of the editor.
3. Setting `editor.scrollBeyondLastColumn`, default 4, sets how many extra columns the view can scroll past the end of the longest line.

### ED-5.29
1. Setting `editor.scrollbar.vertical` chooses `auto` (shown only when needed, the default), `visible` or `hidden` for the vertical scrollbar.
2. Setting `editor.scrollbar.horizontal` chooses `auto` (the default), `visible` or `hidden` for the horizontal scrollbar.
3. Setting `editor.scrollbar.verticalScrollbarSize` sets the thickness of the vertical scrollbar in pixels, default 14.
4. Setting `editor.scrollbar.horizontalScrollbarSize` sets the thickness of the horizontal scrollbar in pixels, default 12.
5. Setting `editor.scrollbar.scrollByPage`, default off, makes a click in the scrollbar track scroll by a page when on, and jump to the clicked spot when off.
6. Setting `editor.scrollbar.ignoreHorizontalScrollbarInContentHeight`, default off, stops the horizontal scrollbar from adding to the scrollable height when on.
7. A narrow strip beside the vertical scrollbar, the overview ruler, shows a mark at the cursor line's position with a border around the strip.
8. Setting `editor.hideCursorInOverviewRuler`, default off, removes the cursor mark from the strip when on, and `editor.overviewRulerBorder`, default on, shows or hides the border around the strip.

### ED-5.30
1. Ctrl+Up and Ctrl+Down (Ctrl+PageUp and Ctrl+PageDown on macOS) scroll the view up or down one line without moving the cursor.
2. Alt+PageUp and Alt+PageDown (Cmd+PageUp and Cmd+PageDown on macOS) scroll the view up or down one page without moving the cursor.
3. `scrollEditorTop` and `scrollEditorBottom` scroll the view to the first or last part of the file and have no default key.
4. `scrollLeft` and `scrollRight` scroll the view sideways and have no default key.
5. `revealLine` scrolls the line given in its arguments to the top, center or bottom of the editor, as set by its `at` argument.
6. `editorScroll` scrolls in the direction given by its `to` argument, by a unit such as line, wrapped line, page or half page and an amount, and has no default key.

### ED-5.31
1. Setting `editor.cursorStyle` chooses the cursor shape, `line` (the default), `block`, `underline`, `line-thin`, `block-outline` or `underline-thin`.
2. Setting `editor.cursorWidth`, default 0, sets the width in pixels of the cursor when its style is `line`.
3. Setting `editor.cursorBlinking` chooses how the cursor blinks, `blink` (the default), `smooth`, `phase`, `expand` or `solid`.
4. Setting `editor.cursorSmoothCaretAnimation` chooses whether the cursor glides between positions, `off` (the default), `explicit` or `on`.
5. Setting `editor.overtypeCursorStyle` chooses the cursor shape while overtype is on, with the same six shapes as `editor.cursorStyle` and `block` as the default.

### ED-5.32
1. Moving the cursor toward the top or bottom edge scrolls the view early so that `editor.cursorSurroundingLines` lines stay visible above and below the cursor, default 0.
2. Setting `editor.cursorSurroundingLinesStyle` chooses whether that margin applies only when the cursor moves by keyboard or command (`default`) or always, including mouse clicks (`all`).

### ED-5.33
1. With the cursor on a name, the other places the same symbol appears in the file are shaded, with reads in one color and writes in another where the language reports that difference.
2. Each shaded symbol also gets a mark in the strip beside the vertical scrollbar.
3. Pressing F7 (`editor.action.wordHighlight.next`) moves the cursor to the next highlighted occurrence.
4. Pressing Shift+F7 (`editor.action.wordHighlight.prev`) moves the cursor to the previous highlighted occurrence.
5. Running Trigger Symbol Highlight (`editor.action.wordHighlight.trigger`) from the Command Palette shows the highlights by hand, and it has no default key.
6. Setting `editor.occurrencesHighlight` chooses `singleFile` (the default), `multiFile` or `off`.
7. Setting `editor.occurrencesHighlightDelay` sets the delay in milliseconds before the highlights appear, default 0, allowed from 0 to 2000.

### ED-5.34
1. Selecting a word or phrase shades every other copy of that text in the file.
2. Each shaded copy also gets a mark in the strip beside the vertical scrollbar.
3. Setting `editor.selectionHighlight`, default on, turns the shading of other copies on or off.
4. Setting `editor.selectionHighlightMaxLength`, default 200, stops the shading when the selection has more characters than this, and 0 means no limit. [stock only]
5. Setting `editor.selectionHighlightMultiline`, default off, allows copies of a selection that spans several lines to be shaded when it is on. [stock only]

### ED-5.35
1. With whitespace shown, each space is drawn as a dot and each tab as an arrow.
2. Setting `editor.renderWhitespace` chooses `none`, `boundary` (all whitespace except single spaces between words), `selection` (only inside selected text, the default), `trailing` (only at line ends) or `all`.
3. Running Toggle Render Whitespace (`editor.action.toggleRenderWhitespace`) from the Command Palette switches the setting to `all` when it is `none`, and to `none` otherwise.
4. Choosing View > Appearance > Render Whitespace opens a submenu with None, Boundary, Selection, Trailing and All, and the current mode is checked. [stock only]
5. Choosing View > Appearance > Render Control Characters, or running Toggle Control Characters (`editor.action.toggleRenderControlCharacter`) from the Command Palette, turns the symbols for control characters such as null on or off.
6. Setting `editor.renderControlCharacters`, default on, controls whether control characters are shown as visible symbols.
7. Setting `editor.experimentalWhitespaceRendering` chooses how the dots and arrows are drawn, `svg` (the default), `font` or `off`.

### ED-5.36
1. Pressing Alt+Z turns word wrap on for the current file so long lines continue on the next row, and pressing it again turns wrap off.
2. Choosing View > Word Wrap, which shows a check mark while wrap is on, or running View: Toggle Word Wrap from the Command Palette does the same as Alt+Z.
3. For a file that is mostly very long lines, a button in the editor title bar reads Enable wrapping for this file, and once wrap is on it reads Disable wrapping for this file.
4. Setting `editor.wordWrap` chooses `off` (the default), `on` (wrap at the window edge), `wordWrapColumn` (wrap at a set column) or `bounded` (wrap at the smaller of the window edge and the column).
5. Setting `editor.wordWrapColumn`, default 80, sets the column used by `wordWrapColumn` and `bounded`.
6. Setting `editor.wrappingIndent` chooses how wrapped rows are indented, `none`, `same` (the default), `indent` or `deepIndent`.
7. Setting `editor.wrappingStrategy` chooses `simple` (the default) or `advanced` for how wrap points are worked out.
8. Setting `editor.wordBreak` chooses `normal` (the default) or `keepAll` for how Chinese, Japanese and Korean text breaks across rows.

### ED-5.37
1. Characters that look like other characters, invisible characters, and non-basic ASCII characters are drawn with a box in the text.
2. Hovering a boxed character explains why it is highlighted and offers to exclude that character, or to allow the characters common in a named language.
3. The hover also offers to turn off highlighting for that kind of character, for characters in comments, or for characters in strings, and a link to adjust settings.
4. Setting `editor.unicodeHighlight.nonBasicASCII` chooses `true`, `false` or `inUntrustedWorkspace`, which is the default.
5. Settings `editor.unicodeHighlight.invisibleCharacters` and `editor.unicodeHighlight.ambiguousCharacters`, both default on, turn the highlighting of those two kinds on or off.
6. Settings `editor.unicodeHighlight.includeComments` (default `inUntrustedWorkspace`) and `editor.unicodeHighlight.includeStrings` (default on) choose whether characters inside comments and strings are highlighted.
7. Setting `editor.unicodeHighlight.allowedCharacters` lists characters that are never highlighted.
8. Setting `editor.unicodeHighlight.allowedLocales` lists locales whose common characters are not highlighted, and the default allows the operating system locale and the VS Code locale.

### ED-5.38
1. Setting `editor.fontFamily` sets the font used in the editor.
2. Setting `editor.fontSize` sets the size of the editor text.
3. Setting `editor.fontWeight` sets the weight of the editor text as `normal`, `bold` or a number from 1 to 1000.
4. Setting `editor.fontLigatures`, default off, turns font ligatures on, or takes a text value for finer control.
5. Setting `editor.lineHeight` sets the line height, where 0 works it out from the font size, values under 8 multiply the font size and larger values are pixels.
6. Setting `editor.letterSpacing` sets the space between letters in pixels.
7. Running Increase Editor Font Size, Decrease Editor Font Size or Reset Editor Font Size (`editor.action.fontZoomIn`, `editor.action.fontZoomOut`, `editor.action.fontZoomReset`) from the Command Palette zooms the editor text, and they have no default key.
8. Setting `editor.mouseWheelZoom`, default off, lets you zoom the editor text by holding Ctrl (Cmd on macOS) and turning the mouse wheel.

### ED-5.39
1. Running Developer: Inspect Editor Tokens and Scopes (`editor.action.inspectTMScopes`) from the Command Palette opens a small panel next to the cursor that first reads Loading and then shows the result.
2. The panel lists the TextMate token at the cursor with details such as its language and its foreground and background colors.
3. The panel also shows the semantic token at the cursor when one exists, and the theme rule that colors the text.
4. When nothing is available, the panel reads No grammar or semantic tokens available.
5. The panel updates as the cursor moves.
6. Pressing Escape closes the panel.

### ED-5.40
1. Opening a very large file shows an information notification that names the file and says tokenization, wrapping, folding, CodeLens, word highlighting and sticky scroll were turned off for it.
2. In that file the listed features are off, so there are no syntax colors, no wrapping, no fold arrows, no CodeLens, no symbol highlighting and no pinned lines.
3. Clicking Forcefully Enable Features in the notification turns `editor.largeFileOptimizations` off and then shows a second notification asking you to reopen the file for the change to take effect.
4. The notification offers a way to stop showing it again.
5. Setting `editor.largeFileOptimizations`, default on, turns large file mode on or off.

### ED-5.41
1. When a file has 1000 or more highlighted odd characters, a one-line banner with a warning icon appears at the top of the editor.
2. The banner text names the most common kind, reading This document contains many non-basic ASCII unicode characters, many ambiguous unicode characters, or many invisible unicode characters.
3. The banner has a link reading Disable Non ASCII Highlight, Disable Ambiguous Highlight or Disable Invisible Highlight, and clicking it turns off that kind of highlighting.
4. The banner has a Close Banner button that hides it until the editor opens another file.

### ED-5.42
1. A line longer than 10000 characters stops being drawn past that point, and this limit is the editor option `stopRenderingLineAfter`, which has no entry in the Settings editor.
2. A line of 20000 characters or more is not syntax colored, and setting `editor.maxTokenizationLineLength`, default 20000, sets that limit.
3. Clicking in the text at or past the drawing cutoff turns the drawing limit off for that editor, so the full lines are drawn.

### ED-5.43
1. Setting `editor.padding.top` sets the empty space in pixels between the top edge of the editor and the first line, from 0 to 1000.
2. Setting `editor.padding.bottom` sets the empty space in pixels between the last line and the bottom edge of the editor, from 0 to 1000.

### ED-5.44
1. The shaded block behind selected text has rounded corners by default.
2. Setting `editor.roundedSelection` to off draws the selection block with square corners.

### ED-5.45
1. Setting `editor.experimentalGpuAcceleration` chooses `off` (the default, page-based drawing) or `on` (GPU drawing of the editor text).
2. When the GPU cannot be set up, a warning offers Use DOM-based rendering, and clicking it sets `editor.experimentalGpuAcceleration` back to off.
3. Running Developer: Debug Editor GPU Renderer (`editor.action.debugEditorGpuRenderer`) from the Command Palette opens a pick list with Log Texture Atlas Stats, Save Texture Atlas and Draw Glyph.
4. Choosing Log Texture Atlas Stats writes the glyph cache statistics to the log.
5. Choosing Save Texture Atlas writes png files of the glyph cache pages into the first workspace folder.
6. Choosing Draw Glyph asks for a character, or a code point prefixed with 0x, and saves a png of that glyph into the first workspace folder.

### ED-5.46
1. Setting `editor.tokenColorCustomizations` in the settings file changes the color and font style of syntax by group, using the names comments, strings, keywords, numbers, types, functions and variables.
2. Its `textMateRules` list changes color and font style for the TextMate scopes you name.
3. Putting the rules under a key in square brackets named for a theme applies them only to that theme, and rules outside such a key apply to every theme.

### ED-5.47
1. Running Developer: Force Retokenize (`editor.action.forceRetokenize`) from the Command Palette throws away the open file's syntax colors and colors the whole file again.
2. The time it took is written to the developer console and nothing is shown on screen.

### ED-5.48
1. Running Start TextMate Syntax Grammar Logging (`editor.action.startDebugTextMate`) from the Command Palette opens a new window on a log file in the temp folder.
2. The same command opens a pinned log tab in the current window.
3. The log tab starts with the lines Open the file you want to test to the side and watch here, and Output mirrored at followed by the log file path.
4. Opening a file beside the log tab makes the grammar tokenizer write its steps into both the tab and the log file, and the tab keeps scrolling to the newest line.

### ED-5.49
1. Certain files show a floating button, or a short row of buttons, in the bottom right corner of the editor.
2. A `.code-workspace` file shows an Open Workspace button, and clicking it opens that workspace (`workbench.action.openWorkspaceFromEditor`).
3. A `launch.json` file shows an Add Configuration button.
4. The user keybindings file shows a Define Keybinding button.
5. A settings sync merge shows a Complete Merge button (`workbench.userDataSync.actions.acceptMerges`). [stock only]
6. The first button is drawn as the primary one and shows its key when it has one.
7. In Cursor each action is drawn as its own click widget (`editor.contrib.floatingClickMenu`), and a multi-button widget also exists. [cursor only]

### ED-5.50
1. The first time you change a file that lives under the `.cursor/worktrees` folder, an Editing in worktree label appears over the editor. [cursor only]
2. The label fades in and then fades out on its own after about three seconds. [cursor only]
3. Opening a file under the worktrees folder without changing it, or changing a file outside that folder, shows no label. [cursor only]

### ED-5.51
1. Turning on `editor.experimental.preferTreeSitter.typescript`, default off, makes TypeScript files take their syntax colors from tree-sitter parsing instead of the TextMate grammar.
2. The same on or off switch exists for CSS, INI and regex as `editor.experimental.preferTreeSitter.css`, `editor.experimental.preferTreeSitter.ini` and `editor.experimental.preferTreeSitter.regex`, each independent of the others.
3. `editor.experimental.asyncTokenization`, default on, runs TextMate tokenization on a web worker so syntax coloring does not hold up typing; turning it off colors on the main thread.
4. In the Settings editor all five settings carry the Experimental tag.

## Unclear

- ED-5.2: the stock source names the position submenu Side, while the feature calls it Position. Piece 5 follows the source.
- ED-5.15: the feature says Remove Manual Folding Ranges removes all hand-made ranges, but the source removes only those that touch the selected lines. Piece 3 follows the source.
- ED-5.35: what Cursor's View > Appearance > Render Whitespace submenu holds is not known because the five set-mode commands are missing from the bundle, so piece 4 is tagged stock only.
- ED-5.49: Complete Merge is tagged stock only because its command id is not in the bundle. It was not checked on screen.
- ED-5.50: the bundle has two contributions with the same text (a centered bubble shown for 2500 ms and a top-right card shown for 3 seconds). Which one shows, or whether both do, was not checked, so the pieces say only that a label appears for about three seconds.
- ED-5.7: Cursor's `markdownEditor.breadcrumbContribution` is a name only, so no piece describes how breadcrumbs look in the Markdown editor.
- ED-5.36 and ED-5.35: word wrap (also ED-2.35) and render whitespace (also ED-2.36) are listed in both ED-5 and ED-2 with the same commands, so one of the two groups should keep these pieces.
