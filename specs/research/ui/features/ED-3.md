Group: ED-3 Find and replace. Section: Editor (05-editor). Date: 2026-10-25. Pass: 2.
Features: 49.

### ED-3.1 Open the find widget
- Tag: both
- Evidence: source
- Boundary: no
- What: Press the find shortcut or choose Edit > Find to open a small find box at the top right of the editor, with the cursor in the search field.
- Commands: actions.find
- Menus: MenubarEditMenu (group 3_find, order 1, title Find); CommandPalette
- Keys: Ctrl+F (Cmd+F on macOS)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (StartFindAction); V\src\vs\editor\contrib\find\browser\findWidget.ts; V\src\vs\editor\contrib\find\browser\findModel.ts (FIND_IDS); C bundle (id actions.find, module vs/editor/contrib/find/browser/findController)

### ED-3.2 Find with arguments
- Tag: both
- Evidence: source
- Boundary: no
- What: A command, meant for keybindings.json, that opens the find widget with the search text, replace text and the regex, whole word, case and preserve-case flags already set.
- Commands: editor.actions.findWithArgs
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (StartFindWithArgsAction, argument schema findArgDescription); C bundle (id editor.actions.findWithArgs)

### ED-3.3 Fill the find box from the selection
- Tag: both
- Evidence: source
- Boundary: no
- What: Opening find puts the selected text, or the word at the cursor, into the find box; Find with Selection does the same for a multi-line selection without moving focus.
- Commands: actions.findWithSelection
- Menus: CommandPalette
- Keys: Cmd+E (macOS only)
- Settings: editor.find.seedSearchStringFromSelection
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (StartFindWithSelectionAction, getSelectionSearchString); V\src\vs\editor\common\config\editorOptions.ts (EditorFind); C bundle (ids actions.findWithSelection, editor.find.seedSearchStringFromSelection)

### ED-3.4 Search as you type
- Tag: both
- Evidence: source
- Boundary: no
- What: While you type in the find box the matches light up and the cursor jumps to the nearest one.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.find.cursorMoveOnType; editor.find.findOnType
- Cursor: changed (editor.find.cursorMoveOnType is in the bundle; editor.find.findOnType and the string "Controls whether the Find Widget should search as you type." are not)
- Source: V\src\vs\editor\contrib\find\browser\findWidget.ts (find input handlers, findOnType); V\src\vs\editor\contrib\find\browser\findModel.ts; V\src\vs\editor\common\config\editorOptions.ts (EditorFind); C bundle (module vs/editor/contrib/find/browser/findWidget, cursorMoveOnType option)

### ED-3.5 Match counter and no-results state
- Tag: both
- Evidence: source
- Boundary: no
- What: The box shows "3 of 12" next to the arrows, or "No results" with a red outline when nothing matches; above 19999 matches it shows "19999+" and a tooltip says only the first results are highlighted.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findWidget.ts (_updateMatchesCount, NLS_MATCHES_LOCATION, NLS_NO_RESULTS); V\src\vs\editor\contrib\find\browser\findModel.ts (MATCHES_LIMIT); C bundle (nls strings "Only the first {0} results are highlighted, but all find operations work on the entire text.", "{0} of {1}")

### ED-3.6 Step to the next or previous match
- Tag: both
- Evidence: source
- Boundary: no
- What: Jump to the next or previous match with keys or the up and down arrow buttons; the search wraps around at the end of the file unless wrapping is turned off.
- Commands: editor.action.nextMatchFindAction; editor.action.previousMatchFindAction
- Menus: CommandPalette
- Keys: F3 (Cmd+G on macOS, F3 also); Shift+F3 (Cmd+Shift+G on macOS, Shift+F3 also); Enter when the find box is focused; Shift+Enter when the find box is focused
- Settings: editor.find.loop
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (NextMatchFindAction, PreviousMatchFindAction, matchFindAction); V\src\vs\editor\contrib\find\browser\findWidget.ts (previous and next buttons); C bundle (ids editor.action.nextMatchFindAction, editor.action.previousMatchFindAction, editor.find.loop)

### ED-3.7 Go to a match by number
- Tag: both
- Evidence: source
- Boundary: no
- What: With the find widget open, Go to Match asks for a number and jumps to that match; a negative number counts from the last match, and the target line is highlighted while you type. The input box itself is the shared quick input (QA-6).
- Commands: editor.action.goToMatchFindAction
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (MoveToMatchFindAction); C bundle (id editor.action.goToMatchFindAction, label "Go to Match...", string "Type a number to go to a specific match (between 1 and {0})")

### ED-3.8 Find the next or previous match of the selected text
- Tag: both
- Evidence: source
- Boundary: no
- What: Jump to the next or previous place where the currently selected text appears, without typing it into the find box first.
- Commands: editor.action.nextSelectionMatchFindAction; editor.action.previousSelectionMatchFindAction
- Menus: CommandPalette
- Keys: Ctrl+F3 (editor focus); Ctrl+Shift+F3 (editor focus)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (NextSelectionMatchFindAction, PreviousSelectionMatchFindAction); C bundle (labels "Find Next Selection", "Find Previous Selection")

### ED-3.9 Match case
- Tag: both
- Evidence: source
- Boundary: no
- What: An "Aa" toggle in the find box makes the search case sensitive; the same key works with the widget closed (see ED-3.13).
- Commands: toggleFindCaseSensitive
- Menus: none found
- Keys: Alt+C (Cmd+Alt+C on macOS), editor focus
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (ToggleCaseSensitiveCommand); V\src\vs\editor\contrib\find\browser\findModel.ts (ToggleCaseSensitiveKeybinding); V\src\vs\editor\contrib\find\browser\findWidget.ts; C bundle (id toggleFindCaseSensitive)

### ED-3.10 Match whole word
- Tag: both
- Evidence: source
- Boundary: no
- What: An "ab" toggle in the find box limits matches to whole words.
- Commands: toggleFindWholeWord
- Menus: none found
- Keys: Alt+W (Cmd+Alt+W on macOS), editor focus
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (ToggleWholeWordCommand); V\src\vs\editor\contrib\find\browser\findModel.ts (ToggleWholeWordKeybinding); C bundle (id toggleFindWholeWord)

### ED-3.11 Use regular expression
- Tag: both
- Evidence: source
- Boundary: no
- What: A ".*" toggle in the find box treats the search text as a regular expression.
- Commands: toggleFindRegex
- Menus: none found
- Keys: Alt+R (Cmd+Alt+R on macOS), editor focus
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (ToggleRegexCommand); V\src\vs\editor\contrib\find\browser\findModel.ts (ToggleRegexKeybinding); C bundle (id toggleFindRegex)

### ED-3.12 Find in selection
- Tag: both
- Evidence: source
- Boundary: no
- What: A selection-icon button limits the search to the selected text and shades that range; it can switch on by itself for selections that cover several lines.
- Commands: toggleFindInSelection
- Menus: none found
- Keys: Alt+L (Cmd+Alt+L on macOS), editor focus
- Settings: editor.find.autoFindInSelection
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (ToggleSearchScopeCommand, autoFindInSelection switch); V\src\vs\editor\contrib\find\browser\findWidget.ts (_updateToggleSelectionFindButton, findSelectionIcon); V\src\vs\editor\contrib\find\browser\findDecorations.ts (findScope class); C bundle (ids toggleFindInSelection, editor.find.autoFindInSelection, label "Find in Selection")

### ED-3.13 Option badges when the widget is closed
- Tag: both
- Evidence: source
- Boundary: no
- What: Pressing a toggle key with the widget closed briefly shows small Match Case, Whole Word and Regex badges at the top right of the editor so you can see the new state; they stay while the mouse is over them.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findOptionsWidget.ts; V\src\vs\editor\contrib\find\browser\findOptionsWidget.css; C bundle (class name findOptionsWidget, id editor.contrib.findOptionsWidget)

### ED-3.14 Open the replace row
- Tag: both
- Evidence: source
- Boundary: no
- What: Replace opens the find widget with a second row for the replacement text; a chevron at the left (Toggle Replace) expands or collapses the row. It does nothing in a read-only editor.
- Commands: editor.action.startFindReplaceAction
- Menus: MenubarEditMenu (group 3_find, order 2, title Replace); CommandPalette
- Keys: Ctrl+H (Cmd+Alt+F on macOS); Tab from the find box moves to the replace box
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (StartFindReplaceAction); V\src\vs\editor\contrib\find\browser\findWidget.ts (_toggleReplaceBtn, findCollapsedIcon, findExpandedIcon); C bundle (id editor.action.startFindReplaceAction, labels "Replace", "Toggle Replace")

### ED-3.15 Replace one match
- Tag: both
- Evidence: source
- Boundary: no
- What: Replace the current match and move to the next, with the Replace button or by pressing Enter in the replace box.
- Commands: editor.action.replaceOne
- Menus: none found
- Keys: Enter when the replace box is focused; Ctrl+Shift+1 when the find widget is visible
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (ReplaceOneAction commands); V\src\vs\editor\contrib\find\browser\findWidget.ts (replace button, findReplaceIcon); V\src\vs\editor\contrib\find\browser\findModel.ts (replace); C bundle (id editor.action.replaceOne)

### ED-3.16 Replace all
- Tag: both
- Evidence: source
- Boundary: no
- What: Replace every match in the file in one step, with the Replace All button or the keys; on a file too large for it a warning says so.
- Commands: editor.action.replaceAll
- Menus: none found
- Keys: Ctrl+Alt+Enter when the find widget is visible; Cmd+Enter on macOS when the replace box is focused
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (ReplaceAllAction commands, too.large.for.replaceall warning); V\src\vs\editor\contrib\find\browser\replaceAllCommand.ts; V\src\vs\editor\contrib\find\browser\findWidget.ts (findReplaceAllIcon); C bundle (id editor.action.replaceAll, label "Replace All")

### ED-3.17 Preserve case
- Tag: both
- Evidence: source
- Boundary: no
- What: An "AB" toggle in the replace row makes the replacement copy the capital letters of each match (all caps, first letter capital, or lower case).
- Commands: togglePreserveCase
- Menus: none found
- Keys: Alt+P (Cmd+Alt+P on macOS), editor focus
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (TogglePreserveCaseCommand); V\src\vs\editor\contrib\find\browser\findModel.ts (TogglePreserveCaseKeybinding); V\src\vs\editor\contrib\find\browser\replacePattern.ts (buildReplaceString with preserveCase); C bundle (id togglePreserveCase, string "Toggle Preserve Case")

### ED-3.18 Capture groups and escape codes in the replacement
- Tag: both
- Evidence: source
- Boundary: no
- What: With regex on, the replacement can use $1 to $n and $& or $0 for matched text, $$ for a dollar, and the codes \n, \t, \\, \u, \U, \l and \L for new lines, tabs and upper or lower casing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\replacePattern.ts (parseReplaceString, ReplacePattern.buildReplaceString); C bundle (method names buildReplaceString, hasReplacementPatterns; the individual escape codes were not read in the bundle)

### ED-3.19 Select all matches
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn every match of the current search into a cursor so you can edit them all at once, then return focus to the text.
- Commands: editor.action.selectAllMatches
- Menus: none found
- Keys: Alt+Enter when the find widget is visible
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (SelectAllMatchesAction command, selectAllMatches); V\src\vs\editor\contrib\find\browser\findModel.ts (selectAllMatches); C bundle (id editor.action.selectAllMatches)

### ED-3.20 Multi-line text in the find and replace boxes
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+Enter inserts a line break in the find or replace box, which then grows taller (up to 118 px) so you can search for or insert several lines.
- Commands: none found
- Menus: none found
- Keys: Ctrl+Enter inside the find box or the replace box (the real Ctrl key on macOS too)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findWidget.ts (_onFindInputKeyDown, _onReplaceInputKeyDown, flexibleHeight, flexibleMaxHeight 118, _cachedHeight); C bundle (module vs/editor/contrib/find/browser/findWidget, flexibleHeight and flexibleMaxH options on the replace input)

### ED-3.21 Find and replace history
- Tag: both
- Evidence: source
- Boundary: no
- What: Up and Down arrow in the find box or the replace box step through earlier search and replace strings, which are kept for the workspace; either history can be turned off, and a keybinding hint shows in the box when Up and Down are the history keys.
- Commands: history.showPrevious; history.showNext
- Menus: none found
- Keys: UpArrow (history.showPrevious, history widget focused, not composing); DownArrow (history.showNext, history widget focused, not composing)
- Settings: editor.find.history; editor.find.replaceHistory
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findWidgetSearchHistory.ts (storage key workbench.find.history); V\src\vs\editor\contrib\find\browser\replaceWidgetHistory.ts (storage key workbench.replace.history); V\src\vs\platform\history\browser\contextScopedHistoryWidget.ts (history.showPrevious, history.showNext); V\src\vs\platform\history\browser\historyWidgetKeybindingHint.ts; C bundle (ids history.showPrevious, history.showNext, workbench.find.history, workbench.replace.history, editor.find.history, editor.find.replaceHistory)

### ED-3.22 Close the find widget
- Tag: both
- Evidence: source
- Boundary: no
- What: Escape (or the Close button) closes the find widget and returns focus to the text.
- Commands: closeFindWidget
- Menus: none found
- Keys: Escape; Shift+Escape (find widget visible, editor focus, not composing)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (CloseFindWidgetCommand, closeFindWidget); V\src\vs\editor\contrib\find\browser\findWidget.ts (close button, label Close); C bundle (id closeFindWidget)

### ED-3.23 Close the widget after landing on a match
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When the find widget is open and you press Find Next or Find Previous from its box and land on a match, the widget closes by itself if the setting is on.
- Commands: editor.action.nextMatchFindAction; editor.action.previousMatchFindAction
- Menus: none found
- Keys: F3; Shift+F3
- Settings: editor.find.closeOnResult
- Cursor: missing (the setting id and its description string "Controls whether the Find Widget closes after an explicit find navigation command lands on a result." are not in the bundle)
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (matchFindAction, shouldCloseOnResult); V\src\vs\editor\common\config\editorOptions.ts (EditorFind)

### ED-3.24 Resize the find widget
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag the left edge of the find widget to make it wider (it starts at 419 px); double-click the edge to reset; in a narrow editor the widget switches to reduced, narrow and collapsed layouts that hide parts of itself such as the match count.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findWidget.ts (FIND_WIDGET_INITIAL_WIDTH, _resizeSash, _tryUpdateWidgetWidth, onDidReset); V\src\vs\editor\contrib\find\browser\findWidget.css (reduced-find-widget, narrow-find-widget, collapsed-find-widget); C bundle (module vs/editor/contrib/find/browser/findWidget)

### ED-3.25 Extra scroll space above the first line
- Tag: both
- Evidence: source
- Boundary: no
- What: While the find widget is open the editor adds a blank band at the top so you can scroll the first line out from under the widget.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.find.addExtraSpaceOnTop
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findWidget.ts (_viewZone, _layoutViewZone, addExtraSpaceOnTop, getViewState); C bundle (setting editor.find.addExtraSpaceOnTop)

### ED-3.26 See matches in the text, scrollbar and minimap
- Tag: both
- Evidence: source
- Boundary: no
- What: Every match gets a highlight in the text, the current match a stronger one, and each match also draws a mark in the scrollbar strip and the minimap.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findDecorations.ts (classes findMatch, currentFindMatch, findScope, overviewRuler and minimap decoration options); C bundle (color ids minimap.findMatchHighlight, editorOverviewRuler.findMatchForeground, class findScope)

### ED-3.27 Share the find text with other macOS apps and the Search view
- Tag: both
- Evidence: source
- Boundary: no
- What: On macOS the find box can read and write the system-wide find pasteboard, so a search typed in the editor shows up in other apps and in the Search view, and the reverse. This is the hand-off between the in-editor find and workspace find.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.find.globalFindClipboard; search.globalFindClipboard
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (getGlobalBufferTerm, setGlobalBufferTerm, seedSearchStringFromGlobalClipboard); V\src\vs\platform\clipboard\common\clipboardService.ts (readFindText, writeFindText); V\src\vs\workbench\contrib\search\browser\searchWidget.ts (useGlobalFindBuffer); V\src\vs\workbench\contrib\search\browser\search.contribution.ts (search.globalFindClipboard); C bundle (ids editor.find.globalFindClipboard, search.globalFindClipboard)

### ED-3.28 Find accessibility help
- Tag: vscode
- Evidence: source
- Boundary: no
- What: With focus anywhere in the find widget, Alt+F1 opens a help page that gives the current search term, the match position, the replace text, and what you hear when you move to a match; the find box label also hints at this key when screen reader hints are on.
- Commands: editor.action.accessibilityHelp
- Menus: none found
- Keys: Alt+F1 when the find widget is focused (findWidgetFocused)
- Settings: accessibility.verbosity.find
- Cursor: missing (the help provider id EditorFindHelp and the module editorFindAccessibilityHelp are not in the bundle; the setting accessibility.verbosity.find and the id editor.action.accessibilityHelp are present, and the bundle uses that setting for the terminal find help)
- Source: V\src\vs\workbench\contrib\codeEditor\browser\editorFindAccessibilityHelp.ts; V\src\vs\workbench\contrib\codeEditor\browser\codeEditor.contribution.ts; V\src\vs\editor\contrib\find\browser\findWidget.ts (accessibilityHelpHintInLabel, aria alerts); V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (accessibility.verbosity.find)

### ED-3.29 Open a new Search Editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Open workspace search as an editor tab instead of the side bar: as a new tab, to the side, or by reusing the last Search Editor; a new one starts from the selected text or the word at the cursor and can run at once.
- Commands: search.action.openNewEditor; search.action.openNewEditorToSide; search.action.openEditor
- Menus: CommandPalette (category Search Editor; titles New Search Editor, Open New Search Editor to the Side, Open Search Editor)
- Keys: none found
- Settings: search.searchEditor.reusePriorSearchConfiguration; search.searchEditor.defaultNumberOfContextLines; search.searchEditor.focusResultsOnSearch; search.seedWithNearestWord; search.searchOnType (registered in V\src\vs\workbench\contrib\search\browser\search.common.contribution.ts)
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts (the three registerAction2 blocks, settings block); V\src\vs\workbench\contrib\searchEditor\browser\searchEditorActions.ts (openNewSearchEditor); V\src\vs\workbench\contrib\searchEditor\browser\constants.ts; C bundle (ids search.action.openNewEditor, search.action.openNewEditorToSide, search.action.openEditor, labels "New Search Editor", "Open Search Editor", "Open New Search Editor to the Side", pane id workbench.editor.searchEditor)

### ED-3.31 Type a query with case, word and regex toggles in the Search Editor
- Tag: both
- Evidence: source
- Boundary: no
- What: The top of the Search Editor has a query box with Match Case, Match Whole Word and Use Regular Expression toggles and a context-lines toggle; changing an option re-runs the search, and a command moves focus back to the box from the results.
- Commands: toggleSearchEditorCaseSensitive; toggleSearchEditorWholeWord; toggleSearchEditorRegex; search.action.focusQueryEditorWidget
- Menus: CommandPalette (category Search Editor)
- Keys: Alt+C; Alt+W; Alt+R (Cmd+Alt+ on macOS), all when the search input box is focused; Escape for Focus Search Editor Input (in the Search Editor)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.ts (queryEditorWidget, SearchWidget with showContextToggle); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts (toggle commands); C bundle (ids toggleSearchEditorCaseSensitive, toggleSearchEditorWholeWord, toggleSearchEditorRegex, search.action.focusQueryEditorWidget)

### ED-3.32 Filter by files to include and exclude
- Tag: both
- Evidence: source
- Boundary: no
- What: A Toggle Search Details button reveals "files to include" and "files to exclude" boxes, a switch to search only open editors, and a switch to use exclude settings and ignore files; commands move focus to each box.
- Commands: search.action.focusFilesToInclude; search.action.focusFilesToExclude
- Menus: CommandPalette (category Search Editor)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.ts (toggleQueryDetailsButton, inputPatternIncludes, inputPatternExcludes, onChangeSearchInEditorsBox, onChangeIgnoreBox); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts; C bundle (ids search.action.focusFilesToInclude, search.action.focusFilesToExclude, strings "Toggle Search Details", "files to include")

### ED-3.33 Read results as a text document
- Tag: both
- Evidence: source
- Boundary: no
- What: Results appear as plain text in an editor: header lines for the query and flags, a count line such as "12 results - 4 files", each file path as a heading, and numbered lines under it; a truncation note appears when the result set was capped.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditorSerialization.ts (serializeSearchConfiguration, serializeSearchResultForEditor, messages numResults, noResults, searchMaxResultsWarning); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.ts; C bundle (module vs/workbench/contrib/searchEditor/browser/searchEditorSerialization, strings "No Results", "The result set only contains a subset of all matches. Be more specific in your search to narrow down the results.")

### ED-3.34 Show context lines around matches
- Tag: both
- Evidence: source
- Boundary: no
- What: Show a chosen number of lines above and below each match: toggle context on or off, add a line, or remove a line; the start value comes from a setting.
- Commands: toggleSearchEditorContextLines; increaseSearchEditorContextLines; decreaseSearchEditorContextLines
- Menus: CommandPalette (category Search Editor)
- Keys: Alt+L (Cmd+Alt+L on macOS); Alt+= ; Alt+- (all in the Search Editor)
- Settings: search.searchEditor.defaultNumberOfContextLines
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts (context line commands); V\src\vs\workbench\contrib\searchEditor\browser\searchEditorActions.ts (modifySearchEditorContextLinesCommand); V\src\vs\workbench\contrib\searchEditor\browser\constants.ts; C bundle (ids toggleSearchEditorContextLines, increaseSearchEditorContextLines, decreaseSearchEditorContextLines, labels "Toggle Context Lines", "Increase Context Lines", "Decrease Context Lines")

### ED-3.35 Search again
- Tag: both
- Evidence: source
- Boundary: no
- What: Re-run the search after editing the query or header lines, with a refresh button in the editor title bar, a Run Search link when results were not loaded, or the key; a search can also fire as you type in the filter boxes.
- Commands: rerunSearchEditorSearch
- Menus: EditorTitle (group navigation, when the active editor is the Search Editor); CompactWindowEditorTitle (same); CommandPalette
- Keys: Ctrl+Shift+R when in the Search Editor
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts (RerunSearchEditorSearchCommandId, searchRefreshIcon); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.ts (triggerSearch, toggleRunAgainMessage, onSearchSubmit); C bundle (id rerunSearchEditorSearch, labels "Search Again", "Run Search")

### ED-3.36 Open a result from the Search Editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Go to a match's file and line from the results by double-click (or single-click peek, by setting), F12 or Ctrl+click on a result line, and Ctrl+click on a file heading; focus can stay in the results after a search.
- Commands: editor.action.goToDeclaration; editor.action.openDeclarationToTheSide; editor.action.peekDefinition
- Menus: none found
- Keys: none found
- Settings: search.searchEditor.doubleClickBehaviour; search.searchEditor.singleClickBehaviour; search.searchEditor.focusResultsOnSearch
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.ts (singleClickBehaviour, doubleClickBehaviour, executeCommand calls); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts (settings block); V\extensions\search-result\src\extension.ts (registerDefinitionProvider, registerDocumentLinkProvider); C bundle (ids search.searchEditor.doubleClickBehaviour, search.searchEditor.singleClickBehaviour, search.searchEditor.focusResultsOnSearch; extension dist\extension.js has the same providers)

### ED-3.37 Delete a file's results
- Tag: both
- Evidence: source
- Boundary: no
- What: Remove one file's whole block of results from the Search Editor text so only the files you care about remain.
- Commands: search.searchEditor.action.deleteFileResults
- Menus: CommandPalette (category Search Editor)
- Keys: Ctrl+Shift+Backspace (in the Search Editor)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts (deleteFileResults action); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.ts (deleteResultBlock); C bundle (id search.searchEditor.action.deleteFileResults, label "Delete File Results")

### ED-3.38 Select all matches in the Search Editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Put a cursor on every match in the results so you can edit them together.
- Commands: selectAllSearchEditorMatches
- Menus: CommandPalette (category Search Editor)
- Keys: Ctrl+Shift+L (in the Search Editor)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts (SelectAllSearchEditorMatchesCommandId); V\src\vs\workbench\contrib\searchEditor\browser\searchEditorActions.ts (selectAllSearchEditorMatchesCommand); C bundle (id selectAllSearchEditorMatches, label "Select All Matches")

### ED-3.39 Save and restore results as a .code-search file
- Tag: both
- Evidence: source
- Boundary: no
- What: A Search Editor starts as an untitled tab that you can save as a `.code-search` file and reopen later, and unsaved ones come back after a restart. The internal command cleanSearchEditorState clears its modified mark.
- Commands: cleanSearchEditorState
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditorInput.ts (SEARCH_EDITOR_EXT, working copy backup); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts (SearchEditorInputSerializer, SearchEditorWorkingCopyEditorHandler, CleanSearchEditorStateCommandId); V\extensions\search-result\package.json (language search-result for .code-search); C bundle (ids workbench.editorinputs.searchEditorInput, cleanSearchEditorState; extension language search-result)

### ED-3.40 Colored results, file outline and header hints
- Tag: both
- Evidence: source
- Boundary: no
- What: The search-result language colors the results, makes match lines bold and context lines dim, lists each file in the Outline, turns file headings into links, hides line numbers by default, and suggests the header lines (Query, Flags, Including, Excluding, ContextLines) and flag names as you type.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\search-result\package.json (language, grammar, configurationDefaults for [search-result]); V\extensions\search-result\src\extension.ts (decorations, registerDocumentSymbolProvider, registerCompletionItemProvider, registerDocumentLinkProvider); V\extensions\search-result\syntaxes\searchResult.tmLanguage.json; C bundle (C\extensions\search-result\dist\extension.js contains registerDocumentSymbolProvider, registerCompletionItemProvider, registerDefinitionProvider, registerDocumentLinkProvider)

### ED-3.41 Invalid regular expression message
- Tag: both
- Evidence: source
- Boundary: no
- What: With Use Regular Expression on, a pattern that does not compile (for example an open bracket) shows a red error box under the find box with the engine's own error text, and the box clears again when the pattern is fixed or regex is turned off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findWidget.ts (validation callback of the find input, new RegExp(value, 'gu'), _findInput.validate in _reveal); C bundle (find widget find input, fragment `validation:p=>{if(p.length===0||!this._findInput.getRegex())return null;try{return new RegExp(p,"gu"),null}catch(g){return{content:g.message}}}`)

### ED-3.42 Find options are remembered for the workspace
- Tag: both
- Evidence: source
- Boundary: no
- What: Match Case, Whole Word, Use Regular Expression and Preserve Case stay as you left them when you open find in another file or reopen the workspace later; Find in Selection is cleared each time you switch files.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findController.ts (loadQueryState, saveQueryState, onDidChangeModel; storage keys editor.matchCase, editor.wholeWord, editor.isRegex, editor.preserveCase in workspace scope); C bundle (same four storage keys in loadQueryState and saveQueryState)

### ED-3.43 Screen reader announcements while finding
- Tag: both
- Evidence: source
- Boundary: no
- What: Each time the match count or current match changes, the find widget speaks it: the full text of the matched line, then a message such as "3 of 12 found for 'foo', at 14:5", or "No results found for 'foo'"; a Find Next or Find Previous that finds nothing says the no-results message again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the _getAriaLabel code and the message texts "{0} found", "{0} found for '{1}'" and "{0} found for '{1}', at {2}" are in the bundle; the second copy of the "found for" text suggests the repeat announcement on Find Next exists too, but that call was not located in the minified code)
- Source: V\src\vs\editor\contrib\find\browser\findWidget.ts (_updateMatchesCount alertFn, _getAriaLabel, ariaSearchNoResult strings); V\src\vs\editor\contrib\find\browser\findController.ts (matchFindAction, alertFn with ariaSearchNoResult); C bundle (class FindWidget _getAriaLabel with nls keys 1114 to 1116, key 1117 for the repeat message)

### ED-3.44 Move around the find widget with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: Tab and Shift+Tab walk through the find box, replace box, option toggles and buttons in a fixed order, and Ctrl+Down (Cmd+Down on macOS) in either box puts the cursor back in the editor without closing the widget.
- Commands: none found
- Menus: none found
- Keys: Tab; Shift+Tab; Ctrl+Down (Cmd+Down on macOS) when the find box or replace box is focused
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\find\browser\findWidget.ts (_onFindInputKeyDown, _onReplaceInputKeyDown, onCaseSensitiveKeyDown, onRegexKeyDown, onPreserveCaseKeyDown, SimpleButton onKeyDown for the close and replace buttons); C bundle (same handlers: focusOnCaseSensitive, focusOnPreserve, CtrlCmd+DownArrow in the find widget)

### ED-3.45 Search Editor tab title and icon
- Tag: both
- Evidence: source
- Boundary: no
- What: A Search Editor tab carries a search icon and the title "Search: <query>" with the query cut to 12 characters plus "...", just "Search" while the query is empty, and "Search: <file name>" for a saved `.code-search` file; the title follows the query as you change it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditorInput.ts (getName, getIcon, SearchEditorIcon registered as search-editor-label-icon, _onDidChangeLabel on config update); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.ts (getAriaLabel); C bundle (getName(e=12) with nls keys 9712 and 9713 "Search: {0}", key 9714 "Search", icon id search-editor-label-icon, key 9711)

### ED-3.46 Step through Search Editor results with F4
- Tag: both
- Evidence: source
- Boundary: no
- What: F4 and Shift+F4 move the cursor to the next or previous highlighted match in the Search Editor results, wrap at the ends, scroll it into view and announce "Matched <text> at <line> in file <path>" to screen readers; the same keys run the Search view's own next and previous result (SB-3.26).
- Commands: search.action.focusNextSearchResult; search.action.focusPreviousSearchResult
- Menus: CommandPalette (category Search; usable when the Search view has results or the active editor is a Search Editor)
- Keys: F4; Shift+F4
- Settings: none found
- Cursor: changed (same two ids and keys; Cursor's Search Editor registers its own internal commands searchEditor.action.focusNextResult and searchEditor.action.focusPreviousResult, and the two search.action ids call them)
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts (FocusNextSearchResultAction, FocusPreviousSearchResultAction, focusNextSearchResult, focusPreviousSearchResult); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.ts (focusNextResult, focusPreviousResult, iterateThroughMatches, findNextRange, findPrevRange, searchResultItem alert); V\src\vs\workbench\contrib\search\common\constants.ts (SearchCommandIds); C bundle (ids search.action.focusNextSearchResult, search.action.focusPreviousSearchResult, searchEditor.action.focusNextResult, searchEditor.action.focusPreviousResult; nls keys 9484 "Focus Next Search Result", 9485 "Focus Previous Search Result" and 9708 "Matched {0} at {1} in file {2}"; keybinding F4 and the Search Editor precondition seen in the bundle)

### ED-3.47 Move between the query boxes and the results with Ctrl+Down and Ctrl+Up
- Tag: both
- Evidence: source
- Boundary: no
- What: In the Search Editor, Ctrl+Down (Cmd+Down on macOS) in a query box hops from the query to files to include, files to exclude and then the results; Ctrl+Up hops back the other way and from the query box wraps to the results. The boxes for include and exclude are only visited while Toggle Search Details has them open.
- Commands: search.focus.nextInputBox; search.focus.previousInputBox
- Menus: none found
- Keys: Ctrl+Down (search.focus.nextInputBox, when in the Search Editor with a search box focused); Ctrl+Up (search.focus.previousInputBox, same condition); both use Cmd on macOS
- Settings: none found
- Cursor: changed (same two ids and keys; Cursor's Search Editor registers internal commands searchEditor.action.focusNextInput and searchEditor.action.focusPreviousInput that the two search.focus ids call)
- Source: V\src\vs\workbench\contrib\search\browser\searchActionsNav.ts (FocusNextInputAction, FocusPreviousInputAction); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.ts (focusNextInput, focusPrevInput, InputBoxFocusedKey bound in createEditor); V\src\vs\workbench\contrib\search\common\constants.ts (SearchCommandIds); C bundle (ids search.focus.nextInputBox, search.focus.previousInputBox, searchEditor.action.focusNextInput, searchEditor.action.focusPreviousInput; nls keys 9479 "Focus Next Input" and 9480 "Focus Previous Input"; the same ids with the Search Editor context key in the keybinding conditions)

### ED-3.48 Pick which notebook cell parts the Search Editor searches
- Tag: both
- Evidence: source
- Boundary: no
- What: While a notebook is open, a filter button joins the toggles in the Search Editor's query box with four choices (Markdown Source, Rendered Markdown, Code Cell Source, Code Cell Output), and the choice goes into the search the editor runs; the same button in the Search view is SB-3.4.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\searchWidget.ts (_notebookFilters, _hasNotebookOpen, filterVisible); V\src\vs\workbench\contrib\search\browser\searchFindInput.ts (NotebookFindInputFilterButton); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.ts (createQueryEditor, readConfigFromWidget notebookSearchConfig, doRunSearch); V\src\vs\workbench\contrib\notebook\browser\contrib\find\notebookFindReplaceWidget.ts (the four labels); C bundle (Search Editor readConfigFromWidget with getNotebookFilters, notebookSearchConfig in the query builder, filterVisible set from _hasNotebookOpen; nls key 9503 "Notebook Find Filters")

### ED-3.49 Highlighted matches in the Search Editor results
- Tag: both
- Evidence: source
- Boundary: no
- What: The matched text on each result line is shaded inside the Search Editor, apart from the bold and dim line styling of ED-3.40; the shading is drawn by the editor itself, follows the `searchEditor.findMatchBackground` and `searchEditor.findMatchBorder` theme colors, and the highlight ranges are stored with the editor state when it has unsaved changes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\searchEditor\browser\searchEditorInput.ts (setMatchRanges, getMatchRanges, SearchEditorFindMatchClass); V\src\vs\workbench\contrib\searchEditor\browser\media\searchEditor.css (.searchEditorFindMatch); V\src\vs\platform\theme\common\colors\searchColors.ts (searchEditor.findMatchBackground, searchEditor.findMatchBorder); V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts (serializer keeps matchRanges for a dirty editor); C bundle (class searchEditorFindMatch, color ids searchEditor.findMatchBackground and searchEditor.findMatchBorder)

### ED-3.50 One-time Ctrl+Enter notice in the replace box on Windows
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: On Windows desktop, the first time Ctrl+Enter is pressed in the replace box and no keybinding handles it, an information notification says "Ctrl+Enter now inserts line break instead of replacing all. You can modify the keybinding for editor.action.replaceAll to override this behavior."; it shows once and is remembered. Stock 1.128.0 no longer has this notice (searched V for the storage key and the text, no hits).
- Commands: none found
- Menus: none found
- Keys: Ctrl+Enter in the replace box (Windows only)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (find widget replace-box key handler, storage key `ctrlEnterReplaceAll.windows.donotask`, nls key 1118; the platform test is on the minified Windows and native flags `ia` and `Q_`, read from the code, not run)

## Handed off

- Find in Files and Replace in Files (Edit menu group 4_find_global, Ctrl+Shift+F, Ctrl+Shift+H; `workbench.action.findInFiles`, `workbench.action.replaceInFiles`) - SB-3
- Find in Folder and Find in Workspace (Explorer context menu, `filesExplorer.findInFolder`, `filesExplorer.findInWorkspace`) - SB-2 and SB-3 (SB-3 owns the search it starts)
- Add Selection to Next Find Match, Move Last Selection to Next Find Match, Select All Occurrences of Find Match (`editor.action.addSelectionToNextFindMatch`, `editor.action.selectHighlights`) - ED-2
- Find box inside webview editors (`editor.action.webvieweditor.showFind`) and in Cursor's own Browser editor (`showBrowserFind` and related) - ED-10
- Find inside notebooks (`notebook.find`) - ED-7
- Find inside the terminal (simple find widget) - PN-3
- The Go to Match input box and quick input look - QA-6

## Unclear

- Pass 1 could not locate the registration site of the `search.searchEditor.*` settings. All five (`doubleClickBehaviour`, `singleClickBehaviour`, `reusePriorSearchConfiguration`, `defaultNumberOfContextLines`, `focusResultsOnSearch`) are registered in `V\src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts`.
- Pass 1 named `search.action.openNewEditorFromView` as the Search view entry; the title-bar Open Results in Editor id is `search.action.openInEditor`. Both are listed in ED-3.30.
- The Edit menu entries Find and Replace are also inventoried under the menu bar group (TB-1); their entry points are repeated here on ED-3.1 and ED-3.14 because the commands belong to this group.
- The Cursor Find accessibility help (Alt+F1 page) is marked missing from name searches only; the generic accessible view is listed by the catalog as unowned.
- Keys marked "Cmd+Alt+ on macOS" for the Search Editor toggles follow the shared find keybinding objects (`ToggleCaseSensitiveKeybinding` and siblings); the Search Editor registers them again under the search input context.
- Defaults for the `editor.find.*` settings were read in V only. Whether Cursor changes any default was not checked.
- ED-3.36 lists three editor commands the Search Editor calls itself on click; they are not its own entry points.
- Ctrl+Shift+J (`workbench.action.search.toggleQueryDetails`) also toggles the details area of ED-3.32 inside a Search Editor; it is a Search view id owned by SB-3, so ED-3.32 does not name it. The setting `search.mode` (Find in Files opens a Search Editor) is SB-3.29.
- The Cursor find widget is not an exact copy of stock 1.128.0: it lacks `editor.find.closeOnResult`, `editor.find.findOnType` and the find accessibility help (ED-3.23, ED-3.28) and still has the Windows Ctrl+Enter notice (ED-3.50). A string comparison of the find, Search Editor and replace texts found no other differences, but behavior differences with no strings were not checked.
- Opening a Search Editor from keybindings.json with arguments (query, filesToInclude, location and so on) uses the same three commands as ED-3.29, so it was not split into its own feature.
- The find highlight and Search Editor highlight theme colors (`editor.findMatchBackground` and siblings, `searchEditor.findMatchBackground`) are mentioned in ED-3.26 and ED-3.49 only; no catalog group owns theme colors.
- Overflow check on 2026-10-25 added 10 features (ED-3.41 to ED-3.50).
