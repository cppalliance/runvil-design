Group: QA-5 Opening quick access and the help list. Pass: 3. Date: 2026-10-25.

### QA-5.1
1. Running `workbench.action.quickOpen` with a text argument, for example from a keybinding rule with `args`, opens the quick access box with that text already typed in the field.
2. The argument text is kept exactly as given, with the caret at its end and nothing selected, so the next key adds to it instead of replacing it.
3. When the argument starts with a provider prefix such as `>`, `@` or `%`, the box opens on that provider's list and uses the rest of the text as the filter.
4. Running the command with no argument opens the plain Go to File list, and the command is listed in the Command Palette as "Go to File...".

### QA-5.3
1. When Quick Open is opened from the command center and the box is empty, the list begins with shortcut rows in this order: Go to File, Command Palette, Search for Text, Go to Symbol in Editor, Start Debugging, Run Task and More.
2. Quick Open opened any other way, such as with the Go to File key, shows no shortcut rows and starts with the recent files only.
3. A "recently opened" heading and the recently opened files follow the shortcut rows.
4. Each shortcut row shows its label, its prefix as the description, and the key bound to its command at the right edge when there is one.
5. Picking a shortcut row switches the box to that list with its prefix typed in, for example the Command Palette row types `>` and the More row opens the help list.
6. Typing any text in the box hides the shortcut rows.
7. In stock an Open Quick Chat row with its quick chat key sits between Search for Text and Go to Symbol in Editor when quick chat is on, and picking it toggles quick chat, but Cursor has no such row. [stock only]

### QA-5.5
1. Typing `?` at the start of the quick access box shows the help list, which lists every quick access prefix with a short description.
2. While the help list is open the field placeholder reads "Type '?' to get help on the actions you can take from here."
3. Picking the More row in the command center Quick Open list opens the help list.
4. Running `workbench.action.quickOpen` with the argument `?` opens the help list directly.

### QA-5.6
1. The help list is sorted by prefix, so the plain Go to File entry, shown as an ellipsis, comes first.
2. Each row shows the prefix as its label and a short description, for example `>` with "Show and Run Commands", and the key bound to the provider's command at the right edge when there is one.
3. The `?` row itself is left out of the list.
4. A provider with several help entries gets one row per entry, for example `@` for Go to Symbol in Editor and `@:` for Go to Symbol in Editor by Category.
5. Each row is read to a screen reader as the prefix and the description joined by a comma.
6. Stock also lists rows for Go to Offset, MCP resources and agent sessions, which Cursor leaves out. [stock only]
7. Cursor adds two agent rows to the list, "Search chat history" and "Switch between recently used agents". [cursor only]

### QA-5.7
1. Pressing Enter on the active help row, or clicking a row, replaces the box text with that row's prefix and opens that list.
2. After the switch the prefix stays as typed with the caret right after it, ready for more typing.
3. Picking the ellipsis row for Go to File returns the box to the plain file list with no prefix.

### QA-5.8
1. Typing `?` followed by a provider's full prefix, for example `?term ` with the trailing space, switches straight to that list without picking a row.
2. The box text is replaced by just that prefix, so `?term ` becomes `term ` and the terminals list opens.
3. A partial prefix such as `?ter`, or a lone `?`, does not switch lists and the help list stays open.

### QA-5.9
1. Typing a provider prefix at the very start of the box, such as `@`, `:`, `#`, `%`, `view ` or `task `, switches to that provider's list and keeps the rest of the text.
2. When prefixes overlap, the longest matching prefix wins, for example `@:` is picked over `@`.
3. Deleting the prefix switches the box back to the plain Go to File list.
4. Only the text after the prefix filters the rows, so the prefix itself never hides a row.
5. The field placeholder changes to the new list's own hint text whenever the list switches.

### QA-5.10
1. Opening a different list while one is open moves your typed text under the new prefix, so typing `foo` in Go to File and then pressing F1 shows `>foo`.
2. Opening the same list again, for example pressing F1 while the Command Palette is open, keeps the box as it is and selects the typed text after the prefix so the next key replaces it.
3. Pressing F1 opens the Command Palette from any list, including Go to File, while Ctrl+Shift+P does so from every list except Go to File, where it moves up the list instead.

### QA-5.11
1. With Go to File open, pressing Ctrl+P or Ctrl+E again moves the highlight down, and Ctrl+Shift+P or Ctrl+Shift+E moves it up, with Mac using only Cmd+P and Cmd+Shift+P (`workbench.action.quickOpenNavigateNextInFilePicker` and its Previous pair).
2. With Open View open, Ctrl+Q moves down and Ctrl+Shift+Q moves up, with Mac using Control+Q and Linux having no default key (`workbench.action.quickOpenNavigateNextInViewPicker` and its Previous pair).
3. With the editor switcher open, Ctrl+Tab moves down and Ctrl+Shift+Tab moves up, with Mac using Control+Tab (`workbench.action.quickOpenNavigateNextInEditorPicker` and its Previous pair).
4. With Open Recent open, Ctrl+R moves down and Ctrl+Shift+R moves up, with Mac using Control+R (`workbench.action.quickOpenNavigateNextInRecentFilesPicker` and its Previous pair).
5. The terminal list has the commands `workbench.action.quickOpenNavigateNextInTerminalPicker` and `workbench.action.quickOpenNavigatePreviousInTerminalPicker` with no default key, so you bind your own.

### QA-5.12
1. The Command Palette lists the commands "Navigate Next in Quick Open" (`workbench.action.quickOpenNavigateNext`) and "Navigate Previous in Quick Open" (`workbench.action.quickOpenNavigatePrevious`).
2. With a key of your own bound to them, Next moves the highlight down the open list and Previous moves it up.
3. Neither command has a default key.
4. Pressing a key bound to either command with a modifier held makes the list ready to open its highlighted row when the modifier is released (see QA-5.13).

### QA-5.13
1. After moving through a list with a held modifier key (Ctrl, Alt, Shift or Cmd, whichever the shortcut uses), letting go of that key opens the highlighted row.
2. A list started by a cycle key such as Ctrl+Tab opens with the input line hidden.
3. If a list was already open when the cycle key was pressed, the input line stays visible.
4. The release only works once, so if no row is highlighted when the key comes up the list stays open and later key releases do nothing.
5. In a list with checkboxes, letting go of the held key opens nothing.

### QA-5.14
1. On Mac, Control+N moves the highlight down and Control+P moves it up in any open list.
2. On Windows and Linux the commands `workbench.action.quickOpenSelectNext` and `workbench.action.quickOpenSelectPrevious` have no default key and can be bound to your own keys.
3. The Command Palette lists the commands as "Select Next in Quick Open" and "Select Previous in Quick Open".
4. Moving with these commands only moves the highlight and never opens a row when a key is released.

### QA-5.15
1. Pressing Escape or Shift+Escape closes any open quick access list without picking a row.
2. The close key is the command `workbench.action.closeQuickOpen`, which acts only while a quick access box is open and can be rebound.

## Unclear

- QA-5.12 piece 4 and QA-5.13 describe the two sides of one behavior (arming and release). If the parent wants one copy, keep QA-5.13 and drop piece 4.
- QA-5.15 and QA-6.22 both cover Escape and `workbench.action.closeQuickOpen`, as the feature files already note. The parent should keep one copy of the Escape piece.
- QA-5.2 and QA-5.4 are also listed as TB-2.1 and TB-2.9, as the feature file notes. The parent should keep one copy of each.
