Section: Autocomplete (AC). Date: 2026-10-25. Pass: 1.

# 06 Autocomplete - VS Code 1.128.0 findings (pass 1, minor groups)

## How to read this file

- Scope of this file: stock VS Code 1.128.0 only (V). The Cursor side is in the sibling file under `cursor\06-autocomplete.md`.
- Path legend: `V` = `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0`. `N` = `C:\Users\Vinnie\src\vscode` (1.144.0, only used to spot newer things). Paths below are relative to V unless they start with `N\`.
- Hard rule honored: only the widgets and the interaction are cataloged. Nothing here describes how suggestions are produced.
- Evidence level on every item below is `source` (read in V) unless said otherwise.
- Tag meaning: `vscode` = in stock only, `both` = also found by searching the Cursor bundle, `cursor` = Cursor only. Tags in this file come from the bundle checks recorded in the Cursor file.

## Proposed minor groups that exist in stock VS Code

Five of the eight proposed groups (AC-1 to AC-5) have a stock counterpart. AC-6, AC-7 and AC-8 are Cursor Tab groups and have no stock counterpart (see the Cursor file). Group ids are stable.

### AC-1 Suggest list (IntelliSense popup)

- Plain name: Suggest list
- One sentence: The dropdown that appears next to the cursor while typing, with icons, labels, a details pane, a small status bar, and keys to pick an item.
- Tag: both
- Evidence: source
- Boundary: no
- Example features: list rows with kind icon, label, detail and description; details side pane with Read More; suggest status bar (Insert or Replace, Show More or Show Less); accept and navigate keys.
- Proof paths:
  - `src\vs\editor\contrib\suggest\browser\suggestWidget.ts` (widget id `editor.widget.suggestWidget`, classes `editor-widget suggest-widget`, `no-icons`, `with-status-bar`, messages "Loading..." and "No suggestions.")
  - `src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts` (row anatomy)
  - `src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts` (details pane)
  - `src\vs\editor\contrib\suggest\browser\suggestWidgetStatus.ts` and `src\vs\editor\contrib\suggest\browser\suggest.ts` (status bar menu id `suggestWidgetStatusBar`, context keys)
  - `src\vs\editor\contrib\suggest\browser\suggestController.ts` (commands and keys)
  - `src\vs\editor\contrib\suggest\browser\media\suggest.css`

Facts seen (for the feature pass):

- Row anatomy: `.icon` (with `span.colorspan` for color swatches), `.contents > .main` with `.icon-label.codicon` (kind icon, class `suggest-icon`), `span.left` holding the label, `span.signature-label` (detail) and `span.qualifier-label` (description), and `span.right` holding `span.details-label` and `span.readMore` (icon id `suggest-more-info`, default chevron-right, title "Read More"). Row classes: `deprecated`, `show-file-icons`, `string-label`, `customcolor`, `can-expand-details`.
- Details pane: container `suggest-details-container`, resizable, close button titled "Close", text "Loading...", modes `no-docs`, `no-type`, `detail-and-doc`, markdown docs. Persisted widget size, reset by `editor.action.resetSuggestSize`.
- Status bar: left actions and right actions bars. Left: "Select" (no focused item), "Insert" or "Replace" (accept), and the opposite alternative. Right: "Show More" or "Show Less" (details toggle). Shown when `editor.suggest.showStatusBar` is on (default false).
- Commands (id, title if any, default keys):
  - `editor.action.triggerSuggest`, "Trigger Suggest", Ctrl+Space or Ctrl+I (mac: Ctrl+Space, Alt+Esc, Cmd+I)
  - `acceptSelectedSuggestion`, Tab, or Enter when acceptSuggestionOnEnter is on and the item makes a text edit (old alias `acceptSelectedSuggestionOnEnter`)
  - `acceptAlternativeSelectedSuggestion`, Shift+Enter or Shift+Tab (insert vs replace flip)
  - `hideSuggestWidget`, Esc or Shift+Esc
  - `selectNextSuggestion`, Down or Ctrl+Down (mac adds Ctrl+N); `selectPrevSuggestion`, Up or Ctrl+Up (mac adds Ctrl+P)
  - `selectNextPageSuggestion` PageDown; `selectPrevPageSuggestion` PageUp; `selectFirstSuggestion` and `selectLastSuggestion` (no keys)
  - `focusSuggestion` (Ctrl+Space or Ctrl+I while the list is open and no item is focused), `focusAndAcceptSuggestion` (no key), `toggleSuggestionDetails` (same keys once an item is focused), `toggleExplainMode` (Ctrl+/), `toggleSuggestionFocus` (Ctrl+Alt+Space)
  - `insertBestCompletion`, `insertNextSuggestion`, `insertPrevSuggestion` (Tab and Shift+Tab, tab completion mode)
  - `suggestWidgetCopy` (Ctrl+C when the details pane has focus)
  - `editor.action.resetSuggestSize`, "Reset Suggest Widget Size"
- Context keys: `suggestWidgetVisible` (declared as `historyNavigationVisible` in `src\vs\platform\history\browser\contextScopedHistoryWidget.ts`), `suggestWidgetHasFocusedSuggestion`, `suggestWidgetDetailsVisible`, `suggestWidgetDetailsFocused`, `suggestWidgetMultipleSuggestions`, `suggestionMakesTextEdit`, `acceptSuggestionOnEnter`, `suggestionHasInsertAndReplaceRange`, `suggestionInsertMode`, `suggestionCanResolve`, `atEndOfWord`.
- Commit characters: `src\vs\editor\contrib\suggest\browser\suggestCommitCharacters.ts`, controlled by `editor.acceptSuggestionOnCommitCharacter`.
- Sorting display and memory: `suggestMemory.ts` (`editor.suggestSelection`, `editor.suggest.shareSuggestSelections`), `completionModel.ts`, `wordDistance.ts` (`editor.suggest.localityBonus`).
- Alternatives (cycling other suggestions after accept): `suggestAlternatives.ts`, context key `hasOtherSuggestions`.
- Theme colors: `editorSuggestWidget.background`, `.border`, `.foreground`, `.selectedForeground`, `.selectedIconForeground`, `.selectedBackground`, `.highlightForeground`, `.focusHighlightForeground`, `editorSuggestWidgetStatus.foreground`.
- Emmet through the suggest list (only because it surfaces here): `V\extensions\emmet\package.json` settings `emmet.showExpandedAbbreviation` (never, always, inMarkupAndStylesheetFilesOnly), `emmet.showAbbreviationSuggestions`, `emmet.triggerExpansionOnTab`, `emmet.showSuggestionsAsSnippets`, `emmet.useInlineCompletions` (ghost text). Emmet's other commands (wrap, balance, tag edits) are not autocomplete UI and are left out.
- Settings to attach later (all `editor.` keys):
  - `quickSuggestions` (object with other, comments, strings; defaults on, off, off), `quickSuggestionsDelay`, `suggestOnTriggerCharacters`, `acceptSuggestionOnEnter` (on, smart, off), `acceptSuggestionOnCommitCharacter`, `tabCompletion` (on, off, onlySnippets), `snippetSuggestions` (top, bottom, inline, none; default inline), `suggestSelection` (first, recentlyUsed, recentlyUsedByPrefix), `suggestFontSize`, `suggestLineHeight`, `wordBasedSuggestions` (off, offWithInlineSuggestions, currentDocument, matchingDocuments, allDocuments; default offWithInlineSuggestions; in `src\vs\editor\common\config\editorConfigurationSchema.ts`)
  - `suggest.insertMode`, `suggest.filterGraceful`, `suggest.localityBonus`, `suggest.matchOnWordStartOnly`, `suggest.shareSuggestSelections`, `suggest.selectionMode`, `suggest.showIcons`, `suggest.showStatusBar`, `suggest.showInlineDetails`, `suggest.showDeprecated`, `suggest.preview`, `suggest.snippetsPreventQuickSuggestions`, `suggest.filteredTypes`, `suggest.maxVisibleSuggestions` (marked deprecated in V, widget is resizable), and one `suggest.show<Kind>` toggle per item kind (Methods, Functions, Constructors, Fields, Variables, Classes, Structs, Interfaces, Modules, Properties, Events, Operators, Units, Values, Constants, Enums, EnumMembers, Keywords, Words, Colors, Files, References, Customcolors, Folders, TypeParameters, Issues, Users, Snippets)
  - Source for all of the above: `src\vs\editor\common\config\editorOptions.ts`

### AC-2 Ghost text and inline suggestions

- Plain name: Ghost text and inline suggestions
- One sentence: Gray text shown in the editor at the cursor, with a small hover toolbar, keys to accept all, a word, or a line, and keys to cycle; also holds the stock inline edit UI (gutter pill, jump, side by side previews).
- Tag: both
- Evidence: source
- Boundary: no (widgets only; the provider behind it is out of scope)
- Example features: ghost text in the editor; inline suggestion toolbar (Accept, Accept Word, Accept Line, previous and next with n/m counter); accept word or line and next or previous keys; inline edit gutter indicator and its menu (Go To / Accept, Reject, Show Collapsed, Snooze, Settings).
- Proof paths:
  - `src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts` (commands, keys, toolbar menu items)
  - `src\vs\editor\contrib\inlineCompletions\browser\controller\commandIds.ts`
  - `src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionContextKeys.ts`
  - `src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts` and `ghostTextView.css`
  - `src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\inlineCompletionsHintsWidget.ts` and `hoverParticipant.ts`
  - `src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\` (views, gutter indicator, theme)
  - `src\vs\editor\browser\services\inlineCompletionsService.ts` (snooze)
  - `src\vs\workbench\contrib\inlineCompletions\browser\inlineCompletionLanguageStatusBarContribution.ts`

Facts seen:

- Ghost text view CSS classes: `ghost-text`, `ghost-text-decoration`, `ghost-text-decoration-preview`, `ghost-text-hidden`, `ghost-text-view-warning-widget` (and `-icon`), `inline-completion-text-to-replace`. Colors: `editorGhostText.foreground`, `.background`, `.border` (in `src\vs\editor\common\core\editorColorRegistry.ts`).
- Toolbar (hints widget): root class `inlineSuggestionsHints`, previous and next chevrons (icon ids `inline-suggestion-hints-previous` and `-next`), an `availableSuggestionCount` label formatted `n/m`, and a toolbar fed by menu `MenuId.InlineSuggestionToolbar`. Also menu `MenuId.InlineCompletionsActions` (marked for deprecation in code). Shown per `editor.inlineSuggest.showToolbar`.
- Toolbar menu items (title, group, order): Accept (primary, 2), Accept Word (primary, 2), Accept Line (secondary, 2), Always Show Toolbar (toggle, secondary, 10, id `editor.action.inlineSuggest.toggleAlwaysShowToolbar`).
- Hover: `hoverParticipant.ts` shows "Suggestion:" with the same actions; accessibility text "There are inline completions here".
- Commands (id, title, default keys):
  - `editor.action.inlineSuggest.trigger`, "Trigger Inline Suggestion" (args: showNoResultNotification, providerId, explicit; message "No inline suggestion is available.")
  - `editor.action.inlineSuggest.showNext`, "Show Next Inline Suggestion", Alt+]
  - `editor.action.inlineSuggest.showPrevious`, "Show Previous Inline Suggestion", Alt+[
  - `editor.action.inlineSuggest.acceptNextWord`, "Accept Next Word Of Inline Suggestion", Ctrl+Right when the cursor is before ghost text and not in accessibility mode
  - `editor.action.inlineSuggest.acceptNextLine`, "Accept Next Line Of Inline Suggestion" (no key)
  - `editor.action.inlineSuggest.commit`, "Accept Inline Suggestion", Tab (several guards: suggest widget not visible, tab does not move focus, hover not focused, no selection, indentation smaller than tab size)
  - `editor.action.inlineSuggest.commitAlternativeAction`, "Accept Inline Suggestion Alternative Action", Shift+Tab (inline edits only)
  - `editor.action.inlineSuggest.hide`, "Hide Inline Suggestion", Esc (menu title "Reject" in the inline edits actions menu)
  - `editor.action.inlineSuggest.jump`, "Jump to next inline edit", Tab when `tabShouldJumpToInlineEdit` (menu title "Jump")
  - `editor.action.inlineSuggest.toggleShowCollapsed`, "Toggle Inline Suggestions Show Collapsed"
  - `editor.action.inlineSuggest.snooze`, "Snooze Inline Suggestions" (quick pick placeholder "Select snooze duration for Inline Suggestions", items 1, 5, 10, 15, 30 and 60 minutes, "Custom...", plus a last used entry; custom input placeholder "Duration in minutes (e.g. 90)"); `editor.action.inlineSuggest.cancelSnooze`, "Cancel Snooze Inline Suggestions"
  - `editor.action.inlineSuggest.dev.extractRepro`, "Developer: Extract Inline Suggest State"
  - `editor.action.inlineSuggest.renameSymbol` id exists in `commandIds.ts`; its UI was not read
- Context keys: `inlineSuggestionVisible`, `inlineSuggestionAlternativeActionVisible`, `inlineSuggestionHasIndentation`, `inlineSuggestionHasIndentationLessThanTabSize`, `inlineSuggestionSuppressSuggestions`, `cursorBeforeGhostText`, `cursorInIndentation`, `editor.hasSelection`, `cursorAtInlineEdit`, `inlineEditIsVisible`, `tabShouldJumpToInlineEdit`, `tabShouldAcceptInlineEdit`, `inInlineEditsPreviewEditor`, `inlineCompletions.snoozed`.
- Language status entry (not the main status bar, it lives in the language status popup): id `inlineSuggestions`, name "Inline Suggestions", detail "Inline suggestions", labels "Loading...", "Inline completion available", "Inline edit available", "No inline suggestion available"; hidden when AI features are disabled.
- Inline edit view kinds (enum `InlineCompletionViewKind`): GhostText, Custom, SideBySide, Deletion, InsertionInline, InsertionMultiLine, WordReplacements, LineReplacement, Collapsed, JumpTo. Tab action states: jump, accept, inactive.
- Gutter indicator: class `inline-edits-view-gutter-indicator`; icons keyboard-tab, arrow-right, check (on hover), keyboard-tab-above, keyboard-tab-below. Its hover menu items: "Go To / Accept" (check), "Reject" (close), optional alternative action, "Show Expanded" or "Show Collapsed", "Snooze" (bell-slash), "Settings" (gear, opens `workbench.action.openSettings` with `@tag:nextEditSuggestions`).
- Word replacement hover text: "{0} 1 occurrence", "{0} {1} occurrences", "[shift] show occurrences".
- Other CSS classes: `inline-edits-collapsed-view`, `inline-edits-custom-view`, `inline-edit-jump-to-widget`, `inline-edit-jump-to-decoration`, `inline-edits-long-distance-hint-widget`, `inlineCompletions-char-insert`, `inlineCompletions-char-delete`, `inlineCompletions-line-insert`, `inlineCompletions-line-delete`, `inline-edit-alternative-action-label`.
- Theme colors: `inlineEdit.originalBackground`, `.modifiedBackground`, `.originalChangedLineBackground`, `.originalChangedTextBackground`, `.modifiedChangedLineBackground`, `.modifiedChangedTextBackground`, `.originalBorder`, `.modifiedBorder`, `.tabWillAcceptModifiedBorder`, `.tabWillAcceptOriginalBorder`, and `inlineEdit.gutterIndicator.` with `background`, `primary`, `secondary`, `successful` each in Foreground, Border and Background variants (`src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\theme.ts`).
- Accessible view: `inlineCompletionsAccessibleView.ts` (hint "Inspect this in the accessible view ({0})").
- Settings to attach later (`editor.inlineSuggest.` prefix, defaults from `editorOptions.ts`): `enabled` (true), `showToolbar` (always, onHover, never; default onHover), `syntaxHighlightingEnabled` (true), `suppressSuggestions` (false), `suppressInSnippetMode` (true), `minShowDelay` (0), `fontFamily` ('default'), `edits.allowCodeShifting` (always, horizontal, never), `edits.renderSideBySide` (auto, never), `edits.showCollapsed` (false), `edits.showLongDistanceHint` (true, experimental); editor options `screenReaderAnnounceInlineSuggestion` and `inlineCompletionsAccessibilityVerbose`. Provider-facing experimental keys exist (`experimental.suppressInlineSuggestions`, `experimental.emptyResponseInformation`, `experimental.showOnSuggestConflict`, `triggerCommandOnProviderChange`) and are left out of the UI catalog.

### AC-3 Parameter hints and signature help

- Plain name: Parameter hints
- One sentence: A small popup above or below the cursor that shows the signature of the call being typed, highlights the current parameter, and lets you flip through overloads.
- Tag: both
- Evidence: source
- Boundary: no
- Example features: signature popup with the active parameter highlighted; overload switcher (previous, "n/m", next); trigger and close keys; docs text under the signature.
- Proof paths:
  - `src\vs\editor\contrib\parameterHints\browser\parameterHintsWidget.ts`
  - `src\vs\editor\contrib\parameterHints\browser\parameterHints.ts`
  - `src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts`
  - `src\vs\editor\contrib\parameterHints\browser\provideSignatureHelp.ts` (context keys)
  - `src\vs\editor\contrib\parameterHints\browser\parameterHints.css`

Facts seen:

- Widget DOM: `.parameter-hints-widget` containing `.phwrapper` with `.controls` (`.button` previous using icon id `parameter-hints-previous`, default chevron-up; `.overloads` text; `.button` next using `parameter-hints-next`, default chevron-down), and a body with `.signature` (`.code` with `.parameter.active` spans) and `.docs`. Modifier class `multiple` when several signatures exist.
- Commands (id, title, keys): `editor.action.triggerParameterHints`, "Trigger Parameter Hints", Ctrl+Shift+Space; `closeParameterHints`, Esc or Shift+Esc; `showPrevParameterHint`, Up or Alt+Up (mac adds Ctrl+P); `showNextParameterHint`, Down or Alt+Down (mac adds Ctrl+N). Up and Down only act while the widget is visible and has more than one signature.
- Context keys: `parameterHintsVisible`, `parameterHintsMultipleSignatures`.
- Screen reader alert "{0}, hint". Theme color `editorHoverWidget.highlightForeground` colors the active parameter.
- Settings to attach later: `editor.parameterHints.enabled` (default true, also controls trigger while typing), `editor.parameterHints.cycle` (default true, wrap around overloads). Source: `src\vs\editor\common\config\editorOptions.ts`.

### AC-4 Snippets

- Plain name: Snippets
- One sentence: Reusable code templates that insert text with tab stops, placeholders and choices, plus the pickers and files used to insert and manage them.
- Tag: both
- Evidence: source
- Boundary: no
- Example features: tab stops and placeholder highlighting while a snippet is active; choice placeholder dropdown; Insert Snippet picker; Configure Snippets (snippet files).
- Proof paths:
  - `src\vs\editor\contrib\snippet\browser\snippetController2.ts` and `snippetSession.ts` and `snippetSession.css`
  - `src\vs\workbench\contrib\snippets\browser\snippets.contribution.ts`
  - `src\vs\workbench\contrib\snippets\browser\commands\insertSnippet.ts`, `configureSnippets.ts`, `surroundWithSnippet.ts`, `fileTemplateSnippets.ts`
  - `src\vs\workbench\contrib\snippets\browser\snippetPicker.ts`, `snippetCompletionProvider.ts`, `snippetCodeActionProvider.ts`, `tabCompletion.ts`

Facts seen:

- Active snippet: context keys `inSnippetMode`, `hasNextTabstop`, `hasPrevTabstop`, `inSnippetChoice`. Keys: `jumpToNextSnippetPlaceholder` Tab, `jumpToPrevSnippetPlaceholder` Shift+Tab, `leaveSnippet` Esc or Shift+Esc. Decoration classes `snippet-placeholder` and `finish-snippet-placeholder`. Theme colors `editor.snippetTabstopHighlightBackground`, `editor.snippetTabstopHighlightBorder`, `editor.snippetFinalTabstopHighlightBackground`, `editor.snippetFinalTabstopHighlightBorder` (in `src\vs\platform\theme\common\colors\editorColors.ts`).
- Choice placeholders open a suggest-style dropdown of the options; the item command title is "Go to next placeholder...".
- Tab completion of a snippet prefix: command `insertSnippet`, Tab, when `hasSnippetCompletions` and `atEndOfWord` (setting `editor.tabCompletion`).
- Snippets in the suggest list: item kind Snippet, detail "{description or name} ({source})", disambiguation label "{label}, {name}".
- Commands: `editor.action.insertSnippet`, "Insert Snippet" (alias `editor.action.showSnippets`); `editor.action.surroundWithSnippet`, "Surround with Snippet..."; `workbench.action.populateFileFromSnippet`, "Fill File with Snippet"; `workbench.action.openSnippets`, "Configure Snippets" (menu entries: Preferences menu "Snippets" group 2_configuration order 5, and the Manage gear menu group 2_configuration order 5).
- Insert Snippet picker: placeholder "Select a snippet", separators "User Snippets" and "Workspace Snippets", row buttons "Hide from IntelliSense" and "Show in IntelliSense", row description "(hidden from IntelliSense)", empty message "No snippet available".
- Configure Snippets picker: placeholder "Select Snippets File or Create Snippets", separators "Existing Snippets" and "New Snippets", entries "New Global Snippets file..." and "New Snippets file for '{0}'...", file name prompt "Type snippet file name" with errors "Invalid file name" and "'{0}' already exists".
- Code actions from snippets (light bulb): entries "{snippet name}", "More...", "Start with Snippet", "Start with: {0}". Controlled by `editor.snippets.codeActions.enabled` (default true).
- Snippet file format (JSON schema, for the settings and files pass): keys prefix, body, description, scope, isFileTemplate, include, exclude.
- Settings to attach later: `editor.snippetSuggestions` (top, bottom, inline, none; default inline), `editor.suggest.showSnippets`, `editor.suggest.snippetsPreventQuickSuggestions`, `editor.tabCompletion`, `editor.snippets.codeActions.enabled`, `editor.inlineSuggest.suppressInSnippetMode`.

### AC-5 Terminal suggestions

- Plain name: Terminal suggestions
- One sentence: The suggest popup inside the integrated terminal for commands, arguments, flags, files and folders, with its own status bar menu and keys.
- Tag: both
- Evidence: source
- Boundary: no
- Example features: terminal suggest popup with symbol icons per kind (file, folder, alias, flag, branch, commit); popup status bar menu (selection mode, show on type, Learn More, Configure); details pane toggle; accept with Tab or Enter and run on Enter.
- Proof paths:
  - `src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts` (commands, keys, menu)
  - `src\vs\workbench\contrib\terminalContrib\suggest\common\terminal.suggest.ts` (command ids)
  - `src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts` (settings)
  - `src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts` (widget host, kind labels, font info)
  - `src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSymbolIcons.ts` and `media\terminalSymbolIcons.css`
  - `src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts` (shared widget, same layout as the editor suggest list) with `simpleSuggestWidgetDetails.ts`, `simpleSuggestWidgetRenderer.ts`, `media\suggest.css`
  - `src\vs\platform\actions\common\actions.ts` (menu id `MenubarTerminalSuggestStatusMenu`)
  - `extensions\terminal-suggest\package.json` and `package.nls.json` (extension UI is one command only)

Facts seen:

- Commands (id, title): `workbench.action.terminal.triggerSuggest` "Trigger Suggest" (Ctrl+Space, mac Ctrl+Space, when terminal focused and widget hidden and `terminal.integrated.suggest.enabled`); `selectPrevSuggestion` "Select the Previous Suggestion" (Up; at the first item with no navigation it can hide the widget and walk history); `selectPrevPageSuggestion` (PageUp); `selectNextSuggestion` "Select the Next Suggestion" (Down); `selectNextPageSuggestion` (PageDown); `acceptSelectedSuggestion` "Insert" (Tab, and Enter under conditions); `acceptSelectedSuggestionEnter` "Accept Selected Suggestion (Enter)"; `hideSuggestWidget` "Hide Suggest Widget" (Esc); `hideSuggestWidgetAndNavigateHistory`; `suggestToggleDetails` "Suggest Toggle Details" (Ctrl+Space or Ctrl+I); `suggestToggleDetailsFocus` "Suggest Toggle Suggestion Focus" (Ctrl+Alt+Space); `terminalSuggestToggleExplainMode` "Suggest Toggle Explain Modes" (Ctrl+/); `resetSuggestWidgetSize` "Reset Suggest Widget Size". All ids are under `workbench.action.terminal.`.
- Status bar menu items (menu `MenubarTerminalSuggestStatusMenu`, all `f1: false`): `changeSelectionModeNever` "Selection Mode: None", `changeSelectionModePartial` "Selection Mode: Partial (Tab)", `changeSelectionModeAlways` "Selection Mode: Always (Tab or Enter)" (only the one matching the current setting shows), `showSuggestOnType` "Show IntelliSense while typing. This enables the quick suggestions for commands and arguments, and suggest on trigger characters settings.", `doNotShowSuggestOnType` "Don't show IntelliSense unless triggered explicitly. This disables the quick suggestions and suggest on trigger characters settings.", `suggestLearnMore` "Learn More", `configureSuggestSettings` "Configure". Tooltips exist for the three selection modes.
- Context keys: `terminalSuggestWidgetVisible`, `simpleSuggestWidgetHasFocusedSuggestion`, `simpleSuggestWidgetHasNavigated`, `simpleSuggestWidgetFirstSuggestionFocused`, `simpleSuggestWidgetExplicitlyInvoked`.
- Item kind labels shown in the list: File, Folder, Symbolic Link File, Symbolic Link Folder, Method, Alias, Argument, Option, Option Value, Flag, Commit, Branch, Tag, Stash, Remote, Pull Request, Pull Request (Done), Inline Suggestion. Each has a `terminalSymbol*Icon` and a `terminalSymbolIcon.*Foreground` color.
- Settings to attach later (`terminal.integrated.suggest.` prefix): `enabled` (true), `quickSuggestions` (object with commands, arguments, unknown, each on or off; all default off), `suggestOnTriggerCharacters` (false), `runOnEnter` (never, exactMatch, exactMatchIgnoreExtension, always; default never), `selectionMode` (partial, always, never; default partial), `showStatusBar` (true), `cdPath` (off, relative, absolute; default absolute), `inlineSuggestion` (off, alwaysOnTopExceptExactMatch, alwaysOnTop; default alwaysOnTop), `upArrowNavigatesHistory` (true), `insertTrailingSpace` (false), `providers` (object, per provider on or off), `windowsExecutableExtensions` (object). Source: `terminalSuggestConfiguration.ts`.
- Built-in extension `terminal-suggest`: contributes one command `terminal.integrated.suggest.clearCachedGlobals`, "Clear Suggest Cached Globals", category Terminal (palette only), and one terminal completion provider entry (a description, no UI). Everything else in that folder is the completion source and is left out.
- Discoverability tracker (`terminalSuggestShownTracker.ts`, storage keys `terminal.suggest.increasedDiscoverability` and `...Count`) and a command id `workbench.action.terminal.resetDiscoverability` are declared; the command registration was not found in the files read.

## Stock features with a Cursor twin that matters for later passes

- The stock inline suggestion machinery (AC-2) is the base that the Cursor bundle still contains. Cursor Tab (AC-6 to AC-8) is a separate surface. See the Cursor file.

## Left out (stock side) and why

- All of `src\vs\editor\contrib\inlineCompletions\browser\model\` (provider calls, ranking, source fetching, edit kind, typing speed, change recorder, rename processor): generation and telemetry, not UI.
- `telemetry.ts`, `structuredLogger.ts`, `changeRecorder.ts` in the inline completions folder: telemetry only.
- `src\vs\editor\contrib\suggest\browser\suggestModel.ts`, `completionModel.ts`, `suggestMemory.ts` ranking internals: only the visible effects (settings that change order and selection) are cataloged.
- Terminal suggest completion sources (`terminalCompletionService.ts`, `lspCompletionProviderAddon.ts`, `lspTerminalModelContentProvider.ts`, `extensions\terminal-suggest\src\` specs, fig parser, shell builtin caches): produce items, no UI.
- Emmet commands other than abbreviation suggestions and expansion (wrap, balance, match tag, update image size, math, increment): editing commands, not autocomplete.
- `renameSymbolTrackerService.ts` and the `editor.action.inlineSuggest.renameSymbol` flow: ids found, no UI read in this pass.
- Provider-facing experimental inline suggest settings listed in AC-2.
- Extension host and marketplace aspects of completion providers: out of scope per the brief.

## Newer than 1.128 (seen only in N)

- Setting `editor.inlineSuggest.edits.longDistanceHintContextLineCount` (number, 0 to 10, tags nextEditSuggestions and experimental): `N\src\vs\editor\common\config\editorOptions.ts`.
- Editor option `suggest.fitWidthToDetails` (CSS class `fit-width-to-details`, grows the list to fit inline detail text): `N\src\vs\editor\contrib\suggest\browser\suggestWidget.ts`, `media\suggest.css`. It is an editor option, no settings key was found; N's chat input uses it.
- Theme color `editorSuggestWidget.focusOutline` and high contrast variants for the selected row colors: `N\src\vs\editor\contrib\suggest\browser\suggestWidget.ts`.
- `editor.suggest.maxVisibleSuggestions` loses its deprecation entry in N (still deprecated and resizable in V).
- Snippet picker (Configure Snippets) moves the language of the active file to the front with tooltip "This file type was detected from the current file." and swaps label and description: `N\src\vs\workbench\contrib\snippets\browser\commands\configureSnippets.ts`.
- Inline edits view data gets `isForAnotherDocument` (suggestion that targets a different document): `N\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViewInterface.ts` and `inlineEditsView.ts`.
- Suggest details close button gains an aria label: `N\src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts`.

## Could not verify

- `editor.action.inlineSuggest.renameSymbol`: only the id was seen (`commandIds.ts`), the visible UI was not read.
- Which exact keys fire for `terminalSuggest` Enter acceptance in every selection mode: read from the `when` clause only, not run.
- `workbench.action.terminal.resetDiscoverability`: id declared, registration not located.
- Default keybindings on Linux and Windows for a few mac-only secondaries (Ctrl+N, Ctrl+P) were read from code but not checked in the live keybindings editor.
- Whether the language status entry for inline suggestions is visible without a Copilot-style provider installed (it is gated on an entitlement sentiment flag).

## Recommendation on the grouping

- Recommend the eight-group split (AC-1 to AC-8). Confidence: high - five groups map to five distinct source folders and menus in V, and the user asked for depth on Cursor Tab, which needs its own groups. The stock inline edit UI (gutter pill, side by side, jump) could be split from AC-2 as a ninth group; confidence medium that keeping it inside AC-2 is better because Cursor ships it but routes its own Tab surface elsewhere.
