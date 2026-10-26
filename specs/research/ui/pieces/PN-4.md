Group: PN-4 Problems. Pass: 3. Date: 2026-10-25.

### PN-4.1
1. Choosing Problems from the View menu opens the panel on the Problems tab and moves focus into the problem list.
2. Pressing `Ctrl+Shift+M` (`workbench.actions.view.problems`, titled Toggle Problems) opens the Problems tab and focuses it, and pressing it again while the Problems view has focus hides the panel.
3. Toggle Problems (Errors, Warnings, Infos) in the Command Palette (`workbench.actions.view.toggleProblems`) shows the Problems view when it is hidden and closes it when it is already visible.
4. Focus Problems (Errors, Warnings, Infos) in the Command Palette (`workbench.action.problems.focus`) opens the Problems tab and moves focus into it without ever closing it.
5. Clicking the Problems item in the status bar shows the Problems view when it is hidden and closes it when it is visible.
6. When the Problems view has been removed from its container, no Problems tab shows in the panel.

### PN-4.2
1. In tree mode, which is the default look, every file that has problems is one row and its problems sit nested under it.
2. Each file row shows a count badge with how many problems that file has.
3. A problem row shows the severity icon, the message with all of its lines, the source and code beside the message, and `[Ln, Col]` at the end.
4. A code that has a link target shows as a clickable link that opens its target.
5. File rows expand and collapse to show or hide their problems.
6. The Collapse All button in the Problems header (`workbench.actions.treeView.workbench.panel.markers.view.collapseAll`) collapses every file row, and it is only shown in tree mode.
7. Hovering over the message of a problem row shows its full message in a hover.

### PN-4.3
1. In table mode every problem is its own row with no grouping by file.
2. The table has five columns, in this order: an icon column, Code, Message, File and Source.
3. The icon column shows the severity icon of each problem.
4. The Code column shows the source name and the code, and the code is a link when the diagnostic gives a link target.
5. The File column shows the workspace-relative path of the file followed by `[Ln, Col]`.
6. Text typed in the filter box is highlighted in the matching Message, File, Code and Source cells.

### PN-4.4
1. In tree mode the Problems header has a button with the View as Table label that switches the list to the flat table.
2. In table mode the same spot has a button with the View as Tree label that switches the list back to the tree.
3. The mode a person picks is saved for the workspace and is used again the next time the workspace opens.
4. Setting `problems.defaultViewMode` chooses the mode used before any choice has been saved, with values `tree` (default) and `table`.
5. Switching between tree and table keeps the problems that were selected still selected in the new view.

### PN-4.5
1. Typing plain text in the filter box in the Problems header hides every problem whose message, file name, source and code do not match it.
2. Comma separated glob patterns such as `**/*.ts` show only problems from files that match.
3. A pattern with a leading `!` hides the files that match it, for example `!**/node_modules/**`.
4. A pattern that starts with a dot, such as `.js`, is treated as a file extension pattern and shows only problems from files with that extension.
5. Past filter texts are kept in a history that can be recalled from the filter box.
6. Pressing `Ctrl+F` while the Problems list has focus moves focus into the filter box (`problems.action.focusFilter`), and pressing `Ctrl+Down` in the filter box moves focus back into the list (`problems.action.focusProblemsFromFilter`).
7. Pressing `Escape` in the filter box clears the filter text (`problems.action.clearFilterText`).
8. Typing a letter while the list has focus moves focus to the filter box.
9. The filter text is saved for the workspace and is still there when the workspace is reopened.
10. In Cursor the filter box placeholder reads "Filter (e.g. text, **/*.ts, !**/node_modules/**)" with no `@source:ts` tip. [cursor only]

### PN-4.6
1. Typing `@source:ts` in the filter box keeps only problems reported by the source `ts`, and the match ignores upper and lower case. [stock only]
2. Using more than one `@source:` term shows a problem when its source matches any one of them. [stock only]
3. Putting `!` in front, as in `!@source:eslint`, hides problems from that source. [stock only]
4. A source name with spaces is wrapped in quotes, as in `@source:"my source"`. [stock only]
5. The `@source:` term is taken out of the filter text, so any other text or glob in the box still filters as usual alongside it. [stock only]
6. The stock filter box placeholder ends with `@source:ts` as a tip. [stock only]

### PN-4.7
1. A More Filters button with a filter icon sits next to the filter box and opens a menu with Show Errors, Show Warnings and Show Infos.
2. Each of the three entries shows a check when it is on, and unticking one hides the problems of that severity.
3. All three are on by default.
4. The More Filters button is highlighted whenever any filter is active.
5. The tick choices are saved for the workspace.

### PN-4.8
1. The More Filters menu has a Show Active File Only entry, in a group below the three severity entries.
2. Ticking Show Active File Only limits the list to problems in the file of the active editor.
3. While Show Active File Only is on, the list changes to the new file each time the active editor changes.
4. The More Filters menu has a Show Excluded Files entry that is ticked by default, which shows problems from every file including those that match `files.exclude`.
5. Unticking Show Excluded Files hides problems in files that match the `files.exclude` patterns of their workspace folder.
6. Both choices are saved for the workspace.

### PN-4.9
1. When filters hide some problems, the filter box shows a badge such as "Showing 3 of 12" with the visible count and the total.
2. The badge is not shown when no problems are hidden or when the workspace has no problems.
3. When there are no problems at all, the body of the view shows "No problems have been detected in the workspace." instead of an empty list.
4. With Show Active File Only on and no problems in the current file, the body shows "No problems have been detected in the current file."
5. When filters hide every problem, the body shows "No results found with provided filter criteria." followed by a Clear Filters link.
6. Clicking Clear Filters, or pressing `Enter` or `Space` on it, empties the filter text and turns Show Errors, Show Warnings and Show Infos and Show Excluded Files back on.
7. A screen reader hears "Showing N problems" or "Showing N of M problems" when rows are listed, and hears the same text as the body message when the list is empty.

### PN-4.10
1. Clicking a problem row opens its file in the editor with the cursor on the problem, and focus stays in the Problems list.
2. Moving the selection with the arrow keys opens each problem the same way and highlights the problem's range in the editor while focus stays in the list.
3. Pressing `Enter` (`problems.action.open`) or double-clicking a problem row opens its file with the cursor on the problem, pins the tab and moves focus into the editor.
4. On macOS, `Cmd+Down` on a focused problem row does the same as `Enter`.
5. Pressing `Enter` or double-clicking a file row does not jump to the editor, only problem rows do.

### PN-4.11
1. Pressing `Ctrl+Enter` on a focused problem row (`problems.action.openToSide`) opens its file in the side editor group with the cursor on the problem.
2. On macOS the key for opening to the side is `Ctrl+Enter`, not `Cmd+Enter`.
3. Clicking a problem row with `Ctrl`, `Cmd` or `Alt` held opens its file in the side editor group.

### PN-4.12
1. Setting `problems.autoReveal` turns on automatic reveal of the active file's problems in the list, as a true or false value that is true by default.
2. With it on, switching the active editor scrolls the Problems list to the problems of the new file and reveals them.
3. With it on, the list also reveals the current file's problems when that file gets its first problem.
4. Automatic reveal does not happen while Show Active File Only is on.

### PN-4.13
1. A selected or focused problem row, in tree or table mode, shows a light bulb when a language provider offers a quick fix for it, and a row with no fix shows none.
2. Clicking the light bulb (tooltip "Show fixes") opens a menu of the quick fixes for that problem, quick-fix kind only.
3. Pressing `Ctrl+.` on a focused problem row opens the same list of fixes (`problems.action.showQuickFixes`).
4. Right-clicking a problem row puts the same fixes at the top of its context menu.
5. A row whose fix can be applied automatically shows a stronger light bulb icon.
6. The light bulb and the fixes only appear once the file's text has loaded, so there can be a short delay after the row is selected.
7. Picking a fix opens the file at the problem without taking focus and then applies the fix.

### PN-4.14
1. Right-clicking a problem row, in tree or table mode, opens a context menu with the quick fixes listed first.
2. Under the fixes the menu has Copy and Copy Message.
3. Menu entries that have a keyboard shortcut show it next to their name, for example `Ctrl+C` next to Copy.
4. Right-clicking a related-information row shows a menu with only Copy Message.
5. In Cursor the menu also has Add to Chat and Fix with Agent in a separate group after the stock entries, when a problem row is focused. [cursor only]

### PN-4.15
1. Choosing Copy from the context menu with one or more problem rows selected puts those problems on the clipboard as JSON text wrapped in square brackets.
2. Each copied problem holds its severity, message, range, source, code, file path and any related information.
3. Copying with a file row selected includes every problem under that file.
4. Pressing `Ctrl+C` while the Problems list has focus does the same as Copy (`problems.action.copy`), unless a related-information row is focused.
5. Copy Message (`problems.action.copyMessage`) puts only the message text of the selected problems on the clipboard, one line per problem.
6. Copy Message on a related-information row copies that related message (`problems.action.copyRelatedInformationMessage`).

### PN-4.16
1. A problem whose message has more than one line shows a small chevron at the end of the first line of its row, and a one-line message has no active chevron.
2. By default every line of a long message is shown, with the chevron pointing up and the tooltip "Show message in single line".
3. Clicking the chevron cuts that row down to its first line, points the chevron down and changes the tooltip to "Show message in multiple lines", and clicking again shows all lines.
4. The Command Palette entries "Show message in multiple lines" and "Show message in single line" (Problems category) change every row at once, and they are listed only while the Problems view is visible.
5. The choice is saved for the workspace, and showing multiple lines is the default.
6. A message line longer than 1000 characters is cut with an ellipsis.

### PN-4.17
1. A problem whose diagnostic points to other locations shows an expand arrow on its row.
2. Expanding the row shows one child row per location, each with the file name, the message and `[Ln, Col]`.
3. Clicking a child row, or pressing `Enter` on it, opens that location in the editor.
4. Clicking the problem row itself does not expand it, only the arrow does.
5. A screen reader hears the problem's label followed by "This problem has references to N locations."

### PN-4.18
1. A number badge sits on the Problems tab in the panel with the total of errors, warnings and infos, and its tooltip reads "Total N Problems".
2. The badge disappears when the total is zero.
3. The number follows the problems as they are added and removed, and it does not change when filters hide rows in the list.
4. When the Problems view is moved into the side bar or the auxiliary bar, the same badge shows on that container's icon instead.
5. Setting `workbench.panel.showLabels` chooses whether the Problems tab and the other panel tabs show a text label or only an icon, with values `true` (label, default) and `false` (icon).

### PN-4.19
1. Setting `problems.visibility`, on by default, switches problem markers on or off across the workbench, and turning it off removes the squiggles in editors.
2. Turning `problems.visibility` off also removes the error and warning marks on files and folders in the Explorer and on elements in the Outline.
3. While the setting is off, a warning item with the whole-word icon appears in the status bar next to the Problems counts, with the tooltip "Problems are turned off. Click to open settings."
4. Clicking that status bar item opens the Settings editor filtered to `@id:problems.visibility`.
5. The item disappears as soon as the setting is turned back on, without a reload.
6. Turning the setting off does not hide the Problems list or the Problems counts in the status bar.

### PN-4.20
1. With focus in the Problems filter box, Open Accessibility Help (`editor.action.accessibilityHelp`, `Alt+F1`, and on Linux `Alt+Shift+F1` with `Alt+F1` as the second key) opens a page titled "Accessibility Help: Problems Panel Filter". [stock only]
2. The page explains that the filter is not a navigating search, that it hides problems that do not match, and that focus stays in the box while the list updates. [stock only]
3. The page describes the filter syntax, where plain text matches the message, file path or code and `!text` hides matches. [stock only]
4. The page names the Errors, Warnings, Info and Active File Only toggles and says they work together with the text filter. [stock only]
5. The page lists the keyboard moves: Down Arrow into the list, Tab to the toggles, Enter on a problem, the next and previous problem keys, and Escape to clear the filter. [stock only]
6. The page lists the settings `accessibility.verbosity.find`, `problems.autoReveal`, `problems.defaultViewMode`, `problems.sortOrder` and `problems.showCurrentInStatus`, with the key that opens Settings. [stock only]
7. Setting `accessibility.verbosity.find` controls whether the filter box announces the hint that Accessibility Help is available. [stock only]

## Unclear

- PN-4.1: the command `workbench.action.showErrorsWarnings` opens the view but has no title and is not in the Command Palette, so I did not give it its own piece.
- PN-4.14: the two Cursor chat entries are named in one piece only because the feature is flagged boundary, and what they do is chat so it is not described.

