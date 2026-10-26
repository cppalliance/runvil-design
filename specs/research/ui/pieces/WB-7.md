Group: WB-7 Editor context menus and gutter menus. Pass: 3. Date: 2026-10-25.

### WB-7.1
1. Right-clicking an action button in a toolbar built from a menu (the editor title bar, view title bars, Source Control repository rows, the notebook toolbar) opens a menu with Configure Keybinding when the action can have a key, then Hide '<action>' and a checklist of every hideable action in that toolbar.
2. Choosing Hide, or unchecking an entry in the checklist, removes the button from the toolbar and moves it into the "..." overflow menu, and checking it again puts it back.
3. When anything in that toolbar is hidden, the menu also ends with Reset Menu, which shows everything again for that toolbar, and the same row is in the menu opened from the "..." button.
4. An action that cannot be hidden shows a greyed "Hide" label in place of Hide '<action>'.
5. "View: Reset All Menus" brings back every hidden action in every toolbar at once and writes "did RESET all menu hidden states" to the log.
6. Toolbars set not to allow hiding, such as the title bar, the Command Center and the side bar and panel tab strips, show no hide rows.

### WB-7.2
1. Right-clicking text in the editor focuses the editor, moves the cursor to the click point unless the click is inside an existing selection, and opens the context menu at the mouse position.
2. The menu lists the rows that features add to `EditorContext`, grouped with separators, shows each row's key binding on the right, and leaves out a submenu that has no visible rows.
3. `Shift+F10` (Show Editor Context Menu) or the Menu key opens the same menu just below the cursor, after scrolling the cursor into view.
4. Scrolling the mouse wheel while the menu is open closes it.
5. Right-clicking the find box or another widget over the editor, or injected text such as an inlay hint, shows the native text menu instead.
6. Right-clicking the scrollbar or minimap opens the minimap menu from ED-5.2 instead of this one.
7. With `editor.contextmenu` set to false, a right-click only moves the cursor and gives no menu, and `Shift+F10` does nothing.
8. Small editors such as the comment box use the `SimpleEditorContext` menu, which holds edit rows such as Cut, Copy, Paste, Undo and Select All instead of the full list.

## Unclear
none
