Group: AC-7 Cursor Tab jump and imports. Pass: 3. Date: 2026-10-25.

### AC-7.1
1. When the next edit is in the open file and its line is on screen, a small pill sits just after the end of that line and reads a key chip with the accept key in upper case ("TAB" unless you rebound it) followed by muted text "to jump here".
2. The pill has a faint tint of the suggest list highlight color, a 4px corner radius, a soft shadow and a 2px accent bar on its left edge.
3. The pill text is about half strength, its font is the `editor.fontSize` minus 2, and its height is one line (from `editor.lineHeight`) minus 2px.
4. On an empty target line the pill text starts 2px in, and otherwise 0.5rem in.
5. The pill ignores the mouse, so it cannot be clicked.

### AC-7.2
1. When the next edit is in this file but its line is outside the visible lines, a pill appears at the top center of the editor (arrow up, target above) or at the bottom center (arrow down, target below).
2. The pill reads the key chip (accept key in upper case, "TAB" as fallback), then "to next move", then the arrow icon.
3. The pill is small (20px tall), blurred behind (backdrop blur 15px), tinted 7 percent, has a 6px corner radius, and the whole widget is drawn at 90 percent scale.
4. When any inline diff is open, the top pill sits lower, with a 56px margin instead of 24px.

### AC-7.3
1. When the target line scrolls into view, the pill hides and the line hint (AC-7.1) shows.
2. When the target line scrolls out of view, the line hint hides and the pill (AC-7.2) shows, with its arrow flipping to up or down to match the side.
3. The pill also re-checks which side to show when the cursor moves, the selection changes, or the proposed jump target changes.

### AC-7.4
1. Pressing the mouse button on the pill runs the accept action, which takes the same jump as the Tab key (AC-7.5).
2. The pill shows a pointer cursor over it in its minimized form.
3. Pressing the mouse button on the go-back pill (AC-7.12) goes back (`workbench.action.navigateBack`) instead of jumping.

### AC-7.5
1. With a jump hint or pill showing and no suggestion to accept at the cursor, pressing Tab moves the cursor to the first non-blank character of the target line.
2. If the target line was outside the view, the jump scrolls it to the center.
3. The jump clears the hint or pill.
4. If a suggestion for the current spot is also showing and it came from a jump proposal, the first Tab only jumps and a following Tab then accepts it (read from code, not run).
5. A Tab press is ignored for jumping when the cursor line is empty (AC-7.16).

### AC-7.6
1. After a jump, a new Cursor Tab suggestion can show at the landing line, so one may appear right after the cursor arrives.
2. A second Tab can accept the suggestion that appeared on arrival.
3. This also holds when the jump hint was shown again after Esc (read from the code path after a jump, not run).

### AC-7.7
1. When the next edit is in a different file, a pill at the bottom center of the editor reads the key chip, then "to".
2. After "to", the pill shows the file's icon and its file name (last path part only, bold italic).
3. The pill ends with an arrow pointing right.
4. Clicking the cross-file pill or pressing Tab starts the jump (AC-7.8, AC-7.10).
5. While the cross-file pill is showing, the context key `tabPredictionGoToFileVisibleKey` ("Whether a cursor prediction for a target file is currently visible") is true.

### AC-7.8
1. On a cross-file pill whose code preview is hidden, the first Tab (or click) does not jump and only reveals the code preview above the pill.
2. The next Tab after the preview shows jumps to the other file.
3. The context key `tabPredictionCodePreviewVisibleKey` ("Whether a code preview for a target file is currently visible") turns true when the preview shows.

### AC-7.9
1. The code preview is a small read only editor, 300px wide and 7 lines tall, drawn above the cross-file pill.
2. The preview shows the target file in its own language, centered on the target line (3 lines above it).
3. Every line in the preview except the target line is dimmed to half strength.
4. The target line in the preview carries a bright "TAB to jump here" hint (AC-7.1).

### AC-7.10
1. A jump to another file switches to that file and puts the cursor on the target line (first non-blank character), scrolling it to the center if needed.
2. If an editor group already shows the file, that group is used and nothing new opens.
3. If no group shows the file, the file is opened in the active group (not to the side) and focus moves to it.

### AC-7.11
1. When the target file is already open in another editor group, the active editor shows the cross-file pill (AC-7.7).
2. The other editor shows the "TAB to jump here" line hint on the target line if that line is visible there (AC-7.1).
3. If the target line is not visible in the other editor, it shows the off-screen pill without a file name (AC-7.2).
4. Scrolling the other editor swaps between the line hint and the pill, as in AC-7.3.

### AC-7.12
1. After a jump that switched files, a pill at the bottom center shows a key chip with the key bound to Go Back and the text "to go back".
2. Clicking the pill or pressing that key goes back (`workbench.action.navigateBack`, Alt+Left on Windows, Ctrl+- on Mac, Ctrl+Alt+- on Linux).
3. The go-back pill is shown at most twice in a window session, and the count is kept in memory and not saved.
4. The go-back pill is not shown at all if Go Back has no key.
5. The go-back pill hides when the selection changes or the editor loses focus.

### AC-7.13
1. With no Tab suggestion showing, pressing Esc (no Ctrl, Alt or Cmd) clears the jump hint and pill, and a cross-file code preview.
2. If an import hint is also showing, the first Esc dismisses the import hint and swallows the key, and a second Esc clears the jump.
3. Clearing by Esc counts as a rejection of that target (AC-7.15).
4. With a Tab suggestion showing, Esc rejects the suggestion instead, and the jump hint stays or is shown again (AC-6.17).

### AC-7.14
1. The hint and pills disappear when you type or otherwise change the file, and that counts as a rejection of the target.
2. The hint and pills disappear when the editor loses text focus, and that counts as a rejection.
3. The hint and pills disappear when the editor shows another file or is closed, and that counts as a rejection.
4. Resizing the window clears the hint and pills in every editor without counting a rejection.
5. Deleting or removing the target file clears the hint and pills.

### AC-7.15
1. A jump hint is not shown when the target is within 1 line of the cursor.
2. A jump hint is not shown when the target lies inside the range of the suggestion you just accepted.
3. A jump hint is not shown when the target overlaps an inline diff that is still being written.
4. A jump hint is not shown when its line number is past the end of the file.
5. A jump hint is not shown for a line you dismissed twice in the last 30 seconds, and the last 5 dismissed lines are remembered.
6. Where the jump is shown in place of an edit, it is also skipped when the target is under 5 lines from a spot you already jumped to since your last edit.
7. Where the jump is shown in place of an edit, it is also skipped straight after a jump.

### AC-7.16
1. When the cursor is on an empty line, pressing Tab does not jump and the cursor does not move.
2. Where the jump is shown in place of an edit, the jump is not offered at all on an empty line, because Tab would indent there.
3. In Vim modes other than INSERT the empty line rule is skipped.

### AC-7.17
1. If a Tab preview box or gray text would cover the jump target line, the jump hint and pill are hidden so they do not collide.
2. If a suggestion covers the line of an import hint, the import hint is hidden the same way.
3. After Esc rejects the suggestion, the hidden hints are shown again.

### AC-7.18
1. When the target is in a `.ipynb` notebook and its line is on screen, the "TAB to jump here" line hint (AC-7.1) is used.
2. When the target is in a `.ipynb` notebook and its line is off screen, the off-screen pill (AC-7.2) is used, and the cross-file pill with a file name is not.

### AC-7.19
1. A small chip appears at the end of the line that holds an unresolved name, reading `import {name} (press {key} or ESC)`, for example "import useState (press Tab or ESC)".
2. The key in the chip is the label of the accept key as the keybinding service prints it, and "TAB" if none is found.
3. The chip has a 4px corner radius, a border in the link color, text 0.85em in the editor font and editor foreground, 80 percent opacity, and a gap of 0.75em before it.
4. The chip ignores the mouse, so it is keyboard only.
5. Only one import hint shows at a time.

### AC-7.20
1. The import hint is drawn only when the "Imports" switch in Cursor Settings is on (AC-8).
2. The import hint is drawn only when the open file is a supported language (AC-7.25).
3. The import hint is drawn only when the editor has the focus.
4. The import hint is drawn only when an error or warning for an unresolved name sits within 20 lines of the cursor (TypeScript "Cannot find name" errors 2304, 2503 and 2552, or a Python "X is not defined" or unknown-name error).
5. The import hint is drawn only when the language service has an add-import fix for that name.
6. In the shipped defaults the import hint is limited further to names inside text you accepted from Cursor Tab (or just produced by an inline edit) in the last 30 seconds.

### AC-7.21
1. After you have accepted three imports, the chip shrinks to the up-arrow form `↑ import from "module"` (or just `↑ import`), with no key text.
2. The count of accepted imports is stored and never resets.
3. The short form stays in use, because the full wording only returns while the pointer is over the chip and the chip ignores the mouse (read from code, not run).

### AC-7.22
1. While the import hint shows, the unresolved name itself is marked.
2. By default the mark is a yellow double underline over a warning-colored tint, which is the stock warning squiggle look.
3. If the stored style flag is anything other than "squiggle", the mark is a blue tint in the link color at 17 percent opacity instead, and the flag is an app storage value (default "squiggle"), not a setting.

### AC-7.23
1. Pressing Tab applies the language service's add-import fix for the marked name, so the import line appears the way a Quick Fix would add it.
2. After the import is accepted, the hint and the mark disappear.
3. After the import is accepted, the counter of accepted imports goes up by one.
4. An import hint takes priority over every other use of Tab, because it is checked before a suggestion or a jump.
5. If the hint's line is not on screen when Tab is pressed, the hint is hidden and Tab carries on with its other jobs.

### AC-7.24
1. Esc dismisses the import hint and the key press is swallowed, but only when nothing is selected and not in Vim INSERT mode.
2. When a Tab suggestion is showing, Esc rejects that first (AC-6.17).
3. The import hint is dropped when the editor loses text focus or closes.
4. The import hint is dropped when a Tab suggestion covers its line.
5. The import hint is dropped when the marked name is scrolled out of view or no longer matches the error.
6. After a dismissal, the next candidate, if any, can be shown.

### AC-7.25
1. Only JavaScript and TypeScript files (`.js`, `.ts`, `.jsx`, `.tsx`) get the import hint by default.
2. Python files (`.py`) get the import hint only when Python imports are turned on (the "Auto Import for Python" BETA row in Cursor Settings, or a server flag).
3. For Python files the hint reacts to the Python language servers' "not defined" and unknown-name errors.
4. In the short form (AC-7.21) the module text is taken from the quick fix title in one of the forms `import from "x"` (TypeScript), `from x import`, `import x` or `import x as y` (Python).
5. No other language gets the import hint.

## Unclear

- All pieces here are cursor only by nature, so the `[cursor only]` suffix is left off on every piece, as the assignment allowed.
- AC-7.5 piece 4, AC-7.6 and AC-7.21 piece 3 are read from code paths and not run, as the feature blocks say.
- AC-7.8 piece 3 and AC-7.7 piece 5 are context keys that a person only meets in keybinding `when` clauses, so they are not visible on screen.
- AC-7.15 pieces 6 and 7 refer to the path where a jump is shown in place of an edit, which the feature block calls the shortcut path and does not explain further for a person using the editor.
- Hover on the go-back, off-screen and cross-file pills was not found, so no hover piece is written (from the feature block).
