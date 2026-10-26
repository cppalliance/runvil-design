Group: PN-5 Output and logs. Section: Panel (PN). Date: 2026-10-25. Pass: 2.
Features: 20.

Short names used in Source: V is the stock 1.128.0 tree, N is the 1.144.0 clone, C bundle is `workbench.desktop.main.js` in the Cursor install, C ext is the `extensions` folder of the Cursor install, and NLS is `out\nls.messages.json` next to the bundle. Unless a path says otherwise, V paths below start at `V\src\vs\workbench\contrib\output\browser\` (output) or `V\src\vs\workbench\contrib\logs\` (logs). The view id is `workbench.panel.output`, so the filter toggle command ids below are built from it (V builds them from a template, so the full ids do not appear as plain strings in V or in the C bundle). Line numbers are from the files as read and can drift by a line or two. Cursor findings come from strings, compiled code and extension files only. Nothing was run on screen. N has the same Output and logs command ids as V, so nothing here is `newer-than-1.128`.

### PN-5.1 Open, toggle and focus the Output view
- Tag: both
- Evidence: source
- Boundary: no
- What: Show the Output tab, toggle it, or move focus into it from the View menu (Output), from Ctrl+Shift+U (Linux: Ctrl+K Ctrl+H), or from the Command Palette. The view shows the channel last used in this workspace, and the first registered channel when none was chosen yet.
- Commands: workbench.action.output.toggleOutput; workbench.panel.output.view.focus
- Menus: MenubarViewMenu (group 5_panel, order 1, generated from the view's open command); CommandPalette
- Keys: Ctrl+Shift+U (Linux: Ctrl+K Ctrl+H)
- Settings: none found
- Cursor: present (toggleOutput with the same key and the Linux chord found; the generated focus command comes from the registerFocusViewAction code that Cursor still has)
- Source: output.contribution.ts (72-102); outputServices.ts (307-395); V\src\vs\workbench\services\views\browser\viewsService.ts (471-575); research entry-points 01-title-bar (View menu row Output, 5_panel order 1); C bundle (Output container with storageId and order 1, view with openCommandActionDescriptor and keybindings); NLS 8533-8535

### PN-5.2 Switch the channel from the drop-down
- Tag: both
- Evidence: source
- Boundary: no
- What: The Output title row has a drop-down that shows the current channel name. Open it to pick another channel, with a check on the active one. Channels are grouped: extension channels first, then built-in ones, then user channels (imported or combined logs), with separators. New channels appear in the list while Output is open, and removed ones disappear. The chosen channel is remembered for the workspace.
- Commands: workbench.output.action.switchBetweenOutputs; workbench.action.output.show.<channel id>
- Menus: ViewTitle (submenu workbench.output.menu.switchOutput, group navigation, order 1, shows the selection); workbench.output.menu.switchOutput (groups 0_ext_outputchannels, 1_core_outputchannels, 2_user_outputchannels)
- Keys: none found
- Settings: none found
- Cursor: present (switchBetweenOutputs, the submenu id, the three group names and the per-channel show id template all found; the channel descriptors also have a workspaceId in Cursor)
- Source: output.contribution.ts (133-191); outputServices.ts (307-345, 404-415); outputView.ts (128-136); V\src\vs\workbench\services\output\common\output.ts (descriptor types); C bundle (registerSwitchOutputAction); NLS 8493-8494

### PN-5.3 Pick a channel from a list
- Tag: both
- Evidence: source
- Boundary: no
- What: Three Command Palette commands open a quick pick of channels. Show Output Channels lists every channel (extension channels first, then built-in ones). Show Logs lists only log channels, with extension logs under a separator. Show Window Log jumps straight to the Window log. Choosing an entry opens Output on that channel. Show Logs and Show Window Log are in the Developer category.
- Commands: workbench.action.showOutputChannels; workbench.action.showLogs; workbench.action.showWindowLog
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (all three ids found; labels Show Output Channels..., Show Logs..., Show Window Log and the placeholders Select Output Channel and Select Log found in NLS)
- Source: output.contribution.ts (272-308, 532-573); V\src\vs\workbench\contrib\logs\common\logs.contribution.ts (182-198); V\src\vs\workbench\services\log\common\logConstants.ts (9-11); C bundle (registerShowOutputChannelsAction, registerShowLogsAction, ShowWindowLogAction); NLS (labels matched by text)

### PN-5.4 Set the log level
- Tag: both
- Evidence: source
- Boundary: no
- What: For a log channel the Output title has a gear menu (Set Log Level...) with Trace, Debug, Info, Warning, Error and Off, a check on the current level, and Set As Default, which is enabled when the level differs from the default. The Command Palette command Set Log Level... (Developer) opens a picker with an All section that sets the level for everything, then every log channel, with a button on each level to make it the default. The telemetry log cannot be set.
- Commands: workbench.action.output.activeOutputLogLevel.<level>; workbench.action.output.activeOutputLogLevelDefault; workbench.action.setLogLevel; workbench.action.setDefaultLogLevel
- Menus: ViewTitle (submenu workbench.output.menu.logLevel, group navigation, order 6, gear icon, shown only when the active channel's level can be set); workbench.output.menu.logLevel (groups 0_level and 1_default); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (the gear submenu id, both level id prefixes, setLogLevel and setDefaultLogLevel found; labels Set Log Level..., Set As Default and Set Default Log Level found in NLS)
- Source: output.contribution.ts (455-530); outputServices.ts (433-462); V\src\vs\workbench\contrib\logs\common\logsActions.ts (24-158); logs\common\logs.contribution.ts (27-51); C bundle (registerConfigureActiveOutputLogLevelAction, SetLogLevelAction); NLS (labels matched by text)

### PN-5.5 Filter the output text
- Tag: both
- Evidence: source
- Boundary: no
- What: Type in the filter box in the Output header to hide lines that do not match, with the matches highlighted. Stock also takes several comma separated terms (a line shows if any matches), a leading ! to exclude, and quoted terms. Escape in the box clears it. For log channels the filter works on whole log entries, and for plain channels on single lines. The text is saved per workspace.
- Commands: workbench.actions.workbench.panel.output.clearFilterText; output.editor.contrib.filterController
- Menus: none found
- Keys: Escape (when the filter box is focused)
- Settings: none found
- Cursor: changed (the box, the clear command and Escape are present, but the compiled filter matches one plain text only, with no comma list or ! exclusion, and the placeholder is just "Filter" where stock says "Filter (e.g. text, !excludeText, text1,text2)")
- Source: outputView.ts (88-135, 357-560); outputServices.ts (134-182); output.contribution.ts (693-711); V\src\vs\workbench\browser\parts\views\viewFilter.ts (60-235); C bundle (output filter controller with a single findMatches call on the filter text; the placeholder is NLS 8537); NLS 8537

### PN-5.6 Filter log lines by level
- Tag: both
- Evidence: source
- Boundary: no
- What: When a log channel is open, the More Filters button (filter icon) next to the filter box lists Trace, Debug, Info, Warning and Error as toggles. Untick one to hide the log entries of that level. The button gets a highlight when any level is hidden, and the choices are saved per workspace. The entries do not show for plain channels.
- Commands: workbench.actions.workbench.panel.output.toggle.trace; workbench.actions.workbench.panel.output.toggle.debug; workbench.actions.workbench.panel.output.toggle.info; workbench.actions.workbench.panel.output.toggle.warning; workbench.actions.workbench.panel.output.toggle.error
- Menus: submenu.view.filter (More Filters, group 2_log_filter, only while the active channel is a log file)
- Keys: none found
- Settings: none found
- Cursor: present (group 2_log_filter found, the five filter context keys output.filter.trace to output.filter.error found, and the label "Show or hide {0} messages in the output" found in NLS)
- Source: output.contribution.ts (641-691); outputView.ts (196-200, 480-552); V\src\vs\workbench\services\output\common\output.ts (filter context keys); C bundle (registerLogLevelFilterActions, shouldShowLogLevel)

### PN-5.7 Filter log lines by category
- Tag: both
- Evidence: source
- Boundary: no
- What: Some log entries have a category (a sub-source name). When the open log has categories, each one shows up as a toggle in the More Filters menu, above the level toggles. Untick a category to hide its entries. The list is built from the open log, so it changes when you change channel.
- Commands: workbench.actions.workbench.panel.output.toggle.<channel id>:<category>
- Menus: submenu.view.filter (More Filters, group 1_category_filter)
- Keys: none found
- Settings: none found
- Cursor: present (group 1_category_filter, toggleCategory and the category filter context key found)
- Source: outputView.ts (357-446, 498-504, 553-559); outputServices.ts (270-300); V\src\vs\workbench\services\output\common\output.ts (HIDE_CATEGORY_FILTER_CONTEXT); C bundle (output filter controller, shouldShowCategory, toggleCategory)

### PN-5.8 Auto scroll and scroll lock
- Tag: both
- Evidence: source
- Boundary: no
- What: A lock button in the Output title row switches between following new output and staying put (tooltips Turn Auto Scrolling Off and Turn Auto Scrolling On, lock and unlock icons). While following, the view jumps to the last line as text arrives. With smart scroll on (the default), clicking in the text above the last line locks it, and clicking on the last line unlocks it. This is the only setting in the Output group of the Settings editor.
- Commands: workbench.output.action.toggleAutoScroll
- Menus: ViewTitle (group navigation, order 3)
- Keys: none found
- Settings: output.smartScroll.enabled
- Cursor: present (the command, the setting and the context key outputView.scrollLock found; both tooltips and the setting description found in NLS)
- Source: output.contribution.ts (344-370, 839-851); outputView.ts (148-172); V\src\vs\workbench\services\output\common\output.ts (CONTEXT_OUTPUT_SCROLL_LOCK); C bundle (registerToggleAutoScrollAction, setting output.smartScroll.enabled); NLS (labels matched by text)

### PN-5.9 Clear the output
- Tag: both
- Evidence: source
- Boundary: no
- What: Clear the text of the open channel with the clear-all button in the Output title row, with Clear Output in the Command Palette (View category), or from the right-click menu inside the Output text. It plays the clear sound signal for accessibility users.
- Commands: workbench.output.action.clearOutput
- Menus: ViewTitle (group navigation, order 2); CommandPalette; EditorContext (when focus is in the Output text)
- Keys: none found
- Settings: none found
- Cursor: present (the command and the inOutput context key found; label Clear Output found in NLS)
- Source: output.contribution.ts (310-342); outputView.ts (333-340); C bundle (registerClearOutputAction)

### PN-5.10 Open the output in an editor tab or a new window
- Tag: both
- Evidence: source
- Boundary: no
- What: Open Output in Editor opens the open channel as a pinned read-only editor tab. Open Output in New Window does the same in a separate window. Both are title row actions declared with isHiddenByDefault, so a fresh window does not show them as buttons. I could not tell from code alone where a user finds them (see Unclear).
- Commands: workbench.action.openActiveLogOutputFile; workbench.action.openActiveLogOutputFileInNewWindow
- Menus: ViewTitle (group navigation, orders 4 and 5, both hidden by default)
- Keys: none found
- Settings: none found
- Cursor: present (both ids found with the same hidden-by-default menu rows; both labels found in NLS)
- Source: output.contribution.ts (372-415, 444-453); C bundle (registerOpenActiveOutputFileAction, registerOpenActiveOutputFileInAuxWindowAction); NLS 8524-8525

### PN-5.11 Save the output as a file
- Tag: both
- Evidence: source
- Boundary: no
- What: Save Output As... in the Output title row's more actions menu writes the open channel's text to a .log file. A save dialog opens with the channel name as the file name in the default folder.
- Commands: workbench.action.saveActiveLogOutputAs
- Menus: ViewTitle (group 1_export, order 1, in the more actions menu)
- Keys: none found
- Settings: none found
- Cursor: present (id, menu group 1_export and label found; the dialog title Save Output As found in NLS)
- Source: output.contribution.ts (417-442); outputServices.ts (492-540); C bundle (registerSaveActiveOutputAsAction, saveOutputAs); NLS 8526, 8536

### PN-5.12 Export logs
- Tag: both
- Evidence: source
- Boundary: no
- What: Export Logs... shows a multi-select list of log channels (built-in logs, then Extension Logs, then User Logs) and saves the ones you tick into one .log file through a save dialog. It is in the Output title row's more actions menu and in the Command Palette (Developer). In stock the command can also be called with an output path and a list of channel ids to skip the lists.
- Commands: workbench.action.exportLogs
- Menus: ViewTitle (group 1_export, order 2, in the more actions menu); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (the command, menu row and lists are present, but the compiled run method takes no arguments, so the outputPath and outputChannelIds arguments of stock are missing)
- Source: output.contribution.ts (713-783); outputServices.ts (492-540); C bundle (registerExportLogsAction with run(e) reading no arguments, saveOutputAs(...channels)); NLS 8530

### PN-5.13 Import a log file
- Tag: both
- Evidence: source
- Boundary: no
- What: Import Log... opens a file picker for .log files (one or several). The chosen file becomes a new user channel named after the file, and Output switches to it. Picking several files makes one channel with a section per file. Imported channels sit in the user group of the channel drop-down and can be removed (see PN-5.14).
- Commands: workbench.action.importLog
- Menus: ViewTitle (group 2_add, order 2, in the more actions menu); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (id and menu row found; the descriptor also has a workspaceId and a workspace-specific id)
- Source: output.contribution.ts (785-833); C bundle (registerImportLogAction, registerChannel with user:!0 and workspaceId); NLS 8531

### PN-5.14 Combine logs into one channel, and remove custom channels
- Tag: both
- Evidence: source
- Boundary: no
- What: Add Compound Log... lists the log channels (built-in first, then extension logs) as a multi-select and merges the ticked ones into one new channel, shown at once, with the names joined by commas as its label. It is in the Output title row's more actions menu and in the Command Palette (Output category). Remove Output... (Command Palette only) lists the user channels, meaning imported and combined ones, and removes the ticked ones. With none it says "No custom outputs to remove."
- Commands: workbench.action.output.addCompoundLog; workbench.action.output.remove
- Menus: ViewTitle (Add Compound Log, group 2_add, in the more actions menu); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (both ids, group 2_add and the notification text found)
- Source: output.contribution.ts (193-270); outputServices.ts (464-490); C bundle (registerAddCompoundLogAction, registerRemoveLogAction); NLS 8516-8517

### PN-5.15 Open a log file or the logs folder
- Tag: both
- Evidence: source
- Boundary: no
- What: Open Log... (Command Palette, Developer) lists the log channels, extension logs first, and opens the chosen log file in a pinned editor tab. It can also be called with a log id to skip the list. Open Logs Folder reveals the main.log file in the operating system file manager, and Open Extension Logs Folder reveals the first file in the extension logs folder. Both folder commands are desktop only. Stock also has Open Window Log File (Session)..., which picks a session folder and then a log file, but only in the web build.
- Commands: workbench.action.openLogFile; workbench.action.openLogsFolder; workbench.action.openExtensionLogsFolder
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (all three ids and labels found; workbench.action.openSessionLogFile is absent, which matches stock desktop because the session command lives in the web-only logs contribution)
- Source: output.contribution.ts (575-639); V\src\vs\workbench\contrib\logs\electron-browser\logs.contribution.ts (1-35); V\src\vs\workbench\contrib\logs\electron-browser\logsActions.ts (14-50); V\src\vs\workbench\contrib\logs\browser\logs.contribution.ts (24-45); V\src\vs\workbench\workbench.web.main.ts (138); V\src\vs\workbench\workbench.desktop.main.ts (113); C bundle (registerOpenLogFileAction, openLogsFolder, openExtensionLogsFolder); NLS (labels matched by text)

### PN-5.16 Read output and logs in the read-only text area
- Tag: both
- Evidence: source
- Boundary: no
- What: The body of the Output view is a read-only text editor. Lines always wrap, and the line numbers, folding, minimap and current-line highlight are off. Log channels use the Log language, so the built-in log grammar colors timestamps, levels and similar parts. You can select and copy text and use the editor's find widget and its right-click menu there. The editor options can be changed for just this view with a [Log] language override (editor.minimap.enabled and editor.wordWrap are read from it).
- Commands: none found
- Menus: EditorContext (shared entries; Clear Output is added when focus is in the Output text)
- Keys: none found
- Settings: [Log]
- Cursor: present (the compiled Output editor reads the [Log] override for minimap and word wrap, the Log language ids and mime types are registered, and the built-in log extension ships with the same manifest as stock)
- Source: outputView.ts (228-351); output.contribution.ts (53-70); outputChannelModel.ts (248, 393, 405); V\src\vs\workbench\services\output\common\output.ts (18-33); V\extensions\log\package.json; C ext log\package.json; C bundle (OutputEditor getConfigurationOverrides reading "[Log]", ids text/x-code-output and text/x-code-log-output)

### PN-5.17 Click file paths in the output
- Tag: both
- Evidence: source
- Boundary: no
- What: When a folder is open, file paths inside a workspace folder that appear in Output text become links. Ctrl+click (Cmd+click on macOS) opens the file, at a line and column when the path is followed by one in the usual forms (path:8:13, path(8,13), "on line 8, column 13" or ":line 8"). With no folder open there are no links.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the output link computer module and its workspace folders call found)
- Source: outputLinkProvider.ts (40-115); V\src\vs\workbench\contrib\output\common\outputLinkComputer.ts (25-155); outputLinkComputerMain.ts; C bundle (outputLinkComputer, $setWorkspaceFolders)

### PN-5.18 Quiet Output indicator in the status bar
- Tag: both
- Evidence: source
- Boundary: no
- What: With workbench.view.showQuietly set for the Output view, an extension that asks to show a channel while Output is closed gets a small prominent output icon in the status bar instead of the panel opening. The tooltip is "Show {channel} Output Channel", and clicking it opens that channel. The item is part of the extension API path, so you only see it when an extension asks to reveal a channel.
- Commands: workbench.action.output.show.<channel id>
- Menus: none found
- Keys: none found
- Settings: workbench.view.showQuietly
- Cursor: present (the setting, the status item id status.view.showQuietly and the _showChannelQuietly method found; labels Show Output and Show {0} Output Channel found in NLS)
- Source: V\src\vs\workbench\api\browser\mainThreadOutputService.ts (80-112); V\src\vs\workbench\browser\workbench.contribution.ts (691-702); C bundle (reveal method calling _showChannelQuietly, setting workbench.view.showQuietly); NLS 2830

### PN-5.19 Open accessibility help for the Output filter
- Tag: vscode
- Evidence: source
- Boundary: no
- What: With focus in the Output filter box, Open Accessibility Help shows a text page for screen reader users that explains the filter, its syntax (text, !text, comma lists, escaping), keyboard navigation and the Output settings. The page is chosen by the context key outputFilterFocus.
- Commands: editor.action.accessibilityHelp
- Menus: CommandPalette (Open Accessibility Help)
- Keys: Alt+F1 (Linux: Alt+Shift+F1, with Alt+F1 as secondary)
- Settings: accessibility.verbosity.find
- Cursor: missing (searched NLS for "Accessibility Help: Output Panel Filter"; the only filter help page in Cursor is "Accessibility Help: Terminal Find". The generic command editor.action.accessibilityHelp is present)
- Source: outputAccessibilityHelp.ts (1-105); output.contribution.ts (49-52); V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (164-182); C bundle (editor.action.accessibilityHelp found, no Output help provider); NLS (page title absent)

### PN-5.20 Cursor's extra Output channels
- Tag: cursor
- Evidence: bundle
- Boundary: yes - most of these channels log agent, MCP, indexing and account plumbing, so only the rows they add to the channel lists are in scope, not their contents. Only Cursor Tab logs an in-scope feature.
- What: Cursor adds about 34 named channels to the Output drop-down, Show Output Channels and (for the ones created as log channels) Show Logs, plus one "MCP: <server name>" channel per MCP server. From its built-in extensions: Cursor Tab, Cursor Always Local, Cursor Structured Logs, Cursor-always-local debug logs, Filesync, Cursor Indexing & Retrieval, Cursor Git Graph, Cursor Grep Service, Cursor Snapshot Service, Cursor-retrieval debug logs, Cursor Agent Review, Cursor-commits debug logs, Cursor Agent Host, Cursor Agent Host Network, Cursor Agent Host Storage, Cursor Agent Host Structured Logs, Cursor Plugins, Mcp FileSystem Writer, Cursor Agent Exec, Cursor Agent Worker, Local Agent Runtime, MCP Logs, Cursor Computer Use, Cursor IDE Browser Automation, Shadow Workspace, Cursor Resolver, Cursor Resolver Helper, Cursor Socket and NDJSON Ingest. From the main bundle: Cursor Request Traces, Hooks, Analytics Events, Cursor Auth Debug and Utility Host RPC Trace. Cursor Computer Use, Cursor IDE Browser Automation, Cursor Resolver, Cursor Resolver Helper, Hooks, Cursor Auth Debug and Utility Host RPC Trace are plain output channels, and the rest are log channels, so only those get the level filters, the category filters and the log level menu.
- Commands: workbench.action.output.show.<channel id>
- Menus: workbench.output.menu.switchOutput (extension channels in group 0_ext_outputchannels, bundle channels in 1_core_outputchannels); CommandPalette (Show Output Channels, Show Logs, Set Log Level, Open Log, Export Logs)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C ext cursor-always-local, cursor-retrieval, cursor-commits, cursor-agent-host, cursor-agent-exec, cursor-agent-worker, cursor-local-agent-runtime, cursor-mcp, cursor-computer-use, cursor-browser-automation, cursor-shadow-workspace, cursor-resolver, cursor-resolver-helper, cursor-socket and cursor-ndjson-ingest (createOutputChannel calls in each dist script); C bundle (registerChannel calls for Cursor Request Traces, Hooks, Analytics Events and Cursor Auth Debug; the utilityHostRpcTrace contribution); NLS 11661

## Handed off

- The find widget inside the Output text (Ctrl+F in the read-only editor) is the same widget as the editor find: ED-3 owns it.
- The quiet Output indicator in PN-5.18 is drawn in the status bar (`status.view.showQuietly`). I kept it here because its setting and trigger are Output items, but SS-5 may want a cross-reference.

## Unclear

- PN-5.10: Open Output in Editor and Open Output in New Window are declared with isHiddenByDefault on the view title menu. I could not tell from code alone whether a fresh window shows them in the title row, hides them completely, or offers them in a right-click toggle. Check on screen.
- Cursor's channel descriptors have a workspaceId and Cursor builds ids from the channel id plus the workspace id for user channels (imported and combined logs) and some of its own channels. I did not trace what that changes for the user.
- I did not check which of the Cursor channels in PN-5.20 are visible on a fresh start, because many belong to extensions whose activation I did not trace. The list is what the code can create. The dynamic channel name in cursor-agent-host (a channel created from a caller-supplied name) was not resolved.
- The catalog gives PN-5 no boundary tag. I set PN-5.20 to boundary yes because the channel contents come from agent plumbing, but pass 1 treated the Output rows as ordinary. Reverse it if you prefer.
- Whether non-log channels are colored: log channels use the Log language and the log grammar, but I did not trace which grammar a plain Output channel gets.
- The catalog line for PN-5 mentions "Output settings". The Settings editor has one Output setting (`output.smartScroll.enabled`, in PN-5.8). The other settings that touch this group are `workbench.view.showQuietly` (PN-5.18), the `[Log]` override (PN-5.16) and `accessibility.verbosity.find` (PN-5.19), so I attached each to its feature instead of making a separate settings feature.
