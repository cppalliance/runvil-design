Section: Autocomplete (AC). Date: 2026-10-25. Pass: 1.

# 06 Autocomplete - Cursor 3.27.12 findings (pass 1, minor groups)

## How to read this file

- Scope of this file: the Cursor install (C) only. Stock VS Code detail for AC-1 to AC-5 is in the sibling file under `vscode\06-autocomplete.md`.
- Path legend:
  - `C` = `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`
  - `JS` = `C\out\vs\workbench\workbench.desktop.main.js` (main bundle)
  - `CSS` = `C\out\vs\workbench\workbench.desktop.main.css`
  - `NLS` = `C\out\nls.messages.json` (stock labels live here as numbered messages, so labels like "Accept Word" are not greppable in the JS)
  - Extensions: `C\extensions\terminal-suggest`, `C\extensions\emmet`
- Evidence for everything Cursor-specific is `bundle`: strings, ids and CSS classes found in the minified bundle. No source was read. Names inside the bundle are minified (for example the constant `RDe` holds `editor.action.acceptCursorTabSuggestion`), so only strings, ids, labels, context keys and CSS classes are recorded.
- Hard rule honored: only the UI and the interaction. No description of the model, inference, provider, prompts, or how suggestions are generated. Where a control exists next to such a thing (for example a "Model" select), only the control is noted.
- Boundary flag: `yes` where the surface shows model output (Cursor Tab accept and preview surfaces).
- Method: `rg -o` with tight patterns on JS and CSS; the quote character is matched as `\x22` because the shell strips plain double quotes.

## Findings in one paragraph

Cursor still ships the stock editor widgets (suggest list, ghost text and inline suggest toolbar, parameter hints, snippets, terminal suggest) and adds Cursor Tab as a separate set of surfaces layered on the same editor: gray inline text that reuses the stock `ghost-text` CSS class, a preview box with View and Accept buttons, line diff coloring, a diff peek view, "tab to jump" hints, an off-screen and cross-file jump indicator, an import hint, a status bar item with a hover panel, command palette commands, and a Settings page. In the stock areas the Cursor bundle looks like an older VS Code snapshot than V 1.128.0: several 1.128 strings are absent (listed in the "Present in V, not found in Cursor" section).

## Proposed minor groups (8)

Why eight and not six: five groups mirror five stock areas that a person already thinks of as separate things (suggest list, ghost text, parameter hints, snippets, terminal). The user asked for depth on Cursor Tab, and Cursor Tab has three clearly separate things a person does with it (see the suggestion and accept it, jump to the next edit, turn it on or off and tune it), so it gets three groups instead of one. Confidence: high - each group has its own command ids, CSS classes and settings, so a later tool can load them separately.

### AC-1 Suggest list (IntelliSense popup)

- Plain name: Suggest list
- One sentence: The dropdown next to the cursor while typing, with icons, labels, a details pane, a status bar, and keys to pick an item.
- Tag: both
- Evidence: source (V) and bundle (C)
- Boundary: no
- Example features: list rows with kind icon and detail; details side pane; status bar (Insert or Replace, Show More or Show Less); accept and navigate keys.
- Bundle proof (searched, found):
  - Command ids in `JS`: `editor.action.triggerSuggest`, `acceptSelectedSuggestion`, `acceptAlternativeSelectedSuggestion`, `hideSuggestWidget`, `selectNextSuggestion`, `selectPrevSuggestion`, `selectNextPageSuggestion`, `selectPrevPageSuggestion`, `selectFirstSuggestion`, `selectLastSuggestion`, `focusSuggestion`, `focusAndAcceptSuggestion`, `toggleSuggestionDetails`, `toggleExplainMode`, `toggleSuggestionFocus`, `insertBestCompletion`, `insertNextSuggestion`, `insertPrevSuggestion`, `editor.action.resetSuggestSize`
  - Menu id `suggestWidgetStatusBar`; context keys `suggestWidgetVisible`, `suggestWidgetHasFocusedSuggestion`, `suggestWidgetDetailsVisible`, `suggestWidgetMultipleSuggestions`, `suggestionMakesTextEdit`, `suggestionHasInsertAndReplaceRange`, `suggestionInsertMode`, `suggestionCanResolve`, `acceptSuggestionOnEnter`, `atEndOfWord`, `hasOtherSuggestions`
  - Labels in `NLS`: "Trigger Suggest", "Reset Suggest Widget Size", "Show More", "Show Less", "Read More", "No suggestions.", "Loading..."
  - Icon id `suggest-more-info`; colors `editorSuggestWidget.background`, `editorSuggestWidgetStatus.foreground`
  - CSS in `CSS`: `.monaco-editor .suggest-widget` with width 430px (same as V) and a fixed 3px corner radius (V 1.128 uses the `cornerRadius-large` theme token in `suggest.css` instead, another sign of an older snapshot)
  - Settings keys in `JS`: all `editor.suggest.*` keys listed in the V file are present (including every `show<Kind>` toggle, `showInlineDetails`, `showStatusBar`, `preview`, `selectionMode`, `maxVisibleSuggestions`), plus `editor.suggestSelection`, `editor.wordBasedSuggestions`, and bare option ids `quickSuggestionsDelay`, `acceptSuggestionOnEnter`, `acceptSuggestionOnCommitCharacter`, `snippetSuggestions`, `tabCompletion`, `suggestOnTriggerCharacters`, `suggestFontSize`, `suggestLineHeight`
  - Emmet extension present with the same twelve `emmet.*` settings (`C\extensions\emmet\package.json`)
- Not found in `JS`: the command id `suggestWidgetCopy`.
- Cursor difference seen in this area: none in the widget itself. Cursor Tab reuses the suggest widget colors (see AC-6) and the Tab key rules interact with `suggestWidgetVisible` (see AC-6).

### AC-2 Ghost text and inline suggestions

- Plain name: Ghost text and inline suggestions
- One sentence: Gray text at the cursor with a hover toolbar, accept all, word or line, and cycle keys; includes the stock inline edit gutter UI that Cursor still contains.
- Tag: both
- Evidence: source (V) and bundle (C)
- Boundary: no (widgets only; the provider behind it is out of scope)
- Example features: ghost text in the editor; toolbar with accept, accept word, accept line, previous and next; show next and previous keys; inline edit gutter indicator and its menu.
- Bundle proof (searched, found):
  - Command ids: `editor.action.inlineSuggest.commit`, `.hide`, `.jump`, `.showNext`, `.showPrevious`, `.trigger`, `.acceptNextWord`, `.acceptNextLine`, `.toggleAlwaysShowToolbar`, `.toggleShowCollapsed`, `.dev.extractRepro`
  - Menu ids `InlineSuggestionToolbar` and `InlineEditsActions`; context keys `inlineSuggestionVisible`, `inlineSuggestionHasIndentation`, `inlineSuggestionHasIndentationLessThanTabSize`, `inlineSuggestionSuppressSuggestions`, `cursorInIndentation`, `editor.hasSelection`, `inlineEditIsVisible`, `tabShouldJumpToInlineEdit`, `tabShouldAcceptInlineEdit`
  - Labels found in `NLS` or `JS`: "Show Next Inline Suggestion", "Accept Next Line Of Inline Suggestion", "Accept Word", "Accept Line", "Always Show Toolbar", "Jump to next inline edit", "Hide Inline Suggestion", "Suggestion:", "Reject", "Jump", "Show Collapsed", "Show Expanded", "Inline completion available", "Inline edit available", "No inline suggestion available", "Inline Suggestions"
  - Icon ids `inline-suggestion-hints-next`; colors `editorGhostText.foreground`, `inlineEdit.gutterIndicator.primaryBackground`; CSS class `inline-edits-view-gutter-indicator` is in the bundle
  - Settings keys in `JS`: `editor.inlineSuggest.enabled`, `showToolbar`, `syntaxHighlightingEnabled`, `suppressSuggestions`, `fontFamily`, `edits.allowCodeShifting`, `edits.renderSideBySide`, `edits.showCollapsed`; extra keys not registered in V 1.128 (purpose not checked): `editor.inlineSuggest.keepOnBlur`, `editor.inlineSuggest.logFetch`, `editor.inlineSuggest.logFetch.commandId`, `editor.inlineSuggest.logChangeReason.commandId`
- Cursor difference seen in this area:
  - `editor.action.inlineSuggest.acceptNextWord` is redefined in the bundle. Its precondition adds the context key `cppSuggestion` (a Cursor Tab suggestion is showing and partial accepts are on), key Ctrl+Right (CtrlCmd+Right), and it first tries the Cursor Tab partial accept and then falls back to the stock word accept. Same toolbar menu item (`InlineSuggestionToolbar`, group primary, order 2).
  - Cursor hides the stock ghost text while a Cursor Tab suggestion is showing and brings it back afterward (internal methods `disableAndHideCopilotSuggestion` and `reallowCopilotIfWePreviousHidCopilotSuggestions`).
  - Extra ids only in the bundle: `editor.action.inlineSuggest.acceptNextInlineEditPart`, `editor.action.inlineSuggest.triggerInlineEdit`, `editor.action.inlineSuggest.triggerInlineEditExplicit` (also absent from V 1.128.0 and N).
- Not found in `JS` or `NLS` (present in V): `editor.action.inlineSuggest.snooze` and `.cancelSnooze` ("Snooze Inline Suggestions"), `.commitAlternativeAction`, `.renameSymbol`, the long distance hint widget (`inline-edits-long-distance-hint-widget`), settings `minShowDelay`, `suppressInSnippetMode`, `edits.showLongDistanceHint`, the `experimental.*` keys, context keys `cursorBeforeGhostText`, `inlineSuggestionAlternativeActionVisible`, `inlineCompletions.snoozed`, and labels "Snooze", "Go To / Accept" and "Accept Inline Suggestion Alternative Action".
- Cursor's own snooze for Cursor Tab is a separate command (see AC-8).

### AC-3 Parameter hints and signature help

- Plain name: Parameter hints
- One sentence: The popup that shows the signature being typed, the active parameter, and overloads.
- Tag: both
- Evidence: source (V) and bundle (C)
- Boundary: no
- Example features: signature popup; overload switcher; trigger and close keys; docs under the signature.
- Bundle proof (searched, found): command ids `editor.action.triggerParameterHints`, `closeParameterHints`, `showPrevParameterHint`, `showNextParameterHint`; context keys `parameterHintsVisible`, `parameterHintsMultipleSignatures`; icon id `parameter-hints-next`; settings `editor.parameterHints.enabled` and `editor.parameterHints.cycle`; label "Trigger Parameter Hints" in `NLS`; CSS `.monaco-editor .parameter-hints-widget` with `.phwrapper` (max width 440px) and `.multiple`; color `editorHoverWidget.highlightForeground`.
- Cursor difference seen: none.

### AC-4 Snippets

- Plain name: Snippets
- One sentence: Templates with tab stops, placeholders and choices, plus the picker and the snippet files.
- Tag: both
- Evidence: source (V) and bundle (C)
- Boundary: no
- Example features: tab stops and placeholders; Insert Snippet picker; Configure Snippets; snippet code actions.
- Bundle proof (searched, found): ids `jumpToNextSnippetPlaceholder`, `jumpToPrevSnippetPlaceholder`, `leaveSnippet`, `insertSnippet`, `editor.action.insertSnippet`, `editor.action.showSnippets`, `editor.action.surroundWithSnippet`, `workbench.action.openSnippets`, `workbench.action.populateFileFromSnippet`; context keys `hasNextTabstop`, `hasPrevTabstop`, `hasSnippetCompletions`; setting `editor.snippets.codeActions.enabled`; color `editor.snippetTabstopHighlightBackground`; labels in `NLS`: "Insert Snippet", "Configure Snippets", "Fill File with Snippet", "Surround with Snippet...", "Go to next placeholder...", "Select a snippet", "Hide from IntelliSense".
- Not found in `JS` or `NLS`: context key `inSnippetChoice` (label "Whether the current snippet tab stop is a choice").
- Cursor difference seen: none.

### AC-5 Terminal suggestions

- Plain name: Terminal suggestions
- One sentence: The suggest popup inside the integrated terminal for commands, arguments, flags and paths, with a status bar menu.
- Tag: both (older snapshot, see below)
- Evidence: source (V) and bundle (C)
- Boundary: no
- Example features: terminal suggest popup with symbol icons; widget status menu; details toggle; accept with Tab or Enter.
- Bundle proof (searched, found): command ids `workbench.action.terminal.selectPrevSuggestion`, `selectPrevPageSuggestion`, `selectNextSuggestion`, `selectNextPageSuggestion`, `acceptSelectedSuggestion`, `acceptSelectedSuggestionEnter`, `hideSuggestWidget`, `hideSuggestWidgetAndNavigateHistory`, `resetSuggestWidgetSize`, `suggestToggleDetails`, `suggestToggleDetailsFocus`, `configureSuggestSettings`, and `terminalSuggestToggleExplainMode`; menu id `MenubarTerminalSuggestStatusMenu`; context keys `terminalSuggestWidgetVisible`, `simpleSuggestWidgetHasFocusedSuggestion`, `simpleSuggestWidgetHasNavigated`; color `terminalSymbolIcon.fileForeground`; settings `terminal.integrated.suggest.enabled`, `quickSuggestions`, `suggestOnTriggerCharacters`, `runOnEnter`, `showStatusBar`, `cdPath`, `inlineSuggestion`, `providers`, `upArrowNavigatesHistory`, `windowsExecutableExtensions`.
- Extra id only in the bundle: `workbench.action.terminal.requestCompletions` (found in a list of terminal suggest commands; not in V or N, likely an older name of the trigger command, not verified).
- Not found in `JS` or `NLS` (present in V): `workbench.action.terminal.triggerSuggest`, `changeSelectionModeNever`, `Partial`, `Always` and the "Selection Mode: ..." labels, `showSuggestOnType`, `doNotShowSuggestOnType`, `suggestLearnMore`, `resetDiscoverability`, settings `selectionMode` and `insertTrailingSpace`, and the extension command `terminal.integrated.suggest.clearCachedGlobals`. `C\extensions\terminal-suggest\package.json` has no `contributes` block.
- Cursor difference seen: none that changes the UI beyond the missing items above.

### AC-6 Cursor Tab: see a suggestion, accept it, reject it

- Plain name: Cursor Tab suggestions
- One sentence: The gray inline text, preview box, red and green line coloring, and diff preview that show a Cursor Tab edit, plus Tab to accept, a word at a time to partly accept, and Escape to reject.
- Tag: cursor
- Evidence: bundle
- Boundary: yes - every surface here displays model output and the accept action applies it.
- Example features: inline gray text and added-line view zone with a "tab" key label; preview box with "View" and "Accept Tab" buttons; Cursor Tab Preview diff peek; Tab accept, partial accept (next word), Escape reject.
- Proof paths (all in the main bundle, `JS`, plus `CSS` for classes):
  - ids: `editor.action.acceptCursorTabSuggestion`, `editor.action.acceptCppSuggestion`, `editor.action.setCppDiffPeekView`, `editor.action.configureCppDiffPeekView`, `editor.action.updateGhostText`, `editor.action.inlineSuggest.acceptNextWord`
  - strings: "Accept Cursor Tab Suggestion", "Cursor Tab Preview", "View", "Accept", "Tab", "Tab to accept suggestion", "Update Ghost Text", "Set Cursor Tab Diff Peek View", "Configure Cursor Tab Diff Peek View"
  - CSS classes: see the lists below

Facts seen (the shapes a suggestion can take):

- Inline gray text: decoration with inline class `ghost-text-decoration syntax-highlighted cursor-ghost-text` and line class `cpp-inline-suggestion`, description "ghost-text". Cursor CSS dims it (`.monaco-editor .syntax-highlighted.cursor-ghost-text` grayscale 40 percent, opacity .6; `.monaco-editor .cursor-ghost-text` no border, normal style). It reuses stock `ghost-text` classes, so the stock theme colors `editorGhostText.*` also apply.
- Added lines as a view zone: classes `cpp-ghost-text-view-zone-wrapper`, `cpp-ghost-text-view-zone-margin`, `cpp-ghost-text-view-zone-text-wrapper`, `cpp-ghost-text-view-zone-text`. The margin text is the current accept key label in lower case (falls back to "tab") with tooltip "Tab to accept suggestion". Used when the preview box would be clipped, or when forced (storage flag `shouldShowViewZoneWhenPreviewBoxIsClipped6`, not a setting).
- Preview box: a widget `div.cursorGhostTextWidget` (inner `cppOuter`, `cppInner`, `cppButtonContainer`) holding a small embedded editor in `div.cppPreviewBoxDomNode`, drawn with the suggest widget background, 2px radius. A loading look exists (`cursorGhostTextWidgetLoading`, `cppButtonContainerLoading`, `ghosttextButton`, `commandHelpText`, `cursorCppHint` with a progress bar colored border). Action buttons inside it (class `cpp-suggestion-action-buttons`, buttons `cpp-suggestion-action-button`): "View" and "Accept" with a small key chip "Tab" (`cpp-suggestion-tab-button`, 8px text, bordered like the suggest widget). The button row is built from a mouse move handler on the box; exact show rules were not verified.
- "View" opens the diff peek view; "Accept" runs the full accept.
- Diff peek view: a peek widget with title text "Cursor Tab Preview" (`cpp-peek-view-title`, `cpp-peek-view-title-text`) embedding a diff editor. Its title bar menu holds one action, `editor.action.acceptCppSuggestion`, "Accept Cursor Tab Suggestion", icon id `accept-cpp-suggestion` (check mark), menu order 1. It is shown without arrow, resizable, with a frame.
- Line and text coloring that shows what changes: `cpp-suggestion-red-background` (uses the testing failed icon color, opacity .17) and `cpp-suggestion-green-background` (testing passed icon color, opacity .12); `ghost-text-decoration-add` (diff editor inserted line background), `ghost-text-decoration-remove` (opacity .2), `ghost-text-decoration-inline-add` (inserted text background), `ghost-text-decoration-inline-remove-hint` (removed text background with line-through), `ghost-text-decoration-inline-remove-context` (opacity .2), `ghost-text-decoration-test` and `-test2` (test classes, purpose not checked).
- Gutter and highlight: `cpp-suggestion-text-decoration-showing` (faint link-colored background), `cpp-suggestion-text-decoration-showing-gutter` (3px left bar in the link color, 10px left margin), `cpp-suggestion-text-decoration-showing-streaming` (fully transparent while a suggestion is still arriving), and `-debug`, `-debug-streaming`, `-debug-streaming-pending` variants.
- Accept key: command `editor.action.acceptCursorTabSuggestion`, label "Accept Cursor Tab Suggestion", default key Tab (weight 500), precondition context key `cpp.shouldAcceptTab` equals true. Other places that read that key: the Search view tree and the Problems view table also catch Tab and run the Tab accept when the key is true (so Tab can accept while focus is in those views).
- When `cpp.shouldAcceptTab` is true (from the update function): when the accept command is not bound to Tab it is always true; false when the editor lacks focus while the suggest widget is visible; false with a multi-line selection unless vim is in a non-insert mode; true while an import hint shows; false when a "tab inserts a tab instead" rule applies (rule internals not read); otherwise true when a same-file suggestion, an off-screen jump, or a cross-file jump is showing. Also true inside the Find widget when a stored flag `cppInCmdF` is on (flag purpose inferred from the name, not verified).
- First Tab behavior on a cross-file jump: if the go-to-file indicator is visible and its code preview is not, the first Tab shows the preview instead of accepting.
- Partial accept: only when setting `cursor.cpp.enablePartialAccepts` is on. The stock `editor.action.inlineSuggest.acceptNextWord` (Ctrl+Right, toolbar item "Accept Word") is the key; context key `cppSuggestion` becomes true while a Cursor Tab suggestion exists and the setting is on.
- Reject: Escape with no modifiers while a Cursor Tab suggestion is showing clears it, then re-checks whether to show the "tab to jump" hint and the import hint. Shift+Tab is ignored by the Tab key handler. Alt held is treated as a signal to drop an older pending suggestion (method name `eventShouldKillPrevCpp`, behavior not verified).
- Not shown in the UI: telemetry event names (`cursor.acceptcppsuggestion`, `cursor.acceptcppsuggestionpartial`, `cursor.rejectcppsuggestion`, `cursor.revertcppsuggestion`, `cursor.peekcppsuggestion`, `cursor.suggestcpp`, `cursor.fullcppsuggestion`) were seen and are left out.

### AC-7 Cursor Tab: jump to the next edit and imports

- Plain name: Cursor Tab jump and imports
- One sentence: The small hints that tell you the next edit is elsewhere (below the line, off screen, or in another file) and that Tab will take you there, plus the import hint.
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the hints appear only because the model proposed a next edit or import.
- Example features: "tab to jump here" line hint; off-screen indicator "TAB next move" with up or down arrow; cross-file indicator with file icon, path and code preview; "to go back" indicator; import hint "import {name} (press Tab or ESC)".
- Proof paths (main bundle `JS` plus `CSS`): classes `cursor-prediction-hint-line-widget`, `cursorPredictionOutOfRangeIndicator`, `cursorPredictionOutOfRangeIndicatorMinimized`, `cursor-prediction-dimmed-line`, `cursor-prediction-dimmed-line-inline`, `import-prediction-widget`, `cpp-pending-import-decoration`, `tab-key`, `jump-to-edit`, `file-path`, `file-icon`; strings "to jump here", "next move", " to go back", "to ", "import ", "(press ", " or ESC)"; context keys `tabPredictionGoToFileVisibleKey` and `tabPredictionCodePreviewVisibleKey`.

Facts seen:

- Hint line widget (editor content widget id prefix `cursorPredictionHintLineWidget`, class `fancy`): a key chip (`span.tab-key`, the current Tab accept key label) followed by "to jump here" (`span.jump-to-edit`, muted). A 2px left accent bar uses the suggest widget focus highlight color. Height is line height minus 2px, text is editor font size minus 2. A `highlight` state raises opacity.
- Off-screen indicator (`cursorPredictionOutOfRangeIndicator`, with a minimized pill `...Minimized`, 20px tall, clickable): key chip with the accept key label in upper case (fallback "TAB") plus "next move" when the next edit is in this file but outside the visible range; arrow-up when it is above, arrow-down when below. Clicking runs the accept command. It shows or hides on scroll, cursor move, selection change and when the prediction changes.
- Cross-file variant: same chip plus "to" then a file icon and the target file name or path in bold italic (`file-path`), arrow-right icon, and an embedded code preview of the target with the surrounding lines dimmed (opacity .5). It is tied to the two context keys above.
- Go-back indicator: key chip showing the binding of `workbench.action.navigateBack`, then " to go back"; it hides when the selection changes or the editor blurs, and is shown at most twice (constant `MAX_GO_BACK_INDICATOR_DISPLAYS` equals 2).
- Import hint (`import-prediction-widget`, class `fancy`, pointer events off, placeholder-colored text, editor font): text like `import {symbol} (press {Tab key} or ESC)`, or the module name parsed from the action title (`import from "x"`, `from x import`, `import x`, `import x as y`). After the user has accepted an import 3 times the text only shows on hover (storage counter `howManyTimesHasUserAcceptedImportPrediction`). Escape dismisses it. It is controlled by the "Imports" toggle in Cursor Settings (AC-8).
- Stock inline edit jump UI (`editor.action.inlineSuggest.jump`, "Jump to next inline edit") also exists in the bundle (AC-2) and is a different surface.
- The constant `cursorPrediction` (string, bundle) is the internal name of this feature; no label with that name was found.

### AC-8 Cursor Tab: status bar, commands and settings

- Plain name: Cursor Tab controls
- One sentence: How you see whether Cursor Tab is on, snooze or turn it off, and tune what it does, from the status bar item, the command palette, and Cursor Settings.
- Tag: cursor
- Evidence: bundle
- Boundary: no - these are controls and status, not model output. The status hover has a "Model" select; the control is noted here, what it selects is out of scope.
- Example features: status bar item "Cursor Tab" with a hover panel (disable globally or per language, snooze); commands Enable, Disable, Toggle, Snooze, Unsnooze; Cursor Settings "Cursor Tab" section; the two `cursor.cpp.*` settings.
- Proof paths (main bundle `JS` plus `CSS`): status bar entry, command ids and labels below, settings registration (section id `cpp`, title "Cursor Tab"), Cursor Settings entry with anchor `cursor-settings-cursor-tab`.

Facts seen:

- Status bar item: registered with name "Cursor Tab", right side, priority 1000. Text is "Cursor Tab", "Cursor Tab (Snoozed)" or "Cursor Tab (Ignored)" (file matches the cursor ignore rules); the aria label is the same. CSS classes: `copilot-plus-plus-status-on`, `-off` (opacity .5 and line-through) and `-snoozed`. Clicking runs `statusBar.entry.showTooltip` when signed in, otherwise `editor.cpp.login`, "Log In to Cursor". The tooltip is a custom element that stays open on click.
- Status hover panel contents (class `cpp-status-bar-hover-element` rows):
  - Checkbox row "Disable globally" and, when there is an active language, a second checkbox "Disable for {language}" (class `cpp-status-bar-toggle-checkbox`)
  - Row "Model" with a select (UI control; options not cataloged)
  - Row with a link "Snooze" (or "Unsnooze") and an info icon with tooltip "Temporarily disable Cursor Tab suggestions for a specified duration. You can unsnooze at any time."; while snoozed it shows "({time} remaining)" counting down every second
  - Other states: "Requires pro (custom model). " with an "Upgrade here." link; "Please log in"; "Cursor Tab is unavailable in Cursor Private Inference"; "Cursor Tab: Off under private inference"
- Commands (id, label, palette visibility):
  - `editor.action.enableCppGlobally`, "Enable Cursor Tab", palette
  - `editor.cpp.disableenabled`, "Disable Cursor Tab", palette
  - `editor.cpp.toggle`, "Toggle Cursor Tab", palette
  - `editor.cpp.snooze`, "Snooze Cursor Tab", palette, only when enabled and not snoozed; quick pick placeholder "Select Cursor Tab snooze duration", items "1 minute", "5 minutes", "15 minutes", "30 minutes", "1 hour", "3 hours", each with description "Snooze for {label}"
  - `editor.cpp.unsnooze`, "Unsnooze Cursor Tab", palette, only when snoozed
  - `editor.cpp.login`, "Log In to Cursor", palette
  - `editor.cpp.openPro`, "Open Cursor Tab Pro Pricing", hidden from palette
  - `workbench.action.copyRecentCppActions`, "Developer: Copy Recent Cursor Tab Actions", palette; messages "No Cursor Tab actions found" and "Copied N Cursor Tab request IDs + diffs to clipboard" (boundary: yes, shows model output)
  - `workbench.action.reportCppAction`, "Developer: Report Cursor Tab Action" (the registered title says "See Recent Cursor Tab Actions"), palette; quick pick placeholder "Select a Cursor Tab request (last 20 requests)" (boundary: yes)
  - `workbench.action.disableAutocompletes`: constant found, no command or label found using it
  - A table that maps `editor.cpp.snooze`, `editor.cpp.unsnooze`, `editor.cpp.toggle`, `editor.cpp.disableenabled` and `editor.action.enableCppGlobally` to a group named `tab_controls` (alongside other groups such as `editor_ai_actions`); what reads the table was not checked
- Context keys: `cppEnabled` ("Whether Cursor Tab is currently enabled", default true), `cppSnoozed` ("Whether Cursor Tab is currently snoozed"), `cppSuggestion`, `cpp.shouldAcceptTab`, `tabPredictionGoToFileVisibleKey`, `tabPredictionCodePreviewVisibleKey`.
- Registered settings (settings.json, configuration section id `cpp`, title "Cursor Tab"):
  - `cursor.cpp.disabledLanguages`: array of strings, default `["markdown","plaintext"]`, "Disable Cursor Tab for these languages"
  - `cursor.cpp.enablePartialAccepts`: boolean, default false, "Enable partial accepts for Cursor Tab, using the editor.action.inlineSuggest.acceptNextWord keybinding"
- Cursor Settings page (entry registered in the settings search list under category id `indexing`, id `cursor-tab`, label "Cursor Tab"; the category's visible label was not found). Rows, in order:
  - "Cursor Tab" toggle, "Context-aware, multi-line suggestions around your cursor based on recent edits"
  - (shown only when on) "Partial Accepts", "Accept the next word of a suggestion via {keybinding}", with an info button "Keybinding: {keybinding}" (the keybinding label falls back to the string "Alt+]" if none is found)
  - "Suggestions While Commenting", "Allow Tab to trigger while in a comment region"
  - "Whitespace-Only Suggestions", "Suggest edits like new lines and indentation that modify whitespace only"
  - "Imports", "Automatically import necessary modules for TypeScript" (or "TypeScript and Python")
  - "Auto Import for Python" with a BETA badge, "Enable auto import for Python. This is a beta feature." (shown only when Imports is on and an internal config flag `pythonEnabled` is not already on)
  - "Ignored Files", "Glob patterns for files where Cursor Tab will not suggest", a list field with placeholder "e.g., *.md, **/generated/**"
  - Notices under the section: "CPP and auto-imports are temporarily disabled", "CPP is temporarily disabled", "Auto-imports are temporarily disabled"
- Stored app flags behind the page (application user storage, not settings.json): `cppEnabled`, `cppSnoozed`, `cppSnoozeDuration` (default 5 minutes), `cppIgnorePatterns`, `cppTriggerInComments`, `cppShowWhitespaceOnlyChanges`, `cppAutoImportEnabled`, `userHasEnabledImportPredictionForPython`, `cppModel`.

## Cursor additions (quick index)

- Commands: `editor.action.acceptCursorTabSuggestion` ("Accept Cursor Tab Suggestion", Tab), `editor.action.acceptCppSuggestion` (peek title action), `editor.action.updateGhostText` ("Update Ghost Text"), `editor.action.setCppDiffPeekView`, `editor.action.configureCppDiffPeekView`, `editor.action.enableCppGlobally` ("Enable Cursor Tab"), `editor.cpp.disableenabled` ("Disable Cursor Tab"), `editor.cpp.toggle` ("Toggle Cursor Tab"), `editor.cpp.snooze` ("Snooze Cursor Tab"), `editor.cpp.unsnooze` ("Unsnooze Cursor Tab"), `editor.cpp.login` ("Log In to Cursor"), `editor.cpp.openPro` ("Open Cursor Tab Pro Pricing"), `workbench.action.copyRecentCppActions`, `workbench.action.reportCppAction`, `cursorai.action.cppAfterOnboarding` (constant, no label or behavior found), `workbench.action.disableAutocompletes` (constant only).
- Redefined stock command: `editor.action.inlineSuggest.acceptNextWord` (adds the `cppSuggestion` condition).
- Context keys: `cppEnabled`, `cppSnoozed`, `cppSuggestion`, `cpp.shouldAcceptTab`, `tabPredictionGoToFileVisibleKey`, `tabPredictionCodePreviewVisibleKey`.
- Settings keys: `cursor.cpp.disabledLanguages`, `cursor.cpp.enablePartialAccepts`.
- Labels and strings: "Cursor Tab", "Cursor Tab (Snoozed)", "Cursor Tab (Ignored)", "Cursor Tab Preview", "to jump here", "next move", "to go back", "Partial Accepts", "Suggestions While Commenting", "Whitespace-Only Suggestions", "Imports", "Auto Import for Python", "Ignored Files".
- Status bar: one item, name "Cursor Tab", right side, priority 1000.
- Module name seen in the bundle: `cppDebouncingService.js` (timing for when suggestions show, left out).
- CSS classes: `cpp-*` (ghost text view zone, suggestion decorations, action buttons, status bar rows, peek title), `cursorGhostTextWidget`, `cppPreviewBoxDomNode`, `cursor-ghost-text`, `cursor-prediction-*`, `cursorPredictionOutOfRangeIndicator*`, `import-prediction-widget`, `copilot-plus-plus-status-*`, and stock-derived `ghost-text-decoration-*` variants.

## Present in V 1.128.0 but not found in the Cursor bundle (searched ids and labels)

- Inline suggest: `editor.action.inlineSuggest.snooze`, `.cancelSnooze`, `.commitAlternativeAction`, `.renameSymbol`; the long distance hint widget; settings `editor.inlineSuggest.minShowDelay`, `suppressInSnippetMode`, `edits.showLongDistanceHint`, `experimental.*`, `triggerCommandOnProviderChange`; context keys `inlineSuggestionAlternativeActionVisible`, `inlineCompletions.snoozed`, `cursorBeforeGhostText`; labels "Snooze", "Go To / Accept", "Accept Inline Suggestion Alternative Action".
- Suggest and snippets: command `suggestWidgetCopy`, context key `inSnippetChoice`.
- Terminal suggest: `workbench.action.terminal.triggerSuggest`, `changeSelectionModeNever`, `Partial`, `Always` (and the "Selection Mode: ..." labels), `showSuggestOnType`, `doNotShowSuggestOnType`, `suggestLearnMore`, `resetDiscoverability`, settings `terminal.integrated.suggest.selectionMode` and `insertTrailingSpace`, extension command `terminal.integrated.suggest.clearCachedGlobals`.
- Reading: in these stock areas the Cursor bundle behaves like an older VS Code snapshot than 1.128.0. For catalog tags, items in this list should stay `vscode` (V only), not `both`.

## Cross-section candidates seen (not cataloged here)

- Browser URL bar: a ghost completion inside the address field (classes `url-input-ghost`, `url-input-ghost-typed`, `url-input-ghost-completion`, input placeholder "Enter URL or search...") and an address suggestion dropdown (`omnibox-dropdown`, `omnibox-item-*`, `omnibox-delete-button`, `omnibox-bookmark-indicator`). Both belong with the Browser feature; no model output seen.
- Chat input ghost text node type `composer-ghost-text` (class `composer-ghost-text`): chat, out of scope.
- Cursor Settings page itself is a Cursor in-scope feature owned by another section; only the Cursor Tab labels are recorded in AC-8.

## Left out and why

- Anything about how suggestions are produced or timed: `cppDebouncingService`, request and stream handling, telemetry event names (`cursor.*cppsuggestion*`), model list contents behind the "Model" select, privacy and routing text, upsell and plan strings other than the two visible hover states, the report and recent actions debug payloads.
- Chat and agent surfaces that happen to use ghost text or typeahead (composer ghost text, @-mention menus, Cmd+K prompt bar, terminal chat): out of scope per the brief.
- Stock provider internals and telemetry for inline completions and terminal suggest (see the V file).
- Extension host and marketplace parts of completion providers.
- The `terminal-suggest` extension source folder beyond its manifest.

## Newer than 1.128

- Nothing Cursor-specific is newer than 1.128. The stock items seen only in N are listed in the V file (`editor.inlineSuggest.edits.longDistanceHintContextLineCount`, `suggest.fitWidthToDetails` option, `editorSuggestWidget.focusOutline`, snippet picker active-language hint, `isForAnotherDocument` in inline edit view data).

## Could not verify

- Exact trigger rules for the Cursor Tab view zone, preview box and diff coloring (which shape shows for which edit): only the code paths and classes were seen, nothing was run.
- Whether the stock inline edit UI (gutter pill, side by side, jump) ever appears in Cursor: the code and labels are present, but no run was done.
- Visible label of the Cursor Settings category that contains the Cursor Tab section (category id `indexing`).
- What reads the `tab_controls` command group table.
- Purpose of `workbench.action.disableAutocompletes`, `cursorai.action.cppAfterOnboarding`, the `cppInCmdF` flag, and the Alt key rule `eventShouldKillPrevCpp`.
- Default keybindings for Cursor-added commands other than Tab accept: no other default key was found in the code read; the user keybindings editor was not opened.
- Whether `workbench.action.terminal.requestCompletions` is a live command or a leftover id.
- Light and dark colors of Cursor-only CSS were taken from variable names in the CSS; the resolved colors were not checked.

