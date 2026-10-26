Group: AC-7 Cursor Tab jump and imports. Section: Autocomplete. Date: 2026-10-25. Pass: 2.
Features: 25.

### AC-7.1 "TAB to jump here" line hint
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it appears only because a next edit was proposed.
- What: When the next edit is in the open file and its line is on screen, a small pill sits just after the end of that line. It reads a key chip with the accept key in upper case ("TAB" unless you rebound it) followed by muted text "to jump here". The pill has a faint tint of the suggest list highlight color, a 4px corner radius, a soft shadow and a 2px accent bar on its left edge; its text is about half strength, the font is the editor size minus 2, and the height is one line minus 2px. On an empty target line the text starts 2px in, otherwise 0.5rem. It ignores the mouse (pointer events off), so it cannot be clicked. Cursor Tab calls this widget the "hint line widget".
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: editor.fontSize; editor.lineHeight
- Cursor: cursor only
- Source: C bundle (hint widget class at byte offset ~40746326: content widget id `cursorPredictionHintLineWidget...`, nodes `span.cursor-prediction-hint-line-widget`, `span.tab-key`, `span.jump-to-edit` "to jump here", class `fancy`; CSS `.cursor-prediction-hint-line-widget`, `.cursor-prediction-hint-line-widget:before`, `.tab-key`)

### AC-7.2 Off-screen "TAB to next move" pill with an arrow
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it appears only because a next edit was proposed.
- What: When the next edit is in this file but its line is outside the visible lines, a pill appears at the top center of the editor (arrow up, target above) or at the bottom center (arrow down, target below). It reads the key chip (accept key in upper case, "TAB" as fallback), then "to next move" (the template text is "to " plus "next move"), then the arrow icon. The pill is small (20px tall), blurred behind (backdrop blur 15px), tinted 7 percent, with a 6px corner radius, and the whole widget is drawn at 90 percent scale. When any inline diff is open, the top pill sits lower (a 56px margin instead of 24px).
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Cursor: cursor only
- Source: C bundle (indicator component at byte offset ~40749424: templates `div.cursorPredictionOutOfRangeIndicator`, `div.cursorPredictionOutOfRangeIndicatorMinimized`, `span.tab-key`, `span.jump-to-edit` "to "; arrow icons `Oe.arrowUp` and `Oe.arrowDown`; overlay widget id `tooltip.editorUiWidget` at ~40756352 with position preference 2; CSS `.cursorPredictionOutOfRangeIndicator`, `.cursorPredictionOutOfRangeIndicatorMinimized`)

### AC-7.3 Hint and pill swap as you scroll
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it follows a proposed next edit.
- What: Scrolling the editor swaps the two same-file indicators: when the target line scrolls into view the pill hides and the line hint (AC-7.1) shows; when it scrolls out of view the hint hides and the pill (AC-7.2) shows, with its arrow flipping to up or down to match the side. The pill also re-checks when the cursor moves, the selection changes, or the prediction changes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`handleEditorScroll` at byte offset ~40776319; the indicator component's scroll, cursor and selection listeners at ~40749424)

### AC-7.4 Click the pill to jump
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it follows a proposed next edit.
- What: Pressing the mouse button on the pill runs the accept action, which takes the same jump as the Tab key (AC-7.5). The pill has a pointer cursor in its minimized form. For the go-back pill (AC-7.12) the click goes back instead.
- Commands: editor.action.acceptCursorTabSuggestion; workbench.action.navigateBack
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (indicator component: `mousedown` handler runs `editor.action.acceptCursorTabSuggestion`, or `workbench.action.navigateBack` for the go-back variant; CSS `.cursorPredictionOutOfRangeIndicatorMinimized` with `cursor:pointer`)

### AC-7.5 Tab jumps to the next edit
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it moves to a proposed edit location.
- What: With a jump hint or pill showing and no suggestion to accept at the cursor, Tab moves the cursor to the first non-blank character of the target line and scrolls that line to the center if it was outside the view. The hint or pill is cleared by the jump. If a suggestion for the current spot is also showing and it came from a jump proposal, the first Tab only jumps, and a following Tab then accepts it (read from the code path, not run). A press is ignored when the cursor line is empty (AC-7.16).
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`maybeAcceptCursorPrediction` at byte offset ~40761500: `setPosition({lineNumber, column: first non-whitespace}, "cursorPrediction")` and `revealLineInCenterIfOutsideViewport`; `tabToLineBeforeAcceptingCpp` at ~40772962; `handleTabAction` at ~40739781)

### AC-7.6 A suggestion can appear on arrival
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the new suggestion is generated content.
- What: After a jump, a new Cursor Tab suggestion can show at the landing line (a flag is set when the hint is created or shown again after Esc), so a suggestion may appear right after the cursor arrives and a second Tab can accept it. Whether one appears was not run; this is read from the code path that follows a jump.
- Commands: none found
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`retriggerCppOnAccept` set in `manuallyCreateCursorPrediction` and `maybeShowHintLineWidget`, used at the end of `maybeAcceptCursorPrediction`, byte offset ~40761500)

### AC-7.7 Cross-file pill with file icon, name and arrow
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it appears only because a next edit in another file was proposed.
- What: When the next edit is in a different file, the pill at the bottom center of the editor reads the key chip, then "to", then the file's icon and its file name (last path part only, bold italic), then an arrow pointing right. Clicking it or pressing Tab starts the jump (AC-7.8, AC-7.10). While it is showing, the context key `tabPredictionGoToFileVisibleKey` ("Whether a cursor prediction for a target file is currently visible") is true.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true; when tabPredictionGoToFileVisibleKey
- Settings: none found
- Cursor: cursor only
- Source: C bundle (indicator component at byte offset ~40749424: `JWw` template `div > span.file-path`, file icon component with `fileName`, arrow `Oe.arrowRight`; key declared at ~40643366; CSS `.cursorPredictionOutOfRangeIndicator .file-path`, `.file-icon`)

### AC-7.8 First Tab reveals the code preview
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the preview shows a proposed edit location.
- What: On a cross-file pill whose code preview is hidden, the first Tab (or click) does not jump: it only reveals the code preview above the pill. The next Tab then jumps. The context key `tabPredictionCodePreviewVisibleKey` ("Whether a code preview for a target file is currently visible") turns true when the preview shows.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true; when tabPredictionCodePreviewVisibleKey
- Settings: none found
- Cursor: cursor only
- Source: C bundle (accept action at byte offset ~40745774: if go-to-file key is true and code-preview key is false it calls `setPreviewVisible(!0)` and returns; keys declared at ~40643366)

### AC-7.9 Code preview of the target file
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the preview shows a proposed edit location.
- What: The code preview is a small read only editor, 300px wide and 7 lines tall, drawn above the cross-file pill. It shows the target file in its own language, centered on the target line (3 lines above it). Every line except the target line is dimmed to half strength, and the target line carries a bright "TAB to jump here" hint (AC-7.1).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (code preview builder `rGw` at byte offset ~40754567: height of 7 lines, width 300px, decorations `cursor-prediction-dimmed-line` and `cursor-prediction-dimmed-line-inline`, the hint widget `highlight()`; CSS `.cursor-prediction-dimmed-line-inline{opacity:.5}`)

### AC-7.10 Jump across files opens the target file
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it moves to a proposed edit location in another file.
- What: A jump to another file switches to that file and puts the cursor on the target line (first non-blank character), scrolling it to the center if needed. If an editor group already shows the file, that group is used and nothing new opens. If no group shows it, the file is opened in the active group (not to the side) with the focus moved to it.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`maybeAcceptCursorPrediction` at byte offset ~40761500: group lookup by `activeEditor.resource`, helper `oGw` calling the opener with `{openToSide: inNewTab, editorOptions: {revealIfVisible, revealIfOpened, preserveFocus}}` then `r.focus()`)

### AC-7.11 Target file already open in a split
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it points at a proposed edit location in another editor.
- What: When the target file is already open in another editor group, the active editor shows the cross-file pill (AC-7.7) and the other editor gets the markers for its own file: the "TAB to jump here" line hint on the target line if that line is visible there (AC-7.1), or the off-screen pill without a file name (AC-7.2) if it is not. Scrolling the other editor swaps between the two, as in AC-7.3.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`createMultifileCursorPrediction` at byte offset ~40769674: shows the pill with a relative path in the active editor, a hint widget or a pill with an empty path in the editor that holds the target; `handleEditorScroll` at ~40776319 with the `!s&&...` branch)

### AC-7.12 "to go back" pill after a cross-file jump
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it follows a jump to a proposed edit location.
- What: After a jump that switched files, a pill at the bottom center shows a key chip with the key bound to Go Back (for example Alt+Left on Windows) and the text "to go back". Clicking it or pressing that key goes back. It is shown at most twice in a window session (the count is kept in memory, not saved) and is not shown at all if Go Back has no key. It hides when the selection changes or the editor loses focus.
- Commands: workbench.action.navigateBack
- Menus: none found
- Keys: Alt+Left on Windows (Ctrl+- on Mac, Ctrl+Alt+- on Linux)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`goBackIndicatorDisplayCount` and `MAX_GO_BACK_INDICATOR_DISPLAYS=2` in the prediction service at byte offset ~40758857; indicator component with `isGoBackIndicator` and template `span.jump-to-edit` " to go back" at ~40749424); V\src\vs\workbench\browser\parts\editor\editorActions.ts (`NavigateBackwardsAction`, keys)

### AC-7.13 Esc dismisses the jump hint
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it dismisses a proposed edit location.
- What: With no Tab suggestion showing, Esc clears the jump hint and pill (and a cross-file code preview). If an import hint is also showing, the first Esc dismisses the import hint and swallows the key, and a second Esc clears the jump. Clearing by Esc counts as a rejection of that target (AC-7.15). With a Tab suggestion showing, Esc rejects the suggestion instead and the jump hint stays or is shown again (AC-6.17).
- Commands: none found
- Menus: none found
- Keys: Escape (no Ctrl, Alt or Cmd); not a registered key binding
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`handleKeyDownForCppKeys` at byte offset ~40645600 to ~40648200: `clearCursorPrediction()` in the Escape branch, after `maybeRejectShownImport`; `clearPrediction` at ~40764097)

### AC-7.14 Jump hint cleared by edits, focus loss, file switch and resize
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it removes a proposed edit location.
- What: The hint and pills disappear (and the target counts as rejected, except for resize) when you type or otherwise change the file, when the editor loses text focus, when the editor shows another file, or when the editor is closed. Resizing the window clears the hint and pills in every editor without counting a rejection. Deleting or removing the target file clears them too.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`createPrediction` at byte offset ~40767469: listeners `onDidChangeModel`, `onDidBlurEditorText`, `onDidChangeModelContent` calling `clearPrediction({registerReject:!0})`; window resize listener in the service constructor at ~40760408 and `clearPredictionForDeletedFile` at ~40765566)

### AC-7.15 When no jump is offered
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it limits when a proposed edit location is shown.
- What: A jump hint is skipped when the target is within 1 line of the cursor, when it lies inside the range of the suggestion you just accepted, when it overlaps an inline diff that is still being written, when its line number is past the end of the file, and when you dismissed that same line twice in the last 30 seconds (the last 5 dismissed lines are remembered). In the shortcut path where the jump is shown in place of an edit, it is also skipped when the target is under 5 lines from a spot you already jumped to since your last edit, and straight after a jump.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`createPrediction` and `createMultifileCursorPrediction` at byte offsets ~40767469 and ~40769674; `wasRecentlyRejected` with `REJECTION_EXPIRY_MS=30*1e3` and `MAX_REJECTED_PREDICTIONS=5` at ~40777532; bad-heuristic check `isValidCursorPredictionCase` at ~40758107; `immediatelyShowNextCursorPrediction` at ~40713228)

### AC-7.16 Tab on an empty line does not jump
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it decides whether a proposed jump is taken.
- What: When the cursor is on an empty line, Tab does not jump: the jump handler returns without moving the cursor, and in the shortcut path the jump is not offered at all because Tab would indent there. In Vim modes other than INSERT the rule is skipped. This is the jump version of the blank line rule for suggestions (AC-6.21).
- Commands: none found
- Menus: none found
- Keys: Tab
- Settings: none found
- Cursor: cursor only
- Source: C bundle (prediction service `shouldTabInsteadOfAccepting(editor, model)` at byte offset ~40763478; `cursorPredictorWouldTabInsteadOfAccepting` at ~40668152 used by `immediatelyShowNextCursorPrediction`)

### AC-7.17 Hint hides under a suggestion box
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it resolves a clash between two generated hints.
- What: If a Tab preview box or gray text would cover the jump target line, the jump hint and pill are hidden so they do not collide; the import hint is hidden the same way when a suggestion covers its line. After Esc rejects the suggestion, the hints are shown again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`hideWidgetsIfConflictsWithCppSuggestion` in the prediction service at byte offset ~40766323 and in the import service at ~40807456; called from the preview box code in `showChangesOnTheRight`)

### AC-7.18 Jump targets in notebooks
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it points at a proposed edit location.
- What: When the target is in a `.ipynb` notebook, the jump is treated as an in-editor jump: the line hint and the off-screen pill (AC-7.1, AC-7.2) are used, not the cross-file pill with a file name.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`manuallyCreateCursorPrediction` at byte offset ~40773201: `predictedUri?.path.endsWith(".ipynb")` chooses `createPrediction` over `createMultifileCursorPrediction`)

### AC-7.19 Import hint "import X (press Tab or ESC)"
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it offers a suggested import.
- What: A small chip appears at the end of the line that holds an unresolved name, reading `import {name} (press {key} or ESC)`, for example "import useState (press Tab or ESC)". The key is the label of the accept key as the keybinding service prints it, "TAB" if none is found. The chip has a 4px corner radius, a border in the link color, text 0.85em in the editor font and editor foreground, 80 percent opacity, and a gap of 0.75em before it. It ignores the mouse (pointer events off), so it is keyboard only. Only one import hint shows at a time.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true; Escape
- Settings: none found
- Cursor: cursor only
- Source: C bundle (import widget class at byte offset ~40778198: node `div.importPredictionWidgetBackground` with class `import-prediction-widget fancy`, `updateTextContent` at ~40780065; CSS `.import-prediction-widget`, `.import-prediction-widget.fancy span`)

### AC-7.20 When the import hint shows
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it offers a suggested import.
- What: The hint is drawn only when all of these hold: the "Imports" switch in Cursor Settings is on (AC-8); the open file is JavaScript or TypeScript (`.js`, `.ts`, `.jsx`, `.tsx`), or Python (`.py`) when Python imports are on; the editor has the focus; an error or warning for an unresolved name sits within 20 lines of the cursor (TypeScript "Cannot find name" errors 2304, 2503 and 2552, Python "X is not defined" or an unknown-name error); and the language service has an add-import fix for it. In the shipped defaults it is limited further to names inside text you accepted from Cursor Tab (or just produced by an inline edit) in the last 30 seconds.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (import service at byte offset ~40782091: `doesSupportFile` at ~40784439, marker-to-name table for TypeScript codes 2304, 2503, 2552 and Python rules, distance constant 20; `markerTouchesGreens` with a 30 second window; defaults `shouldOnlyImportOnAccept:!0` at ~17166266)

### AC-7.21 Short import hint after three accepted imports
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it offers a suggested import.
- What: After you have accepted three imports (a stored counter that never resets), the chip shrinks to the up-arrow form `↑ import from "module"` (or just `↑ import`), with no key text. The code switches back to the full wording while the pointer is over the chip, but the chip ignores the mouse, so the full wording probably never returns in use (not run).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`updateTextContent` at byte offset ~40780065: `howManyTimesHasUserAcceptedImportPrediction<3||_isHovering`, strings `\u2191 import from "..."` and `\u2191 import`; `mouseenter` and `mouseleave` listeners on the chip; CSS `.import-prediction-widget{pointer-events:none}`)

### AC-7.22 Mark on the unresolved name
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it marks a name for a suggested import.
- What: While the import hint shows, the unresolved name itself is marked. By default it is a yellow double underline over a warning-colored tint (the stock warning squiggle look, class `squiggly-ai cpp-pending-import-decoration`). If the stored style flag is anything other than "squiggle", it is a blue tint in the link color at 17 percent opacity instead. The style flag is an app storage value (default "squiggle"), not a setting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`showImport` at byte offset ~40798935: decoration class by `cppAutoImportDecorationStyle`; default `cppAutoImportDecorationStyle:"squiggle"` at ~17166266; CSS `.monaco-editor .squiggly-ai`, `.auto-import-suggestion-blue-background`, `.cpp-pending-import-decoration`)

### AC-7.23 Accept the import with Tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it applies a suggested import.
- What: Tab applies the language service's add-import fix for the marked name, so the import line appears the way a Quick Fix would add it. The hint and the mark disappear and a counter of accepted imports goes up by one. An import hint takes priority over every other use of Tab: it is checked before a suggestion or a jump. If the hint's line is not on screen when Tab is pressed, the hint is hidden and Tab carries on with its other jobs.
- Commands: editor.action.acceptCursorTabSuggestion
- Menus: none found
- Keys: Tab when cpp.shouldAcceptTab == true
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`maybeAcceptShownImport` at byte offset ~40795753, called first in `handleTabAction` at ~40739781; `updateShouldAcceptTabContextKey` sets the key true while `isShowingImportSuggestion()`)

### AC-7.24 Dismiss the import hint
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it dismisses a suggested import.
- What: Esc dismisses the import hint and the key press is swallowed, but only when nothing is selected and not in Vim INSERT mode; when a Tab suggestion is showing, Esc rejects that first (AC-6.17). The hint is also dropped when the editor loses text focus, when the editor closes, when a Tab suggestion covers its line, and when the marked name is scrolled out of view or no longer matches the error. After a dismissal the next candidate, if any, can be shown.
- Commands: none found
- Menus: none found
- Keys: Escape (no Ctrl, Alt or Cmd); not a registered key binding
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`maybeRejectShownImport` at byte offset ~40796987; blur listener in `showImport` at ~40798935; `hideWidgetsIfConflictsWithCppSuggestion` at ~40807456; Esc branch in `handleKeyDownForCppKeys` at ~40645600)

### AC-7.25 Import hint by language
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it offers a suggested import.
- What: Only JavaScript and TypeScript files (`.js`, `.ts`, `.jsx`, `.tsx`) get the hint by default. Python files (`.py`) get it only when Python imports are turned on (the "Auto Import for Python" BETA row in Cursor Settings, or a server flag), and then the hint reacts to the Python language servers' "not defined" and unknown-name errors. In the short form (AC-7.21) the module text is taken from the quick fix title in one of the forms `import from "x"` (TypeScript), `from x import`, `import x` or `import x as y` (Python). No other language gets the hint.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`doesSupportFile` at byte offset ~40784439 with the `.py` extension gated by `pythonEnabled()`; marker tables for sources `ts`, `Pylance`, `basedpyright` and `Pyrefly` near ~40782000; title regex in `updateTextContent` at ~40780065)

## Handed off

- The "Imports" and "Auto Import for Python" switches, the Cursor Tab on/off and snooze controls, and the status bar item: AC-8.
- The accept action for suggestions, Esc to reject a suggestion, the Search and Problems Tab handlers, and the preview box, peek and gray text shapes: AC-6.
- The stock inline edit jump (`editor.action.inlineSuggest.jump`, "Jump to next inline edit") and the inline edit gutter indicator: AC-2.
- The stock Go Back command and its default keys (shown here only because the go-back pill uses it): the editor group that owns navigation history (ED section).

## Unclear

- Whether the line hint, the pill and the code preview ever differ in the shipped build: the state of the stored flag `isFusedCursorPredictionModel` (it changes a few rules: whether Tab on an empty line is skipped, and whether typing clears the hint) is set at run time and was not checked.
- Whether Tab still indents on an empty line when a jump hint is showing: the jump handler returns without moving, but the key rule `cpp.shouldAcceptTab` stays true for a jump, so the Tab key press may be consumed. Not run.
- Whether the second Tab after a cross-file jump, or after a jump that is followed by a new suggestion, behaves as described in AC-7.5 and AC-7.6: read from code, not run.
- The preview-visible flag is set to true on the first Tab and no place that sets it back to false was found, so the code preview may stay visible for later cross-file pills in the same session.
- Hover on the go-back, off-screen and cross-file pills: only a mouse down handler was found, no hover content.
- Resolved light and dark colors of the Cursor-only CSS were not computed.
