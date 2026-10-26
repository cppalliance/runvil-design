Group: ED-8 Settings, shortcuts and configuration editors. Section: Editor. Date: 2026-10-25. Pass: 2.
Features: 72.

### ED-8.1 Open the Settings editor
- Tag: both
- Evidence: source
- Boundary: no
- What: The person opens the Settings editor as an editor tab, on the User scope, from the File > Preferences menu, the Manage gear menu, a key, or the Command Palette. Open Accessibility Settings opens it filtered to the accessibility tag.
- Commands: workbench.action.openSettings; workbench.action.openSettings2; workbench.action.openGlobalSettings; workbench.action.openAccessibilitySettings; workbench.action.openGlobalSettingsCursorAlias
- Menus: MenubarPreferencesMenu (group 2_configuration, order 2); GlobalActivity (group 2_configuration, order 2); CommandPalette
- Keys: Ctrl+, (workbench.action.openSettings)
- Settings: workbench.settings.editor
- Cursor: changed (the pane, the commands and the menu rows are present, but Ctrl+, is taken by Cursor Settings outside the Agents window, so stock Settings opens through the "VS Code Settings" menu row, the key Ctrl+Shift+, or Open Settings (UI); the stock "Settings" row in File > Preferences is shown only for one window context, see ED-8.21)
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:227-270,313-323,373-385; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts; V\src\vs\workbench\browser\workbench.contribution.ts:717; C bundle (module vs/workbench/contrib/preferences/browser/settingsEditorPane.contribution, pane "Settings Editor 2", id `workbench.editor.settings2`; module vs/workbench/contrib/preferences/browser/preferences.contribution, `workbench.action.openSettings` with weight 200 and Ctrl+,; `cursor.openVSCodeSettingsFromMenu` titled "VS Code Settings" with Ctrl+Shift+, and menu MenubarPreferencesMenu order 2 when not the Agents window)

### ED-8.2 Search and filter settings
- Tag: both
- Evidence: source
- Boundary: no
- What: A search box at the top of the Settings editor narrows the list as the person types. A Filter Settings drop-down and a suggestion list add filters (modified, extension ID, feature, tag, language, setting ID, online services, organization policies, stable, preview, experimental, advanced). The Modified filter shows only settings the person changed. A result count, a "No Settings Found" message, a Clear Filters link and Up-arrow search history sit around it. Stock can also show AI-recommended results.
- Commands: settings.action.search; settings.action.clearSearchResults; settings.action.suggestFilters (search bar button id); settings.action.showAIResults (search bar button id); settings.action.showPreviousSearch; settings.action.focusSettingsFromSearch; settings.action.focusSettingsFromSearchOnEnter; settings.action.toggleAiSearch; settings.filterByOnline; settings.filterByTelemetry; settings.filterUntrusted
- Menus: MenubarPreferencesMenu (group 3_settings, order 1, the Online Services Settings row)
- Keys: Ctrl+F (settings.action.search); Escape when the search box has focus (clear results); Up (show previous search); Down or Enter (focus first setting); Ctrl+I when AI results exist (toggle AI search)
- Settings: workbench.settings.enableNaturalLanguageSearch; workbench.settings.showAISearchToggle
- Cursor: changed (search box, Filter Settings drop-down, `settings.action.suggestFilters`, Clear Filters, and the Modified, Extension ID, Feature, Tag, Language and Online services rows are present; the AI search toggle `settings.action.toggleAiSearch`, `settings.action.showAIResults`, setting `workbench.settings.showAISearchToggle`, the "Show AI-recommended results" label and the rows Setting ID, Organization policies, Stable, Preview, Experimental and Advanced were not found)
- Source: V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts:97,123-140,353-357,780-835,1036-1040; V\src\vs\workbench\contrib\preferences\browser\settingsSearchMenu.ts:16; V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:55-69,477-545,606-760; V\src\vs\workbench\contrib\preferences\common\preferences.ts:62-66,126-128; C bundle (modules vs/workbench/contrib/preferences/browser/settingsSearchMenu and settingsEditor; labels "Search settings", "Filter Settings", "Modified", "Extension ID...", "Feature...", "Tag...", "Language...", "Online services", "Clear Filters")

### ED-8.3 Switch settings scope
- Tag: both
- Evidence: source
- Boundary: no
- What: A Settings Switcher bar under the search box picks which scope the editor reads and writes: User, Remote (only in a remote window, with the host name), Workspace (shows the workspace name, disabled with no folder open), and a Folder drop-down that appears in a multi-root workspace. A folder can also be chosen from the Explorer right-click menu.
- Commands: workbench.action.openWorkspaceSettings; workbench.action.openFolderSettings; workbench.action.openRemoteSettings; _workbench.action.openFolderSettings; workbench.action.openGlobalSettings
- Menus: ExplorerContext (group 2_workspace, order 20, on a root folder); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\preferences\browser\preferencesWidgets.ts:42-130,219-370; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts (settingsTargetsWidget and onDidSettingsTargetChange); V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:354-368,406-470,555-570; C bundle (module vs/workbench/contrib/preferences/browser/preferencesWidgets; label "Settings Switcher"; ids `workbench.action.openWorkspaceSettings`, `workbench.action.openFolderSettings`, `_workbench.action.openFolderSettings`, `workbench.action.openRemoteSettings`)

### ED-8.4 Browse settings by table of contents
- Tag: both
- Evidence: source
- Boundary: no
- What: A contents tree on the left lists setting groups (Commonly Used, Text Editor, Workbench, Window, Chat, Features, Application, Security, and one group per extension). Clicking a group scrolls the list to it, and the tree follows the scroll. While searching, the tree either hides or filters to groups that have matches.
- Commands: settings.action.focusTOC; settings.action.focusLevelUp; settings.action.focusSettingsList
- Menus: none found
- Keys: Left when a setting row has focus (focus the contents tree); Escape (move focus up one level); Enter (focus settings list)
- Settings: workbench.settings.settingsSearchTocBehavior
- Cursor: present
- Source: V\src\vs\workbench\contrib\preferences\browser\tocTree.ts; V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts:63,75,127,169,181,288,385,432; V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:770-860; V\src\vs\workbench\contrib\preferences\common\preferencesContribution.ts:118; C bundle (modules vs/workbench/contrib/preferences/browser/tocTree and settingsLayout; labels "Commonly Used", "Text Editor", "{0}, group"; setting `workbench.settings.settingsSearchTocBehavior`)

### ED-8.5 Change a setting with its widget
- Tag: both
- Evidence: source
- Boundary: no
- What: Each setting row shows a title, description and a control that fits its type: checkbox, drop-down, number box, text box (single or multi-line), list, object table, exclude and include pattern lists, or an "Edit in settings.json" link for complex values. A gear menu on the row resets the setting, copies its ID, copies it as JSON or as a URL, and applies it to all profiles.
- Commands: settings.resetSetting; settings.copySettingId; settings.copySettingAsJSON; settings.copySettingAsURL; settings.applyToAllProfiles; settings.stopSyncingSetting; settings.action.focusSettingControl; settings.action.showContextMenu
- Menus: none found
- Keys: Enter (focus the setting control); Shift+F9 (show setting context menu)
- Settings: workbench.settings.alwaysShowAdvancedSettings; workbench.settings.applyToAllProfiles
- Cursor: changed (all widget renderers, Reset Setting, Copy Setting ID, Copy Setting as JSON, Copy Setting as URL, Apply Setting to all Profiles, Add Item, Add Pattern and "Edit in settings.json" are present; the "Sync This Setting" action `settings.stopSyncingSetting` and the setting `workbench.settings.alwaysShowAdvancedSettings` were not found)
- Source: V\src\vs\workbench\contrib\preferences\browser\settingsTree.ts (renderers SettingBoolRenderer, SettingEnumRenderer, SettingNumberRenderer, SettingTextRenderer, SettingMultilineTextRenderer, SettingArrayRenderer, SettingObjectRenderer, SettingBoolObjectRenderer, SettingExcludeRenderer, SettingIncludeRenderer, SettingComplexRenderer; action classes with ids settings.copySettingId, settings.copySettingAsJSON, settings.copySettingAsURL, settings.stopSyncingSetting, settings.applyToAllProfiles); V\src\vs\workbench\contrib\preferences\browser\settingsWidgets.ts; V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:797-845; C bundle (modules vs/workbench/contrib/preferences/browser/settingsTree and settingsWidgets; ids settings.resetSetting, settings.copySettingId, settings.copySettingAsJSON, settings.copySettingAsURL, settings.applyToAllProfiles)

### ED-8.6 Read setting indicators
- Tag: both
- Evidence: source
- Boundary: no
- What: Small labels next to a setting title say where its value comes from: "Modified in" or "Also modified in" other scopes, "Default value changed" with the extension or language that changed it, "Applies to all profiles", "Requires workspace trust", "Managed by organization", Preview, Experimental, Advanced, and "Not synced" for settings ignored by Settings Sync. Hovering a label explains it, and the Workspace Trust one links to the trust editor. The header sync controls (Backup and Sync Settings, Last synced) belong to Settings Sync, which the catalog left out.
- Commands: settings.filterUntrusted; workbench.trust.manage
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (Modified in, Also modified in, Default value changed, Applies to all profiles, Requires workspace trust, Managed by organization, Preview and Experimental are present; "Not synced", "Cannot be changed in Agents window", "Backup and Sync Settings" and "Last synced: {0}" were not found)
- Source: V\src\vs\workbench\contrib\preferences\browser\settingsEditorSettingIndicators.ts; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts (turnOnSyncButton, lastSyncedLabel); C bundle (module vs/workbench/contrib/preferences/browser/settingsEditorSettingIndicators; labels "Modified in", "Default value changed", "Requires workspace trust", "Managed by organization", "Applies to all profiles")

### ED-8.7 Set language-specific settings
- Tag: both
- Evidence: source
- Boundary: no
- What: The person picks a language from a list (Configure Language Specific Settings) and the editor opens the settings file at that language's override block. In the Settings editor a Language filter (`@lang:`) shows a language's values, and the scope tabs gain a language suffix while it is active.
- Commands: workbench.action.configureLanguageBasedSettings
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\preferences\browser\preferencesActions.ts:30-80; V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:341-350; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts (languageFilter, lines 353 and 1276-1303 and 1878-1893); V\src\vs\workbench\contrib\preferences\browser\preferencesWidgets.ts:327-340; V\src\vs\workbench\contrib\preferences\common\preferences.ts:106,115; C bundle (id `workbench.action.configureLanguageBasedSettings`, label "Configure Language Specific Settings...", label "Language...")

### ED-8.8 Open settings or shortcuts as JSON
- Tag: both
- Evidence: source
- Boundary: no
- What: Each editor has a button in the editor title bar that flips to its JSON file: Open Settings (JSON) in the Settings editor, Open Keyboard Shortcuts (JSON) in the Keyboard Shortcuts editor, and the matching Open Settings (UI) and Open Keyboard Shortcuts buttons in the JSON files. The Command Palette also opens user, application, workspace, folder, remote, default settings JSON and default shortcuts JSON. Inside keybindings.json, Define Keybinding opens a key-capture box.
- Commands: settings.switchToJSON; _workbench.openUserSettingsEditor; workbench.action.openSettingsJson; workbench.action.openApplicationSettingsJson; workbench.action.openWorkspaceSettingsFile; workbench.action.openFolderSettingsFile; workbench.action.openRemoteSettingsFile; workbench.action.openRawDefaultSettings; workbench.action.openGlobalKeybindingsFile; workbench.action.openDefaultKeybindingsFile; editor.action.defineKeybinding
- Menus: EditorTitle (navigation, order 1, when the Settings editor or keybindings editor is active, or the matching JSON file); ModalEditorEditorTitle (same rows, stock only); EditorContent (Define Keybinding in keybindings.json); CommandPalette
- Keys: Ctrl+K Ctrl+K (editor.action.defineKeybinding, when the resource is the user keybindings file)
- Settings: workbench.settings.editor; workbench.settings.useSplitJSON; workbench.settings.openDefaultSettings; workbench.settings.openDefaultKeybindings
- Cursor: changed (the title-bar toggles, the JSON open commands and Define Keybinding are present; the stock "Preferences Editor" pane that shows default settings beside the JSON (`workbench.editor.preferences`) and the ModalEditorEditorTitle rows were not found)
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:67,275-312,327-340,389-405,555-585,925-968,1287-1316,1402-1470; V\src\vs\workbench\contrib\preferences\browser\preferencesEditor.ts:39; V\src\vs\workbench\browser\workbench.contribution.ts:546-561; V\src\vs\workbench\contrib\preferences\browser\preferencesRenderers.ts; V\src\vs\workbench\contrib\preferences\browser\keybindingsEditorContribution.ts; V\src\vs\workbench\contrib\preferences\browser\keybindingWidgets.ts; C bundle (ids `settings.switchToJSON`, `_workbench.openUserSettingsEditor`, `workbench.action.openGlobalKeybindingsFile`, `workbench.action.openDefaultKeybindingsFile`, `editor.action.defineKeybinding`; modules vs/workbench/contrib/preferences/browser/preferencesRenderers and keybindingWidgets; no match for `workbench.editor.preferences`)

### ED-8.9 Browse and search keyboard shortcuts
- Tag: both
- Evidence: source
- Boundary: no
- What: The Keyboard Shortcuts editor lists every command in a table with the columns Command, Keybinding, When and Source. A search box filters by plain words or by `@command:`, `@keybinding:`, `@source:` (user, default, system, extension) and `@ext:`. Three rows in the editor title menu jump to System, Extension or User shortcuts, a sort button flips between alphabetical and precedence order, and search history is kept.
- Commands: workbench.action.openGlobalKeybindings; keybindings.editor.searchKeybindings; keybindings.editor.clearSearchResults; keybindings.editor.clearSearchHistory; keybindings.editor.toggleSortByPrecedence; keybindings.editor.showDefaultKeybindings; keybindings.editor.showExtensionKeybindings; keybindings.editor.showUserKeybindings; keybindings.editor.focusKeybindings
- Menus: MenubarPreferencesMenu (group 2_configuration, order 4); GlobalActivity (group 2_configuration, order 4); EditorTitle (group 1_keyboard_preferences_actions for the three Show rows); CommandPalette
- Keys: Ctrl+K Ctrl+S (workbench.action.openGlobalKeybindings); Ctrl+F (search, when the editor is active); Alt+P (sort by precedence); Ctrl+Down (move from the search box to the table); Escape (clear search, when the box has text)
- Settings: none found
- Cursor: present (every `keybindings.editor.*` id, the labels "Type to search in keybindings", "Sort by Precedence (Highest first)", "Show System Keybindings", "Show Extension Keybindings", "Show User Keybindings" and "Clear Keyboard Shortcuts Search History" were found; the Cursor Settings General tab also has a Keyboard Shortcuts row whose Open button runs `workbench.action.openGlobalKeybindings`)
- Source: V\src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts:145-148,369,488-509; V\src\vs\workbench\services\preferences\browser\keybindingsEditorModel.ts:43-47; V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:877-930,973-1075,1153-1195,1246-1258; V\src\vs\workbench\contrib\preferences\common\preferences.ts:74-105; C bundle (module vs/workbench/contrib/preferences/browser/keybindingsEditor, pane "Keybindings Editor", id `workbench.editor.keybindings`; ids `keybindings.editor.*`; Cursor registration of `workbench.action.openGlobalKeybindings` with the same chord and menu rows)

### ED-8.10 Record keys to search shortcuts
- Tag: both
- Evidence: source
- Boundary: no
- What: A Record Keys button inside the search box switches the box into key capture. A "Recording Keys" badge shows, the placeholder reads "Recording Keys. Press Escape to exit", and pressing a key combination lists the commands bound to it.
- Commands: keybindings.editor.recordSearchKeys
- Menus: none found
- Keys: Alt+K, or Cmd+Alt+K on macOS (keybindings.editor.recordSearchKeys, when the search box has focus); Escape (leave recording)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts:145,369-375,451; V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:1166-1180; C bundle (labels "Record Keys", "Recording Keys", "Recording Keys. Press Escape to exit"; id `keybindings.editor.recordSearchKeys`)

### ED-8.11 Add, change, remove or reset a shortcut
- Tag: both
- Evidence: source
- Boundary: no
- What: Pressing Enter or double-clicking a row opens a key-capture box that says "Press desired key combination and then press ENTER" and warns when other commands already use that combination. It accepts chords. A right-click menu on a row offers Change Keybinding or Add Keybinding, Remove Keybinding and Reset Keybinding, and the Copy, Copy Command ID and Copy Command Title items. A pencil or plus icon on the row does the same as Enter.
- Commands: keybindings.editor.defineKeybinding; keybindings.editor.addKeybinding; keybindings.editor.removeKeybinding; keybindings.editor.resetKeybinding; keybindings.editor.copyKeybindingEntry; keybindings.editor.copyCommandKeybindingEntry; keybindings.editor.copyCommandTitle
- Menus: none found (the row menu is built in code in keybindingsEditor.ts, not through a MenuId)
- Keys: Enter (keybindings.editor.defineKeybinding, when a row has focus); Ctrl+K Ctrl+A (add); Delete, or Cmd+Backspace on macOS (remove, when the table has focus); Ctrl+C (copy row)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts:748-830; V\src\vs\workbench\contrib\preferences\browser\keybindingWidgets.ts:135-260; V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:1085-1150,1207-1245; C bundle (module vs/workbench/contrib/preferences/browser/keybindingWidgets; labels "Change Keybinding...", "Add Keybinding...", "Remove Keybinding", "Reset Keybinding", "Copy Command ID", "Copy Command Title", "Press desired key combination and then press ENTER.")

### ED-8.12 Edit a shortcut's when expression
- Tag: both
- Evidence: source
- Boundary: no
- What: Choosing Change When Expression in the row menu turns the When cell into a text box with context-key suggestions. Enter saves the new condition and Escape drops it.
- Commands: keybindings.editor.defineWhenExpression; keybindings.editor.acceptWhenExpression; keybindings.editor.rejectWhenExpression
- Menus: none found (row menu built in code)
- Keys: Ctrl+K Ctrl+E (define); Enter (accept, when the When cell has focus and no suggestion is open); Escape (reject)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts:502-509,784-792; V\src\vs\workbench\contrib\preferences\browser\keybindingWidgets.ts; V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:1111-1123,1259-1285; C bundle (label "Change When Expression"; ids `keybindings.editor.defineWhenExpression`, `keybindings.editor.acceptWhenExpression`, `keybindings.editor.rejectWhenExpression`)

### ED-8.13 Show conflicting shortcuts
- Tag: both
- Evidence: source
- Boundary: no
- What: Show Same Keybindings in the row menu fills the search box with that row's key combination so every command sharing it lists together. The key-capture box also says how many existing commands already use the combination being typed.
- Commands: keybindings.editor.showConflicts
- Menus: none found (row menu built in code)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts:329,748,809-818; V\src\vs\workbench\contrib\preferences\browser\keybindingWidgets.ts:230; V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:1194-1206; C bundle (label "Show Same Keybindings"; id `keybindings.editor.showConflicts`; labels "{0} existing commands have this keybinding", "1 existing command has this keybinding")

### ED-8.14 Profiles editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Open Profiles (UI) opens a Profiles editor tab. A list on the left shows each profile with a Built-in label and an Active mark, a New Profile button (with a From Template submenu) and an Import Profile row (from a file or a URL). The right side edits the chosen profile: Name, Icon, Use for Current Window, Use for New Windows, Copy from, Contents (Settings, Keyboard Shortcuts, Snippets, Tasks, MCP Servers, Extensions, each with a Default, None or Copy source) and Folders & Workspaces (Add Folder). Create, Cancel, Export and Delete buttons finish the work. Commands also switch, save, export, delete and open a new window with a profile.
- Commands: workbench.profiles.actions.manageProfiles; workbench.profiles.actions.createProfile; workbench.profiles.actions.createFromCurrentProfile; workbench.profiles.actions.switchProfile; workbench.profiles.actions.exportProfile; workbench.profiles.actions.deleteProfile; workbench.profiles.actions.newWindowWithProfile; workbench.profiles.actions.createTemporaryProfile; workbench.profiles.actions.cleanupProfiles; workbench.profiles.actions.resetWorkspaces; iconSelectBox.focusDown; iconSelectBox.focusNext; iconSelectBox.focusPrevious; iconSelectBox.focusUp; iconSelectBox.selectFocused; workbench.profiles.actions.help
- Menus: GlobalActivity (Profiles submenu, when profiles exist); MenubarPreferencesMenu (Profiles submenu, or a Switch Profile row when none exist); MenubarFileMenu (New Window with Profile); MenubarShare (Export Profile); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (pane "Profiles Editor", id `workbench.editor.userDataProfiles`, all 12 `workbench.profiles.actions.*` ids found in V and the labels "Use for Current Window", "Use for New Windows", "Copy from", "Folders & Workspaces", "New Profile", "Import Profile..." were found)
- Source: V\src\vs\workbench\contrib\userDataProfile\browser\userDataProfilesEditor.ts:101,204-320,538,935-1260,1368-1380,1553-1670; V\src\vs\workbench\contrib\userDataProfile\browser\userDataProfilesEditorModel.ts:989-1114; V\src\vs\workbench\contrib\userDataProfile\browser\userDataProfile.ts:180-210,325-360,398-530; C bundle (modules vs/workbench/contrib/userDataProfile/browser/userDataProfilesEditor, userDataProfile and userDataProfileActions; ids `workbench.profiles.actions.*`)

### ED-8.15 Pick a color theme
- Tag: both
- Evidence: source
- Boundary: no
- What: Color Theme opens a picker of the installed color themes in light, dark and high contrast groups. Moving with the arrow keys previews each theme, Enter keeps it, and a button on the picker shows whether the system color mode is followed and opens its settings. Other rows ask to install or browse more themes. Extra commands flip between light and dark, generate a theme from the current settings and open the Marketplace theme list.
- Commands: workbench.action.selectTheme; workbench.action.toggleLightDarkThemes; workbench.action.generateColorTheme; workbench.action.browseColorThemesInMarketplace; workbench.action.tryNewDefaultThemes
- Menus: MenubarPreferencesMenu (Themes submenu, group 2_configuration, order 7; Color Theme is order 1); GlobalActivity (the same Themes submenu); CommandPalette
- Keys: Ctrl+K Ctrl+T (workbench.action.selectTheme)
- Settings: workbench.colorTheme; workbench.preferredDarkColorTheme; workbench.preferredLightColorTheme; workbench.preferredHighContrastColorTheme; workbench.preferredHighContrastLightColorTheme; window.autoDetectColorScheme; window.autoDetectHighContrast
- Cursor: changed (the picker, its group names, the system color mode button text and the three extra commands are present; `workbench.action.tryNewDefaultThemes` ("Try New Default Themes") was not found; Cursor adds its own built-in themes in `C\extensions\theme-cursor`: Cursor Dark Midnight, Cursor Dark High Contrast, Cursor Dark, Cursor Light, Cursor Light Colorblind, Cursor Dark Colorblind, Cursor Light High Contrast)
- Source: V\src\vs\workbench\contrib\themes\browser\themes.contribution.ts:411-470,568-600,738-800,852-860,903-940; V\src\vs\workbench\services\themes\common\workbenchThemeService.ts:24-38; C\extensions\theme-cursor\package.json; C bundle (module vs/workbench/contrib/themes/browser/themes.contribution; `workbench.action.selectTheme` with weight 200 and Ctrl+K Ctrl+T; labels "Color Theme", "light themes", "dark themes", "high contrast themes", "Detect system color mode enabled. Click to configure.")

### ED-8.16 Pick a file icon theme
- Tag: both
- Evidence: source
- Boundary: no
- What: File Icon Theme opens a picker of the installed file icon themes, plus a None row that turns file icons off. The arrow keys preview each theme and Enter keeps it.
- Commands: workbench.action.selectIconTheme
- Menus: MenubarPreferencesMenu (Themes submenu, File Icon Theme at order 2); GlobalActivity (the same Themes submenu); CommandPalette
- Keys: none found
- Settings: workbench.iconTheme
- Cursor: present
- Source: V\src\vs\workbench\contrib\themes\browser\themes.contribution.ts:496-530,924-931; C bundle (module vs/workbench/contrib/themes/browser/themes.contribution; id `workbench.action.selectIconTheme`; labels "File Icon Theme", "Disable File Icons", "Select File Icon Theme (Up/Down Keys to Preview)")

### ED-8.17 Pick a product icon theme
- Tag: both
- Evidence: source
- Boundary: no
- What: Product Icon Theme opens a picker of the installed icon sets for the workbench itself (Default and any others). The arrow keys preview each set and Enter keeps it.
- Commands: workbench.action.selectProductIconTheme
- Menus: MenubarPreferencesMenu (Themes submenu, Product Icon Theme at order 3); GlobalActivity (the same Themes submenu); CommandPalette
- Keys: none found
- Settings: workbench.productIconTheme
- Cursor: present
- Source: V\src\vs\workbench\contrib\themes\browser\themes.contribution.ts:533-565,932-940; C bundle (module vs/workbench/contrib/themes/browser/themes.contribution; id `workbench.action.selectProductIconTheme`; labels "Product Icon Theme", "Select Product Icon Theme (Up/Down Keys to Preview)")

### ED-8.18 Workspace Trust editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Manage Workspace Trust opens a Workspace Trust editor tab. The header says whether the window is trusted or in Restricted Mode and offers Trust and Don't Trust buttons. A list says what is on or off (tasks, debugging, workspace settings, extensions), with links to the affected settings and extensions. A Trusted Folders & Workspaces table (Host, Path) has Add Folder plus edit and delete icons. Cursor ships the trust setting off by default, so the command is unavailable until a person turns it on.
- Commands: workbench.trust.manage; workbench.trust.configure
- Menus: CommandPalette (both commands are `f1`; other entries come from the Restricted Mode status item, the notice strip and trust prompts, which sections 09, 02 and 12 own)
- Keys: none found
- Settings: security.workspace.trust.enabled; security.workspace.trust.startupPrompt; security.workspace.trust.banner; security.workspace.trust.untrustedFiles; security.workspace.trust.emptyWindow
- Cursor: present (pane "Workspace Trust Editor", id `workbench.editor.workspaceTrust`, `workbench.trust.manage` and `workbench.trust.configure`, labels "Manage Workspace Trust", "You are in Restricted Mode", "Trusted Folders & Workspaces", "Don't Trust")
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts:745-790; V\src\vs\workbench\contrib\workspace\browser\workspaceTrustEditor.ts:669; V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts; C bundle (module vs/workbench/contrib/workspace/browser/workspaceTrustEditor; `workbench.trust.manage` seen 7 times)

### ED-8.19 Open Cursor Settings
- Tag: cursor
- Evidence: bundle
- Boundary: yes - most tabs of this page configure the agent, its models, cloud agents, tools and plan.
- What: Ctrl+Shift+J (or Ctrl+, outside the Agents window) opens Cursor Settings as a pinned editor tab in the last editor group, adding a group on the right if there is none. A sidebar on the left lists the tabs, with an arrow icon on the rows that launch another page, thin dividers between tab groups and a count badge on the Models tab. A search over all tab entries sits on the page (Ctrl+F while it has focus) and jumps to a tab and an item. A Docs link sits at the foot of the sidebar. Each per-tab command opens it on that tab, and the open command accepts a tab id and an item to scroll to.
- Commands: aiSettings.action.open; aiSettings.action.focusSearch ("Focus AI Settings Search"); aiSettings.action.open.<tab> ("Cursor Settings: <tab name>", Command Palette, for the tabs general, chat, background-composer, models, git-prs, rules, mcp, hooks, browser and beta); aiSettings.action.openhidden
- Menus: MenubarPreferencesMenu (group 2_configuration, order 1.5, row "Cursor Settings"); CommandPalette (the open command and each per-tab command). The title bar gear button is owned by TB-5.
- Keys: Ctrl+Shift+J (aiSettings.action.open); Ctrl+, (aiSettings.action.open, when not in the Agents window); Ctrl+F (aiSettings.action.focusSearch, when the settings pane has focus)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (aiSettingsService.js and the workbench contribution that opens `workbench.editor.aisettings.input` in the last group through `renderPopup`; command `aiSettings.action.open` with its keybinding list and menu row; tab list `rPa`, tab titles `NDr`, divider groups `bOk`, footer list `tOk` (only Docs is listed; a Contact entry is defined but not in the list); context keys `isSettingsPaneOpen` and `isSettingsPaneFocused`; sidebar cell classes `cursor-settings-sidebar-cell-label` and `cursor-settings-sidebar-link-external`)

### ED-8.20 General tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - mixed tab: account, startup, layout and editor rows, plus agent notification and window-restoration rows.
- What: The tab that opens first. A Sign In row when signed out, or a Cursor Account row (manage account and billing) when signed in. Then a Preferences group with Editor Settings (an Open button that opens User Settings), Keyboard Shortcuts (an Open button that opens the shortcuts editor) and Import Settings from VS Code, Reset "Don't Ask Again" Dialogs, Layout, Startup, Notifications, Status Bar and Tips. A menu bar icon row shows on macOS and a system tray icon row on other systems. Verify Extension Signatures shows only when a feature gate is on. A few rows show only in the Agents window.
- Commands: aiSettings.action.open.general; workbench.action.openGlobalSettings; workbench.action.openGlobalKeybindings
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (tab list `rPa` and titles `NDr`; settings search registry entries for tab general: sign-in, cursor-account, preferences, editor-settings, keyboard-shortcuts, import-vscode, reset-dialogs, layout, startup, notifications, status-bar, tips, menu-bar-icon, system-tray-icon, verify-extension-signatures, plus agent rows such as auto-continue-interrupted-agents, chat-editor-tabs, completion-sound, system-notifications, open-agents-on-startup)

### ED-8.21 VS Code Settings tab
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A sidebar row with an arrow icon that closes Cursor Settings and opens the stock Settings editor (ED-8.1). It shows in the IDE because the Appearance tab is off there. A matching "VS Code Settings" row in the File > Preferences menu replaces the stock Settings row outside the Agents window.
- Commands: cursor.openVSCodeSettingsFromMenu; workbench.action.openSettings
- Menus: MenubarPreferencesMenu (group 2_configuration, order 2, row "VS Code Settings", when not in the Agents window)
- Keys: Ctrl+Shift+, (cursor.openVSCodeSettingsFromMenu, when not in the Agents window and a focus condition that was not resolved)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (tab launcher map `tVm` from `vscode-settings` to `workbench.action.openSettings`; command class with id `cursor.openVSCodeSettingsFromMenu`, title "VS Code Settings"; visibility function `mvw`, where `vscode-settings` is shown only when the Appearance tab is off; sidebar click handler that closes settings and runs the mapped command)

### ED-8.22 Plan & Usage tab
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Shows the signed-in person's plan usage: a Personal Usage section with included-usage and on-demand meters, a credits card, and a Monthly Limit control (a fixed amount, or unlimited on-demand). It is drawn only when the person is signed in and the account has on-demand spillover (a tiered team plan, an enterprise user, or auto spillover). A usage bar widget also opens it (where that widget sits was not traced). The plan modals that sit beside it belong to DL-6.
- Commands: aiSettings.action.open (with the tab argument plan-usage)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (visibility function `mvw` branch for plan-usage; strings "Personal Usage", "Monthly Limit", component names `plan-usage-credits-card` and `plan-usage-included-usage-limit`; settings registry entry plan-usage/monthly-limit; the usage bar handler that calls `setSettingsOpenData({openTab:"plan-usage"})` then `aiSettingsService.openPopup()`)

### ED-8.23 Agents tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - this tab holds the agent's run, approval, context, terminal and display options.
- What: The tab titled Agents (id chat). It is a long page of toggles and pickers in groups named Run Mode, Execution and Approvals, Context and Tools, Terminal and Editing, Conversation and Inline Diffs, with rows such as a read allowlist, web search and web fetch switches, queue messages, text size, auto format, third-party configs and voice submit keywords. A few rows show only in the IDE and a few only in the Agents window.
- Commands: aiSettings.action.open.chat
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (tab list `rPa`; titles `NDr`; settings registry entries for tab chat, 31 of them, with surface values ide and glass, for example run-mode, execution-approvals, context-tools, terminal-editing, conversation, inline-diffs, read-allowlist, web-search, web-fetch, text-size, auto-format)

### ED-8.24 Cloud Agents tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the tab connects and sets up the cloud agents.
- What: The tab titled Cloud Agents (id background-composer). It shows an Integrations list with Connect or Reconnect buttons. Before that it can show a loading ring, a "Cloud Agents Unavailable" card when Privacy Mode is on (with an Open Privacy Settings button), or a "Get Started" card that asks for an open folder with a Git repository (with an Open Folder button).
- Commands: aiSettings.action.open.background-composer; cursor.connectSlack
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (tab list `rPa`; titles `NDr`; strings "Loading Cloud Agents settings...", "Cloud Agents Unavailable", "Cloud Agents require data storage to function.", "Open Privacy Settings", "Get Started", "Cloud Agents require a Git repository in an open folder.", "Integrations", "Reconnect")

### ED-8.25 Models tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the tab picks and keys the models the agent uses.
- What: The tab titled Models. It has a model list with an Add Custom Model button, a collapsible API Keys section with an Override OpenAI Base URL switch and its base URL box, Task Models, an Explore Subagent Model picker and, in local mode, a Gateway entry. A count badge shows on the sidebar row while a Claude or OpenAI key warning is open.
- Commands: aiSettings.action.open.models
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (tab list `rPa`; titles `NDr`; strings "Add Custom Model", "API Keys", "Override OpenAI Base URL", "Base URL"; settings registry entries models/explore-subagent-model, models/gateway, models/task-models; badge count function `vvw`)

### ED-8.26 Voice tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the options set the voice used with agent chats; the voice call itself is out of scope.
- What: The tab titled Voice. Its rows are the default Voice, Language, Speed, and Submit Keywords (words that send the prompt while dictating, shown only when a realtime-voice condition holds). The tab is drawn only when the voice feature gate is on.
- Commands: aiSettings.action.open (with the tab argument voice)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (visibility function `mvw` branch for voice and the gate value passed in as `voiceTabEnabled`; settings registry entries voice/agent-voice, voice/language, voice/speed, voice/submit-keywords; module realtimeVoiceSettings.js)

### ED-8.27 Source Control tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the rows set how the agent names branches, signs commits and pull requests, and keeps worktrees.
- What: The tab titled Source Control (id git-prs). Groups: Attribution (Commit Attribution, PR Attribution), Branches (Branch Prefix), Pull Requests (PR Link Destination and Review Provider, only when signed in) and Cursor-Managed Worktrees (Worktree Cleanup, Max Worktrees, Max Total Size in GB).
- Commands: aiSettings.action.open.git-prs
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (tab list `rPa`; titles `NDr`; settings registry entries for tab git-prs: attribution, commit-attribution, pr-attribution, branches, branch-prefix, pull-requests, pr-link-destination, review-provider, managed, cleanup, max-count, max-size; redirect map `SOk` sends the hidden worktrees tab id here)

### ED-8.28 Customize tab (launcher)
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it leads to the page that manages agent plugins, tools, skills, rules and hooks.
- What: A sidebar row with an arrow icon that closes Cursor Settings and opens the Customize editor (ED-8.33). It is drawn in the IDE. It has no page of its own.
- Commands: workbench.action.openCustomizeEditor
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (launcher id list `oOk` and check `uei`; sidebar click handler `U` that closes settings and runs `V$n`; visibility function `mvw` branch for customize with the value `customizeSidebarTabEnabled`, passed in as true for the IDE)

### ED-8.29 Browser & Network tab
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The tab titled Browser & Network (id browser). It holds connection options for Cursor itself: HTTP Compatibility Mode (a picker for proxy and VPN setups), Required Domains (the list a firewall must allow, with a Copy Domains button) and Network Diagnostics (a button that checks connectivity to Cursor services). The redirect map sends the hidden network tab id here.
- Commands: aiSettings.action.open.browser
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (tab list `rPa`; titles `NDr`; strings "HTTP Compatibility Mode", "Required Domains", "Copy Domains", "Network Diagnostics", "Check network connectivity to all Cursor services"; settings registry entries browser/http-compatibility, browser/required-domains, browser/diagnostics)

### ED-8.30 Code Intelligence tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the codebase index and ignore rules feed the agent, and the tab also holds the Cursor Tab switch (owned by AC-8).
- What: The tab titled Code Intelligence (id indexing). Groups: Codebase (Ignore Files in .cursorignore, Hierarchical Cursor Ignore, Ignore Symlinks in Cursor Ignore Search), Cursor Tab (an on or off row) and LSP (Enable LSPs, Maximum Local and Remote LSP Workspaces, Enable LSPs for Worktrees; these LSP rows are marked for the Agents window). It is drawn in the IDE by default. The redirect map sends the hidden Tab id here.
- Commands: aiSettings.action.open (with the tab argument indexing)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (tab list `rPa`; titles `NDr`; settings registry entries for tab indexing: codebase, ignore-files, cursorignore-files, hierarchical-ignore, ignore-symlinks, cursor-tab, lsp, lsp-enabled, lsp-max-local-workspaces, lsp-max-remote-workspaces, lsp-worktrees-enabled)

### ED-8.31 Beta tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - mixed tab: the Update Access row is not obviously agent related, the Dedicated Agent Runtime row is.
- What: The tab titled Beta. It has an Update Access row (its options were not read) and a Dedicated Agent Runtime switch (runs Private Inference in a separate local process, needs a reload).
- Commands: aiSettings.action.open.beta
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (tab list `rPa`; titles `NDr`; settings registry entries beta/update-access and beta/dedicated-local-agent-runtime; string "Update Access"; the options inside the picker were not read)

### ED-8.32 Developer tab
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The tab titled Developer. It holds diagnostic rows: Tracing, Local Trace Mode and Continuous RPC File Recording. It is drawn only for builds that are not packaged, for extension development, or for people flagged as developers. What the traces record was not checked.
- Commands: aiSettings.action.open (with the tab argument developer)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (visibility function `mvw` branch for developer with the value `developerToolsEnabled`, built from `isBuilt`, `isExtensionDevelopment` and the dev-user server flag; settings registry entries developer/tracing, developer/local-trace, developer/continuous-rpc)

### ED-8.33 Open Customize
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the page manages the agent's plugins, tools, skills, subagents, rules, commands and hooks.
- What: Open Customize opens the Customize editor as a pinned tab, re-using one that is already open, and it comes back after a reload. A tab strip holds seven tabs (Plugins, MCPs, Skills, Subagents, Rules, Commands, Hooks, one entry each below). Each tab has a heading, a one-line description with a docs link, a search box, a scope menu, grouped lists (User, Team, Workspace, with sources such as plugin, dashboard and project) and an empty state; the search, scope, group and empty-state parts are read from module names and were not seen on screen. The Cursor Settings Customize row and the Cursor Settings tab ids plugins, rules, mcp and hooks open this page.
- Commands: workbench.action.openCustomizeEditor ("Open Customize"); workbench.action.customize.focusSearch ("Focus Customize Search")
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/contrib/customizeEditor/browser/customizeEditor.contribution; pane "Customize" id `workbench.editor.customize` with an editor serializer; command class registered as `V$n` with id `workbench.action.openCustomizeEditor`; modules Customize.js, CustomizeEngine.js, CustomizeManageView.js, CustomizeManageSearch.js, CustomizeManageScopeMenu.js, CustomizeManageGroups.js, CustomizeManageEmptyState.js; list structure read from module names and the constants `SXt`, `aUh`, `cUh` and `sUh` in CustomizeManage.utils.js)

### ED-8.34 Plugins tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the tab manages plugins that bundle agent rules, skills, subagents, commands, MCP servers and hooks.
- What: The Customize tab headed "Extend Cursor with Plugins". Its description says plugins bundle rules, skills, subagents, commands, MCP servers and hooks into one installable package, and it links to the plugins docs.
- Commands: workbench.action.customize.openPlugins ("Open Plugins")
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (CustomizeManage.utils.js tab list `SXt` and heading table `lIe`; customizeManageCommandDescriptors.js; customizeMigration.js map from the Cursor Settings tab id plugins to this tab)

### ED-8.35 MCPs tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the tab lists the MCP servers the agent can call.
- What: The Customize tab headed "Connect External Tools with MCP". Its description says Model Context Protocol servers connect Cursor to tools such as Linear, Figma and Notion, with a docs link. It has its own list view and a target filter (user, workspace or cloud).
- Commands: workbench.action.customize.openMCPs ("Open MCPs"); aiSettings.action.open.mcp (routes here; its `mcpTarget` argument picks the scope)
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (CustomizeManage.utils.js `lIe`; CustomizeManageMcp.js; routing function `$8o` in customizeCommandRouting.js; mcpFilterTabSelection.js default filter `cloud`; customizeMigration.js map mcp to mcps)

### ED-8.36 Skills tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the tab manages skills the agent applies.
- What: The Customize tab headed "Teach Cursor New Skills". Its description says skills package domain knowledge and workflows, with a docs link. A "Skill Publishing" entry sits in the editor title menu, and a separate command opens the skill publishing log.
- Commands: workbench.action.customize.openSkills ("Open Skills"); workbench.action.customize.openSkillPublishLogs ("Open Skill Publishing Logs"); cursor.publishSkill.copyLink
- Menus: EditorTitle (a "Skill Publishing" entry on the Customize editor title; the menu id was not resolved; the log opens as an output channel, which section 07 owns)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (CustomizeManage.utils.js `lIe`; customizeManageCommandDescriptors.js; module vs/workbench/contrib/customizeEditor/browser/skillPublishEditorTitle, nls 5464; logger `cursorSkillPublish` named "Skill Publishing")

### ED-8.37 Subagents tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the tab manages the subagents the agent can delegate to.
- What: The Customize tab headed "Delegate Work to Subagents" (tab id agents). Its description says subagents are specialized assistants that run in their own context window, with a docs link.
- Commands: workbench.action.customize.openSubagents ("Open Subagents")
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (CustomizeManage.utils.js `lIe` and `p8n`; customizeManageCommandDescriptors.js)

### ED-8.38 Rules tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the tab manages the rules the agent follows.
- What: The Customize tab headed "Guide Agent with Rules". Its description says rules give the agent persistent, system-level instructions, with a docs link. The Cursor Settings tab id rules (titled "Rules, Skills, Subagents") opens this tab, or the Commands or Skills tab when it carries a scroll target for them.
- Commands: workbench.action.customize.openRules ("Open Rules"); aiSettings.action.open.rules (routes here)
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (CustomizeManage.utils.js `lIe`; customizeManageCommandDescriptors.js; routing function `$8o` in customizeCommandRouting.js; customizeMigration.js map rules to rules)

### ED-8.39 Commands tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the tab manages reusable prompts the agent runs by name.
- What: The Customize tab headed "Create Reusable Commands". Its description says commands are reusable prompts invoked with /name, with a docs link. Opening a global command from its list shows the Global Command Preview (ED-10).
- Commands: workbench.action.customize.openCommands ("Open Commands")
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (CustomizeManage.utils.js `lIe` and `p8n`; customizeManageCommandDescriptors.js; global command preview strings recorded in the Cursor pass 1 file for ED-10)

### ED-8.40 Hooks tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the tab manages the scripts that run at the agent's lifecycle events.
- What: The Customize tab headed "Automate with Hooks". Its description says hooks run custom scripts at lifecycle events, with a docs link. It has its own view. A first-time team hooks notice has a View Hooks button that opens this tab.
- Commands: workbench.action.customize.openHooks ("Open Hooks"); aiSettings.action.open.hooks (routes here)
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (CustomizeManage.utils.js `lIe`; CustomizeManageHooks.js; customizeManageCommandDescriptors.js; customizeMigration.js map hooks to hooks; the notice with id prefix `cursorHooks/teamHooksFirstTimeNotification_` that calls `openPopup("hooks")`)

### ED-8.41 Show only modified settings
- Tag: both
- Evidence: source
- Boundary: no
- What: The Modified row in the Filter Settings drop-down (or typing `@modified` in the search box) shows only the settings the person has changed in the scope that is selected. Choosing the row again, or clearing the box, removes the filter.
- Commands: settings.action.suggestFilters (opens the drop-down that holds the row; the row itself is a toggle action with the id modifiedSettingsSearch, not a registered command)
- Menus: none found (the row is built in code in the Filter Settings drop-down)
- Keys: none found
- Settings: none found
- Cursor: present (the toggle action id `modifiedSettingsSearch` with the query `@modified` was found in the Filter Settings menu)
- Source: V\src\vs\workbench\contrib\preferences\browser\settingsSearchMenu.ts:120-135; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts:124,1310-1330; V\src\vs\workbench\contrib\preferences\browser\settingsTreeModels.ts:393,1232-1236; V\src\vs\workbench\contrib\preferences\common\preferences.ts:102; C bundle (module vs/workbench/contrib/preferences/browser/settingsSearchMenu, action `modifiedSettingsSearch`)

### ED-8.42 Open settings filtered to online services, telemetry or untrusted settings
- Tag: both
- Evidence: source
- Boundary: no
- What: File > Preferences > Online Services Settings opens the Settings editor with the search box set to `@tag:usesOnlineServices`, which lists the settings that call an online service. Two more commands with no menu row do the same for `@tag:telemetry` (Telemetry Settings) and for the Workspace scope settings that need Workspace Trust (Show untrusted workspace settings). Links inside other text blocks call them. In stock the first two reuse an open Settings editor by typing the query into its search box.
- Commands: settings.filterByOnline; settings.filterByTelemetry; settings.filterUntrusted
- Menus: MenubarPreferencesMenu (group 3_settings, order 1, the Online Services Settings row)
- Keys: none found
- Settings: telemetry.enableCrashReporter
- Cursor: changed (all three ids and the menu row are present; the online and telemetry handlers always open through the preferences service and do not check for an open Settings editor first; the telemetry id is also the target of a command link in a Cursor text block that links cursor.com/privacy)
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:68-72,476-545; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts (focusSearch); C bundle (module vs/workbench/contrib/preferences/browser/preferences.contribution; ids `settings.filterByOnline`, `settings.filterUntrusted`, `settings.filterByTelemetry`; menu MenubarPreferencesMenu group 3_settings order 1)

### ED-8.43 Move around the Settings editor with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: The Settings editor has its own focus path. Down in the search box moves to the first setting (when no suggestion list is open), Enter in the search box does the same, and Up from the first setting goes back to the search box. Left on a setting row moves to the contents tree, Enter on a tree row moves into the settings list, Enter on a setting row focuses its control, and Escape moves focus up one level (control, then row, then tree). Shift+F9 opens the row's gear menu.
- Commands: settings.action.focusSettingsFile; settings.action.focusSettingsFromSearch; settings.action.focusSettingsFromSearchOnEnter; settings.action.focusSearchFromSettings; settings.action.focusSettingsList; settings.action.focusTOC; settings.action.focusSettingControl; settings.action.focusLevelUp; settings.action.showContextMenu
- Menus: none found
- Keys: Down (settings.action.focusSettingsFile and settings.action.focusSettingsFromSearch, when the search box has focus and no suggestion is open); Enter (settings.action.focusSettingsFromSearchOnEnter, same condition); Up (settings.action.focusSearchFromSettings, when the first row has focus); Left (settings.action.focusTOC, when a row has focus); Enter (settings.action.focusSettingsList, when a tree row has focus); Enter (settings.action.focusSettingControl, when a row has focus); Escape (settings.action.focusLevelUp, when focus is not in the search box or a JSON editor); Shift+F9 (settings.action.showContextMenu)
- Settings: none found
- Cursor: changed (`settings.action.focusSettingsFile`, `focusSettingsFromSearch`, `focusSettingsList`, `focusTOC`, `focusSettingControl` and `focusLevelUp` and the Show Setting Context Menu command were found; `settings.action.focusSettingsFromSearchOnEnter` and `settings.action.focusSearchFromSettings` were not found)
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:58-72,645-880; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts (focusSearch, navigateSearchHistoryNextOrFocusSettings, focusTOC, showContextMenu); C bundle (module vs/workbench/contrib/preferences/browser/preferences.contribution; exact id searches for each command above)

### ED-8.44 Follow a setting link
- Tag: both
- Evidence: source
- Boundary: no
- What: A setting's description can name another setting, written as `#editor.fontSize#`, and the editor draws it as a link. Clicking it scrolls the list to that setting and moves focus to its control. If a search or a contents tree group hides the target, the editor first clears the group focus and, if needed, searches for `@id:<setting>` so the target shows. Other links in descriptions open as normal links, and command links run their command.
- Commands: none found (handled inside the editor)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the link event `onDidClickSettingLink` and the setting renderer module were found)
- Source: V\src\vs\workbench\contrib\preferences\browser\settingsTree.ts:816-822,1090-1110; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts:938-1001,1181; C bundle (module vs/workbench/contrib/preferences/browser/settingsTree, event `onDidClickSettingLink`)

### ED-8.45 Jump to the scope that also changes a setting
- Tag: both
- Evidence: source
- Boundary: no
- What: When a setting is also changed in one other scope, the label reads "Modified in" or "Also modified in" followed by the scope name (User, Workspace or Remote) as a link. When it is changed in several scopes or has language default overrides, the label reads "Modified elsewhere" and its hover lists each scope or language as a link. Clicking a link, or pressing Space or Enter on the inline one, switches the Settings Switcher to that scope, drops any language filter, applies the language filter when the link is a language, and filters the list to that one setting by its id. The hover of the "Managed by organization" label has a View policy settings link that applies the organization policy filter in the same way.
- Commands: none found (handled inside the editor; the policy link has the action id `_settings.action.viewPolicySettings`)
- Menus: none found
- Keys: Space or Enter (on the inline scope link)
- Settings: none found
- Cursor: present (the click event `onDidClickOverrideElement` and the indicator module were found)
- Source: V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts:1196-1210; V\src\vs\workbench\contrib\preferences\browser\settingsEditorSettingIndicators.ts:349-475; C bundle (module vs/workbench/contrib/preferences/browser/settingsEditorSettingIndicators, event `onDidClickOverrideElement`)

### ED-8.46 Review settings that need Workspace Trust
- Tag: both
- Evidence: source
- Boundary: no
- What: In an untrusted window, on the Workspace or Folder scope, a "Workspace Trust" group is added at the top of the contents tree when the workspace file or folder has configured settings that need trust. Its list shows those settings so the person can see which values are held back. Because Cursor ships Workspace Trust off by default (ED-8.18), the group rarely shows there.
- Commands: none found (the group is drawn from the model)
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.enabled
- Cursor: present (the contents tree entry with the id `workspaceTrust` and its label was found)
- Source: V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts:1588-1598; V\src\vs\workbench\contrib\preferences\browser\settingsTree.ts (resolveConfiguredUntrustedSettings); C bundle (module vs/workbench/contrib/preferences/browser/settingsEditor, tree group id `workspaceTrust`)

### ED-8.47 Edit entries of list, object and pattern settings
- Tag: both
- Evidence: source
- Boundary: no
- What: List settings (string arrays and the exclude and include pattern lists) show each entry as a row with an edit pencil and a remove cross, plus a reset icon on rows that differ from the default. Object settings show key and value columns with the same row icons. The edit icon turns the row into inline inputs. Rows of list settings can be dragged to a new place.
- Commands: workbench.action.editListItem; workbench.action.removeListItem; workbench.action.resetListItem (these are action ids on the row icons, not registered commands)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the three row action ids and the list row classes were found)
- Source: V\src\vs\workbench\contrib\preferences\browser\settingsWidgets.ts:480-545,985-1025; V\src\vs\workbench\contrib\preferences\browser\settingsTree.ts (SettingArrayRenderer, SettingObjectRenderer, SettingExcludeRenderer, SettingIncludeRenderer); C bundle (module vs/workbench/contrib/preferences/browser/settingsWidgets, ids `workbench.action.editListItem`, `workbench.action.removeListItem`, `workbench.action.resetListItem`)

### ED-8.48 Extension prompts in the Settings editor
- Tag: both
- Evidence: source
- Boundary: no
- What: When a search also matches settings of extensions that are not installed, a "Show matching extensions" button is added after the results and opens the extension search filtered to those extension ids (the extension search and the extension page belong to the extension system, which is left out). A second row type shows a recommended extension as its own group with Show Extension and Dismiss buttons; it needs experiment data and a gallery lookup, so it may never show. Where the first set of results comes from was not traced.
- Commands: workbench.extensions.action.showExtensionsWithIds; extension.open
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the row templates `settings.newExtensions.template` and `settings.extensionToggle.template` and the ids `workbench.extensions.action.showExtensionsWithIds` and `extension.open` were found; whether the experiment data reaches Cursor was not checked)
- Source: V\src\vs\workbench\contrib\preferences\browser\settingsTree.ts:1162-1200,2144-2196; V\src\vs\workbench\contrib\preferences\browser\settingsTreeModels.ts:1150-1163; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts:1483-1580; C bundle (module vs/workbench/contrib/preferences/browser/settingsTree, templates `settings.newExtensions.template` and `settings.extensionToggle.template`)

### ED-8.49 Settings Sync controls in the Settings editor header
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Settings Sync itself is outside the catalog (decision 9); this is only the part drawn inside the Settings editor. At the right of the Settings editor header, a "Backup and Sync Settings" button turns Settings Sync on while it is off. Once sync is on, or while it is working, the button is replaced by a "Last synced: <time>" label that refreshes every minute.
- Commands: workbench.userDataSync.actions.turnOn
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing (no match for "Backup and Sync Settings", the class `settings-right-controls`, the class `turn-on-sync` or `workbench.userDataSync.actions.turnOn` in the bundle)
- Source: V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts:2262-2338

### ED-8.50 Not synced marker and Sync This Setting
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Settings Sync itself is outside the catalog (decision 9); this is only the marker and the row action drawn on a setting. With Settings Sync on, a setting on the sync ignore list shows a "Not synced" label whose hover says "This setting is ignored during sync". The setting's gear menu has a checked Sync This Setting row that adds or removes the setting in the ignore list. The row is left out for settings that do not allow sync to be ignored.
- Commands: settings.stopSyncingSetting
- Menus: none found (a row in the setting's gear menu)
- Keys: none found
- Settings: settingsSync.ignoredSettings
- Cursor: missing (no match for the class `setting-item-ignored`, "Not synced" as a setting label or "Sync This Setting"; the setting `settingsSync.ignoredSettings` is still registered)
- Source: V\src\vs\workbench\contrib\preferences\browser\settingsEditorSettingIndicators.ts:152-166,293-302,590-597; V\src\vs\workbench\contrib\preferences\browser\settingsTree.ts:2288-2294,2800-2845

### ED-8.51 Pick a value from the gutter in a settings JSON file
- Tag: both
- Evidence: source
- Boundary: no
- What: In the user or workspace settings.json, a small edit icon appears in the glyph margin on the line of a boolean or enum setting, under the text cursor or the mouse. Clicking it opens a menu with true and false, or the enum values. Choosing one writes the value, selects it and highlights it. When two settings share a line each one gets a sub menu. Settings held by an organization policy are skipped, and the icon does not show if the glyph margin is off.
- Commands: none found (the menu rows have the action ids truthyValue, falsyValue and the enum value)
- Menus: none found (a context menu opened from the gutter icon)
- Keys: none found
- Settings: editor.glyphMargin
- Cursor: present (the actions `truthyValue` and `falsyValue` and the edit widget code were found)
- Source: V\src\vs\workbench\contrib\preferences\browser\preferencesRenderers.ts:60-150,175-452; V\src\vs\workbench\contrib\preferences\browser\preferencesWidgets.ts (EditPreferenceWidget); C bundle (module vs/workbench/contrib/preferences/browser/preferencesRenderers, actions `truthyValue` and `falsyValue`)

### ED-8.52 See warnings for settings that cannot apply in a settings JSON file
- Tag: both
- Evidence: source
- Boundary: no
- What: In settings.json and in the settings part of a .code-workspace file, an entry that cannot take effect gets a squiggle and a message. The messages cover settings set by system policy, an override block for a setting that cannot be overridden by language, a setting that cannot apply while a non-default profile is active or is set to apply in all profiles, window or machine or application scope settings in the wrong file or window, settings that need a trusted workspace, unknown settings, and unsupported properties in a workspace file. The trust case has a Manage Workspace Trust quick fix.
- Commands: workbench.trust.manage (the quick fix on an untrusted-setting warning)
- Menus: none found (markers and the Quick Fix lightbulb)
- Keys: none found
- Settings: none found
- Cursor: present (the marker owner `UnsupportedSettingsRenderer` was found; the message texts are stored as numbered strings, so the wording was not matched)
- Source: V\src\vs\workbench\contrib\preferences\browser\preferencesRenderers.ts:487-790,912-945; C bundle (module vs/workbench/contrib/preferences/browser/preferencesRenderers, marker owner `UnsupportedSettingsRenderer`)

### ED-8.53 Show default settings beside the settings JSON
- Tag: both
- Evidence: source
- Boundary: no
- What: With `workbench.settings.openDefaultSettings` or `workbench.settings.useSplitJSON` on, opening a settings JSON file opens a side-by-side editor with the read-only default settings on the left and the file on the right. Open Default Settings (JSON) opens the read-only defaults alone from the Command Palette.
- Commands: workbench.action.openRawDefaultSettings
- Menus: CommandPalette
- Keys: none found
- Settings: workbench.settings.openDefaultSettings; workbench.settings.useSplitJSON
- Cursor: present (the command, both settings and the split-open method `doOpenSplitJSON` were found; the stock "Preferences Editor" pane id `workbench.editor.preferences` was not found and the split uses a plain side-by-side input)
- Source: V\src\vs\workbench\services\preferences\browser\preferencesService.ts:410-445; V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:325-336; V\src\vs\workbench\browser\workbench.contribution.ts:546-556; C bundle (settings `workbench.settings.openDefaultSettings` and `workbench.settings.useSplitJSON`, method `doOpenSplitJSON`)

### ED-8.54 Open keyboard shortcuts as JSON
- Tag: both
- Evidence: source
- Boundary: no
- What: A button in the title bar of the Keyboard Shortcuts editor opens `keybindings.json`, creating it with a short comment and an empty array when it is missing. In that file a matching title button, Open Keyboard Shortcuts, goes back to the table. The Command Palette also has Open Default Keyboard Shortcuts (JSON), a read-only list of every default binding. With `workbench.settings.openDefaultKeybindings` on, opening `keybindings.json` also opens the defaults on its left.
- Commands: workbench.action.openGlobalKeybindingsFile; workbench.action.openDefaultKeybindingsFile; workbench.action.openGlobalKeybindings (the button back to the table)
- Menus: EditorTitle (group navigation, Open Keyboard Shortcuts (JSON) when the Keyboard Shortcuts editor is active, Open Keyboard Shortcuts when the user keybindings file is active); ModalEditorEditorTitle (the same two rows, stock only); CommandPalette
- Keys: none found
- Settings: workbench.settings.openDefaultKeybindings
- Cursor: changed (both open commands and the setting were found; the ModalEditorEditorTitle rows were not found)
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:880-968; V\src\vs\workbench\services\preferences\browser\preferencesService.ts:342-373; C bundle (ids `workbench.action.openGlobalKeybindingsFile`, `workbench.action.openDefaultKeybindingsFile`, setting `workbench.settings.openDefaultKeybindings`)

### ED-8.55 Define a keybinding in keybindings.json
- Tag: both
- Evidence: source
- Boundary: no
- What: With the user `keybindings.json` active, Ctrl+K Ctrl+K opens a key-capture box over the editor that says "Press desired key combination and then press ENTER" and tells how many existing commands already use the combination. Enter inserts a `{ "key": ..., "command": ..., "when": ... }` snippet at a fitting place in the array, with the command id and the when clause as tab stops.
- Commands: editor.action.defineKeybinding
- Menus: EditorContent (when the resource is the user keybindings file); CommandPalette
- Keys: Ctrl+K Ctrl+K (editor.action.defineKeybinding, when the resource is the user keybindings file)
- Settings: none found
- Cursor: present (the id `editor.action.defineKeybinding`, the widget id `editor.contrib.defineKeybinding` and the key-capture module were found)
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:1287-1312; V\src\vs\workbench\contrib\preferences\browser\keybindingsEditorContribution.ts:36-95; V\src\vs\workbench\contrib\preferences\browser\keybindingWidgets.ts:135-260; V\src\vs\workbench\contrib\preferences\common\smartSnippetInserter.ts; C bundle (id `editor.action.defineKeybinding`, contribution id `editor.contrib.defineKeybinding`)

### ED-8.56 Keyboard layout hints in keybindings.json
- Tag: both
- Evidence: source
- Boundary: no
- What: In the user `keybindings.json` each "key" value is checked against the current keyboard layout. When the text does not match what the layout produces, the value gets an info mark and its hover says "<keys> for your current keyboard layout" (with the US keys added when they differ). When the combination cannot be typed on the layout at all, the value gets an error mark and the hover says so. Marks also show in the overview ruler.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the decoration classes `keybindingInfo` and `keybindingError` and the decoration code were found)
- Source: V\src\vs\workbench\contrib\preferences\browser\keybindingsEditorContribution.ts:97-245; C bundle (module vs/workbench/contrib/preferences/browser/keybindingsEditorContribution, classes `keybindingInfo` and `keybindingError`)

### ED-8.57 Troubleshoot keyboard shortcuts
- Tag: both
- Evidence: source
- Boundary: no
- What: Toggle Keyboard Shortcuts Troubleshooting (Command Palette, Developer category) turns key logging on or off. When it turns on it opens the Window log in the Output panel (section 07 owns the panel), and every key press flashes a red box for 800 milliseconds around the element that received the key. Running the command again stops both.
- Commands: workbench.action.toggleKeybindingsLog; workbench.action.showWindowLog (run when logging turns on)
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (the id `workbench.action.toggleKeybindingsLog`, its title and the marker class `focus-troubleshooting-marker` were found)
- Source: V\src\vs\workbench\contrib\keybindings\browser\keybindings.contribution.ts:20-112; V\src\vs\workbench\services\log\common\logConstants.ts:11; C bundle (id `workbench.action.toggleKeybindingsLog`, title "Toggle Keyboard Shortcuts Troubleshooting", class `focus-troubleshooting-marker`)

### ED-8.58 Import settings from VS Code
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: In the Preferences group of the General tab, the Import Settings from VS Code row has an Import button. Pressing it opens a small confirm box that says "Are you sure? This replaces your settings and extensions with your vscode ones. This cannot be undone." with Cancel and Confirm. Confirm runs the import. The button reads Importing while it works and Success! for a few seconds after, and a notification says "VS Code import completed!" or "Error importing extensions, please try again." The row text also mentions keybindings; the confirm text does not.
- Commands: cursorai.action.importvscode
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (General tab row component with label "Import Settings from VS Code", settings registry entry general/import-vscode with surface ide, constant `Tkn` = `cursorai.action.importvscode` called with the argument true, confirm text constant `uww`, timer constant `dww` of 5000 ms)

### ED-8.59 Reset "Don't Ask Again" dialogs
- Tag: cursor
- Evidence: bundle
- Boundary: yes - two of the listed entries restore agent warnings (About Allowlist, Run Everything Mode Warning); the dialogs themselves belong to section 12.
- What: In the Preferences group of the General tab, the row Reset "Don't Ask Again" Dialogs ("See warnings and tips that you've hidden") has a Show or Hide button. Show opens a "Hidden dialogs" list with one row per dialog the person marked "don't ask again", each with a Restore button, and a Reset All button below. With nothing hidden it says "No Hidden Dialogs Yet" and explains that hidden dialogs will appear there.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component beside the General tab row, label constant `pww`, empty text constant `hww`, list of two agent warnings `Edg`, storage key `dialogDontAskAgainPreferences`, settings registry entry general/reset-dialogs)

### ED-8.60 Run network diagnostics
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: On the Browser & Network tab, the Network Diagnostics row ("Check network connectivity to all Cursor services") has a Run Diagnostic button that shows a spinner and the text Running Diagnostic while it works. A results panel opens under the row with one line per check, each with its own log lines, a Show Logs or Hide Logs button and a Copy results button.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (network tab component with labels "Network Diagnostics", "Run Diagnostic", "Running Diagnostic", "Show Logs", "Hide Logs", "Copy results"; the diagnostic runner that logs `NetworkDiagnostics[...]` lines; settings registry entry browser/diagnostics)

### ED-8.61 Search Cursor Settings
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A search box at the top of the Cursor Settings sidebar reads "Search settings" and shows the key for the focus command after it. While it is empty, Up and Down move a highlight through the tabs and Enter opens the highlighted tab. Once something is typed the tab list is replaced by a result list across all tabs, each result with the tab name on its right, or "No results found." Up, Down and Enter pick a result, which opens its tab and scrolls to the item; a result that stands for another page, such as VS Code Settings, closes Cursor Settings and runs its command. Escape leaves the box. The box is hidden when the pane is narrow and the sidebar shrinks to icons.
- Commands: aiSettings.action.focusSearch ("Focus AI Settings Search")
- Menus: none found
- Keys: Ctrl+F (aiSettings.action.focusSearch, while the Cursor Settings pane has focus)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (sidebar component with the input class `cursor-settings-sidebar-search-input`, placeholder "Search settings", result list with "No results found.", constant `ids` = `aiSettings.action.focusSearch`, the pane component that sets compact mode at a width of 710 px or less)

### ED-8.62 Customize banner on the General tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the banner leads to Customize, which manages the agent's plugins, MCPs, skills and rules.
- What: Above the General tab a banner says "Plugins, MCPs, Skills, and Rules have moved to Customize" with the line "We've introduced a new home for all the ways to customize Cursor." It has a Dismiss button and an Open Customize button that opens the Customize editor on its Skills tab and also dismisses the banner. It shows only on the General tab and only until dismissed.
- Commands: workbench.action.openCustomizeEditor (reached by the Open Customize button)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (banner component with the class `cursor-settings-inline-banner`, title constant `rug`, show condition function `Zfw` for the general tab and the stored flag `dismissedGeneralCustomizeBanner`, view constant `Qfw` for the skills tab, open helper `gOe` that runs `workbench.action.openCustomizeEditor` with the argument `initialView`)

### ED-8.63 Account header in the Cursor Settings sidebar
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: When the person is signed in with a known email, the top of the Cursor Settings sidebar shows a round avatar with the first letter of the email, the email and the plan name. When the pane is narrow (710 px or less) and the sidebar shrinks to icons, hovering the header shows the email and plan in a tooltip.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (sidebar header with classes `cursor-settings-sidebar-header`, `cursor-settings-sidebar-avatar`, `cursor-settings-sidebar-header-email` and `cursor-settings-sidebar-header-plan`; plan label function `TDm`; compact mode flag set at 710 px)

### ED-8.64 Upgrade and referral prompts in the Cursor Settings sidebar
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The plan and referral modals these open belong to DL-6. Below the sidebar tabs, a signed-in person on the free plan sees an upgrade button, "Upgrade to Pro" or the label of the next upgrade step, that starts the upgrade flow. Anyone else who is eligible for the referral program sees a gift pill with the reward text, an X that appears on hover to dismiss it for good, and a click that opens a referral link box. Both prompts are hidden while the sidebar search box has text, and in the narrow icon sidebar they show only their icon.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (upgrade cell component with class `cursor-settings-upgrade-cta-cell`, referral cell component with class `cursor-settings-referral-cta-cell`, stored flag `dismissedP2PReferralCta`, tracking names `settings_sidebar.upgrade_pro_clicked` and `p2p_referral.cta_pill_clicked`)

### ED-8.65 Create a new item in a Customize tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the items are the agent's skills, subagents, rules, commands and MCP servers.
- What: In the Skills, Subagents, Rules and Commands tabs, each scope group other than Team and Plugin has a New button that creates an item of that tab's kind in that scope (for example user or workspace). In the MCPs tab a New button creates a server in the account, user or workspace target, and it becomes a menu when more than one target is available. What happens after the click (a new file, or a chat) was not read.
- Commands: workbench.action.newCursorRule; workbench.action.newProjectCursorCommand; workbench.action.newUserCursorCommand
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (Customize group builder with `addLabel:"New"`, `createKind` and `createScope`; the create callback `onCreatePrimitive`; MCP targets dashboard, user and project; modules CustomizeManageGroups.js and CustomizeManageMcp.js)

### ED-8.66 Add a plugin
- Tag: cursor
- Evidence: bundle
- Boundary: yes - plugins bundle agent rules, skills, subagents, commands, MCP servers and hooks.
- What: In the Plugins tab the Installed group has an Add button with a menu named Add plugin source: From Marketplace, From GitHub Repository and From Local Repository. From Marketplace opens the Customize browse view. The two repository rows show only when the page supplies them; without them Add goes straight to the browse view.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (Customize group with `title:"Installed"`, `addLabel:"Add"` and `removable:true`; add menu component with labels "From Marketplace", "From GitHub Repository", "From Local Repository" and aria label "Add plugin source"; handlers `onBrowse`, `onAddPluginFromGitHubRepo` and `onAddPluginFromLocalRepo`)

### ED-8.67 Uninstall a plugin
- Tag: cursor
- Evidence: bundle
- Boundary: yes - plugins bundle agent rules, skills, subagents, commands, MCP servers and hooks.
- What: The Installed group of the Plugins tab has an Edit button that switches the list into an edit mode and reads Done while it is on. In that mode installed plugins can be removed. The handler names show uninstall can be from the whole install, from one project or from one scope. The controls shown in the edit mode were not read.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (group action bar with the labels "Edit" and "Done" shown when the group is removable and `onUninstallPlugin` exists; handlers `handleUninstall`, `handleUninstallFromProject` and `handleUninstallScope`)

### ED-8.68 Configure runtime arguments
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Preferences: Configure Runtime Arguments to open the argv.json file, where startup options such as the display language, hardware acceleration and log level are set.
- Commands: workbench.action.configureRuntimeArguments
- Menus: CommandPalette (Preferences category)
- Keys: none found
- Settings: none found
- Cursor: present - the command and its label (nls 11997) are in the bundle; the argv schema was not checked
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:52-72; V\src\vs\workbench\electron-browser\desktop.contribution.ts:404-488 (argv schema vscode://schemas/argv); C bundle (id workbench.action.configureRuntimeArguments, category Preferences)

### ED-8.69 Create a rule from the selected text
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the rule it saves is guidance the agent follows, though the command itself does not call a model.
- What: Select text in an editor, right-click and choose Create Rule to save it as a new manual rule file in the workspace's .cursor/rules folder, then use the Open Rule button on the notice to open it.
- Commands: cursor.createRuleFromSelection; cursor.openCreatedRule
- Menus: EditorContext (group navigation, order 20, only when the editor has a non-empty selection)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (action class with id cursor.createRuleFromSelection, title Create Rule, description "Create a cursor rule from the highlighted text", menu EditorContext group navigation order 20, precondition hasNonEmptySelection; rule service createRule(name, text, false); notification action id cursor.openCreatedRule with label Open Rule and tooltip "Open the generated rule file"; name helpers that cut to 12 characters and add a numeric suffix). Pass 1 notes list cursor.createRuleFromSelection as a chat entry point and the coverage script put it in its agentish list, so it is in no work file; it is added here as a related id.

### ED-8.70 Proxy and certificate settings
- Tag: both
- Evidence: source
- Boundary: no
- What: The person sets how the app's own web requests reach the internet - a proxy address, hosts that skip it, proxy credentials and certificate checks - in the Settings editor under Application > Proxy.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: http.proxy; http.noProxy; http.proxyAuthorization; http.proxyKerberosServicePrincipal; http.proxyStrictSSL; http.systemCertificates; http.systemCertificatesNode
- Cursor: changed - http.proxy, http.noProxy, http.proxyAuthorization, http.proxyKerberosServicePrincipal, http.proxyStrictSSL and http.systemCertificates are registered in the bundle; http.systemCertificatesNode was not found; Cursor also has its own Browser & Network tab (ED-8.29) for HTTP Compatibility Mode
- Source: V\src\vs\platform\request\common\request.ts (lines 236-361); V\src\vs\platform\request\node\requestService.ts (lines 85-143); V\src\vs\platform\windows\electron-main\windowImpl.ts (lines 1114-1132); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (lines 389-390); C bundle (the http configuration node with the six ids above, 0 hits for http.systemCertificatesNode)

### ED-8.71 Key matching settings
- Tag: both
- Evidence: source
- Boundary: no
- What: The person chooses how key presses are matched to shortcuts and whether AltGraph counts as Ctrl+Alt, in the Settings editor under Application > Keyboard.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: keyboard.dispatch; keyboard.mapAltGrToCtrlAlt
- Cursor: present - both ids were found in the bundle and nls 2118 holds the dispatch description; keyboard.layout (the web-only layout setting, mapped to SS-3.38) was not found
- Source: V\src\vs\platform\keyboardLayout\common\keyboardConfig.ts (lines 21-52); V\src\vs\workbench\services\keybinding\electron-browser\nativeKeyboardLayout.ts (lines 40-70); V\src\vs\workbench\services\keybinding\common\windowsKeyboardMapper.ts (lines 414-415); V\src\vs\workbench\services\keybinding\common\macLinuxKeyboardMapper.ts (lines 992-993); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (line 395); C bundle (ids keyboard.dispatch and keyboard.mapAltGrToCtrlAlt, nls 2118)

### ED-8.72 Completions and checks in keybindings.json
- Tag: both
- Evidence: source
- Boundary: no
- What: While editing the user `keybindings.json`, the person gets completions for command ids, hover descriptions and error marks for bad entries, and in stock can add `"systemWide": true` to make a key work even when the app is not focused.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - the schema (id vscode://schemas/keybindings, title "Keybindings configuration"), its field descriptions, the multiple-commands error text and the configuration-editing extension that ties it to `keybindings.json` were found; the `systemWide` property, its two warning notifications and the code that registers the shortcuts have no hit in the bundle
- Source: V\src\vs\workbench\services\keybinding\browser\keybindingService.ts (class KeybindingsJsonSchema, lines 836-1016); V\extensions\configuration-editing\package.json (jsonValidation entries for keybindings.json); V\src\vs\workbench\contrib\keybindings\electron-browser\systemWideKeybindings.contribution.ts; V\src\vs\workbench\contrib\keybindings\electron-browser\systemWideKeybindings.specification.md; V\src\vs\platform\globalKeybindings\electron-main\globalKeybindingsMainService.ts; C bundle (schema id and nls strings, C extensions\configuration-editing\package.json; 0 hits for systemWide)

## Handed off

- Keyboard layout picker, command `workbench.action.openKeyboardLayoutPicker` titled "Change Keyboard Layout", opened from a "Layout: {0}" status item - SS-3 (low confidence, the item is not tied to the active editor). Stock only, the bundle has no match.
- Manage menu rows Profiles and Themes (GlobalActivity) - AB-4. File > New Window with Profile and Share > Export Profile rows - TB-1.
- Restricted Mode status item and the trust notice strip - SS-4 and BN-2. Trust prompts and the Manage Workspace Trust entry in dialogs - DL-5.
- Cursor Settings gear button in the title bar - TB-5. Plan, pricing and upgrade modals opened from Plan & Usage - DL-6.
- The Cursor Tab on or off row inside the Code Intelligence tab - AC-8.
- The Skill Publishing log, which opens as an output channel - the Panel section (07), no group named.
- Explorer right-click row Open Folder Settings (`_workbench.action.openFolderSettings`) - SB group that owns the Explorer context menu. It is listed here only as an entry point of ED-8.3.
- The Window log that Toggle Keyboard Shortcuts Troubleshooting opens (ED-8.57) shows as an output channel - the Panel section (07), no group named.
- The referral link box and the upgrade flow opened from the sidebar prompts (ED-8.64) - DL-6.
- Four Developer-category commands in the Customize editor, all `f1` and gated by a developer condition: `workbench.action.customize.clearPrimitiveAuthorshipCache`, `workbench.action.customize.clearPrimitiveSourceCache`, `workbench.action.customize.forceSkillMigrationBanner` and `workbench.action.customize.clearForcedSkillMigrationBanner` - the Developer-category commands list in the catalog (no group named). Debug tools, not user features.
- The "import from VS Code" toast during first-run onboarding (storage key `onboarding.importFromVSCodeToastEligibleAt`) - ED-9 or the first-run onboarding owner (low confidence, only the key was seen). The Import button in Cursor Settings is ED-8.58.

## Unclear

- Tab count for Cursor Settings. The tab id list has 18 ids, but only 13 are drawn in the IDE, so ED-8.20 to ED-8.32 are the drawn tabs (General, VS Code Settings, Plan & Usage, Agents, Cloud Agents, Models, Voice, Source Control, Customize, Browser & Network, Code Intelligence, Beta, Developer). The other 5 were not given their own feature: Appearance is drawn only when a flag the IDE call sets to false is on (the one caller in the IDE bundle passes `appearanceEnabled: false`), and its rows are marked for the Agents window, which is out of scope and was not read; Plugins, Rules, Tools & MCPs and Hooks are never drawn as tabs and route to the Customize tabs, so they sit as entry points on ED-8.34, ED-8.35, ED-8.38 and ED-8.40. Four more ids (fun, network, worktrees, tab) are not in the 18-id list and exist only in the title map, where they redirect to appearance, browser, git-prs and indexing. If one entry per id is wanted anyway, add the five as ED-8.41 and up so the existing ids stay put (confidence high, ids never change once assigned).
- Some Cursor Settings tabs are drawn only under conditions: Plan & Usage needs a signed-in account with on-demand spillover, Voice needs a feature gate, Developer needs a dev build or a dev flag. Which of these a given person sees was not checked on screen.
- The 40-feature cap forced merges. The Modified filter is inside ED-8.2, the sync indicator is inside ED-8.6, and settings JSON and shortcuts JSON share ED-8.8, as the catalog example does. Seen but not given a feature: Toggle Keyboard Shortcuts Troubleshooting (`workbench.action.toggleKeybindingsLog`, present in Cursor); the settings editor keyboard navigation commands (focus the list, the control, up one level) beyond what ED-8.4 and ED-8.5 name; the Extensions group's "new extensions" prompt row in the Settings editor (`SettingNewExtensionsRenderer`).
- The Settings editor header sync controls ("Backup and Sync Settings" button, "Last synced" label) and the row action "Sync This Setting" belong to Settings Sync, which the catalog left out (decision 9). They are named in ED-8.5 and ED-8.6 only as stock items Cursor lacks.
- `cursor.openCursorSettings`, which the first pass listed as a command, has one hit in the bundle and it is a caller in the Cloud Agents tab. No registration was found, and the caller passes `settingsTab` where the registered open command reads `openTab`. It is not listed as an entry point.
- Pass 1 left it open whether Cursor Settings is an editor tab or an overlay. The bundle shows the command calls `openPopup`, and a workbench contribution answers by opening the editor input `workbench.editor.aisettings.input` as a pinned editor in the last editor group. The internal word popup is only a name. This was read in strings, not run.
- The stock Settings row in File > Preferences is shown only when context `yI` is true and "VS Code Settings" when it is false. `yI` is read as the Agents window context because Ctrl+, uses its negation for Cursor Settings. The name was not resolved. The extra condition on Ctrl+Shift+, (`dn.focus` negated) was not resolved either.
- Setting ids for Cursor Settings and Customize rows were not found: the tab values appear to live in Cursor's own storage, not in the configuration registry. Their Settings fields say `none found`. Where the values are stored was not traced.
- The theme pickers carry Install and Browse rows that lead to the Marketplace. That flow is the extension system and is left out.
- Cursor ships Workspace Trust off by default (catalog), and a team admin value `workspaceTrustControls.enabled` can turn the setting on. How that shows in the editor was not checked.
- Nothing in ED-8 is newer than 1.128. The quoted command and setting ids under `preferences`, `userDataProfile`, `workspace\browser` and `themes` in N match V except that V has one extra unused constant (workbench.preferences.action.openPreferencesEditor), and the file lists match.
- Not checked: how any page looks on screen, the options inside the Update Access picker, what the Developer tab traces record, and where the usage bar that opens Plan & Usage sits.
- Exact id searches of the bundle found neither `settings.action.showPreviousSearch` nor `settings.action.focusSettingsFromSearchOnEnter`, which ED-8.2 and ED-8.43 name for stock. The bundle has a method named `showPreviousSearchQuery`, so the Up-arrow search history probably exists in another form. ED-8.2 was left as written.
- Seen in the overflow check and not given a feature: the Customize editor shows gated "leaderboard" code (client gate `customize_page_leaderboard_enabled`, default false) and per-skill actions named `onPublishSkill`, `onCopySkillLink`, `onSyncSkill` and `onUnpublishSkill`, but no labels or on-screen form were found, so they stay inside ED-8.36; the stock commands `_developer.getConfigurationInformation` and `_getAllSettings` are internal and have no screen; the MCP warning drawn in settings.json (`McpSettingsRenderer`) is MCP, which is out of scope; the Cursor Settings General tab rows Layout, Startup, Notifications, Status Bar and Tips are setting rows, not features; the five tab ids listed in the first Unclear line were not added as features (Appearance is Agents window only, the other four only route to Customize tabs).
- Overflow check on 2026-10-25 added 27 features (ED-8.41 to ED-8.67).
