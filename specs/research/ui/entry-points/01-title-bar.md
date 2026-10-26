Section: Title bar (TB). Date: 2026-10-25. Pass: 1.

# Title bar entry points: the menu bar inventory

This file answers "where are the menu items and commands". It lists every item of the application menu bar (File, Edit, Selection, View, Go, Run, Terminal, Help and their submenus), then the other menus that draw on the title bar (layout controls, command center, title bar toolbar, right-click menus, Accounts, Manage).

## How to read it

- V is stock VS Code 1.128.0. C is Cursor 3.27.12 (VS Code base 1.128.0). Paths in the "Registered in V" column are relative to `src\vs` inside V.
- Menu ids live in `src\vs\platform\actions\common\actions.ts` lines 117-145 (menubar ids) and 181-184 (title bar ids).
- The Run menu is `MenubarDebugMenu`. There is no `MenubarRunMenu` in V or in Cursor (checked in both). The top-level label "Run" is registered in `workbench/contrib/debug/browser/debug.contribution.ts:261`, not in `menubar.contribution.ts`.
- A row is one registration. A command that is registered twice with different `when` conditions (for example Open Folder, Close Folder, Move Side Bar Left and Right) shows as two rows, and only one of them is visible at a time.
- Labels are the exact V strings. `&&` marks the mnemonic letter. The menu bar shows the mnemonic title, and the toggled mnemonic title when the action has `toggled.mnemonicTitle` (see `menubarControl.ts:564-578`).
- Group strings sort the menu: the number before the underscore orders the groups, and `order` sorts inside a group. A separator is drawn between groups.
- The Cursor column was filled by searching the Cursor bundle (`out\vs\workbench\workbench.desktop.main.js`) for each registration. I matched by label, group and order, then by command id. Cursor registrations are minified, so many Cursor ids are not readable. The Cursor status words are:
  - `both` - same label, group and order found in Cursor.
  - `both (...)` - found, with the difference named.
  - `vscode only (...)` - not found in Cursor, with the check I made.
  - `cursor only` - found only in Cursor.
- Default keybindings are not captured here.
- I read the menu registrations with a scratch parser over all `.ts` files under `src\vs` (tests and `src\vs\sessions` skipped), 259 references to `MenuId.Menubar*`. I then fixed the gaps by hand (Go to Definition and friends, Terminal items, View menu items generated from views, a few ids built from constants). The parser output is not committed.

## Totals

Counts are registrations. "C" is the number found in the Cursor bundle (items present in both plus Cursor-only items).

| Menu id | Shown as | V items | C items |
|-|-|-|-|
| `MenubarMainMenu` | top-level strip | 9 | 9 |
| `MenubarFileMenu` | File | 27 | 27 |
| `MenubarEditMenu` | Edit | 13 | 13 |
| `MenubarSelectionMenu` | Selection | 17 | 17 |
| `MenubarViewMenu` | View | 17 | 15 |
| `MenubarGoMenu` | Go | 19 | 21 |
| `MenubarDebugMenu` | Run | 16 | 16 |
| `MenubarTerminalMenu` | Terminal | 12 | 11 |
| `MenubarHelpMenu` | Help | 19 | 9 |
| `MenubarPreferencesMenu` | Preferences (macOS app menu, File submenu elsewhere) | 12 | 11 |
| `MenubarRecentMenu` | File > Open Recent | 3 | 3 |
| `MenubarAppearanceMenu` | View > Appearance | 25 | 24 |
| `MenubarLayoutMenu` | View > Editor Layout | 17 | 17 |
| `MenubarSwitchEditorMenu` | Go > Switch Editor | 10 | 10 |
| `MenubarSwitchGroupMenu` | Go > Switch Group | 11 | 11 |
| `MenubarNewBreakpointMenu` | Run > New Breakpoint | 7 | 7 |
| `MenubarShare` | File > Share | 1 (plus 1 from the `github` extension) | 1 |
| `MenubarHomeMenu` | web-only home indicator menu | 0 | 0 |
| `MenubarCopy` | Edit > Copy As | 0 | 0 |

`MenubarHomeMenu` and `MenubarCopy` have no built-in core items. They only fill from extensions through the `menus` contribution point keys `menuBar/home` (proposed API `contribMenuBarHome`, web only) and `menuBar/edit/copy` (`workbench/services/actions/common/menusExtensionPoint.ts:133-144`). `MenubarShare` has the key `file/share` (proposed API `contribShareMenu`, `menusExtensionPoint.ts:423`).

I searched every `package.json` under `V\extensions` for `menuBar/` and `file/share`. None of the six in-scope built-in extensions (git, markdown-language-features, merge-conflict, simple-browser, search-result, references-view) targets a menu bar id. The only manifest that does is the built-in `github` extension (not in scope), which adds `github.copyVscodeDevLinkFile` to `file/share` (`V\extensions\github\package.json:132-137`, group `0_vscode@0`, when `github.hasGitHubRepo && remoteName != 'codespaces'`). That makes File > Share hold 2 items in V when you count extensions. The Cursor copy of the `github` extension (`...\resources\app\extensions\github`) is a 2.6 KB stub with no menu contributions, so Cursor has 1 (Export Profile).

`MenubarTerminalSuggestStatusMenu` is not a menu bar menu (it is the terminal suggest status row) and is left out. The Edit > Copy As row in the Edit table is the submenu entry that points at `MenubarCopy`.

The View menu rows for Explorer, Search, Source Control, Run, Extensions, Testing, Chat, Problems, Output, Debug Console and Terminal are not registered in one place. Each view container carries an `openCommandActionDescriptor` and `workbench/services/views/browser/viewsService.ts:457` turns it into a menu item whose group comes from the container's default location (sidebar `3_sidebar`, secondary side bar `4_auxbar`, panel `5_panel`). The `.ts` file named in those rows is where the descriptor lives.

## Menu bar tables

### MenubarMainMenu - Top-level bar (the strip of menu titles)

V items: 9. Present in Cursor: 9. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
|  | 1 | &&File | `(submenu) MenubarFileMenu` | workbench/browser/parts/titlebar/menubar.contribution.ts:10 | both |  |
|  | 2 | &&Edit | `(submenu) MenubarEditMenu` | workbench/browser/parts/titlebar/menubar.contribution.ts:20 | both |  |
|  | 3 | &&Selection | `(submenu) MenubarSelectionMenu` | workbench/browser/parts/titlebar/menubar.contribution.ts:30 | both |  |
|  | 4 | &&View | `(submenu) MenubarViewMenu` | workbench/browser/parts/titlebar/menubar.contribution.ts:40 | both |  |
|  | 5 | &&Go | `(submenu) MenubarGoMenu` | workbench/browser/parts/titlebar/menubar.contribution.ts:50 | both (Cursor title is a runtime value) |  |
|  | 6 | &&Run | `(submenu) MenubarDebugMenu` | workbench/contrib/debug/browser/debug.contribution.ts:261 | both | IsSessionsWindowContext.negate() |
|  | 7 | &&Terminal | `(submenu) MenubarTerminalMenu` | workbench/browser/parts/titlebar/menubar.contribution.ts:60 | both |  |
|  | 8 | &&Help | `(submenu) MenubarHelpMenu` | workbench/browser/parts/titlebar/menubar.contribution.ts:70 | both |  |
|  | 9 | &&Preferences | `(submenu) MenubarPreferencesMenu` | workbench/browser/parts/titlebar/menubar.contribution.ts:80 | both | IsMacNativeContext |

### MenubarFileMenu - File

V items: 27. Present in Cursor: 26. Cursor only: 1.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_new | 1 | &&New Text File | `workbench.action.files.newUntitledFile` | workbench/contrib/files/browser/fileActions.contribution.ts:678 | both |  |
| 1_new | 2 | New File... | `welcome.showNewFileEntries` | workbench/contrib/welcomeViews/common/newFile.contribution.ts:40 | vscode only (no MenubarFileMenu entry; welcome.showNewFileEntries still used by the welcome page) | IsSessionsWindowContext.negate() |
| 1_new | 3 | New &&Window | `workbench.action.newWindow` | workbench/browser/actions/windowActions.ts:430 | both |  |
| 1_new | 4 | New Window with Profile | `(submenu) OpenProfileMenu (local const)` | workbench/contrib/userDataProfile/browser/userDataProfile.ts:201 | both (title key 11531, group 1_new, order 4) |  |
| 2_open | 1 | &&Open File... | `workbench.action.files.openFile` | workbench/browser/actions/workspaceActions.ts:339 | both | IsMacNativeContext.toNegated() |
| 2_open | 1 | &&Open... | `workbench.action.files.openFileFolder` | workbench/browser/actions/workspaceActions.ts:369 | both | ContextKeyExpr.and(IsMacNativeContext, OpenFolderWorkspaceSupportContext) |
| 2_open | 2 | Open &&Folder... | `workbench.action.files.openFolder` | workbench/browser/actions/workspaceActions.ts:349 | both | OpenFolderWorkspaceSupportContext |
| 2_open | 2 | Open &&Folder... | `workbench.action.files.openFolderViaWorkspace` | workbench/browser/actions/workspaceActions.ts:359 | both | ContextKeyExpr.and(OpenFolderWorkspaceSupportContext.toNegated(), WorkbenchStateContext.is |
| 2_open | 3 | Open Wor&&kspace from File... | `workbench.action.openWorkspace` | workbench/browser/actions/workspaceActions.ts:379 | both (Cursor group/order 2_open/4) | EnterMultiRootWorkspaceSupportContext |
| 2_open | 4 | Open &&Recent | `(submenu) MenubarRecentMenu` | workbench/browser/actions/windowActions.ts:515 | both (Cursor group/order 2_open/5) |  |
| 3_workspace | 1 | A&&dd Folder to Workspace... | `addRootFolder` | workbench/browser/actions/workspaceActions.ts:389 | both | ContextKeyExpr.and(ContextKeyExpr.or(EnterMultiRootWorkspaceSupportContext, WorkbenchState |
| 3_workspace | 2 | Save Workspace As... | `workbench.action.saveWorkspaceAs` | workbench/browser/actions/workspaceActions.ts:399 | both | ContextKeyExpr.and(EnterMultiRootWorkspaceSupportContext, IsSessionsWindowContext.negate() |
| 3_workspace | 3 | Duplicate Workspace | `workbench.action.duplicateWorkspaceInNewWindow` | workbench/browser/actions/workspaceActions.ts:409 | both | ContextKeyExpr.and(EnterMultiRootWorkspaceSupportContext, IsSessionsWindowContext.negate() |
| 4_save | 1 | &&Save | `workbench.action.files.save` | workbench/contrib/files/browser/fileActions.contribution.ts:687 | both |  |
| 4_save | 2 | Save &&As... | `workbench.action.files.saveAs` | workbench/contrib/files/browser/fileActions.contribution.ts:697 | both |  |
| 4_save | 3 | Save A&&ll | `saveAll` | workbench/contrib/files/browser/fileActions.contribution.ts:707 | both |  |
| 5_autosave | 1 | A&&uto Save | `workbench.action.toggleAutoSave` | workbench/contrib/files/browser/fileActions.contribution.ts:717 | both |  |
| 5_autosave | 2 | &&Preferences | `(submenu) MenubarPreferencesMenu` | workbench/contrib/preferences/browser/preferences.contribution.ts:1532 | both | IsMacNativeContext.toNegated() // on macOS native the preferences menu is separate under t |
| 6_close | 1 | Re&&vert File | `workbench.action.files.revert` | workbench/contrib/files/browser/fileActions.contribution.ts:727 | both |  |
| 6_close | 2 | &&Close Editor | `workbench.action.closeActiveEditor` | workbench/contrib/files/browser/fileActions.contribution.ts:742 | both |  |
| 6_close | 3 | Close &&Folder | `workbench.action.closeFolder` | workbench/browser/actions/workspaceActions.ts:419 | both | ContextKeyExpr.and(WorkbenchStateContext.isEqualTo('folder'), EmptyWorkspaceSupportContext |
| 6_close | 3 | Close &&Workspace | `workbench.action.closeFolder` | workbench/browser/actions/workspaceActions.ts:429 | both | ContextKeyExpr.and(WorkbenchStateContext.isEqualTo('workspace'), EmptyWorkspaceSupportCont |
| 6_close | 3.5 | Close Re&&mote Connection | `workbench.action.remote.close` | workbench/contrib/remote/browser/remoteIndicator.ts:222 | both | IsSessionsWindowContext.negate() |
| 6_close | 4 | Clos&&e Window | `workbench.action.closeWindow` | workbench/electron-browser/actions/windowActions.ts:55 | both |  |
| 45_share | 1 | Share | `(submenu) MenubarShare` | workbench/browser/parts/editor/editor.contribution.ts:729 | both | IsSessionsWindowContext.negate() |
| z_ConfirmClose | 1 | Confirm Before Close | `workbench.action.toggleConfirmBeforeClose` | workbench/browser/actions/windowActions.ts:504 | both | IsWebContext |
| z_Exit | 1 | E&&xit | `workbench.action.quit` | workbench/electron-browser/desktop.contribution.ts:129 | both | IsMacContext.toNegated() |
| 1_new | 3.5 | New Agents Window / Switch to Agents Window | `cursor.openOrFocusGlassWindow` | (not in V) | cursor only - Agents Window, out of scope |  |

### MenubarEditMenu - Edit

V items: 13. Present in Cursor: 13. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_do | 1 | &&Undo | `undo` | editor/browser/editorExtensions.ts:654 | both |  |
| 1_do | 2 | &&Redo | `redo` | editor/browser/editorExtensions.ts:683 | both |  |
| 2_ccp | 1 | Cu&&t | `editor.action.clipboardCutAction` | editor/contrib/clipboard/browser/clipboard.ts:54 | both |  |
| 2_ccp | 2 | &&Copy | `editor.action.clipboardCopyAction` | editor/contrib/clipboard/browser/clipboard.ts:91 | both |  |
| 2_ccp | 3 | Copy As | `(submenu) MenubarCopy` | editor/contrib/clipboard/browser/clipboard.ts:113 | both |  |
| 2_ccp | 4 | &&Paste | `editor.action.clipboardPasteAction` | editor/contrib/clipboard/browser/clipboard.ts:132 | both |  |
| 3_find | 1 | &&Find | `actions.find` | editor/contrib/find/browser/findController.ts:576 | both |  |
| 3_find | 2 | &&Replace | `editor.action.startFindReplaceAction` | editor/contrib/find/browser/findController.ts:1015 | both |  |
| 4_find_global | 1 | Find &&in Files | `workbench.action.findInFiles` | workbench/contrib/search/browser/searchActionsFind.ts:244 | both | IsSessionsWindowContext.negate() |
| 4_find_global | 2 | Replace in Files | `workbench.action.replaceInFiles` | workbench/contrib/search/browser/searchActionsNav.ts:409 | both | IsSessionsWindowContext.negate() |
| 5_insert | 1 | &&Toggle Line Comment | `editor.action.commentLine` | editor/contrib/comment/browser/comment.ts:93 | both |  |
| 5_insert | 2 | Toggle &&Block Comment | `editor.action.blockComment` | editor/contrib/comment/browser/comment.ts:149 | both |  |
| 5_insert | 3 | Emmet: E&&xpand Abbreviation | `editor.emmet.action.expandAbbreviation` | workbench/contrib/emmet/browser/actions/expandAbbreviation.ts:33 | both | IsSessionsWindowContext.negate() |

### MenubarSelectionMenu - Selection

V items: 17. Present in Cursor: 17. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_basic | 1 | &&Select All | `editor.action.selectAll` | editor/browser/editorExtensions.ts:711 | both |  |
| 1_basic | 2 | &&Expand Selection | `editor.action.smartSelect.expand` | editor/contrib/smartSelect/browser/smartSelect.ts:166 | both |  |
| 1_basic | 3 | &&Shrink Selection | `editor.action.smartSelect.shrink` | editor/contrib/smartSelect/browser/smartSelect.ts:194 | both |  |
| 2_line | 1 | &&Copy Line Up | `editor.action.copyLinesUpAction` | editor/contrib/linesOperations/browser/linesOperations.ts:91 | both |  |
| 2_line | 2 | Co&&py Line Down | `editor.action.copyLinesDownAction` | editor/contrib/linesOperations/browser/linesOperations.ts:114 | both |  |
| 2_line | 3 | Mo&&ve Line Up | `editor.action.moveLinesUpAction` | editor/contrib/linesOperations/browser/linesOperations.ts:206 | both |  |
| 2_line | 4 | Move &&Line Down | `editor.action.moveLinesDownAction` | editor/contrib/linesOperations/browser/linesOperations.ts:229 | both |  |
| 2_line | 5 | &&Duplicate Selection | `editor.action.duplicateSelection` | editor/contrib/linesOperations/browser/linesOperations.ts:132 | both |  |
| 3_multi | 2 | &&Add Cursor Above | `editor.action.insertCursorAbove` | editor/contrib/multicursor/browser/multicursor.ts:63 | both |  |
| 3_multi | 3 | A&&dd Cursor Below | `editor.action.insertCursorBelow` | editor/contrib/multicursor/browser/multicursor.ts:115 | both |  |
| 3_multi | 4 | Add C&&ursors to Line Ends | `editor.action.insertCursorAtEndOfEachLineSelected` | editor/contrib/multicursor/browser/multicursor.ts:163 | both |  |
| 3_multi | 5 | Add &&Next Occurrence | `editor.action.addSelectionToNextFindMatch` | editor/contrib/multicursor/browser/multicursor.ts:702 | both |  |
| 3_multi | 6 | Add P&&revious Occurrence | `editor.action.addSelectionToPreviousFindMatch` | editor/contrib/multicursor/browser/multicursor.ts:721 | both |  |
| 3_multi | 7 | Select All &&Occurrences | `editor.action.selectHighlights` | editor/contrib/multicursor/browser/multicursor.ts:776 | both |  |
| 4_config | 1 | Switch to Alt+Click for Multi-Cursor | `workbench.action.toggleMultiCursorModifier` | workbench/contrib/codeEditor/browser/toggleMultiCursorModifier.ts:73 | both | multiCursorModifier.isEqualTo('ctrlCmd') |
| 4_config | 1 | Switch to Cmd+Click for Multi-Cursor | `workbench.action.toggleMultiCursorModifier` | workbench/contrib/codeEditor/browser/toggleMultiCursorModifier.ts:82 | both (Cursor title is a runtime value) | multiCursorModifier.isEqualTo('altKey') |
| 4_config | 2 | Column &&Selection Mode | `editor.action.toggleColumnSelection` | workbench/contrib/codeEditor/browser/toggleColumnSelection.ts:32 | both |  |

### MenubarViewMenu - View

V items: 17. Present in Cursor: 15. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_open | 1 | &&Command Palette... | `workbench.action.showCommands` | workbench/contrib/quickaccess/browser/quickAccess.contribution.ts:55 | both |  |
| 1_open | 2 | &&Open View... | `workbench.action.openView` | workbench/contrib/quickaccess/browser/quickAccess.contribution.ts:73 | both |  |
| 2_appearance | 1 | &&Appearance | `(submenu) MenubarAppearanceMenu` | workbench/browser/actions/layoutActions.ts:276 | both | IsSessionsWindowContext.negate() |
| 2_appearance | 2 | Editor &&Layout | `(submenu) MenubarLayoutMenu` | workbench/browser/parts/editor/editor.contribution.ts:738 | both | IsSessionsWindowContext.negate() |
| 3_sidebar | 0 | &&Explorer | `workbench.view.explorer` | workbench/contrib/files/browser/explorerViewlet.ts (menu item generated in workbench/services/views/browser/viewsService.ts:457) | both |  |
| 3_sidebar | 1 | &&Search | `workbench.view.search` | workbench/contrib/search/browser/search.contribution.ts (menu item generated in workbench/services/views/browser/viewsService.ts:457) | both |  |
| 3_sidebar | 2 | Source &&Control | `workbench.view.scm` | workbench/contrib/scm/browser/scm.contribution.ts (menu item generated in workbench/services/views/browser/viewsService.ts:457) | both |  |
| 3_sidebar | 3 | &&Run | `workbench.view.debug` | workbench/contrib/debug/browser/debug.contribution.ts (menu item generated in workbench/services/views/browser/viewsService.ts:457) | both |  |
| 3_sidebar | 4 | E&&xtensions | `workbench.view.extensions` | workbench/contrib/extensions/browser/extensions.contribution.ts (menu item generated in workbench/services/views/browser/viewsService.ts:457) | both |  |
| 3_sidebar | 4 | T&&esting | `workbench.view.extension.test` | workbench/contrib/testing/browser/testing.contribution.ts (menu item generated in workbench/services/views/browser/viewsService.ts:457) | both |  |
| 4_auxbar | 1 | &&Chat | `workbench.panel.chat` | workbench/contrib/chat/browser/chatParticipant.contribution.ts (menu item generated in workbench/services/views/browser/viewsService.ts:457) | vscode only (no such view entry in bundle) |  |
| 4_auxbar | 2 | &&Browser | `workbench.action.browser.openOrList` | workbench/contrib/browserView/electron-browser/features/browserTabManagementFeatures.ts:520 | vscode only (not found in bundle) |  |
| 5_panel | 0 | &&Problems | `workbench.actions.view.problems` | workbench/contrib/markers/browser/markers.contribution.ts (menu item generated in workbench/services/views/browser/viewsService.ts:457) | both |  |
| 5_panel | 1 | &&Output | `workbench.action.output.toggleOutput` | workbench/contrib/output/browser/output.contribution.ts (menu item generated in workbench/services/views/browser/viewsService.ts:457) | both |  |
| 5_panel | 2 | De&&bug Console | `workbench.debug.action.toggleRepl` | workbench/contrib/debug/browser/debug.contribution.ts (menu item generated in workbench/services/views/browser/viewsService.ts:457) | both |  |
| 5_panel | 3 | &&Terminal | `workbench.action.terminal.toggleTerminal` | workbench/contrib/terminal/browser/terminal.contribution.ts (menu item generated in workbench/services/views/browser/viewsService.ts:457) | both |  |
| 6_editor | 1 | &&Word Wrap | `editor.action.toggleWordWrap` | workbench/contrib/codeEditor/browser/toggleWordWrap.ts:337 | both |  |

### MenubarGoMenu - Go

V items: 19. Present in Cursor: 19. Cursor only: 2.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_history_nav | 1 | &&Back | `workbench.action.navigateBack` | workbench/browser/parts/editor/editorActions.ts:1478 | both |  |
| 1_history_nav | 2 | &&Forward | `workbench.action.navigateForward` | workbench/browser/parts/editor/editorActions.ts:1443 | both |  |
| 1_history_nav | 3 | &&Last Edit Location | `workbench.action.navigateToLastEditLocation` | workbench/browser/parts/editor/editor.contribution.ts:942 | both |  |
| 2_editor_nav | 1 | Switch &&Editor | `(submenu) MenubarSwitchEditorMenu` | workbench/browser/parts/editor/editor.contribution.ts:1046 | both |  |
| 2_editor_nav | 2 | Switch &&Group | `(submenu) MenubarSwitchGroupMenu` | workbench/browser/parts/editor/editor.contribution.ts:1162 | both |  |
| 3_global_nav | 1 | Go to &&File... | `workbench.action.quickOpen` | workbench/contrib/files/browser/fileActions.contribution.ts:754 | both |  |
| 3_global_nav | 2 | Go to Symbol in &&Workspace... | `workbench.action.showAllSymbols` | workbench/contrib/search/browser/searchActionsSymbol.ts:35 | both |  |
| 4_symbol_nav | 1 | Go to &&Symbol in Editor... | `workbench.action.gotoSymbol` | workbench/contrib/codeEditor/browser/quickaccess/gotoSymbolQuickAccess.ts:294 | both |  |
| 4_symbol_nav | 2 | Go to &&Definition | `editor.action.revealDefinition` | editor/contrib/gotoSymbol/browser/goToCommands.ts:304 | both (Cursor title is a runtime value) |  |
| 4_symbol_nav | 3 | Go to &&Declaration | `editor.action.revealDeclaration` | editor/contrib/gotoSymbol/browser/goToCommands.ts:425 | both (Cursor title is a runtime value) |  |
| 4_symbol_nav | 3 | Go to &&Type Definition | `editor.action.goToTypeDefinition` | editor/contrib/gotoSymbol/browser/goToCommands.ts:514 | both (Cursor title is a runtime value) |  |
| 4_symbol_nav | 4 | Go to &&Implementations | `editor.action.goToImplementation` | editor/contrib/gotoSymbol/browser/goToCommands.ts:600 | both (Cursor title is a runtime value) |  |
| 4_symbol_nav | 5 | Go to &&References | `editor.action.goToReferences` | editor/contrib/gotoSymbol/browser/goToCommands.ts:689 | both (Cursor title is a runtime value) |  |
| 5_infile_nav | 1 | Go to &&Line/Column... | `workbench.action.gotoLine` | workbench/contrib/quickaccess/browser/quickAccess.contribution.ts:82 | both |  |
| 5_infile_nav | 2 | Go to &&Bracket | `editor.action.jumpToBracket` | editor/contrib/bracketMatching/browser/bracketMatching.ts:412 | both |  |
| 6_problem_nav | 1 | Next &&Problem | `editor.action.marker.nextInFiles` | editor/contrib/gotoError/browser/gotoError.ts:253 | both |  |
| 6_problem_nav | 2 | Previous &&Problem | `editor.action.marker.prevInFiles` | editor/contrib/gotoError/browser/gotoError.ts:275 | both |  |
| 7_change_nav | 1 | Next &&Change | `editor.action.dirtydiff.next` | workbench/contrib/scm/browser/quickDiffWidget.ts:915 | both |  |
| 7_change_nav | 2 | Previous &&Change | `editor.action.dirtydiff.previous` | workbench/contrib/scm/browser/quickDiffWidget.ts:924 | both |  |
| 4_symbol_nav | 5 | Add Symbol to Current Chat | `editor.action.addSymbolToChat` | (not in V) | cursor only - chat, out of scope |  |
| 4_symbol_nav | 6 | Add Symbol to New Chat | `editor.action.addSymbolToNewChat` | (not in V) | cursor only - chat, out of scope |  |

### MenubarDebugMenu - Run (the id is MenubarDebugMenu, there is no MenubarRunMenu)

V items: 16. Present in Cursor: 16. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_debug | 1 | &&Start Debugging | `workbench.action.debug.start` | workbench/contrib/debug/browser/debug.contribution.ts:271 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 1_debug | 2 | Run &&Without Debugging | `workbench.action.debug.run` | workbench/contrib/debug/browser/debug.contribution.ts:281 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 1_debug | 3 | &&Stop Debugging | `workbench.action.debug.stop` | workbench/contrib/debug/browser/debug.contribution.ts:291 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 1_debug | 4 | &&Restart Debugging | `workbench.action.debug.restart` | workbench/contrib/debug/browser/debug.contribution.ts:302 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 2_configuration | 1 | Open &&Configurations | `workbench.action.debug.configure` | workbench/contrib/debug/browser/debugViewlet.ts:221 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 2_configuration | 2 | A&&dd Configuration... | `debug.addConfiguration` | workbench/contrib/debug/browser/debug.contribution.ts:315 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 3_step | 1 | Step &&Over | `workbench.action.debug.stepOver` | workbench/contrib/debug/browser/debug.contribution.ts:326 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 3_step | 2 | Step &&Into | `workbench.action.debug.stepInto` | workbench/contrib/debug/browser/debug.contribution.ts:337 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 3_step | 3 | Step O&&ut | `workbench.action.debug.stepOut` | workbench/contrib/debug/browser/debug.contribution.ts:348 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 3_step | 4 | &&Continue | `workbench.action.debug.continue` | workbench/contrib/debug/browser/debug.contribution.ts:359 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 4_new_breakpoint | 1 | Toggle &&Breakpoint | `editor.debug.action.toggleBreakpoint` | workbench/contrib/debug/browser/debugEditorActions.ts:54 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 4_new_breakpoint | 2 | &&New Breakpoint | `(submenu) MenubarNewBreakpointMenu` | workbench/contrib/debug/browser/debug.contribution.ts:382 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 5_breakpoints | 1 | &&Enable All Breakpoints | `workbench.debug.viewlet.action.enableAllBreakpoints` | workbench/contrib/debug/browser/breakpointsView.ts:2259 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 5_breakpoints | 2 | Disable A&&ll Breakpoints | `workbench.debug.viewlet.action.disableAllBreakpoints` | workbench/contrib/debug/browser/breakpointsView.ts:2289 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 5_breakpoints | 3 | Remove &&All Breakpoints | `workbench.debug.viewlet.action.removeAllBreakpoints` | workbench/contrib/debug/browser/breakpointsView.ts:2226 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| z_install | 1 | &&Install Additional Debuggers... | `debug.installAdditionalDebuggers` | workbench/contrib/debug/browser/debug.contribution.ts:415 | both |  |

### MenubarTerminalMenu - Terminal

V items: 12. Present in Cursor: 11. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_create | 1 | &&New Terminal | `workbench.action.terminal.new` | workbench/contrib/terminal/browser/terminalMenus.ts:45 | both |  |
| 1_create | 2 | New Terminal &&Window | `workbench.action.terminal.newInNewWindow` | workbench/contrib/terminal/browser/terminalMenus.ts:56 | vscode only (key miNewInNewWindow absent from Cursor terminalMenus strings; 4 items found) | TerminalContextKeys.processSupported |
| 1_create | 2 | &&Split Terminal | `workbench.action.terminal.split` | workbench/contrib/terminal/browser/terminalMenus.ts:69 | both | TerminalContextKeys.processSupported |
| 3_run | 1 | &&Run Task... | `workbench.action.tasks.runTask` | workbench/contrib/tasks/browser/task.contribution.ts:165 | both | TaskExecutionSupportedContext |
| 3_run | 2 | Run &&Build Task... | `workbench.action.tasks.build` | workbench/contrib/tasks/browser/task.contribution.ts:175 | both | TaskExecutionSupportedContext |
| 3_run | 3 | Run &&Active File | `workbench.action.terminal.runActiveFile` | workbench/contrib/terminal/browser/terminalMenus.ts:82 | both | TerminalContextKeys.processSupported |
| 3_run | 4 | Run &&Selected Text | `workbench.action.terminal.runSelectedText` | workbench/contrib/terminal/browser/terminalMenus.ts:94 | both | TerminalContextKeys.processSupported |
| 5_manage | 1 | Show Runnin&&g Tasks... | `workbench.action.tasks.showTasks` | workbench/contrib/tasks/browser/task.contribution.ts:186 | both | TaskExecutionSupportedContext |
| 5_manage | 2 | R&&estart Running Task... | `workbench.action.tasks.restartTask` | workbench/contrib/tasks/browser/task.contribution.ts:197 | both | TaskExecutionSupportedContext |
| 5_manage | 3 | &&Terminate Task... | `workbench.action.tasks.terminate` | workbench/contrib/tasks/browser/task.contribution.ts:208 | both | TaskExecutionSupportedContext |
| 7_configure | 1 | &&Configure Tasks... | `workbench.action.tasks.configureTaskRunner` | workbench/contrib/tasks/browser/task.contribution.ts:220 | both | TaskExecutionSupportedContext |
| 7_configure | 2 | Configure De&&fault Build Task... | `workbench.action.tasks.configureDefaultBuildTask` | workbench/contrib/tasks/browser/task.contribution.ts:230 | both | TaskExecutionSupportedContext |

### MenubarHelpMenu - Help

V items: 19. Present in Cursor: 7. Cursor only: 2.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_welcome | 1 | Welcome | `workbench.action.openWalkthrough` | workbench/contrib/welcomeGettingStarted/browser/gettingStarted.contribution.ts:48 | vscode only (Cursor registers workbench.action.openWalkthrough with an empty title and no menu) |  |
| 1_welcome | 2 | Show All Commands | `workbench.action.showCommands` | workbench/contrib/quickaccess/browser/quickAccess.contribution.ts:64 | both |  |
| 1_welcome | 3 | &&Documentation | `workbench.action.openDocumentationUrl` | workbench/browser/actions/helpActions.ts:139 | vscode only (not found in bundle) |  |
| 1_welcome | 3 | Open Walkthrough... | `welcome.showAllWalkthroughs` | workbench/contrib/welcomeGettingStarted/browser/gettingStarted.contribution.ts:208 | vscode only (not found in bundle) |  |
| 1_welcome | 3 | Editor Playgrou&&nd | `workbench.action.showInteractivePlayground` | workbench/contrib/welcomeWalkthrough/browser/walkThrough.contribution.ts:42 | vscode only (not found in bundle) |  |
| 1_welcome | 5 | Show &&Release Notes | `update.showCurrentReleaseNotes` | workbench/contrib/update/browser/update.contribution.ts:55 | both |  |
| 1_welcome | 6 | Get Started with Accessibility Features | `workbench.action.getStartedWithAccessibilityFeatures` | workbench/browser/actions/helpActions.ts:325 | vscode only (not found in bundle) | IsSessionsWindowContext.negate() |
| 1_welcome | 7 | Ask @vscode | `workbench.action.askVScode` | workbench/browser/actions/helpActions.ts:358 | vscode only (not found in bundle) | ContextKeyExpr.and(ContextKeyExpr.equals('chatSetupHidden', false), ContextKeyExpr.equals( |
| 2_reference | 1 | &&Keyboard Shortcuts Reference | `workbench.action.keybindingsReference` | workbench/browser/actions/helpActions.ts:42 | vscode only (not found in bundle) |  |
| 2_reference | 2 | &&Video Tutorials | `workbench.action.openVideoTutorialsUrl` | workbench/browser/actions/helpActions.ts:75 | vscode only (not found in bundle) |  |
| 2_reference | 3 | Tips and Tri&&cks | `workbench.action.openTipsAndTricksUrl` | workbench/browser/actions/helpActions.ts:107 | vscode only (not found in bundle) |  |
| 3_feedback | 1 | &&Join Us on YouTube | `workbench.action.openYouTubeUrl` | workbench/browser/actions/helpActions.ts:194 | vscode only (not found in bundle) |  |
| 3_feedback | 2 | &&Search Feature Requests | `workbench.action.openRequestFeatureUrl` | workbench/browser/actions/helpActions.ts:226 | vscode only (not found in bundle) |  |
| 3_feedback | 3 | Report &&Issue | `workbench.action.openIssueReporter` | workbench/contrib/issue/common/issue.contribution.ts:123 | both |  |
| 4_legal | 1 | View &&License | `workbench.action.openLicenseUrl` | workbench/browser/actions/helpActions.ts:258 | both |  |
| 4_legal | 2 | Privac&&y Statement | `workbench.action.openPrivacyStatementUrl` | workbench/browser/actions/helpActions.ts:296 | vscode only (not found in bundle) |  |
| 5_tools | 1 | Toggle Developer Tools | `workbench.action.toggleDevTools` | workbench/electron-browser/actions/developerActions.ts:38 | both |  |
| 5_tools | 2 | Open &&Process Explorer | `workbench.action.openProcessExplorer` | workbench/contrib/processExplorer/browser/processExplorer.contribution.ts:163 | both | supported |
| z_about | 1 | &&About | `workbench.action.showAboutDialog` | workbench/browser/actions/windowActions.ts:399 | both | IsMacNativeContext.toNegated() |
| 3_feedback | 4 | Give &&Feedback... | `cursor.giveFeedback` | (not in V) | cursor only |  |
| 5_tools | 2 | Open &&Extension Monitor | `(minified, not resolved)` | (not in V) | cursor only - extension tooling, out of scope |  |

### MenubarPreferencesMenu - Preferences (macOS native app menu; also a File submenu on Windows and Linux)

V items: 12. Present in Cursor: 9. Cursor only: 2.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 2_configuration | 1 | Profile ({0}) (dynamic title) | `(submenu) ProfilesMenu (local const)` | workbench/contrib/userDataProfile/browser/userDataProfile.ts:189 | both (appendMenuItem with title getter) | ContextKeyExpr.and(HAS_PROFILES_CONTEXT, IsSessionsWindowContext.negate()) |
| 2_configuration | 1 | &&Profiles | `workbench.profiles.actions.manageProfiles` | workbench/contrib/userDataProfile/browser/userDataProfile.ts:377 | both | ContextKeyExpr.and(HAS_PROFILES_CONTEXT.negate(), IsSessionsWindowContext.negate()) |
| 2_configuration | 2 | &&Settings | `workbench.action.openSettings` | workbench/contrib/preferences/browser/preferences.contribution.ts:245 | both (Cursor shows it only when isGlass is true; IDE window shows VS Code Settings instead) |  |
| 2_configuration | 3 | &&Extensions | `workbench.view.extensions` | workbench/contrib/extensions/browser/extensions.contribution.ts:667 | both | IsSessionsWindowContext.negate() |
| 2_configuration | 4 | Keyboard Shortcuts | `workbench.action.openGlobalKeybindings` | workbench/contrib/preferences/browser/preferences.contribution.ts:922 | both |  |
| 2_configuration | 5 | Configure Snippets | `workbench.action.openSnippets` | workbench/contrib/snippets/browser/commands/configureSnippets.ts:256 | both |  |
| 2_configuration | 6 | Tasks | `workbench.action.tasks.openUserTasks` | workbench/contrib/tasks/browser/task.contribution.ts:372 | both (Cursor title is a runtime value) | TaskExecutionSupportedContext |
| 2_configuration | 7 | &&Themes | `(submenu) ThemesSubMenu (local const; Color Theme, File Icon Theme, Product Icon Theme)` | workbench/contrib/themes/browser/themes.contribution.ts:909 | both |  |
| 3_configuration | 2 | Backup and Sync Settings... | `workbench.userDataSync.actions.turnOn` | workbench/contrib/userDataSync/browser/userDataSync.ts:779 | vscode only (not found in bundle) |  |
| 3_configuration | 2 | Show Conflicts (dynamic title) | `workbench.userDataSync.actions.showConflicts` | workbench/contrib/userDataSync/browser/userDataSync.ts:900 | vscode only (not found in bundle) | CONTEXT_HAS_CONFLICTS |
| 3_configuration | 2 | Settings Sync is On | `workbench.userDataSync.actions.manage` | workbench/contrib/userDataSync/browser/userDataSync.ts:929 | vscode only (not found in bundle) |  |
| 3_settings | 1 | &&Online Services Settings | `settings.filterByOnline` | workbench/contrib/preferences/browser/preferences.contribution.ts:480 | both |  |
| 2_configuration | 1.5 | Cursor Settings | `aiSettings.action.open` | (not in V) | cursor only - keys Ctrl/Cmd+Shift+J, and Ctrl/Cmd+, when isGlass is false |  |
| 2_configuration | 2 | &&VS Code Settings | `(minified, not resolved)` | (not in V) | cursor only - when isGlass is false |  |

### MenubarRecentMenu - File > Open Recent submenu (the recent folders, workspaces and files rows are generated at runtime by menubarControl.ts and are not registered items)

V items: 3. Present in Cursor: 3. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_editor | 1 | &&Reopen Closed Editor | `workbench.action.reopenClosedEditor` | workbench/browser/parts/editor/editor.contribution.ts:710 | both |  |
| y_more | 1 | &&More... | `workbench.action.openRecent` | workbench/browser/actions/windowActions.ts:299 | both |  |
| z_clear | 1 | &&Clear Recently Opened... | `workbench.action.clearRecentFiles` | workbench/browser/parts/editor/editor.contribution.ts:720 | both |  |

### MenubarAppearanceMenu - View > Appearance submenu

V items: 25. Present in Cursor: 23. Cursor only: 1.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_toggle_view | 1 | &&Full Screen | `workbench.action.toggleFullScreen` | workbench/browser/actions/windowActions.ts:348 | both |  |
| 1_toggle_view | 2 | Zen Mode | `workbench.action.toggleZenMode` | workbench/browser/workbench.zenMode.contribution.ts:116 | both |  |
| 1_toggle_view | 3 | &&Centered Layout | `workbench.action.toggleCenteredLayout` | workbench/browser/actions/layoutActions.ts:80 | both | IsSessionsWindowContext.negate() |
| 2_workbench_layout | 0 | Menu &&Bar | `workbench.action.toggleMenuBar` | workbench/browser/actions/layoutActions.ts:745 | both | IsSessionsWindowContext.negate() |
| 2_workbench_layout | 1 | &&Primary Side Bar | `workbench.action.toggleSidebarVisibility` | workbench/browser/actions/layoutActions.ts:316 | both |  |
| 2_workbench_layout | 2 | &&Secondary Side Bar | `workbench.action.toggleAuxiliaryBar` | workbench/browser/parts/auxiliarybar/auxiliaryBarActions.ts:62 | both |  |
| 2_workbench_layout | 3 | S&&tatus Bar | `workbench.action.toggleStatusbarVisibility` | workbench/browser/actions/layoutActions.ts:413 | both | IsSessionsWindowContext.negate() |
| 2_workbench_layout | 5 | &&Panel | `workbench.action.togglePanel` | workbench/browser/parts/panel/panelActions.ts:54 | both |  |
| 2_workbench_layout | 6 | Custom Title Bar | `toggle.toggleCustomTitleBar` | workbench/browser/parts/titlebar/titlebarActions.ts:131 | vscode only (not found in bundle) | ContextKeyExpr.or( ContextKeyExpr.and( ContextKeyExpr.equals(TitleBarStyleContext.key, Tit |
| 3_workbench_layout_move | 2 | &&Move Primary Side Bar Right | `workbench.action.toggleSidebarPosition` | workbench/browser/actions/layoutActions.ts:232 | both | ContextKeyExpr.and(ContextKeyExpr.notEquals('config.workbench.sideBar.location', 'right'), |
| 3_workbench_layout_move | 2 | &&Move Primary Side Bar Left | `workbench.action.toggleSidebarPosition` | workbench/browser/actions/layoutActions.ts:242 | both | ContextKeyExpr.and(ContextKeyExpr.equals('config.workbench.sideBar.location', 'right'), Is |
| 3_workbench_layout_move | 2 | Activity Bar Position | `(submenu) ActivityBarPositionMenu` | workbench/browser/parts/activitybar/activitybarPart.ts:584 | vscode only (not found in bundle) | IsSessionsWindowContext.negate() |
| 3_workbench_layout_move | 4 | Panel Position | `(submenu) PanelPositionMenu` | workbench/browser/parts/panel/panelActions.ts:183 | both |  |
| 3_workbench_layout_move | 5 | Align Panel | `(submenu) PanelAlignmentMenu` | workbench/browser/parts/panel/panelActions.ts:218 | both |  |
| 3_workbench_layout_move | 10 | Tab Bar | `(submenu) EditorTabsBarShowTabsSubmenu` | workbench/browser/actions/layoutActions.ts:543 | both | ContextKeyExpr.and(InEditorZenModeContext.negate(), IsSessionsWindowContext.negate()) |
| 3_workbench_layout_move | 10 | Tab Bar | `(submenu) EditorTabsBarShowTabsZenModeSubmenu` | workbench/browser/workbench.zenMode.contribution.ts:88 | both | InEditorZenModeContext |
| 3_workbench_layout_move | 11 | Editor Actions Position | `(submenu) EditorActionsPositionSubmenu` | workbench/browser/actions/layoutActions.ts:653 | both | IsSessionsWindowContext.negate() |
| 4_editor | 1 | &&Minimap | `editor.action.toggleMinimap` | workbench/contrib/codeEditor/browser/toggleMinimap.ts:28 | both |  |
| 4_editor | 2 | &&Breadcrumbs | `breadcrumbs.toggle` | workbench/browser/parts/editor/breadcrumbsControl.ts:712 | both (Cursor label is Toggle Breadcrumbs, no toggled mnemonic title) |  |
| 4_editor | 3 | &&Sticky Scroll | `editor.action.toggleStickyScroll` | editor/contrib/stickyScroll/browser/stickyScrollActions.ts:38 | both |  |
| 4_editor | 4 | Render Whitespace | `(submenu) EditorRenderWhitespaceSubmenu` | workbench/contrib/codeEditor/browser/toggleRenderWhitespace.ts:141 | both |  |
| 4_editor | 5 | Render &&Control Characters | `editor.action.toggleRenderControlCharacter` | workbench/contrib/codeEditor/browser/toggleRenderControlCharacter.ts:28 | both |  |
| 5_zoom | 1 | &&Zoom In | `workbench.action.zoomIn` | workbench/electron-browser/actions/windowActions.ts:160 | both |  |
| 5_zoom | 2 | &&Zoom Out | `workbench.action.zoomOut` | workbench/electron-browser/actions/windowActions.ts:193 | both |  |
| 5_zoom | 3 | &&Reset Zoom | `workbench.action.zoomReset` | workbench/electron-browser/actions/windowActions.ts:221 | both |  |
| 1_toggle_view | 4 | Open Browser | `workbench.action.openBrowserEditor` | (not in V) | cursor only - when context key mHo (browser tab enabled) |  |

### MenubarLayoutMenu - View > Editor Layout submenu

V items: 17. Present in Cursor: 17. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_split | 1 | Split &&Up | `workbench.action.splitEditorUp` | workbench/browser/parts/editor/editor.contribution.ts:746 | both |  |
| 1_split | 2 | Split &&Down | `workbench.action.splitEditorDown` | workbench/browser/parts/editor/editor.contribution.ts:758 | both |  |
| 1_split | 3 | Split &&Left | `workbench.action.splitEditorLeft` | workbench/browser/parts/editor/editor.contribution.ts:770 | both |  |
| 1_split | 4 | Split &&Right | `workbench.action.splitEditorRight` | workbench/browser/parts/editor/editor.contribution.ts:782 | both |  |
| 2_split_in_group | 1 | Split in &&Group | `workbench.action.splitEditorInGroup` | workbench/browser/parts/editor/editor.contribution.ts:794 | both | ActiveEditorCanSplitInGroupContext |
| 2_split_in_group | 1 | Join in &&Group | `workbench.action.joinEditorInGroup` | workbench/browser/parts/editor/editor.contribution.ts:807 | both | SideBySideEditorActiveContext |
| 3_new_window | 1 | &&Move Editor into New Window | `workbench.action.moveEditorToNewWindow` | workbench/browser/parts/editor/editor.contribution.ts:820 | both |  |
| 3_new_window | 2 | &&Copy Editor into New Window | `workbench.action.copyEditorToNewWindow` | workbench/browser/parts/editor/editor.contribution.ts:832 | both |  |
| 4_layouts | 1 | &&Single | `workbench.action.editorLayoutSingle` | workbench/browser/parts/editor/editor.contribution.ts:844 | both |  |
| 4_layouts | 3 | &&Two Columns | `workbench.action.editorLayoutTwoColumns` | workbench/browser/parts/editor/editor.contribution.ts:856 | both |  |
| 4_layouts | 4 | T&&hree Columns | `workbench.action.editorLayoutThreeColumns` | workbench/browser/parts/editor/editor.contribution.ts:868 | both |  |
| 4_layouts | 5 | T&&wo Rows | `workbench.action.editorLayoutTwoRows` | workbench/browser/parts/editor/editor.contribution.ts:880 | both |  |
| 4_layouts | 6 | Three &&Rows | `workbench.action.editorLayoutThreeRows` | workbench/browser/parts/editor/editor.contribution.ts:892 | both |  |
| 4_layouts | 7 | &&Grid (2x2) | `workbench.action.editorLayoutTwoByTwoGrid` | workbench/browser/parts/editor/editor.contribution.ts:904 | both |  |
| 4_layouts | 8 | Two R&&ows Right | `workbench.action.editorLayoutTwoRowsRight` | workbench/browser/parts/editor/editor.contribution.ts:916 | both |  |
| 4_layouts | 9 | Two &&Columns Bottom | `workbench.action.editorLayoutTwoColumnsBottom` | workbench/browser/parts/editor/editor.contribution.ts:928 | both |  |
| 5_flip | 1 | Flip &&Layout | `workbench.action.toggleEditorGroupLayout` | workbench/contrib/files/browser/views/openEditorsView.ts:871 | both |  |

### MenubarSwitchEditorMenu - Go > Switch Editor submenu

V items: 10. Present in Cursor: 10. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_sideBySide | 1 | &&First Side in Editor | `workbench.action.focusFirstSideEditor` | workbench/browser/parts/editor/editor.contribution.ts:954 | both | ContextKeyExpr.or(SideBySideEditorActiveContext, TextCompareEditorActiveContext) |
| 1_sideBySide | 2 | &&Second Side in Editor | `workbench.action.focusSecondSideEditor` | workbench/browser/parts/editor/editor.contribution.ts:964 | both | ContextKeyExpr.or(SideBySideEditorActiveContext, TextCompareEditorActiveContext) |
| 2_any | 1 | &&Next Editor | `workbench.action.nextEditor` | workbench/browser/parts/editor/editor.contribution.ts:974 | both |  |
| 2_any | 2 | &&Previous Editor | `workbench.action.previousEditor` | workbench/browser/parts/editor/editor.contribution.ts:983 | both |  |
| 3_any_used | 1 | &&Next Used Editor | `workbench.action.openNextRecentlyUsedEditor` | workbench/browser/parts/editor/editor.contribution.ts:992 | both |  |
| 3_any_used | 2 | &&Previous Used Editor | `workbench.action.openPreviousRecentlyUsedEditor` | workbench/browser/parts/editor/editor.contribution.ts:1001 | both |  |
| 4_group | 1 | &&Next Editor in Group | `workbench.action.nextEditorInGroup` | workbench/browser/parts/editor/editor.contribution.ts:1010 | both |  |
| 4_group | 2 | &&Previous Editor in Group | `workbench.action.previousEditorInGroup` | workbench/browser/parts/editor/editor.contribution.ts:1019 | both |  |
| 5_group_used | 1 | &&Next Used Editor in Group | `workbench.action.openNextRecentlyUsedEditorInGroup` | workbench/browser/parts/editor/editor.contribution.ts:1028 | both |  |
| 5_group_used | 2 | &&Previous Used Editor in Group | `workbench.action.openPreviousRecentlyUsedEditorInGroup` | workbench/browser/parts/editor/editor.contribution.ts:1037 | both |  |

### MenubarSwitchGroupMenu - Go > Switch Group submenu

V items: 11. Present in Cursor: 11. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_focus_index | 1 | Group &&1 | `workbench.action.focusFirstEditorGroup` | workbench/browser/parts/editor/editor.contribution.ts:1054 | both |  |
| 1_focus_index | 2 | Group &&2 | `workbench.action.focusSecondEditorGroup` | workbench/browser/parts/editor/editor.contribution.ts:1063 | both |  |
| 1_focus_index | 3 | Group &&3 | `workbench.action.focusThirdEditorGroup` | workbench/browser/parts/editor/editor.contribution.ts:1072 | both |  |
| 1_focus_index | 4 | Group &&4 | `workbench.action.focusFourthEditorGroup` | workbench/browser/parts/editor/editor.contribution.ts:1082 | both |  |
| 1_focus_index | 5 | Group &&5 | `workbench.action.focusFifthEditorGroup` | workbench/browser/parts/editor/editor.contribution.ts:1092 | both |  |
| 2_next_prev | 1 | &&Next Group | `workbench.action.focusNextGroup` | workbench/browser/parts/editor/editor.contribution.ts:1102 | both |  |
| 2_next_prev | 2 | &&Previous Group | `workbench.action.focusPreviousGroup` | workbench/browser/parts/editor/editor.contribution.ts:1112 | both |  |
| 3_directional | 1 | Group &&Left | `workbench.action.focusLeftGroup` | workbench/browser/parts/editor/editor.contribution.ts:1122 | both |  |
| 3_directional | 2 | Group &&Right | `workbench.action.focusRightGroup` | workbench/browser/parts/editor/editor.contribution.ts:1132 | both |  |
| 3_directional | 3 | Group &&Above | `workbench.action.focusAboveGroup` | workbench/browser/parts/editor/editor.contribution.ts:1142 | both |  |
| 3_directional | 4 | Group &&Below | `workbench.action.focusBelowGroup` | workbench/browser/parts/editor/editor.contribution.ts:1152 | both |  |

### MenubarNewBreakpointMenu - Run > New Breakpoint submenu

V items: 7. Present in Cursor: 7. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
| 1_breakpoints | 1 | &&Conditional Breakpoint... | `editor.debug.action.conditionalBreakpoint` | workbench/contrib/debug/browser/debugEditorActions.ts:108 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 1_breakpoints | 1 | &&Edit Breakpoint | `editor.debug.action.editBreakpoint` | workbench/contrib/debug/browser/debugEditorActions.ts:194 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 1_breakpoints | 2 | Inline Breakp&&oint | `editor.debug.action.toggleInlineBreakpoint` | workbench/contrib/debug/browser/debug.contribution.ts:372 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 1_breakpoints | 3 | &&Function Breakpoint... | `workbench.debug.viewlet.action.addFunctionBreakpointAction` | workbench/contrib/debug/browser/breakpointsView.ts:1969 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 1_breakpoints | 4 | &&Data Breakpoint... | `workbench.debug.viewlet.action.addDataBreakpointOnAddress` | workbench/contrib/debug/browser/breakpointsView.ts:2125 | both | CONTEXT_SET_DATA_BREAKPOINT_BYTES_SUPPORTED |
| 1_breakpoints | 4 | &&Logpoint... | `editor.debug.action.addLogPoint` | workbench/contrib/debug/browser/debugEditorActions.ts:136 | both | CONTEXT_DEBUGGERS_AVAILABLE |
| 1_breakpoints | 4 | &&Triggered Breakpoint... | `editor.debug.action.triggerByBreakpoint` | workbench/contrib/debug/browser/debugEditorActions.ts:166 | both | CONTEXT_DEBUGGERS_AVAILABLE |

### MenubarShare - File > Share submenu

V items: 1. Present in Cursor: 1. Cursor only: 0.

| Group | Order | Label | Command id | Registered in V | Cursor | When (V) |
|-|-|-|-|-|-|-|
|  |  | Export Profile ({0})... | `workbench.profiles.actions.exportProfile` | workbench/contrib/userDataProfile/browser/userDataProfile.ts:426 | both (Cursor title is a runtime value) |  |

Extension rows for File > Share: `github.copyVscodeDevLinkFile` (Copy vscode.dev Link), from `V\extensions\github\package.json:132`. Not in Cursor.

### MenubarHomeMenu and MenubarCopy

No built-in items in V or in Cursor (see Totals). The key names are `menuBar/home` and `menuBar/edit/copy`.

## Submenus that are not `Menubar*` ids

These hang under View > Appearance, File and Preferences. They use ordinary menu ids, so they do not show up under a `Menubar` search. Cursor status is in the last column.

| Submenu (menu id) | Opened from | Items | Registered in V | Cursor |
|-|-|-|-|-|
| `ActivityBarPositionMenu` | Appearance > Activity Bar Position | Default, Top, Bottom, Hidden (`workbench.action.activityBarLocation.default/top/bottom/hide`) | `workbench/browser/parts/activitybar/activitybarPart.ts:492-570` | vscode only: the string `ActivityBarPositionMenu` has 0 hits in the Cursor bundle, although the four `workbench.action.activityBarLocation.*` commands are still there |
| `PanelPositionMenu` | Appearance > Panel Position | Top, Left, Bottom, Right (generated in a loop) | `workbench/browser/parts/panel/panelActions.ts:183-218` | both (submenu entry found, children not re-read in Cursor) |
| `PanelAlignmentMenu` | Appearance > Align Panel | Left, Center, Right, Justify (generated in a loop) | `workbench/browser/parts/panel/panelActions.ts:218-243` | both (submenu entry found, children not re-read in Cursor) |
| `EditorTabsBarShowTabsSubmenu` | Appearance > Tab Bar | Multiple Tabs, Single Tab, Hidden (`workbench.action.showMultipleEditorTabs`, `showEditorTab`, `hideEditorTabs`) | `workbench/browser/parts/editor/editor.contribution.ts:377-379` | both (submenu entry found, children not re-read in Cursor) |
| `EditorTabsBarShowTabsZenModeSubmenu` | Appearance > Tab Bar (in Zen Mode) | Multiple Tabs, Single Tab, Hidden (`workbench.action.zenShowMultipleEditorTabs`, `zenShowEditorTab`, `zenHideEditorTabs`) | `editor.contribution.ts:382-384` | both (submenu entry found, children not re-read in Cursor) |
| `EditorActionsPositionSubmenu` | Appearance > Editor Actions Position | Tab Bar, Title Bar, Hidden (`workbench.action.editorActionsDefault`, `editorActionsTitleBar`, `hideEditorActions`) | `editor.contribution.ts:387-389` | both (submenu entry found, children not re-read in Cursor) |
| `EditorRenderWhitespaceSubmenu` | Appearance > Render Whitespace | None, Boundary, Selection, Trailing, All (`editor.action.renderWhitespace.*`) | `workbench/contrib/codeEditor/browser/toggleRenderWhitespace.ts:25-97` | both (submenu entry found, children not re-read in Cursor) |
| `ThemesSubMenu` (local const) | Preferences > Themes and Manage > Themes | Color Theme (`workbench.action.selectTheme`), File Icon Theme (`workbench.action.selectIconTheme`), Product Icon Theme (`workbench.action.selectProductIconTheme`) | `workbench/contrib/themes/browser/themes.contribution.ts:885-935` | both (Themes entry present; children not re-read) |
| `ProfilesMenu` (local const) | Preferences > Profile | profile list, created at runtime | `workbench/contrib/userDataProfile/browser/userDataProfile.ts:160-200` | both (submenu present; children not read) |
| `OpenProfileMenu` (local const) | File > New Window with Profile | profile list, created at runtime | `userDataProfile.ts:201` | both (submenu present; children not read) |

## Runtime rows that are not registered items

- File > Open Recent: reopen closed editor, then up to 10 recent folders, workspaces and files, then More..., then Clear Recently Opened. The recent rows come from `menubarControl.ts:173-181` (`MAX_MENU_RECENT_ENTRIES = 10`) using the commands `openRecentFolder`, `openRecentWorkspace`, `openRecentFile` (`menubarControl.ts:240-250`).
- Help menu on Windows and Linux: right after About, the custom menu bar adds one update row built in `menubarControl.ts:380-420` and `440-455`. The ids are `update.check` (Check for Updates...), `update.checking`, `update.downloadNow`, `update.downloading`, `update.install`, `update.updating`, `update.cancelling`, `update.restart` (Restart to Update). Cursor has the same set minus `update.cancelling` (the Cursor strings stop at "Restart to &&Update"; the V string `cancellingUpdate` is not in Cursor).
- Web only: the home indicator menu appends `MenubarHomeMenu` rows to the File menu (`menubarControl.ts:590-660`).
- Top-level title strip: `menubarControl.ts` builds the strip from `MenubarMainMenu` and uses each submenu's `mnemonicTitle`. On macOS with the native menu, `src/vs/platform/menubar/electron-main/menubar.ts` builds the same strip as an OS menu and adds the app menu (About, Preferences, Services, Hide, Hide Others, Show All, Quit) and the Window menu (Minimize, Zoom, Bring All to Front, Switch Window..., New Tab, Show Previous Tab, Show Next Tab, Move Tab to New Window, Merge All Windows). The Cursor bundle has the same native strings (`vs/platform/menubar/electron-main/menubar`, 35 keys), so the structure is the same. I read this only on the source side and in the strings. I did not look at a running macOS menu.
- Focus command: `workbench.actions.menubar.focus`, label "Focus Application Menu" (`menubarControl.ts:300`). Same in Cursor (string `focusMenu`).

## What changed in Cursor, menu bar only

Details and evidence for each live in the Cursor research file for this section.

- Removed from Help: Welcome (command stays, menu entry and title removed), Open Walkthrough..., Editor Playground, Documentation, Keyboard Shortcuts Reference, Video Tutorials, Tips and Tricks, Join Us on YouTube, Search Feature Requests, Privacy Statement, Get Started with Accessibility Features, Ask @vscode.
- Removed from File: New File... and the second order shift (Open Workspace from File is order 4 and Open Recent is order 5, one higher than V).
- Removed from View: Chat and Browser (`workbench.action.browser.openOrList`).
- Removed from View > Appearance: Custom Title Bar toggle, Activity Bar Position submenu.
- Removed from Terminal: New Terminal Window.
- Removed from Preferences: Backup and Sync Settings..., Settings Sync is On, Show Conflicts (the whole `userDataSync` set is gone from the bundle: 0 hits for `userDataSync.actions`).
- Added: Preferences > Cursor Settings (`aiSettings.action.open`, order 1.5), Preferences > VS Code Settings, Help > Give Feedback... (`cursor.giveFeedback`), Help > Open Extension Monitor, View > Appearance > Open Browser (`workbench.action.openBrowserEditor`), Go > Add Symbol to Current Chat and Add Symbol to New Chat (`editor.action.addSymbolToChat`, `editor.action.addSymbolToNewChat`), File > New Agents Window / Switch to Agents Window (`cursor.openOrFocusGlassWindow`).
- Changed: the top-level Preferences title has no mnemonic in Cursor ("Preferences" instead of "&&Preferences"). Settings shows only when the context key `isGlass` is true (Agents window). In the IDE window you get Cursor Settings and VS Code Settings instead. Run, Selection and Go are hidden when `isGlass` is true.


## Other menus that draw on the title bar (V registrations)

These are the menus the title bar part reads (`workbench/browser/parts/titlebar/titlebarPart.ts:622,778,791`, `commandCenterControl.ts:48,56`, `globalCompositeBar.ts:295,609`). Same parser, same hand fixes. Cursor notes for each menu follow the tables.

Update rows for Manage: `appendUpdateMenuItems(MenuId.GlobalActivity, '7_update')` at `workbench/contrib/update/browser/update.ts:319` adds the seven rows `update.check`, `update.checking`, `update.downloadNow`, `update.downloading`, `update.install`, `update.updating`, `update.restart` (helper defined at `update.ts:80`). They are not in the Manage table below.

### LayoutControlMenu - The layout control area at the right of the title bar

V registrations: 12.

| Group | Order | Label | Command id | Registered in V |
|-|-|-|-|-|
| 1_workbench_layout |  | Configure Layout | `(submenu) LayoutControlMenuSubmenu` | workbench/browser/actions/layoutActions.ts:174 |
| navigation | 0 | Toggle Primary Side Bar (side bar on left) | `workbench.action.toggleSidebarVisibility` | workbench/browser/actions/layoutActions.ts:353 |
| navigation | 2 | Toggle Primary Side Bar (side bar on right) | `workbench.action.toggleSidebarVisibility` | workbench/browser/actions/layoutActions.ts:372 |
| navigation |  | Customize Layout... | `workbench.action.customizeLayout` | workbench/browser/actions/layoutActions.ts:1432 |
| 1_layout |  | Customize Layout... | `workbench.action.customizeLayout` | workbench/browser/actions/layoutActions.ts:1441 |
| navigation |  | Toggle Secondary Side Bar (left icon) | `workbench.action.toggleAuxiliaryBar` | workbench/browser/parts/auxiliarybar/auxiliaryBarActions.ts:143 |
| navigation |  | Toggle Secondary Side Bar (right icon) | `workbench.action.toggleAuxiliaryBar` | workbench/browser/parts/auxiliarybar/auxiliaryBarActions.ts:162 |
| navigation | 0 | Turn On Compact Mode | `workbench.action.enableCompactAuxiliaryWindow` | workbench/browser/parts/editor/auxiliaryEditorPart.ts:75 |
| navigation | 0 | Turn Off Compact Mode | `workbench.action.disableCompactAuxiliaryWindow` | workbench/browser/parts/editor/auxiliaryEditorPart.ts:96 |
| navigation | 1 | Toggle Panel | `workbench.action.togglePanel` | workbench/browser/parts/panel/panelActions.ts:322 |
| navigation | 1 | Turn On Always on Top | `workbench.action.enableWindowAlwaysOnTop` | workbench/electron-browser/actions/windowActions.ts:526 |
| navigation | 1 | Turn Off Always on Top | `workbench.action.disableWindowAlwaysOnTop` | workbench/electron-browser/actions/windowActions.ts:556 |

### LayoutControlMenuSubmenu - Configure Layout dropdown

V registrations: 4.

| Group | Order | Label | Command id | Registered in V |
|-|-|-|-|-|
| 0_workbench_layout | 0 | &&Primary Side Bar | `workbench.action.toggleSidebarVisibility` | workbench/browser/actions/layoutActions.ts:311 |
| z_end |  | Customize Layout... | `workbench.action.customizeLayout` | workbench/browser/actions/layoutActions.ts:1428 |
| 0_workbench_layout | 1 | &&Secondary Side Bar | `workbench.action.toggleAuxiliaryBar` | workbench/browser/parts/auxiliarybar/auxiliaryBarActions.ts:57 |
| 0_workbench_layout | 4 | &&Panel | `workbench.action.togglePanel` | workbench/browser/parts/panel/panelActions.ts:58 |

### CommandCenter - Command center row (back, forward, search box, share)

V registrations: 5.

| Group | Order | Label | Command id | Registered in V |
|-|-|-|-|-|
|  | 2 | &&Forward | `workbench.action.navigateForward` | workbench/browser/parts/editor/editorActions.ts:1444 |
|  | 1 | &&Back | `workbench.action.navigateBack` | workbench/browser/parts/editor/editorActions.ts:1479 |
|  | 101 | Command Center | `(submenu) CommandCenterCenter` | workbench/browser/parts/titlebar/commandCenterControl.ts:266 |
|  | 10002 | Agents | `(submenu) AgentsTitleBarControlMenu` | workbench/contrib/chat/browser/agentSessions/experiments/agentSessionsExperiments.contribution.ts:254 |
|  | 3 | Share... | `workbench.action.share` | workbench/contrib/share/browser/share.contribution.ts:101 |

### CommandCenterCenter - Inside the command center search box

V registrations: 2.

| Group | Order | Label | Command id | Registered in V |
|-|-|-|-|-|
|  | 100 | Quick Open | `workbench.action.quickOpenWithModes` | workbench/browser/actions/quickAccessActions.ts:161 |
|  | 1 | Debug | `(submenu) DebugToolBar` | workbench/contrib/debug/browser/debugToolBar.ts:438 |

### TitleBar - Title bar toolbar (before layout controls)

V registrations: 4.

| Group | Order | Label | Command id | Registered in V |
|-|-|-|-|-|
| navigation | 10000 | Toggle Notifications | `notifications.toggleList` | workbench/browser/parts/notifications/notificationsCommands.ts:331 |
| navigation | 10 | Browser | `workbench.action.browser.openOrList` | workbench/contrib/browserView/electron-browser/features/browserTabManagementFeatures.ts:489 |
| navigation | 1 | Chat | `(submenu) ChatTitleBarMenu` | workbench/contrib/chat/browser/agentSessions/experiments/agentSessionsExperiments.contribution.ts:268 |
| TitleBarLeadingActionsGroup | -1000 | Open in Agents | `workbench.action.openWorkspaceInAgentsWindow` | workbench/contrib/chat/electron-browser/agentSessions/agentSessionsActions.ts:56 |

### TitleBarAdjacentCenter - Title bar toolbar next to the center (update indicator)

V registrations: 2.

| Group | Order | Label | Command id | Registered in V |
|-|-|-|-|-|
|  | 0 | Sign In | `workbench.action.chat.triggerSetupFromTitleBar (ChatSetupSignInTitleBarAction.ID, name not read)` | workbench/contrib/chat/browser/chatSetup/chatSetupContributions.ts:402 |
|  | 0 | Update | `workbench.actions.updateIndicator` | workbench/contrib/update/browser/updateTitleBarEntry.ts:64 |

### TitleBarContext - Right-click menu on the title bar

V registrations: 5.

| Group | Order | Label | Command id | Registered in V |
|-|-|-|-|-|
| 2_config | 0 | Menu Bar | `workbench.action.toggleMenuBar` | workbench/browser/actions/layoutActions.ts:759-770 (loop over TitleBarContext and TitleBarTitleContext, Windows, Linux and web only) |
| 2_config | 1, 2, 4, 5, 8 | Command Center / Navigation Controls / Layout Controls / Notifications / Integrated Browser (toggles) | `toggle.window.commandCenter, toggle.workbench.navigationControl.enabled, toggle.workbench.layoutControl.enabled, toggle.workbench.notifications.showInTitleBar, toggle.workbench.browser.showInTitleBar (one action class, ToggleTitleBarConfigAction)` | workbench/browser/parts/titlebar/titlebarActions.ts:40 |
| 3_toggle | 0 | Hide Custom Title Bar | `toggle.window.customTitleBarVisibility` | workbench/browser/parts/titlebar/titlebarActions.ts:92 |
| 3_toggle | 1 | Hide Custom Title Bar In Full Screen | `toggle.window.customTitleBarVisibility.windowed` | workbench/browser/parts/titlebar/titlebarActions.ts:110 |
| 2_config | 3 | Editor Actions | `toggle.workbench.editor.editorActionsLocation` | workbench/browser/parts/titlebar/titlebarActions.ts:240 |

### TitleBarTitleContext - Right-click menu on the title text (macOS)

V registrations: 6.

| Group | Order | Label | Command id | Registered in V |
|-|-|-|-|-|
| 2_config | 0 | Menu Bar | `workbench.action.toggleMenuBar` | workbench/browser/actions/layoutActions.ts:759-770 (loop over TitleBarContext and TitleBarTitleContext, Windows, Linux and web only) |
| 2_config | 1, 2, 4, 5, 8 | Command Center / Navigation Controls / Layout Controls / Notifications / Integrated Browser (toggles) | `same toggle.* actions as TitleBarContext` | workbench/browser/parts/titlebar/titlebarActions.ts:46 |
| 3_toggle | 0 | Hide Custom Title Bar | `toggle.window.customTitleBarVisibility` | workbench/browser/parts/titlebar/titlebarActions.ts:93 |
| 3_toggle | 1 | Hide Custom Title Bar In Full Screen | `toggle.window.customTitleBarVisibility.windowed` | workbench/browser/parts/titlebar/titlebarActions.ts:111 |
| 2_config | 3 | Editor Actions | `toggle.workbench.editor.editorActionsLocation` | workbench/browser/parts/titlebar/titlebarActions.ts:241 |
| 1_file | -i | Path segments of the document (mac title bar only) | `(dynamic, one per path segment)` | workbench/electron-browser/window.ts:684 |

### AccountsContext - Accounts menu

V registrations: 18.

| Group | Order | Label | Command id | Registered in V |
|-|-|-|-|-|
|  | 100 | Manage Extension Account Preferences... | `_manageAccountPreferencesForExtension` | workbench/contrib/authentication/browser/actions/manageAccountPreferencesForExtensionAction.ts:26 |
|  | 100 | Manage Language Model Access... | `workbench.action.chat.manageLanguageModelAuthentication` | workbench/contrib/chat/browser/actions/chatLanguageModelActions.ts:33 |
| 2_copilot |  | Sign in to use GitHub Copilot... | `workbench.action.chat.triggerSetupFromAccounts` | workbench/contrib/chat/browser/chatSetup/chatSetupContributions.ts:369 |
| 2_editSessions |  | Turn on Cloud Changes... | `workbench.editSessions.actions.signIn` | workbench/contrib/editSessions/browser/editSessionsStorageService.ts:475 |
| 2_editSessions |  | Turn on Cloud Changes... (1) | `{{ id, title: localize('sign in badge', 'Turn on Cloud Changes... (1)'), }}` | workbench/contrib/editSessions/browser/editSessionsStorageService.ts:487 |
| 2_editSessions |  | Turn off Cloud Changes... | `workbench.editSessions.actions.resetAuth` | workbench/contrib/editSessions/browser/editSessionsStorageService.ts:510 |
|  |  | Sign in to access Extensions Marketplace | `workbench.extensions.actions.gallery.signIn` | workbench/contrib/extensions/browser/extensions.contribution.ts:2118 |
| 2_remoteTunnel |  | Turn on Remote Tunnel Access... | `workbench.remoteTunnel.actions.turnOn` | workbench/contrib/remoteTunnel/electron-browser/remoteTunnel.contribution.ts:503 |
| 2_remoteTunnel |  | Remote Tunnel Access is On | `workbench.remoteTunnel.actions.manage` | workbench/contrib/remoteTunnel/electron-browser/remoteTunnel.contribution.ts:601 |
| 2_remoteTunnel |  | Remote Tunnel Access is Connecting | `workbench.remoteTunnel.actions.connecting` | workbench/contrib/remoteTunnel/electron-browser/remoteTunnel.contribution.ts:620 |
| 1_settings | 2 | Backup and Sync Settings... | `workbench.userDataSync.actions.turnOn` | workbench/contrib/userDataSync/browser/userDataSync.ts:784 |
| 1_settings |  | Turning on Settings Sync... | `workbench.userData.actions.turningOn` | workbench/contrib/userDataSync/browser/userDataSync.ts:811 |
| 1_settings |  | Sign in to Sync Settings (1) | `{{ id, title: localize('sign in accounts', "Sign in to Sync Settings (1)"), }}` | workbench/contrib/userDataSync/browser/userDataSync.ts:867 |
| 1_settings |  | Settings Sync is On | `workbench.userDataSync.actions.manage` | workbench/contrib/userDataSync/browser/userDataSync.ts:935 |
| 3_accessRequests |  | Grant access to {0} for {1}... (dynamic) | `{providerId}{extensionId}Access (dynamic)` | workbench/services/authentication/browser/authenticationExtensionsService.ts:401 |
| 2_signInRequests |  | Sign in with {0} to use {1} (dynamic) | `(dynamic)` | workbench/services/authentication/browser/authenticationExtensionsService.ts:463 |
| 3_accessRequests |  | Grant access to {0} for {1}... (dynamic, MCP) | `{providerId}{mcpServerId}Access (dynamic)` | workbench/services/authentication/browser/authenticationMcpService.ts:453 |
| 2_signInRequests |  | Sign in with {0} to use {1} (dynamic, MCP) | `(dynamic)` | workbench/services/authentication/browser/authenticationMcpService.ts:513 |

### GlobalActivity - Manage menu (gear)

V registrations: 16.

| Group | Order | Label | Command id | Registered in V |
|-|-|-|-|-|
| 2_configuration | 3 | Extensions | `workbench.view.extensions` | workbench/contrib/extensions/browser/extensions.contribution.ts:676 |
| 2_configuration | 2 | &&Settings | `workbench.action.openSettings` | workbench/contrib/preferences/browser/preferences.contribution.ts:241 |
| 2_configuration | 4 | Open Keyboard Shortcuts | `workbench.action.openGlobalKeybindings` | workbench/contrib/preferences/browser/preferences.contribution.ts:909 |
| 1_command | 1 | Command Palette... | `workbench.action.showCommands` | workbench/contrib/quickaccess/browser/quickAccess.contribution.ts:91 |
| 2_configuration | 5 | Configure Snippets | `workbench.action.openSnippets` | workbench/contrib/snippets/browser/commands/configureSnippets.ts:257 |
| 2_configuration | 6 | Tasks | `workbench.action.tasks.openUserTasks` | workbench/contrib/tasks/browser/task.contribution.ts:363 |
| 2_configuration | 7 | Themes | `(submenu) ThemesSubMenu` | workbench/contrib/themes/browser/themes.contribution.ts:903 |
| 7_update | 1 | Show Update Release Notes | `update.showUpdateReleaseNotes` | workbench/contrib/update/browser/update.ts:333 |
| 7_update |  | Switch to Insiders Version... | `(SwitchQuality commandId, web only)` | workbench/contrib/update/browser/update.ts:371 |
| 2_configuration | 1 | Profile ({0}) (dynamic) | `(submenu) ProfilesMenu` | workbench/contrib/userDataProfile/browser/userDataProfile.ts:180 |
| 2_configuration | 1 | &&Profiles | `workbench.profiles.actions.manageProfiles` | workbench/contrib/userDataProfile/browser/userDataProfile.ts:371 |
| 3_configuration | 2 | Backup and Sync Settings... | `workbench.userDataSync.actions.turnOn` | workbench/contrib/userDataSync/browser/userDataSync.ts:774 |
| 3_configuration | 2 | Turning on Settings Sync... | `workbench.userData.actions.turningOn` | workbench/contrib/userDataSync/browser/userDataSync.ts:806 |
| 3_configuration | 2 | Sign in to Sync Settings | `workbench.userData.actions.signin` | workbench/contrib/userDataSync/browser/userDataSync.ts:853 |
| 3_configuration | 2 | Show Conflicts (dynamic title) | `workbench.userDataSync.actions.showConflicts` | workbench/contrib/userDataSync/browser/userDataSync.ts:895 |
| 3_configuration | 2 | Settings Sync is On | `workbench.userDataSync.actions.manage` | workbench/contrib/userDataSync/browser/userDataSync.ts:923 |



### Cursor notes for the menus above

Found by `[.]MenuId-name` searches in the Cursor bundle and by mapping `k(index, null)` string calls to `out\nls.keys.json` and `out\nls.messages.json`.

- `LayoutControlMenu` (Cursor, 7 items found plus the Configure Layout submenu): the group is `2_pane_toggles` instead of V's `navigation`. Items: Toggle Primary Side Bar (`workbench.action.toggleSidebarVisibility`, order 0 for a left side bar, 2 for a right one), Toggle Agents Side Bar (`workbench.action.toggleUnifiedSidebar`, two variants, agent feature), Maximize Chat Size (`workbench.action.maximizeChatSize`, agent feature), Toggle Panel (`workbench.action.togglePanel`, order 1), and group `6_workbench_toggles` order 7: Open Layout Settings Menu (`workbench.action.openLayoutSettingsMenu`, gear icon, a no-op that only stops the click). Still there: the Configure Layout submenu (when `workbench.layoutControl.type` is `menu`). Not found: the Secondary Side Bar toggle, Customize Layout in the `navigation`/`1_layout` groups, the compact mode and always-on-top rows for auxiliary windows.
- `LayoutControlMenuSubmenu` (Cursor, 4 items): Primary Side Bar, Secondary Side Bar, Panel, Customize Layout... Same as V.
- `CommandCenter` (Cursor, 4): Back (order 1), Forward (order 2) with `config.workbench.navigationControl.enabled`, and the Command Center submenu (order 101). Not found: Share... (`workbench.action.share` appears twice in the bundle but not as a CommandCenter item), and the Agents control.
- `CommandCenterCenter` (Cursor, 2): Quick Open (`workbench.action.quickOpenWithModes`, order 100) and the Debug toolbar submenu. Same as V.
- `TitleBar` (Cursor): Private Inference Mode (`privateInference.turnOn`, `privateInference.turnOff`, lock icon, group `navigation`). V's rows (notification bell `notifications.toggleList`, Browser, Chat, Open in Agents) are not registered on `MenuId.TitleBar` in Cursor. Cursor draws its own title bar buttons with plain action lists instead (see the Cursor research file, TB-5).
- `TitleBarAdjacentCenter`: 0 hits in the Cursor bundle. V's Update row (`workbench.actions.updateIndicator`) is gone with it (0 hits). Cursor's update control is `cursor.titlebar.installUpdate`, built in code.
- `TitleBarContext` and `TitleBarTitleContext` (Cursor): the generic toggles (Navigation Controls, Layout Controls, Agents Window Button), Editor Actions (order 3, group `2_config`), Menu Bar, and a new row Restore Native Title Bar (`toggle.window.titleBarStyle`, group `4_restore_native_title`, shown when the title bar style is `custom`). Not found: the Command Center and Notifications toggles, Hide Custom Title Bar, Hide Custom Title Bar In Full Screen.
- `AccountsContext` (Cursor, 5 found): remote tunnel rows (3), the dynamic access request and sign in request rows (2), and a placeholder `noAuthenticationProviders`. Not found: all settings sync, cloud changes, Copilot sign in, language model access and extension marketplace sign in rows.
- `GlobalActivity` (Cursor, 16 found): Command Palette, Settings, Open Keyboard Shortcuts, Extensions, Configure Snippets, Tasks, Themes, Profile (submenu with a runtime title), the seven `update.*` rows and Show Update Release Notes. Not found: the settings sync rows.
- The Cursor title bar part never shows Accounts or Manage: its `activityActionsEnabled` getter returns `false` (see the Cursor research file, TB-6).

