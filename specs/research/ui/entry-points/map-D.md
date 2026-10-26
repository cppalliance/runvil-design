Bucket: D. Date: 2026-10-25. Pass: 4.
MAP | browser.configureSearchEngine | ED-10.68 | command | gear button on the search row of the stock location bar picker, opens the search engine setting; not a registered command; [stock only]
MAP | browser.favorites.delete | ED-10.37 | command | trash button on a Favorites row in the stock location bar picker (rows shown in ED-10.81); not a registered command; [stock only]
MAP | browser.history.delete | ED-10.65 | command | Remove from History button on a history row in the stock location bar suggestions; not a registered command; [stock only]
MAP | browser.toggleFavorite | ED-10.37 | command | star toggle inside the stock location bar picker (add or remove favorite); not a registered command; [stock only]
MAP | _markdown.copyImage | ED-10.16 | command | script missed it because of the leading underscore; ED-10.16 already names it (Markdown preview image right-click Copy Image)
MAP | _markdown.openFrontMatterSettings | ED-10.17 | command | script missed it because of the leading underscore; ED-10.17 already names it (front matter right-click)
MAP | _markdown.openImage | ED-10.16 | command | script missed it because of the leading underscore; ED-10.16 already names it (local image right-click Open Image)
NEW | browser.splitEditorWithNewBrowserTab | NEW-D-1 | command | Cursor override used by Split Editor in Group when the active editor is a Browser tab: pairs it with a new Browser tab on the same page
NEW | browser.splitEditorWithNewBrowserTabToDirection | NEW-D-1 | command | Cursor override used by Split Editor Up, Down, Left and Right: opens a new Browser tab on the same page in the new group
EXCL | browserView.blur | command | agent | agent tool helper that blurs the page view; no menu, key or button
MAP | cursor.browserAutomation.captureWebviewScreenshot | ED-10.53 | command | low level capture that the Browser Menu Take Screenshot and Capture Area Screenshot reach through cursor.browserAutomation.internal.captureScreenshot; the agent tool shares it
EXCL | cursor.browserAutomation.internal.navigateWebview | command | internal | helper that navigates a view to a url; registered but nothing in the bundle calls it
EXCL | cursor.browserOriginAllowlist.ensureNavigationAllowed | command | agent | checks an agent navigation against an administrator allowlist and throws; no screen lists or edits the allowlist
EXCL | cursor.browserOriginAllowlist.ensurePageOriginAllowed | command | agent | same administrator allowlist check for agent browser tools; only seen as an error text in a tool result
EXCL | cursor.browserView.closeTab | command | agent | agent tool for closing a tab; the person closes a tab through ED-10.30
EXCL | cursor.browserView.executeJavaScript | command | agent | agent tool that runs script in the page
EXCL | cursor.browserView.executeJavaScriptInIsolatedWorld | command | agent | agent tool that runs script in an isolated world
EXCL | cursor.browserView.focusOrShowTab | command | agent | agent tool that brings a tab (maybe hidden) to the front; called from the browser automation extension
EXCL | cursor.browserView.getConsoleLogs | command | agent | agent tool that reads page console logs
EXCL | cursor.browserView.getNetworkRequests | command | agent | agent tool that reads page network requests
MAP | cursor.browserView.getRecordingType | ED-10.59 | command | the red Recording indicator on the page reads this value and draws the text Recording followed by it
EXCL | cursor.browserView.getTitle | command | agent | agent tool that reads the page title
EXCL | cursor.browserView.getURL | command | agent | agent tool that reads the page address
EXCL | cursor.browserView.isHeadless | command | agent | agent tool that asks whether a tab is a hidden agent tab
MAP | cursor.browserView.isLocked | ED-10.59 | command | the lock overlay reads this state; person-side control is the Take control button
EXCL | cursor.browserView.listTabs | command | agent | agent tool that lists open tabs
EXCL | cursor.browserView.navigate | command | agent | agent tool that loads a url; the person types in the location bar (ED-10.31)
EXCL | cursor.browserView.newHeadlessTab | command | agent | agent tool that opens a tab with no visible editor
EXCL | cursor.browserView.newTab | command | agent | agent tool that opens a tab; the person opens one with ED-10.29; called by the browser automation extension
MAP | cursor.browserView.resize | ED-10.60 | command | agent resize sets the viewport override that the phone icon badge and its Reset button show; the setter has no person-side entry
EXCL | cursor.browserView.selectTab | command | agent | agent tool that activates a tab
EXCL | cursor.browserView.sendCDPCommand | command | agent | agent tool that sends a raw DevTools protocol command
MAP | cursor.browserView.setLocked | ED-10.59 | command | turns the lock overlay on or off; the Take control button calls the same setter with false
MAP | cursor.browserView.setRecordingType | ED-10.59 | command | sets the value drawn after Recording in the red indicator; answers ED-10.59 open question on what shows
EXCL | cursor.browserView.showHeadlessTab | command | agent | agent tool that turns a hidden agent tab into an editor tab
EXCL | cursor.browserView.takeScreenshot | command | agent | agent tool with ref, element and fullPage options; the person path is ED-10.53
EXCL | cursor.browserView.updateScreenshot | command | agent | feeds the screenshot store for agent tab thumbnails
MAP | cursor.connectSlack | ED-8.24 | command | opens the Slack install url in the system browser; hidden from the palette; visible entry is the Connect Slack row in the Integrations list of the Cloud Agents tab
EXCL | cursor.dev.crashExtensionHost | command | developer-only | Developer: Crash Extension Host (with slow recovery); palette entry only when isDevelopment or isDevUser is set
EXCL | cursor.dev.crashRemoteExtensionHost | command | developer-only | Developer: Crash Remote Extension Host (with slow recovery); same dev-only gate, also remote
MAP | cursor.enableAutoReviewOnCommit | SB-4.39 | command | sets autoRunBugbotOnCommit to true; only called from the in-app promotion modal; hidden from the palette
EXCL | cursor.mentionLinkToolbar | command | agent | small toolbar over a mention link in the chat input; hidden from the palette
NEW | cursor.openCreatedRule | NEW-D-2 | command | Open Rule button on the notice shown after Create Rule from the editor right-click menu; the parent command cursor.createRuleFromSelection is not in any work file
EXCL | cursor.publicLogCapture | command | internal | analytics event sender called by the feedback pane; hidden from the palette
MAP | cursor.publishSkill.copyLink | ED-8.36 | command | Copy link button on the notice "Published skill to your team" after publishing from the Skills tab
NEW | cursor.removevim | NEW-D-3 | command | palette entry Cursor Vim: Remove Vim Mode uninstalls the Vim extension and reloads the window
EXCL | cursor.spaceXMode | command | not-ui | not a command; a source id for default setting overrides (dev container and port forwarding defaults) when a hidden mode is on
EXCL | cursor.tsgoDisableAutoImportsInternalExperiment | command | not-ui | not a command; a source id for default setting overrides in an internal tsgo auto import experiment
NEW | cursorAuth.toggleAuthDebug | NEW-D-4 | command | palette entry Developer: Toggle Auth Debugging, no condition; opens the Cursor Auth Debug output channel
EXCL | cursorAuth.triggerTokenRefresh | command | not-ui | palette entry Trigger Token Refresh with no visible result; auth code also calls it after a refused token
EXCL | cursorai.action.switchToDynamicModelSlug | command | agent | switches the chat model from a remote config value; hidden from the palette
EXCL | cursorai.action.switchToModelSlug | command | agent | switches the chat model; called by the in-app promotion modal; hidden from the palette
MAP | DebugDisassemblyContext | ED-10.45 | menu | right-click menu inside the Disassembly View with Copy Address and Toggle Breakpoint; the menu id is not in the Cursor bundle
MAP | DebugCreateConfiguration | SB-5.5 | menu | extra rows at the end of the Select debugger list shown when launch.json is created; proposed extension point, no stock rows
MAP | DebugScopesContext | SB-5.11 | menu | right-click on a scope header in the Variables view; the only stock row is Add to Chat (chat, not cataloged); the menu id is not in the Cursor bundle
EXCL | EditorTitleLayout | menu | agent | layout buttons on the editor tab strip, filled only by the Agents window code in the sessions folder; the id is not in the Cursor bundle
EXCL | MenubarHomeMenu | menu | web-only | home indicator menu in the web build only
MAP | SCMResourceContext | SB-4.2 | menu | right-click a changed file in the Source Control list (scm/resourceState/context): Stage, Unstage, Discard and Open
EXCL | SCMResourceContextShare | menu | extension-system | Share submenu in the changed file right-click menu; filled only by share providers from extensions and off by default; Cursor link rows sit in other Share menus (ED-1.28)
MAP | SCMResourceFolderContext | SB-4.2 | menu | right-click a folder in the tree view of the Source Control list (scm/resourceFolder/context): stage or discard the folder
MAP | SCMSourceControlInline | SB-4.28 | menu | repository row menu (scm/repository) with Pull, Push, Fetch, Checkout and the Git submenus; fits the repository rows of the Repositories view
MAP | SCMHistoryTitle | SB-4.34 | menu | title buttons of the Graph view (scm/history/title): repository and ref pickers, refresh, fetch, pull, push
MAP | SCMHistoryItemRefContext | SB-4.36 | menu | right-click a branch or tag badge in the Graph (scm/historyItemRef/context): checkout, delete
MAP | SCMArtifactGroupContext | SB-4.30 | menu | right-click a group such as Branches or Tags in the experimental Repositories explorer (scm/artifactGroup/context); [stock only], missing in Cursor
MAP | SCMArtifactContext | SB-4.30 | menu | right-click a branch, tag, stash or worktree in the experimental Repositories explorer (scm/artifact/context); [stock only], missing in Cursor
MAP | TestMessageContent | PN-6.22 | menu | floating menu over the failure message pane in Test Results and the test peek (testing/message/content); no built in rows, only extensions add rows
MAP | ViewItemContext | SB-7.24 | menu | view/item/context right-click on a row of a contributed tree view; the in-scope user is the References view (Copy, Copy All, Copy Path); other users are Settings Sync and Edit Sessions views which are out of scope
EXCL | TimelineTitleContext | menu | not-ui | defined in actions.ts but never used by any registration or consumer in 1.128.0; Cursor only defines it too
EXCL | MultiDiffEditorContent | menu | agent | floating toolbar over the multi-file diff editor; the only core row is the chat Explain Changes action; the id is not in the Cursor bundle
EXCL | BrowserChatActionsMenu | menu | agent | stock Browser submenu of Add to Chat actions (element, console logs, screenshot); chat, handed off in ED-10; not in the Cursor bundle
EXCL | AgentSessionsViewerFilterSubMenu | menu | agent | filter submenu of the agent sessions list in the chat view
EXCL | AgentSessionsContext | menu | agent | right-click menu of an agent session row
EXCL | AgentSessionSectionContext | menu | agent | right-click menu of an agent sessions section header
EXCL | AgentSessionsCreateSubMenu | menu | agent | submenu for creating a new agent session type
EXCL | AgentSessionsToolbar | menu | agent | toolbar of the agent sessions list
EXCL | AgentSessionItemToolbar | menu | agent | inline buttons on an agent session row
EXCL | AgentSessionSectionToolbar | menu | agent | inline buttons on an agent sessions section header
EXCL | SessionItemContextMenu | menu | agent | right-click menu of a session row in the sessions app (Agents window code)
EXCL | SessionHeaderContext | menu | agent | session header menu in the sessions app (Agents window code)
EXCL | AgentsChangesToolbar | menu | agent | toolbar of the changes view in the sessions app (Agents window code)
EXCL | AgentsChangesPrimaryActionSubMenu | menu | agent | primary action dropdown of the changes view in the sessions app
EXCL | AgentsChangeInlineToolbar | menu | agent | inline buttons on a changed file row in the sessions app
