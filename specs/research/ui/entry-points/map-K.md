Bucket: K. Date: 2026-10-25. Pass: 4.
MAP | Increase Multi Cursor Limit | ED-2.1 | label | button on the warning shown when the cursor count hits editor.multiCursorLimit, opens Settings on that setting
MAP | Toggle Show Moved Code Blocks | ED-6.5 | label | title of diffEditor.toggleShowMovedCodeBlocks
MAP | Toggle Use Inline View When Space Is Limited | ED-6.2 | label | title of diffEditor.toggleUseInlineViewWhenSpaceIsLimited
MAP | Exit Compare Move | ED-6.5 | label | title of diffEditor.exitCompareMove, Escape while comparing a moved block
MAP | Go to Next Difference | ED-6.6 | label | title of editor.action.accessibleDiffViewer.next, F7
MAP | Go to Previous Difference | ED-6.6 | label | title of editor.action.accessibleDiffViewer.prev, Shift+F7
MAP | Set Selection Anchor | ED-2.8 | label | title of editor.action.setSelectionAnchor
MAP | Cancel Selection Anchor | ED-2.8 | label | title of editor.action.cancelSelectionAnchor
MAP | Go to Next Problem in Files (Error, Warning, Info) | ED-4.27 | label | title of editor.action.marker.nextInFiles, F8
MAP | Go to Previous Problem in Files (Error, Warning, Info) | ED-4.27 | label | title of editor.action.marker.prevInFiles, Shift+F8
MAP | Go to Any Symbol | ED-4.33 | label | title of the generic action behind editor.action.goToLocations, an API command with no palette row; it gives the peek or goto-and-step result
MAP | Show Previous Inline Suggestion | AC-2.9 | label | title of editor.action.inlineSuggest.showPrevious, Alt+[
MAP | Expand Line Selection | ED-2.6 | label | title of expandLineSelection
MAP | Add Selection to Previous Find Match | ED-2.3 | label | title of editor.action.addSelectionToPreviousFindMatch
MAP | Select the next editor sticky scroll line | ED-5.6 | label | title of editor.action.selectNextStickyScrollLine, Down while the pinned lines have focus
MAP | Select the previous sticky scroll line | ED-5.6 | label | title of editor.action.selectPreviousStickyScrollLine, Up
MAP | Go to the focused sticky scroll line | ED-5.6 | label | title of editor.action.goToFocusedStickyScrollLine, Enter
MAP | Disable highlighting of characters in comments | ED-5.37 | label | hover action of the confusing character box, sets editor.unicodeHighlight.includeComments
MAP | Disable highlighting of characters in strings | ED-5.37 | label | hover action, sets editor.unicodeHighlight.includeStrings
MAP | Disable highlighting of invisible characters | ED-5.37 | label | hover action, sets editor.unicodeHighlight.invisibleCharacters
MAP | Disable highlighting of non basic ASCII characters | ED-5.37 | label | hover action, sets editor.unicodeHighlight.nonBasicASCII
MAP | Show Exclude Options | ED-5.37 | label | title of editor.action.unicodeHighlight.showExcludeOptions, the picker the hover link opens
MAP | Allow unicode characters that are more common in the language "" | ED-5.37 | label | row of the exclude picker, adds the locale to editor.unicodeHighlight.allowedLocales
MAP | Hide action widget | ED-4.14 | label | title of hideCodeActionWidget, Escape
MAP | Clear action widget filter | ED-4.14 | label | title of clearFilterCodeActionWidget, stock only
MAP | Select previous action | ED-4.14 | label | title of selectPrevCodeAction, Up
MAP | Select next action | ED-4.14 | label | title of selectNextCodeAction, Down
MAP | Collapse section | ED-4.14 | label | title of collapseSectionCodeAction, Left on a header, stock only
MAP | Toggle section | ED-4.14 | label | title of toggleSectionCodeAction, Space on a header, stock only
MAP | Accept selected action | ED-4.14 | label | title of acceptSelectedCodeAction, Enter
MAP | Preview selected action | ED-4.14 | label | title of previewSelectedCodeAction, Ctrl+Enter
MAP | Hide '' | WB-7.1 | label | right-click row Hide '{action}' on a menu driven toolbar
EXCL | Start Tracking Disposables | label | developer-only | workbench.action.startTrackDisposables is registered only when product.commit is empty, same call as bucket A
EXCL | Snapshot Tracked Disposables | label | developer-only | workbench.action.snapshotTrackedDisposables, development builds only
EXCL | Stop Tracking Disposables | label | developer-only | workbench.action.stopTrackDisposables, development builds only
MAP | Signup for the VS Code Newsletter | TB-1.30 | label | Help row workbench.action.openNewsletterSignupUrl, stock only, listed in that feature's commands; no hit in the Cursor bundle
MAP | Toggle Primary Side Bar Visibility | SB-1.1 | label | title of workbench.action.toggleSidebarVisibility
MAP | Panel / | SB-1.14 | label | separator heading in the Move View picker, shows the views of each panel container
MAP | Secondary Side Bar / | SB-1.14 | label | separator heading in the Move View picker, shows the views of each secondary side bar container
NEW | Folder Opened in a Window | NEW-K-1 | label | tooltip of a window icon on an Open Recent row, stock only; the Open Recent picker itself has no feature
NEW | Workspace Opened in a Window | NEW-K-1 | label | same icon for a workspace row, stock only
NEW | Folder Opened in Active Window | NEW-K-1 | label | same icon when the folder is open in the window you are in, stock only
NEW | Workspace Opened in Active Window | NEW-K-1 | label | same for a workspace, stock only
NEW | Quick Open Recent | NEW-K-1 | label | title of workbench.action.quickOpenRecent, the quick-navigate form of the Open Recent picker; DL-2.14 only covers its unsaved-files button
MAP | Duplicate Workspace | WB-9.1 | label | File menu row (group 3_workspace) for workbench.action.duplicateWorkspaceInNewWindow; WB-9.1 lists only the palette entry
EXCL | Reset User Data | label | web-only | workbench.action.resetUserData is registered by the web workbench only, to clear browser storage
MAP | Show Next in Accessible View | AC-2.38 | label | title of editor.action.accessibleViewNext, Alt+]; also steps notifications (NT-3.12) and comments, only where the open view supports navigation
EXCL | Accessible View: Next Code Block | label | agent | precondition limits it to the chat, inline chat and quick chat providers; not in the Cursor bundle
EXCL | Accessible View: Previous Code Block | label | agent | same chat-only precondition as Next Code Block
MAP | Show Previous in Accessible View | AC-2.38 | label | title of editor.action.accessibleViewPrevious, Alt+[
MAP | Go To Symbol in Accessible View | WB-4.4 | label | title of editor.action.accessibleViewGoToSymbol
MAP | Accessibility Help Configure Unassigned Keybindings | WB-4.1 | label | title of editor.action.accessibilityHelpConfigureKeybindings, Alt+K
MAP | Accessibility Help Configure Assigned Keybindings | WB-4.1 | label | title of editor.action.accessibilityHelpConfigureAssignedKeybindings, Alt+A
MAP | Accessibility Help Open Help Link | WB-4.2 | label | title of editor.action.accessibilityHelpOpenHelpLink, Alt+H
MAP | Reset Emulation | ED-10.61 | label | title of workbench.action.browser.resetEmulation, listed in that feature; stock Browser only
MAP | Close Find Widget | ED-10.77 | label | title of workbench.action.browser.hideFind, Escape; stock Browser only
MAP | Remove from Favorites | ED-10.37 | label | star button and delete icon in the stock Browser's favorites; Cursor uses its bookmarks bar
MAP | Open Integrated Browser | TB-5.9 | label | title of workbench.action.browser.open, listed in that feature as the title bar button's target; also a palette row, stock only
MAP | Toggle Multi-Cursor Modifier | ED-2.2 | label | title of workbench.action.toggleMultiCursorModifier
MAP | Set Render Whitespace to None | ED-2.36 | label | title of editor.action.renderWhitespace.none; ED-5.35 lists the same ids
MAP | Set Render Whitespace to Boundary | ED-2.36 | label | title of editor.action.renderWhitespace.boundary
MAP | Set Render Whitespace to Selection | ED-2.36 | label | title of editor.action.renderWhitespace.selection
MAP | Set Render Whitespace to Trailing | ED-2.36 | label | title of editor.action.renderWhitespace.trailing
MAP | Set Render Whitespace to All | ED-2.36 | label | title of editor.action.renderWhitespace.all
NEW | Toggle Auto Save | NEW-K-2 | label | workbench.action.toggleAutoSave and its File menu row; ED-1 notes say no group owns auto save, and no feature lists the command
MAP | ( • ) | SB-7.14 | label | tab title of an old copy opened from Local History, shown as name (source • date); the compare titles use the same pattern (SB-7.13)
MAP | Focus problems view | PN-4.5 | label | title of problems.action.focusProblemsFromFilter, Ctrl+Down in the filter box
MAP | Focus problems filter | PN-4.5 | label | title of problems.action.focusFilter, Ctrl+F in the Problems view
EXCL | Reset Onboarding Shown State | label | developer-only | Developer palette row for the stock spotlight tour engine, same call as bucket B for the command id; no copy in the Cursor bundle
MAP | Switch Output | PN-5.2 | label | title of the channel drop-down submenu and workbench.output.action.switchBetweenOutputs
MAP | Toggle Auto Scrolling | PN-5.8 | label | title of workbench.output.action.toggleAutoScroll, the lock button in the Output title
MAP | Clear filters text | PN-5.5 | label | title of the Escape command that empties the Output filter box
MAP | Keyboard Layouts () | SS-3.38 | label | heading in the Change Keyboard Layout picker, web build only; feature lists the picker
MAP | Open Workspace Settings (JSON) | ED-8.8 | label | title of workbench.action.openWorkspaceSettingsFile
MAP | Open Folder Settings (JSON) | ED-8.8 | label | title of workbench.action.openFolderSettingsFile
MAP | Focus Settings Search | ED-8.2 | label | title of settings.action.search, Ctrl+F in the Settings editor
MAP | Clear Settings Search Results | ED-8.2 | label | title of settings.action.clearSearchResults, Escape in the search box
MAP | Focus settings file | ED-8.43 | label | title of settings.action.focusSettingsFile, Down in the search box
MAP | Show Previous Search in Settings | ED-8.2 | label | title of settings.action.showPreviousSearch, Up in the search box
MAP | Focus First Setting from Search | ED-8.43 | label | title of settings.action.focusSettingsFromSearchOnEnter, Enter in the search box
MAP | Focus Settings Search from Settings | ED-8.43 | label | title of settings.action.focusSearchFromSettings, Up on the first row
MAP | Focus Settings Table of Contents | ED-8.4 | label | title of settings.action.focusTOC; ED-8.43 lists the Left key for it
MAP | Focus Setting Control | ED-8.43 | label | title of settings.action.focusSettingControl, Enter on a row
MAP | Execute REPL input | ED-7.39 | label | title of repl.execute, Ctrl+Enter or Shift+Enter in the REPL editor
MAP | Create New Terminal (In Active Workspace) | PN-2.2 | label | title of workbench.action.terminal.newInActiveWorkspace, palette only
MAP | Focus Previous Terminal in Terminal Group | PN-2.22 | label | title of workbench.action.terminal.focusPreviousPane
MAP | Focus Next Terminal in Terminal Group | PN-2.22 | label | title of workbench.action.terminal.focusNextPane
MAP | Resize Terminal Left | PN-2.23 | label | title of workbench.action.terminal.resizePaneLeft
MAP | Resize Terminal Right | PN-2.23 | label | title of workbench.action.terminal.resizePaneRight
MAP | Resize Terminal Up | PN-2.23 | label | title of workbench.action.terminal.resizePaneUp
MAP | Resize Terminal Down | PN-2.23 | label | title of workbench.action.terminal.resizePaneDown
MAP | Focus Terminal Tabs View | PN-2.18 | label | title of workbench.action.terminal.focusTabs
MAP | Focus Next Terminal Group | PN-2.20 | label | title of workbench.action.terminal.focusNext
MAP | Focus Previous Terminal Group | PN-2.20 | label | title of workbench.action.terminal.focusPrevious
MAP | Run Selected Text In Active Terminal | PN-2.10 | label | title of workbench.action.terminal.runSelectedText
MAP | Run Active File In Active Terminal | PN-2.10 | label | title of workbench.action.terminal.runActiveFile
MAP | Scroll Down (Line) | PN-3.33 | label | title of workbench.action.terminal.scrollDown
MAP | Scroll Down (Page) | PN-3.33 | label | title of workbench.action.terminal.scrollDownPage
MAP | Scroll Up (Line) | PN-3.33 | label | title of workbench.action.terminal.scrollUp
MAP | Scroll Up (Page) | PN-3.33 | label | title of workbench.action.terminal.scrollUpPage
MAP | Clear Selection | PN-3.19 | label | title of workbench.action.terminal.clearSelection, Esc
MAP | Select to Previous Command | PN-3.9 | label | title of workbench.action.terminal.selectToPreviousCommand
MAP | Select to Next Command | PN-3.9 | label | title of workbench.action.terminal.selectToNextCommand
MAP | Split Terminal (In Active Workspace) | PN-2.21 | label | title of workbench.action.terminal.splitInActiveWorkspace, palette only
MAP | Kill the Active Terminal Instance | PN-2.28 | label | title of workbench.action.terminal.kill
MAP | Kill the Active Terminal in Editor Area | PN-2.27 | label | title of workbench.action.terminal.killEditor, listed in that feature
MAP | Focus Accessible Terminal View | PN-3.22 | label | title of workbench.action.terminal.focusAccessibleBuffer, Alt+F2
MAP | Accessible Buffer Go to Next Command | PN-3.22 | label | title of workbench.action.terminal.accessibleBufferGoToNextCommand, Alt+Down
MAP | Accessible Buffer Go to Previous Command | PN-3.22 | label | title of workbench.action.terminal.accessibleBufferGoToPreviousCommand, Alt+Up
MAP | Scroll to Accessible View Bottom | PN-3.22 | label | title of workbench.action.terminal.scrollToBottomAccessibleView
MAP | Scroll to Accessible View Top | PN-3.22 | label | title of workbench.action.terminal.scrollToTopAccessibleView
MAP | Selection Mode: Partial (Tab) | AC-5.10 | label | title of workbench.action.terminal.changeSelectionModePartial
MAP | Selection Mode: Always (Tab or Enter) | AC-5.10 | label | title of workbench.action.terminal.changeSelectionModeAlways
MAP | Select the Previous Suggestion | AC-5.7 | label | title of workbench.action.terminal.selectPrevSuggestion
MAP | Select the Previous Page Suggestion | AC-5.7 | label | title of workbench.action.terminal.selectPrevPageSuggestion
MAP | Select the Next Suggestion | AC-5.7 | label | title of workbench.action.terminal.selectNextSuggestion
MAP | Suggest Toggle Suggestion Focus | AC-5.13 | label | title of workbench.action.terminal.suggestToggleDetailsFocus
MAP | Suggest Toggle Details | AC-5.13 | label | title of workbench.action.terminal.suggestToggleDetails
MAP | Select the Next Page Suggestion | AC-5.7 | label | title of workbench.action.terminal.selectNextPageSuggestion
MAP | Accept Selected Suggestion (Enter) | AC-5.9 | label | title of workbench.action.terminal.acceptSelectedSuggestionEnter
MAP | Hide Suggest Widget and Navigate History | AC-5.7 | label | title of workbench.action.terminal.hideSuggestWidgetAndNavigateHistory, listed in that feature
MAP | product icon themes | ED-8.17 | label | separator heading in the product icon theme picker
NEW | Apply Update from File | NEW-K-3 | label | Windows-only Developer palette row _update.applyupdate, not gated to development builds; Cursor words it Apply Update...
MAP | Export Profile () | ED-8.14 | label | Share menu row Export Profile ({name})... for workbench.profiles.actions.exportProfile
MAP | Save Current Profile As | ED-8.14 | label | title of workbench.profiles.actions.createFromCurrentProfile
MAP | Delete Profile | ED-8.14 | label | title of workbench.profiles.actions.deleteProfile
MAP | Cleanup Profiles | ED-8.14 | label | title of workbench.profiles.actions.cleanupProfiles, listed in that feature
MAP | Reset Workspace Profiles Associations | ED-8.14 | label | title of workbench.profiles.actions.resetWorkspaces, listed in that feature
MAP | Mark Step Complete | ED-9.9 | label | title of welcome.markStepComplete
MAP | Mark Step Incomplete | ED-9.9 | label | title of welcome.markStepIncomplete
MAP | Create New File () | ED-9.25 | label | row Create New File ({name}) that the New File picker offers while typing a name
NEW | Close Active Window | NEW-K-4 | label | button tooltip in the Switch Window picker, stock only; the picker has no feature, only the macOS Window menu row is listed (TB-1.25)
EXCL | Troubleshooting Guide | label | not-ui | button of the vscode:showCredentialsError notice; the handler exists in window.ts but nothing in V, N or the Cursor out folder sends that message
NEW | Keybindings configuration | NEW-K-5 | label | title of the keybindings.json schema; its completions and checks have no feature, and stock adds systemWide shortcuts
EXCL | User View Container | label | not-ui | placeholder title of a generated container, code comment says it should not be shown anywhere
