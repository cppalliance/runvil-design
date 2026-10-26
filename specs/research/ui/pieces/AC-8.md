Group: AC-8 Cursor Tab controls. Pass: 3. Date: 2026-10-25.

### AC-8.1
1. A right-side status bar item named "Cursor Tab" shows the text "Cursor Tab" when Cursor Tab is on and the open file is allowed.
2. While a snooze is running the item text reads "Cursor Tab (Snoozed)" and the text is not struck through.
3. When the open file matches the cursor ignore rules the item text reads "Cursor Tab (Ignored)", and "(Snoozed)" wins if both are true.
4. When Cursor Tab is not active for the open file the item is drawn dimmed to half opacity with a line through the text (class `copilot-plus-plus-status-off`).
5. The dimmed, struck-through look appears when Cursor Tab is off globally or is not allowed for the signed-in account.
6. The dimmed, struck-through look appears when the file's language is disabled, the file matches an Ignored Files pattern, or the file is blocked by the cursor ignore rules.
7. The dimmed, struck-through look appears when the file is over 5,000,000 characters or is a `.env` file opened as plaintext.
8. The item's text is also its aria label, so a screen reader announces the same text that is drawn.

### AC-8.2
1. Hovering the "Cursor Tab" status item shows a small panel.
2. Clicking the status item while signed in keeps the panel open instead of hiding it, using `statusBar.entry.showTooltip`.
3. Clicking the status item while signed out runs `editor.cpp.login` (Log In to Cursor) instead of showing the panel.
4. The panel rows from top to bottom are the "Disable globally" checkbox row, the "Disable for {language}" checkbox row, the "Model" row with a select, and the Snooze row.
5. The "Disable for {language}" row shows only while a text editor is active.
6. Each panel row is one line with 4px padding, the label on the left and the control on the right.

### AC-8.3
1. Running "Enable Cursor Tab" from the Command Palette (`editor.action.enableCppGlobally`) turns Cursor Tab on.
2. Running "Disable Cursor Tab" from the Command Palette (`editor.cpp.disableenabled`) turns Cursor Tab off.
3. Running "Toggle Cursor Tab" from the Command Palette (`editor.cpp.toggle`) flips Cursor Tab between on and off.
4. Ticking the "Disable globally" checkbox in the status panel turns Cursor Tab off and unticking it turns Cursor Tab on.
5. The "Cursor Tab" switch in Cursor Settings changes the same on or off flag as the commands and the panel checkbox.
6. Turning Cursor Tab on with Enable, Toggle, or the Cursor Settings switch also ends a running snooze.
7. Changing the on or off flag updates the context key `cppEnabled`, and the flag lives in app storage so no setting id for it shows in settings.json.

### AC-8.4
1. Ticking "Disable for {language}" in the status panel adds the active editor's language to the user setting `cursor.cpp.disabledLanguages`, and unticking removes it.
2. The "Disable for {language}" checkbox shows ticked whenever the active editor's language is in `cursor.cpp.disabledLanguages`.
3. For an untitled file the language word in the row is "untitled".
4. Changing the editor's language rebuilds the "Disable for {language}" row for the new language.
5. Setting `cursor.cpp.disabledLanguages` sits in a settings group titled "Cursor Tab" as an array of strings, default `["markdown","plaintext"]`, with the description "Disable Cursor Tab for these languages".
6. Cursor Tab is also off for `.env` style files opened as plaintext, without any setting.

### AC-8.5
1. The "Model" row of the status panel holds a bordered drop-down with a small triangle.
2. The first option in the drop-down reads "auto (default)", followed by the names the app keeps refreshed.
3. Picking an option stores the choice, and picking "auto (default)" clears the stored choice.
4. The Model drop-down exists only in the status panel, and the Cursor Tab group in Cursor Settings has no Model row.

### AC-8.6
1. Running "Snooze Cursor Tab" from the Command Palette (`editor.cpp.snooze`) opens a quick pick with the placeholder "Select Cursor Tab snooze duration".
2. The quick pick has six rows, each with the description "Snooze for {label}": "1 minute", "5 minutes", "15 minutes", "30 minutes", "1 hour", "3 hours".
3. Picking a row turns Cursor Tab off for that time and the status item text becomes "Cursor Tab (Snoozed)".
4. "Snooze Cursor Tab" shows in the Command Palette only while Cursor Tab is on and not snoozed, and does nothing if Cursor Tab is off.
5. The "Snooze" link in the status panel runs the same command.
6. An info icon beside the Snooze link has the tooltip "Temporarily disable Cursor Tab suggestions for a specified duration. You can unsnooze at any time."
7. The Snooze row is dimmed and ignores clicks when Cursor Tab is off and not snoozed.

### AC-8.7
1. Running "Unsnooze Cursor Tab" from the Command Palette (`editor.cpp.unsnooze`) ends a snooze at once and puts the on or off flag back to what it was before the snooze.
2. "Unsnooze Cursor Tab" shows in the Command Palette only while Cursor Tab is snoozed.
3. While snoozed, the Snooze link in the status panel turns into "Unsnooze".
4. A grey "({time} remaining)" count next to the Unsnooze link ticks every second, written "Xh Ym" above an hour and "Xm Ys" below an hour.
5. When the snooze time runs out Cursor Tab turns itself back on.
6. A snooze that ended while the app was closed is cleared the next time the status item is built.
7. Unticking "Disable globally" in the status panel during a snooze also ends the snooze.

### AC-8.8
1. When signed out, the status panel shows the one line "Please log in" instead of its rows.
2. When signed in but Cursor Tab is not allowed for the account, the panel shows "Requires pro (custom model). " followed by an "Upgrade here." link instead of its rows.
3. Clicking the "Upgrade here." link starts the upgrade flow.
4. In local mode the panel shows "Cursor Tab is unavailable in Cursor Private Inference".
5. When private inference turns the Tab feature off the panel shows "Cursor Tab: Off under private inference".
6. In all of these states the status item text stays "Cursor Tab" and the item is drawn dimmed.

### AC-8.9
1. Running "Log In to Cursor" from the Command Palette (`editor.cpp.login`) starts the sign-in flow.
2. "Log In to Cursor" is in the Command Palette at all times, with no condition, even when already signed in.
3. Clicking the "Cursor Tab" status item while signed out runs the same sign-in command.

### AC-8.10
1. The Cursor Settings "Code Intelligence" tab (id `indexing`) has a group named "Cursor Tab" with the anchor `cursor-settings-cursor-tab`.
2. The first row of the group is a switch labelled "Cursor Tab" with the description "Context-aware, multi-line suggestions around your cursor based on recent edits".
3. The six tuning rows (Partial Accepts, Suggestions While Commenting, Whitespace-Only Suggestions, Imports, Auto Import for Python, Ignored Files) show only while the "Cursor Tab" switch is on.
4. A muted line under the group reads "CPP and auto-imports are temporarily disabled", "CPP is temporarily disabled" or "Auto-imports are temporarily disabled" when the server side turns those off.
5. The old hidden Cursor Settings tab id `tab` redirects to `indexing`.
6. The Cursor Settings search has an entry "Cursor Tab", and the category also answers to "cursor tab" and "tab".

### AC-8.11
1. The "Partial Accepts" row has a switch and the description "Accept the next word of a suggestion via {keybinding}".
2. A small info button beside the "Partial Accepts" label has the tooltip "Keybinding: {keybinding}" and the aria label "Partial Accepts keybinding: {keybinding}".
3. The {keybinding} text is the key bound to `editor.action.inlineSuggest.acceptNextWord`, and reads "Alt+]" when no key is bound.
4. The switch writes the user setting `cursor.cpp.enablePartialAccepts` (boolean, default false) with the description "Enable partial accepts for Cursor Tab, using the editor.action.inlineSuggest.acceptNextWord keybinding".
5. With the switch on, the context key `cppSuggestion` is true while a Cursor Tab suggestion exists, so the word-accept key also works on Cursor Tab suggestions.
6. The inline suggestion toolbar shows an "Accept Word" item (group primary, order 2) for `editor.action.inlineSuggest.acceptNextWord`, bound to `CtrlCmd+Right` while the editor is writable and a suggestion is visible or `cppSuggestion` is true.

### AC-8.12
1. The "Suggestions While Commenting" switch has the description "Allow Tab to trigger while in a comment region".
2. The "Suggestions While Commenting" switch is off by default.
3. With the switch on, Cursor Tab can trigger while the cursor is in a comment region.
4. The choice is kept as an app flag, so no settings.json key exists for it.

### AC-8.13
1. The "Whitespace-Only Suggestions" switch has the description "Suggest edits like new lines and indentation that modify whitespace only".
2. The "Whitespace-Only Suggestions" switch is off by default.
3. With the switch on, Cursor Tab can suggest edits that change only whitespace, such as new lines and indentation.
4. The choice is kept as an app flag, so no settings.json key exists for it.

### AC-8.14
1. The "Imports" switch has the description "Automatically import necessary modules for TypeScript".
2. The "Imports" description reads "...for TypeScript and Python" when the server config already includes Python.
3. The "Imports" row is hidden when the server has turned auto import off and you never set the switch.
4. The "Imports" switch is off by default.
5. Switching "Imports" off also dismisses an import hint that is on screen.
6. The choice is kept as an app flag, so no settings.json key exists for it.

### AC-8.15
1. The "Auto Import for Python" switch has a small "BETA" badge beside its label and the description "Enable auto import for Python. This is a beta feature."
2. The "Auto Import for Python" row shows only when the "Imports" row is shown and switched on.
3. The "Auto Import for Python" row is hidden when the server config already includes Python.
4. The "Auto Import for Python" switch is off by default.
5. The choice is kept as an app flag, so no settings.json key exists for it.

### AC-8.16
1. The "Ignored Files" row has the description "Glob patterns for files where Cursor Tab will not suggest".
2. The row holds an editable pattern list field, aligned to the top of the row, with the aria label "Ignored Files" and the placeholder "e.g., *.md, **/generated/**".
3. Cursor Tab does not suggest in a file that matches one of the patterns.
4. For a matching file the status item is dimmed and struck through but its text stays "Cursor Tab", not "Cursor Tab (Ignored)", which is for the cursor ignore files.
5. The patterns are kept as an app flag, so no settings.json key exists for them.

### AC-8.17
1. The context key `cppEnabled` ("Whether Cursor Tab is currently enabled", default true) can be used in `when` clauses.
2. The context key `cppSnoozed` ("Whether Cursor Tab is currently snoozed") can be used in `when` clauses.
3. The context key `cppSuggestion` is true when a Cursor Tab suggestion exists and partial accepts is on.
4. The context key `cpp.shouldAcceptTab` says whether the Tab key should accept a suggestion, and is always true when the accept command is rebound away from Tab.
5. The Search view and the Problems view also read `cpp.shouldAcceptTab` so they can catch the Tab key.
6. Pressing Tab runs `editor.action.acceptCursorTabSuggestion` when `cpp.shouldAcceptTab` is true.
7. Pressing `CtrlCmd+Right` runs `editor.action.inlineSuggest.acceptNextWord` when the editor is writable and `inlineSuggestionVisible` or `cppSuggestion` is true.
8. No default key is bound to Enable, Disable, Toggle, Snooze, Unsnooze or Log In, so a person can bind them in keyboard shortcuts using these keys.

### AC-8.18
1. "Developer: Copy Recent Cursor Tab Actions" (`workbench.action.copyRecentCppActions`) is in the Command Palette for everyone, with no condition.
2. Running it copies the newest ten recorded Cursor Tab requests to the clipboard.
3. After copying, the message "Copied N Cursor Tab request IDs + diffs to clipboard" shows.
4. When there are no recorded requests the error "No Cursor Tab actions found" shows.

### AC-8.19
1. "Developer: Report Cursor Tab Action" (`workbench.action.reportCppAction`) is in the Command Palette for everyone, and opens a quick pick with the placeholder "Select a Cursor Tab request (last 20 requests)".
2. Each row of the first quick pick is named "{request id} - {time}".
3. When there are no recorded requests the error "No Cursor Tab actions found" shows.
4. Picking a row opens a second quick pick with the placeholder "Choose an action" and the rows "Report Good", "Report Bad" and "Report with Comment".
5. "Report with Comment" asks for text in an input box with the placeholder "Type your comment here...".
6. The "Copy Request ID" row is greyed with "Disabled in privacy mode" when privacy mode is on.
7. The "Open in Prompt Quality" row shows only for developer users (context key `isDevUser`).

## Unclear
- All 19 features here are Cursor only, so the `[cursor only]` marker is left off every piece. The whole group is Cursor only, and a tag on every line would add nothing.
- AC-8.6: the stored default snooze length of 5 minutes when none is stored has no visible trigger in the feature block, so it has no piece. The "3 hours" row is coded as 4 hours in the bundle, and the piece names the label only.
- AC-8.3: the three palette commands each get a piece, and the Cursor Settings switch is covered here as an entry point and in AC-8.10 as a row. The two do not repeat the same behavior.
- AC-8.5: the feature says picking an option reloads the Cursor Tab setup. That has no visible effect described, so it is not a piece.
- AC-8.10: the open command `aiSettings.action.open` and the tab frame are owned by ED-8.19 and ED-8.30, so they have no piece here.
- AC-8.11: the key chord and toolbar item are listed as entry points, and the accept action itself is AC-6. The "Alt+]" fallback is shown in the row text only and was not checked against a real binding.
- AC-8.17: pieces 6 and 7 are entry points only. The accept actions are AC-6.
- AC-8.18 and AC-8.19: the copied text and the row previews are Tab output and are not described. Only the commands and pickers have pieces. The registered original text of AC-8.19 is "See Recent Cursor Tab Actions", and the piece uses the Developer label from the feature block.
