Group: AC-6 Cursor Tab suggestions. Section: Autocomplete. Date: 2026-10-25. Pass: 2.
Features: 30.

### AC-6.1 Gray inline text at the cursor
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the text is a generated suggestion.
- What: When a suggestion only adds characters (no removal, up to 20 character changes and 20 word changes) and the new text lands at or after the cursor, the new text shows as dimmed, syntax colored gray text inside the line at the insertion point. It is upright, has no border, and the existing code stays untouched. If any of the insertion would land before the cursor, the editor uses the preview box (AC-6.3) instead. The stock theme colors `editorGhostText.*` also apply because the text reuses the stock `ghost-text` classes. The Command Palette also has an "Update Ghost Text" command that redraws it.
- Commands: editor.action.updateGhostText
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module, byte offset ~24936000 to ~24960000: decoration with `inlineClassName` "ghost-text-decoration syntax-highlighted cursor-ghost-text", line class `cpp-inline-suggestion`, helper that builds inline and full-line pieces; CSS `.monaco-editor .syntax-highlighted.cursor-ghost-text` grayscale 40% and opacity .6, `.monaco-editor .cursor-ghost-text` no border and normal style); V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css (the stock classes it reuses)

### AC-6.2 Gray added lines under the cursor line
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the lines are a generated suggestion.
- What: When a suggestion adds whole new lines and passes the same test as AC-6.1, the new lines appear as dimmed, syntax colored gray lines under the current line. The editor opens blank room for them (they are not real text). There is no box and no margin label in this shape. Several added lines show as a stack in the order they will be inserted.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module: line renderer that writes `div.suggest-preview-text` rows with `view-line`, and the view zone call `addZone({afterLineNumber, heightInLines, domNode})`; CSS `.syntax-highlighted.cursor-ghost-text`)

### AC-6.3 Preview box for edits to existing text
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the box content is a generated suggestion.
- What: When a suggestion changes or removes existing text, would insert before the cursor, or is large, a small bordered box appears to the right of the code, level with the first changed line. It shows the new version of the changed lines in the editor's own font, colors and language. The box uses the suggest list background and border colors and a 2px corner radius. The inner editor is read only and drawn bare: no line numbers, folding, minimap, indent guides, current-line highlight, visible scroll bars, whitespace dots or hover.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module: content widget `div.cursorGhostTextWidget` with inner `cppOuter`, `cppInner`, `cppButtonContainer` and `div.cppPreviewBoxDomNode` holding a small embedded editor; CSS `.cursorGhostTextWidget`, `.cppPreviewBoxDomNode .monaco-editor`)

### AC-6.4 Multi-line preview box size and position
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the box content is a generated suggestion.
- What: The box is one row tall for each changed line and as wide as the longest changed line (a tab counts as the tab size). Its left edge starts 24px past the end of the longest line in the changed range, so it never covers that code. Unless `editor.wordWrap` is exactly "on", the box is held at least 2 columns inside the right edge of the editor; with word wrap "on" it is not held back. The box scrolls its inner editor so the changed lines are the ones you see.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.wordWrap
- Cursor: cursor only
- Source: C bundle (ghost text widget module: box placement code that computes a left margin of 24px plus the column gap, and sets the box width and height from the line count; reads editor option 137)

### AC-6.5 Green tint on new words in the box
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the box content is a generated suggestion.
- What: Inside the preview box, the characters or words the suggestion adds carry the diff editor's inserted text background (a green tint). Words that stay the same carry no tint.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module: decoration class `ghost-text-decoration-inline-add` on the added ranges; CSS `.monaco-editor .ghost-text-decoration-inline-add` using the theme color `diffEditor.insertedTextBackground`)

### AC-6.6 Red tint on the code that will change
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the tint marks a generated suggestion.
- What: While the preview box shows, the words in your real code that the suggestion would replace or remove are tinted red (the "testing failed" icon color at 17 percent opacity). The tint goes away with the box.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module: `showChangesOnTheRightAndMaybeShowReds` adds decorations with class `cpp-suggestion-red-background`; CSS `.cpp-suggestion-red-background` using the theme color `testing.iconFailed`)

### AC-6.7 Deletion display in the box
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the box content is a generated suggestion.
- What: When a suggestion only deletes one stretch of text, the box shows the lines that would go, dimmed (opacity .2) and struck through, with the box padding removed. Nothing new is shown. If the deletion is in several separate places the code draws no box in this path.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module: the branch for a deletion result, rows `div.cppCodeLine` built with `ghost-text-decoration-remove` for leading space and `ghost-text-decoration-inline-remove` for the text; CSS `.ghost-text-decoration-remove`, `.ghost-text-decoration-inline-remove` with line-through)

### AC-6.8 View and Accept strip on the box
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the buttons act on a generated suggestion.
- What: Moving the pointer over the preview box adds a thin footer under the code with two small buttons: "View" and "Accept". "Accept" carries a tiny bordered "Tab" chip (always the word Tab, not read from your keybinding). The buttons are dimmed (opacity .6) and light up on hover. The footer is removed when the pointer leaves the box. Only the preview box has this footer, not the gray text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module, byte offset ~24940864: `createActionButtons` builds `div.cpp-suggestion-action-buttons` with `button.cpp-suggestion-action-button` "View" and "Accept" and `span.cpp-suggestion-tab-button` "Tab", added on `mousemove` and removed on `mouseleave`; CSS `.cpp-suggestion-action-button`, `.cpp-suggestion-tab-button`)

### AC-6.9 Accept by clicking the Accept button
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it applies a generated suggestion.
- What: Pressing the mouse button on "Accept" in the box footer applies the whole suggestion, the same result as the Tab key (AC-6.14), without needing keyboard focus on the key. The click does not move the editor cursor first.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module, `createActionButtons`: `mousedown` handler on the Accept button calls the cpp service full accept after `stopPropagation`)

### AC-6.10 View button opens the diff peek
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the peek shows a generated suggestion.
- What: Pressing the mouse button on "View" in the box footer opens the "Cursor Tab Preview" diff peek (AC-6.11) for that suggestion, showing the whole file before and after.
- Commands: editor.action.setCppDiffPeekView
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module, `createActionButtons`: `mousedown` on View builds original and new file text and runs `editor.action.setCppDiffPeekView`; the command is registered at byte offset ~40857246 with title "Set Cursor Tab Diff Peek View" and no palette entry)

### AC-6.11 "Cursor Tab Preview" diff peek
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the peek shows a generated suggestion.
- What: An in-editor peek panel with the title text "Cursor Tab Preview" opens under the suggestion's last line. It holds a read only inline diff (not side by side) of the file before and after, with sticky scroll off, scrolled to the first change. It has a frame, no arrow, can be resized, and starts 10 lines tall. The title bar carries the actions in AC-6.12 and AC-6.13. A second command, "Configure Cursor Tab Diff Peek View", changes the diff editor options while a peek is open; it has no palette entry and no UI.
- Commands: editor.action.setCppDiffPeekView; editor.action.configureCppDiffPeekView
- Menus: cpp-diff-peek-view/title
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (peek widget class with `showFrame:!0,showArrow:!1,isResizeable:!0`, title node `cpp-peek-view-title-text` "Cursor Tab Preview", menu id `cpp-diff-peek-view/title`, height constant 10, at byte offset ~24926200 to ~24929500; commands at ~40857246); V\src\vs\editor\contrib\peekView\browser\peekView.ts (base peek widget)

### AC-6.12 Accept from the peek title bar
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it applies a generated suggestion.
- What: The peek title bar has one action, a check mark with the tooltip "Accept Cursor Tab Suggestion" (icon id `accept-cpp-suggestion`, menu order 1). Clicking it applies the suggestion shown in the peek and clears the peek suggestion. The action has no Command Palette entry and no key.
- Commands: editor.action.acceptCppSuggestion
- Menus: cpp-diff-peek-view/title
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (action class at byte offset ~40856497: id `editor.action.acceptCppSuggestion`, label "Accept Cursor Tab Suggestion", icon `Oe.check`, `menu:{id:cpp-diff-peek-view/title,order:1}`, no `f1`)

### AC-6.13 Close the diff peek
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the peek shows a generated suggestion.
- What: The peek closes with the stock "Close" X action in its title bar. It also closes by itself when the active editor group changes, when the editor or its file changes, when the editor is disposed, when a new Tab suggestion appears, or when the stored peek suggestion is cleared.
- Commands: peekview.close
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (peek widget class constructor: disposers on `onDidChangeActiveGroup`, editor `onDidDispose`, `onDidChangeModel`, `peekViewSuggestion` and `suggestion` events); V\src\vs\editor\contrib\peekView\browser\peekView.ts (stock `peekview.close` action with label "Close")

### AC-6.14 Accept with Tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it applies a generated suggestion.
- What: Tab applies the whole suggestion and clears it and its tints; for a plain one-line insertion the cursor ends right after the inserted text. When Tab is pressed the editor tries these in order: accept a shown import hint (AC-7.23), accept the suggestion for the current spot, accept the nearest extra suggestion that is within 5 lines of the cursor (AC-6.27), or jump to the next edit if a jump hint shows (AC-7.5). The Cursor Tab key is bound at weight 500, above the stock suggest list Tab (weight 190 in V), so a showing Tab suggestion wins over the suggest list. The key is only active while `cpp.shouldAcceptTab` is true. If a cross-file jump pill is showing and its code preview is not yet visible, the first Tab only reveals the preview (AC-7.8).
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Cursor: cursor only
- Source: C bundle (accept action class at byte offset ~40745774: id `editor.action.acceptCursorTabSuggestion`, label "Accept Cursor Tab Suggestion", `precondition: cpp.shouldAcceptTab == true`, `kbOpts {primary: Tab, weight: 500}`; `handleTabAction` at ~40739781 with the order above; `updateShouldAcceptTabContextKey` at ~40742000); V\src\vs\platform\keybinding\common\keybindingsRegistry.ts (weight tiers); V\src\vs\editor\contrib\suggest\browser\suggestController.ts (stock Tab)

### AC-6.15 Accept key follows your keybinding
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the key accepts a generated suggestion.
- What: The accept action is a normal command, so the Tab binding can be changed in Keyboard Shortcuts. When it is no longer bound to Tab, `cpp.shouldAcceptTab` stays true and the new key works whenever a suggestion or jump exists. The key chips on the hint widgets and the margin label follow the new binding (upper case in the jump hints, lower case in the margin label, label text as the keybinding service prints it). The "Tab" chip on the box's Accept button is fixed text and does not follow the binding, and the Search and Problems views still listen for the Tab key itself.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`updateShouldAcceptTabContextKey` reads `lookupKeybinding(editor.action.acceptCursorTabSuggestion)` and sets `boundToTab`; hint widgets call `lookupKeybinding(...).getLabel()`; `createActionButtons` writes the literal "Tab"; Search view tree and Problems view table `onKeyDown` check `ve.Tab`)

### AC-6.16 Partial accept, one word at a time
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it applies part of a generated suggestion.
- What: With the setting "Enable partial accepts" on, Ctrl+Right (Cmd+Right on Mac) takes only the next word of the Cursor Tab suggestion instead of all of it. The word break follows the file language's word rules. The rest of the suggestion is re-compared with the new text, and the accepted part is tinted like AC-6.24 (the remainder staying on screen is read from the code path, not run). With the setting off, `cppSuggestion` is false, the command does not apply to Cursor Tab, and Ctrl+Right is the normal word move. Cursor redefined the stock "Accept Next Word Of Inline Suggestion" command for this: its precondition adds `cppSuggestion`, it asks the Cursor Tab accept first, and falls back to the stock word accept. The stock toolbar item for this command is for the stock ghost text, which is hidden while a Tab suggestion shows (AC-6.26), so no toolbar button is drawn for Tab suggestions (not run).
- Commands: editor.action.inlineSuggest.acceptNextWord
- Menus: InlineSuggestionToolbar
- Keys: Ctrl+Right (Cmd+Right on Mac) when the editor is writable and (inlineSuggestionVisible or cppSuggestion)
- Settings: cursor.cpp.enablePartialAccepts
- Cursor: changed (stock command and key, redefined with the `cppSuggestion` condition and a Cursor Tab first path)
- Source: C bundle (command class at byte offset ~24656892: id `editor.action.inlineSuggest.acceptNextWord`, `kbOpts {primary: CtrlCmd|RightArrow, weight: 101}`; context key `cppSuggestion` declared at ~24651804; `acceptNextWord` and `makeEditsForNextWordAccept` in the cpp service at ~40669263 and ~40722906; setting registered with default false at ~38004908); V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (stock command, line 141)

### AC-6.17 Reject with Esc
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it dismisses a generated suggestion.
- What: Pressing Esc (no Ctrl, Alt or Cmd held, Shift allowed) while a suggestion shows clears it and all of its tints. Afterward a waiting "tab to jump here" hint or pill (AC-7.1, AC-7.2) and a waiting import hint (AC-7.19) are shown again. The key press is not swallowed, so Esc also does its normal job (for example closing a widget). Esc is a raw key listener in the cpp service, not a registered command or key binding, so it does not appear in Keyboard Shortcuts. With no Tab suggestion showing, Esc instead dismisses an import hint first and a jump hint second (AC-7.13, AC-7.24).
- Commands: none found
- Menus: none found
- Keys: Escape (no Ctrl, Alt or Cmd) while a Tab suggestion shows; not a registered key binding
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`handleKeyDownForCppKeys` in the cpp service, byte offset ~40645600 to ~40648200: checks `ue.key==="Escape"` with no ctrl, alt or meta, then `rejectAndResetAllCppSuggestions`, `maybeShowHintLineWidget`, `showCorrectUI`)

### AC-6.18 Type along with the suggestion
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the suggestion is generated content.
- What: Typing the characters the suggestion would insert next keeps the suggestion showing, and it shrinks as you go. When the text in the suggestion range finally equals the suggestion, it clears. Typing something that does not follow the suggestion rejects it, and the editor may then show a different one. An edit that adds a newline or deletes text and does not follow the suggestion always rejects it. This is how the gray text of AC-6.1 gets "typed through", as with ordinary inline suggestions.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`continueModelChangeListener` at byte offset ~40679000: compares the range text with the suggestion text, `isOnShortestEditPath`, and `isGuaranteedCacheMiss` which treats any newline or removed text as a mismatch)

### AC-6.19 Move away to dismiss
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it dismisses a generated suggestion.
- What: Moving the cursor or selection so it no longer touches the suggestion's range (a click elsewhere, the arrow keys, a jump by other means) rejects the suggestion and clears its tints. A waiting jump hint is shown again. Moves made by Cursor Tab itself (its own jump, peek and next-suggestion moves) do not count as moving away.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (cpp service `onDidChangeCursorPosition` listener, byte offset ~40673000 to ~40676000: intersects the selection with the suggestion range and calls `rejectAndResetAllCppSuggestions` when it does not; ignores sources `cpp-peek`, `cpp-next-suggestion`, `cpp-revert` and `cursorPrediction`)

### AC-6.20 Box hides on focus loss or file switch
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it hides a generated suggestion.
- What: The preview box and gray text are hidden when the editor loses text focus, when the editor shows a different file, and when text changes while the box's anchor line is the last line of the file. They are also hidden if the pointer enters the box with the left button held down. Hiding the box only fades it out and clears its pieces; the suggestion state itself is handled by AC-6.17 to AC-6.19.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module constructor, byte offset ~24936500: listeners `onDidBlurEditorText`, `onDidChangeModel`, `onDidChangeModelContent`, and `mouseenter` with `(buttons&1)===1` call `hideAllInEditor_doesntChangeModel`)

### AC-6.21 Tab indents on a blank line instead of accepting
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it decides whether a generated suggestion is accepted.
- What: When the cursor is on a line that holds only whitespace and the suggestion's next characters at the cursor begin with whitespace, Tab inserts an ordinary indent instead of accepting, so you can indent a new line. It applies only before a jump has been accepted for that suggestion, and not in Vim modes other than INSERT. The Tab accept key is switched off by the rule (`cpp.shouldAcceptTab` false) so the stock Tab runs.
- Commands: none found
- Menus: none found
- Keys: Tab
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`shouldTabInsteadOfAccepting` in the cpp service at byte offset ~40668361; helper that tests for a whitespace-only line at ~40639266; `updateShouldAcceptTabContextKey` at ~40742000)

### AC-6.22 Selection, Shift+Tab and Vim rules
- Tag: cursor
- Evidence: bundle
- Boundary: yes - they decide whether a generated suggestion is accepted.
- What: With a selection that spans more than one line, Tab is not captured, so it indents the selection as usual (the Search view and Problems list handlers are the exception, see AC-6.23). Shift+Tab is never captured. When the Vim extension's status bar item shows any mode other than INSERT, the multi-line selection rule and the blank line rule (AC-6.21) are skipped, so Tab can accept. Accept is also off when the suggest list is open while the editor does not have focus, or when no editor or file is found.
- Commands: none found
- Menus: none found
- Keys: Tab; Shift+Tab
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`updateShouldAcceptTabContextKey` at byte offset ~40742000, `handleKeyDownForCppKeys` ignoring `Tab` with `shiftKey`, `isInVimNonInsertMode` reading the status bar entry `vscodevim.vim.primary`)

### AC-6.23 Accept while focus is in Find, Search, Problems or Peek References
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it applies a generated suggestion.
- What: A showing suggestion can still be accepted with Tab when keyboard focus is not in the editor text but in one of four places tied to that editor: the Find box (with a non-empty selection in the editor), a focused match in the Search results, the Problems list, or the preview editor inside a Peek References widget. After accepting from the Find box, focus returns to the Find input without selecting its text. The Find case is controlled by a stored flag (`cppInCmdF`, default on) that is not a setting.
- Commands: none found
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`isTextFocusedOrSimilarlyFocused`, `isFindFocused`, `isProblemFocused`, `isReferenceFocused`, `isSearchViewFocused` at byte offset ~40700846 to ~40702000; Search view tree `onKeyDown` at ~35984362 and Problems widget `onKeyDown` at ~36607328, both calling `handleTabAction` with "searchView" or "problemsView"; default `cppInCmdF:!0` at ~17166508)

### AC-6.24 Green tint on accepted text
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it marks generated text.
- What: After a suggestion (or a word of it) is accepted, the text that was just inserted or changed is tinted green (the "testing passed" icon color at 12 percent opacity) so you can see what landed. The tint stays through typing and is cleared when the cursor moves again while no suggestion is showing (read from the code path, not run).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`updateSuggestionGreenHighlights` at byte offset ~40722906 with class `cpp-suggestion-green-background`; clearing in the `onDidChangeCursorPosition` listener via `didShowGreenHighlights`; CSS `.cpp-suggestion-green-background` using the theme color `testing.iconPassed`)

### AC-6.25 Next suggestion right after an accept
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the follow-up is another generated suggestion.
- What: After an accept succeeds, a follow-up can show straight away without typing: another suggestion for the spot, or a jump hint or pill for the next edit (AC-7.1, AC-7.2). If the accept did nothing (nothing was applied), no follow-up is shown.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`cleanupAfterAcceptSuggestion` at byte offset ~40729009 with `shouldDisplayFollowup`, a stored follow-up lookup and `displayNextActionIfAvailable`)

### AC-6.26 Stock gray suggestion yields to Cursor Tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it swaps one generated suggestion for another.
- What: While a Cursor Tab suggestion is showing, the stock VS Code gray inline suggestion and its hover toolbar are hidden, so only one gray suggestion is on screen. When the Cursor Tab suggestion is gone, the stock one is unhidden and asked for again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.enabled
- Cursor: cursor only
- Source: C bundle (cpp service methods `disableAndHideCopilotSuggestion` at byte offset ~40725849 and `reallowCopilotIfWePreviousHidCopilotSuggestions` at ~40667467, which re-triggers the stock controller); V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (the stock controller it drives)

### AC-6.27 Several boxes at once
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the boxes are generated suggestions.
- What: More than one suggestion can be on screen. Each extra one is drawn as its own preview box (never as gray text) beside its own changed lines. An extra box is dropped if its range falls within 2 lines of the main suggestion's range. When no main suggestion is showing, Tab accepts the first extra one that lies within 5 lines of the cursor. There is no key to cycle between them: no next or previous command was found for Cursor Tab, and the stock "Show Next Inline Suggestion" commands only act on the stock gray text.
- Commands: none found
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text controller `refreshAdditionalWidgets` and `update` at byte offset ~24962565 to ~24965500, calling the widget with `forceDiffBox:!0`; `handleTabAction` (starts at ~40739781) branch for `additionalSuggestion` with the 5-line window)

### AC-6.28 Added lines with a green bar and a key label
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the block is a generated suggestion.
- What: A third drawing for new lines: a block of lines under the changed lines with a 2px green bar on the right edge of the left margin and a small key label in the margin to its left. The label shows the current accept key in lower case ("tab" if none is found) and has the tooltip "Tab to accept suggestion". The original lines it would replace get a faint "unchanged code" background (opacity .35) and the block a faint blue tint. In the shipped defaults this shape is not reachable: it is used only when the Cursor Settings Development toggle "Show View Zone When Preview Box Is Clipped" is on (default off, and the Development section needs a development-tooling flag that is off) and the preview box would be clipped by the window. The older description "gray text with a tab margin label" does not match the code: the margin label belongs to this block, not to the gray text of AC-6.1.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module: `updateViewZone`, `createColoredLeftMargin` at byte offset ~24959889, `isPreviewBoxClipped`, `updateViewZone` at ~24957397; default `shouldShowViewZoneWhenPreviewBoxIsClipped6:!1` at ~17164970; Development row label "Show View Zone When Preview Box Is Clipped" at ~38267943; `developmentTooling:!1` at ~395975; CSS `.cpp-ghost-text-view-zone-margin`, `.cpp-ghost-text-view-zone-text`, `.view-zone-original-range-decoration`, `.view-zone-in-view-zone-decoration`)

### AC-6.29 Colors follow the file language
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it styles generated text.
- What: The preview box and gray lines are colored in the language of the open file: the box's inner editor gets the file's language id (and is switched when the file language changes), and gray lines are colored by a temporary model of that language. Tab characters count as the file's tab size when the box width is worked out. The language does nothing else to how the suggestion looks; language only decides whether Tab is on for the file (AC-8.4) and which files get the import hint (AC-7.25).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.tabSize
- Cursor: cursor only
- Source: C bundle (ghost text widget module: `setLanguage(t.getLanguageId())` on the box model, the temporary tokenize model `cpp-ghost-text-temp-tokenize-anysphere`, tab size read from `getOptions().tabSize`)

### AC-6.30 Preview box in notebook cells
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the box content is a generated suggestion.
- What: In notebook files (`.ipynb` files and notebook cells) the preview box is drawn 12px taller than the line count needs, and its inner editor is scrolled using the real editor's line positions instead of a fixed row height. This is the only per-file-type difference found in how the box looks.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ghost text widget module: `.ipynb` check adding 12 to the box height at byte offset ~24950771; helper treating schemes `vscode-notebook-cell` and `ipynb` and `.ipynb` paths as notebooks at ~24962244)

## Handed off

- The status bar item, Enable, Disable, Toggle, Snooze and Unsnooze commands, "Disable for {language}", Ignored Files, and the "Partial Accepts", "Suggestions While Commenting", "Whitespace-Only Suggestions" and "Imports" rows in Cursor Settings: AC-8.
- The "tab to jump here" hint, off-screen and cross-file pills, go-back pill, code preview and the import hint: AC-7.
- The stock inline edit surface (gutter indicator, side-by-side edit view, `editor.action.inlineSuggest.jump`) that still exists in the bundle: AC-2.
- The stock Show Next, Show Previous, Accept Line and toolbar for stock inline suggestions: AC-2.
- The Cursor Settings Development section toggles ("Show View Zone When Preview Box Is Clipped", "Force View Zones"): whichever group owns the Cursor Settings page (not in AC-6 to AC-8).
- Commands "Developer: Copy Recent Cursor Tab Actions" and "Developer: Report Cursor Tab Action": AC-8.

## Unclear

- The `w` test in the box-or-gray-text decision (a pattern of unchanged text next to an added newline that forces the box) was read but not worked out into a plain rule, so AC-6.1 and AC-6.2 say "passes the same test" for multi-line additions.
- Legacy highlight classes `cpp-suggestion-text-decoration-showing`, `-showing-gutter`, `-showing-streaming` and the `-debug` variants exist in CSS. The first is applied only when a stored config flag `isGhostText` is not true; the gutter class is only referenced when clearing; the streaming class is fully transparent. Whether any of them can be seen in normal use was not checked. The box and gray text themselves only draw when `isGhostText` is true; its value arrives at run time and was not checked.
- CSS under `.cursorGhostTextWidget` for `cursorCppHint`, `cursorGhostTextWidgetLoading`, `cppButtonContainerLoading`, `ghosttextButton`, `multiEditButton`, `bottomTriangle`, `cursorHiddenButton` and `commandHelpText` has no JS user in the bundle, so it looks like dead styling. `cpp-peek-view-title` has no CSS rule.
- The Alt rule: a method `eventShouldKillPrevCpp` returns true when Alt is held, but no caller was found, so no Alt behavior is listed.
- A method `shouldShowHoverAt` exists on the ghost text controller but no hover provider that uses it was found, so no hover on the gray text is listed.
- Whether Esc closes the diff peek was not confirmed (stock peek has a Close X; no Esc binding found in the stock peek file).
- Whether the remaining part of a suggestion stays on screen after a partial accept, and when exactly the green tint clears, were read from code paths and not run.
- Resolved light and dark colors of the Cursor-only CSS were not computed.
