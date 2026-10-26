Group: AC-4 Snippets. Section: Autocomplete. Date: 2026-10-25. Pass: 2.
Features: 21.

### AC-4.1 Jump between tab stops
- Tag: both
- Evidence: source
- Boundary: no
- What: After a snippet is inserted, Tab moves to the next tab stop and Shift+Tab to the previous one, with each stop selected (or given a cursor when it is empty); stops with the same number are linked and typed into together, and snippet mode ends when the last stop (`$0` or the highest number) is reached.
- Commands: jumpToNextSnippetPlaceholder; jumpToPrevSnippetPlaceholder
- Menus: none found
- Keys: Tab when inSnippetMode and hasNextTabstop and textInputFocus; Shift+Tab when inSnippetMode and hasPrevTabstop and textInputFocus
- Settings: none found
- Cursor: present (both ids, both keys and the context keys inSnippetMode, hasNextTabstop and hasPrevTabstop found; Cursor Tab's accept key is also bound to Tab at weight 500 against 130 here and applies when cpp.shouldAcceptTab is true, so it should win while a Cursor Tab suggestion shows, see AC-6, not run)
- Source: V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (context keys lines 59-61, _updateState lines 230-259, commands and keys lines 347-371); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (OneSnippet.move lines 93-190, next and prev lines 767-781); V\src\vs\editor\contrib\snippet\browser\snippet.md (Tabstops); C bundle (ids jumpToNextSnippetPlaceholder and jumpToPrevSnippetPlaceholder with Tab and Shift+Tab, weight 130; acceptCursorTabSuggestion with weight 500)

### AC-4.2 Placeholder highlight
- Tag: both
- Evidence: source
- Boundary: no
- What: Every tab stop of the active snippet is outlined and tinted, and the final `$0` stop gets its own colors; the marks vanish when snippet mode ends, and only the current stop grows when you type at its edges. The colors are the theme colors editor.snippetTabstopHighlightBackground, editor.snippetTabstopHighlightBorder, editor.snippetFinalTabstopHighlightBackground and editor.snippetFinalTabstopHighlightBorder.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (CSS classes snippet-placeholder and finish-snippet-placeholder with the four theme color ids found in the bundle and the CSS file)
- Source: V\src\vs\editor\contrib\snippet\browser\snippetSession.css (lines 6-20); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (decoration options lines 36-41, active and inactive switching lines 164-185); V\src\vs\platform\theme\common\colors\editorColors.ts (lines 248-263); C bundle (CSS .monaco-editor .snippet-placeholder; color ids)

### AC-4.3 Placeholders with default text
- Tag: both
- Evidence: source
- Boundary: no
- What: A placeholder such as `${1:name}` inserts its default text selected so typing replaces it, placeholders can sit inside other placeholders, a placeholder you deleted is skipped on the next jump, and a placeholder can carry a transform that rewrites its text when you move off it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (same session code: transform on move, collapsed placeholder skip and nesting found in the bundle)
- Source: V\src\vs\editor\contrib\snippet\browser\snippetParser.ts (Placeholder lines 228-283, Transform lines 318-366); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (transform on move lines 105-125, selecting the stop lines 155-165, skip of collapsed stops lines 126-190 and 192-209); V\src\vs\editor\contrib\snippet\browser\snippet.md (Placeholders); C bundle (placeholder decoration code)

### AC-4.4 Pick a choice from a dropdown
- Tag: both
- Evidence: source
- Boundary: no
- What: A choice placeholder such as `${1|one,two,three|}` opens a suggest-style list of its values as soon as the stop becomes active, in the order written; picking a value inserts it and jumps to the next stop, and Shift+Tab goes back to the previous stop even while the list is open.
- Commands: jumpToNextSnippetPlaceholder; jumpToPrevSnippetPlaceholder
- Menus: none found
- Keys: Shift+Tab when inSnippetMode and hasPrevTabstop and inSnippetChoice and textInputFocus (an extra rule at higher weight, so the list's own Shift+Tab does not win)
- Settings: none found
- Cursor: changed (the choice list provider snippetChoiceCompletions and the item command "Go to next placeholder..." are in the bundle; the context key inSnippetChoice and the Shift+Tab rule that uses it are not, so Shift+Tab in a choice list is probably claimed by the suggest list's alternative-accept key, inferred and not run)
- Source: V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (inSnippetChoice line 62, choice provider and next-placeholder command lines 166-220, _handleChoice lines 261-288, Shift+Tab rule lines 370-378); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (activeChoice lines 274-304); V\src\vs\editor\contrib\snippet\browser\snippetParser.ts (Choice lines 285-316); C bundle (provider name, command title message 1539)

### AC-4.5 Variables in snippets
- Tag: both
- Evidence: source
- Boundary: no
- What: `$NAME` and `${NAME:default}` in a snippet are replaced at insert time with the file name or path, the selected text, the current line or word, the clipboard, the date and time, the workspace name, comment markers, a random number or a UUID, and a transform can reshape the value; an unknown name is inserted as plain text and turned into a placeholder.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (the name list in the bundle has 35 of the 39 stock names; CURRENT_MILLISECOND, CURRENT_MILLISECONDS_UNIX, CURRENT_TIMEZONE_NAME and TM_DIRECTORY_BASE are absent)
- Source: V\src\vs\editor\contrib\snippet\browser\snippetVariables.ts (KnownSnippetVariableNames lines 20-60, resolvers from line 62); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (resolver chain lines 566-575); V\src\vs\workbench\contrib\snippets\browser\snippetsFile.ts (unknown name becomes a placeholder lines 62-76); V\src\vs\editor\contrib\snippet\browser\snippet.md (Variables); C bundle (name table ending UUID:!0, four names searched with 0 hits)

### AC-4.6 Nested snippets
- Tag: both
- Evidence: source
- Boundary: no
- What: Inserting a snippet while another one is active (for example from the suggest list or the Insert Snippet picker while a placeholder is selected) places the new snippet's tab stops inside the current stop, and Tab walks through them before continuing with the outer snippet's remaining stops.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (merge path and nesting level code found in the bundle)
- Source: V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (merge branch lines 132-162); V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (OneSnippet.merge lines 306-360, SnippetSession.merge lines 728-765); C bundle (session merge code)

### AC-4.7 Leave a snippet early
- Tag: both
- Evidence: source
- Boundary: no
- What: Esc or Shift+Esc ends snippet mode on the spot and collapses the selection to the main cursor; snippet mode also ends by itself when the cursor leaves every tab stop or when undo goes back before the insert, and the command Accept Snippet (no key, not in the palette) runs through all remaining stops to the end.
- Commands: leaveSnippet; acceptSnippet
- Menus: none found
- Keys: Escape; Shift+Escape when inSnippetMode and textInputFocus (leaveSnippet); none found (acceptSnippet)
- Settings: none found
- Cursor: present (both ids, the Esc keys and the cancel with selection reset found)
- Source: V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (_updateState lines 230-259, finish and cancel lines 290-318, commands lines 380-401); C bundle (ids leaveSnippet and acceptSnippet)

### AC-4.8 Insert at every cursor with matching indentation
- Tag: both
- Evidence: source
- Boundary: no
- What: With several cursors a snippet is inserted at each one, linked tab stops are selected at all cursors together, and every extra line takes the indentation of the line it lands on (normalized to the file's spaces or tabs).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (adjustWhitespace, per-selection edit building and the overwriteBefore and overwriteAfter options found in the bundle)
- Source: V\src\vs\editor\contrib\snippet\browser\snippetSession.ts (adjustWhitespace lines 427-478, per-selection insert lines 505-587, moving all snippets lines 783-790); V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (default options lines 41-50); C bundle (default option object with adjustWhitespace:!0)

### AC-4.9 Tab completion of snippet prefixes
- Tag: both
- Evidence: source
- Boundary: no
- What: With `editor.tabCompletion` set to onlySnippets, typing a snippet prefix and pressing Tab inserts the snippet with no list; when several snippets share the prefix a small list of them opens, and a selection that equals a prefix counts as well. The setting is off by default.
- Commands: insertSnippet; editor.tabCompletionController
- Menus: none found
- Keys: Tab when editorTextFocus and tabDoesNotMoveFocus and not inSnippetMode and hasSnippetCompletions
- Settings: editor.tabCompletion
- Cursor: present (id insertSnippet on Tab with weight 100, context key hasSnippetCompletions and the setting with default off found)
- Source: V\src\vs\workbench\contrib\snippets\browser\tabCompletion.ts (context key line 31, _update and _updateSnippets lines 66-155, performSnippetCompletions lines 157-188, command and key lines 194-206); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (getNonWhitespacePrefix lines 512-535); V\src\vs\editor\common\config\editorOptions.ts (tabCompletion lines 6708-6720); C bundle (id insertSnippet, hasSnippetCompletions, tabCompletion default "off")

### AC-4.10 Snippets in the suggest list
- Tag: both
- Evidence: source
- Boundary: no
- What: Snippets whose prefix matches what you type (matched from any word start or symbol, not only the start of the line) show as rows with the snippet icon, the prefix as the label, the snippet name as the description and "{description or name} ({source})" as the detail, plus a code preview of the body in the details pane; rows with the same prefix get ", {name}" added, and your own snippets sort before extension snippets.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.snippetSuggestions; editor.suggest.showSnippets
- Cursor: present (provider name snippetCompletions, option snippetSuggestions with default inline, setting showSnippets, the markAsUsed command and the "{0}, {1}" label message found)
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetCompletionProvider.ts (row fields lines 34-71, matching and disambiguation lines 90-200); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (provider registration line 235); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (sort order lines 560-575); V\src\vs\editor\contrib\suggest\browser\completionModel.ts (lines 59-75); V\src\vs\editor\common\config\editorOptions.ts (lines 6654-6667); C bundle (class with label and detail fields, _snippet.markAsUsed)

### AC-4.11 Insert Snippet picker
- Tag: both
- Evidence: source
- Boundary: no
- What: Insert Snippet from the Command Palette opens a picker of the snippets for the language at the cursor, grouped under User Snippets, Workspace Snippets and each extension's name, each row showing the prefix (or name) with the description or body beneath, filterable on both, recently used first within a group; the pick is inserted as a snippet, and a keybinding can pass `snippet`, `name` or `langId` to skip the picker or choose another language.
- Commands: editor.action.insertSnippet; editor.action.showSnippets
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (ids and alias, labels "Insert Snippet", "Select a snippet", "User Snippets", "Workspace Snippets" and "No snippet available" found)
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\insertSnippet.ts (action lines 47-150, args lines 19-45); V\src\vs\workbench\contrib\snippets\browser\snippetPicker.ts (lines 17-103); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (recency sort lines 331-345); V\src\vs\workbench\contrib\snippets\browser\snippets.contribution.ts (alias lines 26-27); C bundle (ids, NLS labels)

### AC-4.12 Hide an extension snippet from IntelliSense
- Tag: both
- Evidence: source
- Boundary: no
- What: In the Insert Snippet and Surround pickers every extension snippet has an eye-closed button "Hide from IntelliSense" that removes it from the suggest list and tags the row "(hidden from IntelliSense)", and the button then becomes an eye "Show in IntelliSense"; the choice is kept per profile.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (labels "Hide from IntelliSense", "Show in IntelliSense", "(hidden from IntelliSense)" and the storage key snippets.ignoredSnippets found; the key is stored state, not a setting)
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetPicker.ts (buttons lines 56-96); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (SnippetEnablement lines 132-168, isEnabled and updateEnablement lines 245-251); C bundle (NLS labels, snippets.ignoredSnippets)

### AC-4.13 Surround with Snippet
- Tag: both
- Evidence: source
- Boundary: no
- What: With text selected, Surround with Snippet... in the Command Palette opens the picker limited to snippets whose body uses the selection (`$TM_SELECTED_TEXT` or `$SELECTION`) and wraps the selected text with the pick; it does nothing when no snippet qualifies.
- Commands: editor.action.surroundWithSnippet
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (id and label "Surround with Snippet..." found)
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\surroundWithSnippet.ts (lines 20-72); V\src\vs\workbench\contrib\snippets\browser\snippetsFile.ts (usesSelection lines 82-86 and 142-144); C bundle (id editor.action.surroundWithSnippet with title message 9753)

### AC-4.14 Fill File with Snippet
- Tag: both
- Evidence: source
- Boundary: no
- What: Fill File with Snippet in the Command Palette lists the file-template snippets (those marked `isFileTemplate`) that match the open file, grouped by language, and replaces the whole file with the pick after switching the file to that language.
- Commands: workbench.action.populateFileFromSnippet
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (id and label "Fill File with Snippet" found)
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\fileTemplateSnippets.ts (lines 19-112); C bundle (id workbench.action.populateFileFromSnippet, NLS label)

### AC-4.15 Surround With snippets in the light bulb
- Tag: both
- Evidence: source
- Boundary: no
- What: With text selected, the light bulb menu lists up to four matching snippets by name under a "Surround With" header (when headers are shown) plus a "More..." row that opens Surround with Snippet...; choosing a snippet wraps the selection. Turning `editor.snippets.codeActions.enabled` off removes these rows.
- Commands: editor.action.surroundWithSnippet
- Menus: none found
- Keys: none found
- Settings: editor.snippets.codeActions.enabled
- Cursor: present (the setting, the provider limit constant _MAX_CODE_ACTIONS, the overflow row and the label "More..." found; the group header title found in NLS)
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetCodeActionProvider.ts (SurroundWithSnippetCodeActionProvider lines 22-67, enable switch lines 125-150); V\src\vs\workbench\contrib\snippets\browser\snippets.contribution.ts (setting lines 36-48); V\src\vs\editor\contrib\codeAction\browser\codeActionMenu.ts (group title line 31); C bundle (editor.snippets.codeActions.enabled, _MAX_CODE_ACTIONS, NLS "Surround With")

### AC-4.16 Start with a snippet in an empty file
- Tag: both
- Evidence: source
- Boundary: no
- What: In an empty file the light bulb offers "Start with: {snippet name}" for up to four file-template snippets of that language (under the same "Surround With" header) plus a "Start with Snippet" row that opens Fill File with Snippet; choosing one fills the file.
- Commands: workbench.action.populateFileFromSnippet
- Menus: none found
- Keys: none found
- Settings: editor.snippets.codeActions.enabled
- Cursor: present (labels "Start with: " and "Start with Snippet" found in NLS, same provider code and setting)
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetCodeActionProvider.ts (FileTemplateCodeActionProvider lines 69-123); C bundle (NLS labels, provider code with the same limit and overflow row)

### AC-4.17 Configure Snippets picker
- Tag: both
- Evidence: source
- Boundary: no
- What: Configure Snippets (File > Preferences > Snippets, the Manage gear's Snippets row, or the Command Palette) opens a picker with "Existing Snippets" (your snippet files, each with the languages it covers, `.code-snippets` files first) and "New Snippets" (New Global Snippets file..., one New Snippets file for '{folder}'... per workspace folder, and every language that has no file yet); picking an existing file opens it in an editor tab.
- Commands: workbench.action.openSnippets
- Menus: MenubarPreferencesMenu (group 2_configuration, order 5); GlobalActivity (group 2_configuration, order 5); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (id, short title "Snippets", both menu registrations and the picker labels found; in Cursor's default layout the Manage gear is not drawn, so the menu bar and palette are the routes, see the Cursor activity bar notes and AB-4.18)
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\configureSnippets.ts (computePicks lines 37-142, action and menus lines 245-262, pick list lines 264-314); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (snippet files lines 263-266); C bundle (id workbench.action.openSnippets with title message 9748, NLS "Select Snippets File or Create Snippets", "Existing Snippets", "New Snippets")

### AC-4.18 Create a snippets file
- Tag: both
- Evidence: source
- Boundary: no
- What: Picking New Global Snippets file... or New Snippets file for '{folder}'... asks for a name ("Type snippet file name", rejecting empty, invalid and existing names), creates a `.code-snippets` file with a commented template in the user snippets folder or the folder's `.vscode` folder and opens it; picking a language with no file yet writes `<language>.json` in the user snippets folder with a language template and opens it.
- Commands: workbench.action.openSnippets
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (name prompt, validation messages and both templates found; the paragraph and example about restricting snippets with include and exclude patterns is not in the Cursor templates)
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\configureSnippets.ts (createSnippetFile lines 144-210, createLanguageSnippetFile lines 212-243, create branches lines 273-314); C bundle (template text starting "Place your", NLS "Type snippet file name", "is not a valid file name"; search for "include/exclude patterns" returned 0 hits)

### AC-4.19 Edit snippet files in the editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Snippet files open as normal editor tabs in the "Code Snippets" language (`.code-snippets` files and the JSON files in the user snippets folder), which is JSON with comments, with property completion and error squiggles from a schema, an "Empty snippet" skeleton in the completion list, suggestions that also open inside strings, and snippets that reload as soon as the file is saved.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: [snippets] editor.quickSuggestions; [snippets] editor.suggest.insertMode
- Cursor: changed (language id snippets, file match rules, both schema ids, the [snippets] defaults and the reload on save found; the schema has the properties prefix, isFileTemplate, body and description but not include or exclude)
- Source: V\extensions\json\package.json (language lines 91-101, grammar lines 123-126); V\extensions\configuration-editing\package.json (schema matches lines 143-162); V\extensions\json-language-features\package.json (language defaults lines 164-169); V\src\vs\workbench\contrib\snippets\browser\snippets.contribution.ts (schemas lines 52-136); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (folder watch and reload lines 451-502); C bundle (schema object with prefix, isFileTemplate, body, description only)

### AC-4.20 Choose which languages and prefixes a snippet answers to
- Tag: both
- Evidence: source
- Boundary: no
- What: A snippet in `typescript.json` applies to TypeScript only, one in a `.code-snippets` file applies to the languages in its `scope` field (to every language when `scope` is empty), and one in a workspace folder's `.vscode` folder shows only in that workspace under "Workspace Snippets"; `prefix` may be one string or a list, and a snippet with no prefix never appears in the suggest list and is reached only from the pickers and the light bulb.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (isGlobalSnippets test on .code-snippets, scope select code, includeNoPrefixSnippets option and the scope schema text found)
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetsFile.ts (file kinds lines 194-258, scopes and prefixes lines 296-371); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (workspace and user folders lines 420-502, no-prefix filter lines 311-315); V\src\vs\workbench\contrib\snippets\browser\snippets.contribution.ts (scope lines 109-131); C bundle (class fields isFileTemplate, scopes, name, prefix; select method)

### AC-4.21 Limit a snippet to certain files
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A snippet can carry `include` and `exclude` lists of globs so it only shows for matching files, with exclude winning; a pattern with a slash is matched against the absolute path and any other pattern against the file name, ignoring case.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing (the Snippet class in the bundle has no include or exclude fields, the schema has no such properties, and the NLS text "to include the snippet for specific files" has 0 hits)
- Source: V\src\vs\workbench\contrib\snippets\browser\snippetsFile.ts (fields lines 118-119, isFileIncluded lines 146-170, parsing lines 323-340); V\src\vs\workbench\contrib\snippets\browser\snippets.contribution.ts (schema lines 74-88); V\src\vs\workbench\contrib\snippets\browser\snippetsService.ts (filter lines 322-325); C bundle (Snippet constructor field list, schema object)

## Handed off
- The Snippets row in File > Preferences and in the Manage gear menu: TB-1 and AB-4 (AB-4.18 already lists the Manage row, confidence high; AC-4.17 lists only the picker behind them).
- The Snippets page rows in the Settings editor for `editor.snippetSuggestions`, `editor.tabCompletion` and `editor.snippets.codeActions.enabled`: ED-8 (confidence medium, the settings are attached above but the Settings editor page that shows them was not read here).

## Unclear
- Overlap with AC-1: AC-1.12 lists snippet rows with `editor.snippetSuggestions` and `editor.suggest.showSnippets`, and AC-4.10 lists the same two settings because the parent asked for them here. AC-4.10 covers what a snippet row contains and how it matches, AC-1.12 covers placement. A later tool should attach each setting to one owner.
- Two settings react to snippet mode and are owned elsewhere: `editor.suggest.snippetsPreventQuickSuggestions` (AC-1.2) and `editor.inlineSuggest.suppressInSnippetMode` (AC-2.34). Neither is repeated in a feature here.
- Emmet: the parent said Emmet is ED-2, but the ED-2 notes hand abbreviation suggestions and expansion to AC-1 and AC-1's own notes treat it as pass 1 context. No Emmet feature is listed in AC-4. Someone should confirm which group owns it.
- Newer than 1.128 (seen only in N, not listed as a feature because it tweaks AC-4.17): the Configure Snippets picker moves the language of the active editor to the top of each group, adds a `*` to its description and a tooltip "This file type was detected from the current file.", and swaps label and description of the not-yet-created language rows (`N\src\vs\workbench\contrib\snippets\browser\commands\configureSnippets.ts`). Neither the label nor the tooltip is in the Cursor bundle. Other N differences in the snippets folder are internal (pending work tracking), and the editor snippet folder and the snippet options in `editorOptions.ts` are identical to V.
- Cursor Tab and the Tab key: the Cursor Tab accept command is bound to Tab with weight 500 and the snippet jump with weight 130, so while a Cursor Tab suggestion is showing Tab should accept it rather than jump to the next tab stop. This comes from reading the registrations only, nothing was run, and the full rule set is in AC-6.
- The Shift+Tab behavior inside a choice list (AC-4.4) is inferred from the missing inSnippetChoice rule, not run.
- Extension-contributed snippets (the `snippets` contribution point) show in the pickers under the extension's name. The contribution point itself is extension machinery and is out of scope, so no feature covers it.
- `acceptSnippet` has no default key and no palette label, so the only way to reach it is a user keybinding or an extension command call; AC-4.7 lists it for completeness.
- No Cursor-only snippet UI was found: searches for snippet names with cursor, cpp, composer or aichat prefixes returned nothing.
