Group: ED-5 Editor view and orientation aids. Section: Editor (05-editor). Date: 2026-10-25. Pass: 2.
Features: 51.

### ED-5.1 Show or hide the minimap
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn the small overview of the whole file at the right edge of the editor on or off from View > Appearance, the Command Palette, or the minimap's right-click menu.
- Commands: editor.action.toggleMinimap
- Menus: MenubarAppearanceMenu (group 4_editor, order 1, title Minimap); CommandPalette; the scrollbar right-click menu built in code (item Minimap)
- Keys: none found
- Settings: editor.minimap.enabled
- Cursor: present
- Source: V\src\vs\workbench\contrib\codeEditor\browser\toggleMinimap.ts; V\src\vs\editor\browser\viewParts\minimap\minimap.ts; V\src\vs\editor\contrib\contextmenu\browser\contextmenu.ts (_showScrollbarContextMenu); V\src\vs\editor\common\config\editorOptions.ts (EditorMinimap); C bundle (id editor.action.toggleMinimap, label "Toggle Minimap", module vs/workbench/contrib/codeEditor/browser/toggleMinimap)

### ED-5.2 Choose the minimap's size, side, slider and detail
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click the scrollbar or minimap to switch Render Characters, Vertical size (Proportional, Fill, Fit), Slider (Mouse Over, Always) and Position (Left, Right); settings add scale, max column and auto-hide (always, on mouse over, or only while scrolling).
- Commands: none found
- Menus: scrollbar right-click menu built in code (items Minimap, Render Characters, Vertical size, Slider, Position)
- Keys: none found
- Settings: editor.minimap.size; editor.minimap.side; editor.minimap.showSlider; editor.minimap.renderCharacters; editor.minimap.scale; editor.minimap.maxColumn; editor.minimap.autohide; editor.minimap
- Cursor: present
- Source: V\src\vs\editor\contrib\contextmenu\browser\contextmenu.ts (context.minimap.* strings); V\src\vs\editor\common\config\editorOptions.ts (EditorMinimap); V\src\vs\editor\browser\viewParts\minimap\minimap.ts; V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (Minimap group); C bundle (labels "Render Characters", "Vertical size", "Proportional", "Mouse Over", "Slider"; settings editor.minimap.size, editor.minimap.side, editor.minimap.showSlider, editor.minimap.autohide)

### ED-5.3 Show section headers in the minimap
- Tag: both
- Evidence: source
- Boundary: no
- What: Comments that start with MARK: (and region markers) are drawn as larger titled labels in the minimap so you can find sections at a glance.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.minimap.showMarkSectionHeaders; editor.minimap.showRegionSectionHeaders; editor.minimap.markSectionHeaderRegex; editor.minimap.sectionHeaderFontSize; editor.minimap.sectionHeaderLetterSpacing
- Cursor: present
- Source: V\src\vs\editor\contrib\sectionHeaders\browser\sectionHeaders.ts (SectionHeaderDetector); V\src\vs\editor\browser\viewParts\minimap\minimap.ts; V\src\vs\editor\common\config\editorOptions.ts (EditorMinimap); C bundle (name sectionHeaderDetector, settings editor.minimap.showMarkSectionHeaders, editor.minimap.showRegionSectionHeaders, editor.minimap.markSectionHeaderRegex)

### ED-5.4 Show or hide sticky scroll
- Tag: both
- Evidence: source
- Boundary: no
- What: Keep the headers of the nested scopes you are inside (class, function, block) pinned at the top of the editor while you scroll; turn it on or off from View > Appearance or the right-click menu on the pinned lines.
- Commands: editor.action.toggleStickyScroll
- Menus: MenubarAppearanceMenu (group 4_editor, order 3, title Sticky Scroll); StickyScrollContext; CommandPalette
- Keys: none found
- Settings: editor.stickyScroll.enabled
- Cursor: present
- Source: V\src\vs\editor\contrib\stickyScroll\browser\stickyScrollActions.ts (toggleStickyScroll); V\src\vs\editor\contrib\stickyScroll\browser\stickyScrollController.ts (StickyScrollContext menu); V\src\vs\editor\contrib\stickyScroll\browser\stickyScrollWidget.ts; C bundle (id editor.action.toggleStickyScroll, label "Toggle Editor Sticky Scroll", setting editor.stickyScroll.enabled)

### ED-5.5 Choose which scopes stick and how many
- Tag: both
- Evidence: source
- Boundary: no
- What: Set the most lines that can stick (default 5), which source of structure decides what sticks (outline, folding ranges or indentation), and whether the pinned lines scroll sideways with the editor.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.stickyScroll.maxLineCount; editor.stickyScroll.defaultModel; editor.stickyScroll.scrollWithEditor
- Cursor: present
- Source: V\src\vs\editor\common\config\editorOptions.ts (EditorStickyScroll); V\src\vs\editor\contrib\stickyScroll\browser\stickyScrollModelProvider.ts; C bundle (settings editor.stickyScroll.maxLineCount, editor.stickyScroll.defaultModel, editor.stickyScroll.scrollWithEditor)

### ED-5.6 Move through sticky scroll lines with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: Focus the pinned lines, move up and down through them, jump to the chosen line with Enter, or press Escape to go back to the text.
- Commands: editor.action.focusStickyScroll; editor.action.selectNextStickyScrollLine; editor.action.selectPreviousStickyScrollLine; editor.action.goToFocusedStickyScrollLine; editor.action.selectEditor
- Menus: CommandPalette (Focus Editor Sticky Scroll)
- Keys: DownArrow (select next line); UpArrow (select previous line); Enter (go to focused line); Escape (select editor); all when the sticky scroll has focus; Focus Editor Sticky Scroll has no default key
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\stickyScroll\browser\stickyScrollActions.ts; C bundle (ids editor.action.focusStickyScroll, editor.action.selectNextStickyScrollLine, editor.action.goToFocusedStickyScrollLine, editor.action.selectEditor, label "Focus Editor Sticky Scroll")

### ED-5.7 Show or hide the breadcrumbs bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Show or hide the path bar above the text that lists the folders, file and the symbols around the cursor, from View > Appearance, the pinned-lines menu, or a key when it is off.
- Commands: breadcrumbs.toggle; breadcrumbs.toggleToOn
- Menus: MenubarAppearanceMenu (group 4_editor, order 2, title Breadcrumbs); StickyScrollContext; NotebookToolbar (group notebookLayout, order 2; notebook side is ED-7); CommandPalette
- Keys: Ctrl+Shift+. for breadcrumbs.toggleToOn when breadcrumbs.enabled is off
- Settings: breadcrumbs.enabled
- Cursor: changed (commands and setting present; the pass 1 menu bar inventory notes the Appearance row is labeled Toggle Breadcrumbs without the toggled mnemonic title)
- Source: V\src\vs\workbench\browser\parts\editor\breadcrumbsControl.ts (breadcrumbs.toggle, toggleToOn); V\src\vs\workbench\browser\parts\editor\breadcrumbs.ts; C bundle (ids breadcrumbs.toggle, breadcrumbs.toggleToOn, label "Toggle Breadcrumbs", setting breadcrumbs.enabled; Cursor-only workbench contribution markdownEditor.breadcrumbContribution)

### ED-5.8 Choose what the breadcrumbs show
- Tag: both
- Evidence: source
- Boundary: no
- What: Show the file path, the symbol path, both or only the last item; sort symbols by position, name or type; hide the icons; and pick which kinds of symbol appear (one on or off setting for each of files, classes, methods, variables and the rest).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: breadcrumbs.filePath; breadcrumbs.symbolPath; breadcrumbs.symbolSortOrder; breadcrumbs.icons; breadcrumbs.showFiles; breadcrumbs.showClasses; breadcrumbs.showMethods; breadcrumbs.showVariables; breadcrumbs.showArrays; breadcrumbs.showBooleans; breadcrumbs.showConstants; breadcrumbs.showConstructors; breadcrumbs.showEnumMembers; breadcrumbs.showEnums; breadcrumbs.showEvents; breadcrumbs.showFields; breadcrumbs.showFunctions; breadcrumbs.showInterfaces; breadcrumbs.showKeys; breadcrumbs.showModules; breadcrumbs.showNamespaces; breadcrumbs.showNull; breadcrumbs.showNumbers; breadcrumbs.showObjects; breadcrumbs.showOperators; breadcrumbs.showPackages; breadcrumbs.showProperties; breadcrumbs.showStrings; breadcrumbs.showStructs; breadcrumbs.showTypeParameters
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\breadcrumbs.ts (the breadcrumbs.* configuration block, 26 show* kinds); V\src\vs\workbench\browser\parts\editor\breadcrumbsModel.ts; C bundle (settings breadcrumbs.filePath, breadcrumbs.symbolPath, breadcrumbs.symbolSortOrder, breadcrumbs.icons, breadcrumbs.showFiles, breadcrumbs.showClasses)

### ED-5.9 Browse from a breadcrumb
- Tag: both
- Evidence: source
- Boundary: no
- What: Click a breadcrumb item, or use the keys, to open a picker of sibling files and folders or of the symbols in the file, and jump there; arrows move along the items, Enter opens, Space reveals without leaving, Escape returns to the text. A setting swaps the picker for a Quick Pick.
- Commands: breadcrumbs.focusAndSelect; breadcrumbs.focus; breadcrumbs.focusNext; breadcrumbs.focusPrevious; breadcrumbs.focusNextWithPicker; breadcrumbs.focusPreviousWithPicker; breadcrumbs.selectFocused; breadcrumbs.revealFocused; breadcrumbs.revealFocusedFromTreeAside; breadcrumbs.selectEditor
- Menus: CommandPalette (Focus and Select Breadcrumbs, Focus Breadcrumbs)
- Keys: Ctrl+Shift+. (focus and select); Ctrl+Shift+; (focus); RightArrow; LeftArrow; Ctrl+RightArrow and Ctrl+LeftArrow (Alt on macOS) to move and open the picker; Enter; Space; Ctrl+Enter (open to the side from the tree); Escape; the item keys apply when breadcrumbs are visible and active
- Settings: breadcrumbs.useQuickPick
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\breadcrumbsControl.ts (command registrations and keybindings); V\src\vs\workbench\browser\parts\editor\breadcrumbsPicker.ts; V\src\vs\workbench\browser\parts\editor\breadcrumbs.ts (UseQuickPick); C bundle (ids breadcrumbs.focusAndSelect, breadcrumbs.focus, breadcrumbs.focusNext, breadcrumbs.focusPreviousWithPicker, breadcrumbs.selectFocused, breadcrumbs.revealFocused, breadcrumbs.selectEditor, label "Focus and Select Breadcrumbs", setting breadcrumbs.useQuickPick)

### ED-5.10 Copy the breadcrumbs path
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Right-click an editor tab and choose Copy Breadcrumbs Path to copy the symbol path at the cursor, joined with a separator you can set.
- Commands: breadcrumbs.copyPath
- Menus: EditorTitleContext (group 1_cutcopypaste, order 100, when breadcrumbs are possible and have symbols); CommandPalette
- Keys: none found
- Settings: breadcrumbs.symbolPathSeparator
- Cursor: missing (id breadcrumbs.copyPath, label "Copy Breadcrumbs Path" and setting breadcrumbs.symbolPathSeparator are not in the bundle)
- Source: V\src\vs\workbench\browser\parts\editor\breadcrumbsControl.ts (breadcrumbs.copyPath); V\src\vs\workbench\browser\parts\editor\breadcrumbs.ts (SymbolPathSeparator)

### ED-5.11 Fold or unfold the region at the cursor
- Tag: both
- Evidence: source
- Boundary: no
- What: Collapse or expand the block the cursor is in, one level or all the way down, so you can hide code you are not reading.
- Commands: editor.fold; editor.unfold; editor.toggleFold; editor.foldRecursively; editor.unfoldRecursively; editor.toggleFoldRecursively
- Menus: CommandPalette
- Keys: Ctrl+Shift+[ (fold; Cmd+Alt+[ on macOS); Ctrl+Shift+] (unfold; Cmd+Alt+] on macOS); Ctrl+K Ctrl+L (toggle fold); Ctrl+K Ctrl+[ (fold recursively); Ctrl+K Ctrl+] (unfold recursively); Ctrl+K Ctrl+Shift+L (toggle fold recursively); all when the editor text has focus and folding is on
- Settings: editor.folding
- Cursor: present
- Source: V\src\vs\editor\contrib\folding\browser\folding.ts (FoldAction, UnfoldAction, ToggleFoldAction and the recursive variants); C bundle (ids editor.fold, editor.unfold, editor.toggleFold, editor.foldRecursively, editor.unfoldRecursively, editor.toggleFoldRecursively, module vs/editor/contrib/folding/browser/folding)

### ED-5.12 Fold or unfold every region
- Tag: both
- Evidence: source
- Boundary: no
- What: Collapse or expand all regions in the file at once, or all except the one you have selected.
- Commands: editor.foldAll; editor.unfoldAll; editor.foldAllExcept; editor.unfoldAllExcept
- Menus: CommandPalette
- Keys: Ctrl+K Ctrl+0 (fold all); Ctrl+K Ctrl+J (unfold all); Ctrl+K Ctrl+- (fold all except selected); Ctrl+K Ctrl+= (unfold all except selected); editor text focus
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\folding\browser\folding.ts (FoldAllAction, UnfoldAllAction, FoldAllExceptAction, UnfoldAllExceptAction); C bundle (ids editor.foldAll, editor.unfoldAll, editor.foldAllExcept, editor.unfoldAllExcept, label "Fold All Except Selected")

### ED-5.13 Fold to a chosen level
- Tag: both
- Evidence: source
- Boundary: no
- What: Collapse every region nested at depth 1 through 7 in one step, which gives a quick outline of the file at the depth you pick.
- Commands: editor.foldLevel1 to editor.foldLevel7 (ids are built as editor.foldLevel plus the number)
- Menus: CommandPalette (labels Fold Level 1 to Fold Level 7)
- Keys: Ctrl+K Ctrl+1 to Ctrl+K Ctrl+7; editor text focus
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\folding\browser\folding.ts (FoldLevelAction, loop for levels 1 to 7); C bundle (prefix editor.foldLevel, label "Fold Level")

### ED-5.14 Fold block comments and marker regions
- Tag: both
- Evidence: source
- Boundary: no
- What: Collapse all block comments in one step, or collapse and expand every region marked with region start and end comments.
- Commands: editor.foldAllBlockComments; editor.foldAllMarkerRegions; editor.unfoldAllMarkerRegions
- Menus: CommandPalette
- Keys: Ctrl+K Ctrl+/ (fold all block comments); Ctrl+K Ctrl+8 (fold all regions); Ctrl+K Ctrl+9 (unfold all regions); editor text focus
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\folding\browser\folding.ts (FoldAllBlockCommentsAction, FoldAllRegionsAction, UnfoldAllRegionsAction); C bundle (ids editor.foldAllBlockComments, editor.foldAllMarkerRegions, editor.unfoldAllMarkerRegions)

### ED-5.15 Create or remove your own folding ranges
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn the selected lines into a folding range of your own (shown with its own marker), or remove all the ranges you made by hand.
- Commands: editor.createFoldingRangeFromSelection; editor.removeManualFoldingRanges
- Menus: CommandPalette
- Keys: Ctrl+K Ctrl+, (create from selection); Ctrl+K Ctrl+. (remove manual ranges); editor text focus
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\folding\browser\folding.ts (FoldRangeFromSelectionAction, RemoveFoldRangeFromSelectionAction); V\src\vs\editor\contrib\folding\browser\foldingDecorations.ts; C bundle (ids editor.createFoldingRangeFromSelection, editor.removeManualFoldingRanges, label "Create Folding Range from Selection", module vs/editor/contrib/folding/browser/foldingDecorations)

### ED-5.16 Jump between folds
- Tag: both
- Evidence: source
- Boundary: no
- What: Move the cursor to the start of the next or previous folding range, or up to the fold that contains the current line.
- Commands: editor.gotoNextFold; editor.gotoPreviousFold; editor.gotoParentFold
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\folding\browser\folding.ts (GotoNextFoldAction, GotoPreviousFoldAction, GotoParentFoldAction); C bundle (ids editor.gotoNextFold, editor.gotoPreviousFold, editor.gotoParentFold)

### ED-5.17 Fold import blocks
- Tag: both
- Evidence: source
- Boundary: no
- What: Collapse or expand the block of import lines at the top of a file, and optionally start with imports collapsed whenever a file opens.
- Commands: editor.toggleImportFold
- Menus: CommandPalette
- Keys: none found
- Settings: editor.foldingImportsByDefault
- Cursor: present
- Source: V\src\vs\editor\contrib\folding\browser\folding.ts (ToggleImportFoldAction, foldedImports); C bundle (id editor.toggleImportFold, label "Toggle Import Fold", option name foldingImportsByDefault)

### ED-5.18 Folding gutter arrows and clicks
- Tag: both
- Evidence: source
- Boundary: no
- What: Arrows in the gutter next to line numbers fold and unfold regions; choose whether they always show or only when the mouse is over the gutter, whether the active region is highlighted, and whether clicking the "..." after a folded line unfolds it. Shift-click or middle-click acts on nested regions, and Alt-click folds the others around it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.showFoldingControls; editor.foldingHighlight; editor.unfoldOnClickAfterEndOfLine
- Cursor: present
- Source: V\src\vs\editor\contrib\folding\browser\folding.ts (onEditorMouseDown, unfoldOnClickAfterEndOfLine); V\src\vs\editor\contrib\folding\browser\foldingDecorations.ts; V\src\vs\editor\contrib\folding\browser\folding.css; C bundle (option names showFoldingControls, foldingHighlight, unfoldOnClickAfterEndOfLine)

### ED-5.19 Choose how fold ranges are found
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn folding on or off, choose whether regions come from the language or from indentation, set the most regions per file, and pick a default folding provider from an installed language extension.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.folding; editor.foldingStrategy; editor.foldingMaximumRegions; editor.defaultFoldingRangeProvider
- Cursor: present
- Source: V\src\vs\editor\common\config\editorOptions.ts (folding, foldingStrategy, foldingMaximumRegions); V\src\vs\workbench\contrib\folding\browser\folding.contribution.ts (DefaultFoldingRangeProvider); V\src\vs\editor\contrib\folding\browser\syntaxRangeProvider.ts; V\src\vs\editor\contrib\folding\browser\indentRangeProvider.ts; C bundle (settings editor.folding, editor.foldingStrategy, editor.foldingMaximumRegions, editor.defaultFoldingRangeProvider)

### ED-5.20 Line numbers and gutter width
- Tag: both
- Evidence: source
- Boundary: no
- What: Show line numbers as absolute, relative to the cursor, every tenth line, or off; also set the narrowest width of the number column, choose whether the last empty line after a final newline gets a number (on, off or dimmed), and keep the icon margin on or off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.lineNumbers; editor.lineNumbersMinChars; editor.renderFinalNewline; editor.glyphMargin
- Cursor: present
- Source: V\src\vs\editor\browser\viewParts\lineNumbers\lineNumbers.ts; V\src\vs\editor\browser\viewParts\glyphMargin\glyphMargin.ts; V\src\vs\editor\browser\viewParts\linesDecorations\linesDecorations.ts; V\src\vs\editor\common\config\editorOptions.ts (lineNumbers, lineNumbersMinChars, renderFinalNewline, glyphMargin); C bundle (setting editor.lineNumbers; option names lineNumbersMinChars, renderFinalNewline, glyphMargin)

### ED-5.21 Vertical rulers
- Tag: both
- Evidence: source
- Boundary: no
- What: Draw thin vertical lines at chosen columns (for example 80 and 120), each with an optional color, to show line length limits.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.rulers
- Cursor: present
- Source: V\src\vs\editor\browser\viewParts\rulers\rulers.ts; V\src\vs\editor\common\config\editorOptions.ts (rulers); C bundle (setting editor.rulers)

### ED-5.22 Highlight the current line
- Tag: both
- Evidence: source
- Boundary: no
- What: Tint the line the cursor is on, the gutter beside it, both, or neither, and optionally only while the editor has focus.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.renderLineHighlight; editor.renderLineHighlightOnlyWhenFocus
- Cursor: present
- Source: V\src\vs\editor\browser\viewParts\currentLineHighlight\currentLineHighlight.ts; V\src\vs\editor\common\config\editorOptions.ts (renderLineHighlight, renderLineHighlightOnlyWhenFocus); C bundle (option names renderLineHighlight, renderLineHighlightOnlyWhenFocus)

### ED-5.23 Indent and bracket pair guides
- Tag: both
- Evidence: source
- Boundary: no
- What: Thin vertical lines show each indent level, the active level can be emphasized, and extra guides join the two ends of the bracket pair around the cursor (vertical, and optionally horizontal).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.guides.indentation; editor.guides.highlightActiveIndentation; editor.guides.bracketPairs; editor.guides.bracketPairsHorizontal; editor.guides.highlightActiveBracketPair; editor.guides
- Cursor: present
- Source: V\src\vs\editor\browser\viewParts\indentGuides\indentGuides.ts; V\src\vs\editor\common\config\editorOptions.ts (EditorGuides); C bundle (settings editor.guides.indentation, editor.guides.highlightActiveIndentation, editor.guides.bracketPairs, editor.guides.bracketPairsHorizontal, editor.guides.highlightActiveBracketPair)

### ED-5.24 Highlight the matching bracket
- Tag: both
- Evidence: source
- Boundary: no
- What: When the cursor is next to a bracket, it and its partner get a box so you can see where the pair ends; choose always, only when the cursor is touching it, or never. The Go to Bracket commands belong to ED-2.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.matchBrackets; editor.language.brackets
- Cursor: present
- Source: V\src\vs\editor\contrib\bracketMatching\browser\bracketMatching.ts (BracketMatchingController decorations); V\src\vs\editor\common\config\editorOptions.ts (matchBrackets); C bundle (id editor.contrib.bracketMatchingController, option name matchBrackets)

### ED-5.25 Color bracket pairs
- Tag: both
- Evidence: source
- Boundary: no
- What: Matching brackets are colored by nesting depth so pairs are easy to follow; the colors can cycle together or separately for each bracket type.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.bracketPairColorization.enabled; editor.bracketPairColorization.independentColorPoolPerBracketType; editor.bracketPairColorization; editor.language.colorizedBracketPairs
- Cursor: present
- Source: V\src\vs\editor\common\model\bracketPairsTextModelPart\colorizedBracketPairsDecorationProvider.ts; V\src\vs\editor\common\model\bracketPairsTextModelPart\bracketPairsImpl.ts; V\src\vs\editor\common\config\editorOptions.ts (bracketPairColorization); C bundle (settings editor.bracketPairColorization.enabled, editor.bracketPairColorization.independentColorPoolPerBracketType)

### ED-5.26 Smooth scrolling and wheel speed
- Tag: both
- Evidence: source
- Boundary: no
- What: Animate scrolling, set how far one wheel notch moves the text, how much faster it moves while holding Alt, whether a diagonal scroll locks to one axis, and (in stock) make touchpad scrolling inertial.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.smoothScrolling; editor.mouseWheelScrollSensitivity; editor.fastScrollSensitivity; editor.scrollPredominantAxis; editor.inertialScroll
- Cursor: changed (smoothScrolling, mouseWheelScrollSensitivity, fastScrollSensitivity and scrollPredominantAxis are in the bundle; inertialScroll is not)
- Source: V\src\vs\editor\common\config\editorOptions.ts (smoothScrolling, mouseWheelScrollSensitivity, fastScrollSensitivity, scrollPredominantAxis, inertialScroll); V\src\vs\editor\browser\controller\mouseHandler.ts; C bundle (settings editor.mouseWheelScrollSensitivity, editor.fastScrollSensitivity; option names smoothScrolling, scrollPredominantAxis)

### ED-5.27 Use the middle mouse button in the editor
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Press the middle button to start auto-scrolling (a dot appears and the text moves as you drag away from it), or choose that a middle click opens links or acts like Ctrl+click.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.scrollOnMiddleClick; editor.mouseMiddleClickAction
- Cursor: missing (neither setting name nor the middleScroll contribution is in the bundle)
- Source: V\src\vs\editor\contrib\middleScroll\browser\middleScrollController.ts; V\src\vs\editor\contrib\middleScroll\browser\middleScroll.css; V\src\vs\editor\common\config\editorOptions.ts (scrollOnMiddleClick, mouseMiddleClickAction)

### ED-5.28 Scroll past the last line
- Tag: both
- Evidence: source
- Boundary: no
- What: Let the last line of the file scroll up toward the top of the editor, leaving empty space below it, and let the view scroll a few columns past the end of the longest line.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.scrollBeyondLastLine; editor.scrollBeyondLastColumn
- Cursor: present
- Source: V\src\vs\editor\common\config\editorOptions.ts (scrollBeyondLastLine, scrollBeyondLastColumn); V\src\vs\editor\common\viewLayout\viewLayout.ts; C bundle (setting editor.scrollBeyondLastLine; option name scrollBeyondLastColumn)

### ED-5.29 Scrollbars and the overview ruler
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose when the vertical and horizontal scrollbars show, how thick they are, whether a click scrolls by page, and whether the strip beside the scrollbar (the overview ruler) shows a border or a mark for the cursor. Right-clicking the scrollbar opens the minimap menu from ED-5.2.
- Commands: none found
- Menus: scrollbar right-click menu built in code
- Keys: none found
- Settings: editor.scrollbar.vertical; editor.scrollbar.horizontal; editor.scrollbar.verticalScrollbarSize; editor.scrollbar.horizontalScrollbarSize; editor.scrollbar.scrollByPage; editor.scrollbar.ignoreHorizontalScrollbarInContentHeight; editor.hideCursorInOverviewRuler; editor.overviewRulerBorder; editor.scrollbar
- Cursor: present
- Source: V\src\vs\editor\browser\viewParts\editorScrollbar\editorScrollbar.ts; V\src\vs\editor\browser\viewParts\overviewRuler\overviewRuler.ts; V\src\vs\editor\common\config\editorOptions.ts (EditorScrollbar, hideCursorInOverviewRuler, overviewRulerBorder); C bundle (settings editor.scrollbar.vertical, editor.scrollbar.horizontal, editor.scrollbar.verticalScrollbarSize, editor.scrollbar.scrollByPage; option names hideCursorInOverviewRuler, overviewRulerBorder)

### ED-5.30 Scroll with keyboard commands
- Tag: both
- Evidence: source
- Boundary: no
- What: Scroll the view by a line or a page, or to the top, bottom, left or right, without moving the cursor; one more command reveals a given line with the position you choose.
- Commands: scrollLineUp; scrollLineDown; scrollPageUp; scrollPageDown; scrollEditorTop; scrollEditorBottom; scrollLeft; scrollRight; revealLine; editorScroll
- Menus: none found
- Keys: Ctrl+Up (macOS Ctrl+PageUp) for scrollLineUp; Ctrl+Down (macOS Ctrl+PageDown) for scrollLineDown; Alt+PageUp (macOS Cmd+PageUp) for scrollPageUp; Alt+PageDown (macOS Cmd+PageDown) for scrollPageDown; the others have no default key
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\browser\coreCommands.ts (scroll commands near the scrollLineUp, scrollPageUp, scrollEditorTop, scrollLeft and revealLine registrations); C bundle (ids scrollLineUp, scrollLineDown, scrollPageUp, scrollPageDown, scrollEditorTop, scrollEditorBottom, scrollLeft, scrollRight, revealLine, editorScroll)

### ED-5.31 Choose the cursor style
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick the cursor shape (line, block, underline, thin line, outline block, thin underline), its width, how it blinks (blink, smooth, phase, expand, solid), whether it glides between positions, and its shape while overtype is on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.cursorStyle; editor.cursorWidth; editor.cursorBlinking; editor.cursorSmoothCaretAnimation; editor.overtypeCursorStyle; editor.cursorHeight
- Cursor: present
- Source: V\src\vs\editor\browser\viewParts\viewCursors\viewCursors.ts; V\src\vs\editor\common\config\editorOptions.ts (cursorStyle, cursorWidth, cursorBlinking, cursorSmoothCaretAnimation, overtypeCursorStyle); C bundle (settings editor.cursorStyle, editor.cursorWidth, editor.cursorBlinking; option names cursorSmoothCaretAnimation, overtypeCursorStyle)

### ED-5.32 Keep lines around the cursor
- Tag: both
- Evidence: source
- Boundary: no
- What: Always keep a set number of lines visible above and below the cursor as you move, either only when moving by keyboard or always.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.cursorSurroundingLines; editor.cursorSurroundingLinesStyle
- Cursor: present
- Source: V\src\vs\editor\common\config\editorOptions.ts (cursorSurroundingLines, cursorSurroundingLinesStyle); V\src\vs\editor\browser\viewParts\viewCursors\viewCursors.ts; C bundle (option names cursorSurroundingLines, cursorSurroundingLinesStyle)

### ED-5.33 Highlight other uses of a symbol and jump between them
- Tag: both
- Evidence: source
- Boundary: no
- What: Put the cursor on a name and the other places it appears are shaded (reads in one color, writes in another); jump to the next or previous highlight, or trigger the highlight by hand.
- Commands: editor.action.wordHighlight.next; editor.action.wordHighlight.prev; editor.action.wordHighlight.trigger
- Menus: CommandPalette
- Keys: F7 (next); Shift+F7 (previous); Trigger Symbol Highlight has no default key; editor text focus
- Settings: editor.occurrencesHighlight; editor.occurrencesHighlightDelay
- Cursor: present
- Source: V\src\vs\editor\contrib\wordHighlighter\browser\wordHighlighter.ts; V\src\vs\editor\common\config\editorOptions.ts (occurrencesHighlight, occurrencesHighlightDelay); C bundle (ids editor.action.wordHighlight.next, editor.action.wordHighlight.prev, editor.action.wordHighlight.trigger, label "Go to Next Symbol Highlight", id editor.contrib.wordHighlighter)

### ED-5.34 Highlight copies of the selected text
- Tag: both
- Evidence: source
- Boundary: no
- What: Select a word or phrase and every other copy in the file is shaded, with a mark in the scrollbar; settings cap the length of text that triggers it and allow it for multi-line selections.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.selectionHighlight; editor.selectionHighlightMaxLength; editor.selectionHighlightMultiline
- Cursor: changed (editor.selectionHighlight and the contribution editor.contrib.selectionHighlighter are in the bundle; selectionHighlightMaxLength and selectionHighlightMultiline are not)
- Source: V\src\vs\editor\contrib\multicursor\browser\multicursor.ts (SelectionHighlighter); V\src\vs\editor\common\config\editorOptions.ts (selectionHighlight, selectionHighlightMaxLength, selectionHighlightMultiline); C bundle (id editor.contrib.selectionHighlighter, setting editor.selectionHighlight)

### ED-5.35 Show whitespace and control characters
- Tag: both
- Evidence: source
- Boundary: no
- What: Draw dots and arrows for spaces and tabs (never, at word boundaries, in the selection, at line ends only, or everywhere) and show control characters such as null as visible symbols; switch from View > Appearance or the Command Palette.
- Commands: editor.action.toggleRenderWhitespace; editor.action.renderWhitespace.none; editor.action.renderWhitespace.boundary; editor.action.renderWhitespace.selection; editor.action.renderWhitespace.trailing; editor.action.renderWhitespace.all; editor.action.toggleRenderControlCharacter
- Menus: MenubarAppearanceMenu (group 4_editor, order 4, submenu Render Whitespace; order 5, Render Control Characters); EditorRenderWhitespaceSubmenu (group 1_config, orders 1 to 5: None, Boundary, Selection, Trailing, All); CommandPalette (the two toggle commands)
- Keys: none found
- Settings: editor.renderWhitespace; editor.renderControlCharacters; editor.experimentalWhitespaceRendering
- Cursor: changed (the two toggle commands, editor.renderWhitespace and editor.renderControlCharacters are present; the five editor.action.renderWhitespace.* commands and their "Set Render Whitespace to" labels are not in the bundle)
- Source: V\src\vs\workbench\contrib\codeEditor\browser\toggleRenderWhitespace.ts; V\src\vs\workbench\contrib\codeEditor\browser\toggleRenderControlCharacter.ts; V\src\vs\editor\browser\viewParts\whitespace\whitespace.ts; V\src\vs\editor\common\config\editorOptions.ts (renderWhitespace, renderControlCharacters, experimentalWhitespaceRendering); C bundle (ids editor.action.toggleRenderWhitespace, editor.action.toggleRenderControlCharacter, labels "Toggle Render Whitespace", "Render Whitespace", "Toggle Control Characters")

### ED-5.36 Wrap long lines
- Tag: both
- Evidence: source
- Boundary: no
- What: Toggle word wrap for the current file with Alt+Z or View > Word Wrap, or let a title bar button turn it on or off for files dominated by very long lines; settings choose off, on, at a fixed column, or the smaller of the window and the column, plus indent of wrapped lines.
- Commands: editor.action.toggleWordWrap
- Menus: MenubarViewMenu (group 6_editor, order 1, title Word Wrap); EditorTitle (group navigation, order 1, "Enable wrapping for this file" and "Disable wrapping for this file", for long-line files); CommandPalette (View: Toggle Word Wrap)
- Keys: Alt+Z
- Settings: editor.wordWrap; editor.wordWrapColumn; editor.wrappingIndent; editor.wrappingStrategy; editor.wordBreak; editor.wordWrapOverride2; editor.wrapOnEscapedLineFeeds
- Cursor: present
- Source: V\src\vs\workbench\contrib\codeEditor\browser\toggleWordWrap.ts; V\src\vs\editor\common\config\editorOptions.ts (wordWrap, wordWrapColumn, wrappingIndent, wrappingStrategy, wordBreak); C bundle (id editor.action.toggleWordWrap, labels "View: Toggle Word Wrap", "Enable wrapping for this file", "Disable wrapping for this file", option names wordWrap, wordWrapColumn, wrappingIndent)

### ED-5.37 Highlight confusing Unicode characters
- Tag: both
- Evidence: source
- Boundary: no
- What: Characters that look like others, invisible characters and non-basic ASCII characters get a box; hovering one offers to stop highlighting that character, that kind, comments, or strings, and settings keep allow lists by character or locale.
- Commands: editor.action.unicodeHighlight.disableHighlightingOfAmbiguousCharacters; editor.action.unicodeHighlight.disableHighlightingOfInvisibleCharacters; editor.action.unicodeHighlight.disableHighlightingOfNonBasicAsciiCharacters; editor.action.unicodeHighlight.disableHighlightingInComments; editor.action.unicodeHighlight.disableHighlightingInStrings; editor.action.unicodeHighlight.showExcludeOptions
- Menus: none found (the actions are offered inside the hover)
- Keys: none found
- Settings: editor.unicodeHighlight.nonBasicASCII; editor.unicodeHighlight.invisibleCharacters; editor.unicodeHighlight.ambiguousCharacters; editor.unicodeHighlight.includeComments; editor.unicodeHighlight.includeStrings; editor.unicodeHighlight.allowedCharacters; editor.unicodeHighlight.allowedLocales; editor.unicodeHighlight
- Cursor: present
- Source: V\src\vs\editor\contrib\unicodeHighlighter\browser\unicodeHighlighter.ts; V\src\vs\editor\common\config\editorOptions.ts (unicodeHighlight); C bundle (the six editor.action.unicodeHighlight.* ids, label "Disable highlighting of ambiguous characters", module vs/editor/contrib/unicodeHighlighter/browser/unicodeHighlighter)

### ED-5.38 Change the editor font, size and spacing
- Tag: both
- Evidence: source
- Boundary: no
- What: Set the editor font, size, weight, ligatures, line height and letter spacing, and zoom the text with Increase, Decrease and Reset Editor Font Size or by holding Ctrl and turning the mouse wheel. Whole-window zoom is TB-3.
- Commands: editor.action.fontZoomIn; editor.action.fontZoomOut; editor.action.fontZoomReset
- Menus: CommandPalette
- Keys: none found
- Settings: editor.fontSize; editor.fontFamily; editor.fontWeight; editor.fontLigatures; editor.lineHeight; editor.letterSpacing; editor.mouseWheelZoom; editor.allowVariableFonts; editor.allowVariableFontsInAccessibilityMode; editor.allowVariableLineHeights; editor.fontVariations
- Cursor: present
- Source: V\src\vs\editor\contrib\fontZoom\browser\fontZoom.ts; V\src\vs\editor\browser\controller\mouseHandler.ts (mouseWheelZoom); V\src\vs\editor\common\config\editorOptions.ts (fontSize, fontFamily, fontWeight, fontLigatures, lineHeight, letterSpacing, mouseWheelZoom); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (Font group); C bundle (ids editor.action.fontZoomIn, editor.action.fontZoomOut, editor.action.fontZoomReset, label "Increase Editor Font Size", settings editor.fontSize, editor.fontFamily; option names mouseWheelZoom, letterSpacing)

### ED-5.39 Inspect tokens and scopes
- Tag: both
- Evidence: source
- Boundary: no
- What: A developer command that opens a small panel next to the cursor showing the syntax token and scope details for the text at the cursor (TextMate token, semantic token, and the theme rule that colors it); it updates as the cursor moves and Escape closes it.
- Commands: editor.action.inspectTMScopes
- Menus: CommandPalette (Developer: Inspect Editor Tokens and Scopes)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\codeEditor\browser\inspectEditorTokens\inspectEditorTokens.ts; C bundle (id editor.action.inspectTMScopes, label "Developer: Inspect Editor Tokens and Scopes", module vs/workbench/contrib/codeEditor/browser/inspectEditorTokens/inspectEditorTokens)

### ED-5.40 Large file mode
- Tag: both
- Evidence: source
- Boundary: no
- What: When a file is very large the editor turns off tokenization, wrapping, folding, CodeLens, word highlighting and sticky scroll to stay responsive, and shows a notification that names the file with a Forcefully Enable Features button.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.largeFileOptimizations
- Cursor: present
- Source: V\src\vs\workbench\contrib\codeEditor\browser\largeFileOptimizations.ts (LargeFileOptimizationsWarner); V\src\vs\editor\common\model\textModel.ts (isTooLargeForTokenization, isTooLargeForSyncing); C bundle (id editor.contrib.largeFileOptimizationsWarner, label "Forcefully Enable Features", setting editor.largeFileOptimizations)

### ED-5.41 Warning banner for files with many odd Unicode characters
- Tag: both
- Evidence: source
- Boundary: no
- What: When a file has 1000 or more highlighted odd characters (non-basic ASCII, ambiguous or invisible), a one-line banner with a warning icon appears at the top of the editor and names the most common kind, with a link that turns that kind of highlighting off and a close button that hides the banner until the editor opens another file.
- Commands: editor.action.unicodeHighlight.disableHighlightingOfNonBasicAsciiCharacters; editor.action.unicodeHighlight.disableHighlightingOfAmbiguousCharacters; editor.action.unicodeHighlight.disableHighlightingOfInvisibleCharacters (the banner link runs one of these; they are also named in ED-5.37)
- Menus: none found (the link and the close button live inside the banner)
- Keys: none found
- Settings: editor.unicodeHighlight.nonBasicASCII; editor.unicodeHighlight.ambiguousCharacters; editor.unicodeHighlight.invisibleCharacters
- Cursor: present (id unicodeHighlightBanner, the banner.close action and the three "This document contains many ..." messages are in the bundle)
- Source: V\src\vs\editor\contrib\unicodeHighlighter\browser\bannerController.ts; V\src\vs\editor\contrib\unicodeHighlighter\browser\unicodeHighlighter.ts (_updateState); V\src\vs\editor\common\services\unicodeTextModelHighlighter.ts (MAX_RESULT_LENGTH 1000 sets hasMore); C bundle (id unicodeHighlightBanner, action banner.close, nls messages "This document contains many non-basic ASCII unicode characters", "This document contains many ambiguous unicode characters", "This document contains many invisible unicode characters")

### ED-5.42 Stop drawing and coloring very long lines
- Tag: both
- Evidence: source
- Boundary: no
- What: A line longer than 10000 characters stops being drawn past that point, and a line of 20000 characters or more is not syntax colored, so files with huge single lines stay responsive; clicking in the text past the cutoff draws the full line again. The hover notice about this is part of ED-4.1.
- Commands: none found (the Don't Show Again link in the hover runs editor.action.hideLongLineWarningHover, named in ED-4.1)
- Menus: none found
- Keys: none found
- Settings: editor.maxTokenizationLineLength; editor.stopRenderingLineAfter (an editor option with no Settings editor entry, named in the hover text)
- Cursor: present (the key editor.maxTokenizationLineLength with default 20000, the option name stopRenderingLineAfter with default 10000 and the id editor.contrib.longLinesHelper are in the bundle)
- Source: V\src\vs\editor\contrib\longLinesHelper\browser\longLinesHelper.ts; V\src\vs\editor\common\config\editorOptions.ts (stopRenderingLineAfter, default 10000); V\src\vs\editor\common\config\editorConfigurationSchema.ts (maxTokenizationLineLength, default 20000); V\src\vs\workbench\services\textMate\browser\tokenizationSupport\tokenizationSupportWithLineLimit.ts; V\src\vs\editor\contrib\hover\browser\markdownHoverParticipant.ts (the hover notice); C bundle (key editor.maxTokenizationLineLength, option stopRenderingLineAfter, id editor.contrib.longLinesHelper)

### ED-5.43 Add space above the first line and below the last line
- Tag: both
- Evidence: source
- Boundary: no
- What: Set a number of pixels of empty space, from 0 to 1000, between the top edge of the editor and the first line, and another between the last line and the bottom edge.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.padding.top; editor.padding.bottom; editor.padding
- Cursor: present (both setting ids with the same 0 to 1000 range are in the bundle)
- Source: V\src\vs\editor\common\config\editorOptions.ts (EditorPadding); V\src\vs\editor\common\viewLayout\viewLayout.ts (padding passed to the lines layout); C bundle (ids editor.padding.top, editor.padding.bottom, nls "Controls the amount of space between the top edge of the editor and the first line.")

### ED-5.44 Round the corners of selections
- Tag: both
- Evidence: source
- Boundary: no
- What: The shaded block behind selected text has rounded corners by default, and a setting switches to square corners.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.roundedSelection
- Cursor: present (the option name roundedSelection, default true, and its description "Controls whether selections should have rounded corners." are in the bundle)
- Source: V\src\vs\editor\browser\viewParts\selections\selections.ts; V\src\vs\editor\common\config\editorOptions.ts (roundedSelection); C bundle (option name roundedSelection, nls "Controls whether selections should have rounded corners.")

### ED-5.45 Draw the editor with the GPU (experimental)
- Tag: both
- Evidence: source
- Boundary: no
- What: An experimental setting switches the editor from the usual page-based text drawing (off, the default) to GPU drawing, with a warning that offers Use DOM-based rendering if the GPU cannot be set up, and a developer command offers three glyph cache tools: Log Texture Atlas Stats, Save Texture Atlas (writes png files into the first workspace folder) and Draw Glyph.
- Commands: editor.action.debugEditorGpuRenderer
- Menus: CommandPalette (Developer: Debug Editor GPU Renderer)
- Keys: none found
- Settings: editor.experimentalGpuAcceleration
- Cursor: present (the key editor.experimentalGpuAcceleration, the id editor.action.debugEditorGpuRenderer, the label "Developer: Debug Editor GPU Renderer", the pick labels "Save Texture Atlas" and "Draw Glyph", and the code that sets the key back to off from the warning prompt are in the bundle)
- Source: V\src\vs\editor\common\config\editorOptions.ts (experimentalGpuAcceleration, tagged experimental); V\src\vs\editor\browser\gpu\viewGpuContext.ts (device request and the Use DOM-based rendering prompt); V\src\vs\editor\contrib\gpu\browser\gpuActions.ts (DebugEditorGpuRendererAction); C bundle (key editor.experimentalGpuAcceleration, id editor.action.debugEditorGpuRenderer, nls module vs/editor/contrib/gpu/browser/gpuActions)

### ED-5.46 Override syntax colors for token scopes
- Tag: both
- Evidence: source
- Boundary: no
- What: Change the color and font style of syntax in the editor from the settings file, either by group (comments, strings, keywords, numbers, types, functions, variables) or by TextMate scope rules, for every theme or only for one named theme; the scope names to use can be read with ED-5.39, and the semantic version of this setting is in ED-4.23.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.tokenColorCustomizations
- Cursor: present (the key is in the bundle's theme settings constants next to editor.semanticTokenColorCustomizations)
- Source: V\src\vs\workbench\services\themes\common\themeConfiguration.ts (tokenColorSchema, tokenColorCustomizationSchema, tokenColorCustomizations getter); V\src\vs\workbench\services\themes\common\colorThemeData.ts; C bundle (constant TOKEN_COLOR_CUSTOMIZATIONS = "editor.tokenColorCustomizations")

### ED-5.47 Force syntax coloring to run again (developer)
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Developer: Force Retokenize from the Command Palette to throw away the open file's syntax colors and color the whole file again; the time it took is written to the developer console, not shown on screen.
- Commands: editor.action.forceRetokenize
- Menus: CommandPalette (Developer: Force Retokenize)
- Keys: none found
- Settings: none found
- Cursor: present (id editor.action.forceRetokenize and label "Developer: Force Retokenize" are in the bundle)
- Source: V\src\vs\editor\contrib\tokenization\browser\tokenization.ts (ForceRetokenizeAction); C bundle (id editor.action.forceRetokenize, label "Developer: Force Retokenize", nls module vs/editor/contrib/tokenization/browser/tokenization)

### ED-5.48 Log TextMate grammar activity (developer)
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Start TextMate Syntax Grammar Logging from the Command Palette to open a log tab and a second window on the mirrored log file in the temp folder, then open the file to test beside it and watch the grammar tokenizer write its steps into both.
- Commands: editor.action.startDebugTextMate
- Menus: CommandPalette (category Developer)
- Keys: none found
- Settings: none found
- Cursor: present (id editor.action.startDebugTextMate and title "Start TextMate Syntax Grammar Logging" with category Developer are in the bundle; the nls module is vs/workbench/contrib/codeEditor/electron-sandbox/startDebugTextMate, a desktop-only command)
- Source: V\src\vs\workbench\contrib\codeEditor\electron-browser\startDebugTextMate.ts; C bundle (id editor.action.startDebugTextMate, nls module vs/workbench/contrib/codeEditor/electron-sandbox/startDebugTextMate)

### ED-5.49 Floating buttons in the editor corner
- Tag: both
- Evidence: source
- Boundary: no
- What: Certain files show a floating button, or a short row of them, in the bottom right corner of the editor - Open Workspace on a .code-workspace file, Add Configuration on launch.json, Define Keybinding on the user keybindings file, and Complete Merge in a settings sync merge - where the first button is drawn as the primary one and shows its key when it has one; the Add Server button on mcp.json and the Resolve Conflicts with AI button use the same corner but belong to MCP and chat, which are out of scope.
- Commands: workbench.action.openWorkspaceFromEditor; workbench.userDataSync.actions.acceptMerges (Add Configuration is named in SB-5 and Define Keybinding in ED-8)
- Menus: EditorContent
- Keys: none found
- Settings: none found
- Cursor: changed (Cursor draws each action as its own click widget through editor.contrib.floatingClickMenu and also has a multi-button widget; the stock floating toolbar id editor.contrib.floatingToolbar and the id workbench.userDataSync.actions.acceptMerges are not in the bundle; workbench.action.openWorkspaceFromEditor is present)
- Source: V\src\vs\editor\contrib\floatingMenu\browser\floatingMenu.ts; V\src\vs\editor\contrib\floatingMenu\browser\floatingMenu.css; V\src\vs\editor\contrib\floatingMenu\browser\floatingMenu.contribution.ts; V\src\vs\workbench\browser\codeeditor.ts (FloatingEditorClickMenu, an older click widget that nothing registers in V); V\src\vs\workbench\contrib\workspaces\browser\workspaces.contribution.ts (openWorkspaceFromEditor); V\src\vs\workbench\contrib\userDataSync\browser\userDataSync.ts (acceptMerges); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (Add Configuration); V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts (Define Keybinding); C bundle (ids editor.contrib.floatingClickMenu, editor.overlayWidget.floatingClickWidget, editor.overlayWidget.multiButtonFloatingWidget, workbench.action.openWorkspaceFromEditor with title "Open Workspace")

### ED-5.50 "Editing in worktree" label
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the worktree folders exist for agent runs, and the label tells you that a file you edit is inside one
- What: The first time you change a file that lives under the Cursor worktrees folder (.cursor/worktrees), a short "Editing in worktree" label fades in over the editor and fades out after about three seconds.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (editor contribution worktreeEditingHint, a centered bubble shown for 2500 ms with css class worktreeEditingHintWidget__bubble, which checks the path against the worktree manager's root folder; editor.contrib.worktreeNotification, a card at the top right shown for 3 seconds with widget id editor.worktreeNotificationWidget, which checks the path for the constant ".cursor/worktrees"; both use the text "Editing in worktree" and both wait for the first change to the file; whether both show together was not checked on screen)

### ED-5.51 Choose the syntax coloring engine (experimental)
- Tag: both
- Evidence: source
- Boundary: no
- What: Experimental settings switch syntax coloring for TypeScript, CSS, INI and regex files from the TextMate grammars to tree-sitter parsing, and move tokenization onto a background worker.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.experimental.asyncTokenization; editor.experimental.preferTreeSitter.typescript; editor.experimental.preferTreeSitter.css; editor.experimental.preferTreeSitter.ini; editor.experimental.preferTreeSitter.regex
- Cursor: present (all five keys are in the bundle; the four tree-sitter keys carry the tags experimental and onExP)
- Source: V\src\vs\editor\common\config\editorConfigurationSchema.ts; V\src\vs\workbench\services\treeSitter\browser\treeSitterLibraryService.ts (supportsLanguage reads the per-language setting); V\src\vs\editor\common\model\tokens\tokenizationTextModelPart.ts (chooses the tree-sitter or TextMate backend); V\src\vs\workbench\services\textMate\browser\textMateTokenizationFeatureImpl.ts (async tokenization switch); C bundle (keys editor.experimental.asyncTokenization, editor.experimental.preferTreeSitter.css, .typescript, .ini, .regex)

## Handed off

- Line number gutter right-click menu (`MenuId.EditorLineNumberContext`: breakpoints, quick diff, Cursor's Copy cursor.com link) and what the icon margin holds - ED-11
- Toggle Tab Key Moves Focus (`editor.action.toggleTabFocusMode`, Ctrl+M) - ED-2
- The read-only editor message ("Cannot edit in read-only editor", `editor.contrib.readOnlyMessageController`) and Set Active Editor Read-only or Writeable in Session - ED-2 (the message controller is not in the Cursor bundle)
- Toggle Screen Reader Accessibility Mode (`editor.action.toggleScreenReaderAccessibilityMode`, Shift+Alt+F1) and Announce Cursor Position (`editor.action.announceCursorPosition`, missing in Cursor) - no owner (the catalog lists the accessible view as unowned)
- The "Unusual Line Terminators" confirm box (`editor.unusualLineTerminators`) - DL-1
- Go to Line, Go to Symbol and Go to Offset pickers - QA-3
- Whole-window zoom (`workbench.action.zoomIn`, `window.zoomLevel`) - TB-3
- Terminal sticky scroll and terminal mouse-wheel zoom - PN-3
- Notebook sticky scroll, notebook line numbers and the notebook breadcrumbs toolbar entry - ED-7
- Diff editor breadcrumbs and the diff word wrap toggle - ED-6
- Outline view and Timeline view (the lists behind breadcrumb symbols) - SB-7
- Line and column, indentation and wrap indicators in the status bar - SS-3
- Go to Bracket, Select to Bracket and Remove Brackets commands (the display side is ED-5.24) - ED-2
- Developer: Inspect Key Mappings and Inspect Key Mappings (JSON) (`workbench.action.inspectKeyMappings`, `workbench.action.inspectKeyMappingsJSON`; each opens an untitled editor with the keybinding debug dump; both present in Cursor) - ED-8
- Word wrap and Render whitespace are listed in both ED-2 (ED-2.35, ED-2.36) and here (ED-5.36, ED-5.35) with the same commands - one of the two groups should keep them

## Unclear

- Many `editor.*` settings have no literal id string in the Cursor bundle, because the editor builds the id from the option name at run time. Where the Cursor field says present for these, the proof is the quoted option name (for example renderLineHighlight, matchBrackets, smoothScrolling, glyphMargin, cursorSurroundingLines, wordWrapColumn, mouseWheelZoom). This covers ED-5.18, 5.20, 5.22, 5.24, 5.26, 5.28, 5.29, 5.31, 5.32, 5.33, 5.35, 5.36 and 5.38. Defaults in Cursor were not compared with stock.
- Pass 1 filed the Go to Next, Previous and Trigger Symbol Highlight commands under ED-4. They are kept here in ED-5.33 because the request put word highlight in ED-5.
- Cursor's View > Appearance > Render Whitespace submenu: the entry exists in the pass 1 menu inventory, but the five set-mode commands that fill it are not in the bundle, so what the submenu holds in Cursor is not known.
- Newer than 1.128 (seen only in N, not features of their own): `editor.occurrencesHighlightFromAllProviders` (for ED-5.33), `editor.selectedTextMatchMode` (for ED-5.34), `editor.wordWrapIndicator` (for ED-5.36), `editor.fullwidthCharacterWidth` and an `editor.font` option (for ED-5.38), `breadcrumbs.showEditorType` (for ED-5.8). All are missing in Cursor.
- Cursor's `markdownEditor.breadcrumbContribution` (breadcrumbs for the Markdown editor, ED-5.7) was seen as a name only. How it looks was not checked.
- The sticky scroll controller name store.contrib.stickyScrollController appears twice in the Cursor bundle; the two uses were not read, so whether one is a Cursor change is not known.
- The accessibility toggles in this group (screen reader mode, tab focus mode) and the Alt+F1 help pages have no owning group; see Handed off.
- Overflow check on 2026-10-25 added 10 features (ED-5.41 to ED-5.50).

