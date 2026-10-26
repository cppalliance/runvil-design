Group: ED-4 Language help in the editor. Section: Editor. Date: 2026-10-25. Pass: 2.
Features: 42.

### ED-4.1 Hover on code
- Tag: both
- Evidence: source
- Boundary: no
- What: Rest the pointer on code to see a floating card with docs, types and links that stays open when you move onto it, with plus and minus buttons for more or less detail and, in stock, a Copy button on code blocks.
- Commands: editor.action.increaseHoverVerbosityLevel; editor.action.decreaseHoverVerbosityLevel; editor.action.hideLongLineWarningHover
- Menus: none found
- Keys: none found
- Settings: editor.hover.enabled; editor.hover.delay; editor.hover.sticky; editor.hover.hidingDelay; editor.hover.above; editor.hover.showLongLineWarning; editor.hover
- Cursor: changed (hover and verbosity ids found; no hover Copy button module, no `editor.hover.showLongLineWarning` setting and no `editor.action.hideLongLineWarningHover` id)
- Source: V\src\vs\editor\contrib\hover\browser\contentHoverController.ts; V\src\vs\editor\contrib\hover\browser\markdownHoverParticipant.ts; V\src\vs\editor\contrib\hover\browser\hoverCopyButton.ts; V\src\vs\editor\contrib\hover\browser\hoverContribution.ts; V\src\vs\editor\common\config\editorOptions.ts (lines 2360 to 2400); C bundle (modules hoverActionIds, markdownHoverParticipant with 9 strings, no hoverCopyButton)

### ED-4.2 Show, focus and scroll the hover with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: Press the hover key to open the hover card at the text cursor without the mouse, press it again to focus the card, then scroll it with arrow, page, Home and End keys, and hide it with Hide Hover.
- Commands: editor.action.showHover; editor.action.hideHover; editor.action.scrollUpHover; editor.action.scrollDownHover; editor.action.scrollLeftHover; editor.action.scrollRightHover; editor.action.pageUpHover; editor.action.pageDownHover; editor.action.goToTopHover; editor.action.goToBottomHover
- Menus: none found
- Keys: Ctrl+K Ctrl+I (editorTextFocus) for Show or Focus Hover; Up, Down, Left, Right (hoverFocused) scroll; PageUp or Alt+Up and PageDown or Alt+Down (hoverFocused); Home or Ctrl+Up and End or Ctrl+Down (hoverFocused)
- Settings: none found
- Cursor: changed (all ten ids found as quoted strings; the Ctrl+K chord start is replaced, so Show or Focus Hover is Ctrl+M Ctrl+I on Windows and Linux and Cmd+R Cmd+I on Mac)
- Source: V\src\vs\editor\contrib\hover\browser\hoverActions.ts (lines 28 to 430); V\src\vs\editor\contrib\hover\browser\hoverActionIds.ts; C bundle (module hoverActionIds, hoverActions with 24 strings)

### ED-4.3 Go to Definition, and open it to the side
- Tag: both
- Evidence: source
- Boundary: no
- What: Jump to where the symbol under the cursor is defined, or open that definition in a split beside the current editor, with a peek or a list when there are several results depending on settings.
- Commands: editor.action.revealDefinition; editor.action.revealDefinitionAside
- Menus: EditorContext (navigation); MenubarGoMenu (4_symbol_nav)
- Keys: F12 (editorTextFocus); Ctrl+K F12 (editorTextFocus); web only: Ctrl+F12 and Ctrl+K Ctrl+F12
- Settings: editor.gotoLocation.multiple; editor.gotoLocation.multipleDefinitions; editor.gotoLocation.alternativeDefinitionCommand
- Cursor: changed (both ids found; F12 and the context and Go menu entries match stock, but Open Definition to the Side starts with Ctrl+M on Windows and Linux and Cmd+R on Mac instead of Ctrl+K)
- Source: V\src\vs\editor\contrib\gotoSymbol\browser\goToCommands.ts (lines 276 to 341); V\src\vs\editor\contrib\gotoSymbol\browser\symbolNavigation.ts; V\src\vs\editor\common\config\editorOptions.ts (lines 2214 to 2270); C bundle (module vs/editor/contrib/gotoSymbol/browser/goToCommands with 39 strings)

### ED-4.4 Ctrl+click definition link and the definition preview hover
- Tag: both
- Evidence: source
- Boundary: no
- What: Hold Ctrl (Cmd on Mac) over a symbol to underline it as a link and preview its definition in the hover, click to jump, Ctrl+Alt+click to open it to the side, or run Show Definition Preview Hover to get the same preview from the keyboard.
- Commands: editor.action.showDefinitionPreviewHover
- Menus: none found
- Keys: none found
- Settings: editor.definitionLinkOpensInPeek; editor.multiCursorModifier; editor.mouseMiddleClickAction
- Cursor: present (editor.action.showDefinitionPreviewHover found; `definitionLinkOpensInPeek` option name found)
- Source: V\src\vs\editor\contrib\gotoSymbol\browser\link\goToDefinitionAtPosition.ts; V\src\vs\editor\contrib\gotoSymbol\browser\link\clickLinkGesture.ts (lines 101 to 112); V\src\vs\editor\contrib\hover\browser\hoverActions.ts (lines 108 to 150); C bundle (modules goToDefinitionAtPosition, hoverActionIds)

### ED-4.5 Go to Declaration
- Tag: both
- Evidence: source
- Boundary: no
- What: Jump to the place a symbol is declared, which can differ from where it is defined in languages that split the two.
- Commands: editor.action.revealDeclaration
- Menus: EditorContext (navigation); MenubarGoMenu (4_symbol_nav)
- Keys: none found
- Settings: editor.gotoLocation.multipleDeclarations; editor.gotoLocation.alternativeDeclarationCommand
- Cursor: present (editor.action.revealDeclaration found)
- Source: V\src\vs\editor\contrib\gotoSymbol\browser\goToCommands.ts (lines 398 to 445); V\src\vs\editor\common\config\editorOptions.ts (lines 2234 to 2270); C bundle (module goToCommands)

### ED-4.6 Go to Type Definition
- Tag: both
- Evidence: source
- Boundary: no
- What: Jump from a variable or expression to the definition of its type.
- Commands: editor.action.goToTypeDefinition
- Menus: EditorContext (navigation); MenubarGoMenu (4_symbol_nav)
- Keys: none found
- Settings: editor.gotoLocation.multipleTypeDefinitions; editor.gotoLocation.alternativeTypeDefinitionCommand
- Cursor: present (editor.action.goToTypeDefinition found)
- Source: V\src\vs\editor\contrib\gotoSymbol\browser\goToCommands.ts (lines 485 to 520); V\src\vs\editor\common\config\editorOptions.ts (lines 2230 to 2270); C bundle (module goToCommands)

### ED-4.7 Go to Implementations
- Tag: both
- Evidence: source
- Boundary: no
- What: Jump from an interface, abstract member or base method to the code that implements it, with a peek or list when there are several.
- Commands: editor.action.goToImplementation
- Menus: EditorContext (navigation); MenubarGoMenu (4_symbol_nav)
- Keys: Ctrl+F12 (editorTextFocus)
- Settings: editor.gotoLocation.multipleImplementations; editor.gotoLocation.alternativeImplementationCommand
- Cursor: present (editor.action.goToImplementation found; Ctrl+F12 read in the bundle)
- Source: V\src\vs\editor\contrib\gotoSymbol\browser\goToCommands.ts (lines 571 to 606); V\src\vs\editor\common\config\editorOptions.ts (lines 2238 to 2270); C bundle (module goToCommands)

### ED-4.8 Go to References
- Tag: both
- Evidence: source
- Boundary: no
- What: Find every use of the symbol under the cursor and jump to it, or open them together in the peek window when there is more than one.
- Commands: editor.action.goToReferences; editor.cancelOperation
- Menus: EditorContext (navigation); MenubarGoMenu (4_symbol_nav)
- Keys: Shift+F12 (editorTextFocus)
- Settings: editor.gotoLocation.multipleReferences; editor.gotoLocation.alternativeReferenceCommand
- Cursor: present (editor.action.goToReferences found)
- Source: V\src\vs\editor\contrib\gotoSymbol\browser\goToCommands.ts (lines 662 to 700); V\src\vs\editor\contrib\gotoSymbol\browser\referencesModel.ts; V\src\vs\editor\common\config\editorOptions.ts (lines 2242 to 2270); C bundle (module goToCommands, referencesModel with 8 strings)

### ED-4.9 Peek commands (Peek submenu)
- Tag: both
- Evidence: source
- Boundary: no
- What: Open a definition, declaration, type definition, implementation or the references inside an inline panel under the current line instead of leaving the file, from the Peek submenu of the right-click menu.
- Commands: editor.action.peekDefinition; editor.action.peekDeclaration; editor.action.peekTypeDefinition; editor.action.peekImplementation; editor.action.referenceSearch.trigger; editor.action.findReferences
- Menus: EditorContext (submenu titled Peek); EditorContextPeek (peek)
- Keys: Alt+F12 for Peek Definition (Ctrl+Shift+F10 on Linux); Ctrl+Shift+F12 for Peek Implementations
- Settings: editor.gotoLocation.multiple
- Cursor: present (all five ids found)
- Source: V\src\vs\editor\contrib\gotoSymbol\browser\goToCommands.ts (lines 42 to 49, 341 to 372, 447 to 460, 525 to 545, 611 to 640, 702 to 725); V\src\vs\platform\actions\common\actions.ts (EditorContextPeek); C bundle (module goToCommands)

### ED-4.10 Peek view window
- Tag: both
- Evidence: source
- Boundary: no
- What: The inline panel that peek and Go to References open under the current line, with a title bar showing the file and result count, a results tree beside an editable preview editor, keys to step through results, open one to the side, swap focus, and close.
- Commands: togglePeekWidgetFocus; goToNextReference; goToPreviousReference; closeReferenceSearch; revealReference; openReferenceToSide; openReference; editor.action.peekLocations
- Menus: none found
- Keys: Ctrl+K F2 (toggle focus); F4 or F12 (next result, referenceSearchVisible or inReferenceSearchEditor); Shift+F4 or Shift+F12 (previous result); Escape or Shift+Escape (close, unless editor.stablePeek); Enter (open the focused result); Ctrl+Enter (open to the side, Ctrl+Enter with the Control key on Mac)
- Settings: editor.peekWidgetDefaultFocus; editor.stablePeek; editor.definitionLinkOpensInPeek
- Cursor: changed (togglePeekWidgetFocus, goToNextReference, goToPreviousReference, closeReferenceSearch, revealReference, openReferenceToSide and editor.stablePeek all found; the toggle-focus chord is Ctrl+M F2 on Windows and Linux and Cmd+R F2 on Mac instead of Ctrl+K F2)
- Source: V\src\vs\editor\contrib\peekView\browser\peekView.ts; V\src\vs\editor\contrib\gotoSymbol\browser\peek\referencesController.ts (lines 315 to 430); V\src\vs\editor\contrib\gotoSymbol\browser\peek\referencesWidget.ts; V\src\vs\editor\contrib\gotoSymbol\browser\peek\referencesTree.ts; V\src\vs\editor\common\config\editorConfigurationSchema.ts (line 89); C bundle (modules referencesController, referencesWidget, referencesTree, peekView with 17 strings)

### ED-4.11 Open peeked references in chat
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: A button at the left of the peek window title bar labeled Open in Chat that makes a new chat and adds every peeked reference with a few lines of context; boundary because the result goes to the agent chat.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/editor/contrib/gotoSymbol/browser/peek/referencesWidget, nls index 1222 key label.composer "Open in Chat", action id peekview.composer, class peekview-composer-button, method _openReferencesInComposer; stock has 3 strings in this module, Cursor 4)

### ED-4.12 Rename Symbol, with preview
- Tag: both
- Evidence: source
- Boundary: no
- What: Press F2 on a symbol to type a new name in a small box at the cursor, then Enter renames it everywhere or Ctrl+Enter shows the changes in a preview first, with a button for generated name suggestions when a provider supplies them.
- Commands: editor.action.rename; acceptRenameInput; acceptRenameInputWithPreview; cancelRenameInput; focusNextRenameSuggestion; focusPreviousRenameSuggestion
- Menus: EditorContext (1_modification)
- Keys: F2 (editorTextFocus); Enter; Ctrl+Enter (when editor.rename.enablePreview is on); Escape or Shift+Escape; Down and Up to move through name suggestions
- Settings: editor.rename.enablePreview
- Cursor: present (all six ids found; the rename widget module has 7 strings, the same as stock)
- Source: V\src\vs\editor\contrib\rename\browser\rename.ts (lines 361 to 490); V\src\vs\editor\contrib\rename\browser\renameWidget.ts; C bundle (modules rename with 11 strings, renameWidget with 7)

### ED-4.13 Code action lightbulb
- Tag: both
- Evidence: source
- Boundary: no
- What: A lightbulb icon shows at the cursor or in the gutter when fixes or refactors are available, with a different icon for an auto fix or an AI fix, and clicking it opens the code actions menu.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.lightbulb.enabled; editor.codeActions.triggerOnFocusChange; editor.lightbulb
- Cursor: present (editor.lightbulb.enabled and editor.codeActions.triggerOnFocusChange found; module lightBulbWidget has the same 9 strings as stock)
- Source: V\src\vs\editor\contrib\codeAction\browser\lightBulbWidget.ts; V\src\vs\editor\contrib\codeAction\browser\codeActionController.ts; V\src\vs\editor\contrib\codeAction\browser\codeActionModel.ts; V\src\vs\editor\contrib\codeAction\browser\codeActionContributions.ts (lines 50 to 62); V\src\vs\editor\common\config\editorOptions.ts (lines 3091 to 3100); C bundle (module lightBulbWidget)

### ED-4.14 Code actions menu and Quick Fix
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+. to open a list of fixes and refactors at the cursor, grouped under headers such as Quick Fix and Extract, pick with the keyboard, preview with Ctrl+Enter, and in stock filter the list and collapse its sections.
- Commands: editor.action.quickFix; editor.action.codeAction; hideCodeActionWidget; selectPrevCodeAction; selectNextCodeAction; acceptSelectedCodeAction; previewSelectedCodeAction; collapseSectionCodeAction; expandSectionCodeAction; toggleSectionCodeAction; clearFilterCodeActionWidget
- Menus: InlineChatEditorAffordance (1_quickfix, stock only)
- Keys: Ctrl+. (textInputFocus) opens Quick Fix; Escape or Shift+Escape hides; Up and Down move; Enter or Ctrl+. applies; Ctrl+Enter previews; Left, Right and Space work on section headers
- Settings: editor.codeActionWidget.showHeaders; editor.codeActionWidget.includeNearbyQuickFixes
- Cursor: changed (quick fix, codeAction, hide, select, accept and preview ids found; no section collapse, expand, toggle or filter ids, no filter-box strings, and Quick Fix is a plain editor action with no inline chat menu entry)
- Source: V\src\vs\editor\contrib\codeAction\browser\codeActionCommands.ts (lines 68 to 130); V\src\vs\editor\contrib\codeAction\browser\codeActionMenu.ts (lines 23 to 33); V\src\vs\editor\contrib\codeAction\browser\codeActionController.ts (lines 382 to 455); V\src\vs\platform\actionWidget\browser\actionWidget.ts (lines 245 to 430); V\src\vs\platform\actionWidget\browser\actionList.ts (lines 36 and 37); C bundle (modules codeActionCommands with 29 strings, codeActionMenu with 8, actionWidget with 7, actionList with 4)

### ED-4.15 Refactor
- Tag: both
- Evidence: source
- Boundary: no
- What: Open a menu of refactorings for the current selection, such as extract, inline, rewrite and move, from the keyboard or the right-click menu, then run one or preview it.
- Commands: editor.action.refactor; editor.action.codeAction
- Menus: EditorContext (1_modification)
- Keys: Ctrl+Shift+R (textInputFocus; the Control key plus Shift+R on Mac)
- Settings: none found
- Cursor: present (editor.action.refactor and editor.action.codeAction found)
- Source: V\src\vs\editor\contrib\codeAction\browser\codeActionCommands.ts (lines 97 to 180); V\src\vs\editor\contrib\codeAction\browser\codeActionMenu.ts (lines 23 to 33); C bundle (module codeActionCommands)

### ED-4.16 Source actions, Organize Imports, Fix All and code actions on save
- Tag: both
- Evidence: source
- Boundary: no
- What: Run file-wide clean-ups such as Organize Imports or Fix All from the Source Action menu, with keys for Organize Imports and Auto Fix, and optionally run chosen ones each time the file is saved.
- Commands: editor.action.sourceAction; editor.action.organizeImports; editor.action.fixAll; editor.action.autoFix
- Menus: EditorContext (1_modification)
- Keys: Shift+Alt+O for Organize Imports (textInputFocus); Shift+Alt+. for Auto Fix (Cmd+Alt+. on Mac)
- Settings: editor.codeActionsOnSave
- Cursor: present (all four ids and editor.codeActionsOnSave found)
- Source: V\src\vs\editor\contrib\codeAction\browser\codeActionCommands.ts (lines 183 to 300); V\src\vs\workbench\contrib\codeActions\browser\codeActionsContribution.ts (lines 65 to 78); V\src\vs\workbench\contrib\codeEditor\browser\saveParticipants.ts (lines 273 to 440); C bundle (modules codeActionCommands, codeActionsContribution with 12 strings)

### ED-4.17 Format document or selection
- Tag: both
- Evidence: source
- Boundary: no
- What: Reformat the whole file or only the selected text with the language's formatter, pick another formatter with the With... variants, and get asked to choose a default when several formatters apply.
- Commands: editor.action.formatDocument; editor.action.formatSelection; editor.action.formatDocument.multiple; editor.action.formatSelection.multiple; editor.action.formatDocument.none; editor.action.format
- Menus: EditorContext (1_modification)
- Keys: Shift+Alt+F (Ctrl+Shift+I on Linux; editorTextFocus); Ctrl+K Ctrl+F for Format Selection
- Settings: editor.defaultFormatter
- Cursor: changed (all five ids found; Format Selection is Ctrl+M Ctrl+F on Windows and Linux and Cmd+R Cmd+F on Mac instead of Ctrl+K Ctrl+F)
- Source: V\src\vs\editor\contrib\format\browser\formatActions.ts (lines 212 to 288); V\src\vs\workbench\contrib\format\browser\formatActionsMultiple.ts (lines 170 to 400); V\src\vs\workbench\contrib\format\browser\formatActionsNone.ts; C bundle (modules formatActions with 2 strings, formatActionsMultiple with 20, formatActionsNone with 4)

### ED-4.18 Format modified lines
- Tag: both
- Evidence: source
- Boundary: no
- What: Format only the lines that differ from the source control version of the file, so the change stays small.
- Commands: editor.action.formatChanges
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (editor.action.formatChanges found)
- Source: V\src\vs\workbench\contrib\format\browser\formatModified.ts; C bundle (module formatModified with 1 string)

### ED-4.19 Format on save, paste and type
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn on automatic formatting when you save (whole file or only modified lines), when you paste, or when you finish typing a line.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.formatOnSave; editor.formatOnSaveMode; editor.formatOnPaste; editor.formatOnType
- Cursor: present (editor.formatOnSave and editor.formatOnSaveMode found; the `formatOnPaste` and `formatOnType` option names and their descriptions found)
- Source: V\src\vs\workbench\contrib\files\browser\files.contribution.ts (lines 391 to 409); V\src\vs\workbench\contrib\codeEditor\browser\saveParticipants.ts (lines 216 to 272); V\src\vs\editor\contrib\format\browser\formatActions.ts (lines 31 to 210); V\src\vs\editor\common\config\editorOptions.ts (lines 6346 to 6351)

### ED-4.20 CodeLens
- Tag: both
- Evidence: source
- Boundary: no
- What: Small gray clickable text above a line, such as a reference count or a Run Test link, supplied by the language, plus a command that lists every lens on the current line in a picker.
- Commands: codelens.showLensesInCurrentLine
- Menus: none found
- Keys: none found
- Settings: editor.codeLens; editor.codeLensFontFamily; editor.codeLensFontSize
- Cursor: present (codelens.showLensesInCurrentLine found; the three setting descriptions found in the Cursor strings)
- Source: V\src\vs\editor\contrib\codelens\browser\codelensController.ts (lines 460 to 520); V\src\vs\editor\contrib\codelens\browser\codelensWidget.ts; V\src\vs\editor\contrib\codelens\browser\codeLensCache.ts; V\src\vs\editor\common\config\editorOptions.ts (lines 6127 to 6140); C bundle (module codelensController with 2 strings)

### ED-4.21 Inlay hints
- Tag: both
- Evidence: source
- Boundary: no
- What: Small gray labels inside the code, such as parameter names and types, that you can hover, Ctrl+click to follow, double-click to insert into the code, or right-click for go-to actions, with a screen reader command that reads a line together with its hints.
- Commands: inlayHints.startReadingLineWithHint; inlayHints.stopReadingLineWithHint
- Menus: EditorContext (its go-to items are reused for the hint right-click menu)
- Keys: Ctrl+Alt held down flips the hints when the setting is onUnlessPressed or offUnlessPressed; Escape (isReadingLineWithInlayHints) stops reading; Ctrl+click (Cmd on Mac) on a hint part follows it
- Settings: editor.inlayHints.enabled; editor.inlayHints.fontSize; editor.inlayHints.fontFamily; editor.inlayHints.padding; editor.inlayHints.maximumLength; editor.inlayHints
- Cursor: present (both reading ids found; all five settings found or described in the Cursor strings)
- Source: V\src\vs\editor\contrib\inlayHints\browser\inlayHintsController.ts (lines 205 to 425); V\src\vs\editor\contrib\inlayHints\browser\inlayHintsHover.ts; V\src\vs\editor\contrib\inlayHints\browser\inlayHintsLocations.ts; V\src\vs\workbench\contrib\inlayHints\browser\inlayHintsAccessibilty.ts; V\src\vs\editor\common\config\editorOptions.ts (lines 3244 to 3275); C bundle (modules inlayHintsHover with 8 strings, inlayHintsAccessibilty with 4)

### ED-4.22 Document highlights
- Tag: both
- Evidence: source
- Boundary: no
- What: Putting the cursor on a symbol shades its other uses in the file (reads and writes in different shades) with marks in the scrollbar, and F7 and Shift+F7 step through them.
- Commands: editor.action.wordHighlight.next; editor.action.wordHighlight.prev; editor.action.wordHighlight.trigger
- Menus: none found
- Keys: F7 and Shift+F7 (editorTextFocus, when highlights exist); Trigger Symbol Highlight has no default key
- Settings: editor.occurrencesHighlight; editor.occurrencesHighlightDelay
- Cursor: present (all three ids found; occurrence highlight setting descriptions found)
- Source: V\src\vs\editor\contrib\wordHighlighter\browser\wordHighlighter.ts (lines 925 to 975); V\src\vs\editor\contrib\wordHighlighter\browser\highlightDecorations.ts; V\src\vs\editor\contrib\wordHighlighter\browser\textualHighlightProvider.ts; V\src\vs\editor\common\config\editorOptions.ts (lines 6463 to 6480); C bundle (modules wordHighlighter with 3 strings, highlightDecorations with 9)

### ED-4.23 Semantic highlighting
- Tag: both
- Evidence: source
- Boundary: no
- What: The language adds colors based on meaning (a parameter versus a local, a type versus a class) on top of the grammar colors, controlled by a setting and the theme, with per-token-type color overrides.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.semanticHighlighting.enabled; editor.semanticTokenColorCustomizations
- Cursor: present (both settings found as quoted strings)
- Source: V\src\vs\editor\contrib\semanticTokens\browser\documentSemanticTokens.ts; V\src\vs\editor\contrib\semanticTokens\browser\viewportSemanticTokens.ts; V\src\vs\editor\contrib\semanticTokens\common\semanticTokensConfig.ts; V\src\vs\editor\common\config\editorConfigurationSchema.ts (lines 79 to 88); V\src\vs\workbench\services\themes\common\themeConfiguration.ts (line 183)

### ED-4.24 Error and warning squiggles
- Tag: both
- Evidence: source
- Boundary: no
- What: Wavy underlines mark code with problems (red errors, yellow warnings, blue infos, faint dots for hints) with matching marks in the scrollbar ruler and the minimap.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (editorError.foreground, editorWarning.foreground, editorInfo.foreground, editorHint.foreground and the `squiggly-error` class found)
- Source: V\src\vs\editor\common\services\markerDecorationsService.ts (lines 190 to 300); V\src\vs\editor\browser\services\markerDecorations.ts; V\src\vs\editor\common\core\editorColorRegistry.ts; C bundle (theme color ids and class names, no module of its own)

### ED-4.25 Faded unused code and struck-through deprecated code
- Tag: both
- Evidence: source
- Boundary: no
- What: Code the language marks as unused is faded out and names it marks as deprecated are drawn with a line through them, each switchable by a setting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.showUnused; editor.showDeprecated
- Cursor: present (editorUnnecessaryCode.opacity found; the two setting descriptions "Controls fading out of unused code" and "Controls strikethrough deprecated variables" found)
- Source: V\src\vs\editor\common\services\markerDecorationsService.ts (lines 194 to 295); V\src\vs\editor\common\config\editorOptions.ts (lines 6646 to 6651); V\src\vs\editor\common\core\editorColorRegistry.ts (lines 65 and 66); V\src\vs\editor\browser\widget\codeEditor\codeEditorWidget.ts

### ED-4.26 Problem hover
- Tag: both
- Evidence: source
- Boundary: no
- What: Hovering a squiggle shows the message, source and code, then a status row with View Problem and Quick Fix... plus "Checking for quick fixes..." or "No quick fixes available" while it looks.
- Commands: editor.action.marker.next; editor.action.quickFix
- Menus: MarkerHoverStatusBar (stock only)
- Keys: Alt+F8 and Ctrl+. are shown beside View Problem and Quick Fix...
- Settings: editor.hover.enabled; editor.hover.delay
- Cursor: changed (View Problem, Quick Fix... and the AI action row are in the bundle; no MarkerHoverStatusBar menu actions and no sparkle icon on the AI action)
- Source: V\src\vs\editor\contrib\hover\browser\markerHoverParticipant.ts (lines 196 to 306); C bundle (module markerHoverParticipant with 5 strings, method _renderMarkerStatusbar)

### ED-4.27 Go to next and previous problem
- Tag: both
- Evidence: source
- Boundary: no
- What: Alt+F8 steps through errors, warnings and infos in the current file and opens a small inline widget under the problem with its message, source, code and "n of m problems" with up and down arrows, while F8 does the same across files.
- Commands: editor.action.marker.next; editor.action.marker.prev; editor.action.marker.nextInFiles; editor.action.marker.prevInFiles; closeMarkersNavigation
- Menus: MenubarGoMenu (6_problem_nav, Next Problem and Previous Problem); gotoErrorTitleMenu (the widget arrows)
- Keys: Alt+F8 and Shift+Alt+F8 (editor focus); F8 and Shift+F8 (editor focus, across files); Escape or Shift+Escape (markersNavigationVisible)
- Settings: none found
- Cursor: present (all five ids found)
- Source: V\src\vs\editor\contrib\gotoError\browser\gotoError.ts (lines 190 to 305); V\src\vs\editor\contrib\gotoError\browser\gotoErrorWidget.ts (lines 207 to 360); V\src\vs\editor\contrib\gotoError\browser\markerNavigationService.ts; C bundle (modules gotoError with 9 strings, gotoErrorWidget with 14)

### ED-4.28 Document links
- Tag: both
- Evidence: source
- Boundary: no
- What: URLs, file paths and command links in the text become links when you hold Ctrl (Cmd on Mac), with a tooltip that says Follow link or Execute command, and Open Link opens the one under the cursor.
- Commands: editor.action.openLink
- Menus: none found
- Keys: none found
- Settings: editor.links
- Cursor: present (editor.action.openLink found; the setting description "Controls whether the editor should detect links and make them clickable" found)
- Source: V\src\vs\editor\contrib\links\browser\links.ts (lines 360 to 410); V\src\vs\editor\contrib\links\browser\getLinks.ts; V\src\vs\editor\common\config\editorOptions.ts (line 6388); C bundle (module links with 10 strings)

### ED-4.29 Inline color swatches and the color picker
- Tag: both
- Evidence: source
- Boundary: no
- What: Colors such as hex, rgb and hsl values show a small swatch in the text, and clicking or hovering it opens a picker with a saturation box, hue strip, opacity strip and a button that cycles the color format.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.colorDecorators; editor.colorDecoratorsActivatedOn; editor.colorDecoratorsLimit; editor.defaultColorDecorators
- Cursor: present (colorDecorators, colorDecoratorsActivatedOn and defaultColorDecorators option names found; the four setting descriptions found; picker modules colorPickerHeader and colorPickerCloseButton present)
- Source: V\src\vs\editor\contrib\colorPicker\browser\colorDetector.ts; V\src\vs\editor\contrib\colorPicker\browser\hoverColorPicker\hoverColorPickerParticipant.ts; V\src\vs\editor\contrib\colorPicker\browser\colorPickerWidget.ts; V\src\vs\editor\contrib\colorPicker\browser\colorPickerParts\colorPickerHeader.ts; V\src\vs\editor\contrib\colorPicker\browser\defaultDocumentColorProvider.ts; V\src\vs\editor\common\config\editorOptions.ts (lines 6142 to 6160, 6840 to 6850); C bundle (modules hoverColorPickerParticipant, colorPickerHeader, colorPickerCloseButton)

### ED-4.30 Standalone color picker
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Show or Focus Standalone Color Picker from the Command Palette to pick a color at the cursor with no existing value in the text, press Enter to insert it as hex, rgb or hsl text, and Escape to hide it.
- Commands: editor.action.showOrFocusStandaloneColorPicker; editor.action.hideColorPicker; editor.action.insertColorWithStandaloneColorPicker
- Menus: CommandPalette
- Keys: Escape (standaloneColorPickerVisible); Enter (standaloneColorPickerFocused)
- Settings: none found
- Cursor: present (all three ids found; module standaloneColorPickerActions has 7 strings)
- Source: V\src\vs\editor\contrib\colorPicker\browser\standaloneColorPicker\standaloneColorPickerActions.ts; V\src\vs\editor\contrib\colorPicker\browser\standaloneColorPicker\standaloneColorPickerController.ts; V\src\vs\editor\contrib\colorPicker\browser\standaloneColorPicker\standaloneColorPickerWidget.ts; C bundle (module standaloneColorPickerActions)

### ED-4.31 Peek Call Hierarchy
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose Peek Call Hierarchy from the Peek submenu of the right-click menu or from the Command Palette to open an inline panel under the line that lists the callers of the symbol under the cursor beside a preview of each call site, switch the list to the calls the symbol makes and back with the Show Outgoing Calls and Show Incoming Calls title buttons, re-root the list on the focused entry with Refocus Call Hierarchy, and open an entry in the editor with Enter or a double-click; the last direction used and the panel layout are remembered.
- Commands: editor.showCallHierarchy; editor.showIncomingCalls; editor.showOutgoingCalls; editor.refocusCallHierarchy; editor.closeCallHierarchy
- Menus: EditorContextPeek (navigation, only when a call hierarchy provider exists); callhierarchy/title (the peek title buttons); CommandPalette (Peek Call Hierarchy)
- Keys: Shift+Alt+H (editorTextFocus) opens it and, while the panel shows, swaps between incoming and outgoing calls (the References view command in SB-7.27 binds the same key); Shift+Enter (callHierarchyVisible) refocuses; Escape (callHierarchyVisible, unless editor.stablePeek) closes
- Settings: editor.stablePeek
- Cursor: present (all five ids found as quoted strings; the labels "Peek Call Hierarchy", "Show Incoming Calls", "Show Outgoing Calls", "Refocus Call Hierarchy" and "Callers of '{0}'" found in the Cursor strings; storage keys callHierarchy/defaultDirection and callHierarchyPeekLayout found)
- Source: V\src\vs\workbench\contrib\callHierarchy\browser\callHierarchy.contribution.ts (lines 30 to 32, 80 to 170, 175 to 300); V\src\vs\workbench\contrib\callHierarchy\browser\callHierarchyPeek.ts (lines 46 to 51, 69, 236 to 290, 340 to 378); V\src\vs\platform\actions\common\actions.ts (EditorContextPeek); C bundle (ids editor.showCallHierarchy, editor.showIncomingCalls, editor.showOutgoingCalls, editor.refocusCallHierarchy, editor.closeCallHierarchy; nls 4957 to 4960)

### ED-4.32 Peek Type Hierarchy
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose Peek Type Hierarchy from the Peek submenu of the right-click menu or from the Command Palette to open the same kind of inline panel with the subtypes of the type under the cursor beside a preview, switch to its supertypes and back with the Show Supertypes and Show Subtypes title buttons, re-root the list on the focused entry with Refocus Type Hierarchy, and open an entry in the editor with Enter or a double-click; the last direction used and the panel layout are remembered.
- Commands: editor.showTypeHierarchy; editor.showSupertypes; editor.showSubtypes; editor.refocusTypeHierarchy; editor.closeTypeHierarchy
- Menus: EditorContextPeek (navigation, only when a type hierarchy provider exists); typehierarchy/title (the peek title buttons); CommandPalette (Peek Type Hierarchy)
- Keys: no key opens it; Shift+Alt+H (while the panel shows) swaps between supertypes and subtypes; Shift+Enter (typeHierarchyVisible) refocuses; Escape (typeHierarchyVisible, unless editor.stablePeek) closes
- Settings: editor.stablePeek
- Cursor: present (all five ids found as quoted strings; the labels "Peek Type Hierarchy" and "Show Supertypes" found in the Cursor strings; storage keys typeHierarchy/defaultDirection and typeHierarchyPeekLayout found)
- Source: V\src\vs\workbench\contrib\typeHierarchy\browser\typeHierarchy.contribution.ts (lines 46, 92, 106, 175 to 292); V\src\vs\workbench\contrib\typeHierarchy\browser\typeHierarchyPeek.ts (lines 47 to 51, 70, 263 to 285, 335 to 372); V\src\vs\platform\actions\common\actions.ts (EditorContextPeek); C bundle (ids editor.showTypeHierarchy, editor.showSupertypes, editor.showSubtypes, editor.refocusTypeHierarchy, editor.closeTypeHierarchy; nls 11460 to 11464)

### ED-4.33 Step through several go-to results with F12
- Tag: both
- Evidence: source
- Boundary: no
- What: When a go-to command finds several places and the matching editor.gotoLocation setting is goto, the editor jumps to the first one and the status bar reads "Symbol 1 of 3, F12 for next", then F12 moves to the next place and wraps around, and Escape or moving the cursor to some other spot ends the stepping.
- Commands: editor.gotoNextSymbolFromResult; editor.gotoNextSymbolFromResult.cancel; editor.action.goToLocations
- Menus: none found
- Keys: F12 (hasSymbols); Escape (hasSymbols)
- Settings: editor.gotoLocation.multiple; editor.gotoLocation.multipleDefinitions; editor.gotoLocation.multipleDeclarations; editor.gotoLocation.multipleTypeDefinitions; editor.gotoLocation.multipleImplementations; editor.gotoLocation.multipleReferences
- Cursor: present (both ids found as quoted strings with F12 as the key; the status strings "Symbol {0} of {1}, {2} for next" and "Symbol {0} of {1}" found at nls 1235 and 1236)
- Source: V\src\vs\editor\contrib\gotoSymbol\browser\symbolNavigation.ts (lines 22 to 180); V\src\vs\editor\contrib\gotoSymbol\browser\goToCommands.ts (lines 177 to 200); C bundle (module vs/editor/contrib/gotoSymbol/browser/symbolNavigation)

### ED-4.34 Message bubble at the cursor
- Tag: both
- Evidence: source
- Boundary: no
- What: When a language action has nothing to show, a small bubble fades in beside the cursor with the reason, such as "No definition found for 'name'", "No implementation found for 'name'", "No related tests found." or the reason a rename or a disabled code action cannot run, and it closes with Escape, when the cursor moves, when the editor loses focus, or when the pointer drifts far from it (the read-only editor warning uses the same bubble).
- Commands: leaveEditorMessage
- Menus: none found
- Keys: Escape (messageVisible)
- Settings: none found
- Cursor: present (leaveEditorMessage with the Escape key found as a quoted string; "No definition found for '{0}'" and the other no-result strings found in the Cursor strings)
- Source: V\src\vs\editor\contrib\message\browser\messageController.ts; V\src\vs\editor\contrib\gotoSymbol\browser\goToCommands.ts (lines 142 to 146, 259 to 263, 386 to 390, 473 to 477, 559 to 563); V\src\vs\editor\contrib\rename\browser\rename.ts (lines 216, 232); V\src\vs\editor\contrib\codeAction\browser\codeActionController.ts (lines 253, 263); V\src\vs\editor\contrib\readOnlyMessage\browser\contribution.ts (lines 27 to 38); C bundle (messageController module with leaveEditorMessage and messageVisible)

### ED-4.35 Go to or peek a related test or related code
- Tag: both
- Evidence: source
- Boundary: no
- What: When a test extension supplies the links, right-click code and choose Go to Related Test or Peek Related Test to reach the tests that cover it, and right-click inside a test and choose Go to Related Code or Peek Related Code to reach the code it exercises, with the go variants jumping straight there for one result and peeking for several, and "No related tests found." or "No related code found." appearing in the message bubble when there is none.
- Commands: testing.goToRelatedTest; testing.peekRelatedTest; testing.goToRelatedCode; testing.peekRelatedCode
- Menus: EditorContext (group testing)
- Keys: none found
- Settings: none found
- Cursor: present (all four ids found as quoted strings; "Related Tests", "Related Code", "No related tests found." and "No related code found." found in the Cursor strings)
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (lines 1039 to 1050, 1849 to 1985); V\src\vs\workbench\contrib\testing\common\testingContextKeys.ts (lines 32, 33); V\src\vs\workbench\contrib\testing\common\constants.ts (lines 91 to 94); C bundle (ids testing.goToRelatedTest, testing.peekRelatedTest, testing.goToRelatedCode, testing.peekRelatedCode; nls 11115 to 11118)

### ED-4.36 Show or hide disabled code actions
- Tag: both
- Evidence: source
- Boundary: no
- What: When some refactorings or source actions cannot run at the cursor, the bar at the bottom of the Quick Fix, Refactor and Source Action menus shows a Show Disabled row that adds those actions in gray with the reason they cannot run and turns into Hide Disabled, and documentation links supplied by a language extension sit in the same bar.
- Commands: showMoreActions; hideMoreActions
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (both ids found as quoted strings; "Show Disabled" and "Hide Disabled" found at nls 1001 and 1000)
- Source: V\src\vs\editor\contrib\codeAction\browser\codeActionController.ts (lines 252 to 268, 325, 422 to 462); V\src\vs\editor\contrib\codeAction\browser\codeActionMenu.ts (lines 47, 78); V\src\vs\editor\contrib\codeAction\browser\codeActionCommands.ts (lines 124, 176, 218, 298); C bundle (module vs/editor/contrib/codeAction/browser/codeActionController, ids showMoreActions and hideMoreActions)

### ED-4.37 Resize the hover card
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag an edge or corner of the hover card to make it taller or wider (the edges that can be dragged depend on whether it opened above or below the text), and later hovers in the same window open at the size you chose until the window reloads.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the widget id `editor.contrib.resizableContentHoverWidget` and its `_lastDimensions` memory found in the bundle)
- Source: V\src\vs\editor\contrib\hover\browser\resizableContentWidget.ts (lines 21 to 108); V\src\vs\editor\contrib\hover\browser\contentHoverWidget.ts (lines 24 to 71, 169 to 176, 308 to 309, 388 to 403); C bundle (module contentHoverWidget with `editor.contrib.resizableContentHoverWidget`)

### ED-4.38 Resize the peek window and keep its size
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag the bottom edge of the peek window to change how many lines tall it is and drag the divider between the preview and the results list to change how the width is shared (the preview gets 70 percent and the window is 18 lines tall at first), and the next peek opens with the layout you left.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the storage key peekViewLayout and the peek frame options with isResizeable set found in the bundle)
- Source: V\src\vs\editor\contrib\gotoSymbol\browser\peek\referencesWidget.ts (lines 166 to 183, 268 to 313, 406 to 430, 465 to 467); V\src\vs\editor\contrib\gotoSymbol\browser\peek\referencesController.ts (lines 99 to 108); V\src\vs\editor\contrib\zoneWidget\browser\zoneWidget.ts (line 183); C bundle (storage key peekViewLayout; peek widget options with isResizeable)

### ED-4.39 Read a hover in the accessible view
- Tag: both
- Evidence: source
- Boundary: no
- What: With a screen reader, focus the hover card (press the Show or Focus Hover key twice) and press the accessible view key to read its content as plain text in a separate view that has plus and minus actions for more or less detail, or press the accessibility help key while a hover shows to hear which hover actions exist, and a setting adds a spoken hint with the key when the hover takes focus.
- Commands: editor.action.accessibleView; editor.action.accessibilityHelp; editor.action.increaseHoverVerbosityLevelFromAccessibleView; editor.action.decreaseHoverVerbosityLevelFromAccessibleView
- Menus: CommandPalette (Open Accessible View and Open Accessibility Help, shared with every accessible view)
- Keys: Alt+F2 (accessible view, when the hover has focus); Alt+F1 (accessibility help, when a hover shows); on Linux the primary keys are Alt+Shift+F2 and Alt+Shift+F1 with the Alt-only keys as second choices
- Settings: accessibility.verbosity.hover
- Cursor: present (the hover accessible view class with id hover and setting key accessibility.verbosity.hover, both FromAccessibleView ids and the nls modules hoverAccessibleViews with 2 strings and contentHoverRendered with 3 hover status bar strings found in the bundle)
- Source: V\src\vs\editor\contrib\hover\browser\hoverAccessibleViews.ts; V\src\vs\editor\contrib\hover\browser\hoverContribution.ts (lines 52 to 54); V\src\vs\editor\contrib\hover\browser\hoverActionIds.ts (lines 18 to 22); V\src\vs\editor\contrib\hover\browser\contentHoverWidget.ts (line 354); V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 164 to 200); V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (lines 61, 168 to 171); C bundle (modules hoverAccessibleViews, contentHoverRendered)

### ED-4.40 Go to Problem at Cursor
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Press Ctrl+Alt+F8 (Cmd+Alt+F8 on Mac) with the cursor on a squiggled problem to open the inline problem widget right there, showing its message, source and code, with the view scrolled to center it if it was off screen, instead of stepping to the next problem.
- Commands: editor.action.marker.this
- Menus: CommandPalette (Go to Problem at Cursor (Error, Warning, Info))
- Keys: Ctrl+Alt+F8 (Cmd+Alt+F8 on Mac) when the editor has focus
- Settings: none found
- Cursor: cursor only (id `editor.action.marker.this`, label at nls 1154 and the method showMarkerAtCursorIfExists found in the Cursor bundle; searched V and N for the id and found none, so it is tagged cursor even though the surrounding go to problem code is stock)
- Source: C bundle (module vs/editor/contrib/gotoError/browser/gotoError, class with alias "Go to Problem at Cursor (Error, Warning, Info)", marker controller method showMarkerAtCursorIfExists, constants module holding `editor.action.marker.this`, nls 1154)

### ED-4.41 Undo prompts after multi-file refactors
- Tag: both
- Evidence: source
- Boundary: no
- What: After a rename or refactor that changed several files, pressing Undo asks "Would you like to undo 'name' across all files?" with Undo in N Files, Undo this File and Cancel buttons, a warning lists the files that changed since and nothing is undone, Redo gives matching "Could not redo" warnings, and an edit that came from a different source first asks "Would you like to undo 'name'?" with Yes or No.
- Commands: undo; redo
- Menus: MenubarEditMenu (Undo and Redo, group 1_do); CommandPalette
- Keys: Ctrl+Z (Undo); Ctrl+Y or Ctrl+Shift+Z (Redo); Cmd+Shift+Z on Mac
- Settings: none found
- Cursor: present (the prompt text "Would you like to undo '{0}' across all files?", "&&Undo in {0} Files", "Undo this &&File" and "Would you like to undo '{0}'?" found in the Cursor strings, and the method _confirmAndExecuteWorkspaceUndo found in the bundle)
- Source: V\src\vs\platform\undoRedo\common\undoRedoService.ts (lines 817 to 900, 940 to 990, 1100 to 1130, 1173 to 1255); V\src\vs\editor\browser\editorExtensions.ts (lines 646 to 696); C bundle (module vs/platform/undoRedo/common/undoRedoService)

### ED-4.42 Choose the order Next Problem follows
- Tag: both
- Evidence: source
- Boundary: no
- What: A setting decides whether Go to Next Problem and Go to Previous Problem walk each file's problems by severity (errors, then warnings, then infos, the default) or in the order they appear in the text; the same setting also sorts the Problems list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: problems.sortOrder
- Cursor: present (the setting registration with the key `problems.sortOrder` and the marker sort code that reads it found in the bundle)
- Source: V\src\vs\editor\contrib\gotoError\browser\markerNavigationService.ts (lines 51 to 62); V\src\vs\workbench\contrib\markers\browser\markers.contribution.ts (lines 116 to 124); C bundle (setting problems.sortOrder)

## Handed off
- Sticky scroll (`editor.action.toggleStickyScroll`, `editor.stickyScroll.*`): ED-5. It reads symbols or folding ranges, but everything the person sees is an orientation aid.
- Breadcrumbs symbol path and its pickers: ED-5.
- Inspect Editor Tokens and Scopes (shows semantic and grammar tokens): ED-5.
- Go to Symbol in Editor (`workbench.action.gotoSymbol`, Ctrl+Shift+O, the `@:` category) and Go to Line: QA-3. The Go menu entries for them are in TB-1.
- Go to Symbol in Workspace (`workbench.action.showAllSymbols`): QA-2.
- Outline view with Follow Cursor, sorting and filtering: SB-7.
- Find All References view, Find All Implementations, Show Call Hierarchy and Show Type Hierarchy (`references-view` extension, `editor.showCallHierarchy`, `editor.showTypeHierarchy`): SB-7.
- Linked editing (`editor.action.linkedEditing`, `editor.linkedEditing`, and the deprecated alias `editor.renameOnType`): ED-2.
- Selection highlight of other copies of the selected text (`editor.selectionHighlight`, in the multicursor code): ED-2.
- Gutter icon hover (glyph margin hover, `glyphHoverController.ts`): ED-11. In V the code that sets glyph hover messages is the breakpoint and test decorations, the diff editor's hidden-regions widget, the settings editor widgets and a chat plan-review contribution.
- Limit indicator status items that appear when color decorators (or folding ranges) hit their limit (`V\src\vs\workbench\contrib\limitIndicator\browser\limitIndicator.contribution.ts`, status name "Color Decorator Status"): SS-3.
- Debug hover, inline values and breakpoint widgets: ED-11.
- Refactor Preview panel opened by a rename preview or a previewed code action: PN-7.
- Problems panel list and its Quick Fix: PN-4. Problem counts in the status bar: SS-5.
- Trusted domain prompt shown when a document link opens an outside site (`trustedDomainsValidator.ts`): DL-5.
- Merge conflict CodeLens from the merge-conflict extension: ED-6.
- Parameter hints and signature help: AC-3 (already catalogued, not repeated here).
- The editor right-click menu as a whole (`MenuId.EditorContext`, the Show Context Menu command `editor.action.showContextMenu` on Shift+F10 in `V\src\vs\editor\contrib\contextmenu\browser\contextmenu.ts`, the `editor.contextmenu` option, the Cut, Copy, Paste, Share and Command Palette rows, and the line-number gutter menu `EditorLineNumberContext`): no group owns it, so the parent should place it. ED-2 is the closest fit because the menu hosts the clipboard and multi-cursor items (confidence medium - several groups put rows in it and none owns the frame). ED-4 only owns its own rows, listed in ED-4.3 and ED-4.5 to ED-4.9, ED-4.12, ED-4.15 to ED-4.17, ED-4.31, ED-4.32 and ED-4.35.
- Overflow check note: SB-7 handed the editor peek versions of Show Call Hierarchy and Show Type Hierarchy back to ED-4, so they are now ED-4.31 and ED-4.32. Only the References view versions (`references-view.showCallHierarchy` and `references-view.showTypeHierarchy`, SB-7.27 and SB-7.28) stay with SB-7. The earlier line above that names `editor.showCallHierarchy` and `editor.showTypeHierarchy` under SB-7 is out of date.

## Unclear
- No Error Lens style display exists. I searched V, N and the Cursor bundle for `errorLens` and `inlineDiagnostic` and found nothing, so the catalog's "error lens style hover" is the problem hover in ED-4.26 and there is no inline end-of-line message feature to list.
- Cursor's marker hover, Quick Fix action, code action widget and hover card all differ from V 1.128.0 in small ways (see the `changed` entries). The base of those parts looks older than 1.128. These differences are missing ids and strings, not observed behavior.
- `editor.action.refactor.preview` is declared in `codeAction.ts` line 36 but nothing in V registers or uses it, and the Cursor bundle has no match. It is not listed as a feature.
- `editor.occurrencesHighlightFromAllProviders` exists only in N (newer-than-1.128). It is not listed under ED-4.22 Settings because Cursor lacks it and V does not have it.
- Keys are V defaults for Windows, with Linux or Mac noted where they differ. In the Cursor bundle I read the keys of Show or Focus Hover, Go to Definition, Go to Implementation, Open Definition to the Side, Toggle Peek Focus and Format Selection. The single keys (F12, Ctrl+F12) match V. Every chord that starts with Ctrl+K in V starts with a constant `zd` (2091) in Cursor, with `Oh` (2096) for Mac. These are defined next to the KeyMod enum in Cursor's key code module and are used by 96 keybinding registrations. I decoded them against the enums in the same bundle as CtrlCmd+KeyM (Ctrl+M) and CtrlCmd+KeyR (Cmd+R). This changes four ED-4 features (ED-4.2, ED-4.3, ED-4.10, ED-4.17) and every other group that has a Ctrl+K chord. I did not see how the Keyboard Shortcuts editor displays them. The other keys in this file were not read in the Cursor bundle.
- Rename suggestions only appear if a provider supplies names. I did not check whether any built-in extension in V or Cursor supplies them.
- The Install Formatter button in the no-formatter prompt (ED-4.17) leads to the extension marketplace, which is out of scope. I did not follow it.
- No group owns the editor right-click menu (`EditorContext`) as a whole. ED-4 items sit in it next to cut, copy, paste and Command Palette entries.
- The accessible view for hover content (`HoverAccessibleView` and `HoverAccessibilityHelp` in `V\src\vs\editor\contrib\hover\browser\hoverAccessibleViews.ts`) has no owner. It is cross-cutting and was not listed or checked in the Cursor bundle.
- Cursor chat entry points that sit near these features (`composer.fixerrormessage`, `editor.action.addSymbolToChat`, the "Add to Chat" and "Fix with Agent" rows in the Problems panel) are chat surfaces and were left out. The one exception is ED-4.11, which is a button inside a peek window.
- The overflow check listed the hover accessible view as ED-4.39 and checked it in the Cursor bundle, so the earlier note above that it has no owner and was not checked no longer applies. The accessible view for other surfaces and the editor accessibility help page (`editorAccessibilityHelp.ts`) still have no owner.
- `editor.cancelOperation` (Escape, context key cancellableOperation) cancels a slow lookup such as Peek References. It has no label and no menu entry, so it was not listed as a feature.
- `problems.sortOrder` is now ED-4.42. PN-4 handed it to ED-4 because Next Problem reads it, and no existing feature named it.
- Cursor draws its own small toolbar on selected code (Add to Chat and Quick Edit buttons, class `cursorHoverWidget` in its hover widget module, labeled "Toolbar on Selection" in Cursor Settings). It is a chat entry point and was left out like the other chat entries above.
- Searched N (1.144.0) for ED-4 command ids missing from V and found none, so nothing here is tagged `newer-than-1.128`.
- Overflow check on 2026-10-25 added 12 features (ED-4.31 to ED-4.42).
