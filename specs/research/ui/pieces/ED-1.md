Group: ED-1 Editor groups, tabs and layout. Pass: 3. Date: 2026-10-25.

### ED-1.1
1. Right-clicking blank space in the tab strip opens a menu with a Tab Bar submenu that offers Multiple Tabs, Single Tab and Hidden with the current choice checked, and the same submenu sits under View > Appearance (`workbench.action.showMultipleEditorTabs`, `workbench.action.showEditorTab`, `workbench.action.hideEditorTabs`).
2. Setting `workbench.editor.showTabs` chooses how editors show, with values `multiple` (one tab per editor, the default), `single` (the active editor as one large tab) and `none` (no title area).
3. The Editor Actions Position submenu, in the tab strip menu and under View > Appearance, moves the editor action buttons to the Tab Bar, the Title Bar or Hidden with the current choice checked, and the Command Palette has Move Editor Actions to Title Bar, Move Editor Actions to Tab Bar, Hide Editor Actions and Show Editor Actions for the same choice (`workbench.action.editorActionsTitleBar`, `workbench.action.editorActionsDefault`, `workbench.action.hideEditorActions`, `workbench.action.showEditorActions`).
4. Setting `workbench.editor.editorActionsLocation` chooses where editor actions show, with values `default` (the tab bar, or the title bar when tabs are hidden), `titleBar` and `hidden`.
5. Setting `workbench.editor.alwaysShowEditorActions` chooses whether editor actions stay visible when the editor group is not active, off by default.
6. Choosing Configure Tabs in the tab strip menu opens Settings filtered to the tab settings, and choosing Configure Editors in the editor title overflow menu opens Settings filtered to the editor settings (`workbench.action.configureEditorTabs`, `workbench.action.configureEditor`).

### ED-1.2
1. Setting `workbench.editor.tabSizing` chooses tab width, with values `fit` (full label, the default), `shrink` (tabs get narrower when space runs out) and `fixed` (all tabs the same width, narrower when space runs out).
2. Settings `workbench.editor.tabSizingFixedMinWidth` (default 50, minimum 38) and `workbench.editor.tabSizingFixedMaxWidth` (default 160, minimum 38) set the narrowest and widest tab when `tabSizing` is `fixed`.
3. Setting `window.density.editorTabHeight` chooses tab height, with values `default` and `compact`, and it also sets the height of the title control bar when tabs are not shown as multiple.
4. Setting `workbench.editor.wrapTabs` chooses whether tabs that do not fit wrap onto more rows or stay in one row with a scrollbar, off by default and ignored unless `workbench.editor.showTabs` is `multiple`.
5. Setting `workbench.editor.scrollToSwitchTabs` chooses whether scrolling the mouse wheel over the tabs opens each tab or only reveals it, off by default, with Shift held flipping the choice while it is held.
6. Setting `workbench.editor.titleScrollbarSizing` chooses the height of the scrollbar under tabs and breadcrumbs, with values `default` and `large`.
7. Setting `workbench.editor.titleScrollbarVisibility` chooses when the tab scrollbar shows, with values `auto` (only when needed), `visible` and `hidden`. [stock only]

### ED-1.3
1. Each tab shows the file's icon and its file name.
2. When two tabs in one group share a file name, each label adds the distinguishing folder parts of its path.
3. Setting `workbench.editor.labelFormat` chooses the tab label, with values `default`, `short` (name and directory name), `medium` (name and path relative to the workspace folder) and `long` (name and absolute path).
4. Setting `workbench.editor.untitled.labelFormat` chooses the label of untitled tabs, with values `content` (taken from the first line, the default) and `name`.
5. Setting `workbench.editor.customLabels.enabled` turns custom tab labels on or off (on by default), and `workbench.editor.customLabels.patterns` maps file patterns to label templates.
6. Setting `workbench.editor.showIcons` chooses whether tabs show an icon, on by default and needing a file icon theme.
7. Settings `workbench.editor.decorations.badges` and `workbench.editor.decorations.colors` choose whether tabs show file decoration badges and colors, both on by default.
8. Hovering a tab shows the full path, and for a preview tab the hover adds a preview note with a gear link named Configure Preview Mode that opens `workbench.editor.enablePreview` in Settings.
9. Setting `workbench.editor.showTabIndex` shows each tab's index number in the tab, off by default and ignored unless `workbench.editor.showTabs` is `multiple`. [stock only]

### ED-1.4
1. Each tab has a close button that closes its editor when clicked (`workbench.action.closeActiveEditor`).
2. A pinned tab shows an unpin button in the same place, and clicking it unpins the tab (`workbench.action.unpinEditor`).
3. A tab with unsaved changes shows a dot in place of the close icon, and a pinned tab with unsaved changes shows a pinned-dot icon.
4. Setting `workbench.editor.tabActionLocation` chooses which side of the tab holds the close and unpin buttons, with values `left` and `right` (the default).
5. Settings `workbench.editor.tabActionCloseVisibility` and `workbench.editor.tabActionUnpinVisibility` choose whether the close and unpin buttons show, both on by default.
6. Setting `workbench.editor.highlightModifiedTabs` draws a colored top border on tabs with unsaved changes, off by default.
7. When tabs are shown as single or hidden, Close and Unpin buttons appear in the editor toolbar instead of on a tab.

### ED-1.5
1. A file opened with a single click shows its name in italics in a preview tab, and the next file opened the same way replaces it, with at most one preview tab per group.
2. Making a change in the preview editor, or double-clicking the file in the Explorer, makes the tab a normal tab with no italics.
3. Setting `workbench.editor.enablePreview` turns preview tabs on or off, on by default.
4. The Enable Preview Editors row in the editor title overflow menu shows checked while preview is on, and choosing it flips the setting (`workbench.action.toggleKeepEditors`).
5. Setting `workbench.editor.enablePreviewFromQuickOpen` chooses whether files opened from Quick Open show as preview tabs, off by default, with Ctrl held before selecting opening a non-preview tab when it is on.
6. Setting `workbench.editor.enablePreviewFromCodeNavigation` chooses whether an editor stays in preview when a code navigation such as Go to Definition starts from it, off by default.

### ED-1.6
1. Right-clicking a preview tab and choosing Keep Open turns it into a normal tab, and the row only shows while `workbench.editor.enablePreview` is on and is disabled once the tab is already kept.
2. Pressing `Ctrl+K Enter`, or running Keep Editor from the Command Palette, keeps the active preview editor open (`workbench.action.keepEditor`).
3. Double-clicking a preview tab makes it a normal tab.

### ED-1.7
1. Choosing Pin from the tab menu or the Command Palette pins the tab and moves it to the front of the strip, and the menu row then reads Unpin (`workbench.action.pinEditor`, `workbench.action.unpinEditor`).
2. Pressing `Ctrl+K Shift+Enter` pins a tab that is not pinned and unpins one that is.
3. Setting `workbench.editor.pinnedTabSizing` chooses how pinned tabs look, with values `normal`, `compact` (icon or first letter only) and `shrink` (a compact fixed size showing part of the name).
4. Setting `workbench.editor.pinnedTabsOnSeparateRow` shows pinned tabs on their own row above the other tabs, off by default, and the Separate Pinned Editor Tabs command flips it from the Command Palette (`workbench.action.toggleSeparatePinnedEditorTabs`).
5. Setting `workbench.editor.preventPinnedEditorClose` chooses when a pinned tab refuses to close, with values `keyboardAndMouse` (the default), `keyboard`, `mouse` and `never`.

### ED-1.8
1. Clicking a tab's close button, or middle-clicking the tab, closes that editor.
2. The Close row in the tab menu, File > Close Editor, the Command Palette and the keyboard close the active editor, using `Ctrl+F4` or `Ctrl+W` on Windows (`workbench.action.closeActiveEditor`).
3. Close Active Pinned Editor closes a pinned tab even when the keyboard close is blocked for pinned tabs (`workbench.action.closeActivePinnedEditor`).
4. Revert File and Close Editor discards unsaved changes and closes the active editor in one step (`workbench.action.revertAndCloseActiveEditor`).
5. Setting `workbench.editor.focusRecentEditorAfterClose` chooses which tab becomes active after a close, the most recently used one when on (the default) or the next one from left to right when off.
6. Setting `workbench.editor.closeOnFileDelete` chooses whether an editor closes by itself when its file is deleted or renamed by another process, off by default, and an editor with unsaved changes never closes this way.

### ED-1.9
1. Choosing Close Others in the tab menu closes every other tab in the group, and the row is disabled when the group has one tab (`workbench.action.closeOtherEditors`, `Cmd+Alt+T` on Mac).
2. Choosing Close to the Right in the tab menu closes the tabs after the clicked one and is disabled on the last tab or while several tabs are selected, and Close Editors to the Left in Group does the same on the other side from the Command Palette (`workbench.action.closeEditorsToTheRight`, `workbench.action.closeEditorsToTheLeft`).
3. Choosing Close Saved, from the tab menu, the editor title overflow menu or `Ctrl+K U`, closes every tab in the group that has no unsaved changes (`workbench.action.closeUnmodifiedEditors`).
4. Choosing Close All, from the tab menu, the editor title overflow menu or `Ctrl+K W`, closes every tab in the group (`workbench.action.closeEditorsInGroup`).
5. Pressing `Ctrl+K Ctrl+W` closes every tab in every group (`workbench.action.closeAllEditors`).
6. Pressing `Ctrl+K Ctrl+Shift+W` closes every tab and every group (`workbench.action.closeAllGroups`).
7. Running `workbench.action.closeGroup` or `workbench.action.closeEditorsAndGroup` removes the active group, pressing `Ctrl+W` on an empty group closes it, and an empty group's menu and toolbar have a Close row.
8. Close Editors in Other Groups and Close Editor in All Groups work from the Command Palette on the other groups, or on the same editor in every group (`workbench.action.closeEditorsInOtherGroups`, `workbench.action.closeEditorInAllGroups`).
9. Setting `workbench.editor.closeEmptyGroups` chooses whether a group closes by itself when its last tab closes, on by default.

### ED-1.10
1. Pressing `Ctrl+Shift+T` brings back the most recently closed tab (`workbench.action.reopenClosedEditor`).
2. File > Open Recent > Reopen Closed Editor does the same and is disabled when there is nothing to reopen.
3. The Command Palette lists Reopen Closed Editor under the View category and runs the same reopen.

### ED-1.11
1. Ctrl-click (Cmd-click on Mac) on a tab adds it to the selection, and Shift-click selects every tab between the active tab and the clicked one.
2. Selected tabs show a selected highlight that differs from the single active tab.
3. Right-clicking a selected tab opens the tab menu, and its close, move, split and new window rows act on every selected tab.
4. While several tabs are selected, rows that need one tab are disabled or hidden, such as Close to the Right, Split in Group and Share.

### ED-1.12
1. Pressing `Ctrl+Shift+PageUp` moves the active tab one place left in its group and `Ctrl+Shift+PageDown` one place right, and on Mac the chords are `Cmd+K Cmd+Shift+Left` and `Cmd+K Cmd+Shift+Right` (`workbench.action.moveEditorLeftInGroup`, `workbench.action.moveEditorRightInGroup`, `moveActiveEditor`).
2. Move Editor to Start and Move Editor to End from the Command Palette send the active tab to either end of the strip (`workbench.action.moveEditorToStart`, `workbench.action.moveEditorToEnd`). [stock only]
3. Setting `workbench.editor.openPositioning` chooses where a newly opened tab lands, with values `left`, `right` (the default), `first` and `last`.
4. Setting `workbench.editor.revealIfOpen` chooses whether opening an editor that is already open in a visible group reveals that group instead of opening a copy in the active group, off by default.

### ED-1.13
1. Pressing `Ctrl+\` or clicking the split button in the editor toolbar splits the active editor into a new group beside it showing the same file, to the right by default (`workbench.action.splitEditor`).
2. Holding Alt while the toolbar split button shows turns it into the other direction, so a right split offers down and a down split offers right.
3. The tab strip menu, the empty group menu and View > Editor Layout offer Split Up, Split Down, Split Left and Split Right (`workbench.action.splitEditorUp`, `workbench.action.splitEditorDown`, `workbench.action.splitEditorLeft`, `workbench.action.splitEditorRight`).
4. Split Editor Orthogonal, with `Ctrl+K Ctrl+\`, splits in the direction at a right angle to the current split direction (`workbench.action.splitEditorOrthogonal`).
5. Setting `workbench.editor.openSideBySideDirection` chooses the default split direction, with values `right` (the default) and `down`, and it also sets what the toolbar split button does.
6. Setting `workbench.editor.splitSizing` chooses group sizes after a split, with values `auto`, `distribute` and `split`.
7. The tab menu in Cursor lists Split Up, Split Down, Split Left and Split Right as flat rows. [cursor only]

### ED-1.14
1. Move Editor into Previous Group and Next Group send the active editor to the neighboring group in that order, with `Ctrl+Alt+Left` and `Ctrl+Alt+Right` (`Cmd+Ctrl+Left` and `Cmd+Ctrl+Right` on Mac) (`workbench.action.moveEditorToPreviousGroup`, `workbench.action.moveEditorToNextGroup`).
2. Move Editor into First Group and Last Group send the active editor to either end of the group order, with `Shift+Alt+1` and `Shift+Alt+9` (`Cmd+Ctrl+1` and `Cmd+Ctrl+9` on Mac) (`workbench.action.moveEditorToFirstGroup`, `workbench.action.moveEditorToLastGroup`).
3. Move Editor into Left, Right, Above and Below Group send the active editor to the group in that direction from the Command Palette (`workbench.action.moveEditorToLeftGroup`, `workbench.action.moveEditorToRightGroup`, `workbench.action.moveEditorToAboveGroup`, `workbench.action.moveEditorToBelowGroup`).
4. Split Editor into Previous, Next, First and Last Group leave the original tab in place and open a copy in that group (`workbench.action.splitEditorToPreviousGroup`, `workbench.action.splitEditorToNextGroup`, `workbench.action.splitEditorToFirstGroup`, `workbench.action.splitEditorToLastGroup`).
5. Split Editor into Left, Right, Above and Below Group leave the original tab in place and open a copy in the group in that direction (`workbench.action.splitEditorToLeftGroup`, `workbench.action.splitEditorToRightGroup`, `workbench.action.splitEditorToAboveGroup`, `workbench.action.splitEditorToBelowGroup`).
6. The tab menu has a Split & Move submenu with Split Up, Down, Left and Right, then Move Above, Below, Left and Right, then Split in Group or Join in Group. [stock only]

### ED-1.15
1. Dragging a tab or file over an editor group shows a drop overlay, and dropping on an edge splits the group in that direction while dropping in the middle moves the editor into the group.
2. Holding Ctrl while dragging (Alt on Mac) copies the editor instead of moving it.
3. Holding Alt while dragging (Shift on Mac) toggles whether dropping on an edge splits the group.
4. Holding Shift while dragging a file over a text editor drops the file into the text, and a prompt reading Hold Shift to drop into editor shows while dragging.
5. Setting `editor.dropIntoEditor.enabled` turns dropping a file into the text of an editor on or off.
6. Setting `workbench.editor.splitOnDragAndDrop` chooses whether dropping an editor or file on the edges of the editor area splits the group, on by default.
7. Dragging a tab out of the window opens it in a new window, and setting `workbench.editor.dragToOpenWindow` turns this on or off, on by default, with Alt held while dragging toggling it for that drag.

### ED-1.16
1. Choosing Single, Two Columns, Three Columns, Two Rows, Three Rows, Grid (2x2), Two Rows Right or Two Columns Bottom from View > Editor Layout rearranges the editor area into that grid of groups.
2. The same eight presets are in the Command Palette (`workbench.action.editorLayoutSingle`, `workbench.action.editorLayoutTwoColumns`, `workbench.action.editorLayoutThreeColumns`, `workbench.action.editorLayoutTwoRows`, `workbench.action.editorLayoutThreeRows`, `workbench.action.editorLayoutTwoByTwoGrid`, `workbench.action.editorLayoutTwoRowsRight`, `workbench.action.editorLayoutTwoColumnsBottom`).
3. Choosing a preset keeps every open editor.
4. A command or extension can set an exact group grid with a layout description and read the current grid back (`layoutEditorGroups`, `vscode.setEditorLayout`, `vscode.getEditorLayout`).

### ED-1.17
1. New Editor Group to the Left, to the Right, Above and Below add an empty group on that side of the active group (`workbench.action.newGroupLeft`, `workbench.action.newGroupRight`, `workbench.action.newGroupAbove`, `workbench.action.newGroupBelow`).
2. Duplicate Editor Group Left, Right, Up and Down create a new group on that side holding the same tabs as the active group and leave the original group's tabs alone (`workbench.action.duplicateActiveEditorGroupLeft`, `workbench.action.duplicateActiveEditorGroupRight`, `workbench.action.duplicateActiveEditorGroupUp`, `workbench.action.duplicateActiveEditorGroupDown`).
3. All eight commands are in the Command Palette and have no default keyboard shortcut.

### ED-1.18
1. Pressing `Ctrl+K Left`, `Ctrl+K Right`, `Ctrl+K Up` or `Ctrl+K Down` moves the active group in that direction in the grid, and Move Editor Group Left, Right, Up and Down do the same from the Command Palette (`workbench.action.moveActiveEditorGroupLeft`, `workbench.action.moveActiveEditorGroupRight`, `workbench.action.moveActiveEditorGroupUp`, `workbench.action.moveActiveEditorGroupDown`).
2. Join Editor Group with Next Group moves the active group's tabs into the next group and removes the active group (`workbench.action.joinTwoGroups`).
3. Join All Editor Groups merges the tabs of every group into one group (`workbench.action.joinAllGroups`).

### ED-1.19
1. Maximize Group, from the editor title overflow menu or an empty group's toolbar, hides the other groups so the active one fills the editor area, and the row only shows while several groups are open (`workbench.action.toggleMaximizeEditorGroup`).
2. While a group is maximized the menu row reads Unmaximize Group and a toolbar button restores the other groups.
3. Pressing `Ctrl+K Ctrl+M` toggles whether the active group is maximized.
4. Maximize Editor Group and Hide Side Bars maximizes the group and also hides the side bars (`workbench.action.maximizeEditorHideSidebar`).
5. Setting `workbench.editor.doubleClickTabToToggleEditorGroupSizes` set to `maximize` makes a double-click on a tab maximize the group, with values `maximize`, `expand` (the default) and `off`.

### ED-1.20
1. Expand Editor Group makes the active group take as much room as possible by shrinking the other groups to their smallest size (`workbench.action.minimizeOtherEditors`).
2. Expand Editor Group and Hide Side Bar does the same and also hides the side bars (`workbench.action.minimizeOtherEditorsHideSidebar`).
3. Reset Editor Group Sizes makes every group the same size (`workbench.action.evenEditorWidths`).
4. Toggle Editor Group Sizes switches between the expanded state and the even state (`workbench.action.toggleEditorWidths`).
5. Setting `workbench.editor.doubleClickTabToToggleEditorGroupSizes` set to `expand`, its default, makes a double-click on a tab expand the group or restore the sizes.

### ED-1.21
1. Reset Editor Group Sizes (Excluding Agent) is in the Command Palette under the View category with no default key (`workbench.action.evenEditorWidthsExcludingAgent`). [cursor only]
2. Running it makes the editor groups the same width. [cursor only]
3. A group whose active editor is an Agent chat keeps its current width when it runs. [cursor only]

### ED-1.22
1. Pressing `Ctrl+1` to `Ctrl+8` focuses the group with that number, and Go > Switch Group lists the same choices (`workbench.action.focusFirstEditorGroup` to `workbench.action.focusEighthEditorGroup`).
2. Pressing `Ctrl+2` to `Ctrl+8` with the number right after the last group adds a new group and focuses it.
3. Focus Last Editor Group focuses the final group in the grid (`workbench.action.focusLastEditorGroup`).
4. Focus Next Group and Focus Previous Group move focus to the neighboring group in order (`workbench.action.focusNextGroup`, `workbench.action.focusPreviousGroup`).
5. Pressing `Ctrl+K Ctrl+Left`, `Ctrl+K Ctrl+Right`, `Ctrl+K Ctrl+Up` or `Ctrl+K Ctrl+Down` focuses the group in that direction, and from an edge group focus wraps around (`workbench.action.focusLeftGroup`, `workbench.action.focusRightGroup`, `workbench.action.focusAboveGroup`, `workbench.action.focusBelowGroup`).
6. The Without Wrap variants of the four direction commands stop at the edge of the grid instead of wrapping (`workbench.action.focusLeftGroupWithoutWrap`, `workbench.action.focusRightGroupWithoutWrap`, `workbench.action.focusAboveGroupWithoutWrap`, `workbench.action.focusBelowGroupWithoutWrap`).
7. Focus Active Editor Group returns focus to the active group, and Navigate Between Editor Groups steps focus through the groups (`workbench.action.focusActiveEditorGroup`, `workbench.action.navigateEditorGroups`).

### ED-1.23
1. Lock Group in the editor title overflow menu, in an empty group's menu, and the lock and unlock commands in the Command Palette lock the group, and the row shows checked while it is locked (`workbench.action.toggleEditorGroupLock`, `workbench.action.lockEditorGroup`, `workbench.action.unlockEditorGroup`).
2. A locked group is not used to open editors by default and only takes an editor when the person picks it, for example by drag and drop, so its active editor is not replaced by accident.
3. In an editor window an empty group's toolbar and the editor toolbar show a lock button that turns into an unlock button while the group is locked.
4. Setting `workbench.editor.autoLockGroups` is a list of editor types, each on or off, and a group is locked automatically when one of the types on opens first in a group while several groups are open, with terminal, simple browser, browser preview and process explorer on by default.

### ED-1.24
1. Pressing `Ctrl+K Ctrl+Shift+\` or choosing Split in Group shows the active editor side by side with itself inside one group (`workbench.action.splitEditorInGroup`).
2. Pressing the same chord, or choosing Join in Group, while a side by side editor is active puts the pair back into one editor (`workbench.action.joinEditorInGroup`).
3. Toggle Split Editor in Group splits or joins in one command (`workbench.action.toggleSplitEditorInGroup`).
4. The Toggle Layout button in the editor toolbar flips the pair between horizontal and vertical (`workbench.action.toggleSplitEditorInGroupLayout`).
5. Focus First Side, Second Side and Other Side in Active Editor move focus between the two sides (`workbench.action.focusFirstSideEditor`, `workbench.action.focusSecondSideEditor`, `workbench.action.focusOtherSideEditor`).
6. Setting `workbench.editor.splitInGroupLayout` chooses how the pair is arranged, with values `horizontal` (left to right, the default) and `vertical` (top to bottom).
7. View > Editor Layout and Go > Switch Editor have rows for splitting, joining and moving focus between the two sides.

### ED-1.25
1. Double-clicking blank space in the tab strip opens a new untitled file at the end of the strip.
2. Right-clicking blank space in the tab strip opens a menu with Split Up, Down, Left and Right, Move into New Window, Copy into New Window, the Tab Bar and Editor Actions Position submenus and Configure Tabs.
3. Right-clicking inside an empty group opens a menu with Split Up, Down, Left and Right, New Window, Lock Group (checked while locked) and Close (shown when several groups are open).
4. An empty group's toolbar shows a Lock Group button in editor windows and a Close Group button in editor windows or when several groups are open.
5. In Cursor the close rows read Close Pane where stock reads Close Group or Close, and the tab strip menu hides its new window rows for Browser tabs. [cursor only]

### ED-1.26
1. Copy Path in the tab menu copies the full path of the file the tab shows (`copyFilePath`).
2. Copy Relative Path in the tab menu copies the path relative to the workspace folder (`copyRelativeFilePath`).
3. Reveal in File Explorer in the tab menu shows the file in the operating system file manager (`revealFileInOS`).
4. Reveal in Explorer View in the tab menu selects the file in the Explorer view (`revealInExplorer`).
5. These four rows show only on tabs that show a file.

### ED-1.27
1. Reopen Editor With... in the tab menu or the Command Palette opens a picker of the editor types that can show the active file and reopens it in the one chosen, and the row only shows when the file has other editor types available (`workbench.action.reopenWithEditor`).
2. Reopen Editor with Text Editor reopens the active file in the plain text editor (`workbench.action.reopenTextEditor`).
3. Toggle Editor Type flips the active file between the text editor and the other editor in one step (`workbench.action.toggleEditorType`).
4. The internal command `reopenActiveEditorWith` reopens the active editor with an editor type passed to it. [stock only]

### ED-1.28
1. Right-clicking a tab and opening its Share submenu shows Copy cursor.com link, which copies a link to the file without a line range (`cursorOrigin.copyCodeLinkWithoutRange`). [cursor only]
2. Right-clicking in the editor and opening its Share submenu shows Copy cursor.com link, which copies a link that includes the selected lines (`cursorOrigin.copyCodeLink`). [cursor only]
3. Right-clicking in the line number gutter shows Copy cursor.com link, which copies a link that includes the selected lines. [cursor only]
4. The Explorer Share submenu shows Copy cursor.com link, which copies a link to the file. [cursor only]
5. The Command Palette has Copy cursor.com link (with selection) and Copy cursor.com link (file only). [cursor only]
6. The menu rows only show when `git.enabled` is on and Git is not reported missing. [cursor only]
7. Choosing Copy cursor.com link (with selection) copies a cursor.com link to the current file that includes the selected line range. [cursor only]
8. Both items are in the Share submenu of the editor's right-click menu, the Share submenu of the tab's right-click menu, the Share submenu of the Explorer, and the line-number gutter menu. [cursor only]
9. Both items work only when `git.enabled` is on and the open file sits in a Git repository that has an eligible remote. [cursor only]

### ED-1.29
1. Pressing `Ctrl+PageDown` opens the next tab and `Ctrl+PageUp` the previous tab, and on Mac `Cmd+Alt+Right`, `Cmd+Alt+Left`, `Cmd+Shift+]` and `Cmd+Shift+[` also do this (`workbench.action.nextEditor`, `workbench.action.previousEditor`).
2. Pressing `Ctrl+K Ctrl+PageDown` and `Ctrl+K Ctrl+PageUp` opens the next and previous tab within the active group only (`workbench.action.nextEditorInGroup`, `workbench.action.previousEditorInGroup`).
3. The First Editor in Group and Last Editor in Group commands open the first and last tab of the group, and the last tab also opens with `Alt+0` or `Ctrl+9` (`Ctrl+0` or `Cmd+9` on Mac) (`workbench.action.firstEditorInGroup`, `workbench.action.lastEditorInGroup`).
4. Pressing `Alt+1` to `Alt+9` opens the tab at that position in the group, with `Ctrl+1` to `Ctrl+9` on Mac (`workbench.action.openEditorAtIndex1` to `workbench.action.openEditorAtIndex9`).
5. Go > Switch Editor lists the next, previous, first and last tab rows.

### ED-1.30
1. Pressing `Ctrl+Tab` opens a picker of the group's editors in most recently used order, `Ctrl+Shift+Tab` opens it stepping the other way, and the same chords keep stepping while the picker is open (`workbench.action.quickOpenPreviousRecentlyUsedEditorInGroup`, `workbench.action.quickOpenLeastRecentlyUsedEditorInGroup`, `workbench.action.quickOpenNavigateNextInEditorPicker`, `workbench.action.quickOpenNavigatePreviousInEditorPicker`).
2. The all-groups picker commands list editors from every group in most recently used order, one starting from the most recent and one from the least recent (`workbench.action.quickOpenPreviousRecentlyUsedEditor`, `workbench.action.quickOpenLeastRecentlyUsedEditor`).
3. Open Next Recently Used Editor and Open Previous Recently Used Editor switch tab straight away with no picker, with in-group versions for the active group only (`workbench.action.openNextRecentlyUsedEditor`, `workbench.action.openPreviousRecentlyUsedEditor`, `workbench.action.openNextRecentlyUsedEditorInGroup`, `workbench.action.openPreviousRecentlyUsedEditorInGroup`).
4. Open Previous Editor from History opens a picker of earlier editors from the editor history (`workbench.action.openPreviousEditorFromHistory`).
5. Go > Switch Editor lists these commands and shows their chords.

### ED-1.31
1. Show All Editors By Appearance, with `Ctrl+K Ctrl+P` (`Cmd+Alt+Tab` on Mac), opens a picker that lists the open editors of every group in the order the tabs appear (`workbench.action.showAllEditors`).
2. Show All Editors By Most Recently Used lists the open editors of every group with the most recently used first (`workbench.action.showAllEditorsByMostRecentlyUsed`).
3. Show Editors in Active Group lists only the active group's editors, and the Show Opened Editors row in the editor title overflow menu opens it (`workbench.action.showEditorsInActiveGroup`, `workbench.action.showEditorsInGroup`).
4. Typing `edt `, `edt mru ` or `edt active ` in Quick Open shows the by appearance, most recently used and active group lists.
5. Each picker row has a Close Editor button that closes that editor from the picker.

### ED-1.32
1. Go > Back and Go > Forward step through earlier cursor and editor positions, with `Alt+Left` and `Alt+Right` on Windows, `Ctrl+Alt+Minus` and `Ctrl+Shift+Minus` on Linux and `Control+Minus` and `Control+Shift+Minus` on Mac, and the Command Center arrows call the same commands (`workbench.action.navigateBack`, `workbench.action.navigateForward`, `workbench.action.navigateLast`).
2. Pressing `Ctrl+K Ctrl+Q` jumps to the last place the person edited (`workbench.action.navigateToLastEditLocation`).
3. The edit location commands step back, forward and to the previous position through edit positions only (`workbench.action.navigateBackInEditLocations`, `workbench.action.navigateForwardInEditLocations`, `workbench.action.navigatePreviousInEditLocations`).
4. The navigation location commands step back, forward, to the previous position and to the last position through navigation positions only (`workbench.action.navigateBackInNavigationLocations`, `workbench.action.navigateForwardInNavigationLocations`, `workbench.action.navigatePreviousInNavigationLocations`, `workbench.action.navigateToLastNavigationLocation`).
5. Clear Editor History asks the person to confirm and then clears the editor history (`workbench.action.clearEditorHistory`).
6. Setting `workbench.editor.navigationScope` chooses how far Go Back and Go Forward reach, with values `default` (all editors and groups), `editorGroup` and `editor`.
7. Setting `workbench.editor.mouseBackForwardToNavigate` chooses whether mouse buttons four and five do Go Back and Go Forward, on by default.
8. Setting `workbench.editor.swipeToNavigate` chooses whether a three-finger horizontal swipe moves between open files on macOS, off by default. [stock only]
9. Clear Editor History Without Confirmation clears the history with no prompt (`workbench.action.clearEditorHistoryWithoutConfirm`). [stock only]

### ED-1.33
1. Move into New Window and Copy into New Window in the tab menu send the editor to a new window, with `Ctrl+K O` copying it (`workbench.action.moveEditorToNewWindow`, `workbench.action.copyEditorToNewWindow`).
2. Move into New Window and Copy into New Window in the tab strip menu send the whole group to a new window (`workbench.action.moveEditorGroupToNewWindow`, `workbench.action.copyEditorGroupToNewWindow`).
3. New Window in an empty group's menu, and New Empty Editor Window, open an empty editor window (`workbench.action.newEmptyEditorWindow`).
4. Dragging a tab out of the window opens it in a new editor window, and setting `workbench.editor.dragToOpenWindow` turns this on or off.
5. Restore Editors into Main Window, run in an editor window, sends everything in that window back to the main window (`workbench.action.restoreEditorsToMainWindow`).
6. View > Editor Layout has rows for moving or copying the editor into a new window.
7. In Cursor the tab menu hides Move into New Window and Copy into New Window for chat and Browser tabs, and the tab strip menu hides its two group rows for Browser tabs. [cursor only]

### ED-1.34
1. Turning on Zen Mode, from `Ctrl+K Z`, View > Appearance or the Command Palette, takes the window to full screen, centers the editor, hides the status bar, activity bar and line numbers, quiets notifications and keeps the tab bar in the chosen style (`workbench.action.toggleZenMode`).
2. Pressing Escape twice leaves Zen Mode, and this only applies while Zen Mode is on (`workbench.action.exitZenMode`).
3. In Zen Mode the tab strip menu shows a Tab Bar submenu with Multiple Tabs, Single Tab and Hidden that changes the Zen Mode tab style (`workbench.action.zenShowMultipleEditorTabs`, `workbench.action.zenShowEditorTab`, `workbench.action.zenHideEditorTabs`).
4. Setting `zenMode.showTabs` chooses the tab style in Zen Mode, with values `multiple` (the default), `single` and `none`.
5. Settings `zenMode.fullScreen` and `zenMode.centerLayout` choose whether Zen Mode also goes full screen and centers the layout, both on by default.
6. Settings `zenMode.hideStatusBar`, `zenMode.hideActivityBar` and `zenMode.hideLineNumbers` choose whether Zen Mode hides the status bar, the activity bar and the line numbers, all on by default.
7. Setting `zenMode.silentNotifications` chooses whether Zen Mode lets only error notifications pop out, on by default.
8. Setting `zenMode.restore` chooses whether a window that closed in Zen Mode comes back in Zen Mode, on by default.

### ED-1.35
1. Toggle Centered Layout, from View > Appearance or the Command Palette, centers the main editor area with margins on both sides (`workbench.action.toggleCenteredLayout`).
2. Setting `workbench.editor.centeredLayoutAutoResize` chooses whether the centered layout grows to full width while several groups are open and returns to its centered width when one group is left, on by default.
3. Setting `workbench.editor.centeredLayoutFixedWidth` chooses whether the centered layout keeps a constant width when the window is resized, off by default.
4. Setting `zenMode.centerLayout` chooses whether turning on Zen Mode also centers the layout, on by default.

### ED-1.36
1. With `workbench.editor.useModal` at its default `some`, editors such as Settings and Keyboard Shortcuts open in a centered overlay instead of a tab. [stock only]
2. The overlay header has a maximize button, a sidebar toggle, and previous and next arrows with an n of m counter (`workbench.action.toggleModalEditorMaximized`, `workbench.action.toggleModalEditorSidebar`, `workbench.action.navigateModalEditorPrevious`, `workbench.action.navigateModalEditorNext`). [stock only]
3. Pressing Escape closes the overlay, and `Alt+Up` and `Alt+Down` move to the previous and next item (`workbench.action.closeModalEditor`). [stock only]
4. The overlay's menu has rows that open the editor in the main window or in a new window (`workbench.action.moveModalEditorToMain`, `workbench.action.moveModalEditorToWindow`). [stock only]
5. Setting `workbench.editor.useModal` chooses which editors use the overlay, with values `off`, `some` and `all`. [stock only]

### ED-1.38
1. With the limit on, opening a tab past the limit closes the least recently used tabs to make room.
2. Setting `workbench.editor.limit.enabled` turns the limit on or off, off by default.
3. Setting `workbench.editor.limit.value` sets the most editors that stay open, 10 by default.
4. Setting `workbench.editor.limit.perEditorGroup` chooses whether the limit counts each group separately or all groups together, off (all groups) by default.
5. Setting `workbench.editor.limit.excludeDirty` chooses whether editors with unsaved changes are left out of the count, off by default.

### ED-1.39
1. With `workbench.experimental.modernUI` on, the active tab joins the editor surface with an outside stroke and curved shoulders. [stock only]
2. Setting `workbench.experimental.modernUIEditorTabStyle` chooses the tab look, with values `connected` (the default) and `pill`. [stock only]
3. With the `pill` style, tabs show as separate rounded pills with no connected stroke or shoulders. [stock only]
4. Setting `workbench.editor.tabActionReserveSpace` chooses whether tabs always keep a column for their close and pin buttons, on by default, and when off clean tabs stay compact. [stock only]
5. In Modern UI a clean inactive tab shows its close button only on hover or keyboard focus, while the active tab and tabs with a dirty or pin marker keep theirs visible. [stock only]

### ED-1.40
1. A dropdown in the breadcrumbs bar lists the editor types that can show the open file. [stock only]
2. A Reopen Editor With submenu in the editor title overflow menu lists the same editor types. [stock only]
3. Each editor type also has an entry labelled Open to the Side that opens it beside the current editor. [stock only]
4. A Set Default for the file pattern row makes the chosen editor the default, and for diffs the row reads Set Default (Diff Only). [stock only]
5. For a diff, Open Original and Open Modified rows open just one side (`editorType.openOriginal`, `editorType.openModified`). [stock only]

### ED-1.41
1. Reopening a file in a group after its tab was closed brings back the scroll position and other view state it had.
2. Setting `workbench.editor.restoreViewState` chooses whether that view state is restored, on by default, with the state kept per group and dropped when the group closes, and it can be set per language.
3. Setting `workbench.editor.sharedViewState` chooses whether the most recent view state from any group is used when the current group has none for that file, off by default.

### ED-1.42
1. A window reopens the groups and tabs it had when it closed. [stock only]
2. Setting `workbench.editor.restoreEditors` set to off makes a window reopen only the tabs that had unsaved changes. [stock only]
3. The restore is skipped in a temporary workspace. [stock only]
4. The restore is skipped when the window was launched with files to open, unless `window.restoreWindows` is set to `preserve`. [stock only]

### ED-1.43
1. In the Reopen Editor With picker the active editor and the default editor are marked, and the Explorer Open With row shows the same picker (`workbench.action.reopenWithEditor`, `explorer.openWith`).
2. A row named Configure default editor for the file extension asks which editor should open that file type from now on.
3. The Select new default editor prompt saves the answer in setting `workbench.editorAssociations`.
4. When two editors both claim the same file, a warning about multiple default editors appears with Configure Default and Keep buttons.
5. Setting `workbench.diffEditorAssociations` saves the default editor for diffs of a file type. [stock only]

### ED-1.44
1. Open Active Editor in New Empty Workspace in the Command Palette opens the file of the active editor in a new window with no folder or workspace open (`workbench.action.files.showOpenedFileInNewWindow`).
2. An error box shows when the active editor holds nothing that can be opened this way.
3. In Cursor the command is titled Open Active File in New Empty Workspace. [cursor only]

### ED-1.45
1. Pressing Enter or Space on a focused tab opens it.
2. Pressing Left or Up moves focus to the previous tab and Right or Down to the next tab.
3. Pressing Home moves focus to the first tab of the group and End to the last tab.
4. Pressing Shift+F10 on a focused tab opens that tab's menu.

### ED-1.46
1. A thin progress bar runs along the top edge of the editor area, just under the tab strip, while an editor in that group is loading or reporting long work.
2. The bar hides when the active editor of the group changes.
3. An empty group shows no progress bar, and the bar hides when the group becomes empty.

### ED-1.47
1. Dragging the thin divider between two groups resizes them.
2. Neither group shrinks below its minimum size while dragging.
3. Double-clicking a divider makes the sibling groups beside it the same size.
4. Setting `workbench.sash.size` sets the width of the grab area of dividers in pixels, from 1 to 20 with a default of 4 (20 on iOS).
5. Setting `workbench.sash.hoverDelay` sets the delay in milliseconds before a divider shows its hover highlight, from 0 to 2000 with a default of 300.

### ED-1.48
1. Dragging the empty part of a group's tab strip picks up the whole group with all its tabs, or only the active editor when tabs are shown as single or hidden.
2. The drag label shows the active tab's name and a plus count for the other tabs.
3. Dropping the group on another group's edge moves it beside that group.
4. Dropping the group in the middle of another group merges its tabs into that group.
5. Holding Ctrl while dragging (Alt on Mac) copies the group instead of moving it.

### ED-1.49
1. Dragging one or several tabs over the strip shows a left or right marker on the tab under the pointer, and releasing moves them to that spot.
2. Holding Ctrl while dragging (Alt on Mac) copies the tabs instead of moving them.
3. A tab dropped among the pinned tabs becomes pinned and one dropped after them becomes unpinned.
4. Holding a drag over another tab for 1.5 seconds switches to that tab.

### ED-1.50
1. While Alt (Option on Mac) is held over a tab that shows a close button, the button becomes Close Others (`workbench.action.closeOtherEditorTabInGroup`). [stock only]
2. The button's tooltip then reads Alt+Click, or Option+Click on Mac. [stock only]
3. Clicking it closes every other tab in the group except pinned ones. [stock only]
4. The button is disabled when the group has only one tab. [stock only]

### ED-1.51
1. "Increase Editor Height" makes the editor area 60 pixels taller and "Decrease Editor Height" makes it 60 pixels shorter, and the neighboring parts give up or take the space.
2. "Increase Editor Width" and "Decrease Editor Width" do the same for width.
3. With one editor group, the whole editor area is resized.
4. With several editor groups, the active group is resized first, and if it cannot move in that direction the whole editor area is resized instead.
5. The step is 60 pixels scaled for the screen, and the commands do nothing while a separate editor window has focus.
6. The four commands are only in the Command Palette and have no default keys, unlike Increase and Decrease Current View Size, which resize the focused part (SB-1.4).

## Unclear

- ED-1.15 and ED-1.33 both own the behavior of dragging a tab out of the window to open a new window, so a tracking count would show it in both features.
- ED-1.42 pieces end with [stock only] because the feature says Cursor is missing, but nobody checked whether Cursor always restores editors at startup.
- ED-1.39, ED-1.40 and ED-1.50 pieces come from VS Code main 1.144.0 and describe behavior newer than the 1.128.0 base, so they are [stock only] and may not belong in a Cursor build target.
- ED-1.28 link rows carry one more menu condition in the Cursor bundle that was not decoded, so piece 6 lists only the two conditions that were read.
- ED-1.13 piece 4 follows the feature's Keys line that gives `Ctrl+K Ctrl+\` to Split Editor Orthogonal, but V gives the same chord to Split Left, Right, Up and Down with no when clause, and which one wins was not tested.
