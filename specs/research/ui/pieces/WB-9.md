Group: WB-9 Windows and workspaces. Pass: 3. Date: 2026-10-25.

### WB-9.1
1. Running the command opens a new window on an untitled multi-folder workspace that lists the same folders as the current window.
2. The workspace settings of the current window are copied into the new workspace.
3. The current window stays open and unchanged.
4. When the current window is remote, the new window uses the same remote connection.
5. The command is shown only when multi-folder workspaces are supported.
6. In stock the command is also hidden in the Agents window; the Cursor bundle has no such condition. [stock only]
7. It is reached only from the Command Palette under the Workspaces category, with no menu row and no key.

### WB-9.2
1. Running the command opens the workspace file of the current multi-folder workspace as a pinned editor tab.
2. The command is available only when the window is on a workspace file, so it does not show for a single folder or an empty window.
3. In stock the command is also hidden in the Agents window; the Cursor bundle has no such condition. [stock only]
4. It is reached only from the Command Palette under the Workspaces category, with no menu row and no key.

### WB-9.3
1. "Shell Command: Install 'cursor' command" puts the cursor command on the path and shows "Shell command 'cursor' successfully installed." or the error text; it exists on macOS only, as in stock where the label is "Install '{0}' command in PATH".
2. "Shell Command: Uninstall 'cursor' command from PATH" removes it and shows "Shell command 'cursor' successfully uninstalled from PATH."; cancelling shows nothing.
3. "Shell Command: Install 'code' command" makes the code command open Cursor and shows "Successfully installed Cursor in the 'code' command." [cursor only]
4. "Shell Command: Uninstall 'code' command from PATH" unlinks Cursor from the code command and shows "Successfully unlinked Cursor from the 'code' command." [cursor only]
5. The two code commands are registered on every platform, while the two cursor commands are registered on macOS only. [cursor only]
6. A sticky notification asks "Want to launch Cursor from the command line? You can add the `code` or `cursor` commands now, or later with the Command Palette key and typing 'install code command'." with buttons Install 'code', Install 'cursor' and Never show again. [cursor only]
7. The notification appears about 5 minutes after start, on Windows and macOS only, at most twice, and not again within 5 days of showing; pressing any of its three buttons stops it for good. [cursor only]
8. All four entries are in the Command Palette under Shell Command and have no menu rows or keys.

### WB-9.4
1. "Switch to Main Window" is listed in the Command Palette only while the active window is an auxiliary window (a secondary editor window), and it brings the main window to the front. [stock only]
2. "Focus Window" brings the current window to the foreground and focuses it even when the app is not the active one, by forcing focus from the operating system. [stock only]
3. Focus Window has no menu row and no key, and its code comment says it is meant as the first step of a `runCommands` chain that must reveal the window before a command such as Quick Open. [stock only]

### WB-9.5
1. After a single real (non virtual) folder opens, and its top level holds exactly one workspace file, an info notification says "This folder contains a workspace file '<name>'. Do you want to open it?" with a Learn more link and an Open Workspace button.
2. When the folder holds several workspace files the text says it "contains multiple workspace files", and its Select Workspace button opens a pick list "Select a workspace to open" of the file names.
3. Pressing Open Workspace, or picking a name from the list, opens that workspace file.
4. The notification has a Don't Show Again choice that is remembered for this workspace only.
5. In a workspace that has been opened before, the notification arrives silently, in the notification center without a pop-up; in a brand new workspace it pops up as usual.

### WB-9.6
1. The Touch Bar shows a Go Back button and a Go Forward button that move through the editor navigation history.
2. When a debugger is available and no session is running, the Touch Bar shows Start Without Debugging and Start Debugging buttons.
3. During a debug session the Touch Bar shows Step Over, Step Into, Step Out, Restart and Stop, plus Continue while stopped and Pause while running.
4. Pressing a button runs its command with the active file as the argument, because the buttons depend on the active editor.
5. The buttons update a short moment after the active editor, the focus or the debug state changes.
6. Setting `keyboard.touchbar.enabled` (on by default, listed only on macOS) turns all the buttons off when it is off, and the change applies at once.
7. Setting `keyboard.touchbar.ignored` takes a list of command ids that must not show on the Touch Bar, for example `workbench.action.navigateBack`.

### WB-9.7
1. Pressing Ctrl+R, or running Open Recent... from the Command Palette or File > Open Recent > More..., opens a list with the placeholder "Select to open (hold Ctrl-key to force new window or Alt-key for same window)", which says Cmd-key and Option-key on macOS.
2. The list has a "folders & workspaces" heading (just "folders" when none of the entries is a workspace) over the recent folders and workspaces, then a "files" heading over the recent files, in most recently used order.
3. Each row shows the entry name with a file, folder or workspace icon and its parent path as a description, and typing in the box narrows the rows by name or path.
4. Choosing a row opens it in the current window, holding Ctrl (Cmd on macOS) while choosing opens it in a new window, and holding Alt (Option on macOS) reuses the current window.
5. When the first recent entry is the workspace you are already in, the highlight starts on the second row.
6. Each row has a Remove from Recently Opened button, and clicking it takes the entry out of the list.
7. A folder or workspace that has unsaved backups shows an always-visible dot button in place of the remove button, so it cannot be removed (see DL-2.14).
8. A folder or workspace that is already open in a window shows an always-visible window icon with the tooltip "Folder Opened in a Window" or "Workspace Opened in a Window", or "Folder Opened in Active Window" or "Workspace Opened in Active Window" for the window you are in, and clicking the first kind removes the entry from the list. [stock only]
9. Quick Open Recent (`workbench.action.quickOpenRecent`) shows the same list in quick-navigate mode with the input box hidden, moves through it with the keys bound to it, and is not shown in the Command Palette.

### WB-9.8
1. Running Switch Window... from the Command Palette opens a list with the placeholder "Select a window to switch to" and one row per open window, each with the window title and a file or folder icon.
2. Choosing a row brings that window to the front and gives it focus.
3. The row for the window you are in carries the description "Current Window", and the highlight starts on the next window below it.
4. The rows of other windows have a Close Window button, and clicking it closes that window and takes its row out of the list.
5. A window with unsaved changes shows an always-visible dot button, and a screen reader reads its row as "{title}, window with unsaved changes".
6. When any window has floating windows, the rows are grouped, with a "window group" heading over each main window followed by its floating windows.
7. The row for the window you are in has an always-visible Close Active Window button. [stock only]
8. Quick Switch Window (`workbench.action.quickSwitchWindow`) shows the same list in quick-navigate mode with the input box hidden, and it is not shown in the Command Palette.
9. On macOS Ctrl+W opens the list, and the OS Window menu has a Switch Window... row.

### WB-9.9
1. Typing Close Other Windows in the Command Palette lists the command with no category and no default key. [stock only]
2. Running it closes every other main window of the app and keeps the window it was run from. [stock only]
3. Each of those windows is closed with the same call its own close button uses, so a window that has unsaved changes follows its normal close flow and may still ask first (see DL-2). [stock only]
4. Floating editor windows are not closed one by one, because only main windows are listed, so they go away with their parent window. [stock only]
5. With a single window open, running the command changes nothing. [stock only]

### WB-9.10
1. Setting `window.newWindowDimensions` chooses how a new window is sized and placed, with values `default`, `inherit`, `offset`, `maximized` and `fullscreen`.
2. `default`, the default, opens each new window in the center of the screen.
3. `inherit` opens a new window with the same size as the last active window, and `offset` does the same with an offset position.
4. `maximized` opens each new window maximized and `fullscreen` opens it in full screen mode.
5. The setting applies only when at least one window is already open, because the first window always restores the size and place it had when it was closed.
6. The setting is an application setting, so a workspace cannot override it.

### WB-9.11
1. Setting `window.closeWhenEmpty` is off by default, and while it is off closing the last editor leaves the empty window open.
2. With the setting on, closing the last editor in a window that has no folder or workspace closes that window.
3. A window that has a folder or workspace open stays open when its last editor closes.
4. The window also stays open while other windows, such as floating editor windows, still have editors open, and while an extension is being developed in it.
5. With the setting on, a floating editor window closes when its last editor closes, even when a folder is open in the main window.
6. The check is debounced, so opening another editor right after the last one closes keeps the window open.

### WB-9.12
1. Setting `window.openWithoutArgumentsInNewWindow` chooses what happens when a second instance starts with no folder or file, with values `on` and `off`.
2. `on` opens a new empty window.
3. `off` focuses the last active running window instead of opening anything.
4. The default is `on` on Windows and Linux and `off` on macOS.
5. The setting is an application setting, and the setting text says it can still be ignored when the new-window or reuse-window command line option is used.

## Unclear
none
