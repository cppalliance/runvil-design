Group: ED-8 Settings, shortcuts and configuration editors. Pass: 3. Date: 2026-10-25.

### ED-8.1
1. Running Open Settings (`workbench.action.openSettings`) opens the Settings editor as an editor tab on the User scope.
2. Choosing Settings in the File > Preferences menu or in the Manage gear menu opens the Settings editor.
3. Pressing `Ctrl+,` opens the Settings editor. [stock only]
4. The Command Palette entries Open Settings (UI) and Open User Settings (`workbench.action.openSettings2`, `workbench.action.openGlobalSettings`) open the Settings editor on the User scope.
5. Open Accessibility Settings (`workbench.action.openAccessibilitySettings`) opens the Settings editor with the search box set to `@tag:accessibility`.
6. Setting `workbench.settings.editor` chooses which editor Open Settings uses, with values `ui` (the Settings editor, the default) and `json` (the settings JSON file).
7. In Cursor, `Ctrl+,` opens Cursor Settings outside the Agents window, and the Settings editor opens from the "VS Code Settings" row in File > Preferences, from `Ctrl+Shift+,` or from Open Settings (UI). [cursor only]

### ED-8.2
1. Typing in the search box at the top of the Settings editor narrows the list to matching settings as the person types, and `Ctrl+F` puts focus in the box (`settings.action.search`).
2. Typing `@` in the search box, or `:` after an `@` filter word, opens a suggestion list of filters, and `@lang:` and `@ext:` list language ids and installed extension ids.
3. The Filter Settings button in the search box opens a drop-down with Modified, Extension ID..., Feature..., Tag..., Language... and Online services rows that add or remove the matching filter text in the box (`settings.action.suggestFilters`).
4. The Filter Settings drop-down also has Setting ID..., Organization policies, Stable, Preview, Experimental and Advanced rows, and Stable, Preview and Experimental exclude each other. [stock only]
5. A result line shows "N Settings Found", "1 Setting Found" or "No Settings Found", and when nothing matches the list is hidden and a Clear Filters link shows.
6. The clear button in the search box, the Clear Filters link and `Escape` in the box (`settings.action.clearSearchResults`) clear the search text and filters.
7. Pressing Up in the search box recalls the previous search (`settings.action.showPreviousSearch`), and pressing Down steps forward again while browsing history.
8. After a search, a toggle in the search bar reads "Show AI-recommended results" when AI results exist, switches them on and off with `Ctrl+I` (`settings.action.toggleAiSearch`), and reads "No AI results available at this time..." when there are none. [stock only]
9. Setting `workbench.settings.showAISearchToggle` chooses whether the AI results toggle is shown, on or off. [stock only]
10. Setting `workbench.settings.enableNaturalLanguageSearch` chooses whether the natural language search mode is used, on or off, default on.

### ED-8.3
1. A Settings Switcher bar under the search box has a User tab that shows and edits the person's own settings.
2. A Workspace tab, labeled with the workspace name, shows and edits the workspace settings, and it is disabled when no folder or workspace is open.
3. In a remote window a Remote tab labeled with the host name shows the settings of the remote machine.
4. In a multi-root workspace a Folder drop-down appears and lists each folder to switch to.
5. Choosing a scope tab reloads the list with that scope's values, and changes made in the list are written to that scope.
6. Open Workspace Settings, Open Folder Settings and Open Remote Settings in the Command Palette (`workbench.action.openWorkspaceSettings`, `workbench.action.openFolderSettings`, `workbench.action.openRemoteSettings`) open the Settings editor already on that scope.
7. Right-clicking a root folder in the Explorer and choosing Open Folder Settings (`_workbench.action.openFolderSettings`) opens the Settings editor on that folder's scope.

### ED-8.4
1. A contents tree at the left of the Settings editor lists setting groups, among them Commonly Used, Text Editor, Workbench, Window, Chat, Features, Application, Security and one group per extension.
2. Groups with subgroups start collapsed and can be expanded, and groups without subgroups show no twistie.
3. Clicking a group scrolls the settings list to that group.
4. As the settings list scrolls, the tree follows and marks the group in view.
5. Setting `workbench.settings.settingsSearchTocBehavior` chooses what the tree does while searching, with values `filter` (the default; show only groups that have matches, and clicking one filters the results to it) and `hide` (hide the tree during a search).
6. Pressing Left on a setting row moves focus to the tree (`settings.action.focusTOC`), and Enter on a tree row moves focus into the settings list (`settings.action.focusSettingsList`).
7. Pressing Escape moves focus up one level toward the tree (`settings.action.focusLevelUp`).

### ED-8.5
1. Each row in the Settings list shows the setting's title, its description and one control that matches the type of the setting.
2. A boolean setting shows a checkbox, and a setting with fixed choices shows a drop-down of those choices.
3. A number setting shows a number box, a string setting shows a text box, and a multi-line string shows a multi-line text box.
4. A list setting shows its entries with an Add Item button, an object setting shows a key and value table, and an exclude or include pattern setting shows its patterns with an Add Pattern button.
5. A setting whose value is too complex for a widget shows an "Edit in settings.json" link that opens the settings file at that setting.
6. A gear icon on each row opens a menu whose Reset Setting row (`settings.resetSetting`) puts the setting back to its default value.
7. The gear menu rows Copy Setting ID, Copy Setting as JSON and Copy Setting as URL put the setting id, a JSON snippet or a settings link on the clipboard (`settings.copySettingId`, `settings.copySettingAsJSON`, `settings.copySettingAsURL`).
8. The gear menu row Apply Setting to all Profiles (`settings.applyToAllProfiles`) makes the setting keep one value across profiles, and it shows only on the User scope for settings that are not application scope.
9. Pressing Enter on a row focuses its control (`settings.action.focusSettingControl`), and `Shift+F9` opens the row's gear menu (`settings.action.showContextMenu`).
10. Setting `workbench.settings.alwaysShowAdvancedSettings` chooses whether advanced settings show without the `@tag:advanced` filter, on or off, default on only in builds that are not stable. [stock only]

### ED-8.6
1. Small labels beside a setting title show where its value comes from, and hovering a label shows a text that explains it.
2. A setting that also has a value in another scope shows "Modified in" or "Also modified in" followed by the scope name.
3. A setting whose default was changed by an extension or a language shows "Default value changed", and the hover names the source.
4. A setting that keeps one value across profiles shows "Applies to all profiles", and the hover says it retains its value when switching profiles.
5. A setting that cannot apply in an untrusted workspace shows "Requires workspace trust" with a shield icon, and the hover has a Manage Workspace Trust link that opens the Workspace Trust editor.
6. A setting held by an organization policy shows "Managed by organization" with a briefcase icon, and the hover says its value cannot be changed.
7. Settings tagged Preview or Experimental show a Preview or Experimental label, and settings tagged Advanced show an Advanced label.
8. A setting that Settings Sync ignores shows a "Not synced" label whose hover says the setting is ignored during sync. [stock only]

### ED-8.7
1. Running Configure Language Specific Settings... from the Command Palette (`workbench.action.configureLanguageBasedSettings`) opens a "Select Language" picker that lists every registered language by name with its id in brackets.
2. Choosing a language opens the settings JSON file at that language's override block.
3. Typing `@lang:<id>` in the Settings editor search box, or choosing Language... in the Filter Settings drop-down, shows that language's values.
4. While a language filter is active the scope tabs gain a language suffix, and changes made in the list are written as that language's overrides.

### ED-8.8
1. In the Settings editor, a title bar button Open Settings (JSON) (`settings.switchToJSON`) opens the user settings JSON file.
2. In the Keyboard Shortcuts editor, a title bar button Open Keyboard Shortcuts (JSON) opens `keybindings.json`.
3. In a user settings JSON file, a title bar button Open Settings (UI) (`_workbench.openUserSettingsEditor`) goes back to the Settings editor.
4. In `keybindings.json`, a title bar button Open Keyboard Shortcuts goes back to the Keyboard Shortcuts editor.
5. Command Palette entries open the user, application, workspace, folder and remote settings JSON files, and the application entry is listed only while a profile other than the default one is active.
6. Open Default Settings (JSON) and Open Default Keyboard Shortcuts (JSON) from the Command Palette open the read-only default files.
7. In the user `keybindings.json`, Define Keybinding in the editor content menu and `Ctrl+K Ctrl+K` open the key-capture box (`editor.action.defineKeybinding`).
8. Settings `workbench.settings.openDefaultSettings`, `workbench.settings.useSplitJSON` and `workbench.settings.openDefaultKeybindings` each turn on opening the read-only defaults beside the JSON file, on or off, default off.
9. The same JSON toggle buttons also show in the modal editor title menu. [stock only]
### ED-8.9
1. Pressing `Ctrl+K Ctrl+S`, or choosing Keyboard Shortcuts in File > Preferences, the Manage gear menu or the Command Palette, opens the Keyboard Shortcuts editor (`workbench.action.openGlobalKeybindings`).
2. The editor lists every command in a table with the columns Command, Keybinding, When and Source.
3. Typing plain words in the search box, which shows "Type to search in keybindings", narrows the table to rows that match.
4. The filters `@command:`, `@keybinding:`, `@source:` (with values user, default, system and extension) and `@ext:` narrow the table by that field.
5. Three rows in the editor title menu, Show System Keybindings, Show Extension Keybindings and Show User Keybindings, fill the search box with the matching `@source:` filter.
6. A sort button, Sort by Precedence (Highest first), flips the table between alphabetical order and precedence order, and `Alt+P` does the same.
7. The search box keeps a history of earlier searches that can be recalled with the history keys, and Clear Keyboard Shortcuts Search History empties it (`keybindings.editor.clearSearchHistory`).
8. The clear button in the search box and `Escape` while the box has text clear the search (`keybindings.editor.clearSearchResults`).
9. `Ctrl+F` puts focus in the search box, and `Ctrl+Down` moves focus from the search box to the table.
10. The Cursor Settings General tab has a Keyboard Shortcuts row whose Open button opens this editor. [cursor only]

### ED-8.10
1. A Record Keys button inside the search box (`keybindings.editor.recordSearchKeys`) switches the box into key capture and shows a "Recording Keys" badge.
2. While recording, the box placeholder reads "Recording Keys. Press Escape to exit".
3. Pressing a key combination while recording fills the box with that combination and the table lists the commands bound to it.
4. Pressing a second combination records a chord of two, and a third press starts the recording over.
5. `Alt+K` (`Cmd+Alt+K` on macOS) with focus in the search box turns recording on or off, and `Escape` leaves recording.

### ED-8.11
1. Pressing Enter on a row or double-clicking it (`keybindings.editor.defineKeybinding`) opens a key-capture box that says "Press desired key combination and then press ENTER."
2. The box shows the pressed combination as key labels, and a second combination is shown after "chord to".
3. When other commands already use the combination, the box says "1 existing command has this keybinding" or "N existing commands have this keybinding".
4. Enter saves the combination to the row, Escape clears the box and a second Escape closes it, and clicking away cancels without saving.
5. A pencil icon on a row that has a key and a plus icon on a row that has none do the same as Enter.
6. Right-clicking a row opens a menu with Change Keybinding... (or Add Keybinding... on a row with no key, with Add Keybinding... also shown next to Change on a row with a key), Remove Keybinding, Reset Keybinding, Copy, Copy Command ID and Copy Command Title.
7. Remove Keybinding (`Delete`) takes the key off a row that has one, and Reset Keybinding restores the default key and is disabled on a row that is already the default.
8. The Copy rows put the row (`Ctrl+C`), the command id or the command title on the clipboard.
9. `Ctrl+K Ctrl+A` adds a keybinding for the focused row (`keybindings.editor.addKeybinding`).
10. When saving fails, a notification says "Error '...' while editing the keybinding. Please open 'keybindings.json' file and check for errors."

### ED-8.12
1. Change When Expression in the row menu (`keybindings.editor.defineWhenExpression`, `Ctrl+K Ctrl+E`) turns the When cell of that row into a text box, and the row is disabled when the command has no key.
2. The text box suggests context keys as the person types.
3. Enter, with no suggestion open, saves the new condition (`keybindings.editor.acceptWhenExpression`).
4. Escape drops the edit and the cell shows its old text again (`keybindings.editor.rejectWhenExpression`).

### ED-8.13
1. Show Same Keybindings in the row menu (`keybindings.editor.showConflicts`) puts the row's key combination, in quotes, in the search box.
2. The table then lists every command that has that combination.
3. The Show Same Keybindings row is disabled when the command has no key.
4. The key-capture box says "1 existing command has this keybinding" or "N existing commands have this keybinding" for the combination being typed.
5. Clicking that message fills the search box with the typed combination, so the commands that share it are listed.

### ED-8.14
1. Open Profiles (UI) (`workbench.profiles.actions.manageProfiles`) opens a Profiles editor tab, from the Manage gear menu, the File > Preferences Profiles submenu or the Command Palette.
2. A list on the left shows each profile, with a Built-in label on the default profile and an Active mark on the one in use.
3. A New Profile button starts a new profile in the right pane, and its arrow opens a From Template submenu.
4. An Import Profile row offers to import a profile from a file or from a URL.
5. The right side of the editor shows the chosen profile's Name and Icon, with Use for Current Window and Use for New Windows options.
6. A Copy from picker sets which profile or template a new profile starts from.
7. A Contents list shows Settings, Keyboard Shortcuts, Snippets, Tasks, MCP Servers and Extensions, each set to Default, None or Copy source.
8. A Folders & Workspaces list shows the folders that use the profile, with an Add Folder button.
9. Create and Cancel buttons finish or abandon a new profile, and Export and Delete buttons act on an existing profile.
10. The Command Palette has commands to switch profile, create a profile from the current one, export, delete, open a new window with a profile and create a temporary profile (`workbench.profiles.actions.switchProfile`, `workbench.profiles.actions.createFromCurrentProfile`, `workbench.profiles.actions.exportProfile`, `workbench.profiles.actions.deleteProfile`, `workbench.profiles.actions.newWindowWithProfile`, `workbench.profiles.actions.createTemporaryProfile`).

### ED-8.15
1. Color Theme (`workbench.action.selectTheme`, `Ctrl+K Ctrl+T`, File > Preferences > Themes, the Manage gear menu or the Command Palette) opens a picker of installed color themes under light, dark and high contrast headings.
2. Moving the highlight with the arrow keys previews each theme on the window, Enter keeps the highlighted theme, and closing the picker any other way returns to the earlier theme.
3. A button on the picker shows whether the system color mode is followed ("Detect system color mode enabled. Click to configure.") and opens the related settings.
4. A row in the picker offers to browse or install more themes.
5. Toggle between Light/Dark Themes (`workbench.action.toggleLightDarkThemes`) switches between the preferred light and preferred dark theme.
6. Generate Color Theme From Current Settings (`workbench.action.generateColorTheme`) builds a theme from the current settings.
7. Settings `workbench.colorTheme`, `workbench.preferredDarkColorTheme`, `workbench.preferredLightColorTheme`, `workbench.preferredHighContrastColorTheme` and `workbench.preferredHighContrastLightColorTheme` each hold a theme name, the active theme and the theme used for each system mode.
8. Setting `window.autoDetectColorScheme` (default off) chooses whether the theme follows the system light or dark mode, and `window.autoDetectHighContrast` (default on) chooses whether it follows a high contrast operating system theme.
9. Cursor adds its own built-in themes: Cursor Dark Midnight, Cursor Dark High Contrast, Cursor Dark, Cursor Light, Cursor Light Colorblind, Cursor Dark Colorblind and Cursor Light High Contrast. [cursor only]
10. Try New Default Themes (`workbench.action.tryNewDefaultThemes`) is offered in the Command Palette. [stock only]

### ED-8.16
1. File Icon Theme (File > Preferences > Themes, the Manage gear menu or the Command Palette, `workbench.action.selectIconTheme`) opens a picker of the installed file icon themes.
2. A None row, "Disable File Icons", turns file icons off.
3. Moving the highlight with the arrow keys previews each file icon theme, Enter keeps it, and closing the picker any other way returns to the earlier theme.
4. A row in the picker offers to browse or install more icon themes.
5. Setting `workbench.iconTheme` holds the active file icon theme, and a value of null shows no file icons.

### ED-8.17
1. Product Icon Theme (File > Preferences > Themes, the Manage gear menu or the Command Palette, `workbench.action.selectProductIconTheme`) opens a picker of the installed icon sets for the workbench, with Default and any others.
2. Moving the highlight with the arrow keys previews each icon set, Enter keeps it, and closing the picker any other way returns to the earlier set.
3. A row in the picker offers to browse or install more icon sets.
4. Setting `workbench.productIconTheme` holds the active product icon theme.

### ED-8.18
1. Manage Workspace Trust (`workbench.trust.manage`) opens a Workspace Trust editor tab whose header reads "You are in Restricted Mode" or, when trusted, "You trust this window", "You trust this folder" or "You trust this workspace".
2. Trust and Don't Trust buttons in the header change the trust state of the window.
3. A list under the header says what is on or off in Restricted Mode (tasks, debugging, workspace settings, extensions), with links to the affected settings and extensions.
4. A Trusted Folders & Workspaces table with Host and Path columns lists the trusted places, with an Add Folder button and edit and delete icons on each row.
5. Configure Workspace Trust Settings (`workbench.trust.configure`) opens the Settings editor filtered to the Workspace Trust settings.
6. Setting `security.workspace.trust.enabled` turns Workspace Trust on or off, and in Cursor it is off by default, so Manage Workspace Trust and Configure Workspace Trust Settings are unavailable until it is on. [cursor only]
7. Setting `security.workspace.trust.emptyWindow` chooses whether an empty window is trusted by default, on or off, default on.
8. Setting `security.workspace.trust.startupPrompt` chooses when the startup trust prompt shows, with values `always`, `once` and `never` (the default).
9. Setting `security.workspace.trust.banner` chooses when the Restricted Mode banner shows, with values `always`, `untilDismissed` (the default) and `never`.
10. Setting `security.workspace.trust.untrustedFiles` chooses how untrusted files opened in a trusted workspace are handled, with values `prompt` (the default), `open` and `newWindow`.
### ED-8.19
1. Pressing `Ctrl+Shift+J` (`aiSettings.action.open`) opens Cursor Settings as a pinned editor tab in the last editor group, and adds a group on the right when there is none. [cursor only]
2. Pressing `Ctrl+,` outside the Agents window, choosing Cursor Settings in File > Preferences, or running the open command from the Command Palette also opens Cursor Settings. [cursor only]
3. A sidebar on the left lists the tabs, with an arrow icon on rows that launch another page and thin dividers between tab groups. [cursor only]
4. A count badge shows on the Models row of the sidebar. [cursor only]
5. A Docs link sits at the foot of the sidebar. [cursor only]
6. Command Palette entries named "Cursor Settings: <tab name>" open Cursor Settings on one tab each, for the tab ids `general`, `chat`, `background-composer`, `models`, `git-prs`, `rules`, `mcp`, `hooks`, `browser` and `beta` (`aiSettings.action.open.<tab>`). [cursor only]
7. The open command accepts a tab id and an item, and opens that tab scrolled to the item. [cursor only]

### ED-8.20
1. General is the first tab that opens in Cursor Settings. [cursor only]
2. A signed-out person sees a Sign In row, and a signed-in person sees a Cursor Account row that leads to account and billing management. [cursor only]
3. A Preferences group has an Editor Settings row whose Open button opens User Settings (`workbench.action.openGlobalSettings`). [cursor only]
4. The Preferences group has a Keyboard Shortcuts row whose Open button opens the Keyboard Shortcuts editor (`workbench.action.openGlobalKeybindings`). [cursor only]
5. The Preferences group also has the rows Import Settings from VS Code and Reset "Don't Ask Again" Dialogs. [cursor only]
6. The tab lists Layout, Startup, Notifications, Status Bar and Tips rows. [cursor only]
7. A menu bar icon row shows on macOS, and a system tray icon row shows on other systems. [cursor only]
8. A Verify Extension Signatures row shows only when a feature gate is on. [cursor only]

### ED-8.21
1. A VS Code Settings row with an arrow icon in the Cursor Settings sidebar closes Cursor Settings and opens the Settings editor. [cursor only]
2. The row shows in the IDE window because the Appearance tab is off there. [cursor only]
3. Outside the Agents window, a "VS Code Settings" row in File > Preferences takes the place of the stock Settings row and opens the Settings editor. [cursor only]
4. `Ctrl+Shift+,` (`cursor.openVSCodeSettingsFromMenu`) opens the Settings editor outside the Agents window. [cursor only]

### ED-8.22
1. The Plan & Usage tab shows a Personal Usage section with an included-usage meter and an on-demand meter. [cursor only]
2. The tab shows a credits card. [cursor only]
3. A Monthly Limit control sets a fixed amount or unlimited on-demand use. [cursor only]
4. The tab is drawn only when the person is signed in and the account has on-demand spillover. [cursor only]
5. A usage bar widget opens Cursor Settings on this tab (`aiSettings.action.open` with the tab `plan-usage`). [cursor only]

### ED-8.23
1. The tab titled Agents (id `chat`) opens from the Command Palette entry "Cursor Settings: Agents" (`aiSettings.action.open.chat`) and shows a long page of toggles and pickers. [cursor only]
2. Its rows are grouped under Run Mode, Execution and Approvals, Context and Tools, Terminal and Editing, and Conversation and Inline Diffs. [cursor only]
3. Rows include a read allowlist, web search and web fetch switches, queue messages, text size, auto format, third-party configs and voice submit keywords. [cursor only]
4. A few rows show only in the IDE window and a few only in the Agents window, so the list differs by window. [cursor only]

### ED-8.24
1. The tab titled Cloud Agents (id `background-composer`, `aiSettings.action.open.background-composer`) shows an Integrations list. [cursor only]
2. Each integration has a Connect button, or a Reconnect button when it is already connected. [cursor only]
3. While the tab loads it shows a loading ring with "Loading Cloud Agents settings...". [cursor only]
4. With Privacy Mode on, the tab shows a "Cloud Agents Unavailable" card that says "Cloud Agents require data storage to function." with an Open Privacy Settings button. [cursor only]
5. Without an open folder that has a Git repository, the tab shows a "Get Started" card that says "Cloud Agents require a Git repository in an open folder." with an Open Folder button. [cursor only]

### ED-8.25
1. The tab titled Models (`aiSettings.action.open.models`) shows a model list with an Add Custom Model button. [cursor only]
2. A collapsible API Keys section can be opened and closed. [cursor only]
3. An Override OpenAI Base URL switch comes with a Base URL box. [cursor only]
4. The tab has a Task Models entry. [cursor only]
5. An Explore Subagent Model row has a picker for the model. [cursor only]
6. In local mode a Gateway entry shows. [cursor only]
7. A count badge shows on the sidebar row while a Claude or OpenAI key warning is open. [cursor only]

### ED-8.26
1. The tab titled Voice shows rows for the default Voice, Language and Speed. [cursor only]
2. A Submit Keywords row shows only when a realtime-voice condition holds. [cursor only]
3. The tab is drawn only when the voice feature gate is on. [cursor only]

### ED-8.27
1. The tab titled Source Control (id `git-prs`, `aiSettings.action.open.git-prs`) has an Attribution group with Commit Attribution and PR Attribution rows. [cursor only]
2. A Branches group has a Branch Prefix row. [cursor only]
3. A Pull Requests group has PR Link Destination and Review Provider rows, and it shows only when the person is signed in. [cursor only]
4. A Cursor-Managed Worktrees group has Worktree Cleanup, Max Worktrees and Max Total Size in GB rows. [cursor only]
5. The hidden worktrees tab id opens this tab. [cursor only]

### ED-8.28
1. A Customize row with an arrow icon shows in the Cursor Settings sidebar in the IDE window and has no page of its own. [cursor only]
2. Clicking the row closes Cursor Settings and opens the Customize editor. [cursor only]

### ED-8.29
1. The tab titled Browser & Network (id `browser`, `aiSettings.action.open.browser`) has an HTTP Compatibility Mode picker meant for proxy and VPN setups. [cursor only]
2. A Required Domains list shows the domains a firewall must allow, with a Copy Domains button. [cursor only]
3. A Network Diagnostics row with the text "Check network connectivity to all Cursor services" has a button that runs the check. [cursor only]
4. The hidden network tab id opens this tab. [cursor only]

### ED-8.30
1. The tab titled Code Intelligence (id `indexing`) has a Codebase group with Ignore Files in .cursorignore, Hierarchical Cursor Ignore and Ignore Symlinks in Cursor Ignore Search rows. [cursor only]
2. A Cursor Tab group has an on or off row. [cursor only]
3. An LSP group has Enable LSPs, Maximum Local and Remote LSP Workspaces and Enable LSPs for Worktrees rows, which are marked for the Agents window. [cursor only]
4. The tab is drawn in the IDE window by default, and the hidden Tab tab id opens it. [cursor only]

### ED-8.31
1. The Command Palette entry "Cursor Settings: Beta" (`aiSettings.action.open.beta`) opens Cursor Settings on the tab titled Beta. [cursor only]
2. The Beta tab has an Update Access row with a picker. [cursor only]
3. The Beta tab has a Dedicated Agent Runtime switch that turns the option on or off, and its text says a reload is needed. [cursor only]

### ED-8.32
1. The tab titled Developer has Tracing, Local Trace Mode and Continuous RPC File Recording rows. [cursor only]
2. The tab is drawn only for builds that are not packaged, for extension development, or for people flagged as developers. [cursor only]
3. The tab opens through the open command with the tab `developer`. [cursor only]
### ED-8.33
1. Running Open Customize (`workbench.action.openCustomizeEditor`) from the Command Palette opens the Customize editor as a pinned tab. [cursor only]
2. Running it while the editor is already open shows that open tab instead of making a second one. [cursor only]
3. The Customize tab comes back after a window reload. [cursor only]
4. A tab strip holds seven tabs: Plugins, MCPs, Skills, Subagents, Rules, Commands and Hooks. [cursor only]
5. Each tab has a heading and a one-line description with a docs link. [cursor only]
6. Each tab has a search box, and Focus Customize Search (`workbench.action.customize.focusSearch`) puts focus in it. [cursor only]
7. Each tab has a scope menu. [cursor only]
8. Each tab shows its items in groups for User, Team and Workspace, with sources such as plugin, dashboard and project. [cursor only]
9. A tab with no items shows an empty state. [cursor only]
10. The Cursor Settings Customize row and the Cursor Settings tab ids `plugins`, `rules`, `mcp` and `hooks` open this editor. [cursor only]

### ED-8.34
1. Open Plugins (`workbench.action.customize.openPlugins`) opens the Customize editor on the Plugins tab, headed "Extend Cursor with Plugins". [cursor only]
2. The description says plugins bundle rules, skills, subagents, commands, MCP servers and hooks into one installable package, and it links to the plugins docs. [cursor only]
3. The Cursor Settings tab id `plugins` opens this tab. [cursor only]

### ED-8.35
1. Open MCPs (`workbench.action.customize.openMCPs`) opens the Customize editor on the MCPs tab, headed "Connect External Tools with MCP". [cursor only]
2. The description says Model Context Protocol servers connect Cursor to tools such as Linear, Figma and Notion, and it has a docs link. [cursor only]
3. The tab has its own list view with a target filter for user, workspace or cloud, and cloud is the default. [cursor only]
4. The Cursor Settings tab id `mcp` (`aiSettings.action.open.mcp`) opens this tab, and its `mcpTarget` argument picks the target. [cursor only]

### ED-8.36
1. Open Skills (`workbench.action.customize.openSkills`) opens the Customize editor on the Skills tab, headed "Teach Cursor New Skills". [cursor only]
2. The description says skills package domain knowledge and workflows, and it has a docs link. [cursor only]
3. A Skill Publishing entry sits in the editor title menu of the Customize editor. [cursor only]
4. Open Skill Publishing Logs (`workbench.action.customize.openSkillPublishLogs`) opens the Skill Publishing log as an output channel. [cursor only]

### ED-8.37
1. Open Subagents (`workbench.action.customize.openSubagents`) opens the Customize editor on the Subagents tab (id `agents`), headed "Delegate Work to Subagents". [cursor only]
2. The description says subagents are specialized assistants that run in their own context window, and it has a docs link. [cursor only]

### ED-8.38
1. Open Rules (`workbench.action.customize.openRules`) opens the Customize editor on the Rules tab, headed "Guide Agent with Rules". [cursor only]
2. The description says rules give the agent persistent, system-level instructions, and it has a docs link. [cursor only]
3. The Cursor Settings tab id `rules`, titled "Rules, Skills, Subagents" (`aiSettings.action.open.rules`), opens this tab. [cursor only]
4. When the open request names a scroll target for Commands or Skills, the Commands tab or the Skills tab opens instead. [cursor only]

### ED-8.39
1. Open Commands (`workbench.action.customize.openCommands`) opens the Customize editor on the Commands tab, headed "Create Reusable Commands". [cursor only]
2. The description says commands are reusable prompts invoked with /name, and it has a docs link. [cursor only]
3. Opening a global command from the list shows the Global Command Preview. [cursor only]

### ED-8.40
1. Open Hooks (`workbench.action.customize.openHooks`) opens the Customize editor on the Hooks tab, headed "Automate with Hooks". [cursor only]
2. The description says hooks run custom scripts at lifecycle events, and it has a docs link. [cursor only]
3. The tab has its own view for hooks. [cursor only]
4. The Cursor Settings tab id `hooks` (`aiSettings.action.open.hooks`) opens this tab. [cursor only]
5. A first-time team hooks notice has a View Hooks button that opens this tab. [cursor only]

### ED-8.41
1. Choosing Modified in the Filter Settings drop-down puts `@modified` in the search box and lists only the settings the person has changed.
2. The Modified row shows a check while the filter is on.
3. Choosing the row again, or clearing the box, removes the filter.
4. The list shows the settings changed in the scope that is selected in the Settings Switcher.
5. Typing `@modified` in the search box gives the same result as choosing the row.

### ED-8.42
1. Choosing Online Services Settings in File > Preferences opens the Settings editor with `@tag:usesOnlineServices` in the search box (`settings.filterByOnline`).
2. Running `settings.filterByTelemetry` opens the Settings editor with `@tag:telemetry` in the search box, and no menu row runs it.
3. Running `settings.filterUntrusted` ("Show untrusted workspace settings") lists the Workspace scope settings that need Workspace Trust.
4. In stock, running the online or telemetry command while a Settings editor is open types the query into that editor's search box instead of opening a new one. [stock only]
5. In Cursor, the online and telemetry commands always open through the preferences service, and a link in a Cursor text block that points to cursor.com/privacy runs `settings.filterByTelemetry`. [cursor only]

### ED-8.43
1. Pressing Down in the search box, with no suggestion list open, moves focus to the first setting (`settings.action.focusSettingsFile`, `settings.action.focusSettingsFromSearch`).
2. Pressing Enter in the search box, with no suggestion list open, moves focus to the first setting (`settings.action.focusSettingsFromSearchOnEnter`). [stock only]
3. Pressing Up on the first setting returns focus to the search box (`settings.action.focusSearchFromSettings`). [stock only]
4. Pressing Left on a setting row moves focus to the contents tree (`settings.action.focusTOC`).
5. Pressing Enter on a tree row moves focus into the settings list (`settings.action.focusSettingsList`).
6. Pressing Enter on a setting row focuses its control (`settings.action.focusSettingControl`).
7. Pressing Escape moves focus up one level, from the control to the row and from the row to the tree, and does nothing special in the search box or a JSON editor (`settings.action.focusLevelUp`).
8. Pressing `Shift+F9` on a row opens its gear menu (`settings.action.showContextMenu`).

### ED-8.44
1. A setting named in another setting's description as `#editor.fontSize#` shows as a link.
2. Clicking the link scrolls the list to that setting and moves focus to its control.
3. When a search or a contents tree group hides the target, the editor first clears the group focus, and if the target is still hidden it searches for `@id:<setting>` so the target shows.
4. Other links in descriptions open as normal links, and command links run their command.

### ED-8.45
1. When a setting is also changed in exactly one other scope, its label reads "Modified in" or "Also modified in" followed by the scope name (User, Workspace or Remote) as a link.
2. When a setting is changed in several scopes or has language default overrides, its label reads "Modified elsewhere" or "Also modified elsewhere" and the hover lists each scope or language as a link.
3. Clicking a scope link, or pressing Space or Enter on the inline one, switches the Settings Switcher to that scope and drops any language filter.
4. After the switch the list is filtered to that one setting by its id.
5. Clicking a language link applies the language filter for that language.
6. The hover of the "Managed by organization" label has a View policy settings link that applies the organization policy filter (`_settings.action.viewPolicySettings`).

### ED-8.46
1. In an untrusted window on the Workspace or Folder scope, a "Workspace Trust" group is added at the top of the contents tree when the workspace file or folder has configured settings that need trust.
2. Choosing the group lists those settings, so the person can see which values are held back.
3. The group does not show on the User scope, in a trusted window, or when no configured setting needs trust.
4. Because Cursor ships `security.workspace.trust.enabled` off, the group rarely shows in Cursor. [cursor only]

### ED-8.47
1. A list setting, including the exclude and include pattern lists, shows each entry as a row with an edit pencil icon and a remove cross icon.
2. A reset icon shows on rows that differ from the default.
3. An object setting shows key and value columns with the same row icons.
4. The edit icon turns the row into inline inputs.
5. An Add Item or Add Pattern button below the rows adds a new entry as inline inputs.
6. Rows of a list setting can be dragged to a new place.

### ED-8.48
1. When a search also matches settings of extensions that are not installed, a "Show matching extensions" button is added after the results.
2. Clicking the button opens the extension search filtered to those extension ids (`workbench.extensions.action.showExtensionsWithIds`).
3. A recommended extension can show as its own group in the list.
4. The recommended extension group has a Show Extension button that opens the extension (`extension.open`) and a Dismiss button that removes the group.
### ED-8.49
1. While Settings Sync is off, a "Backup and Sync Settings" button at the right of the Settings editor header turns Settings Sync on (`workbench.userDataSync.actions.turnOn`). [stock only]
2. Once sync is on, or while it is working, the button is replaced by a "Last synced: <time>" label. [stock only]
3. The "Last synced" label refreshes every minute. [stock only]

### ED-8.50
1. With Settings Sync on, a setting on the sync ignore list shows a "Not synced" label whose hover says "This setting is ignored during sync". [stock only]
2. The setting's gear menu has a Sync This Setting row (`settings.stopSyncingSetting`) that shows a check while the setting syncs. [stock only]
3. Choosing the Sync This Setting row adds the setting to or removes it from the ignore list. [stock only]
4. The Sync This Setting row is left out for settings that do not allow sync to be ignored, and while Settings Sync is off. [stock only]
5. Setting `settingsSync.ignoredSettings` holds the ids of the settings that Settings Sync leaves out. [stock only]

### ED-8.51
1. In the user or workspace `settings.json`, a small edit icon appears in the glyph margin on the line of a boolean or enum setting, under the text cursor or the mouse.
2. Clicking the icon opens a menu with true and false, or with the enum values.
3. Choosing a value writes it into the file, selects it and highlights it.
4. When two settings share a line, each one gets a sub menu.
5. Settings held by an organization policy get no icon.
6. Setting `editor.glyphMargin` turned off hides the icon.

### ED-8.52
1. In `settings.json` and in the settings part of a `.code-workspace` file, an entry that cannot take effect gets a squiggle and a message.
2. A setting that is set by system policy gets a message that says so.
3. A language override block for a setting that cannot be overridden by language gets a message.
4. A setting that cannot apply while a profile other than the default one is active, or that is set to apply in all profiles, gets a message.
5. A window, machine or application scope setting placed in the wrong file or window gets a message.
6. A setting that needs a trusted workspace gets a message with a Manage Workspace Trust quick fix.
7. An unknown setting gets a message.
8. An unsupported property in a workspace file gets a message.

### ED-8.53
1. With `workbench.settings.openDefaultSettings` on, opening a settings JSON file opens a side-by-side editor with the read-only default settings on the left and the file on the right.
2. With `workbench.settings.useSplitJSON` on, editing settings as JSON opens the same side-by-side editor.
3. Open Default Settings (JSON) (`workbench.action.openRawDefaultSettings`) opens the read-only defaults alone from the Command Palette.
4. In Cursor the side-by-side editor is a plain pair of editors and not the stock Preferences Editor pane. [cursor only]

### ED-8.54
1. The title bar button of the Keyboard Shortcuts editor opens `keybindings.json`, and creates it with a short comment and an empty array when it is missing (`workbench.action.openGlobalKeybindingsFile`).
2. In that file a title bar button, Open Keyboard Shortcuts, goes back to the table (`workbench.action.openGlobalKeybindings`).
3. Open Default Keyboard Shortcuts (JSON) (`workbench.action.openDefaultKeybindingsFile`) opens a read-only list of every default binding from the Command Palette.
4. With `workbench.settings.openDefaultKeybindings` on, opening `keybindings.json` also opens the defaults on its left.
5. In stock the same two title bar rows also show in the modal editor title menu. [stock only]

### ED-8.55
1. With the user `keybindings.json` active, pressing `Ctrl+K Ctrl+K` opens a key-capture box over the editor that says "Press desired key combination and then press ENTER."
2. The box says how many existing commands already use the combination.
3. Pressing Enter inserts a `{ "key": ..., "command": ..., "when": ... }` snippet at a fitting place in the array, with the command id and the when clause as tab stops.
4. Define Keybinding in the editor content menu and in the Command Palette opens the same box (`editor.action.defineKeybinding`).
5. Escape clears the combination and a second Escape closes the box, and clicking away closes it without inserting.

### ED-8.56
1. In the user `keybindings.json`, each "key" value is checked against the current keyboard layout, and text that does not match what the layout produces gets an info mark.
2. The hover of the info mark says "<keys> for your current keyboard layout", with the US keys added when they differ.
3. A combination that cannot be typed on the layout gets an error mark, and its hover says so.
4. The marks also show in the overview ruler.

### ED-8.57
1. Toggle Keyboard Shortcuts Troubleshooting (`workbench.action.toggleKeybindingsLog`, Command Palette, Developer category) turns key logging on.
2. When logging turns on, the Window log opens in the Output panel (`workbench.action.showWindowLog`).
3. While logging is on, every key press flashes a red box for 800 milliseconds around the element that received the key.
4. Running the command again turns logging and the red boxes off.

### ED-8.58
1. The Import Settings from VS Code row in the General tab has an Import button. [cursor only]
2. Pressing Import opens a confirm box that says "Are you sure? This replaces your settings and extensions with your vscode ones. This cannot be undone." with Cancel and Confirm buttons. [cursor only]
3. Cancel closes the confirm box without importing. [cursor only]
4. Confirm runs the import (`cursorai.action.importvscode`), and the button reads Importing while it works. [cursor only]
5. When the import ends, the button reads Success! for a few seconds. [cursor only]
6. A notification says "VS Code import completed!", or says "Error importing extensions, please try again." when it fails. [cursor only]

### ED-8.59
1. The Reset "Don't Ask Again" Dialogs row in the General tab says "See warnings and tips that you've hidden" and has a Show button that reads Hide while the list is open. [cursor only]
2. Show opens a "Hidden dialogs" list with one row for each dialog the person marked "don't ask again", each with a Restore button. [cursor only]
3. Clicking Restore on a row makes that dialog show again and takes it out of the list. [cursor only]
4. A Reset All button below the list brings back all hidden dialogs. [cursor only]
5. With nothing hidden, the list says "No Hidden Dialogs Yet" and explains that hidden dialogs will appear there. [cursor only]

### ED-8.60
1. The Network Diagnostics row on the Browser & Network tab has a Run Diagnostic button. [cursor only]
2. While the check runs, the button shows a spinner and the text "Running Diagnostic". [cursor only]
3. A results panel opens under the row with one line per check, each with its own log lines. [cursor only]
4. A Show Logs button shows the log lines of the results, and it reads Hide Logs while they are open. [cursor only]
5. A Copy results button copies the results. [cursor only]

### ED-8.61
1. A search box at the top of the Cursor Settings sidebar shows "Search settings" and the key for the focus command, and `Ctrl+F` puts focus in it while the pane has focus (`aiSettings.action.focusSearch`). [cursor only]
2. While the box is empty, Up and Down move a highlight through the tabs and Enter opens the highlighted tab. [cursor only]
3. Once something is typed, the tab list is replaced by a result list across all tabs, each result with its tab name on the right. [cursor only]
4. When nothing matches, the list says "No results found." [cursor only]
5. Up, Down and Enter pick a result, which opens its tab and scrolls to the item. [cursor only]
6. A result that stands for another page, such as VS Code Settings, closes Cursor Settings and runs its command. [cursor only]
7. Escape leaves the box. [cursor only]
8. When the pane is 710 pixels wide or less, the sidebar shrinks to icons and the search box is hidden. [cursor only]

### ED-8.62
1. A banner above the General tab says "Plugins, MCPs, Skills, and Rules have moved to Customize" with the line "We've introduced a new home for all the ways to customize Cursor." [cursor only]
2. A Dismiss button on the banner hides it for good. [cursor only]
3. An Open Customize button on the banner opens the Customize editor on its Skills tab and also dismisses the banner. [cursor only]
4. The banner shows only on the General tab and only until it is dismissed. [cursor only]

### ED-8.63
1. When the person is signed in with a known email, the top of the Cursor Settings sidebar shows a round avatar with the first letter of the email, the email and the plan name. [cursor only]
2. When the pane is 710 pixels wide or less and the sidebar shrinks to icons, hovering the header shows the email and plan in a tooltip. [cursor only]
3. Without a known email the header is not shown. [cursor only]

### ED-8.64
1. A signed-in person on the free plan sees an upgrade button below the sidebar tabs, reading "Upgrade to Pro" or the label of the next upgrade step, which starts the upgrade flow. [cursor only]
2. A person who is eligible for the referral program sees a gift pill with the reward text. [cursor only]
3. Clicking the gift pill opens a referral link box. [cursor only]
4. An X appears on the gift pill on hover, and clicking it dismisses the pill for good. [cursor only]
5. Both prompts are hidden while the sidebar search box has text. [cursor only]
6. In the narrow icon sidebar, each prompt shows only its icon. [cursor only]

### ED-8.65
1. In the Skills, Subagents, Rules and Commands tabs, each scope group other than Team and Plugin has a New button. [cursor only]
2. Clicking New creates an item of that tab's kind in that group's scope, for example user or workspace. [cursor only]
3. In the MCPs tab a New button creates a server in the account, user or workspace target. [cursor only]
4. When more than one target is available, the MCPs New button becomes a menu to pick the target. [cursor only]

### ED-8.66
1. The Installed group of the Plugins tab has an Add button. [cursor only]
2. Add opens a menu named "Add plugin source" with From Marketplace, From GitHub Repository and From Local Repository. [cursor only]
3. From Marketplace opens the Customize browse view. [cursor only]
4. The two repository rows show only when the page supplies them, and without them Add goes straight to the browse view. [cursor only]

### ED-8.67
1. The Installed group of the Plugins tab has an Edit button that reads Done while edit mode is on. [cursor only]
2. In edit mode, installed plugins can be removed. [cursor only]
3. A plugin can be uninstalled from the whole install, from one project or from one scope. [cursor only]

### ED-8.68
1. Running "Preferences: Configure Runtime Arguments" opens the argv.json file in a pinned editor tab.
2. The file has a schema, so completion and hover text describe the options, such as `locale`, `disable-hardware-acceleration`, `disable-lcd-text`, `force-color-profile`, `enable-crash-reporter`, `proxy-bypass-list`, `log-level`, `js-flags` and `remote-debugging-port`, plus Linux options such as `password-store`.
3. Several descriptions say the app must be restarted before a changed option takes effect.

### ED-8.69
1. With text selected, the editor right-click menu shows Create Rule near the top, in the navigation group at order 20, and it is not available when nothing is selected. [cursor only]
2. Choosing it saves the selected text, trimmed, as a new manual rule file in the workspace's .cursor/rules folder.
3. The rule name is the first six words of the selection joined by dashes, lower-cased with quotes dropped and other punctuation turned into dashes, cut to 12 characters; if no word is usable it uses the first three parts of the file's relative path, or the word rule, and if a rule of that name already exists it adds -1, -2 and so on.
4. A notice reads "Created manual rule '<name>' in .cursor/rules. Attach it explicitly when you want it applied." with an Open Rule button that opens the new rule file in a pinned tab.
5. When the command cannot run it shows a warning instead: "Open a file to create a cursor rule.", "Select the text you want to turn into a rule.", "Highlight some text before creating a rule." or "Open a workspace folder to store cursor rules."; a write failure shows "Failed to create rule: <message>".
6. The rule is created as a manual rule, so it applies only when the person attaches it, as the notice says. [cursor only]

### ED-8.70
1. The Settings editor table of contents has an Application > Proxy section that lists the `http.*` settings that apply to the app.
2. Setting `http.proxy` takes a proxy URL starting with http, https, socks, socks4, socks4a, socks5 or socks5h (an empty value is allowed), and when it is empty the app uses the `http_proxy` and `https_proxy` environment variables.
3. Setting `http.noProxy` lists domain names whose HTTP and HTTPS requests skip the proxy.
4. Setting `http.proxyAuthorization` is sent as the `Proxy-Authorization` header on every request, and it is empty by default.
5. Setting `http.proxyKerberosServicePrincipal` overrides the Kerberos service name used to sign in to the proxy, and when it is empty a name based on the proxy host is used.
6. Setting `http.proxyStrictSSL` (on by default) checks the proxy server certificate against the trusted certificate authorities.
7. Setting `http.systemCertificates` (on by default) loads trusted certificate authorities from the operating system, and on Windows and macOS turning it off needs a window reload.
8. Setting `http.systemCertificatesNode` (experimental) loads the system certificates through Node.js built-in support and needs a window reload. [stock only]
9. When the proxy asks for a username and password, the Proxy Authentication Required box described in DL-1.8 opens.

### ED-8.71
1. Setting `keyboard.dispatch` is listed only on macOS and Linux and chooses `code` (the default and the recommended value) or `keyCode` for how a key press is matched to a shortcut.
2. With `keyCode` the app uses a plain key code mapper in place of the one built from the current keyboard layout.
3. Changing the keyboard settings takes effect right away, without a restart.
4. Setting `keyboard.mapAltGrToCtrlAlt` is listed only on Windows, is off by default, and makes AltGraph combinations count as Ctrl+Alt combinations when matching shortcuts.

### ED-8.72
1. The built-in configuration-editing extension attaches the keybindings schema to the user `keybindings.json` and to each profile's copy, so those files get completion, hover text and error marks.
2. Starting a new entry in the array offers a snippet with `key`, `command` and `when` as tab stops.
3. The `command` value completes every known command id except those that start with an underscore, shows the command's description or menu title beside each, and also accepts the same id with a leading minus sign to remove its default key.
4. The `key`, `when` and `args` properties show the descriptions "Key or key sequence (separated by space)", "Condition when the key is active." and "Arguments to pass to the command to execute."
5. When a command declares the shape of its single argument, `args` is checked against that shape and becomes required if the argument is not optional.
6. A `command` written as a list is marked with "Incorrect type. Expected "string". The field 'command' does not support running multiple commands. Use command 'runCommands' to pass it multiple commands to run."
7. Comments and trailing commas in the file raise no errors.
8. A `systemWide` property, true or false with false as the default, is offered with a description that calls it an OS-wide shortcut that works when the app is not focused, on desktop only, for single key combinations only, with `when` ignored. [stock only]
9. An entry in the user file with `"systemWide": true` runs its command even when the app window is not focused, provided the key is a single combination, and keys from the defaults or from extensions never do. [stock only]
10. A warning notification says the `when` clause is ignored for system-wide keybindings, and another lists key combinations that could not be registered because the OS or another app already holds them. [stock only]

## Unclear

- ED-8.8 piece 8 and the pieces in ED-8.53 and ED-8.54 describe the same visible effect of `workbench.settings.openDefaultSettings`, `workbench.settings.useSplitJSON` and `workbench.settings.openDefaultKeybindings`, because the feature blocks list the same settings under each feature.
- ED-8.22 piece 5: where the usage bar widget sits was not traced, so the piece names only what it opens.
- ED-8.31 piece 2: the options inside the Update Access picker were not read, so none are named.
- ED-8.32: what Tracing, Local Trace Mode and Continuous RPC File Recording record was not checked, so the pieces name only the rows.
- ED-8.33 pieces 6 to 9: the search box, scope menu, grouped lists and empty state were read from module names, not seen on screen.
- ED-8.58: the Import Settings from VS Code row text also mentions keybindings but the confirm text names only settings and extensions, so no piece asserts what the import covers.
- ED-8.65 and ED-8.66: what happens after New, From GitHub Repository or From Local Repository is clicked was not read, so no piece describes it.
- ED-8.67: the controls shown while edit mode is on were not read, so the pieces name only the Edit and Done button and the three uninstall scopes.
- ED-8.43 pieces 2 and 3 are tagged `[stock only]` because exact id searches did not find `settings.action.focusSettingsFromSearchOnEnter` or `settings.action.focusSearchFromSettings` in the Cursor bundle, and ED-8.2 piece 7 is left untagged although `settings.action.showPreviousSearch` was not found either, since the bundle has a method named `showPreviousSearchQuery`, so all three behaviors may exist in Cursor in another form.
- ED-8.48: whether the experiment data that drives the recommended extension group reaches Cursor was not checked, so piece 3 may never show.
- ED-8.14 (Profiles editor) needs more than 10 pieces: the Import Profile choices, the Export flow, the From Template submenu entries and each Contents source are left out, so it should be split.
- ED-8.33 (Open Customize) needs more than 10 pieces: its search, scope menu, groups and empty state are each one piece but have inner behavior that was not read, so it should be split once the on-screen form is known.
- ED-8.5 (Change a setting with its widget) is at 10 pieces with the checkbox and the drop-down merged in piece 2 and the number box and text boxes merged in piece 3, so it could be split if finer counting is wanted.
