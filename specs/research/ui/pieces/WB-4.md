Group: WB-4 Accessibility. Pass: 3. Date: 2026-10-25.

### WB-4.1
1. In an Accessibility Help page that lists commands without keys, `Alt+K` opens a picker titled "Configure keybindings" with the placeholder "Select a command ID to configure a keybinding for it".
2. `Alt+A` opens the same picker for the commands on the page that already have a key.
3. Picking a command opens the Keyboard Shortcuts editor with that command id in its search box, and Escape returns to the help page.
4. A record-keys button for each list appears in the title toolbar of the accessible view, and only when the page has commands of that kind.
5. The help text carries the lines "Configure keybindings for commands that lack them" and "Configure keybindings for commands that already have assignments", each followed by its key.

### WB-4.2
1. With an Accessibility Help page open, `Alt+H` opens that page's "read more" link in the web browser, for example the editor help and the terminal help.
2. A page without a link ignores the key, and its text does not mention it.
3. A page with a link carries the line "Open a browser window with more information related to accessibility" followed by the key.
4. The key works only in the help page, not in the plain accessible view.

### WB-4.3
1. Pressing `Alt+F6` while the accessible view or an Accessibility Help page is open sets that page's verbosity setting, such as `accessibility.verbosity.editor`, to false.
2. A screen reader announces "<setting id> accessibility verbosity is now disabled".
3. The bell-with-a-slash button in the accessible view title toolbar and the Command Palette row "Disable Accessible View Hint" do the same.
4. The command is available only while the page's verbosity setting is on.
5. Each verbosity setting is a true or false value, true by default, and the person can turn it back on in Settings.
6. The help text lists the action as "Disable accessibility verbosity for this feature" followed by the key.

### WB-4.4
1. Pressing `Ctrl+Shift+O` (or `Ctrl+Shift+.`) opens a picker titled "Go to Symbol Accessible View" with the placeholder "Type to search symbols".
2. For a text page each row is a heading, paragraph, code block or list item labelled with its kind, such as "(heading) Title".
3. Choosing a row closes the picker and shows the page scrolled to that line with the cursor in column 1.
4. Escape closes the picker without choosing and puts focus back on the accessible view.
5. The same action is on the title toolbar of the accessible view as a symbol button and in the Command Palette, and it is available only when the page has symbols.
6. The help text lists the action as "Go to a symbol" followed by the key.

### WB-4.5
1. Each event is one setting named `accessibility.signals.<event>` that holds a `sound` value of auto (plays when a screen reader is attached), on or off and, for most events, an `announcement` value of auto (speaks only in screen reader optimized mode) or off.
2. Moving the cursor to a different line that has an error, a warning, a breakpoint or a folded area plays that line's sound or announcement once for the line (`lineHasError`, `lineHasWarning`, `lineHasBreakpoint`, `lineHasFoldedArea`).
3. Placing the cursor on the text of an error or a warning, or having a marker newly appear under a still cursor, plays a separate signal (`positionHasError`, `positionHasWarning`).
4. Stepping through changes in a diff, or moving over a changed line in a quick diff peek, plays a sound for a deleted line or a modified line (`diffLineDeleted`, `diffLineModified`, with `diffLineInserted` for added lines), sound only.
5. Choosing a code action plays one sound when it starts and another when it has been applied (`codeActionTriggered`, `codeActionApplied`), sound only.
6. Saving a file and formatting a file take `userGesture`, `always` or `never` for sound and for announcement, default `never`, so auto save and format on save can be told apart from the person asking for it (`save`, `format`).
7. The debugger stopping on a breakpoint (`onDebugBreak`), a task finishing or failing with a non-zero exit code (`taskCompleted`, `taskFailed`), and a notebook cell finishing or failing (`notebookCellCompleted`, `notebookCellFailed`) each play a sound or announcement.
8. Asking to read the inlay hints of a line that has none plays `noInlayHints`.
9. Help: List Signal Sounds and Help: List Signal Announcements open a checkable list of every signal where the highlighted sound plays as a preview, Enter applies the on and off choices, a gear button on the save and format rows opens that setting in settings.json, and the announcements list changes nothing unless a screen reader is attached.
10. In Cursor the code action triggered and applied signals play the voice recording started and stopped sounds instead of their own sound files. [cursor only]

### WB-4.6
1. `accessibility.signalOptions.volume` sets the loudness of every signal sound from 0 to 100 percent, default 70, and 0 makes them silent.
2. `accessibility.signalOptions.debouncePositionChanges` (default off) makes line and position signals wait before they play, so quick cursor movement does not play every one; with it off they play at once.
3. `accessibility.signalOptions.experimental.delays.general` sets a sound delay (default 400 ms) and an announcement delay (default 3000 ms) for every signal except an error or warning at the cursor position.
4. `accessibility.signalOptions.experimental.delays.errorAtPosition` sets the sound delay (default 1000 ms) and announcement delay (default 3000 ms) for an error under the cursor.
5. `accessibility.signalOptions.experimental.delays.warningAtPosition` sets the sound delay (default 1000 ms) and announcement delay (default 3000 ms) for a warning under the cursor.
6. The three delay settings only take effect while `debouncePositionChanges` is on.
7. Values saved under the older `audioCues.volume`, `audioCues.debouncePositionChanges` and `accessibility.signals.sounds.volume` names move to the new setting names automatically.

### WB-4.7
1. With `accessibility.underlineLinks` on, links inside paragraphs of workbench text, such as notification and dialog messages, are drawn underlined.
2. With the setting off, the default, those links are drawn without an underline.
3. Links in hover cards, the walkthrough and Welcome pages and the language status popup follow the same switch.
4. Links in webview pages are given the same underline choice and change when the setting changes.
5. Changing the setting takes effect at once without reloading the window.
6. High contrast light and dark themes always underline links in paragraphs, whatever the setting says.

### WB-4.8
1. Running Toggle Screen Reader Accessibility Mode from the Command Palette writes `editor.accessibilitySupport` into the User settings, as `on` when screen reader mode was off and as `off` when it was on.
2. A screen reader announcement says "Screen Reader Optimized Mode enabled." or "Screen Reader Optimized Mode disabled." to match the new state.
3. Shift+Alt+F1 (Shift+Alt+F4 on Linux) runs the command from anywhere.
4. While the accessibility help dialog is open, Ctrl+E (Cmd+E on macOS) runs the command, so the mode can be switched without closing the dialog.
5. When the setting is `auto` and a screen reader is detected, the mode already counts as on, so running the command sets the setting to `off`.
6. While the mode is on, the status bar shows the Screen Reader Optimized item (see SS-3.26).

### WB-4.9
1. Typing Announce Cursor Position in the Command Palette runs the command, and it is listed whether or not screen reader mode is on. [stock only]
2. Ctrl+Alt+Shift+G (Cmd+Alt+Shift+G on macOS) runs it, but only while an editor has text focus and screen reader mode is on. [stock only]
3. The screen reader announcement reads "Line {line}, Column {column}" for the primary cursor. [stock only]
4. The column is the visible column, so a tab counts as its width, which matches the position shown in the status bar. [stock only]
5. With no focused editor, or an editor with no text model, nothing is announced. [stock only]

## Unclear
none
