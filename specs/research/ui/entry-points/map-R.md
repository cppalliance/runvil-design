Bucket: R. Date: 2026-10-25. Pass: 4.
NEW | editor.action.announceCursorPosition | NEW-R-6 | command | Announce Cursor Position speaks Line and Column through the screen reader, Ctrl+Alt+Shift+G in screen reader mode; stock only, no hit in the Cursor bundle
EXCL | editor.action.gotoLine | command | other | registered only by the standalone Monaco editor (standaloneGotoLineQuickAccess.ts); the workbench Go to Line is workbench.action.gotoLine in QA-3.1
EXCL | editor.action.quickOutline | command | other | registered only by the standalone Monaco editor; the workbench Go to Symbol is workbench.action.gotoSymbol in QA-3.10
MAP | editor.action.selectionClipboardPaste | ED-2.28 | command | Paste Selection Clipboard, Linux only, reads the primary selection; ED-2.28 already lists editor.selectionClipboard and the Linux selection clipboard
NEW | editor.action.toggleScreenReaderAccessibilityMode | NEW-R-5 | command | Toggle Screen Reader Accessibility Mode flips editor.accessibilitySupport and announces the new state; Shift+Alt+F1; present in Cursor
MAP | editor.cancelOperation | ED-4.8 | command | Escape stops a running editor lookup such as Go to References or Peek References, context key cancellableOperation; no label or menu
EXCL | editor.tabs.addTab | command | agent | Add Tab button of the tab strip; only the Sessions (Agents) window fills its menu, hidden in a normal window, not in the Cursor bundle
MAP | list.find.replInputFocus | PN-6.11 | command | second key path to the Debug Console find box, Ctrl+Alt+F or F3 when the console view has focus; PN-6.11 lists only repl.action.find
EXCL | notebook.cell.chat.fixError | command | agent | Fix Cell Error sends the cell error to chat; not in Cursor
EXCL | notebook.cell.chat.start | command | agent | starts notebook cell chat
EXCL | notebook.cellOutput.addToChat | command | agent | Add Cell Output to Chat
EXCL | notebook.chat.selectAndInsertKernelVariable | command | agent | inserts a kernel variable into a chat request
EXCL | notebook.inlineChat.acceptChangesAndRun | command | agent | Accept and Run button of notebook inline chat
EXCL | scm.editor.triggerSetup | command | agent | Resolve Conflicts with AI, shown only while Copilot chat setup is not complete; not in the Cursor bundle
EXCL | vscode.notebook.variable | command | agent | notebook variable for chat requests
EXCL | vscode.testing.getControllersWithTests | command | internal | API command for extension code, no palette entry or menu
EXCL | vscode.testing.getTestsInFile | command | internal | API command for extension code, no palette entry or menu
NEW | workbench.action.closeOtherWindows | NEW-R-1 | command | Close Other Windows from the Command Palette; stock only, no hit in the Cursor bundle
NEW | workbench.action.inspectKeyMappings | NEW-R-7 | command | Developer: Inspect Key Mappings opens an untitled editor with the keybinding debug dump; present in Cursor
NEW | workbench.action.inspectKeyMappingsJSON | NEW-R-7 | command | Developer: Inspect Key Mappings (JSON) opens the layout and raw mapping as JSON; present in Cursor
MAP | workbench.action.removeRootFolder | SB-2.19 | command | palette form Workspaces: Remove Folder from Workspace... that asks which folder; SB-2.19 lists the Explorer row removeRootFolder
EXCL | workbench.action.showLanguageExtensions | command | extension-system | Search Marketplace Extensions for the extension row at the top of the language picker, opens the Extensions view
NEW | workbench.action.tasks.rerunAllRunningTasks | NEW-R-9 | command | Tasks: Rerun All Running Tasks restarts every running task; stock only, not in the Cursor bundle
NEW | workbench.action.url.openUrl | NEW-R-8 | command | Developer: Open URL asks for a URL and hands it to the URL service; present in Cursor
EXCL | markdown.editor.insertImageFromWorkspace | command | other | Markdown source editing, ruled out of scope
EXCL | markdown.editor.insertLinkFromWorkspace | command | other | Markdown source editing, ruled out of scope
NEW | markdown.findAllFileReferences | NEW-R-10 | command | Find File References on a Markdown file, shown in the References view; borderline since it works on Markdown sources but is navigation not editing, cut it if the Markdown scope is preview only
NEW | debug.confirmOnExit | NEW-R-12 | setting | never or always, asks before the window closes with a live debug session; no feature named it, only hand-off notes
MAP | editor.doubleClickSelectsBlock | ED-2.6 | setting | double-click next to a bracket or quote selects what is inside, default on; ED-2.6 describes it without naming the setting; stock only, not in the Cursor bundle
EXCL | editor.find | setting | not-ui | group name of the editor.find.* settings, not a setting on its own; every child is in ED-3
EXCL | editor.stickyScroll | setting | not-ui | group name of the editor.stickyScroll.* settings, not a setting on its own; the children are in ED-5.4 and ED-5.5
EXCL | search.experimental.useIgnoreFilesInFindFiles | setting | not-ui | changes only how the legacy findFiles extension API treats ignore files, experimental, nothing visible
MAP | search.location | SB-1.12 | setting | deprecated switch that put the Search view in the side bar or the panel; replaced by dragging the view, which SB-1.12 covers; present in Cursor
EXCL | search.maintainFileSearchCache | setting | not-ui | deprecated, the message says the setting is no longer needed, no visible effect
EXCL | search.searchView.keywordSuggestions | setting | agent | keyword suggestions of the stock AI search in the Search view; no hit in the Cursor bundle
EXCL | search.searchView.semanticSearchBehavior | setting | agent | when the stock Search view asks for semantic results from an AI provider; no hit in the Cursor bundle
MAP | search.useRipgrep | SB-3.3 | setting | deprecated, now falls back on search.usePCRE2, which SB-3.3 already lists as deprecated
NEW | window.closeWhenEmpty | NEW-R-3 | setting | closing the last editor also closes the window, default off; present in Cursor
NEW | window.newWindowDimensions | NEW-R-2 | setting | size and place of a new window, five values; present in Cursor
NEW | window.openWithoutArgumentsInNewWindow | NEW-R-4 | setting | on or off, what a second launch without arguments does; default on for Windows and Linux and off for macOS
NEW | workbench.externalBrowser | NEW-R-13 | setting | browser name or path used for http and https links that leave the app; present in Cursor and marked restricted
EXCL | git.addAICoAuthor | setting | agent | adds an AI Co-authored-by trailer to commits that hold AI-generated code; absent from the Cursor git extension
MAP | git.countBadge | AB-3.8 | setting | all, tracked or off for the Git count on the Source Control icon; AB-3.8 names scm.countBadge and testing.countBadge only
EXCL | markdown.editor.pasteUrlAsFormattedLink.enabled | setting | other | Markdown source editing, ruled out of scope
EXCL | markdown.editor.updateLinksOnPaste.enabled | setting | other | Markdown source editing, ruled out of scope
MAP | markdown.links.openLocation | ED-10.13 | setting | opens a linked file in the active group or beside; in the 1.128 source only the preview uses it, same call as the label row in bucket I
EXCL | markdown.occurrencesHighlight.enabled | setting | other | highlights link occurrences in the Markdown source editor, same call as its label row in bucket I
EXCL | markdown.preferredMdPathExtensionStyle | setting | other | link path completion style in Markdown source, ruled out of scope
EXCL | markdown.server.log | setting | developer-only | log level of the Markdown language server, a debugging aid
EXCL | markdown.trace.server | setting | developer-only | traces messages to the Markdown language server, a debugging aid
EXCL | cursor.browserView.configureDialogHandling | command | agent | lets the agent accept or dismiss a page's own alert box, API command with owner agent argument
EXCL | cursor.configureDebugTimeout | command | developer-only | Configure: Debug Timeout Prevention, runs only when isDevelopment or isDevUser is true; picks cursor.debug.timeoutPrevention
EXCL | cursor.useLocalhostUpdateServer | command | developer-only | Developer: Use Localhost Update Server, needs the update_use_localhost feature gate or it shows an error
EXCL | testing.getSelectedProfiles | command | internal | API command for extension code, no palette entry
EXCL | testing.startContinuousRunFromExtension | command | internal | API command for extension code, no palette entry
EXCL | testing.stopContinuousRunFromExtension | command | internal | API command for extension code, no palette entry
EXCL | workbench.action.customize.clearPrimitiveAuthorshipCache | command | developer-only | Developer category, precondition isDevUser, clears a cache of the Customize editor
EXCL | workbench.action.customize.clearPrimitiveSourceCache | command | developer-only | Developer category, precondition isDevUser, clears a cache of the Customize editor
MAP | workbench.action.openGlobalSettingsCursorAlias | ED-8.1 | command | Cursor-only palette row Open VS Code Settings that opens the same User Settings as workbench.action.openGlobalSettings
EXCL | workbench.parts.unifiedsidebar | command | agent | part id of the Agents sidebar, not a command
MAP | DebugCallStackToolbar | PN-6.25 | menu | inline button of a stack frame (callStackWidget.goToFile) in the Test Results stack trace, not the Call Stack view
EXCL | EditorTitleRun | menu | extension-system | submenu behind the Run or Debug split button on the editor title; in Cursor only extensions fill it, and no built-in extension does
EXCL | EditorTabsBarAddTab | menu | agent | menu of the Add Tab button, filled only by the Sessions (Agents) window
MAP | OpenEditorsContextShare | ED-1.28 | menu | Share submenu of the Open Editors right-click menu, sibling of the Explorer and editor Share menus listed there
EXCL | ChatContext | menu | agent | right-click menu of chat messages
EXCL | ChatEditingDeletedNotebookCell | menu | agent | menu of a deleted notebook cell shown by chat editing
EXCL | AgentsTitleBarControlMenu | menu | agent | Agents control of the command center, left out by TB-2
MAP | Move Last Selection to Next Find Match | ED-2.3 | label | title of editor.action.moveSelectionToNextFindMatch, listed in ED-2.3
NEW | Find File References | NEW-R-10 | label | title of markdown.findAllFileReferences, Explorer and tab right-click menus
EXCL | Insert Link to File in Workspace | label | other | title of markdown.editor.insertLinkFromWorkspace, Markdown source editing
EXCL | Insert Image from Workspace | label | other | title of markdown.editor.insertImageFromWorkspace, Markdown source editing
MAP | Reopen as Text | ED-1.27 | label | diff editor title button for custom editors, runs workbench.action.reopenTextEditor which ED-1.27 lists
MAP | Expand Editor Group and Hide Side Bars | ED-1.20 | label | title of workbench.action.minimizeOtherEditorsHideSidebar, listed in ED-1.20
MAP | Quick Open Previous Editor from History | ED-1.30 | label | title of workbench.action.openPreviousEditorFromHistory, listed in ED-1.30
EXCL | Add Tab | label | agent | tooltip of the Add Tab button, only the Sessions (Agents) window shows it
EXCL | Manage Dynamic Authentication Providers | label | agent | Accounts menu row for dynamic (MCP) auth providers, no hit in the Cursor bundle; AB-4 skipped it
NEW | Quick Open Browser Tab | NEW-R-11 | label | title of workbench.action.browser.quickOpen, the open tabs list; stock only
NEW | Toggle Screen Reader Accessibility Mode | NEW-R-5 | label | title of editor.action.toggleScreenReaderAccessibilityMode
NEW | Announce Cursor Position | NEW-R-6 | label | title of editor.action.announceCursorPosition
NEW | Inspect Key Mappings | NEW-R-7 | label | title of workbench.action.inspectKeyMappings, Developer category
NEW | Inspect Key Mappings (JSON) | NEW-R-7 | label | title of workbench.action.inspectKeyMappingsJSON, Developer category
MAP | Paste Selection Clipboard | ED-2.28 | label | title of editor.action.selectionClipboardPaste, Linux only
EXCL | Remove Visualizer | label | extension-system | Variables and Watch row that appears only when an extension supplies a visualizer; no built-in provider
EXCL | Visualize Variable | label | extension-system | Variables and Watch row that appears only when an extension supplies a visualizer; no built-in provider
EXCL | Toggle Notebook Clipboard Troubleshooting | label | developer-only | workbench.action.toggleNotebookClipboardLog, Developer category, same call as the sibling notebook developer commands in bucket C
EXCL | Reset notebook getting started | label | developer-only | workbench.notebook.layout.gettingStarted, Developer category, needs an experimental setting
MAP | Select All Occurrences of Find Match | ED-7.32 | label | title of notebook.selectAllFindMatches, Ctrl+Shift+L in notebook multi-cursor mode
MAP | Add Selection to Next Find Match | ED-7.32 | label | title of notebook.addFindMatchToSelection, Ctrl+D in notebook multi-cursor mode
EXCL | Toggle Notebook Layout Troubleshoot | label | developer-only | notebook.toggleLayoutTroubleshoot, Developer category, same call as bucket C
EXCL | Inspect Notebook Layout | label | developer-only | notebook.inspectLayout, Developer category, same call as bucket C
EXCL | Clear Notebook Editor Type Cache | label | developer-only | notebook.clearNotebookEdtitorTypeCache, Developer category, same call as bucket C
EXCL | Reset Notebook Webview | label | internal | workbench.notebook.layout.webview.reset has f1 false and nothing calls it
EXCL | Toggle Notebook View Zones | label | developer-only | notebook.developer.addViewZones, Developer category, development builds only
NEW | Rerun All Running Tasks | NEW-R-9 | label | title of workbench.action.tasks.rerunAllRunningTasks, Tasks category
MAP | Switch Terminal | PN-2.15 | label | title of workbench.action.terminal.switchTerminal, the terminal drop-down shown when tabs are hidden
MAP | Filter Coverage to Test | ED-13.5 | label | title of testing.coverageFilterToTestInEditor, the filter button of the coverage bar in the editor
NEW | Open URL | NEW-R-8 | label | title of workbench.action.url.openUrl, Developer category
NEW | Close Other Windows | NEW-R-1 | label | title of workbench.action.closeOtherWindows
