Group: WB-3 Focus and navigation between parts. Pass: 3. Date: 2026-10-25.

### WB-3.1
1. In the Problems view and the Comments view, Ctrl+Down in the filter box moves focus to the list, and Ctrl+Up from the list moves it back to the filter box.
2. In the Testing view, Ctrl+Down moves focus into the test tree and Ctrl+Up from the tree moves it to the filter box.
3. In the Settings editor, Ctrl+Down in the search box moves focus to the table of contents on the left, and Ctrl+Up moves it back to the search box.
4. In the Keyboard Shortcuts editor, Ctrl+Down in the search box moves focus to the shortcuts table, and Ctrl+Up moves it back to the search box.
5. In the Debug Console, Ctrl+Down moves focus from the filter box to the output tree and from the tree to the input, and Ctrl+Up from the input moves focus to the tree.
6. In an open comment thread, Ctrl+Down opens the reply box and focuses it, and Ctrl+Up from the reply box moves focus to the comments above it.
7. When a list in these views has focus, the keys only jump between widgets once the list is scrolled to the bottom (Ctrl+Down) or the top (Ctrl+Up); before that they keep scrolling the list.

### WB-3.2
1. Running "Remove keyboard focus from focused element" makes the focused element lose focus.
2. Nothing happens when no element has focus or the focused item is not an HTML element.
3. The command has no default key and no Command Palette entry, so a person reaches it only by assigning a shortcut to it in the Keyboard Shortcuts editor, where it is listed by its title.

### WB-3.3
1. "Navigate to the View on the Left", "Navigate to the View on the Right", "Navigate to the View Above" and "Navigate to the View Below" move keyboard focus to the part that lies in that direction from the focused part.
2. From the editor area, the command first moves to the editor group in that direction, and only leaves the editor area when no group lies that way.
3. Coming back into the editor area from the side bar or panel focuses the active group when no group lies in between, otherwise the first group when moving right and the last group for the other directions.
4. Landing on the side bar, panel or secondary side bar focuses the view container that is open there, and a hidden part is never a target.
5. The commands are in the Command Palette under View and have no default keys and no menu rows.

### WB-3.4
1. "List: Resize Column" is offered in the palette only while a list has focus, and it does nothing unless the focused list is a table.
2. It opens a pick list with the placeholder "Select the column to resize, type to filter." that holds the table's column headings.
3. After you pick a column an input box asks "Please enter a width in percentage for the '<column>' column." with the placeholder "i.e. 20, 60, 100...".
4. Text that is not a whole number shows "Please enter an integer.", and a number outside 1 to 100 shows "Please enter a number greater than 0 and less than or equal to 100."
5. The column takes the new width at once, and cancelling either box or entering nothing leaves the table as it was.
6. It works on every table built with the workbench table widget, such as the Problems table, the Keyboard Shortcuts editor and the Profiles editor, not only one of them.

### WB-3.5
1. With focus on a part of the workbench that has a hover, pressing `Ctrl+K Ctrl+I` shows that hover and puts keyboard focus inside it.
2. The command looks at the focused element and then at each parent until it finds one with a hover, and does nothing visible when none has one.
3. While a hover is open, holding `Alt` locks it so it stays when the pointer leaves, releasing `Alt` closes it if the pointer is outside, and any other key that is not bound hides it and returns focus to the element.
4. Setting `workbench.hover.delay` chooses how many milliseconds the pointer rests before a workbench hover shows (500, or 1500 on macOS).
5. Setting `workbench.hover.reducedDelay` (default 500) is the shorter delay used where faster feedback helps. [stock only]
6. In Cursor the workbench rule is switched off while a text editor has focus, so the key there belongs to the editor's own Show or Focus Hover (ED-4). [cursor only]

## Unclear
none
