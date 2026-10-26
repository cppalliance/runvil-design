Group: ED-9 Welcome, empty and first-run screens. Section: Editor. Date: 2026-10-25. Pass: 2.
Features: 26.

### ED-9.1 Welcome page
- Tag: both
- Evidence: source
- Boundary: no
- What: The Welcome editor tab. It has a product name header with the subtitle "Editing evolved", three lists (Start, Recent, Walkthroughs) and a footer with a "Show welcome page on startup" checkbox. On first launch stock also adds a telemetry notice, and when chat is available a "Try out the new Agents window" banner. Cursor ships an older copy of the same page.
- Commands: workbench.action.openWalkthrough; welcome.goBack
- Menus: MenubarHelpMenu
- Keys: Escape (welcome.goBack, when inWelcomeContext)
- Settings: workbench.welcomePage.preferReducedMotion
- Cursor: changed - same pane "Welcome" and the same lists, but there is no Welcome row in the Help menu and the "Try out the new Agents window" banner string is not in the bundle; the page opens only when `workbench.startupEditor` or a command asks for it
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.ts; V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.contribution.ts; C bundle (modules welcomeGettingStarted/browser/gettingStarted, gettingStarted.contribution)

### ED-9.2 Open the Welcome page or one walkthrough by command
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Welcome from the Command Palette or the Help menu to open the page. The same command takes a walkthrough id, or a category and step, and opens that walkthrough straight to the step, optionally beside the current editor.
- Commands: workbench.action.openWalkthrough; walkthroughs.selectStep
- Menus: MenubarHelpMenu
- Keys: none found
- Settings: none found
- Cursor: changed - `workbench.action.openWalkthrough` is registered with an empty title and no `f1`, so it is not in the palette and has no menu row; `walkthroughs.selectStep` is present
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.contribution.ts; C bundle (module welcomeGettingStarted/browser/gettingStarted.contribution)

### ED-9.3 Choose what opens at startup
- Tag: both
- Evidence: source
- Boundary: no
- What: The setting `workbench.startupEditor` decides what shows when no editor is restored (nothing, the Welcome page, the folder README, a new untitled file, the Welcome page only in an empty window, or a terminal in the editor area, plus one more stock value that opens an agent page left out of this catalog), the checkbox on the Welcome page flips between the Welcome page and nothing, and stock always opens the Welcome page on a first launch.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.startupEditor
- Cursor: changed - default is `none` (stock default is `welcomePage`) and only 6 values exist; `agentSessionsWelcomePage` is not there
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\browser\startupPage.ts; V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.contribution.ts; C bundle (module welcomeGettingStarted/browser/gettingStarted.contribution, nls 11683 to 11689)

### ED-9.4 Start list on the Welcome page
- Tag: both
- Evidence: source
- Boundary: no
- What: The Start column of buttons: New File..., Open... (macOS) or Open File... and Open Folder..., Clone Git Repository..., Connect to..., plus web-only entries. Stock also has "Generate New Workspace..." which starts a chat, and Cursor has no such row. Cursor also keeps an older "Open a Walkthrough..." entry that shows only when every walkthrough card is hidden.
- Commands: welcome.showNewFileEntries; workbench.action.files.openFileFolder; workbench.action.files.openFile; workbench.action.files.openFolder; workbench.action.files.openFolderViaWorkspace; git.clone; remoteHub.openRepository; workbench.action.remote.showMenu; workbench.action.remote.showWebStartEntryActions; welcome.newWorkspaceChat; welcome.showAllWalkthroughs
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - start entries are in the bundle array of `topLevel*` ids (no `topLevelNewWorkspaceChat`, extra `topLevelShowWalkthroughs`); the `welcome.showNewFileEntries` and `welcome.showAllWalkthroughs` ids are named by entries but no registration of either was found
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\common\gettingStartedContent.ts; V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.ts; C bundle (module welcomeGettingStarted/common/gettingStartedContent, nls 11789 to 11808)

### ED-9.5 Recent list on the Welcome page
- Tag: both
- Evidence: source
- Boundary: no
- What: The Recent column shows up to 5 recent folders or workspaces (10 when there are no walkthrough cards). Click opens it in this window, Ctrl or Cmd click opens a new window. "More..." opens the Open Recent picker. With no history it shows "You have no recent folders, open a folder to start." Stock adds a small X per row to remove it from the list.
- Commands: workbench.action.openRecent; workbench.action.files.openFolder
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - list, "More..." and the empty text are present (nls 11706 to 11711); the remove-from-list label `welcomePage.removeRecent` is not in the bundle
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.ts; C bundle (module welcomeGettingStarted/browser/gettingStarted)

### ED-9.6 Walkthrough cards on the Welcome page
- Tag: both
- Evidence: source
- Boundary: no
- What: The Walkthroughs column shows up to 5 cards ranked by featured, new and recently used. Each card has an icon, title, a progress bar ("3 of 8 steps complete"), a New or Updated badge and a Hide button. "More..." opens the full picker. If every card is hidden the column disappears.
- Commands: workbench.action.openWalkthrough
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - same card layout and strings (nls 11713 to 11720), but the built-in card list is empty (see ED-9.10), so only cards from extensions can show
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.ts; C bundle (module welcomeGettingStarted/browser/gettingStarted)

### ED-9.7 Open Walkthrough picker
- Tag: both
- Evidence: source
- Boundary: no
- What: A quick pick titled "Open Walkthrough..." that lists every walkthrough with its description and source, and opens the one you pick. Reached from the "More..." link on the Welcome page, and in stock also from a Help menu row and the Command Palette.
- Commands: welcome.showAllWalkthroughs
- Menus: MenubarHelpMenu
- Keys: none found
- Settings: none found
- Cursor: changed - the picker is there behind the "More..." link (`pickWalkthroughs`, nls 11697), but `welcome.showAllWalkthroughs` has no registration, so no palette or Help entry
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.contribution.ts; V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.ts; C bundle (module welcomeGettingStarted/browser/gettingStarted)

### ED-9.8 Walkthrough steps page
- Tag: both
- Evidence: source
- Boundary: no
- What: Opening a card slides to the detail view. A Go Back button sits on top, then the steps with a completion check, title, description and action links or buttons. The picked step shows its media on the right (image, SVG, markdown or video). The bottom has "Mark Done" and "Next Section", and a "Tip: Use keyboard shortcut" line when a step names a command with a key. Escape goes back to the list.
- Commands: welcome.goBack; walkthroughs.selectStep
- Menus: none found
- Keys: Escape (welcome.goBack, when inWelcomeContext)
- Settings: none found
- Cursor: present - strings for Go Back, Mark Done, Next Section and the keyboard tip are in the bundle (nls 11693, 11721, 11726, 11727); the step-check label is the older "Checkbox for Step {0}: Completed" wording
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.ts; V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStartedDetailsRenderer.ts; C bundle (module welcomeGettingStarted/browser/gettingStarted)

### ED-9.9 Mark a walkthrough step complete or incomplete
- Tag: both
- Evidence: source
- Boundary: no
- What: Click the check next to a step to flip it, or run the commands with a step id. Steps also complete on their own when the event they wait for happens (a command runs, a setting changes, a view opens). The card progress bar follows.
- Commands: welcome.markStepComplete; welcome.markStepIncomplete
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - both ids registered (nls 11679, 11680)
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.contribution.ts; V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStartedService.ts; C bundle (module welcomeGettingStarted/browser/gettingStarted.contribution)

### ED-9.10 Built-in walkthroughs
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Stock ships five walkthroughs: Setup (theme, video tutorial, and Copilot setup steps that hand off to chat), SetupWeb, SetupAccessibility, Beginner ("Learn the Fundamentals": settings, extensions, terminal, debugging, Git, tasks, shortcuts, workspace trust) and notebooks. A Help row also opens the accessibility one.
- Commands: workbench.action.openWalkthrough; workbench.action.getStartedWithAccessibilityFeatures
- Menus: MenubarHelpMenu
- Keys: none found
- Settings: none found
- Cursor: missing - the bundle's built-in walkthrough array is empty and none of the walkthrough or step ids (`Setup`, `pickColorTheme`, `scmClone`, `keybindingsAccessibility` and so on) were found
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\common\gettingStartedContent.ts; V\src\vs\workbench\browser\actions\helpActions.ts; C bundle (module welcomeGettingStarted/common/gettingStartedContent)

### ED-9.11 Reset walkthrough progress
- Tag: both
- Evidence: source
- Boundary: no
- What: A Developer command that clears every step's done state on the Welcome page, so cards look unseen again.
- Commands: resetGettingStartedProgress
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - id and both labels found (nls 11785 to 11787)
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStartedService.ts; C bundle (module welcomeGettingStarted/browser/gettingStartedService)

### ED-9.12 Walkthrough opens when an extension is installed
- Tag: both
- Evidence: source
- Boundary: no
- What: After an extension with a walkthrough installs, its walkthrough opens on its own in the Welcome editor, unless you turned that off. Extensions add cards through the `walkthroughs` contribution point.
- Commands: workbench.action.openWalkthrough
- Menus: none found
- Keys: none found
- Settings: workbench.welcomePage.walkthroughs.openOnInstall
- Cursor: changed - the setting and the open-on-install call are in the bundle, but the default is false (stock default is true)
- Source: V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.contribution.ts; V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStartedExtensionPoint.ts; C bundle (module welcomeGettingStarted/browser/gettingStarted.contribution, nls 11682)

### ED-9.13 Interactive Editor Playground
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A "Playground" editor tab, titled Editor Playground, that walks through editing features with code samples you can type into. It has its own Up, Down, Page Up and Page Down handling. Opened from a Help menu row or the Command Palette.
- Commands: workbench.action.showInteractivePlayground; workbench.action.interactivePlayground.arrowUp; workbench.action.interactivePlayground.arrowDown; workbench.action.interactivePlayground.pageUp; workbench.action.interactivePlayground.pageDown
- Menus: MenubarHelpMenu
- Keys: UpArrow, DownArrow, PageUp, PageDown (when the playground is focused)
- Settings: none found
- Cursor: missing - none of the five ids and no "Playground" pane label found
- Source: V\src\vs\workbench\contrib\welcomeWalkthrough\browser\walkThrough.contribution.ts; V\src\vs\workbench\contrib\welcomeWalkthrough\browser\walkThroughActions.ts; V\src\vs\workbench\contrib\welcomeWalkthrough\browser\editor\editorWalkThrough.ts

### ED-9.14 Agent Sessions Welcome page
- Tag: vscode
- Evidence: source
- Boundary: yes - the page lists chat sessions and promotes Copilot, so its content comes from chat; the page frame is a welcome page
- What: A second welcome editor with Start buttons (Open Recent or Open Folder, New file, Clone Git Repository), a list of agent sessions with "View All Sessions", a walkthrough carousel with Previous and Next, a dismissible Copilot notice, and the "Show welcome page on startup" checkbox. Picked with the `agentSessionsWelcomePage` startup value.
- Commands: workbench.action.openAgentSessionsWelcome
- Menus: none found
- Keys: none found
- Settings: workbench.startupEditor
- Cursor: missing - no command id, pane label or startup value found
- Source: V\src\vs\workbench\contrib\welcomeAgentSessions\browser\agentSessionsWelcome.contribution.ts; V\src\vs\workbench\contrib\welcomeAgentSessions\browser\agentSessionsWelcome.ts

### ED-9.15 Welcome Onboarding 2026 (stock first-run flow)
- Tag: vscode
- Evidence: source
- Boundary: yes - two of the three steps are chat sign-in and "Build with AI Agents"; the "Make It Yours" step (theme and keymap) is plain UI
- What: A full-window first-run overlay with a step list: Sign In (sync settings, AI features, GitHub), Make It Yours (pick a theme and a keyboard mapping), and Build with AI Agents. Shown on first launch when the experimental setting is on, or by command.
- Commands: workbench.action.welcomeOnboarding2026
- Menus: none found
- Keys: none found
- Settings: workbench.welcomePage.experimentalOnboarding
- Cursor: missing - neither the command id nor the setting is in the bundle (Cursor has its own flow, ED-9.16)
- Source: V\src\vs\workbench\contrib\welcomeOnboarding\browser\welcomeOnboarding.contribution.ts; V\src\vs\workbench\contrib\welcomeOnboarding\browser\onboardingVariationA.ts; V\src\vs\workbench\contrib\welcomeOnboarding\common\onboardingTypes.ts

### ED-9.16 Cursor first-run onboarding overlay
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the Plugins step installs plugins for agents and the Data Sharing step sets what Cursor may learn from prompts; the overlay itself is plain UI
- What: A full-window overlay (class `onboarding-v2-overlay`) with back and forward arrow buttons at the top left. Steps run in order: Welcome (Log In, Sign Up, right-click either to copy a sign-in link, and a Skip For Now link), Plugins ("Plugins give your agents important context", cards with Add and Added, then Continue or Start Building, and Skip), and Data Sharing (two choice cards, "Help Improve Cursor" and "Privacy Mode", then Continue or Finish, and Learn More). The Welcome step is skipped when you are already signed in, and the Data Sharing step can be skipped by a server flag. Starting it also switches the window to the unified agent layout with the sidebar on the right. If VS Code settings are found, a dialog offers to import them.
- Commands: cursor.startOnboarding
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (onboarding overlay components in workbench.desktop.main.js, step ids welcome, marketplace, data-sharing; module vs/workbench/contrib/onboarding/electron-sandbox/onboarding.contribution; context key `cursor.onboarding.showing`)

### ED-9.17 Start onboarding again on demand
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A Command Palette entry "Start Onboarding" (category Cursor) that runs the first-run flow again. It also sets a stored flag that the empty-window landing screen reads.
- Commands: cursor.startOnboarding
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (action class with id `cursor.startOnboarding`, title "Start Onboarding", f1)

### ED-9.18 Editor group watermark with shortcut hints
- Tag: both
- Evidence: source
- Boundary: no
- What: When an editor group is empty, a faint logo (stock) and a list of labeled commands with their key chords show in the middle. A row with no key chord is skipped. Stock shows Open Chat and Show All Commands for a folder window and tops up to 3 rows from Go to File, Find in Files, Start Debugging, Toggle Terminal and Open Settings; an empty window adds Open File, Open Folder and Open Recent. Cursor swaps in its own rows (Search Files, Open Browser, New Agent, Show or Hide Terminal, Show Files, Add Folder or Add Repository, Maximize Chat) and each row can be clicked to run it. Stock also has a toolbar slot above the list.
- Commands: workbench.action.showCommands; workbench.action.quickOpen; workbench.action.files.openFile; workbench.action.files.openFolder; workbench.action.files.openFileFolder; workbench.action.openRecent; workbench.action.files.newUntitledFile; workbench.action.findInFiles; workbench.action.terminal.toggleTerminal; workbench.action.debug.start; workbench.action.openSettings; workbench.action.chat.open; composer.openBrowserTab; workbench.action.togglePanel; workbench.view.explorer; workbench.action.addRootFolder; workbench.action.maximizeChatSize
- Menus: EditorGroupWatermarkToolbar
- Keys: none found
- Settings: workbench.tips.enabled
- Cursor: changed - own row set, clickable rows, hidden in a window with no folder (the landing screen takes over); colour id `editorWatermark.foreground`; the id behind the New Agent row is held in a bundle constant and was not resolved
- Source: V\src\vs\workbench\browser\parts\editor\editorGroupWatermark.ts; V\src\vs\platform\actions\common\actions.ts; C bundle (module vs/workbench/browser/parts/editor/editorGroupWatermark, nls 3734 to 3744)

### ED-9.19 Cursor landing screen for an empty window
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: When no folder or workspace is open and no editor is open, Cursor draws its own screen instead of the watermark. It holds a button group, a recent projects list, a plan badge, and (behind feature flags) cloud and agent prompts. Right after onboarding the layout changes (a stored flag marks the first landing). The screen records a viewed event with sign-in and plan details.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/browser/parts/editor/renderEditorEmptyScreen, class `editorCursorMainDiv`, event `editor.landing_screen_empty.viewed`)

### ED-9.20 Landing screen buttons
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A grid of buttons on the empty-window landing screen: New project, Open project, Clone repo, Connect via SSH, New Window. Clone repo runs the Git clone command, and when Git is missing it shows a warning "Clone repo needs Git. Install Git, or enable the built-in Git extension, then restart Cursor." with a Download Git action. Connect via SSH looks for a Remote SSH extension, asks to enable one that is off ("Enable and Reload"), and otherwise opens the Extensions view with `@recommended:remotes`.
- Commands: workbench.action.files.newProject; workbench.action.files.openFolder; git.clone; workbench.action.newWindow
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (modules vs/workbench/browser/parts/editor/renderEditorEmptyScreen, cloneRepoFromEmptyScreen; nls 3400, 3401, 3829, 3830)

### ED-9.21 Landing screen recent projects
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A "Recent projects" list on the landing screen with the first 5 entries. Click opens the project in this window, Ctrl (Cmd on macOS) click opens a new window. Right-click offers "Remove from recents". A "View all (n)" link shows when there are more than 5 and runs a command held in a bundle constant (not resolved, likely the Open Recent picker).
- Commands: editor.landing_screen_empty.remove_recent_project
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/browser/parts/editor/renderEditorEmptyScreen; events editor.landing_screen_empty.recent_project_clicked, remove_recent_project, view_all_clicked; nls 3831)

### ED-9.22 Landing screen plan badge
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A small plan label near the top of the landing screen. Paid plans show the plan name (Pro, Pro+, Ultra, Team or Enterprise) with a Settings link. Free, trial and Pro-trial users see "Free Plan", "Pro Trial" or "Pro+ Trial" with an Upgrade link. Signed-out users see a Sign in link. Settings opens the Cursor Settings General tab.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/browser/parts/editor/renderEditorEmptyScreen; event editor.landing_screen_empty.cta_clicked with cta settings or upgrade)

### ED-9.23 Landing screen cloud and agent prompts
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the buttons start GitHub connection, cloud agent steering, and the Agents window; they are agent entry points on a plain screen
- What: Extra calls to action under the buttons, gated by experiments: a "Connect GitHub" or "Connect Your Repos" button that opens a connect modal, a "Steer from iOS" button with its own modal, and a "Try a new window for running parallel agents" link.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/browser/parts/editor/renderEditorEmptyScreen; experiments ide_connect_git_repos_start, ide_steer_from_cloud_start; class empty-screen-glass-cta)

### ED-9.24 Empty text editor hint
- Tag: both
- Evidence: source
- Boundary: no
- What: In an empty untitled plain text file a one-line italic hint sits on line 1. Stock reads "Select a language (key) to get started. Start typing to dismiss or don't show this again." with the language link opening the language picker and a "don't show" link, and adds a "Generate code" link when inline chat exists. Right-click on the hint offers "Disable Empty Editor Hint". The hint goes away when you type.
- Commands: workbench.action.editor.changeLanguageMode; welcome.showNewFileEntries; inlineChat.start; workench.action.disableEmptyEditorHint
- Menus: none found
- Keys: none found
- Settings: workbench.editor.empty.hint; accessibility.verbosity.emptyEditorHint
- Cursor: changed - older wording "[[Press {0} to generate code.]] Start typing to dismiss." (nls 5014) and an aria label that offers select a language, fill with template, or open a different editor (nls 5015); the "fill with template" call uses `welcome.showNewFileEntries`, which has no registration in the bundle
- Source: V\src\vs\workbench\contrib\codeEditor\browser\emptyTextEditorHint\emptyTextEditorHint.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (module vs/workbench/contrib/codeEditor/browser/emptyTextEditorHint/emptyTextEditorHint)

### ED-9.25 New File picker
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A quick pick titled "New File..." with the placeholder "Select File Type or Enter File Name...". It lists Text File, notebook types and custom editor types, grouped, each with a configure-keybinding button. Typing a name offers "Create New File (name)". Opened from the File menu, the Welcome page Start list, and the empty text editor hint.
- Commands: welcome.showNewFileEntries; workbench.action.files.newUntitledFile
- Menus: MenubarFileMenu; NewFile
- Keys: Alt+CtrlCmd+WinCtrl+N (welcome.showNewFileEntries)
- Settings: none found
- Cursor: missing - the id appears three times in the bundle (as a start entry and as a hint call) but no registration and no "Select File Type or Enter File Name..." string were found
- Source: V\src\vs\workbench\contrib\welcomeViews\common\newFile.contribution.ts; C bundle (searched id and placeholder string)

### ED-9.26 Release notes page
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Show Release Notes opens an editor tab "Release Notes: {version}" that fetches the notes online and renders them as a web page. Key names in the text show your real key chords, setting links can be changed in place with a small popup, the page has a find box, and a "Show release notes after an update" checkbox sits on the page. A Developer command opens the current file as release notes.
- Commands: update.showCurrentReleaseNotes; update.showCurrentReleaseNotesFromCurrentFile; developer.showCurrentFileAsReleaseNotes
- Menus: MenubarHelpMenu
- Keys: none found
- Settings: update.showReleaseNotes
- Cursor: changed - `update.showCurrentReleaseNotes` is registered ("Show Release Notes", Help row) but it only opens the product's release notes URL; there is no release notes editor module and the `update.showReleaseNotes` setting was not found
- Source: V\src\vs\workbench\contrib\update\browser\releaseNotesEditor.ts; V\src\vs\workbench\contrib\update\browser\update.contribution.ts; V\src\vs\platform\update\common\update.config.contribution.ts; C bundle (module vs/workbench/contrib/update/browser/update.contribution, nls 11485 to 11490)

## Handed off
- Empty editor group toolbar (Lock Group, Close Group) and the empty group right-click menu (Split Up, Down, Left, Right, New Window, Lock Group, Close) to ED-1.
- Help menu rows that open a web page or chat (Documentation, Keyboard Shortcuts Reference, Video Tutorials, Tips and Tricks, Join Us on YouTube, Search Feature Requests, View License, Privacy Statement, Ask @vscode) to TB-1.
- Update-finished card and the "View changelog" button after an update to NT-6.
- "Bring your VS Code setup to Cursor" toast to the notifications section (NT).
- "Import settings from VS Code" dialog shown during Cursor onboarding (Import, Import without extensions, Cancel) to the dialogs section (DL).
- Welcome banner strip (`welcomeBanner`) to BN-2.
- Welcome text inside empty tree views (`viewsWelcome`) to SB-1.
- Editor placeholder pages for errors, large files and untrusted workspaces (`editorPlaceholder.ts`) to ED-10.
- Accessible view of the Welcome page (`gettingStartedAccessibleView.ts`) to whoever owns the accessible view (the parent listed it as unowned).

## Unclear
- Where Cursor's first-run onboarding belongs. It is a fixed overlay over the whole window (class `onboarding-v2-overlay`, with its own 35 px drag strip), not an editor tab, so it may fit a window-level group better than the editor section. It is listed here as ED-9.16 because the catalog names first-run onboarding in ED-9.
- Whether Cursor's Welcome page ever opens. The default `workbench.startupEditor` is `none`, the built-in walkthrough array is empty, and the Help row is gone. The page code is there and a command can open it. Nothing was run on screen.
- Landing screen details not read: what changes right after onboarding, the "View all" target, and the Steer from iOS modal contents.
- The stock Survey page (`workbench.action.openSurvey`, a Copilot feedback form) is left out. In stable builds the developer command does nothing and the form is opened by the Copilot extension, so it is only reachable through chat.
- `welcome.showAllWalkthroughs` and `welcome.showNewFileEntries` are named by Cursor's Welcome start entries and the empty text hint, but no registration of either was found in the bundle. They may be registered through a path my string searches did not match.
- Newer than 1.128 (seen only in N, not read): the watermark rows carry an icon each (`N\src\vs\workbench\browser\parts\editor\editorGroupWatermark.ts`), and the release notes folder adds `releaseNotesAccessibilityHelp.ts` and `releaseNotesTryouts.ts` (`N\src\vs\workbench\contrib\update\browser\`).
- The 43 stock getting-started id count from pass 1 was not rerun; ED-9.10 relies on the empty built-in array seen in the bundle instead.
