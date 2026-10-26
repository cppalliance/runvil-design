Group: AC-2 Ghost text and inline suggestions. Pass: 3. Date: 2026-10-25.

### AC-2.1
1. While a suggestion is available, its text shows in the line right after the cursor as italic ghost text, and the cursor stays in front of it.
2. The code to the right of the cursor moves over to make room for the ghost text instead of being covered by it.
3. With syntax colors on, ghost text keeps the syntax colors of the language at 70 percent opacity, so it looks dimmer than real code.

### AC-2.2
1. When a suggestion has more than one line, the first line shows after the cursor and the extra lines show in a block under the cursor line in the editor font.
2. While multi-line ghost text shows, the rest of the original line to the right of the insert point is hidden so it does not collide with the ghost text.
3. When the ghost text has two or more lines, the suggest list opens above the cursor line instead of below it.
4. The extra lines use the same italic ghost look as the first line and follow the syntax color setting.

### AC-2.3
1. For an inline edit shown side by side or as a line replacement, the original lines that change get a line highlight and the exact characters that go get a stronger character highlight.
2. For an inline edit shown as a deletion, the text that will be removed is marked in place in the original lines.
3. For an inline edit shown as an inline insertion, the inserted characters get an insert highlight where they will land.
4. Ghost text that would replace existing text underlines the range it replaces (class `inline-completion-text-to-replace`), though stock 1.128.0 may never produce this case.

### AC-2.4
1. Ghost text with fewer than 3 non-space characters, or an inserted part under 5 characters, gets a dotted underline in the ghost text color. [stock only]
2. Very short ghost text skips the dimmed syntax colors and is drawn in the plain ghost text color so it stays visible. [stock only]
3. Ghost text that is longer than those limits has no dotted underline. [stock only]

### AC-2.5
1. A suggestion that carries a warning draws its ghost text with a warning squiggle and a double underline in the editor warning border color.
2. The toolbar for a suggestion with a warning shows the warning message above its buttons, as plain text or rendered markdown, at most 400 px wide.
3. Hovering the ghost text of a warned suggestion opens the hover with the warning message.

### AC-2.6
1. Setting `editor.inlineSuggest.syntaxHighlightingEnabled` chooses whether ghost text keeps the syntax colors of the language, on or off, default on.
2. With syntax colors off, ghost text is drawn in the single color of the theme color `editorGhostText.foreground`.
3. Setting `editor.inlineSuggest.fontFamily` sets a font family used only for ghost text, and the default value `default` keeps the editor font.
4. Theme colors `editorGhostText.background` and `editorGhostText.border` give ghost text a background and a one pixel border.

### AC-2.7
1. With `editor.suggest.preview` on, the item highlighted in the suggest list also shows in the editor as ghost text at the cursor.
2. Moving the highlight to another list item changes the ghost text to match that item.
3. The part of the preview beyond what you have already typed is drawn in the preview style, and the typed prefix is not repeated.
4. When the previewed item has more than one line, the suggest list moves above the cursor.
5. Setting `editor.suggest.preview` turns the preview on or off, default off.

### AC-2.8
1. Hovering ghost text opens the editor hover with a small toolbar that has an Accept button and an Accept Word button, each showing its key label.
2. The toolbar overflow menu holds Accept Line and an Always Show Toolbar toggle, each showing its key label.
3. Clicking Accept, Accept Word, or Accept Line takes the whole suggestion, the next word, or the next line (`editor.action.inlineSuggest.commit`, `editor.action.inlineSuggest.acceptNextWord`, `editor.action.inlineSuggest.acceptNextLine`).
4. Setting `editor.inlineSuggest.showToolbar` chooses when the toolbar shows, with values `onHover` (default), `always`, and `never`.
5. With `always`, the toolbar is pinned above or below the line that holds the ghost text instead of waiting for a hover.
6. Choosing Always Show Toolbar in the overflow menu flips `editor.inlineSuggest.showToolbar` between `always` and `onHover`, and the menu item shows a check while it is `always` (`editor.action.inlineSuggest.toggleAlwaysShowToolbar`).

### AC-2.9
1. When more than one suggestion is available, the toolbar starts with a previous chevron, an `n/m` count, and a next chevron.
2. Clicking the next or previous chevron shows the next or previous suggestion and the count updates.
3. When there is only one suggestion, the previous and next chevrons are disabled.
4. Pressing `Alt+]` shows the next suggestion and `Alt+[` shows the previous one while the editor is writable and a suggestion is visible (`editor.action.inlineSuggest.showNext`, `editor.action.inlineSuggest.showPrevious`).

### AC-2.10
1. Pressing `Tab` while ghost text shows inserts the whole suggestion and puts the cursor at the end of it (`editor.action.inlineSuggest.commit`).
2. `Tab` does not accept, and does its usual job, while the suggest list is open, while Tab is set to move focus, or while a hover has focus.
3. With text selected, `Tab` does not accept the suggestion and indents the selection as usual. [stock only]
4. When the suggestion starts with more whitespace than one Tab would add, `Tab` indents as usual instead of accepting.
5. Clicking Accept in the toolbar, or choosing the accept row in the gutter menu, takes the whole suggestion the same way.
6. When an inline edit shows and Tab is set to accept rather than jump, `Tab` accepts the edit, and it also accepts inside the inline edits preview editor.
7. In Cursor, `Tab` accepts the suggestion even while text is selected, and Cursor Tab adds its own Tab accept (see AC-6). [cursor only]

### AC-2.11
1. Pressing `Ctrl+Right` (`Cmd+Right` on macOS) with the cursor right in front of ghost text takes only the next word of it (`editor.action.inlineSuggest.acceptNextWord`).
2. The key keeps its normal word-move job when the cursor is not right in front of the ghost text or when screen reader mode is on. [stock only]
3. Clicking Accept Word in the toolbar does the same as the key.
4. A suggestion that carries a snippet is accepted whole instead of one word.
5. In Cursor, the same key also fires for a Cursor Tab suggestion and first tries the Cursor Tab partial accept, with no cursor-position or screen reader guard (see AC-6). [cursor only]

### AC-2.12
1. Accept Next Line takes the ghost text up to and including the next line break (`editor.action.inlineSuggest.acceptNextLine`).
2. Accept Line is in the toolbar overflow menu and has no default key.
3. Running Accept Next Line Of Inline Suggestion from the Command Palette does the same.

### AC-2.13
1. Pressing `Escape` while ghost text or an inline edit shows hides it and keeps focus in the editor (`editor.action.inlineSuggest.hide`).
2. Inside the inline edits preview editor, `Escape` or `Shift+Escape` hides the edit.
3. The Reject row in the gutter indicator menu hides the suggestion the same way.
4. In Cursor, `Escape` rejects a suggestion without needing editor text focus, and also rejects a Cursor Tab suggestion (see AC-6). [cursor only]

### AC-2.14
1. Running Trigger Inline Suggestion from the Command Palette, or a key you bind to `editor.action.inlineSuggest.trigger`, asks for a suggestion at the cursor in a writable editor.
2. When the command is run with `showNoResultNotification` set and no suggestion comes back, an info notice "No inline suggestion is available." shows. [stock only]
3. After the command runs, the inline suggestion sound cue plays for the suggestion that shows.

### AC-2.15
1. Running Trigger Next Edit Suggestion from the Command Palette asks only for an inline edit, never for ghost text (`editor.action.inlineSuggest.triggerInlineEditExplicit`). [cursor only]
2. When no inline edit comes back, an info notice "No inline edit is available." shows. [cursor only]
3. A second command `editor.action.inlineSuggest.triggerInlineEdit` triggers an inline edit but has no label and no Command Palette entry. [cursor only]

### AC-2.16
1. Running Accept Next Inline Edit Part from the Command Palette takes the first piece of a multi-part inline edit (`editor.action.inlineSuggest.acceptNextInlineEditPart`). [cursor only]
2. The command is enabled only while the editor is writable and an inline edit is visible, and it has no default key. [cursor only]

### AC-2.17
1. A small pill in the left gutter marks an inline edit, or a ghost text that offers the edit menu.
2. The pill shows a Tab icon beside the edit when Tab will accept it.
3. The pill shows a Tab-above or Tab-below icon when Tab will jump up or down to an edit that is elsewhere in the file.
4. The pill shows a check icon while you hover it.
5. When the edit is outside the visible part of the file, the pill slides into view at the edge and shows an arrow that points up or down.
6. The pill shows the line number of the edit.
7. Clicking the pill accepts the edit or jumps to it (`editor.action.inlineSuggest.commit`, `editor.action.inlineSuggest.jump`).
8. The pill pulses a few times for new users, and it pulses when you hover the edit.
9. In Cursor, clicking the pill accepts the edit when the pill is docked beside it and jumps to the edit otherwise. [cursor only]

### AC-2.18
1. Hovering the gutter pill opens a small menu whose header shows the name of the provider that made the suggestion.
2. The first row reads Go To / Accept with a check icon and does what Tab would do, jump or accept.
3. A Reject row with a close icon hides the suggestion (`editor.action.inlineSuggest.hide`).
4. A row reading Show Collapsed or Show Expanded flips whether edits show collapsed (`editor.action.inlineSuggest.toggleShowCollapsed`).
5. A Snooze row with a bell-slash icon opens the snooze duration choice (`editor.action.inlineSuggest.snooze`). [stock only]
6. When the edit has an alternative action, an extra row runs it (`editor.action.inlineSuggest.commitAlternativeAction`). [stock only]
7. A Settings row with a gear icon opens Settings filtered to the tag `@tag:nextEditSuggestions` (`workbench.action.openSettings`).
8. Each row shows its key label, and extra commands and a footer action bar follow the rows.

### AC-2.19
1. When an inline edit is elsewhere in the file, pressing `Tab` moves the cursor to the edit instead of accepting it (`editor.action.inlineSuggest.jump`).
2. After the jump, pressing `Tab` a second time accepts the edit.
3. `Tab` jumps only while an inline edit is visible, Tab is not set to move focus, the suggest list is closed, no hover has focus, and Tab is set to jump.
4. The command Jump to next inline edit moves the cursor to the edit the same way as the key.

### AC-2.20
1. An inline edit can show a second action as a small labeled segment next to the replacement, for example Rename. [stock only]
2. Holding `Shift` adds a count to the label, for example "Rename 3 occurrences". [stock only]
3. Pressing `Shift+Tab` while such an edit shows runs the second action, and it also works inside the inline edits preview editor (`editor.action.inlineSuggest.commitAlternativeAction`). [stock only]
4. The gutter menu lists the second action as its own row. [stock only]

### AC-2.21
1. A change that spans lines shows its new code in a bordered preview box next to the original lines when the old and new text both fit across the editor width.
2. The border color of the box follows what Tab will do: accept, jump, or nothing.
3. Clicking the box accepts the edit once the box has been on screen for 350 ms.
4. Setting `editor.inlineSuggest.edits.renderSideBySide` chooses `auto` (default, side by side when there is room, below otherwise) or `never` (always the line replacement view of AC-2.22).
5. Resizing the editor makes the choice between side by side and below be made again for the edit that is showing.

### AC-2.22
1. A larger change shows as a bordered box under the original lines with the new lines inside and the old lines highlighted, when side by side is off or there is not enough room.
2. A one-line insertion that lands before the cursor shows in this box too, so the cursor does not move.
3. A one-line change that is too big for the small word box (AC-2.23) shows in this box.
4. Clicking the box accepts the edit.
5. Setting `editor.inlineSuggest.edits.renderSideBySide` set to `never` makes every larger change use this view.

### AC-2.23
1. A small change inside one line, with each side under 100 characters and no tabs, shows as a small box just under the changed word with the new text.
2. The old word stays highlighted in the line while the small box shows.
3. Clicking the small box accepts the edit.
4. A change that involves a tab, or has either side at 100 characters or longer, does not use the small box and shows in a larger view instead.
5. When the edit has an alternative action, the box gets the second labeled segment described in AC-2.20. [stock only]

### AC-2.24
1. A single-line insertion after the cursor shows inline in the text and shifts the code to the right to make room.
2. A multi-line insertion shows as a bordered block that pushes the lines below it down.
3. Setting `editor.inlineSuggest.edits.allowCodeShifting` chooses `always` (default, both shapes), `horizontal` (only the inline shift), or `never` (a line replacement or side by side view is used instead).

### AC-2.25
1. An edit that only removes text outlines the text to delete in place with a bordered overlay in the original lines.
2. A deletion shows no new text anywhere in the editor.

### AC-2.26
1. With Show Collapsed on, a small downward-pointing triangle at the start of the edit stands in for the full preview until you jump to the edit.
2. Hovering the gutter pill shows the full preview again.
3. The gutter menu row flips between Show Collapsed and Show Expanded.
4. Running Toggle Inline Suggestions Show Collapsed flips the collapsed setting the same way (`editor.action.inlineSuggest.toggleShowCollapsed`).
5. The triangle bobs once for users who have not yet become regular users.
6. Setting `editor.inlineSuggest.edits.showCollapsed` chooses whether edits show collapsed until you jump to them, on or off, default off.

### AC-2.27
1. When a suggestion is only a jump to a position in the file with no text to insert, a small bordered label with a Tab key chip and the words "to jump" shows at the target position. [stock only]
2. Accepting it with `Tab` moves the cursor to that position (`editor.action.inlineSuggest.commit`). [stock only]
3. The label can also be drawn in a blinking-cursor style. [stock only]

### AC-2.28
1. When a suggestion supplies a hint instead of a diff, a small bordered text label shows at the end of the hint's line. [stock only]
2. When the end of that line has no room, the label shows just below or just above the line. [stock only]
3. A suggestion for another file, or one with a display location that you have not jumped to yet, shows as this label instead of a diff. [stock only]
4. The border color of the label follows what Tab will do. [stock only]
5. With the Code hint style, the label text is syntax-colored. [stock only]
6. Clicking the label accepts the suggestion. [stock only]

### AC-2.29
1. When the next edit is outside the visible part of the file or in another file, a small preview box of the target code shows near the cursor line. [stock only]
2. The box sits in free space to the right, or over the code when there is no free space. [stock only]
3. The bottom bar of the box shows the symbol outline at the target, or the icon and name of the target file for another file. [stock only]
4. The bar reads "Tab to jump" (or "Go to suggestion" when Tab is not the key) for the same file, and "Tab to open" or "Go to file" for another file. [stock only]
5. An arrow in the bar shows which direction the target is in. [stock only]
6. Clicking the box jumps to the target (`editor.action.inlineSuggest.jump`). [stock only]
7. Setting `editor.inlineSuggest.edits.showLongDistanceHint` turns this hint on or off, default on, experimental. [stock only]

### AC-2.30
1. Setting `editor.inlineSuggest.edits.longDistanceHintContextLineCount` chooses how many lines above and below the target line the long distance preview shows, from 0 to 10, default 0, experimental. [stock only]
2. With 0 the preview shows only the target line, and with a larger value it also shows that many lines above and below. [stock only]

### AC-2.31
1. After you accept an inline edit, the changed ranges flash the modified-text background color.
2. The flash fades out over about one second.
3. When reduced motion is on (`workbench.reduceMotion`), the flash is skipped.

### AC-2.32
1. Theme colors `inlineEdit.originalBackground` and `inlineEdit.modifiedBackground` set the background behind the original code and behind the new code of an inline edit.
2. Theme colors `inlineEdit.originalChangedLineBackground` and `inlineEdit.originalChangedTextBackground` set the line and character highlight on the original code that will go.
3. Theme colors `inlineEdit.modifiedChangedLineBackground` and `inlineEdit.modifiedChangedTextBackground` set the line and character highlight on the new code.
4. Theme colors `inlineEdit.originalBorder` and `inlineEdit.modifiedBorder` set the border around the original code and around the new code.
5. Theme colors `inlineEdit.tabWillAcceptModifiedBorder` and `inlineEdit.tabWillAcceptOriginalBorder` set the borders used while Tab will accept the edit.
6. Theme colors `inlineEdit.gutterIndicator.*` set the foreground, border, and background of the gutter pill in its plain, primary, secondary, and successful variants.
7. Each of these colors can be overridden under `workbench.colorCustomizations`.

### AC-2.33
1. Setting `editor.inlineSuggest.enabled` turns automatic inline suggestions on or off, default on.
2. With the setting off, no inline suggestion shows on its own as you type.

### AC-2.34
1. Setting `editor.inlineSuggest.minShowDelay` makes a suggestion wait from 0 to 10000 ms after typing before it shows, default 0. [stock only]
2. Setting `editor.inlineSuggest.suppressInSnippetMode`, default on, hides inline suggestions while the tab stops of a snippet are active. [stock only]
3. With `editor.inlineSuggest.suppressInSnippetMode` off, inline suggestions still show while snippet tab stops are active. [stock only]

### AC-2.35
1. With `editor.inlineSuggest.suppressSuggestions` on, the suggest list does not open on its own while an inline suggestion is available.
2. With `editor.inlineSuggest.suppressSuggestions` off, the default, the suggest list can still open on its own next to ghost text.
3. A suggestion can set the same flag for itself (context key `inlineSuggestionSuppressSuggestions`), and the suggest list then stays closed on its own while that suggestion shows.

### AC-2.36
1. Running Snooze Inline Suggestions from the Command Palette opens a quick pick titled "Select snooze duration for Inline Suggestions". [stock only]
2. The quick pick lists 1, 5, 10, 15, 30, and 60 minutes. [stock only]
3. The Custom... row asks "Enter snooze duration in minutes" with the placeholder "Duration in minutes (e.g. 90)". [stock only]
4. After a custom length has been used, a row reading "{n} minutes (Last used)" shows in the quick pick. [stock only]
5. The command also takes a number of minutes as its first argument and snoozes for that long. [stock only]
6. While snoozed, inline suggestions do not show until the time ends. [stock only]
7. Cancel Snooze Inline Suggestions ends the snooze and shows in the Command Palette only while a snooze is on (`inlineCompletions.snoozed`). [stock only]

### AC-2.37
1. The language status popup gets an "Inline Suggestions" row for the focused file, with the detail "Inline suggestions".
2. While a suggestion is being fetched, the row reads "Loading..." and shows as busy.
3. When ghost text is available, the row reads "Inline completion available" with a lightbulb icon.
4. When an inline edit is available, the row reads "Inline edit available" with a lightbulb-sparkle icon.
5. When nothing is available, the row reads "No inline suggestion available" with a circle-slash icon.
6. The row updates after a 300 ms debounce.
7. The row is hidden when AI features are turned off with `chat.disableAIFeatures`. [stock only]
8. In Cursor, the row shows only while the editor has focus and is never hidden by an AI features check. [cursor only]

### AC-2.38
1. With a suggestion showing, `Alt+F2` opens the accessible view with the line plus the ghost text (`Alt+Shift+F2` on Linux, with `Alt+F2` as a second key).
2. For an inline edit, the accessible view shows a readable patch.
3. Show Next and Show Previous inside the accessible view step through the suggestions (`editor.action.accessibleViewNext`, `editor.action.accessibleViewPrevious`).
4. `Ctrl+/` (`Control+/` on macOS) accepts the suggestion while the view shows an inline completion, and the view toolbar has an "Accept Inline Completion" button (`editor.action.accessibleViewAcceptInlineCompletion`).
5. Closing the accessible view hides the suggestion.
6. A screen reader hint "Inspect this in the accessible view ({0})" points to the key, shown when `accessibility.verbosity.inlineCompletions` allows it.

### AC-2.39
1. When a suggestion appears, a screen reader reads it aloud.
2. Setting `editor.screenReaderAnnounceInlineSuggestion` turns that announcement on or off, default on.
3. Settings `accessibility.verbosity.inlineCompletions` and `editor.inlineCompletionsAccessibilityVerbose` choose whether the announcement adds the hint about the accessible view.
4. With the announcement off, a screen reader user gets hover text "Suggestion:" followed by the code, and the hover reads "There are inline completions here".
5. Setting `accessibility.signals.lineHasInlineSuggestion` chooses whether a sound cue plays when a line has an inline suggestion.
6. Setting `accessibility.signals.nextEditSuggestion` chooses whether a second sound cue plays for next edit suggestions. [stock only]

### AC-2.40
1. Running Developer: Extract Inline Suggest State from the Command Palette copies the file text plus the current suggestion, as commented JSON, to the clipboard (`editor.action.inlineSuggest.dev.extractRepro`).
2. In stock, the command is enabled while plain ghost text shows as well as while an inline edit shows. [stock only]
3. In Cursor, the command is enabled only while an inline edit is visible. [cursor only]

### AC-2.41
1. With `editor.quickSuggestions` set to `inline`, the best matching completion shows as ghost text at the end of a word and the suggest list does not open.
2. The same ghost text shows right after a trigger character.
3. The values for `other`, `comments`, and `strings` can each be `inline`, so the ghost text shows only in that kind of code.
4. The `n/m` count and the `Alt+]` and `Alt+[` keys of AC-2.9 step through the other matching completions.
5. While the suggest list is open this ghost text is not shown, so opening the list by hand with `Ctrl+Space` replaces it.

### AC-2.42
1. With more than one cursor, ghost text shows at every cursor where the text the suggestion would replace matches the main cursor, not only at the main one.
2. A plain insertion matches at every cursor, so its ghost text shows at all of them.
3. `Tab` types the same text at each of those cursors and leaves one cursor at the end of each.
4. Accept Word and Accept Line do the same at each of those cursors.
5. Inline edits show only for the main cursor and leave a single cursor after accept.

### AC-2.43
1. When the suggest list opens while ghost text shows, the list starts with the item the ghost text builds on already highlighted, which is the longest list item whose text is the start of the suggestion.
2. When no list item fits, the suggest list highlights its usual first item.

### AC-2.44
1. When an inline suggestion starts with the same text as the highlighted list item and covers at least the same range, the editor shows the whole suggestion as ghost text.
2. The part of the ghost text beyond the list item is drawn in the preview style.
3. This works with `editor.suggest.preview` off, and the plain preview is then used only when no inline suggestion extends the item.
4. Holding `Shift` while the list is open flips the insert or replace range used for this preview, the same way `Shift+Tab` does for accepting.

### AC-2.45
1. The commit message box in Source Control shows ghost text and the toolbar, with the same `Tab`, `Escape`, and Accept Word keys as the main editor.
2. The reply box of a review comment thread shows ghost text and the toolbar the same way.
3. In stock, `Escape` in the commit message box first rejects a showing suggestion and empties the message only when no suggestion is showing (`editor.action.inlineSuggest.hide`, `scm.clearInput`). [stock only]
4. In Cursor, `Escape` in the commit message box empties the message even while ghost text shows, because the clear rule outranks the reject rule. [cursor only]

### AC-2.46
1. Ghost text goes away when you move the cursor yourself with a click or an arrow key, but not when you type.
2. An inline edit stays when you move the cursor yourself.
3. Any suggestion goes away when the editor loses focus, including when you focus another editor.
4. A suggestion stays when focus moves into the gutter indicator menu, the accessible view, or the toolbar overflow menu, and an inline edit you asked for with the trigger command stays too.
5. Inline edits hide while a peek view (peek definition or references) is open.
6. A read-only editor never shows suggestions.

### AC-2.47
1. While you type the same characters as the ghost text, it stays and gets shorter.
2. As soon as you type something that is not the next part of the ghost text, it goes away.
3. Leading indentation is ignored when your typing is compared with the ghost text.
4. The text to the right of the cursor must still fit the rest of the suggestion, or the ghost text goes away.
5. A suggestion that starts on a different line than the cursor is not shown.

### AC-2.48
1. A suggestion with tab stops and placeholders shows as plain ghost text.
2. `Tab` or the Accept button inserts it as a snippet, so the cursor lands on the first tab stop and snippet mode starts.
3. Accept Word and Accept Line take the whole snippet instead of a part of it.
4. While snippet mode is on, other suggestions stay hidden when `editor.inlineSuggest.suppressInSnippetMode` is on. [stock only]

## Unclear
- AC-2.3 piece 4: nothing in 1.128.0 builds the replace type behind the underline, so the underline may never show. Decide whether to build it.
- AC-2.30: the setting is newer-than-1.128 (found only in N), so it may not belong in a 1.128.0 based build.
- AC-2.15 and AC-2.16: the command ids come from older stock code left in the Cursor bundle, origin not verified.
- AC-2.19 and AC-2.13: stock registers Accept, Jump, and Reject entries on the `InlineEditsActions` menu but nothing draws that menu, so no piece covers it. The gutter menu in AC-2.18 is the visible equivalent.
- AC-2.41 piece 4 and AC-2.45 piece 4 follow the pass 2 reading of code and rule weights, not a run on screen.
- AC-2.45 piece 1 covers only the commit message box and the comment reply box. Notebook cells and the chat input also host the controller and are not part of this feature.
- AC-2.1 to AC-2.48: the Cursor field says "present in the bundle", not "reachable on screen", because nothing was run.
