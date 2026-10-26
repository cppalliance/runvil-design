Group: ED-9 Welcome, empty and first-run screens. Pass: 3. Date: 2026-10-25.

### ED-9.1
1. The Welcome tab opens with the product name as a header and the subtitle "Editing evolved" under it.
2. The page has three lists, Start, Recent and Walkthroughs, with a footer below them.
3. The footer has a "Show welcome page on startup" checkbox whose tooltip reads "When checked, this page will be shown on startup."
4. On a first launch the footer also shows a notice that links to the privacy statement and an opt out. [stock only]
5. When chat is available the page shows a "Try out the new Agents window" banner. [stock only]
6. Choosing Welcome in the Help menu opens the Welcome page. [stock only]
7. Setting `workbench.welcomePage.preferReducedMotion` reduces motion on the Welcome page, and it is deprecated in favor of `workbench.reduceMotion`.
8. In Cursor the page is an older copy of the same Welcome tab with no Help menu row and no Agents window banner, and it opens only when `workbench.startupEditor` or a command asks for it. [cursor only]

### ED-9.2
1. Running Welcome from the Command Palette or the Help menu opens the Welcome page. [stock only]
2. Running `workbench.action.openWalkthrough` with a walkthrough id opens the Welcome tab on that walkthrough's steps page.
3. Running it with a category and a step opens that walkthrough straight to that step.
4. Running it with the `toSide` option opens the walkthrough beside the current editor, and the `inactive` option opens it without taking focus.
5. Running `walkthroughs.selectStep` with a step id selects that step on the open walkthrough page.
6. In Cursor `workbench.action.openWalkthrough` has no title and is not offered in the Command Palette or the Help menu, so it only runs by id or from a link. [cursor only]

### ED-9.3
1. Setting `workbench.startupEditor` chooses which editor shows at startup when none are restored from the last session.
2. Value `none` starts without an editor, and value `welcomePage` opens the Welcome page.
3. Value `readme` opens the README when a folder has one and falls back to the Welcome page otherwise, and it is read only from the user's global settings.
4. Value `newUntitledFile` opens a new untitled text file, only when the window is empty.
5. Value `welcomePageInEmptyWorkbench` opens the Welcome page only when the window has no folder.
6. Value `terminal` opens a new terminal in the editor area.
7. Value `agentSessionsWelcomePage` is a seventh stock value that opens an agent page left out of this catalog. [stock only]
8. Ticking or unticking "Show welcome page on startup" on the Welcome page flips the setting between the Welcome page and none.
9. On the very first launch stock always opens the Welcome page. [stock only]
10. In Cursor the default is `none` and the setting has six values, without `agentSessionsWelcomePage`. [cursor only]

### ED-9.4
1. The Start column is a list of buttons, each with a title and a short description, in the order New File..., Open File... and Open Folder..., Clone Git Repository..., Connect to....
2. On macOS the Open File... and Open Folder... buttons are one button named "Open..." with the description "Open a file or folder to start working".
3. Clicking New File... opens the New File picker (`welcome.showNewFileEntries`).
4. Clicking Open File..., Open Folder... or Open... opens the matching file or folder dialog.
5. Clicking Clone Git Repository... starts a Git clone (`git.clone`).
6. Clicking Connect to... opens the remote menu (`workbench.action.remote.showMenu`).
7. In the web build extra Start entries show for opening a remote repository and for the web start actions.
8. A "Generate New Workspace..." button starts a chat to set up a workspace. [stock only]
9. An "Open a Walkthrough..." entry shows in the Start list only when every walkthrough card is hidden. [cursor only]

### ED-9.5
1. The Recent column lists up to 5 recent folders or workspaces, or up to 10 when there are no walkthrough cards, each row showing a name and its path.
2. Clicking a row opens that folder or workspace in this window.
3. Ctrl click, or Cmd click on macOS, opens it in a new window.
4. A "More..." link below the list opens the Open Recent picker (`workbench.action.openRecent`), and its tooltip reads "Show All Recent Folders" with the key.
5. With no history the column says "You have no recent folders, open a folder to start.", and the "open a folder" part is a link that opens the folder dialog.
6. Each Recent row has a small X, with the tooltip "Remove from Recently Opened", that removes the row from the list. [stock only]

### ED-9.6
1. The Walkthroughs column shows up to 5 cards, ranked by featured, then new, then recently used.
2. Each card shows an icon and a title.
3. Each card has a progress bar with the text "N of M steps complete", or "All N steps complete!" when finished.
4. A card that is new shows a "New" badge, one that has changed shows an "Updated" badge, and a featured card shows a star badge.
5. Clicking a card opens that walkthrough's steps page.
6. Each card has a Hide button that removes it from the column, and the choice is kept in the profile.
7. When every card is hidden the Walkthroughs column disappears.
8. A "More..." link below the cards opens the Open Walkthrough picker.
9. In Cursor the built-in card list is empty, so only cards from installed extensions can show. [cursor only]

### ED-9.7
1. The Open Walkthrough picker is a quick pick titled "Open Walkthrough..." that lists every walkthrough with its description and where it comes from.
2. Picking a row opens that walkthrough on its steps page.
3. Clicking "More..." in the Walkthroughs column opens the picker.
4. Choosing Open Walkthrough... in the Help menu or the Command Palette (`welcome.showAllWalkthroughs`) opens the picker. [stock only]
5. In Cursor the picker opens only from the "More..." link on the Welcome page. [cursor only]

### ED-9.8
1. Opening a walkthrough slides the Welcome tab from the card list to its steps page, with a Go Back button on top.
2. Each step row shows a completion check, a title, a description, and its action links or buttons.
3. The picked step shows its media on the right, which can be an image, an SVG, markdown or a video.
4. The bottom of the page has a "Mark Done" button and a "Next Section" button with an arrow.
5. When a step names a command that has a key, a "Tip: Use keyboard shortcut" line shows with that key.
6. Clicking Go Back, or pressing `Escape` (`welcome.goBack`, when in the Welcome page), returns to the card list.
7. A screen reader announces each step check as "<title>: Completed" or "<title>: Not completed". [stock only]
8. In Cursor the step check is announced with the older wording "Checkbox for Step {0}: Completed". [cursor only]

### ED-9.9
1. Clicking the check beside a step flips that step between done and not done.
2. Running `welcome.markStepComplete` or `welcome.markStepIncomplete` with a step id marks that step done or not done.
3. A step completes on its own when the event it waits for happens, such as a command running, a setting changing or a view opening.
4. When a step completes on its own a screen reader announces "Step {0} completed".
5. The progress bar and the "N of M steps complete" text on the card update as steps change.

### ED-9.10
1. The Setup walkthrough, "Get started with VS Code", has steps for choosing your theme, watching video tutorials, and setting up Copilot, where the Copilot steps hand off to chat. [stock only]
2. The SetupWeb walkthrough, "Get Started with VS Code for the Web", has steps for the theme, the menu bar, extensions, language support, settings sync, the Command Palette, opening code and quick open. [stock only]
3. The SetupAccessibility walkthrough, "Get Started with Accessibility Features", has steps for the accessibility help dialog, the accessible view, aria verbosity, signals, keyboard shortcuts, hover, symbols, folding, IntelliSense, settings and dictation. [stock only]
4. The Beginner walkthrough, "Learn the Fundamentals", has steps for settings, extensions, terminal, debugging, Git, tasks, shortcuts and workspace trust. [stock only]
5. The notebooks walkthrough, "Customize Notebooks", has a step for selecting the layout of your notebooks. [stock only]
6. A Help menu row runs `workbench.action.getStartedWithAccessibilityFeatures` and opens the accessibility walkthrough. [stock only]

### ED-9.11
1. Running "Developer: Reset Welcome Page Walkthrough Progress" (`resetGettingStartedProgress`) from the Command Palette clears the done state of every walkthrough step.
2. After the reset, cards that were hidden with the Hide button come back to the Walkthroughs column.
3. After the reset, every card progress bar shows no completed steps, as if the page were new.

### ED-9.12
1. After an extension that has a walkthrough is installed, its walkthrough opens on its own in the Welcome tab.
2. Extensions add their cards through the `walkthroughs` contribution point, and those cards show in the Walkthroughs column.
3. Setting `workbench.welcomePage.walkthroughs.openOnInstall` turns the open-on-install behavior on or off, and it is a machine-wide setting.
4. In stock the setting defaults to on. [stock only]
5. In Cursor the setting defaults to off, so an installed extension's walkthrough does not open on its own. [cursor only]

### ED-9.13
1. Choosing Interactive Editor Playground from the Help menu or the Command Palette (`workbench.action.showInteractivePlayground`) opens a tab titled "Editor Playground". [stock only]
2. The playground walks through editing features with code samples that you can click into and type in. [stock only]
3. While the playground has focus, `UpArrow` and `DownArrow` scroll it up and down by a small step (`workbench.action.interactivePlayground.arrowUp` and `arrowDown`). [stock only]
4. While the playground has focus, `PageUp` and `PageDown` scroll it up and down by the height of the pane (`workbench.action.interactivePlayground.pageUp` and `pageDown`). [stock only]

### ED-9.14
1. Running `workbench.action.openAgentSessionsWelcome`, or setting `workbench.startupEditor` to `agentSessionsWelcomePage`, opens a second welcome tab. [stock only]
2. The page has Start buttons for Open Recent or Open Folder, New file, and Clone Git Repository. [stock only]
3. The page lists agent sessions, with a "View All Sessions" link below them. [stock only]
4. The page has a walkthrough carousel with Previous and Next buttons. [stock only]
5. The page shows a Copilot notice that can be dismissed. [stock only]
6. The page has the "Show welcome page on startup" checkbox. [stock only]
7. Choosing the `agentSessionsWelcomePage` startup value overrides the secondary side bar visibility settings. [stock only]

### ED-9.15
1. Running `workbench.action.welcomeOnboarding2026` opens a full-window first-run overlay with a list of steps. [stock only]
2. On a first launch the overlay opens by itself in place of the classic walkthrough. [stock only]
3. Setting `workbench.welcomePage.experimentalOnboarding` chooses between this overlay and the classic walkthrough on first launch, and it is on by default. [stock only]
4. The Sign In step offers Continue with GitHub, Google, Apple and GitHub Enterprise buttons, and a "Continue without Signing In" button. [stock only]
5. The Make It Yours step lets you pick a color theme, with a hint that more themes can be installed later. [stock only]
6. The Make It Yours step also lets you pick a keyboard mapping, and picking one is announced as "<name> keyboard mapping selected". [stock only]
7. The Build with AI Agents step is the third step in the list, after Sign In and Make It Yours. [stock only]

### ED-9.16
1. Starting onboarding covers the whole window with an overlay that has back and forward arrow buttons at the top left, and the steps run in order. [cursor only]
2. The Welcome step has Log In and Sign Up buttons. [cursor only]
3. Right-clicking Log In or Sign Up offers to copy a sign-in link. [cursor only]
4. The Welcome step has a "Skip For Now" link. [cursor only]
5. The Plugins step says "Plugins give your agents important context" and shows cards that each have an Add button that changes to Added. [cursor only]
6. The Plugins step has a Continue button, which reads Start Building when it is the last step, and a Skip link. [cursor only]
7. The Data Sharing step shows two choice cards, "Help Improve Cursor" and "Privacy Mode", then a Continue button, which reads Finish when it is the last step, and a Learn More link. [cursor only]
8. The Welcome step is skipped when you are already signed in, and a server flag can skip the Data Sharing step. [cursor only]
9. Starting onboarding also switches the window to the unified agent layout with the side bar on the right. [cursor only]
10. When VS Code settings are found on the machine, a dialog offers to import them. [cursor only]

### ED-9.17
1. Running "Cursor: Start Onboarding" (`cursor.startOnboarding`) from the Command Palette starts the first-run overlay again. [cursor only]
2. Running it also stores a flag that the empty-window landing screen reads. [cursor only]

### ED-9.18
1. When an editor group is empty a list of labeled commands with their key chords shows in the middle of it, in the `editorWatermark.foreground` color.
2. A row whose command has no key chord is left out.
3. Setting `workbench.tips.enabled` turns these rows on or off when no editor is open, and it is on by default.
4. In a folder window stock shows Open Chat, when chat is available, and Show All Commands, then adds Go to File, Find in Files, Start Debugging, Toggle Terminal and Open Settings until at least 3 rows show. [stock only]
5. In an empty window stock also shows Open File, Open Folder and Open Recent. [stock only]
6. Stock shows a faint logo above the rows. [stock only]
7. Stock has a toolbar slot above the rows (`EditorGroupWatermarkToolbar`). [stock only]
8. Cursor shows its own rows: Search Files, Open Browser, New Agent, Show or Hide Terminal, Show Files, Add Folder or Add Repository, and Maximize Chat. [cursor only]
9. Clicking a Cursor row runs its command. [cursor only]
10. In Cursor a window with no folder does not show these rows, because the landing screen takes over. [cursor only]

### ED-9.19
1. When no folder or workspace is open and no editor is open, Cursor draws its own landing screen in the editor area instead of the watermark. [cursor only]
2. The landing screen holds a group of buttons, a recent projects list, a plan badge, and, when experiments are on, cloud and agent prompts. [cursor only]
3. Right after onboarding finishes the first landing screen uses a different layout. [cursor only]
4. The landing screen goes away when a folder or an editor opens. [cursor only]

### ED-9.20
1. The landing screen shows a grid of five buttons: New project, Open project, Clone repo, Connect via SSH and New Window. [cursor only]
2. Clicking New project starts a new project (`workbench.action.files.newProject`). [cursor only]
3. Clicking Open project opens the folder dialog (`workbench.action.files.openFolder`). [cursor only]
4. Clicking Clone repo runs the Git clone command (`git.clone`). [cursor only]
5. When Git is missing, Clone repo shows the warning "Clone repo needs Git. Install Git, or enable the built-in Git extension, then restart Cursor." with a Download Git action. [cursor only]
6. Clicking Connect via SSH looks for a Remote SSH extension, and when one is installed but off it asks to enable it with an "Enable and Reload" action. [cursor only]
7. When no Remote SSH extension is installed, Connect via SSH opens the Extensions view with the search `@recommended:remotes`. [cursor only]
8. Clicking New Window opens a new Cursor window (`workbench.action.newWindow`). [cursor only]

### ED-9.21
1. A "Recent projects" list on the landing screen shows the first 5 recent entries. [cursor only]
2. Clicking an entry opens that project in this window. [cursor only]
3. Ctrl click, or Cmd click on macOS, opens it in a new window. [cursor only]
4. Right-clicking an entry offers "Remove from recents". [cursor only]
5. When there are more than 5 entries a "View all (n)" link shows below the list, where n is the total number of recent projects. [cursor only]

### ED-9.22
1. A small plan label sits near the top of the landing screen. [cursor only]
2. For paid plans the label shows the plan name, one of Pro, Pro+, Ultra, Team or Enterprise, with a Settings link. [cursor only]
3. For free, trial and Pro-trial users the label reads "Free Plan", "Pro Trial" or "Pro+ Trial" with an Upgrade link. [cursor only]
4. For signed-out users the label shows a Sign in link. [cursor only]
5. Clicking the Settings link opens the General tab of Cursor Settings. [cursor only]

### ED-9.23
1. When its experiment is on, a "Connect GitHub" or "Connect Your Repos" button under the landing screen buttons opens a connect modal. [cursor only]
2. When its experiment is on, a "Steer from iOS" button opens its own modal. [cursor only]
3. When its experiment is on, a "Try a new window for running parallel agents" link shows under the buttons. [cursor only]
4. When none of these experiments are on the landing screen shows none of these prompts. [cursor only]

### ED-9.24
1. In an empty untitled plain text file a one-line italic hint sits on line 1, and it goes away when you start typing.
2. The stock hint reads "Select a language (key) to get started. Start typing to dismiss or don't show this again.", where the language part is a link that opens the language picker (`workbench.action.editor.changeLanguageMode`). [stock only]
3. The "don't show this again" link in the hint turns the hint off. [stock only]
4. When inline chat exists the stock hint also has a "Generate code" link that starts it (`inlineChat.start`). [stock only]
5. Right-clicking the hint offers "Disable Empty Editor Hint".
6. Setting `workbench.editor.empty.hint` chooses whether the hint shows, with values `text` (default) and `hidden`.
7. Setting `accessibility.verbosity.emptyEditorHint` chooses whether the hint's screen reader label adds a line saying to toggle that setting to disable the hint.
8. In Cursor the hint reads "Press (key) to generate code. Start typing to dismiss.", and its screen reader label offers selecting a language, filling with a template, or opening a different editor. [cursor only]

### ED-9.25
1. The New File picker is a quick pick titled "New File..." with the placeholder "Select File Type or Enter File Name...". [stock only]
2. The picker lists Text File, then notebook types and custom editor types, in groups. [stock only]
3. Each row has a button for configuring its keybinding. [stock only]
4. Typing a name adds a row "Create New File (name)" that creates a file with that name. [stock only]
5. The File menu row New File... opens the picker (`welcome.showNewFileEntries`). [stock only]
6. The key chord `Alt+CtrlCmd+WinCtrl+N` opens the picker. [stock only]
7. The Start list on the Welcome page and the empty text editor hint also open the picker. [stock only]

### ED-9.26
1. In stock, Help, Show Release Notes opens a tab titled "Release Notes: {version}" that fetches the notes online and shows them as a web page. [stock only]
2. Key names in the release notes text show your real key chords. [stock only]
3. Setting links on the page can be changed in place with a small popup. [stock only]
4. The release notes page has a find box. [stock only]
5. A "Show release notes after an update" checkbox sits on the page, and setting `update.showReleaseNotes` controls the same choice. [stock only]
6. A Developer command (`update.showCurrentReleaseNotesFromCurrentFile`) opens the current file as release notes. [stock only]
7. In Cursor, Help, Show Release Notes (`update.showCurrentReleaseNotes`) only opens the product's release notes web page, with no release notes tab and no checkbox. [cursor only]

## Unclear
- ED-9.16 is a window-level overlay, not an editor tab, so a tracker may want it in a window-level group. It is kept here to match the feature list.
- ED-9.2 piece 6, ED-9.7 piece 5 and ED-9.24 piece 8 describe Cursor behavior that pass 2 found only from strings, and `welcome.showNewFileEntries` and `welcome.showAllWalkthroughs` have no registration found in Cursor, so those entry points may exist by a path the searches missed.
- ED-9.19 piece 3 says the layout changes right after onboarding, but what changes was not read, so that piece is not precisely checkable until it is looked at.
- ED-9.21 piece 5 says "View all (n)" opens the full recent list, and the command behind it is a bundle constant that was not resolved.
- ED-9.23 pieces say "When its experiment is on" because the prompts are gated by the experiments `ide_connect_git_repos_start` and `ide_steer_from_cloud_start`, and the contents of the modals were not read.
- ED-9.22 does not say where Upgrade and Sign in lead, because pass 2 only recorded the call to action names.
- ED-9.18 has 10 pieces and describes both the stock rows and the Cursor rows, so it may be better split into a stock watermark feature and a Cursor watermark feature.
- ED-9.3 has 10 pieces because each startup value is its own visible behavior, and it may be better split into one feature for the setting and one for the checkbox.
- The Accessible view of the Welcome page and the telemetry events the landing screen records were left out as not being something a person sees on the page.
