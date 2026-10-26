# Entry point index

Status: pass 4. Every command, menu, setting, and key that the features name, with the features that own it. Generated from the feature files, so it always matches them.

86 groups, 2015 features, 2494 command ids, 281 menu ids, 1269 setting ids, 1171 key bindings.

How to use it: find an id, read the feature ids beside it, and open that feature in its section report (`ui-catalog-NN-name.md`). A command that appears in several features is an entry point to each of them. The full menu bar, item by item, is in `ui-catalog-menu-bar.md`.

## How complete it is

A script collected every command id, menu id, and setting that the in-scope sources register: the VS Code 1.128.0 workbench, the six in-scope built-in extensions, and the ids in the Cursor bundle that stock VS Code lacks. It then subtracted everything a feature already named. The leftovers (531 ids) were each mapped to a feature, turned into a new feature, or excluded with a reason, and the three `_markdown.*` ids the script missed because of the leading underscore were mapped too.

| Source | In scope | Named by a feature before the check | Left over |
|---|---|---|---|
| Core workbench commands | 1325 | 1183 | 142 |
| Built-in extension commands | 231 | 208 | 23 |
| Core settings | 825 | 653 | 172 |
| Built-in extension settings | 151 | 129 | 22 |
| Menu ids | 258 | 177 | 81 (30 after dropping chat and extension menus) |
| Cursor-only command ids (not in stock) | 469 | 297 | 172 |

Two further checks followed. A label check collected 1,707 user-facing action and menu titles from the in-scope sources and looked for each one in the features and pieces (hand-off notes do not count). The 424 titles that did not appear were each mapped, added as a feature, or excluded. A residual check then re-ran the id check against real features only, and resolved the 98 items that earlier passes had only mentioned in hand-off notes.

Core commands registered under out-of-scope folders (chat, inline chat, MCP, extensions, agent sessions, remote, and similar) were not collected: 625 ids. 76 Cursor-only ids with chat or agent in the name were also skipped.

The collector reads string literals and constants, so ids built at run time (for example `openEditorAtIndex1` through `9`) can be missed. Settings that are only editor options with no settings entry are counted when they appear in the editor options table.

## Commands

| Command id | Features |
|---|---|
| `<container` | AB-2.8 |
| `<containerId>.resetViewContainerLocation` | AB-6.10, SB-1.6 |
| `<viewId>.focus` | SB-1.19 |
| `<viewId>.removeView` | SB-1.9 |
| `<viewId>.resetViewLocation` | SB-1.17 |
| `<viewId>.toggleVisibility` | AB-6.10, SB-1.9 |
| `_imagePreview.selectZoomLevel` | ED-10.1, SS-3.40 |
| `_json.configureTrustedDomains` | SS-3.51 |
| `_json.retryResolveSchema` | SS-3.51, SS-3.52 |
| `_json.showAssociatedSchemaList` | SS-3.48, SS-3.49 |
| `_lineSelect` | ED-2.6 |
| `_lineSelectDrag` | ED-2.40 |
| `_manageAccountPreferencesForExtension` | AB-4.4 |
| `_manageTrustedExtensionsForAccount` | AB-4.3 |
| `_markdown.copyImage` | ED-10.16 |
| `_markdown.openFrontMatterSettings` | ED-10.17 |
| `_markdown.openImage` | ED-10.16 |
| `_markdown.preview.showRenderedDiffWarning` | SS-3.39 |
| `_markdown.toggleValidation` | SS-3.53 |
| `_mermaid-markdown.copySource` | ED-10.22 |
| `_mermaid-markdown.openInEditor` | ED-10.22 |
| `_mermaid-markdown.resetPanZoom` | ED-10.22 |
| `_moveTo` | ED-2.40 |
| `_moveToSelect` | ED-2.40 |
| `_notebook.selectKernel` | ED-7.19, ED-7.26, ED-7.38, SS-3.33 |
| `_open.mergeEditor` | ED-6.17 |
| `_revealTestInExplorer` | PN-6.29 |
| `_signOutOfAccount` | AB-4.2 |
| `_typescript.createOrOpenConfig` | SS-3.45 |
| `_typescript.openConfig` | SS-3.45 |
| `_update.applyupdate` | WB-8.23 |
| `_update.showUpdateInfo` | NT-6.13 |
| `_wordSelect` | ED-2.6 |
| `_wordSelectDrag` | ED-2.40 |
| `_workbench.action.openFolderSettings` | ED-8.3 |
| `_workbench.diff` | ED-6.1 |
| `_workbench.openMultiDiffEditor` | SB-4.50, ED-6.13 |
| `_workbench.openScmMultiDiffEditor` | ED-6.13 |
| `_workbench.openUserSettingsEditor` | ED-8.8 |
| `acceptAlternativeSelectedSuggestion` | AC-1.28, AC-1.34 |
| `acceptRenameInput` | ED-4.12 |
| `acceptRenameInputWithPreview` | ED-4.12 |
| `acceptSelectedCodeAction` | ED-4.14 |
| `acceptSelectedSuggestion` | AC-1.28, AC-1.31, AC-1.32 |
| `acceptSnippet` | AC-4.7 |
| `accessibility.announcement.help` | WB-4.5 |
| `actions.find` | ED-3.1, ED-6.38, ED-7.30 |
| `actions.findWithSelection` | ED-3.3 |
| `addCursorsAtSearchResults` | SB-3.22 |
| `addRootFolder` | SB-2.19, DL-3.5 |
| `aiCodeTracking.resetDailyStats` | SS-6.10 |
| `aiFeedback.action.open` | WB-6.6 |
| `aiServerConfigService.forceRefresh` | WB-8.19 |
| `aiSettings.action.focusSearch` | ED-8.19, ED-8.61 |
| `aiSettings.action.open` | TB-5.6, ED-8.19, ED-8.22, ED-8.26, ED-8.30, ED-8.32, AC-8.10 |
| `aiSettings.action.open.<tab>` | ED-8.19 |
| `aiSettings.action.open.background-composer` | ED-8.24 |
| `aiSettings.action.open.beta` | ED-8.31 |
| `aiSettings.action.open.browser` | ED-8.29 |
| `aiSettings.action.open.chat` | ED-8.23 |
| `aiSettings.action.open.general` | ED-8.20 |
| `aiSettings.action.open.git-prs` | ED-8.27 |
| `aiSettings.action.open.hooks` | ED-8.40 |
| `aiSettings.action.open.mcp` | ED-8.35 |
| `aiSettings.action.open.models` | ED-8.25 |
| `aiSettings.action.open.rules` | ED-8.38 |
| `aiSettings.action.openhidden` | ED-8.19 |
| `and` | ED-10.21 |
| `authentication.command.anysphere.signOut` | AB-4.2 |
| `breadcrumbs.copyPath` | ED-5.10 |
| `breadcrumbs.focus` | ED-5.9 |
| `breadcrumbs.focusAndSelect` | ED-5.9 |
| `breadcrumbs.focusNext` | ED-5.9 |
| `breadcrumbs.focusNextWithPicker` | ED-5.9 |
| `breadcrumbs.focusPrevious` | ED-5.9 |
| `breadcrumbs.focusPreviousWithPicker` | ED-5.9 |
| `breadcrumbs.revealFocused` | ED-5.9 |
| `breadcrumbs.revealFocusedFromTreeAside` | ED-5.9 |
| `breadcrumbs.selectEditor` | ED-5.9 |
| `breadcrumbs.selectFocused` | ED-5.9 |
| `breadcrumbs.toggle` | ED-5.7 |
| `breadcrumbs.toggleFromEditorTitle` | ED-7.28 |
| `breadcrumbs.toggleToOn` | ED-5.7 |
| `breakpointWidget.action.acceptInput` | ED-11.5 |
| `browser.configureSearchEngine` | ED-10.68 |
| `browser.favorites.delete` | ED-10.37 |
| `browser.history.delete` | ED-10.65 |
| `browser.splitEditorWithNewBrowserTab` | ED-10.90 |
| `browser.splitEditorWithNewBrowserTabToDirection` | ED-10.90 |
| `browser.toggleFavorite` | ED-10.37 |
| `callStack.collapse` | SB-5.19 |
| `callStackWidget.goToFile` | PN-6.25 |
| `cancelLinkedEditingInput` | ED-2.27 |
| `cancelRenameInput` | ED-4.12 |
| `cancelSelection` | ED-2.41 |
| `cellOutput.enableScrolling` | ED-7.22 |
| `changeEditorIndentation{windowId}` | SS-3.5 |
| `cleanSearchEditorState` | ED-3.39 |
| `clearFilterCodeActionWidget` | ED-4.14 |
| `closeBreakpointWidget` | ED-11.5 |
| `closeFindWidget` | ED-3.22 |
| `closeMarkersNavigation` | ED-4.27 |
| `closeParameterHints` | AC-3.10 |
| `closeQuickDiff` | ED-6.25 |
| `closeReferenceSearch` | ED-4.10 |
| `closeReplaceInFilesWidget` | SB-3.6 |
| `codelens.showLensesInCurrentLine` | ED-4.20 |
| `collapseSectionCodeAction` | ED-4.14 |
| `columnSelect` | ED-2.5 |
| `comments.collapse` | PN-7.6 |
| `comments.expand` | PN-7.6 |
| `comments.reply` | PN-7.7 |
| `commentsClearFilterText` | PN-7.3 |
| `commentsFocusFilter` | PN-7.3 |
| `commentsFocusViewFromFilter` | PN-7.3 |
| `compareFiles` | SB-2.17, SB-2.32, ED-6.11 |
| `compareSelected` | SB-2.17, SB-2.32, ED-6.11 |
| `composer.openBrowserTab` | ED-9.18, QA-1.21 |
| `compositionEnd` | ED-2.42 |
| `compositionStart` | ED-2.42 |
| `compositionType` | ED-2.42 |
| `copyFilePath` | SB-2.16, SB-2.32, ED-1.26 |
| `copyRelativeFilePath` | SB-2.16, SB-2.32, ED-1.26 |
| `cpuMonitor.triggerManualSnapshot` | WB-8.20 |
| `createCursor` | ED-2.2 |
| `Cursor` | ED-10.37 |
| `cursor.browserAutomation.captureWebviewScreenshot` | ED-10.53 |
| `cursor.browserAutomation.internal.captureScreenshot` | ED-10.53 |
| `cursor.browserView.debug.crashRenderer` | ED-10.75 |
| `cursor.browserView.debug.orphanView` | ED-10.75 |
| `cursor.browserView.getRecordingType` | ED-10.59 |
| `cursor.browserView.goBack` | ED-10.32 |
| `cursor.browserView.goForward` | ED-10.32 |
| `cursor.browserView.isLocked` | ED-10.59 |
| `cursor.browserView.reload` | ED-10.33 |
| `cursor.browserView.resize` | ED-10.60 |
| `cursor.browserView.setLocked` | ED-10.59 |
| `cursor.browserView.setRecordingType` | ED-10.59 |
| `cursor.checkonupdate` | NT-6.7 |
| `cursor.closeAgentChangesEditor` | ED-6.52 |
| `cursor.connectSlack` | ED-8.24 |
| `cursor.createRuleFromSelection` | ED-8.69 |
| `cursor.doupdate` | NT-6.4 |
| `cursor.enableAutoReviewOnCommit` | SB-4.39 |
| `cursor.generateGitCommitMessage` | SB-4.14 |
| `cursor.giveFeedback` | WB-6.6 |
| `cursor.installCli` | PN-2.50 |
| `cursor.openAgentChangesEditor` | ED-6.52 |
| `cursor.openBugbotPane` | SB-4.39 |
| `cursor.openCreatedRule` | ED-8.69 |
| `cursor.openCursorWebsite` | ED-10.28 |
| `cursor.openVSCodeSettingsFromMenu` | ED-8.21 |
| `cursor.publishSkill.copyLink` | ED-8.36 |
| `cursor.removevim` | ED-2.44 |
| `cursor.runEditorBugbot` | SB-4.39 |
| `cursor.selectBackend` | SS-6.13 |
| `cursor.showSubscriptionTiersModal` | DL-6.1 |
| `cursor.startOnboarding` | ED-9.16, ED-9.17 |
| `cursor.titlebar.installUpdate` | TB-5.1 |
| `cursor.titlebar.upgradeToPro` | TB-5.7 |
| `cursorai.action.hideNotificationsForMs` | NT-1.15 |
| `cursorai.action.importvscode` | ED-8.58, DL-6.14, DL-6.15 |
| `cursorai.action.showUsagePricingModal` | DL-6.3 |
| `cursorAuth.skipPrivacyModeGracePeriod` | DL-6.11 |
| `cursorAuth.toggleAuthDebug` | WB-8.21 |
| `cursorBlame.openCommit` | ED-6.40 |
| `cursorBlame.openFileBlame` | SB-2.39, ED-6.39 |
| `cursorBlame.viewCommit` | ED-6.40 |
| `cursorBottom` | ED-2.9 |
| `cursorColumnSelectDown` | ED-2.5 |
| `cursorColumnSelectLeft` | ED-2.5 |
| `cursorColumnSelectPageDown` | ED-2.5 |
| `cursorColumnSelectPageUp` | ED-2.5 |
| `cursorColumnSelectRight` | ED-2.5 |
| `cursorColumnSelectUp` | ED-2.5 |
| `cursorDown` | ED-2.9 |
| `cursorEnd` | ED-2.9 |
| `cursorHome` | ED-2.9 |
| `cursorLeft` | ED-2.9 |
| `cursorLineEnd` | ED-2.9 |
| `cursorLineStart` | ED-2.9 |
| `cursorMove` | ED-2.9 |
| `cursorOrigin.copyCodeLink` | ED-1.28 |
| `cursorOrigin.copyCodeLinkWithoutRange` | ED-1.28 |
| `cursorPageDown` | ED-2.9 |
| `cursorPageUp` | ED-2.9 |
| `cursorRedo` | ED-2.31 |
| `cursorRight` | ED-2.9 |
| `cursorTop` | ED-2.9 |
| `cursorUndo` | ED-2.31 |
| `cursorUp` | ED-2.9 |
| `cursorWordLeft` | ED-2.9 |
| `cursorWordPartLeft` | ED-2.9 |
| `cursorWordPartRight` | ED-2.9 |
| `cursorWordRight` | ED-2.9 |
| `cut` | ED-2.28 |
| `debug.action.openDisassemblyView` | ED-10.45 |
| `debug.action.toggleDisassemblyViewSourceCode` | ED-10.45 |
| `debug.addConfiguration` | SB-5.6, QA-4.16 |
| `debug.addToWatchExpressions` | SB-5.17 |
| `debug.breakWhenValueChanges` | SB-5.14 |
| `debug.breakWhenValueIsAccessed` | SB-5.14 |
| `debug.breakWhenValueIsRead` | SB-5.14 |
| `debug.collapseRepl` | PN-6.9 |
| `debug.copyEvaluatePath` | SB-5.13 |
| `debug.copyStackTrace` | SB-5.22 |
| `debug.copyWatchExpression` | SB-5.13 |
| `debug.editBreakpoint` | SB-5.26, SB-5.29 |
| `debug.editBreakpointMode` | SB-5.26 |
| `debug.editFunctionBreakpoint` | SB-5.26 |
| `debug.editFunctionBreakpointHitCount` | SB-5.26 |
| `debug.enableOrDisableBreakpoint` | ED-11.1 |
| `debug.jumpToCursor` | ED-11.21 |
| `debug.openBreakpointToSide` | SB-5.23 |
| `debug.openView` | SB-5.3 |
| `debug.removeBreakpoint` | SB-5.25 |
| `debug.removeWatchExpression` | SB-5.18 |
| `debug.renameWatchExpression` | SB-5.18 |
| `debug.replCopy` | PN-6.15 |
| `debug.replPaste` | PN-6.15 |
| `debug.setVariable` | SB-5.12 |
| `debug.setWatchExpression` | SB-5.18 |
| `debug.startFromConfig` | WB-6.7 |
| `debug.switchTodevStagingEverythingBackend` | SS-6.13 |
| `debug.switchTofullLocalBackend` | SS-6.13 |
| `debug.switchTofullLocalincludingcppBackend` | SS-6.13 |
| `debug.switchTolocalBackend` | SS-6.13 |
| `debug.switchTolocalExceptEmbeddingsBackend` | SS-6.13 |
| `debug.switchTolocalStagingBackend` | SS-6.13 |
| `debug.switchToprodBackend` | SS-6.13 |
| `debug.switchTostagingBackend` | SS-6.13 |
| `debug.switchTostagingLocalWebsiteBackend` | SS-6.13 |
| `debug.toggleBreakpoint` | SB-5.24 |
| `debug.toggleExceptionBreakpoints` | SB-5.29 |
| `debug.toggleReplIgnoreFocus` | PN-6.1 |
| `decreaseSearchEditorContextLines` | ED-3.34 |
| `deleteAllLeft` | ED-2.12 |
| `deleteAllRight` | ED-2.12 |
| `deleteFile` | SB-2.8, DL-4.2, DL-4.3, DL-4.4, DL-4.5 |
| `deleteInsideWord` | ED-2.12 |
| `deleteLeft` | ED-2.12, ED-2.22 |
| `deleteRight` | ED-2.12 |
| `deleteWordLeft` | ED-2.12 |
| `deleteWordPartLeft` | ED-2.12 |
| `deleteWordPartRight` | ED-2.12 |
| `deleteWordRight` | ED-2.12 |
| `developer.aiCodeTracking.revealPaths` | SS-6.16 |
| `developer.aiCodeTracking.scoreCommit` | SS-6.15 |
| `developer.aiCodeTracking.scoreCommitInternal` | SS-6.15 |
| `developer.aiCodeTracking.scoreCommitWithDetails` | SS-6.15 |
| `developer.aiCodeTracking.scoreMultipleCommits` | SS-6.15 |
| `developer.captureAndUploadProcessSnapshot` | WB-8.20 |
| `developer.captureAndUploadRendererHeapSnapshot` | NT-6.16 |
| `developer.showCurrentFileAsReleaseNotes` | ED-9.26 |
| `diff.clipboard.copyDeletedContent` | ED-6.43 |
| `diff.clipboard.copyDeletedLineContent` | ED-6.43 |
| `diff.inline.revertChange` | ED-6.43 |
| `diffEditor.collapseAllUnchangedRegions` | ED-6.4 |
| `diffEditor.exitCompareMove` | ED-6.5 |
| `diffEditor.revert` | ED-6.7 |
| `diffEditor.showAllUnchangedRegions` | ED-6.4 |
| `diffEditor.switchSide` | ED-6.9 |
| `diffEditor.toggleCollapseUnchangedRegions` | ED-6.4 |
| `diffEditor.toggleShowMovedCodeBlocks` | ED-6.5 |
| `diffEditor.toggleUseInlineViewWhenSpaceIsLimited` | ED-6.2 |
| `diffreview.close` | ED-6.6 |
| `editor.action.*` | QA-1.13 |
| `editor.action.acceptCppSuggestion` | AC-6.12 |
| `editor.action.acceptCursorTabSuggestion` | AC-6.14, AC-6.15, AC-7.1, AC-7.2, AC-7.4, AC-7.5, AC-7.7, AC-7.8, AC-7.10, AC-7.19, AC-7.23, AC-8.17 |
| `editor.action.accessibilityHelp` | ED-3.28, ED-4.39, PN-3.22, PN-4.20, PN-5.19 |
| `editor.action.accessibilityHelpConfigureAssignedKeybindings` | WB-4.1 |
| `editor.action.accessibilityHelpConfigureKeybindings` | WB-4.1 |
| `editor.action.accessibilityHelpOpenHelpLink` | WB-4.2 |
| `editor.action.accessibleDiffViewer.next` | ED-6.6 |
| `editor.action.accessibleDiffViewer.prev` | ED-6.6 |
| `editor.action.accessibleView` | ED-4.39, AC-2.38, NT-3.12 |
| `editor.action.accessibleViewAcceptInlineCompletion` | AC-2.38 |
| `editor.action.accessibleViewDisableHint` | WB-4.3 |
| `editor.action.accessibleViewGoToSymbol` | WB-4.4 |
| `editor.action.accessibleViewNext` | AC-2.38 |
| `editor.action.accessibleViewPrevious` | AC-2.38 |
| `editor.action.addCommentLine` | ED-2.16 |
| `editor.action.addCursorsToBottom` | ED-2.1 |
| `editor.action.addCursorsToTop` | ED-2.1 |
| `editor.action.addSelectionToNextFindMatch` | ED-2.3 |
| `editor.action.addSelectionToPreviousFindMatch` | ED-2.3 |
| `editor.action.addSymbolToChat` | TB-1.32 |
| `editor.action.addSymbolToNewChat` | TB-1.32 |
| `editor.action.announceCursorPosition` | WB-4.9 |
| `editor.action.autoFix` | ED-4.16 |
| `editor.action.blockComment` | ED-2.16 |
| `editor.action.cancelSelectionAnchor` | ED-2.8 |
| `editor.action.changeAll` | ED-2.4 |
| `editor.action.changeTabDisplaySize` | ED-2.18, SS-3.7 |
| `editor.action.clipboardCopyAction` | ED-2.28, ED-10.73 |
| `editor.action.clipboardCopyWithSyntaxHighlightingAction` | ED-2.28 |
| `editor.action.clipboardCutAction` | ED-2.28, ED-10.73 |
| `editor.action.clipboardPasteAction` | ED-2.28, ED-10.73 |
| `editor.action.codeAction` | ED-4.14, ED-4.15 |
| `editor.action.commentLine` | ED-2.16 |
| `editor.action.configureCppDiffPeekView` | AC-6.11 |
| `editor.action.copyLinesDownAction` | ED-2.11 |
| `editor.action.copyLinesUpAction` | ED-2.11 |
| `editor.action.debugEditorGpuRenderer` | ED-5.45 |
| `editor.action.decreaseHoverVerbosityLevel` | ED-4.1 |
| `editor.action.decreaseHoverVerbosityLevelFromAccessibleView` | ED-4.39 |
| `editor.action.defineKeybinding` | ED-8.8, ED-8.55 |
| `editor.action.deleteLines` | ED-2.12 |
| `editor.action.detectIndentation` | ED-2.18, SS-3.9 |
| `editor.action.diffReview.next` | ED-6.6 |
| `editor.action.diffReview.prev` | ED-6.6 |
| `editor.action.dirtydiff.next` | ED-6.25 |
| `editor.action.dirtydiff.previous` | ED-6.25 |
| `editor.action.duplicateSelection` | ED-2.11 |
| `editor.action.enableCppGlobally` | AC-8.3 |
| `editor.action.findReferences` | ED-4.9 |
| `editor.action.fixAll` | ED-4.16 |
| `editor.action.focusNextCursor` | ED-2.1 |
| `editor.action.focusPreviousCursor` | ED-2.1 |
| `editor.action.focusStickyScroll` | ED-5.6 |
| `editor.action.fontZoomIn` | ED-5.38 |
| `editor.action.fontZoomOut` | ED-5.38 |
| `editor.action.fontZoomReset` | ED-5.38 |
| `editor.action.forceRetokenize` | ED-5.47 |
| `editor.action.format` | ED-4.17 |
| `editor.action.formatChanges` | ED-4.18 |
| `editor.action.formatDocument` | ED-4.17 |
| `editor.action.formatDocument.multiple` | ED-4.17 |
| `editor.action.formatDocument.none` | ED-4.17 |
| `editor.action.formatSelection` | ED-4.17 |
| `editor.action.formatSelection.multiple` | ED-4.17 |
| `editor.action.goToBottomHover` | ED-4.2 |
| `editor.action.goToDeclaration` | ED-3.36 |
| `editor.action.goToFocusedStickyScrollLine` | ED-5.6 |
| `editor.action.goToImplementation` | ED-4.7 |
| `editor.action.goToLocations` | ED-4.33 |
| `editor.action.goToMatchFindAction` | ED-3.7 |
| `editor.action.goToReferences` | ED-4.8 |
| `editor.action.goToSelectionAnchor` | ED-2.8 |
| `editor.action.goToTopHover` | ED-4.2 |
| `editor.action.goToTypeDefinition` | ED-4.6 |
| `editor.action.hideColorPicker` | ED-4.30 |
| `editor.action.hideHover` | ED-4.2 |
| `editor.action.hideLongLineWarningHover` | ED-4.1 |
| `editor.action.increaseHoverVerbosityLevel` | ED-4.1 |
| `editor.action.increaseHoverVerbosityLevelFromAccessibleView` | ED-4.39 |
| `editor.action.indentationToSpaces` | ED-2.19, SS-3.8 |
| `editor.action.indentationToTabs` | ED-2.19, SS-3.8 |
| `editor.action.indentLines` | ED-2.17 |
| `editor.action.indentUsingSpaces` | ED-2.18, SS-3.6 |
| `editor.action.indentUsingTabs` | ED-2.18, SS-3.6 |
| `editor.action.inlineDiffs.acceptAll` | ED-6.29, ED-7.47 |
| `editor.action.inlineDiffs.acceptPartialEdit` | ED-6.31 |
| `editor.action.inlineDiffs.cancelEdits` | ED-6.32 |
| `editor.action.inlineDiffs.nextChange` | ED-6.33 |
| `editor.action.inlineDiffs.nextDiffFile` | ED-6.33 |
| `editor.action.inlineDiffs.previousChange` | ED-6.33 |
| `editor.action.inlineDiffs.previousDiffFile` | ED-6.33 |
| `editor.action.inlineDiffs.rejectAll` | ED-6.30 |
| `editor.action.inlineDiffs.rejectAllAcrossAllEditors` | ED-6.30 |
| `editor.action.inlineDiffs.rejectPartialEdit` | ED-6.31 |
| `editor.action.inlineDiffs.viewAllChanges` | ED-6.34 |
| `editor.action.inlineSuggest.acceptNextInlineEditPart` | AC-2.16 |
| `editor.action.inlineSuggest.acceptNextLine` | AC-2.8, AC-2.12, AC-2.42, AC-2.48 |
| `editor.action.inlineSuggest.acceptNextWord` | AC-2.8, AC-2.11, AC-2.42, AC-2.48, AC-6.16, AC-8.11, AC-8.17 |
| `editor.action.inlineSuggest.cancelSnooze` | AC-2.36 |
| `editor.action.inlineSuggest.commit` | AC-2.8, AC-2.10, AC-2.17, AC-2.18, AC-2.27, AC-2.42, AC-2.48 |
| `editor.action.inlineSuggest.commitAlternativeAction` | AC-2.18, AC-2.20 |
| `editor.action.inlineSuggest.dev.extractRepro` | AC-2.40 |
| `editor.action.inlineSuggest.hide` | AC-2.13, AC-2.18, AC-2.45 |
| `editor.action.inlineSuggest.jump` | AC-2.17, AC-2.19, AC-2.29 |
| `editor.action.inlineSuggest.renameSymbol` | AC-2.20 |
| `editor.action.inlineSuggest.showNext` | AC-2.9 |
| `editor.action.inlineSuggest.showPrevious` | AC-2.9 |
| `editor.action.inlineSuggest.snooze` | AC-2.18, AC-2.36 |
| `editor.action.inlineSuggest.toggleAlwaysShowToolbar` | AC-2.8 |
| `editor.action.inlineSuggest.toggleShowCollapsed` | AC-2.18, AC-2.26 |
| `editor.action.inlineSuggest.trigger` | AC-2.14 |
| `editor.action.inlineSuggest.triggerInlineEdit` | AC-2.15 |
| `editor.action.inlineSuggest.triggerInlineEditExplicit` | AC-2.15 |
| `editor.action.inPlaceReplace.down` | ED-2.21 |
| `editor.action.inPlaceReplace.up` | ED-2.21 |
| `editor.action.insertColorWithStandaloneColorPicker` | ED-4.30 |
| `editor.action.insertCursorAbove` | ED-2.1 |
| `editor.action.insertCursorAtEndOfEachLineSelected` | ED-2.1 |
| `editor.action.insertCursorBelow` | ED-2.1 |
| `editor.action.insertFinalNewLine` | ED-2.15 |
| `editor.action.insertLineAfter` | ED-2.13 |
| `editor.action.insertLineBefore` | ED-2.13 |
| `editor.action.insertSnippet` | AC-4.11 |
| `editor.action.inspectTMScopes` | ED-5.39 |
| `editor.action.joinLines` | ED-2.13 |
| `editor.action.jumpToBracket` | ED-2.10 |
| `editor.action.linkedEditing` | ED-2.27 |
| `editor.action.marker.next` | ED-4.26, ED-4.27, ED-7.44 |
| `editor.action.marker.nextInFiles` | ED-4.27 |
| `editor.action.marker.prev` | ED-4.27, ED-7.44 |
| `editor.action.marker.prevInFiles` | ED-4.27 |
| `editor.action.marker.this` | ED-4.40 |
| `editor.action.moveCarretLeftAction` | ED-2.21 |
| `editor.action.moveCarretRightAction` | ED-2.21 |
| `editor.action.moveLinesDownAction` | ED-2.11 |
| `editor.action.moveLinesUpAction` | ED-2.11 |
| `editor.action.moveSelectionToNextFindMatch` | ED-2.3 |
| `editor.action.moveSelectionToPreviousFindMatch` | ED-2.3 |
| `editor.action.nextCommentedRangeAction` | ED-12.6 |
| `editor.action.nextCommentingRange` | ED-12.6 |
| `editor.action.nextCommentThreadAction` | ED-12.6 |
| `editor.action.nextMatchFindAction` | ED-3.6, ED-3.23, ED-6.38 |
| `editor.action.nextSelectionMatchFindAction` | ED-3.8 |
| `editor.action.openDeclarationToTheSide` | ED-3.36 |
| `editor.action.openLink` | ED-4.28 |
| `editor.action.organizeImports` | ED-4.16 |
| `editor.action.outdentLines` | ED-2.17 |
| `editor.action.pageDownHover` | ED-4.2 |
| `editor.action.pageUpHover` | ED-4.2 |
| `editor.action.pasteAs` | ED-2.29 |
| `editor.action.pasteAsText` | ED-2.29 |
| `editor.action.peekDeclaration` | ED-4.9 |
| `editor.action.peekDefinition` | ED-3.36, ED-4.9 |
| `editor.action.peekImplementation` | ED-4.9 |
| `editor.action.peekLocations` | ED-4.10 |
| `editor.action.peekTypeDefinition` | ED-4.9 |
| `editor.action.previousCommentedRangeAction` | ED-12.6 |
| `editor.action.previousCommentingRange` | ED-12.6 |
| `editor.action.previousCommentThreadAction` | ED-12.6 |
| `editor.action.previousMatchFindAction` | ED-3.6, ED-3.23, ED-6.38 |
| `editor.action.previousSelectionMatchFindAction` | ED-3.8 |
| `editor.action.quickFix` | ED-4.14, ED-4.26 |
| `editor.action.refactor` | ED-4.15 |
| `editor.action.referenceSearch.trigger` | ED-4.9 |
| `editor.action.reindentlines` | ED-2.19 |
| `editor.action.reindentselectedlines` | ED-2.19 |
| `editor.action.removeBrackets` | ED-2.10 |
| `editor.action.removeCommentLine` | ED-2.16 |
| `editor.action.removeDuplicateLines` | ED-2.14 |
| `editor.action.rename` | ED-4.12 |
| `editor.action.renderWhitespace.all` | ED-2.36, ED-5.35 |
| `editor.action.renderWhitespace.boundary` | ED-2.36, ED-5.35 |
| `editor.action.renderWhitespace.none` | ED-2.36, ED-5.35 |
| `editor.action.renderWhitespace.selection` | ED-2.36, ED-5.35 |
| `editor.action.renderWhitespace.trailing` | ED-2.36, ED-5.35 |
| `editor.action.replaceAll` | ED-3.16 |
| `editor.action.replaceOne` | ED-3.15 |
| `editor.action.resetSuggestSize` | AC-1.19 |
| `editor.action.revealDeclaration` | ED-4.5 |
| `editor.action.revealDefinition` | ED-4.3 |
| `editor.action.revealDefinitionAside` | ED-4.3 |
| `editor.action.reverseLines` | ED-2.14 |
| `editor.action.scrollDownHover` | ED-4.2 |
| `editor.action.scrollLeftHover` | ED-4.2 |
| `editor.action.scrollRightHover` | ED-4.2 |
| `editor.action.scrollUpHover` | ED-4.2 |
| `editor.action.selectAll` | ED-2.6, ED-10.73 |
| `editor.action.selectAllMatches` | ED-3.19 |
| `editor.action.selectEditor` | ED-5.6 |
| `editor.action.selectFromAnchorToCursor` | ED-2.8 |
| `editor.action.selectHighlights` | ED-2.4 |
| `editor.action.selectionClipboardPaste` | ED-2.28 |
| `editor.action.selectNextStickyScrollLine` | ED-5.6 |
| `editor.action.selectPreviousStickyScrollLine` | ED-5.6 |
| `editor.action.selectToBracket` | ED-2.10 |
| `editor.action.setCppDiffPeekView` | AC-6.10, AC-6.11 |
| `editor.action.setSelectionAnchor` | ED-2.8 |
| `editor.action.showContextMenu` | WB-7.2 |
| `editor.action.showDefinitionPreviewHover` | ED-4.4 |
| `editor.action.showHover` | ED-4.2 |
| `editor.action.showOrFocusStandaloneColorPicker` | ED-4.30 |
| `editor.action.showSnippets` | AC-4.11 |
| `editor.action.smartSelect.expand` | ED-2.7 |
| `editor.action.smartSelect.grow` | ED-2.7 |
| `editor.action.smartSelect.shrink` | ED-2.7 |
| `editor.action.sortLinesAscending` | ED-2.14 |
| `editor.action.sortLinesDescending` | ED-2.14 |
| `editor.action.sourceAction` | ED-4.16 |
| `editor.action.startDebugTextMate` | ED-5.48 |
| `editor.action.startFindReplaceAction` | ED-3.14, ED-7.30 |
| `editor.action.submitComment` | ED-12.4 |
| `editor.action.surroundWithSnippet` | AC-4.13, AC-4.15 |
| `editor.action.toggleColumnSelection` | ED-2.5, SS-3.25 |
| `editor.action.toggleMinimap` | ED-5.1 |
| `editor.action.toggleOvertypeInsertMode` | ED-2.26, SS-3.23 |
| `editor.action.toggleRenderControlCharacter` | ED-5.35 |
| `editor.action.toggleRenderWhitespace` | ED-2.36, ED-5.35 |
| `editor.action.toggleScreenReaderAccessibilityMode` | WB-4.8 |
| `editor.action.toggleStickyScroll` | ED-5.4 |
| `editor.action.toggleTabFocusMode` | PN-3.45, SS-3.24 |
| `editor.action.toggleWordWrap` | ED-2.35, ED-5.36 |
| `editor.action.transformToCamelcase` | ED-2.20 |
| `editor.action.transformToKebabcase` | ED-2.20 |
| `editor.action.transformToLowercase` | ED-2.20 |
| `editor.action.transformToPascalcase` | ED-2.20 |
| `editor.action.transformToSnakecase` | ED-2.20 |
| `editor.action.transformToTitlecase` | ED-2.20 |
| `editor.action.transformToUppercase` | ED-2.20 |
| `editor.action.transpose` | ED-2.21 |
| `editor.action.transposeLetters` | ED-2.21 |
| `editor.action.triggerParameterHints` | AC-3.1, AC-3.12 |
| `editor.action.triggerSuggest` | AC-1.1 |
| `editor.action.trimTrailingWhitespace` | ED-2.15 |
| `editor.action.unicodeHighlight.disableHighlightingInComments` | ED-5.37 |
| `editor.action.unicodeHighlight.disableHighlightingInStrings` | ED-5.37 |
| `editor.action.unicodeHighlight.disableHighlightingOfAmbiguousCharacters` | ED-5.37, ED-5.41 |
| `editor.action.unicodeHighlight.disableHighlightingOfInvisibleCharacters` | ED-5.37, ED-5.41 |
| `editor.action.unicodeHighlight.disableHighlightingOfNonBasicAsciiCharacters` | ED-5.37, ED-5.41 |
| `editor.action.unicodeHighlight.showExcludeOptions` | ED-5.37 |
| `editor.action.updateGhostText` | AC-6.1 |
| `editor.action.webvieweditor.findNext` | ED-10.7 |
| `editor.action.webvieweditor.findPrevious` | ED-10.7 |
| `editor.action.webvieweditor.hideFind` | ED-10.7 |
| `editor.action.webvieweditor.showFind` | ED-10.7 |
| `editor.action.wordHighlight.next` | ED-4.22, ED-5.33 |
| `editor.action.wordHighlight.prev` | ED-4.22, ED-5.33 |
| `editor.action.wordHighlight.trigger` | ED-4.22, ED-5.33 |
| `editor.actions.findWithArgs` | ED-3.2 |
| `editor.cancelOperation` | ED-4.8 |
| `editor.changeDropType` | ED-2.29 |
| `editor.changePasteType` | ED-2.29 |
| `editor.closeCallHierarchy` | ED-4.31 |
| `editor.closeTestPeek` | ED-13.4 |
| `editor.closeTypeHierarchy` | ED-4.32 |
| `editor.contrib.contextmenu` | WB-7.2 |
| `editor.contrib.menuPreventer` | TB-1.5 |
| `editor.cpp.disableenabled` | AC-8.3 |
| `editor.cpp.login` | AC-8.2, AC-8.9, SS-6.6 |
| `editor.cpp.snooze` | AC-8.6, AC-8.17 |
| `editor.cpp.toggle` | AC-8.3 |
| `editor.cpp.unsnooze` | AC-8.7, AC-8.17 |
| `editor.createFoldingRangeFromSelection` | ED-5.15 |
| `editor.debug.action.addLogPoint` | ED-11.6 |
| `editor.debug.action.closeExceptionWidget` | ED-11.16 |
| `editor.debug.action.conditionalBreakpoint` | ED-11.5 |
| `editor.debug.action.copyAddress` | ED-10.89 |
| `editor.debug.action.editBreakpoint` | ED-11.8 |
| `editor.debug.action.goToNextBreakpoint` | ED-11.11 |
| `editor.debug.action.goToPreviousBreakpoint` | ED-11.11 |
| `editor.debug.action.runToCursor` | ED-11.20 |
| `editor.debug.action.selectionToRepl` | ED-11.23 |
| `editor.debug.action.selectionToWatch` | ED-11.24 |
| `editor.debug.action.showDebugHover` | ED-11.15 |
| `editor.debug.action.stepIntoTargets` | ED-11.22 |
| `editor.debug.action.toggleBreakpoint` | ED-10.89, ED-11.1 |
| `editor.debug.action.toggleInlineBreakpoint` | ED-11.10 |
| `editor.debug.action.triggerByBreakpoint` | ED-11.7 |
| `editor.detectLanguage` | SS-3.19, SS-3.20 |
| `editor.emmet.action.balanceIn` | ED-2.33 |
| `editor.emmet.action.balanceOut` | ED-2.33 |
| `editor.emmet.action.decrementNumberByOne` | ED-2.34 |
| `editor.emmet.action.decrementNumberByOneTenth` | ED-2.34 |
| `editor.emmet.action.decrementNumberByTen` | ED-2.34 |
| `editor.emmet.action.evaluateMathExpression` | ED-2.34 |
| `editor.emmet.action.expandAbbreviation` | ED-2.32 |
| `editor.emmet.action.incrementNumberByOne` | ED-2.34 |
| `editor.emmet.action.incrementNumberByOneTenth` | ED-2.34 |
| `editor.emmet.action.incrementNumberByTen` | ED-2.34 |
| `editor.emmet.action.matchTag` | ED-2.33 |
| `editor.emmet.action.mergeLines` | ED-2.33 |
| `editor.emmet.action.nextEditPoint` | ED-2.33 |
| `editor.emmet.action.prevEditPoint` | ED-2.33 |
| `editor.emmet.action.reflectCSSValue` | ED-2.34 |
| `editor.emmet.action.removeTag` | ED-2.33 |
| `editor.emmet.action.selectNextItem` | ED-2.33 |
| `editor.emmet.action.selectPrevItem` | ED-2.33 |
| `editor.emmet.action.splitJoinTag` | ED-2.33 |
| `editor.emmet.action.toggleComment` | ED-2.33 |
| `editor.emmet.action.updateImageSize` | ED-2.34 |
| `editor.emmet.action.updateTag` | ED-2.33 |
| `editor.emmet.action.wrapWithAbbreviation` | ED-2.32 |
| `editor.fold` | ED-5.11 |
| `editor.foldAll` | ED-5.12 |
| `editor.foldAllBlockComments` | ED-5.14 |
| `editor.foldAllExcept` | ED-5.12 |
| `editor.foldAllMarkerRegions` | ED-5.14 |
| `editor.foldLevel1` | ED-5.13 |
| `editor.foldRecursively` | ED-5.11 |
| `editor.gotoNextFold` | ED-5.16 |
| `editor.gotoNextSymbolFromResult` | ED-4.33 |
| `editor.gotoNextSymbolFromResult.cancel` | ED-4.33 |
| `editor.gotoParentFold` | ED-5.16 |
| `editor.gotoPreviousFold` | ED-5.16 |
| `editor.hideDropWidget` | ED-2.29 |
| `editor.hidePasteWidget` | ED-2.29 |
| `editor.inlayHints.Reset` | SS-3.32 |
| `editor.landing_screen_empty.remove_recent_project` | ED-9.21 |
| `editor.pasteAs.default` | ED-2.29 |
| `editor.refocusCallHierarchy` | ED-4.31 |
| `editor.refocusTypeHierarchy` | ED-4.32 |
| `editor.removeManualFoldingRanges` | ED-5.15 |
| `editor.showCallHierarchy` | ED-4.31 |
| `editor.showIncomingCalls` | ED-4.31 |
| `editor.showOutgoingCalls` | ED-4.31 |
| `editor.showSubtypes` | ED-4.32 |
| `editor.showSupertypes` | ED-4.32 |
| `editor.showTypeHierarchy` | ED-4.32 |
| `editor.tabCompletionController` | AC-4.9 |
| `editor.toggleFold` | ED-5.11 |
| `editor.toggleFoldRecursively` | ED-5.11 |
| `editor.toggleImportFold` | ED-5.17 |
| `editor.unfold` | ED-5.11 |
| `editor.unfoldAll` | ED-5.12 |
| `editor.unfoldAllExcept` | ED-5.12 |
| `editor.unfoldAllMarkerRegions` | ED-5.14 |
| `editor.unfoldRecursively` | ED-5.11 |
| `editorScroll` | ED-5.30 |
| `editorType.openModified` | ED-1.40 |
| `editorType.openOriginal` | ED-1.40 |
| `editTelemetry.showDebugDetails` | SS-6.5 |
| `editTelemetry.toggleDebugDecorations` | SS-6.5 |
| `expandLineSelection` | ED-2.6 |
| `expandSectionCodeAction` | ED-4.14 |
| `explorer.createFileFromPaste` | SB-2.47 |
| `explorer.download` | SB-2.13 |
| `explorer.newFile` | SB-2.6 |
| `explorer.newFolder` | SB-2.6 |
| `explorer.openAndPassFocus` | SB-2.3 |
| `explorer.openToSide` | SB-2.4, SB-2.32 |
| `explorer.openWith` | SB-2.5, ED-1.43 |
| `explorer.upload` | SB-2.13 |
| `extension.bisect.next` | WB-6.4 |
| `extension.bisect.start` | WB-6.4 |
| `extension.bisect.stop` | WB-6.4 |
| `extension.open` | ED-8.48 |
| `files.openTimeline` | SB-7.10 |
| `filesExplorer.cancelCut` | SB-2.9 |
| `filesExplorer.copy` | SB-2.9, DL-4.9 |
| `filesExplorer.cut` | SB-2.9, DL-4.9 |
| `filesExplorer.findInFolder` | SB-3.14, SB-3.29 |
| `filesExplorer.findInWorkspace` | SB-3.14, SB-3.29 |
| `filesExplorer.openFileInBrowser` | SB-2.38 |
| `filesExplorer.openFilePreserveFocus` | SB-2.3 |
| `filesExplorer.paste` | SB-2.9, DL-4.9, DL-4.12 |
| `fileWorkingCopy.overwrite` | WB-1.1 |
| `fileWorkingCopy.retry` | WB-1.1 |
| `fileWorkingCopy.revert` | WB-1.1 |
| `fileWorkingCopy.saveAs` | WB-1.1 |
| `fileWorkingCopy.saveElevated` | WB-1.1 |
| `fileWorkingCopy.unlock` | WB-1.1 |
| `firstCompressedFolder` | SB-2.21 |
| `focusAndAcceptSuggestion` | AC-1.37 |
| `focusNextRenameSuggestion` | ED-4.12 |
| `focusPreviousRenameSuggestion` | ED-4.12 |
| `focusSuggestion` | AC-1.37 |
| `git.acceptMerge` | ED-6.22 |
| `git.addRemote` | SB-4.26 |
| `git.blame.toggleEditorDecoration` | ED-6.27 |
| `git.blame.toggleStatusBarItem` | SS-4.16 |
| `git.branch` | SB-4.16, SB-4.36 |
| `git.branchFrom` | SB-4.16 |
| `git.checkout` | SB-4.15, SS-4.11 |
| `git.checkoutDetached` | SB-4.15 |
| `git.cherryPick` | SB-4.36 |
| `git.cherryPickAbort` | SB-4.36 |
| `git.clean` | SB-4.3 |
| `git.cleanAll` | SB-4.3 |
| `git.cleanAllTracked` | SB-4.3 |
| `git.cleanAllUntracked` | SB-4.3 |
| `git.clone` | SB-2.37, SB-4.27, SB-4.45, ED-9.4, ED-9.20 |
| `git.cloneRecursive` | SB-4.27, SB-4.40 |
| `git.close` | SB-4.29 |
| `git.closeAllDiffEditors` | SB-4.43 |
| `git.closeAllUnmodifiedEditors` | SB-4.43 |
| `git.closeOtherRepositories` | SB-4.29 |
| `git.commit` | SB-4.7, SB-4.8, SB-4.37 |
| `git.commitAll` | SB-4.9 |
| `git.commitAllAmend` | SB-4.10 |
| `git.commitAllAmendNoVerify` | SB-4.10 |
| `git.commitAllNoVerify` | SB-4.9 |
| `git.commitAllSigned` | SB-4.9 |
| `git.commitAllSignedNoVerify` | SB-4.9 |
| `git.commitAmend` | SB-4.10 |
| `git.commitAmendNoVerify` | SB-4.10 |
| `git.commitEmpty` | SB-4.9 |
| `git.commitEmptyNoVerify` | SB-4.9 |
| `git.commitMessageAccept` | SB-4.12 |
| `git.commitMessageDiscard` | SB-4.12 |
| `git.commitNoVerify` | SB-4.9 |
| `git.commitSigned` | SB-4.9 |
| `git.commitSignedNoVerify` | SB-4.9 |
| `git.commitStaged` | SB-4.9 |
| `git.commitStagedAmend` | SB-4.10 |
| `git.commitStagedAmendNoVerify` | SB-4.10 |
| `git.commitStagedNoVerify` | SB-4.9 |
| `git.commitStagedSigned` | SB-4.9 |
| `git.commitStagedSignedNoVerify` | SB-4.9 |
| `git.compareWithWorkspace` | SB-4.4 |
| `git.continueInLocalClone` | SB-4.27 |
| `git.copyCommitId` | SB-4.36 |
| `git.copyCommitMessage` | SB-4.36 |
| `git.copyContentToClipboard` | SS-4.15 |
| `git.createTag` | SB-4.19, SB-4.36 |
| `git.createWorktree` | SB-4.32 |
| `git.delete` | SB-4.42 |
| `git.deleteBranch` | SB-4.16 |
| `git.deleteRemoteBranch` | SB-4.16 |
| `git.deleteRemoteTag` | SB-4.19 |
| `git.deleteTag` | SB-4.19 |
| `git.deleteWorktree` | SB-4.32 |
| `git.deleteWorktree2` | SB-4.32 |
| `git.deleteWorktreeFromPalette` | SB-4.32 |
| `git.diff.stageHunk` | ED-6.8 |
| `git.diff.stageSelection` | ED-6.8 |
| `git.fetch` | SB-4.22 |
| `git.fetchAll` | SB-4.22, SB-4.34 |
| `git.fetchPrune` | SB-4.22 |
| `git.graph.checkout` | SB-4.36 |
| `git.graph.checkoutDetached` | SB-4.36 |
| `git.graph.cherryPick` | SB-4.36 |
| `git.graph.compareRef` | SB-4.36 |
| `git.graph.compareWithMergeBase` | SB-4.36 |
| `git.graph.compareWithRemote` | SB-4.36 |
| `git.graph.deleteBranch` | SB-4.36 |
| `git.graph.deleteTag` | SB-4.36 |
| `git.ignore` | SB-4.6 |
| `git.init` | SB-4.40 |
| `git.manageUnsafeRepositories` | SB-4.40, SB-4.48 |
| `git.merge` | SB-4.17 |
| `git.mergeAbort` | SB-4.17 |
| `git.migrateWorktreeChanges` | SB-4.32 |
| `git.openAllChanges` | SB-4.4 |
| `git.openChange` | SB-4.4, ED-6.26 |
| `git.openFile` | SB-4.4, SB-4.21, ED-6.26 |
| `git.openFile2` | SB-4.4 |
| `git.openHEADFile` | SB-4.4 |
| `git.openMergeEditor` | ED-6.17 |
| `git.openRepositoriesInParentFolders` | SB-4.29, SB-4.40, SB-4.46 |
| `git.openRepository` | SB-4.29 |
| `git.openWorktree` | SB-4.32 |
| `git.openWorktreeInNewWindow` | SB-4.32 |
| `git.publish` | SB-4.8, SB-4.26, SB-4.34, SS-4.13 |
| `git.pull` | SB-4.23 |
| `git.pullFrom` | SB-4.23 |
| `git.pullRebase` | SB-4.23 |
| `git.pullRef` | SB-4.34 |
| `git.push` | SB-4.24 |
| `git.pushForce` | SB-4.24 |
| `git.pushRef` | SB-4.34 |
| `git.pushTags` | SB-4.19 |
| `git.pushTo` | SB-4.24 |
| `git.pushToForce` | SB-4.24 |
| `git.pushWithTags` | SB-4.19 |
| `git.pushWithTagsForce` | SB-4.19 |
| `git.rebase` | SB-4.18 |
| `git.rebaseAbort` | SB-4.18 |
| `git.refresh` | SB-4.37 |
| `git.removeRemote` | SB-4.26 |
| `git.rename` | SB-4.41 |
| `git.renameBranch` | SB-4.16 |
| `git.reopenClosedRepositories` | SB-4.29, SB-4.40 |
| `git.repositories.checkout` | SB-4.30 |
| `git.repositories.checkoutDetached` | SB-4.30 |
| `git.repositories.compareRef` | SB-4.30 |
| `git.repositories.copyBranchName` | SB-4.30 |
| `git.repositories.copyCommitHash` | SB-4.30 |
| `git.repositories.copyStashName` | SB-4.30 |
| `git.repositories.copyTagName` | SB-4.30 |
| `git.repositories.createBranch` | SB-4.30 |
| `git.repositories.createFrom` | SB-4.30 |
| `git.repositories.createTag` | SB-4.30 |
| `git.repositories.createWorktree` | SB-4.30 |
| `git.repositories.deleteBranch` | SB-4.30 |
| `git.repositories.deleteTag` | SB-4.30 |
| `git.repositories.deleteWorktree` | SB-4.30 |
| `git.repositories.merge` | SB-4.30 |
| `git.repositories.openWorktree` | SB-4.30 |
| `git.repositories.openWorktreeInNewWindow` | SB-4.30 |
| `git.repositories.rebase` | SB-4.30 |
| `git.repositories.stashApply` | SB-4.30 |
| `git.repositories.stashCopyBranchName` | SB-4.30 |
| `git.repositories.stashDrop` | SB-4.30 |
| `git.repositories.stashPop` | SB-4.30 |
| `git.repositories.stashView` | SB-4.30 |
| `git.repositories.worktreeCopyBranchName` | SB-4.30 |
| `git.repositories.worktreeCopyCommitHash` | SB-4.30 |
| `git.repositories.worktreeCopyPath` | SB-4.30 |
| `git.rescan` | SB-4.29 |
| `git.restoreCommitTemplate` | SB-4.12 |
| `git.revealFileInOS.linux` | SB-4.6 |
| `git.revealFileInOS.mac` | SB-4.6 |
| `git.revealFileInOS.windows` | SB-4.6 |
| `git.revealInExplorer` | SB-4.6 |
| `git.revertChange` | ED-6.25 |
| `git.revertSelectedRanges` | ED-6.8 |
| `git.runGitMerge` | ED-6.20 |
| `git.runGitMergeDiff3` | ED-6.20 |
| `git.showOutput` | SB-4.37 |
| `git.stage` | SB-4.2, SB-4.21, ED-6.26 |
| `git.stageAll` | SB-4.2 |
| `git.stageAllMerge` | SB-4.2, SB-4.21 |
| `git.stageAllTracked` | SB-4.2 |
| `git.stageAllUntracked` | SB-4.2 |
| `git.stageChange` | ED-6.25 |
| `git.stageFile` | SB-4.2, ED-6.15 |
| `git.stageSelectedRanges` | ED-6.8 |
| `git.stash` | SB-4.20 |
| `git.stashApply` | SB-4.20 |
| `git.stashApplyEditor` | ED-6.26 |
| `git.stashApplyLatest` | SB-4.20 |
| `git.stashDrop` | SB-4.20 |
| `git.stashDropAll` | SB-4.20 |
| `git.stashDropEditor` | ED-6.26 |
| `git.stashIncludeUntracked` | SB-4.20 |
| `git.stashPop` | SB-4.20 |
| `git.stashPopEditor` | ED-6.26 |
| `git.stashPopLatest` | SB-4.20 |
| `git.stashStaged` | SB-4.20 |
| `git.stashView` | SB-4.20 |
| `git.sync` | SB-4.8, SB-4.25, SS-4.12 |
| `git.syncRebase` | SB-4.25 |
| `git.timeline.compareWithSelected` | SB-7.18 |
| `git.timeline.copyCommitId` | SB-7.19 |
| `git.timeline.copyCommitMessage` | SB-7.19 |
| `git.timeline.openDiff` | SB-7.17 |
| `git.timeline.selectForCompare` | SB-7.18 |
| `git.timeline.viewCommit` | SB-7.17 |
| `git.undoCommit` | SB-4.11 |
| `git.unstage` | SB-4.2, ED-6.26 |
| `git.unstageAll` | SB-4.2 |
| `git.unstageChange` | ED-6.25 |
| `git.unstageFile` | SB-4.2, ED-6.15 |
| `git.unstageSelectedRanges` | ED-6.8 |
| `git.viewChanges` | SB-4.4 |
| `git.viewCommit` | ED-6.27, SS-4.14, SS-4.15 |
| `git.viewStagedChanges` | SB-4.4 |
| `git.viewUntrackedChanges` | SB-4.4 |
| `github.graph.openOnGitHub` | SB-4.36 |
| `github.publish` | SB-4.26 |
| `goToNextReference` | ED-4.10 |
| `goToPreviousReference` | ED-4.10 |
| `hideCodeActionWidget` | ED-4.14 |
| `hideCustomTitleBar` | TB-3.10 |
| `hideCustomTitleBarInFullScreen` | TB-3.11 |
| `hideMoreActions` | ED-4.36 |
| `hideSuggestWidget` | AC-1.38 |
| `history.showNext` | SB-3.5, ED-3.21, PN-6.4 |
| `history.showPrevious` | SB-3.5, ED-3.21, PN-6.4 |
| `iconSelectBox.focusDown` | ED-8.14 |
| `iconSelectBox.focusNext` | ED-8.14 |
| `iconSelectBox.focusPrevious` | ED-8.14 |
| `iconSelectBox.focusUp` | ED-8.14 |
| `iconSelectBox.selectFocused` | ED-8.14 |
| `imagePreview.copyImage` | ED-10.2 |
| `imagePreview.reopenAsPreview` | ED-10.3 |
| `imagePreview.reopenAsText` | ED-10.3 |
| `imagePreview.saveImageAs` | ED-10.2 |
| `imagePreview.zoomIn` | ED-10.1 |
| `imagePreview.zoomOut` | ED-10.1 |
| `increaseSearchEditorContextLines` | ED-3.34 |
| `inlayHints.startReadingLineWithHint` | ED-4.21 |
| `inlayHints.stopReadingLineWithHint` | ED-4.21 |
| `inlineChat.start` | ED-9.24 |
| `inlineDiff.tooManyChangesWarning` | ED-6.28 |
| `insertBestCompletion` | AC-1.39 |
| `insertNextSuggestion` | AC-1.39 |
| `insertPrevSuggestion` | AC-1.39 |
| `insertSnippet` | AC-4.9 |
| `interactive.execute` | ED-7.37 |
| `interactive.history.focus` | ED-7.37 |
| `interactive.history.next` | ED-7.37 |
| `interactive.history.previous` | ED-7.37 |
| `interactive.input.clear` | ED-7.37 |
| `interactive.input.focus` | ED-7.37 |
| `interactive.scrollToBottom` | ED-7.38 |
| `interactive.scrollToTop` | ED-7.38 |
| `ipynb.cleanInvalidImageAttachment` | ED-7.25 |
| `ipynb.newUntitledIpynb` | ED-7.1 |
| `ipynb.openIpynbInNotebookEditor` | ED-7.1 |
| `its` | ED-8.35 |
| `join.browserViews` | DL-2.19 |
| `join.closeStorage` | DL-2.19 |
| `join.disconnectRemote` | DL-2.19 |
| `join.extensionDevelopment` | DL-2.19 |
| `join.fileWorkingCopyManager` | DL-2.19 |
| `join.stopExtensionHosts` | DL-2.19 |
| `join.textFiles` | DL-2.19 |
| `join.windowAnalytics` | DL-2.19 |
| `join.workingCopyBackups` | DL-2.19 |
| `join.workingCopyHistory` | DL-2.19 |
| `js.projectStatus.command` | SS-3.47 |
| `jumpToNextSnippetPlaceholder` | AC-4.1, AC-4.4 |
| `jumpToPrevSnippetPlaceholder` | AC-4.1, AC-4.4 |
| `keybindings.editor.acceptWhenExpression` | ED-8.12 |
| `keybindings.editor.addKeybinding` | ED-8.11 |
| `keybindings.editor.clearSearchHistory` | ED-8.9 |
| `keybindings.editor.clearSearchResults` | ED-8.9 |
| `keybindings.editor.copyCommandKeybindingEntry` | ED-8.11 |
| `keybindings.editor.copyCommandTitle` | ED-8.11 |
| `keybindings.editor.copyKeybindingEntry` | ED-8.11 |
| `keybindings.editor.defineKeybinding` | ED-8.11 |
| `keybindings.editor.defineWhenExpression` | ED-8.12 |
| `keybindings.editor.focusKeybindings` | ED-8.9 |
| `keybindings.editor.recordSearchKeys` | ED-8.10 |
| `keybindings.editor.rejectWhenExpression` | ED-8.12 |
| `keybindings.editor.removeKeybinding` | ED-8.11 |
| `keybindings.editor.resetKeybinding` | ED-8.11 |
| `keybindings.editor.searchKeybindings` | ED-8.9 |
| `keybindings.editor.showConflicts` | ED-8.13 |
| `keybindings.editor.showDefaultKeybindings` | ED-8.9 |
| `keybindings.editor.showExtensionKeybindings` | ED-8.9 |
| `keybindings.editor.showUserKeybindings` | ED-8.9 |
| `keybindings.editor.toggleSortByPrecedence` | ED-8.9 |
| `langugage.update.donotask` | WB-5.1 |
| `lastCompressedFolder` | SB-2.21 |
| `lastCursorLineSelect` | ED-2.2 |
| `lastCursorWordSelect` | ED-2.2 |
| `layoutEditorGroups` | ED-1.16 |
| `leaveEditorMessage` | ED-4.34 |
| `leaveSnippet` | AC-4.7 |
| `lineBreakInsert` | ED-2.13 |
| `list.clear` | SB-2.41 |
| `list.closeFind` | SB-1.23, SB-2.26 |
| `list.collapse` | SB-1.23, SB-2.2 |
| `list.collapseAll` | SB-1.23, SB-2.2 |
| `list.collapseAllToFocus` | SB-1.23 |
| `list.expand` | SB-1.23, SB-2.2 |
| `list.expandSelectionDown` | SB-2.41, ED-7.9 |
| `list.expandSelectionUp` | SB-2.41, ED-7.9 |
| `list.find` | SB-1.23, SB-2.26 |
| `list.find.replInputFocus` | PN-6.11 |
| `list.focusAnyDown` | SB-1.23 |
| `list.focusAnyFirst` | SB-1.23 |
| `list.focusAnyLast` | SB-1.23 |
| `list.focusAnyUp` | SB-1.23 |
| `list.focusDown` | SB-1.23, ED-7.40, NT-3.10 |
| `list.focusFirst` | SB-1.23 |
| `list.focusLast` | SB-1.23 |
| `list.focusPageDown` | SB-1.23 |
| `list.focusPageUp` | SB-1.23 |
| `list.focusParent` | SB-1.23 |
| `list.focusUp` | SB-1.23, ED-7.40, NT-3.10 |
| `list.resizeColumn` | WB-3.4 |
| `list.scrollDown` | SB-2.42 |
| `list.scrollLeft` | SB-2.42 |
| `list.scrollRight` | SB-2.42 |
| `list.scrollUp` | SB-2.42 |
| `list.select` | SB-1.23, SB-2.3 |
| `list.selectAll` | SB-2.41 |
| `list.selectAndPreserveFocus` | SB-1.23 |
| `list.showHover` | SB-2.43 |
| `list.stickyScroll.collapse` | SB-1.24 |
| `list.stickyScrollselect` | SB-1.24 |
| `list.stickyScrolltoggleExpand` | SB-1.24 |
| `list.toggleExpand` | SB-1.23, SB-2.2 |
| `list.toggleFindMatchType` | SB-1.23, SB-2.26 |
| `list.toggleFindMode` | SB-1.23, SB-2.26 |
| `list.toggleSelection` | SB-2.41 |
| `list.triggerTypeNavigation` | SB-1.23, SB-2.27 |
| `loadedScripts.collapse` | SB-5.31 |
| `mainThreadWebview-markdown.preview` | ED-1.23 |
| `markdown.editor.redo` | ED-10.21 |
| `markdown.editor.rename` | ED-10.21 |
| `markdown.editor.smartEnter` | ED-10.21 |
| `markdown.editor.toggleLocked` | ED-10.21 |
| `markdown.editor.triggerSuggest` | ED-10.21 |
| `markdown.editor.undo` | ED-10.21 |
| `markdown.preview.refresh` | ED-10.11 |
| `markdown.preview.toggleLock` | ED-10.10 |
| `markdown.reopenAsPreview` | ED-10.18 |
| `markdown.reopenAsSource` | ED-10.18 |
| `markdown.showLockedPreviewToSide` | ED-10.10 |
| `markdown.showPreview` | ED-10.8 |
| `markdown.showPreviewSecuritySelector` | ED-10.15 |
| `markdown.showPreviewToSide` | ED-10.9 |
| `markdown.showSource` | ED-10.12 |
| `markdown.togglePreview` | ED-10.18 |
| `markdownEditor.editMarkdown` | ED-10.23 |
| `markdownEditor.toggleMode` | ED-10.23, ED-10.24, ED-10.49 |
| `memoryMonitor.triggerManualSnapshot` | NT-6.16 |
| `menu.resetHiddenStates` | WB-7.1 |
| `merge-conflict.accept.all-both` | ED-6.23 |
| `merge-conflict.accept.all-current` | SB-4.21, ED-6.23 |
| `merge-conflict.accept.all-incoming` | SB-4.21, ED-6.23 |
| `merge-conflict.accept.both` | ED-6.23 |
| `merge-conflict.accept.current` | ED-6.23 |
| `merge-conflict.accept.incoming` | ED-6.23 |
| `merge-conflict.accept.selection` | ED-6.23 |
| `merge-conflict.compare` | ED-6.23 |
| `merge-conflict.next` | ED-6.23 |
| `merge-conflict.previous` | ED-6.23 |
| `merge.acceptAllInput1` | ED-6.20 |
| `merge.acceptAllInput2` | ED-6.20 |
| `merge.columnLayout` | ED-6.18 |
| `merge.dev.copyContentsJson` | WB-8.22 |
| `merge.dev.loadContentsFromFolder` | WB-8.22 |
| `merge.dev.openContentsJson` | WB-8.22 |
| `merge.dev.openSelectionInTemporaryMergeEditor` | WB-8.22 |
| `merge.dev.saveContentsToFolder` | WB-8.22 |
| `merge.goToNextUnhandledConflict` | ED-6.21 |
| `merge.goToPreviousUnhandledConflict` | ED-6.21 |
| `merge.mixedLayout` | ED-6.18 |
| `merge.openBaseEditor` | ED-6.18 |
| `merge.openResult` | ED-6.18 |
| `merge.showBase` | ED-6.18 |
| `merge.showBaseCenter` | ED-6.18 |
| `merge.showBaseTop` | ED-6.18 |
| `merge.showNonConflictingChanges` | ED-6.18 |
| `merge.toggleActiveConflictInput1` | ED-6.19 |
| `merge.toggleActiveConflictInput2` | ED-6.19 |
| `mergeEditor.acceptAllCombination` | ED-6.20 |
| `mergeEditor.acceptMerge` | ED-6.22 |
| `mergeEditor.compareInput1WithBase` | ED-6.18 |
| `mergeEditor.compareInput2WithBase` | ED-6.18 |
| `mergeEditor.resetCloseWithConflictsChoice` | ED-6.22 |
| `mergeEditor.resetResultToBaseAndAutoMerge` | ED-6.20 |
| `mergeEditor.toggleBetweenInputs` | ED-6.21 |
| `moveActiveEditor` | ED-1.12 |
| `moveFileToTrash` | SB-2.8, DL-4.1, DL-4.3, DL-4.4, DL-4.5 |
| `multiDiffEditor.collapseAll` | ED-6.13 |
| `multiDiffEditor.expandAll` | ED-6.13 |
| `multiDiffEditor.focusFileList` | ED-6.16 |
| `multiDiffEditor.goToFile` | ED-6.14 |
| `multiDiffEditor.goToNextChange` | ED-6.14 |
| `multiDiffEditor.goToPreviousChange` | ED-6.14 |
| `multiDiffEditor.positionFileListLeft` | ED-6.16 |
| `multiDiffEditor.positionFileListRight` | ED-6.16 |
| `multiDiffEditor.toggleFileList` | ED-6.16 |
| `multiDiffEditor.viewFileListAsList` | ED-6.16 |
| `multiDiffEditor.viewFileListAsTree` | ED-6.16 |
| `nextCompressedFolder` | SB-2.21 |
| `notebook.action.changeTabDisplaySize` | SS-3.35 |
| `notebook.action.convertIndentationToSpaces` | SS-3.35 |
| `notebook.action.convertIndentationToTabs` | SS-3.35 |
| `notebook.action.indentUsingSpaces` | SS-3.35 |
| `notebook.action.indentUsingTabs` | SS-3.35 |
| `notebook.action.toggleNotebookStickyScroll` | ED-7.29 |
| `notebook.addFindMatchToSelection` | ED-7.32 |
| `notebook.cancelExecution` | ED-7.14 |
| `notebook.cell.cancelExecution` | ED-7.13 |
| `notebook.cell.changeLanguage` | ED-7.7, ED-7.16 |
| `notebook.cell.changeToCode` | ED-7.7 |
| `notebook.cell.changeToMarkdown` | ED-7.7 |
| `notebook.cell.clearOutputs` | ED-7.23 |
| `notebook.cell.collapseAllCellInputs` | ED-7.11 |
| `notebook.cell.collapseAllCellOutputs` | ED-7.11 |
| `notebook.cell.collapseCellInput` | ED-7.11 |
| `notebook.cell.collapseCellOutput` | ED-7.11 |
| `notebook.cell.copy` | ED-7.5, ED-7.40 |
| `notebook.cell.copyDown` | ED-7.5 |
| `notebook.cell.copyUp` | ED-7.5 |
| `notebook.cell.cursorPageDown` | ED-7.9 |
| `notebook.cell.cursorPageDownSelect` | ED-7.9 |
| `notebook.cell.cursorPageUp` | ED-7.9 |
| `notebook.cell.cursorPageUpSelect` | ED-7.9 |
| `notebook.cell.cut` | ED-7.5, ED-7.40 |
| `notebook.cell.delete` | ED-7.3, ED-7.40 |
| `notebook.cell.detectLanguage` | ED-7.7, ED-7.16 |
| `notebook.cell.edit` | ED-7.8 |
| `notebook.cell.execute` | ED-7.13 |
| `notebook.cell.executeAndFocusContainer` | ED-7.13, ED-7.40 |
| `notebook.cell.executeAndInsertBelow` | ED-7.13 |
| `notebook.cell.executeAndSelectBelow` | ED-7.13, ED-7.40 |
| `notebook.cell.executeCellAndBelow` | ED-7.14 |
| `notebook.cell.executeCellsAbove` | ED-7.14 |
| `notebook.cell.expandAllCellInputs` | ED-7.11 |
| `notebook.cell.expandAllCellOutputs` | ED-7.11 |
| `notebook.cell.expandCellInput` | ED-7.11 |
| `notebook.cell.expandCellOutput` | ED-7.11 |
| `notebook.cell.focusInOutput` | ED-7.9 |
| `notebook.cell.focusOutOutput` | ED-7.9 |
| `notebook.cell.insertCodeCellAbove` | ED-7.2 |
| `notebook.cell.insertCodeCellAboveAndFocusContainer` | ED-7.2, ED-7.40 |
| `notebook.cell.insertCodeCellAtTop` | ED-7.2 |
| `notebook.cell.insertCodeCellBelow` | ED-7.2, ED-7.26 |
| `notebook.cell.insertCodeCellBelowAndFocusContainer` | ED-7.2, ED-7.40 |
| `notebook.cell.insertMarkdownCellAbove` | ED-7.2 |
| `notebook.cell.insertMarkdownCellAtTop` | ED-7.2 |
| `notebook.cell.insertMarkdownCellBelow` | ED-7.2, ED-7.26 |
| `notebook.cell.joinAbove` | ED-7.6 |
| `notebook.cell.joinBelow` | ED-7.6 |
| `notebook.cell.joinSelected` | ED-7.6 |
| `notebook.cell.moveDown` | ED-7.4 |
| `notebook.cell.moveUp` | ED-7.4 |
| `notebook.cell.openFailureActions` | ED-7.17 |
| `notebook.cell.output.selectAll` | ED-7.48 |
| `notebook.cell.paste` | ED-7.5, ED-7.40 |
| `notebook.cell.pasteAbove` | ED-7.5 |
| `notebook.cell.quitEdit` | ED-7.8 |
| `notebook.cell.split` | ED-7.6, ED-7.40 |
| `notebook.cell.toggleLineNumbers` | ED-7.28, ED-7.40 |
| `notebook.cell.toggleOutputs` | ED-7.11, ED-7.40 |
| `notebook.cell.toggleOutputScrolling` | ED-7.22 |
| `notebook.cellOuput.showEmptyOutputs` | ED-7.21 |
| `notebook.cellOutput.copy` | ED-7.21 |
| `notebook.cellOutput.openInOutputPreview` | ED-7.21 |
| `notebook.cellOutput.openInTextEditor` | ED-7.22 |
| `notebook.cellOutput.saveImage` | ED-7.21 |
| `notebook.centerActiveCell` | ED-7.9, SS-3.34 |
| `notebook.clearAllCellsOutputs` | ED-7.23, ED-7.26 |
| `notebook.clearAllInlineValues` | ED-7.18 |
| `notebook.clearNotebookKernelsMRUCache` | ED-7.19 |
| `notebook.commentSelectedCells` | ED-7.10 |
| `notebook.diff.action.next` | ED-7.36 |
| `notebook.diff.action.previous` | ED-7.36 |
| `notebook.diff.cell.revertInput` | ED-7.36 |
| `notebook.diff.cell.revertMetadata` | ED-7.36 |
| `notebook.diff.cell.revertOutputs` | ED-7.36 |
| `notebook.diff.cell.switchOutputRenderingStyleToText` | ED-7.35 |
| `notebook.diff.cell.toggleCollapseUnchangedRegions` | ED-7.35 |
| `notebook.diff.inline.toggle` | ED-7.35 |
| `notebook.diff.openFile` | ED-7.34 |
| `notebook.diff.revertMetadata` | ED-7.36 |
| `notebook.diff.showMetadata` | ED-7.35 |
| `notebook.diff.showOutputs` | ED-7.35 |
| `notebook.diff.switchToText` | ED-7.34 |
| `notebook.diffEditor.2.cell.revertInput` | ED-7.36 |
| `notebook.diffEditor.2.cell.revertMetadata` | ED-7.36 |
| `notebook.diffEditor.2.cell.revertOutputs` | ED-7.36 |
| `notebook.diffEditor.2.goToCell` | ED-7.34 |
| `notebook.diffEditor.hideUnchangedCells` | ED-7.35 |
| `notebook.diffEditor.showUnchangedCells` | ED-7.35 |
| `notebook.execute` | ED-7.14, ED-7.26 |
| `notebook.find` | ED-7.30, ED-7.40 |
| `notebook.findNext.fromWidget` | ED-7.30 |
| `notebook.findPrevious.fromWidget` | ED-7.30 |
| `notebook.focusBottom` | ED-7.9 |
| `notebook.focusNextEditor` | ED-7.9 |
| `notebook.focusPreviousEditor` | ED-7.9 |
| `notebook.focusTop` | ED-7.9 |
| `notebook.fold` | ED-7.12 |
| `notebook.format` | ED-7.33 |
| `notebook.formatCell` | ED-7.33 |
| `notebook.hideFind` | ED-7.30 |
| `notebook.interruptExecution` | ED-7.14, ED-7.38 |
| `notebook.openVariablesView` | SB-5.16, ED-7.18, ED-7.38 |
| `notebook.outline.toggleCodeCells` | SB-7.6, ED-7.31 |
| `notebook.outline.toggleCodeCellSymbols` | SB-7.6, ED-7.31 |
| `notebook.outline.toggleShowMarkdownHeadersOnly` | SB-7.6, ED-7.31 |
| `notebook.output.pickMimetype` | ED-7.21 |
| `notebook.quitEditAllCells` | ED-7.8 |
| `notebook.renderAllMarkdownCells` | ED-7.8 |
| `notebook.revealLastFailedCell` | ED-7.15 |
| `notebook.revealRunningCell` | ED-7.15, ED-7.38 |
| `notebook.saveMimeTypeOrder` | ED-7.21 |
| `notebook.section.expandSection` | SB-7.6, ED-7.12 |
| `notebook.section.foldSection` | SB-7.6, ED-7.12 |
| `notebook.section.runCells` | SB-7.6, ED-7.12 |
| `notebook.section.runSingleCell` | SB-7.6, ED-7.12 |
| `notebook.selectAllFindMatches` | ED-7.32 |
| `notebook.selectIndentation` | SS-3.35 |
| `notebook.setProfile` | ED-7.28 |
| `notebook.toggle.diff.cell.ignoreTrimWhitespace` | ED-7.35 |
| `notebook.toggleCellToolbarPosition` | ED-7.27 |
| `notebook.toggleCellToolbarPositionFromEditorTitle` | ED-7.28 |
| `notebook.toggleLineNumbers` | ED-7.28, ED-7.40 |
| `notebook.toggleLineNumbersFromEditorTitle` | ED-7.28 |
| `notebook.unfold` | ED-7.12 |
| `noteMultiCursor.deleteLeft` | ED-7.32 |
| `noteMultiCursor.deleteRight` | ED-7.32 |
| `noteMultiCursor.exit` | ED-7.32 |
| `notification.acceptPrimaryAction` | NT-2.17 |
| `notification.clear` | NT-2.14, NT-5.4 |
| `notification.collapse` | NT-2.7 |
| `notification.expand` | NT-2.7 |
| `notification.toggle` | NT-2.7 |
| `notifications.clearAll` | NT-3.9, NT-4.8, NT-5.4 |
| `notifications.focusFirstToast` | NT-1.10 |
| `notifications.focusLastToast` | NT-1.10 |
| `notifications.focusNextToast` | NT-1.10 |
| `notifications.focusPreviousToast` | NT-1.10 |
| `notifications.focusToasts` | NT-1.9 |
| `notifications.hideList` | SS-5.2, NT-3.1, NT-3.7, NT-4.5 |
| `notifications.hideToasts` | NT-1.8, NT-5.7 |
| `notifications.showList` | SS-5.2, NT-1.4, NT-3.1, NT-3.6, NT-4.5, NT-4.8, NT-4.9, NT-5.6, NT-5.7 |
| `notifications.toggleDoNotDisturbMode` | NT-4.1, NT-4.2, NT-4.5, NT-4.6 |
| `notifications.toggleDoNotDisturbModeBySource` | NT-4.2, NT-4.3, NT-4.5 |
| `notifications.toggleList` | TB-5.8, NT-1.4 |
| `openEditors.closeAll` | SB-2.31 |
| `openEditors.newUntitledFile` | SB-2.31, WB-1.5 |
| `openInIntegratedTerminal` | SB-2.18 |
| `openInTerminal` | SB-2.18 |
| `openRecentFile` | TB-1.14 |
| `openRecentFolder` | TB-1.14 |
| `openRecentWorkspace` | TB-1.14 |
| `openReference` | ED-4.10 |
| `openReferenceToSide` | ED-4.10 |
| `outdent` | ED-2.17 |
| `outline.collapse` | SB-7.1 |
| `outline.expand` | SB-7.1 |
| `outline.filterOnType` | SB-7.3 |
| `outline.focus` | SB-7.1 |
| `outline.followCursor` | SB-7.2 |
| `outline.sortByKind` | SB-7.4 |
| `outline.sortByName` | SB-7.4 |
| `outline.sortByPosition` | SB-7.4 |
| `output.editor.contrib.filterController` | PN-5.5 |
| `paste` | ED-2.28 |
| `peekview.close` | AC-6.13 |
| `previewSelectedCodeAction` | ED-4.14 |
| `previousCompressedFolder` | SB-2.21 |
| `problems.action.clearFilterText` | PN-4.5 |
| `problems.action.copy` | PN-4.14, PN-4.15 |
| `problems.action.copyMessage` | PN-4.14, PN-4.15 |
| `problems.action.copyRelatedInformationMessage` | PN-4.14, PN-4.15, PN-4.17 |
| `problems.action.focusFilter` | PN-4.5 |
| `problems.action.focusProblemsFromFilter` | PN-4.5 |
| `problems.action.open` | PN-4.10 |
| `problems.action.openToSide` | PN-4.11 |
| `problems.action.showMultilineMessage` | PN-4.16 |
| `problems.action.showQuickFixes` | PN-4.13 |
| `problems.action.showSinglelineMessage` | PN-4.16 |
| `problems.action.toggleMultiline` | PN-4.16 |
| `progress.button.<label>` | NT-5.3 |
| `progress.cancel` | NT-5.3 |
| `quickInput.accept` | QA-6.22 |
| `quickInput.acceptInBackground` | QA-2.8, QA-3.9, QA-6.16 |
| `quickInput.first` | QA-6.21 |
| `quickInput.hide` | QA-6.22 |
| `quickInput.last` | QA-6.21 |
| `quickInput.next` | QA-6.21 |
| `quickInput.nextSeparator` | QA-6.21 |
| `quickInput.nextSeparatorWithQuickAccessFallback` | QA-6.21 |
| `quickInput.pageNext` | QA-6.21 |
| `quickInput.pagePrevious` | QA-6.21 |
| `quickInput.previous` | QA-6.21 |
| `quickInput.previousSeparator` | QA-6.21 |
| `quickInput.previousSeparatorWithQuickAccessFallback` | QA-6.21 |
| `quickInput.toggleCheckbox` | QA-6.2 |
| `quickInput.toggleHover` | QA-6.14 |
| `redo` | SB-2.10, ED-2.31, ED-2.43, ED-4.41, ED-7.41, ED-10.73, DL-4.13 |
| `refactorPreview.apply` | PN-7.24, DL-1.29 |
| `refactorPreview.discard` | PN-7.25, DL-1.29 |
| `refactorPreview.focus` | PN-7.22 |
| `refactorPreview.groupByFile` | PN-7.26 |
| `refactorPreview.groupByType` | PN-7.26 |
| `refactorPreview.toggleCheckedState` | PN-7.23 |
| `refactorPreview.toggleGrouping` | PN-7.26 |
| `references-view.clear` | SB-7.25 |
| `references-view.clearHistory` | SB-7.26 |
| `references-view.copy` | SB-7.24 |
| `references-view.copyAll` | SB-7.24 |
| `references-view.copyPath` | SB-7.24 |
| `references-view.findImplementations` | SB-7.21 |
| `references-view.findReferences` | SB-7.20 |
| `references-view.next` | SB-7.23 |
| `references-view.pickFromHistory` | SB-7.26 |
| `references-view.prev` | SB-7.23 |
| `references-view.refind` | SB-7.26 |
| `references-view.refresh` | SB-7.25 |
| `references-view.removeCallItem` | SB-7.27 |
| `references-view.removeReferenceItem` | SB-7.24 |
| `references-view.removeTypeItem` | SB-7.28 |
| `references-view.showCallHierarchy` | SB-7.27 |
| `references-view.showIncomingCalls` | SB-7.27 |
| `references-view.showOutgoingCalls` | SB-7.27 |
| `references-view.showSubtypes` | SB-7.28 |
| `references-view.showSupertypes` | SB-7.28 |
| `references-view.showTypeHierarchy` | SB-7.28 |
| `registerWindowTitleVariable` | TB-2.12 |
| `remote.tunnel.changeLocalPort` | PN-7.16 |
| `remote.tunnel.closeCommandPalette` | PN-7.12 |
| `remote.tunnel.closeInline` | PN-7.12 |
| `remote.tunnel.copyAddressCommandPalette` | PN-7.14 |
| `remote.tunnel.copyAddressInline` | PN-7.14 |
| `remote.tunnel.forwardCommandPalette` | PN-7.11 |
| `remote.tunnel.forwardInline` | PN-7.11 |
| `remote.tunnel.label` | PN-7.15 |
| `remote.tunnel.open` | PN-7.13 |
| `remote.tunnel.openCommandPalette` | PN-7.13 |
| `remote.tunnel.openPreview` | PN-7.13 |
| `remote.tunnel.privacy<id>` | PN-7.17 |
| `remote.tunnel.setProtocolHttp` | PN-7.18 |
| `remote.tunnel.setProtocolHttps` | PN-7.18 |
| `remoteHub.openRepository` | ED-9.4 |
| `removeRootFolder` | SB-2.19 |
| `removeSecondaryCursors` | ED-2.1 |
| `renameFile` | SB-2.7 |
| `reopenActiveEditorWith` | ED-1.27 |
| `repl.action.acceptInput` | PN-6.3 |
| `repl.action.copyAll` | PN-6.15 |
| `repl.action.filter` | PN-6.10 |
| `repl.action.find` | PN-6.11 |
| `repl.execute` | ED-7.39 |
| `repl.focusLastItemExecuted` | ED-7.39 |
| `repl.input.focus` | ED-7.39 |
| `replacePreviousChar` | ED-2.42 |
| `rerunSearchEditorSearch` | ED-3.35 |
| `resetGettingStartedProgress` | ED-9.11 |
| `revealFileInOS` | SB-2.15, SB-2.32, ED-1.26 |
| `revealInExplorer` | SB-2.14, ED-1.26 |
| `revealLine` | ED-5.30 |
| `revealReference` | ED-4.10 |
| `scm.acceptInput` | SB-4.7 |
| `scm.clearInput` | SB-4.7, AC-2.45 |
| `scm.clearValidation` | SB-4.7 |
| `scm.forceViewNextCommit` | SB-4.13 |
| `scm.forceViewPreviousCommit` | SB-4.13 |
| `scm.input.cancelAction` | SB-4.14 |
| `scm.openInIntegratedTerminal` | SB-4.29 |
| `scm.openInTerminal` | SB-4.29 |
| `scm.setActiveProvider` | SB-4.28, SS-4.8 |
| `scm.viewNextCommit` | SB-4.13 |
| `scm.viewPreviousCommit` | SB-4.13 |
| `scrollEditorBottom` | ED-5.30 |
| `scrollEditorTop` | ED-5.30 |
| `scrollLeft` | ED-5.30 |
| `scrollLineDown` | ED-5.30 |
| `scrollLineUp` | ED-5.30 |
| `scrollPageDown` | ED-5.30 |
| `scrollPageUp` | ED-5.30 |
| `scrollRight` | ED-5.30 |
| `search.action.cancel` | SB-3.27 |
| `search.action.clearHistory` | SB-3.5 |
| `search.action.clearSearchResults` | SB-3.27 |
| `search.action.collapseSearchResults` | SB-3.19 |
| `search.action.copyAll` | SB-3.24 |
| `search.action.copyMatch` | SB-3.24 |
| `search.action.copyPath` | SB-3.24 |
| `search.action.excludeFileTypeFromSearch` | SB-3.16 |
| `search.action.excludeFromSearch` | SB-3.15 |
| `search.action.expandRecursively` | SB-3.19 |
| `search.action.expandSearchResults` | SB-3.19 |
| `search.action.focusFilesToExclude` | ED-3.32 |
| `search.action.focusFilesToInclude` | ED-3.32 |
| `search.action.focusNextSearchResult` | SB-3.26, ED-3.46 |
| `search.action.focusPreviousSearchResult` | SB-3.26, ED-3.46 |
| `search.action.focusQueryEditorWidget` | ED-3.31 |
| `search.action.focusSearchFromResults` | SB-3.26 |
| `search.action.focusSearchList` | SB-3.26 |
| `search.action.includeFileTypeInSearch` | SB-3.16 |
| `search.action.openEditor` | ED-3.29 |
| `search.action.openInEditor` | SB-3.28 |
| `search.action.openNewEditor` | ED-3.29 |
| `search.action.openNewEditorFromView` | SB-3.28 |
| `search.action.openNewEditorToSide` | ED-3.29 |
| `search.action.openResult` | SB-3.21 |
| `search.action.openResultToSide` | SB-3.21 |
| `search.action.refreshSearchResults` | SB-3.27 |
| `search.action.remove` | SB-3.23 |
| `search.action.replace` | SB-3.7 |
| `search.action.replaceAll` | SB-3.8 |
| `search.action.replaceAllInFile` | SB-3.7 |
| `search.action.replaceAllInFolder` | SB-3.7 |
| `search.action.restrictSearchToFolder` | SB-3.15 |
| `search.action.revealInSideBar` | SB-3.25 |
| `search.action.viewAsList` | SB-3.18 |
| `search.action.viewAsTree` | SB-3.18 |
| `search.focus.nextInputBox` | SB-3.26, ED-3.47 |
| `search.focus.previousInputBox` | SB-3.26, ED-3.47 |
| `search.searchEditor.action.deleteFileResults` | ED-3.37 |
| `selectAllSearchEditorMatches` | ED-3.38 |
| `selectFirstSuggestion` | AC-1.36 |
| `selectForCompare` | SB-2.17, SB-2.32, ED-6.11 |
| `selectLastSuggestion` | AC-1.36 |
| `selectNextCodeAction` | ED-4.14 |
| `selectNextPageSuggestion` | AC-1.36 |
| `selectNextSuggestion` | AC-1.30, AC-1.36 |
| `selectPrevCodeAction` | ED-4.14 |
| `selectPrevPageSuggestion` | AC-1.36 |
| `selectPrevSuggestion` | AC-1.36 |
| `settings.action.clearSearchResults` | ED-8.2 |
| `settings.action.focusLevelUp` | ED-8.4, ED-8.43 |
| `settings.action.focusSearchFromSettings` | ED-8.43 |
| `settings.action.focusSettingControl` | ED-8.5, ED-8.43 |
| `settings.action.focusSettingsFile` | ED-8.43 |
| `settings.action.focusSettingsFromSearch` | ED-8.2, ED-8.43 |
| `settings.action.focusSettingsFromSearchOnEnter` | ED-8.2, ED-8.43 |
| `settings.action.focusSettingsList` | ED-8.4, ED-8.43 |
| `settings.action.focusTOC` | ED-8.4, ED-8.43 |
| `settings.action.search` | ED-8.2 |
| `settings.action.showAIResults` | ED-8.2 |
| `settings.action.showContextMenu` | ED-8.5, ED-8.43 |
| `settings.action.showPreviousSearch` | ED-8.2 |
| `settings.action.suggestFilters` | ED-8.2, ED-8.41 |
| `settings.action.toggleAiSearch` | ED-8.2 |
| `settings.applyToAllProfiles` | ED-8.5 |
| `settings.copySettingAsJSON` | ED-8.5 |
| `settings.copySettingAsURL` | ED-8.5 |
| `settings.copySettingId` | ED-8.5 |
| `settings.filterByOnline` | ED-8.2, ED-8.42 |
| `settings.filterByTelemetry` | ED-8.2, ED-8.42 |
| `settings.filterUntrusted` | ED-8.2, ED-8.6, ED-8.42 |
| `settings.resetSetting` | ED-8.5 |
| `settings.stopSyncingSetting` | ED-8.5, ED-8.50 |
| `settings.switchToJSON` | ED-8.8 |
| `showCustomTitleBar` | TB-3.10 |
| `showEditorScreenReaderNotification` | SS-3.26 |
| `showMoreActions` | ED-4.36 |
| `showNextParameterHint` | AC-3.6, AC-3.7 |
| `showPrevParameterHint` | AC-3.6, AC-3.7 |
| `signals.sounds.help` | WB-4.5 |
| `simpleBrowser.api.open` | ED-10.25 |
| `simpleBrowser.show` | ED-10.25 |
| `statusBar.entry.showTooltip` | AC-8.2, SS-1.11, SS-3.29, SS-6.6 |
| `suggestWidgetCopy` | AC-1.25 |
| `tab` | ED-2.17 |
| `terminal.integrated.suggest.clearCachedGlobals` | AC-5.25 |
| `terminalSuggestToggleExplainMode` | AC-5.14 |
| `testing.callStack.debug` | PN-6.28 |
| `testing.callStack.run` | PN-6.28 |
| `testing.cancelRun` | SB-6.13, PN-6.26 |
| `testing.cancelTestRefresh` | SB-6.14 |
| `testing.clearTestResults` | PN-6.30 |
| `testing.collapseAll` | SB-6.2 |
| `testing.collapsePeekStack` | ED-13.4, PN-6.25 |
| `testing.configureProfile` | SB-6.11 |
| `testing.continuousRunUsingForTest` | SB-6.12, SB-6.15 |
| `testing.coverage` | SB-6.9, SB-6.15 |
| `testing.coverage.close` | SB-6.27 |
| `testing.coverage.goToNextMissedLine` | ED-13.5 |
| `testing.coverage.goToPreviousMissedLine` | ED-13.5 |
| `testing.coverage.uri` | SB-2.49 |
| `testing.coverageAll` | SB-6.9 |
| `testing.coverageAtCursor` | SB-6.30 |
| `testing.coverageCurrentFile` | SB-6.29 |
| `testing.coverageFilterToTest` | SB-6.26 |
| `testing.coverageFilterToTestInEditor` | ED-13.5 |
| `testing.coverageLastRun` | SB-6.21, PN-6.26 |
| `testing.coverageSelected` | SB-6.9 |
| `testing.coverageToggleToolbar` | ED-13.5 |
| `testing.coverageViewChangeSorting` | SB-6.25 |
| `testing.coverageViewCollapseAll` | SB-6.24 |
| `testing.debug` | SB-6.8, SB-6.15 |
| `testing.debug.uri` | SB-2.49 |
| `testing.debugAll` | SB-6.8 |
| `testing.debugAtCursor` | SB-6.30 |
| `testing.debugCurrentFile` | SB-6.29 |
| `testing.debugFailedFromLastRun` | SB-6.21, PN-6.27 |
| `testing.debugFailTests` | SB-6.21 |
| `testing.debugLastRun` | SB-6.20, SB-6.21, PN-6.26 |
| `testing.debugSelected` | SB-6.8 |
| `testing.editFocusedTest` | SB-6.15, SB-6.16 |
| `testing.goToNextMessage` | ED-13.4 |
| `testing.goToPreviousMessage` | ED-13.4 |
| `testing.goToRelatedCode` | ED-4.35 |
| `testing.goToRelatedTest` | ED-4.35 |
| `testing.gutter.cancel` | ED-13.2 |
| `testing.gutter.peekFailure` | ED-13.2 |
| `testing.gutter.reveal` | ED-13.2 |
| `testing.hideTest` | SB-6.15, SB-6.17 |
| `testing.openCoverage` | SB-6.27 |
| `testing.openMessageInEditor` | ED-13.4 |
| `testing.openOutputPeek` | ED-13.4 |
| `testing.peekRelatedCode` | ED-4.35 |
| `testing.peekRelatedTest` | ED-4.35 |
| `testing.refreshTests` | SB-6.14 |
| `testing.reRunFailedFromLastRun` | SB-6.21, PN-6.27 |
| `testing.reRunFailTests` | SB-6.21 |
| `testing.reRunLastRun` | SB-6.20, SB-6.21, PN-6.26 |
| `testing.run` | SB-6.7, SB-6.15 |
| `testing.run.uri` | SB-2.49 |
| `testing.runAll` | SB-6.6 |
| `testing.runAtCursor` | SB-6.30 |
| `testing.runCurrentFile` | SB-6.29 |
| `testing.runSelected` | SB-6.6 |
| `testing.runUsing` | SB-6.10, SB-6.15, ED-13.2 |
| `testing.searchForTestExtension` | SB-6.22 |
| `testing.selectDefaultTestProfiles` | SB-6.11 |
| `testing.showMostRecentOutput` | PN-6.18 |
| `testing.sortByDuration` | SB-6.3 |
| `testing.sortByLocation` | SB-6.3 |
| `testing.sortByStatus` | SB-6.3 |
| `testing.startContinuousRun` | SB-6.12 |
| `testing.stopContinuousRun` | SB-6.12 |
| `testing.toggleContinuousRunForTest` | SB-6.12, SB-6.15 |
| `testing.toggleCoverageInExplorer` | SB-6.28 |
| `testing.toggleInlineCoverage` | ED-13.5 |
| `testing.toggleInlineTestOutput` | ED-13.3 |
| `testing.toggleResultsViewLayout` | PN-6.31 |
| `testing.toggleTestingPeekHistory` | ED-13.4 |
| `testing.unhideAllTests` | SB-6.17 |
| `testing.unhideTest` | SB-6.15, SB-6.17 |
| `testing.viewAsList` | SB-6.2 |
| `testing.viewAsTree` | SB-6.2 |
| `the` | ED-7.47, ED-8.41, ED-8.45, QA-4.9, WB-2.5 |
| `they` | ED-5.41 |
| `timeline.focus` | SB-7.7 |
| `timeline.refresh` | SB-7.7 |
| `timeline.showWorkspaceTimeline` | SB-7.11 |
| `timeline.toggleExcludeSource:<sourceId>` | SB-7.9 |
| `timeline.toggleFollowActiveEditor` | SB-7.8 |
| `toggle.diff.ignoreTrimWhitespace` | ED-6.3 |
| `toggle.diff.renderSideBySide` | ED-6.2 |
| `toggle.toggleCustomTitleBar` | TB-1.31, TB-3.10 |
| `toggle.window.commandCenter` | TB-2.2 |
| `toggle.window.customTitleBarVisibility` | TB-3.10 |
| `toggle.window.customTitleBarVisibility.windowed` | TB-3.11 |
| `toggle.window.titleBarStyle` | TB-3.12 |
| `toggle.workbench.browser.showInTitleBar` | TB-5.9 |
| `toggle.workbench.layoutControl.enabled` | TB-4.1 |
| `toggle.workbench.navigationControl.enabled` | TB-2.5 |
| `toggle.workbench.notifications.showInTitleBar` | TB-5.8 |
| `toggleExplainMode` | AC-1.26 |
| `toggleFindCaseSensitive` | ED-3.9 |
| `toggleFindInSelection` | ED-3.12 |
| `toggleFindRegex` | ED-3.11 |
| `toggleFindWholeWord` | ED-3.10 |
| `togglePeekWidgetFocus` | ED-4.10 |
| `togglePreserveCase` | ED-3.17 |
| `toggleQuickDiffWidgetFocus` | ED-6.48 |
| `toggleSearchCaseSensitive` | SB-3.3 |
| `toggleSearchEditorCaseSensitive` | ED-3.31 |
| `toggleSearchEditorContextLines` | ED-3.34 |
| `toggleSearchEditorRegex` | ED-3.31 |
| `toggleSearchEditorWholeWord` | ED-3.31 |
| `toggleSearchPreserveCase` | SB-3.6 |
| `toggleSearchRegex` | SB-3.3 |
| `toggleSearchWholeWord` | SB-3.3 |
| `toggleSectionCodeAction` | ED-4.14 |
| `toggleSuggestionDetails` | AC-1.21, AC-1.29 |
| `toggleSuggestionFocus` | AC-1.25 |
| `tree.toggleStickyScroll` | SB-1.24 |
| `type` | ED-2.22, ED-2.23 |
| `typescript.selectTypeScriptVersion` | SS-3.43, SS-3.44 |
| `undo` | SB-2.10, ED-2.31, ED-2.43, ED-4.41, ED-7.40, ED-7.41, ED-10.73, DL-4.13 |
| `update.cancelling` | TB-1.17, TB-5.3, AB-4.21 |
| `update.check` | TB-1.17, AB-4.21 |
| `update.checkForUpdate` | NT-6.4, DL-1.21, DL-1.30 |
| `update.checking` | TB-1.17, TB-5.3, AB-4.21 |
| `update.downloading` | TB-1.17, TB-5.3, AB-4.21 |
| `update.downloadNow` | TB-1.17, TB-5.2, TB-5.4, AB-4.21 |
| `update.downloadUpdate` | NT-6.4, NT-6.9, DL-1.21 |
| `update.install` | TB-1.17, TB-5.2, TB-5.4, AB-4.21 |
| `update.installUpdate` | NT-6.4, DL-1.21 |
| `update.restart` | TB-1.17, TB-5.2, TB-5.4, AB-4.21 |
| `update.restartToUpdate` | TB-5.2, NT-6.4, DL-1.21 |
| `update.showCurrentReleaseNotes` | TB-5.4, ED-9.26, NT-6.9, NT-6.13, NT-6.14 |
| `update.showCurrentReleaseNotesFromCurrentFile` | ED-9.26 |
| `update.showUpdateInfo` | NT-6.13 |
| `update.showUpdateReleaseNotes` | AB-4.22 |
| `update.updating` | TB-1.17, TB-5.3, AB-4.21 |
| `urlOpenToast.dontOpen` | ED-10.88 |
| `variables.collapse` | SB-5.11 |
| `views.moveViewDown` | SB-1.15 |
| `views.moveViewLeft` | SB-1.15 |
| `views.moveViewRight` | SB-1.15 |
| `views.moveViewUp` | SB-1.15 |
| `vscode.changes` | ED-6.1 |
| `vscode.diff` | ED-6.1 |
| `vscode.getEditorLayout` | ED-1.16 |
| `vscode.open` | SS-3.46, SS-3.49 |
| `vscode.openFolder` | SB-4.40 |
| `vscode.openIssueReporter` | WB-6.5, WB-6.6 |
| `vscode.peekTestError` | ED-13.3 |
| `vscode.pickMultipleTestProfiles` | SB-6.11 |
| `vscode.pickTestProfile` | SB-6.10 |
| `vscode.revealTest` | SB-6.16, PN-6.29 |
| `vscode.runTestsById` | PN-6.28 |
| `vscode.setEditorLayout` | ED-1.16 |
| `walkthroughs.selectStep` | ED-9.2, ED-9.8 |
| `watch.collapse` | SB-5.18 |
| `welcome.goBack` | ED-9.1, ED-9.8 |
| `welcome.markStepComplete` | ED-9.9 |
| `welcome.markStepIncomplete` | ED-9.9 |
| `welcome.newWorkspaceChat` | ED-9.4 |
| `welcome.showAllWalkthroughs` | TB-1.30, ED-9.4, ED-9.7 |
| `welcome.showNewFileEntries` | ED-9.4, ED-9.24, ED-9.25 |
| `widgetNavigation.focusNext` | WB-3.1 |
| `widgetNavigation.focusPrevious` | WB-3.1 |
| `workbench.action.acceptSelectedQuickOpenItem` | QA-6.22 |
| `workbench.action.activityBar.size.compact` | AB-5.6 |
| `workbench.action.activityBar.size.default` | AB-5.6 |
| `workbench.action.activityBarLocation.bottom` | TB-1.31, AB-5.2, AX-4.3 |
| `workbench.action.activityBarLocation.default` | TB-1.31, AB-5.2, AX-4.3 |
| `workbench.action.activityBarLocation.hide` | TB-1.31, AB-5.2, AB-5.4, AX-4.3 |
| `workbench.action.activityBarLocation.top` | TB-1.31, AB-5.2, AX-4.3 |
| `workbench.action.addComment` | ED-12.2 |
| `workbench.action.addRootFolder` | SB-2.36, SB-4.40, ED-9.18, DL-3.5, DL-5.6 |
| `workbench.action.alignPanelCenter` | PN-1.4 |
| `workbench.action.alignPanelJustify` | PN-1.4 |
| `workbench.action.alignPanelLeft` | PN-1.4 |
| `workbench.action.alignPanelRight` | PN-1.4 |
| `workbench.action.alignQuickInputCenter` | QA-6.17 |
| `workbench.action.alignQuickInputTop` | QA-6.17 |
| `workbench.action.alternativeAcceptSelectedQuickOpenItem` | QA-6.22 |
| `workbench.action.askVScode` | TB-1.30 |
| `workbench.action.auxiliarybar.toggleShowLabels` | AX-4.2 |
| `workbench.action.blur` | WB-3.2 |
| `workbench.action.browseColorThemesInMarketplace` | ED-8.15 |
| `workbench.action.browser.clearEphemeralStorage` | ED-10.64 |
| `workbench.action.browser.clearGlobalStorage` | ED-10.64 |
| `workbench.action.browser.clearWorkspaceStorage` | ED-10.64 |
| `workbench.action.browser.closeAll` | ED-10.66 |
| `workbench.action.browser.closeAllInGroup` | ED-10.66 |
| `workbench.action.browser.findNext` | ED-10.77 |
| `workbench.action.browser.findPrevious` | ED-10.77 |
| `workbench.action.browser.focusUrlInput` | ED-10.81 |
| `workbench.action.browser.goBack` | ED-10.79 |
| `workbench.action.browser.goForward` | ED-10.79 |
| `workbench.action.browser.hardReload` | ED-10.79 |
| `workbench.action.browser.hideFind` | ED-10.77 |
| `workbench.action.browser.managePermissions` | ED-10.63 |
| `workbench.action.browser.newTab` | ED-10.80 |
| `workbench.action.browser.open` | TB-5.9 |
| `workbench.action.browser.openExternal` | ED-10.67 |
| `workbench.action.browser.openFile` | SB-2.38 |
| `workbench.action.browser.openOrList` | TB-5.9 |
| `workbench.action.browser.openSettings` | ED-10.87 |
| `workbench.action.browser.pickDevicePreset` | ED-10.61 |
| `workbench.action.browser.quickOpen` | TB-5.9, ED-10.91 |
| `workbench.action.browser.reload` | ED-10.79 |
| `workbench.action.browser.resetEmulation` | ED-10.61 |
| `workbench.action.browser.resetZoom` | ED-10.76 |
| `workbench.action.browser.setUserAgent` | ED-10.62 |
| `workbench.action.browser.showFind` | ED-10.77 |
| `workbench.action.browser.showHistory` | ED-10.65 |
| `workbench.action.browser.toggleDeviceEmulation` | ED-10.61 |
| `workbench.action.browser.toggleDevTools` | ED-10.78 |
| `workbench.action.browser.toggleFavorite` | ED-10.37 |
| `workbench.action.browser.toggleMobileEmulation` | ED-10.61 |
| `workbench.action.browser.zoomIn` | ED-10.76 |
| `workbench.action.browser.zoomOut` | ED-10.76 |
| `workbench.action.chat.open` | ED-9.18, QA-1.19 |
| `workbench.action.chat.openImageInCarousel` | ED-10.44 |
| `workbench.action.clearCommandHistory` | QA-1.10 |
| `workbench.action.clearEditorHistory` | ED-1.32 |
| `workbench.action.clearEditorHistoryWithoutConfirm` | ED-1.32 |
| `workbench.action.clearLocalePreference` | WB-5.1 |
| `workbench.action.clearRecentFiles` | TB-1.16, TB-1.27, DL-1.26 |
| `workbench.action.closeActiveEditor` | SB-2.30, SB-2.32, ED-1.4, ED-1.8, ED-10.30, DL-2.1 |
| `workbench.action.closeActivePinnedEditor` | ED-1.8 |
| `workbench.action.closeAllEditors` | ED-1.9, DL-2.3 |
| `workbench.action.closeAllGroups` | ED-1.9, DL-2.3 |
| `workbench.action.closeAuxiliaryBar` | AX-1.2 |
| `workbench.action.closeEditorInAllGroups` | ED-1.9 |
| `workbench.action.closeEditorsAndGroup` | ED-1.9 |
| `workbench.action.closeEditorsInGroup` | SB-2.32, ED-1.9, DL-2.1 |
| `workbench.action.closeEditorsInOtherGroups` | ED-1.9 |
| `workbench.action.closeEditorsToTheLeft` | ED-1.9 |
| `workbench.action.closeEditorsToTheRight` | ED-1.9, DL-2.1 |
| `workbench.action.closeFolder` | DL-2.5 |
| `workbench.action.closeGroup` | ED-1.9, ED-1.25 |
| `workbench.action.closeModalEditor` | ED-1.36 |
| `workbench.action.closeOtherEditors` | SB-2.32, ED-1.9, DL-2.1 |
| `workbench.action.closeOtherEditorTabInGroup` | ED-1.50 |
| `workbench.action.closeOtherWindows` | WB-9.9 |
| `workbench.action.closePanel` | PN-1.1, PN-1.20 |
| `workbench.action.closeQuickOpen` | QA-2.19, QA-5.15, QA-6.22 |
| `workbench.action.closeSidebar` | SB-1.1 |
| `workbench.action.closeUnmodifiedEditors` | SB-2.32, ED-1.9 |
| `workbench.action.closeWindow` | TB-3.4, DL-2.5, DL-2.18 |
| `workbench.action.closeWindowOrEditorOnMac` | DL-2.18 |
| `workbench.action.collapseAllComments` | ED-12.7 |
| `workbench.action.compareEditor.focusOtherSide` | ED-6.9 |
| `workbench.action.compareEditor.focusPrimarySide` | ED-6.9 |
| `workbench.action.compareEditor.focusSecondarySide` | ED-6.9 |
| `workbench.action.compareEditor.nextChange` | ED-6.6 |
| `workbench.action.compareEditor.openSide` | ED-6.9 |
| `workbench.action.compareEditor.previousChange` | ED-6.6 |
| `workbench.action.compareEditor.swapSides` | ED-6.9 |
| `workbench.action.configureDoNotDisturbMode` | NT-4.2 |
| `workbench.action.configureEditor` | ED-1.1 |
| `workbench.action.configureEditorLargeFileConfirmation` | DL-4.15 |
| `workbench.action.configureEditorTabs` | ED-1.1 |
| `workbench.action.configureLanguageBasedSettings` | ED-8.7 |
| `workbench.action.configureLocale` | WB-5.1 |
| `workbench.action.configureMeteredConnection` | SS-3.37 |
| `workbench.action.configureNotification` | NT-2.12, NT-4.4 |
| `workbench.action.configureNotificationsPosition` | NT-3.11 |
| `workbench.action.configurePreferredDropAction` | ED-2.29 |
| `workbench.action.configurePreferredPasteAction` | ED-2.29 |
| `workbench.action.configureRuntimeArguments` | ED-8.68 |
| `workbench.action.copyEditorGroupToNewWindow` | ED-1.25, ED-1.33 |
| `workbench.action.copyEditorToNewWindow` | ED-1.33 |
| `workbench.action.copyNotificationMessage` | NT-2.16 |
| `workbench.action.copyRecentCppActions` | AC-8.18 |
| `workbench.action.createTerminalEditor` | PN-2.7 |
| `workbench.action.createTerminalEditorSameGroup` | PN-2.7 |
| `workbench.action.createTerminalEditorSide` | PN-2.7 |
| `workbench.action.customize.focusSearch` | ED-8.33 |
| `workbench.action.customize.openCommands` | ED-8.39 |
| `workbench.action.customize.openHooks` | ED-8.40 |
| `workbench.action.customize.openMCPs` | ED-8.35 |
| `workbench.action.customize.openPlugins` | ED-8.34 |
| `workbench.action.customize.openRules` | ED-8.38 |
| `workbench.action.customize.openSkillPublishLogs` | ED-8.36 |
| `workbench.action.customize.openSkills` | ED-8.36 |
| `workbench.action.customize.openSubagents` | ED-8.37 |
| `workbench.action.customizeLayout` | TB-4.6, TB-4.7, PN-1.4, SS-1.1 |
| `workbench.action.debug.callStackBottom` | SB-5.19 |
| `workbench.action.debug.callStackDown` | SB-5.19 |
| `workbench.action.debug.callStackTop` | SB-5.19 |
| `workbench.action.debug.callStackUp` | SB-5.19 |
| `workbench.action.debug.configure` | AB-6.13, SB-5.5, SB-5.8 |
| `workbench.action.debug.continue` | SB-5.9, SB-5.20, ED-11.17, WB-9.6 |
| `workbench.action.debug.disconnect` | SB-5.9, SB-5.20, ED-11.17 |
| `workbench.action.debug.disconnectAndSuspend` | SB-5.9, SB-5.20 |
| `workbench.action.debug.focusProcess` | SB-5.10, ED-11.17 |
| `workbench.action.debug.nextConsole` | PN-6.16 |
| `workbench.action.debug.pause` | SB-5.9, SB-5.20, ED-11.17, WB-9.6 |
| `workbench.action.debug.prevConsole` | PN-6.16 |
| `workbench.action.debug.restart` | SB-5.9, SB-5.20, ED-11.17, WB-9.6 |
| `workbench.action.debug.restartFrame` | SB-5.21 |
| `workbench.action.debug.reverseContinue` | SB-5.9, ED-11.17 |
| `workbench.action.debug.run` | SB-5.3, WB-9.6 |
| `workbench.action.debug.selectandstart` | SB-5.3, SS-5.10, QA-4.15, QA-5.3, QA-5.6 |
| `workbench.action.debug.selectDebugConsole` | PN-6.16, QA-4.17, QA-5.6 |
| `workbench.action.debug.selectDebugSession` | QA-4.18 |
| `workbench.action.debug.selectRepl` | PN-6.16 |
| `workbench.action.debug.showLoadedScripts` | SB-5.31 |
| `workbench.action.debug.start` | AB-6.13, SB-5.3, SB-5.4, SB-5.8, ED-9.18, WB-9.6 |
| `workbench.action.debug.stepBack` | SB-5.9, ED-11.17 |
| `workbench.action.debug.stepInto` | SB-5.9, SB-5.20, ED-11.17, WB-9.6 |
| `workbench.action.debug.stepIntoTarget` | ED-11.22 |
| `workbench.action.debug.stepOut` | SB-5.9, SB-5.20, ED-11.17, WB-9.6 |
| `workbench.action.debug.stepOver` | SB-5.9, SB-5.20, ED-11.17, WB-9.6 |
| `workbench.action.debug.stop` | SB-5.9, SB-5.20, ED-11.17, WB-9.6 |
| `workbench.action.debug.terminateThread` | SB-5.20 |
| `workbench.action.decreaseViewHeight` | ED-1.51 |
| `workbench.action.decreaseViewSize` | SB-1.4, PN-1.8, PN-1.20 |
| `workbench.action.decreaseViewWidth` | SB-1.4, ED-1.51 |
| `workbench.action.developer.captureAndSendDebuggingData` | WB-8.8 |
| `workbench.action.developer.captureProfile` | WB-8.9 |
| `workbench.action.developer.forceRendererOOM` | WB-8.10 |
| `workbench.action.developer.showProfilingStatus` | WB-8.9 |
| `workbench.action.developer.toggleWatchForCrashes` | WB-8.10 |
| `workbench.action.disableCompactAuxiliaryWindow` | TB-3.21 |
| `workbench.action.disableWindowAlwaysOnTop` | TB-3.20, TB-4.19 |
| `workbench.action.dumpFeatureGates` | WB-8.11 |
| `workbench.action.duplicateActiveEditorGroupDown` | ED-1.17 |
| `workbench.action.duplicateActiveEditorGroupLeft` | ED-1.17 |
| `workbench.action.duplicateActiveEditorGroupRight` | ED-1.17 |
| `workbench.action.duplicateActiveEditorGroupUp` | ED-1.17 |
| `workbench.action.duplicateWorkspaceInNewWindow` | WB-9.1 |
| `workbench.action.editListItem` | ED-8.47 |
| `workbench.action.editor.changeEncoding` | SS-3.11, SS-3.12, SS-3.13 |
| `workbench.action.editor.changeEOL` | SS-3.15 |
| `workbench.action.editor.changeLanguageMode` | ED-9.24, SS-3.17, SS-3.18, SS-3.21, SS-3.22 |
| `workbench.action.editor.nextChange` | ED-6.25 |
| `workbench.action.editor.previousChange` | ED-6.25 |
| `workbench.action.editorActionsDefault` | ED-1.1 |
| `workbench.action.editorActionsTitleBar` | ED-1.1 |
| `workbench.action.editorDictation.start` | ED-2.38 |
| `workbench.action.editorDictation.stop` | ED-2.38 |
| `workbench.action.editorLayoutSingle` | ED-1.16 |
| `workbench.action.editorLayoutThreeColumns` | ED-1.16 |
| `workbench.action.editorLayoutThreeRows` | ED-1.16 |
| `workbench.action.editorLayoutTwoByTwoGrid` | ED-1.16 |
| `workbench.action.editorLayoutTwoColumns` | ED-1.16 |
| `workbench.action.editorLayoutTwoColumnsBottom` | ED-1.16 |
| `workbench.action.editorLayoutTwoRows` | ED-1.16 |
| `workbench.action.editorLayoutTwoRowsRight` | ED-1.16 |
| `workbench.action.enableCompactAuxiliaryWindow` | TB-3.21 |
| `workbench.action.enableWindowAlwaysOnTop` | TB-3.20, TB-4.19 |
| `workbench.action.evenEditorWidths` | ED-1.20 |
| `workbench.action.evenEditorWidthsExcludingAgent` | ED-1.21 |
| `workbench.action.exitZenMode` | ED-1.34, NT-4.10 |
| `workbench.action.expandAllComments` | ED-12.7 |
| `workbench.action.expandUnresolvedComments` | ED-12.7 |
| `workbench.action.exportLogs` | PN-5.12 |
| `workbench.action.files.copyPathOfActiveFile` | SB-2.16 |
| `workbench.action.files.newFile` | DL-3.10 |
| `workbench.action.files.newProject` | ED-9.20 |
| `workbench.action.files.newUntitledFile` | TB-1.27, ED-9.18, ED-9.25, WB-1.5 |
| `workbench.action.files.openFile` | SB-5.8, ED-9.4, ED-9.18, DL-3.1 |
| `workbench.action.files.openFileFolder` | TB-1.27, ED-9.4, ED-9.18, DL-3.3 |
| `workbench.action.files.openFileFolderInNewWindow` | DL-3.3 |
| `workbench.action.files.openFileInNewWindow` | DL-3.1 |
| `workbench.action.files.openFolder` | TB-1.27, SB-2.36, SB-5.8, ED-9.4, ED-9.5, ED-9.18, ED-9.20, DL-3.2 |
| `workbench.action.files.openFolderInNewWindow` | DL-3.2 |
| `workbench.action.files.openFolderViaWorkspace` | SB-2.36, ED-9.4, DL-3.2 |
| `workbench.action.files.openLocalFile` | DL-3.19 |
| `workbench.action.files.openLocalFileFolder` | DL-3.19 |
| `workbench.action.files.openLocalFolder` | DL-3.19 |
| `workbench.action.files.resetActiveEditorReadonlyInSession` | ED-2.37 |
| `workbench.action.files.revealActiveFileInWindows` | SB-2.15 |
| `workbench.action.files.revert` | SB-2.32 |
| `workbench.action.files.revertEditors` | WB-1.2 |
| `workbench.action.files.save` | SB-2.32, ED-7.40, DL-3.7, DL-4.14 |
| `workbench.action.files.saveAll` | SB-2.31 |
| `workbench.action.files.saveAs` | DL-3.6 |
| `workbench.action.files.saveEditors` | WB-1.2 |
| `workbench.action.files.saveFiles` | WB-1.3 |
| `workbench.action.files.saveLocalFile` | DL-3.19 |
| `workbench.action.files.saveWithoutFormatting` | WB-1.4 |
| `workbench.action.files.setActiveEditorReadonlyInSession` | ED-2.37 |
| `workbench.action.files.setActiveEditorWriteableInSession` | ED-2.37 |
| `workbench.action.files.showOpenedFileInNewWindow` | ED-1.44 |
| `workbench.action.files.toggleActiveEditorReadonlyInSession` | ED-2.37 |
| `workbench.action.findInFiles` | SB-3.1, SB-3.29, ED-9.18 |
| `workbench.action.firstEditorInGroup` | ED-1.29 |
| `workbench.action.focusAboveGroup` | ED-1.22 |
| `workbench.action.focusAboveGroupWithoutWrap` | ED-1.22 |
| `workbench.action.focusActiveEditorGroup` | ED-1.22 |
| `workbench.action.focusActivityBar` | AB-1.11, AB-4.23 |
| `workbench.action.focusAuxiliaryBar` | AX-1.3 |
| `workbench.action.focusBanner` | BN-1.6 |
| `workbench.action.focusBelowGroup` | ED-1.22 |
| `workbench.action.focusBelowGroupWithoutWrap` | ED-1.22 |
| `workbench.action.focusBrowserLocationBar` | ED-10.31 |
| `workbench.action.focusCommentOnCurrentLine` | ED-12.6 |
| `workbench.action.focusCommentsPanel` | PN-7.1 |
| `workbench.action.focusEighthEditorGroup` | ED-1.22 |
| `workbench.action.focusFifthEditorGroup` | ED-1.22 |
| `workbench.action.focusFirstEditorGroup` | ED-1.22 |
| `workbench.action.focusFirstSideEditor` | ED-1.24 |
| `workbench.action.focusFourthEditorGroup` | ED-1.22 |
| `workbench.action.focusLastEditorGroup` | ED-1.22 |
| `workbench.action.focusLeftGroup` | ED-1.22 |
| `workbench.action.focusLeftGroupWithoutWrap` | ED-1.22 |
| `workbench.action.focusNextGroup` | ED-1.22 |
| `workbench.action.focusNextPart` | SS-1.7 |
| `workbench.action.focusOtherSideEditor` | ED-1.24 |
| `workbench.action.focusPanel` | PN-1.16, PN-1.20 |
| `workbench.action.focusPreviousGroup` | ED-1.22 |
| `workbench.action.focusPreviousPart` | SS-1.7 |
| `workbench.action.focusQuickOpen` | QA-6.22 |
| `workbench.action.focusRightGroup` | ED-1.22 |
| `workbench.action.focusRightGroupWithoutWrap` | ED-1.22 |
| `workbench.action.focusSecondEditorGroup` | ED-1.22 |
| `workbench.action.focusSecondSideEditor` | ED-1.24 |
| `workbench.action.focusSeventhEditorGroup` | ED-1.22 |
| `workbench.action.focusSideBar` | SB-1.3 |
| `workbench.action.focusSixthEditorGroup` | ED-1.22 |
| `workbench.action.focusStatusBar` | SS-1.7 |
| `workbench.action.focusThirdEditorGroup` | ED-1.22 |
| `workbench.action.focusTitleBar` | TB-1.7 |
| `workbench.action.focusWindow` | WB-9.4 |
| `workbench.action.generateColorTheme` | ED-8.15 |
| `workbench.action.getStartedWithAccessibilityFeatures` | TB-1.30, ED-9.10 |
| `workbench.action.gotoLine` | SS-3.3, QA-3.1, QA-3.2, QA-3.5, QA-3.6, QA-3.7, QA-3.8, QA-5.6 |
| `workbench.action.gotoOffset` | QA-3.3, QA-5.6 |
| `workbench.action.gotoSymbol` | QA-3.7, QA-3.8, QA-3.10, QA-3.11, QA-3.12, QA-3.13, QA-3.14, QA-3.15, QA-5.3, QA-5.6 |
| `workbench.action.hideComment` | ED-12.5 |
| `workbench.action.hideEditorActions` | ED-1.1 |
| `workbench.action.hideEditorTabs` | ED-1.1 |
| `workbench.action.holdLockedScrolling` | SS-3.27 |
| `workbench.action.importLog` | PN-5.13 |
| `workbench.action.increaseViewHeight` | ED-1.51 |
| `workbench.action.increaseViewSize` | SB-1.4, PN-1.8, PN-1.20 |
| `workbench.action.increaseViewWidth` | SB-1.4, ED-1.51 |
| `workbench.action.inspectContextKeys` | WB-8.1 |
| `workbench.action.inspectKeyMappings` | WB-8.24 |
| `workbench.action.inspectKeyMappingsJSON` | WB-8.24 |
| `workbench.action.installCommandLine` | WB-9.3 |
| `workbench.action.interactivePlayground.arrowDown` | ED-9.13 |
| `workbench.action.interactivePlayground.arrowUp` | ED-9.13 |
| `workbench.action.interactivePlayground.pageDown` | ED-9.13 |
| `workbench.action.interactivePlayground.pageUp` | ED-9.13 |
| `workbench.action.issueReporter.captureScreenshot` | WB-6.5 |
| `workbench.action.issueReporter.toggleRecording` | WB-6.5 |
| `workbench.action.joinAllGroups` | ED-1.18 |
| `workbench.action.joinEditorInGroup` | ED-1.24 |
| `workbench.action.joinTwoGroups` | ED-1.18 |
| `workbench.action.keepEditor` | ED-1.6 |
| `workbench.action.keybindingsReference` | TB-1.30 |
| `workbench.action.lastEditorInGroup` | ED-1.29 |
| `workbench.action.localHistory.compareWithFile` | SB-7.13 |
| `workbench.action.localHistory.compareWithPrevious` | SB-7.13 |
| `workbench.action.localHistory.compareWithSelected` | SB-7.13 |
| `workbench.action.localHistory.create` | SB-7.15 |
| `workbench.action.localHistory.delete` | SB-7.15 |
| `workbench.action.localHistory.deleteAll` | SB-7.15 |
| `workbench.action.localHistory.open` | SB-7.14 |
| `workbench.action.localHistory.rename` | SB-7.15 |
| `workbench.action.localHistory.restore` | SB-7.14 |
| `workbench.action.localHistory.restoreViaEditor` | SB-7.14 |
| `workbench.action.localHistory.restoreViaPicker` | SB-7.14 |
| `workbench.action.localHistory.revealInOS` | SB-7.14 |
| `workbench.action.localHistory.selectForCompare` | SB-7.13 |
| `workbench.action.lockEditorGroup` | ED-1.23 |
| `workbench.action.logDataDirectories` | WB-8.12 |
| `workbench.action.logStorage` | WB-8.2 |
| `workbench.action.logWorkingCopies` | WB-8.3 |
| `workbench.action.manageAccounts` | AB-4.2, AB-4.3 |
| `workbench.action.manageTrustedDomain` | DL-5.13 |
| `workbench.action.maximizeAuxiliaryBar` | AX-3.1 |
| `workbench.action.maximizeChatSize` | ED-9.18 |
| `workbench.action.maximizeEditorHideSidebar` | ED-1.19 |
| `workbench.action.mergeAllWindowTabs` | TB-1.25, TB-3.15 |
| `workbench.action.minimizeOtherEditors` | ED-1.20 |
| `workbench.action.minimizeOtherEditorsHideSidebar` | ED-1.20 |
| `workbench.action.moveActiveEditorGroupDown` | ED-1.18 |
| `workbench.action.moveActiveEditorGroupLeft` | ED-1.18 |
| `workbench.action.moveActiveEditorGroupRight` | ED-1.18 |
| `workbench.action.moveActiveEditorGroupUp` | ED-1.18 |
| `workbench.action.moveEditorGroupToNewWindow` | ED-1.25, ED-1.33 |
| `workbench.action.moveEditorLeftInGroup` | ED-1.12 |
| `workbench.action.moveEditorRightInGroup` | ED-1.12 |
| `workbench.action.moveEditorToAboveGroup` | ED-1.14 |
| `workbench.action.moveEditorToBelowGroup` | ED-1.14 |
| `workbench.action.moveEditorToEnd` | ED-1.12 |
| `workbench.action.moveEditorToFirstGroup` | ED-1.14 |
| `workbench.action.moveEditorToLastGroup` | ED-1.14 |
| `workbench.action.moveEditorToLeftGroup` | ED-1.14 |
| `workbench.action.moveEditorToNewWindow` | ED-1.33 |
| `workbench.action.moveEditorToNextGroup` | ED-1.14 |
| `workbench.action.moveEditorToPreviousGroup` | ED-1.14 |
| `workbench.action.moveEditorToRightGroup` | ED-1.14 |
| `workbench.action.moveEditorToStart` | ED-1.12 |
| `workbench.action.moveFocusedView` | SB-1.14 |
| `workbench.action.moveModalEditorToMain` | ED-1.36 |
| `workbench.action.moveModalEditorToWindow` | ED-1.36 |
| `workbench.action.movePanelToSecondarySideBar` | AX-5.3 |
| `workbench.action.movePanelToSidePanel` | AX-5.3 |
| `workbench.action.moveSecondarySideBarToPanel` | PN-1.19 |
| `workbench.action.moveSideBarLeft` | SB-1.2, AX-2.1 |
| `workbench.action.moveSideBarRight` | SB-1.2, AX-2.1 |
| `workbench.action.moveSidePanelToPanel` | PN-1.19 |
| `workbench.action.moveView` | SB-1.14 |
| `workbench.action.moveWindowTabToNewWindow` | TB-1.25, TB-3.15 |
| `workbench.action.navigateBack` | TB-2.4, ED-1.32, AC-7.4, AC-7.12, WB-9.6 |
| `workbench.action.navigateBackInEditLocations` | ED-1.32 |
| `workbench.action.navigateBackInNavigationLocations` | ED-1.32 |
| `workbench.action.navigateDown` | WB-3.3 |
| `workbench.action.navigateEditorGroups` | ED-1.22 |
| `workbench.action.navigateForward` | TB-2.4, ED-1.32, WB-9.6 |
| `workbench.action.navigateForwardInEditLocations` | ED-1.32 |
| `workbench.action.navigateForwardInNavigationLocations` | ED-1.32 |
| `workbench.action.navigateLast` | ED-1.32 |
| `workbench.action.navigateLeft` | WB-3.3 |
| `workbench.action.navigateModalEditorNext` | ED-1.36 |
| `workbench.action.navigateModalEditorPrevious` | ED-1.36 |
| `workbench.action.navigatePreviousInEditLocations` | ED-1.32 |
| `workbench.action.navigatePreviousInNavigationLocations` | ED-1.32 |
| `workbench.action.navigateRight` | WB-3.3 |
| `workbench.action.navigateToLastEditLocation` | ED-1.32 |
| `workbench.action.navigateToLastNavigationLocation` | ED-1.32 |
| `workbench.action.navigateUp` | WB-3.3 |
| `workbench.action.newBrowserTab` | ED-10.29 |
| `workbench.action.newCursorRule` | ED-8.65 |
| `workbench.action.newEmptyEditorWindow` | ED-1.25, ED-1.33 |
| `workbench.action.newGroupAbove` | ED-1.17 |
| `workbench.action.newGroupBelow` | ED-1.17 |
| `workbench.action.newGroupLeft` | ED-1.17 |
| `workbench.action.newGroupRight` | ED-1.17 |
| `workbench.action.newProjectCursorCommand` | ED-8.65 |
| `workbench.action.newUserCursorCommand` | ED-8.65 |
| `workbench.action.newWindow` | TB-1.27, ED-9.20 |
| `workbench.action.newWindowTab` | TB-1.25, TB-3.15 |
| `workbench.action.nextAuxiliaryBarView` | AX-4.9 |
| `workbench.action.nextEditor` | ED-1.29 |
| `workbench.action.nextEditorInGroup` | ED-1.29 |
| `workbench.action.nextPanelView` | PN-1.15 |
| `workbench.action.nextSideBarView` | AB-1.10 |
| `workbench.action.openAccessibilitySettings` | ED-8.1 |
| `workbench.action.openActiveLogOutputFile` | PN-5.10 |
| `workbench.action.openActiveLogOutputFileInNewWindow` | PN-5.10 |
| `workbench.action.openAgentLayoutQuickMenu` | TB-5.6 |
| `workbench.action.openAgentSessionsWelcome` | ED-9.14 |
| `workbench.action.openApplicationSettingsJson` | ED-8.8 |
| `workbench.action.openBrowserEditor` | SB-2.38, ED-10.27, PN-3.46 |
| `workbench.action.openCustomizeEditor` | ED-8.28, ED-8.33, ED-8.62 |
| `workbench.action.openDefaultKeybindingsFile` | ED-8.8, ED-8.54 |
| `workbench.action.openDocumentationUrl` | TB-1.30 |
| `workbench.action.openEditorAtIndex1` | ED-1.29 |
| `workbench.action.openExtensionLogsFolder` | PN-5.15 |
| `workbench.action.openFolderSettings` | ED-8.3 |
| `workbench.action.openFolderSettingsFile` | ED-8.8 |
| `workbench.action.openGlassProcessExplorer` | WB-6.2 |
| `workbench.action.openGlobalKeybindings` | AB-2.9, AB-4.16, AB-6.11, ED-8.9, ED-8.20, ED-8.54, WB-4.1 |
| `workbench.action.openGlobalKeybindingsFile` | ED-8.8, ED-8.54 |
| `workbench.action.openGlobalSettings` | ED-8.1, ED-8.3, ED-8.20 |
| `workbench.action.openGlobalSettingsCursorAlias` | ED-8.1 |
| `workbench.action.openImagesInCarousel` | SB-2.40, ED-10.44 |
| `workbench.action.openIssueReporter` | QA-4.19, WB-6.5, WB-6.6 |
| `workbench.action.openKeyboardLayoutPicker` | SS-3.38 |
| `workbench.action.openLargeFile` | DL-4.15 |
| `workbench.action.openLargeOutput` | ED-7.22 |
| `workbench.action.openLayoutSwitcher` | TB-4.11, TB-4.12, TB-4.13, TB-4.14 |
| `workbench.action.openLicenseUrl` | WB-6.1 |
| `workbench.action.openLogFile` | PN-5.15 |
| `workbench.action.openLogsFolder` | PN-5.15 |
| `workbench.action.openNewsletterSignupUrl` | TB-1.30 |
| `workbench.action.openNextRecentlyUsedEditor` | ED-1.30 |
| `workbench.action.openNextRecentlyUsedEditorInGroup` | ED-1.30 |
| `workbench.action.openPreviousEditorFromHistory` | ED-1.30 |
| `workbench.action.openPreviousRecentlyUsedEditor` | ED-1.30 |
| `workbench.action.openPreviousRecentlyUsedEditorInGroup` | ED-1.30 |
| `workbench.action.openPrivacyStatementUrl` | TB-1.30 |
| `workbench.action.openProcessExplorer` | WB-6.2, WB-6.7 |
| `workbench.action.openRawDefaultSettings` | ED-8.8, ED-8.53 |
| `workbench.action.openRecent` | TB-1.14, SB-2.36, ED-9.5, ED-9.18, DL-2.14, WB-9.7 |
| `workbench.action.openRemoteSettings` | ED-8.3 |
| `workbench.action.openRemoteSettingsFile` | ED-8.8 |
| `workbench.action.openRequestFeatureUrl` | TB-1.30 |
| `workbench.action.openSettings` | AB-4.15, ED-1.3, ED-8.1, ED-8.21, ED-9.18, AC-2.18, SS-3.31, SS-3.50, SS-3.51, SS-3.52, SS-4.15, SS-5.7, SS-5.13, SS-6.4 |
| `workbench.action.openSettings2` | ED-8.1 |
| `workbench.action.openSettingsJson` | ED-8.8, SS-3.49 |
| `workbench.action.openSnippets` | AB-4.18, AC-4.17, AC-4.18 |
| `workbench.action.openTipsAndTricksUrl` | TB-1.30 |
| `workbench.action.openVideoTutorialsUrl` | TB-1.30 |
| `workbench.action.openView` | QA-4.1, QA-5.6 |
| `workbench.action.openWalkthrough` | TB-1.30, ED-9.1, ED-9.2, ED-9.6, ED-9.10, ED-9.12 |
| `workbench.action.openWorkspace` | TB-1.27, DL-3.4 |
| `workbench.action.openWorkspaceConfigFile` | WB-9.2 |
| `workbench.action.openWorkspaceFromEditor` | ED-5.49, WB-9.5 |
| `workbench.action.openWorkspaceInNewWindow` | DL-3.4 |
| `workbench.action.openWorkspaceSettings` | ED-8.3 |
| `workbench.action.openWorkspaceSettingsFile` | ED-8.8, SS-3.49 |
| `workbench.action.openYouTubeUrl` | TB-1.30 |
| `workbench.action.output.activeOutputLogLevel.<level>` | PN-5.4 |
| `workbench.action.output.activeOutputLogLevelDefault` | PN-5.4 |
| `workbench.action.output.addCompoundLog` | PN-5.14 |
| `workbench.action.output.remove` | PN-5.14 |
| `workbench.action.output.show.<channel` | PN-5.2, PN-5.18, PN-5.20 |
| `workbench.action.output.show.{channelId}` | SS-5.14 |
| `workbench.action.output.toggleOutput` | PN-5.1 |
| `workbench.action.pinEditor` | ED-1.7 |
| `workbench.action.populateFileFromSnippet` | AC-4.14, AC-4.16 |
| `workbench.action.positionPanelBottom` | PN-1.3 |
| `workbench.action.positionPanelLeft` | PN-1.3 |
| `workbench.action.positionPanelRight` | PN-1.3 |
| `workbench.action.positionPanelTop` | PN-1.3 |
| `workbench.action.previousAuxiliaryBarView` | AX-4.9 |
| `workbench.action.previousEditor` | ED-1.29 |
| `workbench.action.previousEditorInGroup` | ED-1.29 |
| `workbench.action.previousPanelView` | PN-1.15 |
| `workbench.action.previousSideBarView` | AB-1.10 |
| `workbench.action.problems.focus` | PN-4.1 |
| `workbench.action.quickchat.toggle` | QA-1.19, QA-5.3 |
| `workbench.action.quickInputBack` | QA-6.8 |
| `workbench.action.quickOpen` | ED-9.18, QA-1.3, QA-2.1, QA-2.2, QA-2.4, QA-2.5, QA-2.6, QA-2.7, QA-2.9, QA-2.11, QA-2.20, QA-2.21, QA-2.22, QA-2.23, QA-5.1, QA-5.3, QA-5.5, QA-5.6, QA-5.10 |
| `workbench.action.quickOpenLeastRecentlyUsedEditor` | ED-1.30, QA-4.4 |
| `workbench.action.quickOpenLeastRecentlyUsedEditorInGroup` | ED-1.30, QA-4.2 |
| `workbench.action.quickOpenNavigateNext` | QA-5.12, QA-5.13 |
| `workbench.action.quickOpenNavigateNextInEditorPicker` | ED-1.30, QA-4.2, QA-5.11 |
| `workbench.action.quickOpenNavigateNextInFilePicker` | QA-5.11 |
| `workbench.action.quickOpenNavigateNextInRecentFilesPicker` | QA-5.11 |
| `workbench.action.quickOpenNavigateNextInTerminalPicker` | QA-4.6, QA-5.11 |
| `workbench.action.quickOpenNavigateNextInViewPicker` | QA-5.11 |
| `workbench.action.quickOpenNavigatePrevious` | QA-5.12, QA-5.13 |
| `workbench.action.quickOpenNavigatePreviousInEditorPicker` | ED-1.30, QA-4.2, QA-5.11 |
| `workbench.action.quickOpenNavigatePreviousInFilePicker` | QA-5.11 |
| `workbench.action.quickOpenNavigatePreviousInRecentFilesPicker` | QA-5.11 |
| `workbench.action.quickOpenNavigatePreviousInTerminalPicker` | QA-4.6, QA-5.11 |
| `workbench.action.quickOpenNavigatePreviousInViewPicker` | QA-5.11 |
| `workbench.action.quickOpenPreviousEditor` | QA-2.18 |
| `workbench.action.quickOpenPreviousRecentlyUsedEditor` | ED-1.30, QA-4.4 |
| `workbench.action.quickOpenPreviousRecentlyUsedEditorInGroup` | ED-1.30, QA-4.2 |
| `workbench.action.quickOpenRecent` | DL-2.14, WB-9.7 |
| `workbench.action.quickOpenSelectNext` | QA-5.14 |
| `workbench.action.quickOpenSelectPrevious` | QA-5.14 |
| `workbench.action.quickOpenTerm` | QA-4.6, QA-5.6 |
| `workbench.action.quickOpenView` | QA-4.1 |
| `workbench.action.quickOpenWithModes` | TB-2.1, TB-2.9, QA-5.3 |
| `workbench.action.quickPickManyToggle` | QA-6.2 |
| `workbench.action.quickSwitchWindow` | WB-9.8 |
| `workbench.action.quickTextSearch` | QA-2.14, QA-2.15, QA-2.16, QA-5.3, QA-5.6 |
| `workbench.action.quit` | TB-1.24, DL-2.5, DL-2.10, DL-2.11, DL-2.17 |
| `workbench.action.reloadBrowserTab` | ED-10.33 |
| `workbench.action.reloadWindow` | SS-4.4, DL-2.5 |
| `workbench.action.reloadWindowWithExtensionsDisabled` | WB-8.15 |
| `workbench.action.reloadWindowWithHMR` | WB-8.13 |
| `workbench.action.remote.close` | SS-4.4 |
| `workbench.action.remote.showMenu` | ED-9.4, SS-4.3 |
| `workbench.action.remote.showWebStartEntryActions` | ED-9.4 |
| `workbench.action.removeLargeStorageDatabaseEntries` | WB-8.4 |
| `workbench.action.removeListItem` | ED-8.47 |
| `workbench.action.removeRootFolder` | SB-2.19 |
| `workbench.action.reopenClosedEditor` | TB-1.14, ED-1.10 |
| `workbench.action.reopenTextEditor` | ED-1.27 |
| `workbench.action.reopenWithEditor` | SB-2.32, ED-1.27, ED-1.43 |
| `workbench.action.replaceCommandLine` | WB-9.3 |
| `workbench.action.replaceInFiles` | SB-3.6 |
| `workbench.action.reportAIAction` | QA-6.26 |
| `workbench.action.reportCppAction` | AC-8.19, QA-6.20 |
| `workbench.action.reportPerformanceIssueUsingReporter` | WB-6.5, WB-6.6 |
| `workbench.action.resetFocusedViewLocation` | SB-1.17 |
| `workbench.action.resetListItem` | ED-8.47 |
| `workbench.action.resetViewLocations` | SB-1.16 |
| `workbench.action.restartInTemporaryProfile` | WB-8.13 |
| `workbench.action.restoreAuxiliaryBar` | AX-3.2 |
| `workbench.action.restoreCommandLine` | WB-9.3 |
| `workbench.action.restoreEditorsToMainWindow` | ED-1.33 |
| `workbench.action.revealPathInFinder<n>` | TB-2.7 |
| `workbench.action.revealUserDataFolder` | WB-8.16 |
| `workbench.action.revertAndCloseActiveEditor` | ED-1.8 |
| `workbench.action.saveActiveLogOutputAs` | PN-5.11 |
| `workbench.action.saveWorkspaceAs` | DL-3.9 |
| `workbench.action.search.toggleQueryDetails` | SB-3.10 |
| `workbench.action.selectIconTheme` | AB-4.20, ED-8.16 |
| `workbench.action.selectProductIconTheme` | AB-4.20, ED-8.17 |
| `workbench.action.selectTheme` | AB-4.20, ED-8.15, SS-2.15 |
| `workbench.action.sessionRecording.flagAndAddContext` | SS-6.14 |
| `workbench.action.sessionRecording.reportLag` | SS-6.14 |
| `workbench.action.sessionRecording.reportMenu` | SS-6.14 |
| `workbench.action.setDefaultLogLevel` | PN-5.4 |
| `workbench.action.setLayoutDensity.compact` | TB-4.20 |
| `workbench.action.setLayoutDensity.default` | TB-4.20 |
| `workbench.action.setLogLevel` | PN-5.4 |
| `workbench.action.setNotificationsPosition.bottomLeft` | NT-3.11 |
| `workbench.action.setNotificationsPosition.bottomRight` | NT-3.11 |
| `workbench.action.setNotificationsPosition.topRight` | NT-3.11 |
| `workbench.action.share` | TB-2.16 |
| `workbench.action.showAboutDialog` | TB-1.24, DL-1.17, DL-1.18, DL-1.20 |
| `workbench.action.showAllEditors` | ED-1.31, QA-4.3, QA-5.6 |
| `workbench.action.showAllEditorsByMostRecentlyUsed` | ED-1.31, QA-4.4, QA-5.6 |
| `workbench.action.showAllSymbols` | QA-2.12, QA-5.6 |
| `workbench.action.showCommands` | AB-4.14, ED-9.18, QA-1.1, QA-1.2, QA-1.3, QA-1.4, QA-1.5, QA-1.6, QA-1.7, QA-1.11, QA-1.14, QA-1.16, QA-1.17, QA-1.18, QA-5.3, QA-5.6, QA-5.10 |
| `workbench.action.showContentTracing` | WB-8.18 |
| `workbench.action.showEditorActions` | ED-1.1 |
| `workbench.action.showEditorsInActiveGroup` | ED-1.31, QA-4.2, QA-5.6 |
| `workbench.action.showEditorsInGroup` | ED-1.31, QA-4.2 |
| `workbench.action.showEditorTab` | ED-1.1 |
| `workbench.action.showEmmetCommands` | ED-2.32 |
| `workbench.action.showErrorsWarnings` | PN-4.1 |
| `workbench.action.showGPUInfo` | WB-8.17 |
| `workbench.action.showHover` | WB-3.5 |
| `workbench.action.showInteractivePlayground` | TB-1.30, ED-9.13 |
| `workbench.action.showLogs` | PN-5.3 |
| `workbench.action.showMultipleEditorTabs` | ED-1.1 |
| `workbench.action.showNextWindowTab` | TB-1.25, TB-3.15 |
| `workbench.action.showOutputChannels` | PN-5.3 |
| `workbench.action.showPolicyDiagnostics` | WB-8.5 |
| `workbench.action.showPreviousWindowTab` | TB-1.25, TB-3.15 |
| `workbench.action.showWindowLog` | ED-8.57, ED-10.47, PN-5.3 |
| `workbench.action.splitEditor` | ED-1.13 |
| `workbench.action.splitEditorDown` | ED-1.13, ED-1.25, ED-10.90 |
| `workbench.action.splitEditorInGroup` | ED-1.24, ED-10.90 |
| `workbench.action.splitEditorLeft` | ED-1.13, ED-1.25, ED-10.90 |
| `workbench.action.splitEditorOrthogonal` | ED-1.13 |
| `workbench.action.splitEditorRight` | ED-1.13, ED-1.25, ED-10.90 |
| `workbench.action.splitEditorToAboveGroup` | ED-1.14 |
| `workbench.action.splitEditorToBelowGroup` | ED-1.14 |
| `workbench.action.splitEditorToFirstGroup` | ED-1.14 |
| `workbench.action.splitEditorToLastGroup` | ED-1.14 |
| `workbench.action.splitEditorToLeftGroup` | ED-1.14 |
| `workbench.action.splitEditorToNextGroup` | ED-1.14 |
| `workbench.action.splitEditorToPreviousGroup` | ED-1.14 |
| `workbench.action.splitEditorToRightGroup` | ED-1.14 |
| `workbench.action.splitEditorUp` | ED-1.13, ED-1.25, ED-10.90 |
| `workbench.action.startHeapTracing` | SS-3.36 |
| `workbench.action.startTracing` | SS-3.36 |
| `workbench.action.stopTracing` | SS-3.36 |
| `workbench.action.switchToMainWindow` | WB-9.4 |
| `workbench.action.switchWindow` | TB-1.25, WB-9.8 |
| `workbench.action.syncAccountPolicy` | WB-8.6 |
| `workbench.action.tasks.build` | QA-4.11, WB-2.5 |
| `workbench.action.tasks.configureDefaultBuildTask` | QA-4.14 |
| `workbench.action.tasks.configureDefaultTestTask` | QA-4.14 |
| `workbench.action.tasks.configureTaskRunner` | PN-2.4, QA-4.14 |
| `workbench.action.tasks.manageAutomaticRunning` | WB-2.1 |
| `workbench.action.tasks.openUserTasks` | AB-4.19 |
| `workbench.action.tasks.openWorkspaceFileTasks` | WB-2.2 |
| `workbench.action.tasks.rerunAllRunningTasks` | WB-2.7 |
| `workbench.action.tasks.rerunForActiveTerminal` | PN-2.37 |
| `workbench.action.tasks.reRunTask` | WB-2.3 |
| `workbench.action.tasks.restartTask` | QA-4.13 |
| `workbench.action.tasks.runTask` | PN-2.4, QA-4.8, WB-2.5 |
| `workbench.action.tasks.showLog` | WB-2.4 |
| `workbench.action.tasks.showTasks` | SS-5.11, SS-5.12, QA-4.12 |
| `workbench.action.tasks.terminate` | QA-4.13 |
| `workbench.action.tasks.test` | QA-4.11, WB-2.5 |
| `workbench.action.tasks.toggleProblems` | PN-4.1 |
| `workbench.action.terminal.acceptSelectedSuggestion` | AC-5.8, AC-5.9 |
| `workbench.action.terminal.acceptSelectedSuggestionEnter` | AC-5.9 |
| `workbench.action.terminal.accessibleBufferGoToNextCommand` | PN-3.22 |
| `workbench.action.terminal.accessibleBufferGoToPreviousCommand` | PN-3.22 |
| `workbench.action.terminal.attachToSession` | PN-2.34 |
| `workbench.action.terminal.changeColor` | PN-2.12 |
| `workbench.action.terminal.changeColorActiveTab` | PN-2.12 |
| `workbench.action.terminal.changeIcon` | PN-2.12 |
| `workbench.action.terminal.changeIconActiveTab` | PN-2.12 |
| `workbench.action.terminal.changeSelectionModeAlways` | AC-5.10 |
| `workbench.action.terminal.changeSelectionModeNever` | AC-5.10 |
| `workbench.action.terminal.changeSelectionModePartial` | AC-5.10 |
| `workbench.action.terminal.clear` | PN-2.17, PN-3.18, PN-3.34 |
| `workbench.action.terminal.clearPreviousSessionHistory` | PN-3.12 |
| `workbench.action.terminal.clearSelection` | PN-3.19 |
| `workbench.action.terminal.configureSuggestSettings` | AC-5.17 |
| `workbench.action.terminal.copyAndClearSelection` | PN-3.15 |
| `workbench.action.terminal.copyLastCommand` | PN-3.7 |
| `workbench.action.terminal.copyLastCommandAndLastCommandOutput` | PN-3.7 |
| `workbench.action.terminal.copyLastCommandOutput` | PN-3.7 |
| `workbench.action.terminal.copySelection` | PN-3.15, PN-3.18 |
| `workbench.action.terminal.copySelectionAsHtml` | PN-3.15, PN-3.18 |
| `workbench.action.terminal.detachSession` | PN-2.34 |
| `workbench.action.terminal.doNotShowSuggestOnType` | AC-5.16 |
| `workbench.action.terminal.findNext` | PN-3.1 |
| `workbench.action.terminal.findPrevious` | PN-3.1 |
| `workbench.action.terminal.focus` | PN-2.1, PN-2.17 |
| `workbench.action.terminal.focusAccessibleBuffer` | PN-3.22 |
| `workbench.action.terminal.focusAtIndex1` | PN-2.20 |
| `workbench.action.terminal.focusAtIndex2` | PN-2.20 |
| `workbench.action.terminal.focusAtIndex3` | PN-2.20 |
| `workbench.action.terminal.focusAtIndex4` | PN-2.20 |
| `workbench.action.terminal.focusAtIndex5` | PN-2.20 |
| `workbench.action.terminal.focusAtIndex6` | PN-2.20 |
| `workbench.action.terminal.focusAtIndex7` | PN-2.20 |
| `workbench.action.terminal.focusAtIndex8` | PN-2.20 |
| `workbench.action.terminal.focusAtIndex9` | PN-2.20 |
| `workbench.action.terminal.focusFind` | PN-3.1 |
| `workbench.action.terminal.focusHover` | PN-2.14 |
| `workbench.action.terminal.focusNext` | PN-2.20 |
| `workbench.action.terminal.focusNextPane` | PN-2.22 |
| `workbench.action.terminal.focusPrevious` | PN-2.20 |
| `workbench.action.terminal.focusPreviousPane` | PN-2.22 |
| `workbench.action.terminal.focusTabs` | PN-2.18 |
| `workbench.action.terminal.fontZoomIn` | PN-3.30 |
| `workbench.action.terminal.fontZoomOut` | PN-3.30 |
| `workbench.action.terminal.fontZoomReset` | PN-3.30 |
| `workbench.action.terminal.goToRecentDirectory` | PN-3.13 |
| `workbench.action.terminal.hideFind` | PN-3.1 |
| `workbench.action.terminal.hideSuggestWidget` | AC-5.12 |
| `workbench.action.terminal.hideSuggestWidgetAndNavigateHistory` | AC-5.7 |
| `workbench.action.terminal.join` | PN-2.24 |
| `workbench.action.terminal.joinActiveTab` | PN-2.24 |
| `workbench.action.terminal.kill` | PN-2.17, PN-2.28 |
| `workbench.action.terminal.killActiveTab` | PN-2.15, PN-2.28 |
| `workbench.action.terminal.killAll` | PN-2.28 |
| `workbench.action.terminal.killEditor` | PN-2.27 |
| `workbench.action.terminal.killViewOrEditor` | PN-2.28, PN-3.18 |
| `workbench.action.terminal.launchHelp` | PN-2.32 |
| `workbench.action.terminal.learnMore` | PN-2.13, PN-2.38 |
| `workbench.action.terminal.moveIntoNewWindow` | PN-2.25 |
| `workbench.action.terminal.moveToEditor` | PN-2.25 |
| `workbench.action.terminal.moveToTerminalPanel` | PN-2.26 |
| `workbench.action.terminal.new` | PN-2.2, QA-4.6 |
| `workbench.action.terminal.newInActiveWorkspace` | PN-2.2 |
| `workbench.action.terminal.newInNewWindow` | PN-2.6 |
| `workbench.action.terminal.newLocal` | PN-2.2 |
| `workbench.action.terminal.newWithCwd` | PN-2.8 |
| `workbench.action.terminal.newWithProfile` | PN-2.3, PN-2.4, QA-4.6 |
| `workbench.action.terminal.openDetectedLink` | PN-3.5 |
| `workbench.action.terminal.openFileLink` | PN-3.5 |
| `workbench.action.terminal.openNativeConsole` | PN-2.41 |
| `workbench.action.terminal.openSettings` | PN-2.4 |
| `workbench.action.terminal.openUrlLink` | PN-3.5 |
| `workbench.action.terminal.paste` | PN-3.16, PN-3.18 |
| `workbench.action.terminal.pastePwsh` | PN-3.16 |
| `workbench.action.terminal.pasteSelection` | PN-3.16 |
| `workbench.action.terminal.recordSession` | PN-3.40, SS-5.16 |
| `workbench.action.terminal.relaunch` | PN-2.13, PN-2.32, PN-3.37 |
| `workbench.action.terminal.rename` | PN-2.11, QA-4.7 |
| `workbench.action.terminal.renameActiveTab` | PN-2.11 |
| `workbench.action.terminal.renameWithArg` | PN-2.11 |
| `workbench.action.terminal.resetDiscoverability` | AC-5.18 |
| `workbench.action.terminal.resetSuggestWidgetSize` | AC-5.3 |
| `workbench.action.terminal.resizePaneDown` | PN-2.23 |
| `workbench.action.terminal.resizePaneLeft` | PN-2.23 |
| `workbench.action.terminal.resizePaneRight` | PN-2.23 |
| `workbench.action.terminal.resizePaneUp` | PN-2.23 |
| `workbench.action.terminal.restartPtyHost` | PN-3.40, SS-5.15 |
| `workbench.action.terminal.runActiveFile` | PN-2.10 |
| `workbench.action.terminal.runRecentCommand` | PN-3.12 |
| `workbench.action.terminal.runSelectedText` | PN-2.10 |
| `workbench.action.terminal.scrollDown` | PN-3.33 |
| `workbench.action.terminal.scrollDownPage` | PN-3.33 |
| `workbench.action.terminal.scrollToBottom` | PN-3.33 |
| `workbench.action.terminal.scrollToBottomAccessibleView` | PN-3.22 |
| `workbench.action.terminal.scrollToNextCommand` | PN-3.8 |
| `workbench.action.terminal.scrollToPreviousCommand` | PN-3.8 |
| `workbench.action.terminal.scrollToTop` | PN-3.33 |
| `workbench.action.terminal.scrollToTopAccessibleView` | PN-3.22 |
| `workbench.action.terminal.scrollUp` | PN-3.33 |
| `workbench.action.terminal.scrollUpPage` | PN-3.33 |
| `workbench.action.terminal.searchWorkspace` | PN-3.2 |
| `workbench.action.terminal.selectAll` | PN-3.18, PN-3.19 |
| `workbench.action.terminal.selectDefaultShell` | PN-2.4, PN-2.5 |
| `workbench.action.terminal.selectNextPageSuggestion` | AC-5.7 |
| `workbench.action.terminal.selectNextSuggestion` | AC-5.7 |
| `workbench.action.terminal.selectPrevPageSuggestion` | AC-5.7 |
| `workbench.action.terminal.selectPrevSuggestion` | AC-5.7 |
| `workbench.action.terminal.selectToNextCommand` | PN-3.9 |
| `workbench.action.terminal.selectToNextLine` | PN-3.9 |
| `workbench.action.terminal.selectToPreviousCommand` | PN-3.9 |
| `workbench.action.terminal.selectToPreviousLine` | PN-3.9 |
| `workbench.action.terminal.sendSequence` | PN-3.35, PN-3.36 |
| `workbench.action.terminal.sendSignal` | PN-3.35 |
| `workbench.action.terminal.setDimensions` | PN-2.40 |
| `workbench.action.terminal.showEnvironmentContributions` | PN-2.13, PN-3.37 |
| `workbench.action.terminal.showQuickFixes` | PN-3.14 |
| `workbench.action.terminal.showSuggestOnType` | AC-5.16 |
| `workbench.action.terminal.showTextureAtlas` | PN-3.40 |
| `workbench.action.terminal.sizeToContentWidth` | PN-2.40, PN-3.18 |
| `workbench.action.terminal.split` | PN-2.4, PN-2.17, PN-2.21 |
| `workbench.action.terminal.splitActiveTab` | PN-2.15, PN-2.21 |
| `workbench.action.terminal.splitInActiveWorkspace` | PN-2.21 |
| `workbench.action.terminal.suggestLearnMore` | AC-5.17 |
| `workbench.action.terminal.suggestToggleDetails` | AC-5.13 |
| `workbench.action.terminal.suggestToggleDetailsFocus` | AC-5.13 |
| `workbench.action.terminal.switchTerminal` | PN-2.15, PN-2.17 |
| `workbench.action.terminal.toggleFindCaseSensitive` | PN-3.1 |
| `workbench.action.terminal.toggleFindRegex` | PN-3.1 |
| `workbench.action.terminal.toggleFindWholeWord` | PN-3.1 |
| `workbench.action.terminal.toggleStickyScroll` | PN-3.11 |
| `workbench.action.terminal.toggleTerminal` | ED-9.18, PN-2.1 |
| `workbench.action.terminal.triggerSuggest` | AC-5.5 |
| `workbench.action.terminal.unsplit` | PN-2.24 |
| `workbench.action.terminal.writeDataToTerminal` | PN-3.40 |
| `workbench.action.toggleActivityBarVisibility` | AB-5.5 |
| `workbench.action.toggleAutoSave` | WB-1.6 |
| `workbench.action.toggleAuxiliaryBar` | TB-4.5, TB-4.6, TB-4.12, AX-1.1, AX-1.2, AX-4.5 |
| `workbench.action.toggleCenteredLayout` | ED-1.35 |
| `workbench.action.toggleCommenting` | ED-12.7 |
| `workbench.action.toggleCompactAuxiliaryWindow` | TB-3.21 |
| `workbench.action.toggleConfirmBeforeClose` | DL-2.10 |
| `workbench.action.toggleDevTools` | WB-8.14 |
| `workbench.action.toggleEditorGroupLayout` | SB-2.31 |
| `workbench.action.toggleEditorGroupLock` | ED-1.23, ED-1.25 |
| `workbench.action.toggleEditorType` | ED-1.27 |
| `workbench.action.toggleEditorVisibility` | PN-1.7, PN-1.20 |
| `workbench.action.toggleEditorWidths` | ED-1.20 |
| `workbench.action.toggleFullScreen` | TB-3.13 |
| `workbench.action.toggleInlineDiffs` | ED-6.28 |
| `workbench.action.toggleKeepEditors` | ED-1.5 |
| `workbench.action.toggleKeybindingsLog` | ED-8.57 |
| `workbench.action.toggleLightDarkThemes` | ED-8.15 |
| `workbench.action.toggleLockedScrolling` | SS-3.27 |
| `workbench.action.toggleMaximizedAuxiliaryBar` | AX-3.1, AX-3.3 |
| `workbench.action.toggleMaximizedPanel` | PN-1.5, PN-1.17 |
| `workbench.action.toggleMaximizeEditorGroup` | ED-1.19 |
| `workbench.action.toggleMenuBar` | TB-1.4 |
| `workbench.action.toggleModalEditorMaximized` | ED-1.36 |
| `workbench.action.toggleModalEditorSidebar` | ED-1.36 |
| `workbench.action.toggleMultiCursorModifier` | ED-2.2 |
| `workbench.action.togglePanel` | TB-4.4, TB-4.6, TB-4.12, ED-9.18, PN-1.1, PN-1.2, PN-1.17, PN-1.20 |
| `workbench.action.toggleScreencastMode` | WB-8.7 |
| `workbench.action.toggleSearchOnType` | SB-3.2 |
| `workbench.action.toggleSeparatePinnedEditorTabs` | ED-1.7 |
| `workbench.action.toggleSidebarPosition` | TB-4.9, AB-6.12, SB-1.2, SB-1.6, AX-2.1, AX-2.2, AX-4.5 |
| `workbench.action.toggleSidebarVisibility` | TB-4.3, TB-4.6, TB-4.12, AB-6.10, SB-1.1, SB-1.6 |
| `workbench.action.toggleSplitEditorInGroup` | ED-1.24 |
| `workbench.action.toggleSplitEditorInGroupLayout` | ED-1.24 |
| `workbench.action.toggleStatusbarVisibility` | SS-1.1 |
| `workbench.action.toggleWindowAlwaysOnTop` | TB-3.20, TB-4.19 |
| `workbench.action.toggleWindowTabsBar` | TB-3.15 |
| `workbench.action.toggleZenMode` | AB-5.8, ED-1.34, NT-4.10 |
| `workbench.action.troubleshootIssue.start` | WB-6.3 |
| `workbench.action.troubleshootIssue.stop` | WB-6.3 |
| `workbench.action.tryNewDefaultThemes` | ED-8.15 |
| `workbench.action.uninstallCommandLine` | WB-9.3 |
| `workbench.action.unlockEditorGroup` | ED-1.23 |
| `workbench.action.unpinActiveEditor` | SB-2.30 |
| `workbench.action.unpinEditor` | ED-1.4, ED-1.7 |
| `workbench.action.url.openUrl` | WB-8.25 |
| `workbench.action.webview.openDeveloperTools` | ED-10.42 |
| `workbench.action.webview.reloadCssAction` | ED-10.74 |
| `workbench.action.webview.reloadWebviewAction` | ED-10.41 |
| `workbench.action.welcomeOnboarding2026` | ED-9.15 |
| `workbench.action.zenHideEditorTabs` | ED-1.34 |
| `workbench.action.zenShowEditorTab` | ED-1.34 |
| `workbench.action.zenShowMultipleEditorTabs` | ED-1.34 |
| `workbench.action.zoomIn` | TB-3.16, SS-5.13 |
| `workbench.action.zoomOut` | TB-3.16, SS-5.13 |
| `workbench.action.zoomReset` | TB-3.16, SS-5.13 |
| `workbench.actions.accounts` | AB-4.1, AB-4.24 |
| `workbench.actions.manage` | AB-4.12, AB-4.24 |
| `workbench.actions.menubar.focus` | TB-1.7 |
| `workbench.actions.problems.quickfix` | PN-4.13 |
| `workbench.actions.table.workbench.panel.markers.view.viewAsTable` | PN-4.3, PN-4.4 |
| `workbench.actions.table.workbench.panel.markers.view.viewAsTree` | PN-4.2, PN-4.4 |
| `workbench.actions.treeView.testExplorer.filter` | SB-6.4 |
| `workbench.actions.treeView.workbench.panel.markers.view.collapseAll` | PN-4.2 |
| `workbench.actions.updateIndicator` | TB-5.1 |
| `workbench.actions.view.problems` | PN-4.1 |
| `workbench.actions.view.toggleProblems` | PN-4.1, SS-5.6 |
| `workbench.actions.workbench.panel.comments.toggleResolvedComments` | PN-7.4 |
| `workbench.actions.workbench.panel.comments.toggleSortByResource` | PN-7.5 |
| `workbench.actions.workbench.panel.comments.toggleSortByUpdatedAt` | PN-7.5 |
| `workbench.actions.workbench.panel.comments.toggleUnResolvedComments` | PN-7.4 |
| `workbench.actions.workbench.panel.markers.view.toggleActiveFile` | PN-4.8 |
| `workbench.actions.workbench.panel.markers.view.toggleErrors` | PN-4.7 |
| `workbench.actions.workbench.panel.markers.view.toggleExcludedFiles` | PN-4.8 |
| `workbench.actions.workbench.panel.markers.view.toggleInfos` | PN-4.7 |
| `workbench.actions.workbench.panel.markers.view.toggleWarnings` | PN-4.7 |
| `workbench.actions.workbench.panel.output.clearFilterText` | PN-5.5 |
| `workbench.actions.workbench.panel.output.toggle.<channel` | PN-5.7 |
| `workbench.actions.workbench.panel.output.toggle.debug` | PN-5.6 |
| `workbench.actions.workbench.panel.output.toggle.error` | PN-5.6 |
| `workbench.actions.workbench.panel.output.toggle.info` | PN-5.6 |
| `workbench.actions.workbench.panel.output.toggle.trace` | PN-5.6 |
| `workbench.actions.workbench.panel.output.toggle.warning` | PN-5.6 |
| `workbench.banner.focusBanner` | BN-1.7 |
| `workbench.banner.focusNextAction` | BN-1.7 |
| `workbench.banner.focusPreviousAction` | BN-1.7 |
| `workbench.debug.action.copyAll` | PN-6.15 |
| `workbench.debug.action.focusBreakpointsView` | SB-5.23 |
| `workbench.debug.action.focusCallStackView` | SB-5.19 |
| `workbench.debug.action.focusRepl` | PN-6.1 |
| `workbench.debug.action.focusVariablesView` | SB-5.11 |
| `workbench.debug.action.focusWatchView` | SB-5.17 |
| `workbench.debug.action.toggleRepl` | PN-6.1 |
| `workbench.debug.panel.action.clearReplAction` | PN-6.14 |
| `workbench.debug.viewlet.action.addDataBreakpointOnAddress` | SB-5.28 |
| `workbench.debug.viewlet.action.addFunctionBreakpointAction` | SB-5.27 |
| `workbench.debug.viewlet.action.addWatchExpression` | SB-5.17 |
| `workbench.debug.viewlet.action.copyAllWatchExpressions` | SB-5.13 |
| `workbench.debug.viewlet.action.copyValue` | SB-5.13 |
| `workbench.debug.viewlet.action.copyWorkspaceVariableValue` | SB-5.16 |
| `workbench.debug.viewlet.action.disableAllBreakpoints` | SB-5.24 |
| `workbench.debug.viewlet.action.editDataBreakpointOnAddress` | SB-5.28 |
| `workbench.debug.viewlet.action.enableAllBreakpoints` | SB-5.24 |
| `workbench.debug.viewlet.action.reapplyBreakpointsAction` | SB-5.24 |
| `workbench.debug.viewlet.action.removeAllBreakpoints` | SB-5.25 |
| `workbench.debug.viewlet.action.removeAllWatchExpressions` | SB-5.18 |
| `workbench.debug.viewlet.action.removeBreakpoint` | SB-5.25 |
| `workbench.debug.viewlet.action.toggleBreakpointsActivatedAction` | SB-5.24 |
| `workbench.debug.viewlet.action.toggleBreakpointsPresentation` | SB-5.30 |
| `workbench.debug.viewlet.action.viewMemory` | SB-5.15 |
| `workbench.debug.watchExpressionsView` | SB-5.17 |
| `workbench.dialog.choice.<n>` | NT-2.9 |
| `workbench.editor.processExplorer` | WB-6.2 |
| `workbench.editSessions.actions.resetAuth` | AB-4.9 |
| `workbench.editSessions.actions.signIn` | AB-4.9 |
| `workbench.explorer.fileView.focus` | SB-2.1 |
| `workbench.extensions.action.showExtensionsWithIds` | ED-8.48 |
| `workbench.files.action.acceptLocalChanges` | ED-6.45 |
| `workbench.files.action.closeGroup` | SB-2.30 |
| `workbench.files.action.collapseExplorerFolders` | SB-2.2 |
| `workbench.files.action.compareFileWith` | ED-6.11 |
| `workbench.files.action.compareNewUntitledTextFiles` | ED-6.44 |
| `workbench.files.action.compareWithClipboard` | ED-6.12 |
| `workbench.files.action.compareWithSaved` | SB-2.32, ED-6.10 |
| `workbench.files.action.createFileFromExplorer` | SB-2.6 |
| `workbench.files.action.createFolderFromExplorer` | SB-2.6 |
| `workbench.files.action.createMissingFile` | ED-10.71 |
| `workbench.files.action.focusFilesExplorer` | SB-2.1 |
| `workbench.files.action.focusOpenEditorsView` | SB-2.30 |
| `workbench.files.action.openFolder` | ED-10.72 |
| `workbench.files.action.refreshFilesExplorer` | SB-2.20 |
| `workbench.files.action.resolveConflict` | ED-6.45 |
| `workbench.files.action.resolveConflictLearnMore` | ED-6.45 |
| `workbench.files.action.reveal` | ED-10.72 |
| `workbench.files.action.revertLocalChanges` | ED-6.45 |
| `workbench.files.action.saveAllInGroup` | SB-2.30, SB-2.32 |
| `workbench.files.action.showActiveFileInExplorer` | SB-2.14 |
| `workbench.input.interactive` | ED-7.37 |
| `workbench.notebook.layout.configure` | ED-7.26, ED-7.28 |
| `workbench.notebook.layout.configure.editorTitle` | ED-7.28 |
| `workbench.notebook.layout.select` | ED-7.28 |
| `workbench.notification.neverShowAgain` | NT-2.11 |
| `workbench.output.action.clearOutput` | PN-5.9 |
| `workbench.output.action.switchBetweenOutputs` | PN-5.2 |
| `workbench.output.action.toggleAutoScroll` | PN-5.8 |
| `workbench.panel.markers.view.focus` | PN-4.1 |
| `workbench.panel.output.view.focus` | PN-5.1 |
| `workbench.panel.repl` | PN-6.1 |
| `workbench.panel.repl.view` | PN-6.1 |
| `workbench.panel.repl.view.focus` | PN-6.1 |
| `workbench.panel.testResults` | PN-6.18 |
| `workbench.panel.testResults.view.focus` | PN-6.18 |
| `workbench.profiles.actions.cleanupProfiles` | ED-8.14 |
| `workbench.profiles.actions.createFromCurrentProfile` | ED-8.14 |
| `workbench.profiles.actions.createProfile` | ED-8.14 |
| `workbench.profiles.actions.createTemporaryProfile` | ED-8.14 |
| `workbench.profiles.actions.deleteProfile` | ED-8.14 |
| `workbench.profiles.actions.exportProfile` | ED-8.14 |
| `workbench.profiles.actions.help` | ED-8.14 |
| `workbench.profiles.actions.manageProfiles` | AB-4.17, ED-8.14 |
| `workbench.profiles.actions.newWindowWithProfile` | ED-8.14 |
| `workbench.profiles.actions.profileEntry.{profile` | AB-4.17 |
| `workbench.profiles.actions.resetWorkspaces` | ED-8.14 |
| `workbench.profiles.actions.switchProfile` | AB-4.17, ED-8.14 |
| `workbench.remoteTunnel.actions.connecting` | AB-4.6 |
| `workbench.remoteTunnel.actions.manage` | AB-4.6 |
| `workbench.remoteTunnel.actions.turnOn` | AB-4.6 |
| `workbench.scm.action.collapseAll` | SB-4.5 |
| `workbench.scm.action.collapseAllRepositories` | SB-4.28 |
| `workbench.scm.action.expandAllRepositories` | SB-4.28 |
| `workbench.scm.action.focusNextInput` | SB-4.52 |
| `workbench.scm.action.focusNextResourceGroup` | SB-4.52 |
| `workbench.scm.action.focusPreviousInput` | SB-4.52 |
| `workbench.scm.action.focusPreviousResourceGroup` | SB-4.52 |
| `workbench.scm.action.graph.copyHistoryItemId` | SB-4.34 |
| `workbench.scm.action.graph.openFile` | SB-4.49 |
| `workbench.scm.action.graph.openInCursorBlame` | SB-4.36, ED-6.40 |
| `workbench.scm.action.graph.pickHistoryItemRefs` | SB-4.34 |
| `workbench.scm.action.graph.pickRepository` | SB-4.34 |
| `workbench.scm.action.graph.refresh` | SB-4.34 |
| `workbench.scm.action.graph.revealCurrentHistoryItem` | SB-4.34 |
| `workbench.scm.action.graph.setListViewMode` | SB-4.34, SB-4.49 |
| `workbench.scm.action.graph.setTreeViewMode` | SB-4.34, SB-4.49 |
| `workbench.scm.action.graph.viewChanges` | SB-4.36 |
| `workbench.scm.action.openBugbotSettings` | SB-4.39 |
| `workbench.scm.action.openReviewChangesPane` | SB-4.38, ED-6.35 |
| `workbench.scm.action.repositories.setSortKey.discoveryTime` | SB-4.28 |
| `workbench.scm.action.repositories.setSortKey.name` | SB-4.28 |
| `workbench.scm.action.repositories.setSortKey.path` | SB-4.28 |
| `workbench.scm.action.setListViewMode` | SB-4.5 |
| `workbench.scm.action.setListViewModeNavigation` | SB-4.5 |
| `workbench.scm.action.setSortKey.name` | SB-4.5 |
| `workbench.scm.action.setSortKey.path` | SB-4.5 |
| `workbench.scm.action.setSortKey.status` | SB-4.5 |
| `workbench.scm.action.setTreeViewMode` | SB-4.5 |
| `workbench.scm.action.setTreeViewModeNavigation` | SB-4.5 |
| `workbench.scm.action.toggleQuickDiffVisibility.<provider` | ED-6.24 |
| `workbench.scm.action.toggleRepositoryVisibility.<provider` | SB-4.28 |
| `workbench.scm.action.triggerAgentReview` | SB-4.39 |
| `workbench.scm.history` | SB-4.34 |
| `workbench.statusBar.clearFocus` | SS-1.8 |
| `workbench.statusBar.focusFirst` | SS-1.8 |
| `workbench.statusBar.focusLast` | SS-1.8 |
| `workbench.statusBar.focusNext` | SS-1.8 |
| `workbench.statusBar.focusPrevious` | SS-1.8 |
| `workbench.trust.configure` | ED-8.18, DL-5.10 |
| `workbench.trust.manage` | BN-2.1, SB-4.40, ED-8.6, ED-8.18, ED-8.52, ED-10.46, SS-3.51, SS-4.5, DL-5.9 |
| `workbench.userData.actions.signin` | AB-4.7 |
| `workbench.userData.actions.turningOn` | AB-4.7 |
| `workbench.userDataSync.actions.acceptMerges` | ED-5.49 |
| `workbench.userDataSync.actions.manage` | AB-4.7 |
| `workbench.userDataSync.actions.showConflicts` | AB-4.8 |
| `workbench.userDataSync.actions.turnOn` | AB-4.7, ED-8.49 |
| `workbench.view.debug` | AB-1.9, SB-5.1 |
| `workbench.view.explorer` | AB-1.9, SB-2.1, ED-9.18 |
| `workbench.view.extension.test` | AB-1.9, SB-6.1 |
| `workbench.view.extensions` | AB-1.9 |
| `workbench.view.scm` | AB-1.9, SB-4.1 |
| `workbench.view.search` | AB-1.9, SB-3.1 |
| `workbench.view.testing.focus` | SB-6.1 |
| `workench.action.disableEmptyEditorHint` | ED-9.24 |
| `workspaces.dontPromptToOpen` | WB-9.5 |
| `{providerId}:{extensionId}:signIn{n}` | AB-4.5 |
| `{providerId}{extensionId}Access` | AB-4.5 |
| `~remote.forwardedPorts.focus` | PN-7.10, SS-4.6 |

## Menus

| Menu id | Features |
|---|---|
| `a` | SB-5.5, ED-6.16 |
| `AccessibleView` | AC-2.38, PN-7.7, WB-4.1, WB-4.3, WB-4.4 |
| `AccountsContext` | AB-4.1, AB-4.4, AB-4.5, AB-4.6, AB-4.7, AB-4.9, AB-4.24 |
| `action` | SB-4.10 |
| `ActivityBarPositionMenu` | TB-1.18, TB-1.31, AB-5.2, AB-5.4, AX-4.3, AX-4.5 |
| `AuxiliaryBarTitle` | AX-1.2, AX-3.1, AX-3.3, AX-4.10 |
| `both` | QA-4.13 |
| `Browser` | ED-10.53, ED-10.54, ED-10.55, ED-10.57 |
| `BrowserActionsToolbar` | ED-10.61, ED-10.63, ED-10.64, ED-10.65, ED-10.67, ED-10.76, ED-10.77, ED-10.78, ED-10.80, ED-10.87 |
| `BrowserEmulationToolbar` | ED-10.61, ED-10.62 |
| `BrowserNavigationToolbar` | ED-10.79 |
| `BulkEditContext` | PN-7.23, PN-7.24, PN-7.25, PN-7.26, DL-1.29 |
| `BulkEditTitle` | PN-7.26 |
| `callhierarchy/title` | ED-4.31 |
| `Clear` | PN-5.16 |
| `Color` | ED-8.15 |
| `CommandCenter` | TB-2.1, TB-2.4, TB-2.16, ED-1.32 |
| `CommandCenterCenter` | TB-2.1, ED-11.19 |
| `CommandPalette` | TB-1.7, TB-1.16, TB-3.4, TB-3.13, TB-3.15, TB-3.16, TB-3.20, TB-3.21, TB-4.7, TB-4.11, TB-4.19, BN-1.6, AB-1.9, AB-1.10, AB-1.11, AB-5.2, AB-5.4, SB-1.3, SB-1.4, SB-1.14, SB-1.16, SB-1.17, SB-1.19, SB-1.24, SB-2.1, SB-2.2, SB-2.6, SB-2.14, SB-2.15, SB-2.16, SB-2.20, SB-2.39, SB-3.28, SB-5.5, SB-5.6, SB-5.19, SB-5.27, SB-5.29, SB-5.31, SB-6.6, SB-6.8, SB-6.9, SB-6.11, SB-6.12, SB-6.13, SB-6.14, SB-6.21, SB-6.26, SB-6.27, SB-6.28, SB-6.29, SB-6.30, SB-7.14, SB-7.15, ED-1.6, ED-1.7, ED-1.8, ED-1.9, ED-1.12, ED-1.14, ED-1.16, ED-1.17, ED-1.18, ED-1.20, ED-1.21, ED-1.22, ED-1.27, ED-1.28, ED-1.29, ED-1.30, ED-1.31, ED-1.33, ED-1.43, ED-1.44, ED-1.51, ED-2.6, ED-2.28, ED-2.31, ED-3.1, ED-3.2, ED-3.3, ED-3.6, ED-3.7, ED-3.8, ED-3.14, ED-3.29, ED-3.31, ED-3.32, ED-3.34, ED-3.35, ED-3.37, ED-3.38, ED-3.46, ED-4.30, ED-4.31, ED-4.32, ED-4.39, ED-4.40, ED-4.41, ED-5.1, ED-5.4, ED-5.6, ED-5.7, ED-5.9, ED-5.10, ED-5.11, ED-5.12, ED-5.13, ED-5.14, ED-5.15, ED-5.16, ED-5.17, ED-5.33, ED-5.35, ED-5.36, ED-5.38, ED-5.39, ED-5.45, ED-5.47, ED-5.48, ED-6.2, ED-6.4, ED-6.6, ED-6.7, ED-6.9, ED-6.10, ED-6.11, ED-6.12, ED-6.13, ED-6.14, ED-6.20, ED-6.21, ED-6.22, ED-6.23, ED-6.27, ED-6.28, ED-6.29, ED-6.30, ED-6.31, ED-6.32, ED-6.33, ED-6.34, ED-6.39, ED-6.40, ED-6.44, ED-6.52, ED-7.29, ED-7.37, ED-7.39, ED-7.41, ED-8.1, ED-8.3, ED-8.7, ED-8.8, ED-8.9, ED-8.14, ED-8.15, ED-8.16, ED-8.17, ED-8.18, ED-8.19, ED-8.20, ED-8.23, ED-8.24, ED-8.25, ED-8.27, ED-8.29, ED-8.31, ED-8.33, ED-8.34, ED-8.35, ED-8.37, ED-8.38, ED-8.39, ED-8.40, ED-8.53, ED-8.54, ED-8.55, ED-8.57, ED-8.68, ED-10.91, ED-12.2, ED-12.6, ED-12.7, ED-13.4, AC-2.12, AC-2.14, AC-2.15, AC-2.16, AC-2.36, AC-2.38, AC-2.40, AC-4.11, AC-4.13, AC-4.14, AC-4.17, AC-5.3, AC-5.25, AC-8.3, AC-8.6, AC-8.7, AC-8.9, AC-8.18, AC-8.19, PN-1.1, PN-1.2, PN-1.3, PN-1.4, PN-1.5, PN-1.7, PN-1.8, PN-1.15, PN-1.16, PN-1.19, PN-2.41, PN-2.50, PN-4.1, PN-4.16, PN-4.20, PN-5.1, PN-5.3, PN-5.4, PN-5.9, PN-5.12, PN-5.13, PN-5.14, PN-5.15, PN-5.19, PN-5.20, PN-6.1, PN-6.14, PN-6.16, PN-6.18, PN-6.26, PN-6.27, PN-6.30, PN-7.11, PN-7.12, PN-7.13, PN-7.14, AX-1.2, AX-1.3, AX-3.1, AX-3.2, AX-4.3, AX-4.9, AX-5.3, SS-1.7, SS-6.10, SS-6.13, SS-6.14, SS-6.15, SS-6.16, NT-1.9, NT-2.17, NT-3.6, NT-3.7, NT-3.9, NT-3.12, NT-4.1, NT-4.3, NT-4.8, NT-5.4, NT-6.4, NT-6.13, NT-6.16, QA-1.1, QA-1.10, QA-1.13, QA-1.21, QA-2.14, QA-3.3, QA-4.1, QA-4.2, QA-4.3, QA-4.4, QA-4.6, QA-4.8, QA-4.11, QA-4.12, QA-4.13, QA-4.14, QA-4.15, QA-4.17, QA-4.18, QA-5.1, QA-5.12, QA-5.14, QA-6.20, QA-6.26, DL-1.17, DL-1.20, DL-1.26, DL-2.3, DL-2.14, DL-2.18, DL-3.1, DL-3.2, DL-3.3, DL-3.4, DL-3.5, DL-3.6, DL-3.9, DL-5.9, DL-5.10, DL-5.13, DL-6.1, DL-6.3, DL-6.14, WB-1.3, WB-1.4, WB-1.5, WB-1.6, WB-2.1, WB-2.2, WB-2.3, WB-2.4, WB-2.7, WB-3.3, WB-3.4, WB-4.3, WB-4.4, WB-4.5, WB-4.8, WB-4.9, WB-5.1, WB-6.1, WB-6.2, WB-6.3, WB-6.4, WB-6.5, WB-6.6, WB-7.1, WB-8.1, WB-8.2, WB-8.3, WB-8.4, WB-8.5, WB-8.6, WB-8.7, WB-8.8, WB-8.9, WB-8.10, WB-8.11, WB-8.12, WB-8.13, WB-8.14, WB-8.15, WB-8.16, WB-8.17, WB-8.18, WB-8.19, WB-8.20, WB-8.22, WB-8.23, WB-8.24, WB-8.25, WB-9.1, WB-9.2, WB-9.3, WB-9.4, WB-9.7, WB-9.8, WB-9.9 |
| `commandPalette` | SB-4.14, SB-4.29, SB-4.32, SB-4.39, SB-4.41, SB-4.42, SB-4.43, SB-4.48, SB-7.20, SB-7.23, SB-7.24, SB-7.25, SB-7.26, ED-2.32, ED-2.33, ED-2.34, ED-2.44, ED-7.1, ED-10.1, ED-10.2, ED-10.3, ED-10.8, ED-10.9, ED-10.10, ED-10.11, ED-10.12, ED-10.15, ED-10.16, ED-10.17, ED-10.18, ED-10.21, ED-10.22, ED-10.23, ED-10.24, ED-10.25, ED-10.28, ED-10.29, ED-10.31, ED-10.33, ED-10.41, ED-10.42, ED-10.45, ED-10.61, ED-10.62, ED-10.63, ED-10.64, ED-10.65, ED-10.66, ED-10.67, ED-10.74, ED-10.75, ED-10.77, ED-10.78, ED-10.79, ED-10.80, ED-10.81, SS-3.44, SS-4.3, SS-4.4, SS-4.5, SS-4.11, SS-4.12, SS-4.13, SS-4.16, SS-5.2, SS-5.10, SS-5.15, SS-5.16, NT-4.5, WB-8.21 |
| `CommentActions` | ED-12.4 |
| `CommentEditorActions` | ED-12.4 |
| `CommentsViewThreadActions` | PN-7.7 |
| `CommentThreadActions` | ED-12.1 |
| `CommentThreadAdditionalActions` | ED-12.1 |
| `CommentThreadCommentContext` | ED-12.1 |
| `CommentThreadTitle` | ED-12.1 |
| `CommentThreadTitleContext` | ED-12.1 |
| `CommentTitle` | ED-12.4 |
| `CompactWindowEditorTitle` | ED-3.35, ED-6.13, ED-6.14, PN-2.7, PN-3.8, PN-3.12, PN-3.13 |
| `Configure` | QA-4.14 |
| `contributes.customEditors` | ED-10.1, ED-10.4, ED-10.5, ED-10.20 |
| `contributes.menus` | ED-12.1, ED-12.4, ED-13.1, ED-13.3 |
| `contributes.menus.commandPalette` | QA-1.13 |
| `copy` | WB-8.22 |
| `cpp-diff-peek-view/title` | AC-6.11, AC-6.12 |
| `Cursor` | TB-4.3, TB-4.4, TB-5.1 |
| `Cursor:` | ED-10.25 |
| `DebugBreakpointsContext` | SB-5.24, SB-5.25, SB-5.26, SB-5.28, SB-5.29 |
| `DebugCallStackContext` | SB-5.20, SB-5.21, SB-5.22, ED-10.45 |
| `DebugCallStackToolbar` | PN-6.25 |
| `DebugConsoleContext` | PN-6.9, PN-6.11, PN-6.14, PN-6.15 |
| `DebugCreateConfiguration` | SB-5.5 |
| `DebugDisassemblyContext` | ED-10.45, ED-10.89 |
| `DebugHoverContext` | ED-11.15 |
| `DebugScopesContext` | SB-5.11 |
| `DebugToolBar` | SB-5.9, SB-5.10, ED-11.17, ED-11.19 |
| `DebugToolBarStop` | SB-5.9, ED-11.17 |
| `DebugVariablesContext` | SB-5.12, SB-5.13, SB-5.14, SB-5.15, SB-5.17 |
| `DebugWatchContext` | SB-5.13, SB-5.14, SB-5.15, SB-5.17, SB-5.18 |
| `diffEditor/gutter/hunk` | ED-6.8 |
| `diffEditor/gutter/selection` | ED-6.8 |
| `DiffEditorHunkToolbar` | ED-6.7 |
| `DiffEditorSelectionToolbar` | ED-6.7 |
| `each` | SB-2.49 |
| `Edit` | SB-5.26 |
| `editor/content` | SB-4.12, ED-6.17, ED-6.22 |
| `editor/context` | SB-7.20, SB-7.21, SB-7.27, SB-7.28, ED-6.8 |
| `editor/title` | SB-4.12, ED-6.8, ED-6.23, ED-6.26, ED-10.3, ED-10.9, ED-10.10, ED-10.11, ED-10.12, ED-10.15, ED-10.18 |
| `editor/title/context` | ED-10.8 |
| `EditorActionsPositionSubmenu` | TB-1.18, ED-1.1 |
| `EditorContent` | SB-5.6, ED-5.49, ED-8.8, ED-8.55 |
| `EditorContext` | SB-6.29, SB-6.30, ED-2.4, ED-2.28, ED-4.3, ED-4.5, ED-4.6, ED-4.7, ED-4.8, ED-4.9, ED-4.12, ED-4.15, ED-4.16, ED-4.17, ED-4.21, ED-4.35, ED-6.39, ED-7.27, ED-7.33, ED-8.69, ED-10.45, ED-11.10, ED-11.20, ED-11.21, ED-11.22, ED-11.23, ED-11.24, PN-5.9, PN-5.16, QA-1.1, WB-7.2 |
| `EditorContextCopy` | ED-2.28 |
| `EditorContextCursorBlame` | ED-6.39, ED-6.40 |
| `EditorContextPeek` | ED-4.9, ED-4.31, ED-4.32 |
| `EditorContextShare` | ED-1.28, ED-7.27 |
| `EditorGroupWatermarkToolbar` | ED-9.18 |
| `EditorLineNumberContext` | ED-1.28, ED-6.24, ED-11.4, ED-13.2 |
| `EditorRenderWhitespaceSubmenu` | TB-1.18, ED-2.36, ED-5.35 |
| `EditorSplitMoveSubmenu` | ED-1.14, ED-1.24 |
| `EditorTabsBarContext` | ED-1.1, ED-1.13, ED-1.25, ED-1.33, ED-1.34, PN-2.7, QA-2.1, WB-1.5 |
| `EditorTabsBarShowTabsSubmenu` | TB-1.18, ED-1.1 |
| `EditorTabsBarShowTabsZenModeSubmenu` | TB-1.18, ED-1.34 |
| `EditorTitle` | SB-2.38, SB-7.14, ED-1.1, ED-1.4, ED-1.5, ED-1.8, ED-1.9, ED-1.13, ED-1.19, ED-1.23, ED-1.24, ED-1.31, ED-1.40, ED-2.35, ED-3.35, ED-5.36, ED-6.2, ED-6.3, ED-6.4, ED-6.5, ED-6.6, ED-6.9, ED-6.13, ED-6.14, ED-6.16, ED-6.18, ED-6.21, ED-6.45, ED-7.14, ED-7.15, ED-7.18, ED-7.19, ED-7.23, ED-7.26, ED-7.28, ED-7.34, ED-7.35, ED-7.36, ED-8.8, ED-8.9, ED-8.36, ED-8.54, ED-10.27, PN-2.7, PN-2.10, PN-2.21, PN-2.27, PN-3.8, PN-3.12, PN-3.13, QA-4.2, DL-2.1 |
| `EditorTitleContext` | SB-2.14, SB-2.15, SB-2.16, SB-2.17, SB-2.38, ED-1.6, ED-1.7, ED-1.8, ED-1.9, ED-1.11, ED-1.13, ED-1.14, ED-1.26, ED-1.27, ED-1.33, ED-1.43, ED-1.45, ED-5.10, ED-6.11, ED-10.23, ED-10.30, PN-2.11, PN-2.12, PN-2.26, PN-2.27, PN-2.40, DL-2.1 |
| `EditorTitleContextShare` | ED-1.28 |
| `EmptyEditorGroup` | ED-1.9, ED-1.19, ED-1.23, ED-1.25 |
| `EmptyEditorGroupContext` | ED-1.9, ED-1.13, ED-1.23, ED-1.25, ED-1.33, PN-2.7, QA-2.1, WB-1.5 |
| `explorer/context` | ED-10.8 |
| `ExplorerContext` | SB-2.4, SB-2.5, SB-2.6, SB-2.7, SB-2.8, SB-2.9, SB-2.13, SB-2.15, SB-2.16, SB-2.17, SB-2.18, SB-2.19, SB-2.38, SB-2.39, SB-2.40, SB-2.49, SB-3.14, SB-7.10, ED-6.11, ED-8.3, ED-10.44, DL-3.5, DL-4.1, DL-4.2, DL-4.9, DL-4.12 |
| `ExplorerContextShare` | ED-1.28 |
| `file/newFile` | ED-7.1 |
| `git.branch` | SB-4.16, SB-4.17, SB-4.18, SB-4.26 |
| `git.changes` | SB-4.2, SB-4.3 |
| `git.commit` | SB-4.9, SB-4.10, SB-4.11, SB-4.18 |
| `git.pullpush` | SB-4.22, SB-4.23, SB-4.24, SB-4.25 |
| `git.remotes` | SB-4.26 |
| `git.repositories.stash` | SB-4.30 |
| `git.stash` | SB-4.20 |
| `git.tags` | SB-4.19 |
| `git.worktrees` | SB-4.32 |
| `GlobalActivity` | AB-4.7, AB-4.8, AB-4.12, AB-4.14, AB-4.15, AB-4.16, AB-4.17, AB-4.18, AB-4.19, AB-4.20, AB-4.21, AB-4.22, AB-4.24, ED-8.1, ED-8.9, ED-8.14, ED-8.15, ED-8.16, ED-8.17, AC-4.17, QA-1.1 |
| `gotoErrorTitleMenu` | ED-4.27 |
| `Help:` | WB-4.5 |
| `hidden` | ED-6.35 |
| `ids` | ED-10.38 |
| `InlineChatEditorAffordance` | ED-4.14 |
| `InlineCompletionsActions` | AC-2.8 |
| `InlineEditsActions` | AC-2.10, AC-2.13, AC-2.19 |
| `InlineSuggestionToolbar` | AC-2.8, AC-2.10, AC-2.11, AC-2.12, AC-6.16, AC-8.11 |
| `InteractiveCellDelete` | ED-7.3, ED-7.38 |
| `InteractiveCellExecute` | ED-7.38 |
| `InteractiveCellTitle` | ED-7.38 |
| `InteractiveInputExecute` | ED-7.37 |
| `InteractiveToolbar` | ED-7.14, ED-7.15, ED-7.18, ED-7.19, ED-7.38, ED-7.39 |
| `IssueReporter` | QA-4.19 |
| `LayoutControlMenu` | TB-3.20, TB-3.21, TB-4.3, TB-4.4, TB-4.5, TB-4.6, TB-4.7, TB-4.10, TB-4.19, SB-1.1, PN-1.20 |
| `LayoutControlMenuSubmenu` | TB-4.3, TB-4.4, TB-4.5, TB-4.6, TB-4.7, SB-1.1, PN-1.1, AX-1.1 |
| `LayoutDensityMenu` | TB-4.20 |
| `macOS` | DL-1.17, DL-2.11 |
| `MarkerHoverStatusBar` | ED-4.26 |
| `menu` | SB-5.20 |
| `menu.view.filter` | PN-4.7 |
| `MenubarAppearanceMenu` | TB-1.4, TB-1.18, TB-1.31, TB-3.10, TB-3.13, TB-3.16, AB-5.2, SB-1.1, SB-1.2, ED-1.1, ED-1.34, ED-1.35, ED-2.36, ED-5.1, ED-5.4, ED-5.7, ED-5.35, ED-10.27, PN-1.1, PN-1.3, PN-1.4, AX-1.1, SS-1.1, NT-4.10 |
| `MenubarCopy` | ED-2.28 |
| `MenubarDebugMenu` | TB-1.1, TB-1.23, SB-5.3, SB-5.5, SB-5.6, SB-5.24, SB-5.25, ED-11.1 |
| `MenubarEditMenu` | TB-1.1, SB-2.10, SB-3.1, SB-3.6, ED-2.16, ED-2.28, ED-2.31, ED-2.32, ED-3.1, ED-3.14, ED-4.41, ED-7.41, DL-4.13 |
| `MenubarFileMenu` | TB-1.1, TB-1.14, TB-1.20, TB-1.27, TB-3.4, ED-1.8, ED-8.14, ED-9.25, SS-4.4, DL-2.1, DL-2.5, DL-2.10, DL-2.17, DL-2.18, DL-3.1, DL-3.2, DL-3.3, DL-3.4, DL-3.5, DL-3.6, DL-3.7, DL-3.9, WB-1.5, WB-1.6 |
| `MenubarGoMenu` | TB-1.1, TB-1.21, TB-1.22, TB-1.32, TB-2.4, ED-1.32, ED-2.10, ED-4.3, ED-4.5, ED-4.6, ED-4.7, ED-4.8, ED-4.27, ED-6.25, ED-7.44, SS-3.3, QA-2.1, QA-2.12, QA-3.1, QA-3.10 |
| `MenubarHelpMenu` | TB-1.1, TB-1.17, TB-1.27, TB-1.30, ED-9.1, ED-9.2, ED-9.7, ED-9.10, ED-9.13, ED-9.26, QA-1.1, DL-1.17, DL-1.20, DL-1.30, WB-6.1, WB-6.2, WB-6.5, WB-6.6, WB-8.14 |
| `MenubarLayoutMenu` | TB-1.19, SB-2.31, ED-1.13, ED-1.16, ED-1.24, ED-1.33 |
| `MenubarMainMenu` | TB-1.1, TB-1.24 |
| `MenubarNewBreakpointMenu` | TB-1.23, SB-5.27, SB-5.28, ED-11.5, ED-11.6, ED-11.7, ED-11.8, ED-11.10 |
| `MenubarPreferencesMenu` | TB-1.20, TB-1.24, ED-8.1, ED-8.2, ED-8.9, ED-8.14, ED-8.15, ED-8.16, ED-8.17, ED-8.19, ED-8.21, ED-8.42, AC-4.17 |
| `MenubarRecentMenu` | TB-1.14, TB-1.15, TB-1.16, ED-1.10, DL-1.26, DL-2.14, WB-9.7 |
| `MenubarSelectionMenu` | TB-1.1, ED-2.1, ED-2.2, ED-2.3, ED-2.4, ED-2.5, ED-2.6, ED-2.7, ED-2.11, SS-3.25 |
| `MenubarShare` | ED-8.14 |
| `MenubarSwitchEditorMenu` | TB-1.21, ED-1.24, ED-1.29, ED-1.30 |
| `MenubarSwitchGroupMenu` | TB-1.22, ED-1.22 |
| `MenubarTerminalMenu` | TB-1.1, PN-2.2, PN-2.6, PN-2.10, PN-2.21, SS-5.11, QA-4.8, QA-4.11, QA-4.12, QA-4.13, QA-4.14 |
| `MenubarTerminalSuggestStatusMenu` | AC-5.10, AC-5.15, AC-5.16, AC-5.17 |
| `MenubarViewMenu` | TB-1.1, TB-1.18, TB-1.19, AB-1.9, SB-2.1, SB-3.1, SB-5.1, SB-6.1, ED-2.35, ED-5.36, PN-2.1, PN-4.1, PN-5.1, PN-6.1, QA-1.1, QA-4.1 |
| `MergeBaseToolbar` | ED-6.18 |
| `MergeInput1Toolbar` | ED-6.18, ED-6.20 |
| `MergeInput2Toolbar` | ED-6.18, ED-6.20 |
| `MergeInputResultToolbar` | ED-6.20 |
| `modalEditor/editorTitle` | ED-10.18 |
| `ModalEditorEditorTitle` | ED-1.36, ED-8.8, ED-8.54 |
| `ModalEditorTitle` | ED-1.36 |
| `ModalEditorTitleContext` | ED-1.36 |
| `multiDiffEditor/resource/title` | SB-4.2, ED-6.15 |
| `MultiDiffEditorFileToolbar` | ED-6.14, ED-7.34, ED-7.36 |
| `NewFile` | ED-9.25 |
| `notebook` | ED-5.7 |
| `notebook/cell/title` | ED-7.27 |
| `notebook/toolbar` | ED-7.26 |
| `NotebookCellBetween` | ED-7.2 |
| `NotebookCellDelete` | ED-7.3, ED-7.27 |
| `NotebookCellExecute` | ED-7.13, ED-7.14 |
| `NotebookCellExecuteGoTo` | ED-7.15 |
| `NotebookCellExecutePrimary` | ED-7.13 |
| `NotebookCellInsert` | ED-7.2 |
| `NotebookCellListTop` | ED-7.2 |
| `NotebookCellTitle` | ED-7.2, ED-7.4, ED-7.5, ED-7.6, ED-7.7, ED-7.8, ED-7.12, ED-7.14, ED-7.23, ED-7.27, ED-7.28 |
| `NotebookDiffCellInputTitle` | ED-7.35, ED-7.36 |
| `NotebookDiffCellMetadataTitle` | ED-7.36 |
| `NotebookDiffCellOutputsTitle` | ED-7.35, ED-7.36 |
| `NotebookDiffDocumentMetadata` | ED-7.36 |
| `NotebookEditorLayoutConfigure` | ED-7.28 |
| `NotebookKernelSource` | ED-7.19 |
| `NotebookOutlineActionMenu` | SB-7.6, ED-7.12, ED-7.31 |
| `NotebookOutlineFilter` | SB-7.6, ED-7.31 |
| `NotebookOutputToolbar` | ED-7.21, ED-7.23 |
| `NotebookStickyScrollContext` | ED-7.12, ED-7.29 |
| `NotebookToolbar` | ED-5.7, ED-7.2, ED-7.14, ED-7.15, ED-7.18, ED-7.19, ED-7.23, ED-7.26, ED-7.28 |
| `NotebookToolbarContext` | ED-7.26, ED-7.29 |
| `NotebookVariablesContext` | SB-5.16 |
| `NotificationsCenterPositionMenu` | NT-3.11 |
| `only` | AC-5.10 |
| `Open` | ED-6.6 |
| `OpenEditorsContext` | SB-2.4, SB-2.15, SB-2.16, SB-2.17, SB-2.32, ED-6.10, ED-6.11 |
| `OpenEditorsContextShare` | ED-1.28 |
| `order` | ED-5.35 |
| `other` | ED-8.18 |
| `PanelAlignmentMenu` | TB-1.18, PN-1.4, PN-1.13 |
| `PanelPositionMenu` | TB-1.18, PN-1.3, PN-1.13 |
| `PanelTitle` | PN-1.1, PN-1.5, PN-1.17 |
| `ProblemsPanelContext` | PN-4.13, PN-4.14, PN-4.15, PN-4.17 |
| `Process` | WB-6.7 |
| `ProfilesMenu` | TB-1.20, AB-4.17 |
| `Quick` | QA-4.1, WB-9.7, WB-9.8 |
| `Remove` | SB-5.25 |
| `ReplInputExecute` | ED-7.39 |
| `Report` | WB-6.6 |
| `scm/artifact/context` | SB-4.30 |
| `scm/artifactGroup/context` | SB-4.30 |
| `scm/change/title` | ED-6.25 |
| `scm/history/title` | SB-4.22, SB-4.26, SB-4.34, SB-4.49 |
| `scm/historyItem/context` | SB-4.36 |
| `scm/historyItem/hover` | SB-4.34 |
| `scm/historyItemRef/context` | SB-4.36 |
| `scm/inputBox` | SB-4.14 |
| `scm/repositories/title` | SB-4.29 |
| `scm/repository` | SS-4.11 |
| `scm/resourceFolder/context` | SB-4.2, SB-4.3, SB-4.6 |
| `scm/resourceGroup/context` | SB-4.1, SB-4.2, SB-4.3, SB-4.4, SB-4.21 |
| `scm/resourceState/context` | SB-4.2, SB-4.3, SB-4.4, SB-4.6, SB-4.21, ED-6.23 |
| `scm/sourceControl` | SB-4.29, SB-4.32 |
| `scm/sourceControl/title` | SB-4.29 |
| `scm/title` | SB-4.5, SB-4.7, SB-4.9, SB-4.15, SB-4.16, SB-4.19, SB-4.20, SB-4.22, SB-4.23, SB-4.24, SB-4.27, SB-4.37, SS-4.11 |
| `SCMArtifactContext` | SB-4.30 |
| `SCMArtifactGroupContext` | SB-4.30 |
| `SCMChangeContext` | ED-6.25 |
| `SCMHistoryItemChangeContext` | SB-4.49 |
| `SCMHistoryItemContext` | ED-6.40, SS-6.15 |
| `SCMHistoryItemRefContext` | SB-4.36 |
| `SCMHistoryTitle` | SB-4.34 |
| `SCMInputBox` | SB-4.7 |
| `SCMResourceContext` | SB-4.2 |
| `SCMResourceFolderContext` | SB-4.2 |
| `SCMResourceGroupContext` | SB-4.5 |
| `SCMSourceControlInline` | SB-4.28 |
| `SCMSourceControlTitle` | SB-4.28 |
| `SCMTitle` | SB-4.28, SB-4.38, ED-6.35 |
| `scrollbar` | ED-5.2, ED-5.29 |
| `SearchActionMenu` | SB-3.7, SB-3.17, SB-3.23 |
| `SearchContext` | SB-3.7, SB-3.15, SB-3.16, SB-3.19, SB-3.23, SB-3.24, SB-3.25 |
| `SidebarTitle` | SB-1.5 |
| `SimpleEditorContext` | ED-2.6, ED-2.28, ED-2.31, WB-7.2 |
| `StatusBarRemoteIndicatorMenu` | SS-4.3 |
| `StatusBarWindowIndicatorMenu` | SS-4.3 |
| `StickyScrollContext` | ED-5.4, ED-5.7 |
| `submenu.filter.commentSort` | PN-7.5 |
| `submenu.view.filter` | PN-4.7, PN-4.8, PN-5.6, PN-5.7, PN-7.4 |
| `suggestWidgetStatusBar` | AC-1.28, AC-1.29, AC-1.30, AC-1.31, AC-1.34 |
| `tab` | AX-2.2, AX-4.2, AX-4.5, AX-5.2 |
| `Tasks:` | QA-4.11, QA-4.13, QA-4.14 |
| `TerminalEditorInstanceContext` | PN-2.2, PN-2.21, PN-2.27, PN-2.40, PN-3.15, PN-3.16, PN-3.18, PN-3.19, PN-3.34 |
| `TerminalInstanceContext` | PN-2.28, PN-2.37, PN-2.40, PN-3.15, PN-3.16, PN-3.18, PN-3.19, PN-3.34 |
| `TerminalNewDropdownContext` | PN-2.4, PN-2.5, PN-2.6 |
| `TerminalStickyScrollContext` | PN-3.11 |
| `TerminalTabContext` | PN-2.11, PN-2.12, PN-2.18, PN-2.21, PN-2.24, PN-2.25, PN-2.28, PN-2.40 |
| `TerminalTabEmptyAreaContext` | PN-2.2, PN-2.3, PN-2.15, PN-2.16 |
| `Terminate` | QA-4.13 |
| `TestCallStack` | PN-6.28 |
| `TestCoverageFilterItem` | SB-6.26, ED-13.5 |
| `testing.openCoverage` | SB-6.27 |
| `TestItem` | SB-6.7, SB-6.8, SB-6.9, SB-6.10, SB-6.12, SB-6.14, SB-6.15, SB-6.16, SB-6.17 |
| `TestItemGutter` | ED-13.1, ED-13.2 |
| `TestMessageContent` | PN-6.22 |
| `TestMessageContext` | ED-13.3, PN-6.29 |
| `TestPeekElement` | ED-13.4, PN-6.24, PN-6.26, PN-6.27, PN-6.28, PN-6.29 |
| `TestPeekTitle` | ED-13.4, PN-6.25, PN-6.30 |
| `TestProfilesContext` | SB-6.10 |
| `the` | SB-5.21, SB-6.15, ED-5.1, ED-8.36, ED-10.21, ED-10.66, SS-1.1, SS-1.2, SS-1.3, SS-1.15, WB-7.1, WB-9.8 |
| `ThemesSubMenu` | TB-1.20, AB-4.20 |
| `timeline/item/context` | SB-7.17, SB-7.18, SB-7.19 |
| `TimelineFilterSubMenu` | SB-7.9 |
| `TimelineItemContext` | SB-7.13, SB-7.14, SB-7.15 |
| `TimelineTitle` | SB-7.7, SB-7.8, SB-7.9, SB-7.11, SB-7.14 |
| `TitleBar` | TB-5.8, TB-5.9 |
| `TitleBarAdjacentCenter` | TB-5.1 |
| `TitleBarContext` | TB-1.4, TB-2.2, TB-2.5, TB-2.6, TB-3.10, TB-3.11, TB-3.12, TB-4.1, TB-5.8, TB-5.9 |
| `TitleBarTitleContext` | TB-1.4, TB-2.2, TB-2.5, TB-2.7, TB-3.10, TB-3.11, TB-3.12, TB-4.1, TB-5.8, TB-5.9 |
| `titles` | ED-3.29 |
| `TouchBarContext` | ED-1.32, WB-9.6 |
| `TunnelContext` | PN-7.11, PN-7.12, PN-7.13, PN-7.14, PN-7.15, PN-7.16, PN-7.17, PN-7.18 |
| `TunnelLocalAddressInline` | PN-7.13, PN-7.14 |
| `TunnelPortInline` | PN-7.11, PN-7.12, PN-7.15 |
| `TunnelPrivacy` | PN-7.17 |
| `TunnelProtocol` | PN-7.18 |
| `typehierarchy/title` | ED-4.32 |
| `Unhide` | SB-6.17 |
| `usable` | ED-3.46 |
| `Use` | ED-6.2 |
| `view/item/context` | SB-7.24, SB-7.26, SB-7.27, SB-7.28 |
| `view/title` | SB-7.25, SB-7.26, SB-7.27, SB-7.28 |
| `View:` | QA-4.4 |
| `ViewContainerTitle` | AB-6.13, SB-1.5, SB-5.3, SB-5.4, SB-5.5, SB-5.10, ED-11.19, QA-4.15, WB-6.4 |
| `ViewContainerTitleContext` | AB-2.8, AB-5.2, AB-6.10, SB-1.1, SB-1.2, SB-1.6, SB-1.9, AX-1.2, AX-2.2, AX-4.3, AX-5.2 |
| `ViewItemContext` | SB-7.24 |
| `Views` | SB-1.5, SB-1.9, PN-1.13, PN-6.1 |
| `ViewsSubMenu` | AX-4.5 |
| `viewsWelcome` | SB-4.27, SB-4.40 |
| `ViewTitle` | SB-1.8, SB-2.2, SB-2.6, SB-2.20, SB-2.31, SB-3.18, SB-3.19, SB-3.27, SB-3.28, SB-4.39, SB-5.11, SB-5.17, SB-5.18, SB-5.19, SB-5.24, SB-5.25, SB-5.27, SB-5.28, SB-5.30, SB-5.31, SB-6.2, SB-6.3, SB-6.6, SB-6.8, SB-6.9, SB-6.12, SB-6.13, SB-6.14, SB-6.24, SB-6.25, SB-6.26, SB-6.27, SB-7.1, SB-7.2, SB-7.3, SB-7.4, SB-7.6, ED-7.31, PN-1.17, PN-2.2, PN-2.4, PN-2.10, PN-2.15, PN-2.17, PN-2.21, PN-2.28, PN-3.8, PN-3.12, PN-3.13, PN-4.2, PN-4.3, PN-4.4, PN-5.2, PN-5.4, PN-5.8, PN-5.9, PN-5.10, PN-5.11, PN-5.12, PN-5.13, PN-5.14, PN-6.11, PN-6.14, PN-6.16, PN-6.18, PN-6.26, PN-6.30, PN-6.31, PN-7.6, WB-1.5 |
| `ViewTitleContext` | SB-1.9, SB-1.17 |
| `webview/context` | ED-7.21, ED-7.22, ED-10.2, ED-10.16, ED-10.17, ED-10.22, ED-10.73 |
| `workbench.output.menu.logLevel` | PN-5.4 |
| `workbench.output.menu.switchOutput` | PN-5.2, PN-5.20 |
| `Zoom` | ED-10.76 |

## Settings

| Setting id | Features |
|---|---|
| `[Log]` | PN-5.16 |
| `[snippets]` | AC-4.19 |
| `accessibility.debugWatchVariableAnnouncements` | SB-5.17 |
| `accessibility.replEditor.autoFocusReplExecution` | ED-7.39 |
| `accessibility.replEditor.readLastExecutionOutput` | ED-7.39 |
| `accessibility.signalOptions.debouncePositionChanges` | WB-4.6 |
| `accessibility.signalOptions.experimental.delays.errorAtPosition` | WB-4.6 |
| `accessibility.signalOptions.experimental.delays.general` | WB-4.6 |
| `accessibility.signalOptions.experimental.delays.warningAtPosition` | WB-4.6 |
| `accessibility.signalOptions.volume` | WB-4.6 |
| `accessibility.signals.clear` | NT-2.14, NT-3.9 |
| `accessibility.signals.codeActionApplied` | WB-4.5 |
| `accessibility.signals.codeActionTriggered` | WB-4.5 |
| `accessibility.signals.diffLineDeleted` | WB-4.5 |
| `accessibility.signals.diffLineInserted` | WB-4.5 |
| `accessibility.signals.diffLineModified` | WB-4.5 |
| `accessibility.signals.format` | WB-4.5 |
| `accessibility.signals.lineHasBreakpoint` | WB-4.5 |
| `accessibility.signals.lineHasError` | WB-4.5 |
| `accessibility.signals.lineHasFoldedArea` | WB-4.5 |
| `accessibility.signals.lineHasInlineSuggestion` | AC-2.39 |
| `accessibility.signals.lineHasWarning` | WB-4.5 |
| `accessibility.signals.nextEditSuggestion` | AC-2.39 |
| `accessibility.signals.noInlayHints` | WB-4.5 |
| `accessibility.signals.notebookCellCompleted` | WB-4.5 |
| `accessibility.signals.notebookCellFailed` | WB-4.5 |
| `accessibility.signals.onDebugBreak` | WB-4.5 |
| `accessibility.signals.positionHasError` | WB-4.5 |
| `accessibility.signals.positionHasWarning` | WB-4.5 |
| `accessibility.signals.progress` | NT-5.12 |
| `accessibility.signals.save` | WB-4.5 |
| `accessibility.signals.taskCompleted` | WB-4.5 |
| `accessibility.signals.taskFailed` | WB-4.5 |
| `accessibility.signals.terminalBell` | PN-3.20 |
| `accessibility.signals.terminalCommandFailed` | PN-3.23 |
| `accessibility.signals.terminalCommandSucceeded` | PN-3.23 |
| `accessibility.signals.terminalQuickFix` | PN-3.23 |
| `accessibility.underlineLinks` | WB-4.7 |
| `accessibility.verbosity.comments` | WB-4.3 |
| `accessibility.verbosity.debug` | DL-1.38, WB-4.3 |
| `accessibility.verbosity.editor` | WB-4.3 |
| `accessibility.verbosity.emptyEditorHint` | ED-9.24 |
| `accessibility.verbosity.find` | ED-3.28, PN-4.20, PN-5.19, WB-4.3 |
| `accessibility.verbosity.hover` | ED-4.39, WB-4.3 |
| `accessibility.verbosity.inlineCompletions` | AC-2.38, AC-2.39 |
| `accessibility.verbosity.keybindingsEditor` | WB-4.3 |
| `accessibility.verbosity.notebook` | WB-4.3 |
| `accessibility.verbosity.notification` | NT-3.12, WB-4.3 |
| `accessibility.verbosity.sourceControl` | WB-4.3 |
| `accessibility.verbosity.terminal` | PN-3.22, PN-3.24, WB-4.3 |
| `accessibility.voice.speechLanguage` | ED-2.38 |
| `accessibility.windowTitleOptimized` | TB-2.14 |
| `application.shellEnvironmentResolutionTimeout` | NT-5.13 |
| `autoOpenLocalhostUrls` | ED-10.88, PN-3.46 |
| `autoRunBugbotOnCommit` | SB-4.39 |
| `breadcrumbs.enabled` | ED-5.7, ED-7.47 |
| `breadcrumbs.filePath` | ED-5.8 |
| `breadcrumbs.icons` | ED-5.8 |
| `breadcrumbs.showArrays` | ED-5.8 |
| `breadcrumbs.showBooleans` | ED-5.8 |
| `breadcrumbs.showClasses` | ED-5.8 |
| `breadcrumbs.showConstants` | ED-5.8 |
| `breadcrumbs.showConstructors` | ED-5.8 |
| `breadcrumbs.showEnumMembers` | ED-5.8 |
| `breadcrumbs.showEnums` | ED-5.8 |
| `breadcrumbs.showEvents` | ED-5.8 |
| `breadcrumbs.showFields` | ED-5.8 |
| `breadcrumbs.showFiles` | ED-5.8 |
| `breadcrumbs.showFunctions` | ED-5.8 |
| `breadcrumbs.showInterfaces` | ED-5.8 |
| `breadcrumbs.showKeys` | ED-5.8 |
| `breadcrumbs.showMethods` | ED-5.8 |
| `breadcrumbs.showModules` | ED-5.8 |
| `breadcrumbs.showNamespaces` | ED-5.8 |
| `breadcrumbs.showNull` | ED-5.8 |
| `breadcrumbs.showNumbers` | ED-5.8 |
| `breadcrumbs.showObjects` | ED-5.8 |
| `breadcrumbs.showOperators` | ED-5.8 |
| `breadcrumbs.showPackages` | ED-5.8 |
| `breadcrumbs.showProperties` | ED-5.8 |
| `breadcrumbs.showStrings` | ED-5.8 |
| `breadcrumbs.showStructs` | ED-5.8 |
| `breadcrumbs.showTypeParameters` | ED-5.8 |
| `breadcrumbs.showVariables` | ED-5.8 |
| `breadcrumbs.symbolPath` | ED-5.8 |
| `breadcrumbs.symbolPathSeparator` | ED-5.10 |
| `breadcrumbs.symbolSortOrder` | ED-5.8 |
| `breadcrumbs.useQuickPick` | ED-5.9 |
| `chat.disableAIFeatures` | BN-2.8, AC-2.37, SS-6.1, SS-6.2 |
| `comments.collapseOnResolve` | ED-12.1 |
| `comments.maxHeight` | ED-12.1 |
| `comments.openPanel` | PN-7.9 |
| `comments.openView` | PN-7.9 |
| `comments.thread.confirmOnCollapse` | ED-12.5 |
| `comments.useRelativeTime` | ED-12.4, PN-7.1 |
| `comments.visible` | ED-12.2, ED-12.7 |
| `cursor.blame.hoverDelay` | ED-6.39, ED-6.40, ED-6.58 |
| `cursor.cpp.disabledLanguages` | AC-8.1, AC-8.4 |
| `cursor.cpp.enablePartialAccepts` | AC-6.16, AC-8.11 |
| `cursor.inlineDiff.enablePerformanceProtection` | ED-6.28 |
| `debug.allowBreakpointsEverywhere` | ED-11.1 |
| `debug.autoExpandLazyVariables` | SB-5.11 |
| `debug.breakpointsView.presentation` | SB-5.30 |
| `debug.closeReadonlyTabsOnEnd` | SB-5.31 |
| `debug.confirmOnExit` | DL-2.20 |
| `debug.console.acceptSuggestionOnEnter` | PN-6.5 |
| `debug.console.closeOnEnd` | PN-6.2 |
| `debug.console.collapseIdenticalLines` | PN-6.8 |
| `debug.console.fontFamily` | PN-6.13 |
| `debug.console.fontSize` | PN-6.13 |
| `debug.console.historySuggestions` | PN-6.5 |
| `debug.console.lineHeight` | PN-6.13 |
| `debug.console.maximumLines` | PN-6.17 |
| `debug.console.wordWrap` | PN-6.12 |
| `debug.disassemblyView.showSourceCode` | ED-10.45 |
| `debug.enableStatusBarColor` | SS-2.4 |
| `debug.focusEditorOnBreak` | ED-11.25 |
| `debug.focusWindowOnBreak` | ED-11.25 |
| `debug.gutterMiddleClickAction` | ED-11.9 |
| `debug.hideLauncherWhileDebugging` | SB-5.3 |
| `debug.hideSlowPreLaunchWarning` | SB-5.3 |
| `debug.inlineValues` | ED-11.14 |
| `debug.internalConsoleOptions` | PN-6.2 |
| `debug.onTaskErrors` | SB-5.3, DL-1.32 |
| `debug.openDebug` | SB-5.2 |
| `debug.openExplorerOnEnd` | SB-5.2 |
| `debug.saveBeforeStart` | SB-5.3 |
| `debug.showBreakpointsInOverviewRuler` | ED-11.12 |
| `debug.showInlineBreakpointCandidates` | ED-11.10 |
| `debug.showInStatusBar` | SS-5.9, QA-4.15 |
| `debug.showSubSessionsInToolBar` | SB-5.10, ED-11.17 |
| `debug.showVariableTypes` | SB-5.11 |
| `debug.terminal.clearBeforeReusing` | PN-2.44 |
| `debug.toolBarLocation` | SB-5.9, ED-11.19 |
| `diffEditor.codeLens` | ED-6.1 |
| `diffEditor.diffAlgorithm` | ED-6.1 |
| `diffEditor.experimental.showEmptyDecorations` | ED-6.1 |
| `diffEditor.experimental.showMoves` | ED-6.5 |
| `diffEditor.experimental.useTrueInlineView` | ED-6.1, ED-6.43 |
| `diffEditor.hideOriginalLineNumbers` | ED-6.51 |
| `diffEditor.hideUnchangedRegions.contextLineCount` | ED-6.4 |
| `diffEditor.hideUnchangedRegions.enabled` | ED-6.4, ED-6.46, ED-7.35 |
| `diffEditor.hideUnchangedRegions.minimumLineCount` | ED-6.4 |
| `diffEditor.hideUnchangedRegions.revealLineCount` | ED-6.4 |
| `diffEditor.ignoreTrimWhitespace` | ED-6.3, ED-6.41 |
| `diffEditor.maxComputationTime` | ED-6.1 |
| `diffEditor.maxFileSize` | ED-6.1 |
| `diffEditor.renderGutterMenu` | ED-6.1, ED-6.7 |
| `diffEditor.renderIndicators` | ED-6.1 |
| `diffEditor.renderMarginRevertIcon` | ED-6.1, ED-6.7 |
| `diffEditor.renderSideBySide` | ED-6.2, ED-6.43, ED-12.8 |
| `diffEditor.renderSideBySideInlineBreakpoint` | ED-6.2 |
| `diffEditor.useInlineViewWhenSpaceIsLimited` | ED-6.2 |
| `diffEditor.wordWrap` | ED-6.1 |
| `editor.acceptSuggestionOnCommitCharacter` | AC-1.33 |
| `editor.acceptSuggestionOnEnter` | AC-1.32 |
| `editor.accessibilityPageSize` | SS-3.26 |
| `editor.accessibilitySupport` | TB-1.28, PN-3.24, SS-3.26, DL-1.38, WB-4.8 |
| `editor.aiStats.enabled` | SS-6.2, SS-6.3, SS-6.4 |
| `editor.allowVariableFonts` | ED-5.38 |
| `editor.allowVariableFontsInAccessibilityMode` | ED-5.38 |
| `editor.allowVariableLineHeights` | ED-5.38 |
| `editor.autoClosingBrackets` | ED-2.22 |
| `editor.autoClosingComments` | ED-2.22 |
| `editor.autoClosingDelete` | ED-2.22 |
| `editor.autoClosingOvertype` | ED-2.22 |
| `editor.autoClosingQuotes` | ED-2.22 |
| `editor.autoIndent` | ED-2.23 |
| `editor.autoIndentOnPaste` | ED-2.23 |
| `editor.autoIndentOnPasteWithinString` | ED-2.23 |
| `editor.autoSurround` | ED-2.22 |
| `editor.bracketPairColorization` | ED-5.25 |
| `editor.bracketPairColorization.enabled` | ED-5.25 |
| `editor.bracketPairColorization.independentColorPoolPerBracketType` | ED-5.25 |
| `editor.codeActions.triggerOnFocusChange` | ED-4.13 |
| `editor.codeActionsOnSave` | ED-4.16, WB-1.4 |
| `editor.codeActionWidget.includeNearbyQuickFixes` | ED-4.14 |
| `editor.codeActionWidget.showHeaders` | ED-4.14 |
| `editor.codeLens` | ED-4.20 |
| `editor.codeLensFontFamily` | ED-4.20 |
| `editor.codeLensFontSize` | ED-4.20 |
| `editor.colorDecorators` | ED-4.29 |
| `editor.colorDecoratorsActivatedOn` | ED-4.29 |
| `editor.colorDecoratorsLimit` | ED-4.29, SS-3.31 |
| `editor.columnSelection` | ED-2.5, SS-3.25 |
| `editor.comments` | ED-2.16 |
| `editor.comments.ignoreEmptyLines` | ED-2.16 |
| `editor.comments.insertSpace` | ED-2.16 |
| `editor.contextmenu` | WB-7.2 |
| `editor.copyWithSyntaxHighlighting` | ED-2.28 |
| `editor.cursorBlinking` | ED-5.31 |
| `editor.cursorHeight` | ED-5.31 |
| `editor.cursorSmoothCaretAnimation` | ED-5.31 |
| `editor.cursorStyle` | ED-5.31 |
| `editor.cursorSurroundingLines` | ED-5.32 |
| `editor.cursorSurroundingLinesStyle` | ED-5.32 |
| `editor.cursorWidth` | ED-5.31 |
| `editor.defaultColorDecorators` | ED-4.29 |
| `editor.defaultFoldingRangeProvider` | ED-5.19 |
| `editor.defaultFormatter` | ED-4.17, SS-3.31 |
| `editor.definitionLinkOpensInPeek` | ED-4.4, ED-4.10 |
| `editor.detectIndentation` | ED-2.18, SS-3.4, SS-3.9 |
| `editor.doubleClickSelectsBlock` | ED-2.6 |
| `editor.dragAndDrop` | ED-2.30 |
| `editor.dropIntoEditor` | ED-2.29 |
| `editor.dropIntoEditor.enabled` | ED-1.15, ED-2.29 |
| `editor.dropIntoEditor.preferences` | ED-2.29 |
| `editor.dropIntoEditor.showDropSelector` | ED-2.29 |
| `editor.editContext` | ED-2.42 |
| `editor.emptySelectionClipboard` | ED-2.28 |
| `editor.experimental.asyncTokenization` | ED-5.51 |
| `editor.experimental.preferTreeSitter.css` | ED-5.51 |
| `editor.experimental.preferTreeSitter.ini` | ED-5.51 |
| `editor.experimental.preferTreeSitter.regex` | ED-5.51 |
| `editor.experimental.preferTreeSitter.typescript` | ED-5.51 |
| `editor.experimentalGpuAcceleration` | ED-5.45 |
| `editor.experimentalWhitespaceRendering` | ED-5.35 |
| `editor.fastScrollSensitivity` | ED-5.26 |
| `editor.find.addExtraSpaceOnTop` | ED-3.25 |
| `editor.find.autoFindInSelection` | ED-3.12 |
| `editor.find.closeOnResult` | ED-3.23 |
| `editor.find.cursorMoveOnType` | ED-3.4 |
| `editor.find.findOnType` | ED-3.4 |
| `editor.find.globalFindClipboard` | ED-3.27 |
| `editor.find.history` | ED-3.21 |
| `editor.find.loop` | ED-3.6 |
| `editor.find.replaceHistory` | ED-3.21 |
| `editor.find.seedSearchStringFromSelection` | ED-3.3, QA-2.14 |
| `editor.folding` | ED-5.11, ED-5.19 |
| `editor.foldingHighlight` | ED-5.18 |
| `editor.foldingImportsByDefault` | ED-5.17 |
| `editor.foldingMaximumRegions` | ED-5.19, SS-3.31 |
| `editor.foldingStrategy` | ED-5.19 |
| `editor.fontFamily` | ED-5.38 |
| `editor.fontLigatures` | ED-5.38 |
| `editor.fontSize` | ED-5.38, AC-7.1 |
| `editor.fontVariations` | ED-5.38 |
| `editor.fontWeight` | ED-5.38, AC-5.2 |
| `editor.formatOnPaste` | ED-2.24, ED-4.19 |
| `editor.formatOnSave` | ED-4.19, WB-1.4 |
| `editor.formatOnSaveMode` | ED-4.19 |
| `editor.formatOnType` | ED-2.24, ED-4.19 |
| `editor.glyphMargin` | ED-5.20, ED-8.51 |
| `editor.gotoLocation.alternativeDeclarationCommand` | ED-4.5 |
| `editor.gotoLocation.alternativeDefinitionCommand` | ED-4.3 |
| `editor.gotoLocation.alternativeImplementationCommand` | ED-4.7 |
| `editor.gotoLocation.alternativeReferenceCommand` | ED-4.8 |
| `editor.gotoLocation.alternativeTypeDefinitionCommand` | ED-4.6 |
| `editor.gotoLocation.multiple` | ED-4.3, ED-4.9, ED-4.33 |
| `editor.gotoLocation.multipleDeclarations` | ED-4.5, ED-4.33 |
| `editor.gotoLocation.multipleDefinitions` | ED-4.3, ED-4.33 |
| `editor.gotoLocation.multipleImplementations` | ED-4.7, ED-4.33 |
| `editor.gotoLocation.multipleReferences` | ED-4.8, ED-4.33 |
| `editor.gotoLocation.multipleTypeDefinitions` | ED-4.6, ED-4.33 |
| `editor.guides` | ED-5.23 |
| `editor.guides.bracketPairs` | ED-5.23 |
| `editor.guides.bracketPairsHorizontal` | ED-5.23 |
| `editor.guides.highlightActiveBracketPair` | ED-5.23 |
| `editor.guides.highlightActiveIndentation` | ED-5.23 |
| `editor.guides.indentation` | ED-5.23 |
| `editor.hideCursorInOverviewRuler` | ED-5.29 |
| `editor.hover` | ED-4.1 |
| `editor.hover.above` | ED-4.1 |
| `editor.hover.delay` | ED-4.1, ED-4.26, ED-11.15 |
| `editor.hover.enabled` | ED-4.1, ED-4.26 |
| `editor.hover.hidingDelay` | ED-4.1 |
| `editor.hover.showLongLineWarning` | ED-4.1 |
| `editor.hover.sticky` | ED-4.1 |
| `editor.indentSize` | ED-2.18, SS-3.4, SS-3.6, SS-3.35 |
| `editor.inertialScroll` | ED-5.26 |
| `editor.inlayHints` | ED-4.21 |
| `editor.inlayHints.enabled` | ED-4.21 |
| `editor.inlayHints.fontFamily` | ED-4.21 |
| `editor.inlayHints.fontSize` | ED-4.21 |
| `editor.inlayHints.maximumLength` | ED-4.21 |
| `editor.inlayHints.padding` | ED-4.21 |
| `editor.inlineCompletionsAccessibilityVerbose` | AC-2.39 |
| `editor.inlineSuggest` | AC-2.33 |
| `editor.inlineSuggest.edits.allowCodeShifting` | AC-2.24 |
| `editor.inlineSuggest.edits.longDistanceHintContextLineCount` | AC-2.30 |
| `editor.inlineSuggest.edits.renderSideBySide` | AC-2.21, AC-2.22 |
| `editor.inlineSuggest.edits.showCollapsed` | AC-2.18, AC-2.26 |
| `editor.inlineSuggest.edits.showLongDistanceHint` | AC-2.29 |
| `editor.inlineSuggest.enabled` | AC-2.33, AC-6.26 |
| `editor.inlineSuggest.experimental.showOnSuggestConflict` | AC-2.35 |
| `editor.inlineSuggest.experimental.suppressInlineSuggestions` | AC-2.33 |
| `editor.inlineSuggest.fontFamily` | AC-2.6 |
| `editor.inlineSuggest.minShowDelay` | AC-2.34 |
| `editor.inlineSuggest.showToolbar` | AC-2.8 |
| `editor.inlineSuggest.suppressInSnippetMode` | AC-2.34, AC-2.48 |
| `editor.inlineSuggest.suppressSuggestions` | AC-1.5, AC-2.35 |
| `editor.inlineSuggest.syntaxHighlightingEnabled` | AC-2.6 |
| `editor.insertSpaces` | ED-2.18, SS-3.4, SS-3.6, SS-3.35 |
| `editor.language.brackets` | ED-5.24 |
| `editor.language.colorizedBracketPairs` | ED-5.25 |
| `editor.largeFileOptimizations` | ED-5.40 |
| `editor.letterSpacing` | ED-5.38 |
| `editor.lightbulb` | ED-4.13 |
| `editor.lightbulb.enabled` | ED-4.13 |
| `editor.lineHeight` | ED-5.38, AC-7.1 |
| `editor.lineNumbers` | ED-5.20, QA-3.6 |
| `editor.lineNumbersMinChars` | ED-5.20 |
| `editor.linkedEditing` | ED-2.27 |
| `editor.links` | ED-4.28 |
| `editor.matchBrackets` | ED-2.10, ED-5.24 |
| `editor.maxTokenizationLineLength` | ED-5.42 |
| `editor.minimap` | ED-5.2 |
| `editor.minimap.autohide` | ED-5.2 |
| `editor.minimap.enabled` | ED-5.1 |
| `editor.minimap.markSectionHeaderRegex` | ED-5.3 |
| `editor.minimap.maxColumn` | ED-5.2 |
| `editor.minimap.renderCharacters` | ED-5.2 |
| `editor.minimap.scale` | ED-5.2 |
| `editor.minimap.sectionHeaderFontSize` | ED-5.3 |
| `editor.minimap.sectionHeaderLetterSpacing` | ED-5.3 |
| `editor.minimap.showMarkSectionHeaders` | ED-5.3 |
| `editor.minimap.showRegionSectionHeaders` | ED-5.3 |
| `editor.minimap.showSlider` | ED-5.2 |
| `editor.minimap.side` | ED-5.2 |
| `editor.minimap.size` | ED-5.2 |
| `editor.mouseMiddleClickAction` | ED-4.4, ED-5.27 |
| `editor.mouseWheelScrollSensitivity` | ED-5.26 |
| `editor.mouseWheelZoom` | ED-5.38 |
| `editor.multiCursorLimit` | ED-2.1 |
| `editor.multiCursorMergeOverlapping` | ED-2.1 |
| `editor.multiCursorModifier` | ED-2.2, ED-4.4, PN-3.3 |
| `editor.multiCursorPaste` | ED-2.28 |
| `editor.occurrencesHighlight` | ED-4.22, ED-5.33, ED-7.45 |
| `editor.occurrencesHighlightDelay` | ED-4.22, ED-5.33 |
| `editor.overtypeCursorStyle` | ED-2.26, ED-5.31, SS-3.23 |
| `editor.overtypeOnPaste` | ED-2.26, SS-3.23 |
| `editor.overviewRulerBorder` | ED-5.29 |
| `editor.padding` | ED-5.43 |
| `editor.padding.bottom` | ED-5.43 |
| `editor.padding.top` | ED-5.43 |
| `editor.parameterHints` | AC-3.11 |
| `editor.parameterHints.cycle` | AC-3.7, AC-3.10 |
| `editor.parameterHints.enabled` | AC-3.2, AC-3.11 |
| `editor.pasteAs` | ED-2.29 |
| `editor.pasteAs.enabled` | ED-2.29 |
| `editor.pasteAs.preferences` | ED-2.29 |
| `editor.pasteAs.showPasteSelector` | ED-2.29 |
| `editor.peekWidgetDefaultFocus` | ED-4.10 |
| `editor.quickSuggestions` | AC-1.2, AC-1.3, AC-1.5, AC-1.46, AC-2.41 |
| `editor.quickSuggestionsDelay` | AC-1.2 |
| `editor.readOnly` | ED-2.37 |
| `editor.readOnlyMessage` | ED-2.37 |
| `editor.rename.enablePreview` | ED-4.12 |
| `editor.renameOnType` | ED-2.27 |
| `editor.renderControlCharacters` | ED-5.35 |
| `editor.renderFinalNewline` | ED-5.20 |
| `editor.renderLineHighlight` | ED-5.22 |
| `editor.renderLineHighlightOnlyWhenFocus` | ED-5.22 |
| `editor.renderRichScreenReaderContent` | ED-2.42 |
| `editor.renderWhitespace` | ED-2.36, ED-5.35 |
| `editor.roundedSelection` | ED-5.44 |
| `editor.rulers` | ED-5.21 |
| `editor.screenReaderAnnounceInlineSuggestion` | AC-2.39 |
| `editor.scrollbar` | ED-5.29 |
| `editor.scrollbar.horizontal` | ED-5.29 |
| `editor.scrollbar.horizontalScrollbarSize` | ED-5.29 |
| `editor.scrollbar.ignoreHorizontalScrollbarInContentHeight` | ED-5.29 |
| `editor.scrollbar.scrollByPage` | ED-5.29 |
| `editor.scrollbar.vertical` | ED-5.29 |
| `editor.scrollbar.verticalScrollbarSize` | ED-5.29 |
| `editor.scrollBeyondLastColumn` | ED-5.28 |
| `editor.scrollBeyondLastLine` | ED-5.28 |
| `editor.scrollOnMiddleClick` | ED-5.27 |
| `editor.scrollPredominantAxis` | ED-5.26 |
| `editor.selectionClipboard` | ED-2.28 |
| `editor.selectionHighlight` | ED-2.3, ED-5.34, ED-7.45 |
| `editor.selectionHighlightMaxLength` | ED-2.3, ED-5.34 |
| `editor.selectionHighlightMultiline` | ED-2.3, ED-5.34 |
| `editor.semanticHighlighting.enabled` | ED-4.23 |
| `editor.semanticTokenColorCustomizations` | ED-4.23 |
| `editor.showDeprecated` | ED-4.25 |
| `editor.showFoldingControls` | ED-5.18 |
| `editor.showUnused` | ED-4.25 |
| `editor.simple` | ED-2.37 |
| `editor.smartSelect` | ED-2.7 |
| `editor.smartSelect.selectLeadingAndTrailingWhitespace` | ED-2.7 |
| `editor.smartSelect.selectSubwords` | ED-2.7 |
| `editor.smoothScrolling` | ED-5.26 |
| `editor.snippets.codeActions.enabled` | AC-4.15, AC-4.16 |
| `editor.snippetSuggestions` | AC-1.12, AC-4.10 |
| `editor.stablePeek` | ED-4.10, ED-4.31, ED-4.32, ED-13.4 |
| `editor.stickyScroll.defaultModel` | ED-5.5 |
| `editor.stickyScroll.enabled` | ED-5.4 |
| `editor.stickyScroll.maxLineCount` | ED-5.5 |
| `editor.stickyScroll.scrollWithEditor` | ED-5.5 |
| `editor.stickyTabStops` | ED-2.25 |
| `editor.stopRenderingLineAfter` | ED-5.42 |
| `editor.suggest` | AC-1.14 |
| `editor.suggest.filteredTypes` | AC-1.14 |
| `editor.suggest.filterGraceful` | AC-1.11 |
| `editor.suggest.insertMode` | AC-1.34, AC-1.44 |
| `editor.suggest.localityBonus` | AC-1.11 |
| `editor.suggest.matchOnWordStartOnly` | AC-1.11 |
| `editor.suggest.maxVisibleSuggestions` | AC-1.19 |
| `editor.suggest.preview` | AC-1.17, AC-1.44, AC-1.45, AC-2.7, AC-2.44 |
| `editor.suggest.selectionMode` | AC-1.16, AC-1.30 |
| `editor.suggest.shareSuggestSelections` | AC-1.15 |
| `editor.suggest.showClasses` | AC-1.14 |
| `editor.suggest.showColors` | AC-1.14 |
| `editor.suggest.showConstants` | AC-1.14 |
| `editor.suggest.showConstructors` | AC-1.14 |
| `editor.suggest.showCustomcolors` | AC-1.14 |
| `editor.suggest.showDeprecated` | AC-1.10 |
| `editor.suggest.showEnumMembers` | AC-1.14 |
| `editor.suggest.showEnums` | AC-1.14 |
| `editor.suggest.showEvents` | AC-1.14 |
| `editor.suggest.showFields` | AC-1.14 |
| `editor.suggest.showFiles` | AC-1.14 |
| `editor.suggest.showFolders` | AC-1.14 |
| `editor.suggest.showFunctions` | AC-1.14 |
| `editor.suggest.showIcons` | AC-1.7, AC-5.4 |
| `editor.suggest.showInlineDetails` | AC-1.8 |
| `editor.suggest.showInterfaces` | AC-1.14 |
| `editor.suggest.showIssues` | AC-1.14 |
| `editor.suggest.showKeywords` | AC-1.14 |
| `editor.suggest.showMethods` | AC-1.14 |
| `editor.suggest.showModules` | AC-1.14 |
| `editor.suggest.showOperators` | AC-1.14 |
| `editor.suggest.showProperties` | AC-1.14 |
| `editor.suggest.showReferences` | AC-1.14 |
| `editor.suggest.showSnippets` | AC-1.12, AC-4.10 |
| `editor.suggest.showStatusBar` | AC-1.28, AC-1.29 |
| `editor.suggest.showStructs` | AC-1.14 |
| `editor.suggest.showTypeParameters` | AC-1.14 |
| `editor.suggest.showUnits` | AC-1.14 |
| `editor.suggest.showUsers` | AC-1.14 |
| `editor.suggest.showValues` | AC-1.14 |
| `editor.suggest.showVariables` | AC-1.14 |
| `editor.suggest.showWords` | AC-1.13 |
| `editor.suggest.snippetsPreventQuickSuggestions` | AC-1.2 |
| `editor.suggestFontSize` | AC-1.18 |
| `editor.suggestLineHeight` | AC-1.18 |
| `editor.suggestOnTriggerCharacters` | AC-1.4 |
| `editor.suggestSelection` | AC-1.15 |
| `editor.tabCompletion` | AC-1.39, AC-4.9 |
| `editor.tabFocusMode` | SS-3.24 |
| `editor.tabSize` | ED-2.18, AC-6.29, SS-3.4, SS-3.6, SS-3.7, SS-3.35 |
| `editor.tokenColorCustomizations` | ED-5.46 |
| `editor.trimAutoWhitespace` | ED-2.25 |
| `editor.trimWhitespaceOnDelete` | ED-2.25 |
| `editor.unfoldOnClickAfterEndOfLine` | ED-5.18 |
| `editor.unicodeHighlight` | ED-5.37 |
| `editor.unicodeHighlight.allowedCharacters` | ED-5.37 |
| `editor.unicodeHighlight.allowedLocales` | ED-5.37 |
| `editor.unicodeHighlight.ambiguousCharacters` | ED-5.37, ED-5.41 |
| `editor.unicodeHighlight.includeComments` | ED-5.37 |
| `editor.unicodeHighlight.includeStrings` | ED-5.37 |
| `editor.unicodeHighlight.invisibleCharacters` | ED-5.37, ED-5.41 |
| `editor.unicodeHighlight.nonBasicASCII` | ED-5.37, ED-5.41 |
| `editor.unusualLineTerminators` | DL-1.27 |
| `editor.useTabStops` | ED-2.25 |
| `editor.wordBasedSuggestions` | AC-1.5, AC-1.13 |
| `editor.wordBreak` | ED-2.35, ED-5.36 |
| `editor.wordSegmenterLocales` | ED-2.9 |
| `editor.wordSeparators` | ED-2.6, ED-2.9 |
| `editor.wordWrap` | ED-2.35, ED-5.36, AC-6.4 |
| `editor.wordWrapColumn` | ED-2.35, ED-5.36 |
| `editor.wordWrapOverride1` | ED-6.1 |
| `editor.wordWrapOverride2` | ED-5.36 |
| `editor.wrapOnEscapedLineFeeds` | ED-5.36 |
| `editor.wrappingIndent` | ED-2.35, ED-5.36 |
| `editor.wrappingStrategy` | ED-2.35, ED-5.36 |
| `emmet.excludeLanguages` | ED-2.32 |
| `emmet.extensionsPath` | ED-2.32 |
| `emmet.includeLanguages` | ED-2.32 |
| `emmet.preferences` | ED-2.32 |
| `emmet.syntaxProfiles` | ED-2.32 |
| `emmet.triggerExpansionOnTab` | ED-2.32 |
| `emmet.variables` | ED-2.32 |
| `explorer.autoOpenDroppedFile` | SB-2.12, DL-4.10 |
| `explorer.autoReveal` | SB-2.14 |
| `explorer.autoRevealExclude` | SB-2.14 |
| `explorer.compactFolders` | SB-2.21 |
| `explorer.confirmDelete` | SB-2.8, DL-4.1, DL-4.2 |
| `explorer.confirmDragAndDrop` | SB-2.11, DL-4.6, DL-4.7 |
| `explorer.confirmPasteNative` | SB-2.9, DL-4.12 |
| `explorer.confirmUndo` | SB-2.10, DL-4.13 |
| `explorer.copyPathSeparator` | SB-2.16 |
| `explorer.copyRelativePathSeparator` | SB-2.16 |
| `explorer.decorations.badges` | SB-2.25 |
| `explorer.decorations.colors` | SB-2.25, SB-4.53 |
| `explorer.enableDragAndDrop` | SB-2.11, SB-2.48, DL-4.6 |
| `explorer.enableUndo` | SB-2.10, DL-4.13 |
| `explorer.excludeGitIgnore` | SB-2.24, SB-2.26 |
| `explorer.expandSingleFolderWorkspaces` | SB-2.19 |
| `explorer.fileNesting.enabled` | SB-2.22 |
| `explorer.fileNesting.expand` | SB-2.22 |
| `explorer.fileNesting.patterns` | SB-2.22 |
| `explorer.incrementalNaming` | SB-2.9, SB-2.11, DL-4.9 |
| `explorer.openEditors.minVisible` | SB-2.30 |
| `explorer.openEditors.sortOrder` | SB-2.33, SB-2.35 |
| `explorer.openEditors.visible` | SB-2.30 |
| `explorer.sortOrder` | SB-2.23 |
| `explorer.sortOrderLexicographicOptions` | SB-2.23 |
| `explorer.sortOrderReverse` | SB-2.23 |
| `extensions.experimental.issueQuickAccess` | QA-4.19 |
| `files.associations` | SS-3.16, SS-3.21 |
| `files.autoGuessEncoding` | SS-3.12 |
| `files.autoSave` | SB-2.32, SB-2.34, DL-2.3, DL-2.4, WB-1.6 |
| `files.autoSaveDelay` | WB-1.6 |
| `files.autoSaveWhenNoErrors` | WB-1.6 |
| `files.autoSaveWorkspaceFilesOnly` | WB-1.6 |
| `files.candidateGuessEncodings` | SS-3.12 |
| `files.defaultLanguage` | WB-1.5 |
| `files.dialog.defaultPath` | DL-3.20 |
| `files.enableTrash` | SB-2.8, DL-4.1, DL-4.2, DL-4.3 |
| `files.encoding` | SS-3.10, SS-3.12, SS-3.13 |
| `files.eol` | SS-3.14, SS-3.15 |
| `files.exclude` | SB-2.24, SB-3.11, PN-4.8, QA-2.1 |
| `files.hotExit` | DL-2.5, DL-2.6, DL-2.7, DL-2.9 |
| `files.insertFinalNewline` | ED-2.15, WB-1.4 |
| `files.readonlyExclude` | ED-2.37, DL-2.16, DL-4.5 |
| `files.readonlyFromPermissions` | ED-2.37, DL-2.16, DL-4.5 |
| `files.readonlyInclude` | ED-2.37, DL-2.16, DL-4.5 |
| `files.restoreUndoStack` | ED-2.43 |
| `files.saveConflictResolution` | ED-6.45 |
| `files.simpleDialog.enable` | DL-3.1, DL-3.2, DL-3.4, DL-3.6, DL-3.7, DL-3.9, DL-3.11, DL-3.17, DL-3.18 |
| `files.trimFinalNewlines` | ED-2.15, WB-1.4 |
| `files.trimTrailingWhitespace` | ED-2.15, WB-1.4 |
| `files.trimTrailingWhitespaceInRegexAndStrings` | ED-2.15 |
| `files.watcherExclude` | SB-2.46 |
| `files.watcherInclude` | SB-2.46 |
| `git.allowForcePush` | SB-4.24 |
| `git.allowNoVerifyCommit` | SB-4.9, SB-4.10 |
| `git.alwaysShowStagedChangesResourceGroup` | SB-4.1 |
| `git.alwaysSignOff` | SB-4.9 |
| `git.autofetch` | SB-4.22 |
| `git.autofetchPeriod` | SB-4.22 |
| `git.autorefresh` | SB-4.1, SB-4.37 |
| `git.autoRepositoryDetection` | SB-4.29 |
| `git.autoStash` | SB-4.23 |
| `git.blame.editorDecoration.disableHover` | ED-6.27 |
| `git.blame.editorDecoration.enabled` | ED-6.27 |
| `git.blame.editorDecoration.template` | ED-6.27 |
| `git.blame.ignoreWhitespace` | ED-6.27, SS-4.14 |
| `git.blame.statusBarItem.enabled` | SS-4.14, SS-4.16 |
| `git.blame.statusBarItem.template` | SS-4.14, SS-4.16 |
| `git.branchPrefix` | SB-4.16 |
| `git.branchProtection` | SB-4.7, SS-4.9 |
| `git.branchProtectionPrompt` | SB-4.7 |
| `git.branchRandomName.dictionary` | SB-4.16 |
| `git.branchRandomName.enable` | SB-4.16 |
| `git.branchSortOrder` | SB-4.15, SS-4.11 |
| `git.branchValidationRegex` | SB-4.16 |
| `git.branchWhitespaceChar` | SB-4.16 |
| `git.checkoutType` | SB-4.15, SS-4.11 |
| `git.closeDiffOnOperation` | SB-4.4, SB-4.20, SB-4.43 |
| `git.commandsToLog` | SB-4.37 |
| `git.commitShortHashLength` | SS-4.14, SS-4.15 |
| `git.confirmCommittedDelete` | SB-4.42 |
| `git.confirmEmptyCommits` | SB-4.9 |
| `git.confirmForcePush` | SB-4.24 |
| `git.confirmNoVerifyCommit` | SB-4.9 |
| `git.confirmSync` | SB-4.25, SS-4.12 |
| `git.countBadge` | AB-3.8 |
| `git.decorations.enabled` | SB-2.25, SB-4.53 |
| `git.defaultBranchName` | SB-4.16 |
| `git.defaultCloneDirectory` | SB-4.27, SB-4.45 |
| `git.detectSubmodules` | SB-4.29 |
| `git.detectSubmodulesLimit` | SB-4.29 |
| `git.detectWorktrees` | SB-4.32 |
| `git.detectWorktreesLimit` | SB-4.32 |
| `git.diagnosticsCommitHook.enabled` | SB-4.7 |
| `git.diagnosticsCommitHook.sources` | SB-4.7 |
| `git.discardUntrackedChangesToTrash` | SB-4.3 |
| `git.enableCommitSigning` | SB-4.9 |
| `git.enabled` | SB-2.37, SB-4.40 |
| `git.enableSmartCommit` | SB-4.7 |
| `git.enableStatusBarSync` | SS-4.12, SS-4.13 |
| `git.fetchOnPull` | SB-4.22, SB-4.23 |
| `git.followTagsWhenSync` | SB-4.19, SB-4.25 |
| `git.githubAuthentication` | SB-4.44 |
| `git.ignoredRepositories` | SB-4.29 |
| `git.ignoreLegacyWarning` | SB-4.47 |
| `git.ignoreLimitWarning` | SB-4.1 |
| `git.ignoreMissingGitWarning` | SB-4.40, SB-4.47 |
| `git.ignoreRebaseWarning` | SB-4.23 |
| `git.ignoreSubmodules` | SB-4.1 |
| `git.ignoreWindowsGit27Warning` | SB-4.47 |
| `git.inputValidation` | SB-4.7 |
| `git.inputValidationLength` | SB-4.7 |
| `git.inputValidationSubjectLength` | SB-4.7 |
| `git.mergeEditor` | ED-6.17 |
| `git.openAfterClone` | SB-4.27, SB-4.45 |
| `git.openDiffOnClick` | SB-4.4 |
| `git.openRepositoryInParentFolders` | SB-4.46 |
| `git.optimisticUpdate` | SB-4.2 |
| `git.path` | SB-4.40, SB-4.47 |
| `git.postCommitCommand` | SB-4.8 |
| `git.promptToSaveFilesBeforeCommit` | SB-4.7 |
| `git.promptToSaveFilesBeforeStash` | SB-4.20 |
| `git.pruneOnFetch` | SB-4.22 |
| `git.pullBeforeCheckout` | SB-4.15, SS-4.11 |
| `git.pullTags` | SB-4.19, SB-4.23 |
| `git.rebaseWhenSync` | SB-4.25 |
| `git.rememberPostCommitCommand` | SB-4.8 |
| `git.replaceTagsWhenPull` | SB-4.19, SB-4.23 |
| `git.repositoryScanIgnoredFolders` | SB-4.29 |
| `git.repositoryScanMaxDepth` | SB-4.29 |
| `git.requireGitUserConfig` | SB-4.7 |
| `git.scanRepositories` | SB-4.29 |
| `git.showActionButton` | SB-4.8, SB-4.26 |
| `git.showCommitInput` | SB-4.7 |
| `git.showCursorWorktrees` | SB-4.33 |
| `git.showInlineOpenFileAction` | SB-4.4 |
| `git.showProgress` | SB-4.37 |
| `git.showPushSuccessNotification` | SB-4.24 |
| `git.showReferenceDetails` | SB-4.15, SS-4.11 |
| `git.similarityThreshold` | SB-4.1 |
| `git.smartCommitChanges` | SB-4.7 |
| `git.statusLimit` | SB-4.1 |
| `git.suggestSmartCommit` | SB-4.7 |
| `git.supportCancellation` | SB-4.25 |
| `git.terminalAuthentication` | SB-4.44 |
| `git.terminalGitEditor` | SB-4.12 |
| `git.timeline.date` | SB-7.16 |
| `git.timeline.showAuthor` | SB-7.16 |
| `git.timeline.showUncommitted` | SB-7.16 |
| `git.untrackedChanges` | SB-4.1, SB-4.2, SB-4.3 |
| `git.useCommitInputAsStashMessage` | SB-4.20 |
| `git.useEditorAsCommitInput` | SB-4.12 |
| `git.useForcePushIfIncludes` | SB-4.24 |
| `git.useForcePushWithLease` | SB-4.24 |
| `git.useIntegratedAskPass` | SB-4.44 |
| `git.verboseCommit` | SB-4.12 |
| `git.worktreeIncludeFiles` | SB-4.32 |
| `github.gitAuthentication` | SB-4.44 |
| `http.noProxy` | ED-8.70 |
| `http.proxy` | ED-8.70 |
| `http.proxyAuthorization` | ED-8.70 |
| `http.proxyKerberosServicePrincipal` | ED-8.70 |
| `http.proxyStrictSSL` | ED-8.70 |
| `http.systemCertificates` | ED-8.70 |
| `http.systemCertificatesNode` | ED-8.70 |
| `imageCarousel.chat.enabled` | ED-10.44 |
| `imageCarousel.explorerContextMenu.enabled` | SB-2.40, ED-10.44 |
| `interactiveWindow.alwaysScrollOnNewCell` | ED-7.38, ED-7.39 |
| `interactiveWindow.collapseCellInputCode` | ED-7.38 |
| `interactiveWindow.executeWithShiftEnter` | ED-7.37, ED-7.39 |
| `interactiveWindow.promptToSaveOnClose` | ED-7.38 |
| `interactiveWindow.showExecutionHint` | ED-7.37, ED-7.39 |
| `ipynb.pasteImagesAsAttachments.enabled` | ED-7.25 |
| `issueReporter.wizard.enabled` | WB-6.5 |
| `issueReporter.wizard.fullWorkspaceScan` | WB-6.5 |
| `it` | PN-3.46 |
| `js/ts.implicitProjectConfig.checkJs` | SS-3.45, SS-3.47 |
| `js/ts.implicitProjectConfig.experimentalDecorators` | SS-3.45, SS-3.47 |
| `js/ts.implicitProjectConfig.module` | SS-3.45, SS-3.47 |
| `js/ts.implicitProjectConfig.strict` | SS-3.45, SS-3.47 |
| `js/ts.implicitProjectConfig.strictFunctionTypes` | SS-3.45, SS-3.47 |
| `js/ts.implicitProjectConfig.strictNullChecks` | SS-3.45, SS-3.47 |
| `js/ts.implicitProjectConfig.target` | SS-3.45, SS-3.47 |
| `js/ts.tsdk.path` | SS-3.44 |
| `js/ts.tsserver.useSyntaxServer` | SS-3.46 |
| `json.maxItemsComputed` | SS-3.50 |
| `json.schemaDownload.enable` | SS-3.51, SS-3.52 |
| `json.schemaDownload.trustedDomains` | SS-3.51 |
| `json.schemas` | SS-3.48, SS-3.49 |
| `json.validate.enable` | SS-3.48 |
| `keyboard.dispatch` | ED-8.71 |
| `keyboard.layout` | SS-3.38 |
| `keyboard.mapAltGrToCtrlAlt` | ED-8.71 |
| `keyboard.touchbar.enabled` | WB-9.6 |
| `keyboard.touchbar.ignored` | WB-9.6 |
| `launch` | SB-5.5, SB-5.7 |
| `markdown-mermaid.controls.show` | ED-10.22 |
| `markdown-mermaid.darkModeTheme` | ED-10.22 |
| `markdown-mermaid.languages` | ED-10.22 |
| `markdown-mermaid.lightModeTheme` | ED-10.22 |
| `markdown-mermaid.maxHeight` | ED-10.22 |
| `markdown-mermaid.maxTextSize` | ED-10.22 |
| `markdown-mermaid.mouseNavigation.enabled` | ED-10.22 |
| `markdown-mermaid.resizable` | ED-10.22 |
| `markdown.editor.highlightActiveBlock` | ED-10.21 |
| `markdown.editor.quickSuggestions` | ED-10.21 |
| `markdown.experimental.richLinks.enabled` | ED-10.21 |
| `markdown.links.openLocation` | ED-10.13 |
| `markdown.math.enabled` | ED-10.43 |
| `markdown.math.macros` | ED-10.43 |
| `markdown.preview.breaks` | ED-10.14 |
| `markdown.preview.doubleClickToSwitchToEditor` | ED-10.13 |
| `markdown.preview.fontFamily` | ED-10.14 |
| `markdown.preview.fontSize` | ED-10.14 |
| `markdown.preview.frontMatter` | ED-10.17 |
| `markdown.preview.lineHeight` | ED-10.14 |
| `markdown.preview.linkify` | ED-10.14 |
| `markdown.preview.markEditorSelection` | ED-10.13 |
| `markdown.preview.openMarkdownLinks` | ED-10.13 |
| `markdown.preview.scrollEditorWithPreview` | ED-10.13 |
| `markdown.preview.scrollPreviewWithEditor` | ED-10.13 |
| `markdown.preview.typographer` | ED-10.14 |
| `markdown.styles` | ED-10.14 |
| `markdown.suggest.paths.enabled` | AC-1.46 |
| `markdown.suggest.paths.includeWorkspaceHeaderCompletions` | AC-1.46 |
| `markdown.validate.enabled` | SS-3.53 |
| `mediaPreview.video.autoPlay` | ED-10.5 |
| `mediaPreview.video.loop` | ED-10.5 |
| `merge-conflict.autoNavigateNextConflict.enabled` | ED-6.23 |
| `merge-conflict.codeLens.enabled` | ED-6.23 |
| `merge-conflict.decorators.enabled` | ED-6.23 |
| `merge-conflict.diffViewPosition` | ED-6.23 |
| `mergeEditor.diffAlgorithm` | ED-6.18 |
| `mergeEditor.showCheckboxes` | ED-6.50 |
| `mergeEditor.showDeletionMarkers` | ED-6.18 |
| `multiDiffEditor.experimental.fileList` | ED-6.16 |
| `network.meteredConnection` | SS-3.37 |
| `notebook.cellExecutionTimeVerbosity` | ED-7.16 |
| `notebook.cellFailureDiagnostics` | ED-7.17 |
| `notebook.cellFocusIndicator` | ED-7.28 |
| `notebook.cellToolbarLocation` | ED-7.27 |
| `notebook.cellToolbarVisibility` | ED-7.27 |
| `notebook.codeActionsOnSave` | ED-7.33 |
| `notebook.compactView` | ED-7.28 |
| `notebook.confirmDeleteRunningCell` | ED-7.3 |
| `notebook.consolidatedOutputButton` | ED-7.21, ED-7.23 |
| `notebook.consolidatedRunButton` | ED-7.13, ED-7.14, ED-7.27 |
| `notebook.defaultFormatter` | ED-7.33 |
| `notebook.diff.enablePreview` | ED-7.34 |
| `notebook.diff.experimental.toggleInline` | ED-7.35 |
| `notebook.diff.ignoreMetadata` | ED-7.35 |
| `notebook.diff.ignoreOutputs` | ED-7.35 |
| `notebook.diff.overviewRuler` | ED-7.34 |
| `notebook.displayOrder` | ED-7.21 |
| `notebook.dragAndDropEnabled` | ED-7.4 |
| `notebook.editorOptionsCustomizations` | ED-7.28, SS-3.35 |
| `notebook.experimental.enableNewDiffEditor` | ED-7.34 |
| `notebook.experimental.openGettingStarted` | ED-7.28 |
| `notebook.find.filters` | ED-7.30 |
| `notebook.formatOnCellExecution` | ED-7.33 |
| `notebook.formatOnSave.enabled` | ED-7.33 |
| `notebook.globalToolbar` | ED-7.14, ED-7.15, ED-7.26, SS-3.33 |
| `notebook.globalToolbarShowLabel` | ED-7.26 |
| `notebook.gotoSymbols.showAllSymbols` | QA-3.14 |
| `notebook.inlineValues` | ED-7.18 |
| `notebook.insertFinalNewline` | ED-7.33 |
| `notebook.insertToolbarLocation` | ED-7.2, ED-7.26 |
| `notebook.lineNumbers` | ED-7.28 |
| `notebook.markdown.lineHeight` | ED-7.28, ED-7.46 |
| `notebook.markup.fontFamily` | ED-7.28, ED-7.46 |
| `notebook.markup.fontSize` | ED-7.28, ED-7.46 |
| `notebook.multiCursor.enabled` | ED-7.32 |
| `notebook.navigation.allowNavigateToSurroundingCells` | ED-7.9 |
| `notebook.outline.showCodeCells` | SB-7.6, ED-7.31 |
| `notebook.outline.showCodeCellSymbols` | SB-7.6, ED-7.31 |
| `notebook.outline.showMarkdownHeadersOnly` | SB-7.6, ED-7.31 |
| `notebook.output.fontFamily` | ED-7.20 |
| `notebook.output.fontSize` | ED-7.20 |
| `notebook.output.lineHeight` | ED-7.20 |
| `notebook.output.linkifyFilePaths` | ED-7.20 |
| `notebook.output.minimalErrorRendering` | ED-7.20 |
| `notebook.output.openInPreviewEditor.enabled` | ED-7.21 |
| `notebook.output.scrolling` | ED-7.22 |
| `notebook.output.textLineLimit` | ED-7.20, ED-7.22 |
| `notebook.output.wordWrap` | ED-7.20, ED-7.22 |
| `notebook.scrolling.revealNextCellOnExecute` | ED-7.13 |
| `notebook.showCellStatusBar` | ED-7.16 |
| `notebook.showFoldingControls` | ED-7.12 |
| `notebook.stickyScroll.enabled` | ED-7.29 |
| `notebook.stickyScroll.mode` | ED-7.29 |
| `notebook.undoRedoPerCell` | ED-7.41 |
| `notebook.variablesView` | SB-5.16, ED-7.18 |
| `outline.collapseItems` | SB-7.1, SB-7.5 |
| `outline.icons` | SB-7.5 |
| `outline.problems.badges` | SB-7.5 |
| `outline.problems.colors` | SB-7.5 |
| `outline.problems.enabled` | SB-7.5 |
| `outline.showArrays` | SB-7.5 |
| `outline.showBooleans` | SB-7.5 |
| `outline.showClasses` | SB-7.5 |
| `outline.showConstants` | SB-7.5 |
| `outline.showConstructors` | SB-7.5 |
| `outline.showEnumMembers` | SB-7.5 |
| `outline.showEnums` | SB-7.5 |
| `outline.showEvents` | SB-7.5 |
| `outline.showFields` | SB-7.5 |
| `outline.showFiles` | SB-7.5 |
| `outline.showFunctions` | SB-7.5 |
| `outline.showInterfaces` | SB-7.5 |
| `outline.showKeys` | SB-7.5 |
| `outline.showMethods` | SB-7.5 |
| `outline.showModules` | SB-7.5 |
| `outline.showNamespaces` | SB-7.5 |
| `outline.showNull` | SB-7.5 |
| `outline.showNumbers` | SB-7.5 |
| `outline.showObjects` | SB-7.5 |
| `outline.showOperators` | SB-7.5 |
| `outline.showPackages` | SB-7.5 |
| `outline.showProperties` | SB-7.5 |
| `outline.showStrings` | SB-7.5 |
| `outline.showStructs` | SB-7.5 |
| `outline.showTypeParameters` | SB-7.5 |
| `outline.showVariables` | SB-7.5 |
| `output.smartScroll.enabled` | PN-5.8 |
| `problems.autoReveal` | PN-4.12 |
| `problems.decorations.enabled` | SB-2.25 |
| `problems.defaultViewMode` | PN-4.2, PN-4.3, PN-4.4 |
| `problems.showCurrentInStatus` | SS-5.8 |
| `problems.sortOrder` | ED-4.42 |
| `problems.visibility` | PN-4.19, SS-5.7 |
| `references.preferredLocation` | SB-7.20 |
| `remote.autoForwardPorts` | PN-7.19 |
| `remote.autoForwardPortsFallback` | PN-7.19 |
| `remote.autoForwardPortsSource` | PN-7.19 |
| `remote.forwardOnOpen` | PN-7.19 |
| `remote.localPortHost` | PN-7.19 |
| `remote.otherPortsAttributes` | PN-7.20 |
| `remote.portsAttributes` | PN-7.20 |
| `remote.restoreForwardedPorts` | PN-7.19 |
| `scm.alwaysShowActions` | SB-4.1 |
| `scm.alwaysShowRepositories` | SB-4.28 |
| `scm.autoReveal` | SB-4.4 |
| `scm.compactFolders` | SB-4.5, SB-4.49 |
| `scm.countBadge` | AB-3.8 |
| `scm.defaultViewMode` | SB-4.5 |
| `scm.defaultViewSortKey` | SB-4.5 |
| `scm.diffDecorations` | ED-6.24 |
| `scm.diffDecorationsGutterAction` | ED-6.25 |
| `scm.diffDecorationsGutterPattern` | ED-6.24 |
| `scm.diffDecorationsGutterVisibility` | ED-6.24 |
| `scm.diffDecorationsGutterWidth` | ED-6.24 |
| `scm.diffDecorationsIgnoreTrimWhitespace` | ED-6.24 |
| `scm.graph.badges` | SB-4.34 |
| `scm.graph.pageOnScroll` | SB-4.34 |
| `scm.graph.pageSize` | SB-4.34 |
| `scm.graph.showIncomingChanges` | SB-4.35 |
| `scm.graph.showOutgoingChanges` | SB-4.35 |
| `scm.inputFontFamily` | SB-4.7 |
| `scm.inputFontSize` | SB-4.7 |
| `scm.inputMaxLineCount` | SB-4.7 |
| `scm.inputMinLineCount` | SB-4.7 |
| `scm.maxRenderedResources` | SB-4.1 |
| `scm.providerCountBadge` | SB-4.28 |
| `scm.repositories.explorer` | SB-4.30 |
| `scm.repositories.selectionMode` | SB-4.30 |
| `scm.repositories.sortOrder` | SB-4.28 |
| `scm.repositories.visible` | SB-4.28 |
| `scm.showActionButton` | SB-4.8 |
| `scm.showInputActionButton` | SB-4.14 |
| `scm.workingSets.default` | SB-4.31 |
| `scm.workingSets.enabled` | SB-4.31 |
| `screencastMode.fontSize` | WB-8.7 |
| `screencastMode.keyboardOptions` | WB-8.7 |
| `screencastMode.keyboardOverlayTimeout` | WB-8.7 |
| `screencastMode.mouseIndicatorColor` | WB-8.7 |
| `screencastMode.mouseIndicatorSize` | WB-8.7 |
| `screencastMode.verticalOffset` | WB-8.7 |
| `search.actionsPosition` | SB-3.17 |
| `search.collapseResults` | SB-3.19 |
| `search.decorations.badges` | SB-3.17 |
| `search.decorations.colors` | SB-3.17 |
| `search.defaultViewMode` | SB-3.18 |
| `search.exclude` | SB-2.26, SB-3.11, QA-2.1 |
| `search.experimental.closedNotebookRichContentResults` | SB-3.4 |
| `search.followSymlinks` | SB-3.11, QA-2.1 |
| `search.globalFindClipboard` | SB-3.1, ED-3.27 |
| `search.location` | SB-1.12 |
| `search.maxResults` | SB-3.30, QA-2.14 |
| `search.mode` | SB-3.14, SB-3.29 |
| `search.quickAccess.excludeSymbolProviders` | QA-2.13 |
| `search.quickAccess.preserveInput` | QA-2.17 |
| `search.quickOpen.history.filterSortOrder` | QA-2.2 |
| `search.quickOpen.includeHistory` | QA-2.2 |
| `search.quickOpen.includeSymbols` | QA-2.11 |
| `search.quickOpen.useVscodeSearch` | QA-2.20, QA-2.21 |
| `search.ripgrep.maxThreads` | SB-3.2 |
| `search.searchEditor.defaultNumberOfContextLines` | ED-3.29, ED-3.34 |
| `search.searchEditor.doubleClickBehaviour` | ED-3.36 |
| `search.searchEditor.focusResultsOnSearch` | ED-3.29, ED-3.36 |
| `search.searchEditor.reusePriorSearchConfiguration` | ED-3.29 |
| `search.searchEditor.singleClickBehaviour` | ED-3.36 |
| `search.searchOnType` | SB-3.2, ED-3.29 |
| `search.searchOnTypeDebouncePeriod` | SB-3.2 |
| `search.seedOnFocus` | SB-3.1 |
| `search.seedWithNearestWord` | SB-3.1, ED-3.29 |
| `search.showLineNumbers` | SB-3.17 |
| `search.smartCase` | SB-3.3, QA-2.14 |
| `search.sortOrder` | SB-3.20, QA-2.14 |
| `search.useGlobalIgnoreFiles` | SB-3.11, QA-2.1 |
| `search.useIgnoreFiles` | SB-3.11, QA-2.1 |
| `search.useParentIgnoreFiles` | SB-3.11, QA-2.1 |
| `search.usePCRE2` | SB-3.3 |
| `search.useReplacePreview` | SB-3.9 |
| `search.useRipgrep` | SB-3.3 |
| `security.allowedUNCHosts` | DL-5.16 |
| `security.promptForLocalFileProtocolHandling` | DL-5.15 |
| `security.promptForRemoteFileProtocolHandling` | DL-5.15 |
| `security.restrictUNCAccess` | DL-1.38, DL-5.16 |
| `security.workspace.trust.banner` | BN-2.2, ED-8.18 |
| `security.workspace.trust.emptyWindow` | ED-8.18, DL-5.4, DL-5.8 |
| `security.workspace.trust.enabled` | BN-2.1, ED-8.18, ED-8.46, ED-10.46, SS-4.5, DL-1.38, DL-5.1, DL-5.5, DL-5.6, DL-5.7, DL-5.9, DL-5.10 |
| `security.workspace.trust.startupPrompt` | BN-2.1, ED-8.18, DL-5.1, DL-5.3 |
| `security.workspace.trust.untrustedFiles` | ED-8.18, DL-5.4, DL-5.8 |
| `settingsSync.ignoredExtensions` | AB-4.7 |
| `settingsSync.ignoredSettings` | AB-4.7, ED-8.50 |
| `settingsSync.keybindingsPerPlatform` | AB-4.7 |
| `simpleBrowser.focusLockIndicator.enabled` | ED-10.26 |
| `task.allowAutomaticTasks` | WB-2.1 |
| `task.problemMatchers.neverPrompt` | WB-2.5 |
| `task.quickOpen.detail` | QA-4.8, QA-4.9 |
| `task.quickOpen.history` | QA-4.8, QA-4.9, QA-4.10 |
| `task.quickOpen.showAll` | QA-4.8 |
| `task.quickOpen.skip` | QA-4.8, WB-2.6 |
| `task.reconnection` | PN-2.33 |
| `telemetry.editStats.enabled` | SS-6.5 |
| `telemetry.editStats.showDecorations` | SS-6.5 |
| `telemetry.editStats.showStatusBar` | SS-6.5 |
| `telemetry.enableCrashReporter` | ED-8.42 |
| `telemetry.feedback.enabled` | QA-4.19, DL-1.38, WB-6.5 |
| `terminal.explorerKind` | SB-2.18 |
| `terminal.external.linuxExec` | PN-2.41 |
| `terminal.external.osxExec` | PN-2.41 |
| `terminal.external.windowsExec` | PN-2.41 |
| `terminal.integrated.accessibleViewFocusOnCommandExecution` | PN-3.22 |
| `terminal.integrated.accessibleViewPreserveCursorPosition` | PN-3.22 |
| `terminal.integrated.allowChords` | PN-3.36 |
| `terminal.integrated.allowedLinkSchemes` | PN-3.3 |
| `terminal.integrated.allowInUntrustedWorkspace` | PN-2.9 |
| `terminal.integrated.allowMnemonics` | PN-3.36 |
| `terminal.integrated.altClickMovesCursor` | PN-3.18 |
| `terminal.integrated.automationProfile.linux` | PN-2.43, PN-2.44 |
| `terminal.integrated.automationProfile.osx` | PN-2.43, PN-2.44 |
| `terminal.integrated.automationProfile.windows` | PN-2.43, PN-2.44 |
| `terminal.integrated.autoReplies` | PN-3.44 |
| `terminal.integrated.bellDuration` | PN-3.20 |
| `terminal.integrated.commandsToSkipShell` | PN-3.36 |
| `terminal.integrated.confirmOnExit` | PN-2.30 |
| `terminal.integrated.confirmOnKill` | PN-2.29 |
| `terminal.integrated.copyOnSelection` | PN-3.15 |
| `terminal.integrated.cursorBlinking` | PN-3.26 |
| `terminal.integrated.cursorStyle` | PN-3.26 |
| `terminal.integrated.cursorStyleInactive` | PN-3.26 |
| `terminal.integrated.cursorWidth` | PN-3.26 |
| `terminal.integrated.customGlyphs` | PN-3.25 |
| `terminal.integrated.cwd` | PN-2.8 |
| `terminal.integrated.defaultLocation` | PN-2.2, PN-2.7 |
| `terminal.integrated.defaultProfile.linux` | PN-2.4, PN-2.5 |
| `terminal.integrated.defaultProfile.osx` | PN-2.4, PN-2.5 |
| `terminal.integrated.defaultProfile.windows` | PN-2.4, PN-2.5 |
| `terminal.integrated.detectLocale` | PN-2.42 |
| `terminal.integrated.developer.devMode` | PN-3.40 |
| `terminal.integrated.drawBoldTextInBrightColors` | PN-3.27 |
| `terminal.integrated.enableBell` | PN-3.20 |
| `terminal.integrated.enableFileLinks` | PN-3.3 |
| `terminal.integrated.enableImages` | PN-3.29 |
| `terminal.integrated.enableKittyKeyboardProtocol` | PN-3.36 |
| `terminal.integrated.enableMultiLinePasteWarning` | PN-3.17 |
| `terminal.integrated.enableNotifications` | PN-3.21 |
| `terminal.integrated.enablePersistentSessions` | PN-2.33 |
| `terminal.integrated.enableVisualBell` | PN-2.13, PN-3.20 |
| `terminal.integrated.enableWin32InputMode` | PN-3.36 |
| `terminal.integrated.env.linux` | PN-2.42 |
| `terminal.integrated.env.osx` | PN-2.42 |
| `terminal.integrated.env.windows` | PN-2.42 |
| `terminal.integrated.environmentChangesRelaunch` | PN-2.13, PN-3.37 |
| `terminal.integrated.fastScrollSensitivity` | PN-3.33 |
| `terminal.integrated.focusAfterRun` | PN-2.10 |
| `terminal.integrated.fontFamily` | AC-5.2, PN-3.27 |
| `terminal.integrated.fontLigatures.enabled` | PN-3.28 |
| `terminal.integrated.fontLigatures.fallbackLigatures` | PN-3.28 |
| `terminal.integrated.fontLigatures.featureSettings` | PN-3.28 |
| `terminal.integrated.fontRendering` | PN-3.31 |
| `terminal.integrated.fontSize` | AC-5.2, PN-3.27, PN-3.30 |
| `terminal.integrated.fontWeight` | PN-3.27 |
| `terminal.integrated.fontWeightBold` | PN-3.27 |
| `terminal.integrated.gpuAcceleration` | PN-3.25 |
| `terminal.integrated.hideOnLastClosed` | PN-2.31 |
| `terminal.integrated.hideOnStartup` | PN-2.31 |
| `terminal.integrated.ignoreBracketedPasteMode` | PN-3.17 |
| `terminal.integrated.ignoreProcessNames` | PN-2.29 |
| `terminal.integrated.inheritEnv` | PN-2.42 |
| `terminal.integrated.letterSpacing` | PN-3.27 |
| `terminal.integrated.lineHeight` | AC-5.2, PN-3.27 |
| `terminal.integrated.localEchoEnabled` | PN-3.38 |
| `terminal.integrated.localEchoExcludePrograms` | PN-3.38 |
| `terminal.integrated.localEchoLatencyThreshold` | PN-3.38 |
| `terminal.integrated.localEchoStyle` | PN-3.38 |
| `terminal.integrated.macOptionClickForcesSelection` | PN-3.18 |
| `terminal.integrated.macOptionIsMeta` | PN-3.36 |
| `terminal.integrated.middleClickBehavior` | PN-3.18 |
| `terminal.integrated.minimumContrastRatio` | PN-3.27 |
| `terminal.integrated.mouseWheelScrollSensitivity` | PN-3.33 |
| `terminal.integrated.mouseWheelZoom` | PN-3.30 |
| `terminal.integrated.persistentSessionReviveProcess` | PN-2.33 |
| `terminal.integrated.persistentSessionScrollback` | PN-2.33, PN-2.39 |
| `terminal.integrated.profiles.linux` | PN-2.3, PN-2.4, PN-2.5 |
| `terminal.integrated.profiles.osx` | PN-2.3, PN-2.4, PN-2.5 |
| `terminal.integrated.profiles.windows` | PN-2.3, PN-2.4, PN-2.5 |
| `terminal.integrated.rescaleOverlappingGlyphs` | PN-3.25 |
| `terminal.integrated.rightClickBehavior` | PN-3.18 |
| `terminal.integrated.scrollback` | PN-2.39 |
| `terminal.integrated.sendKeybindingsToShell` | PN-3.36 |
| `terminal.integrated.shellIntegration.decorationsEnabled` | PN-3.6 |
| `terminal.integrated.shellIntegration.enabled` | AC-5.1, PN-2.38 |
| `terminal.integrated.shellIntegration.history` | PN-3.12, PN-3.13 |
| `terminal.integrated.shellIntegration.quickFixEnabled` | PN-3.14 |
| `terminal.integrated.shellIntegration.showCommandGuide` | PN-3.10 |
| `terminal.integrated.shellIntegration.timeout` | PN-2.38 |
| `terminal.integrated.showExitAlert` | PN-2.32 |
| `terminal.integrated.showLinkHover` | PN-3.4 |
| `terminal.integrated.smoothScrolling` | PN-3.33 |
| `terminal.integrated.splitCwd` | PN-2.8 |
| `terminal.integrated.stickyScroll.enabled` | PN-3.11 |
| `terminal.integrated.stickyScroll.ignoredCommands` | PN-3.11 |
| `terminal.integrated.stickyScroll.maxLineCount` | PN-3.11 |
| `terminal.integrated.suggest.cdPath` | AC-5.21 |
| `terminal.integrated.suggest.enabled` | AC-5.1, AC-5.5 |
| `terminal.integrated.suggest.inlineSuggestion` | AC-5.23 |
| `terminal.integrated.suggest.insertTrailingSpace` | AC-5.11 |
| `terminal.integrated.suggest.providers` | AC-5.19, AC-5.20, AC-5.22, AC-5.24 |
| `terminal.integrated.suggest.quickSuggestions` | AC-5.6, AC-5.16, AC-5.20, AC-5.21 |
| `terminal.integrated.suggest.runOnEnter` | AC-5.9 |
| `terminal.integrated.suggest.selectionMode` | AC-5.10 |
| `terminal.integrated.suggest.showStatusBar` | AC-5.15 |
| `terminal.integrated.suggest.suggestOnTriggerCharacters` | AC-5.6, AC-5.16 |
| `terminal.integrated.suggest.upArrowNavigatesHistory` | AC-5.7 |
| `terminal.integrated.suggest.windowsExecutableExtensions` | AC-5.19 |
| `terminal.integrated.tabs.allowAgentCliTitle` | PN-2.11 |
| `terminal.integrated.tabs.defaultColor` | PN-2.12 |
| `terminal.integrated.tabs.defaultIcon` | PN-2.12 |
| `terminal.integrated.tabs.description` | PN-2.11 |
| `terminal.integrated.tabs.enableAnimation` | PN-2.13, PN-2.35 |
| `terminal.integrated.tabs.enabled` | PN-2.15, PN-2.17 |
| `terminal.integrated.tabs.focusMode` | PN-2.18 |
| `terminal.integrated.tabs.hideCondition` | PN-2.15, PN-2.17 |
| `terminal.integrated.tabs.location` | PN-2.16 |
| `terminal.integrated.tabs.separator` | PN-2.11 |
| `terminal.integrated.tabs.showActions` | PN-2.17 |
| `terminal.integrated.tabs.showActiveTerminal` | PN-2.17 |
| `terminal.integrated.tabs.title` | PN-2.11 |
| `terminal.integrated.tabStopWidth` | PN-3.27 |
| `terminal.integrated.textBlinking` | PN-3.26 |
| `terminal.integrated.unicodeVersion` | PN-3.27 |
| `terminal.integrated.useWslProfiles` | PN-2.3, PN-2.5 |
| `terminal.integrated.windowsEnableConpty` | PN-2.47 |
| `terminal.integrated.windowsReparentConsoleWindow` | PN-3.47 |
| `terminal.integrated.windowsUseConptyDll` | PN-2.47 |
| `terminal.integrated.wordSeparators` | PN-3.19 |
| `terminal.sourceControlRepositoriesKind` | SB-4.29 |
| `testing.alwaysRevealTestOnStateChange` | SB-6.18 |
| `testing.automaticallyOpenPeekView` | ED-13.3, ED-13.4 |
| `testing.automaticallyOpenPeekViewDuringAutoRun` | ED-13.3, ED-13.4 |
| `testing.automaticallyOpenTestResults` | SB-6.23, PN-6.19 |
| `testing.countBadge` | AB-3.8, SB-6.20 |
| `testing.coverageBarThresholds` | SB-6.24, ED-13.5 |
| `testing.coverageMinimapEnabled` | ED-13.5 |
| `testing.coverageToolbarEnabled` | ED-13.5 |
| `testing.defaultGutterClickAction` | ED-13.1 |
| `testing.displayedCoveragePercent` | SB-6.24, ED-13.5 |
| `testing.followRunningTest` | SB-6.18 |
| `testing.gutterEnabled` | ED-13.1 |
| `testing.openTesting` | PN-6.19 |
| `testing.resultsView.layout` | PN-6.31 |
| `testing.saveBeforeTest` | SB-6.6, SB-6.7, SB-6.8, SB-6.9, SB-6.29, SB-6.30 |
| `testing.showAllMessages` | ED-13.3 |
| `testing.showCoverageInExplorer` | SB-6.28 |
| `timeline.pageOnScroll` | SB-7.7 |
| `timeline.pageSize` | SB-7.7 |
| `typescript.enablePromptUseWorkspaceTsdk` | SS-3.44 |
| `typescript.tsdk` | SS-3.44 |
| `typescript.tsserver.useSyntaxServer` | SS-3.46 |
| `update.channel` | NT-6.7 |
| `update.enableWindowsBackgroundUpdates` | AB-4.21, NT-6.7, WB-8.23 |
| `update.mode` | TB-5.4, AB-4.21, NT-6.7, DL-1.38 |
| `update.showPostInstallInfo` | NT-6.13, NT-6.14 |
| `update.showReleaseNotes` | AB-4.22, ED-9.26, NT-6.14 |
| `update.titleBar` | TB-5.1, TB-5.5 |
| `urlOpenToast.neverShowAgainv1` | ED-10.88 |
| `window.autoDetectColorScheme` | ED-8.15 |
| `window.autoDetectHighContrast` | ED-8.15 |
| `window.border` | TB-3.18 |
| `window.clickThroughInactive` | TB-3.19, DL-1.38 |
| `window.closeWhenEmpty` | WB-9.11 |
| `window.commandCenter` | TB-2.1, TB-2.2, TB-2.8, TB-2.9, BN-1.3 |
| `window.confirmBeforeClose` | TB-1.24, DL-1.2, DL-2.10, DL-2.11, DL-2.12 |
| `window.confirmSaveUntitledWorkspace` | DL-1.2, DL-2.13 |
| `window.controlsStyle` | TB-3.2, TB-3.3, TB-3.9, DL-1.11, DL-1.38 |
| `window.customMenuBarAltFocus` | TB-1.5 |
| `window.customTitleBarVisibility` | TB-1.31, TB-3.10, TB-3.11, TB-3.13 |
| `window.density.editorTabHeight` | ED-1.2 |
| `window.density.layout` | TB-4.20 |
| `window.dialogStyle` | DL-1.1, DL-1.3, DL-1.4, DL-1.6, DL-1.9, DL-1.17, DL-2.1, DL-2.2 |
| `window.doubleClickIconToClose` | TB-3.7 |
| `window.enableMenuBarMnemonics` | TB-1.6 |
| `window.menuBarVisibility` | TB-1.3, TB-1.4, TB-1.5, TB-1.13, TB-4.8 |
| `window.menuStyle` | TB-1.2, TB-1.28, DL-1.38 |
| `window.nativeFullScreen` | TB-3.14, DL-1.38 |
| `window.nativeTabs` | TB-1.25, TB-3.15, DL-1.38 |
| `window.newWindowDimensions` | WB-9.10 |
| `window.openFilesInNewWindow` | DL-3.1, DL-3.3 |
| `window.openFoldersInNewWindow` | DL-3.2, DL-3.3 |
| `window.openWithoutArgumentsInNewWindow` | WB-9.12 |
| `window.restoreFullscreen` | TB-3.13 |
| `window.restoreWindows` | ED-1.42, DL-2.7 |
| `window.title` | TB-2.3, TB-2.8, TB-2.9, TB-2.10, TB-2.12, TB-2.13, TB-2.14 |
| `window.titleBarStyle` | TB-1.1, TB-1.2, TB-3.1, TB-3.12, DL-1.11, DL-1.38 |
| `window.titleSeparator` | TB-2.3, TB-2.11 |
| `window.zoomLevel` | TB-3.16, SS-5.13 |
| `window.zoomPerWindow` | TB-3.16, SS-5.13 |
| `workbench.accounts.showAvatar` | AB-4.11 |
| `workbench.activityBar.autoHide` | AB-5.7, AX-4.4 |
| `workbench.activityBar.compact` | AB-5.6, AB-5.9 |
| `workbench.activityBar.iconClickBehavior` | AB-1.4 |
| `workbench.activityBar.location` | TB-1.13, TB-1.31, TB-4.8, AB-4.24, AB-5.1, AB-5.2, AB-5.3, AB-5.4, AB-5.5, AB-5.6, AB-5.7, AX-1.2, AX-4.3, AX-4.4 |
| `workbench.activityBar.orientation` | AB-6.1, AB-6.2, SB-1.21, SB-1.22, DL-1.38 |
| `workbench.activityBar.visible` | AB-5.4 |
| `workbench.browser.dataStorage` | ED-10.64 |
| `workbench.browser.enableRemoteProxy` | ED-10.86 |
| `workbench.browser.favorites` | ED-10.37 |
| `workbench.browser.maxHistoryEntries` | ED-10.65 |
| `workbench.browser.newTabPlacement` | ED-10.70, ED-10.80 |
| `workbench.browser.openLocalhostLinks` | ED-10.69 |
| `workbench.browser.pageZoom` | ED-10.76 |
| `workbench.browser.searchEngine` | ED-10.68 |
| `workbench.browser.showInTitleBar` | TB-5.9 |
| `workbench.colorCustomizations` | BN-1.8, AB-1.2, AB-3.7, SB-4.53, AC-1.20, AC-1.42, AC-2.32, PN-3.41, PN-3.42, PN-3.43, SS-2.15 |
| `workbench.colorTheme` | AB-4.20, ED-8.15, SS-2.13, SS-2.15 |
| `workbench.commandPalette.experimental.askChatLocation` | QA-1.19 |
| `workbench.commandPalette.experimental.enableNaturalLanguageSearch` | QA-1.17 |
| `workbench.commandPalette.experimental.suggestCommands` | QA-1.18 |
| `workbench.commandPalette.history` | QA-1.9 |
| `workbench.commandPalette.preserveInput` | QA-1.11 |
| `workbench.commandPalette.showAskInChat` | QA-1.19 |
| `workbench.diffEditorAssociations` | ED-1.43, ED-10.19 |
| `workbench.editor.alwaysShowEditorActions` | ED-1.1 |
| `workbench.editor.autoLockGroups` | ED-1.23 |
| `workbench.editor.centeredLayoutAutoResize` | ED-1.35 |
| `workbench.editor.centeredLayoutFixedWidth` | ED-1.35 |
| `workbench.editor.closeEmptyGroups` | ED-1.9 |
| `workbench.editor.closeOnFileDelete` | ED-1.8 |
| `workbench.editor.customLabels.enabled` | ED-1.3, QA-2.1 |
| `workbench.editor.customLabels.patterns` | ED-1.3 |
| `workbench.editor.decorations.badges` | ED-1.3 |
| `workbench.editor.decorations.colors` | SB-4.53, ED-1.3 |
| `workbench.editor.defaultBinaryEditor` | ED-10.6 |
| `workbench.editor.doubleClickTabToToggleEditorGroupSizes` | ED-1.19, ED-1.20 |
| `workbench.editor.dragToOpenWindow` | ED-1.15, ED-1.33 |
| `workbench.editor.editorActionsLocation` | ED-1.1 |
| `workbench.editor.empty.hint` | ED-9.24 |
| `workbench.editor.enablePreview` | SB-2.3, ED-1.5, ED-1.6, QA-2.9 |
| `workbench.editor.enablePreviewFromCodeNavigation` | ED-1.5 |
| `workbench.editor.enablePreviewFromQuickOpen` | ED-1.5, QA-2.7, QA-2.9, QA-3.8 |
| `workbench.editor.focusRecentEditorAfterClose` | ED-1.8 |
| `workbench.editor.highlightModifiedTabs` | ED-1.4 |
| `workbench.editor.historyBasedLanguageDetection` | SS-3.18 |
| `workbench.editor.labelFormat` | ED-1.3 |
| `workbench.editor.languageDetection` | SS-3.18, SS-3.19, SS-3.20 |
| `workbench.editor.languageDetectionHints` | SS-3.20 |
| `workbench.editor.limit.enabled` | ED-1.38 |
| `workbench.editor.limit.excludeDirty` | ED-1.38 |
| `workbench.editor.limit.perEditorGroup` | ED-1.38 |
| `workbench.editor.limit.value` | ED-1.38 |
| `workbench.editor.mouseBackForwardToNavigate` | ED-1.32 |
| `workbench.editor.navigationScope` | ED-1.32 |
| `workbench.editor.openPositioning` | ED-1.12 |
| `workbench.editor.openSideBySideDirection` | ED-1.13, QA-2.7, QA-3.8 |
| `workbench.editor.pinnedTabSizing` | ED-1.7 |
| `workbench.editor.pinnedTabsOnSeparateRow` | ED-1.7 |
| `workbench.editor.preferHistoryBasedLanguageDetection` | SS-3.18 |
| `workbench.editor.preventPinnedEditorClose` | ED-1.7 |
| `workbench.editor.restoreEditors` | ED-1.42 |
| `workbench.editor.restoreViewState` | ED-1.41 |
| `workbench.editor.revealIfOpen` | ED-1.12 |
| `workbench.editor.scrollToSwitchTabs` | ED-1.2 |
| `workbench.editor.sharedViewState` | ED-1.41 |
| `workbench.editor.showIcons` | ED-1.3 |
| `workbench.editor.showTabIndex` | ED-1.3 |
| `workbench.editor.showTabs` | TB-2.3, ED-1.1 |
| `workbench.editor.splitInGroupLayout` | ED-1.24 |
| `workbench.editor.splitOnDragAndDrop` | ED-1.15 |
| `workbench.editor.splitSizing` | ED-1.13 |
| `workbench.editor.swipeToNavigate` | ED-1.32 |
| `workbench.editor.tabActionCloseVisibility` | ED-1.4 |
| `workbench.editor.tabActionLocation` | ED-1.4 |
| `workbench.editor.tabActionReserveSpace` | ED-1.39 |
| `workbench.editor.tabActionUnpinVisibility` | ED-1.4 |
| `workbench.editor.tabSizing` | ED-1.2 |
| `workbench.editor.tabSizingFixedMaxWidth` | ED-1.2 |
| `workbench.editor.tabSizingFixedMinWidth` | ED-1.2 |
| `workbench.editor.titleScrollbarSizing` | ED-1.2 |
| `workbench.editor.titleScrollbarVisibility` | ED-1.2 |
| `workbench.editor.untitled.labelFormat` | ED-1.3 |
| `workbench.editor.useModal` | ED-1.36 |
| `workbench.editor.wrapTabs` | ED-1.2 |
| `workbench.editorAssociations` | ED-1.43, ED-10.18, ED-10.20 |
| `workbench.editorLargeFileConfirmation` | DL-4.15 |
| `workbench.enableExperiments` | DL-1.38 |
| `workbench.experimental.modernUI` | TB-4.20, ED-1.39, PN-1.22 |
| `workbench.experimental.modernUIEditorTabStyle` | ED-1.39 |
| `workbench.experimental.share.enabled` | TB-2.16 |
| `workbench.externalBrowser` | DL-5.17 |
| `workbench.fontAliasing` | WB-5.2 |
| `workbench.hover.delay` | SB-2.43, WB-3.5 |
| `workbench.hover.reducedDelay` | WB-3.5 |
| `workbench.iconTheme` | AB-4.20, ED-8.16 |
| `workbench.layoutControl.enabled` | TB-4.1 |
| `workbench.layoutControl.type` | TB-4.2, TB-4.3, TB-4.4, TB-4.5, TB-4.6, TB-4.7 |
| `workbench.list.defaultFindMatchType` | SB-1.23, SB-2.26 |
| `workbench.list.defaultFindMode` | SB-1.23, SB-2.26 |
| `workbench.list.fastScrollSensitivity` | SB-2.42 |
| `workbench.list.horizontalScrolling` | SB-1.23, SB-2.42 |
| `workbench.list.mouseWheelScrollSensitivity` | SB-2.42 |
| `workbench.list.multiSelectModifier` | SB-1.23, SB-2.41 |
| `workbench.list.openMode` | SB-1.23, SB-2.3, SB-3.21, SB-6.16 |
| `workbench.list.scrollByPage` | SB-2.42 |
| `workbench.list.smoothScrolling` | SB-2.42 |
| `workbench.list.typeNavigationMode` | SB-1.23, SB-2.27 |
| `workbench.localHistory.enabled` | SB-7.12 |
| `workbench.localHistory.exclude` | SB-7.12 |
| `workbench.localHistory.maxFileEntries` | SB-7.12 |
| `workbench.localHistory.maxFileSize` | SB-7.12 |
| `workbench.localHistory.mergeWindow` | SB-7.12 |
| `workbench.navigationControl.enabled` | TB-2.4, TB-2.5 |
| `workbench.notifications.position` | TB-5.8, NT-1.12, NT-1.13, NT-3.4, NT-3.11 |
| `workbench.notifications.showInTitleBar` | TB-5.8 |
| `workbench.panel.defaultLocation` | TB-4.8, PN-1.3, PN-1.9 |
| `workbench.panel.opensMaximized` | PN-1.6 |
| `workbench.panel.showLabels` | PN-1.11, PN-1.13, PN-4.18 |
| `workbench.preferredDarkColorTheme` | ED-8.15 |
| `workbench.preferredHighContrastColorTheme` | ED-8.15 |
| `workbench.preferredHighContrastLightColorTheme` | ED-8.15 |
| `workbench.preferredLightColorTheme` | ED-8.15 |
| `workbench.productIconTheme` | AB-4.20, ED-8.17 |
| `workbench.quickOpen.closeOnFocusLost` | QA-2.19, QA-6.16 |
| `workbench.quickOpen.preserveInput` | QA-2.17 |
| `workbench.quickOpen.width` | QA-6.19 |
| `workbench.reduceMotion` | AC-2.31, NT-1.2 |
| `workbench.remoteIndicator.showExtensionRecommendations` | SS-4.3 |
| `workbench.sash.hoverDelay` | ED-1.47 |
| `workbench.sash.size` | ED-1.47 |
| `workbench.secondarySideBar.defaultVisibility` | AX-1.4, AX-3.4 |
| `workbench.secondarySideBar.forceMaximized` | AX-3.5 |
| `workbench.secondarySideBar.showLabels` | AX-4.2 |
| `workbench.settings.alwaysShowAdvancedSettings` | ED-8.5 |
| `workbench.settings.applyToAllProfiles` | ED-8.5 |
| `workbench.settings.editor` | ED-8.1, ED-8.8 |
| `workbench.settings.enableNaturalLanguageSearch` | ED-8.2 |
| `workbench.settings.openDefaultKeybindings` | ED-8.8, ED-8.54 |
| `workbench.settings.openDefaultSettings` | ED-8.8, ED-8.53 |
| `workbench.settings.settingsSearchTocBehavior` | ED-8.4 |
| `workbench.settings.showAISearchToggle` | ED-8.2 |
| `workbench.settings.useSplitJSON` | ED-8.8, ED-8.53 |
| `workbench.sideBar.location` | TB-1.13, TB-4.3, TB-4.8, TB-4.9, AB-6.12, SB-1.2, AX-2.1, AX-2.2, NT-1.14, NT-6.8 |
| `workbench.startupEditor` | ED-9.3, ED-9.14 |
| `workbench.statusBar.visible` | TB-4.8, SS-1.1, SS-1.15 |
| `workbench.tips.enabled` | ED-9.18 |
| `workbench.tree.enableStickyScroll` | SB-1.24 |
| `workbench.tree.expandMode` | SB-1.24, SB-2.2, SB-2.3 |
| `workbench.tree.indent` | SB-1.24, SB-2.29 |
| `workbench.tree.renderIndentGuides` | SB-1.24, SB-2.29 |
| `workbench.tree.stickyScrollMaxItemCount` | SB-1.24 |
| `workbench.trustedDomains.promptInTrustedWorkspace` | DL-5.11, DL-5.14 |
| `workbench.view.alwaysShowHeaderActions` | SB-1.8 |
| `workbench.view.showQuietly` | PN-5.18, SS-5.14 |
| `workbench.welcomePage.experimentalOnboarding` | ED-9.15 |
| `workbench.welcomePage.preferReducedMotion` | ED-9.1 |
| `workbench.welcomePage.walkthroughs.openOnInstall` | ED-9.12 |
| `zenMode.centerLayout` | ED-1.34, ED-1.35 |
| `zenMode.fullScreen` | ED-1.34 |
| `zenMode.hideActivityBar` | AB-5.8, ED-1.34 |
| `zenMode.hideLineNumbers` | ED-1.34 |
| `zenMode.hideStatusBar` | ED-1.34 |
| `zenMode.restore` | ED-1.34 |
| `zenMode.showTabs` | ED-1.34 |
| `zenMode.silentNotifications` | ED-1.34, NT-4.10 |

## Keybindings

Keys are listed as the features record them, with their context in parentheses when the explorer wrote one. Platform differences are in the text.

| Key | Features |
|---|---|
| (focus) | ED-5.9 |
| A | ED-7.40 |
| add Shift to extend (all when textInputFocus | ED-2.9 |
| all enabled by the context key activeEditorIsReviewChanges | ED-6.38 |
| all handled inside the tab, not registered as workbench keybindings | ED-1.45 |
| all need editor text focus, and the file keys need inline diffs in some editor | ED-6.33 |
| all need terminalFocus and terminalSuggestWidgetVisible | AC-5.7 |
| all when editorTextFocus and the list is visible | AC-1.36 |
| all when inQuickOpen | QA-6.8 |
| all when the box shows a quick pick or tree | QA-6.21 |
| all when the editor text has focus and folding is on | ED-5.11 |
| all when the sticky scroll has focus | ED-5.6 |
| all with a terminal in focus | PN-3.36 |
| all with a terminal in focus in the normal buffer | PN-3.33 |
| Alt (tap and release, handled by the menu bar widget, not a keybinding rule) | TB-1.5 |
| Alt on macOS) | ED-2.9, ED-2.12 |
| Alt while dragging (Shift on Mac) to toggle splitting | ED-1.15 |
| Alt while dragging out of the window to toggle new window | ED-1.15 |
| Alt+- (all in the Search Editor) | ED-3.34 |
| Alt+0 or Ctrl+9 (last tab in group, Ctrl+0 or Cmd+9 on Mac) | ED-1.29 |
| Alt+1 to Alt+9 (tab by position, Ctrl+1 to Ctrl+9 on Mac) | ED-1.29 |
| Alt+= | ED-3.34 |
| Alt+[ (show previous, same when) | AC-2.9 |
| Alt+] (show next, when editor writable and inlineSuggestionVisible) | AC-2.9 |
| Alt+A (accessibilityHelpIsShown and accessibleViewHasAssignedKeybindings) | WB-4.1 |
| Alt+C | SB-3.3, ED-3.31 |
| Alt+C (Cmd+Alt+C on macOS) on toggleFindCaseSensitive | PN-3.1 |
| Alt+C (Cmd+Alt+C on macOS), editor focus | ED-3.9 |
| Alt+click on a tab's close button (Option+click on Mac) | ED-1.50 |
| Alt+Ctrl+S (Alt+Cmd+S on macOS) | TB-2.16 |
| Alt+CtrlCmd+WinCtrl+N (welcome.showNewFileEntries) | ED-9.25 |
| Alt+Delete (clear cell outputs), when notebookEditorFocused && !inputFocus && notebookCellHasOutputs && editable | ED-7.23 |
| Alt+Down (mac also Ctrl+N) for the next, all when editorFocus and parameterHintsVisible and parameterHintsMultipleSignatures | AC-3.6 |
| Alt+Down (next item) | ED-1.36 |
| Alt+Down (scm.forceViewNextCommit) | SB-4.13 |
| Alt+Down and Alt+Up for the headings with held-key fallback (Cmd+Down and Cmd+Up on Mac) | QA-6.21 |
| Alt+Down and Alt+Up on the two go-to-command commands while the accessible view shows the terminal | PN-3.22 |
| Alt+Down Arrow (when the history input has focus and history navigation is enabled) | PN-6.4 |
| Alt+Down, when notebookEditorFocused && !inputFocus | ED-7.4 |
| Alt+E | TB-1.6 |
| Alt+E kills the selected rows while the list has focus (a key handler on the list, not a keybinding rule) | WB-6.2 |
| Alt+End Alt+End (focus most recent execution) | ED-7.39 |
| Alt+Enter | QA-2.22, QA-2.23 |
| Alt+Enter (Cmd+Enter on macOS) when the Search view is focused and has results | SB-3.28 |
| Alt+Enter (Cmd+Enter on macOS) when there are results and the Search view is focused (search.action.openInEditor) | SB-3.28 |
| Alt+Enter (execute and insert below), when notebookCellListFocused | ED-7.13 |
| Alt+Enter (modifier read by the picker, no keybinding rule) | QA-2.7, QA-3.8 |
| Alt+Enter (when the focused view is workbench.view.testing) | SB-6.16 |
| Alt+Enter when the find widget is visible | ED-3.19 |
| Alt+F | TB-1.6 |
| Alt+F1 (accessibility help, when a hover shows) | ED-4.39 |
| Alt+F1 (Alt+Shift+F1 on Linux) on editor.action.accessibilityHelp | PN-3.22 |
| Alt+F1 (Linux: Alt+Shift+F1, with Alt+F1 as secondary) | PN-4.20, PN-5.19 |
| Alt+F1 when the find widget is focused (findWidgetFocused) | ED-3.28 |
| Alt+F10 | ED-12.6 |
| Alt+F10 (workbench.actions.menubar.focus, when IsWebContext) | TB-1.7 |
| Alt+F11 on Windows web) | SB-5.9 |
| Alt+F12 for Peek Definition (Ctrl+Shift+F10 on Linux) | ED-4.9 |
| Alt+F2 (accessible view, when the hover has focus) | ED-4.39 |
| Alt+F2 (Alt+Shift+F2 on Linux) | NT-3.12 |
| Alt+F2 (Alt+Shift+F2 on Linux) on focusAccessibleBuffer when screen reader mode is on and the terminal has focus | PN-3.22 |
| Alt+F2 to open (Linux: Alt+Shift+F2, with Alt+F2 as secondary) | AC-2.38 |
| Alt+F3 (next change in the peek) and Shift+Alt+F3 (previous) | ED-6.25 |
| Alt+F3 (next change) | ED-7.36 |
| Alt+F4 (Windows and Linux) | DL-2.18 |
| Alt+F4 and Ctrl+Shift+W (Windows and Linux) | TB-3.4 |
| Alt+F5 (next change) and Shift+Alt+F5 (previous change) when a text compare editor is visible | ED-6.6 |
| Alt+F5 (next change) and Shift+Alt+F5 (previous change) when the active editor is the multi-file diff | ED-6.14 |
| Alt+F5 (next change) and Shift+Alt+F5 (previous) with text focus | ED-6.25 |
| Alt+F6 (when the accessible view or help is shown and its hint setting is on) | WB-4.3 |
| Alt+F8 (next) | ED-7.44 |
| Alt+F8 (testing.goToNextMessage) | ED-13.4 |
| Alt+F8 and Ctrl+. are shown beside View Problem and Quick Fix... | ED-4.26 |
| Alt+F8 and Shift+Alt+F8 (editor focus) | ED-4.27 |
| Alt+F8 and Shift+Alt+F8 (next and previous failure, when the peek is visible) | ED-13.4 |
| Alt+F9 | ED-12.6 |
| Alt+F9 (next uncovered line) | ED-13.5 |
| Alt+G | TB-1.6 |
| Alt+H (derived from the `&&` letters in the labels, not rules in the keybinding registry) | TB-1.6 |
| Alt+H (previous diff file) | ED-6.33 |
| Alt+H (testing.toggleTestingPeekHistory, when the peek is visible) | ED-13.4 |
| Alt+H (toggle history, when the peek is visible) | ED-13.4 |
| Alt+H (when accessibilityHelpIsShown) | WB-4.2 |
| Alt+J (next change) | ED-6.33 |
| Alt+K (accessibilityHelpIsShown and accessibleViewHasUnassignedKeybindings) | WB-4.1 |
| Alt+K (previous change) | ED-6.33 |
| Alt+K, or Cmd+Alt+K on macOS (keybindings.editor.recordSearchKeys, when the search box has focus) | ED-8.10 |
| Alt+L (Cmd+Alt+L on macOS) | ED-3.34 |
| Alt+L (Cmd+Alt+L on macOS), editor focus | ED-3.12 |
| Alt+L (next diff file) | ED-6.33 |
| Alt+Left (goBack) | ED-10.79 |
| Alt+Left (Windows) | QA-6.8 |
| Alt+Left and Alt+Right on Windows | ED-1.32 |
| Alt+Left and Alt+Right send word movement | PN-3.36 |
| Alt+Left and Alt+Up on workbench.action.terminal.focusPreviousPane | PN-2.22 |
| Alt+Left and Ctrl+[ (back) | ED-10.32 |
| Alt+Left on Windows (Ctrl+- on Mac, Ctrl+Alt+- on Linux) | AC-7.12 |
| Alt+P (Cmd+Alt+P on macOS), editor focus | ED-3.17 |
| Alt+P (sort by precedence) | ED-8.9 |
| Alt+P for Preserve Case (Cmd+Alt+P on macOS) | SB-3.6 |
| Alt+PageDown (macOS Cmd+PageDown) for scrollPageDown | ED-5.30 |
| Alt+PageUp (macOS Cmd+PageUp) for scrollPageUp | ED-5.30 |
| Alt+R | TB-1.6 |
| Alt+R (Cmd+Alt+ on macOS), all when the search input box is focused | ED-3.31 |
| Alt+R (Cmd+Alt+R on macOS) on toggleFindRegex | PN-3.1 |
| Alt+R (Cmd+Alt+R on macOS), editor focus | ED-3.11 |
| Alt+R, all when the Search view is focused (Cmd+Alt+C, Cmd+Alt+W and Cmd+Alt+R on macOS, where the case toggle also needs a file or folder row not to be focused) | SB-3.3 |
| Alt+Right (goForward) | ED-10.79 |
| Alt+Right and Alt+Down on workbench.action.terminal.focusNextPane (when terminalFocus and a split terminal is active | PN-2.22 |
| Alt+Right and Ctrl+] (forward) | ED-10.32 |
| Alt+S | TB-1.6 |
| Alt+Shift+D (editor text focus) | SS-3.19 |
| Alt+Shift+D (shown in the tooltip) | SS-3.20 |
| Alt+Shift+Down, when notebookEditorFocused && !inputFocus (Windows also takes Ctrl+Insert, Shift+Delete and Shift+Insert) | ED-7.5 |
| Alt+Shift+Up | ED-7.5 |
| Alt+T | TB-1.6 |
| Alt+Up | ED-7.4 |
| Alt+Up (mac also Ctrl+P) for the previous signature and Down | AC-3.6 |
| Alt+Up (previous item) | ED-1.36 |
| Alt+Up (scm.forceViewPreviousCommit) | SB-4.13 |
| Alt+Up and Alt+Down (move line) | ED-2.11 |
| Alt+Up and Alt+Down send Ctrl+Up and Ctrl+Down | PN-3.36 |
| Alt+Up Arrow | PN-6.4 |
| Alt+V | TB-1.6 |
| Alt+W | SB-3.3, ED-3.31 |
| Alt+W (Cmd+Alt+W on macOS) on toggleFindWholeWord | PN-3.1 |
| Alt+W (Cmd+Alt+W on macOS), editor focus | ED-3.10 |
| Alt+Z | ED-2.35, ED-5.36 |
| Alt+Z on workbench.action.terminal.sizeToContentWidth (when terminalFocus) | PN-2.40 |
| Arrow keys | ED-2.9 |
| B | ED-7.40 |
| Back - Alt+Left (Windows), Ctrl+- (macOS), Ctrl+Alt+- (Linux), plus the browser Back key | TB-2.4 |
| Backspace and Delete (textInputFocus) | ED-2.12 |
| Backspace and Delete, when notebook.multiCursor.enabled is true and a cell editor is focused | ED-7.32 |
| binding is commented out in the source) | SB-6.1 |
| both need editor text focus and inline diffs in the active editor | ED-6.31 |
| both only while a history box is focused and there is an entry to move to | SB-3.5 |
| both use Cmd on macOS | ED-3.47, WB-3.1 |
| both when a file or match row is focused | SB-3.21 |
| C | ED-7.40 |
| C mnemonic on the Copy button (OS box on Windows and Linux) | DL-1.18 |
| change drop type, when the drop widget shows) | ED-2.29 |
| Cmd replaces Ctrl on macOS | TB-3.16 |
| Cmd+A on selectAll (macOS only, when any terminal has focus) | PN-3.19 |
| Cmd+Alt+[ on macOS) | ED-5.11, ED-7.12 |
| Cmd+Alt+] on macOS) | ED-5.11 |
| Cmd+Alt+] on macOS), when notebookEditorFocused && !inputFocus | ED-7.12 |
| Cmd+Alt+Backspace (macOS) | DL-4.2 |
| Cmd+Alt+H (Hide Others) | TB-1.24 |
| Cmd+Alt+I, Cmd+Alt+J or Cmd+Alt+C on Mac | ED-10.35 |
| Cmd+Alt+T on Mac for Close Others | ED-1.9 |
| Cmd+Alt+T on macOS (closeOtherEditors) | DL-2.1 |
| Cmd+Alt+Tab on Mac) | ED-1.31 |
| Cmd+Backspace on macOS (when notificationFocus) | NT-2.14 |
| Cmd+Backspace on macOS) | NT-5.4 |
| Cmd+Backspace or Delete (macOS, same condition) | DL-4.1 |
| Cmd+D (macOS only) | DL-1.12 |
| Cmd+Down on macOS (secondary) | PN-4.10 |
| Cmd+E (macOS only) | ED-3.3 |
| Cmd+E on Mac (toggleLocked) | ED-10.21 |
| Cmd+Enter on macOS when the replace box is focused | ED-3.16 |
| Cmd+H (Hide) | TB-1.24 |
| Cmd+K on Mac) | SB-1.15 |
| Cmd+K on macOS only (when the console view is focused) | PN-6.14 |
| Cmd+K on macOS when the terminal has focus and screen reader mode is off, or when the accessible view shows the terminal | PN-3.34 |
| Cmd+Left, Cmd+Right and Cmd+Backspace on macOS | PN-3.36 |
| Cmd+M (Minimize) | TB-1.25 |
| Cmd+O (macOS native) | DL-3.3 |
| Cmd+O (macOS, when remoteFileDialogVisible) | DL-3.19 |
| Cmd+P (Mac) | QA-2.1 |
| Cmd+Q | DL-2.11 |
| Cmd+Q (macOS and Linux) | DL-2.17 |
| Cmd+Q (macOS and Linux, quit) | DL-2.5, DL-2.10 |
| Cmd+Q (workbench.action.quit) | TB-1.24 |
| Cmd+Shift+B on Mac | QA-4.11 |
| Cmd+Shift+O (Mac) | QA-3.10 |
| Cmd+Shift+W (macOS) | TB-3.4, DL-2.18 |
| Cmd+Shift+W on macOS, Alt+F4 on Windows and Linux, or Ctrl+Shift+W (closeWindow) | DL-2.5 |
| Cmd+Shift+Z on Mac | ED-4.41 |
| Cmd+Shift+Z on macOS) - these are the standard editor keys, answered by the notebook while focus is inside it | ED-7.41 |
| Cmd+T (Mac) | QA-2.12 |
| Cmd+W (macOS, closeWindowOrEditorOnMac) | DL-2.18 |
| Cmd+W (macOS, when no editor is visible and there is a single group) | TB-3.4 |
| Cmd+Z and Cmd+Shift+Z (macOS) | DL-4.13 |
| Control+- (Mac) | QA-6.8 |
| Control+G (Mac) | QA-3.1 |
| Control+Minus and Control+Shift+Minus on Mac | ED-1.32 |
| Control+Q on Mac | QA-4.1 |
| Copy with Syntax Highlighting has no default key | ED-2.28 |
| Ctrl or Alt held means copy) | ED-2.30 |
| Ctrl while dragging (Alt on Mac) to copy | ED-1.15, ED-1.48, ED-1.49 |
| Ctrl+ | SB-6.29, SB-6.30, ED-13.4, ED-13.5, PN-6.18, PN-6.26 |
| Ctrl+, | AB-4.15 |
| Ctrl+, (aiSettings.action.open, when not in the Agents window) | ED-8.19 |
| Ctrl+, (Cmd+, on macOS, when not isGlass) | TB-5.6 |
| Ctrl+, (Cmd+, on macOS, workbench.action.openSettings) | TB-1.20 |
| Ctrl+, (workbench.action.openSettings) | ED-8.1 |
| ctrl+- (also ctrl+shift+- and ctrl+numpad-) for zoom out | SS-5.13 |
| Ctrl+- (zoom out) | ED-10.34 |
| Ctrl+- (zoomOut) | ED-10.76 |
| Ctrl+. (change paste type, when the paste widget shows | ED-2.29 |
| Ctrl+. (Cmd+. on macOS) on showQuickFixes when the terminal has focus | PN-3.14 |
| Ctrl+. (textInputFocus) opens Quick Fix | ED-4.14 |
| Ctrl+. (when a problem row is focused) | PN-4.13 |
| Ctrl+/ (mac: Cmd+/) when terminalFocus, terminalIsOpen and terminalSuggestWidgetVisible | AC-5.14 |
| Ctrl+/ (toggle line comment) | ED-2.16 |
| Ctrl+/ to accept while the view shows an inline completion (Control+/ on macOS) | AC-2.38 |
| Ctrl+/ while the list is visible | AC-1.26 |
| Ctrl+/, when notebookEditorFocused && notebookEditable && !inputFocus | ED-7.10 |
| Ctrl+0 (Cmd+0 on Mac) | SB-1.3 |
| Ctrl+0 (reset) | ED-10.34 |
| Ctrl+1 to Ctrl+8 (group by number, and Ctrl+2 to Ctrl+8 add a new group when the number is the one right after the last group) | ED-1.22 |
| ctrl+= (also ctrl+shift+= and ctrl+numpad+) for zoom in | SS-5.13 |
| Ctrl+= (zoomIn) | ED-10.76 |
| Ctrl+= or Ctrl++ (zoom in) | ED-10.34 |
| Ctrl+[ (outdent line) | ED-2.17 |
| Ctrl+\ (Split Editor) | ED-1.13 |
| Ctrl+] (indent line) | ED-2.17 |
| Ctrl+` on workbench.action.terminal.toggleTerminal (Mac Ctrl+`) | PN-2.1 |
| Ctrl+A (Cmd+A on macOS), when the notebook editor is focused and a cell output is focused | ED-7.48 |
| Ctrl+A (list.selectAll, list supports multi-select) | SB-2.41 |
| Ctrl+A (select all) | ED-2.6 |
| Ctrl+Alt held down flips the hints when the setting is onUnlessPressed or offUnlessPressed | ED-4.21 |
| Ctrl+Alt+- (Linux) | QA-6.8 |
| Ctrl+Alt+/ (Cmd+Alt+/ on macOS) | TB-5.9 |
| Ctrl+Alt+B (Cmd+Alt+B on Mac) | AX-1.1 |
| Ctrl+Alt+B (Cmd+Alt+B on macOS) | TB-4.5 |
| Ctrl+Alt+Backspace (remove brackets) | ED-2.10 |
| Ctrl+Alt+C (Cmd+Alt+C on macOS, copy watch expression, Watch focused) | SB-5.13 |
| Ctrl+Alt+C (copyFilePath, no editor focus | SB-2.16 |
| Ctrl+Alt+C, Shift+Alt+C on Windows (search.action.copyPath, file or folder row with a path) | SB-3.24 |
| Ctrl+Alt+Down and Ctrl+Alt+Up for the headings (Cmd+Alt+Down and Cmd+Alt+Up on Mac) | QA-6.21 |
| Ctrl+Alt+Enter on Windows), when notebookCellListFocused | ED-7.13 |
| Ctrl+Alt+Enter when the find widget is visible | ED-3.16 |
| Ctrl+Alt+Enter, when the Replace row is active and the editor find widget is not showing | SB-3.8 |
| Ctrl+Alt+F | SB-1.23 |
| Ctrl+Alt+F (when in the debug console or the console view is focused) | PN-6.11 |
| Ctrl+Alt+F or F3 (list.find, list that supports find is focused) | SB-2.26 |
| Ctrl+Alt+F8 (Cmd+Alt+F8 on Mac) when the editor has focus | ED-4.40 |
| Ctrl+Alt+G sends Ctrl+G | PN-3.36 |
| Ctrl+Alt+Left (previous group) | ED-1.14 |
| Ctrl+Alt+Minus and Ctrl+Shift+Minus on Linux | ED-1.32 |
| ctrl+alt+o (Cmd+Alt+O on macOS) | SS-4.3 |
| Ctrl+Alt+PageDown and Ctrl+Alt+PageUp on the line commands (Ctrl+Shift+Down and Ctrl+Shift+Up on Linux) | PN-3.33 |
| Ctrl+Alt+R (Ctrl+Option+R on macOS) on runRecentCommand when the terminal has focus and screen reader mode is off | PN-3.12 |
| Ctrl+Alt+R (revealFileInOS, when no editor text has focus | SB-2.15 |
| Ctrl+Alt+R sends Ctrl+R to the shell in screen reader mode | PN-3.24 |
| Ctrl+Alt+Right (next group) | ED-1.14 |
| Ctrl+Alt+Shift+G (Cmd+Alt+Shift+G on macOS) when editorTextFocus and accessibilityModeEnabled | WB-4.9 |
| Ctrl+Alt+Space (mac: Ctrl+Alt+Space) when editorTextFocus and the list is visible | AC-1.25 |
| Ctrl+Alt+Space (mac: Ctrl+Alt+Space) when not editorTextInputFocus | AC-5.13 |
| Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS) | TB-4.11, TB-4.12, TB-4.13, TB-4.14 |
| Ctrl+Alt+Up and Ctrl+Alt+Down (editorTextFocus | ED-2.1 |
| Ctrl+Alt+V (Windows also Alt+Backquote | ED-2.38 |
| Ctrl+B (Cmd+B on Mac) | SB-1.1 |
| Ctrl+B (Cmd+B on macOS) | TB-4.3 |
| Ctrl+Backspace (Option+Backspace on macOS) and Ctrl+Delete (Option+Delete on macOS) delete a word | PN-3.36 |
| Ctrl+Backspace and Ctrl+Delete (delete word | ED-2.12 |
| Ctrl+C | ED-7.5 |
| Ctrl+C (Cmd+C on macOS, Variables or Watch focused and no input open) | SB-5.13 |
| Ctrl+C (copy row) | ED-8.11 |
| Ctrl+C (Copy, when the Problems view is focused) | PN-4.14 |
| Ctrl+C (Copy, when the Problems view is focused, the list is visible and no related-information row is focused) | PN-4.15 |
| Ctrl+C (filesExplorer.copy) | SB-2.9 |
| Ctrl+C (search.action.copyMatch, file or match row) | SB-3.24 |
| Ctrl+C (when the Ports view is focused and one forwarded or detected port is selected) | PN-7.14 |
| Ctrl+C or Cmd+C (Explorer focused) | DL-4.9 |
| Ctrl+C, and Ctrl+Insert on Windows, when suggestWidgetDetailsFocused | AC-1.25 |
| Ctrl+click (Cmd on Mac) on a hint part follows it | ED-4.21 |
| Ctrl+click or Cmd+click | ED-1.11 |
| Ctrl+Cmd+F (macOS) | TB-3.13 |
| Ctrl+Cmd+Left, Right, Up and Down on the four resize commands (Mac only, when terminalFocus) | PN-2.23 |
| Ctrl+D (add next match) | ED-7.32 |
| Ctrl+D (both) | ED-10.37 |
| Ctrl+D (editor focus) | ED-2.3 |
| Ctrl+Down (Cmd+Down on macOS) when the find box or replace box is focused | ED-3.44 |
| Ctrl+Down (list.scrollDown, list focused and not scrolled to the bottom) | SB-2.42 |
| Ctrl+Down (mac adds Ctrl+N) when the list is visible | AC-1.30 |
| Ctrl+Down (macOS Ctrl+PageDown) for scrollLineDown | ED-5.30 |
| Ctrl+Down (move from the search box to the table) | ED-8.9 |
| Ctrl+Down (search.focus.nextInputBox, in an input box) | SB-3.26 |
| Ctrl+Down (search.focus.nextInputBox, when in the Search Editor with a search box focused) | ED-3.47 |
| Ctrl+Down (when the filter box is focused) | PN-4.5 |
| Ctrl+Down (widgetNavigation.focusNext, when a navigable container has focus and either no list is focused or the list is scrolled to the bottom) | WB-3.1 |
| Ctrl+Down Arrow (when the filter has focus, moves to the list) | PN-7.3 |
| Ctrl+Down on Windows or Ctrl+Shift+Down elsewhere (focus in output) | ED-7.9 |
| Ctrl+Down on workbench.action.terminal.focus (only when accessibility mode is on and the accessible view for the terminal is open on its last line) | PN-2.1 |
| Ctrl+Down when a history cell is focused, or Alt+Home Alt+Home (focus input editor) | ED-7.39 |
| Ctrl+E (Cmd+E on macOS) while the accessibility help dialog is shown (accessibilityHelpIsShown) | WB-4.8 |
| Ctrl+E (Windows and Linux only) | QA-2.1 |
| Ctrl+End (scroll to bottom), when the active editor is the Interactive Window (Cmd+Up and Cmd+Down on macOS) | ED-7.38 |
| Ctrl+End and Ctrl+Home (Shift+End and Shift+Home on Linux) on the scroll commands in that view | PN-3.22 |
| Ctrl+End and Ctrl+Home on bottom and top (Shift+End and Shift+Home on Linux) | PN-3.33 |
| Ctrl+Enter | QA-2.22 |
| Ctrl+Enter (Cmd+Enter on macOS) and Alt+Enter (when the Breakpoints view is focused) | SB-5.23 |
| Ctrl+Enter (complete merge, in the merge editor) | ED-6.22 |
| Ctrl+Enter (Control+Enter on macOS) | SB-3.21 |
| Ctrl+Enter (editor text focus, when the active editor has inline diffs) | ED-6.29 |
| Ctrl+Enter (editor.action.submitComment, when the comment editor is focused) | ED-12.4 |
| Ctrl+Enter (execute) | ED-7.37, ED-7.39 |
| Ctrl+Enter (insert line below) | ED-2.13 |
| Ctrl+Enter (new line in the box, Control+Enter on macOS) | SB-3.2 |
| Ctrl+Enter (open to the side from the tree) | ED-5.9 |
| Ctrl+Enter (open to the side, Ctrl+Enter with the Control key on Mac) | ED-4.10 |
| Ctrl+Enter (pins when the preview setting is on) | QA-2.9 |
| Ctrl+Enter (same) | QA-2.7, QA-3.8 |
| Ctrl+Enter (scm.acceptInput, when scmRepository) | SB-4.7 |
| Ctrl+Enter (when a preview is active and the Refactor Preview view is focused) | PN-7.24 |
| Ctrl+Enter (when a problem row is focused) | PN-4.11 |
| Ctrl+Enter (when editor.rename.enablePreview is on) | ED-4.12 |
| Ctrl+Enter (when the Explorer viewlet has focus | SB-2.4 |
| Ctrl+Enter in the replace box (Windows only) | ED-3.50 |
| Ctrl+Enter insert code cell below and Ctrl+Shift+Enter insert code cell above, when notebookCellListFocused && !inputFocus (Cmd on macOS) | ED-7.2 |
| Ctrl+Enter inside the find box or the replace box (the real Ctrl key on macOS too) | ED-3.20 |
| Ctrl+Enter on macOS (WinCtrl) | PN-4.11 |
| Ctrl+Enter or Cmd+Enter (refactorPreview.apply, when the preview pane is focused) | DL-1.29 |
| Ctrl+Enter previews | ED-4.14 |
| Ctrl+Enter, shown on Keep Notebook (the key of editor.action.inlineDiffs.acceptAll as listed in ED-6.29) | ED-7.47 |
| Ctrl+Enter, when editorTextFocus && inputFocus && notebookEditorFocused or notebookCellListFocused | ED-7.40 |
| Ctrl+Escape, Alt+Backspace on Windows (workbench.action.hideComment, when a comment widget is visible) | ED-12.5 |
| Ctrl+F (aiSettings.action.focusSearch, when the settings pane has focus) | ED-8.19 |
| Ctrl+F (aiSettings.action.focusSearch, while the Cursor Settings pane has focus) | ED-8.61 |
| Ctrl+F (Cmd+F on macOS) | ED-3.1 |
| Ctrl+F (Cmd+F on macOS) on focusFind when a terminal or the find box has focus | PN-3.1 |
| Ctrl+F (find) | ED-6.38 |
| Ctrl+F (notebook.find), when notebookEditorFocused and the cell editor is not focused | ED-7.30 |
| Ctrl+F (search, when the editor is active) | ED-8.9 |
| Ctrl+F (sent from the page preload as find-in-page) | ED-10.36 |
| Ctrl+F (settings.action.search) | ED-8.2 |
| Ctrl+F (showFind) | ED-10.77 |
| Ctrl+F (showFind, when a webview editor is active and the text editor is not focused) | ED-10.7 |
| Ctrl+F (when text input focus in the console input) | PN-6.10 |
| Ctrl+F (when the Comments view is focused) | PN-7.3 |
| Ctrl+F (when the Problems view is focused) | PN-4.5 |
| Ctrl+F12 (editorTextFocus) | ED-4.7 |
| Ctrl+F2 (change all occurrences) | ED-2.4 |
| Ctrl+F3 (editor focus) | ED-3.8 |
| Ctrl+F4 and Ctrl+W on Windows | AX-3.2 |
| Ctrl+F5 (Ctrl+F5 on macOS, using the Control key) | SB-5.3 |
| Ctrl+G | QA-3.1 |
| Ctrl+G (Cmd+G on macOS) when the terminal has focus | PN-3.13 |
| Ctrl+G (mac Ctrl+G) | SS-3.3 |
| Ctrl+H (Cmd+Alt+F on macOS) | ED-3.14 |
| Ctrl+H (Cmd+Y on Mac) when a Browser tab is active | ED-10.65 |
| Ctrl+H (replace, through editor.action.startFindReplaceAction) | ED-7.30 |
| Ctrl+Home (scroll to top) | ED-7.38 |
| Ctrl+Home and Ctrl+End | ED-2.9, ED-7.9 |
| Ctrl+Home and Ctrl+End (Control on Mac) | QA-6.21 |
| Ctrl+I (mac: Ctrl+Space, Alt+Esc, Cmd+I) when editorTextFocus | AC-1.1 |
| Ctrl+I (mac: Ctrl+Space, Cmd+I) when editorTextFocus, the list is visible and a row is focused | AC-1.21 |
| Ctrl+I (mac: Ctrl+Space, Cmd+I) when the list is visible and no row is focused | AC-1.37 |
| Ctrl+I (same keys as AC-1.21) | AC-1.29 |
| Ctrl+I when AI results exist (toggle AI search) | ED-8.2 |
| Ctrl+J (Cmd+J on macOS) | TB-4.4, PN-1.2, PN-1.20 |
| Ctrl+K C | ED-6.12 |
| Ctrl+K Ctrl+, (create from selection) | ED-5.15 |
| Ctrl+K Ctrl+- (fold all except selected) | ED-5.12 |
| Ctrl+K Ctrl+. (remove manual ranges) | ED-5.15 |
| Ctrl+K Ctrl+/ (fold all block comments) | ED-5.14 |
| Ctrl+K Ctrl+0 (fold all) | ED-5.12 |
| Ctrl+K Ctrl+1 to Ctrl+K Ctrl+7 | ED-5.13 |
| Ctrl+K Ctrl+8 (fold all regions) | ED-5.14 |
| Ctrl+K Ctrl+9 (unfold all regions) | ED-5.14 |
| Ctrl+K Ctrl+= (unfold all except selected) | ED-5.12 |
| Ctrl+K Ctrl+[ (fold recursively) | ED-5.11 |
| Ctrl+K Ctrl+\ (Split Editor Orthogonal | ED-1.13 |
| Ctrl+K Ctrl+] (unfold recursively) | ED-5.11 |
| Ctrl+K Ctrl+A (add) | ED-8.11 |
| Ctrl+K Ctrl+Alt+C (when activeCursorHasCommentingRange) | ED-12.2 |
| Ctrl+K Ctrl+Alt+Down | ED-12.6 |
| Ctrl+K Ctrl+Alt+S (stage selected ranges, text focus on a file) | ED-6.8 |
| Ctrl+K Ctrl+Alt+Up | ED-12.6 |
| Ctrl+K Ctrl+B (set anchor) | ED-2.8 |
| Ctrl+K Ctrl+C (add line comment) | ED-2.16 |
| Ctrl+K Ctrl+C (collapse or expand input) | ED-7.11 |
| Ctrl+K Ctrl+D (move last selection to next match) | ED-2.3 |
| Ctrl+K Ctrl+Down | ED-1.22 |
| Ctrl+K Ctrl+E (define) | ED-8.12 |
| Ctrl+K Ctrl+F for Format Selection | ED-4.17 |
| Ctrl+K Ctrl+I (editorTextFocus) for Show or Focus Hover | ED-4.2 |
| Ctrl+K Ctrl+I (list.showHover, list focused) | SB-2.43 |
| Ctrl+K Ctrl+I (stock: any focus, weight EditorCore | WB-3.5 |
| Ctrl+K Ctrl+I (when editorTextFocus, while debugging) | ED-11.15 |
| Ctrl+K Ctrl+I on workbench.action.terminal.focusHover (when terminalTabsFocus or terminalFocus) | PN-2.14 |
| Ctrl+K Ctrl+J (unfold all) | ED-5.12 |
| Ctrl+K Ctrl+K (editor.action.defineKeybinding, when the resource is the user keybindings file) | ED-8.8, ED-8.55 |
| Ctrl+K Ctrl+K (select from anchor to cursor) | ED-2.8 |
| Ctrl+K Ctrl+L (toggle fold) | ED-5.11 |
| Ctrl+K Ctrl+Left | ED-1.22 |
| Ctrl+K Ctrl+M | ED-1.19 |
| Ctrl+K Ctrl+N (unstage selected ranges, right side of a diff of an index file) | ED-6.8 |
| Ctrl+K Ctrl+O (not macOS, when remoteFileDialogVisible) | DL-3.19 |
| Ctrl+K Ctrl+O (Windows and Linux only) | DL-3.2 |
| Ctrl+K Ctrl+P (Cmd+Alt+Tab on Mac) | QA-4.3 |
| Ctrl+K Ctrl+P (Show All Editors By Appearance | ED-1.31 |
| Ctrl+K Ctrl+PageDown and Ctrl+K Ctrl+PageUp (next and previous in group) | ED-1.29 |
| Ctrl+K Ctrl+Q (last edit location) | ED-1.32 |
| Ctrl+K Ctrl+R (revert selected ranges, text focus on a file) | ED-6.8 |
| Ctrl+K Ctrl+R (workbench.action.keybindingsReference) | TB-1.30 |
| Ctrl+K Ctrl+Right | ED-1.22 |
| Ctrl+K Ctrl+S | AB-4.16 |
| Ctrl+K Ctrl+S (workbench.action.openGlobalKeybindings) | TB-1.20, ED-8.9 |
| Ctrl+K Ctrl+Shift+\ (Split Editor in Group) | ED-10.90 |
| Ctrl+K Ctrl+Shift+\ (Split in Group, or Join in Group when a side by side editor is active) | ED-1.24 |
| Ctrl+K Ctrl+Shift+\ (split, when editorTextFocus in an editable cell) | ED-7.6 |
| Ctrl+K Ctrl+Shift+L (toggle fold recursively) | ED-5.11 |
| Ctrl+K Ctrl+Shift+N | NT-3.6 |
| ctrl+k ctrl+shift+n (notifications.showList) | SS-5.2 |
| Ctrl+K Ctrl+Shift+N (notifications.showList) | NT-4.8 |
| Ctrl+K Ctrl+Shift+N (opens the center, which hides the toasts first) | NT-1.4 |
| Ctrl+K Ctrl+Shift+O (open the active side, in a diff editor) | ED-6.9 |
| Ctrl+K Ctrl+Shift+S on Windows | WB-1.4 |
| Ctrl+K Ctrl+Shift+W (close all groups) | ED-1.9 |
| Ctrl+K Ctrl+Shift+W (closeAllGroups) | DL-2.3 |
| Ctrl+K Ctrl+T (Color Theme only) | AB-4.20 |
| Ctrl+K Ctrl+T (workbench.action.selectTheme) | ED-8.15 |
| Ctrl+K Ctrl+U (remove line comment) | ED-2.16 |
| Ctrl+K Ctrl+Up | ED-1.22 |
| Ctrl+K Ctrl+W (close all editors) | ED-1.9 |
| Ctrl+K Ctrl+W (closeAllEditors) | DL-2.3 |
| Ctrl+K Ctrl+X (trim trailing whitespace) | ED-2.15 |
| Ctrl+K D | ED-6.10 |
| Ctrl+K Down (move group) | ED-1.18 |
| Ctrl+K Down Arrow | SB-1.15 |
| Ctrl+K E (workbench.files.action.focusOpenEditorsView | SB-2.30 |
| Ctrl+K Enter | ED-1.6 |
| Ctrl+K F (closeFolder) | DL-2.5 |
| Ctrl+K F12 (editorTextFocus) | ED-4.3 |
| Ctrl+K F2 (Cmd+K F2 on macOS), when a quick diff peek is visible | ED-6.48 |
| Ctrl+K F2 (toggle focus) | ED-4.10 |
| Ctrl+K Left | ED-1.18 |
| Ctrl+K Left Arrow | SB-1.15 |
| Ctrl+K M (change language), when notebookEditorFocused && editable | ED-7.7 |
| Ctrl+K M (not while a notebook editor has focus) | SS-3.17 |
| Ctrl+K O (Copy Editor into New Window) | ED-1.33 |
| Ctrl+K P (workbench.action.files.copyPathOfActiveFile) | SB-2.16 |
| Ctrl+K R (workbench.action.files.revealActiveFileInWindows) | SB-2.15 |
| Ctrl+K Right | ED-1.18 |
| Ctrl+K Right Arrow (when a view has focus | SB-1.15 |
| Ctrl+K S on Linux and Cmd+K S on macOS | WB-1.4 |
| Ctrl+K Shift+Enter (pins when the tab is not pinned, unpins when it is) | ED-1.7 |
| Ctrl+K T (collapse or expand output), when notebookCellListFocused && !inputFocus | ED-7.11 |
| Ctrl+K U (close saved) | ED-1.9 |
| Ctrl+K Up | ED-1.18 |
| Ctrl+K Up Arrow | SB-1.15 |
| Ctrl+K V (when editorFocus and the editor language is Markdown and !notebookEditorFocused) | ED-10.9 |
| Ctrl+K W (close all in group) | ED-1.9 |
| Ctrl+K W (closeEditorsInGroup) | DL-2.1 |
| Ctrl+K Y (toggle scroll cell output), when notebookCellListFocused && !inputFocus && notebookCellHasOutputs | ED-7.22 |
| Ctrl+K Z (toggle) | ED-1.34 |
| Ctrl+K Z (workbench.action.toggleZenMode) | NT-4.10 |
| Ctrl+L (center active cell) | ED-7.9 |
| Ctrl+L (Cmd+L on Mac), when a Browser editor is active | ED-10.81 |
| Ctrl+L (expandLineSelection, textInputFocus) | ED-2.6 |
| Ctrl+L (when the active editor is the Browser editor and no quick input is open) | ED-10.31 |
| Ctrl+Left (list.collapseAll) | SB-2.2 |
| Ctrl+Left and Ctrl+Right (by word | ED-2.9 |
| Ctrl+M (Ctrl+Shift+M on macOS) on editor.action.toggleTabFocusMode | PN-3.45 |
| Ctrl+M (mac Ctrl+Shift+M) | SS-3.24 |
| Ctrl+N (Cmd+N on macOS) on workbench.action.files.newUntitledFile | WB-1.5 |
| Ctrl+N (Mac, Control key, when inQuickOpen) | QA-5.14 |
| Ctrl+N or Cmd+N (reject partial) | ED-6.31 |
| Ctrl+Numpad0 (resetZoom) | ED-10.76 |
| ctrl+numpad0 for reset | SS-5.13 |
| Ctrl+O (not macOS native) | DL-3.1 |
| Ctrl+O (not macOS, when remoteFileDialogVisible) | DL-3.19 |
| Ctrl+P | QA-2.1 |
| Ctrl+P (Mac, Control key, when inQuickOpen) | QA-5.14 |
| Ctrl+P and Ctrl+E for next, Ctrl+Shift+P and Ctrl+Shift+E for previous (Go to File, when inQuickOpen && inFilesPicker | QA-5.11 |
| Ctrl+PageDown (next) | ED-1.29, PN-6.16 |
| Ctrl+PageDown on workbench.action.terminal.focusNext and Ctrl+PageUp on workbench.action.terminal.focusPrevious (when terminalFocus and not terminalEditorFocus | PN-2.20 |
| Ctrl+PageUp (previous) | ED-1.29, PN-6.16 |
| Ctrl+Q (workbench.action.quickOpenView | QA-4.1 |
| Ctrl+Q and Ctrl+Shift+Q (Open View, when inQuickOpen && inViewsPicker | QA-5.11 |
| Ctrl+R (when the active editor is the Browser editor) | ED-10.33 |
| Ctrl+R (workbench.action.openRecent | DL-2.14 |
| Ctrl+R (workbench.action.openRecent, Ctrl+R on macOS too, using the Control key) | TB-1.14 |
| Ctrl+R and Ctrl+Shift+R (Open Recent, when inQuickOpen && inRecentFilesPicker | QA-5.11 |
| Ctrl+R and F5 (reload, when browserFocused) | ED-10.79 |
| Ctrl+R in development builds only (reloadWindow) | DL-2.5 |
| Ctrl+R on runRecentCommand in screen reader mode | PN-3.24 |
| Ctrl+R on runRecentCommand when screen reader mode is on and the terminal or accessible terminal view has focus | PN-3.12 |
| Ctrl+R on workbench.action.openRecent (the Control key on macOS too) | WB-9.7 |
| Ctrl+Right (Cmd+Right on Mac) when the editor is writable and (inlineSuggestionVisible or cppSuggestion) | AC-6.16 |
| Ctrl+Right (Cmd+Right on macOS) when editor writable, inlineSuggestionVisible, cursorBeforeGhostText, and accessibility mode off | AC-2.11 |
| Ctrl+RightArrow and Ctrl+LeftArrow (Alt on macOS) to move and open the picker | ED-5.9 |
| Ctrl+S | DL-3.7, DL-4.14 |
| Ctrl+Shift+ | ED-5.9 |
| Ctrl+Shift+, (cursor.openVSCodeSettingsFromMenu, when not in the Agents window and a focus condition that was not resolved) | ED-8.21 |
| Ctrl+Shift+, (replace with previous value) | ED-2.21 |
| Ctrl+Shift+-, when editorTextFocus && inputFocus && notebookEditorFocused && !notebookOutputFocused | ED-7.40 |
| Ctrl+Shift+. (focus and select) | ED-5.9 |
| Ctrl+Shift+. (replace with next value) | ED-2.21 |
| Ctrl+Shift+. for breadcrumbs.toggleToOn when breadcrumbs.enabled is off | ED-5.7 |
| Ctrl+Shift+1 (all three, when the Replace row is active and the matching row kind is focused) | SB-3.7 |
| Ctrl+Shift+1 when the find widget is visible | ED-3.15 |
| Ctrl+Shift+2 sends NUL | PN-3.36 |
| Ctrl+Shift+5 on workbench.action.terminal.split (when terminalFocus | PN-2.21 |
| Ctrl+Shift+5 on workbench.action.terminal.splitActiveTab (when terminalTabsFocus | PN-2.21 |
| Ctrl+Shift+6 sends RS | PN-3.36 |
| Ctrl+Shift+[ (fold | ED-5.11, ED-7.12 |
| Ctrl+Shift+\ (go to bracket) | ED-2.10 |
| Ctrl+Shift+\ on workbench.action.terminal.focusTabs (when terminalTabsFocus or terminalFocus) | PN-2.18 |
| Ctrl+Shift+] (unfold | ED-5.11, ED-7.12 |
| Ctrl+Shift+` on workbench.action.terminal.new (Mac Ctrl+Shift+`) | PN-2.2 |
| Ctrl+Shift+A (Cmd+Shift+A on macOS) when a Browser editor is active | ED-10.91 |
| Ctrl+Shift+A (Cmd+Shift+A on macOS), when screen reader mode is on and (notificationFocus or notificationToastsVisible) | NT-2.17 |
| Ctrl+Shift+Alt+` on workbench.action.terminal.newInNewWindow (Mac Ctrl+Shift+Alt+`) | PN-2.6 |
| Ctrl+Shift+Alt+C (copyRelativeFilePath, no editor focus | SB-2.16 |
| Ctrl+Shift+Alt+Left, Right, Up, Down, PageUp, PageDown (textInputFocus | ED-2.5 |
| Ctrl+Shift+B (workbench.action.tasks.build | QA-4.11 |
| Ctrl+Shift+Backspace (editor text focus, only while a diff is actively generating) | ED-6.32 |
| Ctrl+Shift+Backspace (in the Search Editor) | ED-3.37 |
| Ctrl+Shift+Backspace (Undo Edits, editor text focus, when the active editor has inline diffs) | ED-6.30 |
| Ctrl+Shift+C (Cmd+C on macOS) on copySelection and Ctrl+C on Windows on copyAndClearSelection, when text is selected and the terminal has focus | PN-3.15 |
| Ctrl+Shift+C on workbench.action.terminal.new (web build only, when terminal is not focused) | PN-2.2 |
| Ctrl+Shift+C on workbench.action.terminal.openNativeConsole (when terminal is not focused | PN-2.41 |
| Ctrl+Shift+D (Cmd+Shift+D on macOS) | SB-5.1 |
| Ctrl+Shift+D (workbench.view.debug) | AB-1.9 |
| Ctrl+Shift+E (workbench.view.explorer) | AB-1.9, SB-2.1 |
| Ctrl+Shift+Enter (insert line above) | ED-2.13 |
| Ctrl+Shift+Enter (list.toggleSelection) | SB-2.41 |
| Ctrl+Shift+Enter (Replace All in a file or folder) | SB-3.7 |
| Ctrl+Shift+F (Cmd+Shift+F on macOS) when the terminal has focus, a process is supported and text is selected | PN-3.2 |
| Ctrl+Shift+F (workbench.action.findInFiles) | SB-3.1 |
| Ctrl+Shift+F12 for Peek Implementations | ED-4.9 |
| Ctrl+Shift+F2 (start linked editing) | ED-2.27 |
| Ctrl+Shift+F3 (editor focus) | ED-3.8 |
| Ctrl+Shift+F5 (restart) | ED-11.17 |
| Ctrl+Shift+F5 (Restart, in debug mode) | SB-5.9 |
| Ctrl+Shift+G (Cmd+Shift+G on macOS) on openDetectedLink when the accessible view shows the terminal | PN-3.5 |
| Ctrl+Shift+G (Control+Shift+G on Mac) opens the view | SB-4.1 |
| Ctrl+Shift+G (workbench.view.scm) | AB-1.9 |
| Ctrl+Shift+H (workbench.action.replaceInFiles) | SB-3.6 |
| Ctrl+Shift+I | ED-10.35 |
| Ctrl+Shift+I (Cmd+Option+I on macOS) | WB-8.14 |
| Ctrl+Shift+I (toggle inline coverage) | ED-13.5 |
| Ctrl+Shift+J (aiSettings.action.open) | ED-8.19 |
| Ctrl+Shift+J (Cmd+Shift+J on macOS) | TB-5.6 |
| Ctrl+Shift+J when the Search view is focused (also works in the Search Editor) | SB-3.10 |
| Ctrl+Shift+K (delete line) | ED-2.12 |
| Ctrl+Shift+L (in the Search Editor) | ED-3.38 |
| Ctrl+Shift+L (select all matches) | ED-7.32 |
| Ctrl+Shift+L (select all occurrences) | ED-2.4 |
| Ctrl+Shift+L when the Search view is visible and a file or match row is focused | SB-3.22 |
| Ctrl+Shift+Left on workbench.action.terminal.resizePaneLeft and Ctrl+Shift+Right on workbench.action.terminal.resizePaneRight (Linux only, when terminalFocus) | PN-2.23 |
| Ctrl+Shift+M (workbench.actions.view.problems) | PN-4.1 |
| Ctrl+Shift+O | QA-3.14 |
| Ctrl+Shift+O (Cmd+Shift+O on macOS) on openDetectedLink when the terminal has focus | PN-3.5 |
| Ctrl+Shift+O (when the accessible view and accessibility help are not shown) | QA-3.10 |
| Ctrl+Shift+O and Ctrl+Shift+. (when the accessible view or help is shown and the page supports symbols) | WB-4.4 |
| Ctrl+Shift+P | AB-4.14 |
| Ctrl+Shift+P (Cmd+Shift+P on Mac, not bound on Firefox) | QA-1.1 |
| Ctrl+Shift+P (opens the palette from any list except Go to File, where the same keys move up the list, see QA-5.11) | QA-5.10 |
| Ctrl+Shift+PageUp and Ctrl+Shift+PageDown (on Mac the chords Cmd+K Cmd+Shift+Left and Cmd+K Cmd+Shift+Right) | ED-1.12 |
| Ctrl+Shift+R (Cmd+Shift+R on macOS), when the context key composerFocused is set | ED-6.52 |
| Ctrl+Shift+R (hardReload, when browserFocused) | ED-10.79 |
| Ctrl+Shift+R (textInputFocus | ED-4.15 |
| Ctrl+Shift+R on workbench.action.tasks.rerunForActiveTerminal (when terminalFocus | PN-2.37 |
| Ctrl+Shift+R when in the Search Editor | ED-3.35 |
| Ctrl+Shift+S | DL-3.6 |
| Ctrl+Shift+S (capture a screenshot) and Ctrl+Shift+R (start or stop recording), Cmd instead of Ctrl on macOS, only while an issue reporter wizard tab is open | WB-6.5 |
| Ctrl+Shift+S (workbench.action.issueReporter.captureScreenshot) and Ctrl+Shift+R (workbench.action.issueReporter.toggleRecording), Cmd on macOS, both only while an Issue Reporter tab is open | WB-6.5 |
| Ctrl+Shift+S or Cmd+Shift+S (when remoteFileDialogVisible) | DL-3.19 |
| Ctrl+Shift+Space (Cmd+Shift+Space on macOS) when editorTextFocus | AC-3.1, AC-3.12 |
| Ctrl+Shift+T | ED-1.10 |
| Ctrl+Shift+T (toggle between inputs, in the merge editor) | ED-6.21 |
| Ctrl+Shift+T (workbench.action.reopenClosedEditor) | TB-1.14 |
| Ctrl+Shift+Tab (Control+Tab and Control+Shift+Tab on Mac) | ED-1.30 |
| Ctrl+Shift+U (Linux: Ctrl+K Ctrl+H) | PN-5.1 |
| Ctrl+Shift+Up (focus out of output) | ED-7.9 |
| Ctrl+Shift+Up on selectToPreviousCommand and Ctrl+Shift+Down on selectToNextCommand (Cmd+Shift on macOS), when the terminal has focus | PN-3.9 |
| Ctrl+Shift+V (paste above) | ED-7.5 |
| Ctrl+Shift+V (when markdownEditorActive, shared with the Markdown Editor) | ED-10.24 |
| Ctrl+Shift+V on markdownEditor.toggleMode (when markdownEditorActive) | ED-10.23 |
| Ctrl+Shift+V on paste (secondary on Windows, primary on Linux) | PN-3.16 |
| Ctrl+Shift+W (Windows and Linux) | DL-2.18 |
| Ctrl+Shift+X (workbench.view.extensions) | AB-1.9 |
| Ctrl+Shift+Y (workbench.debug.action.toggleRepl) | PN-6.1 |
| Ctrl+Shift+Y on Windows, Cmd+Y on macOS and Ctrl+Y on Linux (accept partial) | ED-6.31 |
| Ctrl+Space | AC-1.1, AC-1.21, AC-1.29, AC-1.37 |
| Ctrl+Space (Control+Space on Mac | QA-6.14 |
| Ctrl+Space (mac: Ctrl+Space) when terminalFocus and terminalSuggestWidgetVisible is false and config.terminal.integrated.suggest.enabled is true | AC-5.5 |
| Ctrl+Space (triggerSuggest) | ED-10.21 |
| Ctrl+Space or Ctrl+I (mac: Ctrl+Space or Cmd+I) when terminalFocus, terminalIsOpen, terminalSuggestWidgetVisible and simpleSuggestWidgetHasFocusedSuggestion | AC-5.13 |
| Ctrl+Space, Alt+Space and Shift+Enter in PowerShell with shell integration and win32 input mode | PN-3.36 |
| Ctrl+T | QA-2.12 |
| Ctrl+T (macOS, transposeLetters) | ED-2.21 |
| Ctrl+T (when a Browser editor is active) | ED-10.80 |
| Ctrl+T (when the active editor is the Browser editor) | ED-10.29 |
| Ctrl+Tab | ED-1.30 |
| Ctrl+Tab and Ctrl+Shift+Tab (Control+Tab and Control+Shift+Tab on Mac | QA-4.2 |
| Ctrl+Tab and Ctrl+Shift+Tab (editor switcher, when inQuickOpen && inEditorsPicker | QA-5.11 |
| Ctrl+U (cursor undo) | ED-2.31 |
| Ctrl+Up (focus history), all when the active editor is the Interactive Window | ED-7.37 |
| Ctrl+Up (list.scrollUp, list focused and not scrolled to the top) | SB-2.42 |
| Ctrl+Up (macOS Ctrl+PageUp) for scrollLineUp | ED-5.30 |
| Ctrl+Up (search.action.focusSearchFromResults, when the first match is focused or accessibility mode is on) | SB-3.26 |
| Ctrl+Up (search.focus.previousInputBox, in an input box other than the search box) | SB-3.26 |
| Ctrl+Up (search.focus.previousInputBox, same condition) | ED-3.47 |
| Ctrl+Up (widgetNavigation.focusPrevious, same container condition and the list is scrolled to the top) | WB-3.1 |
| Ctrl+Up on scrollToPreviousCommand and Ctrl+Down on scrollToNextCommand (Cmd on macOS), when the terminal has focus and screen reader mode is off | PN-3.8 |
| Ctrl+V | ED-7.5 |
| Ctrl+V (browser paste event while the tree has focus, no command key rule) | SB-2.47 |
| Ctrl+V (Cmd+V on macOS) on paste | PN-3.16 |
| Ctrl+V (filesExplorer.paste) | SB-2.9 |
| Ctrl+V on pastePwsh (Windows, PowerShell, screen reader mode off) | PN-3.16 |
| Ctrl+V or Cmd+V (Explorer focused) | DL-4.9, DL-4.12 |
| Ctrl+W (Cmd+W on Mac) | AX-3.2 |
| Ctrl+W (on Windows Ctrl+F4, with Ctrl+W as a second key) | ED-1.8 |
| Ctrl+W on an empty group closes the group | ED-1.9 |
| Ctrl+W on macOS (the Control key) on workbench.action.switchWindow | WB-9.8 |
| Ctrl+W on workbench.action.terminal.killEditor (when terminalFocus and terminalEditorFocus | PN-2.27 |
| Ctrl+W, or Cmd+W on macOS (closeActiveEditor | DL-2.1 |
| Ctrl+X | ED-7.5 |
| Ctrl+X (filesExplorer.cut) | SB-2.9 |
| Ctrl+X or Cmd+X (Explorer focused) | DL-4.9 |
| Ctrl+X, Ctrl+C, Ctrl+V | ED-2.28 |
| Ctrl+Y and Ctrl+Shift+Z (redo | ED-2.31, ED-7.41 |
| Ctrl+Y or Ctrl+Shift+Z (Explorer focused, redo) | DL-4.13 |
| Ctrl+Y or Ctrl+Shift+Z (redo | SB-2.10 |
| Ctrl+Y or Ctrl+Shift+Z (Redo) | ED-4.41 |
| Ctrl+Z (Explorer focused) | DL-4.13 |
| Ctrl+Z (undo) | SB-2.10, ED-2.31, ED-7.41 |
| Ctrl+Z (Undo) | ED-4.41 |
| Ctrl/Cmd+ | SB-6.6, SB-6.8, SB-6.9, SB-6.13, SB-6.14, SB-6.21 |
| CtrlCmd+Right (editor.action.inlineSuggest.acceptNextWord, when editor.writable and (inlineSuggestionVisible or cppSuggestion)) | AC-8.11, AC-8.17 |
| Cursor: only when the editor does not have text focus, weight 199) | WB-3.5 |
| D D | ED-7.40 |
| Delete (Cmd+Backspace on macOS | PN-7.12 |
| Delete (Cmd+Backspace on macOS) when the Search view is visible and a result row is focused | SB-3.23 |
| Delete (Cmd+Backspace on macOS), when notebookEditorFocused && !inputFocus && !notebookOutputInputFocused | ED-7.3 |
| Delete (Cmd+Backspace on macOS, Watch focused and no input open) | SB-5.18 |
| Delete (Cmd+Backspace on macOS, when the Breakpoints view is focused and no condition box has focus) | SB-5.25 |
| Delete (deleteFile, when trash is unavailable | SB-2.8 |
| Delete (moveFileToTrash | SB-2.8 |
| Delete (notification.clear, when a notification has focus | NT-5.4 |
| Delete (when notificationFocus) | NT-2.14 |
| Delete (when the Explorer is focused and the item can go to trash) | DL-4.1 |
| Delete (when the item cannot go to trash) | DL-4.2 |
| Delete on workbench.action.terminal.killActiveTab (when terminalTabsFocus | PN-2.28 |
| Delete, or Cmd+Backspace on macOS (remove, when the table has focus) | ED-8.11 |
| Down | AC-1.30, AC-5.7, NT-1.10 |
| Down (scm.viewNextCommit, when scmInputIsInLastPosition) | SB-4.13 |
| Down (settings.action.focusSettingsFile and settings.action.focusSettingsFromSearch, when the search box has focus and no suggestion is open) | ED-8.43 |
| Down (when a list has focus) | NT-3.10 |
| Down and Up (also Ctrl+Down and Ctrl+Up) | QA-6.21 |
| Down and Up to move through name suggestions | ED-4.12 |
| Down Arrow | SB-1.23, PN-6.4, QA-3.13 |
| Down Arrow (with a view header focused) | SB-1.7 |
| Down Arrow and Alt+Down Arrow (history.showNext) | SB-3.5 |
| Down or Ctrl+Down (mac adds Ctrl+N) | AC-1.36 |
| Down or Enter (focus first setting) | ED-8.2 |
| DownArrow | TB-1.8, TB-1.10, TB-1.12, TB-1.13 |
| DownArrow (history.showNext, history widget focused, not composing) | ED-3.21 |
| DownArrow (select next line) | ED-5.6 |
| duplicateSelection has no default key | ED-2.11 |
| each when activeCustomEditorId == 'vscode.markdown.editor' and markdownEditorFocus | ED-10.21 |
| each when browserFocused | ED-10.76 |
| editor text focus | ED-5.12, ED-5.13, ED-5.14, ED-5.15, ED-5.33 |
| editorTextFocus) | ED-4.17 |
| End | AB-1.12, SB-1.23, PN-1.10 |
| End (last tab) | ED-1.45 |
| End (lastCompressedFolder) | SB-2.21 |
| End when statusBarFocused (focusLast) | SS-1.8 |
| Enter | TB-1.8, TB-1.10, TB-1.13, AB-1.12, SB-1.7, SB-1.23, ED-4.12, ED-5.9, PN-1.10, QA-2.22, QA-3.7, QA-6.4, DL-1.12, DL-1.22, DL-1.25 |
| Enter (accept while a field has focus) | DL-1.7 |
| Enter (accept, when the When cell has focus and no suggestion is open) | ED-8.12 |
| Enter (acceptSelectedSuggestion) when simpleSuggestWidgetHasFocusedSuggestion and (simpleSuggestWidgetExplicitlyInvoked, or selectionMode is not partial, or the first row is not focused, or simpleSuggestWidgetHasNavigated) | AC-5.9 |
| Enter (acceptSelectedSuggestionEnter) when config.terminal.integrated.suggest.runOnEnter is not never | AC-5.9 |
| Enter (also Cmd+Down on macOS) | SB-3.21 |
| Enter (and Alt, Ctrl, Cmd or Shift with Enter | QA-6.22 |
| Enter (breakpointWidget.action.acceptInput, when the breakpoint widget input has focus) | ED-11.5 |
| Enter (Close) | DL-2.18 |
| Enter (copy version info and close) | DL-1.20 |
| Enter (Cursor About window: copy and close) | DL-1.18 |
| Enter (default button) | DL-1.3, DL-1.16 |
| Enter (edit), when notebookCellListFocused && !inputFocus | ED-7.8 |
| Enter (execute, when it is false) | ED-7.37 |
| Enter (execute, when it is false), when the active editor is the REPL editor and the cell list is not focused | ED-7.39 |
| Enter (explorer.openAndPassFocus, when a non-folder is focused | SB-2.3 |
| Enter (findNext) and Shift+Enter (findPrevious) when the find widget is focused | ED-10.7 |
| Enter (findNext, when browserFindWidgetFocused) | ED-10.77 |
| Enter (focus settings list) | ED-8.4 |
| Enter (focus the setting control) | ED-8.5 |
| Enter (go to focused line) | ED-5.6 |
| Enter (keybindings.editor.defineKeybinding, when a row has focus) | ED-8.11 |
| Enter (list.stickyScrollselect | SB-1.24 |
| Enter (log in) | DL-1.8 |
| Enter (on the More button, opens it) | TB-1.12 |
| Enter (open the focused result) | ED-4.10 |
| Enter (primary button) | DL-1.1 |
| Enter (Quit) | DL-2.17 |
| Enter (run now) | SB-3.2 |
| Enter (settings.action.focusSettingControl, when a row has focus) | ED-8.43 |
| Enter (settings.action.focusSettingsFromSearchOnEnter, same condition) | ED-8.43 |
| Enter (settings.action.focusSettingsList, when a tree row has focus) | ED-8.43 |
| Enter (standaloneColorPickerFocused) | ED-4.30 |
| Enter (the list's own open key | PN-4.17 |
| Enter (when a problem row is focused) | PN-4.10 |
| Enter (when text input focus and in the debug console) | PN-6.3 |
| Enter and Shift+Enter inside the find input | PN-3.1 |
| Enter in the focused tree (hard-coded key handler, not a keybinding rule) | SB-6.7 |
| Enter is ignored while an input method is composing) | QA-6.22 |
| Enter on the box (default button) | ED-10.71 |
| Enter or Ctrl+. applies | ED-4.14 |
| Enter or Escape closes it | DL-1.4 |
| Enter or Space (open the focused tab) | ED-1.45 |
| Enter or Space (while the button has focus) | TB-2.9 |
| Enter or Space on a focused item | SS-1.9 |
| Enter or Space while the button has focus | TB-2.9 |
| Enter when suggestWidgetVisible, editorTextInputFocus, acceptSuggestionOnEnter and suggestionMakesTextEdit | AC-1.32 |
| Enter when the find box is focused | ED-3.6 |
| Enter when the replace box is focused | ED-3.15 |
| Esc (all with a list focused) | SB-1.23 |
| Esc on clearSelection when text is selected and the find box is not visible | PN-3.19 |
| Esc on hideFind while the find box is visible | PN-3.1 |
| Escape | TB-1.10, ED-3.22, ED-5.9, AC-1.38, AC-3.10, AC-4.7, AC-7.19, QA-2.19, QA-3.5, QA-3.13, QA-5.15, DL-1.12, DL-1.25 |
| Escape (and Alt, Ctrl or Cmd with Escape) | QA-6.22 |
| Escape (bannerFocused) | BN-1.7 |
| Escape (callHierarchyVisible, unless editor.stablePeek) closes | ED-4.31 |
| Escape (cancel anchor) | ED-2.8 |
| Escape (cancel button) | DL-1.3 |
| Escape (cancel unless sticky) | DL-1.16 |
| Escape (cancel) | DL-1.8, DL-2.8 |
| Escape (Cancel) | DL-2.17, DL-2.18 |
| Escape (cancel) - both only while the in-page box is open, see DL-1.12 | DL-1.1 |
| Escape (cancel, while the linked input is active) | ED-2.27 |
| Escape (clear search, when the box has text) | ED-8.9 |
| Escape (close the peek) | ED-6.25 |
| Escape (close) | ED-1.36, DL-1.20 |
| Escape (close, in the peek) | ED-13.4 |
| Escape (closes the margin hover) | ED-6.58 |
| Escape (Cursor About window: close) | DL-1.18 |
| Escape (diffEditor.exitCompareMove, only while comparing moved code) | ED-6.5 |
| Escape (editor.closeTestPeek, when editor.stablePeek is off) | ED-13.4 |
| Escape (exit) | ED-7.32 |
| Escape (filesExplorer.cancelCut, when something is cut) | SB-2.9 |
| Escape (hasSymbols) | ED-4.33 |
| Escape (hide find), when the find widget is visible | ED-7.30 |
| Escape (hide the widget) | ED-2.29 |
| Escape (hideFind, when browserFindWidgetVisible) | ED-10.77 |
| Escape (hideFind, when the find widget is visible) | ED-10.7 |
| Escape (isReadingLineWithInlayHints) stops reading | ED-4.21 |
| Escape (leave recording) | ED-8.10 |
| Escape (list.clear, when something is selected or focused) | SB-2.41 |
| Escape (list.closeFind, tree find box open) | SB-2.26 |
| Escape (menu bar focused, no menu open) | TB-1.8 |
| Escape (messageVisible) | ED-4.34 |
| Escape (move focus up one level) | ED-8.4 |
| Escape (no Ctrl, Alt or Cmd) | AC-7.13, AC-7.24 |
| Escape (no Ctrl, Alt or Cmd) while a Tab suggestion shows | AC-6.17 |
| Escape (notifications.hideToasts, when notificationToastsVisible) | NT-5.7 |
| Escape (quit edit), when notebookEditorFocused && inputFocus | ED-7.8 |
| Escape (reject) | ED-8.12 |
| Escape (removeSecondaryCursors, when there is more than one cursor) | ED-2.1 |
| Escape (same result, see DL-1.12) | DL-1.14 |
| Escape (scm.clearInput, when no suggest widget and no validation message) | SB-4.7 |
| Escape (scm.clearInput, when scmRepository, suggest list hidden, no inline suggestion visible, no validation message, no non-empty selection) | AC-2.45 |
| Escape (scm.clearValidation, when a validation message is shown) | SB-4.7 |
| Escape (search.action.cancel, when the Search view is visible and a list has focus) | SB-3.27 |
| Escape (see DL-1.25) | DL-1.22 |
| Escape (see QA-6.22) | QA-6.4 |
| Escape (select editor) | ED-5.6 |
| Escape (settings.action.focusLevelUp, when focus is not in the search box or a JSON editor) | ED-8.43 |
| Escape (standaloneColorPickerVisible) | ED-4.30 |
| Escape (stop a running search, from the box) | SB-3.2 |
| Escape (stop, while dictation is running) | ED-2.38 |
| Escape (typeHierarchyVisible, unless editor.stablePeek) closes | ED-4.32 |
| Escape (welcome.goBack, when inWelcomeContext) | ED-9.1, ED-9.8 |
| Escape (when editorTextFocus, same weight as closing the suggest list) | AC-2.13 |
| Escape (when notificationCenterVisible) | NT-3.7 |
| Escape (when notificationToastsVisible && notificationFocus, high weight) | NT-1.8 |
| Escape (when notificationToastsVisible, low weight so editor widgets win first) | NT-1.8 |
| Escape (when the exception widget is visible) | ED-11.16 |
| Escape (when the filter box is focused) | PN-4.5, PN-5.5 |
| Escape (when the filter has focus, clears it) | PN-7.3 |
| Escape and Shift+Escape (closeBreakpointWidget, when text input has focus) | ED-11.5 |
| Escape and Shift+Escape (textInputFocus, when the selection is not empty | ED-2.41 |
| Escape and Shift+Escape (workbench.action.hideComment, when the comment editor or a comment has focus) | ED-12.5 |
| Escape Escape (exit, only while in Zen Mode) | ED-1.34 |
| Escape Escape (workbench.action.exitZenMode, when InEditorZenModeContext) | NT-4.10 |
| Escape for Focus Search Editor Input (in the Search Editor) | ED-3.31 |
| Escape or Shift+Escape | ED-4.12 |
| Escape or Shift+Escape (close, unless editor.stablePeek) | ED-4.10 |
| Escape or Shift+Escape (markersNavigationVisible) | ED-4.27 |
| Escape or Shift+Escape hides | ED-4.14 |
| Escape or Shift+Escape inside the inline edits preview editor | AC-2.13 |
| escape when NotificationsCenterVisibleContext (notifications.hideList) | SS-5.2 |
| Escape when statusBarFocused (clearFocus) | SS-1.8 |
| Escape when terminalFocus and terminalSuggestWidgetVisible | AC-5.12 |
| Escape when the Replace box is focused (closeReplaceInFilesWidget) | SB-3.6 |
| Escape when the search box has focus (clear results) | ED-8.2 |
| Escape, Left or Right while an item with a command has focus (closes the hover) | SS-1.10 |
| every command has a Select twin such as cursorLeftSelect) | ED-2.9 |
| F | ED-7.40 |
| F1 | AB-4.14, QA-1.1, QA-5.10 |
| F10 (step over) | ED-11.17 |
| F10 (Step Over, when stopped) | SB-5.9 |
| F11 | TB-3.13 |
| F11 (step into) | ED-11.17 |
| F11 (Step Into, when a session is active | SB-5.9 |
| F12 | ED-10.35, ED-10.78 |
| F12 (editorTextFocus) | ED-4.3 |
| F12 (hasSymbols) | ED-4.33 |
| F2 (editorTextFocus) | ED-4.12 |
| F2 (Enter on macOS | PN-7.15 |
| F2 (Enter on macOS, when the Variables view is focused) | SB-5.12 |
| F2 (Enter on macOS, when the Watch view is focused) | SB-5.18 |
| F2 (rename) | ED-10.21 |
| F2 (renameFile, files Explorer focused, item not a root and writable | SB-2.7 |
| F2 on workbench.action.terminal.renameActiveTab (when terminalTabsFocus | PN-2.11 |
| F3 | SB-1.23, ED-3.23 |
| F3 (Cmd+G on macOS) on findNext | PN-3.1 |
| F3 (Cmd+G on macOS, F3 also) | ED-3.6 |
| F3 (findNext, when browserFindWidgetVisible) | ED-10.77 |
| F3 (next match, Cmd+G on macOS) | ED-6.38 |
| F3 and Shift+F3 (Cmd+G and Cmd+Shift+G on macOS), when the find widget is focused | ED-7.30 |
| F4 | ED-3.46 |
| F4 (search.action.focusNextSearchResult) | SB-3.26 |
| F4 (when reference-list.hasResult) | SB-7.23 |
| F4 or F12 (next result, referenceSearchVisible or inReferenceSearchEditor) | ED-4.10 |
| F5 (continue) | ED-11.17 |
| F5 (Continue, when stopped) | SB-5.9 |
| F5 (when debuggersAvailable and debugState is inactive) | SB-5.3 |
| F5 and Ctrl+F5 when no debugger extension exists (opens this view) | SB-5.3 |
| F5 from inside the page | ED-10.33 |
| F6 (Focus Next Part) | SS-1.7 |
| F6 (pause) | ED-11.17 |
| F6 (Pause, when running) | SB-5.9 |
| F7 (next difference) and Shift+F7 (previous difference) in a diff editor | ED-6.6 |
| F7 (next) | ED-5.33 |
| F7 and Shift+F7 (editorTextFocus, when highlights exist) | ED-4.22 |
| F8 and Shift+F8 (editor focus, across files) | ED-4.27 |
| F9 (editor.debug.action.toggleBreakpoint, when the disassembly view is focused) | ED-10.89 |
| F9 (when editorTextFocus or the disassembly view is focused) | ED-11.1 |
| first and last have none | AC-1.36 |
| Focus Editor Sticky Scroll has no default key | ED-5.6 |
| Focus Status Bar has no default key | SS-1.7 |
| focusAndAcceptSuggestion has none | AC-1.37 |
| Forward - Alt+Right (Windows), Ctrl+Shift+- (macOS and Linux), plus the browser Forward key | TB-2.4 |
| held key mode) | ED-2.38 |
| Home | AB-1.12, SB-1.23, PN-1.10 |
| Home (first tab) | ED-1.45 |
| Home (firstCompressedFolder) | SB-2.21 |
| Home and End | ED-2.9 |
| Home or Ctrl+Up and End or Ctrl+Down (hoverFocused) | ED-4.2 |
| Home when statusBarFocused (focusFirst) | SS-1.8 |
| Home, End, PageUp, PageDown (Linux) | TB-1.10 |
| in Cursor the key and the command are off while a separate auxiliary window has focus | AX-1.1 |
| Insert | ED-2.26 |
| Insert (mac Alt+Cmd+O) | SS-3.23 |
| J | ED-7.40 |
| K | ED-7.40 |
| L, all when notebookEditorFocused && !inputFocus && !notebookOutputFocused | ED-7.40 |
| Left (list.collapse) | SB-2.2 |
| Left (list.stickyScroll.collapse, pinned row focused | SB-1.24 |
| Left (previousCompressedFolder, compact row focused) | SB-2.21 |
| Left (settings.action.focusTOC, when a row has focus) | ED-8.43 |
| Left (when notificationFocus) | NT-2.7 |
| Left and Right arrows (rail focused) | AB-1.12 |
| Left and Right arrows or Tab (move between buttons), see DL-1.12 | DL-1.3 |
| Left Arrow | SB-1.7, SB-1.23, PN-1.10, DL-1.12 |
| Left Arrow (dropdown to button) | SB-5.4 |
| Left Arrow and Right Arrow (while the viewer has focus) | ED-10.44 |
| Left Arrow, Up Arrow (bannerFocused) | BN-1.7 |
| Left or Up (previous tab) | ED-1.45 |
| Left or Up when statusBarFocused (focusPrevious) | SS-1.8 |
| Left when a setting row has focus (focus the contents tree) | ED-8.4 |
| Left, Right and Space work on section headers | ED-4.14 |
| LeftArrow | TB-1.8, TB-1.10, ED-5.9 |
| LeftArrow (on the hamburger and the activity icons) | TB-1.13 |
| Linux also Shift+Insert (paste) | ED-2.28 |
| Linux Ctrl+Shift+A) | ED-2.16 |
| Linux Ctrl+Shift+Alt+Up and Down) | ED-2.11 |
| Linux Shift+Alt+Up and Shift+Alt+Down) | ED-2.1 |
| M (to markdown), when notebookEditorFocused && !inputFocus && !notebookOutputFocused | ED-7.7 |
| Mac Alt+Cmd with the arrow) | PN-2.22 |
| Mac Cmd+\ with Ctrl+Shift+5 as the second key) | PN-2.21 |
| Mac Cmd+\) | PN-2.21 |
| Mac Cmd+Backspace or Delete) | PN-2.28 |
| Mac Cmd+Shift+] and Cmd+Shift+[) | PN-2.20 |
| Mac Cmd+Shift+C) | PN-2.41 |
| Mac Ctrl+Shift+R) | PN-2.37 |
| Mac Enter) | PN-2.11 |
| Mac uses Cmd+P and Cmd+Shift+P only) | QA-5.11 |
| Mac uses Control+Q, none on Linux) | QA-5.11 |
| Mac uses Control+R) | QA-5.11 |
| Mac uses Control+Tab) | QA-5.11 |
| macOS also Cmd+Down) | SB-1.24 |
| macOS also Cmd+Up) | SB-1.24 |
| macOS Alt+Cmd+O | ED-2.26 |
| macOS Cmd+Alt+0) | SB-2.31 |
| macOS Cmd+Alt+Backspace) | SB-2.8 |
| macOS Cmd+Backspace (delete all left) and Ctrl+K (delete all right) | ED-2.12 |
| macOS Cmd+Backspace or Delete) | SB-2.8 |
| macOS Cmd+Backspace) | SB-2.8 |
| macOS Cmd+Down) | SB-2.3 |
| macOS Cmd+K E) | SB-2.30 |
| macOS Cmd+Shift+Z) | SB-2.10, ED-2.31 |
| macOS Control+Enter) | SB-2.4 |
| macOS Ctrl+J (join lines) | ED-2.13 |
| macOS Ctrl+O (lineBreakInsert) | ED-2.13 |
| macOS Enter) | SB-2.7 |
| middle click on a tab | ED-1.8 |
| mouse back and forward buttons | ED-1.32 |
| no `when`, weight 200 | SS-6.13 |
| no default key on Linux) | QA-4.1 |
| no key opens it | ED-4.32 |
| none for the two line commands | PN-3.9 |
| none on Windows and Linux | PN-3.34 |
| not a registered key binding | AC-6.17, AC-7.13, AC-7.24 |
| O | ED-7.40 |
| on Linux Ctrl+- and Ctrl+Numpad-) | TB-3.16 |
| on Linux the primary keys are Alt+Shift+F2 and Alt+Shift+F1 with the Alt-only keys as second choices | ED-4.39 |
| on Mac also Cmd+Alt+Right, Cmd+Alt+Left, Cmd+Shift+] and Cmd+Shift+[ | ED-1.29 |
| on Mac Cmd+Ctrl+Left, Right, 1 and 9 | ED-1.14 |
| on macOS Ctrl+Cmd+Shift+Right and Left | ED-2.7 |
| on Windows Ctrl+F4 with Ctrl+W as the second key) | PN-2.27 |
| only when editor.tabCompletion is on, at the end of a word, the list is hidden and no snippet is active | AC-1.39 |
| only while the editor is writable, has a completion provider and the list is hidden | AC-1.1 |
| only while the part is maximized | AX-3.2 |
| or inlineEditIsVisible with tabShouldAcceptInlineEdit and the same other guards) | AC-2.10 |
| PageDown | AC-5.7 |
| PageDown and PageUp | QA-6.21 |
| PageDown or Ctrl+PageDown | AC-1.36 |
| PageDown or End (all when notificationFocus && notificationToastsVisible) | NT-1.10 |
| PageUp | AC-5.7 |
| PageUp and PageDown | ED-2.9, ED-7.9 |
| PageUp and PageDown (jump to the start or end of the text) | SB-3.2 |
| PageUp or Alt+Up and PageDown or Alt+Down (hoverFocused) | ED-4.2 |
| PageUp or Ctrl+PageUp | AC-1.36 |
| PageUp or Home | NT-1.10 |
| problems.action.open needs a problem row focused) | PN-4.17 |
| quick access prefixes edt (appearance), edt mru (most recently used) and edt active (active group) | ED-1.31 |
| Reject All Edits has a key expression written with a key code where a modifier is expected, so its real binding is not resolved | ED-6.30 |
| Report Issue has a two key chord whose second key is Ctrl+G (Cmd+G on macOS) | WB-6.6 |
| Reset Zoom - Ctrl+Numpad0 | TB-3.16 |
| Right (list.expand) | SB-2.2 |
| Right (nextCompressedFolder) | SB-2.21 |
| Right (when notificationFocus) | NT-2.7 |
| Right Arrow | SB-1.7, SB-1.23, PN-1.10, DL-1.12 |
| Right Arrow (button to dropdown) | SB-5.4 |
| Right arrow (quickInput.acceptInBackground, when the box shows a quick pick and either the filter field is not focused or its caret is at the end) | QA-6.16 |
| Right Arrow (when in quick pick and focus is in the list or the cursor is at the end of the input box) | QA-2.8, QA-3.9 |
| Right Arrow, Down Arrow (bannerFocused) | BN-1.7 |
| Right or Down (next tab) | ED-1.45 |
| Right or Down when statusBarFocused (focusNext) | SS-1.8 |
| RightArrow | TB-1.8, TB-1.10, TB-1.13, ED-5.9 |
| Run Test Task has no default key | QA-4.11 |
| S | ED-7.40 |
| sent from the page preload | ED-10.32, ED-10.34 |
| Shift (held while the suggest list is open, flips insert and replace for the preview) | AC-2.44 |
| Shift held down (a key listener, not a keybinding rule | AC-1.44 |
| Shift held while following the link forces the system browser | PN-3.46 |
| Shift while dragging to drop into the editor | ED-1.15 |
| Shift+Alt+. for Auto Fix (Cmd+Alt+. on Mac) | ED-4.16 |
| Shift+Alt+0 (workbench.action.toggleEditorGroupLayout | SB-2.31 |
| Shift+Alt+1 (first group) | ED-1.14 |
| Shift+Alt+9 (last group) | ED-1.14 |
| Shift+Alt+A (toggle block comment | ED-2.16 |
| Shift+Alt+F (Ctrl+Shift+I on Linux | ED-4.17 |
| Shift+Alt+F (Ctrl+Shift+I on Linux), when the cell editor text is not focused (Format Notebook) and when it is focused (Format Cell) | ED-7.33 |
| Shift+Alt+F when a folder row with a path is focused (search.action.restrictSearchToFolder) | SB-3.15 |
| Shift+Alt+F when the Explorer is focused on a folder (filesExplorer.findInFolder) | SB-3.14 |
| Shift+Alt+F1 (Shift+Alt+F4 on Linux) | WB-4.8 |
| Shift+Alt+F10 | ED-12.6 |
| Shift+Alt+F12 (when editorHasReferenceProvider) | SB-7.20 |
| Shift+Alt+F3 (previous change), when the active editor is the Notebook Diff Editor | ED-7.36 |
| Shift+Alt+F8 (previous), when editor focus is in a cell | ED-7.44 |
| Shift+Alt+F8 (testing.goToPreviousMessage) | ED-13.4 |
| Shift+Alt+F9 | ED-12.6 |
| Shift+Alt+F9 (previous uncovered line) | ED-13.5 |
| Shift+Alt+H (editorTextFocus) opens it and, while the panel shows, swaps between incoming and outgoing calls (the References view command in SB-7.27 binds the same key) | ED-4.31 |
| Shift+Alt+H (when editorHasCallHierarchyProvider) | SB-7.27 |
| Shift+Alt+H (while the panel shows) swaps between supertypes and subtypes | ED-4.32 |
| Shift+Alt+I (cursors to line ends) | ED-2.1 |
| Shift+Alt+O for Organize Imports (textInputFocus) | ED-4.16 |
| Shift+Alt+Right (expand) and Shift+Alt+Left (shrink) | ED-2.7 |
| Shift+Alt+Up and Shift+Alt+Down (copy line | ED-2.11 |
| Shift+click (mouse gestures, no default key) | ED-1.11 |
| Shift+click extends the selection) | ED-2.40 |
| Shift+Cmd+] and Shift+Cmd+[ on macOS (when in the debug console) | PN-6.16 |
| Shift+Ctrl+V on markdown.showPreview in Cursor only (when editorLangId == markdown && !notebookEditorFocused) | ED-10.8 |
| Shift+Ctrl+V on markdown.togglePreview (when !terminalFocus and a Markdown-like text editor has focus, or activeCustomEditorId == 'vscode.markdown.preview.editor') | ED-10.18 |
| Shift+Delete | DL-4.2 |
| Shift+Delete (deleteFile | SB-2.8 |
| Shift+Down (list.expandSelectionDown, list supports multi-select) | SB-2.41 |
| Shift+Enter | AC-1.34, DL-1.22, DL-1.25 |
| Shift+Enter (callHierarchyVisible) refocuses | ED-4.31 |
| Shift+Enter (execute and select below), when notebookCellListFocused && !inlineChatFocused | ED-7.13 |
| Shift+Enter (execute, when interactiveWindow.executeWithShiftEnter is true) | ED-7.37, ED-7.39 |
| Shift+Enter (findPrevious, when browserFindWidgetFocused) | ED-10.77 |
| Shift+Enter (typeHierarchyVisible) refocuses | ED-4.32 |
| Shift+Enter when the find box is focused | ED-3.6 |
| Shift+Enter, same plus notebookCellType == 'code' | ED-7.40 |
| Shift+Escape (closeQuickOpen, when in a quick pick) | QA-2.19 |
| Shift+Escape (find widget visible, editor focus, not composing) | ED-3.22 |
| Shift+Escape (when inQuickOpen) | QA-5.15 |
| Shift+Escape (workbench.action.closeQuickOpen, when inQuickOpen) | QA-6.22 |
| Shift+Escape when editorFocus and parameterHintsVisible | AC-3.10 |
| Shift+Escape when editorTextInputFocus and the list is visible | AC-1.38 |
| Shift+Escape when inSnippetMode and textInputFocus (leaveSnippet) | AC-4.7 |
| Shift+F10 (editor.action.showContextMenu, when textInputFocus) | WB-7.2 |
| Shift+F10 (tab menu) | ED-1.45 |
| Shift+F11 (step out) | ED-11.17 |
| Shift+F11 (Step Out, when stopped) | SB-5.9 |
| Shift+F12 (editorTextFocus) | ED-4.8 |
| Shift+F3 | ED-3.23 |
| Shift+F3 (Cmd+Shift+G on macOS) on findPrevious | PN-3.1 |
| Shift+F3 (Cmd+Shift+G on macOS, Shift+F3 also) | ED-3.6 |
| Shift+F3 (findPrevious, when browserFindWidgetVisible) | ED-10.77 |
| Shift+F3 (previous match, Cmd+Shift+G on macOS) | ED-6.38 |
| Shift+F4 | ED-3.46 |
| Shift+F4 (search.action.focusPreviousSearchResult) | SB-3.26 |
| Shift+F4 (when reference-list.hasResult) | SB-7.23 |
| Shift+F4 or Shift+F12 (previous result) | ED-4.10 |
| Shift+F5 (stop) | ED-11.17 |
| Shift+F5 (Stop, or Disconnect for attach sessions, in debug mode) | SB-5.9 |
| Shift+F6 (Focus Previous Part) | SS-1.7 |
| Shift+F7 (previous) | ED-5.33 |
| Shift+F9 (settings.action.showContextMenu) | ED-8.43 |
| Shift+F9 (show setting context menu) | ED-8.5 |
| Shift+F9 (when editorTextFocus) | ED-11.10 |
| Shift+Insert on pasteSelection (Linux only) | PN-3.16 |
| Shift+L | ED-7.40 |
| Shift+PageDown and Shift+PageUp on the page commands (plain PageDown and PageUp on macOS) | PN-3.33 |
| Shift+PageUp and Shift+PageDown | ED-7.9 |
| Shift+Tab | ED-3.44, AC-6.22, DL-1.12 |
| Shift+Tab (insertPrevSuggestion) | AC-1.39 |
| Shift+Tab (only as the first key press while the tip shows, when the tip has an action) | PN-2.49 |
| Shift+Tab (when inlineSuggestionAlternativeActionVisible and inlineEditIsVisible) | AC-2.20 |
| Shift+Tab also inside the inline edits preview editor | AC-2.20 |
| Shift+Tab when editorTextInputFocus and a row is focused | AC-1.34 |
| Shift+Tab when inSnippetMode and hasPrevTabstop and inSnippetChoice and textInputFocus (an extra rule at higher weight, so the list's own Shift+Tab does not win) | AC-4.4 |
| Shift+Tab when inSnippetMode and hasPrevTabstop and textInputFocus | AC-4.1 |
| Shift+Up (list.expandSelectionUp) | SB-2.41 |
| Shift+Up and Shift+Down (list.expandSelection), mostly with notebookEditorFocused | ED-7.9 |
| Space | TB-1.13, SB-1.7, SB-1.23, ED-5.9, DL-1.12 |
| Space (filesExplorer.openFilePreserveFocus, when a non-folder is focused) | SB-2.3 |
| Space (list.stickyScrolltoggleExpand) | SB-1.24 |
| Space (list.toggleExpand) | SB-2.2 |
| Space (macOS and Linux) | TB-1.10 |
| Space (quickInput.toggleCheckbox, when the box shows a quick pick or tree and the filter field is not focused) | QA-6.2 |
| Space (toggles the focused checkbox) | DL-1.2 |
| Space (when a list has focus and a preview is active) | PN-7.23 |
| Space (when the Breakpoints view is focused and no input has focus) | SB-5.24 |
| Space (with a tab focused) | PN-1.10 |
| Space - hard-coded in the action bar, not rebindable | AB-1.12 |
| Space or Enter (on the inline scope link) | ED-8.45 |
| Space or Enter (when notificationFocus) | NT-2.7 |
| stock binds it only when the `isDevelopment` context is true, Cursor binds it with no `when` at weight 250 | WB-8.14 |
| Tab | ED-3.44, AC-6.21, AC-6.22, AC-7.16, DL-1.12 |
| Tab (editor.action.acceptCursorTabSuggestion, when cpp.shouldAcceptTab is true, weight 500) | AC-8.17 |
| Tab (insertBestCompletion, and insertNextSuggestion when hasOtherSuggestions) | AC-1.39 |
| Tab (moves to it) | DL-1.2 |
| Tab (next control) | SB-3.2 |
| Tab (next field) | DL-1.7 |
| Tab (reaches links) | DL-1.15 |
| Tab (when config.emmet.triggerExpansionOnTab is on, with editorTextFocus and tabDoesNotMoveFocus) | ED-2.32 |
| Tab (when inlineEditIsVisible, not editorTabMovesFocus, not suggestWidgetVisible, not editorHoverFocused, tabShouldJumpToInlineEdit) | AC-2.19 |
| Tab (when inlineSuggestionVisible, not editorTabMovesFocus, not suggestWidgetVisible, not editorHoverFocused, no selection, inlineSuggestionHasIndentationLessThanTabSize | AC-2.10 |
| Tab also inside the inline edits preview editor | AC-2.10 |
| Tab and Shift+Tab (editorTextFocus and tabDoesNotMoveFocus) | ED-2.17 |
| Tab and Shift+Tab (macOS only) | TB-1.8 |
| Tab from the find box moves to the replace box | ED-3.14 |
| Tab when cpp.shouldAcceptTab == true | AC-6.14, AC-6.15, AC-6.23, AC-6.27, AC-7.1, AC-7.2, AC-7.5, AC-7.6, AC-7.7, AC-7.8, AC-7.10, AC-7.19, AC-7.23 |
| Tab when editorTextFocus and tabDoesNotMoveFocus and not inSnippetMode and hasSnippetCompletions | AC-4.9 |
| Tab when inSnippetMode and hasNextTabstop and textInputFocus | AC-4.1 |
| Tab when simpleSuggestWidgetHasFocusedSuggestion, terminalFocus and terminalSuggestWidgetVisible | AC-5.8 |
| Tab when suggestWidgetVisible and editorTextInputFocus (a row must be focused) | AC-1.31 |
| terminal list has no default key | QA-5.11 |
| the accept keys themselves are in AC-1.34) | AC-1.44 |
| the command needs editorHasSignatureHelpProvider | AC-3.1 |
| the Control key on macOS too) | DL-2.14 |
| the Control key plus Shift+R on Mac) | ED-4.15 |
| the cycle commands apply when inQuickOpen and inEditorsPicker) | QA-4.2 |
| the first key is the Cursor chord prefix constant `zd` (`Oh` on macOS) that ED-4 decoded as Ctrl+M (Cmd+R), which I did not decode again | WB-6.6 |
| the follow modifier itself is the one in PN-3.3 | PN-3.46 |
| the item keys apply when breadcrumbs are visible and active | ED-5.9 |
| the Menu key also opens it | WB-7.2 |
| the other split keys are the stock ones listed in ED-1.13 | ED-10.90 |
| the others have no default key | ED-5.30 |
| the same chord on workbench.view.search sits behind a never-matching when clause, so it does nothing | SB-3.1 |
| the same chords step an open editor picker | ED-1.30 |
| the same with Alt, Ctrl and Cmd held | QA-6.16 |
| the source gives the same chord to Split Left, Right, Up and Down with no when clause) | ED-1.13 |
| the two in-group commands need a non-empty active group | QA-4.2 |
| then A (testing.runAll) | SB-6.6 |
| then C (run) | SB-6.30 |
| then Ctrl+C (debug) | SB-6.30 |
| then Ctrl+F (debug) | SB-6.29 |
| then Ctrl+L (debug last) | PN-6.26 |
| then Ctrl+M (Peek Output) | ED-13.4 |
| then Ctrl+O (testing.showMostRecentOutput) | PN-6.18 |
| then Ctrl+Shift+C (coverage), all when the editor has text focus (Cmd on macOS) | SB-6.30 |
| then Ctrl+Shift+F (coverage), all when the editor has text focus | SB-6.29 |
| then Ctrl+Shift+L (coverage last) | PN-6.26 |
| then Ctrl+X (cancel) | PN-6.26 |
| then Ctrl/Cmd+A (testing.debugAll) | SB-6.8 |
| then Ctrl/Cmd+E (debugFailTests) | SB-6.21 |
| then Ctrl/Cmd+L (debugLastRun) | SB-6.21 |
| then Ctrl/Cmd+R (when testing.canRefresh) | SB-6.14 |
| then Ctrl/Cmd+Shift+A (testing.coverageAll) | SB-6.9 |
| then Ctrl/Cmd+Shift+L (coverageLastRun) | SB-6.21 |
| then Ctrl/Cmd+X | SB-6.13 |
| then E (reRunFailTests) | SB-6.21 |
| then F (run) | SB-6.29 |
| then L (rerun last) | PN-6.26 |
| then L (reRunLastRun) | SB-6.21 |
| Trigger Symbol Highlight has no default key | ED-4.22, ED-5.33 |
| unbound on Linux) | ED-2.5 |
| Up | AC-3.6, NT-1.10 |
| Up (scm.viewPreviousCommit, when scmRepository and scmInputIsInFirstPosition and no suggest widget) | SB-4.13 |
| Up (settings.action.focusSearchFromSettings, when the first row has focus) | ED-8.43 |
| Up (show previous search) | ED-8.2 |
| Up (when a list has focus) | NT-3.10 |
| Up and Down (history, at the input's first or last line, suggest list hidden) | ED-7.37 |
| Up and Down arrows (rail focused) | AB-1.12 |
| Up and Down at the cell edge (focusPreviousEditor, focusNextEditor) | ED-7.9 |
| Up and Down move | ED-4.14 |
| Up Arrow | SB-1.7, SB-1.23, PN-6.4, QA-3.13 |
| Up Arrow and Alt+Up Arrow (history.showPrevious) | SB-3.5 |
| Up or Ctrl+Up (mac adds Ctrl+P) | AC-1.36 |
| Up when simpleSuggestWidgetHasNavigated is false and upArrowNavigatesHistory is true (history) | AC-5.7 |
| Up when simpleSuggestWidgetHasNavigated or upArrowNavigatesHistory is false (previous row) | AC-5.7 |
| Up, Down, Left, Right (hoverFocused) scroll | ED-4.2 |
| UpArrow | TB-1.10, TB-1.13 |
| UpArrow (history.showPrevious, history widget focused, not composing) | ED-3.21 |
| UpArrow (select previous line) | ED-5.6 |
| UpArrow, DownArrow, PageUp, PageDown (when the playground is focused) | ED-9.13 |
| V | ED-7.40 |
| web only: Ctrl+F12 and Ctrl+K Ctrl+F12 | ED-4.3 |
| when a single forwarded port is focused) | PN-7.15 |
| when cpp.shouldAcceptTab | AC-8.17 |
| when cppEnabled | AC-8.17 |
| when cppSnoozed | AC-8.17 |
| when cppSuggestion | AC-8.17 |
| when tabPredictionCodePreviewVisibleKey | AC-7.8 |
| when tabPredictionGoToFileVisibleKey | AC-7.7 |
| when TaskCommandsRegistered) | QA-4.11 |
| when the box shows a quick pick or tree) | QA-6.14 |
| when the Ports view is focused and the port can be closed) | PN-7.12 |
| WinCtrl is the Windows key on Windows and Ctrl on macOS | SS-6.13 |
| WinCtrl+Alt+J (join with next) | ED-7.6 |
| WinCtrl+Alt+Shift+J (join with previous), when notebookEditorFocused (WinCtrl is Ctrl on macOS, the Windows key elsewhere) | ED-7.6 |
| WinCtrl+Enter (execute | ED-7.13 |
| WinCtrl+Enter (render markdown, Ctrl+Alt+Enter on Windows) | ED-7.8 |
| WinCtrl+Shift+Alt+1 (debug.switchToprodBackend) | SS-6.13 |
| WinCtrl+Shift+Alt+2 (staging) | SS-6.13 |
| WinCtrl+Shift+Alt+3 (stagingLocalWebsite) | SS-6.13 |
| WinCtrl+Shift+Alt+4 (local) | SS-6.13 |
| WinCtrl+Shift+Alt+5 (fullLocal) | SS-6.13 |
| WinCtrl+Shift+Alt+6 (fullLocalincludingcpp) | SS-6.13 |
| WinCtrl+Shift+Alt+7 (localExceptEmbeddings) | SS-6.13 |
| WinCtrl+Shift+Alt+8 (devStagingEverything) | SS-6.13 |
| WinCtrl+Shift+Alt+9 (localStaging) | SS-6.13 |
| Windows also Ctrl+F4) | DL-2.1 |
| Windows also Shift+Delete (cut), Ctrl+Insert (copy), Shift+Insert (paste) | ED-2.28 |
| Windows Ctrl+K Ctrl+Shift+C) | SB-2.16 |
| Windows Shift+Alt+C) | SB-2.16 |
| Windows Shift+Alt+R) | SB-2.15 |
| with several cursors Escape first drops the extra cursors, see ED-2.1) | ED-2.41 |
| workbench.action.quickPickManyToggle has no default key | QA-6.2 |
| workbench.view.extension.test has no key | AB-1.9 |
| workbench.view.search is bound to Ctrl+Shift+F only behind a never-matching condition, the real Ctrl+Shift+F belongs to Find in Files | AB-1.9 |
| X | ED-7.40 |
| Y (to code) | ED-7.7 |
| Z | ED-7.40 |
| Zoom In - Ctrl+= (also Ctrl+Shift+= and Ctrl+Numpad+) | TB-3.16 |
| Zoom Out - Ctrl+- (also Ctrl+Shift+- and Ctrl+Numpad- | TB-3.16 |

## Excluded entries

Ids that the coverage check found and decided not to catalog, with the reason. Excluding something here does not say the product lacks it. It says it is outside the catalog's scope or has no user-visible UI of its own.

| Reason | Count |
|---|---|
| agent | 95 |
| internal | 53 |
| other | 51 |
| developer-only | 45 |
| not-ui | 39 |
| extension-system | 26 |
| web-only | 9 |
| remote | 7 |
| auto-excluded before review (third-party ids, chat and extension menus, agent settings) | 81 |

### agent

- `Accept and Run` (label): notebook.inlineChat.acceptChangesAndRun, button of the notebook inline chat; not in Cursor
- `Accept the changes and run the cell` (label): tooltip of the same notebook inline chat button
- `accessibility.signals.editsKept` (setting): Played only from chat editing code (chatEditing session and notebook integration) when edits are kept
- `accessibility.signals.editsUndone` (setting): Played only from chat editing code when edits are undone
- `Accessible View: Next Code Block` (label): precondition limits it to the chat, inline chat and quick chat providers; not in the Cursor bundle
- `Accessible View: Previous Code Block` (label): same chat-only precondition as Next Code Block
- `Add Tab` (label): tooltip of the Add Tab button, only the Sessions (Agents) window shows it
- `AgentsChangeInlineToolbar` (menu): inline buttons on a changed file row in the sessions app
- `AgentsChangesPrimaryActionSubMenu` (menu): primary action dropdown of the changes view in the sessions app
- `AgentsChangesToolbar` (menu): toolbar of the changes view in the sessions app (Agents window code)
- `AgentSessionItemToolbar` (menu): inline buttons on an agent session row
- `AgentSessionsContext` (menu): right-click menu of an agent session row
- `AgentSessionsCreateSubMenu` (menu): submenu for creating a new agent session type
- `AgentSessionSectionContext` (menu): right-click menu of an agent sessions section header
- `AgentSessionSectionToolbar` (menu): inline buttons on an agent sessions section header
- `AgentSessionsToolbar` (menu): toolbar of the agent sessions list
- `AgentSessionsViewerFilterSubMenu` (menu): filter submenu of the agent sessions list in the chat view
- `AgentsTitleBarControlMenu` (menu): Agents control of the command center, left out by TB-2
- `All variables in` (label): row in the chat context picker, full label "All variables in {0}" with the scope name
- `BrowserChatActionsMenu` (menu): stock Browser submenu of Add to Chat actions (element, console logs, screenshot); chat, handed off in ED-10; not in the Cursor bundle
- `browserView.blur` (command): agent tool helper that blurs the page view; no menu, key or button
- `ChatContext` (menu): right-click menu of chat messages
- `ChatEditingDeletedNotebookCell` (menu): menu of a deleted notebook cell shown by chat editing
- `cursor.browserOriginAllowlist.ensureNavigationAllowed` (command): checks an agent navigation against an administrator allowlist and throws; no screen lists or edits the allowlist
- `cursor.browserOriginAllowlist.ensurePageOriginAllowed` (command): same administrator allowlist check for agent browser tools; only seen as an error text in a tool result
- `cursor.browserView.closeTab` (command): agent tool for closing a tab; the person closes a tab through ED-10.30
- `cursor.browserView.configureDialogHandling` (command): lets the agent accept or dismiss a page's own alert box, API command with owner agent argument
- `cursor.browserView.executeJavaScript` (command): agent tool that runs script in the page
- `cursor.browserView.executeJavaScriptInIsolatedWorld` (command): agent tool that runs script in an isolated world
- `cursor.browserView.focusOrShowTab` (command): agent tool that brings a tab (maybe hidden) to the front; called from the browser automation extension
- `cursor.browserView.getConsoleLogs` (command): agent tool that reads page console logs
- `cursor.browserView.getNetworkRequests` (command): agent tool that reads page network requests
- `cursor.browserView.getTitle` (command): agent tool that reads the page title
- `cursor.browserView.getURL` (command): agent tool that reads the page address
- `cursor.browserView.isHeadless` (command): agent tool that asks whether a tab is a hidden agent tab
- `cursor.browserView.listTabs` (command): agent tool that lists open tabs
- `cursor.browserView.navigate` (command): agent tool that loads a url; the person types in the location bar (ED-10.31)
- `cursor.browserView.newHeadlessTab` (command): agent tool that opens a tab with no visible editor
- `cursor.browserView.newTab` (command): agent tool that opens a tab; the person opens one with ED-10.29; called by the browser automation extension
- `cursor.browserView.selectTab` (command): agent tool that activates a tab
- `cursor.browserView.sendCDPCommand` (command): agent tool that sends a raw DevTools protocol command
- `cursor.browserView.showHeadlessTab` (command): agent tool that turns a hidden agent tab into an editor tab
- `cursor.browserView.takeScreenshot` (command): agent tool with ref, element and fullPage options; the person path is ED-10.53
- `cursor.browserView.updateScreenshot` (command): feeds the screenshot store for agent tab thumbnails
- `cursor.mentionLinkToolbar` (command): small toolbar over a mention link in the chat input; hidden from the palette
- `cursorai.action.switchToDynamicModelSlug` (command): switches the chat model from a remote config value; hidden from the palette
- `cursorai.action.switchToModelSlug` (command): switches the chat model; called by the in-app promotion modal; hidden from the palette
- `deeplink.command.create` (command): Creates a Cursor custom command file from a cursor:// link; custom commands are agent prompts
- `deeplink.prompt.prefill` (command): Opens the Composer with a prefilled prompt from a cursor:// link
- `deeplink.prReview.open` (command): Opens a GitHub pull request link in Review Changes pull request mode, which ED-6 left out as background agent and Bugbot flow
- `deeplink.rule.create` (command): Creates a Cursor rule file from a cursor:// link; rules configure the agent
- `editor.tabs.addTab` (command): Add Tab button of the tab strip; only the Sessions (Agents) window fills its menu, hidden in a normal window, not in the Cursor bundle
- `EditorTabsBarAddTab` (menu): menu of the Add Tab button, filled only by the Sessions (Agents) window
- `EditorTitleLayout` (menu): layout buttons on the editor tab strip, filled only by the Agents window code in the sessions folder; the id is not in the Cursor bundle
- `Explain Changes` (label): workbench.scm.action.graph.summarizeHistoryItem sends a commit to chat; not in Cursor
- `Expression Value` (label): row in the chat context picker for debug values (debugChatIntegration.ts); the full label is "Expression Value..."
- `Fix Cell Error` (label): notebook.cell.chat.fixError sends the error to chat; not in Cursor
- `git.addAICoAuthor` (setting): adds an AI Co-authored-by trailer to commits that hold AI-generated code; absent from the Cursor git extension
- `Manage Dynamic Authentication Providers` (label): Accounts menu row for dynamic (MCP) auth providers, no hit in the Cursor bundle; AB-4 skipped it
- `MultiDiffEditorContent` (menu): floating toolbar over the multi-file diff editor; the only core row is the chat Explain Changes action; the id is not in the Cursor bundle
- `notebook.cell.chat.explainError` (command): sends the cell error to the chat panel with an explain request
- `notebook.cell.chat.fixError` (command): Fix Cell Error sends the cell error to chat; not in Cursor
- `notebook.cell.chat.start` (command): starts notebook cell chat
- `notebook.cell.chat.startAtTop` (command): Generate button above the first cell that starts chat to write code
- `notebook.cellOutput.addToChat` (command): Add Cell Output to Chat
- `notebook.chat.selectAndInsertKernelVariable` (command): inserts a kernel variable into a chat request
- `notebook.inlineChat.acceptChangesAndRun` (command): Accept and Run button of notebook inline chat
- `scm.editor.triggerSetup` (command): Resolve Conflicts with AI, shown only while Copilot chat setup is not complete; not in the Cursor bundle
- `search.searchView.keywordSuggestions` (setting): keyword suggestions of the stock AI search in the Search view; no hit in the Cursor bundle
- `search.searchView.semanticSearchBehavior` (setting): when the stock Search view asks for semantic results from an AI provider; no hit in the Cursor bundle
- `SessionHeaderContext` (menu): session header menu in the sessions app (Agents window code)
- `SessionItemContextMenu` (menu): right-click menu of a session row in the sessions app (Agents window code)
- `shellExec.sandboxAppArmorSuggestion` (command): Don't Show Again id of a terminal sandbox warning for the agent shell tool on Linux
- `shellExec.sandboxKernelTooOld` (command): Same sandbox warning for old Linux kernels
- `Type an expression to evaluate` (label): disabled hint row in the chat context picker, full label ends with "..."
- `vscode.notebook.variable` (command): notebook variable for chat requests
- `workbench.action.agentSessions.openFile` (command): Declared in V under src/vs/sessions (Agents window changes view); no hit in the Cursor main bundle; the work file path is wrong
- `workbench.action.agentToggleSidebarVisibility` (command): Declared in V under src/vs/sessions layoutActions (Agents window); no hit in the Cursor main bundle; the work file path is wrong
- `workbench.action.automationsEditor.save` (command): Save Automation, Ctrl+S inside the Cursor Automations editor, which the catalog left out as an agent surface (decision 7)
- `workbench.action.chat.upgradePlan` (command): Copilot plan upgrade command; the V line in the work file is a false hit, the real registration is in chat/browser/chatSetup; not in the Cursor bundle
- `workbench.action.devDelayNextSmartModeClassifier` (command): Delays the Auto-review (agent tool approval) check; also needs a local backend
- `workbench.action.devForceNextSmartModeClassifierBlock` (command): Forces the next Auto-review check to block; also needs a local backend
- `workbench.action.devSetAnyrunCluster` (command): Picks the cluster for background agents; enabled only when isDevelopment is true
- `workbench.action.exportNotepads` (command): Developer: Export Notepads to Files; shown only when legacy chat Notepads exist in storage
- `workbench.action.terminal.debugCheckExecutionServiceVersion` (command): Debug: Check Terminal Execution Service Version; shows version v2 or v3 of the service the agent uses to run terminal commands
- `workbench.actions.accounts.signIn` (command): Sign In for the Copilot default account (chat entitlement); not in the Cursor bundle
- `workbench.browser.experimentalUserTools.enabled` (setting): only adds experimental tools to the Browser's Add to Chat menu; Cursor does not have it
- `workbench.debug.action.addScopeToChat` (command): Add to Chat row in the Variables scope menu
- `workbench.debug.action.addVariableToChat` (command): Add to Chat row in the Variables right-click menu
- `workbench.debug.action.addWatchExpressionToChat` (command): Add to Chat row in the Watch right-click menu
- `workbench.hover.reducedDelay` (setting): only used by hovers in the chat status panel and the agent sessions list; Cursor does not have it
- `workbench.parts.unifiedsidebar` (command): part id of the Agents sidebar, not a command
- `workbench.scm.action.graph.addHistoryItemChangeToChat` (command): Graph row that adds a commit change to chat
- `workbench.scm.action.graph.addHistoryItemToChat` (command): Graph row that adds a commit to chat
- `workbench.scm.action.graph.summarizeHistoryItem` (command): Graph row that asks chat to summarize a commit

### developer-only

- `Attach to Current Code Renderer` (label): no palette entry, throws "Refusing to attach to renderer outside of development context" unless the window is an extension development or test window; not in Cursor
- `Clear Notebook Editor Type Cache` (label): notebook.clearNotebookEdtitorTypeCache, Developer category, same call as bucket C
- `Controls the logging level of the Markdown language server` (label): markdown.server.log (off, debug, trace), a debugging aid for the Markdown language server
- `cursor.configureDebugTimeout` (command): Configure: Debug Timeout Prevention, runs only when isDevelopment or isDevUser is true; picks cursor.debug.timeoutPrevention
- `cursor.dev.crashExtensionHost` (command): Developer: Crash Extension Host (with slow recovery); palette entry only when isDevelopment or isDevUser is set
- `cursor.dev.crashRemoteExtensionHost` (command): Developer: Crash Remote Extension Host (with slow recovery); same dev-only gate, also remote
- `cursor.useLocalhostUpdateServer` (command): Developer: Use Localhost Update Server, needs the update_use_localhost feature gate or it shows an error
- `debug.attachToCurrentCodeRenderer` (command): throws unless the window is in extension development; no menu or key; no copy in the Cursor bundle
- `developer.analytics.clearCollectedEvents` (command): Not in the palette (f1 false) and needs the isDevelopment key; test API for analytics events
- `developer.analytics.getCollectedEvents` (command): Same as above
- `developer.analytics.getFilteredEvents` (command): Same as above
- `developer.analytics.startEventCollection` (command): Same as above
- `developer.analytics.stopEventCollection` (command): Same as above
- `editor.experimental.asyncTokenizationLogging` (setting): Writes tokenization logs, described as for debugging only
- `editor.experimental.asyncTokenizationVerification` (setting): Checks async tokens against the old path, for debugging only
- `Inspect Notebook Layout` (label): notebook.inspectLayout, Developer category, same call as bucket C
- `markdown.server.log` (setting): log level of the Markdown language server, a debugging aid
- `markdown.trace.server` (setting): traces messages to the Markdown language server, a debugging aid
- `notebook.clearNotebookEdtitorTypeCache` (command): Developer category palette command that clears an internal editor type cache
- `notebook.developer.addCellOverlays` (command): Toggle Notebook Cell Overlays needs the isDevelopment key; ED-7 already lists the sibling notebook developer commands as not catalogued
- `notebook.developer.addViewZones` (command): Developer category and only enabled in a development build
- `notebook.inspectLayout` (command): Developer category palette command that logs cell layout to the console
- `notebook.toggleLayoutTroubleshoot` (command): Developer category palette command that toggles layout debugging
- `Reset notebook getting started` (label): workbench.notebook.layout.gettingStarted, Developer category, needs an experimental setting
- `Reset Onboarding Shown State` (label): Developer palette row for the stock spotlight tour engine, same call as bucket B for the command id; no copy in the Cursor bundle
- `Snapshot Tracked Disposables` (label): workbench.action.snapshotTrackedDisposables, development builds only
- `Start Tracking Disposables` (label): workbench.action.startTrackDisposables is registered only when product.commit is empty, same call as bucket A
- `Stop Tracking Disposables` (label): workbench.action.stopTrackDisposables, development builds only
- `Toggle Notebook Clipboard Troubleshooting` (label): workbench.action.toggleNotebookClipboardLog, Developer category, same call as the sibling notebook developer commands in bucket C
- `Toggle Notebook Layout Troubleshoot` (label): notebook.toggleLayoutTroubleshoot, Developer category, same call as bucket C
- `Toggle Notebook View Zones` (label): notebook.developer.addViewZones, Developer category, development builds only
- `Traces the communication between VS Code and the Markdown language server` (label): markdown.trace.server (off, messages, verbose), a debugging aid for the Markdown language server
- `workbench.action.clearStatsigCache` (command): Developer command enabled only when the isDevelopment key is true and registered only in builds without a commit
- `workbench.action.customize.clearPrimitiveAuthorshipCache` (command): Developer category, precondition isDevUser, clears a cache of the Customize editor
- `workbench.action.customize.clearPrimitiveSourceCache` (command): Developer category, precondition isDevUser, clears a cache of the Customize editor
- `workbench.action.devAutoLoginFakeForTesting` (command): Dev Auto Login with fake tokens; enabled only when isDevelopment is true
- `workbench.action.hangRendererThread` (command): Freezes the window for 30 s; enabled only when isDevelopment is true
- `workbench.action.injectCursorSocketFailure` (command): Enabled only when isDevelopment or isDevUser is true; kills the agent socket connections for testing
- `workbench.action.onboarding.resetShownState` (command): Developer: Reset Onboarding Shown State resets the stock spotlight tour engine, which has no tours registered in 1.128 (only tests register scenarios); missing in Cursor
- `workbench.action.runCursorWorktreeCleanupDev` (command): Run Cursor Worktree Cleanup (Dev); enabled only when isDevelopment is true
- `workbench.action.snapshotTrackedDisposables` (command): Registered only when product.commit is empty (development builds), in stock and in Cursor
- `workbench.action.startTrackDisposables` (command): Registered only when product.commit is empty (development builds), in stock and in Cursor
- `workbench.action.stopTrackDisposables` (command): Registered only when product.commit is empty (development builds); opens a leak report only for developers
- `workbench.action.toggleNotebookClipboardLog` (command): Developer category palette command that turns on clipboard logging
- `workbench.notebook.layout.gettingStarted` (command): Developer category row that resets a stored flag; needs an experimental setting; missing from Cursor

### extension-system

- `A very basic built-in webview for displaying web content` (label): description line of the simple-browser extension; its command is ED-10.25
- `debug.installAdditionalDebuggers` (command): Run menu row and the Install buttons only open the Extensions view search for debuggers
- `EditorTitleRun` (menu): submenu behind the Run or Debug split button on the editor title; in Cursor only extensions fill it, and no built-in extension does
- `extensionService.versionMismatchCrash` (command): Notification id for an extension host crash with exit code 55 (version mismatch) that offers a restart
- `exthost.action.toggleExtHostProfiler` (command): Developer: Start Network and IPC Profiler for the extension host
- `files.participants.timeout` (setting): only read by the working copy participant that exists for extension onWill file events
- `Git SCM Integration` (label): description line of the Git extension, shown only in the Extensions view
- `Highlighting and commands for inline merge conflicts` (label): description line of the Merge Conflict extension, shown in the Extensions view
- `http.electronFetch` (setting): only changes the fetch that local extensions get
- `http.experimental.networkInterfaceCheckInterval` (setting): only read by the extension host proxy resolver; Cursor does not have it
- `http.experimental.systemCertificatesV2` (setting): only read by the extension host proxy resolver
- `http.fetchAdditionalSupport` (setting): only extends the fetch used by extensions
- `http.proxySupport` (setting): description says it is the proxy support for extensions
- `http.webSocketAdditionalSupport` (setting): only extends the WebSocket used by extensions; Cursor does not have it
- `Markdown Language Features` (label): display name of the Markdown extension, shown in the Extensions view
- `Provides rich language support for Markdown` (label): description line of the Markdown extension, shown in the Extensions view
- `Provides syntax highlighting and language features for tabbed search results` (label): description line of the search-result extension; the Search Editor itself is ED-3.29
- `Reference Search results as separate, stable view in the sidebar` (label): description line of the references-view extension, shown in the Extensions view
- `Reference Search View` (label): display name of the references-view extension; the view itself is SB-7.20
- `Remove Visualizer` (label): Variables and Watch row that appears only when an extension supplies a visualizer; no built-in provider
- `Required for loading styles configured in the workspace` (label): untrusted workspace note of the Markdown extension, shown as the extension status message in the Extensions view and editor
- `SCMResourceContextShare` (menu): Share submenu in the changed file right-click menu; filled only by share providers from extensions and off by default; Cursor link rows sit in other Share menus (ED-1.28)
- `Visualize Variable` (label): Variables and Watch row that appears only when an extension supplies a visualizer; no built-in provider
- `workbench.action.extensionHostProfiler.stopAndSave` (command): Plain command of the extension host profile service; stock code for it is under contrib/extensions
- `workbench.action.previewColorTheme` (command): Called with an extension argument from the extension page to preview a theme; the live preview inside the Color Theme picker is a different path
- `workbench.action.showLanguageExtensions` (command): Search Marketplace Extensions for the extension row at the top of the language picker, opens the Extensions view

### internal

- `cursor.browserAutomation.internal.navigateWebview` (command): helper that navigates a view to a url; registered but nothing in the bundle calls it
- `cursor.publicLogCapture` (command): analytics event sender called by the feedback pane; hidden from the palette
- `deeplink.routeToWorkspaceName` (command): Helper that focuses the window of a named workspace and re-opens a deep link in it; no UI of its own
- `editor.action.goToLocation` (command): action id of a throwaway class created inside goToLocations and never registered
- `editor.lineDecorationsWidth` (setting): API-only editor option with no settings schema, set in code for embedded editors
- `editor.mouseStyle` (setting): API-only editor option (text, default, copy) with no settings schema
- `editor.overviewRulerLanes` (setting): API-only editor option with no settings schema; the overview ruler itself is ED-5.29
- `editor.placeholder` (setting): API-only editor option read by the placeholder text contribution of embedded editors; not a user setting
- `editor.renderValidationDecorations` (setting): API-only editor option (editable, on, off) with no settings schema
- `editor.revealHorizontalRightPadding` (setting): API-only editor option set only by the simple editor options helper
- `editor.selectOnLineNumbers` (setting): API-only editor option, default true; the visible click on a line number is already in ED-2.6
- `editor.tabIndex` (setting): API-only editor option with no settings schema
- `editor.useShadowDOM` (setting): API-only editor option with no settings schema
- `editor.wordWrapBreakAfterCharacters` (setting): API-only editor option with no settings schema, not set anywhere in the workbench
- `editor.wordWrapBreakBeforeCharacters` (setting): API-only editor option with no settings schema, not set anywhere in the workbench
- `Execute Notebook Variable Provider` (label): _executeNotebookVariableProvider, f1 false, API command that returns variables to extension code
- `Get Explorer Selection` (label): _testing.getExplorerSelection, no palette entry, API command for extension code; SB-6 already lists it as having no UI
- `Get Repositories` (label): git.api.getRepositories, category Git API, hidden from the palette and called by other extensions
- `Get Repository State` (label): git.api.getRepositoryState, category Git API, hidden from the palette and called by other extensions
- `Get Search Results` (label): search.action.getSearchResults, f1 false, returns the results as text to callers; SB-3 already lists it as having no UI
- `Get Selected Profiles` (label): testing.getSelectedProfiles, no palette entry, API command for extension code; SB-6 already lists it as having no UI
- `git.api.getRemoteSources` (command): API command hidden from the palette and called by extensions
- `git.api.getRepositories` (command): API command hidden from the palette and called by extensions
- `git.api.getRepositoryState` (command): API command hidden from the palette and called by extensions
- `git.fetchRef` (command): hidden from the palette, no menu row and no caller in the source; sibling of the Graph pull and push ref commands
- `Keypresses that should be handled by the focused element in the cell output` (label): title of notebook.cell.nullAction, a no-op that binds the arrow keys so the list does not steal them from an input inside an output
- `Move Views` (label): title of the extension API command vscode.moveViews, which takes viewIds and destinationId and has no palette entry, menu or key; the picker version is SB-1.14; the same command id was already excluded as internal in pass 4
- `notebook.cell.nullAction` (command): no-op that only reserves Up and Down for the output page; not in the palette
- `repositoryTracker.getStoredPath` (command): Helper command with no UI that returns a stored repository path
- `repositoryTracker.isOrInCurrentWorkspace` (command): Helper command with no UI
- `repositoryTracker.removeStoredPath` (command): Helper command with no UI
- `Reset Notebook Webview` (label): workbench.notebook.layout.webview.reset has f1 false and nothing calls it
- `solidjs.getTotalObservers` (command): Registered only in unbuilt or smoke test runs, returns a number for metrics
- `testing.getSelectedProfiles` (command): API command for extension code, no palette entry
- `testing.startContinuousRunFromExtension` (command): API command for extension code, no palette entry
- `testing.stopContinuousRunFromExtension` (command): API command for extension code, no palette entry
- `vscode.menubar.separator` (command): Marker id for separators in the data sent to the native macOS menu
- `vscode.moveViews` (command): Extension API command with object arguments, has a title but no palette entry, menu or key; the picker version is SB-1.14
- `vscode.newWindow` (command): Extension API command that forwards to the New Window code with reuseWindow and remoteAuthority options, no UI of its own
- `vscode.removeFromRecentlyOpened` (command): Extension API command that takes a path, no UI of its own; the clear list entry is TB-1.16
- `vscode.testing.getControllersWithTests` (command): API command for extension code, no palette entry or menu
- `vscode.testing.getTestsInFile` (command): API command for extension code, no palette entry or menu
- `workbench.action.devBatchSetApplicationStorageForTesting` (command): Test hook, no palette entry, throws unless the smoke test driver flag is on
- `workbench.action.devGetApplicationStorageForTesting` (command): Test hook, no palette entry, smoke test driver only
- `workbench.action.devGetAutorunSettingForTesting` (command): Test hook, no palette entry, smoke test driver only
- `workbench.action.devGetConfiguredHookStepsForTesting` (command): Plain registered command used by tests, no title or menu
- `workbench.action.devGetLoadedPluginsSummaryForTesting` (command): Test hook, no palette entry, smoke test driver only
- `workbench.action.devGetModelConfigForSurfaceForTesting` (command): Test hook, no palette entry, smoke test driver only
- `workbench.action.devRefreshTeamAdminSettings` (command): Test hook, no palette entry, smoke test driver only
- `workbench.action.devSetApplicationStorageForTesting` (command): Test hook, no palette entry, smoke test driver only
- `workbench.action.devWaitForSlashMenuSourcesForTesting` (command): Test hook, no palette entry, smoke test driver only
- `workbench.action.terminal.addTooltip` (command): No palette entry; helper for a terminal hint card whose two switches are false in the bundle, so nothing shows
- `workbench.notebook.layout.webview.reset` (command): hidden from the palette and called by no menu or code in the source

### not-ui

- `cursor.spaceXMode` (command): not a command; a source id for default setting overrides (dev container and port forwarding defaults) when a hidden mode is on
- `cursor.tsgoDisableAutoImportsInternalExperiment` (command): not a command; a source id for default setting overrides in an internal tsgo auto import experiment
- `cursorAuth.triggerTokenRefresh` (command): palette entry Trigger Token Refresh with no visible result; auth code also calls it after a refused token
- `editor.allowOverflow` (setting): Embedder API option with no settings schema
- `editor.ariaLabel` (setting): Embedder API option, default label "Editor content", no settings schema
- `editor.ariaRequired` (setting): Embedder API option with no settings schema
- `editor.auto` (setting): Not a setting; an extraction artifact from nls keys such as editor.autoClosingBrackets.* that are real settings owned by ED-2
- `editor.automaticLayout` (setting): Embedder API option with no settings schema
- `editor.disableLayerHinting` (setting): Embedder API option with no settings schema
- `editor.disableMonospaceOptimizations` (setting): Embedder API option with no settings schema
- `editor.domReadOnly` (setting): Embedder API option with no settings schema
- `editor.experimental.treeSitterTelemetry` (setting): Collects telemetry while tree-sitter parses; no visible effect, and stock 1.128 has no reader for it
- `editor.extraEditorClassName` (setting): Embedder API option with no settings schema
- `editor.find` (setting): group name of the editor.find.* settings, not a setting on its own; every child is in ED-3
- `editor.fixedOverflowWidgets` (setting): Embedder API option with no settings schema
- `editor.inDiffEditor` (setting): Embedder API option with no settings schema
- `editor.inlineSuggest.experimental.emptyResponseInformation` (setting): Controls sending request information to the provider; no visible effect and not in the Cursor bundle
- `editor.inlineSuggest.triggerCommandOnProviderChange` (setting): Picks an internal refresh path when a provider changes (marked TODO to remove in source); not in the Cursor bundle
- `editor.none` (setting): false hit, only a test string in the configuration resolver tests
- `editor.stickyScroll` (setting): group name of the editor.stickyScroll.* settings, not a setting on its own; the children are in ED-5.4 and ED-5.5
- `notebook.diff.cell.switchOutputRenderingStyle` (command): only in commented-out code; the live sibling switchOutputRenderingStyleToText is in ED-7.35
- `Run Clean Task` (label): registration is commented out in task.contribution.ts, so the command does not exist; not in Cursor
- `Run Rebuild Task` (label): registration is commented out in task.contribution.ts, so the command does not exist; not in Cursor
- `search.experimental.useIgnoreFilesInFindFiles` (setting): changes only how the legacy findFiles extension API treats ignore files, experimental, nothing visible
- `search.maintainFileSearchCache` (setting): deprecated, the message says the setting is no longer needed, no visible effect
- `Switch Outputs Rendering` (label): only inside commented-out code in notebookDiffActions.ts and not in Cursor; the live command is Switch Output Rendering (ED-7.35)
- `TimelineTitleContext` (menu): defined in actions.ts but never used by any registration or consumer in 1.128.0; Cursor only defines it too
- `Troubleshooting Guide` (label): button of the vscode:showCredentialsError notice; the handler exists in window.ts but nothing in V, N or the Cursor out folder sends that message
- `User View Container` (label): placeholder title of a generated container, code comment says it should not be shown anywhere
- `workbench.action.tasks.clean` (command): only in commented-out code and never registered; no copy in the Cursor bundle
- `workbench.action.tasks.rebuild` (command): only in commented-out code and never registered; no copy in the Cursor bundle
- `workbench.files.openFile` (command): telemetry event name used when a file row is opened in the Explorer; behavior is SB-2.3
- `workbench.parts.activitybar` (command): Not a command; this is the part id string from the Parts enum used as an argument; AB-1 to AB-6 describe the part
- `workbench.parts.auxiliarybar` (command): Not a command; part id string; AX-1 and AX-2 describe the part
- `workbench.parts.editor` (command): Not a command; part id string; ED-1 describes the editor area
- `workbench.parts.panel` (command): Not a command; part id string; PN-1 describes the panel
- `workbench.parts.statusbar` (command): Not a command; part id string; SS-1 describes the status bar
- `workbench.parts.titlebar` (command): Not a command; part id string; TB-1 to TB-5 describe the title bar
- `workbench.reduceTransparency` (setting): in 1.128.0 it only sets a CSS class that no stylesheet uses; Cursor lacks it and has its own cursor.general.reduceTransparency

### other

- `Always insert Markdown links` (label): value of the markdown.editor.drop.enabled, filePaste.enabled and pasteUrlAsFormattedLink.enabled settings; Markdown source editing, out of scope
- `Always update links automatically` (label): value of markdown.updateLinksOnFileMove.enabled; update links on move was ruled out of scope
- `Controls if files created by drop or paste should overwrite existing files` (label): markdown.copyFiles.overwriteBehavior; paste and drop of files was ruled out of scope
- `Controls whether error reporting is enabled in Markdown files` (label): markdown.validate.enabled; link validation was ruled out of scope, though SS-3.53 still names this setting for its status row
- `Controls whether link occurrences in the current document are highlighted` (label): markdown.occurrencesHighlight.enabled; highlights links in the Markdown source editor, not the preview
- `dev.azure.com` (command): Not a command: the Azure DevOps host name in remote URL parsing
- `Do not copy external files into the workspace` (label): value of markdown.editor.drop.copyIntoWorkspace and filePaste.copyIntoWorkspace; Markdown source editing, out of scope
- `eamodio.gitlens` (command): Not a command: a GitLens extension id in the list of AI autocomplete extensions that Cursor skips or disables when importing from VS Code
- `editor.action.gotoLine` (command): registered only by the standalone Monaco editor (standaloneGotoLineQuickAccess.ts); the workbench Go to Line is workbench.action.gotoLine in QA-3.1
- `editor.action.gotoOffset` (command): standalone Monaco editor id only; the workbench Go to Offset is workbench.action.gotoOffset in QA-3.3; the script source line points at the wrong file
- `editor.action.inspectTokens` (command): standalone Monaco editor only and not in the Cursor bundle; the workbench token inspector is editor.action.inspectTMScopes in ED-5.39
- `editor.action.quickOutline` (command): registered only by the standalone Monaco editor; the workbench Go to Symbol is workbench.action.gotoSymbol in QA-3.10
- `editor.action.toggleHighContrast` (command): standalone Monaco editor only; not in the workbench or in the Cursor bundle
- `Enable updating links when a directory is moved or renamed in the workspace` (label): markdown.updateLinksOnFileMove.enableForDirectories; update links on move was ruled out of scope
- `If a file with the same name already exists, overwrite it` (label): value overwrite of markdown.copyFiles.overwriteBehavior; paste and drop of files was ruled out of scope
- `Insert Image from Workspace` (label): title of markdown.editor.insertImageFromWorkspace, Markdown source editing
- `Insert Link to File in Workspace` (label): title of markdown.editor.insertLinkFromWorkspace, Markdown source editing
- `markdown.copyFiles.destination` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.copyFiles.overwriteBehavior` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.editor.drop.copyIntoWorkspace` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.editor.drop.enabled` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.editor.filePaste.audioSnippet` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.editor.filePaste.copyIntoWorkspace` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.editor.filePaste.enabled` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.editor.filePaste.videoSnippet` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.editor.insertImageFromWorkspace` (command): Markdown source editing, ruled out of scope
- `markdown.editor.insertLinkFromWorkspace` (command): Markdown source editing, ruled out of scope
- `markdown.editor.pasteUrlAsFormattedLink.enabled` (setting): Markdown source editing, ruled out of scope
- `markdown.editor.updateLinksOnPaste.enabled` (setting): Markdown source editing, ruled out of scope
- `markdown.occurrencesHighlight.enabled` (setting): highlights link occurrences in the Markdown source editor, same call as its label row in bucket I
- `markdown.preferredMdPathExtensionStyle` (setting): link path completion style in Markdown source, ruled out of scope
- `markdown.updateLinksOnFileMove.enabled` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.updateLinksOnFileMove.enableForDirectories` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.updateLinksOnFileMove.include` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.validate.duplicateLinkDefinitions.enabled` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.validate.fileLinks.enabled` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.validate.fileLinks.markdownFragmentLinks` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.validate.fragmentLinks.enabled` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.validate.ignoredLinks` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.validate.referenceLinks.enabled` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `markdown.validate.unusedLinkDefinitions.enabled` (setting): Markdown source editing is outside the scope (Markdown preview only)
- `Never create Markdown links` (label): value of the markdown.editor.drop.enabled, filePaste.enabled and pasteUrlAsFormattedLink.enabled settings; Markdown source editing, out of scope
- `Never try to update link and don't prompt` (label): value of markdown.updateLinksOnFileMove.enabled; update links on move was ruled out of scope
- `Prompt on each file move` (label): value of markdown.updateLinksOnFileMove.enabled; update links on move was ruled out of scope
- `supermaven.supermaven` (command): Not a command: a Supermaven extension id in the AI autocomplete extension list used by the VS Code import
- `The glob pattern to match file paths against. Set to true to enable the pattern` (label): markdown.updateLinksOnFileMove.include; update links on move was ruled out of scope
- `Try to copy external image and video files into the workspace` (label): value of markdown.editor.drop.copyIntoWorkspace and filePaste.copyIntoWorkspace; Markdown source editing, out of scope
- `Validate duplicated definitions in the current file` (label): markdown.validate.duplicateLinkDefinitions.enabled; link validation was ruled out of scope
- `Validate link definitions that are unused in the current file` (label): markdown.validate.unusedLinkDefinitions.enabled; link validation was ruled out of scope
- `vscjava.migrate-java-to-azure` (command): Not a command: an extension id in the AI autocomplete extension list used by the VS Code import
- `workbench.userDataSync.actions.downloadSyncActivity` (command): Settings Sync is out of scope (catalog decision 9); Developer category and missing in Cursor

### remote

- `airlock.openLog` (command): Action id on the failure notice of Cursor's airlock connection (Dev Containers based remote connect)
- `airlock.reloadWindow` (command): Second action id on the same airlock failure notice
- `http.useLocalProxyConfiguration` (setting): only applies as a remote setting during remote development
- `remote.reconnectGaveUp` (command): Notification id "Unable to reconnect to the remote workspace" with a Reconnect Now button
- `remote.tunnelsView.autoForwardNeverShow` (command): Don't Show Again id of the forwarded port notification
- `workbench.action.simulatePermanentRemoteDisconnect` (command): Connection: Simulate Permanent Disconnect; enabled only for development or dev users
- `workbench.action.simulateRemoteConnectionDrop` (command): Connection: Simulate Connection Drop; enabled only for development or dev users

### web-only

- `issueReporter.experimental.webReporter` (setting): registered only by the web build's issue contribution
- `MenubarHomeMenu` (menu): home indicator menu in the web build only
- `Reset User Data` (label): workbench.action.resetUserData is registered by the web workbench only, to clear browser storage
- `workbench.action.download` (command): Precondition IsWebContext and shown only on the web status bar remote menu; Cursor has the id but the desktop app never shows it
- `workbench.action.resetUserData` (command): Registered in web.main.ts for the browser build only; no hit in the Cursor bundle
- `workbench.experimental.requestHidDevice` (command): Extension API command in browser window.ts for web only; no hit in the Cursor bundle
- `workbench.experimental.requestSerialPort` (command): Extension API command in browser window.ts for web only; no hit in the Cursor bundle
- `workbench.experimental.requestUsbDevice` (command): Extension API command in browser window.ts for web only; no hit in the Cursor bundle
- `workbench.extensions.action.openInstalledWebExtensionsResource` (command): Registered only when isWeb, precondition IsWebContext

