# 6. Autocomplete (AC)

Only the UI and the interaction. The model that produces suggestions is out.

8 groups, 227 features. Each feature lists its entry points and the pieces that finish it.

## AC-1 Suggest list

(both). The IntelliSense dropdown with icons, details pane, status bar and accept keys. Examples: rows with kind icon and detail; the details pane with Read More; the Insert and Replace status bar; Tab, Enter and arrow keys.

46 features.

#### AC-1.1 Open the list by hand (Trigger Suggest)

- Tag: both. Evidence: source. Boundary: no. Cursor: present (same id, label "Trigger Suggest" and same keys in the bundle)
- What: Press Ctrl+Space or Ctrl+I in an editor to open the suggest list at the cursor, or run Trigger Suggest from the Command Palette.
- Commands: editor.action.triggerSuggest
- Menus: none found
- Keys: Ctrl+Space; Ctrl+I (mac: Ctrl+Space, Alt+Esc, Cmd+I) when editorTextFocus; only while the editor is writable, has a completion provider and the list is hidden
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (TriggerSuggestAction, lines 795-832); V\src\vs\editor\contrib\quickAccess\browser\commandsQuickAccess.ts (editor actions listed in the palette); C bundle (id editor.action.triggerSuggest, message 1605 "Trigger Suggest")
- Pieces:
  1. Pressing `Ctrl+Space` or `Ctrl+I` in a writable editor opens the suggest list at the cursor.
  2. On macOS the keys that open the list are `Ctrl+Space`, `Alt+Esc` and `Cmd+I`.
  3. Running `Trigger Suggest` (`editor.action.triggerSuggest`) from the Command Palette opens the suggest list at the cursor.
  4. The keys and the command open the list only while it is hidden, and do nothing in a read-only editor or in a language that has no completion provider.

#### AC-1.2 Open the list by itself while typing (quick suggestions)

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (settings found with values on, inline, off only; stock 1.128 adds offWhenInlineCompletions and defaults code to it, Cursor defaults code to on)
- What: While you type a word the list opens on its own after a short delay, unless quick suggestions are off, a snippet is active and the snippet rule is on, or an inline suggestion is showing and suppresses it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.quickSuggestions; editor.quickSuggestionsDelay; editor.suggest.snippetsPreventQuickSuggestions
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (_doTriggerQuickSuggest lines 387-455, LineContext.shouldAutoTrigger lines 67-87); V\src\vs\editor\common\config\editorOptions.ts (EditorQuickSuggestions lines 3744-3860, quickSuggestionsDelay default 10 at line 6516); C bundle (quickSuggestions defaults other on, comments off, strings off; enum on, inline, off)
- Pieces:
  1. While you type a word in an editor, the suggest list opens on its own after a short delay.
  2. Setting `editor.quickSuggestionsDelay` sets how many milliseconds the list waits before it opens on its own, with a default of 10.
  3. Setting `editor.quickSuggestions` chooses whether typing opens the list, with values `on`, `inline` (the suggestion is drawn as ghost text and not as a list) and `off`.
  4. Setting `editor.suggest.snippetsPreventQuickSuggestions` chooses whether the list stays closed while typing inside an active snippet, with values on and off, default off.
  5. In Cursor the `editor.quickSuggestions` entry for ordinary code defaults to `on`, so typing in code opens the list on its own out of the box. [cursor only]

#### AC-1.3 Quick suggestions inside comments and strings

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the object with other, comments and strings is in the bundle, defaults on, off, off; the Cursor Tab row "Suggestions While Commenting" is a different control, see AC-8)
- What: Turn quick suggestions on or off separately for comments, for strings and for other code, so the list does or does not open on its own while you type inside a comment or a string.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.quickSuggestions
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (token type check via getStandardTokenType, lines 421-431); V\src\vs\editor\contrib\suggest\browser\suggest.ts (QuickSuggestionsOptions.valueFor, line 454); V\src\vs\editor\common\config\editorOptions.ts (lines 3790-3806); C bundle (class with defaults, valueFor with comments 1 and strings 2)
- Pieces:
  1. Setting `editor.quickSuggestions` has three separate entries, `other`, `comments` and `strings`, so typing in ordinary code, inside a comment and inside a string can each open the list on their own or not.
  2. By default the list opens on its own while typing in ordinary code and stays closed while typing inside a comment or inside a string.
  3. Setting the `comments` entry to `on` makes the list open on its own while you type inside a comment.
  4. Setting the `strings` entry to `on` makes the list open on its own while you type inside a string.
  5. Setting the `other` entry to `off` stops the list from opening on its own in ordinary code while the comment and string entries keep their own values.

#### AC-1.4 Open the list on trigger characters

- Tag: both. Evidence: source. Boundary: no. Cursor: present (option id suggestOnTriggerCharacters found, default true)
- What: Typing a character that the language registers as a trigger, such as a dot, opens the list at once and keeps rows that came from other sources.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggestOnTriggerCharacters
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (_updateTriggerCharacters, lines 221-301); V\src\vs\editor\common\config\editorOptions.ts (lines 6691-6694); C bundle (option id and description in message 679)
- Pieces:
  1. Typing a trigger character such as a dot opens the list at once, without waiting for the quick suggestion delay.
  2. Typing a trigger character while the list is showing rows adds the rows from the sources that registered that character and keeps the rows that came from other sources.
  3. Setting `editor.suggestOnTriggerCharacters` chooses whether trigger characters open the list, with values on and off, default on.

#### AC-1.5 Hold back the list while an inline suggestion is showing

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the suppressSuggestions check is in the bundle; the values offWhenInlineCompletions and offWithInlineSuggestions are not)
- What: When ghost text is on screen the list can stay out of the way - quick suggestions and trigger characters are skipped if inline suggestions suppress it, and stock 1.128 can also turn quick suggestions or word-based suggestions off only while inline completions are present.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.suppressSuggestions; editor.quickSuggestions; editor.wordBasedSuggestions
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (canShowQuickSuggest and canShowSuggestOnTriggerCharacters lines 110-131, wait for inline completions lines 423-521); V\src\vs\editor\common\config\editorConfigurationSchema.ts (lines 66-77); V\src\vs\editor\browser\services\editorWorkerService.ts (line 281); C bundle (function reading inlineSuggestionVisible and suppressSuggestions; zero hits for offWhenInlineCompletions)
- Pieces:
  1. While ghost text from an inline suggestion is on screen, typing does not open the list on its own and trigger characters do not open it either, when inline suggestions are set to suppress it.
  2. Setting `editor.inlineSuggest.suppressSuggestions` chooses whether an inline suggestion holds the list back, with values on and off.
  3. Stock VS Code 1.128 adds the value `offWhenInlineCompletions` to `editor.quickSuggestions`, which turns opening on its own off only while inline completions are present, and Cursor does not have it. [stock only]
  4. Stock VS Code 1.128 adds the value `offWithInlineSuggestions` to `editor.wordBasedSuggestions`, which turns word rows off only while inline suggestions are present, and Cursor does not have it. [stock only]

#### AC-1.6 Loading and No suggestions messages

- Tag: both. Evidence: source. Boundary: no. Cursor: present (messages "Loading..." and "No suggestions." found, widget state code toggles the message class)
- What: After a manual open the list shows a small "Loading..." box after a short delay and then either the rows or "No suggestions."; an automatic open that finds nothing stays hidden.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (LOADING_MESSAGE and NO_SUGGESTIONS_MESSAGE lines 104-105, _setState lines 454-515, showTriggered lines 531-541); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (delays 50 and 250 ms, line 233); C bundle (messages 201 and 1617)
- Pieces:
  1. After a manual open, a small "Loading..." box appears at the cursor if the sources have not answered after a short delay.
  2. When the answers arrive, the "Loading..." box is replaced by the rows.
  3. When a manual open finds nothing, the box reads "No suggestions."
  4. When an automatic open finds nothing, the list stays hidden and no message is shown.

#### AC-1.7 Kind icon on each row, with color swatch and file icons

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (28 kinds found; stock 1.128 has 29 with a Tool kind and its tools icon, which the bundle lacks)
- What: Each row starts with a small icon for its kind (method, function, class, snippet and so on), a color swatch for color items, and the file icon theme's icon for file and folder items.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.showIcons
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (renderElement lines 163-203, color extractor lines 31-57); V\src\vs\editor\common\languages.ts (CompletionItemKinds lines 389-470); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (icon, no-icons, customcolor rules lines 313-347); C bundle (kind icon map ends Issue then Snippet; message labels 828-855 have no Tool)
- Pieces:
  1. Each row starts with a small icon for the kind of its item, such as method, function, class or snippet.
  2. A row for a color item shows a small color swatch in place of the kind icon.
  3. Rows for file and folder items show the icon from the active file icon theme.
  4. Setting `editor.suggest.showIcons` chooses whether rows show icons, with values on and off, default on.
  5. Stock VS Code 1.128 has a 29th kind, Tool, drawn with a tools icon, and Cursor has 28 kinds and no Tool icon. [stock only]

#### AC-1.8 Row text: label, signature and detail

- Tag: both. Evidence: source. Boundary: no. Cursor: present (setting and the row classes signature-label, details-label, string-label, can-expand-details found in bundle and CSS)
- What: After the icon each row shows the label, a dimmed signature when the item has a structured label, and on the right a smaller detail or description (always for structured labels, only on the focused row for plain-detail items).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.showInlineDetails
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (renderElement lines 163-244; the qualifier-label span is created at line 124 but no code fills it); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 140-300); C bundle (CSS rules for the same classes)
- Pieces:
  1. After the icon, each row shows the label of its item as the main text.
  2. When an item has a structured label, a dimmed signature follows the label on the same row.
  3. Rows with a structured label always show a smaller description or detail text at the right end.
  4. Rows whose item has only plain detail text show that detail at the right end only on the highlighted row.
  5. Setting `editor.suggest.showInlineDetails` chooses whether detail text shows inline with the label or only in the details pane, with values on and off, default on.

#### AC-1.9 Matched characters highlighted in the label

- Tag: both. Evidence: source. Boundary: no. Cursor: present (colors editorSuggestWidget.highlightForeground and focusHighlightForeground found, CSS rule with :not(.frozen) found)
- What: The letters you typed are drawn bold in a highlight color inside each label, with a second color for the selected row, and the bold is dropped while the list is frozen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (createMatches(element.score), line 172); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 148-162); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (color registrations lines 41-51); C bundle (color ids and CSS)
- Pieces:
  1. The letters you typed are drawn bold in a highlight color inside each row's label wherever they match.
  2. The selected row draws its matched letters in a second highlight color that differs from the one on other rows.
  3. While the list is frozen on its last results, the matched letters are not bold.

#### AC-1.10 Deprecated items struck through

- Tag: both. Evidence: source. Boundary: no. Cursor: present (setting and the CSS rule for .monaco-icon-label.deprecated found)
- What: Items the provider marks as deprecated are dimmed with a line through the name and no match highlighting, and they can be hidden from the list entirely.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.showDeprecated
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (lines 206-210); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 301-309); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (createSuggestFilter, lines 678-713); C bundle (setting and CSS)
- Pieces:
  1. A row for an item the source marks as deprecated is dimmed and its label has a line through it.
  2. A deprecated row gets no match highlighting on its label.
  3. Setting `editor.suggest.showDeprecated` chooses whether deprecated rows are listed at all, with values on and off, default on.

#### AC-1.11 Filter and order the list as you type

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all three settings found, frozen state class toggled in the widget code)
- What: Typing narrows the list by fuzzy match and sorts it by match quality, sort text and optionally closeness to the cursor; after a manual open the list freezes on its last results instead of closing when nothing matches, until you type a non-word character.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.filterGraceful; editor.suggest.localityBonus; editor.suggest.matchOnWordStartOnly
- Source: V\src\vs\editor\contrib\suggest\browser\completionModel.ts (_createCachedState lines 123-230); V\src\vs\editor\contrib\suggest\browser\suggest.ts (comparators lines 339-386); V\src\vs\editor\contrib\suggest\browser\wordDistance.ts; V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (_onNewContext lines 716-840); C bundle (settings; classList.toggle("frozen"))
- Pieces:
  1. Typing while the list is open narrows it to the rows whose labels fuzzy match what you typed.
  2. The remaining rows are ordered by match quality and then by the sort text the source gave.
  3. Setting `editor.suggest.filterGraceful` chooses whether filtering and sorting forgive small typos, with values on and off, default on.
  4. Setting `editor.suggest.localityBonus` chooses whether words that appear close to the cursor sort higher, with values on and off, default off.
  5. Setting `editor.suggest.matchOnWordStartOnly` chooses whether the first typed letter must match at the start of a word, with values on and off, default on.
  6. After a manual open, if nothing matches any more the list freezes on its last results instead of closing, until you type a character that is not part of a word.

#### AC-1.12 Snippet items in the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (option id snippetSuggestions with top, bottom, inline, none and the setting showSnippets found)
- What: Snippets from the user, the workspace and extensions show as rows with the snippet icon and their prefix, placed at the top, at the bottom, mixed in by sort text, or left out.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.snippetSuggestions; editor.suggest.showSnippets
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (snippetSortOrder lines 560-575); V\src\vs\editor\contrib\suggest\browser\suggest.ts (SnippetSortOrder lines 165-175, comparators 358-386); V\src\vs\workbench\contrib\snippets\browser\snippetCompletionProvider.ts (label, detail and sort text lines 52-59); V\src\vs\editor\common\config\editorOptions.ts (lines 6654-6667)
- Pieces:
  1. Snippets from the user, the workspace and extensions appear as rows with the snippet icon and the snippet prefix as the label.
  2. A snippet row shows the snippet name as its description, and its detail text is the snippet description followed by the snippet source in brackets.
  3. Setting `editor.snippetSuggestions` chooses where snippet rows sit, with values `top`, `bottom`, `inline` (mixed in by sort text) and `none` (left out).
  4. Setting `editor.suggest.showSnippets` chooses whether snippet rows are listed, with values on and off, default on.

#### AC-1.13 Word-based suggestions in the list

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (setting found with four values and default matchingDocuments; stock 1.128 has five values and defaults to offWithInlineSuggestions)
- What: Plain words found in the current or other open documents appear as rows with the text icon, with scope from the current document up to all documents, or off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.wordBasedSuggestions; editor.suggest.showWords
- Source: V\src\vs\editor\common\config\editorConfigurationSchema.ts (lines 66-77); V\src\vs\editor\browser\services\editorWorkerService.ts (lines 270-305); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (showWords maps to the Text kind, line 703); C bundle (schema entry with enum off, currentDocument, matchingDocuments, allDocuments)
- Pieces:
  1. Plain words found in documents appear as rows with the text icon.
  2. Setting `editor.wordBasedSuggestions` chooses which documents the words come from, with values `off`, `currentDocument`, `matchingDocuments` and `allDocuments`.
  3. Setting `editor.suggest.showWords` chooses whether word rows are listed, with values on and off, default on.
  4. In Cursor the setting `editor.wordBasedSuggestions` defaults to `matchingDocuments`. [cursor only]

#### AC-1.14 Show or hide rows by kind

- Tag: both. Evidence: source. Boundary: no. Cursor: present (every show setting above found by search; filteredTypes found as the deprecated object)
- What: One on or off setting per kind - methods, functions, fields, classes, keywords, files, colors, users, issues and more - removes rows of that kind from the list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.showMethods; editor.suggest.showFunctions; editor.suggest.showConstructors; editor.suggest.showFields; editor.suggest.showVariables; editor.suggest.showClasses; editor.suggest.showStructs; editor.suggest.showInterfaces; editor.suggest.showModules; editor.suggest.showProperties; editor.suggest.showEvents; editor.suggest.showOperators; editor.suggest.showUnits; editor.suggest.showValues; editor.suggest.showConstants; editor.suggest.showEnums; editor.suggest.showEnumMembers; editor.suggest.showKeywords; editor.suggest.showColors; editor.suggest.showFiles; editor.suggest.showReferences; editor.suggest.showCustomcolors; editor.suggest.showFolders; editor.suggest.showTypeParameters; editor.suggest.showUsers; editor.suggest.showIssues; editor.suggest.filteredTypes; editor.suggest
- Source: V\src\vs\editor\common\config\editorOptions.ts (EditorSuggest lines 5093-5412); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (createSuggestFilter lines 678-713); C bundle (setting ids)
- Pieces:
  1. Turning off one of the kind settings removes the rows of that kind from the list, and every kind setting is on by default.
  2. Settings `editor.suggest.showMethods`, `editor.suggest.showFunctions`, `editor.suggest.showConstructors`, `editor.suggest.showFields`, `editor.suggest.showVariables`, `editor.suggest.showProperties` and `editor.suggest.showEvents` each choose whether rows of that member kind are listed, with values on and off.
  3. Settings `editor.suggest.showClasses`, `editor.suggest.showStructs`, `editor.suggest.showInterfaces`, `editor.suggest.showModules`, `editor.suggest.showEnums` and `editor.suggest.showTypeParameters` each choose whether rows of that type kind are listed, with values on and off.
  4. Settings `editor.suggest.showOperators`, `editor.suggest.showUnits`, `editor.suggest.showValues`, `editor.suggest.showConstants` and `editor.suggest.showEnumMembers` each choose whether rows of that value kind are listed, with values on and off.
  5. Setting `editor.suggest.showKeywords` chooses whether keyword rows are listed, with values on and off.
  6. Settings `editor.suggest.showColors`, `editor.suggest.showCustomcolors`, `editor.suggest.showFiles`, `editor.suggest.showFolders` and `editor.suggest.showReferences` each choose whether rows of that kind are listed, with values on and off.
  7. Settings `editor.suggest.showUsers` and `editor.suggest.showIssues` each choose whether user and issue rows are listed, with values on and off.
  8. Setting `editor.suggest.filteredTypes` is shown as deprecated with a message that points to the separate show settings, and a kind set to false in it turns the matching show setting off when that one has no value of its own.

#### AC-1.15 Pre-select the item you used last time

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both settings and the memory code reading editor.suggestSelection found)
- What: The row that is highlighted when the list opens can be the first one, the one you last accepted here, or the one you accepted for the same typed prefix, and that memory can be shared across workspaces.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggestSelection; editor.suggest.shareSuggestSelections
- Source: V\src\vs\editor\contrib\suggest\browser\suggestMemory.ts (NoMemory, LRUMemory, PrefixMemory, storage prefix suggest/memories, lines 20-300); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (selectors lines 150-153, 241-250); V\src\vs\editor\common\config\editorOptions.ts (lines 6695-6707, 5160-5165)
- Pieces:
  1. With `editor.suggestSelection` set to `first`, the first row is highlighted whenever the list opens.
  2. With `editor.suggestSelection` set to `recentlyUsed`, the row you last accepted in a similar place is highlighted when the list opens, if it is in the list.
  3. With `editor.suggestSelection` set to `recentlyUsedByPrefix`, the row you accepted the last time you typed the same prefix is highlighted when the list opens.
  4. Setting `editor.suggest.shareSuggestSelections` chooses whether the remembered choices are shared between workspaces and windows, with values on and off, default off.

#### AC-1.16 Selection mode when the list opens by itself

- Tag: both. Evidence: source. Boundary: no. Cursor: present (setting with always, never, whenTriggerCharacter, whenQuickSuggestion found)
- What: When the list opens on its own, choose whether a row is highlighted at once, never, only after a trigger character, or only while typing; a manual open always highlights a row.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.selectionMode
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 256-274); V\src\vs\editor\common\config\editorOptions.ts (lines 5166-5177); C bundle (setting id and enum)
- Pieces:
  1. A manual open always highlights a row, whatever `editor.suggest.selectionMode` says.
  2. Setting `editor.suggest.selectionMode` chooses whether an automatic open highlights a row, with values `always`, `never`, `whenTriggerCharacter` and `whenQuickSuggestion`, default `always`.
  3. With `never`, an automatic open shows the list with no row highlighted, so Tab and Enter insert nothing until you move the highlight.
  4. With `whenTriggerCharacter` a row is highlighted only when a trigger character opened the list, and with `whenQuickSuggestion` only when typing opened it.

#### AC-1.17 Preview the highlighted item in the editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present (setting and the previewMode default subwordSmart found; the drawing code was not traced)
- What: With preview on, the text that the highlighted row would insert is drawn as ghost text in the editor behind the list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.preview
- Source: V\src\vs\editor\common\config\editorOptions.ts (preview and previewMode lines 5099-5109, 5192-5196, 5380); V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts; V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (lines 131-133); C bundle (setting id)
- Pieces:
  1. With `editor.suggest.preview` on, the text that the highlighted row would insert is drawn as ghost text in the editor behind the list.
  2. Moving the highlight to another row redraws the ghost text for that row.
  3. Closing the list removes the ghost text.
  4. Setting `editor.suggest.preview` chooses whether the highlighted row is previewed in the editor, with values on and off, default off.

#### AC-1.18 Font size and row height of the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (option ids suggestFontSize and suggestLineHeight found)
- What: Set the list's text size and row height (0 means use the editor's values), which also sizes the status bar row and the details text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggestFontSize; editor.suggestLineHeight
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (configureFont lines 132-157); V\src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts (_configureFont lines 88-105); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (getLayoutInfo lines 924-941); V\src\vs\editor\common\config\editorOptions.ts (lines 6681-6690)
- Pieces:
  1. Setting `editor.suggestFontSize` sets the text size of the list, and a value of 0 uses the editor font size.
  2. Setting `editor.suggestLineHeight` sets the row height of the list, and a value of 0 uses the editor line height, with a minimum of 8.
  3. The status bar row and the details pane text follow the same list font size.

#### AC-1.19 Place and resize the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (reset id, label "Reset Suggest Widget Size", the suggestWidget.size storage key and the 430 px default found; maxVisibleSuggestions is deprecated in both)
- What: The list opens below the cursor, or above it when there is no room, starts 430 wide and 12 rows high, can be dragged by its edges and remembers that size; Reset Suggest Widget Size clears the memory.
- Commands: editor.action.resetSuggestSize
- Menus: none found
- Keys: none found
- Settings: editor.suggest.maxVisibleSuggestions
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (resize handlers lines 172-210, _layout lines 815-900, PersistedWidgetSize lines 68-100); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1137-1150); V\src\vs\editor\common\config\editorOptions.ts (line 5202); C bundle (id, message 1606)
- Pieces:
  1. The list opens below the cursor, or above it when there is no room below.
  2. The list opens 430 pixels wide and 12 rows high.
  3. Dragging an edge of the list resizes it, and the next list opens at the size you left it.
  4. Running `Reset Suggest Widget Size` (`editor.action.resetSuggestSize`) from the Command Palette clears the remembered size.
  5. Setting `editor.suggest.maxVisibleSuggestions` is shown as deprecated with a message that the list can now be resized.

#### AC-1.20 Theme colors and corners of the list

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (all nine editorSuggestWidget color ids found; the frame uses a fixed 3px radius and no large shadow token where stock 1.128 uses the corner radius and shadow tokens)
- What: The list, its border, the selected row, match highlights, status text and icons take their colors from named theme colors that a theme or the person can override, and the frame uses a rounded corner and a shadow.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (lines 41-51); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 7-35); V\src\vs\workbench\services\themes\common\themeConfiguration.ts (line 140); C bundle (color ids; CSS rule .monaco-editor .suggest-widget with border-radius 3px)
- Pieces:
  1. The list background, border and text take their colors from the theme colors `editorSuggestWidget.background`, `editorSuggestWidget.border` and `editorSuggestWidget.foreground`.
  2. The selected row takes its colors from `editorSuggestWidget.selectedBackground`, `editorSuggestWidget.selectedForeground` and `editorSuggestWidget.selectedIconForeground`.
  3. Match highlights and the status bar text take their colors from `editorSuggestWidget.highlightForeground`, `editorSuggestWidget.focusHighlightForeground` and `editorSuggestWidgetStatus.foreground`.
  4. Setting `workbench.colorCustomizations` lets a person override any of these list colors.
  5. The list frame is drawn with a rounded corner and a shadow.
  6. In Cursor the frame has a fixed corner radius of 3px and its shadow does not use the large shadow token. [cursor only]

#### AC-1.21 Open and close the details pane with the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, keys and the expandSuggestionDocs storage key found)
- What: With a row highlighted, Ctrl+Space or Ctrl+I opens the details pane beside the list and the same keys close it, a small X in the pane closes it too, and the open or closed state is remembered between sessions.
- Commands: toggleSuggestionDetails
- Menus: none found
- Keys: Ctrl+Space; Ctrl+I (mac: Ctrl+Space, Cmd+I) when editorTextFocus, the list is visible and a row is focused
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1013-1038); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (toggleDetails lines 718-732, _isDetailsVisible lines 943-950); V\src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts (close button lines 63-66, 196-203); C bundle (id toggleSuggestionDetails with the same keys)
- Pieces:
  1. With a row highlighted, pressing `Ctrl+Space` or `Ctrl+I` opens the details pane beside the list.
  2. Pressing the same keys again closes the details pane, and on macOS the keys are `Ctrl+Space` and `Cmd+I`.
  3. Clicking the small X in the corner of the details pane closes it.
  4. Whether the details pane is open or closed is remembered, so the next list opens the same way.

#### AC-1.22 Read More chevron on the focused row

- Tag: both. Evidence: source. Boundary: no. Cursor: present (icon id suggest-more-info and the label "Read More" found)
- What: When the focused item has docs, a small chevron titled "Read More" appears at the right end of the row on hover and a click opens the details pane; it stays hidden while the status bar is shown.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (icon line 26, readMore lines 129-130 and 226-243); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 76-80, 160-180, 266-300); C bundle (message 1627, icon registration)
- Pieces:
  1. When the highlighted item has docs to show, a small chevron titled "Read More" appears at the right end of that row while the pointer is over it.
  2. Clicking the "Read More" chevron opens the details pane.
  3. The "Read More" chevron stays hidden while the status bar is shown.

#### AC-1.23 Details pane contents and Markdown docs

- Tag: both. Evidence: source. Boundary: no. Cursor: present (details widget classes no-docs, no-type, detail-and-doc, markdown-docs and the "Loading..." text found)
- What: The pane shows the item's detail text (type or signature) at the top and its docs below, as plain text or Markdown with links and code, scrolls inside, and shows "Loading..." while the item is still resolving.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts (renderItem lines 130-208, renderLoading lines 122-128, canExpandCompletionItem lines 22-24); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (resolve and re-render lines 403-445); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 350-470)
- Pieces:
  1. The details pane shows the detail text of the item, such as its type or signature, at the top.
  2. The details pane shows the docs of the item below the detail text, as plain text or as Markdown with links and code.
  3. Long content scrolls inside the details pane.
  4. While the item is still being resolved, the details pane shows "Loading...".
  5. An item with no docs or no detail text leaves out the matching section of the pane.

#### AC-1.24 Details pane placement and resizing

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (east, west and south placements found; no north placement, so the pane is not put above when the list is above the cursor)
- What: The pane sits to the right of the list, or to the left or below it when there is no room (above it when the list is drawn above the cursor), can be resized from its edges, and keeps the size you drag it to until the list closes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts (SuggestDetailsOverlay lines 263-488, north placement lines 417-425, 460-463, 477-480); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (_positionDetails lines 917-922); C bundle (placeAtAnchor with three placements)
- Pieces:
  1. The details pane sits to the right of the list by default.
  2. The details pane moves to the left of the list or below it when there is no room on the right.
  3. Dragging an edge of the details pane resizes it.
  4. The details pane keeps the size you dragged it to until the list closes.
  5. Stock VS Code 1.128 puts the details pane above the list when the list is drawn above the cursor, and Cursor has no placement above. [stock only]

#### AC-1.25 Focus, scroll and copy in the details pane

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (toggleSuggestionFocus found; no suggestWidgetCopy command and no suggestWidgetDetailsFocused key, an older keydown handler lets Ctrl+C through and sends other keys back to the editor)
- What: Ctrl+Alt+Space moves focus into the pane (opening it if needed) and back to the list, the page, first and last commands scroll the pane while it has focus, and Ctrl+C copies selected text from it.
- Commands: toggleSuggestionFocus; suggestWidgetCopy
- Menus: none found
- Keys: Ctrl+Alt+Space (mac: Ctrl+Alt+Space) when editorTextFocus and the list is visible; Ctrl+C, and Ctrl+Insert on Windows, when suggestWidgetDetailsFocused
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1050-1135); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (toggleDetailsFocus lines 702-716, details focus tracker lines 298-306, page and first or last in State.Details lines 602-684); C bundle (id toggleSuggestionFocus; onDetailsKeyDown handler; zero hits for suggestWidgetCopy and suggestWidgetDetailsFocused)
- Pieces:
  1. Pressing `Ctrl+Alt+Space` while the list is open moves focus into the details pane, opening the pane first if it is closed, and pressing it again returns focus to the list.
  2. While the details pane has focus, the page commands scroll it by a page and the first and last commands scroll it to the top and to the bottom.
  3. Pressing `Ctrl+C` while the details pane has focus copies the selected text.
  4. Clicking inside the details pane moves focus into it so its text can be selected.
  5. Stock VS Code 1.128 runs the copy through the command `suggestWidgetCopy`, bound to `Ctrl+C` and, on Windows, `Ctrl+Insert` while the pane has focus, and Cursor has neither the command nor that focus condition. [stock only]
  6. In Cursor, while the details pane has focus, any key other than `Ctrl+C` (`Cmd+C` on macOS) and the modifier keys sends focus back to the editor. [cursor only]

#### AC-1.26 Explain mode

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id toggleExplainMode and key Ctrl+/ found)
- What: Ctrl+/ turns the details pane into a debug view that shows the match score, typed prefix, word, sort index, commit characters and provider name of the highlighted item.
- Commands: toggleExplainMode
- Menus: none found
- Keys: Ctrl+/ while the list is visible
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1040-1048); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (toggleExplainMode lines 761-770); V\src\vs\editor\contrib\suggest\browser\suggestWidgetDetails.ts (lines 134-146)
- Pieces:
  1. Pressing `Ctrl+/` while a row is highlighted turns the details pane into a debug view, opening the pane first if it is closed.
  2. The debug view shows the match score, the typed prefix, the word that was matched, the locality distance, the sort index with the sort text it is based on, and the commit characters of the highlighted item.
  3. The debug view shows the name of the provider that gave the highlighted item in place of the detail text.
  4. Pressing `Ctrl+/` again turns the pane back into the normal details view.
  5. With no row highlighted, `Ctrl+/` does nothing.

#### AC-1.27 Grow the list to fit inline details

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (no hit for fitWidthToDetails or the fit-width-to-details class in the bundle or CSS)
- What: newer-than-1.128. The editor option suggest.fitWidthToDetails widens the list so the longest label and its inline detail fit, up to the editor width, unless the person has dragged a size.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: N\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (applyFitWidthStyle, _measureContentWidth); N\src\vs\editor\contrib\suggest\browser\media\suggest.css; N\src\vs\editor\common\config\editorOptions.ts (lines 5044, 5183, 5454; option only, no settings key)
- Pieces:
  1. In stock VS Code main (added after 1.128.0), with the `fitWidthToDetails` suggest option on, the list opens wide enough to fit its longest label and that row's inline detail. [stock only]
  2. The widened list never grows wider than the editor. [stock only]
  3. When a size you dragged is remembered, the list keeps that size and is not widened. [stock only]

#### AC-1.28 Status bar with Insert or Replace

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (Insert, Replace and the alternate entries found; they lack the has-focused-suggestion condition that stock 1.128 puts on each)
- What: A thin bar under the rows (off by default) shows the accept action for the focused row as Insert or Replace, followed by the opposite action when the item supports both, so you can see what Tab and Shift+Tab will do.
- Commands: acceptSelectedSuggestion; acceptAlternativeSelectedSuggestion
- Menus: suggestWidgetStatusBar (group left, orders 1 and 2)
- Keys: none found
- Settings: editor.suggest.showStatusBar
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (menu entries lines 857-905); V\src\vs\editor\contrib\suggest\browser\suggestWidgetStatus.ts; V\src\vs\editor\contrib\suggest\browser\suggest.ts (menu id, line 45); V\src\vs\editor\contrib\suggest\browser\media\suggest.css (lines 38-80); C bundle (menu id suggestWidgetStatusBar, seven entries, messages 1598-1604)
- Pieces:
  1. A thin status bar under the rows shows the accept action for the highlighted row, as Insert or as Replace depending on `editor.suggest.insertMode`.
  2. When the item supports both, the status bar also shows the opposite action next to the first one.
  3. Clicking an Insert or Replace action in the status bar accepts the highlighted row in that mode.
  4. Setting `editor.suggest.showStatusBar` chooses whether the status bar is shown, with values on and off, default off.
  5. In Cursor the Insert and Replace actions in the status bar are not limited to the moments when a row is highlighted. [cursor only]

#### AC-1.29 Show More or Show Less in the status bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present (two right-group entries with Show Less and Show More found, messages 1603 and 1604)
- What: At the right of the status bar a text action reads Show More while the details pane is closed and Show Less while it is open, and a click toggles the pane; it shows only when the item has docs, a different detail or can be resolved.
- Commands: toggleSuggestionDetails
- Menus: suggestWidgetStatusBar (group right, order 1)
- Keys: Ctrl+Space; Ctrl+I (same keys as AC-1.21)
- Settings: editor.suggest.showStatusBar
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1013-1038, canResolve lines 211-212); V\src\vs\editor\contrib\suggest\browser\suggestWidgetStatus.ts (left and right action bars lines 40-80); C bundle (menu entries with CanResolve)
- Pieces:
  1. At the right of the status bar a text action reads "Show More" while the details pane is closed, and clicking it opens the pane.
  2. The same action reads "Show Less" while the details pane is open, and clicking it closes the pane.
  3. The action appears only when the highlighted item has docs, a detail text that differs from its label, or can still be resolved.
  4. The action is visible only while `editor.suggest.showStatusBar` is on.

#### AC-1.30 Select action in the status bar when no row is focused

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (the selectNextSuggestion registration in the bundle has no status bar entry and the message table has no "Select" between Replace and Show Less)
- What: When the list is open with nothing highlighted (selection mode never, or a trigger-character open), the status bar offers "Select" on the left, which highlights the next row; stock 1.128 shows Insert and Replace only once a row is focused.
- Commands: selectNextSuggestion
- Menus: suggestWidgetStatusBar (group left, order 0)
- Keys: Down; Ctrl+Down (mac adds Ctrl+N) when the list is visible
- Settings: editor.suggest.selectionMode
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 923-941); C bundle (selectNextSuggestion registration ends after kbOpts; menu entries 1598-1604 only)
- Pieces:
  1. In stock VS Code 1.128, when the list is open with no row highlighted and the status bar is shown, the status bar offers "Select" at the left. [stock only]
  2. Clicking "Select" highlights the next row. [stock only]
  3. Once a row is highlighted, "Select" gives way to the Insert and Replace actions. [stock only]
  4. When `editor.suggest.selectionMode` leaves no row highlighted, for example with the value `never`, the status bar offers "Select" instead of Insert or Replace. [stock only]

#### AC-1.31 Accept with Tab

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (id and key found; Tab is also bound to editor.action.acceptCursorTabSuggestion at weight 500 against 190 here, active while cpp.shouldAcceptTab is true, see AC-6)
- What: With a row highlighted, Tab inserts it, closes the list and runs any extra edits or follow-up command that the item has.
- Commands: acceptSelectedSuggestion
- Menus: suggestWidgetStatusBar (the Insert entry, see AC-1.28)
- Keys: Tab when suggestWidgetVisible and editorTextInputFocus (a row must be focused)
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (command lines 842-858, _insertSuggestion lines 309-510); C bundle (id acceptSelectedSuggestion with weight 190; id editor.action.acceptCursorTabSuggestion with precondition cpp.shouldAcceptTab and weight 500)
- Pieces:
  1. With a row highlighted, pressing `Tab` inserts that row and closes the list.
  2. Accepting a row also applies any extra edits it carries, such as an added import, and runs any follow-up command it names.
  3. With the list open and no row highlighted, `Tab` does not insert a row.
  4. In Cursor, `Tab` is also bound to `editor.action.acceptCursorTabSuggestion` with a higher priority, so while a Cursor Tab suggestion is waiting `Tab` accepts that suggestion and not the highlighted row. [cursor only]

#### AC-1.32 Accept with Enter (accept on enter)

- Tag: both. Evidence: source. Boundary: no. Cursor: present (command, the old alias acceptSelectedSuggestionOnEnter, and the values on, smart, off found)
- What: Enter also inserts the highlighted row when accept-on-enter is on, and with the smart value only when the row would change the text, so Enter still makes a new line after you have typed the whole word.
- Commands: acceptSelectedSuggestion
- Menus: none found
- Keys: Enter when suggestWidgetVisible, editorTextInputFocus, acceptSuggestionOnEnter and suggestionMakesTextEdit
- Settings: editor.acceptSuggestionOnEnter
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (makes-text-edit context lines 184-206, accept-on-enter key lines 288-296, alias line 909); V\src\vs\editor\common\config\editorOptions.ts (lines 5956-5968); C bundle (registration with Km.AcceptSuggestionsOnEnter and Km.MakesTextEdit)
- Pieces:
  1. With accept on enter on, pressing `Enter` with a row highlighted inserts that row and closes the list.
  2. Setting `editor.acceptSuggestionOnEnter` chooses whether `Enter` accepts a row, with values `on`, `smart` and `off`.
  3. With `smart`, `Enter` inserts the highlighted row only when doing so would change the text, so after you have typed the whole word `Enter` makes a new line.
  4. With `off`, `Enter` never accepts a row.
  5. A keybinding that uses the old command id `acceptSelectedSuggestionOnEnter` still accepts the highlighted row.

#### AC-1.33 Accept with a commit character

- Tag: both. Evidence: source. Boundary: no. Cursor: present (option id found and the commit-character check on typing found in the bundle)
- What: Typing a character that the highlighted item lists as a commit character, such as a semicolon or a dot, inserts that item first and then types the character.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.acceptSuggestionOnCommitCharacter
- Source: V\src\vs\editor\contrib\suggest\browser\suggestCommitCharacters.ts; V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 168-170); V\src\vs\editor\common\config\editorOptions.ts (lines 5952-5955); C bundle (acceptCharacters.has check inside onWillType)
- Pieces:
  1. Typing a commit character of the highlighted item, such as a semicolon or a dot, inserts that item first and then types the character.
  2. Setting `editor.acceptSuggestionOnCommitCharacter` chooses whether commit characters accept a row, with values on and off, default on.
  3. Only the commit characters of the highlighted item count, so any other character is simply typed and the list narrows.

#### AC-1.34 Accept as Insert or Replace

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, both keys and the setting found)
- What: Choose whether accepting overwrites the text to the right of the cursor (replace) or leaves it (insert) by default, and flip that for one accept with Shift+Enter or Shift+Tab when the item supports both.
- Commands: acceptAlternativeSelectedSuggestion
- Menus: suggestWidgetStatusBar (the alternate entry, see AC-1.28)
- Keys: Shift+Enter; Shift+Tab when editorTextInputFocus and a row is focused
- Settings: editor.suggest.insertMode
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (getOverwriteInfo lines 572-589, command lines 881-905, context keys lines 157-159 and 205-208); V\src\vs\editor\common\config\editorOptions.ts (lines 5141-5150); C bundle (registration with Shift+Enter and Shift+Tab)
- Pieces:
  1. Accepting a row in insert mode leaves the text to the right of the cursor in place.
  2. Accepting a row in replace mode overwrites the text to the right of the cursor that the item covers.
  3. Setting `editor.suggest.insertMode` chooses which mode a normal accept uses, with values `insert` and `replace`, default `insert`.
  4. Pressing `Shift+Enter` or `Shift+Tab` with a row highlighted accepts it in the opposite mode when the item supports both.

#### AC-1.35 Accept by clicking a row

- Tag: both. Evidence: source. Boundary: no. Cursor: present (_onListMouseDownOrTap and _onEditorMouseDown handlers found in the widget code)
- What: Clicking or tapping a row inserts it and keeps focus in the editor, and clicking inside the list never steals focus from the text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (lines 278-279, 328-371); C bundle (same method names)
- Pieces:
  1. Clicking or tapping a row inserts that row and closes the list.
  2. After a row is inserted by click, keyboard focus is in the editor.
  3. Pressing the mouse down inside the list does not take focus away from the text.

#### AC-1.36 Move through the list with the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all six ids and the key sets found)
- What: Up and Down move the highlight and wrap at the ends, PageUp and PageDown move a page, and first and last commands exist without default keys; the keys act only when the list has more than one row or nothing is highlighted yet.
- Commands: selectNextSuggestion; selectPrevSuggestion; selectNextPageSuggestion; selectPrevPageSuggestion; selectFirstSuggestion; selectLastSuggestion
- Menus: none found
- Keys: Down or Ctrl+Down (mac adds Ctrl+N); Up or Ctrl+Up (mac adds Ctrl+P); PageDown or Ctrl+PageDown; PageUp or Ctrl+PageUp; first and last have none; all when editorTextFocus and the list is visible
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 921-990); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (selectNext and related lines 602-684)
- Pieces:
  1. Pressing `Down` or `Ctrl+Down` moves the highlight to the next row, and on macOS `Ctrl+N` does the same.
  2. Pressing `Up` or `Ctrl+Up` moves the highlight to the previous row, and on macOS `Ctrl+P` does the same.
  3. The highlight wraps from the last row to the first row and from the first row to the last.
  4. Pressing `PageDown` or `Ctrl+PageDown` moves the highlight down one page, and `PageUp` or `Ctrl+PageUp` moves it up one page.
  5. Commands `selectFirstSuggestion` and `selectLastSuggestion` move the highlight to the first and last row, and they have no default keys.
  6. These keys move the highlight only when the list has more than one row or no row is highlighted yet.

#### AC-1.37 Focus the first row from the editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids found, key set matches)
- What: When the list is open with nothing highlighted, pressing the trigger key again highlights the first row, and a command exists that highlights and accepts in one step.
- Commands: focusSuggestion; focusAndAcceptSuggestion
- Menus: none found
- Keys: Ctrl+Space; Ctrl+I (mac: Ctrl+Space, Cmd+I) when the list is visible and no row is focused; focusAndAcceptSuggestion has none
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 992-1011); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (focusSelected lines 596-600)
- Pieces:
  1. With the list open and no row highlighted, pressing `Ctrl+Space` or `Ctrl+I` highlights the first row, and on macOS the keys are `Ctrl+Space` and `Cmd+I`.
  2. Command `focusAndAcceptSuggestion` highlights the first row and accepts it in one step, and it has no default key.

#### AC-1.38 Close the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id and keys found, blur handler cancels and clears)
- What: Escape or Shift+Escape closes the list and the details pane, and the list also closes when the editor loses focus, the cursor moves to another line, or nothing matches after an automatic open.
- Commands: hideSuggestWidget
- Menus: none found
- Keys: Escape; Shift+Escape when editorTextInputFocus and the list is visible
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 911-922, blur handler lines 281-286); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (cancel paths lines 346-374 and 722-735)
- Pieces:
  1. Pressing `Escape` or `Shift+Escape` while the list is open closes the list and the details pane.
  2. The list closes when the editor loses focus.
  3. The list closes when the cursor moves to another line.
  4. A list that opened by itself closes when what you type no longer matches any row.

#### AC-1.39 Tab completion without the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all three ids and the setting found)
- What: With tab completion on, Tab at the end of a word inserts the best match straight away, and Tab again or Shift+Tab then steps through the other matches; the onlySnippets value completes only snippet prefixes (see AC-4).
- Commands: insertBestCompletion; insertNextSuggestion; insertPrevSuggestion
- Menus: none found
- Keys: Tab (insertBestCompletion, and insertNextSuggestion when hasOtherSuggestions); Shift+Tab (insertPrevSuggestion); only when editor.tabCompletion is on, at the end of a word, the list is hidden and no snippet is active
- Settings: editor.tabCompletion
- Source: V\src\vs\editor\contrib\suggest\browser\suggestController.ts (lines 1062-1117); V\src\vs\editor\contrib\suggest\browser\suggestAlternatives.ts; V\src\vs\editor\contrib\suggest\browser\wordContextKey.ts; V\src\vs\editor\common\config\editorOptions.ts (lines 6708-6720)
- Pieces:
  1. With `editor.tabCompletion` on, pressing `Tab` at the end of a word with the list hidden inserts the best match straight away, with no list shown.
  2. Pressing `Tab` again steps to the next match when there are others, and `Shift+Tab` steps back to the previous one.
  3. Setting `editor.tabCompletion` chooses whether `Tab` completes a word, with values `on`, `off` and `onlySnippets`, default `off`.
  4. `Tab` completes this way only at the end of a word, while the list is hidden and no snippet is active.

#### AC-1.40 Screen reader support for the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (aria label messages 1619-1623, status messages and the additional edits alert 1597 found)
- What: The list is exposed as a listbox and the text area points at the focused row, each row is read with its label, signature, description and kind (plus detail and docs when the pane is open), "Loading..." and "No suggestions." are announced, and accepting an item with extra edits announces how many.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (accessibilityProvider lines 232-264, activeDescendant lines 391 and 445, status lines 487 and 497); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (_alertCompletionItem lines 591-595); V\src\vs\editor\browser\controller\editContext\textArea\textAreaEditContext.ts (lines 645-660); V\src\vs\editor\browser\controller\editContext\native\screenReaderSupport.ts (lines 177-185); C bundle (NLS 1597, 1617-1623)
- Pieces:
  1. The list is exposed to screen readers as a listbox, and the text area points at the highlighted row.
  2. Each row is read with its label, signature, description and kind.
  3. While the details pane is open, a row is also read with its detail text and docs.
  4. "Loading..." and "No suggestions." are announced when they appear.
  5. Accepting an item that has extra edits announces how many edits were made.

#### AC-1.41 Keep the open list in step when you delete or move along the word

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the cursor-change check that only reacts to keyboard and deleteLeft moves, and the retrigger that passes providerItemsToReuse, were found in the bundle)
- What: While the list is open, Backspace and the cursor keys re-filter it at the new cursor position - moving left onto a word asks for fresh rows, starting a new word to the right asks again and reuses the rows it already has, and the list closes when the cursor goes to another line or the line's leading whitespace changes (for example when the word is outdented).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (_onCursorChange lines 345-376, _onNewContext lines 717-841); C bundle (source check for keyboard and deleteLeft, retrigger with providerItemsToReuse)
- Pieces:
  1. Pressing `Backspace` while the list is open re-filters the list for the new cursor position.
  2. Moving the cursor left with the arrow keys onto another word refreshes the list with rows for that word.
  3. Starting a new word to the right of the cursor shows rows for the new word and reuses the rows the list already has.
  4. Moving the cursor with the mouse, or making a selection, closes the list instead of re-filtering it.
  5. The list closes when the leading whitespace of the line changes, for example when the word is outdented.

#### AC-1.42 Color the kind icons by kind

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all 33 symbolIcon color ids and the rules that color .codicon-symbol-* icons inside the editor were found; the workbench-side rule differs but the list sits inside the editor)
- What: Each kind icon in the list takes its own color from a per-kind theme color (method, function, class, variable, keyword, snippet and others; 33 colors in all, shared with the Outline and breadcrumbs), and a theme or the person can override each one; a color item shows its swatch instead (see AC-1.7).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Source: V\src\vs\editor\contrib\symbolIcons\browser\symbolIcons.ts (33 registerColor calls, lines 10-124); V\src\vs\editor\contrib\symbolIcons\browser\symbolIcons.css (lines 8-75); V\src\vs\editor\contrib\suggest\browser\suggestWidgetRenderer.ts (kind icon class, line 202); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (import that loads the colors, line 23); C bundle (symbolIcon.*Foreground ids; CSS rule .monaco-editor .codicon.codicon-symbol-method colored by the vscode-symbolIcon-methodForeground variable)
- Pieces:
  1. Each kind icon takes its own color from a per-kind theme color, such as `symbolIcon.methodForeground`, `symbolIcon.functionForeground` and `symbolIcon.classForeground`.
  2. There are 33 kind colors, and the same colors are used for the matching icons in the Outline and in the breadcrumbs.
  3. Setting `workbench.colorCustomizations` lets a person override each kind color.

#### AC-1.43 Highlight the row that matches the ghost text

- Tag: both. Evidence: source. Boundary: no. Cursor: present (a selector registered at priority 100 that reads the selected inline completion as a single text edit was found in the bundle; whether any provider feeds ghost text to it in Cursor was not checked)
- What: When an inline suggestion (ghost text) is on screen and the list opens, the row whose text is the start of the ghost text is highlighted first, and the longest such row wins, ahead of the remembered choice from AC-1.15, so the list and the ghost text line up.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts (selector lines 56-89); V\src\vs\editor\contrib\inlineCompletions\browser\model\singleTextEditHelpers.ts (singleTextEditAugments lines 26-29); V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (preselector source, lines 138-142); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (registerSelector line 764, default selector lines 150-153); C bundle (registerSelector with priority:100; selectedInlineCompletion toSingleTextEdit)
- Pieces:
  1. When ghost text from an inline suggestion is on screen and the list opens, the row whose text is the start of the ghost text is highlighted first.
  2. When several rows start the ghost text, the longest one is highlighted.
  3. The row that matches the ghost text wins over the remembered choice from `editor.suggestSelection`.
  4. When no row starts the ghost text, the usual highlight choice applies.

#### AC-1.44 Hold Shift to preview the other Insert or Replace result

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the key down and key up handlers that set isShiftKeyPressed and refresh the preview were found in the same adapter class)
- What: With the list preview on, holding Shift redraws the ghost text of the highlighted row as it would look if accepted with the opposite insert or replace mode, which is what Shift+Enter or Shift+Tab will then do; letting go of Shift draws the normal preview again.
- Commands: none found
- Menus: none found
- Keys: Shift held down (a key listener, not a keybinding rule; the accept keys themselves are in AC-1.34)
- Settings: editor.suggest.preview; editor.suggest.insertMode
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts (key listeners lines 42-54, fromSuggestion with toggleMode lines 181-196); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (getOverwriteInfo with toggleMode, lines 573-589); V\src\vs\editor\common\config\editorOptions.ts (preview default false, line 5105); C bundle (onKeyDown and onKeyUp handlers with isShiftKeyPressed)
- Pieces:
  1. With `editor.suggest.preview` on, holding `Shift` redraws the ghost text of the highlighted row as it would look if accepted in the opposite insert or replace mode.
  2. Letting go of `Shift` draws the normal preview again.
  3. Pressing `Shift+Enter` or `Shift+Tab` while the opposite-mode preview is showing inserts that previewed result.
  4. With `editor.suggest.preview` off, holding `Shift` draws no ghost text, and `editor.suggest.insertMode` decides which mode counts as the normal one.

#### AC-1.45 Move the list above the cursor while the preview spans several lines

- Tag: both. Evidence: source. Boundary: no. Cursor: present (forceRenderingAbove and stopForceRenderingAbove on the list, the 150 px check and the primaryGhostText line count of 2 or more were found in the bundle)
- What: When the previewed text of the highlighted row shows as ghost text on two or more lines, the list is drawn above the cursor, as long as there are at least 150 px of room there, so it does not cover the ghost lines; it goes back to the normal placement once the preview is gone.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.preview
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (lines 302-314); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (state with suggestItem, lines 659-679); V\src\vs\editor\contrib\suggest\browser\suggestWidget.ts (forceRenderingAbove lines 879-880 and 951-960); C bundle (forceRenderingAbove with lineCount>=2 and g>150)
- Pieces:
  1. With `editor.suggest.preview` on, when the previewed text of the highlighted row shows as ghost text on two or more lines, the list is drawn above the cursor so it does not cover the ghost lines.
  2. The list moves above the cursor only when there are at least 150 pixels of room above it.
  3. The list returns to its normal placement once the preview is gone.

#### AC-1.46 Path and header suggestions in Markdown links

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both markdown.suggest settings, their labels and the [markdown] quick suggestions default are in the Cursor copy of the extension manifest)
- What: In a Markdown file, typing inside a link target lists files, folders and headers as rows in the suggest list; one setting turns the path rows off, another chooses whether headers from other workspace files appear (never, after ##, or after # or ##), and Markdown files also default quick suggestions to off for other code, comments and strings.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: markdown.suggest.paths.enabled; markdown.suggest.paths.includeWorkspaceHeaderCompletions; editor.quickSuggestions
- Source: V\extensions\markdown-language-features\package.json (settings lines 418-439, [markdown] configurationDefaults lines 833-842); V\extensions\markdown-language-features\package.nls.json (configuration.markdown.suggest.paths.*); C\extensions\markdown-language-features\package.json and package.nls.json
- Pieces:
  1. In a Markdown file, typing inside a link target lists files and folders as rows in the suggest list.
  2. Header rows also appear in the same list for link targets.
  3. Setting `markdown.suggest.paths.enabled` chooses whether path rows are offered, with values on and off, default on.
  4. Setting `markdown.suggest.paths.includeWorkspaceHeaderCompletions` chooses when headers from other workspace files appear, with values `never`, `onDoubleHash` (after `##`) and `onSingleOrDoubleHash` (after `#` or `##`), default `onDoubleHash`.
  5. Markdown files default `editor.quickSuggestions` to `off` for other code, comments and strings, so the list does not open on its own while typing in Markdown.


## AC-2 Ghost text and inline suggestions

(both). Gray inline text with a hover toolbar, accept word and line, next and previous, plus the stock inline edit gutter UI. Examples: ghost text; the toolbar (Accept, Accept Word, Accept Line, n of m); Alt+] and Alt+[; the inline edit gutter menu.

48 features.

#### AC-2.1 See ghost text at the cursor

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Gray italic text appears right after the cursor as the proposed completion, drawn as injected text inside the line with the syntax colors dimmed to 70 percent opacity, and the cursor stays in front of it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css; V\src\vs\editor\contrib\inlineCompletions\browser\model\computeGhostText.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts; C bundle (decoration description `ghost-text-decoration`, classes `ghost-text-decoration`, `ghost-text-decoration-preview`, `syntax-highlighted`)
- Pieces:
  1. While a suggestion is available, its text shows in the line right after the cursor as italic ghost text, and the cursor stays in front of it.
  2. The code to the right of the cursor moves over to make room for the ghost text instead of being covered by it.
  3. With syntax colors on, ghost text keeps the syntax colors of the language at 70 percent opacity, so it looks dimmer than real code.

#### AC-2.2 See multi-line ghost text under the line

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a suggestion has more than one line, the extra lines are drawn in a view zone under the cursor line in the editor font, the rest of the original line to the right of the insert point is hidden while it shows, and the suggest list is moved above the cursor when the ghost text has two or more lines.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts (AdditionalLinesWidget, class `suggest-preview-text`, decoration `ghost-text-hidden`); V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (forceRenderingAbove); C bundle (`suggest-preview-text`, `ghost-text-hidden`, `forceRenderingAbove`)
- Pieces:
  1. When a suggestion has more than one line, the first line shows after the cursor and the extra lines show in a block under the cursor line in the editor font.
  2. While multi-line ghost text shows, the rest of the original line to the right of the insert point is hidden so it does not collide with the ghost text.
  3. When the ghost text has two or more lines, the suggest list opens above the cursor line instead of below it.
  4. The extra lines use the same italic ghost look as the first line and follow the syntax color setting.

#### AC-2.3 See the text a suggestion will replace or delete

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: For an inline edit shown side by side, as a line replacement, as a deletion, or as an inline insertion, the original text is marked in the editor with line and character highlights for removed text and inserted text. Plain ghost text can also underline the range it replaces (`inline-completion-text-to-replace`), but no code in V constructs that replace type (searched V\src for `new GhostTextReplacement`, no match), so the underline may never show in 1.128.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\originalEditorInlineDiffView.ts (classes `inlineCompletions-line-insert`, `-line-delete`, `-char-insert`, `-char-delete`, `-original-lines`); V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\model\ghostText.ts (GhostTextReplacement); C bundle (`inlineCompletions-char-insert`, `inlineCompletions-char-delete`, `inlineCompletions-line-insert`, `inlineCompletions-line-delete`, `inline-completion-text-to-replace`; `inlineCompletions-original-lines` not found)
- Pieces:
  1. For an inline edit shown side by side or as a line replacement, the original lines that change get a line highlight and the exact characters that go get a stronger character highlight.
  2. For an inline edit shown as a deletion, the text that will be removed is marked in place in the original lines.
  3. For an inline edit shown as an inline insertion, the inserted characters get an insert highlight where they will land.
  4. Ghost text that would replace existing text underlines the range it replaces (class `inline-completion-text-to-replace`), though stock 1.128.0 may never produce this case.

#### AC-2.4 See very short ghost text with a dotted underline

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: Ghost text with fewer than 3 non-space characters, or an inserted part under 5 characters, gets a dotted underline in the ghost text color and skips the syntax-color treatment, so tiny suggestions stay visible.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts (class `short-text`, `highlightShortSuggestions: true` in inlineSuggestionsView.ts); V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts; C bundle (`short-text` searched in JS and CSS, 0 hits)
- Pieces:
  1. Ghost text with fewer than 3 non-space characters, or an inserted part under 5 characters, gets a dotted underline in the ghost text color. [stock only]
  2. Very short ghost text skips the dimmed syntax colors and is drawn in the plain ghost text color so it stays visible. [stock only]
  3. Ghost text that is longer than those limits has no dotted underline. [stock only]

#### AC-2.5 See a warning on a suggestion

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a suggestion carries a warning, the ghost text gets a warning squiggle with a double underline in the editor warning color, and the toolbar shows the warning message (plain text or markdown, up to 400 px wide) above its buttons; hovering the ghost text opens it. An orange warning icon widget at the end of the line also exists in code, but a constant (`USE_SQUIGGLES_FOR_WARNING`) switches it off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css; V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\inlineCompletionsHintsWidget.ts (class `warningMessage`); V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\hoverParticipant.ts (getWarningWidgetContext); C bundle (`warningMessage`, `ghost-text-view-warning-widget`, CSS `.ghost-text-decoration.warning`)
- Pieces:
  1. A suggestion that carries a warning draws its ghost text with a warning squiggle and a double underline in the editor warning border color.
  2. The toolbar for a suggestion with a warning shows the warning message above its buttons, as plain text or rendered markdown, at most 400 px wide.
  3. Hovering the ghost text of a warned suggestion opens the hover with the warning message.

#### AC-2.6 Style the ghost text

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Choose whether ghost text keeps syntax colors, pick a font family just for ghost text, and set its foreground, background, and border with the theme colors `editorGhostText.foreground`, `editorGhostText.background`, and `editorGhostText.border`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.syntaxHighlightingEnabled; editor.inlineSuggest.fontFamily
- Source: V\src\vs\editor\common\config\editorOptions.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts (font family style sheet); V\src\vs\editor\common\core\editorColorRegistry.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css; C bundle (both setting keys, the three `editorGhostText.*` color ids, NLS "Controls whether to show syntax highlighting for inline suggestions in the editor." and "Controls the font family of the inline suggestions.")
- Pieces:
  1. Setting `editor.inlineSuggest.syntaxHighlightingEnabled` chooses whether ghost text keeps the syntax colors of the language, on or off, default on.
  2. With syntax colors off, ghost text is drawn in the single color of the theme color `editorGhostText.foreground`.
  3. Setting `editor.inlineSuggest.fontFamily` sets a font family used only for ghost text, and the default value `default` keeps the editor font.
  4. Theme colors `editorGhostText.background` and `editorGhostText.border` give ghost text a background and a one pixel border.

#### AC-2.7 Preview the highlighted suggest list item as ghost text

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With `editor.suggest.preview` on (default off), the item highlighted in the suggest list also shows in the editor as ghost text, with the part beyond the typed prefix in the preview style, and the suggest list moves above the cursor when the preview is multi-line. The owner of the suggest list itself is AC-1, the rendering is the ghost text view here.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.preview
- Source: V\src\vs\editor\common\config\editorOptions.ts (suggest.preview, previewMode default subwordSmart, no settings key for previewMode); V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts; V\src\vs\editor\contrib\inlineCompletions\browser\model\computeGhostText.ts; V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts; C bundle (`editor.suggest.preview`, `forceRenderingAbove`, NLS "Controls whether to preview the suggestion outcome in the editor.")
- Pieces:
  1. With `editor.suggest.preview` on, the item highlighted in the suggest list also shows in the editor as ghost text at the cursor.
  2. Moving the highlight to another list item changes the ghost text to match that item.
  3. The part of the preview beyond what you have already typed is drawn in the preview style, and the typed prefix is not repeated.
  4. When the previewed item has more than one line, the suggest list moves above the cursor.
  5. Setting `editor.suggest.preview` turns the preview on or off, default off.

#### AC-2.8 Use the inline suggestion toolbar

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A small toolbar offers Accept and Accept Word as buttons, with Accept Line and an Always Show Toolbar toggle in its overflow menu, each showing its key label. With the default `onHover` it opens in the editor hover over ghost text, with `always` it is pinned above or below the ghost text line, with `never` it is off. Provider actions can join the overflow menu through `InlineCompletionsActions` (proposed API, extension fed, not cataloged here).
- Commands: editor.action.inlineSuggest.commit; editor.action.inlineSuggest.acceptNextWord; editor.action.inlineSuggest.acceptNextLine; editor.action.inlineSuggest.toggleAlwaysShowToolbar
- Menus: InlineSuggestionToolbar; InlineCompletionsActions (contributes.menus key `editor/inlineCompletions/actions`, marked for deprecation in code)
- Keys: none found
- Settings: editor.inlineSuggest.showToolbar
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\inlineCompletionsHintsWidget.ts (root class `inlineSuggestionsHints`); V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\inlineCompletionsHintsWidget.css; V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\hoverParticipant.ts; V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts; V\src\vs\platform\actions\common\actions.ts (MenuId); V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts; C bundle (class `inlineSuggestionsHints`, menu `InlineSuggestionToolbar`, labels "Accept", "Accept Word", "Accept Line", "Always Show Toolbar", setting `editor.inlineSuggest.showToolbar`)
- Pieces:
  1. Hovering ghost text opens the editor hover with a small toolbar that has an Accept button and an Accept Word button, each showing its key label.
  2. The toolbar overflow menu holds Accept Line and an Always Show Toolbar toggle, each showing its key label.
  3. Clicking Accept, Accept Word, or Accept Line takes the whole suggestion, the next word, or the next line (`editor.action.inlineSuggest.commit`, `editor.action.inlineSuggest.acceptNextWord`, `editor.action.inlineSuggest.acceptNextLine`).
  4. Setting `editor.inlineSuggest.showToolbar` chooses when the toolbar shows, with values `onHover` (default), `always`, and `never`.
  5. With `always`, the toolbar is pinned above or below the line that holds the ghost text instead of waiting for a hover.
  6. Choosing Always Show Toolbar in the overflow menu flips `editor.inlineSuggest.showToolbar` between `always` and `onHover`, and the menu item shows a check while it is `always` (`editor.action.inlineSuggest.toggleAlwaysShowToolbar`).

#### AC-2.9 Step through suggestions with n of m and the arrow keys

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When more than one suggestion is available, the toolbar starts with a previous chevron, an `n/m` count, and a next chevron (buttons are disabled when there is only one), and Alt+] and Alt+[ move to the next and previous suggestion. The three toolbar items are added in code, not as menu entries.
- Commands: editor.action.inlineSuggest.showNext; editor.action.inlineSuggest.showPrevious
- Menus: none found
- Keys: Alt+] (show next, when editor writable and inlineSuggestionVisible); Alt+[ (show previous, same when)
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\inlineCompletionsHintsWidget.ts (icons `inline-suggestion-hints-previous` and `-next`, class `availableSuggestionCount`); V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts; C bundle (ids `editor.action.inlineSuggest.showNext` and `.showPrevious` with Alt+] and Alt+[, icons `inline-suggestion-hints-next` and `-previous`, class `availableSuggestionCount`)
- Pieces:
  1. When more than one suggestion is available, the toolbar starts with a previous chevron, an `n/m` count, and a next chevron.
  2. Clicking the next or previous chevron shows the next or previous suggestion and the count updates.
  3. When there is only one suggestion, the previous and next chevrons are disabled.
  4. Pressing `Alt+]` shows the next suggestion and `Alt+[` shows the previous one while the editor is writable and a suggestion is visible (`editor.action.inlineSuggest.showNext`, `editor.action.inlineSuggest.showPrevious`).

#### AC-2.10 Accept the suggestion with Tab

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the Tab rule has no selection check; Cursor Tab adds its own Tab accept)
- What: Tab takes the whole suggestion, unless the suggest list is open, Tab is set to move focus, a hover has focus, text is selected, or the suggestion starts with more whitespace than one Tab would add (then Tab indents as usual). The same accept also runs from the Accept button in the toolbar and from the gutter menu, and it takes an inline edit when Tab is set to accept rather than jump. Cursor has its own Tab accept for Cursor Tab, handed off to AC-6.
- Commands: editor.action.inlineSuggest.commit
- Menus: InlineSuggestionToolbar; InlineEditsActions
- Keys: Tab (when inlineSuggestionVisible, not editorTabMovesFocus, not suggestWidgetVisible, not editorHoverFocused, no selection, inlineSuggestionHasIndentationLessThanTabSize; or inlineEditIsVisible with tabShouldAcceptInlineEdit and the same other guards); Tab also inside the inline edits preview editor
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (AcceptInlineCompletion); V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionContextKeys.ts; C bundle (id `editor.action.inlineSuggest.commit`, label "Accept Inline Suggestion", Tab rule at weight 200, menus `InlineSuggestionToolbar` and `InlineEditsActions`)
- Pieces:
  1. Pressing `Tab` while ghost text shows inserts the whole suggestion and puts the cursor at the end of it (`editor.action.inlineSuggest.commit`).
  2. `Tab` does not accept, and does its usual job, while the suggest list is open, while Tab is set to move focus, or while a hover has focus.
  3. With text selected, `Tab` does not accept the suggestion and indents the selection as usual. [stock only]
  4. When the suggestion starts with more whitespace than one Tab would add, `Tab` indents as usual instead of accepting.
  5. Clicking Accept in the toolbar, or choosing the accept row in the gutter menu, takes the whole suggestion the same way.
  6. When an inline edit shows and Tab is set to accept rather than jump, `Tab` accepts the edit, and it also accepts inside the inline edits preview editor.
  7. In Cursor, `Tab` accepts the suggestion even while text is selected, and Cursor Tab adds its own Tab accept (see AC-6). [cursor only]

#### AC-2.11 Accept the next word with Ctrl+Right

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the key rule also fires for a Cursor Tab suggestion and first tries the Cursor Tab partial accept, with no cursorBeforeGhostText or screen reader guard)
- What: Ctrl+Right (Cmd+Right on macOS) takes only the next word of the ghost text when the cursor sits right in front of it and screen reader mode is off. The Accept Word button in the toolbar does the same. A suggestion with a snippet is accepted whole instead.
- Commands: editor.action.inlineSuggest.acceptNextWord
- Menus: InlineSuggestionToolbar
- Keys: Ctrl+Right (Cmd+Right on macOS) when editor writable, inlineSuggestionVisible, cursorBeforeGhostText, and accessibility mode off
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (AcceptNextWordOfInlineCompletion); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (acceptNextWord, _acceptNext); C bundle (id `editor.action.inlineSuggest.acceptNextWord`, label "Accept Next Word Of Inline Suggestion", toolbar title "Accept Word")
- Pieces:
  1. Pressing `Ctrl+Right` (`Cmd+Right` on macOS) with the cursor right in front of ghost text takes only the next word of it (`editor.action.inlineSuggest.acceptNextWord`).
  2. The key keeps its normal word-move job when the cursor is not right in front of the ghost text or when screen reader mode is on. [stock only]
  3. Clicking Accept Word in the toolbar does the same as the key.
  4. A suggestion that carries a snippet is accepted whole instead of one word.
  5. In Cursor, the same key also fires for a Cursor Tab suggestion and first tries the Cursor Tab partial accept, with no cursor-position or screen reader guard (see AC-6). [cursor only]

#### AC-2.12 Accept the next line

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Takes the ghost text up to and including the next line break. It has no default key and is reached from the toolbar overflow menu (Accept Line) or the command palette.
- Commands: editor.action.inlineSuggest.acceptNextLine
- Menus: InlineSuggestionToolbar; CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (AcceptNextLineOfInlineCompletion); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (acceptNextLine); C bundle (id `editor.action.inlineSuggest.acceptNextLine`, label "Accept Next Line Of Inline Suggestion", toolbar title "Accept Line", secondary group)
- Pieces:
  1. Accept Next Line takes the ghost text up to and including the next line break (`editor.action.inlineSuggest.acceptNextLine`).
  2. Accept Line is in the toolbar overflow menu and has no default key.
  3. Running Accept Next Line Of Inline Suggestion from the Command Palette does the same.

#### AC-2.13 Reject a suggestion with Esc

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the Esc rule has no editorTextFocus guard; Esc also rejects a Cursor Tab suggestion, handed off to AC-6)
- What: Esc hides the ghost text or inline edit and keeps focus in the editor. Reject also shows in the gutter menu. The registered `InlineEditsActions` entries (Accept, Jump, Reject) have no code in V that draws them as a menu, so only the gutter menu shows Reject.
- Commands: editor.action.inlineSuggest.hide
- Menus: InlineEditsActions
- Keys: Escape (when editorTextFocus, same weight as closing the suggest list); Escape or Shift+Escape inside the inline edits preview editor
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (HideInlineCompletion); C bundle (id `editor.action.inlineSuggest.hide`, label "Hide Inline Suggestion", Esc at weight 190, menu `InlineEditsActions` title "Reject")
- Pieces:
  1. Pressing `Escape` while ghost text or an inline edit shows hides it and keeps focus in the editor (`editor.action.inlineSuggest.hide`).
  2. Inside the inline edits preview editor, `Escape` or `Shift+Escape` hides the edit.
  3. The Reject row in the gutter indicator menu hides the suggestion the same way.
  4. In Cursor, `Escape` rejects a suggestion without needing editor text focus, and also rejects a Cursor Tab suggestion (see AC-6). [cursor only]

#### AC-2.14 Trigger an inline suggestion by hand

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (command and label "Trigger Inline Suggestion" present; the bundle shows no argument schema and no no-result notice)
- What: The command asks for a suggestion at the cursor from the palette or a key you bind. Stock takes optional arguments (showNoResultNotification, providerId, explicit) and can show the notice "No inline suggestion is available." when nothing comes back; it also plays the suggestion sound cue.
- Commands: editor.action.inlineSuggest.trigger
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (TriggerInlineSuggestionAction); C bundle (id `editor.action.inlineSuggest.trigger`; NLS "No inline suggestion is available." searched, 0 hits)
- Pieces:
  1. Running Trigger Inline Suggestion from the Command Palette, or a key you bind to `editor.action.inlineSuggest.trigger`, asks for a suggestion at the cursor in a writable editor.
  2. When the command is run with `showNoResultNotification` set and no suggestion comes back, an info notice "No inline suggestion is available." shows. [stock only]
  3. After the command runs, the inline suggestion sound cue plays for the suggestion that shows.

#### AC-2.15 Trigger a next edit suggestion by hand

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The palette command "Trigger Next Edit Suggestion" asks only for an inline edit (not ghost text) and shows the notice "No inline edit is available." when none comes back. A second id, `editor.action.inlineSuggest.triggerInlineEdit`, has no label. Neither id is in V 1.128.0 or N, so these look like older stock code left in the bundle; origin not verified.
- Commands: editor.action.inlineSuggest.triggerInlineEditExplicit; editor.action.inlineSuggest.triggerInlineEdit
- Menus: CommandPalette (only the labeled `...Explicit` command)
- Keys: none found
- Settings: none found
- Source: C bundle (ids `editor.action.inlineSuggest.triggerInlineEditExplicit` and `.triggerInlineEdit`, label "Trigger Next Edit Suggestion", notice "No inline edit is available.")
- Pieces:
  1. Running Trigger Next Edit Suggestion from the Command Palette asks only for an inline edit, never for ghost text (`editor.action.inlineSuggest.triggerInlineEditExplicit`). [cursor only]
  2. When no inline edit comes back, an info notice "No inline edit is available." shows. [cursor only]
  3. A second command `editor.action.inlineSuggest.triggerInlineEdit` triggers an inline edit but has no label and no Command Palette entry. [cursor only]

#### AC-2.16 Accept the next part of an inline edit

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The palette command "Accept Next Inline Edit Part" takes the first piece of a multi-part inline edit. It has no default key. Not in V 1.128.0 or N, so it looks like older stock code left in the bundle; origin not verified.
- Commands: editor.action.inlineSuggest.acceptNextInlineEditPart
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (id `editor.action.inlineSuggest.acceptNextInlineEditPart`, label "Accept Next Inline Edit Part", precondition editor writable and inlineEditIsVisible)
- Pieces:
  1. Running Accept Next Inline Edit Part from the Command Palette takes the first piece of a multi-part inline edit (`editor.action.inlineSuggest.acceptNextInlineEditPart`). [cursor only]
  2. The command is enabled only while the editor is writable and an inline edit is visible, and it has no default key. [cursor only]

#### AC-2.17 See the inline edit gutter indicator

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (older gutter code: click accepts when the pill is docked, otherwise jumps; the new-user storage key `inlineEditsGutterIndicatorUserKind` is present)
- What: A small pill in the left gutter marks an inline edit (or a ghost text that offers the edit menu). Beside the edit it shows a Tab icon when Tab will accept, a Tab-above or Tab-below icon when Tab will jump up or down to the edit, and a check when hovered. When the edit is outside the viewport the pill slides into view and shows an arrow that points up or down. It carries the line number, accepts or jumps on click, and pulses a few times for new users and when you hover the edit.
- Commands: editor.action.inlineSuggest.commit; editor.action.inlineSuggest.jump
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\components\gutterIndicatorView.ts (class `inline-edits-view-gutter-indicator`, icons keyboard-tab, arrow-right, check, keyboard-tab-above, keyboard-tab-below); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsNewUsers.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts; C bundle (class `inline-edits-view-gutter-indicator`, storage key `inlineEditsGutterIndicatorUserKind`, `triggerAnimation`)
- Pieces:
  1. A small pill in the left gutter marks an inline edit, or a ghost text that offers the edit menu.
  2. The pill shows a Tab icon beside the edit when Tab will accept it.
  3. The pill shows a Tab-above or Tab-below icon when Tab will jump up or down to an edit that is elsewhere in the file.
  4. The pill shows a check icon while you hover it.
  5. When the edit is outside the visible part of the file, the pill slides into view at the edge and shows an arrow that points up or down.
  6. The pill shows the line number of the edit.
  7. Clicking the pill accepts the edit or jumps to it (`editor.action.inlineSuggest.commit`, `editor.action.inlineSuggest.jump`).
  8. The pill pulses a few times for new users, and it pulses when you hover the edit.
  9. In Cursor, clicking the pill accepts the edit when the pill is docked beside it and jumps to the edit otherwise. [cursor only]

#### AC-2.18 Use the gutter indicator menu

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (no Snooze row and no alternative action row; the Go To / Accept row switches between accept and jump with the Tab action; Reject, Show Expanded, Show Collapsed, and Settings are present)
- What: Hovering the pill opens a small menu with a header (the provider name), Go To / Accept (check icon), Reject (close icon), an optional alternative action, Show Collapsed or Show Expanded, Snooze (bell-slash icon), and Settings (gear icon, opens Settings filtered to the tag `@tag:nextEditSuggestions`). Provider commands and a footer action bar follow. Each row shows its key label.
- Commands: editor.action.inlineSuggest.commit; editor.action.inlineSuggest.hide; editor.action.inlineSuggest.commitAlternativeAction; editor.action.inlineSuggest.toggleShowCollapsed; editor.action.inlineSuggest.snooze; workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.showCollapsed
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\components\gutterIndicatorMenu.ts; C bundle (row ids `gotoAndAccept`, `reject`, `showExpanded`, `showCollapsed`, `settings`; NLS "Go To", "Accept", "Reject", "Show Expanded", "Show Collapsed", "Settings"; `inline-edit-gutter-indicator-menu-separator`)
- Pieces:
  1. Hovering the gutter pill opens a small menu whose header shows the name of the provider that made the suggestion.
  2. The first row reads Go To / Accept with a check icon and does what Tab would do, jump or accept.
  3. A Reject row with a close icon hides the suggestion (`editor.action.inlineSuggest.hide`).
  4. A row reading Show Collapsed or Show Expanded flips whether edits show collapsed (`editor.action.inlineSuggest.toggleShowCollapsed`).
  5. A Snooze row with a bell-slash icon opens the snooze duration choice (`editor.action.inlineSuggest.snooze`). [stock only]
  6. When the edit has an alternative action, an extra row runs it (`editor.action.inlineSuggest.commitAlternativeAction`). [stock only]
  7. A Settings row with a gear icon opens Settings filtered to the tag `@tag:nextEditSuggestions` (`workbench.action.openSettings`).
  8. Each row shows its key label, and extra commands and a footer action bar follow the rows.

#### AC-2.19 Jump to the next inline edit with Tab

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When an inline edit is somewhere else in the file, Tab moves the cursor to it first (the gutter pill shows a Tab-above or Tab-below icon) and a second Tab accepts it. The registered menu entry is titled Jump and is hidden when the cursor is already at the edit, but nothing in V draws that menu (see Unclear).
- Commands: editor.action.inlineSuggest.jump
- Menus: InlineEditsActions
- Keys: Tab (when inlineEditIsVisible, not editorTabMovesFocus, not suggestWidgetVisible, not editorHoverFocused, tabShouldJumpToInlineEdit)
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (JumpToNextInlineEdit); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (jump); C bundle (id `editor.action.inlineSuggest.jump`, label "Jump to next inline edit", Tab at weight 201)
- Pieces:
  1. When an inline edit is elsewhere in the file, pressing `Tab` moves the cursor to the edit instead of accepting it (`editor.action.inlineSuggest.jump`).
  2. After the jump, pressing `Tab` a second time accepts the edit.
  3. `Tab` jumps only while an inline edit is visible, Tab is not set to move focus, the suggest list is closed, no hover has focus, and Tab is set to jump.
  4. The command Jump to next inline edit moves the cursor to the edit the same way as the key.

#### AC-2.20 Accept the alternative action (Rename) with Shift+Tab

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: An inline edit can carry a second action, drawn as a small labeled segment next to the replacement (for example Rename, with a count of occurrences when you hold Shift: "Rename 3 occurrences"). Shift+Tab runs it, and the gutter menu lists it as a row.
- Commands: editor.action.inlineSuggest.commitAlternativeAction; editor.action.inlineSuggest.renameSymbol
- Menus: none found
- Keys: Shift+Tab (when inlineSuggestionAlternativeActionVisible and inlineEditIsVisible); Shift+Tab also inside the inline edits preview editor
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (AcceptInlineCompletionAlternativeAction); V\src\vs\editor\contrib\inlineCompletions\browser\controller\commandIds.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsWordReplacementView.ts (class `inline-edit-alternative-action-label`); V\src\vs\editor\contrib\inlineCompletions\browser\model\renameSymbolProcessor.ts (label "Rename"); C bundle (ids, class, and label "Accept Inline Suggestion Alternative Action" searched, 0 hits)
- Pieces:
  1. An inline edit can show a second action as a small labeled segment next to the replacement, for example Rename. [stock only]
  2. Holding `Shift` adds a count to the label, for example "Rename 3 occurrences". [stock only]
  3. Pressing `Shift+Tab` while such an edit shows runs the second action, and it also works inside the inline edits preview editor (`editor.action.inlineSuggest.commitAlternativeAction`). [stock only]
  4. The gutter menu lists the second action as its own row. [stock only]

#### AC-2.21 See a larger inline edit side by side

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A change that spans lines shows the new code in a bordered, clickable preview box next to the original lines when the old and new text both fit across the editor width, and the border color follows what Tab will do (accept, jump, or nothing). Clicking the box accepts the edit once it has been on screen for 350 ms. Set `renderSideBySide` to `never` to always use the line replacement view (AC-2.22) instead.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.renderSideBySide
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsSideBySideView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (_determineView); C bundle (`sideBySide` view kind, class `originalCornerCutoutSideBySide`, setting `editor.inlineSuggest.edits.renderSideBySide`, NLS "Larger suggestions will show side by side...")
- Pieces:
  1. A change that spans lines shows its new code in a bordered preview box next to the original lines when the old and new text both fit across the editor width.
  2. The border color of the box follows what Tab will do: accept, jump, or nothing.
  3. Clicking the box accepts the edit once the box has been on screen for 350 ms.
  4. Setting `editor.inlineSuggest.edits.renderSideBySide` chooses `auto` (default, side by side when there is room, below otherwise) or `never` (always the line replacement view of AC-2.22).
  5. Resizing the editor makes the choice between side by side and below be made again for the edit that is showing.

#### AC-2.22 See an inline edit as a line replacement

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A one-line change, a larger change when side by side is off or does not fit, and a one-line insertion before the cursor all show as a bordered box under the original lines with the new lines inside and the old lines highlighted. Clicking accepts it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.renderSideBySide
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsLineReplacementView.ts (class `line-replacement`); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (_determineView); C bundle (`lineReplacement` view kind, class `modifiedLinesLineReplacement`)
- Pieces:
  1. A larger change shows as a bordered box under the original lines with the new lines inside and the old lines highlighted, when side by side is off or there is not enough room.
  2. A one-line insertion that lands before the cursor shows in this box too, so the cursor does not move.
  3. A one-line change that is too big for the small word box (AC-2.23) shows in this box.
  4. Clicking the box accepts the edit.
  5. Setting `editor.inlineSuggest.edits.renderSideBySide` set to `never` makes every larger change use this view.

#### AC-2.23 See an inline edit as a word replacement

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (word replacement view present; no alternative action segment)
- What: A small change inside one line (each side under 100 characters, no tabs) shows as a small box just under the changed word with the new text, while the old word stays highlighted in the line. Clicking accepts it. When the edit has an alternative action it adds the second segment described in AC-2.20.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsWordReplacementView.ts (class `word-replacement`, MAX_LENGTH 100); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts; C bundle (class `word-replacement`, view kind `wordReplacements`)
- Pieces:
  1. A small change inside one line, with each side under 100 characters and no tabs, shows as a small box just under the changed word with the new text.
  2. The old word stays highlighted in the line while the small box shows.
  3. Clicking the small box accepts the edit.
  4. A change that involves a tab, or has either side at 100 characters or longer, does not use the small box and shows in a larger view instead.
  5. When the edit has an alternative action, the box gets the second labeled segment described in AC-2.20. [stock only]

#### AC-2.24 See an inline edit inserted in place

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: An edit that only adds text is drawn where it will land. A single-line insertion after the cursor shows inline and shifts the code to make room. A multi-line insertion shows as a bordered block that pushes the lines below down. `allowCodeShifting` set to `always` (default) allows both, `horizontal` allows only the inline shift, and `never` uses a line replacement or side by side view instead.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.allowCodeShifting
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsInsertionView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\originalEditorInlineDiffView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (_determineView); C bundle (view kinds `insertionInline` and `insertionMultiLine`, class `originalOverlayInsertion`, setting `editor.inlineSuggest.edits.allowCodeShifting`)
- Pieces:
  1. A single-line insertion after the cursor shows inline in the text and shifts the code to the right to make room.
  2. A multi-line insertion shows as a bordered block that pushes the lines below it down.
  3. Setting `editor.inlineSuggest.edits.allowCodeShifting` chooses `always` (default, both shapes), `horizontal` (only the inline shift), or `never` (a line replacement or side by side view is used instead).

#### AC-2.25 See an inline edit as a deletion

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: An edit that only removes text outlines the text to delete in place with a bordered overlay in the original lines; no new text appears.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsDeletionView.ts (classes `originalOverlayDeletion`, `originalSeparatorDeletion`); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts; C bundle (class `originalOverlayDeletion`, view kind `deletion`)
- Pieces:
  1. An edit that only removes text outlines the text to delete in place with a bordered overlay in the original lines.
  2. A deletion shows no new text anywhere in the editor.

#### AC-2.26 See an inline edit collapsed

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With Show Collapsed on, the full preview is replaced by a small downward-pointing triangle at the start of the edit until you jump to it. Hovering the gutter pill shows the full preview again. The gutter menu row and the command flip between Show Collapsed and Show Expanded, and the triangle bobs once for users who have not yet become regular users.
- Commands: editor.action.inlineSuggest.toggleShowCollapsed
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.showCollapsed
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsCollapsedView.ts (classes `inline-edits-collapsed-view`, `collapsedView`, `collapsedViewPath`); V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (ToggleInlineCompletionShowCollapsed); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts; C bundle (id `editor.action.inlineSuggest.toggleShowCollapsed`, label "Toggle Inline Suggestions Show Collapsed", class `inline-edits-collapsed-view`, setting `editor.inlineSuggest.edits.showCollapsed`)
- Pieces:
  1. With Show Collapsed on, a small downward-pointing triangle at the start of the edit stands in for the full preview until you jump to the edit.
  2. Hovering the gutter pill shows the full preview again.
  3. The gutter menu row flips between Show Collapsed and Show Expanded.
  4. Running Toggle Inline Suggestions Show Collapsed flips the collapsed setting the same way (`editor.action.inlineSuggest.toggleShowCollapsed`).
  5. The triangle bobs once for users who have not yet become regular users.
  6. Setting `editor.inlineSuggest.edits.showCollapsed` chooses whether edits show collapsed until you jump to them, on or off, default off.

#### AC-2.27 See a "to jump" label at the jump target

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: When a suggestion is only a jump to a position in the file (no text to insert), a small bordered label with the Tab key chip and the words "to jump" is drawn at the target position (class `inline-edit-jump-to-widget`). A blinking-cursor style of the same widget exists in code.
- Commands: editor.action.inlineSuggest.commit
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\jumpToView.ts (decoration `inline-edit-jump-to-decoration`); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (view kind `jumpTo`); C bundle (`inline-edit-jump-to-widget`, `inline-edit-jump-to-decoration`, `"jumpTo"` searched, 0 hits)
- Pieces:
  1. When a suggestion is only a jump to a position in the file with no text to insert, a small bordered label with a Tab key chip and the words "to jump" shows at the target position. [stock only]
  2. Accepting it with `Tab` moves the cursor to that position (`editor.action.inlineSuggest.commit`). [stock only]
  3. The label can also be drawn in a blinking-cursor style. [stock only]

#### AC-2.28 See a hint label for an edit shown elsewhere

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: When a suggestion supplies a hint (text plus a range in the current file) instead of a diff, a small bordered text label is drawn at the end of that line, or just below or above it when the end of the line has no room. This view is chosen when the edit targets another document, or has a display location and has not been jumped to yet. The border color follows the Tab action, the text can be syntax-colored when the hint style is Code, and clicking accepts.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsCustomView.ts (class `inline-edits-custom-view`); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (view kind `custom`); V\src\vs\editor\common\languages.ts (InlineCompletionHintStyle Code and Label); C bundle (`inline-edits-custom-view` searched, 0 hits)
- Pieces:
  1. When a suggestion supplies a hint instead of a diff, a small bordered text label shows at the end of the hint's line. [stock only]
  2. When the end of that line has no room, the label shows just below or just above the line. [stock only]
  3. A suggestion for another file, or one with a display location that you have not jumped to yet, shows as this label instead of a diff. [stock only]
  4. The border color of the label follows what Tab will do. [stock only]
  5. With the Code hint style, the label text is syntax-colored. [stock only]
  6. Clicking the label accepts the suggestion. [stock only]

#### AC-2.29 See the long distance hint

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: When the next edit is outside the visible part of the file, or in another file, a small preview box of the target code appears near the cursor line, in free space on the right or over the code if there is none. Its bottom bar shows the symbol outline at the target (or the target file icon and name) and the label "Tab to jump" ("Go to suggestion" if Tab is not the key), "Tab to open" or "Go to file" for another file, plus an arrow showing the direction. Clicking it jumps. It exists in 1.128.0, so it is not newer; the bundle has none of it.
- Commands: editor.action.inlineSuggest.jump
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.showLongDistanceHint
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\longDistanceHint\inlineEditsLongDistanceHint.ts (class `inline-edits-long-distance-hint-widget`); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\longDistanceHint\longDistancePreviewEditor.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\longDistanceHint\longDistnaceWidgetPlacement.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (_getLongDistanceHintState); V\src\vs\editor\common\config\editorOptions.ts (experimental, tag nextEditSuggestions, default true); C bundle (`long-distance`, `longDistance`, `showLongDistanceHint`, `inline-edits-long-distance-hint-widget` searched, 0 hits)
- Pieces:
  1. When the next edit is outside the visible part of the file or in another file, a small preview box of the target code shows near the cursor line. [stock only]
  2. The box sits in free space to the right, or over the code when there is no free space. [stock only]
  3. The bottom bar of the box shows the symbol outline at the target, or the icon and name of the target file for another file. [stock only]
  4. The bar reads "Tab to jump" (or "Go to suggestion" when Tab is not the key) for the same file, and "Tab to open" or "Go to file" for another file. [stock only]
  5. An arrow in the bar shows which direction the target is in. [stock only]
  6. Clicking the box jumps to the target (`editor.action.inlineSuggest.jump`). [stock only]
  7. Setting `editor.inlineSuggest.edits.showLongDistanceHint` turns this hint on or off, default on, experimental. [stock only]

#### AC-2.30 Choose how many context lines the long distance hint shows

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: newer-than-1.128. A setting from 0 to 10 (default 0, experimental) for how many lines above and below the target line the long distance preview shows; 0 shows only the target line.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.longDistanceHintContextLineCount
- Source: N\src\vs\editor\common\config\editorOptions.ts; N\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\longDistanceHint\longDistancePreviewEditor.ts; C bundle (key searched, 0 hits)
- Pieces:
  1. Setting `editor.inlineSuggest.edits.longDistanceHintContextLineCount` chooses how many lines above and below the target line the long distance preview shows, from 0 to 10, default 0, experimental. [stock only]
  2. With 0 the preview shows only the target line, and with a larger value it also shows that many lines above and below. [stock only]

#### AC-2.31 See an accepted inline edit fade out

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: After you accept an inline edit, the changed ranges flash the modified-text background and fade out over about one second. It is skipped when motion is reduced.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.reduceMotion
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (FadeoutDecoration); V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css (class `edits-fadeout-decoration`); V\src\vs\platform\accessibility\browser\accessibilityService.ts; C bundle (class `edits-fadeout-decoration`, 1000 ms animation)
- Pieces:
  1. After you accept an inline edit, the changed ranges flash the modified-text background color.
  2. The flash fades out over about one second.
  3. When reduced motion is on (`workbench.reduceMotion`), the flash is skipped.

#### AC-2.32 Recolor the inline edit UI with theme colors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Theme colors change how inline edits look: `inlineEdit.originalBackground`, `.modifiedBackground`, `.originalChangedLineBackground`, `.originalChangedTextBackground`, `.modifiedChangedLineBackground`, `.modifiedChangedTextBackground`, `.originalBorder`, `.modifiedBorder`, `.tabWillAcceptModifiedBorder`, `.tabWillAcceptOriginalBorder`, and the gutter pill colors `inlineEdit.gutterIndicator.*` in plain, primary, secondary, and successful variants (foreground, border, background).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\theme.ts; C bundle (ids `inlineEdit.gutterIndicator.primaryBackground`, `.successfulBackground`, `.secondaryBackground`, `inlineEdit.tabWillAcceptModifiedBorder`, `inlineEdit.tabWillAcceptOriginalBorder`, `inlineEdit.modifiedBorder`, `inlineEdit.originalBorder`, `inlineEdit.originalBackground`)
- Pieces:
  1. Theme colors `inlineEdit.originalBackground` and `inlineEdit.modifiedBackground` set the background behind the original code and behind the new code of an inline edit.
  2. Theme colors `inlineEdit.originalChangedLineBackground` and `inlineEdit.originalChangedTextBackground` set the line and character highlight on the original code that will go.
  3. Theme colors `inlineEdit.modifiedChangedLineBackground` and `inlineEdit.modifiedChangedTextBackground` set the line and character highlight on the new code.
  4. Theme colors `inlineEdit.originalBorder` and `inlineEdit.modifiedBorder` set the border around the original code and around the new code.
  5. Theme colors `inlineEdit.tabWillAcceptModifiedBorder` and `inlineEdit.tabWillAcceptOriginalBorder` set the borders used while Tab will accept the edit.
  6. Theme colors `inlineEdit.gutterIndicator.*` set the foreground, border, and background of the gutter pill in its plain, primary, secondary, and successful variants.
  7. Each of these colors can be overridden under `workbench.colorCustomizations`.

#### AC-2.33 Turn inline suggestions on or off

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: One setting (default on) controls whether inline suggestions show automatically as you type.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.enabled; editor.inlineSuggest; editor.inlineSuggest.experimental.suppressInlineSuggestions
- Source: V\src\vs\editor\common\config\editorOptions.ts; C bundle (key `editor.inlineSuggest.enabled`, NLS "Controls whether to automatically show inline suggestions in the editor.")
- Pieces:
  1. Setting `editor.inlineSuggest.enabled` turns automatic inline suggestions on or off, default on.
  2. With the setting off, no inline suggestion shows on its own as you type.

#### AC-2.34 Hold suggestions back after typing or in snippet mode

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: Two settings delay or silence suggestions: `minShowDelay` waits 0 to 10000 ms (default 0) after typing before a suggestion shows, and `suppressInSnippetMode` (default on) hides inline suggestions while a snippet's tab stops are active.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.minShowDelay; editor.inlineSuggest.suppressInSnippetMode
- Source: V\src\vs\editor\common\config\editorOptions.ts; C bundle (`minShowDelay` and `suppressInSnippetMode` searched, 0 hits; NLS "Controls the minimal delay" and "Controls whether inline suggestions are suppressed when in snippet mode" 0 hits)
- Pieces:
  1. Setting `editor.inlineSuggest.minShowDelay` makes a suggestion wait from 0 to 10000 ms after typing before it shows, default 0. [stock only]
  2. Setting `editor.inlineSuggest.suppressInSnippetMode`, default on, hides inline suggestions while the tab stops of a snippet are active. [stock only]
  3. With `editor.inlineSuggest.suppressInSnippetMode` off, inline suggestions still show while snippet tab stops are active. [stock only]

#### AC-2.35 Keep the suggest list quiet while ghost text shows

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With `suppressSuggestions` on (default off), the suggest list does not open on its own while an inline suggestion is available. The context key `inlineSuggestionSuppressSuggestions` carries the flag a suggestion sets for itself.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.suppressSuggestions; editor.inlineSuggest.experimental.showOnSuggestConflict
- Source: V\src\vs\editor\common\config\editorOptions.ts; V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionContextKeys.ts; V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts; C bundle (key `editor.inlineSuggest.suppressSuggestions`, context key `inlineSuggestionSuppressSuggestions`, NLS "Controls how inline suggestions interact with the suggest widget...")
- Pieces:
  1. With `editor.inlineSuggest.suppressSuggestions` on, the suggest list does not open on its own while an inline suggestion is available.
  2. With `editor.inlineSuggest.suppressSuggestions` off, the default, the suggest list can still open on its own next to ghost text.
  3. A suggestion can set the same flag for itself (context key `inlineSuggestionSuppressSuggestions`), and the suggest list then stays closed on its own while that suggestion shows.

#### AC-2.36 Snooze inline suggestions

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: "Snooze Inline Suggestions" opens a quick pick titled "Select snooze duration for Inline Suggestions" with 1, 5, 10, 15, 30 and 60 minutes, "Custom..." (asks "Enter snooze duration in minutes", placeholder "Duration in minutes (e.g. 90)"), and a "{n} minutes (Last used)" row when you used a custom length before. The command also accepts a minutes number as its first argument. "Cancel Snooze Inline Suggestions" ends it and only shows in the palette while snoozed (context key `inlineCompletions.snoozed`). The gutter menu has a Snooze row (AC-2.18). Cursor's own snooze for Cursor Tab is a different command, handed off to AC-8.
- Commands: editor.action.inlineSuggest.snooze; editor.action.inlineSuggest.cancelSnooze
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\browser\services\inlineCompletionsService.ts; V\src\vs\editor\contrib\inlineCompletions\browser\inlineCompletions.contribution.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\components\gutterIndicatorMenu.ts; C bundle (both ids, `inlineCompletions.snoozed`, `inlineCompletions.lastSnoozeDuration`, and the labels "Snooze Inline Suggestions", "Cancel Snooze Inline Suggestions", "Select snooze duration for Inline Suggestions" searched in the bundle and NLS, 0 hits; `editor.cpp.snooze` found, owned by AC-8)
- Pieces:
  1. Running Snooze Inline Suggestions from the Command Palette opens a quick pick titled "Select snooze duration for Inline Suggestions". [stock only]
  2. The quick pick lists 1, 5, 10, 15, 30, and 60 minutes. [stock only]
  3. The Custom... row asks "Enter snooze duration in minutes" with the placeholder "Duration in minutes (e.g. 90)". [stock only]
  4. After a custom length has been used, a row reading "{n} minutes (Last used)" shows in the quick pick. [stock only]
  5. The command also takes a number of minutes as its first argument and snoozes for that long. [stock only]
  6. While snoozed, inline suggestions do not show until the time ends. [stock only]
  7. Cancel Snooze Inline Suggestions ends the snooze and shows in the Command Palette only while a snooze is on (`inlineCompletions.snoozed`). [stock only]

#### AC-2.37 See inline suggestion status in the language status popup

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (an editor contribution `vs.editor.contrib.inlineCompletionLanguageStatusBarContribution` instead of a workbench contribution; no chat entitlement check; shows only while the editor has focus)
- What: The language status popup (the language indicator on the status bar) gets an "Inline Suggestions" row for the focused file with the detail "Inline suggestions" and one of four states: "Loading..." (busy), "Inline completion available" (short icon lightbulb), "Inline edit available" (lightbulb-sparkle), or "No inline suggestion available" (circle-slash), updated after a 300 ms debounce. Stock hides the row when AI features are turned off. This is the workbench contribution the status bar uses for inline completions.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: chat.disableAIFeatures
- Source: V\src\vs\workbench\contrib\inlineCompletions\browser\inlineCompletions.contribution.ts; V\src\vs\workbench\contrib\inlineCompletions\browser\inlineCompletionLanguageStatusBarContribution.ts (id `vs.contrib.inlineCompletionLanguageStatusBarContribution`, status id `inlineSuggestions`); V\src\vs\workbench\services\chat\common\chatEntitlementService.ts (hidden state follows `chat.disableAIFeatures`); C bundle (status id `inlineSuggestions`, id `vs.editor.contrib.inlineCompletionLanguageStatusBarContribution`, NLS "Loading...", "Inline completion available", "Inline edit available", "No inline suggestion available", "Inline suggestions"; `IChatEntitlementService` and `chat.disableAIFeatures` searched, 0 hits)
- Pieces:
  1. The language status popup gets an "Inline Suggestions" row for the focused file, with the detail "Inline suggestions".
  2. While a suggestion is being fetched, the row reads "Loading..." and shows as busy.
  3. When ghost text is available, the row reads "Inline completion available" with a lightbulb icon.
  4. When an inline edit is available, the row reads "Inline edit available" with a lightbulb-sparkle icon.
  5. When nothing is available, the row reads "No inline suggestion available" with a circle-slash icon.
  6. The row updates after a 300 ms debounce.
  7. The row is hidden when AI features are turned off with `chat.disableAIFeatures`. [stock only]
  8. In Cursor, the row shows only while the editor has focus and is never hidden by an AI features check. [cursor only]

#### AC-2.38 Inspect a suggestion in the accessible view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With a suggestion showing, Alt+F2 opens the accessible view with the line plus ghost text, or a readable patch for an inline edit. Show Next and Show Previous inside the view step through suggestions, Ctrl+/ accepts it (button "Accept Inline Completion" in the view toolbar), and closing the view hides the suggestion. A screen reader hint "Inspect this in the accessible view ({0})" points to it.
- Commands: editor.action.accessibleView; editor.action.accessibleViewNext; editor.action.accessibleViewPrevious; editor.action.accessibleViewAcceptInlineCompletion
- Menus: AccessibleView; CommandPalette
- Keys: Alt+F2 to open (Linux: Alt+Shift+F2, with Alt+F2 as secondary); Ctrl+/ to accept while the view shows an inline completion (Control+/ on macOS)
- Settings: accessibility.verbosity.inlineCompletions
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\inlineCompletionsAccessibleView.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts; V\src\vs\workbench\contrib\accessibility\common\accessibilityCommands.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts; C bundle (ids `editor.action.accessibleViewAcceptInlineCompletion`, `editor.action.accessibleViewNext`, `editor.action.accessibleViewPrevious`, setting `accessibility.verbosity.inlineCompletions`, NLS "Accept Inline Completion", "Inspect this in the accessible view ({0})")
- Pieces:
  1. With a suggestion showing, `Alt+F2` opens the accessible view with the line plus the ghost text (`Alt+Shift+F2` on Linux, with `Alt+F2` as a second key).
  2. For an inline edit, the accessible view shows a readable patch.
  3. Show Next and Show Previous inside the accessible view step through the suggestions (`editor.action.accessibleViewNext`, `editor.action.accessibleViewPrevious`).
  4. `Ctrl+/` (`Control+/` on macOS) accepts the suggestion while the view shows an inline completion, and the view toolbar has an "Accept Inline Completion" button (`editor.action.accessibleViewAcceptInlineCompletion`).
  5. Closing the accessible view hides the suggestion.
  6. A screen reader hint "Inspect this in the accessible view ({0})" points to the key, shown when `accessibility.verbosity.inlineCompletions` allows it.

#### AC-2.39 Hear and read suggestions with a screen reader or audio cue

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the nextEditSuggestion signal key is not in the bundle; the announce option, the verbosity option, the line cue key, and the hover texts are)
- What: When a suggestion appears, a screen reader reads it aloud (on by default), with the accessible view hint added when the verbosity setting allows. If the announce setting is off, a screen reader user gets the hover text "Suggestion:" plus the code, and the hover reads "There are inline completions here". A sound cue can play when a line has an inline suggestion, and a second cue exists for next edit suggestions.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.screenReaderAnnounceInlineSuggestion; editor.inlineCompletionsAccessibilityVerbose; accessibility.verbosity.inlineCompletions; accessibility.signals.lineHasInlineSuggestion; accessibility.signals.nextEditSuggestion
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (_provideScreenReaderUpdate, playAccessibilitySignal); V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\hoverParticipant.ts (renderScreenReaderText, getAccessibleContent); V\src\vs\platform\accessibilitySignal\browser\accessibilitySignalService.ts; V\src\vs\editor\common\config\editorOptions.ts; C bundle (`screenReaderAnnounceInlineSuggestion`, `inlineCompletionsAccessibilityVerbose`, `accessibility.signals.lineHasInlineSuggestion`, NLS "Suggestion:", "There are inline completions here"; `accessibility.signals.nextEditSuggestion` searched, 0 hits)
- Pieces:
  1. When a suggestion appears, a screen reader reads it aloud.
  2. Setting `editor.screenReaderAnnounceInlineSuggestion` turns that announcement on or off, default on.
  3. Settings `accessibility.verbosity.inlineCompletions` and `editor.inlineCompletionsAccessibilityVerbose` choose whether the announcement adds the hint about the accessible view.
  4. With the announcement off, a screen reader user gets hover text "Suggestion:" followed by the code, and the hover reads "There are inline completions here".
  5. Setting `accessibility.signals.lineHasInlineSuggestion` chooses whether a sound cue plays when a line has an inline suggestion.
  6. Setting `accessibility.signals.nextEditSuggestion` chooses whether a second sound cue plays for next edit suggestions. [stock only]

#### AC-2.40 Copy a repro of the current suggestion

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (precondition is only inlineEditIsVisible, stock also allows inlineSuggestionVisible)
- What: The palette command "Developer: Extract Inline Suggest State" copies the file text plus the current suggestion as commented JSON to the clipboard so a bug can be reproduced.
- Commands: editor.action.inlineSuggest.dev.extractRepro
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (DevExtractReproSample); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (extractReproSample); C bundle (id `editor.action.inlineSuggest.dev.extractRepro`, label "Developer: Extract Inline Suggest State")
- Pieces:
  1. Running Developer: Extract Inline Suggest State from the Command Palette copies the file text plus the current suggestion, as commented JSON, to the clipboard (`editor.action.inlineSuggest.dev.extractRepro`).
  2. In stock, the command is enabled while plain ghost text shows as well as while an inline edit shows. [stock only]
  3. In Cursor, the command is enabled only while an inline edit is visible. [cursor only]

#### AC-2.41 See quick suggestions as ghost text

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the provider class is in the bundle with `_getTriggerCharacterInfo`, a `canBeReused` check and the `valueFor(...)!=="inline"` test; NLS "Quick suggestions show as ghost text" found; the bundle has the values on, inline and off but not offWhenInlineCompletions, see AC-1.3)
- What: Set `editor.quickSuggestions` to `inline` (for all code, or only for other code, comments or strings) and, at the end of a word or right after a trigger character, the best matching completion shows as ghost text and the suggest list does not open. The provider hands over every matching item, with the one picked last time first and the first six resolved early, so the n/m count and the next and previous keys of AC-2.9 should step through them (not run). This provider gives nothing while the suggest list is open, so a list opened by hand with Ctrl+Space replaces the ghost text. The setting itself is owned by AC-1.3, the ghost text that results is here.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.quickSuggestions
- Source: V\src\vs\editor\contrib\suggest\browser\suggestInlineCompletions.ts (SuggestInlineCompletions, registered through V\src\vs\editor\editor.all.ts); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (quick suggest gate, value `inline` returns early); V\src\vs\editor\contrib\suggest\browser\suggest.ts (QuickSuggestionsOptions); V\src\vs\editor\common\config\editorOptions.ts (EditorQuickSuggestions, enum descriptions "Quick suggestions show as ghost text"); C bundle (provider class around `_getTriggerCharacterInfo`; NLS "Quick suggestions show as ghost text")
- Pieces:
  1. With `editor.quickSuggestions` set to `inline`, the best matching completion shows as ghost text at the end of a word and the suggest list does not open.
  2. The same ghost text shows right after a trigger character.
  3. The values for `other`, `comments`, and `strings` can each be `inline`, so the ghost text shows only in that kind of code.
  4. The `n/m` count and the `Alt+]` and `Alt+[` keys of AC-2.9 step through the other matching completions.
  5. While the suggest list is open this ghost text is not shown, so opening the list by hand with `Ctrl+Space` replaces it.

#### AC-2.42 Use ghost text with several cursors

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the bundle carries one ghost text per cursor in the model state and a secondary edit helper, and the view loops over those ghost texts; the accept code sets one selection per edit; not run)
- What: With more than one cursor, ghost text is drawn at every cursor where the text the suggestion would replace is the same as at the main cursor (a plain insertion matches at every cursor), not only at the main one. Tab, Accept Word and Accept Line then type the same text at each of those cursors and leave one cursor at the end of each. Inline edits (AC-2.21 to AC-2.29) are for the main cursor only and keep a single cursor after accept.
- Commands: editor.action.inlineSuggest.commit; editor.action.inlineSuggest.acceptNextWord; editor.action.inlineSuggest.acceptNextLine
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (getSecondaryEdits, ghostTexts, accept, _acceptNext); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts (_ghostTexts); V\src\vs\editor\contrib\inlineCompletions\test\browser\getSecondaryEdits.test.ts; C bundle (`ghostTexts`, the secondary edit helper next to `inlineCompletionPartialAccept`, view field `_ghostTexts`)
- Pieces:
  1. With more than one cursor, ghost text shows at every cursor where the text the suggestion would replace matches the main cursor, not only at the main one.
  2. A plain insertion matches at every cursor, so its ghost text shows at all of them.
  3. `Tab` types the same text at each of those cursors and leaves one cursor at the end of each.
  4. Accept Word and Accept Line do the same at each of those cursors.
  5. Inline edits show only for the main cursor and leave a single cursor after accept.

#### AC-2.43 Pick the list item that matches the ghost text

- Tag: both. Evidence: source. Boundary: no. Cursor: present (a selector registered on the suggest controller with priority 100 is in the bundle)
- What: When the suggest list opens while ghost text is showing, the list starts with the item the ghost text builds on already highlighted: the longest list item whose text is the start of the inline suggestion. If no item fits, the list picks its usual first item. The suggest list itself is AC-1.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts (registerSelector priority 100, singleTextEditAugments); V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (preselector passed to ObservableSuggestWidgetAdapter); V\src\vs\editor\contrib\inlineCompletions\browser\model\singleTextEditHelpers.ts; C bundle (`registerSelector({priority:100,select:...` with the same preselector)
- Pieces:
  1. When the suggest list opens while ghost text shows, the list starts with the item the ghost text builds on already highlighted, which is the longest list item whose text is the start of the suggestion.
  2. When no list item fits, the suggest list highlights its usual first item.

#### AC-2.44 See ghost text continue past the highlighted list item

- Tag: both. Evidence: source. Boundary: no. Cursor: present (`_computeAugmentation`, the preview-enabled check that returns early only when there is no augmentation, and the Shift key handlers are in the bundle)
- What: When an inline suggestion starts with the same text as the highlighted list item and covers at least the same range, the editor shows the whole inline suggestion as ghost text, and the part beyond the list item is drawn in the preview style. This works even with `editor.suggest.preview` off (AC-2.7); the preview is then used only when no inline suggestion extends the item. Holding Shift while the list is open flips the insert or replace range used for this preview, the same way Shift+Tab does for accepting. The experimental `showOnSuggestConflict` setting can turn this off, it is provider facing and not cataloged.
- Commands: none found
- Menus: none found
- Keys: Shift (held while the suggest list is open, flips insert and replace for the preview)
- Settings: editor.suggest.preview
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (_computeAugmentation, state derivation with fullEditPreviewLength); V\src\vs\editor\contrib\inlineCompletions\browser\model\computeGhostText.ts (previewSuffixLength); V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts (isShiftKeyPressed, getOverwriteInfo); V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts (class `ghost-text-decoration-preview`); C bundle (`_computeAugmentation`, `isShiftKeyPressed`)
- Pieces:
  1. When an inline suggestion starts with the same text as the highlighted list item and covers at least the same range, the editor shows the whole suggestion as ghost text.
  2. The part of the ghost text beyond the list item is drawn in the preview style.
  3. This works with `editor.suggest.preview` off, and the plain preview is then used only when no inline suggestion extends the item.
  4. Holding `Shift` while the list is open flips the insert or replace range used for this preview, the same way `Shift+Tab` does for accepting.

#### AC-2.45 Use inline suggestions in the commit message and comment boxes

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the controller is in both lists, the 18 entry list of the commit box and the 10 entry list of the comment reply box; the bundle's `scm.clearInput` rule has no inline suggestion guard and no validation message guard, and its weight 200 is above the weight 190 of Esc for Hide Inline Suggestion, so Esc in the commit box should empty the message even while ghost text shows; read from the rule weights, not run)
- What: The commit message box in Source Control and the reply box of a review comment thread both host ghost text, with the same Tab, Esc, Accept Word keys and the same toolbar as the main editor. In the commit box stock lets Esc reject a showing suggestion first and only empties the message when no suggestion is showing. Notebook cell editors and the chat input also host the same controller, those are not part of this feature (chat is out of scope).
- Commands: editor.action.inlineSuggest.hide; scm.clearInput
- Menus: none found
- Keys: Escape (scm.clearInput, when scmRepository, suggest list hidden, no inline suggestion visible, no validation message, no non-empty selection)
- Settings: none found
- Source: V\src\vs\workbench\contrib\scm\browser\scmInput.ts (contribution list with InlineCompletionsController.ID); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (scm.clearInput, weight WorkbenchContrib); V\src\vs\workbench\contrib\comments\browser\simpleCommentEditor.ts (contribution list with InlineCompletionsController.ID); V\src\vs\workbench\contrib\notebook\browser\notebookEditorWidget.ts (clearActiveCellWidgets); C bundle (`scm.clearInput` rule with weight 200; the two `getSomeEditorContributions` lists that hold the controller class `J1`)
- Pieces:
  1. The commit message box in Source Control shows ghost text and the toolbar, with the same `Tab`, `Escape`, and Accept Word keys as the main editor.
  2. The reply box of a review comment thread shows ghost text and the toolbar the same way.
  3. In stock, `Escape` in the commit message box first rejects a showing suggestion and empties the message only when no suggestion is showing (`editor.action.inlineSuggest.hide`, `scm.clearInput`). [stock only]
  4. In Cursor, `Escape` in the commit message box empties the message even while ghost text shows, because the clear rule outranks the reject rule. [cursor only]

#### AC-2.46 See a suggestion step aside

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the cursor move rule that stops only ghost text, the blur rule with `accessibleViewIsShown`, `keepOnBlur` and the toolbar `dropDownVisible` checks, the peek counter `openedPeekWidgets` and `_hasVisiblePeekWidgets`, and the read-only guard on the model are all in the bundle)
- What: Ghost text goes away when you move the cursor yourself (a click or an arrow key, not typing), but an inline edit stays. Any suggestion goes away when the editor loses focus, unless focus is in the gutter indicator menu, the accessible view is open, the toolbar's overflow menu is open, or it is an inline edit you asked for with the trigger command. Focusing another editor ends the suggestion in the first one. Inline edits also hide while a peek view (peek definition or references) is open, and a read-only editor never shows suggestions. The hidden setting `editor.inlineSuggest.keepOnBlur` keeps them (listed in Unclear, not a user setting).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (model with isReadonly guard, selections handler, focus handler, _instances); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (_hasVisiblePeekWidgets); V\src\vs\editor\browser\observableCodeEditor.ts (openedPeekWidgets); V\src\vs\editor\contrib\peekView\browser\peekView.ts (counter); C bundle (same checks in the controller and model classes)
- Pieces:
  1. Ghost text goes away when you move the cursor yourself with a click or an arrow key, but not when you type.
  2. An inline edit stays when you move the cursor yourself.
  3. Any suggestion goes away when the editor loses focus, including when you focus another editor.
  4. A suggestion stays when focus moves into the gutter indicator menu, the accessible view, or the toolbar overflow menu, and an inline edit you asked for with the trigger command stays too.
  5. Inline edits hide while a peek view (peek definition or references) is open.
  6. A read-only editor never shows suggestions.

#### AC-2.47 Type along with a suggestion

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the same check is in the bundle: indentation trimmed, then a `startsWith` test and a substring match; module fragment found)
- What: While you type the same characters as the ghost text, it stays and gets shorter. As soon as you type something that is not the next part of it, it goes away. Leading indentation is ignored in the comparison, and the text to the right of the cursor must still fit the rest of the suggestion. A suggestion that starts on a different line than the cursor is not shown.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionIsVisible.ts; V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineSuggestionItem.ts (isVisible); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (visibility filter); C bundle (visibility check next to `canBeReused` and `reuse` in the inline completion item class)
- Pieces:
  1. While you type the same characters as the ghost text, it stays and gets shorter.
  2. As soon as you type something that is not the next part of the ghost text, it goes away.
  3. Leading indentation is ignored when your typing is compared with the ghost text.
  4. The text to the right of the cursor must still fit the rest of the suggestion, or the ghost text goes away.
  5. A suggestion that starts on a different line than the cursor is not shown.

#### AC-2.48 Accept a suggestion that is a snippet

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the snippet branch that inserts through the snippet controller, and the accept-whole check in the next word and next line code, are in the bundle; the setting `suppressInSnippetMode` is missing there, see AC-2.34)
- What: A suggestion that carries a snippet (tab stops and placeholders) shows as plain ghost text. Tab or the Accept button inserts it as a snippet, so the cursor lands on the first tab stop and snippet mode (AC-4) starts. Accept Word and Accept Line do not take a part of it, they accept the whole snippet. While snippet mode is on, other suggestions stay hidden if `suppressInSnippetMode` is on (AC-2.34).
- Commands: editor.action.inlineSuggest.commit; editor.action.inlineSuggest.acceptNextWord; editor.action.inlineSuggest.acceptNextLine
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.suppressInSnippetMode
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (accept with snippetInfo, _acceptNext); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineSuggestionItem.ts (snippetInfo); V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (insert); C bundle (`i.snippetInfo` branch calling the snippet controller `insert`, `s.snippetInfo||s.filterText!==s.insertText` then accept)
- Pieces:
  1. A suggestion with tab stops and placeholders shows as plain ghost text.
  2. `Tab` or the Accept button inserts it as a snippet, so the cursor lands on the first tab stop and snippet mode starts.
  3. Accept Word and Accept Line take the whole snippet instead of a part of it.
  4. While snippet mode is on, other suggestions stay hidden when `editor.inlineSuggest.suppressInSnippetMode` is on. [stock only]


## AC-3 Parameter hints

(both). The signature popup with the active parameter and an overload switcher. Examples: signature with active parameter; overload up and down; Ctrl+Shift+Space and Esc; docs under the signature.

13 features.

#### AC-3.1 Open parameter hints by hand

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, label "Trigger Parameter Hints", the key with the same modifiers and the hasSignatureHelpProvider precondition found)
- What: Press Ctrl+Shift+Space (Cmd+Shift+Space on macOS) with the cursor inside a call to open the signature popup at once, or run Trigger Parameter Hints from the Command Palette; nothing happens when no signature help provider covers the file.
- Commands: editor.action.triggerParameterHints
- Menus: none found
- Keys: Ctrl+Shift+Space (Cmd+Shift+Space on macOS) when editorTextFocus; the command needs editorHasSignatureHelpProvider
- Settings: none found
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHints.ts (TriggerParameterHintsAction lines 75-96, manual trigger with zero delay lines 70-72); V\src\vs\editor\common\editorContextKeys.ts (line 87); V\src\vs\editor\browser\widget\codeEditor\codeEditorWidget.ts (lines 2335 and 2416); V\src\vs\editor\contrib\quickAccess\browser\commandsQuickAccess.ts (editor actions listed in the palette, lines 42-55); C bundle (id editor.action.triggerParameterHints, NLS message 1453)
- Pieces:
  1. Pressing `Ctrl+Shift+Space` (`Cmd+Shift+Space` on macOS) with the cursor inside a call opens the signature popup at once (`editor.action.triggerParameterHints`).
  2. Running Trigger Parameter Hints from the Command Palette opens the popup the same way.
  3. The key works only while the editor text has focus.
  4. When no signature help provider covers the file, the key and the command do nothing.

#### AC-3.2 Open hints by typing a trigger character

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the model code reads signatureHelpTriggerCharacters and signatureHelpRetriggerCharacters from each provider; the same settings key found)
- What: Typing a character that the language lists as a signature trigger (for TypeScript and JavaScript that is `(`, `,` or `<`) opens the popup about 120 ms later, as long as automatic hints are on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.parameterHints.enabled
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (DEFAULT_DELAY line 52, trigger characters collected in onModelChanged lines 250-278, onDidType lines 280-294); V\extensions\typescript-language-features\src\languageFeatures\signatureHelp.ts (trigger characters lines 16-17, registration lines 131-135); C bundle (signatureHelpTriggerCharacters in the model code)
- Pieces:
  1. Typing a character that the language lists as a signature trigger opens the popup about 120 ms later.
  2. For TypeScript and JavaScript the trigger characters are `(`, `,`, and `<`.
  3. Setting `editor.parameterHints.enabled` chooses whether typing a trigger character opens the popup on its own, on or off, default on.

#### AC-3.3 Hints follow your typing and cursor

- Tag: both. Evidence: source. Boundary: no. Cursor: present (retrigger characters, the isRetrigger flag and activeSignatureHelp pass-through found in the bundle model code)
- What: While the popup is open, typing, deleting or moving the cursor with the keys refreshes it so the bold parameter follows the comma you are in, and a retrigger character (TypeScript lists `)`) refreshes it too; a refresh keeps the overload you were on when the language reports the same signature again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (isTriggered lines 244-248, onCursorChange lines 296-302, onModelContentChange lines 304-308, doTrigger lines 176-233, mergeTriggerContexts lines 324-340); V\extensions\typescript-language-features\src\languageFeatures\signatureHelp.ts (getActiveSignature keeps the previous label, lines 52-63; retrigger reason lines 103-122); C bundle (isRetrigger in the provide call)
- Pieces:
  1. While the popup is open, typing or deleting refreshes it so the bold parameter follows the comma you are in.
  2. Moving the cursor with the keys refreshes the popup the same way.
  3. Typing a retrigger character (TypeScript lists `)`) refreshes the popup too.
  4. A refresh keeps the overload you were on when the language reports the same signature again.

#### AC-3.4 Signature with the active parameter highlighted

- Tag: both. Evidence: source. Boundary: no. Cursor: present (class "parameter active", the label offset search, hasDocs and the color id found; CSS rule for the bold highlighted span found)
- What: The popup shows the whole signature on one line in the editor font with the parameter you are on in bold and the list highlight color (theme color editorHoverWidget.highlightForeground); a signature with no parameters shows its plain label, and the active parameter is the signature's own, falling back to the overall one.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsWidget.ts (render lines 183-211, renderParameters lines 297-311, getParameterLabelOffsets lines 313-329, color line 367); V\src\vs\editor\contrib\parameterHints\browser\parameterHints.css (.code lines 74-76, .parameter.active lines 149-152); C bundle (CSS .parameter-hints-widget .signature .parameter.active)
- Pieces:
  1. The popup shows the whole signature on one line in the editor font.
  2. The parameter you are on is drawn in bold in the highlight color (theme color `editorHoverWidget.highlightForeground`).
  3. A signature with no parameters shows its plain label.
  4. The active parameter is the one the signature names for itself, and the overall active parameter is used when the signature names none.

#### AC-3.5 Overload counter and arrows

- Tag: both. Evidence: source. Boundary: no. Cursor: present (controls, overloads and button classes, both icon ids parameter-hints-previous and parameter-hints-next, the padStart counter code and the multiple CSS rules found)
- What: When the language returns more than one signature the popup gains a narrow strip on its left with an up chevron, a zero padded counter such as 2/3 (or 02/12 with twelve), a down chevron and a faint divider; clicking a chevron shows the previous or next signature and puts focus back in the editor.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsWidget.ts (icons lines 32-33, DOM and click handlers lines 69-88, multiple flag lines 190-193, counter lines 239-241, next and previous lines 331-339); V\src\vs\editor\contrib\parameterHints\browser\parameterHints.css (.multiple lines 31-43, .controls, .button and .overloads lines 118-147); C bundle (CSS .multiple and .multiple .body:before)
- Pieces:
  1. When the language returns more than one signature, the popup gets a narrow strip on its left.
  2. The strip shows an up chevron, a counter such as 2/3, a down chevron, and a faint divider.
  3. The counter is zero padded to the width of the total, for example 02/12 with twelve signatures.
  4. Clicking a chevron shows the previous or next signature and puts focus back in the editor.
  5. With only one signature the strip is not shown.

#### AC-3.6 Switch overloads with the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids, the same primary and secondary keys, and the two context keys found)
- What: With the popup open and more than one signature, Up or Alt+Up shows the previous signature and Down or Alt+Down the next (on macOS also Ctrl+P and Ctrl+N), and the popup takes those keys from the editor only while it is open.
- Commands: showPrevParameterHint; showNextParameterHint
- Menus: none found
- Keys: Up; Alt+Up (mac also Ctrl+P) for the previous signature and Down; Alt+Down (mac also Ctrl+N) for the next, all when editorFocus and parameterHintsVisible and parameterHintsMultipleSignatures
- Settings: none found
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHints.ts (weight line 101, commands lines 117-141); V\src\vs\editor\contrib\parameterHints\browser\provideSignatureHelp.ts (context keys lines 19-22); C bundle (ids showPrevParameterHint and showNextParameterHint with weight 175)
- Pieces:
  1. With the popup open and more than one signature, `Up` or `Alt+Up` shows the previous signature (`showPrevParameterHint`).
  2. With the popup open and more than one signature, `Down` or `Alt+Down` shows the next signature (`showNextParameterHint`).
  3. On macOS `Ctrl+P` also shows the previous signature and `Ctrl+N` also shows the next.
  4. The popup takes these keys from the editor only while it is open with more than one signature, and otherwise the keys move the cursor as usual.

#### AC-3.7 Wrap around or stop at the last overload

- Tag: both. Evidence: source. Boundary: no. Cursor: present (settings key editor.parameterHints.cycle with default true found)
- What: With wrap around on, Down on the last signature jumps to the first and Up on the first jumps to the last; with it off, going past either end closes the popup instead.
- Commands: showPrevParameterHint; showNextParameterHint
- Menus: none found
- Keys: none found
- Settings: editor.parameterHints.cycle
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (next lines 129-146, previous lines 148-165); V\src\vs\editor\common\config\editorOptions.ts (EditorParameterHints lines 3647-3705, cycle at lines 3685-3689); C bundle (editor.parameterHints.cycle with default e.cycle and cycle:!0)
- Pieces:
  1. With wrap around on, `Down` on the last signature shows the first one, and `Up` on the first signature shows the last one.
  2. With wrap around off, going past either end closes the popup.
  3. Setting `editor.parameterHints.cycle` chooses wrap around, on or off, default on.

#### AC-3.8 Documentation under the signature

- Tag: both. Evidence: source. Boundary: no. Cursor: present (docs element toggled with the empty class, has-docs on the signature, markdown-docs class and the hasDocs test found; CSS rules for .docs found)
- What: Below the signature the popup shows the documentation of the active parameter first and then the documentation of the whole signature, as plain text or rendered markdown (code spans, code blocks, links), set off from the signature by a faint rule; with no documentation the area is hidden, and the text can be selected with the mouse.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsWidget.ts (docs rendering lines 213-238, hasDocs lines 281-295, markdown renderer lines 270-279, userSelect line 99); V\src\vs\editor\contrib\parameterHints\browser\parameterHints.css (.has-docs rule lines 63-72, .docs rules lines 78-117); C bundle (render code with has-docs and empty toggles; CSS .signature.has-docs:after)
- Pieces:
  1. Below the signature the popup shows the documentation of the active parameter first and then the documentation of the whole signature.
  2. Documentation shows as plain text or as rendered markdown with code spans, code blocks, and links.
  3. A faint rule sets the documentation apart from the signature.
  4. With no documentation the documentation area is hidden.
  5. The documentation text can be selected with the mouse.

#### AC-3.9 Popup position, size and font

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (position, max height, 440 px width and font code all found; the frame rule found in the CSS declares border and z-index only, where V 1.128 also declares a corner radius and a shadow from theme tokens, the same older-snapshot pattern seen on the suggest widget; a shadow set by another rule was not ruled out)
- What: The popup sits above the cursor line (below when there is no room), moves with the cursor, is at most 440 px wide and at most a quarter of the editor height (never less than 250 px) tall with its own scrollbar for long documentation, and uses the editor's font size with the signature in the editor font.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsWidget.ts (allowEditorOverflow lines 54-55, scrollbar lines 90-97, font lines 111-140, getPosition lines 173-181, updateMaxHeight lines 352-364); V\src\vs\editor\contrib\parameterHints\browser\parameterHints.css (frame lines 6-18, width lines 25-29); C bundle (CSS .parameter-hints-widget frame rule and .phwrapper max-width 440px; updateMaxHeight code)
- Pieces:
  1. The popup sits above the cursor line, below it when there is no room above, and it moves with the cursor.
  2. The popup is at most 440 px wide.
  3. The popup is at most as tall as the larger of a quarter of the editor height and 250 px, and long documentation scrolls inside it with its own scrollbar.
  4. The popup uses the font size of the editor, with the signature in the editor font.
  5. The popup frame has a border, a rounded corner, and a drop shadow taken from theme tokens. [stock only]

#### AC-3.10 Close the hints

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, both keys and the visible precondition found; the blur and mouse-click cancel paths are in the model code)
- What: Esc or Shift+Esc closes the popup while the editor has focus; it also closes when the editor loses focus, on a mouse click in the editor text, when the language returns no signature, and at the end of the list when wrap around is off.
- Commands: closeParameterHints
- Menus: none found
- Keys: Escape; Shift+Escape when editorFocus and parameterHintsVisible
- Settings: editor.parameterHints.cycle
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHints.ts (lines 101-116); V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (blur line 83, mouse click lines 296-298, empty result lines 216-222, end of list lines 140-143 and 159-162)
- Pieces:
  1. Pressing `Escape` or `Shift+Escape` while the editor has focus closes the popup (`closeParameterHints`).
  2. The popup closes when the editor loses focus.
  3. A mouse click in the editor text closes the popup.
  4. The popup closes when the language returns no signature for the spot.
  5. With wrap around off (`editor.parameterHints.cycle`), going past the end of the list closes the popup.

#### AC-3.11 Turn automatic hints off

- Tag: both. Evidence: source. Boundary: no. Cursor: present (key found with default true)
- What: Switching `editor.parameterHints.enabled` off stops the popup from opening on its own when you type a trigger character and closes one that is open at that moment; Ctrl+Shift+Space still opens it by hand.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.parameterHints.enabled; editor.parameterHints
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (triggerOnType and cancel on change lines 310-316, onDidType guard lines 280-283); V\src\vs\editor\common\config\editorOptions.ts (lines 3680-3684); V\src\vs\editor\contrib\parameterHints\browser\parameterHints.ts (manual trigger lines 70-72, 89-95)
- Pieces:
  1. Turning `editor.parameterHints.enabled` off, default on, stops the popup from opening on its own when you type a trigger character.
  2. Turning it off while the popup is open closes the popup at that moment.
  3. With it off, `Ctrl+Shift+Space` still opens the popup by hand.

#### AC-3.12 Hints inside strings and comments

- Tag: both. Evidence: bundle. Boundary: no. Cursor: present (the TypeScript library shipped in the Cursor install has the rule; V's own TypeScript library is not in the V tree, so stock was read only through its provider, which passes the same typed-character reason)
- What: Typing a trigger character inside a string or a comment does not open the popup for TypeScript and JavaScript, because the language service returns nothing for a typed character there, and that rule is not applied to the Ctrl+Shift+Space trigger; the editor itself has no string or comment rule, so each language decides for itself.
- Commands: editor.action.triggerParameterHints
- Menus: none found
- Keys: Ctrl+Shift+Space (Cmd+Shift+Space on macOS) when editorTextFocus
- Settings: none found
- Source: C\extensions\node_modules\typescript\lib\typescript.js (lines 178240-178241 string and comment test for a typed character, lines 178255-178268 syntactic owner test); V\extensions\typescript-language-features\src\languageFeatures\signatureHelp.ts (toTsTriggerReason lines 103-122); V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (onDidType lines 280-294, no token type test)
- Pieces:
  1. Typing a trigger character inside a string or a comment does not open the popup for TypeScript and JavaScript.
  2. `Ctrl+Shift+Space` still opens the popup inside a string or a comment, because the rule applies only to a typed character (`editor.action.triggerParameterHints`).
  3. For other languages, whether a typed trigger character opens the popup in a string or a comment is left to that language, because the editor has no string or comment rule of its own.

#### AC-3.13 Screen reader announcement of the active parameter

- Tag: both. Evidence: source. Boundary: no. Cursor: present (message "{0}, hint" found in the NLS file and the announcedLabel check in the widget code)
- What: Screen readers are told the active parameter's label, then its documentation and the signature's documentation, followed by "hint", and only when the active parameter changes so typing does not repeat it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsWidget.ts (render lines 242-264, announcedLabel lines 52 and 168); C bundle (NLS "{0}, hint")
- Pieces:
  1. A screen reader is told the label of the active parameter, then its documentation and the documentation of the signature, followed by "hint".
  2. The announcement is made only when the active parameter changes, so typing inside the same parameter does not repeat it.


## AC-4 Snippets

(both). Tab stops, placeholders and choices, plus the insert and configure pickers. Examples: tab stops and placeholder highlight; the Insert Snippet picker; Configure Snippets; snippet code actions.

21 features.

#### AC-4.1 Jump between tab stops

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids, both keys and the context keys inSnippetMode, hasNextTabstop and hasPrevTabstop found; Cursor Tab's accept key is also bound to Tab at weight 500 against 130 here and applies when cpp.shouldAcceptTab is true, so it should win while a Cursor Tab suggestion shows, see AC-6, not run)
- What: After a snippet is inserted, Tab moves to the next tab stop and Shift+Tab to the previous one, with each stop selected (or given a cursor when it is empty); stops with the same number are linked and typed into together, and snippet mode ends when the last stop (`$0` or the highest number) is reached.
- Commands: jumpToNextSnippetPlaceholder; jumpToPrevSnippetPlaceholder
- Menus: none found
- Keys: Tab when inSnippetMode and hasNextTabstop and textInputFocus; Shift+Tab when inSnippetMode and hasPrevTabstop and textInputFocus
- Settings: none found
- Source: V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (context keys lines 59-61, _updateState lines 230-259, commands and keys lines 347-371); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (OneSnippet.move lines 93-190, next and prev lines 767-781); V\src\vs\editor\contrib\snippet\browser\snippet.md (Tabstops); C bundle (ids jumpToNextSnippetPlaceholder and jumpToPrevSnippetPlaceholder with Tab and Shift+Tab, weight 130; acceptCursorTabSuggestion with weight 500)
- Pieces:
  1. After a snippet is inserted, pressing Tab moves to the next tab stop and selects its placeholder text, or puts a bare cursor there when the stop is empty.
  2. Pressing Shift+Tab moves back to the previous tab stop in the same way.
  3. Tab stops that share a number are linked, so they are selected together and typing into one types into all of them.
  4. Reaching the last stop (`$0`, or the highest number when there is no `$0`) ends snippet mode, and Tab goes back to its normal job.
  5. Tab and Shift+Tab only jump while snippet mode is active and a next or previous stop exists (`inSnippetMode`, `hasNextTabstop`, `hasPrevTabstop`), so on the first stop Shift+Tab does not jump.

#### AC-4.2 Placeholder highlight

- Tag: both. Evidence: source. Boundary: no. Cursor: present (CSS classes snippet-placeholder and finish-snippet-placeholder with the four theme color ids found in the bundle and the CSS file)
- What: Every tab stop of the active snippet is outlined and tinted, and the final `$0` stop gets its own colors; the marks vanish when snippet mode ends, and only the current stop grows when you type at its edges. The colors are the theme colors editor.snippetTabstopHighlightBackground, editor.snippetTabstopHighlightBorder, editor.snippetFinalTabstopHighlightBackground and editor.snippetFinalTabstopHighlightBorder.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\snippet\browser\snippetSession.css (lines 6-20); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (decoration options lines 36-41, active and inactive switching lines 164-185); V\src\vs\platform\theme\common\colors\editorColors.ts (lines 248-263); C bundle (CSS .monaco-editor .snippet-placeholder; color ids)
- Pieces:
  1. While a snippet is active, each of its tab stops is outlined and tinted so you can see where Tab will go.
  2. The final `$0` stop is marked with its own background and border, different from the other stops.
  3. All the marks disappear when snippet mode ends.
  4. Typing at the edge of the current stop grows only that stop's mark, and the marks of the other stops keep their size.
  5. The theme colors `editor.snippetTabstopHighlightBackground` and `editor.snippetTabstopHighlightBorder` set the look of normal stops, and `editor.snippetFinalTabstopHighlightBackground` and `editor.snippetFinalTabstopHighlightBorder` set the look of the final stop.

#### AC-4.3 Placeholders with default text

- Tag: both. Evidence: source. Boundary: no. Cursor: present (same session code: transform on move, collapsed placeholder skip and nesting found in the bundle)
- What: A placeholder such as `${1:name}` inserts its default text selected so typing replaces it, placeholders can sit inside other placeholders, a placeholder you deleted is skipped on the next jump, and a placeholder can carry a transform that rewrites its text when you move off it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\snippet\browser\snippetParser.ts (Placeholder lines 228-283, Transform lines 318-366); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (transform on move lines 105-125, selecting the stop lines 155-165, skip of collapsed stops lines 126-190 and 192-209); V\src\vs\editor\contrib\snippet\browser\snippet.md (Placeholders); C bundle (placeholder decoration code)
- Pieces:
  1. A placeholder such as `${1:name}` inserts its default text already selected, so typing replaces it.
  2. A placeholder can contain other placeholders, and the outer one stays marked as active while an inner one is selected.
  3. If you delete a placeholder's default text, the next Tab or Shift+Tab jump skips that stop.
  4. A placeholder that has a transform gets its text rewritten when you move off it to another stop.

#### AC-4.4 Pick a choice from a dropdown

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the choice list provider snippetChoiceCompletions and the item command "Go to next placeholder..." are in the bundle; the context key inSnippetChoice and the Shift+Tab rule that uses it are not, so Shift+Tab in a choice list is probably claimed by the suggest list's alternative-accept key, inferred and not run)
- What: A choice placeholder such as `${1|one,two,three|}` opens a suggest-style list of its values as soon as the stop becomes active, in the order written; picking a value inserts it and jumps to the next stop, and Shift+Tab goes back to the previous stop even while the list is open.
- Commands: jumpToNextSnippetPlaceholder; jumpToPrevSnippetPlaceholder
- Menus: none found
- Keys: Shift+Tab when inSnippetMode and hasPrevTabstop and inSnippetChoice and textInputFocus (an extra rule at higher weight, so the list's own Shift+Tab does not win)
- Settings: none found
- Source: V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (inSnippetChoice line 62, choice provider and next-placeholder command lines 166-220, _handleChoice lines 261-288, Shift+Tab rule lines 370-378); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (activeChoice lines 274-304); V\src\vs\editor\contrib\snippet\browser\snippetParser.ts (Choice lines 285-316); C bundle (provider name, command title message 1539)
- Pieces:
  1. A choice placeholder such as `${1|one,two,three|}` opens a suggest-style list of its values as soon as that stop becomes active.
  2. The values are listed in the order they are written in the snippet.
  3. Picking a value inserts it and jumps straight to the next tab stop.
  4. Pressing Shift+Tab while the choice list is open goes back to the previous tab stop instead of acting on the list (`inSnippetChoice`). [stock only]
  5. In Cursor, Shift+Tab with the choice list open is handled by the suggest list's own keys instead of going back to the previous tab stop (inferred from the missing `inSnippetChoice` rule, not run). [cursor only]

#### AC-4.5 Variables in snippets

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the name list in the bundle has 35 of the 39 stock names; CURRENT_MILLISECOND, CURRENT_MILLISECONDS_UNIX, CURRENT_TIMEZONE_NAME and TM_DIRECTORY_BASE are absent)
- What: `$NAME` and `${NAME:default}` in a snippet are replaced at insert time with the file name or path, the selected text, the current line or word, the clipboard, the date and time, the workspace name, comment markers, a random number or a UUID, and a transform can reshape the value; an unknown name is inserted as plain text and turned into a placeholder.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\snippet\browser\snippetVariables.ts (KnownSnippetVariableNames lines 20-60, resolvers from line 62); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (resolver chain lines 566-575); V\src\vs\workbench\contrib\snippets\browser\snippetsFile.ts (unknown name becomes a placeholder lines 62-76); V\src\vs\editor\contrib\snippet\browser\snippet.md (Variables); C bundle (name table ending UUID:!0, four names searched with 0 hits)
- Pieces:
  1. A variable such as `$TM_FILENAME` or `${TM_FILENAME:default}` in a snippet is replaced with its value at the moment the snippet is inserted.
  2. File and workspace variables insert the file name, the name without its extension, the folder, the full path, the path relative to the workspace, the workspace name and the workspace folder (`TM_FILENAME`, `TM_FILENAME_BASE`, `TM_DIRECTORY`, `TM_FILEPATH`, `RELATIVE_FILEPATH`, `WORKSPACE_NAME`, `WORKSPACE_FOLDER`).
  3. Editor variables insert the selected text, the current line, the current word, the line index or number, the cursor index or number and the clipboard text (`TM_SELECTED_TEXT`, `SELECTION`, `TM_CURRENT_LINE`, `TM_CURRENT_WORD`, `TM_LINE_INDEX`, `TM_LINE_NUMBER`, `CURSOR_INDEX`, `CURSOR_NUMBER`, `CLIPBOARD`).
  4. Date and time variables insert the current year, month, date, hour, minute, second, day name, month name, Unix seconds and time zone offset (`CURRENT_YEAR`, `CURRENT_YEAR_SHORT`, `CURRENT_MONTH`, `CURRENT_DATE`, `CURRENT_HOUR`, `CURRENT_MINUTE`, `CURRENT_SECOND`, `CURRENT_DAY_NAME`, `CURRENT_DAY_NAME_SHORT`, `CURRENT_MONTH_NAME`, `CURRENT_MONTH_NAME_SHORT`, `CURRENT_SECONDS_UNIX`, `CURRENT_TIMEZONE_OFFSET`).
  5. Comment variables insert the comment markers of the language at the cursor (`LINE_COMMENT`, `BLOCK_COMMENT_START`, `BLOCK_COMMENT_END`).
  6. Random variables insert a random number, a random hex string or a new UUID each time (`RANDOM`, `RANDOM_HEX`, `UUID`).
  7. A transform on a variable, such as `${TM_FILENAME/(.*)\..+$/$1/}`, reshapes the value before it is inserted.
  8. A name that is not a known variable and has no default, such as `${foo}`, is inserted as plain text and turned into a placeholder you can Tab to.
  9. Cursor's list of known variable names lacks `CURRENT_MILLISECOND`, `CURRENT_MILLISECONDS_UNIX`, `CURRENT_TIMEZONE_NAME` and `TM_DIRECTORY_BASE`, which stock VS Code knows. [cursor only]

#### AC-4.6 Nested snippets

- Tag: both. Evidence: source. Boundary: no. Cursor: present (merge path and nesting level code found in the bundle)
- What: Inserting a snippet while another one is active (for example from the suggest list or the Insert Snippet picker while a placeholder is selected) places the new snippet's tab stops inside the current stop, and Tab walks through them before continuing with the outer snippet's remaining stops.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (merge branch lines 132-162); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (OneSnippet.merge lines 306-360, SnippetSession.merge lines 728-765); C bundle (session merge code)
- Pieces:
  1. Inserting a snippet while another snippet is active places the new snippet's tab stops inside the current stop instead of ending the first snippet.
  2. This happens when you pick a snippet from the suggest list or from the Insert Snippet picker while a placeholder is selected.
  3. Pressing Tab walks through the inner snippet's stops first, then continues with the outer snippet's remaining stops.

#### AC-4.7 Leave a snippet early

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids, the Esc keys and the cancel with selection reset found)
- What: Esc or Shift+Esc ends snippet mode on the spot and collapses the selection to the main cursor; snippet mode also ends by itself when the cursor leaves every tab stop or when undo goes back before the insert, and the command Accept Snippet (no key, not in the palette) runs through all remaining stops to the end.
- Commands: leaveSnippet; acceptSnippet
- Menus: none found
- Keys: Escape; Shift+Escape when inSnippetMode and textInputFocus (leaveSnippet); none found (acceptSnippet)
- Settings: none found
- Source: V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (_updateState lines 230-259, finish and cancel lines 290-318, commands lines 380-401); C bundle (ids leaveSnippet and acceptSnippet)
- Pieces:
  1. Pressing Esc or Shift+Esc while snippet mode is active ends it on the spot and collapses the selection to the main cursor (`leaveSnippet`).
  2. Snippet mode ends by itself when the cursor moves out of every tab stop.
  3. Snippet mode ends by itself when undo goes back to before the snippet was inserted.
  4. The command `acceptSnippet` runs through all the remaining stops and ends snippet mode at the end of the snippet, and it has no default key and no Command Palette entry.

#### AC-4.8 Insert at every cursor with matching indentation

- Tag: both. Evidence: source. Boundary: no. Cursor: present (adjustWhitespace, per-selection edit building and the overwriteBefore and overwriteAfter options found in the bundle)
- What: With several cursors a snippet is inserted at each one, linked tab stops are selected at all cursors together, and every extra line takes the indentation of the line it lands on (normalized to the file's spaces or tabs).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (adjustWhitespace lines 427-478, per-selection insert lines 505-587, moving all snippets lines 783-790); V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (default options lines 41-50); C bundle (default option object with adjustWhitespace:!0)
- Pieces:
  1. With several cursors, inserting a snippet puts it at every cursor.
  2. Linked tab stops are selected at all cursors together when you jump with Tab.
  3. Every extra line of the snippet takes the indentation of the line it lands on, at each cursor.
  4. The indentation of the inserted lines is converted to the spaces or tabs the file uses.

#### AC-4.9 Tab completion of snippet prefixes

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id insertSnippet on Tab with weight 100, context key hasSnippetCompletions and the setting with default off found)
- What: With `editor.tabCompletion` set to onlySnippets, typing a snippet prefix and pressing Tab inserts the snippet with no list; when several snippets share the prefix a small list of them opens, and a selection that equals a prefix counts as well. The setting is off by default.
- Commands: insertSnippet; editor.tabCompletionController
- Menus: none found
- Keys: Tab when editorTextFocus and tabDoesNotMoveFocus and not inSnippetMode and hasSnippetCompletions
- Settings: editor.tabCompletion
- Source: V\src\vs\workbench\contrib\snippets\browser\tabCompletion.ts (context key line 31, _update and _updateSnippets lines 66-155, performSnippetCompletions lines 157-188, command and key lines 194-206); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (getNonWhitespacePrefix lines 512-535); V\src\vs\editor\common\config\editorOptions.ts (tabCompletion lines 6708-6720); C bundle (id insertSnippet, hasSnippetCompletions, tabCompletion default "off")
- Pieces:
  1. With `editor.tabCompletion` set to `onlySnippets`, typing a snippet prefix and pressing Tab inserts that snippet with no list.
  2. When several snippets share the typed prefix, pressing Tab opens a small list of them to pick from.
  3. Selected text that equals a snippet prefix also counts, so pressing Tab on that selection inserts the snippet.
  4. Setting `editor.tabCompletion` chooses how Tab completes, with values `on`, `off` and `onlySnippets`, and the default is `off`.
  5. When the word before the cursor matches no snippet prefix, or a snippet is already active, Tab does its usual job instead.

#### AC-4.10 Snippets in the suggest list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (provider name snippetCompletions, option snippetSuggestions with default inline, setting showSnippets, the markAsUsed command and the "{0}, {1}" label message found)
- What: Snippets whose prefix matches what you type (matched from any word start or symbol, not only the start of the line) show as rows with the snippet icon, the prefix as the label, the snippet name as the description and "{description or name} ({source})" as the detail, plus a code preview of the body in the details pane; rows with the same prefix get ", {name}" added, and your own snippets sort before extension snippets.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.snippetSuggestions; editor.suggest.showSnippets
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetCompletionProvider.ts (row fields lines 34-71, matching and disambiguation lines 90-200); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (provider registration line 235); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (sort order lines 560-575); V\src\vs\editor\contrib\suggest\browser\completionModel.ts (lines 59-75); V\src\vs\editor\common\config\editorOptions.ts (lines 6654-6667); C bundle (class with label and detail fields, _snippet.markAsUsed)
- Pieces:
  1. A snippet whose prefix matches what you type shows as a row in the suggest list with the snippet icon and the prefix as its label.
  2. The row shows the snippet name as its description and "{description or name} ({source})" as its detail text.
  3. The details pane shows a code preview of the snippet body for the focused snippet row.
  4. A snippet matches from any word start or symbol in what you type, not only from the start of the line.
  5. Rows that share the same prefix get ", {name}" added to the label so they can be told apart.
  6. Your own snippets are sorted before snippets from extensions.
  7. Setting `editor.snippetSuggestions` chooses where snippet rows sit in the list, with values `top`, `bottom`, `inline` and `none`, and the default is `inline`.
  8. Setting `editor.suggest.showSnippets` turns snippet rows in the suggest list on or off.

#### AC-4.11 Insert Snippet picker

- Tag: both. Evidence: source. Boundary: no. Cursor: present (ids and alias, labels "Insert Snippet", "Select a snippet", "User Snippets", "Workspace Snippets" and "No snippet available" found)
- What: Insert Snippet from the Command Palette opens a picker of the snippets for the language at the cursor, grouped under User Snippets, Workspace Snippets and each extension's name, each row showing the prefix (or name) with the description or body beneath, filterable on both, recently used first within a group; the pick is inserted as a snippet, and a keybinding can pass `snippet`, `name` or `langId` to skip the picker or choose another language.
- Commands: editor.action.insertSnippet; editor.action.showSnippets
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\insertSnippet.ts (action lines 47-150, args lines 19-45); V\src\vs\workbench\contrib\snippets\browser\snippetPicker.ts (lines 17-103); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (recency sort lines 331-345); V\src\vs\workbench\contrib\snippets\browser\snippets.contribution.ts (alias lines 26-27); C bundle (ids, NLS labels)
- Pieces:
  1. Running Insert Snippet from the Command Palette (also reachable as `editor.action.showSnippets`) opens a picker with the placeholder "Select a snippet" that lists the snippets for the language at the cursor.
  2. The rows are grouped under separators named "User Snippets", "Workspace Snippets" and the name of each extension that contributes snippets.
  3. Each row shows the prefix as its label, or the snippet name when there is no prefix, with the description beneath it, or the body when there is no description.
  4. Typing in the picker filters on both the label and the text beneath it.
  5. Within a group, recently used snippets are listed first.
  6. Picking a row inserts the snippet at the cursor and starts snippet mode.
  7. When the language has no snippets, the picker shows the message "No snippet available".
  8. A keybinding that passes `snippet` to `editor.action.insertSnippet` inserts that text as a snippet, and one that passes `name` inserts the snippet with that name, both without opening the picker.
  9. A keybinding that passes `langId` to `editor.action.insertSnippet` opens the picker for that language instead of the language at the cursor.
  10. Insert Snippet is unavailable in a read-only editor.

#### AC-4.12 Hide an extension snippet from IntelliSense

- Tag: both. Evidence: source. Boundary: no. Cursor: present (labels "Hide from IntelliSense", "Show in IntelliSense", "(hidden from IntelliSense)" and the storage key snippets.ignoredSnippets found; the key is stored state, not a setting)
- What: In the Insert Snippet and Surround pickers every extension snippet has an eye-closed button "Hide from IntelliSense" that removes it from the suggest list and tags the row "(hidden from IntelliSense)", and the button then becomes an eye "Show in IntelliSense"; the choice is kept per profile.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetPicker.ts (buttons lines 56-96); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (SnippetEnablement lines 132-168, isEnabled and updateEnablement lines 245-251); C bundle (NLS labels, snippets.ignoredSnippets)
- Pieces:
  1. In the Insert Snippet and Surround with Snippet pickers, every snippet from an extension has an eye-closed button with the tooltip "Hide from IntelliSense".
  2. Clicking that button removes the snippet from the suggest list, and the picker refreshes in place.
  3. A hidden snippet stays in the picker with "(hidden from IntelliSense)" on its row, and its button becomes an eye with the tooltip "Show in IntelliSense".
  4. Clicking the eye button brings the snippet back into the suggest list.
  5. Rows for your own and workspace snippets have no such button.
  6. The hidden or shown choice is kept per profile, so it is still there after a restart.

#### AC-4.13 Surround with Snippet

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id and label "Surround with Snippet..." found)
- What: With text selected, Surround with Snippet... in the Command Palette opens the picker limited to snippets whose body uses the selection (`$TM_SELECTED_TEXT` or `$SELECTION`) and wraps the selected text with the pick; it does nothing when no snippet qualifies.
- Commands: editor.action.surroundWithSnippet
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\surroundWithSnippet.ts (lines 20-72); V\src\vs\workbench\contrib\snippets\browser\snippetsFile.ts (usesSelection lines 82-86 and 142-144); C bundle (id editor.action.surroundWithSnippet with title message 9753)
- Pieces:
  1. With text selected, running Surround with Snippet... from the Command Palette opens a picker with the placeholder "Select a snippet".
  2. The picker lists only snippets whose body uses the selection (`$TM_SELECTED_TEXT` or `$SELECTION`).
  3. Picking a snippet wraps the selected text with it and starts snippet mode.
  4. When no snippet qualifies, the command does nothing and no picker opens.
  5. The command is unavailable in a read-only editor and when nothing is selected.

#### AC-4.14 Fill File with Snippet

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id and label "Fill File with Snippet" found)
- What: Fill File with Snippet in the Command Palette lists the file-template snippets (those marked `isFileTemplate`) that match the open file, grouped by language, and replaces the whole file with the pick after switching the file to that language.
- Commands: workbench.action.populateFileFromSnippet
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\fileTemplateSnippets.ts (lines 19-112); C bundle (id workbench.action.populateFileFromSnippet, NLS label)
- Pieces:
  1. Running Fill File with Snippet from the Command Palette opens a picker with the placeholder "Select a snippet" that lists the file-template snippets (marked `isFileTemplate`) that match the open file.
  2. The rows are grouped under a separator for each language, and each row shows the prefix or name with the description beneath it.
  3. Picking a snippet switches the file to that snippet's language and replaces the whole file with the snippet.
  4. When no file-template snippet matches the open file, the command does nothing and no picker opens.

#### AC-4.15 Surround With snippets in the light bulb

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the setting, the provider limit constant _MAX_CODE_ACTIONS, the overflow row and the label "More..." found; the group header title found in NLS)
- What: With text selected, the light bulb menu lists up to four matching snippets by name under a "Surround With" header (when headers are shown) plus a "More..." row that opens Surround with Snippet...; choosing a snippet wraps the selection. Turning `editor.snippets.codeActions.enabled` off removes these rows.
- Commands: editor.action.surroundWithSnippet
- Menus: none found
- Keys: none found
- Settings: editor.snippets.codeActions.enabled
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetCodeActionProvider.ts (SurroundWithSnippetCodeActionProvider lines 22-67, enable switch lines 125-150); V\src\vs\workbench\contrib\snippets\browser\snippets.contribution.ts (setting lines 36-48); V\src\vs\editor\contrib\codeAction\browser\codeActionMenu.ts (group title line 31); C bundle (editor.snippets.codeActions.enabled, _MAX_CODE_ACTIONS, NLS "Surround With")
- Pieces:
  1. With text selected, the light bulb menu lists snippets that can wrap the selection, each row showing the snippet name.
  2. At most four snippet rows show, and a "More..." row follows them when more than four match.
  3. Choosing the "More..." row opens the Surround with Snippet... picker.
  4. The rows sit under a "Surround With" group header when the menu shows group headers.
  5. Choosing a snippet row wraps the selection with that snippet straight away, without opening a picker.
  6. Setting `editor.snippets.codeActions.enabled` set to false removes these rows from the light bulb menu, and its default is true.

#### AC-4.16 Start with a snippet in an empty file

- Tag: both. Evidence: source. Boundary: no. Cursor: present (labels "Start with: " and "Start with Snippet" found in NLS, same provider code and setting)
- What: In an empty file the light bulb offers "Start with: {snippet name}" for up to four file-template snippets of that language (under the same "Surround With" header) plus a "Start with Snippet" row that opens Fill File with Snippet; choosing one fills the file.
- Commands: workbench.action.populateFileFromSnippet
- Menus: none found
- Keys: none found
- Settings: editor.snippets.codeActions.enabled
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetCodeActionProvider.ts (FileTemplateCodeActionProvider lines 69-123); C bundle (NLS labels, provider code with the same limit and overflow row)
- Pieces:
  1. In an empty file, the light bulb offers a "Start with: {snippet name}" row for each file-template snippet of the file's language.
  2. At most four such rows show, and a "Start with Snippet" row follows them when more match.
  3. Choosing the "Start with Snippet" row opens the Fill File with Snippet picker.
  4. The rows sit under the same "Surround With" group header as the surround rows.
  5. Choosing a "Start with: ..." row fills the file with that snippet.
  6. Setting `editor.snippets.codeActions.enabled` set to false removes these rows from the light bulb menu.
  7. In a file that already has content, these rows never show.

#### AC-4.17 Configure Snippets picker

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, short title "Snippets", both menu registrations and the picker labels found; in Cursor's default layout the Manage gear is not drawn, so the menu bar and palette are the routes, see the Cursor activity bar notes and AB-4.18)
- What: Configure Snippets (File > Preferences > Snippets, the Manage gear's Snippets row, or the Command Palette) opens a picker with "Existing Snippets" (your snippet files, each with the languages it covers, `.code-snippets` files first) and "New Snippets" (New Global Snippets file..., one New Snippets file for '{folder}'... per workspace folder, and every language that has no file yet); picking an existing file opens it in an editor tab.
- Commands: workbench.action.openSnippets
- Menus: MenubarPreferencesMenu (group 2_configuration, order 5); GlobalActivity (group 2_configuration, order 5); CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\configureSnippets.ts (computePicks lines 37-142, action and menus lines 245-262, pick list lines 264-314); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (snippet files lines 263-266); C bundle (id workbench.action.openSnippets with title message 9748, NLS "Select Snippets File or Create Snippets", "Existing Snippets", "New Snippets")
- Pieces:
  1. Configure Snippets opens a picker with the placeholder "Select Snippets File or Create Snippets".
  2. The picker opens from File > Preferences > Snippets, from the Snippets row in the Manage gear menu, and from Configure Snippets in the Command Palette.
  3. The "Existing Snippets" group lists your own snippet files, with `.code-snippets` files first and the rest sorted by name.
  4. Each existing row shows the file name with a description such as "(global)", up to four language names with "..." after them, or "(TypeScript)" for a language file.
  5. The "New Snippets" group lists "New Global Snippets file...", one "New Snippets file for '{folder}'..." row for each workspace folder, and every language that has no file yet.
  6. Picking an existing file opens it in an editor tab.
  7. Typing in the picker filters on both the file name and its description.
  8. When you have no snippet files yet, the picker shows only the "New Snippets" group.

#### AC-4.18 Create a snippets file

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (name prompt, validation messages and both templates found; the paragraph and example about restricting snippets with include and exclude patterns is not in the Cursor templates)
- What: Picking New Global Snippets file... or New Snippets file for '{folder}'... asks for a name ("Type snippet file name", rejecting empty, invalid and existing names), creates a `.code-snippets` file with a commented template in the user snippets folder or the folder's `.vscode` folder and opens it; picking a language with no file yet writes `<language>.json` in the user snippets folder with a language template and opens it.
- Commands: workbench.action.openSnippets
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\configureSnippets.ts (createSnippetFile lines 144-210, createLanguageSnippetFile lines 212-243, create branches lines 273-314); C bundle (template text starting "Place your", NLS "Type snippet file name", "is not a valid file name"; search for "include/exclude patterns" returned 0 hits)
- Pieces:
  1. Picking "New Global Snippets file..." asks for a file name in an input with the placeholder "Type snippet file name", and the new file goes in the user snippets folder.
  2. Picking "New Snippets file for '{folder}'..." asks for a name the same way, and the new file goes in that folder's `.vscode` folder.
  3. The name input rejects an empty name with "Invalid file name", an invalid name with "'{name}' is not a valid file name" and an existing name with "'{name}' already exists".
  4. After a valid name, a `.code-snippets` file (the extension is added when missing) is created with a commented template that explains scope, prefix, body and description and shows an example, and it opens in an editor tab.
  5. Picking a language that has no file yet writes `<language>.json` in the user snippets folder with a commented template for that language and opens it.
  6. The template has no paragraph or example about restricting snippets with `include` and `exclude` patterns, where stock VS Code includes one. [cursor only]

#### AC-4.19 Edit snippet files in the editor

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (language id snippets, file match rules, both schema ids, the [snippets] defaults and the reload on save found; the schema has the properties prefix, isFileTemplate, body and description but not include or exclude)
- What: Snippet files open as normal editor tabs in the "Code Snippets" language (`.code-snippets` files and the JSON files in the user snippets folder), which is JSON with comments, with property completion and error squiggles from a schema, an "Empty snippet" skeleton in the completion list, suggestions that also open inside strings, and snippets that reload as soon as the file is saved.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: [snippets] editor.quickSuggestions; [snippets] editor.suggest.insertMode
- Source: V\extensions\json\package.json (language lines 91-101, grammar lines 123-126); V\extensions\configuration-editing\package.json (schema matches lines 143-162); V\extensions\json-language-features\package.json (language defaults lines 164-169); V\src\vs\workbench\contrib\snippets\browser\snippets.contribution.ts (schemas lines 52-136); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (folder watch and reload lines 451-502); C bundle (schema object with prefix, isFileTemplate, body, description only)
- Pieces:
  1. Snippet files (`.code-snippets` files and the JSON files in the user snippets folder) open as normal editor tabs in the "Code Snippets" language, which is JSON with comments.
  2. Typing in a snippet file offers property completions with descriptions from a schema (`prefix`, `body`, `description`, `isFileTemplate`, and `scope` in `.code-snippets` files).
  3. A snippet entry that breaks the schema, such as one with no `body` or with an unknown property, shows an error squiggle.
  4. The completion list includes an "Empty snippet" skeleton that inserts a whole snippet entry with tab stops for its name, prefix, body and description.
  5. Setting `[snippets]` `editor.quickSuggestions` is `{ "strings": true }`, so suggestions also open while you type inside strings.
  6. Setting `[snippets]` `editor.suggest.insertMode` is `replace`, so accepting a suggestion replaces the word to the right of the cursor.
  7. Saving a snippet file reloads its snippets at once, so edits show in the suggest list and the pickers without a restart.
  8. In Cursor, the snippet file schema offers `prefix`, `isFileTemplate`, `body` and `description` but not `include` or `exclude`. [cursor only]

#### AC-4.20 Choose which languages and prefixes a snippet answers to

- Tag: both. Evidence: source. Boundary: no. Cursor: present (isGlobalSnippets test on .code-snippets, scope select code, includeNoPrefixSnippets option and the scope schema text found)
- What: A snippet in `typescript.json` applies to TypeScript only, one in a `.code-snippets` file applies to the languages in its `scope` field (to every language when `scope` is empty), and one in a workspace folder's `.vscode` folder shows only in that workspace under "Workspace Snippets"; `prefix` may be one string or a list, and a snippet with no prefix never appears in the suggest list and is reached only from the pickers and the light bulb.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetsFile.ts (file kinds lines 194-258, scopes and prefixes lines 296-371); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (workspace and user folders lines 420-502, no-prefix filter lines 311-315); V\src\vs\workbench\contrib\snippets\browser\snippets.contribution.ts (scope lines 109-131); C bundle (class fields isFileTemplate, scopes, name, prefix; select method)
- Pieces:
  1. A snippet in a language file such as `typescript.json` in the user snippets folder applies to that language only.
  2. A snippet in a `.code-snippets` file applies to the languages named in its `scope` field, and to every language when `scope` is empty or missing.
  3. A snippet in a workspace folder's `.vscode` folder shows only in that workspace, under "Workspace Snippets" in the pickers.
  4. `prefix` can be one string or a list of strings, and each prefix in the list brings up the snippet.
  5. A snippet with no `prefix` never appears in the suggest list and is reached only from the pickers and the light bulb.

#### AC-4.21 Limit a snippet to certain files

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (the Snippet class in the bundle has no include or exclude fields, the schema has no such properties, and the NLS text "to include the snippet for specific files" has 0 hits)
- What: A snippet can carry `include` and `exclude` lists of globs so it only shows for matching files, with exclude winning; a pattern with a slash is matched against the absolute path and any other pattern against the file name, ignoring case.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetsFile.ts (fields lines 118-119, isFileIncluded lines 146-170, parsing lines 323-340); V\src\vs\workbench\contrib\snippets\browser\snippets.contribution.ts (schema lines 74-88); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (filter lines 322-325); C bundle (Snippet constructor field list, schema object)
- Pieces:
  1. A snippet with an `include` list of globs shows only for files that match at least one of the globs. [stock only]
  2. A snippet with an `exclude` list of globs is hidden for files that match any of the globs, and exclude wins when a file matches both lists. [stock only]
  3. A pattern that contains a slash is matched against the absolute path of the file. [stock only]
  4. A pattern with no slash is matched against the file name only. [stock only]
  5. Matching ignores letter case. [stock only]
  6. Snippet files offer `include` and `exclude` as properties in completion, with their descriptions, and do not mark them as errors. [stock only]


## AC-5 Terminal suggestions

(both). The suggest popup inside the terminal with its own status menu. Examples: the popup with symbol icons; the widget status menu; details toggle; Tab and Enter accept.

25 features.

#### AC-5.1 Turn terminal suggestions on or off

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (setting present but default false and tagged preview; text lists PowerShell v7+, zsh, bash, fish and says shell integration must be true)
- What: Switch the terminal suggest popup on or off with one setting; it needs shell integration and only works in supported shells (Windows PowerShell, PowerShell 7+, zsh, bash, fish).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.enabled; terminal.integrated.shellIntegration.enabled
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; C bundle (module terminalSuggestConfiguration.js, nls 10951)
- Pieces:
  1. Setting `terminal.integrated.suggest.enabled` turns the terminal suggest popup on or off.
  2. The popup needs shell integration turned on and working (`terminal.integrated.shellIntegration.enabled`), and stays closed without it.
  3. The popup only works in supported shells: PowerShell 7+, zsh, bash and fish (stock VS Code also lists Windows PowerShell).
  4. The setting is on by default, so the popup works without setup. [stock only]
  5. The setting is off by default and tagged preview, so it has to be turned on before the popup shows. [cursor only]

#### AC-5.2 See the suggest popup at the prompt cursor

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (widget with the same 430 px layout and the two message strings is present; the first row is always focused on show, and no explicit-invoke context key or loading trigger was found)
- What: A list opens next to the prompt cursor (below it, or above when there is no room, and flipped left near the right edge), 430 px wide and up to 12 rows tall by default; each row has a kind icon, the label with matched letters in bold, a faded detail text, and a right side description; it also shows in terminals in the editor area, and shows "Loading..." and "No suggestions." messages when opened by hand.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.fontFamily; terminal.integrated.fontSize; terminal.integrated.lineHeight; editor.fontWeight
- Source: V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts; V\src\vs\workbench\services\suggest\browser\simpleSuggestWidgetRenderer.ts; V\src\vs\workbench\services\suggest\browser\media\suggest.css; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts; C bundle (SimpleSuggestWidget class, nls 13099 to 13102)
- Pieces:
  1. A list opens next to the prompt cursor, below it, or above it when there is no room below.
  2. Near the right edge of the terminal the list moves left so it stays in view.
  3. The list is 430 px wide and up to 12 rows tall by default.
  4. Each row shows a kind icon, the label with the typed letters in bold, a faded detail text, and a description on the right side.
  5. The list also shows in terminals that live in the editor area, not only in the panel.
  6. When opened by hand and nothing matches, the list shows the message "No suggestions."
  7. When opened by hand, the list shows the message "Loading..." until results arrive. [stock only]
  8. Settings `terminal.integrated.fontFamily`, `terminal.integrated.fontSize`, `terminal.integrated.lineHeight` and `editor.fontWeight` set the font used in the rows.
  9. In Cursor the first row is always focused when the list shows. [cursor only]

#### AC-5.3 Resize the popup and reset its size

- Tag: both. Evidence: source. Boundary: no. Cursor: present (command id and title "Reset Suggest Widget Size" found; storage key terminal.integrated.suggestSize found)
- What: Drag the popup edges to resize it, the size is remembered across sessions, and a palette command puts it back to the default.
- Commands: workbench.action.terminal.resetSuggestWidgetSize
- Menus: CommandPalette (f1 defaults to true, title shows as Terminal: Reset Suggest Widget Size)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (PersistedWidgetSize); V\src\vs\workbench\contrib\terminal\common\terminalStorageKeys.ts; C bundle (module terminal.suggest.js region, nls 10911)
- Pieces:
  1. Dragging the edges of the popup resizes it.
  2. The size is remembered across sessions, so the next popup opens at the size you left it.
  3. The Command Palette entry "Terminal: Reset Suggest Widget Size" (`workbench.action.terminal.resetSuggestWidgetSize`) puts the popup back to its default size.

#### AC-5.4 Kind icons on every suggestion

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (9 icons found: alias, argument, file, flag, folder, inline-suggestion, method, option, option-value, with matching colors; no symbolic link, commit, branch, tag, stash, remote, pull request or plain text icons; the showIcons setting is read by the terminal widget)
- What: Each row gets a colored symbol icon for its kind (file, folder, symbolic link file and folder, method for commands, alias, argument, option, option value, flag, commit, branch, tag, stash, remote, pull request, completed pull request, inline suggestion star, and a key icon for plain text); file and folder rows switch to file-icon-theme icons when the active theme has them; turning icons off hides them; each icon has a `terminalSymbolIcon.*Foreground` theme color.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.showIcons
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSymbolIcons.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\media\terminalSymbolIcons.css; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (kind to icon map); V\src\vs\workbench\services\suggest\browser\simpleSuggestWidgetRenderer.ts; C bundle (CSS codicon-terminal-symbol-*, nls 10933 to 10950)
- Pieces:
  1. Each row has a colored kind icon for the common kinds: file, folder, method (commands), alias, argument, option, option value, flag, and the inline suggestion star.
  2. File and folder rows switch to icons from the file icon theme when the active theme has them.
  3. Setting `editor.suggest.showIcons` set to false hides the icons on the rows.
  4. Each icon has its own theme color named `terminalSymbolIcon.*Foreground`, so a theme can recolor it.
  5. Symbolic link file and symbolic link folder rows have their own icons. [stock only]
  6. Commit, branch, tag, stash, remote, pull request and completed pull request rows have their own icons. [stock only]
  7. Plain text rows get a key icon. [stock only]

#### AC-5.5 Trigger suggestions by hand

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the id workbench.action.terminal.requestCompletions, title "Request Completions", is registered with the same Ctrl+Space key and the same when clause; triggerSuggest and "Trigger Suggest" for the terminal not found; in PowerShell with shell integration the Ctrl+Space key also sends the sequence ESC [24~e to the shell)
- What: Press Ctrl+Space in the terminal to open the popup on demand; a manual open selects the first row, can fall back to files and folders, and shows "No suggestions." when empty; the same key toggles the details pane once the popup is open.
- Commands: workbench.action.terminal.triggerSuggest
- Menus: none found
- Keys: Ctrl+Space (mac: Ctrl+Space) when terminalFocus and terminalSuggestWidgetVisible is false and config.terminal.integrated.suggest.enabled is true
- Settings: terminal.integrated.suggest.enabled
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminal.suggest.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (requestCompletions); C bundle (module terminal.suggest.js region, nls 10910)
- Pieces:
  1. Pressing Ctrl+Space in the terminal opens the suggest popup on demand when `terminal.integrated.suggest.enabled` is on and the popup is not already showing.
  2. A manual open focuses the first row.
  3. A manual open can fall back to files and folders when nothing else fits.
  4. When a manual open finds nothing, the popup shows "No suggestions."
  5. Pressing Ctrl+Space again while the popup is open toggles the details pane.
  6. In Cursor the command is `workbench.action.terminal.requestCompletions` with the title "Request Completions", bound to the same Ctrl+Space key and condition, in place of `workbench.action.terminal.triggerSuggest`. [cursor only]

#### AC-5.6 Show suggestions while typing

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (both settings present; defaults are commands on, arguments on, unknown off, and trigger characters true, where V defaults all off)
- What: The popup opens by itself as you type, for the first word (commands), later words (arguments) or unclear spots (unknown, with files and folders as a fallback), and on trigger characters (a dash after a space, a slash after a folder, and any provider trigger character).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.quickSuggestions; terminal.integrated.suggest.suggestOnTriggerCharacters
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (_sync); C bundle (module terminalSuggestConfiguration.js, nls 10953 to 10957)
- Pieces:
  1. With the `commands` key of `terminal.integrated.suggest.quickSuggestions` set to `on`, typing the first word of a command line opens the popup by itself with commands.
  2. With the `arguments` key set to `on`, typing any later word opens the popup by itself with arguments.
  3. With the `unknown` key set to `on`, typing where it is unclear what fits opens the popup by itself, with files and folders as a fallback.
  4. With `terminal.integrated.suggest.suggestOnTriggerCharacters` on, typing a trigger character opens the popup: a dash after a space, a slash after a folder, or any character a provider asks for.
  5. By default none of these open the popup by itself, so typing never shows it until a setting is turned on. [stock only]
  6. By default `commands` and `arguments` are `on`, `unknown` is `off` and `suggestOnTriggerCharacters` is true. [cursor only]

#### AC-5.7 Move through the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all five ids found with the same keys and Up conditions; setting present)
- What: Down and Up move one row and wrap around at the ends, PageDown and PageUp move a page; Up on the first row, before you have pressed Down, closes the popup and sends Up to the shell so you walk command history (unless the history setting is off, then Up goes to the last row).
- Commands: workbench.action.terminal.selectNextSuggestion; workbench.action.terminal.selectPrevSuggestion; workbench.action.terminal.selectNextPageSuggestion; workbench.action.terminal.selectPrevPageSuggestion; workbench.action.terminal.hideSuggestWidgetAndNavigateHistory
- Menus: none found
- Keys: Down; PageDown; PageUp; Up when simpleSuggestWidgetHasNavigated or upArrowNavigatesHistory is false (previous row); Up when simpleSuggestWidgetHasNavigated is false and upArrowNavigatesHistory is true (history); all need terminalFocus and terminalSuggestWidgetVisible
- Settings: terminal.integrated.suggest.upArrowNavigatesHistory
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts (selectNext, selectPrevious); C bundle (module terminal.suggest.js region, nls 10912 to 10922)
- Pieces:
  1. Pressing Down moves to the next row and wraps from the last row to the first.
  2. Pressing Up moves to the previous row and wraps from the first row to the last once you have pressed Down.
  3. PageDown and PageUp move the focus by one page of rows.
  4. Pressing Up on the first row before you have pressed Down closes the popup and sends Up to the shell, so you walk through command history.
  5. Setting `terminal.integrated.suggest.upArrowNavigatesHistory` set to false makes Up on the first row move to the last row instead and keeps the popup open, and its default is true.
  6. While the popup is closed, Down, Up, PageDown and PageUp go to the shell as usual.

#### AC-5.8 Accept a suggestion with Tab or a click

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (id and title "Insert" found with Tab and no focused-row condition; also placed on the popup status row as the left item "Insert (Tab)" through menu MenubarTerminalSuggestStatusMenu)
- What: Tab inserts the focused row, replacing the typed piece of the word (and the rest of the word to the right), then closes the popup; clicking a row does the same without taking focus from the terminal.
- Commands: workbench.action.terminal.acceptSelectedSuggestion
- Menus: none found
- Keys: Tab when simpleSuggestWidgetHasFocusedSuggestion, terminalFocus and terminalSuggestWidgetVisible
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (acceptSelectedSuggestion); V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts (_onListMouseDownOrTap); C bundle (module terminal.suggest.js region, nls 10919)
- Pieces:
  1. Pressing Tab inserts the focused row into the command line in place of the part of the word you typed, then closes the popup.
  2. The rest of the word to the right of the cursor is replaced too.
  3. Clicking a row inserts it the same way and keeps keyboard focus in the terminal.
  4. When the popup is open with no row focused, Tab goes to the shell instead of accepting. [stock only]
  5. In Cursor the accept command is titled "Insert", and an "Insert (Tab)" button at the left of the popup's status row accepts the focused row. [cursor only]

#### AC-5.9 Accept with Enter and optionally run the command

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (acceptSelectedSuggestionEnter and runOnEnter present with the same four run modes; the setting has an extra value "ignore", which is the default, and the Enter key rule is "runOnEnter is not ignore", so by default Enter goes straight to the shell; no Enter key on acceptSelectedSuggestion)
- What: Enter inserts the focused row when it is allowed to (see selection mode), and the run-on-Enter setting decides whether Enter is also sent so the command runs (never, only when typed in full, same ignoring file extension, or always).
- Commands: workbench.action.terminal.acceptSelectedSuggestion; workbench.action.terminal.acceptSelectedSuggestionEnter
- Menus: none found
- Keys: Enter (acceptSelectedSuggestion) when simpleSuggestWidgetHasFocusedSuggestion and (simpleSuggestWidgetExplicitlyInvoked, or selectionMode is not partial, or the first row is not focused, or simpleSuggestWidgetHasNavigated); Enter (acceptSelectedSuggestionEnter) when config.terminal.integrated.suggest.runOnEnter is not never
- Settings: terminal.integrated.suggest.runOnEnter
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (runOnEnter switch); V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; C bundle (module terminalSuggestConfiguration.js, nls 10958 to 10963, 10920)
- Pieces:
  1. Pressing Enter with a row focused inserts that row into the command line.
  2. With selection mode `partial`, Enter does not accept the first row until you press Down or open the popup by hand, and until then Enter goes to the shell. [stock only]
  3. Setting `terminal.integrated.suggest.runOnEnter` chooses whether Enter also runs the command after inserting, with values `never`, `exactMatch` (when the row is typed in full), `exactMatchIgnoreExtension` (also when a file is typed without its extension) and `always`.
  4. With `always`, Enter inserts the row and also runs the command, and with `never` Enter only inserts it.
  5. The setting has an extra value `ignore`, which is the default, and with it Enter goes straight to the shell without accepting a row. [cursor only]

#### AC-5.10 Choose the selection mode

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched the three ids, the setting id and the labels "Selection Mode: None", "Partial (Tab)" and "Always (Tab or Enter)"; none found; the popup always focuses the first row)
- What: Pick how an auto-opened popup selects a row: partial (first row only outlined, Tab accepts, Enter accepts only after you press Down), always (first row filled, Tab or Enter accepts) or never (nothing selected until Down); a manual open always selects the first row; the status row button cycles none, partial, always and shows the current mode with a tooltip.
- Commands: workbench.action.terminal.changeSelectionModeNever; workbench.action.terminal.changeSelectionModePartial; workbench.action.terminal.changeSelectionModeAlways
- Menus: MenubarTerminalSuggestStatusMenu (group left, order 1; only the item matching the current mode shows, and only when quickSuggestions or suggestOnTriggerCharacters equals true as written)
- Keys: none found
- Settings: terminal.integrated.suggest.selectionMode
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts (SuggestSelectionMode, partial-selection class); V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts
- Pieces:
  1. Setting `terminal.integrated.suggest.selectionMode` chooses how an auto-opened popup selects a row, with values `partial`, `always` and `never`, and the default is `partial`. [stock only]
  2. In `partial` mode the first row is only outlined, Tab accepts it, and Enter accepts it only after you press Down. [stock only]
  3. In `always` mode the first row is filled in, and Tab or Enter accepts it. [stock only]
  4. In `never` mode no row is selected until you press Down. [stock only]
  5. A manual open always selects the first row, whatever the mode. [stock only]
  6. The status row shows one button named for the current mode ("Selection Mode: None", "Partial (Tab)" or "Always (Tab or Enter)"), and clicking it switches to the next mode in the order none, partial, always. [stock only]
  7. The selection mode button only shows when `terminal.integrated.suggest.quickSuggestions` or `terminal.integrated.suggest.suggestOnTriggerCharacters` is set to the plain value `true`. [stock only]

#### AC-5.11 Add a trailing space after accepting

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched the setting id and its text; none found; the accept code in the bundle has no space step)
- What: When on, accepting a suggestion also types a space and opens suggestions again for the next word; folders and symbolic link folders never get the space.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.insertTrailingSpace
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (acceptSelectedSuggestion)
- Pieces:
  1. With `terminal.integrated.suggest.insertTrailingSpace` on, accepting a suggestion also types a space after it, and the setting is off by default. [stock only]
  2. After the space is typed, the popup opens again for the next word. [stock only]
  3. Folder and symbolic link folder rows never get the trailing space. [stock only]

#### AC-5.12 Close the popup

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, title "Hide Suggest Widget" and Escape key found)
- What: Esc closes the popup; it also closes on its own when the terminal scrolls, when you click in the terminal or focus leaves it (except into the details pane), when you type a space, when the typed text no longer matches anything, and when the command line is run.
- Commands: workbench.action.terminal.hideSuggestWidget
- Menus: none found
- Keys: Escape when terminalFocus and terminalSuggestWidgetVisible
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (hideSuggestWidget callers); C bundle (module terminal.suggest.js region, nls 10921)
- Pieces:
  1. Pressing Esc closes the popup.
  2. The popup closes on its own when the terminal scrolls.
  3. The popup closes on its own when you click in the terminal or focus leaves it, except when focus moves into the details pane.
  4. The popup closes on its own when you type a space.
  5. The popup closes on its own when the typed text no longer matches anything.
  6. The popup closes on its own when the command line is run.

#### AC-5.13 Show and focus the details pane

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (both ids, titles and keys found; the details overlay has no placement restriction, so west is allowed)
- What: Open a side pane with the focused row's detail text and docs (markdown), with a Close button and a "Loading..." state; it sits east, south or north of the list but never west, only opens for rows with docs or a detail that differs from the label, and its open state is remembered; a second key moves focus into the pane to scroll and copy text.
- Commands: workbench.action.terminal.suggestToggleDetails; workbench.action.terminal.suggestToggleDetailsFocus
- Menus: none found
- Keys: Ctrl+Space or Ctrl+I (mac: Ctrl+Space or Cmd+I) when terminalFocus, terminalIsOpen, terminalSuggestWidgetVisible and simpleSuggestWidgetHasFocusedSuggestion; Ctrl+Alt+Space (mac: Ctrl+Alt+Space) when not editorTextInputFocus
- Settings: none found
- Source: V\src\vs\workbench\services\suggest\browser\simpleSuggestWidgetDetails.ts; V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts (toggleDetails, toggleDetailsFocus); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; C bundle (module terminal.suggest.js region, nls 10916 and 10917)
- Pieces:
  1. Pressing Ctrl+Space or Ctrl+I (Cmd+I on a Mac) while the popup has a focused row opens a side pane with that row's detail text and docs rendered as markdown.
  2. Pressing the same key again closes the pane.
  3. The pane shows a Close button, and shows "Loading..." while its docs load.
  4. The pane only opens for rows that have docs or a detail text that differs from the label.
  5. The pane sits to the east of the list, or south or north when there is no room, and never to the west. [stock only]
  6. In Cursor the pane has no placement limit, so it can also open to the west of the list. [cursor only]
  7. Whether the pane is open is remembered, so the next popup opens with the pane open or closed as you left it.
  8. Pressing Ctrl+Alt+Space moves focus into the pane, where you can scroll and select or copy its text.

#### AC-5.14 Explain mode

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, title "Suggest Toggle Explain Modes" and Ctrl+/ key found)
- What: A developer aid that fills the details pane with the focused row's score, prefix, replacement range, index and provider name.
- Commands: terminalSuggestToggleExplainMode
- Menus: none found
- Keys: Ctrl+/ (mac: Cmd+/) when terminalFocus, terminalIsOpen and terminalSuggestWidgetVisible
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\services\suggest\browser\simpleSuggestWidgetDetails.ts (renderItem explainMode); C bundle (module terminal.suggest.js region, nls 10915)
- Pieces:
  1. Pressing Ctrl+/ (Cmd+/ on a Mac) while the popup is showing turns explain mode on.
  2. In explain mode the details pane shows the focused row's score, prefix, replacement range, index and provider name.
  3. Pressing Ctrl+/ again turns explain mode off.

#### AC-5.15 Status row at the bottom of the popup

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (row and setting present, default true; items are text buttons drawn as "Label (key)", with only two entries, Insert on the left and Configure on the right)
- What: A thin row of small buttons under the list, split into a left group and a right group, that can be hidden; in V it holds the selection mode switch on the left and the show-while-typing switch, Learn More and Configure on the right.
- Commands: none found
- Menus: MenubarTerminalSuggestStatusMenu
- Keys: none found
- Settings: terminal.integrated.suggest.showStatusBar
- Source: V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts; V\src\vs\editor\contrib\suggest\browser\suggestWidgetStatus.ts; V\src\vs\platform\actions\common\actions.ts (MenubarTerminalSuggestStatusMenu); V\src\vs\platform\actions\browser\menuEntryActionViewItem.ts; C bundle (status bar class Mqn, nls 10965)
- Pieces:
  1. A thin row of small buttons shows under the list, split into a left group and a right group.
  2. Setting `terminal.integrated.suggest.showStatusBar` shows or hides this row, and its default is true.
  3. The row holds the selection mode switch on the left, and the show-while-typing switch, Learn More and Configure on the right. [stock only]
  4. In Cursor the buttons are text buttons drawn as "Label (key)", with only two entries: Insert on the left and Configure on the right. [cursor only]

#### AC-5.16 Switch show-while-typing from the popup

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched both ids and both sentences; none found)
- What: An eye-closed icon button on the status row turns typing suggestions on (commands, arguments and trigger characters), and an eye icon button turns them off; the tooltip is a full sentence saying what changes.
- Commands: workbench.action.terminal.showSuggestOnType; workbench.action.terminal.doNotShowSuggestOnType
- Menus: MenubarTerminalSuggestStatusMenu (group right, order 1)
- Keys: none found
- Settings: terminal.integrated.suggest.quickSuggestions; terminal.integrated.suggest.suggestOnTriggerCharacters
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminal.suggest.ts
- Pieces:
  1. An eye-closed icon button in the right group of the status row, shown while typing suggestions are off, turns them on when clicked by setting commands, arguments and trigger characters on. [stock only]
  2. An eye icon button in the same place, shown while typing suggestions are on, turns them off when clicked. [stock only]
  3. The button's tooltip is a full sentence saying what clicking it will change. [stock only]
  4. The button is the first one in the right group of the status row. [stock only]

#### AC-5.17 Learn More and Configure buttons

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (Configure found with title "Configure", menu group right order 1, and a key Ctrl+Shift+, when the popup is visible; suggestLearnMore not found)
- What: A question mark button opens the terminal IntelliSense docs page, and a gear button opens Settings filtered to terminal.integrated.suggest.
- Commands: workbench.action.terminal.suggestLearnMore; workbench.action.terminal.configureSuggestSettings
- Menus: MenubarTerminalSuggestStatusMenu (group right, order 2 for Learn More, order 3 for Configure)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminal.suggest.ts; C bundle (module terminal.suggest.js region, nls 10909)
- Pieces:
  1. A question mark button on the status row opens the terminal IntelliSense docs page, and it is the second button in the right group. [stock only]
  2. A gear button on the status row opens Settings filtered to `terminal.integrated.suggest`, and in stock it is the third button in the right group.
  3. In Cursor the Configure button is a text button titled "Configure" and is the first item in the right group. [cursor only]
  4. In Cursor, pressing Ctrl+Shift+, while the popup is showing opens the same filtered Settings view. [cursor only]

#### AC-5.18 First-use highlight on the status buttons

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched the class increased-discoverability, the storage key and the command id; none found)
- What: For the first few uses (up to 10 shows, or 10 seconds) the first right-hand status button is drawn in the primary button colors to draw the eye to it; the state is stored for the whole app.
- Commands: workbench.action.terminal.resetDiscoverability
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestShownTracker.ts; V\src\vs\workbench\services\suggest\browser\media\suggest.css; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminal.suggest.ts
- Pieces:
  1. For the first few uses, the first button in the right group of the status row is drawn in the primary button colors. [stock only]
  2. The highlight ends for good after the popup has shown 10 times, or 10 seconds after its first show, whichever comes first. [stock only]
  3. The count and the finished state are stored for the whole app, so they are shared by all windows and terminals and survive a restart. [stock only]
  4. The highlight only shows while the status row is visible. [stock only]

#### AC-5.19 Command and alias suggestions

- Tag: both. Evidence: source. Boundary: no. Cursor: present (kinds Method and Alias, labels "Method" and "Alias", icons and the windowsExecutableExtensions setting found; the bundled terminal-suggest extension emits Method and Alias)
- What: Typing the first word lists commands you can run, with shell built-ins and functions and programs on PATH shown as "Method" rows and shell aliases as "Alias" rows; on Windows the listed executable file types can be changed.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.windowsExecutableExtensions; terminal.integrated.suggest.providers
- Source: V\extensions\terminal-suggest\src\terminalSuggestMain.ts; V\extensions\terminal-suggest\src\shell\common.ts; V\extensions\terminal-suggest\src\shell\pwsh.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionItem.ts; C bundle (extension terminal-suggest dist\terminalSuggestMain.js, nls 10925 and 10926)
- Pieces:
  1. Typing the first word of a command line lists commands you can run: shell built-ins, functions and programs on PATH show as rows of kind "Method".
  2. Shell aliases show as rows of kind "Alias" with the alias icon.
  3. On Windows, setting `terminal.integrated.suggest.windowsExecutableExtensions` chooses which executable file types are listed: set an extension to false to drop it, or to true to add one that is not in the default list.
  4. Setting the command provider to false in `terminal.integrated.suggest.providers` removes the command and alias rows from the popup.

#### AC-5.20 Argument, option and flag suggestions

- Tag: both. Evidence: source. Boundary: no. Cursor: present (kinds Argument, Option, OptionValue and Flag, labels, icons and colors found; the bundled extension emits all four)
- What: After a known command, the popup lists its subcommands and arguments, options, option values and flags, each with its own kind label and icon and a description in the details pane.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.quickSuggestions; terminal.integrated.suggest.providers
- Source: V\extensions\terminal-suggest\src\terminalSuggestMain.ts; V\extensions\terminal-suggest\src\fig\figInterface.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (kind labels); C bundle (extension terminal-suggest dist\terminalSuggestMain.js, nls 10927 to 10930)
- Pieces:
  1. After a known command, the popup lists that command's subcommands and arguments, options, option values and flags.
  2. Each kind has its own label and icon: "Argument", "Option", "OptionValue" and "Flag".
  3. The details pane shows a description for the focused row.
  4. With the `arguments` key of `terminal.integrated.suggest.quickSuggestions` set to `on`, typing a later word opens these suggestions by itself.
  5. Setting the command provider to false in `terminal.integrated.suggest.providers` removes these rows from the popup.

#### AC-5.21 File and folder suggestions

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (File and Folder kinds, labels, icons and the cdPath setting found; symbolic link file and folder kinds not found)
- What: When a path is expected, the popup lists files and folders (and symbolic links) from the working directory or the typed path, plus `../` and `~` entries, and with the cd command it can list children of the $CDPATH folders.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.cdPath; terminal.integrated.suggest.quickSuggestions
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionService.ts (resolveResources); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionItem.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; C bundle (nls 10923, 10924, 10966 to 10969)
- Pieces:
  1. When a path is expected, the popup lists files and folders from the working directory or from the path you have typed.
  2. Symbolic links show as symbolic link file and symbolic link folder rows. [stock only]
  3. The list also includes `../` and `~` entries.
  4. With the `cd` command, setting `terminal.integrated.suggest.cdPath` lists the children of the folders in `$CDPATH`, with values `off`, `relative` and `absolute`, and the default is `absolute`.
  5. Typing a path opens the file and folder suggestions by itself when the matching key of `terminal.integrated.suggest.quickSuggestions` is `on`.

#### AC-5.22 Git suggestions

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (a git command spec with branch lookups is in the bundled extension, drawn with the fig://icon?type=git icon; the branch, tag, commit, stash, remote and pull request kinds, icons and colors, and any gh spec, were not found)
- What: Git and GitHub CLI commands list branches, tags, commits, stashes, remotes and pull requests as their own kinds, each with a git icon (branch, tag, commit, stash, remote, pull request, completed pull request) and a color.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.providers
- Source: V\extensions\terminal-suggest\src\completions\git.ts; V\extensions\terminal-suggest\src\completions\gh.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionItem.ts (kind enum); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSymbolIcons.ts; C bundle (extension terminal-suggest dist\terminalSuggestMain.js)
- Pieces:
  1. After `git`, the popup lists git subcommands, options and branch names for commands that take a branch, drawn with a git icon.
  2. Branches, tags, commits, stashes and remotes show as their own row kinds, each with its own git icon and color. [stock only]
  3. After a GitHub CLI command, pull requests show as "pull request" rows, and merged ones as "completed pull request" rows. [stock only]
  4. Setting the command provider to false in `terminal.integrated.suggest.providers` removes the git rows from the popup.
  5. In Cursor, git rows are all drawn with one generic git icon from the command spec, since the per-kind icons and colors do not exist there. [cursor only]

#### AC-5.23 Shell inline suggestion row

- Tag: both. Evidence: source. Boundary: no. Cursor: present (kind, label "Inline Suggestion", star icon, color and setting with the same three values found)
- What: When the shell already shows its own gray inline suggestion (zsh, fish, bash, PowerShell, Git Bash) the popup adds a row labelled "Inline Suggestion" with a star icon, keeps it on top or just below an exact match, and the Right arrow key accepts it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.inlineSuggestion
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (_inlineCompletion, _refreshInlineCompletion); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionModel.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; C bundle (nls 10931, 10932, 10970 to 10973)
- Pieces:
  1. When the shell already shows its own gray inline suggestion, the popup adds a row labelled "Inline Suggestion" with a star icon.
  2. The row appears for shells that draw their own gray text: zsh, fish, bash, PowerShell and Git Bash.
  3. Setting `terminal.integrated.suggest.inlineSuggestion` chooses how the row is placed, with values `alwaysOnTop` (the default, row always first), `alwaysOnTopExceptExactMatch` (an exact match sorts above it) and `off` (no row).
  4. Pressing the Right arrow key accepts the inline suggestion.

#### AC-5.24 Turn suggestion providers on or off

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (setting present; its default lists terminal-suggest and pwsh-shell-integration, both true, and no lsp entry was found)
- What: One settings object lists every suggestion provider by id with an on or off value, so a whole category can be dropped from the popup; the language server provider (Python terminals only) is off by default and the others are on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.providers
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts (registerTerminalSuggestProvidersConfiguration); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts (TerminalSuggestProvidersConfigurationManager); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionService.ts (_getEnabledProviders); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\lspTerminalUtil.ts; C bundle (module terminalSuggestConfiguration.js, nls 10952)
- Pieces:
  1. Setting `terminal.integrated.suggest.providers` lists every suggestion provider by id with a true or false value.
  2. Setting a provider's id to false removes the rows it supplies from the popup.
  3. Providers that are not mentioned stay on.
  4. The language server provider shows only in Python terminals, and it is off by default while the other providers are on. [stock only]
  5. In Cursor the default lists only `terminal-suggest` and `pwsh-shell-integration`, both true, with no language server entry. [cursor only]

#### AC-5.25 Clear cached shell globals

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched the id in the bundle and in the bundled terminal-suggest extension, and its package.json has no contributes block; none found)
- What: A palette command, Terminal: Clear Suggest Cached Globals, throws away the saved list of shell built-ins and aliases so the next popup reads them fresh.
- Commands: terminal.integrated.suggest.clearCachedGlobals
- Menus: CommandPalette (extension command with no menus entry, so it is listed by default)
- Keys: none found
- Settings: none found
- Source: V\extensions\terminal-suggest\package.json; V\extensions\terminal-suggest\package.nls.json; V\extensions\terminal-suggest\src\terminalSuggestMain.ts
- Pieces:
  1. Running "Terminal: Clear Suggest Cached Globals" (`terminal.integrated.suggest.clearCachedGlobals`) from the Command Palette throws away the saved list of shell built-ins and aliases. [stock only]
  2. The next popup reads the shell's built-ins and aliases fresh, so newly defined aliases and functions show up. [stock only]


## AC-6 Cursor Tab suggestions

(cursor) [boundary]. The ways a Tab edit is shown, plus Tab to accept, partial accept, and Esc to reject. Examples: gray inline text with a "tab" margin label; a preview box with View and Accept; the "Cursor Tab Preview" diff peek; red and green line coloring. Accept is `editor.action.acceptCursorTabSuggestion`.

30 features.

#### AC-6.1 Gray inline text at the cursor

- Tag: cursor. Evidence: bundle. Boundary: yes - the text is a generated suggestion.. Cursor: cursor only
- What: When a suggestion only adds characters (no removal, up to 20 character changes and 20 word changes) and the new text lands at or after the cursor, the new text shows as dimmed, syntax colored gray text inside the line at the insertion point. It is upright, has no border, and the existing code stays untouched. If any of the insertion would land before the cursor, the editor uses the preview box (AC-6.3) instead. The stock theme colors `editorGhostText.*` also apply because the text reuses the stock `ghost-text` classes. The Command Palette also has an "Update Ghost Text" command that redraws it.
- Commands: editor.action.updateGhostText
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module, byte offset ~24936000 to ~24960000: decoration with `inlineClassName` "ghost-text-decoration syntax-highlighted cursor-ghost-text", line class `cpp-inline-suggestion`, helper that builds inline and full-line pieces; CSS `.monaco-editor .syntax-highlighted.cursor-ghost-text` grayscale 40% and opacity .6, `.monaco-editor .cursor-ghost-text` no border and normal style); V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css (the stock classes it reuses)
- Pieces:
  1. When a suggestion only adds characters and the new text lands at or after the cursor, the new text shows as dimmed, syntax colored gray text inside the line at the insertion point, and the existing code stays untouched.
  2. The gray text is upright and has no border.
  3. A suggestion shows as gray text only when it removes nothing and adds up to 20 character changes and 20 word changes; one that removes text or would land before the cursor uses the preview box instead (AC-6.3).
  4. The stock theme colors `editorGhostText.*` also style the gray text, because it reuses the stock `ghost-text` classes.
  5. Running the Command Palette entry "Update Ghost Text" (`editor.action.updateGhostText`) redraws the gray text.

#### AC-6.2 Gray added lines under the cursor line

- Tag: cursor. Evidence: bundle. Boundary: yes - the lines are a generated suggestion.. Cursor: cursor only
- What: When a suggestion adds whole new lines and passes the same test as AC-6.1, the new lines appear as dimmed, syntax colored gray lines under the current line. The editor opens blank room for them (they are not real text). There is no box and no margin label in this shape. Several added lines show as a stack in the order they will be inserted.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module: line renderer that writes `div.suggest-preview-text` rows with `view-line`, and the view zone call `addZone({afterLineNumber, heightInLines, domNode})`; CSS `.syntax-highlighted.cursor-ghost-text`)
- Pieces:
  1. When a suggestion adds whole new lines and passes the same test as AC-6.1, the new lines show as dimmed, syntax colored gray lines under the current line.
  2. The editor opens blank room under the current line for the gray lines, so they are not real text.
  3. Several added lines show as a stack in the order they will be inserted.
  4. This shape has no box and no margin label.

#### AC-6.3 Preview box for edits to existing text

- Tag: cursor. Evidence: bundle. Boundary: yes - the box content is a generated suggestion.. Cursor: cursor only
- What: When a suggestion changes or removes existing text, would insert before the cursor, or is large, a small bordered box appears to the right of the code, level with the first changed line. It shows the new version of the changed lines in the editor's own font, colors and language. The box uses the suggest list background and border colors and a 2px corner radius. The inner editor is read only and drawn bare: no line numbers, folding, minimap, indent guides, current-line highlight, visible scroll bars, whitespace dots or hover.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module: content widget `div.cursorGhostTextWidget` with inner `cppOuter`, `cppInner`, `cppButtonContainer` and `div.cppPreviewBoxDomNode` holding a small embedded editor; CSS `.cursorGhostTextWidget`, `.cppPreviewBoxDomNode .monaco-editor`)
- Pieces:
  1. When a suggestion changes or removes existing text, a small bordered box appears to the right of the code, level with the first changed line, and shows the new version of the changed lines.
  2. The box is also used when the suggestion would insert text before the cursor or is large.
  3. The box text uses the editor's own font, colors and language.
  4. The box uses the suggest list background and border colors and a 2px corner radius.
  5. The editor inside the box is read only.
  6. The editor inside the box is drawn bare, with no line numbers, folding, minimap, indent guides, current-line highlight, visible scroll bars, whitespace dots or hover.

#### AC-6.4 Multi-line preview box size and position

- Tag: cursor. Evidence: bundle. Boundary: yes - the box content is a generated suggestion.. Cursor: cursor only
- What: The box is one row tall for each changed line and as wide as the longest changed line (a tab counts as the tab size). Its left edge starts 24px past the end of the longest line in the changed range, so it never covers that code. Unless `editor.wordWrap` is exactly "on", the box is held at least 2 columns inside the right edge of the editor; with word wrap "on" it is not held back. The box scrolls its inner editor so the changed lines are the ones you see.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.wordWrap
- Source: C bundle (ghost text widget module: box placement code that computes a left margin of 24px plus the column gap, and sets the box width and height from the line count; reads editor option 137)
- Pieces:
  1. The box is one row tall for each changed line.
  2. The box is as wide as the longest changed line, with a tab counting as the tab size.
  3. The left edge of the box starts 24px past the end of the longest line in the changed range, so it never covers that code.
  4. The box scrolls its inner editor so the changed lines are the ones you see.
  5. Setting `editor.wordWrap` changes the right limit of the box: with the value `on` the box is not held back from the right edge, and with any other value (`off`, `wordWrapColumn`, `bounded`) it is held at least 2 columns inside the right edge of the editor.

#### AC-6.5 Green tint on new words in the box

- Tag: cursor. Evidence: bundle. Boundary: yes - the box content is a generated suggestion.. Cursor: cursor only
- What: Inside the preview box, the characters or words the suggestion adds carry the diff editor's inserted text background (a green tint). Words that stay the same carry no tint.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module: decoration class `ghost-text-decoration-inline-add` on the added ranges; CSS `.monaco-editor .ghost-text-decoration-inline-add` using the theme color `diffEditor.insertedTextBackground`)
- Pieces:
  1. Inside the preview box, the characters or words the suggestion adds carry a green tint.
  2. Words that stay the same carry no tint.
  3. The green tint uses the theme color `diffEditor.insertedTextBackground`, so changing that color changes the tint.

#### AC-6.6 Red tint on the code that will change

- Tag: cursor. Evidence: bundle. Boundary: yes - the tint marks a generated suggestion.. Cursor: cursor only
- What: While the preview box shows, the words in your real code that the suggestion would replace or remove are tinted red (the "testing failed" icon color at 17 percent opacity). The tint goes away with the box.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module: `showChangesOnTheRightAndMaybeShowReds` adds decorations with class `cpp-suggestion-red-background`; CSS `.cpp-suggestion-red-background` using the theme color `testing.iconFailed`)
- Pieces:
  1. While the preview box shows, the words in your real code that the suggestion would replace or remove are tinted red.
  2. The red tint uses the theme color `testing.iconFailed` at 17 percent opacity.
  3. The red tint goes away when the box goes away.

#### AC-6.7 Deletion display in the box

- Tag: cursor. Evidence: bundle. Boundary: yes - the box content is a generated suggestion.. Cursor: cursor only
- What: When a suggestion only deletes one stretch of text, the box shows the lines that would go, dimmed (opacity .2) and struck through, with the box padding removed. Nothing new is shown. If the deletion is in several separate places the code draws no box in this path.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module: the branch for a deletion result, rows `div.cppCodeLine` built with `ghost-text-decoration-remove` for leading space and `ghost-text-decoration-inline-remove` for the text; CSS `.ghost-text-decoration-remove`, `.ghost-text-decoration-inline-remove` with line-through)
- Pieces:
  1. When a suggestion only deletes one stretch of text, the box shows the lines that would go, dimmed to opacity .2 and struck through.
  2. In the deletion box nothing new is shown and the box padding is removed.
  3. When the deletion is in several separate places, no box is drawn.

#### AC-6.8 View and Accept strip on the box

- Tag: cursor. Evidence: bundle. Boundary: yes - the buttons act on a generated suggestion.. Cursor: cursor only
- What: Moving the pointer over the preview box adds a thin footer under the code with two small buttons: "View" and "Accept". "Accept" carries a tiny bordered "Tab" chip (always the word Tab, not read from your keybinding). The buttons are dimmed (opacity .6) and light up on hover. The footer is removed when the pointer leaves the box. Only the preview box has this footer, not the gray text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module, byte offset ~24940864: `createActionButtons` builds `div.cpp-suggestion-action-buttons` with `button.cpp-suggestion-action-button` "View" and "Accept" and `span.cpp-suggestion-tab-button` "Tab", added on `mousemove` and removed on `mouseleave`; CSS `.cpp-suggestion-action-button`, `.cpp-suggestion-tab-button`)
- Pieces:
  1. Moving the pointer over the preview box adds a thin footer under the code with two small buttons, "View" and "Accept".
  2. The "Accept" button carries a tiny bordered "Tab" chip that always reads Tab and does not follow your keybinding.
  3. The footer buttons are dimmed (opacity .6) and light up on hover.
  4. The footer is removed when the pointer leaves the box.
  5. Only the preview box has this footer, not the gray text.

#### AC-6.9 Accept by clicking the Accept button

- Tag: cursor. Evidence: bundle. Boundary: yes - it applies a generated suggestion.. Cursor: cursor only
- What: Pressing the mouse button on "Accept" in the box footer applies the whole suggestion, the same result as the Tab key (AC-6.14), without needing keyboard focus on the key. The click does not move the editor cursor first.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module, `createActionButtons`: `mousedown` handler on the Accept button calls the cpp service full accept after `stopPropagation`)
- Pieces:
  1. Pressing the mouse button on "Accept" in the box footer applies the whole suggestion, with the same result as the Tab key (AC-6.14).
  2. The click on "Accept" does not move the editor cursor first.

#### AC-6.10 View button opens the diff peek

- Tag: cursor. Evidence: bundle. Boundary: yes - the peek shows a generated suggestion.. Cursor: cursor only
- What: Pressing the mouse button on "View" in the box footer opens the "Cursor Tab Preview" diff peek (AC-6.11) for that suggestion, showing the whole file before and after.
- Commands: editor.action.setCppDiffPeekView
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module, `createActionButtons`: `mousedown` on View builds original and new file text and runs `editor.action.setCppDiffPeekView`; the command is registered at byte offset ~40857246 with title "Set Cursor Tab Diff Peek View" and no palette entry)
- Pieces:
  1. Pressing the mouse button on "View" in the box footer opens the "Cursor Tab Preview" diff peek (AC-6.11) for that suggestion.
  2. The peek opened by "View" shows the whole file before and after the suggestion.
  3. The command `editor.action.setCppDiffPeekView` ("Set Cursor Tab Diff Peek View") opens the peek and has no Command Palette entry, so "View" is the only visible way to run it.

#### AC-6.11 "Cursor Tab Preview" diff peek

- Tag: cursor. Evidence: bundle. Boundary: yes - the peek shows a generated suggestion.. Cursor: cursor only
- What: An in-editor peek panel with the title text "Cursor Tab Preview" opens under the suggestion's last line. It holds a read only inline diff (not side by side) of the file before and after, with sticky scroll off, scrolled to the first change. It has a frame, no arrow, can be resized, and starts 10 lines tall. The title bar carries the actions in AC-6.12 and AC-6.13. A second command, "Configure Cursor Tab Diff Peek View", changes the diff editor options while a peek is open; it has no palette entry and no UI.
- Commands: editor.action.setCppDiffPeekView; editor.action.configureCppDiffPeekView
- Menus: cpp-diff-peek-view/title
- Keys: none found
- Settings: none found
- Source: C bundle (peek widget class with `showFrame:!0,showArrow:!1,isResizeable:!0`, title node `cpp-peek-view-title-text` "Cursor Tab Preview", menu id `cpp-diff-peek-view/title`, height constant 10, at byte offset ~24926200 to ~24929500; commands at ~40857246); V\src\vs\editor\contrib\peekView\browser\peekView.ts (base peek widget)
- Pieces:
  1. An in-editor peek panel titled "Cursor Tab Preview" opens under the suggestion's last line.
  2. The peek holds a read only inline diff (not side by side) of the file before and after the suggestion.
  3. The diff in the peek has sticky scroll off and is scrolled to the first change.
  4. The peek has a frame and no arrow, and it can be resized.
  5. The peek starts 10 lines tall.
  6. The peek title bar carries an Accept action (AC-6.12) and the Close action (AC-6.13).
  7. The command `editor.action.configureCppDiffPeekView` ("Configure Cursor Tab Diff Peek View") changes the diff editor options while a peek is open, and has no Command Palette entry or other visible control.

#### AC-6.12 Accept from the peek title bar

- Tag: cursor. Evidence: bundle. Boundary: yes - it applies a generated suggestion.. Cursor: cursor only
- What: The peek title bar has one action, a check mark with the tooltip "Accept Cursor Tab Suggestion" (icon id `accept-cpp-suggestion`, menu order 1). Clicking it applies the suggestion shown in the peek and clears the peek suggestion. The action has no Command Palette entry and no key.
- Commands: editor.action.acceptCppSuggestion
- Menus: cpp-diff-peek-view/title
- Keys: none found
- Settings: none found
- Source: C bundle (action class at byte offset ~40856497: id `editor.action.acceptCppSuggestion`, label "Accept Cursor Tab Suggestion", icon `Oe.check`, `menu:{id:cpp-diff-peek-view/title,order:1}`, no `f1`)
- Pieces:
  1. The peek title bar has one action of its own, a check mark with the tooltip "Accept Cursor Tab Suggestion".
  2. Clicking the check mark applies the suggestion shown in the peek and clears the peek suggestion, which closes the peek (AC-6.13).
  3. The action `editor.action.acceptCppSuggestion` has no Command Palette entry and no key, so the check mark is the only way to run it.

#### AC-6.13 Close the diff peek

- Tag: cursor. Evidence: bundle. Boundary: yes - the peek shows a generated suggestion.. Cursor: cursor only
- What: The peek closes with the stock "Close" X action in its title bar. It also closes by itself when the active editor group changes, when the editor or its file changes, when the editor is disposed, when a new Tab suggestion appears, or when the stored peek suggestion is cleared.
- Commands: peekview.close
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (peek widget class constructor: disposers on `onDidChangeActiveGroup`, editor `onDidDispose`, `onDidChangeModel`, `peekViewSuggestion` and `suggestion` events); V\src\vs\editor\contrib\peekView\browser\peekView.ts (stock `peekview.close` action with label "Close")
- Pieces:
  1. Clicking the stock "Close" X (`peekview.close`) in the peek title bar closes the peek.
  2. The peek closes by itself when the active editor group changes.
  3. The peek closes by itself when the editor or its file changes, or when the editor is closed.
  4. The peek closes by itself when a new Tab suggestion appears or when the stored peek suggestion is cleared.

#### AC-6.14 Accept with Tab

- Tag: cursor. Evidence: bundle. Boundary: yes - it applies a generated suggestion.. Cursor: cursor only
- What: Tab applies the whole suggestion and clears it and its tints; for a plain one-line insertion the cursor ends right after the inserted text. When Tab is pressed the editor tries these in order: accept a shown import hint (AC-7.23), accept the suggestion for the current spot, accept the nearest extra suggestion that is within 5 lines of the cursor (AC-6.27), or jump to the next edit if a jump hint shows (AC-7.5). The Cursor Tab key is bound at weight 500, above the stock suggest list Tab (weight 190 in V), so a showing Tab suggestion wins over the suggest list. The key is only active while `cpp.shouldAcceptTab` is true. If a cross-file jump pill is showing and its code preview is not yet visible, the first Tab only reveals the preview (AC-7.8).
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Source: C bundle (accept action class at byte offset ~40745774: id `editor.action.acceptCursorTabSuggestion`, label "Accept Cursor Tab Suggestion", `precondition: cpp.shouldAcceptTab == true`, `kbOpts {primary: Tab, weight: 500}`; `handleTabAction` at ~40739781 with the order above; `updateShouldAcceptTabContextKey` at ~40742000); V\src\vs\platform\keybinding\common\keybindingsRegistry.ts (weight tiers); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (stock Tab)
- Pieces:
  1. Pressing Tab while a suggestion shows applies the whole suggestion and clears it and its tints.
  2. After a plain one-line insertion is accepted, the cursor ends right after the inserted text.
  3. When Tab is pressed the editor tries these in order: accept a shown import hint (AC-7.23), accept the suggestion for the current spot, accept the nearest extra suggestion within 5 lines of the cursor (AC-6.27), then jump to the next edit if a jump hint shows (AC-7.5).
  4. With both a Tab suggestion and the stock suggest list open, Tab accepts the Tab suggestion and not the list item.
  5. The Tab accept key is only active while `cpp.shouldAcceptTab` is true, and otherwise the stock Tab runs.
  6. With a cross-file jump pill showing and its code preview not yet visible, the first Tab only reveals the preview (AC-7.8).

#### AC-6.15 Accept key follows your keybinding

- Tag: cursor. Evidence: bundle. Boundary: yes - the key accepts a generated suggestion.. Cursor: cursor only
- What: The accept action is a normal command, so the Tab binding can be changed in Keyboard Shortcuts. When it is no longer bound to Tab, `cpp.shouldAcceptTab` stays true and the new key works whenever a suggestion or jump exists. The key chips on the hint widgets and the margin label follow the new binding (upper case in the jump hints, lower case in the margin label, label text as the keybinding service prints it). The "Tab" chip on the box's Accept button is fixed text and does not follow the binding, and the Search and Problems views still listen for the Tab key itself.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Source: C bundle (`updateShouldAcceptTabContextKey` reads `lookupKeybinding(editor.action.acceptCursorTabSuggestion)` and sets `boundToTab`; hint widgets call `lookupKeybinding(...).getLabel()`; `createActionButtons` writes the literal "Tab"; Search view tree and Problems view table `onKeyDown` check `ve.Tab`)
- Pieces:
  1. The Tab binding of `editor.action.acceptCursorTabSuggestion` can be changed in Keyboard Shortcuts like any other command.
  2. After it is rebound, the new key accepts whenever a suggestion or jump exists, because `cpp.shouldAcceptTab` stays true.
  3. The key chips on the hint widgets show the new binding in upper case, and the margin label shows it in lower case, using the label text the keybinding service prints.
  4. The "Tab" chip on the box's Accept button is fixed text and does not follow the new binding.
  5. The Search and Problems views still react to the Tab key itself, not to the new binding.

#### AC-6.16 Partial accept, one word at a time

- Tag: cursor. Evidence: bundle. Boundary: yes - it applies part of a generated suggestion.. Cursor: changed (stock command and key, redefined with the `cppSuggestion` condition and a Cursor Tab first path)
- What: With the setting "Enable partial accepts" on, Ctrl+Right (Cmd+Right on Mac) takes only the next word of the Cursor Tab suggestion instead of all of it. The word break follows the file language's word rules. The rest of the suggestion is re-compared with the new text, and the accepted part is tinted like AC-6.24 (the remainder staying on screen is read from the code path, not run). With the setting off, `cppSuggestion` is false, the command does not apply to Cursor Tab, and Ctrl+Right is the normal word move. Cursor redefined the stock "Accept Next Word Of Inline Suggestion" command for this: its precondition adds `cppSuggestion`, it asks the Cursor Tab accept first, and falls back to the stock word accept. The stock toolbar item for this command is for the stock ghost text, which is hidden while a Tab suggestion shows (AC-6.26), so no toolbar button is drawn for Tab suggestions (not run).
- Commands: editor.action.inlineSuggest.acceptNextWord
- Menus: InlineSuggestionToolbar
- Keys: Ctrl+Right (Cmd+Right on Mac) when the editor is writable and (inlineSuggestionVisible or cppSuggestion)
- Settings: cursor.cpp.enablePartialAccepts
- Source: C bundle (command class at byte offset ~24656892: id `editor.action.inlineSuggest.acceptNextWord`, `kbOpts {primary: CtrlCmd|RightArrow, weight: 101}`; context key `cppSuggestion` declared at ~24651804; `acceptNextWord` and `makeEditsForNextWordAccept` in the cpp service at ~40669263 and ~40722906; setting registered with default false at ~38004908); V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (stock command, line 141)
- Pieces:
  1. With the setting on, pressing Ctrl+Right (Cmd+Right on Mac) while the editor is writable takes only the next word of the Cursor Tab suggestion instead of all of it.
  2. The word break follows the word rules of the file's language.
  3. The accepted word is tinted like AC-6.24, and the rest of the suggestion is compared again with the new text (whether the remainder stays on screen is read from code, not run).
  4. Setting `cursor.cpp.enablePartialAccepts` ("Enable partial accepts", default off) chooses whether Ctrl+Right takes one word of a Tab suggestion; with it off, `cppSuggestion` is false and Ctrl+Right is the normal word move.
  5. The stock command `editor.action.inlineSuggest.acceptNextWord` ("Accept Next Word Of Inline Suggestion") now tries the Cursor Tab word accept first and falls back to the stock word accept for stock inline suggestions.
  6. No toolbar button is drawn for taking a word of a Tab suggestion, because the stock `InlineSuggestionToolbar` item belongs to the stock gray text, which is hidden while a Tab suggestion shows (AC-6.26; not run).

#### AC-6.17 Reject with Esc

- Tag: cursor. Evidence: bundle. Boundary: yes - it dismisses a generated suggestion.. Cursor: cursor only
- What: Pressing Esc (no Ctrl, Alt or Cmd held, Shift allowed) while a suggestion shows clears it and all of its tints. Afterward a waiting "tab to jump here" hint or pill (AC-7.1, AC-7.2) and a waiting import hint (AC-7.19) are shown again. The key press is not swallowed, so Esc also does its normal job (for example closing a widget). Esc is a raw key listener in the cpp service, not a registered command or key binding, so it does not appear in Keyboard Shortcuts. With no Tab suggestion showing, Esc instead dismisses an import hint first and a jump hint second (AC-7.13, AC-7.24).
- Commands: none found
- Menus: none found
- Keys: Escape (no Ctrl, Alt or Cmd) while a Tab suggestion shows; not a registered key binding
- Settings: none found
- Source: C bundle (`handleKeyDownForCppKeys` in the cpp service, byte offset ~40645600 to ~40648200: checks `ue.key==="Escape"` with no ctrl, alt or meta, then `rejectAndResetAllCppSuggestions`, `maybeShowHintLineWidget`, `showCorrectUI`)
- Pieces:
  1. Pressing Esc (no Ctrl, Alt or Cmd held, Shift allowed) while a suggestion shows clears the suggestion and all of its tints.
  2. After the suggestion is cleared, a waiting "tab to jump here" hint or pill (AC-7.1, AC-7.2) and a waiting import hint (AC-7.19) are shown again.
  3. The Esc press is not swallowed, so Esc also does its normal job, for example closing a widget.
  4. Esc for this is not a registered key binding, so it does not appear in Keyboard Shortcuts.
  5. With no Tab suggestion showing, Esc dismisses an import hint first and a jump hint second (AC-7.13, AC-7.24).

#### AC-6.18 Type along with the suggestion

- Tag: cursor. Evidence: bundle. Boundary: yes - the suggestion is generated content.. Cursor: cursor only
- What: Typing the characters the suggestion would insert next keeps the suggestion showing, and it shrinks as you go. When the text in the suggestion range finally equals the suggestion, it clears. Typing something that does not follow the suggestion rejects it, and the editor may then show a different one. An edit that adds a newline or deletes text and does not follow the suggestion always rejects it. This is how the gray text of AC-6.1 gets "typed through", as with ordinary inline suggestions.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`continueModelChangeListener` at byte offset ~40679000: compares the range text with the suggestion text, `isOnShortestEditPath`, and `isGuaranteedCacheMiss` which treats any newline or removed text as a mismatch)
- Pieces:
  1. Typing the characters the suggestion would insert next keeps the suggestion showing, and it shrinks as you go.
  2. When the text in the suggestion range finally equals the suggestion, the suggestion clears.
  3. Typing something that does not follow the suggestion rejects it, and the editor may then show a different one.
  4. An edit that adds a newline or deletes text and does not follow the suggestion always rejects it.

#### AC-6.19 Move away to dismiss

- Tag: cursor. Evidence: bundle. Boundary: yes - it dismisses a generated suggestion.. Cursor: cursor only
- What: Moving the cursor or selection so it no longer touches the suggestion's range (a click elsewhere, the arrow keys, a jump by other means) rejects the suggestion and clears its tints. A waiting jump hint is shown again. Moves made by Cursor Tab itself (its own jump, peek and next-suggestion moves) do not count as moving away.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (cpp service `onDidChangeCursorPosition` listener, byte offset ~40673000 to ~40676000: intersects the selection with the suggestion range and calls `rejectAndResetAllCppSuggestions` when it does not; ignores sources `cpp-peek`, `cpp-next-suggestion`, `cpp-revert` and `cursorPrediction`)
- Pieces:
  1. Moving the cursor or selection so it no longer touches the suggestion's range (a click elsewhere, the arrow keys, a jump by other means) rejects the suggestion and clears its tints.
  2. After the suggestion is rejected by moving away, a waiting jump hint is shown again.
  3. Moves made by Cursor Tab itself (its own jump, peek and next-suggestion moves) do not count as moving away.

#### AC-6.20 Box hides on focus loss or file switch

- Tag: cursor. Evidence: bundle. Boundary: yes - it hides a generated suggestion.. Cursor: cursor only
- What: The preview box and gray text are hidden when the editor loses text focus, when the editor shows a different file, and when text changes while the box's anchor line is the last line of the file. They are also hidden if the pointer enters the box with the left button held down. Hiding the box only fades it out and clears its pieces; the suggestion state itself is handled by AC-6.17 to AC-6.19.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module constructor, byte offset ~24936500: listeners `onDidBlurEditorText`, `onDidChangeModel`, `onDidChangeModelContent`, and `mouseenter` with `(buttons&1)===1` call `hideAllInEditor_doesntChangeModel`)
- Pieces:
  1. The preview box and gray text hide when the editor loses text focus.
  2. The preview box and gray text hide when the editor shows a different file.
  3. The preview box and gray text hide when text changes while the box's anchor line is the last line of the file.
  4. The preview box and gray text hide when the pointer enters the box with the left button held down.
  5. Hiding the box only fades it out and clears its pieces, and whether the suggestion itself stays is decided by AC-6.17 to AC-6.19.

#### AC-6.21 Tab indents on a blank line instead of accepting

- Tag: cursor. Evidence: bundle. Boundary: yes - it decides whether a generated suggestion is accepted.. Cursor: cursor only
- What: When the cursor is on a line that holds only whitespace and the suggestion's next characters at the cursor begin with whitespace, Tab inserts an ordinary indent instead of accepting, so you can indent a new line. It applies only before a jump has been accepted for that suggestion, and not in Vim modes other than INSERT. The Tab accept key is switched off by the rule (`cpp.shouldAcceptTab` false) so the stock Tab runs.
- Commands: none found
- Menus: none found
- Keys: Tab
- Settings: none found
- Source: C bundle (`shouldTabInsteadOfAccepting` in the cpp service at byte offset ~40668361; helper that tests for a whitespace-only line at ~40639266; `updateShouldAcceptTabContextKey` at ~40742000)
- Pieces:
  1. When the cursor is on a line that holds only whitespace and the suggestion's next characters at the cursor begin with whitespace, Tab inserts an ordinary indent instead of accepting, because the Tab accept key is switched off (`cpp.shouldAcceptTab` false) and the stock Tab runs.
  2. This blank line rule applies only before a jump has been accepted for that suggestion.
  3. In Vim modes other than INSERT the blank line rule is skipped, so Tab can accept.

#### AC-6.22 Selection, Shift+Tab and Vim rules

- Tag: cursor. Evidence: bundle. Boundary: yes - they decide whether a generated suggestion is accepted.. Cursor: cursor only
- What: With a selection that spans more than one line, Tab is not captured, so it indents the selection as usual (the Search view and Problems list handlers are the exception, see AC-6.23). Shift+Tab is never captured. When the Vim extension's status bar item shows any mode other than INSERT, the multi-line selection rule and the blank line rule (AC-6.21) are skipped, so Tab can accept. Accept is also off when the suggest list is open while the editor does not have focus, or when no editor or file is found.
- Commands: none found
- Menus: none found
- Keys: Tab; Shift+Tab
- Settings: none found
- Source: C bundle (`updateShouldAcceptTabContextKey` at byte offset ~40742000, `handleKeyDownForCppKeys` ignoring `Tab` with `shiftKey`, `isInVimNonInsertMode` reading the status bar entry `vscodevim.vim.primary`)
- Pieces:
  1. With a selection that spans more than one line, Tab is not captured and indents the selection as usual (the Search view and Problems list handlers are the exception, AC-6.23).
  2. Shift+Tab is never captured, so it keeps its stock meaning.
  3. When the Vim extension's status bar item shows any mode other than INSERT, the multi-line selection rule and the blank line rule (AC-6.21) are skipped, so Tab can accept.
  4. Tab does not accept when the suggest list is open while the editor does not have focus.
  5. Tab does not accept when no editor or file is found.

#### AC-6.23 Accept while focus is in Find, Search, Problems or Peek References

- Tag: cursor. Evidence: bundle. Boundary: yes - it applies a generated suggestion.. Cursor: cursor only
- What: A showing suggestion can still be accepted with Tab when keyboard focus is not in the editor text but in one of four places tied to that editor: the Find box (with a non-empty selection in the editor), a focused match in the Search results, the Problems list, or the preview editor inside a Peek References widget. After accepting from the Find box, focus returns to the Find input without selecting its text. The Find case is controlled by a stored flag (`cppInCmdF`, default on) that is not a setting.
- Commands: none found
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Source: C bundle (`isTextFocusedOrSimilarlyFocused`, `isFindFocused`, `isProblemFocused`, `isReferenceFocused`, `isSearchViewFocused` at byte offset ~40700846 to ~40702000; Search view tree `onKeyDown` at ~35984362 and Problems widget `onKeyDown` at ~36607328, both calling `handleTabAction` with "searchView" or "problemsView"; default `cppInCmdF:!0` at ~17166508)
- Pieces:
  1. With keyboard focus in the Find box and a non-empty selection in the editor, Tab accepts a showing suggestion, controlled by a stored flag (`cppInCmdF`, default on) that is not a setting.
  2. After accepting from the Find box, focus returns to the Find input without selecting its text.
  3. With keyboard focus on a match in the Search results, Tab accepts a showing suggestion.
  4. With keyboard focus in the Problems list, Tab accepts a showing suggestion.
  5. With keyboard focus in the preview editor inside a Peek References widget, Tab accepts a showing suggestion.

#### AC-6.24 Green tint on accepted text

- Tag: cursor. Evidence: bundle. Boundary: yes - it marks generated text.. Cursor: cursor only
- What: After a suggestion (or a word of it) is accepted, the text that was just inserted or changed is tinted green (the "testing passed" icon color at 12 percent opacity) so you can see what landed. The tint stays through typing and is cleared when the cursor moves again while no suggestion is showing (read from the code path, not run).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`updateSuggestionGreenHighlights` at byte offset ~40722906 with class `cpp-suggestion-green-background`; clearing in the `onDidChangeCursorPosition` listener via `didShowGreenHighlights`; CSS `.cpp-suggestion-green-background` using the theme color `testing.iconPassed`)
- Pieces:
  1. After a suggestion (or a word of it) is accepted, the text that was just inserted or changed is tinted green.
  2. The green tint uses the theme color `testing.iconPassed` at 12 percent opacity.
  3. The tint stays through typing and is cleared when the cursor moves again while no suggestion is showing (read from code, not run).

#### AC-6.25 Next suggestion right after an accept

- Tag: cursor. Evidence: bundle. Boundary: yes - the follow-up is another generated suggestion.. Cursor: cursor only
- What: After an accept succeeds, a follow-up can show straight away without typing: another suggestion for the spot, or a jump hint or pill for the next edit (AC-7.1, AC-7.2). If the accept did nothing (nothing was applied), no follow-up is shown.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`cleanupAfterAcceptSuggestion` at byte offset ~40729009 with `shouldDisplayFollowup`, a stored follow-up lookup and `displayNextActionIfAvailable`)
- Pieces:
  1. After an accept succeeds, another suggestion for the spot can show straight away without typing.
  2. After an accept succeeds, a jump hint or pill for the next edit can show straight away (AC-7.1, AC-7.2).
  3. If the accept did nothing because nothing was applied, no follow-up is shown.

#### AC-6.26 Stock gray suggestion yields to Cursor Tab

- Tag: cursor. Evidence: bundle. Boundary: yes - it swaps one generated suggestion for another.. Cursor: cursor only
- What: While a Cursor Tab suggestion is showing, the stock VS Code gray inline suggestion and its hover toolbar are hidden, so only one gray suggestion is on screen. When the Cursor Tab suggestion is gone, the stock one is unhidden and asked for again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.enabled
- Source: C bundle (cpp service methods `disableAndHideCopilotSuggestion` at byte offset ~40725849 and `reallowCopilotIfWePreviousHidCopilotSuggestions` at ~40667467, which re-triggers the stock controller); V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (the stock controller it drives)
- Pieces:
  1. While a Cursor Tab suggestion is showing, the stock gray inline suggestion is hidden, so only one gray suggestion is on screen.
  2. While a Cursor Tab suggestion is showing, the hover toolbar of the stock inline suggestion is hidden too.
  3. When the Cursor Tab suggestion is gone, the stock suggestion is unhidden and asked for again.
  4. Setting `editor.inlineSuggest.enabled` (true or false) controls whether stock inline suggestions show at all, and so whether there is one to hide and bring back.

#### AC-6.27 Several boxes at once

- Tag: cursor. Evidence: bundle. Boundary: yes - the boxes are generated suggestions.. Cursor: cursor only
- What: More than one suggestion can be on screen. Each extra one is drawn as its own preview box (never as gray text) beside its own changed lines. An extra box is dropped if its range falls within 2 lines of the main suggestion's range. When no main suggestion is showing, Tab accepts the first extra one that lies within 5 lines of the cursor. There is no key to cycle between them: no next or previous command was found for Cursor Tab, and the stock "Show Next Inline Suggestion" commands only act on the stock gray text.
- Commands: none found
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Source: C bundle (ghost text controller `refreshAdditionalWidgets` and `update` at byte offset ~24962565 to ~24965500, calling the widget with `forceDiffBox:!0`; `handleTabAction` (starts at ~40739781) branch for `additionalSuggestion` with the 5-line window)
- Pieces:
  1. More than one suggestion can be on screen, and each extra one is drawn as its own preview box beside its own changed lines, never as gray text.
  2. An extra box is dropped if its range falls within 2 lines of the main suggestion's range.
  3. When no main suggestion is showing, Tab accepts the first extra one that lies within 5 lines of the cursor.
  4. There is no key to cycle between the boxes, and the stock "Show Next Inline Suggestion" commands only act on the stock gray text.

#### AC-6.28 Added lines with a green bar and a key label

- Tag: cursor. Evidence: bundle. Boundary: yes - the block is a generated suggestion.. Cursor: cursor only
- What: A third drawing for new lines: a block of lines under the changed lines with a 2px green bar on the right edge of the left margin and a small key label in the margin to its left. The label shows the current accept key in lower case ("tab" if none is found) and has the tooltip "Tab to accept suggestion". The original lines it would replace get a faint "unchanged code" background (opacity .35) and the block a faint blue tint. In the shipped defaults this shape is not reachable: it is used only when the Cursor Settings Development toggle "Show View Zone When Preview Box Is Clipped" is on (default off, and the Development section needs a development-tooling flag that is off) and the preview box would be clipped by the window. The older description "gray text with a tab margin label" does not match the code: the margin label belongs to this block, not to the gray text of AC-6.1.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module: `updateViewZone`, `createColoredLeftMargin` at byte offset ~24959889, `isPreviewBoxClipped`, `updateViewZone` at ~24957397; default `shouldShowViewZoneWhenPreviewBoxIsClipped6:!1` at ~17164970; Development row label "Show View Zone When Preview Box Is Clipped" at ~38267943; `developmentTooling:!1` at ~395975; CSS `.cpp-ghost-text-view-zone-margin`, `.cpp-ghost-text-view-zone-text`, `.view-zone-original-range-decoration`, `.view-zone-in-view-zone-decoration`)
- Pieces:
  1. A block of added lines can be drawn under the changed lines with a 2px green bar on the right edge of the left margin.
  2. A small key label in the margin to the left of the bar shows the current accept key in lower case ("tab" if none is found), with the tooltip "Tab to accept suggestion".
  3. The original lines the block would replace get a faint "unchanged code" background (opacity .35) and the block gets a faint blue tint.
  4. This shape is used only when the Cursor Settings Development toggle "Show View Zone When Preview Box Is Clipped" is on and the preview box would be clipped by the window.
  5. In the shipped defaults this shape is not reachable, because the toggle is off by default and the Development section needs a development-tooling flag that is off.

#### AC-6.29 Colors follow the file language

- Tag: cursor. Evidence: bundle. Boundary: yes - it styles generated text.. Cursor: cursor only
- What: The preview box and gray lines are colored in the language of the open file: the box's inner editor gets the file's language id (and is switched when the file language changes), and gray lines are colored by a temporary model of that language. Tab characters count as the file's tab size when the box width is worked out. The language does nothing else to how the suggestion looks; language only decides whether Tab is on for the file (AC-8.4) and which files get the import hint (AC-7.25).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.tabSize
- Source: C bundle (ghost text widget module: `setLanguage(t.getLanguageId())` on the box model, the temporary tokenize model `cpp-ghost-text-temp-tokenize-anysphere`, tab size read from `getOptions().tabSize`)
- Pieces:
  1. The preview box and the gray lines are colored in the language of the open file.
  2. The box's inner editor switches to the new language when the file language changes.
  3. Setting `editor.tabSize` sets how many columns a tab character counts as when the box width is worked out.

#### AC-6.30 Preview box in notebook cells

- Tag: cursor. Evidence: bundle. Boundary: yes - the box content is a generated suggestion.. Cursor: cursor only
- What: In notebook files (`.ipynb` files and notebook cells) the preview box is drawn 12px taller than the line count needs, and its inner editor is scrolled using the real editor's line positions instead of a fixed row height. This is the only per-file-type difference found in how the box looks.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (ghost text widget module: `.ipynb` check adding 12 to the box height at byte offset ~24950771; helper treating schemes `vscode-notebook-cell` and `ipynb` and `.ipynb` paths as notebooks at ~24962244)
- Pieces:
  1. In notebook files (`.ipynb` files and notebook cells) the preview box is drawn 12px taller than the line count needs.
  2. In notebook files the box's inner editor is scrolled using the real editor's line positions instead of a fixed row height.


## AC-7 Cursor Tab jump and imports

(cursor) [boundary]. Hints that the next edit is elsewhere and Tab will go there, plus the import hint. Examples: the "tab to jump here" line hint; the off-screen "TAB next move" arrow; a cross-file path with code preview; "import X (press Tab or ESC)".

25 features.

#### AC-7.1 "TAB to jump here" line hint

- Tag: cursor. Evidence: bundle. Boundary: yes - it appears only because a next edit was proposed.. Cursor: cursor only
- What: When the next edit is in the open file and its line is on screen, a small pill sits just after the end of that line. It reads a key chip with the accept key in upper case ("TAB" unless you rebound it) followed by muted text "to jump here". The pill has a faint tint of the suggest list highlight color, a 4px corner radius, a soft shadow and a 2px accent bar on its left edge; its text is about half strength, the font is the editor size minus 2, and the height is one line minus 2px. On an empty target line the text starts 2px in, otherwise 0.5rem. It ignores the mouse (pointer events off), so it cannot be clicked. Cursor Tab calls this widget the "hint line widget".
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: editor.fontSize; editor.lineHeight
- Source: C bundle (hint widget class at byte offset ~40746326: content widget id `cursorPredictionHintLineWidget...`, nodes `span.cursor-prediction-hint-line-widget`, `span.tab-key`, `span.jump-to-edit` "to jump here", class `fancy`; CSS `.cursor-prediction-hint-line-widget`, `.cursor-prediction-hint-line-widget:before`, `.tab-key`)
- Pieces:
  1. When the next edit is in the open file and its line is on screen, a small pill sits just after the end of that line and reads a key chip with the accept key in upper case ("TAB" unless you rebound it) followed by muted text "to jump here".
  2. The pill has a faint tint of the suggest list highlight color, a 4px corner radius, a soft shadow and a 2px accent bar on its left edge.
  3. The pill text is about half strength, its font is the `editor.fontSize` minus 2, and its height is one line (from `editor.lineHeight`) minus 2px.
  4. On an empty target line the pill text starts 2px in, and otherwise 0.5rem in.
  5. The pill ignores the mouse, so it cannot be clicked.

#### AC-7.2 Off-screen "TAB to next move" pill with an arrow

- Tag: cursor. Evidence: bundle. Boundary: yes - it appears only because a next edit was proposed.. Cursor: cursor only
- What: When the next edit is in this file but its line is outside the visible lines, a pill appears at the top center of the editor (arrow up, target above) or at the bottom center (arrow down, target below). It reads the key chip (accept key in upper case, "TAB" as fallback), then "to next move" (the template text is "to " plus "next move"), then the arrow icon. The pill is small (20px tall), blurred behind (backdrop blur 15px), tinted 7 percent, with a 6px corner radius, and the whole widget is drawn at 90 percent scale. When any inline diff is open, the top pill sits lower (a 56px margin instead of 24px).
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Source: C bundle (indicator component at byte offset ~40749424: templates `div.cursorPredictionOutOfRangeIndicator`, `div.cursorPredictionOutOfRangeIndicatorMinimized`, `span.tab-key`, `span.jump-to-edit` "to "; arrow icons `Oe.arrowUp` and `Oe.arrowDown`; overlay widget id `tooltip.editorUiWidget` at ~40756352 with position preference 2; CSS `.cursorPredictionOutOfRangeIndicator`, `.cursorPredictionOutOfRangeIndicatorMinimized`)
- Pieces:
  1. When the next edit is in this file but its line is outside the visible lines, a pill appears at the top center of the editor (arrow up, target above) or at the bottom center (arrow down, target below).
  2. The pill reads the key chip (accept key in upper case, "TAB" as fallback), then "to next move", then the arrow icon.
  3. The pill is small (20px tall), blurred behind (backdrop blur 15px), tinted 7 percent, has a 6px corner radius, and the whole widget is drawn at 90 percent scale.
  4. When any inline diff is open, the top pill sits lower, with a 56px margin instead of 24px.

#### AC-7.3 Hint and pill swap as you scroll

- Tag: cursor. Evidence: bundle. Boundary: yes - it follows a proposed next edit.. Cursor: cursor only
- What: Scrolling the editor swaps the two same-file indicators: when the target line scrolls into view the pill hides and the line hint (AC-7.1) shows; when it scrolls out of view the hint hides and the pill (AC-7.2) shows, with its arrow flipping to up or down to match the side. The pill also re-checks when the cursor moves, the selection changes, or the prediction changes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`handleEditorScroll` at byte offset ~40776319; the indicator component's scroll, cursor and selection listeners at ~40749424)
- Pieces:
  1. When the target line scrolls into view, the pill hides and the line hint (AC-7.1) shows.
  2. When the target line scrolls out of view, the line hint hides and the pill (AC-7.2) shows, with its arrow flipping to up or down to match the side.
  3. The pill also re-checks which side to show when the cursor moves, the selection changes, or the proposed jump target changes.

#### AC-7.4 Click the pill to jump

- Tag: cursor. Evidence: bundle. Boundary: yes - it follows a proposed next edit.. Cursor: cursor only
- What: Pressing the mouse button on the pill runs the accept action, which takes the same jump as the Tab key (AC-7.5). The pill has a pointer cursor in its minimized form. For the go-back pill (AC-7.12) the click goes back instead.
- Commands: editor.action.acceptCursorTabSuggestion; workbench.action.navigateBack
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (indicator component: `mousedown` handler runs `editor.action.acceptCursorTabSuggestion`, or `workbench.action.navigateBack` for the go-back variant; CSS `.cursorPredictionOutOfRangeIndicatorMinimized` with `cursor:pointer`)
- Pieces:
  1. Pressing the mouse button on the pill runs the accept action, which takes the same jump as the Tab key (AC-7.5).
  2. The pill shows a pointer cursor over it in its minimized form.
  3. Pressing the mouse button on the go-back pill (AC-7.12) goes back (`workbench.action.navigateBack`) instead of jumping.

#### AC-7.5 Tab jumps to the next edit

- Tag: cursor. Evidence: bundle. Boundary: yes - it moves to a proposed edit location.. Cursor: cursor only
- What: With a jump hint or pill showing and no suggestion to accept at the cursor, Tab moves the cursor to the first non-blank character of the target line and scrolls that line to the center if it was outside the view. The hint or pill is cleared by the jump. If a suggestion for the current spot is also showing and it came from a jump proposal, the first Tab only jumps, and a following Tab then accepts it (read from the code path, not run). A press is ignored when the cursor line is empty (AC-7.16).
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Source: C bundle (`maybeAcceptCursorPrediction` at byte offset ~40761500: `setPosition({lineNumber, column: first non-whitespace}, "cursorPrediction")` and `revealLineInCenterIfOutsideViewport`; `tabToLineBeforeAcceptingCpp` at ~40772962; `handleTabAction` at ~40739781)
- Pieces:
  1. With a jump hint or pill showing and no suggestion to accept at the cursor, pressing Tab moves the cursor to the first non-blank character of the target line.
  2. If the target line was outside the view, the jump scrolls it to the center.
  3. The jump clears the hint or pill.
  4. If a suggestion for the current spot is also showing and it came from a jump proposal, the first Tab only jumps and a following Tab then accepts it (read from code, not run).
  5. A Tab press is ignored for jumping when the cursor line is empty (AC-7.16).

#### AC-7.6 A suggestion can appear on arrival

- Tag: cursor. Evidence: bundle. Boundary: yes - the new suggestion is generated content.. Cursor: cursor only
- What: After a jump, a new Cursor Tab suggestion can show at the landing line (a flag is set when the hint is created or shown again after Esc), so a suggestion may appear right after the cursor arrives and a second Tab can accept it. Whether one appears was not run; this is read from the code path that follows a jump.
- Commands: none found
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Source: C bundle (`retriggerCppOnAccept` set in `manuallyCreateCursorPrediction` and `maybeShowHintLineWidget`, used at the end of `maybeAcceptCursorPrediction`, byte offset ~40761500)
- Pieces:
  1. After a jump, a new Cursor Tab suggestion can show at the landing line, so one may appear right after the cursor arrives.
  2. A second Tab can accept the suggestion that appeared on arrival.
  3. This also holds when the jump hint was shown again after Esc (read from the code path after a jump, not run).

#### AC-7.7 Cross-file pill with file icon, name and arrow

- Tag: cursor. Evidence: bundle. Boundary: yes - it appears only because a next edit in another file was proposed.. Cursor: cursor only
- What: When the next edit is in a different file, the pill at the bottom center of the editor reads the key chip, then "to", then the file's icon and its file name (last path part only, bold italic), then an arrow pointing right. Clicking it or pressing Tab starts the jump (AC-7.8, AC-7.10). While it is showing, the context key `tabPredictionGoToFileVisibleKey` ("Whether a cursor prediction for a target file is currently visible") is true.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true; when tabPredictionGoToFileVisibleKey
- Settings: none found
- Source: C bundle (indicator component at byte offset ~40749424: `JWw` template `div > span.file-path`, file icon component with `fileName`, arrow `Oe.arrowRight`; key declared at ~40643366; CSS `.cursorPredictionOutOfRangeIndicator .file-path`, `.file-icon`)
- Pieces:
  1. When the next edit is in a different file, a pill at the bottom center of the editor reads the key chip, then "to".
  2. After "to", the pill shows the file's icon and its file name (last path part only, bold italic).
  3. The pill ends with an arrow pointing right.
  4. Clicking the cross-file pill or pressing Tab starts the jump (AC-7.8, AC-7.10).
  5. While the cross-file pill is showing, the context key `tabPredictionGoToFileVisibleKey` ("Whether a cursor prediction for a target file is currently visible") is true.

#### AC-7.8 First Tab reveals the code preview

- Tag: cursor. Evidence: bundle. Boundary: yes - the preview shows a proposed edit location.. Cursor: cursor only
- What: On a cross-file pill whose code preview is hidden, the first Tab (or click) does not jump: it only reveals the code preview above the pill. The next Tab then jumps. The context key `tabPredictionCodePreviewVisibleKey` ("Whether a code preview for a target file is currently visible") turns true when the preview shows.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true; when tabPredictionCodePreviewVisibleKey
- Settings: none found
- Source: C bundle (accept action at byte offset ~40745774: if go-to-file key is true and code-preview key is false it calls `setPreviewVisible(!0)` and returns; keys declared at ~40643366)
- Pieces:
  1. On a cross-file pill whose code preview is hidden, the first Tab (or click) does not jump and only reveals the code preview above the pill.
  2. The next Tab after the preview shows jumps to the other file.
  3. The context key `tabPredictionCodePreviewVisibleKey` ("Whether a code preview for a target file is currently visible") turns true when the preview shows.

#### AC-7.9 Code preview of the target file

- Tag: cursor. Evidence: bundle. Boundary: yes - the preview shows a proposed edit location.. Cursor: cursor only
- What: The code preview is a small read only editor, 300px wide and 7 lines tall, drawn above the cross-file pill. It shows the target file in its own language, centered on the target line (3 lines above it). Every line except the target line is dimmed to half strength, and the target line carries a bright "TAB to jump here" hint (AC-7.1).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (code preview builder `rGw` at byte offset ~40754567: height of 7 lines, width 300px, decorations `cursor-prediction-dimmed-line` and `cursor-prediction-dimmed-line-inline`, the hint widget `highlight()`; CSS `.cursor-prediction-dimmed-line-inline{opacity:.5}`)
- Pieces:
  1. The code preview is a small read only editor, 300px wide and 7 lines tall, drawn above the cross-file pill.
  2. The preview shows the target file in its own language, centered on the target line (3 lines above it).
  3. Every line in the preview except the target line is dimmed to half strength.
  4. The target line in the preview carries a bright "TAB to jump here" hint (AC-7.1).

#### AC-7.10 Jump across files opens the target file

- Tag: cursor. Evidence: bundle. Boundary: yes - it moves to a proposed edit location in another file.. Cursor: cursor only
- What: A jump to another file switches to that file and puts the cursor on the target line (first non-blank character), scrolling it to the center if needed. If an editor group already shows the file, that group is used and nothing new opens. If no group shows it, the file is opened in the active group (not to the side) with the focus moved to it.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Source: C bundle (`maybeAcceptCursorPrediction` at byte offset ~40761500: group lookup by `activeEditor.resource`, helper `oGw` calling the opener with `{openToSide: inNewTab, editorOptions: {revealIfVisible, revealIfOpened, preserveFocus}}` then `r.focus()`)
- Pieces:
  1. A jump to another file switches to that file and puts the cursor on the target line (first non-blank character), scrolling it to the center if needed.
  2. If an editor group already shows the file, that group is used and nothing new opens.
  3. If no group shows the file, the file is opened in the active group (not to the side) and focus moves to it.

#### AC-7.11 Target file already open in a split

- Tag: cursor. Evidence: bundle. Boundary: yes - it points at a proposed edit location in another editor.. Cursor: cursor only
- What: When the target file is already open in another editor group, the active editor shows the cross-file pill (AC-7.7) and the other editor gets the markers for its own file: the "TAB to jump here" line hint on the target line if that line is visible there (AC-7.1), or the off-screen pill without a file name (AC-7.2) if it is not. Scrolling the other editor swaps between the two, as in AC-7.3.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`createMultifileCursorPrediction` at byte offset ~40769674: shows the pill with a relative path in the active editor, a hint widget or a pill with an empty path in the editor that holds the target; `handleEditorScroll` at ~40776319 with the `!s&&...` branch)
- Pieces:
  1. When the target file is already open in another editor group, the active editor shows the cross-file pill (AC-7.7).
  2. The other editor shows the "TAB to jump here" line hint on the target line if that line is visible there (AC-7.1).
  3. If the target line is not visible in the other editor, it shows the off-screen pill without a file name (AC-7.2).
  4. Scrolling the other editor swaps between the line hint and the pill, as in AC-7.3.

#### AC-7.12 "to go back" pill after a cross-file jump

- Tag: cursor. Evidence: bundle. Boundary: yes - it follows a jump to a proposed edit location.. Cursor: cursor only
- What: After a jump that switched files, a pill at the bottom center shows a key chip with the key bound to Go Back (for example Alt+Left on Windows) and the text "to go back". Clicking it or pressing that key goes back. It is shown at most twice in a window session (the count is kept in memory, not saved) and is not shown at all if Go Back has no key. It hides when the selection changes or the editor loses focus.
- Commands: workbench.action.navigateBack
- Menus: none found
- Keys: Alt+Left on Windows (Ctrl+- on Mac, Ctrl+Alt+- on Linux)
- Settings: none found
- Source: C bundle (`goBackIndicatorDisplayCount` and `MAX_GO_BACK_INDICATOR_DISPLAYS=2` in the prediction service at byte offset ~40758857; indicator component with `isGoBackIndicator` and template `span.jump-to-edit` " to go back" at ~40749424); V\src\vs\workbench\browser\parts\editor\editorActions.ts (`NavigateBackwardsAction`, keys)
- Pieces:
  1. After a jump that switched files, a pill at the bottom center shows a key chip with the key bound to Go Back and the text "to go back".
  2. Clicking the pill or pressing that key goes back (`workbench.action.navigateBack`, Alt+Left on Windows, Ctrl+- on Mac, Ctrl+Alt+- on Linux).
  3. The go-back pill is shown at most twice in a window session, and the count is kept in memory and not saved.
  4. The go-back pill is not shown at all if Go Back has no key.
  5. The go-back pill hides when the selection changes or the editor loses focus.

#### AC-7.13 Esc dismisses the jump hint

- Tag: cursor. Evidence: bundle. Boundary: yes - it dismisses a proposed edit location.. Cursor: cursor only
- What: With no Tab suggestion showing, Esc clears the jump hint and pill (and a cross-file code preview). If an import hint is also showing, the first Esc dismisses the import hint and swallows the key, and a second Esc clears the jump. Clearing by Esc counts as a rejection of that target (AC-7.15). With a Tab suggestion showing, Esc rejects the suggestion instead and the jump hint stays or is shown again (AC-6.17).
- Commands: none found
- Menus: none found
- Keys: Escape (no Ctrl, Alt or Cmd); not a registered key binding
- Settings: none found
- Source: C bundle (`handleKeyDownForCppKeys` at byte offset ~40645600 to ~40648200: `clearCursorPrediction()` in the Escape branch, after `maybeRejectShownImport`; `clearPrediction` at ~40764097)
- Pieces:
  1. With no Tab suggestion showing, pressing Esc (no Ctrl, Alt or Cmd) clears the jump hint and pill, and a cross-file code preview.
  2. If an import hint is also showing, the first Esc dismisses the import hint and swallows the key, and a second Esc clears the jump.
  3. Clearing by Esc counts as a rejection of that target (AC-7.15).
  4. With a Tab suggestion showing, Esc rejects the suggestion instead, and the jump hint stays or is shown again (AC-6.17).

#### AC-7.14 Jump hint cleared by edits, focus loss, file switch and resize

- Tag: cursor. Evidence: bundle. Boundary: yes - it removes a proposed edit location.. Cursor: cursor only
- What: The hint and pills disappear (and the target counts as rejected, except for resize) when you type or otherwise change the file, when the editor loses text focus, when the editor shows another file, or when the editor is closed. Resizing the window clears the hint and pills in every editor without counting a rejection. Deleting or removing the target file clears them too.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`createPrediction` at byte offset ~40767469: listeners `onDidChangeModel`, `onDidBlurEditorText`, `onDidChangeModelContent` calling `clearPrediction({registerReject:!0})`; window resize listener in the service constructor at ~40760408 and `clearPredictionForDeletedFile` at ~40765566)
- Pieces:
  1. The hint and pills disappear when you type or otherwise change the file, and that counts as a rejection of the target.
  2. The hint and pills disappear when the editor loses text focus, and that counts as a rejection.
  3. The hint and pills disappear when the editor shows another file or is closed, and that counts as a rejection.
  4. Resizing the window clears the hint and pills in every editor without counting a rejection.
  5. Deleting or removing the target file clears the hint and pills.

#### AC-7.15 When no jump is offered

- Tag: cursor. Evidence: bundle. Boundary: yes - it limits when a proposed edit location is shown.. Cursor: cursor only
- What: A jump hint is skipped when the target is within 1 line of the cursor, when it lies inside the range of the suggestion you just accepted, when it overlaps an inline diff that is still being written, when its line number is past the end of the file, and when you dismissed that same line twice in the last 30 seconds (the last 5 dismissed lines are remembered). In the shortcut path where the jump is shown in place of an edit, it is also skipped when the target is under 5 lines from a spot you already jumped to since your last edit, and straight after a jump.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`createPrediction` and `createMultifileCursorPrediction` at byte offsets ~40767469 and ~40769674; `wasRecentlyRejected` with `REJECTION_EXPIRY_MS=30*1e3` and `MAX_REJECTED_PREDICTIONS=5` at ~40777532; bad-heuristic check `isValidCursorPredictionCase` at ~40758107; `immediatelyShowNextCursorPrediction` at ~40713228)
- Pieces:
  1. A jump hint is not shown when the target is within 1 line of the cursor.
  2. A jump hint is not shown when the target lies inside the range of the suggestion you just accepted.
  3. A jump hint is not shown when the target overlaps an inline diff that is still being written.
  4. A jump hint is not shown when its line number is past the end of the file.
  5. A jump hint is not shown for a line you dismissed twice in the last 30 seconds, and the last 5 dismissed lines are remembered.
  6. Where the jump is shown in place of an edit, it is also skipped when the target is under 5 lines from a spot you already jumped to since your last edit.
  7. Where the jump is shown in place of an edit, it is also skipped straight after a jump.

#### AC-7.16 Tab on an empty line does not jump

- Tag: cursor. Evidence: bundle. Boundary: yes - it decides whether a proposed jump is taken.. Cursor: cursor only
- What: When the cursor is on an empty line, Tab does not jump: the jump handler returns without moving the cursor, and in the shortcut path the jump is not offered at all because Tab would indent there. In Vim modes other than INSERT the rule is skipped. This is the jump version of the blank line rule for suggestions (AC-6.21).
- Commands: none found
- Menus: none found
- Keys: Tab
- Settings: none found
- Source: C bundle (prediction service `shouldTabInsteadOfAccepting(editor, model)` at byte offset ~40763478; `cursorPredictorWouldTabInsteadOfAccepting` at ~40668152 used by `immediatelyShowNextCursorPrediction`)
- Pieces:
  1. When the cursor is on an empty line, pressing Tab does not jump and the cursor does not move.
  2. Where the jump is shown in place of an edit, the jump is not offered at all on an empty line, because Tab would indent there.
  3. In Vim modes other than INSERT the empty line rule is skipped.

#### AC-7.17 Hint hides under a suggestion box

- Tag: cursor. Evidence: bundle. Boundary: yes - it resolves a clash between two generated hints.. Cursor: cursor only
- What: If a Tab preview box or gray text would cover the jump target line, the jump hint and pill are hidden so they do not collide; the import hint is hidden the same way when a suggestion covers its line. After Esc rejects the suggestion, the hints are shown again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`hideWidgetsIfConflictsWithCppSuggestion` in the prediction service at byte offset ~40766323 and in the import service at ~40807456; called from the preview box code in `showChangesOnTheRight`)
- Pieces:
  1. If a Tab preview box or gray text would cover the jump target line, the jump hint and pill are hidden so they do not collide.
  2. If a suggestion covers the line of an import hint, the import hint is hidden the same way.
  3. After Esc rejects the suggestion, the hidden hints are shown again.

#### AC-7.18 Jump targets in notebooks

- Tag: cursor. Evidence: bundle. Boundary: yes - it points at a proposed edit location.. Cursor: cursor only
- What: When the target is in a `.ipynb` notebook, the jump is treated as an in-editor jump: the line hint and the off-screen pill (AC-7.1, AC-7.2) are used, not the cross-file pill with a file name.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`manuallyCreateCursorPrediction` at byte offset ~40773201: `predictedUri?.path.endsWith(".ipynb")` chooses `createPrediction` over `createMultifileCursorPrediction`)
- Pieces:
  1. When the target is in a `.ipynb` notebook and its line is on screen, the "TAB to jump here" line hint (AC-7.1) is used.
  2. When the target is in a `.ipynb` notebook and its line is off screen, the off-screen pill (AC-7.2) is used, and the cross-file pill with a file name is not.

#### AC-7.19 Import hint "import X (press Tab or ESC)"

- Tag: cursor. Evidence: bundle. Boundary: yes - it offers a suggested import.. Cursor: cursor only
- What: A small chip appears at the end of the line that holds an unresolved name, reading `import {name} (press {key} or ESC)`, for example "import useState (press Tab or ESC)". The key is the label of the accept key as the keybinding service prints it, "TAB" if none is found. The chip has a 4px corner radius, a border in the link color, text 0.85em in the editor font and editor foreground, 80 percent opacity, and a gap of 0.75em before it. It ignores the mouse (pointer events off), so it is keyboard only. Only one import hint shows at a time.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true; Escape
- Settings: none found
- Source: C bundle (import widget class at byte offset ~40778198: node `div.importPredictionWidgetBackground` with class `import-prediction-widget fancy`, `updateTextContent` at ~40780065; CSS `.import-prediction-widget`, `.import-prediction-widget.fancy span`)
- Pieces:
  1. A small chip appears at the end of the line that holds an unresolved name, reading `import {name} (press {key} or ESC)`, for example "import useState (press Tab or ESC)".
  2. The key in the chip is the label of the accept key as the keybinding service prints it, and "TAB" if none is found.
  3. The chip has a 4px corner radius, a border in the link color, text 0.85em in the editor font and editor foreground, 80 percent opacity, and a gap of 0.75em before it.
  4. The chip ignores the mouse, so it is keyboard only.
  5. Only one import hint shows at a time.

#### AC-7.20 When the import hint shows

- Tag: cursor. Evidence: bundle. Boundary: yes - it offers a suggested import.. Cursor: cursor only
- What: The hint is drawn only when all of these hold: the "Imports" switch in Cursor Settings is on (AC-8); the open file is JavaScript or TypeScript (`.js`, `.ts`, `.jsx`, `.tsx`), or Python (`.py`) when Python imports are on; the editor has the focus; an error or warning for an unresolved name sits within 20 lines of the cursor (TypeScript "Cannot find name" errors 2304, 2503 and 2552, Python "X is not defined" or an unknown-name error); and the language service has an add-import fix for it. In the shipped defaults it is limited further to names inside text you accepted from Cursor Tab (or just produced by an inline edit) in the last 30 seconds.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (import service at byte offset ~40782091: `doesSupportFile` at ~40784439, marker-to-name table for TypeScript codes 2304, 2503, 2552 and Python rules, distance constant 20; `markerTouchesGreens` with a 30 second window; defaults `shouldOnlyImportOnAccept:!0` at ~17166266)
- Pieces:
  1. The import hint is drawn only when the "Imports" switch in Cursor Settings is on (AC-8).
  2. The import hint is drawn only when the open file is a supported language (AC-7.25).
  3. The import hint is drawn only when the editor has the focus.
  4. The import hint is drawn only when an error or warning for an unresolved name sits within 20 lines of the cursor (TypeScript "Cannot find name" errors 2304, 2503 and 2552, or a Python "X is not defined" or unknown-name error).
  5. The import hint is drawn only when the language service has an add-import fix for that name.
  6. In the shipped defaults the import hint is limited further to names inside text you accepted from Cursor Tab (or just produced by an inline edit) in the last 30 seconds.

#### AC-7.21 Short import hint after three accepted imports

- Tag: cursor. Evidence: bundle. Boundary: yes - it offers a suggested import.. Cursor: cursor only
- What: After you have accepted three imports (a stored counter that never resets), the chip shrinks to the up-arrow form `↑ import from "module"` (or just `↑ import`), with no key text. The code switches back to the full wording while the pointer is over the chip, but the chip ignores the mouse, so the full wording probably never returns in use (not run).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`updateTextContent` at byte offset ~40780065: `howManyTimesHasUserAcceptedImportPrediction<3||_isHovering`, strings `\u2191 import from "..."` and `\u2191 import`; `mouseenter` and `mouseleave` listeners on the chip; CSS `.import-prediction-widget{pointer-events:none}`)
- Pieces:
  1. After you have accepted three imports, the chip shrinks to the up-arrow form `↑ import from "module"` (or just `↑ import`), with no key text.
  2. The count of accepted imports is stored and never resets.
  3. The short form stays in use, because the full wording only returns while the pointer is over the chip and the chip ignores the mouse (read from code, not run).

#### AC-7.22 Mark on the unresolved name

- Tag: cursor. Evidence: bundle. Boundary: yes - it marks a name for a suggested import.. Cursor: cursor only
- What: While the import hint shows, the unresolved name itself is marked. By default it is a yellow double underline over a warning-colored tint (the stock warning squiggle look, class `squiggly-ai cpp-pending-import-decoration`). If the stored style flag is anything other than "squiggle", it is a blue tint in the link color at 17 percent opacity instead. The style flag is an app storage value (default "squiggle"), not a setting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`showImport` at byte offset ~40798935: decoration class by `cppAutoImportDecorationStyle`; default `cppAutoImportDecorationStyle:"squiggle"` at ~17166266; CSS `.monaco-editor .squiggly-ai`, `.auto-import-suggestion-blue-background`, `.cpp-pending-import-decoration`)
- Pieces:
  1. While the import hint shows, the unresolved name itself is marked.
  2. By default the mark is a yellow double underline over a warning-colored tint, which is the stock warning squiggle look.
  3. If the stored style flag is anything other than "squiggle", the mark is a blue tint in the link color at 17 percent opacity instead, and the flag is an app storage value (default "squiggle"), not a setting.

#### AC-7.23 Accept the import with Tab

- Tag: cursor. Evidence: bundle. Boundary: yes - it applies a suggested import.. Cursor: cursor only
- What: Tab applies the language service's add-import fix for the marked name, so the import line appears the way a Quick Fix would add it. The hint and the mark disappear and a counter of accepted imports goes up by one. An import hint takes priority over every other use of Tab: it is checked before a suggestion or a jump. If the hint's line is not on screen when Tab is pressed, the hint is hidden and Tab carries on with its other jobs.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Source: C bundle (`maybeAcceptShownImport` at byte offset ~40795753, called first in `handleTabAction` at ~40739781; `updateShouldAcceptTabContextKey` sets the key true while `isShowingImportSuggestion()`)
- Pieces:
  1. Pressing Tab applies the language service's add-import fix for the marked name, so the import line appears the way a Quick Fix would add it.
  2. After the import is accepted, the hint and the mark disappear.
  3. After the import is accepted, the counter of accepted imports goes up by one.
  4. An import hint takes priority over every other use of Tab, because it is checked before a suggestion or a jump.
  5. If the hint's line is not on screen when Tab is pressed, the hint is hidden and Tab carries on with its other jobs.

#### AC-7.24 Dismiss the import hint

- Tag: cursor. Evidence: bundle. Boundary: yes - it dismisses a suggested import.. Cursor: cursor only
- What: Esc dismisses the import hint and the key press is swallowed, but only when nothing is selected and not in Vim INSERT mode; when a Tab suggestion is showing, Esc rejects that first (AC-6.17). The hint is also dropped when the editor loses text focus, when the editor closes, when a Tab suggestion covers its line, and when the marked name is scrolled out of view or no longer matches the error. After a dismissal the next candidate, if any, can be shown.
- Commands: none found
- Menus: none found
- Keys: Escape (no Ctrl, Alt or Cmd); not a registered key binding
- Settings: none found
- Source: C bundle (`maybeRejectShownImport` at byte offset ~40796987; blur listener in `showImport` at ~40798935; `hideWidgetsIfConflictsWithCppSuggestion` at ~40807456; Esc branch in `handleKeyDownForCppKeys` at ~40645600)
- Pieces:
  1. Esc dismisses the import hint and the key press is swallowed, but only when nothing is selected and not in Vim INSERT mode.
  2. When a Tab suggestion is showing, Esc rejects that first (AC-6.17).
  3. The import hint is dropped when the editor loses text focus or closes.
  4. The import hint is dropped when a Tab suggestion covers its line.
  5. The import hint is dropped when the marked name is scrolled out of view or no longer matches the error.
  6. After a dismissal, the next candidate, if any, can be shown.

#### AC-7.25 Import hint by language

- Tag: cursor. Evidence: bundle. Boundary: yes - it offers a suggested import.. Cursor: cursor only
- What: Only JavaScript and TypeScript files (`.js`, `.ts`, `.jsx`, `.tsx`) get the hint by default. Python files (`.py`) get it only when Python imports are turned on (the "Auto Import for Python" BETA row in Cursor Settings, or a server flag), and then the hint reacts to the Python language servers' "not defined" and unknown-name errors. In the short form (AC-7.21) the module text is taken from the quick fix title in one of the forms `import from "x"` (TypeScript), `from x import`, `import x` or `import x as y` (Python). No other language gets the hint.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`doesSupportFile` at byte offset ~40784439 with the `.py` extension gated by `pythonEnabled()`; marker tables for sources `ts`, `Pylance`, `basedpyright` and `Pyrefly` near ~40782000; title regex in `updateTextContent` at ~40780065)
- Pieces:
  1. Only JavaScript and TypeScript files (`.js`, `.ts`, `.jsx`, `.tsx`) get the import hint by default.
  2. Python files (`.py`) get the import hint only when Python imports are turned on (the "Auto Import for Python" BETA row in Cursor Settings, or a server flag).
  3. For Python files the hint reacts to the Python language servers' "not defined" and unknown-name errors.
  4. In the short form (AC-7.21) the module text is taken from the quick fix title in one of the forms `import from "x"` (TypeScript), `from x import`, `import x` or `import x as y` (Python).
  5. No other language gets the import hint.


## AC-8 Cursor Tab controls

(cursor). The status bar item, commands and settings to turn Tab on or off, snooze it and tune it. Examples: the "Cursor Tab" status item with a hover panel; Enable, Disable, Toggle and Snooze commands; the Cursor Settings rows (Partial Accepts, Suggestions While Commenting, Imports, Ignored Files); `cursor.cpp.*` settings.

19 features.

#### AC-8.1 Cursor Tab status item and its text states

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A right-side status bar item named "Cursor Tab" (priority 1000) that shows "Cursor Tab", "Cursor Tab (Snoozed)" while a snooze runs, or "Cursor Tab (Ignored)" when the open file matches the cursor ignore rules (snoozed wins if both are true). The text is also the aria label. When Tab is not active for the open file the item is drawn dimmed (opacity .5) with a line through the text (class `copilot-plus-plus-status-off`). That look applies when Tab is off globally, not allowed for the account, off for the file language, the file matches an Ignored Files pattern, the file is blocked by cursor ignore, the file is over 5,000,000 characters, or it is a `.env` file in plaintext. The snoozed state uses its own class and is not struck through. Cross-listed in SS-6 (listed there as a cross-reference only).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: cursor.cpp.disabledLanguages
- Source: C bundle (status entry builder `statusBarElementProps` at byte offset ~40664627, registered as `addEntry(...,"Cursor Tab",1,1e3)`; `isCppEnabledForEditor`; `shouldSkipCppBecauseFileIsTooLarge`; CSS `.copilot-plus-plus-status-off`); V has no hit for `Cursor Tab`, `editor.cpp.` or `cppEnabled`
- Pieces:
  1. A right-side status bar item named "Cursor Tab" shows the text "Cursor Tab" when Cursor Tab is on and the open file is allowed.
  2. While a snooze is running the item text reads "Cursor Tab (Snoozed)" and the text is not struck through.
  3. When the open file matches the cursor ignore rules the item text reads "Cursor Tab (Ignored)", and "(Snoozed)" wins if both are true.
  4. When Cursor Tab is not active for the open file the item is drawn dimmed to half opacity with a line through the text (class `copilot-plus-plus-status-off`).
  5. The dimmed, struck-through look appears when Cursor Tab is off globally or is not allowed for the signed-in account.
  6. The dimmed, struck-through look appears when the file's language is disabled, the file matches an Ignored Files pattern, or the file is blocked by the cursor ignore rules.
  7. The dimmed, struck-through look appears when the file is over 5,000,000 characters or is a `.env` file opened as plaintext.
  8. The item's text is also its aria label, so a screen reader announces the same text that is drawn.

#### AC-8.2 Open the Cursor Tab panel from the status item

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: changed (the item and click command exist in V's status bar; the hover-stays-open flag `dontHideHoverOnClick` is not in V)
- What: Hovering the status item shows a small panel, and clicking it while signed in keeps the panel open (Cursor adds a "do not hide the hover on click" flag that V lacks, and uses the stock `statusBar.entry.showTooltip` command so the click shows the hover). When signed out the click runs Log In to Cursor instead. Panel rows, top to bottom: the "Disable globally" checkbox row, the "Disable for {language}" checkbox row (only when a text editor is active), the "Model" row with a select, and the Snooze row. Each row is a flex line with 4px padding and the label on the left, the control on the right. The panel text changes when Tab is unavailable, see AC-8.8.
- Commands: statusBar.entry.showTooltip; editor.cpp.login
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\statusbar\browser\statusbar.ts (`ShowTooltipCommand`); V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts; C bundle (entry fields `tooltip`, `dontHideHoverOnClick:!0`, `command: signedIn ? statusBar.entry.showTooltip : editor.cpp.login`; CSS `.cpp-status-bar-hover-element`, `.cpp-status-bar-toggle-checkbox`)
- Pieces:
  1. Hovering the "Cursor Tab" status item shows a small panel.
  2. Clicking the status item while signed in keeps the panel open instead of hiding it, using `statusBar.entry.showTooltip`.
  3. Clicking the status item while signed out runs `editor.cpp.login` (Log In to Cursor) instead of showing the panel.
  4. The panel rows from top to bottom are the "Disable globally" checkbox row, the "Disable for {language}" checkbox row, the "Model" row with a select, and the Snooze row.
  5. The "Disable for {language}" row shows only while a text editor is active.
  6. Each panel row is one line with 4px padding, the label on the left and the control on the right.

#### AC-8.3 Turn Cursor Tab on or off

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Three Command Palette commands flip one app-wide on or off flag: "Enable Cursor Tab", "Disable Cursor Tab" and "Toggle Cursor Tab". The same flag is set by the "Disable globally" checkbox in the status panel (checked means off) and by the "Cursor Tab" switch in Cursor Settings (AC-8.10). Turning Tab on, by any of these, also ends a running snooze. The flag is stored in app storage, not in settings.json, so there is no setting id. Using any of them updates the context key `cppEnabled`.
- Commands: editor.action.enableCppGlobally; editor.cpp.disableenabled; editor.cpp.toggle
- Menus: CommandPalette (f1 true, no `when`)
- Keys: none found
- Settings: none found
- Source: C bundle (labels "Enable Cursor Tab", "Disable Cursor Tab", "Toggle Cursor Tab" at byte offset ~40857926 to ~40859500; checkbox code `_renderStatusCheckBox`; the effect that sets `cppEnabledContextKey` and calls `unsnoozeCpp` when the flag turns true); V has no hit
- Pieces:
  1. Running "Enable Cursor Tab" from the Command Palette (`editor.action.enableCppGlobally`) turns Cursor Tab on.
  2. Running "Disable Cursor Tab" from the Command Palette (`editor.cpp.disableenabled`) turns Cursor Tab off.
  3. Running "Toggle Cursor Tab" from the Command Palette (`editor.cpp.toggle`) flips Cursor Tab between on and off.
  4. Ticking the "Disable globally" checkbox in the status panel turns Cursor Tab off and unticking it turns Cursor Tab on.
  5. The "Cursor Tab" switch in Cursor Settings changes the same on or off flag as the commands and the panel checkbox.
  6. Turning Cursor Tab on with Enable, Toggle, or the Cursor Settings switch also ends a running snooze.
  7. Changing the on or off flag updates the context key `cppEnabled`, and the flag lives in app storage so no setting id for it shows in settings.json.

#### AC-8.4 Turn Cursor Tab off for a language

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The panel's second checkbox, "Disable for {language}", adds or removes the language of the active editor (the word "untitled" for untitled files) in the user setting `cursor.cpp.disabledLanguages`. Checked means the language is in the list. The same list can be edited in the Settings editor or settings.json, where it sits in a group titled "Cursor Tab" (configuration id `cpp`): array of strings, default `["markdown","plaintext"]`, description "Disable Cursor Tab for these languages". Changing the editor language rebuilds the row. Tab is also off for `.env` style files opened as plaintext without any setting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: cursor.cpp.disabledLanguages
- Source: C bundle (`registerConfiguration({id:"cpp",title:"Cursor Tab",...})` at byte offset ~38005058; `setCppDisabledForLanguage` writes target 2 (user); `isCppDisabledForLanguage`; `hD_` with the `.env` list)
- Pieces:
  1. Ticking "Disable for {language}" in the status panel adds the active editor's language to the user setting `cursor.cpp.disabledLanguages`, and unticking removes it.
  2. The "Disable for {language}" checkbox shows ticked whenever the active editor's language is in `cursor.cpp.disabledLanguages`.
  3. For an untitled file the language word in the row is "untitled".
  4. Changing the editor's language rebuilds the "Disable for {language}" row for the new language.
  5. Setting `cursor.cpp.disabledLanguages` sits in a settings group titled "Cursor Tab" as an array of strings, default `["markdown","plaintext"]`, with the description "Disable Cursor Tab for these languages".
  6. Cursor Tab is also off for `.env` style files opened as plaintext, without any setting.

#### AC-8.5 Pick the Cursor Tab model from the panel

- Tag: cursor. Evidence: bundle. Boundary: no - the control only. The names behind it and what each one does are out of scope.. Cursor: cursor only
- What: The panel's "Model" row holds a bordered drop-down (an HTML select with a small triangle). The first option reads "auto (default)", followed by the names the app keeps refreshed. Picking one stores it (picking the default clears it) and reloads the Cursor Tab setup. The Cursor Settings page has no Model row (see Unclear).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`_renderCppModelSelect` at byte offset ~40662617; label "Model"; stored flag `cppModel`; default label `auto (default)`)
- Pieces:
  1. The "Model" row of the status panel holds a bordered drop-down with a small triangle.
  2. The first option in the drop-down reads "auto (default)", followed by the names the app keeps refreshed.
  3. Picking an option stores the choice, and picking "auto (default)" clears the stored choice.
  4. The Model drop-down exists only in the status panel, and the Cursor Tab group in Cursor Settings has no Model row.

#### AC-8.6 Snooze Cursor Tab for a set time

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: "Snooze Cursor Tab" opens a quick pick with the placeholder "Select Cursor Tab snooze duration" and six rows, each with the description "Snooze for {label}": "1 minute", "5 minutes", "15 minutes", "30 minutes", "1 hour", "3 hours" (the 3 hours row is coded as 4 hours, see Unclear). The command shows in the palette only while Tab is on and not snoozed, and does nothing if Tab is off. The panel's "Snooze" link runs the same command, and an info icon beside it carries the tooltip "Temporarily disable Cursor Tab suggestions for a specified duration. You can unsnooze at any time." The Snooze row is dimmed and ignores clicks when Tab is off and not snoozed. A snooze turns the on or off flag off, remembers the earlier state, and the status text becomes "Cursor Tab (Snoozed)". If no length is stored the default is 5 minutes.
- Commands: editor.cpp.snooze
- Menus: CommandPalette (f1 true, `when` not cppSnoozed and cppEnabled)
- Keys: none found
- Settings: none found
- Source: C bundle (class for `editor.cpp.snooze` at byte offset ~40858871; duration table; `snoozeCpp`; `_renderSnoozeButtons` at ~40743559; `getSnoozeDuration` default `300*1e3`)
- Pieces:
  1. Running "Snooze Cursor Tab" from the Command Palette (`editor.cpp.snooze`) opens a quick pick with the placeholder "Select Cursor Tab snooze duration".
  2. The quick pick has six rows, each with the description "Snooze for {label}": "1 minute", "5 minutes", "15 minutes", "30 minutes", "1 hour", "3 hours".
  3. Picking a row turns Cursor Tab off for that time and the status item text becomes "Cursor Tab (Snoozed)".
  4. "Snooze Cursor Tab" shows in the Command Palette only while Cursor Tab is on and not snoozed, and does nothing if Cursor Tab is off.
  5. The "Snooze" link in the status panel runs the same command.
  6. An info icon beside the Snooze link has the tooltip "Temporarily disable Cursor Tab suggestions for a specified duration. You can unsnooze at any time."
  7. The Snooze row is dimmed and ignores clicks when Cursor Tab is off and not snoozed.

#### AC-8.7 Unsnooze Cursor Tab and watch the time left

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: "Unsnooze Cursor Tab" ends a snooze at once and puts the on or off flag back to what it was before. It shows in the palette only while snoozed. In the panel the Snooze link turns into "Unsnooze" with a grey "({time} remaining)" count that ticks every second, written "Xh Ym" above an hour or "Xm Ys" below it. When the time runs out Tab turns itself back on, and a snooze that ended while the app was closed is cleared the next time the status item is built. Unticking "Disable globally" also ends a snooze.
- Commands: editor.cpp.unsnooze
- Menus: CommandPalette (f1 true, `when` cppSnoozed)
- Keys: none found
- Settings: none found
- Source: C bundle (class for `editor.cpp.unsnooze`; `unsnoozeCpp`; `formatSnoozedTimeRemaining`; `maybeReenableCppIfSnoozed`; stored flags `cppSnoozed`, `cppSnoozeDuration`, `cppEnabledBeforeSnooze`)
- Pieces:
  1. Running "Unsnooze Cursor Tab" from the Command Palette (`editor.cpp.unsnooze`) ends a snooze at once and puts the on or off flag back to what it was before the snooze.
  2. "Unsnooze Cursor Tab" shows in the Command Palette only while Cursor Tab is snoozed.
  3. While snoozed, the Snooze link in the status panel turns into "Unsnooze".
  4. A grey "({time} remaining)" count next to the Unsnooze link ticks every second, written "Xh Ym" above an hour and "Xm Ys" below an hour.
  5. When the snooze time runs out Cursor Tab turns itself back on.
  6. A snooze that ended while the app was closed is cleared the next time the status item is built.
  7. Unticking "Disable globally" in the status panel during a snooze also ends the snooze.

#### AC-8.8 See why Cursor Tab is not available

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: When Tab cannot be switched in the panel it shows one line instead of the rows. "Please log in" when signed out. "Requires pro (custom model). " followed by an "Upgrade here." link when signed in but Tab is not allowed for the account (the link starts the upgrade flow, not traced). "Cursor Tab is unavailable in Cursor Private Inference" in local mode, and "Cursor Tab: Off under private inference" when private inference turns the Tab feature off. The status text stays "Cursor Tab" and the item is dimmed.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (strings "Please log in", "Requires pro (custom model). ", "Upgrade here.", "Cursor Tab is unavailable in Cursor Private Inference", "Off under private inference" in `statusBarElementProps`)
- Pieces:
  1. When signed out, the status panel shows the one line "Please log in" instead of its rows.
  2. When signed in but Cursor Tab is not allowed for the account, the panel shows "Requires pro (custom model). " followed by an "Upgrade here." link instead of its rows.
  3. Clicking the "Upgrade here." link starts the upgrade flow.
  4. In local mode the panel shows "Cursor Tab is unavailable in Cursor Private Inference".
  5. When private inference turns the Tab feature off the panel shows "Cursor Tab: Off under private inference".
  6. In all of these states the status item text stays "Cursor Tab" and the item is drawn dimmed.

#### AC-8.9 Log in to Cursor from Cursor Tab

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: "Log In to Cursor" runs the sign-in flow. It is in the Command Palette at all times (no `when`), and it is what a click on the status item runs when you are signed out.
- Commands: editor.cpp.login
- Menus: CommandPalette (f1 true)
- Keys: none found
- Settings: none found
- Source: C bundle (class with `ID="editor.cpp.login"`, `LABEL="Log In to Cursor"` at byte offset ~40644045; the status entry `command` field)
- Pieces:
  1. Running "Log In to Cursor" from the Command Palette (`editor.cpp.login`) starts the sign-in flow.
  2. "Log In to Cursor" is in the Command Palette at all times, with no condition, even when already signed in.
  3. Clicking the "Cursor Tab" status item while signed out runs the same sign-in command.

#### AC-8.10 Cursor Tab section in Cursor Settings

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Inside the Cursor Settings "Code Intelligence" tab (id `indexing`) there is a group named "Cursor Tab" (anchor `cursor-settings-cursor-tab`). Its first row is a switch labelled "Cursor Tab" with the description "Context-aware, multi-line suggestions around your cursor based on recent edits". It is the same on or off flag as AC-8.3. The six tuning rows (AC-8.11 to AC-8.16) show only while the switch is on. Under the group a muted line can appear: "CPP and auto-imports are temporarily disabled", "CPP is temporarily disabled" or "Auto-imports are temporarily disabled", when the server side turns those off. The old hidden tab id `tab` redirects to `indexing`, and the Cursor Settings search has an entry "Cursor Tab" (the category also answers to "cursor tab" and "tab"). The tab frame itself is ED-8.30 and the open command is ED-8.19.
- Commands: aiSettings.action.open
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (settings page component with `accessibleLabel:"Cursor Tab"` at byte offset ~38364061; search entry `Kl("indexing","cursor-tab",...)` at ~38093771; redirect map `{...,tab:"indexing"}` at ~33333321; category title map `indexing:"Code Intelligence"`)
- Pieces:
  1. The Cursor Settings "Code Intelligence" tab (id `indexing`) has a group named "Cursor Tab" with the anchor `cursor-settings-cursor-tab`.
  2. The first row of the group is a switch labelled "Cursor Tab" with the description "Context-aware, multi-line suggestions around your cursor based on recent edits".
  3. The six tuning rows (Partial Accepts, Suggestions While Commenting, Whitespace-Only Suggestions, Imports, Auto Import for Python, Ignored Files) show only while the "Cursor Tab" switch is on.
  4. A muted line under the group reads "CPP and auto-imports are temporarily disabled", "CPP is temporarily disabled" or "Auto-imports are temporarily disabled" when the server side turns those off.
  5. The old hidden Cursor Settings tab id `tab` redirects to `indexing`.
  6. The Cursor Settings search has an entry "Cursor Tab", and the category also answers to "cursor tab" and "tab".

#### AC-8.11 Turn partial accepts on or off

- Tag: cursor. Evidence: bundle. Boundary: no - the switch only; the partial accept action itself is AC-6.. Cursor: cursor only
- What: The "Partial Accepts" row, with the description "Accept the next word of a suggestion via {keybinding}" and a small info button beside the label whose tooltip reads "Keybinding: {keybinding}" (aria label "Partial Accepts keybinding: {keybinding}"). The keybinding text is looked up for `editor.action.inlineSuggest.acceptNextWord` and falls back to the string "Alt+]" when none is bound. The switch writes the user setting `cursor.cpp.enablePartialAccepts` (boolean, default false, description "Enable partial accepts for Cursor Tab, using the editor.action.inlineSuggest.acceptNextWord keybinding"). With it on, the context key `cppSuggestion` is true while a Cursor Tab suggestion exists, so the word-accept key also works for Cursor Tab.
- Commands: editor.action.inlineSuggest.acceptNextWord
- Menus: InlineSuggestionToolbar (group primary, order 2, the stock "Accept Word" item that Cursor redefined)
- Keys: CtrlCmd+Right (editor.action.inlineSuggest.acceptNextWord, when editor.writable and (inlineSuggestionVisible or cppSuggestion))
- Settings: cursor.cpp.enablePartialAccepts
- Source: C bundle (row code beside `label:"Partial Accepts"` at byte offset ~38364061; configuration registration at ~38005058; redefined command class at ~24657132; `_bindCppSuggestionKey`)
- Pieces:
  1. The "Partial Accepts" row has a switch and the description "Accept the next word of a suggestion via {keybinding}".
  2. A small info button beside the "Partial Accepts" label has the tooltip "Keybinding: {keybinding}" and the aria label "Partial Accepts keybinding: {keybinding}".
  3. The {keybinding} text is the key bound to `editor.action.inlineSuggest.acceptNextWord`, and reads "Alt+]" when no key is bound.
  4. The switch writes the user setting `cursor.cpp.enablePartialAccepts` (boolean, default false) with the description "Enable partial accepts for Cursor Tab, using the editor.action.inlineSuggest.acceptNextWord keybinding".
  5. With the switch on, the context key `cppSuggestion` is true while a Cursor Tab suggestion exists, so the word-accept key also works on Cursor Tab suggestions.
  6. The inline suggestion toolbar shows an "Accept Word" item (group primary, order 2) for `editor.action.inlineSuggest.acceptNextWord`, bound to `CtrlCmd+Right` while the editor is writable and a suggestion is visible or `cppSuggestion` is true.

#### AC-8.12 Allow Cursor Tab while typing in comments

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The "Suggestions While Commenting" switch, with the description "Allow Tab to trigger while in a comment region". Off by default. It is stored as an app flag (`cppTriggerInComments`), not a settings.json key.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`label:"Suggestions While Commenting"`, stored flag `cppTriggerInComments`)
- Pieces:
  1. The "Suggestions While Commenting" switch has the description "Allow Tab to trigger while in a comment region".
  2. The "Suggestions While Commenting" switch is off by default.
  3. With the switch on, Cursor Tab can trigger while the cursor is in a comment region.
  4. The choice is kept as an app flag, so no settings.json key exists for it.

#### AC-8.13 Allow whitespace-only suggestions

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The "Whitespace-Only Suggestions" switch, with the description "Suggest edits like new lines and indentation that modify whitespace only". Off by default. Stored as an app flag (`cppShowWhitespaceOnlyChanges`).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`label:"Whitespace-Only Suggestions"`, stored flag `cppShowWhitespaceOnlyChanges`)
- Pieces:
  1. The "Whitespace-Only Suggestions" switch has the description "Suggest edits like new lines and indentation that modify whitespace only".
  2. The "Whitespace-Only Suggestions" switch is off by default.
  3. With the switch on, Cursor Tab can suggest edits that change only whitespace, such as new lines and indentation.
  4. The choice is kept as an app flag, so no settings.json key exists for it.

#### AC-8.14 Turn Tab auto imports on or off

- Tag: cursor. Evidence: bundle. Boundary: no - the switch only; the import hint it controls is AC-7.. Cursor: cursor only
- What: The "Imports" switch, with the description "Automatically import necessary modules for TypeScript", or "...for TypeScript and Python" when the server config already includes Python. The row is hidden when the server has turned auto import off and you never set it. Off by default. Switching it off also dismisses an import hint that is on screen. Stored as an app flag (`cppAutoImportEnabled`).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`label:"Imports"`, `hideShownImport`, stored flags `cppAutoImportEnabled` and `backendHasDisabledCppAutoImport`)
- Pieces:
  1. The "Imports" switch has the description "Automatically import necessary modules for TypeScript".
  2. The "Imports" description reads "...for TypeScript and Python" when the server config already includes Python.
  3. The "Imports" row is hidden when the server has turned auto import off and you never set the switch.
  4. The "Imports" switch is off by default.
  5. Switching "Imports" off also dismisses an import hint that is on screen.
  6. The choice is kept as an app flag, so no settings.json key exists for it.

#### AC-8.15 Turn on Auto Import for Python (BETA)

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The "Auto Import for Python" switch, with a small "BETA" badge beside the label and the description "Enable auto import for Python. This is a beta feature." It shows only when the Imports row is shown and on, and the server config does not already include Python. Off by default. Stored as an app flag (`userHasEnabledImportPredictionForPython`).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`label:"Auto Import for Python"` with `children:"BETA"`, stored flag `userHasEnabledImportPredictionForPython`)
- Pieces:
  1. The "Auto Import for Python" switch has a small "BETA" badge beside its label and the description "Enable auto import for Python. This is a beta feature."
  2. The "Auto Import for Python" row shows only when the "Imports" row is shown and switched on.
  3. The "Auto Import for Python" row is hidden when the server config already includes Python.
  4. The "Auto Import for Python" switch is off by default.
  5. The choice is kept as an app flag, so no settings.json key exists for it.

#### AC-8.16 Set ignored file patterns for Cursor Tab

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The "Ignored Files" row, with the description "Glob patterns for files where Cursor Tab will not suggest" and an editable pattern list field (aria label "Ignored Files", placeholder "e.g., *.md, **/generated/**"), aligned to the top of the row. A file that matches is skipped, and the status item shows its dimmed, struck-through look with the text still "Cursor Tab". This is separate from the cursor ignore files, which give the text "Cursor Tab (Ignored)" (ED-8.30 holds the ignore settings). Stored as an app flag (`cppIgnorePatterns`).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`label:"Ignored Files"`, `placeholder:"e.g., *.md, **/generated/**"`, `isCppEnabledForEditor` reading `cppIgnorePatterns`)
- Pieces:
  1. The "Ignored Files" row has the description "Glob patterns for files where Cursor Tab will not suggest".
  2. The row holds an editable pattern list field, aligned to the top of the row, with the aria label "Ignored Files" and the placeholder "e.g., *.md, **/generated/**".
  3. Cursor Tab does not suggest in a file that matches one of the patterns.
  4. For a matching file the status item is dimmed and struck through but its text stays "Cursor Tab", not "Cursor Tab (Ignored)", which is for the cursor ignore files.
  5. The patterns are kept as an app flag, so no settings.json key exists for them.

#### AC-8.17 Use Cursor Tab state in key rules

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Four context keys can be used in `when` clauses in keybindings: `cppEnabled` ("Whether Cursor Tab is currently enabled", default true), `cppSnoozed` ("Whether Cursor Tab is currently snoozed"), `cppSuggestion` (a Cursor Tab suggestion exists and partial accepts is on) and `cpp.shouldAcceptTab` (whether the Tab key should accept; always true when the accept command is rebound away from Tab; the Search view and Problems view also read it to catch Tab). Snooze uses `cppEnabled` and `cppSnoozed` as preconditions and Unsnooze uses `cppSnoozed`. No default key is bound to Enable, Disable, Toggle, Snooze, Unsnooze or Log In. The Tab accept key and the word-accept key are listed here as entry points only, the actions are AC-6. In the Keys list below, entries starting with "when" are context keys, not key chords.
- Commands: editor.cpp.snooze; editor.cpp.unsnooze; editor.action.acceptCursorTabSuggestion; editor.action.inlineSuggest.acceptNextWord
- Menus: none found
- Keys: Tab (editor.action.acceptCursorTabSuggestion, when cpp.shouldAcceptTab is true, weight 500); CtrlCmd+Right (editor.action.inlineSuggest.acceptNextWord, when editor.writable and (inlineSuggestionVisible or cppSuggestion)); when cppEnabled; when cppSnoozed; when cppSuggestion; when cpp.shouldAcceptTab
- Settings: none found
- Source: C bundle (`new Dt("cppSnoozed",...)` and `new Dt("cppEnabled",...)` at byte offset ~40643238; `cppSuggestion` key; accept command class with `kbOpts:{primary:ve.Tab,weight:500}` at ~40745786; `VKa="cpp.shouldAcceptTab"`; Search view and Problems view `onKeyDown` handlers)
- Pieces:
  1. The context key `cppEnabled` ("Whether Cursor Tab is currently enabled", default true) can be used in `when` clauses.
  2. The context key `cppSnoozed` ("Whether Cursor Tab is currently snoozed") can be used in `when` clauses.
  3. The context key `cppSuggestion` is true when a Cursor Tab suggestion exists and partial accepts is on.
  4. The context key `cpp.shouldAcceptTab` says whether the Tab key should accept a suggestion, and is always true when the accept command is rebound away from Tab.
  5. The Search view and the Problems view also read `cpp.shouldAcceptTab` so they can catch the Tab key.
  6. Pressing Tab runs `editor.action.acceptCursorTabSuggestion` when `cpp.shouldAcceptTab` is true.
  7. Pressing `CtrlCmd+Right` runs `editor.action.inlineSuggest.acceptNextWord` when the editor is writable and `inlineSuggestionVisible` or `cppSuggestion` is true.
  8. No default key is bound to Enable, Disable, Toggle, Snooze, Unsnooze or Log In, so a person can bind them in keyboard shortcuts using these keys.

#### AC-8.18 Developer: Copy Recent Cursor Tab Actions

- Tag: cursor. Evidence: bundle. Boundary: yes - the copied text is made of recorded Tab requests and their diffs, which are model output. Only the command is cataloged.. Cursor: cursor only
- What: Developer command "Developer: Copy Recent Cursor Tab Actions" (original English "Copy Recent Cursor Tab Actions"). It copies the newest ten Tab requests (request id and recorded diff) to the clipboard and shows "Copied N Cursor Tab request IDs + diffs to clipboard". When there are none it shows the error "No Cursor Tab actions found". It is in the Command Palette for everyone, no `when`.
- Commands: workbench.action.copyRecentCppActions
- Menus: CommandPalette (f1 true)
- Keys: none found
- Settings: none found
- Source: C bundle (class with `ID="workbench.action.copyRecentCppActions"` at byte offset ~32980061; strings "No Cursor Tab actions found", "Copied ... Cursor Tab request IDs + diffs to clipboard")
- Pieces:
  1. "Developer: Copy Recent Cursor Tab Actions" (`workbench.action.copyRecentCppActions`) is in the Command Palette for everyone, with no condition.
  2. Running it copies the newest ten recorded Cursor Tab requests to the clipboard.
  3. After copying, the message "Copied N Cursor Tab request IDs + diffs to clipboard" shows.
  4. When there are no recorded requests the error "No Cursor Tab actions found" shows.

#### AC-8.19 Developer: Report Cursor Tab Action

- Tag: cursor. Evidence: bundle. Boundary: yes - the first list shows recorded Tab output as row descriptions. Only the pickers are cataloged.. Cursor: cursor only
- What: Developer command "Developer: Report Cursor Tab Action" (the registered original text is "See Recent Cursor Tab Actions"). It opens a quick pick with the placeholder "Select a Cursor Tab request (last 20 requests)", rows named "{request id} - {time}". Picking a row opens a second quick pick with the placeholder "Choose an action": "Report Good", "Report Bad", "Report with Comment" (asks for text in an input box, placeholder "Type your comment here..."), "Copy Request ID" (greyed with "Disabled in privacy mode" when privacy mode is on) and, for developer users only (context key `isDevUser`), "Open in Prompt Quality". Fails with "No Cursor Tab actions found" when the list is empty. The row preview box next to these rows is QA-6.
- Commands: workbench.action.reportCppAction
- Menus: CommandPalette (f1 true)
- Keys: none found
- Settings: none found
- Source: C bundle (class with `ID="workbench.action.reportCppAction"` at byte offset ~32980497; picker function with the two placeholders; `isDevUser` context key)
- Pieces:
  1. "Developer: Report Cursor Tab Action" (`workbench.action.reportCppAction`) is in the Command Palette for everyone, and opens a quick pick with the placeholder "Select a Cursor Tab request (last 20 requests)".
  2. Each row of the first quick pick is named "{request id} - {time}".
  3. When there are no recorded requests the error "No Cursor Tab actions found" shows.
  4. Picking a row opens a second quick pick with the placeholder "Choose an action" and the rows "Report Good", "Report Bad" and "Report with Comment".
  5. "Report with Comment" asks for text in an input box with the placeholder "Type your comment here...".
  6. The "Copy Request ID" row is greyed with "Disabled in privacy mode" when privacy mode is on.
  7. The "Open in Prompt Quality" row shows only for developer users (context key `isDevUser`).
