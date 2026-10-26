Section: Editor (ED). Date: 2026-10-25. Pass: 1.

# 05 Editor - Cursor 3.27.12 findings (pass 1, minor groups)

## How to read this file

- Scope of this file: what the Cursor install has for the section, checked against the stock list in the sibling file `vscode\05-editor.md`. Group ids ED-1 to ED-10 are the same in both files and are stable.
- Path legend: `C` = `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`. The main bundle is `C\out\vs\workbench\workbench.desktop.main.js`. Label strings live in `C\out\nls.messages.json` and module names in `C\out\nls.keys.json` (the bundle calls `k(N,null)` or `De(N,"...")` where N is the index into the messages file). Built-in extensions are in `C\extensions`.
- Evidence is `bundle` for everything here. No Cursor source exists; I read command ids, pane ids, labels and module names as strings. `N` (1.144.0) was used only for the "newer than 1.128" lists in the stock file.
- Tag rule: `both` means a command id, label or setting name from the stock list was found as a quoted string in the Cursor bundle. `cursor` means the item exists only in Cursor. `vscode` means I searched and did not find it. Searches were exact quoted-string checks, so an id built at run time from parts would show as absent (called out where it matters).
- Method for the "found" counts below: I collected every command-like id (`id: '...'`, `ID = '...'`, `registerCommand('...')`) from the stock source folder for that group and tested each one against the Cursor bundle. The counts are in the table.
- Autocomplete (suggest, inline completions and ghost text, parameter hints, signature help, snippet placeholders, Cursor Tab with its `editor.cpp.*` commands) is section 06 and is not grouped here.

## Coverage of the stock list in the Cursor bundle

| Group | Stock ids tested | Found in Cursor | Absent in Cursor |
|-|-|-|-|
| ED-1 groups, tabs, layout | 148 (`parts\editor`) | 139 | `workbench.action.moveEditorToStart`, `moveEditorToEnd`, `clearEditorHistoryWithoutConfirm`, `enableCompactAuxiliaryWindow`, `disableCompactAuxiliaryWindow`, `toggleCompactAuxiliaryWindow`, `editor.tabs.addTab`, `breadcrumbs.copyPath` (the ninth, `mainThreadWebview-simpleBrowser.view`, is not a command) |
| ED-2 editing | multicursor 13, linesOperations 26, comment 4, indentation 8, caretOperations 3, smartSelect 2, clipboard 4, dropOrPasteInto 4, bracketMatching 3, codeEditor 29 | all but 8 | `editor.action.reverseLines`, five `editor.action.renderWhitespace.*` modes, `editor.action.announceCursorPosition`, `workbench.action.gotoOffset` |
| ED-3 find | `FIND_IDS` 18 (including `togglePreserveCase`), searchEditor 2 | all 20 | none |
| ED-4 language help | gotoSymbol 20, gotoError 5, rename 6, codeAction 2, format 3 and 5, codelens 1, links 1, wordHighlighter 3, colorPicker 3, plus `editor.action.showHover`, `quickFix`, `refactor`, `sourceAction`, `organizeImports` | all | none |
| ED-5 view aids | stickyScroll 7, folding 20, fontZoom 3, unicodeHighlighter 7, breadcrumbs (`toggle`, `focus`, `toggleToOn`, `focusAndSelect`), toggles for minimap, word wrap, whitespace, column selection, overtype, tab focus | all except `breadcrumbs.copyPath` | `breadcrumbs.copyPath` |
| ED-6 diff, merge | diffEditor 9, multiDiffEditor 7, mergeEditor 30, files compare 38, localHistory 13 | 5 of 7, 28 of 30, 37 of 38, all others | `multiDiffEditor.goToNextChange`, `goToPreviousChange`, `mergeEditor.acceptAllCombination`, `mergeEditor.toggleBetweenInputs`, `workbench.files.openFile` |
| ED-7 notebooks | notebook 92, interactive 11, replNotebook 4 | 79, 11, 4 | 13 notebook ids: all the chat ones (`notebook.cell.chat.start`, `startAtTop`, `notebook.cellOutput.addToChat`, `notebook.inlineChat.acceptChangesAndRun`, `notebook.chat.selectAndInsertKernelVariable`) plus `notebook.findNext.fromWidget`, `findPrevious.fromWidget`, `notebook.diff.cell.switchOutputRenderingStyle`, `notebookOutputEditor`, `vscode.notebook.variable`, `workbench.notebook.layout.select`, `workbench.notebook.layout.gettingStarted` |
| ED-8 settings editors | preferences 37, keybindings 1, userDataProfile 13, workspace 4 | 35, all, all, all | `_developer.getConfigurationInformation`, `settings.stopSyncingSetting` |
| ED-9 welcome and empty | gettingStarted 43, welcomeWalkthrough 5, welcomeAgentSessions 1, welcomeOnboarding 1, welcomeViews 2 | 18, 0, 0, 0, 2 | 25 getting-started ids (all the built-in walkthrough step and category ids such as `pickColorTheme`, `scmClone`, `keybindingsAccessibility`, plus `welcome.showAllWalkthroughs`), all Playground ids including `workbench.action.showInteractivePlayground`, `agentSessionsWelcomePage`, `workbench.action.welcomeOnboarding2026` |
| ED-10 viewers | webviewPanel 5, webview 1, customEditor 2, browserView 14, imageCarousel 5 | 5, 1, 2, 0, 0 | all 14 stock Browser ids (`workbench.action.browser.zoomIn`, `toggleDeviceEmulation`, `browser.toggleFavorite`, `workbench.editorinputs.browser`, and so on), all 5 Images Preview ids including `workbench.action.chat.openImageInCarousel` |

## Proposed minor groups, Cursor side

### ED-1 Editor groups, tabs and layout

- Plain name: Editor groups, tabs and layout
- Tag: both
- Evidence: bundle
- Boundary: no
- What Cursor has: 139 of 148 stock ids (table above). Menu registrations for `EditorTitle`, `EditorTitleContext`, `EditorTabsBarContext` (with submenus `EditorActionsPositionSubmenu`, `EditorTabsBarShowTabsSubmenu`, `EditorTabsBarShowTabsZenModeSubmenu`), `EmptyEditorGroupContext`. Settings found by name: `workbench.editor.showTabs`, `editorActionsLocation`, `tabSizing`, `pinnedTabSizing`, `pinnedTabsOnSeparateRow`, `wrapTabs`, `enablePreview` and its two variants, `openPositioning`, `splitSizing`, `splitInGroupLayout`, `autoLockGroups`, `limit`, `centeredLayoutFixedWidth`, `doubleClickTabToToggleEditorGroupSizes`, and others.
- Differences from stock:
  - Not found in Cursor: the Modal editor part (zero matches for `ModalEditor`, `modalEditor`, `workbench.editor.useModal`), the compact auxiliary window commands, and the settings `workbench.editor.swipeToNavigate`, `restoreEditors`, `showTabIndex`.
  - Cursor-only: `workbench.action.evenEditorWidthsExcludingAgent`, title "Reset Editor Group Sizes (Excluding Agent)" (nls module `vs/workbench/browser/parts/editor/editorActions.cursor`, index 3568). It makes sense only because an Agent chat can sit in an editor group.
  - Cursor-only: a globe button in the editor title bar, command `workbench.action.openBrowserEditor`, label "Open Browser" (nls `vs/workbench/browser/parts/editor/editor.contribution :: openBrowser`, index 3474), registered on `MenuId.EditorTitle` in group `0_global`. See the Browser entry under ED-10.
  - Cursor-only: "Copy cursor.com link" on the editor title menu (`cursorOrigin.copyCodeLinkEditorTitle.menu`, nls module `vs/workbench/contrib/cursorOrigin/browser/cursorOrigin.contribution`).
  - Layout toggles for the Agents window layout live in `vs/workbench/browser/actions/layoutActions` and `agentLayoutQuickMenu`; those belong to section 01 and the Agents window and are left out.
- Proof strings: `workbench.action.splitEditor`, `workbench.action.editorLayoutTwoColumns`, `workbench.action.joinAllGroups`, `workbench.action.pinEditor`, `workbench.action.keepEditor`, `workbench.action.toggleEditorGroupLock`, `workbench.action.toggleMaximizeEditorGroup`, `workbench.action.moveEditorToNewWindow`, `workbench.action.copyEditorToNewWindow`, `workbench.action.newEmptyEditorWindow`, `workbench.action.reopenClosedEditor`, `workbench.action.showAllEditorsByMostRecentlyUsed`, `workbench.action.toggleEditorType`, `workbench.action.configureEditorTabs`, `workbench.action.showMultipleEditorTabs`, `workbench.action.hideEditorTabs`, `workbench.action.toggleZenMode`, `workbench.action.toggleCenteredLayout`, `files.autoSave` (all found as quoted strings in `workbench.desktop.main.js`).

### ED-2 Editing text

- Plain name: Editing text
- Tag: both
- Evidence: bundle
- Boundary: no
- What Cursor has: every multi-cursor, line operation, comment, indentation, caret, smart select, clipboard, paste and drop, and bracket id from stock (see table; nls modules `vs/editor/contrib/multicursor/browser/multicursor`, `linesOperations`, `comment`, `indentation`, `caretOperations`, `smartSelect`, `clipboard`, `dropOrPasteInto`, `bracketMatching`, `linkedEditing`, `anchorSelect`, `lineSelection`, `cursorUndo`, `wordOperations` exist in `nls.keys.json`). `V\extensions\emmet` is present in `C\extensions\emmet` with the same 23 commands and 12 settings. Editor dictation is present: `workbench.action.editorDictation.start` is in Cursor's tracked-command list.
- Differences from stock:
  - Not found in Cursor: `editor.action.reverseLines` (Reverse lines), the five `editor.action.renderWhitespace.*` mode commands, `editor.action.announceCursorPosition`, `workbench.action.gotoOffset`. Note the ED-5 whitespace toggle `editor.action.toggleRenderWhitespace` is found.
  - Cursor-only additions that sit on editor menus: "Copy cursor.com link" (with selection, or file only) on the editor context menu, the line-number gutter menu (`MenuId.EditorLineNumberContext`, command `cursorOrigin.copyCodeLink`), and the Explorer; commands `cursorOrigin.copyCodeLink` and `cursorOrigin.copyCodeLinkWithoutRange`, category "Cursor Origin".
  - Cursor-only submenu on the editor context menu: "Cursor Blame" (see ED-6).
  - Editor context entries that hand text to chat (`editor.action.addSymbolToChat`, `editor.action.addSymbolToNewChat`, `cursor.createRuleFromSelection`, `composer.addfilestocomposer`, `composer.addsymbolstocomposer`, `composer.fixerrormessage`, `editor.action.knowledgeBase.add`, `editor.action.knowledgeBase.list`) are chat entry points and are left out. They appear in the list of tracked commands labeled `editor_ai_actions` and `knowledge_base`.
- Proof strings: `editor.action.insertCursorAbove`, `editor.action.addSelectionToNextFindMatch`, `editor.action.moveLinesUpAction`, `editor.action.commentLine`, `editor.action.blockComment`, `editor.action.reindentlines`, `editor.action.smartSelect.expand`, `cursorUndo`, `editor.action.linkedEditing`, `editor.action.jumpToBracket`, `editor.action.setSelectionAnchor`, `editor.action.pasteAs`, `editor.action.toggleColumnSelection`, `editor.action.toggleOvertypeInsertMode`, `workbench.action.toggleMultiCursorModifier`, `workbench.action.editor.changeLanguageMode`, `workbench.action.editor.changeEOL`, `workbench.action.editor.changeEncoding`, `workbench.action.files.setActiveEditorReadonlyInSession`.

### ED-3 Find and replace

- Plain name: Find and replace
- Tag: both
- Evidence: bundle
- Boundary: no
- What Cursor has: all 18 `FIND_IDS` strings (`actions.find`, `editor.action.nextMatchFindAction`, `editor.action.startFindReplaceAction`, `editor.action.replaceAll`, `toggleFindRegex`, and the rest). The Search Editor: nls module `vs/workbench/contrib/searchEditor/browser/searchEditor.contribution` (pane label "Search Editor", index 9682; "Toggle Context Lines", "Increase Context Lines", "Decrease Context Lines"), ids `search.action.openNewEditor`, `search.action.openNewEditorToSide`, `search.action.openEditor`, and the tracked editor input `workbench.editorinputs.searchEditorInput`. `C\extensions\search-result\package.json` matches stock (language and grammar only).
- Differences from stock: none seen. Cursor's own Browser editor has a separate find box (context keys `browserFindWidgetVisible`, `browserFindWidgetFocused`, command names `showBrowserFind`, `nextBrowserFindMatch`, `previousBrowserFindMatch`, `hideBrowserFind`); that belongs to ED-10.
- Proof strings: `actions.find`, `editor.action.replaceOne`, `search.action.openNewEditor`, `workbench.editorinputs.searchEditorInput`.

### ED-4 Language help in the editor

- Plain name: Language help in the editor
- Tag: both
- Evidence: bundle
- Boundary: no
- What Cursor has: every stock id tested in this group is present (go to definition family 20 of 20, rename 6 of 6, go to error 5 of 5, format 8 of 8, code lens, links, word highlighter, color picker, hover `editor.action.showHover`, code actions `editor.action.quickFix`, `refactor`, `sourceAction`, `organizeImports`). Cursor's `nls.keys.json` lists localized modules under `vs/editor/contrib/` for `hover` (6 modules), `gotoSymbol` (7), `codeAction` (6), `rename` (2), `gotoError` (2), `colorPicker` (4), `wordHighlighter` (2), and one each for `inlayHints`, `codelens`, `peekView`.
- Differences from stock: none in the stock features. Cursor adds a delay setting for its Blame hovers, `cursor.blame.hoverDelay` ("Controls the delay in milliseconds before hovers in the Cursor Blame panes appear. If not set, uses `editor.hover.delay`", nls index 5389). Call hierarchy and type hierarchy commands are present (`editor.showCallHierarchy`, `editor.showTypeHierarchy`, listed in the tracked-command map) but those open views and are left out.
- Proof strings: `editor.action.revealDefinition`, `editor.action.peekDefinition`, `editor.action.referenceSearch.trigger`, `editor.action.rename`, `editor.action.formatDocument`, `editor.action.marker.next`, `codelens.showLensesInCurrentLine`, `editor.action.openLink`, `editor.action.wordHighlight.next`, `editor.action.hideColorPicker`, `workbench.action.gotoLine`, `workbench.action.gotoSymbol`, `workbench.action.showAllSymbols`.

### ED-5 Editor view and orientation aids

- Plain name: Editor view and orientation aids
- Tag: both
- Evidence: bundle
- Boundary: no
- What Cursor has: sticky scroll 7 of 7, folding 20 of 20, font zoom 3 of 3, unicode highlighter 7 of 7, the minimap toggle (`editor.action.toggleMinimap`), word wrap toggle (`editor.action.toggleWordWrap`), whitespace toggle (`editor.action.toggleRenderWhitespace`), control characters toggle, tab focus mode (`editor.action.toggleTabFocusMode`), breadcrumbs commands `breadcrumbs.toggle`, `breadcrumbs.focus`, `breadcrumbs.toggleToOn`, `breadcrumbs.focusAndSelect`; settings `breadcrumbs.enabled`, `editor.stickyScroll.enabled`, `editor.minimap.enabled`, `editor.wordWrap` found. nls module `vs/workbench/browser/parts/editor/breadcrumbsControl` has 11 strings.
- Differences from stock: `breadcrumbs.copyPath` not found as a quoted string; the five render-whitespace mode commands are also absent (ED-2 list). Cursor-only: a Markdown editor breadcrumb contribution, workbench contribution id `markdownEditor.breadcrumbContribution` (see ED-10).
- Proof strings: `editor.action.toggleStickyScroll`, `editor.action.goToFocusedStickyScrollLine`, `editor.foldAll`, `editor.fold`, `editor.unfold`, `editor.action.fontZoomIn`, `editor.action.toggleRenderControlCharacter`.

### ED-6 Diffs, merges and change review

- Plain name: Diffs, merges and change review
- Tag: both (the stock part), plus several `cursor` items (listed below)
- Evidence: bundle
- Boundary: yes - the inline diffs, Review Changes and Multi File Edits show the agent's edits and are driven by the agent. Cursor Blame and File Blame are not agent output (they show git history) but are gated by a feature gate and a team setting, so they are listed here with the same flag.
- What Cursor has of the stock part: diff editor ids 9 of 9, merge editor 28 of 30, multi-file diff editor 5 of 7, file compare commands 37 of 38, local history 13 of 13. Editor panes "Text Diff Editor" (index 3409), "Binary Diff Editor" (3410), "Side by Side Editor" (3411), "Merge Editor" (7812), "Multi Diff Editor" (7921), "Notebook Diff Editor" (two panes). `C\extensions\merge-conflict` has the same 10 commands, 4 settings and CodeLens behavior, plus one more (see the extension table). Merge-conflict analytics list `merge-conflict.accept.current` and `merge-conflict.accept.incoming` as tracked commands.
- Differences from stock: the four absent ids listed in the coverage table; no stock pane is missing.
- Cursor additions in this group (details in the "Cursor additions" section):
  - Inline diffs in the code editor (`vs/editor/contrib/inlineDiffs`): command family `editor.action.inlineDiffs.*`, context keys `inlineDiffs.activeEditorWithDiffs` and `inlineDiffs.hasInlineDiffsInAnyEditor`, `workbench.action.toggleInlineDiffs` ("Toggle Inline Diffs").
  - Review Changes editor (`workbench.editor.reviewchanges`).
  - Multi File Edits editor (`inlineMultiDiffEditor`).
  - Cursor Blame editor and File Blame editor (`workbench.editor.cursorblame`, `workbench.editor.cursorfileblame`).
- Proof strings: `diffEditor.toggleCollapseUnchangedRegions`, `diffEditor.toggleShowMovedCodeBlocks`, `diffEditor.switchSide`, `diffEditor.revert`, `workbench.action.compareEditor.focusOtherSide`, `workbench.files.action.compareWithSaved`, `workbench.files.action.compareWithClipboard`, `multiDiffEditor.collapseAll`, `multiDiffEditor.expandAll`, `merge.goToNextUnhandledConflict`, `mergeEditor.acceptMerge`, `merge.acceptAllInput1`, `workbench.action.localHistory.compareWithFile`.

### ED-7 Notebooks, Interactive Window and REPL

- Plain name: Notebooks, Interactive Window and REPL
- Tag: both
- Evidence: bundle
- Boundary: no
- What Cursor has: editor panes "Notebook Editor", "Notebook Diff Editor" (registered twice), "Interactive Window", "REPL Editor"; pane ids `workbench.editor.notebook`, `workbench.editor.notebookTextDiffEditor`, `workbench.editor.notebookMultiTextDiffEditor`, `workbench.editor.interactive`, `workbench.editor.repl`; commands `notebook.cell.execute`, `notebook.execute`, `interactive.execute`, `repl.execute`. The 79 of 92 stock notebook ids found include the cell insert, delete, move, split, join, collapse, output and diff commands. `C\extensions\ipynb`, `notebook-renderers` and `markdown-math` have the same manifests as stock. 71 notebook modules appear in `nls.keys.json`.
- Differences from stock:
  - Not found: the notebook chat ids (cell chat, Add Cell Output to Chat, inline chat accept and run), and the Notebook Output Editor pane id `notebookOutputEditor`.
  - Cursor-only: the extension `C\extensions\vscode-jupyter-keymap` (17 keybindings; no stock copy in V `extensions`).
  - Tracked editor inputs: `workbench.input.interactive` (labeled `interactive_window`), `workbench.editorinputs.replEditorInput` (`repl_editor`), `workbench.input.notebook` (`notebook_editor`).
- Proof strings: `notebook.cell.insertCodeCellBelow`, `notebook.cell.split`, `notebook.clearAllCellsOutputs`, `notebook.diff.action.next`, `_interactive.open` (Open Interactive Window), `interactive.execute`, `repl.focusLastItemExecuted`, `ipynb.newUntitledIpynb`. The public-looking id `interactive.open` is not a command id in stock either; the command is `_interactive.open`.

### ED-8 Settings, shortcuts and configuration editors

- Plain name: Settings, shortcuts and configuration editors
- Tag: both
- Evidence: bundle
- Boundary: yes - Cursor Settings and Customize configure the agent (rules, skills, subagents, hooks, MCP tools, cloud agents, models). The editor panes themselves are in scope; the agent behind them is not.
- What Cursor has of the stock part: panes "Settings Editor 2" (nls module `vs/workbench/contrib/preferences/browser/settingsEditorPane.contribution`, index 8686), "Keybindings Editor" (module `preferences.view.contribution`, index 8627), "Profiles Editor" (11529), "Workspace Trust Editor" (11887). Command ids found: `workbench.action.openSettings`, `openSettings2`, `openSettingsJson`, `openApplicationSettingsJson`, `openWorkspaceSettings`, `openWorkspaceSettingsFile`, `openFolderSettings`, `openRemoteSettings`, `openAccessibilitySettings`, `openGlobalKeybindings`, `openGlobalKeybindingsFile`, `openDefaultKeybindingsFile`, `openRawDefaultSettings`, `configureLanguageBasedSettings`; `keybindings.editor.searchKeybindings`, `recordSearchKeys`, `showConflicts`; `settings.action.search`; `workbench.profiles.actions.manageProfiles`; `workbench.trust.manage`. The stock "Preferences Editor" pane label and `workbench.action.openKeyboardLayoutPicker` were not found.
- Differences from stock: Cursor re-binds the settings keys to its own page (see Cursor Settings below): `Ctrl+,` (when not in the Agents window) and `Ctrl+Shift+J` are keybindings on the Cursor Settings command `aiSettings.action.open`. Stock VS Code Settings is reached from the Cursor Settings tab "VS Code Settings" (which runs `workbench.action.openSettings`) or the command `cursor.openVSCodeSettingsFromMenu` ("Open VS Code Settings"). The command `workbench.action.openSettings` itself is unchanged (title "Open Settings (UI)", index 8605).
- Cursor additions in this group (details below): Cursor Settings (`workbench.editor.aisettings`), Customize (`workbench.editor.customize`).
- Proof strings: `aiSettings.action.open`, `cursor.openCursorSettings`, `workbench.action.openCustomizeEditor`, `workbench.action.customize.openRules`, `workbench.action.openSettings`, `keybindings.editor.showConflicts`.

### ED-9 Welcome, empty and first-run screens

- Plain name: Welcome, empty and first-run screens
- Tag: both (the Welcome page and the watermark exist in both; Cursor changed what they show)
- Evidence: bundle
- Boundary: no (the empty-screen entries "steer from cloud" and "New Agent" start agent flows, but the screen itself is not agent output)
- What Cursor has of the stock part: pane "Welcome" (nls module `vs/workbench/contrib/welcomeGettingStarted/browser/gettingStarted.contribution`, index 11678), command `welcome.goBack`, `workbench.action.openWalkthrough`, `welcome.markStepComplete` and `welcome.markStepIncomplete`, setting `workbench.startupEditor` with six values (none, welcomePage, readme, newUntitledFile, welcomePageInEmptyWorkbench, terminal; indexes 11683 to 11689), editor input tracked as `workbench.editors.gettingStartedInput` (labeled `walkthroughs`). The empty-group watermark module `vs/workbench/browser/parts/editor/editorGroupWatermark` and setting `workbench.tips.enabled`; the empty text hint setting `workbench.editor.empty.hint`. The new-file picker modules `welcomeViews/common/newFile.contribution`.
- Differences from stock:
  - Welcome page content is much thinner: 25 of 43 stock getting-started ids are missing, including every built-in walkthrough step and category (themes, Git setup, accessibility, web). The start entries (New File..., Open..., Open File..., Open Folder..., Clone Git Repository...) are present (nls indexes 11789 to 11800). What the page looks like on screen was not checked.
  - Not found in Cursor: Interactive Playground (`workbench.action.showInteractivePlayground`, pane "Playground"), Agent Sessions Welcome (`workbench.action.openAgentSessionsWelcome`, `agentSessionsWelcomePage` startup value), Welcome Onboarding 2026 (`workbench.action.welcomeOnboarding2026`), the Survey pane.
  - Cursor-only empty editor screen (module `vs/workbench/browser/parts/editor/renderEditorEmptyScreen`, event prefix `editor.landing_screen_empty`): shown when no editor is open. It has buttons New Project (`workbench.action.files.newProject`), Open Project (`workbench.action.files.openFolder`), Clone Repo (module `cloneRepoFromEmptyScreen`; needs Git, offers "Download Git"), Connect via SSH (looks for `ms-vscode-remote.remote-ssh`, `jeanp413.open-remote-ssh`, `anysphere.remote-ssh`, otherwise opens the Extensions view with `@recommended:remotes`), New Window (`workbench.action.newWindow`), a "connect git repos" button and a "steer from cloud" button, a sign-in or plan badge (Enterprise or Team, Pro, Pro+, Ultra, Pro Trial, Free Plan, with Settings and upgrade links), and a recent projects list (first 5 shown, a "view all" link, context menu "Remove from recents" = nls 3831). Opening a recent project opens a window (Ctrl or Cmd opens a new window). Cloud-connect buttons are gated by feature flags (`ide_connect_git_repos_start`, `ide_steer_from_cloud_start`).
  - Cursor watermark (`editorGroupWatermark` nls indexes 3735 to 3743): Search Files, Open Browser, New Agent, Show Terminal, Hide Terminal, Show Files, Add Folder, Add Repository, Maximize Chat. Stock shows Show All Commands, Go to File, Open File, Open Folder, Open Recent, and similar.
  - First-run onboarding flow: analytics events `onboarding.started`, `onboarding.welcome.viewed`, `onboarding.welcome.login_clicked`, `onboarding.welcome.signup_clicked`, `onboarding.welcome.continue_clicked`, `onboarding.data_sharing.viewed`, `onboarding.data_sharing.choice_made`, `onboarding.marketplace.viewed`, `onboarding.marketplace.skip_clicked`, `onboarding.navigation.back_clicked`. Where it renders (editor area, overlay, or its own window) was not determined. A related toast "Bring your VS Code setup to Cursor. Import your settings and extensions for a seamless start." (module `vs/workbench/contrib/onboarding/electron-sandbox/onboarding.contribution`) is a notification and belongs to section 10.
- Proof strings: `welcome.goBack`, `workbench.action.openWalkthrough`, `workbench.startupEditor`, `workbench.tips.enabled`, `workbench.editor.empty.hint`, `workbench.action.files.newProject`, `workbench.action.newWindow`, `editor.landing_screen_empty.viewed`.

### ED-10 Viewers, previews and web editors

- Plain name: Viewers, previews and web editors
- Tag: both (stock-style viewers); several `cursor` items are listed below
- Evidence: bundle
- Boundary: yes - the Cursor Browser can be driven by the agent and can send picked elements to chat, and its code sits in the composer module; Global Command Preview shows an agent command. The viewers themselves are in scope.
- What Cursor has of the stock part: panes "Webview Editor" (two registrations; stock label "webview editor", index 11677), "Binary File Editor" (7221), "Text File Editor" (7220); `C\extensions\media-preview` (3 custom editors: Image, Audio, Video preview), `C\extensions\markdown-language-features` (preview), `C\extensions\simple-browser` (command `simpleBrowser.show`, listed in the tracked-command map as `simple_browser`). Webview editor ids `editor.action.webvieweditor.showFind`, `workbench.action.webview.reloadWebviewAction`, `workbench.action.webview.openDeveloperTools`; custom editor ids `workbench.action.reopenWithEditor`.
- Differences from stock:
  - The stock integrated Browser is not what Cursor ships: all 14 stock ids are absent (`workbench.action.browser.zoomIn`, `toggleDeviceEmulation`, `browser.toggleFavorite`, `workbench.editorinputs.browser`, and so on) and no `workbench.browser.*` setting names were found. Cursor has its own Browser editor, in Cursor additions below. Pane label "Browser" is the same in both.
  - Not found: Images Preview (stock image carousel, all 5 ids absent), and the Markdown extension's newer commands (see the extension table).
  - Cursor's `media-preview` adds `imagePreview.saveImageAs` ("Save As..."), and has no `reopenAsPreview` or `reopenAsText`.
  - The Mermaid markdown extension `mermaid-markdown-features` is not in `C\extensions`.
  - Debug Disassembly pane: label "Disassembly" (index 5650, module `debug.contribution`) and command `debug.action.openDisassemblyView` are present.
- Cursor additions in this group (details below): Browser (`workbench.editor.browserEditor`), Markdown Editor (`workbench.editor.markdown`), Mermaid Editor (`workbench.editor.mermaid`), Global Command Preview (`workbench.editor.globalCommandPreview`).
- Proof strings: `markdown.showPreview`, `markdown.showPreviewToSide`, `markdown.showLockedPreviewToSide`, `simpleBrowser.show`, `editor.action.webvieweditor.showFind`, `workbench.action.openBrowserEditor`, `workbench.action.newBrowserTab`, `markdownEditor.toggleMode`.

## Cursor additions (not in stock VS Code 1.128.0)

Each entry: group, tag, evidence, boundary, then the strings found. Evidence is `bundle` for all of them. Tag is `cursor` unless noted.

### Browser (ED-10; also adds an entry to ED-1 and ED-9)

- Tag: cursor (the stock integrated Browser is a different implementation; see ED-10). Boundary: yes - agent can drive it and "select element" sends to chat; the code sits in the composer module (`vs/workbench/contrib/composer/browser/browserEditor.contribution`).
- Pane label "Browser"; pane id `workbench.editor.browserEditor`; input type `workbench.input.browserEditor`; tab class `browser-editor-tab`.
- Commands: Open Browser `workbench.action.openBrowserEditor` (also an editor title globe button and a watermark entry); New Browser Tab `workbench.action.newBrowserTab` (Ctrl+T while the Browser editor is active); Reload Browser Tab `workbench.action.reloadBrowserTab` (Ctrl+R); Focus Browser Location Bar `workbench.action.focusBrowserLocationBar` (Ctrl+L); Open cursor.com in Browser (nls index 5307, opens the configured default URL, falling back to `https://cursor.com`). All four category View, precondition context key `cursor.browserTabEnabled`.
- Context keys: `browserEditorActive`, `browserViewFocused`, `browserFindWidgetVisible`, `browserFindWidgetFocused`.
- Context menu strings (16): `browser.contextMenu.back`, `forward`, `reload`, `cut`, `copy`, `copyImage`, `copyLinkAddress`, `copyMediaAddress`, `inspectElement`, `openLinkNewTab`, `openLinkBackgroundTab`, `openMediaNewTab`, `paste`, `redo`, `selectAll`, `undo`.
- Toolbar actions seen in the analytics calls: back, forward, refresh, screenshot, select element (selection type `composer-chat`), toggle DevTools. Stored state names: `browserAutomation.bookmarks`, `bookmarkBarVisible`, `history`, `hostZoomLevels`, `lastUrl`, `zoomLevel`.
- Not searched for: the Browser's visible toolbar layout and URL bar behavior (strings only).

### Customize (ED-8)

- Tag: cursor (a stock chat-side "Agent Customizations Editor" exists in V under chat, but its label is not in Cursor). Boundary: yes - manages agent plugins, tools, skills, subagents, rules, commands, hooks.
- Pane label "Customize"; pane id `workbench.editor.customize` (input `workbench.editor.customize.input`); URI scheme `cursor-customize`; label icon id `customize-editor-label-icon`.
- Commands: Open Customize `workbench.action.openCustomizeEditor` (nls 5452); Focus Customize Search `workbench.action.customize.focusSearch` (5455); tab openers `workbench.action.customize.openPlugins`, `openMCPs`, `openSkills`, `openSubagents`, `openRules`, `openCommands`, `openHooks` (labels "Open Plugins", "Open MCPs", "Open Skills", "Open Subagents", "Open Rules", "Open Commands", "Open Hooks"; tab ids plugins, mcps, skills, agents, rules, commands, hooks); `workbench.action.customize.openSkillPublishLogs` ("Open Skill Publishing Logs"); editor title menu "Skill Publishing" (nls 5464, module `skillPublishEditorTitle`).
- Developer-only commands in the same module: Clear Primitive Authorship Cache, Clear Primitive Source Cache, Force Skill Migration Banner, Clear Forced Skill Migration Banner.

### Cursor Blame and File Blame (ED-6)

- Tag: cursor. Boundary: yes - not agent output, but behind feature gate `cursor_blame` and a team setting (`isCursorBlameEnabledForTeam`), so it may not appear for every user.
- Pane labels "Cursor Blame Editor" (`workbench.editor.cursorblame`) and "Cursor File Blame Editor" (`workbench.editor.cursorfileblame`); URI schemes `cursor.blame` and `cursor.fileblame`.
- Commands: `cursorBlame.openFileBlame` ("Open File Blame", category "Cursor Blame", Command Palette); `cursorBlame.openCommit` ("View Commit", prompts "Paste commit hash"); `cursorBlame.viewCommit` ("View commit"); `workbench.scm.action.graph.openInCursorBlame` (Source Control history item context menu, group `0_view`).
- Menus: editor context submenu "Cursor Blame" (new menu id `EditorContextCursorBlame`, group `12_cursor_blame`, shown when the editor has focus and the file is on disk) with "Toggle file blame" (runs `cursorBlame.openFileBlame`) and "View commit"; Explorer context entry "Cursor Blame" (nls key `cursorBlame.explorerContextMenu`).
- Setting: `cursor.blame.hoverDelay`. Messages: "No git repository found", "This line has not been committed yet".
- The editor input holds `commitId`, `subject`, `message`, `repoRoot` for the commit view and `fileUri`, `filePath`, `revisionTimestamp` for the file view.

### Review Changes (ED-6)

- Tag: cursor. Boundary: yes - opened from the agent composer and from Bugbot, as well as from Source Control.
- Pane label "Review Changes Editor"; pane id `workbench.editor.reviewchanges` (input `.input`); URI scheme `cursor.reviewchanges`; resolver label "Review Changes" (priority builtin); context key `activeEditorIsReviewChanges` (nls 4349).
- Entry points found: Source Control view title button `workbench.scm.action.openReviewChangesPane` (title "Review Branch", nls 9381, icon diff, group navigation, order -900, hidden from the palette) which opens the editor in git mode with the label "Diff with Default Branch" or "Diff with {branch}" (9382, 9383); a "Review Changes" button in the composer (analytics `review_changes.opened`, entry point composer); Bugbot (option `fromBugbot`).
- Inside the editor: a Commit action with messages "Commit is unavailable until there are reviewable changes" and "Install Git to enable committing from Review Changes".

### Multi File Edits (ED-6)

- Tag: cursor. Boundary: yes - created for the agent's multi-file edits.
- Pane label "Multi File Edits"; pane id `inlineMultiDiffEditor`; input type `workbench.input.inlineMultiDiffEditor`; resource `inline-multi-diff-editor:<id>`; not serializable (it is not restored on reload); tracked as `inline_multi_diff_editor`.
- It is created through the editor resolver by a method `createInlineMultiDiffEditorInput` that takes a label and a list of resources; the tab name falls back to "Multi File Edits". The caller that opens it was not located.

### Global Command Preview (ED-10)

- Tag: cursor. Boundary: yes - it previews a Cursor command that agents can run.
- Pane label "Global Command Preview"; pane id `globalCommandPreviewEditor`; editor id `workbench.editor.globalCommandPreview`; input `workbench.input.globalCommandPreview`; URI scheme `cursor-command-preview` (authority `global`).
- Content is read-only markdown: a title `# /<command name>`, the description, then "Source: Global command" and "Availability: All agents, Cloud only, or Local only". It opens when a global (not file-backed) command is opened from the commands list, except in the Agents window. Not serializable. No command id of its own was found.

### Inline diffs in the code editor (ED-6)

- Tag: cursor. Boundary: yes - the diffs are the agent's edits drawn inside the normal text editor.
- Module `vs/editor/contrib/inlineDiffs/browser/controllers/inlineDiffController` (nls 1381 and 1382); bundle module names `inlineDiffController.js`, `inlineDiffDeletedCodeMargin.js`, `inlineDiffPartialWidget.js`, `inlineDiffWidget.css`, class `div.inlineDiffViewZone`; editor contribution id `editor.contrib.inlineDiffController`.
- Context keys: `inlineDiffs.activeEditorWithDiffs` ("Whether the active editor has inline diffs"), `inlineDiffs.hasInlineDiffsInAnyEditor`.
- Commands (14 ids): `editor.action.inlineDiffs.acceptAll` ("Accept Edits", Ctrl+Enter), `acceptPartialEdit` ("Accept Partial Edit", Ctrl+Y, or Ctrl+Shift+Y on one platform branch of the code; the flag was not resolved), `rejectAll` ("Undo Edits", Ctrl+Shift+Backspace), `rejectPartialEdit` ("Reject Partial Edit", Ctrl+N), `nextChange` ("Go to Next Change", Alt+J), `previousChange` ("Go to Previous Change", Alt+K), `nextDiffFile` ("Go to Next Diff File", Alt+L), `previousDiffFile` ("Go to Previous Diff File", Alt+H), plus `rejectAllAcrossAllEditors`, `cancelEdits`, `cancelPromptBar`, `focusEditor`, `submitFix`, `viewAllChanges` (no label captured). All keys fire with the editor focused.
- `workbench.action.toggleInlineDiffs` ("Toggle Inline Diffs", description "Toggle visibility of inline diffs in the editor", nls 3163 and 3090; module `browser/actions/layoutActions`, Command Palette).
- Setting `cursor.inlineDiff.enablePerformanceProtection`. Notification when there are too many changes: "Inline diffs have been suppressed for recent changes because there are too many to display." with a "Disable Protection" action (module `contrib/composer/browser/inlineDiffSuppressionNotification`).

### Cursor Settings (ED-8)

- Tag: cursor. Boundary: yes - most tabs configure the agent, models, cloud agents, MCP and plans.
- Pane label "Cursor Settings"; pane id `workbench.editor.aisettings` (input `.input`); URI scheme `cursor.aisettings`; label icon `settings-editor-label-icon`.
- Commands: `aiSettings.action.open` (title "Cursor Settings", Command Palette, File > Preferences menu group `2_configuration` order 1.5, keys Ctrl+Shift+J and Ctrl+, when not in the Agents window); `cursor.openCursorSettings` (same page, accepts a tab id and an element to scroll to, for example `{settingsTab:"general", idToScrollTo:"privacy-mode-section"}`); `aiSettings.action.open.<tab>` titled "Cursor Settings: <tab name>" for the tabs general, chat, background-composer, models, git-prs, rules, mcp, hooks, browser, beta (derived from the registration loop in the bundle, not run); "Focus AI Settings Search" (Ctrl+F while the page is focused); `cursor.openVSCodeSettingsFromMenu` ("Open VS Code Settings").
- Context keys: `isSettingsPaneOpen`, `isSettingsPaneFocused`.
- Tabs (id, label): general General; vscode-settings VS Code Settings (opens stock `workbench.action.openSettings`); appearance Appearance; plan-usage Plan & Usage; chat Agents; background-composer Cloud Agents; models Models; voice Voice; git-prs Source Control; customize Customize; plugins Plugins; rules Rules, Skills, Subagents; mcp Tools & MCPs; hooks Hooks; browser Browser & Network; indexing Code Intelligence; beta Beta; developer Developer. Also defined but hidden by default in the visibility function: fun Fun, tab Tab, network Network, worktrees Worktrees. Footer links: Docs (`https://cursor.com/docs`), Contact (`mailto:hi@cursor.com`). Some tabs show only under conditions (plan-usage when signed in, developer when developer tools are on, voice, customize, appearance each behind a flag).
- Whether the page renders as an editor tab or as an overlay popup was not determined: the command calls an `aiSettingsService.openPopup(tab, id)` method while a pane for `workbench.editor.aisettings` also exists.

### Markdown Editor (ED-10)

- Tag: cursor (stock has a different "Markdown Editor (Experimental)" custom editor in the Markdown extension). Boundary: no.
- Pane label "Markdown Editor"; pane id `workbench.editor.markdown`; input `workbench.input.markdown`; registered for `*.md` (file and user-data scheme) with default priority; skipped for `.plan.md` files and for anything under a `.cursor`, `.claude` or `.codex` folder (user home and workspace roots).
- Modes Rich and Raw. Commands: `markdownEditor.toggleMode` ("Toggle Editor Mode", category "Markdown Editor", Ctrl+Shift+V when `markdownEditorActive`), `markdownEditor.editMarkdown` ("Edit Markdown", on the editor tab context menu for `.md` files, group `2_edit`). nls also has "Edit Source" (7620, use not located). State keys `markdownEditorModePreferences`, context key `markdownEditorActive`, breadcrumb contribution `markdownEditor.breadcrumbContribution`.

### Mermaid Editor (ED-10)

- Tag: cursor (stock handles Mermaid inside Markdown previews through `mermaid-markdown-features`, which Cursor does not ship). Boundary: no.
- Pane label "Mermaid Editor"; pane id `workbench.editor.mermaid`; input `workbench.input.mermaid`; registered for `*.mmd` and `*.mermaid`; icon graph; built on the Markdown Editor input class; tracked as `mermaid_editor`.

### Empty editor screen and watermark (ED-9)

- Tag: cursor (the stock watermark exists, Cursor replaced its entries and added a landing screen). Boundary: no.
- Strings are in the ED-9 section above (modules `renderEditorEmptyScreen`, `cloneRepoFromEmptyScreen`, `editorGroupWatermark`).

### Other editor-area additions

- "Copy cursor.com link" (with selection, or file only) on editor context, editor title, line-number gutter and Explorer menus (ED-1, ED-2). Tag cursor, boundary no. Module `vs/workbench/contrib/cursorOrigin/browser/cursorOrigin.contribution`.
- "Reset Editor Group Sizes (Excluding Agent)" (ED-1), see above.
- `merge-conflict.resolve-in-chat` ("Resolve in Chat") in the merge-conflict extension (ED-6). Tag cursor. Boundary: yes (hands the conflict to chat).

## Built-in extensions in Cursor that belong in this section

Cursor's extension manifests all report version 1.0.0 (stock reports 10.0.0). Counts are from `package.json` `contributes`.

| Extension | Cursor | Stock (V) | Notes |
|-|-|-|-|
| markdown-language-features | 12 commands, 2 keybindings, 40 settings, 1 custom editor | 16, 2, 41, 2 custom editors | Cursor lacks `_markdown.openFrontMatterSettings`, `markdown.reopenAsPreview`, `markdown.reopenAsSource`, `markdown.togglePreview`, setting `markdown.preview.frontMatter`, the `modalEditor/editorTitle` menu, and the "Markdown Editor (Experimental)" custom editor. Cursor binds Shift+Ctrl+V to `markdown.showPreview` (when `editorLangId == markdown`); stock binds it to `markdown.togglePreview`. |
| merge-conflict | 11 commands | 10 | Cursor adds `merge-conflict.resolve-in-chat` and a Command Palette entry |
| simple-browser | 1 command, no settings, no menus | 1 command, 1 setting, 1 menu | Cursor lacks `simpleBrowser.focusLockIndicator.enabled` |
| search-result | language and grammar only | same | identical |
| media-preview | 4 commands (`zoomIn`, `zoomOut`, `copyImage`, `saveImageAs`), 3 custom editors | 5 commands (no `saveImageAs`, adds `reopenAsPreview`, `reopenAsText`) | same 3 custom editors |
| ipynb, notebook-renderers, markdown-math, emmet | same command counts | same | identical |
| mermaid-markdown-features | not present | present | stock only |
| vscode-jupyter-keymap | 17 keybindings | not present | Cursor only |
| references-view, git | present | present | left out here (section 04) |

## Editor panes registered in the Cursor bundle

I found 40 `registerEditorPane` calls by pattern. Grouped by what they are:

- Stock panes found (label, then index if localized): Text Editor (3408), Text Diff Editor (3409), Binary Diff Editor (3410), Side by Side Editor (3411), Text File Editor (7220), Binary File Editor (7221), Settings Editor 2 (8686), Keybindings Editor (8627), Welcome (11678), Search Editor (9682), Multi Diff Editor (7921), Merge Editor (7812), Notebook Editor, Notebook Diff Editor (two), Interactive Window, REPL Editor, Profiles Editor (11529), Workspace Trust Editor (11887), Webview Editor (two, one is index 11677), Disassembly (5650).
- Stock panes not found: Preferences Editor, Playground, Agent Sessions Welcome, Survey, Issue Reporter, Process Explorer (window title string only), Images Preview, Notebook Output Editor.
- Stock panes found but left out here: Extension (6357), Running Extensions (6964).
- Cursor-only panes in scope: Browser, Customize, Cursor Settings, Cursor Blame Editor, Cursor File Blame Editor, Review Changes Editor, Mermaid Editor, Markdown Editor, Multi File Edits, Global Command Preview.
- Cursor-only panes out of scope: Cursor AI (5313, the Agent chat as an editor), Voice call (5324), Canvas, Markdown Plan Editor, Automations (unclear, below).
- One pane registered under the terminal icon id (the terminal editor, a terminal opened in the editor area): stock has it too; it is section 07.

## Left out and why

- Autocomplete: suggest list, inline completions and ghost text, parameter hints, signature help, snippet placeholders, and Cursor Tab (`editor.cpp.snooze`, `editor.cpp.unsnooze`, `editor.cpp.toggle`, `editor.cpp.disableenabled`, `editor.action.enableCppGlobally`) - section 06.
- Agent chat as an editor (`workbench.editor.composer`, `workbench.editor.chatSession`, the "Cursor AI" pane), voice call transcript editor (`workbench.editor.voiceCallTranscript`) - chat is out of scope.
- Canvas (`workbench.editor.canvas`) and Markdown Plan (`workbench.editor.markdownPlan`, command `planEditor.toggleMode`) - cut from scope by the user.
- Extension and Running Extensions editors (`workbench.editor.extension`, `runtimeExtensions`) - extension system.
- Editor menu entries that hand text to chat: `editor.action.addSymbolToChat`, `editor.action.addSymbolToNewChat`, `cursor.createRuleFromSelection`, `composer.addfilestocomposer`, `composer.addsymbolstocomposer`, `composer.fixerrormessage`, `editor.action.knowledgeBase.add`, `editor.action.knowledgeBase.list`, "Add to Chat" on Problems and file references - chat entry points.
- Inline edit prompt bar (Cmd+K) and the commands tracked as `cmdk_extras` - inline chat is out of scope. By name, two ids in the inline diff family (`cancelPromptBar`, `submitFix`) belong to that prompt bar; they are listed in the ED-6 inline diff entry only because they share the prefix.
- Agents window commands (`glass.*`, for example `glass.newBrowser`, `glass.openBrowserTab`, `glass.showBrowserFind`) - the Agents window is out of scope.
- `C\extensions\cursor-browser-automation` and the other `cursor-*` agent plumbing extensions - not opened; they drive the Browser for the agent and are not UI.
- File menu "New Cursor Rule" (`workbench.action.newCursorRule`), "New Project Cursor Command" (`workbench.action.newProjectCursorCommand`), "New User Cursor Command" (`workbench.action.newUserCursorCommand`) - they start agent rule and command authoring; they belong with Customize but their flow runs through chat.
- Agent Review view in Source Control (module `scm/browser/editorBugbotViewPane`, title "Review", view "Agent Review") and `workbench.scm.action.openReviewChangesPane` as a Source Control button - the button is named above, the view is section 04.
- Onboarding toast "Bring your VS Code setup to Cursor" - a notification, section 10.
- Status bar and title bar entries (language and encoding pickers, "Open Cursor Settings" in the title bar, Update control) - sections 09 and 01.
- Issue Reporter and Process Explorer - not found as editor panes in Cursor; no section assigned.
- Debug editor widgets, comment threads, test glyphs, accessible view - same coverage gap as in the stock file.

## Unclear

- Automations (`workbench.editor.automations`): pane label "Automations", input `workbench.editor.automations.input`, URI scheme `cursor-automations`, icon robot, command `workbench.action.openAutomationsEditor` ("Open Automations", nls 4889), "Save Automation" (4890), analytics key `ide_automations_tab_no_search`, module `vs/workbench/contrib/automationsEditor/browser/automationsEditor.contribution`. Its UI lives in the separate bundle `workbench.anysphere-ui-automations.js`, which I did not read. Recorded only; no group made.
- `cursor.tinderdiffeditor`: a URI scheme named in the scheme list next to `cursor.aisettings` and `cursor.reviewchanges`. No pane, label or command with that name was found.
- CSV table mode: storage keys `csvTableModePreferences` and `csvTableDefaultMode` exist in the bundle next to the Markdown editor mode keys, but no CSV editor registration or pane was found.
- Editor "Edit Source" (nls 7620) and the "Review" label from `editorBugbotViewPane` - purpose or surface not determined.

## Newer than 1.128

- Nothing in the Cursor bundle is newer than its 1.128.0 base that I could tell. I searched the bundle for the names of the stock-only-in-N items listed in the stock file (`connectedEditorTabs`, `EditorAddTabControl`, `EditorHeaderControl`, `editorTypePicker`, `multiDiffEditorFileList`, `mediaPreview.contribution`, `browserAutoReload`, `workbench.editor.tabActionReserveSpace`) and found none.

## Could not verify

- On-screen look of anything here. All Cursor evidence is strings and module names; no running Cursor was inspected.
- Default state: whether the Markdown Editor, Cursor Blame, the empty-screen cloud buttons and the Customize tab show for every user. Several are behind feature gates or experiments (`cursor_blame`, `ide_connect_git_repos_start`, `ide_steer_from_cloud_start`, `customizeSidebarTabEnabled`).
- Ids that stock builds from parts at run time (for example `workbench.action.openEditorAtIndex1` to 9) cannot be matched by an exact-string search; I counted such ids as untested, not absent.
- How the Cursor Settings page is hosted (editor tab versus popup), where first-run onboarding renders, and who opens Multi File Edits.
- Keybindings are given only where I read them in the bundle (Browser, inline diffs, Markdown Editor, Cursor Settings).
