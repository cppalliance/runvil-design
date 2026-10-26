Group: AC-6 Cursor Tab suggestions. Pass: 3. Date: 2026-10-25.

### AC-6.1
1. When a suggestion only adds characters and the new text lands at or after the cursor, the new text shows as dimmed, syntax colored gray text inside the line at the insertion point, and the existing code stays untouched.
2. The gray text is upright and has no border.
3. A suggestion shows as gray text only when it removes nothing and adds up to 20 character changes and 20 word changes; one that removes text or would land before the cursor uses the preview box instead (AC-6.3).
4. The stock theme colors `editorGhostText.*` also style the gray text, because it reuses the stock `ghost-text` classes.
5. Running the Command Palette entry "Update Ghost Text" (`editor.action.updateGhostText`) redraws the gray text.

### AC-6.2
1. When a suggestion adds whole new lines and passes the same test as AC-6.1, the new lines show as dimmed, syntax colored gray lines under the current line.
2. The editor opens blank room under the current line for the gray lines, so they are not real text.
3. Several added lines show as a stack in the order they will be inserted.
4. This shape has no box and no margin label.

### AC-6.3
1. When a suggestion changes or removes existing text, a small bordered box appears to the right of the code, level with the first changed line, and shows the new version of the changed lines.
2. The box is also used when the suggestion would insert text before the cursor or is large.
3. The box text uses the editor's own font, colors and language.
4. The box uses the suggest list background and border colors and a 2px corner radius.
5. The editor inside the box is read only.
6. The editor inside the box is drawn bare, with no line numbers, folding, minimap, indent guides, current-line highlight, visible scroll bars, whitespace dots or hover.

### AC-6.4
1. The box is one row tall for each changed line.
2. The box is as wide as the longest changed line, with a tab counting as the tab size.
3. The left edge of the box starts 24px past the end of the longest line in the changed range, so it never covers that code.
4. The box scrolls its inner editor so the changed lines are the ones you see.
5. Setting `editor.wordWrap` changes the right limit of the box: with the value `on` the box is not held back from the right edge, and with any other value (`off`, `wordWrapColumn`, `bounded`) it is held at least 2 columns inside the right edge of the editor.

### AC-6.5
1. Inside the preview box, the characters or words the suggestion adds carry a green tint.
2. Words that stay the same carry no tint.
3. The green tint uses the theme color `diffEditor.insertedTextBackground`, so changing that color changes the tint.

### AC-6.6
1. While the preview box shows, the words in your real code that the suggestion would replace or remove are tinted red.
2. The red tint uses the theme color `testing.iconFailed` at 17 percent opacity.
3. The red tint goes away when the box goes away.

### AC-6.7
1. When a suggestion only deletes one stretch of text, the box shows the lines that would go, dimmed to opacity .2 and struck through.
2. In the deletion box nothing new is shown and the box padding is removed.
3. When the deletion is in several separate places, no box is drawn.

### AC-6.8
1. Moving the pointer over the preview box adds a thin footer under the code with two small buttons, "View" and "Accept".
2. The "Accept" button carries a tiny bordered "Tab" chip that always reads Tab and does not follow your keybinding.
3. The footer buttons are dimmed (opacity .6) and light up on hover.
4. The footer is removed when the pointer leaves the box.
5. Only the preview box has this footer, not the gray text.

### AC-6.9
1. Pressing the mouse button on "Accept" in the box footer applies the whole suggestion, with the same result as the Tab key (AC-6.14).
2. The click on "Accept" does not move the editor cursor first.

### AC-6.10
1. Pressing the mouse button on "View" in the box footer opens the "Cursor Tab Preview" diff peek (AC-6.11) for that suggestion.
2. The peek opened by "View" shows the whole file before and after the suggestion.
3. The command `editor.action.setCppDiffPeekView` ("Set Cursor Tab Diff Peek View") opens the peek and has no Command Palette entry, so "View" is the only visible way to run it.

### AC-6.11
1. An in-editor peek panel titled "Cursor Tab Preview" opens under the suggestion's last line.
2. The peek holds a read only inline diff (not side by side) of the file before and after the suggestion.
3. The diff in the peek has sticky scroll off and is scrolled to the first change.
4. The peek has a frame and no arrow, and it can be resized.
5. The peek starts 10 lines tall.
6. The peek title bar carries an Accept action (AC-6.12) and the Close action (AC-6.13).
7. The command `editor.action.configureCppDiffPeekView` ("Configure Cursor Tab Diff Peek View") changes the diff editor options while a peek is open, and has no Command Palette entry or other visible control.

### AC-6.12
1. The peek title bar has one action of its own, a check mark with the tooltip "Accept Cursor Tab Suggestion".
2. Clicking the check mark applies the suggestion shown in the peek and clears the peek suggestion, which closes the peek (AC-6.13).
3. The action `editor.action.acceptCppSuggestion` has no Command Palette entry and no key, so the check mark is the only way to run it.

### AC-6.13
1. Clicking the stock "Close" X (`peekview.close`) in the peek title bar closes the peek.
2. The peek closes by itself when the active editor group changes.
3. The peek closes by itself when the editor or its file changes, or when the editor is closed.
4. The peek closes by itself when a new Tab suggestion appears or when the stored peek suggestion is cleared.

### AC-6.14
1. Pressing Tab while a suggestion shows applies the whole suggestion and clears it and its tints.
2. After a plain one-line insertion is accepted, the cursor ends right after the inserted text.
3. When Tab is pressed the editor tries these in order: accept a shown import hint (AC-7.23), accept the suggestion for the current spot, accept the nearest extra suggestion within 5 lines of the cursor (AC-6.27), then jump to the next edit if a jump hint shows (AC-7.5).
4. With both a Tab suggestion and the stock suggest list open, Tab accepts the Tab suggestion and not the list item.
5. The Tab accept key is only active while `cpp.shouldAcceptTab` is true, and otherwise the stock Tab runs.
6. With a cross-file jump pill showing and its code preview not yet visible, the first Tab only reveals the preview (AC-7.8).

### AC-6.15
1. The Tab binding of `editor.action.acceptCursorTabSuggestion` can be changed in Keyboard Shortcuts like any other command.
2. After it is rebound, the new key accepts whenever a suggestion or jump exists, because `cpp.shouldAcceptTab` stays true.
3. The key chips on the hint widgets show the new binding in upper case, and the margin label shows it in lower case, using the label text the keybinding service prints.
4. The "Tab" chip on the box's Accept button is fixed text and does not follow the new binding.
5. The Search and Problems views still react to the Tab key itself, not to the new binding.

### AC-6.16
1. With the setting on, pressing Ctrl+Right (Cmd+Right on Mac) while the editor is writable takes only the next word of the Cursor Tab suggestion instead of all of it.
2. The word break follows the word rules of the file's language.
3. The accepted word is tinted like AC-6.24, and the rest of the suggestion is compared again with the new text (whether the remainder stays on screen is read from code, not run).
4. Setting `cursor.cpp.enablePartialAccepts` ("Enable partial accepts", default off) chooses whether Ctrl+Right takes one word of a Tab suggestion; with it off, `cppSuggestion` is false and Ctrl+Right is the normal word move.
5. The stock command `editor.action.inlineSuggest.acceptNextWord` ("Accept Next Word Of Inline Suggestion") now tries the Cursor Tab word accept first and falls back to the stock word accept for stock inline suggestions.
6. No toolbar button is drawn for taking a word of a Tab suggestion, because the stock `InlineSuggestionToolbar` item belongs to the stock gray text, which is hidden while a Tab suggestion shows (AC-6.26; not run).

### AC-6.17
1. Pressing Esc (no Ctrl, Alt or Cmd held, Shift allowed) while a suggestion shows clears the suggestion and all of its tints.
2. After the suggestion is cleared, a waiting "tab to jump here" hint or pill (AC-7.1, AC-7.2) and a waiting import hint (AC-7.19) are shown again.
3. The Esc press is not swallowed, so Esc also does its normal job, for example closing a widget.
4. Esc for this is not a registered key binding, so it does not appear in Keyboard Shortcuts.
5. With no Tab suggestion showing, Esc dismisses an import hint first and a jump hint second (AC-7.13, AC-7.24).

### AC-6.18
1. Typing the characters the suggestion would insert next keeps the suggestion showing, and it shrinks as you go.
2. When the text in the suggestion range finally equals the suggestion, the suggestion clears.
3. Typing something that does not follow the suggestion rejects it, and the editor may then show a different one.
4. An edit that adds a newline or deletes text and does not follow the suggestion always rejects it.

### AC-6.19
1. Moving the cursor or selection so it no longer touches the suggestion's range (a click elsewhere, the arrow keys, a jump by other means) rejects the suggestion and clears its tints.
2. After the suggestion is rejected by moving away, a waiting jump hint is shown again.
3. Moves made by Cursor Tab itself (its own jump, peek and next-suggestion moves) do not count as moving away.

### AC-6.20
1. The preview box and gray text hide when the editor loses text focus.
2. The preview box and gray text hide when the editor shows a different file.
3. The preview box and gray text hide when text changes while the box's anchor line is the last line of the file.
4. The preview box and gray text hide when the pointer enters the box with the left button held down.
5. Hiding the box only fades it out and clears its pieces, and whether the suggestion itself stays is decided by AC-6.17 to AC-6.19.

### AC-6.21
1. When the cursor is on a line that holds only whitespace and the suggestion's next characters at the cursor begin with whitespace, Tab inserts an ordinary indent instead of accepting, because the Tab accept key is switched off (`cpp.shouldAcceptTab` false) and the stock Tab runs.
2. This blank line rule applies only before a jump has been accepted for that suggestion.
3. In Vim modes other than INSERT the blank line rule is skipped, so Tab can accept.

### AC-6.22
1. With a selection that spans more than one line, Tab is not captured and indents the selection as usual (the Search view and Problems list handlers are the exception, AC-6.23).
2. Shift+Tab is never captured, so it keeps its stock meaning.
3. When the Vim extension's status bar item shows any mode other than INSERT, the multi-line selection rule and the blank line rule (AC-6.21) are skipped, so Tab can accept.
4. Tab does not accept when the suggest list is open while the editor does not have focus.
5. Tab does not accept when no editor or file is found.

### AC-6.23
1. With keyboard focus in the Find box and a non-empty selection in the editor, Tab accepts a showing suggestion, controlled by a stored flag (`cppInCmdF`, default on) that is not a setting.
2. After accepting from the Find box, focus returns to the Find input without selecting its text.
3. With keyboard focus on a match in the Search results, Tab accepts a showing suggestion.
4. With keyboard focus in the Problems list, Tab accepts a showing suggestion.
5. With keyboard focus in the preview editor inside a Peek References widget, Tab accepts a showing suggestion.

### AC-6.24
1. After a suggestion (or a word of it) is accepted, the text that was just inserted or changed is tinted green.
2. The green tint uses the theme color `testing.iconPassed` at 12 percent opacity.
3. The tint stays through typing and is cleared when the cursor moves again while no suggestion is showing (read from code, not run).

### AC-6.25
1. After an accept succeeds, another suggestion for the spot can show straight away without typing.
2. After an accept succeeds, a jump hint or pill for the next edit can show straight away (AC-7.1, AC-7.2).
3. If the accept did nothing because nothing was applied, no follow-up is shown.

### AC-6.26
1. While a Cursor Tab suggestion is showing, the stock gray inline suggestion is hidden, so only one gray suggestion is on screen.
2. While a Cursor Tab suggestion is showing, the hover toolbar of the stock inline suggestion is hidden too.
3. When the Cursor Tab suggestion is gone, the stock suggestion is unhidden and asked for again.
4. Setting `editor.inlineSuggest.enabled` (true or false) controls whether stock inline suggestions show at all, and so whether there is one to hide and bring back.

### AC-6.27
1. More than one suggestion can be on screen, and each extra one is drawn as its own preview box beside its own changed lines, never as gray text.
2. An extra box is dropped if its range falls within 2 lines of the main suggestion's range.
3. When no main suggestion is showing, Tab accepts the first extra one that lies within 5 lines of the cursor.
4. There is no key to cycle between the boxes, and the stock "Show Next Inline Suggestion" commands only act on the stock gray text.

### AC-6.28
1. A block of added lines can be drawn under the changed lines with a 2px green bar on the right edge of the left margin.
2. A small key label in the margin to the left of the bar shows the current accept key in lower case ("tab" if none is found), with the tooltip "Tab to accept suggestion".
3. The original lines the block would replace get a faint "unchanged code" background (opacity .35) and the block gets a faint blue tint.
4. This shape is used only when the Cursor Settings Development toggle "Show View Zone When Preview Box Is Clipped" is on and the preview box would be clipped by the window.
5. In the shipped defaults this shape is not reachable, because the toggle is off by default and the Development section needs a development-tooling flag that is off.

### AC-6.29
1. The preview box and the gray lines are colored in the language of the open file.
2. The box's inner editor switches to the new language when the file language changes.
3. Setting `editor.tabSize` sets how many columns a tab character counts as when the box width is worked out.

### AC-6.30
1. In notebook files (`.ipynb` files and notebook cells) the preview box is drawn 12px taller than the line count needs.
2. In notebook files the box's inner editor is scrolled using the real editor's line positions instead of a fixed row height.

## Unclear

- All pieces here are cursor only by nature, so the `[cursor only]` suffix is left off on every piece, as the assignment allowed.
- AC-6.16 is Cursor `changed` (stock command redefined), so its Cursor piece (piece 5) has no suffix either, and it was not marked `[stock only]` because the stock word accept for stock inline suggestions still works.
- AC-6.1 and AC-6.2 depend on a "same test" for multi-line additions that the feature block says was not worked out in plain words, so piece 3 of AC-6.1 only states the clear cases (pure add up to 20 character and 20 word changes, no removal, at or after the cursor).
- AC-6.26 piece 4 states what the stock setting controls from the stock option text, and the feature block does not say how the setting interacts with hiding, so that interaction was not run.
- AC-6.28 is not reachable in the shipped defaults, so its pieces describe a shape that tracking may count as not buildable until the Development toggle is in scope.
- AC-6.16 pieces 3 and 6 and AC-6.24 piece 3 are read from code paths and not run, as the feature blocks say.
