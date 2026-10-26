Bucket: C. Date: 2026-10-25. Pass: 4.
MAP | callStackWidget.goToFile | PN-6.25 | command | Open File button on each stack frame row of the test stack trace widget
EXCL | debug.attachToCurrentCodeRenderer | command | developer-only | throws unless the window is in extension development; no menu or key; no copy in the Cursor bundle
MAP | debug.enableOrDisableBreakpoint | ED-11.1 | command | flips the breakpoint on the cursor line between enabled and disabled; bindable but no default key
EXCL | debug.installAdditionalDebuggers | command | extension-system | Run menu row and the Install buttons only open the Extensions view search for debuggers
NEW | debug.startFromConfig | NEW-C-1 | command | called by the Debug row in the Process Explorer right-click menu to start a Node attach session
MAP | editor.action.findReferences | ED-4.9 | command | argument-taking command that opens the References peek at a given position; called by extensions, no menu or key
MAP | editor.action.format | ED-4.17 | command | old hidden id that formats the selection or the whole file, kept for existing key bindings
EXCL | editor.action.goToLocation | command | internal | action id of a throwaway class created inside goToLocations and never registered
MAP | editor.action.goToLocations | ED-4.33 | command | extension command that jumps to a list of places and honors the editor.gotoLocation.multiple behaviors
EXCL | editor.action.gotoOffset | command | other | standalone Monaco editor id only; the workbench Go to Offset is workbench.action.gotoOffset in QA-3.3; the script source line points at the wrong file
EXCL | editor.action.inspectTokens | command | other | standalone Monaco editor only and not in the Cursor bundle; the workbench token inspector is editor.action.inspectTMScopes in ED-5.39
MAP | editor.action.peekLocations | ED-4.10 | command | opens the peek window on a list of places; the editor.action.showReferences alias that code lens links use points here
EXCL | editor.action.toggleHighContrast | command | other | standalone Monaco editor only; not in the workbench or in the Cursor bundle
NEW | editor.contrib.contextmenu | NEW-C-2 | command | controller id behind the editor right-click menu and Shift+F10; the script saw it through the comment box editor
MAP | editor.contrib.menuPreventer | TB-1.5 | command | contribution id; stops an Alt plus click in the editor from counting as an Alt tap that reveals the menu bar
NEW | editor.debug.action.copyAddress | NEW-C-3 | command | Copy Address row in the Disassembly View right-click menu; stock only
MAP | editor.tabCompletionController | AC-4.9 | command | contribution id of the Tab completion controller for snippet prefixes
EXCL | notebook.cell.chat.explainError | command | agent | sends the cell error to the chat panel with an explain request
EXCL | notebook.cell.chat.startAtTop | command | agent | Generate button above the first cell that starts chat to write code
EXCL | notebook.cell.nullAction | command | internal | no-op that only reserves Up and Down for the output page; not in the palette
NEW | notebook.cell.output.selectAll | NEW-C-4 | command | Ctrl+A with focus in a cell output selects the output text
EXCL | notebook.clearNotebookEdtitorTypeCache | command | developer-only | Developer category palette command that clears an internal editor type cache
EXCL | notebook.developer.addViewZones | command | developer-only | Developer category and only enabled in a development build
EXCL | notebook.diff.cell.switchOutputRenderingStyle | command | not-ui | only in commented-out code; the live sibling switchOutputRenderingStyleToText is in ED-7.35
EXCL | notebook.inspectLayout | command | developer-only | Developer category palette command that logs cell layout to the console
EXCL | notebook.toggleLayoutTroubleshoot | command | developer-only | Developer category palette command that toggles layout debugging
MAP | workbench.action.debug.stepIntoTarget | ED-11.22 | command | second entry point for Step Into Target: Ctrl+F11 and the palette row while stopped
NEW | workbench.action.files.revertEditors | NEW-C-5 | command | Revert or Revert All button on the failed save notification
NEW | workbench.action.files.saveEditors | NEW-C-5 | command | Retry button on the failed save notification
NEW | workbench.action.files.saveFiles | NEW-C-6 | command | File: Save All Files in the palette, skips untitled files
NEW | workbench.action.files.saveWithoutFormatting | NEW-C-7 | command | Ctrl+K S and the palette row save while skipping format on save and other save-time steps
EXCL | workbench.action.tasks.clean | command | not-ui | only in commented-out code and never registered; no copy in the Cursor bundle
NEW | workbench.action.tasks.manageAutomaticRunning | NEW-C-8 | command | Tasks: Manage Automatic Tasks picker and the folder-open task prompt; Cursor changes the default
NEW | workbench.action.tasks.openWorkspaceFileTasks | NEW-C-9 | command | Tasks: Open Workspace Tasks, shown only for a saved multi-root workspace
NEW | workbench.action.tasks.reRunTask | NEW-C-10 | command | Tasks: Rerun Last Task in the palette
EXCL | workbench.action.tasks.rebuild | command | not-ui | only in commented-out code and never registered; no copy in the Cursor bundle
NEW | workbench.action.tasks.showLog | NEW-C-11 | command | Tasks: Show Task Log opens the Tasks channel in the Output panel
MAP | workbench.action.tasks.toggleProblems | PN-4.1 | command | runs the Problems view toggle; no palette row, menu or default key
EXCL | workbench.action.toggleNotebookClipboardLog | command | developer-only | Developer category palette command that turns on clipboard logging
EXCL | workbench.debug.action.addScopeToChat | command | agent | Add to Chat row in the Variables scope menu
EXCL | workbench.debug.action.addVariableToChat | command | agent | Add to Chat row in the Variables right-click menu
EXCL | workbench.debug.action.addWatchExpressionToChat | command | agent | Add to Chat row in the Watch right-click menu
MAP | workbench.debug.watchExpressionsView | SB-5.17 | command | view id of the Watch view, which also acts as its open-view command
EXCL | workbench.files.openFile | command | not-ui | telemetry event name used when a file row is opened in the Explorer; behavior is SB-2.3
EXCL | workbench.notebook.layout.gettingStarted | command | developer-only | Developer category row that resets a stored flag; needs an experimental setting; missing from Cursor
EXCL | workbench.notebook.layout.webview.reset | command | internal | hidden from the palette and called by no menu or code in the source
MAP | workbench.panel.repl | PN-6.1 | command | container id of the Debug Console panel tab
MAP | workbench.panel.repl.view | PN-6.1 | command | view id of the Debug Console, which also acts as its open-view and focus command
NEW | workbench.scm.action.focusNextResourceGroup | NEW-C-12 | command | moves keyboard focus to the next file group in the Changes view; no default key
NEW | workbench.scm.action.focusPreviousInput | NEW-C-12 | command | moves focus to the previous commit message box; no default key
NEW | workbench.scm.action.focusPreviousResourceGroup | NEW-C-12 | command | moves keyboard focus to the previous file group in the Changes view; no default key
EXCL | workbench.scm.action.graph.addHistoryItemChangeToChat | command | agent | Graph row that adds a commit change to chat
EXCL | workbench.scm.action.graph.addHistoryItemToChat | command | agent | Graph row that adds a commit to chat
EXCL | workbench.scm.action.graph.summarizeHistoryItem | command | agent | Graph row that asks chat to summarize a commit
MAP | workbench.scm.history | SB-4.34 | command | view id of the Source Control Graph, which also acts as its open-view command
EXCL | git.api.getRemoteSources | command | internal | API command hidden from the palette and called by extensions
EXCL | git.api.getRepositories | command | internal | API command hidden from the palette and called by extensions
EXCL | git.api.getRepositoryState | command | internal | API command hidden from the palette and called by extensions
MAP | git.commitAllSignedNoVerify | SB-4.9 | command | Commit All (Signed Off, No Verify) in the Commit submenu when no-verify commits are allowed
MAP | git.commitEmptyNoVerify | SB-4.9 | command | Commit Empty (No Verify) palette row when no-verify commits are allowed
MAP | git.commitSignedNoVerify | SB-4.9 | command | Commit (Signed Off, No Verify) in the Commit submenu when no-verify commits are allowed
MAP | git.commitStagedSignedNoVerify | SB-4.9 | command | Commit Staged (Signed Off, No Verify) in the Commit submenu when no-verify commits are allowed
EXCL | git.fetchRef | command | internal | hidden from the palette, no menu row and no caller in the source; sibling of the Graph pull and push ref commands
MAP | git.repositories.copyBranchName | SB-4.30 | command | Copy Branch Name row in the Repositories view branch menu; stock only
MAP | git.repositories.copyCommitHash | SB-4.30 | command | Copy Commit Hash row in the branch and tag menus of the Repositories view; stock only
MAP | git.repositories.copyStashName | SB-4.30 | command | Copy Stash Name row in the stash menu of the Repositories view; stock only
MAP | git.repositories.copyTagName | SB-4.30 | command | Copy Tag Name row in the tag menu of the Repositories view; stock only
MAP | git.repositories.createFrom | SB-4.30 | command | Create from row in the branch menu of the Repositories view; stock only
MAP | git.repositories.deleteTag | SB-4.30 | command | Delete row in the tag menu of the Repositories view; stock only
MAP | git.repositories.openWorktreeInNewWindow | SB-4.30 | command | Open in New Window button and row for a worktree in the Repositories view; stock only
MAP | git.repositories.stashCopyBranchName | SB-4.30 | command | Copy Branch Name row in the stash menu of the Repositories view; stock only
MAP | git.repositories.stashView | SB-4.30 | command | View Stash row in the stash menu of the Repositories view; stock only
MAP | git.repositories.worktreeCopyBranchName | SB-4.30 | command | Copy Branch Name row in the worktree menu of the Repositories view; stock only
MAP | git.repositories.worktreeCopyCommitHash | SB-4.30 | command | Copy Commit Hash row in the worktree menu of the Repositories view; stock only
MAP | git.repositories.worktreeCopyPath | SB-4.30 | command | Copy Worktree Path row in the worktree menu of the Repositories view; stock only
NEW | editor.action.accessibilityHelpConfigureAssignedKeybindings | NEW-C-13 | command | Alt+A in Accessibility Help lists commands that already have keys so you can change them
NEW | editor.action.accessibilityHelpConfigureKeybindings | NEW-C-13 | command | Alt+K in Accessibility Help lists commands without keys so you can add one
NEW | editor.action.accessibilityHelpOpenHelpLink | NEW-C-14 | command | Alt+H in Accessibility Help opens the page's read more link; the script wrongly called this Cursor only
NEW | editor.action.accessibleViewDisableHint | NEW-C-15 | command | Alt+F6 turns off the hint setting of the page being read; stock has it too
NEW | editor.action.accessibleViewGoToSymbol | NEW-C-16 | command | Ctrl+Shift+O in the accessible view opens a symbol picker; stock has it too
MAP | editor.landing_screen_empty.remove_recent_project | ED-9.21 | command | Remove from recents row in the right-click menu of a recent project on the landing screen
MAP | editor.pasteAs.default | ED-2.29 | command | id of the Configure default paste action row in the Paste As picker; not a command
