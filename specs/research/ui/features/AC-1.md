Group: AC-1 Suggest list. Section: Autocomplete. Date: 2026-10-25. Pass: 2.
Features: 46.

### AC-1.1 Open the list by hand (Trigger Suggest)
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Space or Ctrl+I in an editor to open the suggest list at the cursor, or run Trigger Suggest from the Command Palette.
- Commands: editor.action.triggerSuggest
- Menus: none found
- Keys: Ctrl+Space; Ctrl+I (mac: Ctrl+Space, Alt+Esc, Cmd+I) when editorTextFocus; only while the editor is writable, has a completion provider and the list is hidden
- Settings: none found
- Cursor: present (same id, label "Trigger Suggest" and same keys in the bundle)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (TriggerSuggestAction, lines 795-832); V\src\vs\editor\contrib\quickAccess\browser\commandsQuickAccess.ts (editor actions listed in the palette); C bundle (id editor.action.triggerSuggest, message 1605 "Trigger Suggest")

### AC-1.2 Open the list by itself while typing (quick suggestions)
- Tag: both
- Evidence: source
- Boundary: no
- What: While you type a word the list opens on its own after a short delay, unless quick suggestions are off, a snippet is active and the snippet rule is on, or an inline suggestion is showing and suppresses it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.quickSuggestions; editor.quickSuggestionsDelay; editor.suggest.snippetsPreventQuickSuggestions
- Cursor: changed (settings found with values on, inline, off only; stock 1.128 adds offWhenInlineCompletions and defaults code to it, Cursor defaults code to on)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (_doTriggerQuickSuggest lines 387-455, LineContext.shouldAutoTrigger lines 67-87); V\src\vs\editor\common\config\editorOptions.ts (EditorQuickSuggestions lines 3744-3860, quickSuggestionsDelay default 10 at line 6516); C bundle (quickSuggestions defaults other on, comments off, strings off; enum on, inline, off)

### AC-1.3 Quick suggestions inside comments and strings
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn quick suggestions on or off separately for comments, for strings and for other code, so the list does or does not open on its own while you type inside a comment or a string.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.quickSuggestions
- Cursor: present (the object with other, comments and strings is in the bundle, defaults on, off, off; the Cursor Tab row "Suggestions While Commenting" is a different control, see AC-8)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (token type check via getStandardTokenType, lines 421-431); V\src\vs\editor\contrib\suggest\browser\suggest.ts (QuickSuggestionsOptions.valueFor, line 454); V\src\vs\editor\common\config\editorOptions.ts (lines 3790-3806); C bundle (class with defaults, valueFor with comments 1 and strings 2)

### AC-1.4 Open the list on trigger characters
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing a character that the language registers as a trigger, such as a dot, opens the list at once and keeps rows that came from other sources.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggestOnTriggerCharacters
- Cursor: present (option id suggestOnTriggerCharacters found, default true)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (_updateTriggerCharacters, lines 221-301); V\src\vs\editor\common\config\editorOptions.ts (lines 6691-6694); C bundle (option id and description in message 679)

### AC-1.5 Hold back the list while an inline suggestion is showing
- Tag: both
- Evidence: source
- Boundary: no
- What: When ghost text is on screen the list can stay out of the way - quick suggestions and trigger characters are skipped if inline suggestions suppress it, and stock 1.128 can also turn quick suggestions or word-based suggestions off only while inline completions are present.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.suppressSuggestions; editor.quickSuggestions; editor.wordBasedSuggestions
- Cursor: changed (the suppressSuggestions check is in the bundle; the values offWhenInlineCompletions and offWithInlineSuggestions are not)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (canShowQuickSuggest and canShowSuggestOnTriggerCharacters lines 110-131, wait for inline completions lines 423-521); V\src\vs\editor\common\config\editorConfigurationSchema.ts (lines 66-77); V\src\vs\editor\browser\services\editorWorkerService.ts (line 281); C bundle (function reading inlineSuggestionVisible and suppressSuggestions; zero hits for offWhenInlineCompletions)

### AC-1.6 Loading and No suggestions messages
- Tag: both
- Evidence: source
- Boundary: no
- What: After a manual open the list shows a small "Loading..." box after a short delay and then either the rows or "No suggestions."; an automatic open that finds nothing stays hidden.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (messages "Loading..." and "No suggestions." found, widget state code toggles the message class)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (LOADING_MESSAGE and NO_SUGGESTIONS_MESSAGE lines 104-105, _setState lines 454-515, showTriggered lines 531-541); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (delays 50 and 250 ms, line 233); C bundle (messages 201 and 1617)

### AC-1.7 Kind icon on each row, with color swatch and file icons
- Tag: both
- Evidence: source
- Boundary: no
- What: Each row starts with a small icon for its kind (method, function, class, snippet and so on), a color swatch for color items, and the file icon theme's icon for file and folder items.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.showIcons
- Cursor: changed (28 kinds found; stock 1.128 has 29 with a Tool kind and its tools icon, which the bundle lacks)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (renderElement lines 163-203, color extractor lines 31-57); V\src\vs\editor\common\languages.ts (CompletionItemKinds lines 389-470); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (icon, no-icons, customcolor rules lines 313-347); C bundle (kind icon map ends Issue then Snippet; message labels 828-855 have no Tool)

### AC-1.8 Row text: label, signature and detail
- Tag: both
- Evidence: source
- Boundary: no
- What: After the icon each row shows the label, a dimmed signature when the item has a structured label, and on the right a smaller detail or description (always for structured labels, only on the focused row for plain-detail items).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.showInlineDetails
- Cursor: present (setting and the row classes signature-label, details-label, string-label, can-expand-details found in bundle and CSS)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (renderElement lines 163-244; the qualifier-label span is created at line 124 but no code fills it); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 140-300); C bundle (CSS rules for the same classes)

### AC-1.9 Matched characters highlighted in the label
- Tag: both
- Evidence: source
- Boundary: no
- What: The letters you typed are drawn bold in a highlight color inside each label, with a second color for the selected row, and the bold is dropped while the list is frozen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (colors editorSuggestWidget.highlightForeground and focusHighlightForeground found, CSS rule with :not(.frozen) found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (createMatches(element.score), line 172); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 148-162); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (color registrations lines 41-51); C bundle (color ids and CSS)

### AC-1.10 Deprecated items struck through
- Tag: both
- Evidence: source
- Boundary: no
- What: Items the provider marks as deprecated are dimmed with a line through the name and no match highlighting, and they can be hidden from the list entirely.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.showDeprecated
- Cursor: present (setting and the CSS rule for .monaco-icon-label.deprecated found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (lines 206-210); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 301-309); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (createSuggestFilter, lines 678-713); C bundle (setting and CSS)

### AC-1.11 Filter and order the list as you type
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing narrows the list by fuzzy match and sorts it by match quality, sort text and optionally closeness to the cursor; after a manual open the list freezes on its last results instead of closing when nothing matches, until you type a non-word character.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.filterGraceful; editor.suggest.localityBonus; editor.suggest.matchOnWordStartOnly
- Cursor: present (all three settings found, frozen state class toggled in the widget code)
- Source: V\src\vs\editor\contrib\suggest\browser\completionModel.ts (_createCachedState lines 123-230); V\src\vs\editor\contrib\suggest\browser\suggest.ts (comparators lines 339-386); V\src\vs\editor\contrib\suggest\browser\wordDistance.ts; V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (_onNewContext lines 716-840); C bundle (settings; classList.toggle("frozen"))

### AC-1.12 Snippet items in the list
- Tag: both
- Evidence: source
- Boundary: no
- What: Snippets from the user, the workspace and extensions show as rows with the snippet icon and their prefix, placed at the top, at the bottom, mixed in by sort text, or left out.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.snippetSuggestions; editor.suggest.showSnippets
- Cursor: present (option id snippetSuggestions with top, bottom, inline, none and the setting showSnippets found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (snippetSortOrder lines 560-575); V\src\vs\editor\contrib\suggest\browser\suggest.ts (SnippetSortOrder lines 165-175, comparators 358-386); V\src\vs\workbench\contrib\snippets\browser\snippetCompletionProvider.ts (label, detail and sort text lines 52-59); V\src\vs\editor\common\config\editorOptions.ts (lines 6654-6667)

### AC-1.13 Word-based suggestions in the list
- Tag: both
- Evidence: source
- Boundary: no
- What: Plain words found in the current or other open documents appear as rows with the text icon, with scope from the current document up to all documents, or off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.wordBasedSuggestions; editor.suggest.showWords
- Cursor: changed (setting found with four values and default matchingDocuments; stock 1.128 has five values and defaults to offWithInlineSuggestions)
- Source: V\src\vs\editor\common\config\editorConfigurationSchema.ts (lines 66-77); V\src\vs\editor\browser\services\editorWorkerService.ts (lines 270-305); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (showWords maps to the Text kind, line 703); C bundle (schema entry with enum off, currentDocument, matchingDocuments, allDocuments)

### AC-1.14 Show or hide rows by kind
- Tag: both
- Evidence: source
- Boundary: no
- What: One on or off setting per kind - methods, functions, fields, classes, keywords, files, colors, users, issues and more - removes rows of that kind from the list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.showMethods; editor.suggest.showFunctions; editor.suggest.showConstructors; editor.suggest.showFields; editor.suggest.showVariables; editor.suggest.showClasses; editor.suggest.showStructs; editor.suggest.showInterfaces; editor.suggest.showModules; editor.suggest.showProperties; editor.suggest.showEvents; editor.suggest.showOperators; editor.suggest.showUnits; editor.suggest.showValues; editor.suggest.showConstants; editor.suggest.showEnums; editor.suggest.showEnumMembers; editor.suggest.showKeywords; editor.suggest.showColors; editor.suggest.showFiles; editor.suggest.showReferences; editor.suggest.showCustomcolors; editor.suggest.showFolders; editor.suggest.showTypeParameters; editor.suggest.showUsers; editor.suggest.showIssues; editor.suggest.filteredTypes; editor.suggest
- Cursor: present (every show setting above found by search; filteredTypes found as the deprecated object)
- Source: V\src\vs\editor\common\config\editorOptions.ts (EditorSuggest lines 5093-5412); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (createSuggestFilter lines 678-713); C bundle (setting ids)

### AC-1.15 Pre-select the item you used last time
- Tag: both
- Evidence: source
- Boundary: no
- What: The row that is highlighted when the list opens can be the first one, the one you last accepted here, or the one you accepted for the same typed prefix, and that memory can be shared across workspaces.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggestSelection; editor.suggest.shareSuggestSelections
- Cursor: present (both settings and the memory code reading editor.suggestSelection found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestMemory.ts (NoMemory, LRUMemory, PrefixMemory, storage prefix suggest/memories, lines 20-300); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (selectors lines 150-153, 241-250); V\src\vs\editor\common\config\editorOptions.ts (lines 6695-6707, 5160-5165)

### AC-1.16 Selection mode when the list opens by itself
- Tag: both
- Evidence: source
- Boundary: no
- What: When the list opens on its own, choose whether a row is highlighted at once, never, only after a trigger character, or only while typing; a manual open always highlights a row.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.selectionMode
- Cursor: present (setting with always, never, whenTriggerCharacter, whenQuickSuggestion found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 256-274); V\src\vs\editor\common\config\editorOptions.ts (lines 5166-5177); C bundle (setting id and enum)

### AC-1.17 Preview the highlighted item in the editor
- Tag: both
- Evidence: source
- Boundary: no
- What: With preview on, the text that the highlighted row would insert is drawn as ghost text in the editor behind the list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.preview
- Cursor: present (setting and the previewMode default subwordSmart found; the drawing code was not traced)
- Source: V\src\vs\editor\common\config\editorOptions.ts (preview and previewMode lines 5099-5109, 5192-5196, 5380); V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts; V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (lines 131-133); C bundle (setting id)

### AC-1.18 Font size and row height of the list
- Tag: both
- Evidence: source
- Boundary: no
- What: Set the list's text size and row height (0 means use the editor's values), which also sizes the status bar row and the details text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggestFontSize; editor.suggestLineHeight
- Cursor: present (option ids suggestFontSize and suggestLineHeight found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (configureFont lines 132-157); V\src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts (_configureFont lines 88-105); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (getLayoutInfo lines 924-941); V\src\vs\editor\common\config\editorOptions.ts (lines 6681-6690)

### AC-1.19 Place and resize the list
- Tag: both
- Evidence: source
- Boundary: no
- What: The list opens below the cursor, or above it when there is no room, starts 430 wide and 12 rows high, can be dragged by its edges and remembers that size; Reset Suggest Widget Size clears the memory.
- Commands: editor.action.resetSuggestSize
- Menus: none found
- Keys: none found
- Settings: editor.suggest.maxVisibleSuggestions
- Cursor: present (reset id, label "Reset Suggest Widget Size", the suggestWidget.size storage key and the 430 px default found; maxVisibleSuggestions is deprecated in both)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (resize handlers lines 172-210, _layout lines 815-900, PersistedWidgetSize lines 68-100); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1137-1150); V\src\vs\editor\common\config\editorOptions.ts (line 5202); C bundle (id, message 1606)

### AC-1.20 Theme colors and corners of the list
- Tag: both
- Evidence: source
- Boundary: no
- What: The list, its border, the selected row, match highlights, status text and icons take their colors from named theme colors that a theme or the person can override, and the frame uses a rounded corner and a shadow.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Cursor: changed (all nine editorSuggestWidget color ids found; the frame uses a fixed 3px radius and no large shadow token where stock 1.128 uses the corner radius and shadow tokens)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (lines 41-51); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 7-35); V\src\vs\workbench\services\themes\common\themeConfiguration.ts (line 140); C bundle (color ids; CSS rule .monaco-editor .suggest-widget with border-radius 3px)

### AC-1.21 Open and close the details pane with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: With a row highlighted, Ctrl+Space or Ctrl+I opens the details pane beside the list and the same keys close it, a small X in the pane closes it too, and the open or closed state is remembered between sessions.
- Commands: toggleSuggestionDetails
- Menus: none found
- Keys: Ctrl+Space; Ctrl+I (mac: Ctrl+Space, Cmd+I) when editorTextFocus, the list is visible and a row is focused
- Settings: none found
- Cursor: present (id, keys and the expandSuggestionDocs storage key found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1013-1038); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (toggleDetails lines 718-732, _isDetailsVisible lines 943-950); V\src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts (close button lines 63-66, 196-203); C bundle (id toggleSuggestionDetails with the same keys)

### AC-1.22 Read More chevron on the focused row
- Tag: both
- Evidence: source
- Boundary: no
- What: When the focused item has docs, a small chevron titled "Read More" appears at the right end of the row on hover and a click opens the details pane; it stays hidden while the status bar is shown.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (icon id suggest-more-info and the label "Read More" found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (icon line 26, readMore lines 129-130 and 226-243); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 76-80, 160-180, 266-300); C bundle (message 1627, icon registration)

### AC-1.23 Details pane contents and Markdown docs
- Tag: both
- Evidence: source
- Boundary: no
- What: The pane shows the item's detail text (type or signature) at the top and its docs below, as plain text or Markdown with links and code, scrolls inside, and shows "Loading..." while the item is still resolving.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (details widget classes no-docs, no-type, detail-and-doc, markdown-docs and the "Loading..." text found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts (renderItem lines 130-208, renderLoading lines 122-128, canExpandCompletionItem lines 22-24); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (resolve and re-render lines 403-445); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 350-470)

### AC-1.24 Details pane placement and resizing
- Tag: both
- Evidence: source
- Boundary: no
- What: The pane sits to the right of the list, or to the left or below it when there is no room (above it when the list is drawn above the cursor), can be resized from its edges, and keeps the size you drag it to until the list closes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (east, west and south placements found; no north placement, so the pane is not put above when the list is above the cursor)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts (SuggestDetailsOverlay lines 263-488, north placement lines 417-425, 460-463, 477-480); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (_positionDetails lines 917-922); C bundle (placeAtAnchor with three placements)

### AC-1.25 Focus, scroll and copy in the details pane
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+Alt+Space moves focus into the pane (opening it if needed) and back to the list, the page, first and last commands scroll the pane while it has focus, and Ctrl+C copies selected text from it.
- Commands: toggleSuggestionFocus; suggestWidgetCopy
- Menus: none found
- Keys: Ctrl+Alt+Space (mac: Ctrl+Alt+Space) when editorTextFocus and the list is visible; Ctrl+C, and Ctrl+Insert on Windows, when suggestWidgetDetailsFocused
- Settings: none found
- Cursor: changed (toggleSuggestionFocus found; no suggestWidgetCopy command and no suggestWidgetDetailsFocused key, an older keydown handler lets Ctrl+C through and sends other keys back to the editor)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1050-1135); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (toggleDetailsFocus lines 702-716, details focus tracker lines 298-306, page and first or last in State.Details lines 602-684); C bundle (id toggleSuggestionFocus; onDetailsKeyDown handler; zero hits for suggestWidgetCopy and suggestWidgetDetailsFocused)

### AC-1.26 Explain mode
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+/ turns the details pane into a debug view that shows the match score, typed prefix, word, sort index, commit characters and provider name of the highlighted item.
- Commands: toggleExplainMode
- Menus: none found
- Keys: Ctrl+/ while the list is visible
- Settings: none found
- Cursor: present (id toggleExplainMode and key Ctrl+/ found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1040-1048); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (toggleExplainMode lines 761-770); V\src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts (lines 134-146)

### AC-1.27 Grow the list to fit inline details
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. The editor option suggest.fitWidthToDetails widens the list so the longest label and its inline detail fit, up to the editor width, unless the person has dragged a size.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing (no hit for fitWidthToDetails or the fit-width-to-details class in the bundle or CSS)
- Source: N\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (applyFitWidthStyle, _measureContentWidth); N\src\vs\editor\contrib\suggest\browser\media\suggest.css; N\src\vs\editor\common\config\editorOptions.ts (lines 5044, 5183, 5454; option only, no settings key)

### AC-1.28 Status bar with Insert or Replace
- Tag: both
- Evidence: source
- Boundary: no
- What: A thin bar under the rows (off by default) shows the accept action for the focused row as Insert or Replace, followed by the opposite action when the item supports both, so you can see what Tab and Shift+Tab will do.
- Commands: acceptSelectedSuggestion; acceptAlternativeSelectedSuggestion
- Menus: suggestWidgetStatusBar (group left, orders 1 and 2)
- Keys: none found
- Settings: editor.suggest.showStatusBar
- Cursor: changed (Insert, Replace and the alternate entries found; they lack the has-focused-suggestion condition that stock 1.128 puts on each)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (menu entries lines 857-905); V\src\vs\editor\contrib\suggest\browser\suggestWidgetStatus.ts; V\src\vs\editor\contrib\suggest\browser\suggest.ts (menu id, line 45); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 38-80); C bundle (menu id suggestWidgetStatusBar, seven entries, messages 1598-1604)

### AC-1.29 Show More or Show Less in the status bar
- Tag: both
- Evidence: source
- Boundary: no
- What: At the right of the status bar a text action reads Show More while the details pane is closed and Show Less while it is open, and a click toggles the pane; it shows only when the item has docs, a different detail or can be resolved.
- Commands: toggleSuggestionDetails
- Menus: suggestWidgetStatusBar (group right, order 1)
- Keys: Ctrl+Space; Ctrl+I (same keys as AC-1.21)
- Settings: editor.suggest.showStatusBar
- Cursor: present (two right-group entries with Show Less and Show More found, messages 1603 and 1604)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1013-1038, canResolve lines 211-212); V\src\vs\editor\contrib\suggest\browser\suggestWidgetStatus.ts (left and right action bars lines 40-80); C bundle (menu entries with CanResolve)

### AC-1.30 Select action in the status bar when no row is focused
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When the list is open with nothing highlighted (selection mode never, or a trigger-character open), the status bar offers "Select" on the left, which highlights the next row; stock 1.128 shows Insert and Replace only once a row is focused.
- Commands: selectNextSuggestion
- Menus: suggestWidgetStatusBar (group left, order 0)
- Keys: Down; Ctrl+Down (mac adds Ctrl+N) when the list is visible
- Settings: editor.suggest.selectionMode
- Cursor: missing (the selectNextSuggestion registration in the bundle has no status bar entry and the message table has no "Select" between Replace and Show Less)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 923-941); C bundle (selectNextSuggestion registration ends after kbOpts; menu entries 1598-1604 only)

### AC-1.31 Accept with Tab
- Tag: both
- Evidence: source
- Boundary: no
- What: With a row highlighted, Tab inserts it, closes the list and runs any extra edits or follow-up command that the item has.
- Commands: acceptSelectedSuggestion
- Menus: suggestWidgetStatusBar (the Insert entry, see AC-1.28)
- Keys: Tab when suggestWidgetVisible and editorTextInputFocus (a row must be focused)
- Settings: none found
- Cursor: changed (id and key found; Tab is also bound to editor.action.acceptCursorTabSuggestion at weight 500 against 190 here, active while cpp.shouldAcceptTab is true, see AC-6)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (command lines 842-858, _insertSuggestion lines 309-510); C bundle (id acceptSelectedSuggestion with weight 190; id editor.action.acceptCursorTabSuggestion with precondition cpp.shouldAcceptTab and weight 500)

### AC-1.32 Accept with Enter (accept on enter)
- Tag: both
- Evidence: source
- Boundary: no
- What: Enter also inserts the highlighted row when accept-on-enter is on, and with the smart value only when the row would change the text, so Enter still makes a new line after you have typed the whole word.
- Commands: acceptSelectedSuggestion
- Menus: none found
- Keys: Enter when suggestWidgetVisible, editorTextInputFocus, acceptSuggestionOnEnter and suggestionMakesTextEdit
- Settings: editor.acceptSuggestionOnEnter
- Cursor: present (command, the old alias acceptSelectedSuggestionOnEnter, and the values on, smart, off found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (makes-text-edit context lines 184-206, accept-on-enter key lines 288-296, alias line 909); V\src\vs\editor\common\config\editorOptions.ts (lines 5956-5968); C bundle (registration with Km.AcceptSuggestionsOnEnter and Km.MakesTextEdit)

### AC-1.33 Accept with a commit character
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing a character that the highlighted item lists as a commit character, such as a semicolon or a dot, inserts that item first and then types the character.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.acceptSuggestionOnCommitCharacter
- Cursor: present (option id found and the commit-character check on typing found in the bundle)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestCommitCharacters.ts; V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 168-170); V\src\vs\editor\common\config\editorOptions.ts (lines 5952-5955); C bundle (acceptCharacters.has check inside onWillType)

### AC-1.34 Accept as Insert or Replace
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose whether accepting overwrites the text to the right of the cursor (replace) or leaves it (insert) by default, and flip that for one accept with Shift+Enter or Shift+Tab when the item supports both.
- Commands: acceptAlternativeSelectedSuggestion
- Menus: suggestWidgetStatusBar (the alternate entry, see AC-1.28)
- Keys: Shift+Enter; Shift+Tab when editorTextInputFocus and a row is focused
- Settings: editor.suggest.insertMode
- Cursor: present (id, both keys and the setting found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (getOverwriteInfo lines 572-589, command lines 881-905, context keys lines 157-159 and 205-208); V\src\vs\editor\common\config\editorOptions.ts (lines 5141-5150); C bundle (registration with Shift+Enter and Shift+Tab)

### AC-1.35 Accept by clicking a row
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking or tapping a row inserts it and keeps focus in the editor, and clicking inside the list never steals focus from the text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (_onListMouseDownOrTap and _onEditorMouseDown handlers found in the widget code)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (lines 278-279, 328-371); C bundle (same method names)

### AC-1.36 Move through the list with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: Up and Down move the highlight and wrap at the ends, PageUp and PageDown move a page, and first and last commands exist without default keys; the keys act only when the list has more than one row or nothing is highlighted yet.
- Commands: selectNextSuggestion; selectPrevSuggestion; selectNextPageSuggestion; selectPrevPageSuggestion; selectFirstSuggestion; selectLastSuggestion
- Menus: none found
- Keys: Down or Ctrl+Down (mac adds Ctrl+N); Up or Ctrl+Up (mac adds Ctrl+P); PageDown or Ctrl+PageDown; PageUp or Ctrl+PageUp; first and last have none; all when editorTextFocus and the list is visible
- Settings: none found
- Cursor: present (all six ids and the key sets found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 921-990); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (selectNext and related lines 602-684)

### AC-1.37 Focus the first row from the editor
- Tag: both
- Evidence: source
- Boundary: no
- What: When the list is open with nothing highlighted, pressing the trigger key again highlights the first row, and a command exists that highlights and accepts in one step.
- Commands: focusSuggestion; focusAndAcceptSuggestion
- Menus: none found
- Keys: Ctrl+Space; Ctrl+I (mac: Ctrl+Space, Cmd+I) when the list is visible and no row is focused; focusAndAcceptSuggestion has none
- Settings: none found
- Cursor: present (both ids found, key set matches)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 992-1011); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (focusSelected lines 596-600)

### AC-1.38 Close the list
- Tag: both
- Evidence: source
- Boundary: no
- What: Escape or Shift+Escape closes the list and the details pane, and the list also closes when the editor loses focus, the cursor moves to another line, or nothing matches after an automatic open.
- Commands: hideSuggestWidget
- Menus: none found
- Keys: Escape; Shift+Escape when editorTextInputFocus and the list is visible
- Settings: none found
- Cursor: present (id and keys found, blur handler cancels and clears)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 911-922, blur handler lines 281-286); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (cancel paths lines 346-374 and 722-735)

### AC-1.39 Tab completion without the list
- Tag: both
- Evidence: source
- Boundary: no
- What: With tab completion on, Tab at the end of a word inserts the best match straight away, and Tab again or Shift+Tab then steps through the other matches; the onlySnippets value completes only snippet prefixes (see AC-4).
- Commands: insertBestCompletion; insertNextSuggestion; insertPrevSuggestion
- Menus: none found
- Keys: Tab (insertBestCompletion, and insertNextSuggestion when hasOtherSuggestions); Shift+Tab (insertPrevSuggestion); only when editor.tabCompletion is on, at the end of a word, the list is hidden and no snippet is active
- Settings: editor.tabCompletion
- Cursor: present (all three ids and the setting found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1062-1117); V\src\vs\editor\contrib\suggest\browser\suggestAlternatives.ts; V\src\vs\editor\contrib\suggest\browser\wordContextKey.ts; V\src\vs\editor\common\config\editorOptions.ts (lines 6708-6720)

### AC-1.40 Screen reader support for the list
- Tag: both
- Evidence: source
- Boundary: no
- What: The list is exposed as a listbox and the text area points at the focused row, each row is read with its label, signature, description and kind (plus detail and docs when the pane is open), "Loading..." and "No suggestions." are announced, and accepting an item with extra edits announces how many.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (aria label messages 1619-1623, status messages and the additional edits alert 1597 found)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (accessibilityProvider lines 232-264, activeDescendant lines 391 and 445, status lines 487 and 497); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (_alertCompletionItem lines 591-595); V\src\vs\editor\browser\controller\editContext\textArea\textAreaEditContext.ts (lines 645-660); V\src\vs\editor\browser\controller\editContext\native\screenReaderSupport.ts (lines 177-185); C bundle (NLS 1597, 1617-1623)

### AC-1.41 Keep the open list in step when you delete or move along the word
- Tag: both
- Evidence: source
- Boundary: no
- What: While the list is open, Backspace and the cursor keys re-filter it at the new cursor position - moving left onto a word asks for fresh rows, starting a new word to the right asks again and reuses the rows it already has, and the list closes when the cursor goes to another line or the line's leading whitespace changes (for example when the word is outdented).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the cursor-change check that only reacts to keyboard and deleteLeft moves, and the retrigger that passes providerItemsToReuse, were found in the bundle)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (_onCursorChange lines 345-376, _onNewContext lines 717-841); C bundle (source check for keyboard and deleteLeft, retrigger with providerItemsToReuse)

### AC-1.42 Color the kind icons by kind
- Tag: both
- Evidence: source
- Boundary: no
- What: Each kind icon in the list takes its own color from a per-kind theme color (method, function, class, variable, keyword, snippet and others; 33 colors in all, shared with the Outline and breadcrumbs), and a theme or the person can override each one; a color item shows its swatch instead (see AC-1.7).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Cursor: present (all 33 symbolIcon color ids and the rules that color .codicon-symbol-* icons inside the editor were found; the workbench-side rule differs but the list sits inside the editor)
- Source: V\src\vs\editor\contrib\symbolIcons\browser\symbolIcons.ts (33 registerColor calls, lines 10-124); V\src\vs\editor\contrib\symbolIcons\browser\symbolIcons.css (lines 8-75); V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (kind icon class, line 202); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (import that loads the colors, line 23); C bundle (symbolIcon.*Foreground ids; CSS rule .monaco-editor .codicon.codicon-symbol-method colored by the vscode-symbolIcon-methodForeground variable)

### AC-1.43 Highlight the row that matches the ghost text
- Tag: both
- Evidence: source
- Boundary: no
- What: When an inline suggestion (ghost text) is on screen and the list opens, the row whose text is the start of the ghost text is highlighted first, and the longest such row wins, ahead of the remembered choice from AC-1.15, so the list and the ghost text line up.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (a selector registered at priority 100 that reads the selected inline completion as a single text edit was found in the bundle; whether any provider feeds ghost text to it in Cursor was not checked)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts (selector lines 56-89); V\src\vs\editor\contrib\inlineCompletions\browser\model\singleTextEditHelpers.ts (singleTextEditAugments lines 26-29); V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (preselector source, lines 138-142); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (registerSelector line 764, default selector lines 150-153); C bundle (registerSelector with priority:100; selectedInlineCompletion toSingleTextEdit)

### AC-1.44 Hold Shift to preview the other Insert or Replace result
- Tag: both
- Evidence: source
- Boundary: no
- What: With the list preview on, holding Shift redraws the ghost text of the highlighted row as it would look if accepted with the opposite insert or replace mode, which is what Shift+Enter or Shift+Tab will then do; letting go of Shift draws the normal preview again.
- Commands: none found
- Menus: none found
- Keys: Shift held down (a key listener, not a keybinding rule; the accept keys themselves are in AC-1.34)
- Settings: editor.suggest.preview; editor.suggest.insertMode
- Cursor: present (the key down and key up handlers that set isShiftKeyPressed and refresh the preview were found in the same adapter class)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts (key listeners lines 42-54, fromSuggestion with toggleMode lines 181-196); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (getOverwriteInfo with toggleMode, lines 573-589); V\src\vs\editor\common\config\editorOptions.ts (preview default false, line 5105); C bundle (onKeyDown and onKeyUp handlers with isShiftKeyPressed)

### AC-1.45 Move the list above the cursor while the preview spans several lines
- Tag: both
- Evidence: source
- Boundary: no
- What: When the previewed text of the highlighted row shows as ghost text on two or more lines, the list is drawn above the cursor, as long as there are at least 150 px of room there, so it does not cover the ghost lines; it goes back to the normal placement once the preview is gone.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.preview
- Cursor: present (forceRenderingAbove and stopForceRenderingAbove on the list, the 150 px check and the primaryGhostText line count of 2 or more were found in the bundle)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (lines 302-314); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (state with suggestItem, lines 659-679); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (forceRenderingAbove lines 879-880 and 951-960); C bundle (forceRenderingAbove with lineCount>=2 and g>150)

### AC-1.46 Path and header suggestions in Markdown links
- Tag: both
- Evidence: source
- Boundary: no
- What: In a Markdown file, typing inside a link target lists files, folders and headers as rows in the suggest list; one setting turns the path rows off, another chooses whether headers from other workspace files appear (never, after ##, or after # or ##), and Markdown files also default quick suggestions to off for other code, comments and strings.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: markdown.suggest.paths.enabled; markdown.suggest.paths.includeWorkspaceHeaderCompletions; editor.quickSuggestions
- Cursor: present (both markdown.suggest settings, their labels and the [markdown] quick suggestions default are in the Cursor copy of the extension manifest)
- Source: V\extensions\markdown-language-features\package.json (settings lines 418-439, [markdown] configurationDefaults lines 833-842); V\extensions\markdown-language-features\package.nls.json (configuration.markdown.suggest.paths.*); C\extensions\markdown-language-features\package.json and package.nls.json

## Handed off
- Quick suggestions value `inline` (suggest items drawn as ghost text by SuggestInlineCompletions) and the ghost text drawing used by the preview in AC-1.17: AC-2 (confidence high, AC-2 owns ghost text drawing; the setting stays here).
- Tab completion of snippet prefixes (`onlySnippets`, `insertSnippet` on Tab with `hasSnippetCompletions`), the snippet choice dropdown and the Hide from IntelliSense buttons in the Insert Snippet picker: AC-4 (confidence high, all are in the snippet files).
- The terminal suggest popup, which shares this layout through simpleSuggestWidget: AC-5 (confidence high, already its own group).
- Cursor Tab accept on Tab (`editor.action.acceptCursorTabSuggestion`, `cpp.shouldAcceptTab`) and the Search view and Problems view Tab handlers: AC-6 (confidence high, they accept Cursor Tab edits).
- The Text Editor > Suggestions page of the Settings editor (`editor/suggestions`, settings pattern `editor.*suggest*`, present in stock source and the Cursor bundle): ED-8 (confidence high, ED-8 owns the Settings editor; the settings themselves are attached to features above).
- The three lines about suggest in the editor accessibility help (`suggestActions`, `acceptSuggestAction`, `toggleSuggestionFocus` in `V\src\vs\editor\common\standaloneStrings.ts`, shown by `V\src\vs\workbench\contrib\accessibility\browser\editorAccessibilityHelp.ts`; Cursor's first line says "code completions" where stock says "inline suggestions"): no owner in the catalog, nearest is ED-4 (confidence low, the accessible view is listed as unowned).

## Unclear
- Pass 1 recorded `editor.quickSuggestions` defaults as on, off, off and the stock 1.128 source says otherwise: the code default is offWhenInlineCompletions, comments off, strings off, and the allowed values are on, inline, off and offWhenInlineCompletions (`V\src\vs\editor\common\config\editorOptions.ts` lines 3744-3775). The on, off, off defaults and three values match the Cursor bundle, not stock 1.128. Pass 1 also left out that Cursor's `editor.wordBasedSuggestions` has four values and defaults to matchingDocuments.
- The CSS class `docs-side` has rules in `suggest.css` but no code in V 1.128 or the Cursor bundle adds it (only removes it), so details placement is decided by the overlay placement code, not that class. The pass 1 phrase "side or below" is therefore the east, west, south and (stock only) north placement list.
- Each row creates a `span.qualifier-label` in V but no code fills it in 1.128, so the description text shows in the right-hand details label. Pass 1 listed it as the description slot.
- `editor.suggest.showCustomcolors` exists as a setting but V's filter for the Customcolor kind reads `showColors` (`V\src\vs\editor\contrib\suggest\browser\suggestModel.ts` line 707), so that setting looks inert.
- `editor.suggest.previewMode` is a valid option (prefix, subword, subwordSmart) but has no settings key in V or the Cursor bundle.
- Newer-than-1.128 tweaks to existing features, not listed as features: theme color `editorSuggestWidget.focusOutline` and high-contrast variants of the selected row colors, a flat shadow with a high-contrast override in `suggest.css`, an aria label on the details Close button, and `editor.suggest.maxVisibleSuggestions` losing its deprecation entry (`N\src\vs\editor\contrib\suggest\browser\suggestWidget.ts`, `suggestWidgetDetails.ts`, `media\suggest.css`).
- No Cursor-only feature was found in the editor suggest list. Cursor's differences are all older-snapshot pieces (see the changed and missing rows above) plus the shared Tab key with Cursor Tab.
- Emmet abbreviation rows appear in this list but the emmet extension is not on the in-scope extension list, so its settings (`emmet.showExpandedAbbreviation` and the rest) are left out.
- Several callers open this same list from code (settings.json edits and the Settings search bar in `preferencesService.ts` and `settingsSearchMenu.ts`, launch.json Add Configuration, MCP config edits). They add no suggest UI, so they are not listed.
- All Cursor facts come from strings, ids and code fragments in the bundle. Nothing was run on screen, so which of Cursor Tab or the suggest list gets Tab in a given moment (the `cpp.shouldAcceptTab` rule) was read, not observed.
- Overflow check on 2026-10-25 added 6 features (AC-1.41 to AC-1.46).
