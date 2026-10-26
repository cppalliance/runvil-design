Group: PN-4 Problems. Section: Panel (PN). Date: 2026-10-25. Pass: 2.
Features: 20.

Short names used in Source: V is the stock 1.128.0 tree, N is the 1.144.0 clone, C bundle is `workbench.desktop.main.js` in the Cursor install, and NLS is `out\nls.messages.json` next to it. Unless a path says otherwise, V paths below start at `V\src\vs\workbench\contrib\markers\`. The view id is `workbench.panel.markers.view` and the container id is `workbench.panel.markers`, so the toggle and view-mode command ids below are built from that view id (V builds them from a template, so the full ids do not appear as plain strings in V or in the C bundle). Line numbers are from the files as read and can drift by a line or two. Cursor findings come from strings and compiled code only. Nothing was run on screen.

### PN-4.1 Open, toggle and focus the Problems view
- Tag: both
- Evidence: source
- Boundary: no
- What: Show the Problems tab, toggle it, or move focus into it from the View menu (Problems), from Ctrl+Shift+M, from the Command Palette, or by clicking the Problems item in the status bar. The command title reads Toggle Problems because the view lives in the panel. The tab hides itself when the view has been removed from its container.
- Commands: workbench.actions.view.problems; workbench.actions.view.toggleProblems; workbench.action.problems.focus; workbench.action.showErrorsWarnings; workbench.panel.markers.view.focus; workbench.action.tasks.toggleProblems
- Menus: MenubarViewMenu (group 5_panel, order 0, generated from the view's open command); CommandPalette
- Keys: Ctrl+Shift+M (workbench.actions.view.problems)
- Settings: none found
- Cursor: present (all four ids found by string; the generated focus command comes from the same registerFocusViewAction code that Cursor still has; labels Toggle Problems (Errors, Warnings, Infos) and Focus Problems (Errors, Warnings, Infos) found in NLS)
- Source: contrib\markers\browser\markers.contribution.ts (43-90, 132-157, 340-570); contrib\markers\common\markers.ts (8-30); V\src\vs\workbench\services\views\browser\viewsService.ts (471-575); V\src\vs\workbench\contrib\markers\browser\messages.ts (12-13); research entry-points 01-title-bar (View menu rows, 5_panel order 0); C bundle (markers view registration with `workbench.actions.view.problems` and keybinding Ctrl+Shift+M, registerFocusViewAction)

### PN-4.2 See problems as a tree grouped by file
- Tag: both
- Evidence: source
- Boundary: no
- What: The default view is a tree. Each file that has problems is one row with a count badge, and its problems sit under it. A problem row shows the severity icon, the message (all lines of it), the source and code beside the message, and [Ln, Col] at the end. A code that has a link target is a clickable link. Files expand and collapse, and a toolbar button collapses all files (that button only shows in tree mode).
- Commands: workbench.actions.treeView.workbench.panel.markers.view.collapseAll; workbench.actions.table.workbench.panel.markers.view.viewAsTree
- Menus: ViewTitle (Collapse All, group navigation, order 2, tree mode only)
- Keys: none found
- Settings: problems.defaultViewMode
- Cursor: present (the markers tree classes, the collapseAll template id, the row CSS class names marker-code, marker-source and related-info-resource, and the NLS label Collapse All were found)
- Source: contrib\markers\browser\markersTreeViewer.ts (152-226 file row and count, 279-415 marker row, 416-455 related rows); markersView.ts (487-520); markers.contribution.ts (161-184, 534-552); markersModel.ts (38-140); C bundle (markers tree viewer, collapseAll registration with menu Pe.ViewTitle order 2); NLS 7654

### PN-4.3 See problems as a flat table
- Tag: both
- Evidence: source
- Boundary: no
- What: Table mode lists every problem as its own row with no file grouping. The columns are an icon column (severity, with the quick fix light bulb), Code, Message, File and Source. The Code column shows the source name and the code, and the code is a link when the diagnostic gives a target. The File column shows the workspace-relative path and [Ln, Col]. Matches from the filter box are highlighted in the message, file, code and source cells.
- Commands: workbench.actions.table.workbench.panel.markers.view.viewAsTable
- Menus: ViewTitle (View as Table, group navigation, order 3, shown only in tree mode)
- Keys: none found
- Settings: problems.defaultViewMode
- Cursor: present (the markers-table-container class name and the View as Table label were found; the table column labels use the same NLS strings as stock)
- Source: contrib\markers\browser\markersTable.ts (63-335); markersView.ts (464-485); markers.contribution.ts (186-208); C bundle (markers table container, viewAsTable registration); NLS 7637

### PN-4.4 Switch between tree and table
- Tag: both
- Evidence: source
- Boundary: no
- What: One toolbar button in the Problems header flips between the two views. It reads View as Table in tree mode and View as Tree in table mode. The choice is saved per workspace. The setting only decides the mode the first time, before any choice is saved.
- Commands: workbench.actions.table.workbench.panel.markers.view.viewAsTree; workbench.actions.table.workbench.panel.markers.view.viewAsTable
- Menus: ViewTitle (group navigation, order 3, one entry for each direction)
- Keys: none found
- Settings: problems.defaultViewMode
- Cursor: present (both ids and the setting found; the setting default is tree in both)
- Source: markers.contribution.ts (104-109, 161-208); markersView.ts (172-173, 384-392, 531-533, 907-908); C bundle (viewAsTree and viewAsTable registrations, setting problems.defaultViewMode); NLS 7636-7637

### PN-4.5 Filter problems by text and file pattern
- Tag: both
- Evidence: source
- Boundary: no
- What: Type in the filter box in the Problems header to hide rows that do not match. Plain text matches the message, file name, source and code. Comma separated glob patterns such as **/*.ts show only matching files, and a leading ! excludes. Past filters are kept in a history. Ctrl+F in the list jumps to the filter box, Ctrl+Down moves back into the list, and Escape clears the text. Typing a letter anywhere in the list also moves focus to the filter box. The text is saved per workspace.
- Commands: problems.action.focusFilter; problems.action.focusProblemsFromFilter; problems.action.clearFilterText
- Menus: none found
- Keys: Ctrl+F (when the Problems view is focused); Ctrl+Down (when the filter box is focused); Escape (when the filter box is focused)
- Settings: none found
- Cursor: changed (the commands, keys and glob and ! filtering are present; the placeholder reads "Filter (e.g. text, **/*.ts, !**/node_modules/**)" with no "@source:ts" hint, because the source filter code is missing, see PN-4.6)
- Source: markersView.ts (156-167, 222-236, 277-291, 579-581); markersFilterOptions.ts (14-130); markers.contribution.ts (443-475, 514-532); V\src\vs\workbench\browser\parts\views\viewFilter.ts (60-235); messages.ts (40); C bundle (markers filter options and commands; placeholder is NLS 7706); NLS 7705-7706

### PN-4.6 Filter problems by source with @source
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Type @source:ts in the filter box to keep only problems reported by one source, such as ts or eslint. Put ! in front (!@source:eslint) to hide a source. Quote names that have spaces. The tip is part of the placeholder text in stock.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing (searched the C bundle for @source:, includeSourceFilters and matchesSourceFilters, and NLS for the stock placeholder text; the only @source hits are the Keyboard Shortcuts and Settings editors, and the Problems placeholder has no source tip)
- Source: markersFilterOptions.ts (14, 56-60, 86-106, 130-155); markersTreeViewer.ts (509-512); messages.ts (40); C bundle (no source filter in the markers filter options; the @source: regexes found belong to the settings and keybindings search); NLS 7706

### PN-4.7 Show or hide errors, warnings and infos
- Tag: both
- Evidence: source
- Boundary: no
- What: Click the More Filters button (filter icon) next to the filter box and tick or untick Show Errors, Show Warnings and Show Infos. The button gets a highlight whenever any filter is active, and the choices are saved per workspace. All three are on by default.
- Commands: workbench.actions.workbench.panel.markers.view.toggleErrors; workbench.actions.workbench.panel.markers.view.toggleWarnings; workbench.actions.workbench.panel.markers.view.toggleInfos
- Menus: submenu.view.filter (More Filters, group 1_filter, orders 1 to 3); menu.view.filter (the filter icon button)
- Keys: none found
- Settings: none found
- Cursor: present (all three template ids found with the toggled context keys problems.filter.errors, problems.filter.warnings and problems.filter.info; labels found in NLS)
- Source: markers.contribution.ts (210-283); markersViewActions.ts (14-120); markersView.ts (185-192, 284-287, 901-905); V\src\vs\workbench\browser\parts\views\viewFilter.ts (32-60); contrib\markers\common\markers.ts (36-40); C bundle (toggleErrors, toggleWarnings and toggleInfos registrations); NLS 7638-7643

### PN-4.8 Show only the active file, and hide excluded files
- Tag: both
- Evidence: source
- Boundary: no
- What: Two more entries in the More Filters menu. Show Active File Only limits the list to problems in the file of the active editor (the list follows you as you switch tabs, and auto reveal is skipped while it is on). Show Excluded Files is ticked by default, and unticking it hides files that match the files.exclude patterns of their workspace folder.
- Commands: workbench.actions.workbench.panel.markers.view.toggleActiveFile; workbench.actions.workbench.panel.markers.view.toggleExcludedFiles
- Menus: submenu.view.filter (More Filters, group 2_filter, orders 1 and 2)
- Keys: none found
- Settings: files.exclude
- Cursor: present (both template ids found, with the toggled context keys problems.filter.activeFile and problems.filter.excludedFiles; the labels Show Active File Only and Show Excluded Files are in NLS)
- Source: markers.contribution.ts (285-333); markersView.ts (193-200, 395-423, 770-780); markersViewActions.ts (14-120); contrib\markers\common\markers.ts (40-42); messages.ts (33-36); C bundle (toggleActiveFile and toggleExcludedFiles registrations); NLS 7644-7647

### PN-4.9 See how many problems match, and what an empty list means
- Tag: both
- Evidence: source
- Boundary: no
- What: When a filter hides some rows, the filter box shows a badge such as "Showing 3 of 12". When nothing is left, the body shows a message instead of an empty list: "No problems have been detected in the workspace.", "No problems have been detected in the current file." (Active File Only on), or "No results found with provided filter criteria." followed by a Clear Filters link that resets the text and the severity and exclude toggles. A screen reader hears the same counts.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (all four strings, Clear Filters and the Showing {0} of {1} badge text found in NLS)
- Source: markersView.ts (279-291, 685-768); messages.ts (24-27); V\src\vs\workbench\browser\parts\views\viewFilter.ts (updateBadge and checkMoreFilters, 60-235); C bundle (markers view message box code, class name messageAction); NLS 7705 region

### PN-4.10 Open a problem in the editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Click, double-click or press Enter on a problem row to open its file with the cursor on the problem. A single click or arrow-key move opens it without taking focus from the list and highlights the problem range in the editor. Double-click or Enter pins the tab and moves focus into the editor. Related-information rows open their own location the same way. Opening a file row does nothing, only problem rows jump.
- Commands: problems.action.open
- Menus: none found
- Keys: Enter (when a problem row is focused); Cmd+Down on macOS (secondary)
- Settings: none found
- Cursor: present (problems.action.open found, with the range highlight decoration code)
- Source: markers.contribution.ts (44-56); markersView.ts (300-326, 447-449, 782-808); contrib\markers\common\markers.ts (23); C bundle (problems.action.open keybinding rule, rangeHighlightDecorations)

### PN-4.11 Open a problem beside the current editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Enter on a focused problem row (Ctrl+Enter on macOS too) to open its file in the side editor group, with the cursor on the problem. A mouse click with Ctrl, Cmd or Alt held reaches the same code path through the list's open event (listService.ts 732 and 753).
- Commands: problems.action.openToSide
- Menus: none found
- Keys: Ctrl+Enter (when a problem row is focused); Ctrl+Enter on macOS (WinCtrl)
- Settings: none found
- Cursor: present (problems.action.openToSide found)
- Source: markers.contribution.ts (58-70); markersView.ts (300-326, 447-449); contrib\markers\common\markers.ts (24); V\src\vs\platform\list\browser\listService.ts (732, 753); C bundle (problems.action.openToSide keybinding rule)

### PN-4.12 Keep the active file's problems in view
- Tag: both
- Evidence: source
- Boundary: no
- What: With the setting on (default), the list scrolls to and reveals the problems of the file in the active editor whenever the active editor changes, and also when the current file gets its first problem. It is skipped while Show Active File Only is on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: problems.autoReveal
- Cursor: present (setting found)
- Source: markers.contribution.ts (100-104); markersView.ts (586-608, 656-663, 770-780); messages.ts (15); C bundle (setting problems.autoReveal)

### PN-4.13 Quick fix from a problem row
- Tag: both
- Evidence: source
- Boundary: no
- What: The selected or focused problem row (tree or table) shows a light bulb when a language provider offers a quick fix for it. Click the light bulb, press Ctrl+. on the row, or right-click the row to see the fixes (quick-fix kind only), then pick one to apply it. The tooltip reads "Show fixes". A row with an auto-fixable fix gets a stronger bulb. The view waits for the file's text model to load before it can offer fixes. Picking one opens the file at the problem without taking focus, then applies the fix.
- Commands: problems.action.showQuickFixes; workbench.actions.problems.quickfix (action id of the light bulb button, not a registered command)
- Menus: ProblemsPanelContext (quick fix entries are added at the top by the view, not by a menu contribution)
- Keys: Ctrl+. (when a problem row is focused)
- Settings: none found
- Cursor: present (both ids found, plus the markers-panel-action-quickfix class name and the "Show fixes" label; no Cursor-specific fix entry was found in this path)
- Source: markers.contribution.ts (80-93); markersViewActions.ts (122-185); markersTreeViewer.ts (279-345, 574-700); markersTable.ts (63-120); markersView.ts (293-298, 451-460, 838-852); messages.ts (35); C bundle (QuickFixAction and quick fix view item, problems.action.showQuickFixes); NLS 7705 region

### PN-4.14 Right-click menu on a problem row
- Tag: both
- Evidence: source
- Boundary: yes - Cursor adds Add to Chat and Fix with Agent to this menu in a separate group; those two chat hand-offs are not listed, only the stock entries are.
- What: Right-click a problem (tree or table) to get a menu with the quick fixes first, then Copy and Copy Message. On a related-information row only Copy Message shows. The entries show their keyboard shortcut next to them when they have one.
- Commands: problems.action.copy; problems.action.copyMessage; problems.action.copyRelatedInformationMessage
- Menus: ProblemsPanelContext (group navigation)
- Keys: Ctrl+C (Copy, when the Problems view is focused)
- Settings: none found
- Cursor: changed (the stock entries are present; Cursor adds workbench.action.problems.addToChat "Add to Chat" and workbench.action.problems.fixInChat "Fix with Agent" in group z_chat, orders 1 and 2, when a problem row is focused)
- Source: markers.contribution.ts (372-442); markersView.ts (809-852); V\src\vs\platform\actions\common\actions.ts (148); C bundle (six Pe.ProblemsPanelContext registrations: Copy, Copy Message, Copy related message, quick fix path, and the two chat entries); NLS 7662-7664, 7667-7668

### PN-4.15 Copy a problem or its message
- Tag: both
- Evidence: source
- Boundary: no
- What: Select one or more problem rows (or whole file rows) and use Copy, or press Ctrl+C in the list, to put the full problem data on the clipboard as JSON text in square brackets (severity, message, range, source, code, file path and related information). Copy Message puts only the message text there, one line per problem. On a related-information row, Copy Message copies that related message.
- Commands: problems.action.copy; problems.action.copyMessage; problems.action.copyRelatedInformationMessage
- Menus: ProblemsPanelContext (group navigation)
- Keys: Ctrl+C (Copy, when the Problems view is focused, the list is visible and no related-information row is focused)
- Settings: none found
- Cursor: present (all three ids found, with the Ctrl+C keybinding rule)
- Source: markers.contribution.ts (350-442); markersModel.ts (112-118); contrib\markers\common\markers.ts (17-19); C bundle (Copy, Copy Message and related message registrations); NLS 7662-7664

### PN-4.16 Show problem messages on one line or several
- Tag: both
- Evidence: source
- Boundary: no
- What: A problem whose message has more than one line shows a small chevron at the end of its row. Click it to switch between all lines and the first line only (tooltips Show message in single line and Show message in multiple lines). Two Command Palette entries do the same for every row at once. The choice is saved per workspace, and multiple lines is the default. Messages longer than 1000 characters per line are cut with an ellipsis.
- Commands: problems.action.showMultilineMessage; problems.action.showSinglelineMessage; problems.action.toggleMultiline (action id of the row chevron, not a registered command)
- Menus: CommandPalette (shown when the Problems view is visible)
- Keys: none found
- Settings: none found
- Cursor: present (both command ids, the toggleMultiline action id, the two chevron icons markers-view-multi-line-expanded and markers-view-multi-line-collapsed, and both labels found)
- Source: markers.contribution.ts (480-513); markersTreeViewer.ts (100-115, 256-277, 343-385, 600-615); markersView.ts (527-529, 906); contrib\markers\common\markers.ts (21-22); C bundle (the two icons and Usg="problems.action.toggleMultiline")

### PN-4.17 See the related locations of a problem
- Tag: both
- Evidence: source
- Boundary: no
- What: When a diagnostic points to other places (for example "first declared here"), its row gets an expand arrow, and its screen reader label says it has references to N locations. Expanding with the arrow shows one child row per location with the file name, the message and [Ln, Col]. Clicking a child opens that spot in the editor. A plain click on the problem row does not expand it, only the arrow does.
- Commands: problems.action.copyRelatedInformationMessage
- Menus: ProblemsPanelContext (Copy Message, when a related-information row is focused)
- Keys: Enter (the list's own open key; problems.action.open needs a problem row focused)
- Settings: none found
- Cursor: present (the related-information renderer class names and the copy command id found)
- Source: markersTreeViewer.ts (416-455, 510-512); markers.contribution.ts (422-442); markersModel.ts (133-140); messages.ts (57, 64-74); C bundle (RelatedInformationRenderer, class related-info-resource, RELATED_INFORMATION_COPY_MESSAGE_ACTION_ID)

### PN-4.18 Problems count badge on the tab
- Tag: both
- Evidence: source
- Boundary: no
- What: A number badge with the total of errors, warnings and infos sits on the Problems tab in the panel (tooltip "Total N Problems"). It disappears when the count is zero. If the Problems view is dragged into the side bar or auxiliary bar, the same badge shows on that container's icon instead. The tab itself can be icon only or labelled depending on the panel label setting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.panel.showLabels
- Cursor: present (showViewActivity called with the markers view id and a number badge, and the label Total {0} Problems found; registered with disableInGlass so the Agents window skips it)
- Source: markers.contribution.ts (696-721); V\src\vs\workbench\browser\workbench.contribution.ts (573-593 for workbench.panel.showLabels); C bundle (ActivityUpdater class e6a with showViewActivity); NLS 7633

### PN-4.19 Turn problem markers off everywhere
- Tag: both
- Evidence: source
- Boundary: no
- What: The setting problems.visibility (on by default) switches problem markers off across the workbench: squiggles in editors, and error and warning marks in the Explorer and the Outline. When it is off, a warning item with the whole-word icon is added in the status bar next to the Problems counts ("Problems are turned off. Click to open settings.") and opens this setting. I found no read of the setting in the Problems list code or in the count item, so neither is hidden by it. It is listed here because the setting lives in the Problems settings group.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: problems.visibility
- Cursor: present (the setting string, the status.problemsVisibility item with the whole-word icon and the settings link @id:problems.visibility, and the NLS text were found)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (958-969); markers.contribution.ts (578-640); V\src\vs\workbench\browser\parts\editor\textEditor.ts (41-43, 111-112, 171); markersFileDecorations.ts (69-85); V\src\vs\workbench\contrib\outline\browser\outline.contribution.ts (68-78); C bundle (MarkersStatusBarContributions, getMarkersItemTurnedOff); NLS 7626-7627

### PN-4.20 Open accessibility help for the Problems filter
- Tag: vscode
- Evidence: source
- Boundary: no
- What: With focus in the Problems filter box, Open Accessibility Help shows a text page for screen reader users that explains what the filter does, the filter syntax, the severity and scope toggles, keyboard navigation and the problems settings. The page is chosen by the context key problemsFilterFocus.
- Commands: editor.action.accessibilityHelp
- Menus: CommandPalette (Open Accessibility Help)
- Keys: Alt+F1 (Linux: Alt+Shift+F1, with Alt+F1 as secondary)
- Settings: accessibility.verbosity.find
- Cursor: missing (searched NLS for "Accessibility Help: Problems Panel Filter" and "You are in the Problems panel", and the bundle for ProblemsFilterHelp; the only filter help page in Cursor is "Accessibility Help: Terminal Find". The generic command editor.action.accessibilityHelp is present)
- Source: markersAccessibilityHelp.ts (1-110); markers.contribution.ts (723-724); V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (164-182); V\src\vs\workbench\contrib\accessibility\common\accessibilityCommands.ts (7-8); C bundle (editor.action.accessibilityHelp found, no Problems help provider); NLS (page title absent)

## Handed off

- Problems and Problems-turned-off items in the status bar (error and warning counts, tooltip, click toggles the view; `status.problems`, `status.problemsVisibility`): SS-5.
- Current problem text in the status bar when `problems.showCurrentInStatus` is on (`editorStatus.ts` lines 984 and 1035): SS-5 (SS-3 is also a fit because the code sits with the editor status items).
- Go to Next and Previous Problem (F8, Shift+F8, the in-file and in-files variants) and the `problems.sortOrder` setting that sets their order (`markerNavigationService.ts` line 51): ED-4.
- Squiggles and the problem hover in the text editor, which `problems.visibility` also switches (`textEditor.ts` line 171): ED-4.
- Error and warning marks on Explorer files and folders and the `problems.decorations.enabled` setting (`markersFileDecorations.ts`): SB-2.
- Problem marks on Outline elements (`outline.problems.enabled`, `outline.problems.colors`, `outline.problems.badges`): SB-7.

## Unclear

- The brief names a "Show in Editor" row action. I found no command or label with that text in the V markers and output code, and none in the Cursor bundle or NLS table. I treated it as the open actions PN-4.10 and PN-4.11. Tell me if a different action was meant.
- Cursor claims the 1.128.0 base, but its Problems filter has no `@source:` support and no accessibility help page, and its placeholder is the older text. I only report what the bundle contains, and I did not check which Cursor release dropped or never merged these.
- Cursor's two chat entries in the row menu (Add to Chat, Fix with Agent) are in PN-4.14 only as a note. Stock V also has `markersChatContext.ts`, which attaches problems to a chat request; it is not in the Cursor bundle by id or class name and it is chat, so it is not listed.
- Dragging a problem row or file row out of the list (to open or attach it) uses a drag handler in the tree and the table (`MarkersListDnDHandler`, `ResourceListDnDHandler`). I did not list it as a feature because I could not tell where a drop lands.
- Task problem matchers also feed this list, and the list drops duplicate diagnostics from the same source, code, severity, message and range (`markersModel.ts` lines 208-219). The matchers are task configuration, so I did not list them.
- PN-4.19 spans several sections. It is here only because the setting is in the Problems group. The group owners may want to move it.
- Behavior and look were read from code and strings only. I did not open Cursor to see which rows and menus show on a fresh window.
