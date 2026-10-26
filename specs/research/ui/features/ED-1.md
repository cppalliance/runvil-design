Group: ED-1 Editor groups, tabs and layout. Section: Editor. Date: 2026-10-25. Pass: 2.
Features: 50.

### ED-1.1 Choose how the tab strip and editor actions are shown
- Tag: both
- Evidence: source
- Boundary: no
- What: The person picks multiple tabs, one large tab, or no tab strip at all, moves the editor action buttons to the tab bar, the title bar, or hides them, and jumps to the matching settings with Configure Tabs or Configure Editors.
- Commands: workbench.action.showMultipleEditorTabs; workbench.action.showEditorTab; workbench.action.hideEditorTabs; workbench.action.editorActionsDefault; workbench.action.editorActionsTitleBar; workbench.action.hideEditorActions; workbench.action.showEditorActions; workbench.action.configureEditorTabs; workbench.action.configureEditor
- Menus: EditorTabsBarContext; EditorTabsBarShowTabsSubmenu; EditorActionsPositionSubmenu; MenubarAppearanceMenu; EditorTitle
- Keys: none found
- Settings: workbench.editor.showTabs; workbench.editor.editorActionsLocation; workbench.editor.alwaysShowEditorActions
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts; V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts; V\src\vs\workbench\browser\parts\editor\editorTitleControl.ts; V\src\vs\workbench\browser\parts\editor\singleEditorTabsControl.ts; V\src\vs\workbench\browser\parts\editor\noEditorTabsControl.ts; C bundle (all nine command ids and three settings found, showTabs default is multiple)

### ED-1.2 Size tabs and handle too many tabs
- Tag: both
- Evidence: source
- Boundary: no
- What: The person sets whether tabs fit their label, shrink or stay fixed width, how tall they are, and whether extra tabs wrap onto more rows or scroll with a scrollbar or the mouse wheel.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.editor.tabSizing; workbench.editor.tabSizingFixedMinWidth; workbench.editor.tabSizingFixedMaxWidth; window.density.editorTabHeight; workbench.editor.wrapTabs; workbench.editor.scrollToSwitchTabs; workbench.editor.titleScrollbarSizing; workbench.editor.titleScrollbarVisibility
- Cursor: changed (titleScrollbarVisibility not found, the other seven settings found)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts; V\src\vs\workbench\browser\parts\editor\multiRowEditorTabsControl.ts; C bundle (titleScrollbarSizing registered with default and large, no titleScrollbarVisibility string)

### ED-1.3 Read a tab: label, icon, hover and decorations
- Tag: both
- Evidence: source
- Boundary: no
- What: Each tab shows a file icon, a name that adds folder parts when two names clash, file badges and colors, and a hover with the full path that marks a preview tab and links to the preview setting.
- Commands: workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: workbench.editor.labelFormat; workbench.editor.untitled.labelFormat; workbench.editor.customLabels.enabled; workbench.editor.customLabels.patterns; workbench.editor.showIcons; workbench.editor.decorations.badges; workbench.editor.decorations.colors; workbench.editor.showTabIndex
- Cursor: changed (showTabIndex not found, the hover text with the Configure Preview Mode link and the other seven settings found)
- Source: V\src\vs\workbench\browser\parts\editor\editorTabsControl.ts; V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts; V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\services\editor\common\customEditorLabelService.ts; C bundle

### ED-1.4 Use a tab's close and unpin buttons and the unsaved marker
- Tag: both
- Evidence: source
- Boundary: no
- What: The person closes or unpins a tab with its button, sees a dot in place of the close icon when the editor has unsaved changes, and can turn on a colored top border for unsaved tabs.
- Commands: workbench.action.closeActiveEditor; workbench.action.unpinEditor
- Menus: EditorTitle
- Keys: none found
- Settings: workbench.editor.tabActionLocation; workbench.editor.tabActionCloseVisibility; workbench.editor.tabActionUnpinVisibility; workbench.editor.highlightModifiedTabs
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (close and unpin toolbar items used when tabs are hidden, lines 536 to 596); V\src\vs\workbench\browser\workbench.contribution.ts; C bundle

### ED-1.5 Open files in a preview tab
- Tag: both
- Evidence: source
- Boundary: no
- What: A file opened with a single click shows in italics in a preview tab that the next file replaces, until the person edits it, double-clicks it or keeps it open.
- Commands: workbench.action.toggleKeepEditors
- Menus: EditorTitle
- Keys: none found
- Settings: workbench.editor.enablePreview; workbench.editor.enablePreviewFromQuickOpen; workbench.editor.enablePreviewFromCodeNavigation
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorCommands.ts (toggleKeepEditors flips enablePreview); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (Enable Preview Editors toggle on EditorTitle); V\src\vs\workbench\browser\workbench.contribution.ts; C bundle

### ED-1.6 Keep a preview tab open
- Tag: both
- Evidence: source
- Boundary: no
- What: The person turns a preview tab into a normal tab with the Keep Open menu row, a keyboard chord, or a double-click on the tab.
- Commands: workbench.action.keepEditor
- Menus: EditorTitleContext; CommandPalette
- Keys: Ctrl+K Enter
- Settings: workbench.editor.enablePreview
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorCommands.ts; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (Keep Open row shown only when enablePreview is on); V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts (double-click pins a preview tab); C bundle

### ED-1.7 Pin and unpin a tab
- Tag: both
- Evidence: source
- Boundary: no
- What: The person pins a tab so it moves to the front of the strip, optionally shows it compact or on its own row, and keeps it from closing by keyboard or middle click.
- Commands: workbench.action.pinEditor; workbench.action.unpinEditor; workbench.action.toggleSeparatePinnedEditorTabs
- Menus: EditorTitleContext; CommandPalette
- Keys: Ctrl+K Shift+Enter (pins when the tab is not pinned, unpins when it is)
- Settings: workbench.editor.pinnedTabSizing; workbench.editor.pinnedTabsOnSeparateRow; workbench.editor.preventPinnedEditorClose
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorCommands.ts; V\src\vs\workbench\browser\actions\layoutActions.ts (Separate Pinned Editor Tabs); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle

### ED-1.8 Close one editor
- Tag: both
- Evidence: source
- Boundary: no
- What: The person closes the active tab from its button, a middle click, the File menu or the keyboard, force-closes a pinned tab, or reverts and closes in one step.
- Commands: workbench.action.closeActiveEditor; workbench.action.closeActivePinnedEditor; workbench.action.revertAndCloseActiveEditor
- Menus: EditorTitleContext; EditorTitle; CommandPalette; MenubarFileMenu
- Keys: Ctrl+W (on Windows Ctrl+F4, with Ctrl+W as a second key); middle click on a tab
- Settings: workbench.editor.focusRecentEditorAfterClose; workbench.editor.closeOnFileDelete
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorCommands.ts; V\src\vs\workbench\browser\parts\editor\editorActions.ts; V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts (middle click close); V\src\vs\workbench\browser\workbench.contribution.ts; C bundle

### ED-1.9 Close many editors or groups at once
- Tag: both
- Evidence: source
- Boundary: no
- What: The person closes the other tabs, the tabs to the right or left, the saved tabs, every tab in the group, every tab in every group, or the whole group, and empty groups can close by themselves.
- Commands: workbench.action.closeOtherEditors; workbench.action.closeEditorsToTheRight; workbench.action.closeEditorsToTheLeft; workbench.action.closeUnmodifiedEditors; workbench.action.closeEditorsInGroup; workbench.action.closeAllEditors; workbench.action.closeAllGroups; workbench.action.closeGroup; workbench.action.closeEditorsAndGroup; workbench.action.closeEditorsInOtherGroups; workbench.action.closeEditorInAllGroups
- Menus: EditorTitleContext; EditorTitle; CommandPalette; EmptyEditorGroup; EmptyEditorGroupContext
- Keys: Ctrl+K W (close all in group); Ctrl+K U (close saved); Ctrl+K Ctrl+W (close all editors); Ctrl+K Ctrl+Shift+W (close all groups); Ctrl+W on an empty group closes the group; Cmd+Alt+T on Mac for Close Others
- Settings: workbench.editor.closeEmptyGroups
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorCommands.ts; V\src\vs\workbench\browser\parts\editor\editorActions.ts; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (all eleven command ids found)

### ED-1.10 Reopen a closed editor
- Tag: both
- Evidence: source
- Boundary: no
- What: The person brings back the most recently closed tab (the action calls the history service reopenLastClosedEditor).
- Commands: workbench.action.reopenClosedEditor
- Menus: MenubarRecentMenu
- Keys: Ctrl+Shift+T
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (ReopenClosedEditorAction); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (File > Open Recent row, enabled by context key canReopenClosedEditor); C bundle

### ED-1.11 Select several tabs and act on them together
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl-click (Cmd-click on Mac) adds a tab to the selection and Shift-click selects a range, then close, move, split and window actions apply to the whole selection.
- Commands: none found
- Menus: EditorTitleContext
- Keys: Ctrl+click or Cmd+click; Shift+click (mouse gestures, no default key)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts (selection handling and selectEditorsBetween); V\src\vs\workbench\common\contextkeys.ts (multipleEditorsSelectedInGroup, twoEditorsSelectedInGroup); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (tab menu rows that check the selection); C bundle (both context keys found)

### ED-1.12 Move a tab along the strip and choose where new editors open
- Tag: both
- Evidence: source
- Boundary: no
- What: The person moves the active tab left, right, to the start or to the end of its group, and picks whether a newly opened editor lands left or right of the active tab or first or last, or jumps to a copy that is already open.
- Commands: workbench.action.moveEditorLeftInGroup; workbench.action.moveEditorRightInGroup; workbench.action.moveEditorToStart; workbench.action.moveEditorToEnd; moveActiveEditor
- Menus: CommandPalette
- Keys: Ctrl+Shift+PageUp and Ctrl+Shift+PageDown (on Mac the chords Cmd+K Cmd+Shift+Left and Cmd+K Cmd+Shift+Right)
- Settings: workbench.editor.openPositioning; workbench.editor.revealIfOpen
- Cursor: changed (moveEditorToStart and moveEditorToEnd not found, the other commands and both settings found)
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts; V\src\vs\workbench\browser\parts\editor\editorCommands.ts (moveActiveEditor); V\src\vs\workbench\browser\workbench.contribution.ts; C bundle

### ED-1.13 Split the editor into a new group
- Tag: both
- Evidence: source
- Boundary: no
- What: The person splits the active editor right, down, left or up into a new group, with a toolbar button that offers the other direction as its alternate and an orthogonal split that flips the direction.
- Commands: workbench.action.splitEditor; workbench.action.splitEditorOrthogonal; workbench.action.splitEditorUp; workbench.action.splitEditorDown; workbench.action.splitEditorLeft; workbench.action.splitEditorRight
- Menus: EditorTitle; EditorTitleContext; EditorTabsBarContext; EmptyEditorGroupContext; MenubarLayoutMenu
- Keys: Ctrl+\ (Split Editor); Ctrl+K Ctrl+\ (Split Editor Orthogonal; the source gives the same chord to Split Left, Right, Up and Down with no when clause)
- Settings: workbench.editor.openSideBySideDirection; workbench.editor.splitSizing
- Cursor: changed (tab menu lists Split Up, Down, Left and Right as flat rows, stock shows one Split Right or Down row plus a Split & Move submenu)
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (lines 60 to 185); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (menu rows and the toolbar item with alt); V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (nls strings 3456 to 3459 for the flat rows)

### ED-1.14 Move or split an editor into a neighboring group
- Tag: both
- Evidence: source
- Boundary: no
- What: The person sends the active editor, or a copy of it, to the previous, next, first, last, left, right, above or below group, or picks Move Above, Below, Left or Right from the Split & Move submenu.
- Commands: workbench.action.moveEditorToPreviousGroup; workbench.action.moveEditorToNextGroup; workbench.action.moveEditorToFirstGroup; workbench.action.moveEditorToLastGroup; workbench.action.moveEditorToLeftGroup; workbench.action.moveEditorToRightGroup; workbench.action.moveEditorToAboveGroup; workbench.action.moveEditorToBelowGroup; workbench.action.splitEditorToPreviousGroup; workbench.action.splitEditorToNextGroup; workbench.action.splitEditorToFirstGroup; workbench.action.splitEditorToLastGroup; workbench.action.splitEditorToLeftGroup; workbench.action.splitEditorToRightGroup; workbench.action.splitEditorToAboveGroup; workbench.action.splitEditorToBelowGroup
- Menus: EditorSplitMoveSubmenu; EditorTitleContext; CommandPalette
- Keys: Ctrl+Alt+Left (previous group); Ctrl+Alt+Right (next group); Shift+Alt+1 (first group); Shift+Alt+9 (last group); on Mac Cmd+Ctrl+Left, Right, 1 and 9
- Settings: none found
- Cursor: changed (all sixteen commands found, the Split & Move submenu with Move Above, Below, Left and Right is not in Cursor's tab menu)
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (lines 410 to 420); C bundle (EditorSplitMoveSubmenu string absent)

### ED-1.15 Drag tabs and files to move, copy or split
- Tag: both
- Evidence: source
- Boundary: no
- What: Dragging a tab or file over an editor shows a drop overlay where edges split the group and the middle moves it, Ctrl (Alt on Mac) copies, Alt (Shift on Mac) flips splitting, Shift drops a file into the text, and dragging out of the window opens a new window.
- Commands: none found
- Menus: none found
- Keys: Ctrl while dragging (Alt on Mac) to copy; Alt while dragging (Shift on Mac) to toggle splitting; Shift while dragging to drop into the editor; Alt while dragging out of the window to toggle new window
- Settings: workbench.editor.splitOnDragAndDrop; workbench.editor.dragToOpenWindow; editor.dropIntoEditor.enabled
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorDropTarget.ts; V\src\vs\workbench\browser\parts\editor\editorTabsControl.ts (dragToOpenWindow); V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (all three settings and the Hold Shift to drop into editor prompt string found)

### ED-1.16 Choose an editor layout preset
- Tag: both
- Evidence: source
- Boundary: no
- What: The person rearranges the editor area into Single, Two Columns, Three Columns, Two Rows, Three Rows, Grid (2x2), Two Rows Right or Two Columns Bottom, keeping the open editors.
- Commands: workbench.action.editorLayoutSingle; workbench.action.editorLayoutTwoColumns; workbench.action.editorLayoutThreeColumns; workbench.action.editorLayoutTwoRows; workbench.action.editorLayoutThreeRows; workbench.action.editorLayoutTwoByTwoGrid; workbench.action.editorLayoutTwoRowsRight; workbench.action.editorLayoutTwoColumnsBottom; layoutEditorGroups; vscode.setEditorLayout; vscode.getEditorLayout
- Menus: MenubarLayoutMenu; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts; V\src\vs\workbench\browser\parts\editor\editorCommands.ts (layoutEditorGroups and the vscode.setEditorLayout API); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (View > Editor Layout rows, lines 842 to 937); C bundle

### ED-1.17 Add an empty group or duplicate a group
- Tag: both
- Evidence: source
- Boundary: no
- What: The person adds an empty editor group on any side of the active group, or duplicates the active group with its tabs into a new group on a chosen side.
- Commands: workbench.action.newGroupLeft; workbench.action.newGroupRight; workbench.action.newGroupAbove; workbench.action.newGroupBelow; workbench.action.duplicateActiveEditorGroupLeft; workbench.action.duplicateActiveEditorGroupRight; workbench.action.duplicateActiveEditorGroupUp; workbench.action.duplicateActiveEditorGroupDown
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (AbstractCreateEditorGroupAction, AbstractDuplicateGroupAction); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (registrations lines 232 to 273); C bundle (all eight ids found)

### ED-1.18 Move a group or join groups
- Tag: both
- Evidence: source
- Boundary: no
- What: The person moves the active group left, right, up or down in the grid, or joins it into the next group, or joins every group into one.
- Commands: workbench.action.moveActiveEditorGroupLeft; workbench.action.moveActiveEditorGroupRight; workbench.action.moveActiveEditorGroupUp; workbench.action.moveActiveEditorGroupDown; workbench.action.joinTwoGroups; workbench.action.joinAllGroups
- Menus: CommandPalette
- Keys: Ctrl+K Left; Ctrl+K Right; Ctrl+K Up; Ctrl+K Down (move group)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (AbstractMoveGroupAction, JoinTwoGroupsAction, JoinAllGroupsAction); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts; C bundle (all six ids found)

### ED-1.19 Maximize an editor group
- Tag: both
- Evidence: source
- Boundary: no
- What: The person maximizes one group so the others hide, with a title menu row that flips to Unmaximize Group, a toolbar button while maximized, and an option to hide the side bars too.
- Commands: workbench.action.toggleMaximizeEditorGroup; workbench.action.maximizeEditorHideSidebar
- Menus: EditorTitle; EmptyEditorGroup
- Keys: Ctrl+K Ctrl+M
- Settings: workbench.editor.doubleClickTabToToggleEditorGroupSizes
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (lines 1130 to 1195); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts; V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts (double-click on a pinned tab); C bundle

### ED-1.20 Expand a group or reset group sizes
- Tag: both
- Evidence: source
- Boundary: no
- What: The person makes the active group take as much room as possible, optionally hiding the side bars, or evens out every group, or toggles between those two states.
- Commands: workbench.action.minimizeOtherEditors; workbench.action.minimizeOtherEditorsHideSidebar; workbench.action.evenEditorWidths; workbench.action.toggleEditorWidths
- Menus: CommandPalette
- Keys: none found
- Settings: workbench.editor.doubleClickTabToToggleEditorGroupSizes
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (lines 1052 to 1128); V\src\vs\workbench\browser\workbench.contribution.ts (the default for double-click is expand); C bundle (all four ids found)

### ED-1.21 Reset group sizes but leave agent chat groups alone
- Tag: cursor
- Evidence: bundle
- Boundary: yes (it only makes sense because an Agent chat can sit in an editor group, and it skips groups whose active editor is a chat editor)
- What: The person evens out the widths of all editor groups except those showing an agent chat, which keep their size.
- Commands: workbench.action.evenEditorWidthsExcludingAgent
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class registered with f1 and category View, nls module vs/workbench/browser/parts/editor/editorActions.cursor, title index 3568 Reset Editor Group Sizes (Excluding Agent))

### ED-1.22 Move focus between editor groups
- Tag: both
- Evidence: source
- Boundary: no
- What: The person jumps to a group by number, to the next or previous group, to the group in a direction with or without wrapping, back to the active group, or cycles through groups.
- Commands: workbench.action.focusFirstEditorGroup; workbench.action.focusSecondEditorGroup; workbench.action.focusThirdEditorGroup; workbench.action.focusFourthEditorGroup; workbench.action.focusFifthEditorGroup; workbench.action.focusSixthEditorGroup; workbench.action.focusSeventhEditorGroup; workbench.action.focusEighthEditorGroup; workbench.action.focusLastEditorGroup; workbench.action.focusNextGroup; workbench.action.focusPreviousGroup; workbench.action.focusLeftGroup; workbench.action.focusRightGroup; workbench.action.focusAboveGroup; workbench.action.focusBelowGroup; workbench.action.focusLeftGroupWithoutWrap; workbench.action.focusRightGroupWithoutWrap; workbench.action.focusAboveGroupWithoutWrap; workbench.action.focusBelowGroupWithoutWrap; workbench.action.focusActiveEditorGroup; workbench.action.navigateEditorGroups
- Menus: MenubarSwitchGroupMenu; CommandPalette
- Keys: Ctrl+1 to Ctrl+8 (group by number, and Ctrl+2 to Ctrl+8 add a new group when the number is the one right after the last group); Ctrl+K Ctrl+Left; Ctrl+K Ctrl+Right; Ctrl+K Ctrl+Up; Ctrl+K Ctrl+Down
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (lines 277 to 410); V\src\vs\workbench\browser\parts\editor\editorCommands.ts (registerFocusEditorGroupAtIndexCommands, without-wrap variants); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (Go > Switch Group, lines 1052 to 1165); C bundle (all ids found)

### ED-1.23 Lock an editor group
- Tag: both
- Evidence: source
- Boundary: no
- What: The person locks a group so editors do not open into it by default and its active editor is not replaced by accident, and some editor types lock a new group automatically.
- Commands: workbench.action.toggleEditorGroupLock; workbench.action.lockEditorGroup; workbench.action.unlockEditorGroup; mainThreadWebview-markdown.preview
- Menus: EditorTitle; EmptyEditorGroup; EmptyEditorGroupContext
- Keys: none found
- Settings: workbench.editor.autoLockGroups
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorCommands.ts (lock group actions); V\src\vs\workbench\browser\parts\editor\editorConfiguration.ts (autoLockGroups); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (menu rows and the lock toggle button); C bundle

### ED-1.24 Split an editor inside one group
- Tag: both
- Evidence: source
- Boundary: no
- What: The person shows one editor side by side with itself inside a single group, flips that pair between horizontal and vertical, joins it back, and moves focus between the two sides.
- Commands: workbench.action.splitEditorInGroup; workbench.action.joinEditorInGroup; workbench.action.toggleSplitEditorInGroup; workbench.action.toggleSplitEditorInGroupLayout; workbench.action.focusFirstSideEditor; workbench.action.focusSecondSideEditor; workbench.action.focusOtherSideEditor
- Menus: EditorSplitMoveSubmenu; EditorTitle; MenubarLayoutMenu; MenubarSwitchEditorMenu
- Keys: Ctrl+K Ctrl+Shift+\ (Split in Group, or Join in Group when a side by side editor is active)
- Settings: workbench.editor.splitInGroupLayout
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorCommands.ts (lines near SPLIT_EDITOR_IN_GROUP); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (Toggle Layout toolbar item, View > Editor Layout and Go > Switch Editor rows); V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (all seven ids found, menu rows for Split in Group and Join in Group found)

### ED-1.25 Use empty space in the tab strip and in empty groups
- Tag: both
- Evidence: source
- Boundary: no
- What: Double-clicking blank tab strip space opens a new untitled file at the end of the strip, and right-clicking it or an empty group opens a menu with split directions, new window rows, tab bar choices, and lock or close for the group.
- Commands: workbench.action.splitEditorUp; workbench.action.splitEditorDown; workbench.action.splitEditorLeft; workbench.action.splitEditorRight; workbench.action.moveEditorGroupToNewWindow; workbench.action.copyEditorGroupToNewWindow; workbench.action.newEmptyEditorWindow; workbench.action.toggleEditorGroupLock; workbench.action.closeGroup
- Menus: EditorTabsBarContext; EmptyEditorGroup; EmptyEditorGroupContext
- Keys: none found
- Settings: none found
- Cursor: changed (the close rows read Close Pane where stock reads Close Group or Close, and the tab bar Move into New Window row is hidden for Browser tabs)
- Source: V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (lines 353 to 390); V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts (registerTabsContainerListeners, double-click opens an untitled editor); C bundle (nls indexes 3418 to 3433, tab bar rows carry a when clause that excludes the cursor-browser scheme)

### ED-1.26 Right-click a tab for path and reveal actions
- Tag: both
- Evidence: source
- Boundary: no
- What: The tab menu has Copy Path, Copy Relative Path, Reveal in File Explorer and Reveal in Explorer View for tabs that show a file.
- Commands: copyFilePath; copyRelativeFilePath; revealFileInOS; revealInExplorer
- Menus: EditorTitleContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (appendEditorTitleContextMenuItem calls); V\src\vs\workbench\contrib\files\electron-browser\fileActions.contribution.ts (Reveal in File Explorer, desktop only); C bundle (all four ids found)

### ED-1.27 Reopen an editor with another editor type
- Tag: both
- Evidence: source
- Boundary: no
- What: The person re-opens the active file in a different editor, for example the text editor instead of a custom viewer, from the tab menu or the Command Palette, or flips between the text editor and the other editor in one step.
- Commands: workbench.action.reopenWithEditor; workbench.action.toggleEditorType; workbench.action.reopenTextEditor; reopenActiveEditorWith
- Menus: EditorTitleContext; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (reopenActiveEditorWith not found, the other three ids and the Reopen Editor With rows found)
- Source: V\src\vs\workbench\browser\parts\editor\editorCommands.ts (REOPEN_WITH_COMMAND_ID, REOPEN_ACTIVE_EDITOR_WITH_COMMAND_ID); V\src\vs\workbench\browser\parts\editor\editorActions.ts (ToggleEditorTypeAction line 2499, ReOpenInTextEditorAction line 2545); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts; C bundle (nls index 3452 Reopen Editor With...)

### ED-1.28 Copy a cursor.com link to the file or selection
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The person copies a cursor.com link to the current file, or to the selected lines, from a Share submenu on the tab menu or the editor menu, the line number gutter menu, or the Explorer menu.
- Commands: cursorOrigin.copyCodeLink; cursorOrigin.copyCodeLinkWithoutRange
- Menus: EditorTitleContextShare; EditorContextShare; ExplorerContextShare; EditorLineNumberContext; CommandPalette; OpenEditorsContextShare
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (nls module vs/workbench/contrib/cursorOrigin/browser/cursorOrigin.contribution, indexes 5428 to 5432; the menu rows sit in group 0_cursor and need config.git.enabled with no git.missing, plus one more condition that was not decoded); C bundle (nls module vs/workbench/contrib/cursorOrigin/browser/cursorOrigin.contribution; command ids, titles "Copy cursor.com link (with selection)" and "Copy cursor.com link (file only)", menu registrations on the four menus, enabled when config git.enabled is on and a repo with an eligible remote is open)

### ED-1.29 Step to another tab by keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: The person moves to the next or previous tab, the first or last tab of the group, or tab number 1 to 9, with the keyboard or the Go > Switch Editor menu.
- Commands: workbench.action.nextEditor; workbench.action.previousEditor; workbench.action.nextEditorInGroup; workbench.action.previousEditorInGroup; workbench.action.firstEditorInGroup; workbench.action.lastEditorInGroup; workbench.action.openEditorAtIndex1 to workbench.action.openEditorAtIndex9 (built at run time from workbench.action.openEditorAtIndex)
- Menus: MenubarSwitchEditorMenu; CommandPalette
- Keys: Ctrl+PageDown (next); Ctrl+PageUp (previous); Ctrl+K Ctrl+PageDown and Ctrl+K Ctrl+PageUp (next and previous in group); Alt+1 to Alt+9 (tab by position, Ctrl+1 to Ctrl+9 on Mac); Alt+0 or Ctrl+9 (last tab in group, Ctrl+0 or Cmd+9 on Mac); on Mac also Cmd+Alt+Right, Cmd+Alt+Left, Cmd+Shift+] and Cmd+Shift+[
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (lines 1200 to 1420); V\src\vs\workbench\browser\parts\editor\editorCommands.ts (the loop that registers openEditorAtIndex1 to 9); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (Go > Switch Editor rows); C bundle (base string workbench.action.openEditorAtIndex found, the per-index ids are built at run time so they were not matched one by one)

### ED-1.30 Switch editors with Ctrl+Tab in recently used order
- Tag: both
- Evidence: source
- Boundary: no
- What: Holding Ctrl and pressing Tab opens a picker of the group's editors in most recently used order that keeps stepping while the modifier is held, and sibling commands open the next or previous used editor straight away without a picker.
- Commands: workbench.action.quickOpenPreviousRecentlyUsedEditorInGroup; workbench.action.quickOpenLeastRecentlyUsedEditorInGroup; workbench.action.quickOpenPreviousRecentlyUsedEditor; workbench.action.quickOpenLeastRecentlyUsedEditor; workbench.action.openPreviousEditorFromHistory; workbench.action.openNextRecentlyUsedEditor; workbench.action.openPreviousRecentlyUsedEditor; workbench.action.openNextRecentlyUsedEditorInGroup; workbench.action.openPreviousRecentlyUsedEditorInGroup; workbench.action.quickOpenNavigateNextInEditorPicker; workbench.action.quickOpenNavigatePreviousInEditorPicker
- Menus: MenubarSwitchEditorMenu; CommandPalette
- Keys: Ctrl+Tab; Ctrl+Shift+Tab (Control+Tab and Control+Shift+Tab on Mac); the same chords step an open editor picker
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (lines 1777 to 1965); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (lines 311 to 331, quick navigate keybindings); V\src\vs\workbench\browser\parts\editor\editorQuickAccess.ts; C bundle (all eleven ids found)

### ED-1.31 Show All Editors in a picker
- Tag: both
- Evidence: source
- Boundary: no
- What: The person opens a picker that lists open editors across all groups, in order of appearance or most recently used, or only the active group's editors, and can close any row from its button.
- Commands: workbench.action.showAllEditors; workbench.action.showAllEditorsByMostRecentlyUsed; workbench.action.showEditorsInActiveGroup; workbench.action.showEditorsInGroup
- Menus: EditorTitle; CommandPalette
- Keys: Ctrl+K Ctrl+P (Show All Editors By Appearance; Cmd+Alt+Tab on Mac); quick access prefixes edt (appearance), edt mru (most recently used) and edt active (active group)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (lines 1730 to 1775); V\src\vs\workbench\browser\parts\editor\editorQuickAccess.ts (three providers, Close Editor row button); V\src\vs\workbench\browser\parts\editor\editorCommands.ts (SHOW_EDITORS_IN_GROUP); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (Show Opened Editors row on EditorTitle); C bundle (four ids, the prefix strings and the Show Opened Editors label found)

### ED-1.32 Go back, forward and to the last edit location
- Tag: both
- Evidence: source
- Boundary: no
- What: The person steps back and forward through earlier cursor and editor positions, jumps to the last place they edited, and clears the editor history; the Command Center arrows (owned by TB-2) call the same commands.
- Commands: workbench.action.navigateBack; workbench.action.navigateForward; workbench.action.navigateLast; workbench.action.navigateToLastEditLocation; workbench.action.navigateBackInEditLocations; workbench.action.navigateForwardInEditLocations; workbench.action.navigatePreviousInEditLocations; workbench.action.navigateBackInNavigationLocations; workbench.action.navigateForwardInNavigationLocations; workbench.action.navigatePreviousInNavigationLocations; workbench.action.navigateToLastNavigationLocation; workbench.action.clearEditorHistory; workbench.action.clearEditorHistoryWithoutConfirm
- Menus: MenubarGoMenu; CommandCenter; TouchBarContext
- Keys: Alt+Left and Alt+Right on Windows; Ctrl+Alt+Minus and Ctrl+Shift+Minus on Linux; Control+Minus and Control+Shift+Minus on Mac; Ctrl+K Ctrl+Q (last edit location); mouse back and forward buttons
- Settings: workbench.editor.navigationScope; workbench.editor.mouseBackForwardToNavigate; workbench.editor.swipeToNavigate
- Cursor: changed (clearEditorHistoryWithoutConfirm and swipeToNavigate not found, the other eleven commands and the other two settings found)
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (lines 1421 to 1647 and 1968 to 2016); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (Go menu rows, TouchBar rows); V\src\vs\workbench\browser\workbench.contribution.ts; C bundle

### ED-1.33 Put editors in their own window
- Tag: both
- Evidence: source
- Boundary: no
- What: The person moves or copies an editor, or a whole group, into a new window, opens an empty editor window, drags a tab out of the window, and sends everything in an editor window back to the main window.
- Commands: workbench.action.moveEditorToNewWindow; workbench.action.copyEditorToNewWindow; workbench.action.moveEditorGroupToNewWindow; workbench.action.copyEditorGroupToNewWindow; workbench.action.newEmptyEditorWindow; workbench.action.restoreEditorsToMainWindow
- Menus: EditorTitleContext; EditorTabsBarContext; EmptyEditorGroupContext; MenubarLayoutMenu; CommandPalette
- Keys: Ctrl+K O (Copy Editor into New Window)
- Settings: workbench.editor.dragToOpenWindow
- Cursor: changed (the tab menu hides Move into New Window and Copy into New Window for chat and Browser tabs, the tab bar menu hides its two group rows for Browser tabs; all six commands found)
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (lines 2561 to 2730); V\src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (tab, tab bar, empty group and View > Editor Layout rows); V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (EditorTitleContext rows 3462 and 3463 carry a when clause that excludes the composer and cursor-browser schemes, EditorTabsBarContext rows 3432 and 3433 exclude cursor-browser)

### ED-1.34 Zen Mode
- Tag: both
- Evidence: source
- Boundary: no
- What: Handed in from TB-4. Turning on Zen Mode takes the window to full screen, centers the editor, hides the status bar, activity bar and line numbers, quiets notifications, and keeps the tab bar in the chosen style, then a double Escape leaves it.
- Commands: workbench.action.toggleZenMode; workbench.action.exitZenMode; workbench.action.zenShowMultipleEditorTabs; workbench.action.zenShowEditorTab; workbench.action.zenHideEditorTabs
- Menus: MenubarAppearanceMenu; EditorTabsBarContext; EditorTabsBarShowTabsZenModeSubmenu
- Keys: Ctrl+K Z (toggle); Escape Escape (exit, only while in Zen Mode)
- Settings: zenMode.fullScreen; zenMode.centerLayout; zenMode.showTabs; zenMode.hideStatusBar; zenMode.hideActivityBar; zenMode.hideLineNumbers; zenMode.restore; zenMode.silentNotifications
- Cursor: present
- Source: V\src\vs\workbench\browser\workbench.zenMode.contribution.ts; V\src\vs\workbench\browser\actions\layoutActions.ts (zen tab commands and the Customize Layout quick toggle entry); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (Zen variant of the Tab Bar submenu, lines 381 to 384); C bundle (module vs/workbench/browser/actions/layoutActions, all ids and all eight zenMode settings found, toggle key chord matches; what Cursor additionally hides in Zen Mode was not checked)

### ED-1.35 Centered Layout
- Tag: both
- Evidence: source
- Boundary: no
- What: Handed in from TB-4. The person centers the main editor area with margins on both sides so a single editor reads like a page, and the width can follow the window or stay fixed.
- Commands: workbench.action.toggleCenteredLayout
- Menus: MenubarAppearanceMenu
- Keys: none found
- Settings: workbench.editor.centeredLayoutAutoResize; workbench.editor.centeredLayoutFixedWidth; zenMode.centerLayout
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (toggleCenteredLayout); V\src\vs\workbench\browser\workbench.contribution.ts (the two centered layout settings); V\src\vs\workbench\browser\workbench.zenMode.contribution.ts (zenMode.centerLayout); C bundle (command, context key isCenteredLayout and both settings found)

### ED-1.36 Open some editors in a modal overlay
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Stock can open editors such as Settings and Keyboard Shortcuts in a centered overlay with a header that has maximize, a sidebar toggle, previous and next arrows with an n of m counter, and menu rows to open the editor in the main window or a new window.
- Commands: workbench.action.closeModalEditor; workbench.action.moveModalEditorToMain; workbench.action.moveModalEditorToWindow; workbench.action.toggleModalEditorMaximized; workbench.action.toggleModalEditorSidebar; workbench.action.navigateModalEditorPrevious; workbench.action.navigateModalEditorNext
- Menus: ModalEditorTitle; ModalEditorTitleContext; ModalEditorEditorTitle
- Keys: Escape (close); Alt+Up (previous item); Alt+Down (next item)
- Settings: workbench.editor.useModal
- Cursor: missing
- Source: V\src\vs\workbench\browser\parts\editor\modalEditorPart.ts; V\src\vs\workbench\browser\parts\editor\editorCommands.ts (registerModalEditorCommands); V\src\vs\workbench\browser\workbench.contribution.ts (useModal with off, some and all, default some); C bundle (none of the seven ids and not the setting found)

### ED-1.38 Limit how many editors stay open
- Tag: both
- Evidence: source
- Boundary: no
- What: The person caps the number of open tabs, per group or across all groups, so the least recently used ones close when a new one opens, optionally never counting unsaved tabs.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.editor.limit.enabled; workbench.editor.limit.value; workbench.editor.limit.excludeDirty; workbench.editor.limit.perEditorGroup
- Cursor: present
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (lines 440 to 460); V\src\vs\workbench\browser\parts\editor\editorsObserver.ts; C bundle (all four settings found)

### ED-1.39 Connected editor tabs (Modern UI)
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. When Modern UI is on, the active tab joins the editor surface with curved shoulders, or tabs show as separate rounded pills, and tabs can reserve a fixed space for their close and pin buttons.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.experimental.modernUI; workbench.experimental.modernUIEditorTabStyle; workbench.editor.tabActionReserveSpace
- Cursor: missing
- Source: N\src\vs\workbench\contrib\modernUI\README.md; N\src\vs\workbench\contrib\modernUI\browser\connectedEditorTabs.ts; N\src\vs\workbench\contrib\modernUI\browser\modernUI.contribution.ts; N\src\vs\workbench\browser\parts\editor\connectedTabClipping.ts; N\src\vs\workbench\browser\workbench.contribution.ts (lines near 211); C bundle (workbench.experimental.modernUI, modernUIEditorTabStyle, tabActionReserveSpace, the connectedTabsBackground color and the modernTab colors all searched and not found)

### ED-1.40 Switch the editor type from the breadcrumbs bar
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. A dropdown in the breadcrumbs bar, and a Reopen Editor With submenu in the editor title overflow menu, list the editor types for the open file with Open to the Side, Set Default, and Open Original or Open Modified for diffs.
- Commands: editorType.openOriginal; editorType.openModified
- Menus: EditorTitle
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: N\src\vs\workbench\browser\parts\editor\editorTypePicker.ts; N\src\vs\workbench\browser\parts\editor\breadcrumbsControl.ts (editor type node); N\src\vs\workbench\browser\parts\editor\editorGroupView.ts (Reopen Editor With submenu in the overflow menu); C bundle (editorType.openOriginal, editorType.openModified and editor.reopenWith searched and not found)

### ED-1.41 Restore the view of a reopened file
- Tag: both
- Evidence: source
- Boundary: no
- What: When a file is opened again in a group after its tab was closed, the editor comes back with the scroll position and other view state it had, and a second setting reuses the latest view state from any group when the current group has none.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.editor.restoreViewState; workbench.editor.sharedViewState
- Cursor: present (both settings are registered in the bundle, restoreViewState with default true, and the bundle's view state code reads it)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\browser\parts\editor\editorWithViewState.ts (shouldRestoreEditorViewState checks the setting for an editor that is new in the group); V\src\vs\workbench\browser\parts\editor\editorPane.ts (reacts to sharedViewState changes); C bundle (both setting names found, restoreViewState read through the text resource configuration service)

### ED-1.42 Choose whether open editors come back at startup
- Tag: vscode
- Evidence: source
- Boundary: no
- What: By default a window reopens the groups and tabs it had when it closed, and this setting can turn that off so only tabs with unsaved changes come back; the restore is also skipped in a temporary workspace, and when the window was launched with files to open unless window.restoreWindows is set to preserve.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.editor.restoreEditors; window.restoreWindows
- Cursor: missing (the setting name and its description are not in the bundle, which only has the startup timing marks named code/restoreEditors and an internal restoreEditors method, so whether Cursor always restores was not checked; window.restoreWindows was not searched)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (the setting also carries an agentsWindow default that is ignored here); V\src\vs\workbench\browser\layout.ts (EDITOR_RESTORE_EDITORS and shouldRestoreEditors); C bundle (restoreEditors only as timing marks and a method name)

### ED-1.43 Choose the default editor for a file type
- Tag: both
- Evidence: source
- Boundary: no
- What: Handed in from ED-10. In the Reopen Editor With picker the active and default editors are marked, and a Configure default editor for '*.ext'... row asks which editor should open that file type from now on; the answer is saved in the editor associations setting (diffs have a second setting), and a warning with Configure Default and Keep buttons appears when two editors both claim the same file.
- Commands: workbench.action.reopenWithEditor; explorer.openWith
- Menus: EditorTitleContext; CommandPalette
- Keys: none found
- Settings: workbench.editorAssociations; workbench.diffEditorAssociations
- Cursor: changed (workbench.editorAssociations, the Configure default editor row, the Select new default editor prompt and the multiple default editors warning are found; workbench.diffEditorAssociations is not found)
- Source: V\src\vs\workbench\services\editor\browser\editorResolverService.ts (doPickEditor, the configureDefault row, the conflicting defaults prompt); V\src\vs\workbench\services\editor\common\editorResolverService.ts (both setting ids); V\src\vs\workbench\browser\parts\editor\editorConfiguration.ts (associations setting schema built from the registered editors); C bundle (nls strings for the row, the prompt and the warning, setting id workbench.editorAssociations, no diffEditorAssociations)

### ED-1.44 Open the active editor in a new empty window
- Tag: both
- Evidence: source
- Boundary: no
- What: From the Command Palette the person opens the file of the active editor in a new window that has no folder or workspace open, and gets an error box when the active editor holds nothing openable.
- Commands: workbench.action.files.showOpenedFileInNewWindow
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (the command and its action are present, and the title reads Open Active File in New Empty Workspace where stock reads Open Active Editor in New Empty Workspace)
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (OpenActiveFileInEmptyWorkspace, f1 only, category File, needs an empty workspace to be supported); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (registerAction2); C bundle (same command id, title string 7164, class registered)

### ED-1.45 Move around a focused tab strip with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: Once a tab has focus, Enter or Space opens it, the arrow keys step to the previous or next tab, Home and End jump to the first or last tab of the group, and Shift+F10 opens that tab's menu; how the strip first takes focus from the editor was not traced.
- Commands: none found
- Menus: EditorTitleContext (opened by Shift+F10)
- Keys: Enter or Space (open the focused tab); Left or Up (previous tab); Right or Down (next tab); Home (first tab); End (last tab); Shift+F10 (tab menu); all handled inside the tab, not registered as workbench keybindings
- Settings: none found
- Cursor: present (the same key handling, including the focusTabControl open option, is in the bundle)
- Source: V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts (tab KEY_DOWN listener for Shift+F10 and KEY_UP listener for the other keys); C bundle (same handler found)

### ED-1.46 See the progress bar under the tab strip
- Tag: both
- Evidence: source
- Boundary: no
- What: Handed in from NT-5. A thin bar runs along the top edge of the editor area, just under the tab strip, while an editor in that group is loading or reporting long work, it hides when the active editor changes or the group becomes empty, and nothing shows for an empty group.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the bundle has the same group progress logic: the bar placed at the title height minus 2 pixels, hidden on an active editor change, and a no-op runner for an empty group)
- Source: V\src\vs\workbench\browser\parts\editor\editorGroupView.ts (progressBar and its top offset under the title); V\src\vs\workbench\services\progress\browser\progressIndicator.ts (EditorProgressIndicator); V\src\vs\workbench\browser\parts\editor\editorPanes.ts (LongRunningOperation uses IEditorProgressService while an editor opens); C bundle (service id editorProgressService and the matching class)

### ED-1.47 Resize groups by dragging the divider
- Tag: both
- Evidence: source
- Boundary: no
- What: The person drags the thin divider between two groups to resize them, the groups keep their minimum sizes, double-clicking a divider evens out the sibling groups beside it (editor groups define no preferred size), and two workbench settings set how wide the grab area is and how long the hover highlight takes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.sash.size; workbench.sash.hoverDelay
- Cursor: present (both settings are registered, sash size with default 4 and 20 on iOS, hover delay 300; the drag itself was not seen on screen)
- Source: V\src\vs\workbench\browser\parts\editor\editorPart.ts (SerializableGrid of groups and their minimum sizes); V\src\vs\base\browser\ui\grid\grid.ts (onDidSashReset falls back to distributeViewSizes); V\src\vs\workbench\contrib\sash\browser\sash.contribution.ts (the two settings, which also apply to the side bar, panel and views); C bundle (both setting ids and the sash size style variable)

### ED-1.48 Drag a whole group by its tab strip
- Tag: both
- Evidence: source
- Boundary: no
- What: Dragging the empty part of a group's tab strip picks up the whole group with all its tabs (only the active editor when tabs are single or hidden), labelled with the active tab name and a plus count; dropping on another group's edge moves the group beside it, dropping in the middle merges its tabs into that group, and the copy key copies instead of moving.
- Commands: none found
- Menus: none found
- Keys: Ctrl while dragging (Alt on Mac) to copy
- Settings: none found
- Cursor: present (the drag label string with the plus count and the group transfer code are in the bundle)
- Source: V\src\vs\workbench\browser\parts\editor\editorTabsControl.ts (onGroupDragStart); V\src\vs\workbench\browser\parts\editor\editorDropTarget.ts (group branch of handleDrop: moveGroup, copyGroup, mergeGroup); V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts (strip drop with a group transfer); C bundle (nls module editorTabsControl, string 3827, and groupTransfer)

### ED-1.49 Reorder tabs by dragging them along the strip
- Tag: both
- Evidence: source
- Boundary: no
- What: Dragging one or several tabs over the strip shows a left or right marker on the tab under the pointer and releasing moves them there (the copy key copies instead), a tab dropped among the pinned tabs becomes pinned and one dropped after them becomes unpinned, and holding a drag over another tab for 1.5 seconds switches to that tab.
- Commands: none found
- Menus: none found
- Keys: Ctrl while dragging (Alt on Mac) to copy
- Settings: none found
- Cursor: present (the drop-target-left and drop-target-right classes are in the bundle and its CSS)
- Source: V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts (drop target markers, onDrop, DRAG_OVER_OPEN_TAB_THRESHOLD of 1500 ms); V\src\vs\workbench\common\editor\editorGroupModel.ts (moveEditor adjusts the pinned count); C bundle

### ED-1.50 Hold Alt to turn a tab's close button into Close Others
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. While Alt (Option on Mac) is held over a tab that shows a close button, the button becomes Close Others with an Alt+Click tooltip, and the click closes every other tab in the group except pinned ones; it is off when the group has one tab.
- Commands: workbench.action.closeOtherEditorTabInGroup
- Menus: none found
- Keys: Alt+click on a tab's close button (Option+click on Mac)
- Settings: none found
- Cursor: missing (the command id and the Alt+Click string are not in the bundle)
- Source: N\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts (isAltPressed, hoveredTabIndex and the close action swap); N\src\vs\workbench\browser\parts\editor\editorActions.ts (CloseOtherEditorTabsInGroupAction, excludes sticky editors); C bundle (closeOtherEditorTabInGroup and Alt+Click searched and not found)

### ED-1.51 Grow or shrink the editor area with Increase and Decrease Editor Width and Height
- Tag: both
- Evidence: source
- Boundary: no
- What: Four Command Palette commands change the editor area's size by 60 pixels at a time, in width or in height, no matter which part has focus.
- Commands: workbench.action.increaseViewHeight; workbench.action.decreaseViewHeight; workbench.action.increaseViewWidth; workbench.action.decreaseViewWidth
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:1138-1277; V\src\vs\workbench\browser\layout.ts:1799-1860; C bundle (increaseViewHeight and decreaseViewHeight classes calling resizePart with the editor part, 1 hit each, same 60 pixel step)

## Handed off

- The globe Open Browser button on the editor title (`workbench.action.openBrowserEditor`, group 0_global) and the Browser tab label with its page icon (CSS class `browser-editor-tab`): ED-10.
- The Edit Markdown row on the tab menu for .md files (`markdownEditor.editMarkdown`, group 2_edit): ED-10.
- The diff toolbar rows on the editor title (Inline View, Previous Change, Next Change, Swap Left and Right Side, Show Leading/Trailing Whitespace Differences) and Compare Selected on the tab menu (`compareSelected`, `selectForCompare`, `compareFiles`): ED-6.
- The Run or Debug split button on the editor title (`EditorTitleRun`): SB-5.
- The terminal tab rows on the tab menu (Move to Terminal Panel, Change Color, Size to Content Width): the Panel terminal group in section 07.
- The breadcrumbs bar under the tabs: ED-5.
- Back and forward arrows in the Command Center and the Go menu rows only call the commands listed in ED-1.32: TB-2 and TB-1 should point to ED-1.32.
- Toggle Editor Area Visibility (`workbench.action.toggleEditorVisibility`) and the Customize Layout quick toggles for Zen Mode and Centered Layout (`workbench.action.customizeLayout`): TB-4. ED-1.34 and ED-1.35 own the features themselves.
- The Open Editors view and its Toggle Vertical/Horizontal Editor Layout button (`workbench.action.toggleEditorGroupLayout`): SB-2.
- Copy cursor.com link also sits in the editor context Share submenu, the line number gutter menu and the Explorer Share submenu. It is listed once as ED-1.28 with all four surfaces, so ED-2 and SB-2 only need a pointer to it.
- The empty group watermark and the Cursor empty screen: ED-9. Only the right-click menu of an empty group is in ED-1.25.
- The language, end of line and encoding pickers (`workbench.action.editor.changeLanguageMode`, `changeEOL`, `changeEncoding`, registered in `V\src\vs\workbench\browser\parts\editor\editor.contribution.ts` lines 174 to 176): the status bar group in section 09.
- The floating button strip drawn over the editor content (`editor.contrib.floatingToolbar`, fed by `MenuId.EditorContent`) and its Open Workspace button on .code-workspace files (`workbench.action.openWorkspaceFromEditor` in `V\src\vs\workbench\contrib\workspaces\browser\workspaces.contribution.ts`, also in the Cursor bundle): no group owns the host strip, ED-5 is the closest fit, otherwise the Workbench-wide section. ED-8 and SB-5 already list their own buttons on it.
- The sash settings named in ED-1.47 (`workbench.sash.size`, `workbench.sash.hoverDelay`) also resize the side bar, panel and views: SB-1, PN-1 and AX-2 only need a pointer to ED-1.47.
- Clear Recently Opened (`workbench.action.clearRecentFiles`, class in `V\src\vs\workbench\browser\parts\editor\editorActions.ts`, File > Open Recent): TB-1 already lists it.

## Unclear

- Auto save has no owner. `workbench.action.toggleAutoSave`, `files.autoSave`, `files.autoSaveDelay`, `files.autoSaveWorkspaceFilesOnly`, `files.autoSaveWhenNoErrors` and `V\src\vs\workbench\browser\parts\editor\editorAutoSave.ts` all exist in V and the first two plus the delay setting are in the Cursor bundle. The File menu row is in TB-1, and no catalog group covers saving.
- In `V\src\vs\workbench\browser\parts\editor\editorActions.ts` lines 97 to 185, Split Editor Orthogonal, Split Left, Split Right, Split Up and Split Down are all given Ctrl+K Ctrl+\ with no when clause. I did not test which one wins, so ED-1.13 lists the chord once.
- The Add Tab control (`editor.tabs.addTab`, menu `EditorTabsBarAddTab`) is already in V `multiEditorTabsControl.ts`, but only the Sessions (Agents) window fills its menu (`V\src\vs\sessions\contrib\editor\browser\addTabActions.ts`), so it stays hidden in a normal window. It is not in the Cursor bundle. Left out as an agent window surface. The N editor header control (`editorHeaderControl.ts`, used when `showHeader` is set) is left out for the same reason.
- Agent chat tabs in Cursor have their own tab label (`composer-tab-label`), extra tab menu rows such as Rename Chat, and New Agent or Replace Agent buttons on the editor title. They are chat surfaces and were not catalogued. The context key `cursor.chatEditorGroup.enabled` ("Whether chat editor group mode is enabled") may change how groups behave, and was not followed.
- The stock Share submenu (`workbench.action.share`) needs a share provider from an extension, so it is not listed. Cursor puts its own row in the same submenu (see ED-1.28).
- Cursor keys were not compared. Every Keys line comes from V. For Zen Mode the Cursor bundle shows the same Ctrl+K Z chord. Cursor ids and labels were checked in the bundle, but key tables were only read for Zen Mode.
- What Cursor additionally hides or changes while in Zen Mode was not checked.
- Nothing was run on screen, so visual differences in Cursor's tab strip (height, button placement, colors) cannot be told from strings alone. The only Cursor tab CSS found that is not stock is for the chat label and the Browser tab label.
- `reopenActiveEditorWith` (an internal id used by ED-1.27) is not in the Cursor bundle. Whether any Cursor path needs it was not checked.
- The `agentsWindow` default overrides in `V\src\vs\workbench\browser\workbench.contribution.ts` (for example `doubleClickTabToToggleEditorGroupSizes` and `restoreEditors`) apply to the Agents window only and were ignored.
- Overflow check on 2026-10-25 added 10 features (ED-1.41 to ED-1.50).
