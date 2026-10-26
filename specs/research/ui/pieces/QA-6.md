Group: QA-6 Quick pick and input box widget. Pass: 3. Date: 2026-10-25.

### QA-6.1
1. A picker opens as a floating box with a filter field showing the picker's placeholder and a list of rows under it, and keyboard focus starts in the filter field.
2. Typing in the filter field narrows the rows and highlights the matching letters in each row's label.
3. Rows are matched on the label by default, and on the description and detail only when the picker asks for it, while rows marked "always show" stay visible whatever you type.
4. The first row is active when the list opens.
5. Pressing Enter picks the active row and clicking a row picks that row, and either one closes the box.

### QA-6.2
1. When a picker allows many choices, every row shows a checkbox at its left.
2. Pressing Space with focus in the list toggles the active row's checkbox, while Space in the filter field types a space.
3. Pressing Enter confirms all checked rows.
4. A picker can open with some rows already checked.
5. A disabled row shows a greyed checkbox that cannot be changed.
6. Rows marked not pickable show no checkbox.
7. The command `workbench.action.quickPickManyToggle` has no default key and, when bound, toggles the active row's checkbox.
8. The command `quickInput.toggleCheckbox` that Space runs exists only in stock, and Cursor has no command id of that name. [stock only]

### QA-6.3
1. A multi-select picker shows a check-all box at the left of the header with the tooltip "Toggle all checkboxes", and clicking it checks every visible row and clicking it again unchecks them.
2. A count badge at the right of the filter field reads "{0} Selected" with the number of checked rows.
3. An OK button confirms the checked rows, and its label reads "OK" unless the picker sets another label.
4. A picker can show the OK button on a single-select list too.
5. A picker can show a Custom button in the header with its own label and its own hover text.
6. A picker can hide the check-all box or the count badge.
7. In stock a picker can draw the Custom button in a secondary style, which Cursor does not support. [stock only]

### QA-6.4
1. An input prompt shows a text field with the prompt's placeholder and no list under it.
2. The field can open with preset text, with a chosen part of that text already selected.
3. A line under the field reads "{prompt} (Press 'Enter' to confirm or 'Escape' to cancel)".
4. When the prompt has no prompt text, the line under the field reads "Press 'Enter' to confirm your input or 'Escape' to cancel".
5. Pressing Enter confirms the typed text and closes the box, and pressing Escape cancels it.

### QA-6.5
1. An input prompt can switch the field to password mode, where typed characters show as dots.
2. In password mode the placeholder, the prompt line and the validation message work the same as in a normal prompt.
3. Password mode applies only to that prompt, so the next prompt shown in the same box has plain text again.

### QA-6.6
1. A prompt can check the typed text and show a message under the field with an info, warning or error look, which is colored text and background with a border and a matching border on the field.
2. The check runs about 100 ms after you stop typing and runs again when you press Enter.
3. An error blocks Enter and keeps the box open, while a warning or an info message does not block it, and a check result given as plain text counts as an error.
4. The message text can hold clickable links, including command links, and inline icons written as `$(name)`.
5. When the check finds nothing wrong the message and the field border go away.

### QA-6.7
1. A title bar appears above the field when the picker has a title, a step number or title buttons, and it shows the title text.
2. In a multi-step flow the title is followed by the step as "(2/3)", or by just "(2)" when the total is unknown, and with no title the bar shows only the step.
3. An optional description line appears under the title area.
4. With no title, no step and no title button, the title bar is hidden.

### QA-6.8
1. A multi-step picker shows a Back arrow at the left of the title bar.
2. Hovering the Back arrow shows the tooltip "Back", with the bound key added in brackets when there is one.
3. Clicking the Back arrow returns to the previous step.
4. Pressing the key bound to `workbench.action.quickInputBack` while the box is open returns to the previous step, with Alt+Left on Windows, Control+- on Mac and Ctrl+Alt+- on Linux.

### QA-6.9
1. A picker can add icon buttons at the right side of the title bar, as plain buttons or as toggles, for example the Reset and Close buttons of Customize Layout.
2. Title bar buttons marked secondary are tucked into an overflow menu with a label instead of showing as icons.
3. A picker can add inline icon buttons beside the right end of the filter field, for example "Open in Search View" in Quick Search.
4. A picker can add on and off toggle buttons inside the far end of the filter field, for example the zero-based toggle in the stock Go to Offset list.
5. When toggle buttons sit inside the filter field the selected-count badge moves left to make room for them.

### QA-6.10
1. A row can show one or more icon buttons at its right end, for example "Close Editor", "Rename Terminal", "Configure Task" and "Remove from Recently Opened".
2. By default the row buttons appear when you hover the row or move keyboard focus to it, and a button marked "always visible" stays on screen, such as the unsaved dot in the editor lists.
3. Clicking a row button fires that row's action without picking the row.
4. After a row button is clicked the picker decides what happens next, which is keeping the list as it is, dropping the row, refreshing the list or closing the box.
5. A row button can be a toggle that shows an on or off state.
6. Row buttons marked secondary sit in an overflow menu instead of showing as icons.

### QA-6.11
1. Pickers that use separators show heading rows with a label.
2. A heading can also show a description and a tooltip.
3. A thin line marks where each group of rows starts.
4. A heading with buttons becomes its own row that can be focused and shows its buttons, while a plain heading is drawn above its first row.
5. A heading whose rows are all filtered out disappears.

### QA-6.12
1. Cursor's list can show a section header row that cannot be clicked, drawn dimmed at 25 px tall. [cursor only]
2. A section header can be only a thin 5 px divider, with an optional line above it. [cursor only]
3. Rows and section headers can supply their own custom content, and the list measures its height off screen before drawing it. [cursor only]

### QA-6.13
1. A row can show an icon before the label, which is a file type icon, a theme icon written `$(name)` in the label, an image or an icon class.
2. The label shows the matching letters highlighted, with a dimmer description to its right.
3. A row can have a second line of detail text, and rows with detail are taller.
4. A shortcut label at the far right of the row shows the key bound to the command when the picker supplies one.
5. A row can be drawn in italic, as for preview editors, or struck through.
6. A long description or detail gets its own small hover with the full text.
7. Cursor leaves a 3 px gap between rows in the list. [cursor only]

### QA-6.14
1. Pausing the pointer over a row shows a hover to the right of the row with that row's tooltip.
2. Pressing Ctrl+Space, or Control+Space on Mac, while a row is active shows the same hover for that row (`quickInput.toggleHover`).
3. Pressing Ctrl+Space again hides the hover, and while it is on it follows the active row.
4. Rows with no tooltip show no row hover.

### QA-6.15
1. While a picker is busy, a thin progress bar runs under the filter field.
2. The bar appears only after the picker has been busy for 800 ms, so quick lists never show it.
3. The bar goes away as soon as the picker stops being busy.
4. The progress bar is hidden from screen readers.

### QA-6.16
1. Pressing Right arrow, or Alt, Ctrl or Cmd with Right arrow, picks the active row without closing the box, for pickers that allow it such as the editor, terminal, debug console and Go to File lists (`quickInput.acceptInBackground`).
2. That key works when focus is in the list or the caret is at the end of the filter field, and with the caret anywhere else in the field Right arrow just moves the caret.
3. Clicking elsewhere closes the box unless the picker asks to stay open.
4. Setting `workbench.quickOpen.closeOnFocusLost` chooses whether the box closes when it loses focus, with values true, the default, and false.

### QA-6.17
1. The command `workbench.action.alignQuickInputTop` snaps the box to the top of the window under the title bar, which is the default place.
2. The command `workbench.action.alignQuickInputCenter` snaps the box to the vertical center of the window.
3. Customize Layout shows both commands under a "Quick Input Position" heading with a mark on the current one.
4. Neither command is in the Command Palette, so you reach them only from the Customize Layout list.
5. The Reset button of Customize Layout puts the box back at the top.

### QA-6.18
1. Dragging the title bar, or the empty part of the header, moves the box anywhere inside the window.
2. While dragging, the box snaps to the top, the vertical center and the horizontal center when it comes within 20 px of them.
3. When the app draws the title bar itself, the box stays clear of the window controls.
4. The place is saved between sessions as a fraction of the window, so it adapts when the window is resized.
5. Double-clicking the title bar or the header puts the box back at the top.
6. In stock a double-click on a title bar button does not reset the box and the reset also sets the top mark in Customize Layout, which Cursor's older code does not do. [stock only]

### QA-6.19
1. Setting `workbench.quickOpen.width` sets the width cap of the box for Quick Open and every other picker, with a default of 820 and allowed values from 580 to 1000. [cursor only]
2. The box re-lays out as soon as the value changes. [cursor only]
3. The width is still limited to 62 percent of the window, so on a narrow window the box is narrower than the setting. [cursor only]
4. When the setting has no value, the box uses the built-in default width. [cursor only]
5. Stock has no width setting and draws the box at 62 percent of the window up to a fixed 600 px. [stock only]

### QA-6.20
1. When the active row holds Cursor Tab report data, a floating box opens to the right of the list, 420 px wide and 320 px tall, with a rounded border and a shadow. [cursor only]
2. The box shows a heading line with the time, then the details recorded for the row. [cursor only]
3. If the box would run off the right edge of the window, it opens under the row instead, as wide as the row. [cursor only]
4. The box closes when the active row changes to one without data and when the picker closes. [cursor only]
5. For rows that supply their own preview, hovering the row opens the box too. [cursor only]
6. The command `workbench.action.reportCppAction`, listed in the Command Palette as "Developer: Report Cursor Tab Action", opens the picker whose rows show this box. [cursor only]

### QA-6.21
1. Down and Up move the active row by one, and so do Ctrl+Down and Ctrl+Up (`quickInput.next` and `quickInput.previous`).
2. PageDown and PageUp move the active row by a page (`quickInput.pageNext` and `quickInput.pagePrevious`).
3. Ctrl+Home and Ctrl+End, or Control+Home and Control+End on Mac, go to the first and last row (`quickInput.first` and `quickInput.last`).
4. Alt+Down and Alt+Up jump to the next or previous heading, with Cmd+Down and Cmd+Up on Mac (`quickInput.nextSeparatorWithQuickAccessFallback` and `quickInput.previousSeparatorWithQuickAccessFallback`).
5. Ctrl+Alt+Down and Ctrl+Alt+Up always jump to the next or previous heading, with Cmd+Alt+Down and Cmd+Alt+Up on Mac (`quickInput.nextSeparator` and `quickInput.previousSeparator`).
6. In a list opened by a held key, Alt+Up and Alt+Down only step one row instead of jumping to a heading.
7. The page and first and last moves also work with Alt, Ctrl or Cmd added, so a held shortcut such as Ctrl+P can keep stepping through the list.

### QA-6.22
1. Pressing Enter picks the active row, and holding Alt, Ctrl, Cmd or Shift with Enter passes that modifier to the picker, for example to open a file to the side.
2. Enter is ignored while an input method is composing text.
3. Pressing Escape, Shift+Escape, or Alt, Ctrl or Cmd with Escape cancels the box and closes it.
4. The command `workbench.action.acceptSelectedQuickOpenItem` has no default key and, when bound, picks the active row.
5. The command `workbench.action.alternativeAcceptSelectedQuickOpenItem` has no default key and, when bound, picks the active row as if Ctrl were held with Enter.
6. The command `workbench.action.focusQuickOpen` has no default key and, when bound, moves keyboard focus into the box.
7. In Cursor, pressing Tab in the box cycles focus through the row buttons, the filter field, the list, the message links and a custom widget instead of leaving the box. [cursor only]
8. In stock, Enter and Escape run the commands `quickInput.accept` and `quickInput.hide`, which Cursor does not have because its box handles those keys in its own key handler. [stock only]

### QA-6.23
1. The list is exposed to a screen reader as a listbox named "Quick Input" whose rows are options, or checkboxes in a multi-select picker.
2. Each row reads its label, its description and its detail, plus the heading it sits under.
3. The filter field is named from the picker's placeholder and title, or "Type to narrow down results." when it has neither.
4. The result count "{0} Results" and the selected count "{0} Selected" are announced politely as they change.
5. The active row is tied to the filter field with `aria-activedescendant`, so moving the highlight reads the new row.
6. In screen reader mode the list refreshes the focused row after each filter so it is read again.
7. Stock's list focus mode on the filter field, which sets `aria-haspopup` to listbox, does not exist in Cursor. [stock only]

### QA-6.24
1. Rows in the Run Task list and the terminal Run Recent Command list show a pin button on hover with the tooltip "Pin command".
2. Once a row is pinned, its pin button tooltip reads "Pinned command".
3. Pinned rows move into a "pinned" section at the top of the list while the filter field is empty.
4. Pressing the pin button again unpins the row.
5. The pinned choices are saved per workspace.

### QA-6.25
1. Newer than 1.128: in VS Code 1.144.0 the box fades in while growing from 97 percent of its size over 250 ms. [stock only]
2. In VS Code 1.144.0 the box fades out over 150 ms when it closes, and it ignores the pointer while closing. [stock only]
3. The open and close animation plays only when the window uses the modern UI style and motion is enabled. [stock only]

### QA-6.26
1. Running "Developer: Report AI Action" (`workbench.action.reportAIAction`) from the Command Palette opens a quick pick reading "Select a recent AI action". [cursor only]
2. The list holds up to 30 recent actions, each row showing its type, a shortened description and its id, with the time as the description. [cursor only]
3. Picking a row opens a second pick "Choose an action (ai)" with the rows "Report Good", "Report Bad" and "Report with Comment". [cursor only]
4. Picking "Report with Comment" opens an input box with the prompt "Type your comment here...". [cursor only]
5. The "Copy Request ID" row is greyed with "Disabled in privacy mode" when privacy mode is on. [cursor only]
6. The "Open in Prompt Quality" row appears only for developer users. [cursor only]

## Unclear

- QA-6.12: no picker in the main bundle was found that creates a section header or a custom drawn row, so the three pieces cannot be checked on screen until a picker that uses them is found.
- QA-6.20 piece 5: the `customPreview` row field is read by the box but never set in the main bundle, so no picker is known to trigger it. Treat that piece as unconfirmed.
- QA-6.2 piece 8, QA-6.22 piece 8 and QA-6.23 piece 7 name stock features Cursor lacks, but whether a person sees any difference was not checked, as the feature file notes.
- QA-6.22 piece 3 and QA-5.15 both cover Escape closing the box, as the feature files note. The parent should keep one copy.
- QA-6.16 piece 4 repeats the `workbench.quickOpen.closeOnFocusLost` setting that QA-2.19 also covers, on purpose, because here it describes the widget as a whole.
- QA-6.18 piece 6 rests on pass 2 reading Cursor's reset code as older than stock, and nothing was run on screen.
