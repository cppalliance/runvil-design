Group: WB-7 Editor context menus and gutter menus. Section: Workbench-wide. Date: 2026-10-25. Pass: 4.
Features: 2.

### WB-7.1 Hide toolbar actions and reset all menus
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click an action button in a toolbar to hide it, and use Reset Menu or View: Reset All Menus to bring hidden actions back.
- Commands: menu.resetHiddenStates
- Menus: CommandPalette (View category); the right-click menu of menu driven toolbars (rows Hide, a checklist of hideable actions and Reset Menu)
- Keys: none found
- Settings: none found
- Cursor: present - the command (nls 1783), the Hide row and the Reset Menu row (nls 1780 to 1782) are in the bundle
- Source: V\src\vs\platform\actions\common\menuResetAction.ts:12-27; V\src\vs\platform\actions\browser\toolbar.ts:120-290; V\src\vs\workbench\browser\parts\editor\editorTabsControl.ts:265-298; V\src\vs\workbench\browser\parts\views\viewPane.ts:478; V\src\vs\workbench\contrib\scm\browser\scmRepositoryRenderer.ts:97; V\src\vs\workbench\contrib\notebook\browser\viewParts\notebookEditorToolbar.ts:360; C bundle (id menu.resetHiddenStates, class with resetThisMenu)

### WB-7.2 Show the editor right-click menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click in the text of the editor, or press Shift+F10, to open the context menu that lists the editor's commands.
- Commands: editor.action.showContextMenu; editor.contrib.contextmenu (controller id, not a command)
- Menus: EditorContext; SimpleEditorContext (small editors such as the comment box)
- Keys: Shift+F10 (editor.action.showContextMenu, when textInputFocus); the Menu key also opens it
- Settings: editor.contextmenu (an editor option with no description, default true)
- Cursor: present
- Source: V\src\vs\editor\contrib\contextmenu\browser\contextmenu.ts (ContextMenuController, ShowContextMenu); V\src\vs\workbench\contrib\comments\browser\simpleCommentEditor.ts (contextMenuId SimpleEditorContext); V\src\vs\editor\common\config\editorOptions.ts (contextmenu option); C bundle (class with ID editor.contrib.contextmenu, id editor.action.showContextMenu with label Show Editor Context Menu from nls 1046 and Shift+F10)

## Handed off
- none

## Unclear
- Created in pass 4 from the coverage check.
