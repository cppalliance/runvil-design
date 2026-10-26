Group: PN-5 Output and logs. Pass: 3. Date: 2026-10-25.

### PN-5.1
1. Choosing Output from the View menu opens the panel on the Output tab and moves focus into the Output text.
2. Pressing `Ctrl+Shift+U` (on Linux the chord `Ctrl+K Ctrl+H`, `workbench.action.output.toggleOutput`) opens Output and focuses it, and pressing it again while Output has focus hides the panel.
3. Toggle Output in the Command Palette does the same as the key.
4. Focus on Output View in the Command Palette (`workbench.panel.output.view.focus`) opens the Output tab and moves focus into it.
5. When the workspace has no channel chosen yet, the Output view opens on the first registered channel.

### PN-5.2
1. The Output title row has a drop-down that shows the name of the current channel.
2. Opening the drop-down lists the channels with a check beside the active one, and picking a channel switches the Output text to it.
3. The list has three groups with separators between them: extension channels first, then built-in channels, then user channels such as imported or combined logs.
4. Channels that are added while Output is open show up in the list, and channels that are removed disappear from it.
5. The channel picked in the drop-down is remembered for the workspace and shown again the next time Output opens.
6. Running `workbench.action.output.show.<channel id>` opens Output on that channel.

### PN-5.3
1. Show Output Channels... in the Command Palette (Output category) opens a list titled "Select Output Channel" with every channel, extension channels first, then a separator, then built-in channels.
2. Show Logs... in the Command Palette (Developer category) opens a list titled "Select Log" with only log channels, built-in logs first and then an Extension Logs separator with the extension logs.
3. Show Window Log in the Command Palette (Developer category) opens Output straight on the Window log with no list.
4. Choosing an entry in either list opens Output on that channel.

### PN-5.4
1. For a log channel whose level can be set, the Output title row shows a gear button (Set Log Level...) that opens a menu.
2. The gear menu lists Trace, Debug, Info, Warning, Error and Off with a check on the current level, and picking one sets the level for that channel.
3. A Set As Default entry below the levels is enabled only when the channel's level differs from the default, and picking it saves that level as the default.
4. The gear does not show for plain channels, and the telemetry log cannot have its level set.
5. Set Log Level... in the Command Palette (Developer category) opens a picker that starts with an All section listing the six levels, with a check on the current one and the word Default beside the default one, and picking a level sets it for everything.
6. Below the All section the picker lists extension logs and then the other logs, and picking a channel opens a second picker of the six levels for just that channel.
7. In each picker a button on every level that is not the default, with the tooltip "Set as Default Log Level", makes that level the default and closes the picker.

### PN-5.5
1. Typing in the filter box in the Output header hides the lines that do not contain the text and highlights the matches in the lines that stay, without caring about upper and lower case.
2. In a log channel the filter hides or keeps whole log entries, and in a plain channel it hides or keeps single lines.
3. Typing several terms separated by commas shows a line when any one term matches. [stock only]
4. A term with a leading `!` hides the lines that contain it. [stock only]
5. A term in double quotes can contain a comma without being split. [stock only]
6. Pressing `Escape` in the filter box clears the filter text (`workbench.actions.workbench.panel.output.clearFilterText`).
7. The filter text is saved for the workspace.
8. The stock filter box placeholder reads "Filter (e.g. text, !excludeText, text1,text2)". [stock only]
9. In Cursor the filter takes one plain text with no comma list and no `!` exclusion, and its placeholder reads just "Filter". [cursor only]

### PN-5.6
1. While a log channel is open, the More Filters button (filter icon) next to the filter box lists Trace, Debug, Info, Warning and Error as toggles, all ticked by default.
2. Unticking a level hides the log entries of that level.
3. The More Filters button is highlighted while any level is hidden.
4. The ticked and unticked levels are saved for the workspace.
5. The level toggles do not appear while a plain channel is open.

### PN-5.7
1. When the open log has entries with a category, each category appears as a toggle in the More Filters menu, in a group above the level toggles.
2. Unticking a category hides the log entries of that category.
3. The list of categories is built from the open log, so it changes when another channel is opened.
4. The More Filters button is highlighted while a category of the open channel is hidden.
5. The hidden categories are saved for the workspace.

### PN-5.8
1. The Output title row has a lock button next to Clear Output that switches between following new output and staying put.
2. While following, the button shows a lock icon with the tooltip "Turn Auto Scrolling Off", and the view jumps to the last line as new text arrives.
3. Clicking the button stops following and shows an unlock icon with the tooltip "Turn Auto Scrolling On", and new text then no longer moves the view, until the button is clicked again.
4. Setting `output.smartScroll.enabled`, on by default, lets a click in the text above the last line stop following and a click on the last line resume it.
5. Opening a channel in the Output view scrolls it to its last line.

### PN-5.9
1. The clear-all button in the Output title row empties the text of the open channel.
2. Clear Output in the Command Palette (View category) empties the open channel the same way.
3. Right-clicking inside the Output text offers Clear Output in the context menu, and it empties the open channel.
4. Clearing the output plays the clear sound signal for accessibility users.

### PN-5.10
1. Open Output in Editor (`workbench.action.openActiveLogOutputFile`, go-to-file icon) opens the open channel as a pinned read-only editor tab.
2. Open Output in New Window (`workbench.action.openActiveLogOutputFileInNewWindow`, empty-window icon) opens the open channel in a separate window.
3. Both are Output title row actions that are hidden by default, so a fresh window does not show them as buttons.

### PN-5.11
1. Save Output As... in the more actions menu of the Output title row opens a save dialog titled "Save Output As".
2. The dialog starts in the default folder with the channel name plus `.log` as the file name, and its file type filter is limited to `log`.
3. Confirming the dialog writes the text of the open channel to the chosen file, and cancelling writes nothing.

### PN-5.12
1. Export Logs... in the more actions menu of the Output title row, and in the Command Palette (Developer category), opens a list of log channels where several can be ticked.
2. The list shows built-in logs first, then an Extension Logs separator with the extension logs, then a User Logs separator with the user logs.
3. Confirming the ticked logs opens a save dialog and saves them into one `.log` file, named `output.log` by default when more than one is ticked and after the channel when only one is.
4. In stock the command can be called with an output path and a list of channel ids, which skips the list, the save dialog or both. [stock only]
5. In Cursor the command always shows the list and the save dialog, because it ignores any output path or channel ids passed to it. [cursor only]

### PN-5.13
1. Import Log... in the more actions menu of the Output title row, and in the Command Palette (Developer category), opens a file picker titled "Import Log File".
2. The picker accepts one or several files and shows only files of the type Log Files, which is `.log`.
3. Picking one file creates a new user channel named after the file and switches Output to it.
4. Picking several files creates one channel named after the first file, with a section for each file.
5. Imported channels show in the user group of the channel drop-down.

### PN-5.14
1. Add Compound Log... in the more actions menu of the Output title row, and in the Command Palette (Output category), opens a list titled "Select Log" where several log channels can be ticked, built-in logs first and then an Extension Logs separator with the extension logs.
2. Confirming the list merges the ticked logs into one new channel that Output shows at once, with the names of the logs joined by commas as its label.
3. Remove Output... in the Command Palette (Output category) opens a list titled "Select Log" of the user channels, meaning the imported and combined ones, where several can be ticked.
4. Confirming the Remove Output list removes the ticked channels from the channel list.
5. When there are no user channels, Remove Output shows the notification "No custom outputs to remove."

### PN-5.15
1. Open Log... in the Command Palette (Developer category) opens a list titled "Select Log File" of log channels, extension logs first and then a separator and the built-in logs.
2. Picking a log in that list opens its log file in a pinned editor tab.
3. Running Open Log... with a log id such as `"window"` opens that log file straight away with no list.
4. Open Logs Folder reveals the `main.log` file in the operating system file manager, and it only exists in the desktop app.
5. Open Extension Logs Folder reveals the first file in the extension logs folder in the operating system file manager, and it only exists in the desktop app.
6. In the web build, Open Window Log File (Session)... first asks for a session folder and then for a log file to open. [stock only]

### PN-5.16
1. The body of the Output view is a read-only text editor where text can be selected and copied but not changed.
2. Lines always wrap, and line numbers, folding, the minimap and the current-line highlight are off.
3. Log channels use the Log language, so the built-in log grammar colors parts such as timestamps and levels.
4. Right-clicking in the Output text opens the shared editor context menu.
5. Setting `[Log]` with `editor.minimap.enabled` set to true turns the minimap on in the Output view only.
6. Setting `[Log]` with `editor.wordWrap` changes how lines wrap in the Output view only, with the values `off`, `on`, `wordWrapColumn` and `bounded`.

### PN-5.17
1. When a folder is open, file paths inside a workspace folder that appear in the Output text become links.
2. `Ctrl+click` on a link (`Cmd+click` on macOS) opens that file in the editor.
3. When a line follows the path in a form such as `path:8:13`, `path(8,13)`, "on line 8, column 13" or ":line 8", the file opens at that line, and at that column when one is given.
4. When no folder is open, no links appear in the Output text.

### PN-5.18
1. With `workbench.view.showQuietly` turned on for Output, an extension that asks to show a channel while Output is closed makes a small prominent output icon appear in the status bar instead of opening the panel.
2. The icon's tooltip reads "Show {channel} Output Channel", and clicking it opens Output on that channel.
3. Setting `workbench.view.showQuietly` is an object whose `workbench.panel.output` key is true or false, and it is not set by default.
4. When that key is off, or Output is already open, an extension's request to show a channel opens Output on that channel as usual.

### PN-5.19
1. With focus in the Output filter box, Open Accessibility Help (`editor.action.accessibilityHelp`, `Alt+F1`, and on Linux `Alt+Shift+F1` with `Alt+F1` as the second key) opens a page titled "Accessibility Help: Output Panel Filter". [stock only]
2. The page explains that the filter is not a navigating search, that it hides lines that do not match, that focus stays in the box while the output updates, and that new output is filtered as it arrives. [stock only]
3. The page describes the filter syntax: plain text that ignores case, `!text` to hide, an escape for a literal exclamation mark, and comma separated terms where any one may match. [stock only]
4. The page lists the keyboard moves: Down Arrow into the output, Tab to the log level buttons, and Escape to clear the filter. [stock only]
5. The page lists the settings `accessibility.verbosity.find` and `output.smartScroll.enabled`, with the key that opens Settings. [stock only]
6. Setting `accessibility.verbosity.find` controls whether the filter box announces the hint that Accessibility Help is available. [stock only]

### PN-5.20
1. The channel drop-down and Show Output Channels list the channels of Cursor's built-in extensions: Cursor Tab, Cursor Always Local, Cursor Structured Logs, Cursor-always-local debug logs, Filesync, Cursor Indexing & Retrieval, Cursor Git Graph, Cursor Grep Service, Cursor Snapshot Service, Cursor-retrieval debug logs, Cursor Agent Review, Cursor-commits debug logs, Cursor Agent Host, Cursor Agent Host Network, Cursor Agent Host Storage, Cursor Agent Host Structured Logs, Cursor Plugins, Mcp FileSystem Writer, Cursor Agent Exec, Cursor Agent Worker, Local Agent Runtime, MCP Logs, Cursor Computer Use, Cursor IDE Browser Automation, Shadow Workspace, Cursor Resolver, Cursor Resolver Helper, Cursor Socket and NDJSON Ingest. [cursor only]
2. The same lists also show the channels made by Cursor's own code: Cursor Request Traces, Hooks, Analytics Events, Cursor Auth Debug and Utility Host RPC Trace. [cursor only]
3. Each MCP server adds one more channel named "MCP: <server name>". [cursor only]
4. In the channel drop-down the channels from Cursor's built-in extensions sit in the extension group, and the channels from Cursor's own code sit in the built-in group. [cursor only]
5. Cursor channels that are log channels also show in Show Logs and get the level toggles, the category toggles and the log level gear menu. [cursor only]
6. Cursor Computer Use, Cursor IDE Browser Automation, Cursor Resolver, Cursor Resolver Helper, Hooks, Cursor Auth Debug and Utility Host RPC Trace are plain channels, so they get the text filter but none of the log level or category toggles or the log level gear menu. [cursor only]
7. Choosing Cursor Tab opens Output on the log channel where Cursor Tab writes its log. [cursor only]

## Unclear

- PN-5.10: Open Output in Editor and Open Output in New Window are hidden by default on the Output title row, and the source does not show where a person turns them on, so the pieces say only that they are hidden by default.
- PN-5.15: Open Window Log File (Session)... is marked [stock only] because it is a web build command and the desktop app of both stock and Cursor lacks it, so it is not a real Cursor difference.
- PN-5.20: the piece that lists the channel names is long, and the list should be split into one feature per channel family if a tracker needs per-channel progress.

