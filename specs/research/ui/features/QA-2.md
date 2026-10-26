Group: QA-2 Go to File and workspace search. Section: Quick access. Date: 2026-10-25. Pass: 2.
Features: 23.

### QA-2.1 Open a file by name with Go to File
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+P, type part of a file name, and pick a row to open it. Each row shows a file icon, the name, the folder path, and a filled dot when the file has unsaved changes. With no matches the list shows "No matching results".
- Commands: workbench.action.quickOpen
- Menus: MenubarGoMenu (Go to File..., group 3_global_nav); EmptyEditorGroupContext (Open File...); EditorTabsBarContext (Open File...)
- Keys: Ctrl+P; Ctrl+E (Windows and Linux only); Cmd+P (Mac)
- Settings: search.exclude; files.exclude; search.useIgnoreFiles; search.useGlobalIgnoreFiles; search.useParentIgnoreFiles; search.followSymlinks; workbench.editor.customLabels.enabled
- Cursor: changed (same command, keys and menu rows; placeholder reads "Search files, content, and symbols (append : to go to line or @ to go to symbol)" and the picks come from Cursor's own function, see QA-2.20)
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts; V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\searchQuickAccess.contribution.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\services\search\common\queryBuilder.ts; C bundle (command, Eje key table, Go menu item, class vI, messages 9447 and 9448)

### QA-2.2 See recently opened files first
- Tag: both
- Evidence: source
- Boundary: no
- What: With an empty box the list is the editor history under a "recently opened" heading. When you type, matching history rows stay above the "file results" heading and are left out of the file results below.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: search.quickOpen.includeHistory; search.quickOpen.history.filterSortOrder
- Cursor: changed (present, but history rows under /.cursor/worktrees/ are hidden)
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts; C bundle (getEditorHistoryPicks, messages 9418 and 9419, 9633 to 9636)

### QA-2.3 Fuzzy match with highlighted letters
- Tag: both
- Evidence: source
- Boundary: no
- What: Typed letters match out of order across the file name and folder path, and the matched letters are drawn highlighted in each row. A space starts a second search word, a double quote asks for an exact match, and a slash makes the match include the folder path.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\base\common\fuzzyScorer.ts; V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; C bundle (prepareQuery function Xz with the same values and exact-match fields, scoreItemFuzzy)

### QA-2.4 Open a file by typing its path
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing a full path, a path starting with ~, or a path relative to a workspace folder shows that exact file as a single result, even when search excludes would normally hide it.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (stock also fixes the file name casing against the disk, the bundle copy does not)
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; C bundle (getAbsolutePathFileResult, getRelativePathFileResults)

### QA-2.5 Jump to a line and column with a suffix
- Tag: both
- Evidence: source
- Boundary: no
- What: After a file name, type :12, :12:5, :12,5, #12 or (12) to open the file at that line and column. Stock also takes a range such as :20-40 or :20:3-40:5 and selects it. Changing only the suffix keeps the current results in place.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (the bundle pattern has no end-of-range part, and the file opens with the cursor collapsed to the start)
- Source: V\src\vs\workbench\contrib\search\common\search.ts; V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; C bundle (line-colon pattern G4o, openAnything with collapseToStart)

### QA-2.6 Go to a symbol inside a file you are filtering
- Tag: both
- Evidence: source
- Boundary: no
- What: Type @ after a partial file name to list the symbols of the highlighted file. The file opens in a temporary tab, the active symbol is scrolled to the middle and highlighted, and Escape puts the old editor view back.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts; C bundle (getEditorSymbolPicks, doGetEditorSymbolPicks, openTransientEditor)

### QA-2.7 Open to the side or bottom
- Tag: both
- Evidence: source
- Boundary: no
- What: Each file row has an "Open to the Side" button (it reads "Open to the Bottom" when the side direction is down). Alt+Enter does the same, and so does Ctrl+Enter while previews from Quick Open are off, which is the default. The same button sits on symbol rows in the # and @ lists.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: Alt+Enter (modifier read by the picker, no keybinding rule); Ctrl+Enter (same)
- Settings: workbench.editor.openSideBySideDirection; workbench.editor.enablePreviewFromQuickOpen
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (createAnythingPick, openAnything, messages 9427 and 9428)

### QA-2.8 Open files without closing the list
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Right Arrow on a row to open that file in the background as a pinned tab while the list stays open, so several files can be opened in a row. The provider turns this on for files, symbols and Quick Search matches.
- Commands: quickInput.acceptInBackground
- Menus: none found
- Keys: Right Arrow (when in quick pick and focus is in the list or the cursor is at the end of the input box)
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts; V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\contrib\welcomeGettingStarted\common\gettingStartedContent.ts; C bundle (quickInput.acceptInBackground registration)

### QA-2.9 Open as a preview tab or a pinned tab
- Tag: both
- Evidence: source
- Boundary: no
- What: By default a file picked from the list opens as a pinned tab. With the preview setting on it opens as a preview tab (italic, replaced by the next one) and holding Ctrl while picking pins it instead.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: Ctrl+Enter (pins when the preview setting is on)
- Settings: workbench.editor.enablePreviewFromQuickOpen; workbench.editor.enablePreview
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (openAnything pinned option, setting key in the main bundle)

### QA-2.10 Remove a file from recently opened
- Tag: both
- Evidence: source
- Boundary: no
- What: Rows that come from history have an X button, tooltip "Remove from Recently Opened", that drops the file from the list and from history. A row with unsaved changes shows an always-visible filled dot in the same spot.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; C bundle (createAnythingPick, message 9429 and 9430)

### QA-2.11 Add workspace symbols to the file results
- Tag: both
- Evidence: source
- Boundary: no
- What: With the setting on, symbols from language servers are scored together with files and shown under the "file and symbol results" heading in the same list. Only top-level kinds (classes, enums, interfaces, namespaces, modules, packages, files) or symbols with no container are used, and none are added while a line suffix is typed.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: search.quickOpen.includeSymbols
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\symbolsQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts; C bundle (getWorkspaceSymbolPicks, messages 9424, 9631)

### QA-2.12 Go to Symbol in Workspace
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+T, or typing #, lists symbols from all language servers. The box starts filled with the selected or nearby word, each row shows a kind icon, the name, and "container, bullet, relative path", deprecated symbols are struck through, and a second search word filters by container. Enter opens the file with the cursor at the symbol start.
- Commands: workbench.action.showAllSymbols
- Menus: MenubarGoMenu (Go to Symbol in Workspace..., group 3_global_nav, order 2)
- Keys: Ctrl+T; Cmd+T (Mac)
- Settings: none found
- Cursor: changed (same command, key, menu item and placeholder; older copy where the kind icon is part of the label text, and the rows have no shift-to-attach callback)
- Source: V\src\vs\workbench\contrib\search\browser\symbolsQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\searchActionsSymbol.ts; V\src\vs\workbench\contrib\search\browser\searchQuickAccess.contribution.ts; C bundle (class ome, showAllSymbols action, messages 9449 to 9450 and 9491 to 9493)

### QA-2.13 Leave chosen extensions out of # symbol search
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A setting that takes a list of extension ids whose symbol providers are skipped when the # list searches for symbols. It takes effect as soon as it changes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.quickAccess.excludeSymbolProviders
- Cursor: cursor only
- Source: C bundle (class ome, _refreshExcludedProvidersFromConfig, message 9680); not in V or N

### QA-2.14 Search text across files with Quick Search
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing % lists text matches in the workspace, grouped under one heading per file (file name and folder), with the matched text highlighted in each line. It shows up to 30 files and 10 matches per file, then "More" and "See More Files" rows. Run from the command, it starts with the selected editor text.
- Commands: workbench.action.quickTextSearch
- Menus: CommandPalette (Search: Quick Search, f1 only)
- Keys: none found
- Settings: search.maxResults; search.smartCase; search.sortOrder; editor.find.seedSearchStringFromSelection
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\searchActionsTextQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts; V\src\vs\workbench\contrib\search\common\constants.ts; C bundle (QuickTextSearchActionId, Quick Search class, messages 9437 to 9443, 9451, 9452, 9494)

### QA-2.15 Preview each Quick Search match as you move
- Tag: both
- Evidence: source
- Boundary: no
- What: Moving through the match rows opens that file in a temporary tab at the match, without taking focus, after a short pause of 75 ms. Escape puts the earlier editor view back, and accepting clears the match highlights.
- Commands: workbench.action.quickTextSearch
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts; C bundle (Quick Search provide with debounce and openTransientEditor)

### QA-2.16 Move Quick Search results into the Search view
- Tag: both
- Evidence: source
- Boundary: no
- What: An "Open in Search View" button at the right of the input, one on each match row, an "Open File" button on each file heading, and the "More" and "See More Files" rows all hand the results to the Search view (SB-3) and close the list. In stock the input button always moves the search over, and in Cursor it just opens the view when there are no results yet.
- Commands: workbench.action.quickTextSearch
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (input button behaves differently when the result list is empty)
- Source: V\src\vs\workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts; C bundle (Quick Search provide, moveToSearchViewlet, messages 9437 to 9441)

### QA-2.17 Remember the last typed text
- Tag: both
- Evidence: source
- Boundary: no
- What: Two settings bring back the last text typed in Go to File, and separately in Quick Search, the next time the list opens. Both are off by default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.quickOpen.preserveInput; search.quickAccess.preserveInput
- Cursor: present
- Source: V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts; V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts; C bundle (configuration getters in classes vI and the Quick Search class)

### QA-2.18 Open Go to File with the previous file ready
- Tag: both
- Evidence: source
- Boundary: no
- What: A command, with no default key or menu item, that opens Go to File with the second history row active, so Enter reopens the file you were just in.
- Commands: workbench.action.quickOpenPreviousEditor
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts; C bundle (command registration with ItemActivation.SECOND)

### QA-2.19 Keep the list open when focus leaves
- Tag: both
- Evidence: source
- Boundary: no
- What: By default every quick list closes when it loses focus. Turning the setting off keeps Go to File, and every other quick pick, open until you press Escape or pick something.
- Commands: workbench.action.closeQuickOpen
- Menus: none found
- Keys: Escape; Shift+Escape (closeQuickOpen, when in a quick pick)
- Settings: workbench.quickOpen.closeOnFocusLost
- Cursor: present
- Source: V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\services\quickinput\browser\quickInputService.ts; V\src\vs\workbench\browser\actions\quickAccessActions.ts; C bundle (setting in the main bundle, closeQuickOpen key rule)

### QA-2.20 Split results into file results and other results
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor ranks Go to File results in two lists. Up to 6 rows scoring 100 or more go under "file results" (or "file and symbol results"), and every other match goes under "other results", up to 512 rows in all. With several search words, a high-scoring row whose matched runs average 3 letters or fewer is dropped from the results instead of shown.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: search.quickOpen.useVscodeSearch
- Cursor: cursor only
- Source: C bundle (doGetCursorPicks, getCursorAdditionalPicks, constants HIGH_SCORING_THRESHOLD 100 and HIGH_SCORING_CAP_ITEMS 6, messages 9416 and 9424 and 9425)

### QA-2.21 Switch Go to File back to VS Code's own search
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A setting, off by default, that makes Go to File use the stock algorithm instead of Cursor's modified one. The list then has a single "file results" group and no Create File row.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: search.quickOpen.useVscodeSearch
- Cursor: cursor only
- Source: C bundle (configuration getter useVscodeSearch, doGetPicks choosing doGetVscodePicks or doGetCursorPicks, message 9637)

### QA-2.22 Create a file from the Go to File box
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: When nothing matches and the typed text looks like a path (absolute, or contains a slash, or ends in an extension such as .ts, and does not end in a slash), the list shows "No matching results" and a row "Create File '<path>'" with the note "Create a new file at this path". Enter makes the empty file (and any missing folder, under the first workspace folder for a relative path) and opens it. Alt or Ctrl opens it to the side. A failure shows "Failed to create file".
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: Enter; Alt+Enter; Ctrl+Enter
- Settings: none found
- Cursor: cursor only
- Source: C bundle (isValidFilePath, createFilePicksForPath, createNewFile, CREATE_FILE_PICK_ID, messages 9431 to 9433); not in V or N

### QA-2.23 Open a link from Go to File
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. Typing or pasting a link such as scheme://... shows a link row with an Open to the Side button, or an "Invalid Link" row with the note "Enter a valid link to open it from Quick Open." The placeholder gains "or paste a link".
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: Alt+Enter
- Settings: none found
- Cursor: missing (bundle has no "Invalid Link" or "paste a link" text)
- Source: N\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; N\src\vs\workbench\contrib\search\browser\searchQuickAccess.contribution.ts; C bundle searched, no hits

## Handed off

- Status bar line and column item (status.editor.selection, opens Go to Line): SS-3
- Empty editor watermark row "Go to File" with its shortcut: ED-9
- Go menu itself and its rows (Go to File, Go to Symbol in Workspace, Go to Line/Column): TB-1 (the commands and menu ids are listed in the features above)
- Quick Open from the command center, the `?` help list, and the shortcut rows that open Go to File and Quick Search: QA-5
- Navigate Next and Previous in Quick Open (Ctrl+P again while held): QA-5
- Width setting `workbench.quickOpen.width` (Cursor only), row hover, drag, and the shared list and input chrome: QA-6
- Quick Search results landing in the Search view and its results tree: SB-3
- Show All Editors and the editor switcher lists (edt prefixes): QA-4
- Go to Symbol in Editor (@) and Go to Line (:) as standalone lists: QA-3

## Unclear

- Whether Cursor's Go to File box matches file contents. The placeholder says "files, content, and symbols", but the picks code in the bundle builds only file, history and symbol picks. No content search call was found. This repeats an open point from pass 1.
- Shift+Enter on a row adds the file or symbol to the focused chat in stock (attach callback). It is chat, so out of scope and not listed. Cursor's copy of the file and # rows has no attach callback.
- Cursor sends the picked-list shape (separators, sections, resources) from a Go to File accept listener in the provider. It is not visible UI and was not cataloged.
- Cursor pre-warms the file query cache (initializeCaches) and skips file search on remote folders with no search provider. Not visible UI, not cataloged.
- The `?` help entry for Quick Search ("Search for Text", order 25) exists in both builds, but whether Cursor shows the Open Quick Chat row next to it (order 30) was not traced.
- workbench.quickOpen.closeOnFocusLost applies to every quick input, so QA-6 may also want it. It is listed here only because the group brief names the workbench.quickOpen settings.
- Behavior on screen was not watched. Every statement comes from reading source or bundle strings.

