Section: Editor (ED). Date: 2026-10-25. Pass: 1.

# 05 Editor - VS Code 1.128.0 findings (pass 1, minor groups)

## How to read this file

- Scope of this file: stock VS Code 1.128.0 only (V). The Cursor side is in the sibling file `cursor\05-editor.md`. Tags and the `bundle` proof live there.
- Path legend: `V` = `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0`. `N` = `C:\Users\Vinnie\src\vscode` (1.144.0, only used to spot newer things). Paths below are relative to V unless they start with `N\`.
- Evidence is `source` on every item in this file (read in V). The tag on each group says whether the Cursor bundle also has it. A tag of `both` means a command id or label of that group was found in the Cursor bundle (details in the Cursor file).
- Autocomplete (suggest list, inline completions and ghost text, parameter hints, signature help, snippet placeholders) is section 06 and is not grouped here.
- Id rule: group ids ED-1 to ED-10 are stable. Features get `ED-n.m` in the next pass.

## Why 10 groups (the cap for this section)

This is the biggest section. It holds three different kinds of things that people do not mix up: the frame around editors (groups and tabs), the text editor itself (editing, finding, language help, how it looks), and a dozen other editor types (diff and merge, notebooks, settings, welcome pages, viewers). Fewer than 10 would glue unlike things together, for example Settings editors with diff editors, or tabs with find. More than 10 would split the text editor into pieces nobody names. Confidence: medium - the line between ED-4 and ED-5 (language help versus display aids) is a judgment call and could be redrawn.

## Groups at a glance

| Id | Plain name | Tag | Boundary |
|-|-|-|-|
| ED-1 | Editor groups, tabs and layout | both | no |
| ED-2 | Editing text | both | no |
| ED-3 | Find and replace | both | no |
| ED-4 | Language help in the editor | both | no |
| ED-5 | Editor view and orientation aids | both | no |
| ED-6 | Diffs, merges and change review | both | yes |
| ED-7 | Notebooks, Interactive Window and REPL | both | no |
| ED-8 | Settings, shortcuts and configuration editors | both | yes |
| ED-9 | Welcome, empty and first-run screens | both | no |
| ED-10 | Viewers, previews and web editors | both | yes |

## Proposed minor groups

### ED-1 Editor groups, tabs and layout

- Plain name: Editor groups, tabs and layout
- One sentence: The grid of editor groups and the tab strip on top of each one, with splitting, pinning, preview tabs, locking, moving into new windows, and keys and pickers for jumping between open editors.
- Tag: both
- Evidence: source
- Boundary: no
- Example features: split editor right or down and the layout presets (two columns, grid); pin, preview and keep-open tabs; move or copy an editor into a new window; Show All Editors and recently-used editor switching.
- Proof paths:
  - `src\vs\workbench\browser\parts\editor\editorPart.ts` and `editorGroupView.ts` (the group grid and each group)
  - `src\vs\workbench\browser\parts\editor\editorTabsControl.ts`, `multiEditorTabsControl.ts`, `multiRowEditorTabsControl.ts`, `singleEditorTabsControl.ts`, `noEditorTabsControl.ts`, `editorTitleControl.ts` (tab modes: multiple, single, none)
  - `src\vs\workbench\browser\parts\editor\editorActions.ts` and `editorCommands.ts` (command bodies)
  - `src\vs\workbench\browser\parts\editor\editor.contribution.ts` (114 `registerAction2(...)` lines at 174 to 310, and the menus `EditorTitle`, `EditorTitleContext`, `EditorTabsBarContext`, `EmptyEditorGroup`, `EmptyEditorGroupContext`, lines 354 to 470)
  - `src\vs\workbench\browser\parts\editor\editorDropTarget.ts` (drop overlay that splits), `editorQuickAccess.ts` (Show All Editors pickers), `editorsObserver.ts` (recently-used order)
  - `src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts` (editors in their own window), `modalEditorPart.ts` (editors in a modal overlay)
  - `src\vs\workbench\browser\workbench.contribution.ts` lines 40 to 460 (about 50 `workbench.editor.*` settings) and `src\vs\workbench\browser\parts\editor\editorConfiguration.ts` (`workbench.editor.autoLockGroups`, `workbench.editor.defaultBinaryEditor`)
  - `src\vs\workbench\browser\actions\layoutActions.ts` (`workbench.action.toggleCenteredLayout`) and `src\vs\workbench\browser\workbench.zenMode.contribution.ts` (`workbench.action.toggleZenMode`)

Facts seen (for the feature pass):

- Tab modes come from `workbench.editor.showTabs`: `multiple`, `single`, `none` (`workbench.contribution.ts` lines 65 to 67). Editor actions can sit in the tab bar, the title bar, or be hidden (`workbench.editor.editorActionsLocation`).
- Tab context menu (`MenuId.EditorTitleContext`): Close, Close Others, Close to the Right, Close Saved, Close All, Reopen Editor With..., Keep Open, Pin, Unpin, Split Right or Down, Split & Move submenu (Split Up/Down/Left/Right, Move Above/Below/Left/Right, Split in Group, Join in Group), Move into New Window, Copy into New Window, Share submenu.
- Tab bar context menu (`MenuId.EditorTabsBarContext`): Split Up/Down/Left/Right, Move into or Copy into New Window, Tab Bar submenu (Multiple Tabs, Single Tab, Hidden), Editor Actions Position submenu (Tab Bar, Title Bar, Hidden), Configure Tabs.
- Empty group menus: Lock Group, Close Group, Split directions, New Window (`editor.contribution.ts` lines 354 to 365).
- Editor title toolbar (`MenuId.EditorTitle`): Inline View toggle for diffs, Show Opened Editors, Close All, Close Saved, Enable Preview Editors, Maximize Group, Lock Group, Configure Editors, plus the Split Editor button (Right, with Down as the alternate).
- Group commands (ids): `workbench.action.splitEditor`, `splitEditorOrthogonal`, `splitEditorInGroup`, `joinEditorInGroup`, `editorLayoutSingle`, `editorLayoutTwoColumns`, `editorLayoutThreeColumns`, `editorLayoutTwoRows`, `editorLayoutThreeRows`, `editorLayoutTwoByTwoGrid`, `editorLayoutTwoRowsRight`, `editorLayoutTwoColumnsBottom`, `newGroupLeft` (Right, Above, Below), `focusLeftGroup` (Right, Above, Below, First, Last, Next, Previous), `moveActiveEditorGroupLeft` (Right, Up, Down), `duplicateActiveEditorGroupLeft` (and the other three directions), `joinTwoGroups`, `joinAllGroups`, `toggleMaximizeEditorGroup`, `minimizeOtherEditors`, `evenEditorWidths`, `toggleEditorWidths`, `toggleEditorGroupLock`.
- Editor commands (ids): `workbench.action.closeActiveEditor`, `closeAllEditors`, `closeOtherEditors`, `closeUnmodifiedEditors`, `closeEditorsToTheRight`, `closeEditorsInOtherGroups`, `pinEditor`, `unpinEditor`, `keepEditor`, `nextEditor`, `previousEditor`, `nextEditorInGroup`, `firstEditorInGroup`, `moveEditorLeftInGroup`, `moveEditorToNextGroup`, `splitEditorToNextGroup`, `moveEditorToNewWindow`, `copyEditorToNewWindow`, `moveEditorGroupToNewWindow`, `restoreEditorsToMainWindow`, `newEmptyEditorWindow`, `reopenClosedEditor`, `navigateBack`, `navigateForward`, `navigateToLastEditLocation`, `showAllEditors`, `showAllEditorsByMostRecentlyUsed`, `showEditorsInActiveGroup`, `quickOpenPreviousRecentlyUsedEditor`, `openNextRecentlyUsedEditor`, `toggleEditorType`, `reopenWithEditor`, `reopenTextEditor`, `clearEditorHistory`, `clearRecentFiles`, `configureEditor`, `configureEditorTabs`, `showMultipleEditorTabs`, `hideEditorTabs`, `editorActionsDefault`, `hideEditorActions`.
- Window modes for editors in their own window: `workbench.action.toggleCompactAuxiliaryWindow` ("Toggle Window Compact Mode"), `enableCompactAuxiliaryWindow` ("Turn On Compact Mode"), `disableCompactAuxiliaryWindow` ("Turn Off Compact Mode") in `auxiliaryEditorPart.ts` lines 54 to 93. Modal editor: `workbench.editor.useModal` (`workbench.contribution.ts` line 357) and menu `MenuId.ModalEditorEditorTitle`.
- Tab-level settings: `workbench.editor.tabSizing`, `tabSizingFixedMinWidth`, `tabSizingFixedMaxWidth`, `pinnedTabSizing`, `pinnedTabsOnSeparateRow`, `preventPinnedEditorClose`, `wrapTabs`, `scrollToSwitchTabs`, `highlightModifiedTabs`, `tabActionLocation`, `tabActionCloseVisibility`, `tabActionUnpinVisibility`, `showTabIndex`, `showIcons`, `labelFormat`, `untitled.labelFormat`, `decorations.badges`, `decorations.colors`, `titleScrollbarSizing`, `titleScrollbarVisibility`, `doubleClickTabToToggleEditorGroupSizes`.
- Open and split behavior settings: `enablePreview`, `enablePreviewFromQuickOpen`, `enablePreviewFromCodeNavigation`, `openPositioning`, `openSideBySideDirection`, `closeEmptyGroups`, `revealIfOpen`, `closeOnFileDelete`, `focusRecentEditorAfterClose`, `splitSizing`, `splitOnDragAndDrop`, `splitInGroupLayout`, `dragToOpenWindow`, `limit.enabled`, `limit.value`, `limit.excludeDirty`, `limit.perEditorGroup`, `navigationScope`, `restoreViewState`, `sharedViewState`, `restoreEditors`, `swipeToNavigate`, `mouseBackForwardToNavigate`, `centeredLayoutAutoResize`, `centeredLayoutFixedWidth`, `useModal`.
- Auto save and dirty marks: `src\vs\workbench\browser\parts\editor\editorAutoSave.ts`, setting `files.autoSave` and `files.autoSaveDelay` in `src\vs\workbench\contrib\files\browser\files.contribution.ts` lines 261 and 275. Save, Save As, Save All, Revert are in `src\vs\workbench\contrib\files\browser\fileActions.contribution.ts` and `fileCommands.ts` (File menu, section 01).
- Status-bar items for language, encoding, EOL come from `editorStatus.ts` in the same folder; the status bar widgets belong to section 09. The pickers (`workbench.action.editor.changeLanguageMode`, `changeEOL`, `changeEncoding`) are registered in `editor.contribution.ts` lines 174 to 176.

### ED-2 Editing text

- Plain name: Editing text
- One sentence: The everyday commands for typing, selecting and reshaping text inside one editor: multi-cursor, line operations, comments, indentation, clipboard and drag-drop, jumping to brackets, linked editing.
- Tag: both
- Evidence: source
- Boundary: no
- Example features: multi-cursor (add cursor above or below, add next occurrence, column selection); line operations (move, copy, delete, sort, join, change case); toggle line or block comment; indentation (detect, convert tabs and spaces).
- Proof paths:
  - `src\vs\editor\contrib\multicursor\browser\multicursor.ts` (13 actions, for example `editor.action.insertCursorAbove`, `editor.action.addSelectionToNextFindMatch`, `editor.action.selectHighlights`, `editor.action.changeAll`)
  - `src\vs\editor\contrib\linesOperations\browser\linesOperations.ts` (25 actions: `editor.action.moveLinesUpAction`, `copyLinesDownAction`, `sortLinesAscending`, `joinLines`, `transformToUppercase`, `transformToSnakecase`, `deleteAllLeft` and more)
  - `src\vs\editor\contrib\comment\browser\comment.ts` (`editor.action.commentLine`, `editor.action.blockComment`, `addCommentLine`, `removeCommentLine`)
  - `src\vs\editor\contrib\indentation\browser\indentation.ts` (Convert Indentation to Spaces or Tabs, Indent Using Tabs or Spaces, Detect Indentation from Content, Reindent Lines)
  - `src\vs\editor\contrib\caretOperations\browser\caretOperations.ts` and `transpose.ts` (Move Selected Text Left or Right, Transpose Letters), `smartSelect\browser\smartSelect.ts` (`editor.action.smartSelect.expand`, `shrink`), `cursorUndo\browser\cursorUndo.ts` (`cursorUndo`, `cursorRedo`), `anchorSelect\browser\anchorSelect.ts` (selection anchor), `lineSelection\browser\lineSelection.ts` (`expandLineSelection`), `wordOperations\browser\wordOperations.ts`, `wordPartOperations`, `inPlaceReplace\browser\inPlaceReplace.ts`, `insertFinalNewline\browser\insertFinalNewLine.ts`
  - `src\vs\editor\contrib\clipboard\browser\clipboard.ts` (Copy with Syntax Highlighting), `src\vs\editor\contrib\dropOrPasteInto\browser\` (`editor.action.pasteAs`, Paste as Text, drop options and paste options pickers, `editor.pasteAs.default`)
  - `src\vs\editor\contrib\linkedEditing\browser\linkedEditing.ts` (`editor.action.linkedEditing`), `bracketMatching\browser\bracketMatching.ts` (Go to Bracket, Select to Bracket, Remove Brackets)
  - `src\vs\workbench\contrib\codeEditor\browser\toggleColumnSelection.ts`, `toggleMultiCursorModifier.ts`, `toggleOvertype.ts` (`editor.action.toggleOvertypeInsertMode`), `saveParticipants.ts` (trim whitespace, final newline, format on save), `dictation\editorDictation.ts` (`workbench.action.editorDictation.start`)
  - `src\vs\workbench\contrib\emmet\browser\` and `V\extensions\emmet\package.json` (23 commands, for example Expand Abbreviation, Wrap with Abbreviation; 12 settings)
  - `src\vs\workbench\contrib\files\browser\fileActions.ts` lines 1364 to 1393 (`workbench.action.files.setActiveEditorReadonlyInSession`, `toggleActiveEditorReadonlyInSession`), and `fileActions.contribution.ts` (Save, Revert)

Facts seen (for the feature pass):

- Menu surfaces: the editor context menu (`MenuId.EditorContext`) and the Selection menu (`MenubarSelectionMenu`) host most of these. Their menu items are spread across the contrib files above.
- Behavior-only features with no menu entry: auto closing brackets and quotes, auto surround, auto indent, type-over, and format on type or paste. They live in `src\vs\editor\common\cursor\` and `src\vs\editor\common\commands\` and are driven by `editor.*` options in `src\vs\editor\common\config\editorOptions.ts`.
- Overtype mode, multi-cursor modifier and column selection have toggles in the codeEditor folder with their own status bar hints (section 09 owns the status bar piece).
- Cross-section: the Selection and Edit menus (section 01), the status bar pickers (section 09) and Explorer file actions (section 04) reach the same commands.

### ED-3 Find and replace

- Plain name: Find and replace
- One sentence: The find and replace box that opens inside an editor, plus the Search Editor that shows workspace search results as a document you can edit and save.
- Tag: both
- Evidence: source
- Boundary: no
- Example features: find widget (match case, whole word, regex, find in selection, match count); replace and replace all; Find Next Selection and Go to Match; Search Editor with context lines.
- Proof paths:
  - `src\vs\editor\contrib\find\browser\findController.ts` (actions Find, Find with Arguments, Find with Selection, Find Next, Find Previous, Go to Match..., Find Next Selection, Find Previous Selection, Replace; ids come from `FIND_IDS` in `findModel.ts` lines 60 to 78: `actions.find`, `actions.findWithSelection`, `editor.actions.findWithArgs`, `editor.action.nextMatchFindAction`, `previousMatchFindAction`, `goToMatchFindAction`, `nextSelectionMatchFindAction`, `previousSelectionMatchFindAction`, `startFindReplaceAction`, `replaceOne`, `replaceAll`, `selectAllMatches`, plus widget keys `closeFindWidget`, `toggleFindCaseSensitive`, `toggleFindWholeWord`, `toggleFindRegex`, `toggleFindInSelection`, `togglePreserveCase`)
  - `src\vs\editor\contrib\find\browser\findWidget.ts`, `findModel.ts`, `findOptionsWidget.ts`, `findState.ts`
  - `src\vs\workbench\contrib\codeEditor\browser\find\simpleFindWidget.ts` (the reused find box in webviews, notebooks' outputs, terminal) and `src\vs\workbench\contrib\webview\browser\webviewFindWidget.ts`
  - `src\vs\workbench\contrib\searchEditor\browser\searchEditor.contribution.ts`, `searchEditor.ts`, `searchEditorActions.ts` (`search.action.openNewEditorFromView`, Toggle Use Regular Expression, Toggle Context Lines, Increase or Decrease Context Lines, Select All Matches; editor pane label "Search Editor")
  - `V\extensions\search-result\package.json` (language `search-result` and grammar for `.code-search` files; no commands)
  - `src\vs\workbench\contrib\codeEditor\browser\editorFindAccessibilityHelp.ts`

Facts seen (for the feature pass):

- The same find widget also drives notebook find (`src\vs\workbench\contrib\notebook\browser\contrib\find\`) and the diff editor's two panes.
- The Search Editor opens from the Search view. Command ids: `search.action.openNewEditorFromView` (`searchEditor.contribution.ts` line 602), `search.action.openNewEditorToSide` (line 46), `search.action.openNewEditor` and `search.action.openEditor` (`src\vs\workbench\contrib\searchEditor\browser\constants.ts` lines 18 and 19). The Search view itself belongs to section 04.
- Settings: `editor.find.*` options (seedSearchStringFromSelection, autoFindInSelection, addExtraSpaceOnTop, loop, cursorMoveOnType, globalFindClipboard, closeOnResult, history, replaceHistory, findOnType) in `src\vs\editor\common\config\editorOptions.ts` lines 1786 to 1854; `search.searchEditor.doubleClickBehaviour` in `searchEditor.contribution.ts` line 66; `search.searchEditor.defaultNumberOfContextLines`, `singleClickBehaviour`, `focusResultsOnSearch` are read in `searchEditorActions.ts` line 214 and `searchEditor.ts` lines 256 and 489 (their registration site was not located).

### ED-4 Language help in the editor

- Plain name: Language help in the editor
- One sentence: The features that understand the code and answer from inside the editor: hover info, go to definition and peek, find references, go to symbol or error, rename, quick fix and refactor, format, CodeLens, inlay hints, link and color helpers.
- Tag: both
- Evidence: source
- Boundary: no
- Example features: hover with definition preview; go to definition, peek definition and references; rename symbol with preview; quick fix and refactor (lightbulb and code action menu).
- Proof paths:
  - `src\vs\editor\contrib\hover\browser\hoverActions.ts` (Show or Focus Hover, Show Definition Preview Hover, Hide Hover, scroll and page hover actions)
  - `src\vs\editor\contrib\gotoSymbol\browser\goToCommands.ts` (Open Definition to the Side, Peek Definition, Peek Declaration, Peek Type Definition, Peek Implementations, Peek References, Go to Any Symbol), `peek\referencesWidget.ts`, `referencesModel.ts`
  - `src\vs\editor\contrib\peekView\browser\peekView.ts` (the inline peek frame used by references, definitions, diff peeks, comments)
  - `src\vs\editor\contrib\gotoError\browser\gotoError.ts` (Go to Next or Previous Problem, the marker navigation widget)
  - `src\vs\editor\contrib\rename\browser\rename.ts`, `renameWidget.ts` (`editor.action.rename`)
  - `src\vs\editor\contrib\codeAction\browser\codeActionCommands.ts` (Quick Fix, Refactor, Source Action, Organize Imports, Fix All, Auto Fix), `codeActionMenu.ts`, `lightBulbWidget.ts`, and `src\vs\workbench\contrib\codeActions\browser\codeActionsContribution.ts` (`editor.codeActionsOnSave`)
  - `src\vs\editor\contrib\format\browser\formatActions.ts` (Format Document, Format Selection) and `src\vs\workbench\contrib\format\browser\formatActionsMultiple.ts`, `formatActionsNone.ts`, `formatModified.ts` (Format Document With..., Format Modified Lines)
  - `src\vs\editor\contrib\codelens\browser\codelensController.ts` (`codelens.showLensesInCurrentLine`), `inlayHints\browser\` and `src\vs\workbench\contrib\inlayHints\browser\inlayHintsAccessibilty.ts`
  - `src\vs\editor\contrib\wordHighlighter\browser\wordHighlighter.ts` (Go to Next or Previous Symbol Highlight, Trigger Symbol Highlight), `links\browser\links.ts` (`editor.action.openLink`), `colorPicker\browser\` (inline color boxes, picker, `editor.action.insertColorWithStandaloneColorPicker`), `documentSymbols\`, `semanticTokens\`
  - `src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts` and `gotoSymbolQuickAccess.ts` (the pickers behind Go to Line and Go to Symbol; section 11 owns the picker UI)

Facts seen (for the feature pass):

- The Go menu and editor context menu surface most of these: Go to Definition (`editor.action.revealDefinition`), Go to Declaration, Go to Type Definition, Go to Implementations, Go to References (`editor.action.goToReferences`), Peek submenu (`MenuId.EditorContextPeek`), Go to Symbol in Editor (`workbench.action.gotoSymbol`), Go to Line (`workbench.action.gotoLine`), Next Problem (`editor.action.marker.next`).
- Language features depend on language extensions. The stock editor shows nothing for hover, rename or references without a provider.
- Settings: `editor.hover.*`, `editor.lightbulb.enabled`, `editor.codeActionsOnSave`, `editor.formatOnSave`, `editor.formatOnPaste`, `editor.formatOnType`, `editor.codeLens`, `editor.inlayHints.*`, `editor.links`, `editor.colorDecorators`, `editor.occurrencesHighlight`, `editor.renameOnType`, `editor.gotoLocation.*`, `editor.peekWidgetDefaultFocus`, all in `src\vs\editor\common\config\editorOptions.ts`.
- Cross-section: call hierarchy and type hierarchy open in views (`src\vs\workbench\contrib\callHierarchy`, `typeHierarchy`) and References view comes from `V\extensions\references-view`; both are left out here. Autocomplete (suggest, parameter hints, inline completions) is section 06.

### ED-5 Editor view and orientation aids

- Plain name: Editor view and orientation aids
- One sentence: How the text is drawn and how you keep your place: minimap, sticky scroll, breadcrumbs, code folding, word wrap, whitespace and indent guides, rulers, line numbers, font zoom, and warnings about odd characters.
- Tag: both
- Evidence: source
- Boundary: no
- Example features: minimap; sticky scroll; breadcrumbs bar with symbol and folder pickers; code folding (fold, unfold, fold level N).
- Proof paths:
  - `src\vs\editor\browser\viewParts\minimap\minimap.ts`, `rulers\`, `lineNumbers\`, `indentGuides\`, `whitespace\`, `overviewRuler\`, `glyphMargin\`, `linesDecorations\`, `currentLineHighlight\`, `viewCursors\`
  - `src\vs\editor\contrib\stickyScroll\browser\stickyScrollActions.ts` (`editor.action.toggleStickyScroll`, select next or previous sticky line, go to focused line, select editor)
  - `src\vs\editor\contrib\folding\browser\folding.ts` (20 fold actions: Fold, Unfold, Toggle Fold, Fold All, Fold Level N, Fold All Block Comments, Fold All Regions, Create Folding Range from Selection, Go to Parent Fold, Toggle Import Fold) and `src\vs\workbench\contrib\folding\browser\folding.contribution.ts`
  - `src\vs\workbench\browser\parts\editor\breadcrumbs.ts` (setting `breadcrumbs.enabled`, `breadcrumbs.filePath`, `breadcrumbs.symbolPath`, `breadcrumbs.symbolSortOrder`, `breadcrumbs.icons`, and 26 `breadcrumbs.show*` toggles, one per symbol kind), `breadcrumbsControl.ts` (13 commands: `breadcrumbs.toggle`, `focus`, `focusAndSelect`, `toggleToOn`, `focusNext`, `focusPrevious`, `selectFocused`, `revealFocused`, `selectEditor`, `copyPath` and 3 more), `breadcrumbsPicker.ts`, `breadcrumbsModel.ts`
  - `src\vs\workbench\contrib\codeEditor\browser\toggleMinimap.ts` (`editor.action.toggleMinimap`), `toggleWordWrap.ts` (`editor.action.toggleWordWrap`), `toggleRenderWhitespace.ts` (`editor.action.toggleRenderWhitespace` plus five `editor.action.renderWhitespace.*` modes), `toggleRenderControlCharacter.ts`
  - `src\vs\editor\contrib\fontZoom\browser\fontZoom.ts` (Increase, Decrease, Reset Editor Font Size), `unicodeHighlighter\browser\unicodeHighlighter.ts` (ambiguous, invisible and non-ASCII character highlights and their exclude options), `unusualLineTerminators\`, `sectionHeaders\` (MARK-style section headers), `readOnlyMessage\`, `placeholderText\`, `middleScroll\`, `longLinesHelper\`
  - `src\vs\workbench\contrib\codeEditor\browser\inspectEditorTokens\inspectEditorTokens.ts` (Developer: Inspect Editor Tokens and Scopes), `largeFileOptimizations.ts`, `accessibility\accessibility.ts` (`editor.action.toggleScreenReaderAccessibilityMode`), `src\vs\editor\contrib\toggleTabFocusMode\` (`editor.action.toggleTabFocusMode`)

Facts seen (for the feature pass):

- The View menu holds Appearance entries for Minimap, Breadcrumbs, Sticky Scroll, Render Whitespace, Word Wrap (section 01 owns the menu bar). The Command Palette lists the folding family.
- The breadcrumbs bar appears above the editor content and opens pickers for folders and symbols. The diff editor has its own breadcrumbs piece in `src\vs\editor\contrib\diffEditorBreadcrumbs\`.
- Settings: `editor.minimap.*`, `editor.stickyScroll.*`, `editor.folding*`, `editor.wordWrap*`, `editor.renderWhitespace`, `editor.guides.*`, `editor.bracketPairColorization.*`, `editor.rulers`, `editor.lineNumbers`, `editor.unicodeHighlight.*`, `editor.mouseWheelZoom`, `editor.cursorStyle`, `editor.scrollbar.*` in `src\vs\editor\common\config\editorOptions.ts`.
- Cross-section: Outline view and Timeline view (section 04), status bar indicators for line, column, spaces, wrap (section 09).

### ED-6 Diffs, merges and change review

- Plain name: Diffs, merges and change review
- One sentence: Editors that compare versions of text: the two-pane or inline diff editor, the multi-file diff editor, the three-way merge editor, and the in-file merge conflict markers with their CodeLens actions.
- Tag: both
- Evidence: source
- Boundary: yes (in Cursor this group also holds the agent's inline diffs, Review Changes and Multi File Edits; see the Cursor file. Stock VS Code has none of those)
- Example features: diff editor (side by side or inline, collapse unchanged regions, moved code, revert block); multi-file diff editor (collapse all, go to file); merge editor (Input 1, Input 2, Result, accept or toggle per conflict); merge conflict CodeLens (Accept Current, Accept Incoming, Accept Both, Compare Changes).
- Proof paths:
  - `src\vs\editor\browser\widget\diffEditor\diffEditorWidget.ts`, `diffEditor.contribution.ts` (Use Inline View When Space Is Limited, Show Moved Code Blocks, Revert Block, Revert Selection, Open Accessible Diff Viewer), `commands.ts`, `components\`, `features\`
  - `src\vs\workbench\browser\parts\editor\textDiffEditor.ts`, `binaryDiffEditor.ts`, `sideBySideEditor.ts`, `diffEditorCommands.ts` (`workbench.action.compareEditor.nextChange`, `previousChange`, `focusPrimarySide`, `focusSecondarySide`, `focusOtherSide`, `openSide`, `swapSides`), `src\vs\workbench\contrib\codeEditor\browser\diffEditorHelper.ts`
  - `src\vs\workbench\contrib\files\browser\fileActions.ts` (`workbench.files.action.compareFileWith` line 499, `compareNewUntitledTextFiles` line 794, `compareWithClipboard` line 823) and `fileConstants.ts` line 17 (`workbench.files.action.compareWithSaved`)
  - `src\vs\workbench\contrib\multiDiffEditor\browser\multiDiffEditor.ts`, `multiDiffEditorInput.ts`, `actions.ts` (`multiDiffEditor.goToFile`, `goToNextChange`, `goToPreviousChange`, `collapseAll`, `expandAll`), `scmMultiDiffSourceResolver.ts`, and the widget in `src\vs\editor\browser\widget\multiDiffEditor\`
  - `src\vs\workbench\contrib\mergeEditor\browser\mergeEditor.contribution.ts`, `commands\commands.ts` (Mixed Layout, Show Non-Conflicting Changes, Show Base, Go to Next or Previous Unhandled Conflict, Toggle Current Conflict from Left or Right, Compare Input 1 or 2 With Base, Accept All from Left or Right, Reset Result, Complete Merge `mergeEditor.acceptMerge`, Toggle Between Merge Editor Inputs), `view\mergeEditor.ts`, `view\conflictActions.ts`, `view\viewZones.ts`, `mergeMarkers\mergeMarkersController.ts`
  - `V\extensions\merge-conflict\package.json` (10 commands: `merge-conflict.accept.current`, `accept.incoming`, `accept.both`, `accept.selection`, `accept.all-current`, `accept.all-incoming`, `accept.all-both`, `next`, `previous`, `compare`; settings `merge-conflict.codeLens.enabled`, `decorators.enabled`, `autoNavigateNextConflict.enabled`, `diffViewPosition`)
  - `src\vs\workbench\contrib\notebook\browser\diff\notebookDiffEditor.ts` (notebook diff; also counted in ED-7)

Facts seen (for the feature pass):

- The diff editor reads many `diffEditor.*` options (renderSideBySide, ignoreTrimWhitespace, hideUnchangedRegions.enabled, experimental showMoves, maxComputationTime). The option definitions are in `src\vs\editor\common\config\diffEditor.ts` and `editorConfigurationSchema.ts`; the widget reads them in `src\vs\editor\browser\widget\diffEditor\diffEditorOptions.ts`. Editor title has an Inline View toggle (`editor.contribution.ts` `MenuId.EditorTitle`, `TOGGLE_DIFF_SIDE_BY_SIDE`).
- Source Control opens diffs and multi-file diffs through `MenuId.SCMResourceContext` actions and the SCM multi-diff resolver. Those entry points belong to section 04.
- Local History compare and restore are in `src\vs\workbench\contrib\localHistory\browser\localHistoryCommands.ts` (`workbench.action.localHistory.compareWithFile` and others). Their view is the Timeline (section 04).
- Peek-style diff on the gutter (quick diff) is in `src\vs\workbench\contrib\scm\browser\quickDiffWidget.ts` and is owned by section 04.

### ED-7 Notebooks, Interactive Window and REPL

- Plain name: Notebooks, Interactive Window and REPL
- One sentence: The cell-based editors: the notebook editor with its toolbar, kernel picker, cells and outputs, the notebook diff editor, the Interactive Window and the REPL editor.
- Tag: both
- Evidence: source
- Boundary: no
- Example features: notebook editor (code and markdown cells, run, kernel picker, outputs); notebook diff editor; Interactive Window (input box under the transcript, history); REPL editor.
- Proof paths:
  - `src\vs\workbench\contrib\notebook\browser\` folders `controller` (coreActions.ts, executeActions.ts, insertCellActions.ts, editActions.ts, cellOperations.ts, layoutActions.ts, sectionActions.ts, foldingController.ts), `view`, `viewParts` (notebookEditorToolbar.ts, notebookKernelView.ts, notebookKernelQuickPickStrategy.ts, notebookTopCellToolbar.ts, notebookEditorStickyScroll.ts, notebookOverviewRuler.ts), `diff`, `outputEditor`, `contrib` (cellCommands, cellStatusBar, find, format, multicursor, navigation, outline, profile, saveParticipants, undoRedo, kernelDetection, notebookVariables, troubleshoot, debug, gettingStarted)
  - about 40 `notebook.cell.*` command ids such as `notebook.cell.execute`, `notebook.cell.executeAndInsertBelow`, `notebook.cell.insertCodeCellBelow`, `notebook.cell.insertMarkdownCellAbove`, `notebook.cell.split`, `notebook.cell.joinAbove`, `notebook.cell.moveUp`, `notebook.cell.changeToMarkdown`, `notebook.cell.clearOutputs`, `notebook.cell.collapseAllCellInputs`; plus `notebook.execute`, `notebook.cancelExecution`, `notebook.interruptExecution`, `notebook.clearAllCellsOutputs`, `notebook.find`, `notebook.format`, `notebook.toggleLineNumbers`, `notebook.toggleCellToolbarPosition`, `notebook.centerActiveCell`
  - notebook diff: `src\vs\workbench\contrib\notebook\browser\diff\notebookDiffActions.ts` (`notebook.diff.action.next`, `previous`, `notebook.diff.cell.revertInput`, `notebook.diff.inline.toggle`, `notebook.diff.showOutputs`, `notebook.diff.showMetadata`)
  - `src\vs\workbench\contrib\interactive\browser\interactive.contribution.ts`, `interactiveEditor.ts`, `interactiveEditorInput.ts` (ids `_interactive.open` titled "Open Interactive Window", `interactive.execute`, `interactive.input.focus`, `interactive.history.previous`, `interactive.history.next`, `interactive.scrollToTop`, `interactive.scrollToBottom`)
  - `src\vs\workbench\contrib\replNotebook\browser\repl.contribution.ts`, `replEditor.ts`, `replEditorInput.ts` (ids `repl.execute`, `repl.input.focus`, `repl.focusLastItemExecuted`, `repl.focusLastReplOutput`)
  - `V\extensions\ipynb\package.json` (notebook type `jupyter-notebook`, commands New Jupyter Notebook `ipynb.newUntitledIpynb`, Open IPYNB File In Notebook Editor, Copy Cell Output, Open Cell Output in Text Editor), `V\extensions\notebook-renderers\package.json` (output renderers), `V\extensions\markdown-math\package.json` (notebook renderer for math)

Facts seen (for the feature pass):

- Notebook pane labels in V (`notebook.contribution.ts` lines 141 to 177): Notebook Editor, Notebook Diff Editor, Notebook Output Editor, and a second pane also labeled Notebook Diff Editor (the multi-file notebook diff, class `NotebookMultiTextDiffEditor`). The Interactive Window and REPL Editor panes are registered in `interactive.contribution.ts` and `repl.contribution.ts`.
- The notebook editor has its own find, multi-cursor, outline sections, folding, sticky scroll, variables view and debug hooks. The Variables and Outline views are side bar or panel parts (sections 04 and 07).
- Notebook inline chat (`notebook.cell.chat.*`) and Add Cell Output to Chat are agent entry points and are left out (chat).
- Settings: `notebook.*` and `interactiveWindow.*` settings are registered in `src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts` and `src\vs\workbench\contrib\interactive\browser\interactive.contribution.ts` (not counted).

### ED-8 Settings, shortcuts and configuration editors

- Plain name: Settings, shortcuts and configuration editors
- One sentence: The editor tabs where you change the product itself: the Settings editor (search, contents tree, user, workspace and folder scopes), the Keyboard Shortcuts editor, JSON settings files, the Profiles editor and the Workspace Trust editor.
- Tag: both
- Evidence: source
- Boundary: yes (in Cursor this group also holds Cursor Settings and Customize, which configure the agent; stock has no equivalent except a chat-side Agent Customizations Editor that sits under chat)
- Example features: Settings editor (search bar, table of contents, per-setting widgets and Modified indicators); Keyboard Shortcuts editor (search, record keys, show conflicts, edit When expression); open settings or shortcuts as JSON; Profiles editor.
- Proof paths:
  - `src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts` (editor pane label "Settings Editor 2"), `settingsTree.ts`, `settingsTreeModels.ts`, `settingsWidgets.ts`, `tocTree.ts`, `settingsLayout.ts` (contents tree: Commonly Used, Text Editor with Cursor, Find, Font, Formatting, Diff Editor, Multi-File Diff Editor, Minimap, Suggestions, Files; Workbench; Window; Chat; Features; and more), `settingsSearchMenu.ts`, `settingsEditorSettingIndicators.ts`, `preferencesSearch.ts`
  - `src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts` (pane "Keybindings Editor"), `keybindingWidgets.ts`, `keybindingsEditorContribution.ts` (`editor.action.defineKeybinding` inside `keybindings.json`), `keyboardLayoutPicker.ts`
  - `src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts` and `preferencesActions.ts` (ids `workbench.action.openSettings`, `openSettings2`, `openSettingsJson`, `openApplicationSettingsJson`, `openGlobalSettings`, `openRawDefaultSettings`, `openWorkspaceSettings`, `openWorkspaceSettingsFile`, `openFolderSettings`, `openFolderSettingsFile`, `openRemoteSettings`, `openAccessibilitySettings`, `openGlobalKeybindings`, `openGlobalKeybindingsFile`, `openDefaultKeybindingsFile`, `openKeyboardLayoutPicker`, `configureLanguageBasedSettings`)
  - keyboard shortcuts editor commands (in `keybindingsEditor.ts` and `preferences.contribution.ts`): `keybindings.editor.searchKeybindings`, `recordSearchKeys`, `showConflicts`, `defineKeybinding`, `defineWhenExpression`, `addKeybinding`, `removeKeybinding`, `resetKeybinding`, `copyKeybindingEntry`, `copyCommandKeybindingEntry`, `copyCommandTitle`, `showDefaultKeybindings`, `showUserKeybindings`, `showExtensionKeybindings`, `toggleSortByPrecedence`, `clearSearchResults`, `clearSearchHistory`
  - settings editor commands: `settings.action.search`, `focusSettingsList`, `focusSettingControl`, `focusTOC`, `focusLevelUp`, `showContextMenu`, `clearSearchResults`, `suggestFilters`, `toggleAiSearch`
  - `src\vs\workbench\contrib\preferences\browser\preferencesEditor.ts` (pane "Preferences Editor", the JSON-with-defaults split view) and `preferencesRenderers.ts` (hover and edit widgets inside settings JSON)
  - `src\vs\workbench\contrib\userDataProfile\browser\userDataProfilesEditor.ts` (pane "Profiles Editor"; command `workbench.profiles.actions.manageProfiles`)
  - `src\vs\workbench\contrib\workspace\browser\workspaceTrustEditor.ts` and `workspace.contribution.ts` line 731 (pane "Workspace Trust Editor"; command `workbench.trust.manage`)
  - `src\vs\workbench\contrib\keybindings\browser\keybindings.contribution.ts` (`workbench.action.toggleKeybindingsLog`, "Toggle Keyboard Shortcuts Troubleshooting")

Facts seen (for the feature pass):

- Menu entries live in the File > Preferences submenu (section 01) and the gear menu in the Activity Bar (section 03). Their targets are the commands above.
- Settings Sync indicators inside the Settings editor come from `src\vs\workbench\contrib\preferences\browser\settingsEditorSettingIndicators.ts` and the sync status entry is owned by section 09.
- Cross-section: the Manage menu (section 03), Quick Access pickers for settings (section 11).

### ED-9 Welcome, empty and first-run screens

- Plain name: Welcome, empty and first-run screens
- One sentence: The pages and hints you see when nothing is open or when starting out: the Welcome page with walkthroughs, the Interactive Playground, the empty editor group watermark, the hint in an empty text file, and first-run onboarding.
- Tag: both
- Evidence: source
- Boundary: no
- Example features: Welcome page (start entries, recent list, walkthrough cards); a walkthrough with steps and a mark-complete control; editor group watermark with keyboard shortcuts; empty text editor hint (select a language, or start with a template).
- Proof paths:
  - `src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.ts` (page and walkthrough detail view), `gettingStartedService.ts`, `gettingStartedContent.ts` (start entries New File..., Open..., Clone Git Repository..., Connect to..., Open Tunnel..., Generate New Workspace...; built-in walkthroughs Setup, Beginner, Accessibility, Web), `gettingStartedExtensionPoint.ts` (walkthroughs contributed by extensions), `startupPage.ts` (setting `workbench.startupEditor`, line 317 of `gettingStarted.contribution.ts`), `gettingStartedAccessibleView.ts`
  - ids: `workbench.action.openWalkthrough`, `welcome.showAllWalkthroughs`, `welcome.markStepComplete`, `welcome.markStepIncomplete`, `welcome.goBack`, `welcome.showNewFileEntries`, `welcome.newWorkspaceChat`, `resetGettingStartedProgress`
  - `src\vs\workbench\contrib\welcomeWalkthrough\browser\walkThrough.contribution.ts` (pane "Playground", command `workbench.action.showInteractivePlayground`, menu "Editor Playground"), `editor\vs_code_editor_walkthrough.ts`, `walkThroughPart.ts`
  - `src\vs\workbench\contrib\welcomeAgentSessions\browser\agentSessionsWelcome.contribution.ts` (pane "Agent Sessions Welcome", command `workbench.action.openAgentSessionsWelcome`, `workbench.startupEditor` value `agentSessionsWelcomePage`)
  - `src\vs\workbench\contrib\welcomeOnboarding\browser\welcomeOnboarding.contribution.ts` and `onboardingVariationA.ts` (command `workbench.action.welcomeOnboarding2026`)
  - `src\vs\workbench\browser\parts\editor\editorGroupWatermark.ts` (watermark entries Show All Commands, Go to File, Open File, Open Folder, Open Recent, New Untitled Text File, Find in Files, Toggle Terminal, Start Debugging, Open Settings, Open Chat; menu `MenuId.EditorGroupWatermarkToolbar`; setting `workbench.tips.enabled`), `editorPlaceholder.ts`
  - `src\vs\workbench\contrib\codeEditor\browser\emptyTextEditorHint\emptyTextEditorHint.ts` (setting `workbench.editor.empty.hint`, `workbench.contribution.ts` line 165)
  - `src\vs\workbench\contrib\welcomeViews\common\newFile.contribution.ts` (the New File... picker, `workbench.action.files.newUntitledFile` "Text File", File > New File entries)
  - `src\vs\workbench\contrib\surveys\browser\survey.contribution.ts` and `surveyEditorPane.ts` (pane "Survey")

Facts seen (for the feature pass):

- `workbench.startupEditor` values in V: none, welcomePage, readme, newUntitledFile, welcomePageInEmptyWorkbench, terminal, agentSessionsWelcomePage.
- Banner-style welcome pieces (`src\vs\workbench\contrib\welcomeBanner`) are section 02. Welcome content inside empty tree views (`welcomeViews\common\viewsWelcome*.ts`) is section 04.
- The Welcome page and Agent Sessions Welcome page bind to chat sentiment and sign-in state (`gettingStarted.contribution.ts` line 73), so their content can shift per user. The pages are in scope; the agent behind them is not.

### ED-10 Viewers, previews and web editors

- Plain name: Viewers, previews and web editors
- One sentence: Editors that show a file or a page without the text editor: image, audio and video viewers, the binary file editor, the Markdown preview, webview and custom editor tabs from extensions, the Simple Browser, the integrated Browser, and a few special panes.
- Tag: both
- Evidence: source
- Boundary: yes (the integrated Browser can be driven by the agent and can send picked elements to chat; the viewer itself is in scope)
- Example features: image viewer (zoom, copy) and binary file editor ("Open Anyway"); Markdown preview (open to the side, locked preview, scroll sync); integrated Browser tab (URL bar, tab list, zoom, device emulation, favorites, history); Simple Browser panel.
- Proof paths:
  - `V\extensions\media-preview\package.json` (custom editors `imagePreview.previewEditor` "Image Preview", `vscode.audioPreview` "Audio Preview", `vscode.videoPreview` "Video Preview"; commands `imagePreview.zoomIn`, `zoomOut`, `copyImage`, `reopenAsPreview`, `reopenAsText`; settings `mediaPreview.video.autoPlay`, `mediaPreview.video.loop`)
  - `src\vs\workbench\contrib\files\browser\editors\binaryFileEditor.ts`, `src\vs\workbench\browser\parts\editor\binaryEditor.ts` (pane "Binary File Editor", setting `workbench.editor.defaultBinaryEditor`), `editorPlaceholder.ts` (placeholder pages for errors, large files, untrusted workspaces)
  - `V\extensions\markdown-language-features\package.json` (preview commands `markdown.showPreview`, `showPreviewToSide`, `showLockedPreviewToSide`, `showSource`, `showPreviewSecuritySelector`, `preview.refresh`, `preview.toggleLock`, `togglePreview`, `reopenAsPreview`; custom editors "Markdown Preview" and "Markdown Editor (Experimental)"; 41 settings)
  - `V\extensions\mermaid-markdown-features\package.json` (Reset Pan and Zoom, Open Diagram in Editor, Copy Diagram Source; settings `markdown-mermaid.*`)
  - `src\vs\workbench\contrib\webviewPanel\browser\webviewEditor.ts`, `webviewCommands.ts` (`editor.action.webvieweditor.showFind`, `hideFind`, `findNext`, `findPrevious`, `workbench.action.webview.reloadWebviewAction`), `src\vs\workbench\contrib\webview\electron-browser\webviewCommands.ts` (`workbench.action.webview.openDeveloperTools`), `src\vs\workbench\contrib\customEditor\browser\customEditors.ts` (custom editor contribution point and Reopen Editor With)
  - `V\extensions\simple-browser\package.json` (command `simpleBrowser.show`; setting `simpleBrowser.focusLockIndicator.enabled`)
  - `src\vs\workbench\contrib\browserView\electron-browser\browserEditor.ts` (pane "Browser"), `browserView.contribution.ts`, `features\browserNavigationFeatures.ts` (Open in External Browser, Browser Settings), `browserEditorZoomFeature.ts` (`workbench.action.browser.zoomIn`, `zoomOut`, `resetZoom`), `browserEditorEmulationFeatures.ts` (`workbench.action.browser.toggleDeviceEmulation`, `toggleMobileEmulation`, `pickDevicePreset`, `setUserAgent`, `resetEmulation`), `browserFavoritesFeature.ts` (`browser.toggleFavorite`), `browserHistoryFeature.ts`, `browserEditorFindFeature.ts`, `browserPermissionsFeature.ts`, `browserTabManagementFeatures.ts`, `browserDevToolsFeature.ts`, `browserDataStorageFeatures.ts` (setting `workbench.browser.dataStorage`), `widgets\browserUrlBarWidget.ts`
  - `src\vs\workbench\contrib\imageCarousel\browser\imageCarouselEditor.ts` (pane "Images Preview", command `workbench.action.chat.openImageInCarousel`)
  - `src\vs\workbench\contrib\debug\browser\disassemblyView.ts` (pane "Disassembly", registered in `src\vs\workbench\contrib\debug\browser\debug.contribution.ts` line 480)

Facts seen (for the feature pass):

- Settings for the integrated Browser: `workbench.browser.pageZoom`, `searchEngine`, `favorites`, `showInTitleBar`, `openLocalhostLinks`, `newTabPlacement`, `dataStorage`, `enableChatTools`, `experimentalUserTools.enabled`, `sendElementsToChat.attachImages`. The agent tools in `browserView\electron-browser\tools\` (click, drag, hover, navigate, screenshot, type, read, run Playwright code) are agent plumbing and are left out.
- In V the Markdown preview toggles with Shift+Ctrl+V and opens to the side with Ctrl+K V (`markdown-language-features\package.json` keybindings). `markdown.togglePreview` also matches the languages prompt, instructions, chatagent and skill.
- The Markdown extension also adds a "Markdown Editor (Experimental)" custom editor in stock (`vscode.markdown.editor`), selected for `*.md` with priority `option`.

## Tag notes for later feature ids (from the Cursor bundle checks)

The group tags above are `both` because most of each group is present in Cursor. These stock items were searched in the Cursor bundle and not found, so at feature level they are `vscode` only (details and counts in `cursor\05-editor.md`):

- ED-1: modal editor part and `workbench.editor.useModal`; compact auxiliary window commands; `workbench.action.moveEditorToStart`, `moveEditorToEnd`, `clearEditorHistoryWithoutConfirm`; settings `workbench.editor.swipeToNavigate`, `restoreEditors`, `showTabIndex`.
- ED-2: `editor.action.reverseLines`, the five `editor.action.renderWhitespace.*` commands, `editor.action.announceCursorPosition`, `workbench.action.gotoOffset`.
- ED-6: `multiDiffEditor.goToNextChange`, `goToPreviousChange`, `mergeEditor.acceptAllCombination`, `mergeEditor.toggleBetweenInputs`.
- ED-8: the "Preferences Editor" pane label and `workbench.action.openKeyboardLayoutPicker`.
- ED-9: Interactive Playground, Agent Sessions Welcome, Welcome Onboarding 2026, Survey pane, and the built-in walkthrough step ids.
- ED-10: the stock integrated Browser (all 14 of its ids), Images Preview (image carousel), `mermaid-markdown-features`, the `markdown.reopenAsPreview`, `reopenAsSource`, `togglePreview` commands.

## Built-in extension contributions that belong in this section (V manifests)

| Extension | Group | What it contributes in V | Path |
|-|-|-|-|
| markdown-language-features | ED-10 | 16 commands, 2 keybindings, 41 settings, 2 custom editors, menus on editor title, explorer context, webview context, command palette, `modalEditor/editorTitle` | `V\extensions\markdown-language-features\package.json` |
| merge-conflict | ED-6 | 10 commands, CodeLens and decorations for conflict markers, 4 settings, menus on editor title and `scm/resourceState/context` | `V\extensions\merge-conflict\package.json` |
| simple-browser | ED-10 | 1 command (`simpleBrowser.show`), 1 setting | `V\extensions\simple-browser\package.json` |
| search-result | ED-3 | language `search-result` and a grammar only; no commands | `V\extensions\search-result\package.json` |
| media-preview | ED-10 | 3 custom editors (image, audio, video), 5 commands, 2 settings | `V\extensions\media-preview\package.json` |
| mermaid-markdown-features | ED-10 | 3 commands, 8 settings, markdown-it plugin and previews | `V\extensions\mermaid-markdown-features\package.json` |
| ipynb, notebook-renderers, markdown-math | ED-7 | notebook type `jupyter-notebook`, 6 commands, output renderers | `V\extensions\ipynb\package.json`, `V\extensions\notebook-renderers\package.json`, `V\extensions\markdown-math\package.json` |
| emmet | ED-2 | 23 commands, 12 settings | `V\extensions\emmet\package.json` |

## Editor panes registered in V (inventory used to build the groups)

| Pane label | Source | Group |
|-|-|-|
| Text Editor, Text File Editor | `src\vs\workbench\browser\parts\editor\editor.contribution.ts` lines 81 to 93, `src\vs\workbench\contrib\files\browser\files.contribution.ts` line 63 | ED-1, ED-2 |
| Text Diff Editor, Binary Diff Editor, Side by Side Editor | `editor.contribution.ts` lines 93 to 118 | ED-6 |
| Binary File Editor | `files.contribution.ts` line 74 | ED-10 |
| Notebook Editor, Notebook Diff Editor (two panes), Notebook Output Editor | `notebook.contribution.ts` lines 141 to 177 | ED-7 |
| Interactive Window, REPL Editor | `interactive.contribution.ts`, `repl.contribution.ts` | ED-7 |
| Settings Editor 2, Preferences Editor, Keybindings Editor | `preferences.contribution.ts` lines 75 to 115 | ED-8 |
| Profiles Editor, Workspace Trust Editor | `userDataProfile.ts` line 113, `workspace.contribution.ts` line 731 | ED-8 |
| Welcome, Playground, Agent Sessions Welcome, Survey | `gettingStarted.contribution.ts` line 118, `walkThrough.contribution.ts` line 21, `agentSessionsWelcome.contribution.ts` line 37, `survey.contribution.ts` line 30 | ED-9 |
| Search Editor | `searchEditor.contribution.ts` line 108 | ED-3 |
| Multi Diff Editor, Merge Editor | `multiDiffEditor.contribution.ts` line 34, `mergeEditor.contribution.ts` line 30 | ED-6 |
| webview editor, Images Preview, Browser, Disassembly | `webviewPanel.contribution.ts` line 24, `imageCarousel.contribution.ts` line 57, `browserView.contribution.ts` line 41, `debug.contribution.ts` line 480 | ED-10 |
| Issue Reporter, Process Explorer | `issue.contribution.ts` line 70 (electron-browser), `processExplorer.contribution.ts` (electron-browser and browser) | left out, unowned (see below) |
| Extension, Running Extensions, MCP Server | extensions and mcp contribs | left out (extension system, MCP) |
| Chat, Debug View, Agent Plugin, Models Management Editor, Agent Customizations Editor | chat contribs | left out (chat) |

## Left out and why

- Autocomplete: suggest list, inline completions and ghost text, parameter hints, signature help, snippet placeholders (`editor\contrib\suggest`, `inlineCompletions`, `parameterHints`, `snippet`) - section 06.
- Extensions editor, Running Extensions editor, MCP Server editor - extension system and MCP are out of scope.
- Chat editor, Debug View (chat), Agent Plugin editor, Models Management Editor, Agent Customizations Editor, inline chat, notebook cell chat (`notebook.cell.chat.*`), Add to Chat and Add Cell Output to Chat - chat and agent surfaces are out of scope.
- Source Control: git extension, quick diff gutter and peek, SCM multi-diff entry points, Timeline and Local History views - section 04.
- Explorer, Open Editors, Outline, References view (`V\extensions\references-view`), Call Hierarchy and Type Hierarchy views, views welcome content - section 04 and the panel sections.
- Status bar entries that come from `editorStatus.ts`, `languageStatus`, indentation, line and column, encoding and EOL, plus the Edit Telemetry AI stats entry - section 09.
- Menu bar items for Edit, Selection, Go, View > Appearance, and layout toggles such as `workbench.action.toggleEditorVisibility` - section 01.
- Go to Line, Go to Symbol and Quick Open pickers - section 11 (only the commands are named in ED-4).
- Terminal in the editor area and the Terminal editor tab - section 07.
- Issue Reporter and Process Explorer editor panes - they are real panes in V but fit no one section; flagged for a later pass (Help menu territory).
- Debug editor widgets (breakpoint glyphs, inline values, exception widget, debug hover), comment thread widgets (`workbench\contrib\comments`) and test run glyphs (`workbench\contrib\testing`) - embedded in the text editor but belong to Debug, Comments and Testing features that none of the 12 sections owns; flagged as a coverage gap.
- Accessible view and accessibility help dialogs (`editor.action.accessibleView*`) - cross-cutting; flagged as a coverage gap.
- Language support extensions (typescript, json, css, html, python, and so on), the `git` extension and `configuration-editing` - out of scope except as hosts of UI listed above.

## Newer than 1.128 (seen only in N, not read in depth)

- `N\src\vs\workbench\browser\parts\editor\editorAddTabControl.ts`, `editorHeaderControl.ts`, `editorTypePicker.ts`, `connectedTabClipping.ts`, `diffEditorCommandsService.ts`, `diffEditor.workbench.contribution.ts` - new pieces of the tab and editor header area.
- `N\src\vs\workbench\contrib\modernUI\` (connected editor tabs, frosted glass, editor border CSS) and `N\extensions\theme-modern-icons`.
- `N\src\vs\workbench\contrib\mediaPreview\` - image, audio and video previews moved into core (`imagePreviewEditor.ts`, `mediaPreviewSaveAs.ts`, `mediaPreviewStatusBar.ts`); in V they are the `media-preview` extension.
- `N\src\vs\workbench\contrib\multiDiffEditor\browser\multiDiffEditorFileList.ts`, `multiDiffEditorWithFileListWidget.ts`, and `N\src\vs\editor\browser\widget\multiDiffEditor\compressedVirtualizedScrollView.ts`, `virtualizedItemManager.ts` - a file list beside the multi-file diff and a virtualized scroll layout.
- `N\src\vs\workbench\contrib\browserView\browser\browserWelcome.ts`, `electron-browser\features\browserAutoReloadFeatures.ts`, `tools\listBrowserPagesTool.ts`.
- `N\src\vs\editor\contrib\dropOrPasteInto\browser\pasteEditSession.ts`, `N\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellEditorOverflowWidgets.ts`.
- `N\extensions\markdown-language-features\package.json` has 64 commands and 81 keybindings (V has 16 and 2).
- Setting `workbench.editor.tabActionReserveSpace` exists in N (`workbench.contribution.ts` line 211) and not in V.
- `N\src\vs\workbench\contrib\canvases` (cut from scope) and `N\src\vs\workbench\contrib\github` (not in scope).

## Could not verify

- Exact registration site of the `search.searchEditor.*` settings other than `doubleClickBehaviour`.
- What the line-number gutter context menu holds in V: `src\vs\workbench\contrib\codeEditor\browser\editorLineNumberMenu.ts` was seen but not read in full.
- Counts of `notebook.*`, `editor.*` and `workbench.editor.*` settings are given only where a file range or list was read.
- The default keybindings for the commands above were not collected; feature pass should read `keybinding:` fields.
