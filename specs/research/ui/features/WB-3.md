Group: WB-3 Focus and navigation between parts. Section: Workbench-wide. Date: 2026-10-25. Pass: 4.
Features: 5.

### WB-3.1 Move between a view's filter box and its list with Ctrl+Up and Ctrl+Down
- Tag: both
- Evidence: source
- Boundary: no
- What: In views that pair a filter or search box with a list, Ctrl+Down (Cmd+Down on macOS) moves keyboard focus from the box into the list and Ctrl+Up moves it back.
- Commands: widgetNavigation.focusNext; widgetNavigation.focusPrevious
- Menus: none found
- Keys: Ctrl+Down (widgetNavigation.focusNext, when a navigable container has focus and either no list is focused or the list is scrolled to the bottom); Ctrl+Up (widgetNavigation.focusPrevious, same container condition and the list is scrolled to the top); both use Cmd on macOS
- Settings: none found (the code reads workbench.navigibleContainer.enableDebug, but no configuration registers it)
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\widgetNavigationCommands.ts:144-175; V\src\vs\workbench\contrib\markers\browser\markersView.ts:204; V\src\vs\workbench\contrib\comments\browser\commentsView.ts:211; V\src\vs\workbench\contrib\comments\browser\commentThreadWidget.ts:101; V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts:273; V\src\vs\workbench\contrib\debug\browser\repl.ts:629; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts:480; V\src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts:155; C bundle (both command ids with the same keys and when clause; registrations named commentThreadWidget, keybindingsEditor, settingsEditor2, repl, testingExplorerView, markersView, commentsView and extensionsView)

### WB-3.2 Remove keyboard focus from the focused element
- Tag: both
- Evidence: source
- Boundary: no
- What: A command that drops keyboard focus from whatever element has it, so typing no longer goes to that element; it is not in the Command Palette and is meant to be bound to a key.
- Commands: workbench.action.blur
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\windowActions.ts:442-460; C bundle (class with id workbench.action.blur, one hit)

### WB-3.3 Navigate to the view on the left, right, above or below
- Tag: both
- Evidence: source
- Boundary: no
- What: Four commands move keyboard focus from the part that has it to the visible part on the chosen side, such as editor to side bar or panel to editor.
- Commands: workbench.action.navigateLeft; workbench.action.navigateRight; workbench.action.navigateUp; workbench.action.navigateDown
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\navigationActions.ts:23-240; C bundle (class holding the neighbor logic for workbench.parts.editor, panel, sidebar and auxiliarybar, and all four ids, 2 hits each). F6 and Shift+F6 (focusNextPart, focusPreviousPart) are a different behavior in the same file and are already named in SS-1.

### WB-3.4 Resize a table column from the Command Palette
- Tag: both
- Evidence: source
- Boundary: no
- What: With a table focused, run List: Resize Column to pick a column and type the width it should have as a percentage, which is the keyboard way to resize columns.
- Commands: list.resizeColumn
- Menus: CommandPalette (List category)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\list\browser\listResizeColumnAction.ts:13-34; V\src\vs\workbench\contrib\list\browser\tableColumnResizeQuickPick.ts; V\src\vs\workbench\contrib\markers\browser\markersTable.ts, V\src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts, V\src\vs\workbench\contrib\userDataProfile\browser\userDataProfilesEditor.ts, V\src\vs\workbench\contrib\workspace\browser\workspaceTrustEditor.ts (tables that use the workbench table widget); C bundle (id list.resizeColumn, nls 7533 and 7534)

### WB-3.5 Show or focus the hover of the focused element
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+K Ctrl+I with focus on a button, tab, status bar item or tree row to show its hover and move keyboard focus into it, without using the mouse.
- Commands: workbench.action.showHover
- Menus: none found
- Keys: Ctrl+K Ctrl+I (stock: any focus, weight EditorCore; Cursor: only when the editor does not have text focus, weight 199)
- Settings: workbench.hover.delay; workbench.hover.reducedDelay
- Cursor: changed - the command and `workbench.hover.delay` are present with a `when` of not editorTextFocus; `workbench.hover.reducedDelay` has 0 hits
- Source: V\src\vs\platform\hover\browser\hoverService.ts:110-123, 506-560; V\src\vs\workbench\browser\workbench.contribution.ts:733-746; C bundle (registerCommandAndKeybindingRule for workbench.action.showHover with Ctrl+K Ctrl+I and the negated editorTextFocus condition; the setting workbench.hover.delay)

## Handed off
- none

## Unclear
- Created in pass 4 from the coverage check.
