Group: AC-3 Parameter hints. Pass: 3. Date: 2026-10-25.

### AC-3.1
1. Pressing `Ctrl+Shift+Space` (`Cmd+Shift+Space` on macOS) with the cursor inside a call opens the signature popup at once (`editor.action.triggerParameterHints`).
2. Running Trigger Parameter Hints from the Command Palette opens the popup the same way.
3. The key works only while the editor text has focus.
4. When no signature help provider covers the file, the key and the command do nothing.

### AC-3.2
1. Typing a character that the language lists as a signature trigger opens the popup about 120 ms later.
2. For TypeScript and JavaScript the trigger characters are `(`, `,`, and `<`.
3. Setting `editor.parameterHints.enabled` chooses whether typing a trigger character opens the popup on its own, on or off, default on.

### AC-3.3
1. While the popup is open, typing or deleting refreshes it so the bold parameter follows the comma you are in.
2. Moving the cursor with the keys refreshes the popup the same way.
3. Typing a retrigger character (TypeScript lists `)`) refreshes the popup too.
4. A refresh keeps the overload you were on when the language reports the same signature again.

### AC-3.4
1. The popup shows the whole signature on one line in the editor font.
2. The parameter you are on is drawn in bold in the highlight color (theme color `editorHoverWidget.highlightForeground`).
3. A signature with no parameters shows its plain label.
4. The active parameter is the one the signature names for itself, and the overall active parameter is used when the signature names none.

### AC-3.5
1. When the language returns more than one signature, the popup gets a narrow strip on its left.
2. The strip shows an up chevron, a counter such as 2/3, a down chevron, and a faint divider.
3. The counter is zero padded to the width of the total, for example 02/12 with twelve signatures.
4. Clicking a chevron shows the previous or next signature and puts focus back in the editor.
5. With only one signature the strip is not shown.

### AC-3.6
1. With the popup open and more than one signature, `Up` or `Alt+Up` shows the previous signature (`showPrevParameterHint`).
2. With the popup open and more than one signature, `Down` or `Alt+Down` shows the next signature (`showNextParameterHint`).
3. On macOS `Ctrl+P` also shows the previous signature and `Ctrl+N` also shows the next.
4. The popup takes these keys from the editor only while it is open with more than one signature, and otherwise the keys move the cursor as usual.

### AC-3.7
1. With wrap around on, `Down` on the last signature shows the first one, and `Up` on the first signature shows the last one.
2. With wrap around off, going past either end closes the popup.
3. Setting `editor.parameterHints.cycle` chooses wrap around, on or off, default on.

### AC-3.8
1. Below the signature the popup shows the documentation of the active parameter first and then the documentation of the whole signature.
2. Documentation shows as plain text or as rendered markdown with code spans, code blocks, and links.
3. A faint rule sets the documentation apart from the signature.
4. With no documentation the documentation area is hidden.
5. The documentation text can be selected with the mouse.

### AC-3.9
1. The popup sits above the cursor line, below it when there is no room above, and it moves with the cursor.
2. The popup is at most 440 px wide.
3. The popup is at most as tall as the larger of a quarter of the editor height and 250 px, and long documentation scrolls inside it with its own scrollbar.
4. The popup uses the font size of the editor, with the signature in the editor font.
5. The popup frame has a border, a rounded corner, and a drop shadow taken from theme tokens. [stock only]

### AC-3.10
1. Pressing `Escape` or `Shift+Escape` while the editor has focus closes the popup (`closeParameterHints`).
2. The popup closes when the editor loses focus.
3. A mouse click in the editor text closes the popup.
4. The popup closes when the language returns no signature for the spot.
5. With wrap around off (`editor.parameterHints.cycle`), going past the end of the list closes the popup.

### AC-3.11
1. Turning `editor.parameterHints.enabled` off, default on, stops the popup from opening on its own when you type a trigger character.
2. Turning it off while the popup is open closes the popup at that moment.
3. With it off, `Ctrl+Shift+Space` still opens the popup by hand.

### AC-3.12
1. Typing a trigger character inside a string or a comment does not open the popup for TypeScript and JavaScript.
2. `Ctrl+Shift+Space` still opens the popup inside a string or a comment, because the rule applies only to a typed character (`editor.action.triggerParameterHints`).
3. For other languages, whether a typed trigger character opens the popup in a string or a comment is left to that language, because the editor has no string or comment rule of its own.

### AC-3.13
1. A screen reader is told the label of the active parameter, then its documentation and the documentation of the signature, followed by "hint".
2. The announcement is made only when the active parameter changes, so typing inside the same parameter does not repeat it.

## Unclear
- AC-3.12: the string and comment rule was read only in the TypeScript language service. The PHP provider and the HTML embedded JavaScript mode were not read, so their behavior is unknown.
- AC-3.7 piece 3: the option comment in stock says the default is false, but the code and the Cursor bundle default to on, so the piece says on.
- AC-3.9 piece 5: Cursor's frame rule shows only a border, but a shadow set by another rule was not ruled out.
- AC-3.1 to AC-3.13: the popup adds a `visible` class 100 ms after showing, and no rule for it was found, so a fade-in is not covered by any piece.
