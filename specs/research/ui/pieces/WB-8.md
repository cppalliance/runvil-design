Group: WB-8 Developer commands. Pass: 3. Date: 2026-10-25.

### WB-8.1
1. After the command runs, the mouse pointer becomes a crosshair everywhere and a translucent red box follows the element under the pointer.
2. Clicking an element writes every context key value at that element to the console of the developer tools.
3. The click is swallowed, so the clicked element does not react to it.
4. After one click the crosshair and the red box go away.
5. The command is in the Command Palette under Developer and has no menu row or key; the result is seen only by opening the developer tools console.

### WB-8.2
1. Running the command writes the storage database contents to the developer tools console.
2. An info box then says "The storage database contents have been logged to the developer tools." with the detail "Open developer tools from the menu and select the Console tab."
3. The command is in the Command Palette under Developer and has no menu row or key.

### WB-8.3
1. The log gets a Working Copies list, one line per working copy with its address and type id, and a dirty one starts with a filled circle.
2. The log also gets a Backups list in the same line format.
3. A list with nothing in it shows as "<none>".
4. The Output view opens on the Window log channel without taking keyboard focus.
5. The command is in the Command Palette under Developer and has no menu row or key.

### WB-8.4
1. The command opens a multi-select picker with the placeholder "Select large entries to remove from storage", one row per storage key with its size, biggest first.
2. Each row's detail says "Scope: Global, Profile or Workspace, Target: Machine or User".
3. In a built app only entries over 16 KB are listed, while a development build lists every key.
4. For the default profile the Profile scope is skipped so no key shows twice.
5. With nothing to remove, the picker's description says "There are no large storage entries to remove."
6. The picker has a custom Remove button instead of OK and no select-all box.
7. Pressing Remove opens a warning box "Do you want to remove the selected storage entries from the database?" that lists the keys and warns "This action is irreversible and may result in data loss!"
8. After confirming, the entries are deleted and the storage database is compacted for each affected scope.

### WB-8.5
1. Running the command opens a pinned untitled Markdown editor headed "VS Code Policy Diagnostics" with a warning that the file may hold sensitive information. [stock only]
2. The report has System Information with the time generated, product name and version, and commit. [stock only]
3. Account Information shows a summary and a property table, with the session id and analytics tracking id hidden as ***. [stock only]
4. Account Policy Gate shows its state and reason with a legend of the possible states. [stock only]
5. Managed Settings lists the native MDM, GitHub server, and managed-settings.json sources, how each key resolves, and the effective values. [stock only]
6. Policy-Controlled Settings is split into Applied Policy and Non-applied Policy. [stock only]
7. Authentication Information lists providers and session details, and any lookup that fails is written into the report as an error line. [stock only]
8. The command is in the Command Palette under Developer and has no menu row or key. [stock only]

### WB-8.6
1. Running the command forces a refresh of the default account and its policy data. [stock only]
2. On success an info box says "Account policy has been synced." [stock only]
3. On failure an error box says "Failed to sync account policy." with the error message as detail. [stock only]
4. The command is in the Command Palette under Developer and has no menu row or key. [stock only]

### WB-8.7
1. Running the command turns Screencast Mode on, and running it again turns it off and removes the overlays.
2. While the mouse button is down a colored circle sits over the pointer and follows it, shrinking a little while the mouse is dragged.
3. Each key press adds a key label to a large overlay near the bottom of the window, with the command name in front of it when the keys match a shortcut.
4. The overlay clears when a modifier key, Backspace, Escape or an arrow key starts a new entry, when more than about 20 keys are shown, or after the timeout.
5. It works across all open windows, including separate editor windows.
6. Setting `screencastMode.verticalOffset` sets the overlay height from the bottom as a percentage (0 to 90, default 20) and `screencastMode.fontSize` sets its font size (20 to 100, default 56).
7. Setting `screencastMode.keyboardOptions` turns on or off raw keys, keyboard shortcuts, command names, command group names, and single editor cursor moves, with only group names off by default.
8. Setting `screencastMode.keyboardOverlayTimeout` sets how long the overlay stays (500 to 5000 ms, default 800).
9. Settings `screencastMode.mouseIndicatorColor` (default #FF0000) and `screencastMode.mouseIndicatorSize` (20 to 100 px, default 20) change the circle, and all six settings apply live while it is on.

### WB-8.8
1. Running the command opens a multi-select picker titled "Capture and Send Debugging Data" with the placeholder "Select debugging data to collect. Press Enter to continue.", and the picker stays open when focus moves away. [cursor only]
2. The picker rows are Renderer heap snapshot, User database, Logs, Feature flags and extension summary, Process diagnostics, and Agent transcripts (last 7 days), each with a one-line detail. [cursor only]
3. Logs, Feature flags and extension summary, and Process diagnostics start ticked, and the rest start unticked. [cursor only]
4. The details for User database and Agent transcripts warn that chat and prompt data, which may include attached code, is uploaded even in privacy mode. [cursor only]
5. An extra row, Agent host heap snapshots, shows only when the agent host saved heap snapshots in the last day. [cursor only]
6. After Enter, an input box asks "What went wrong? (optional)", and closing it stops the capture. [cursor only]
7. A progress notification reads "Capturing and sending debugging data..." or, when there is no upload address, "Capturing debugging data...". [cursor only]
8. The end shows one message: "Debugging data was sent successfully.", or "Debugging data saved locally." with a Reveal in File Explorer button, or "Failed to capture or send debugging data." with the error; which of the first two shows was not traced. [cursor only]
9. The command is in the Command Palette under Developer and has no menu row or key. [cursor only]

### WB-8.9
1. Running "Developer: Capture Performance Profile" first opens a confirm box "Capture Performance Profile" saying a CPU and allocations profile of the current window will be created and uploaded securely for analysis, with Continue and Cancel. [cursor only]
2. After Continue, a progress notification reads "Capturing performance profile...". [cursor only]
3. A successful start shows an info message "Performance profile started." with a Profile ID. [cursor only]
4. A failed start shows "Failed to start performance profile." or "Error starting performance profile: {message}". [cursor only]
5. "Developer: Show Performance Profiling Status" shows one info message: "Performance Profiling Status: Privacy mode: enabled (profiling forbidden) or disabled (profiling allowed), Active profiles: n, Scheduled profiles: n". [cursor only]
6. Both commands are in the Command Palette under Developer and have no menu rows or keys. [cursor only]

### WB-8.10
1. Turning crash watching on shows a confirm box "Toggle Watch for Crashes" warning that the editor may get slower and debugging data is sent automatically if a crash happens, with Continue and Cancel. [cursor only]
2. After the toggle, an info message says "Watching for crashes." or "Stopped watching for crashes."; turning it off asks for no confirmation. [cursor only]
3. "Developer: Force Renderer OOM Crash" opens a confirm box saying it will use memory until the renderer crashes, the window will freeze and then reload, and it is meant for testing the watcher, with Crash Now and Cancel. [cursor only]
4. After Crash Now the window stops responding, crashes, and reloads. [cursor only]
5. Both commands are in the Command Palette under Developer and have no menu rows or keys. [cursor only]

### WB-8.11
1. Running the command writes a file named feature-gates plus a timestamp with a .json ending into the .cursor/feature-gates folder of the home folder, and creates the folder when it is missing. [cursor only]
2. An info notification says "Feature gates dumped to" followed by the file path. [cursor only]
3. The notification has a Reveal in File Explorer button that shows the file in the operating system's file manager. [cursor only]
4. The command is in the Command Palette under Developer and has no menu row or key. [cursor only]

### WB-8.12
1. Running the command writes a banner and one line for each of 21 folders and files, such as User Data Directory, Logs Home, Extensions Path, Cache Home and Argv Resource, to the log. [cursor only]
2. An info notification titled "VS Code Data Directories:" lists the same paths and ends with "To reset VS Code data, delete the User Data Directory and restart VS Code." [cursor only]
3. The notification has a Copy to Clipboard button, and after it runs the message "Data directories copied to clipboard" or "Failed to copy to clipboard" shows. [cursor only]
4. The command is in the Command Palette under Developer and has no menu row or key. [cursor only]

### WB-8.13
1. Running "Developer: Reload With HMR" reloads the window with hot module replacement turned on, with no confirmation. [cursor only]
2. Running "Developer: Restart in Temporary Profile" relaunches the whole app with no confirmation. [cursor only]
3. The relaunched app uses a new empty user data folder named vscode-temp-profile plus a timestamp in the temp folder, so it starts with default settings and no saved window state. [cursor only]
4. The normal user data folder is left untouched by the restart. [cursor only]
5. Both commands are in the Command Palette under Developer and have no menu rows or keys. [cursor only]

### WB-8.14
1. Choosing Help > Toggle Developer Tools opens the developer tools for the active window, and choosing it again closes them.
2. The Command Palette entry "Developer: Toggle Developer Tools" does the same thing.
3. With several windows open the command acts only on the window that has focus.
4. Pressing `Ctrl+Shift+I` (`Cmd+Option+I` on macOS) toggles the tools in every Cursor build. [cursor only]
5. In a release build of stock the key does nothing, because it is bound only in a development build, while the Help row and the palette entry still work. [stock only]

### WB-8.15
1. Running "Developer: Reload with Extensions Disabled" reloads the window at once, with no question asked first, and the window comes back with all extensions off.
2. The command exists only in the Command Palette under Developer, with no menu row and no key.
3. The extensions are off for that reload only, so an ordinary Reload Window afterwards brings them back (the flag is passed for one reload; I did not run it).

### WB-8.16
1. Running "Developer: Reveal User Data Folder" opens the file manager (Explorer on Windows, Finder on macOS) with the user data folder selected in its parent folder.
2. Nothing shows inside the editor window itself, so there is no dialog, notification or editor tab.
3. The command exists only in the Command Palette under Developer, with no menu row and no key.

### WB-8.17
1. Running "Developer: Show GPU Info" opens a separate window, a child of the current window, that shows the Chromium `chrome://gpu` page.
2. Running the command again while that window is open does not make a second one: it restores the window if it is minimized and brings it to the front.
3. After you close the window, running the command opens a fresh one.
4. The command exists only in the Command Palette under Developer, with no menu row and no key.

### WB-8.18
1. Running "Developer: Show Content Tracing" opens a window with the Chromium `chrome://tracing` page, which stays hidden until the page has loaded and then shows. [stock only]
2. Unlike the GPU info window, the tracing window is not a child of the editor window that opened it, so it has no parent to follow. [stock only]
3. Running the command again while the window is open restores it if minimized and brings it to the front instead of opening another. [stock only]
4. Cursor has no such command in its Command Palette. [stock only]

### WB-8.19
1. Running "Developer: Force Refresh Server Config" asks the server for the configuration again without waiting for the normal refresh. [cursor only]
2. On success an info notification "Server config refreshed" appears, with the line "Request ID: <id>" under it. [cursor only]
3. On failure an error notification "Failed to refresh server config" appears with the same Request ID line. [cursor only]
4. The command exists only in the Command Palette, with no menu row and no key. [cursor only]

### WB-8.20
1. Running "Developer: Capture and Upload Process Snapshot" starts the capture straight away, with no confirmation first, and writes the snapshot file to disk. [cursor only]
2. While the file uploads, a progress message "Uploading snapshot to Cursor..." shows in the window. [cursor only]
3. A notification then says "Snapshot uploaded successfully. Thank you!", or "Snapshot upload failed. The snapshot is still saved on disk.", or "Could not upload snapshot: unable to get upload URL." [cursor only]
4. If the capture itself fails, an error notification reads "An error occurred while capturing debug snapshot: <message>". [cursor only]
5. Running the command while another check or snapshot is in progress shows nothing and only writes a warning to the log. [cursor only]

### WB-8.21
1. The palette entry is titled "Developer: Toggle Auth Debugging" (the word Developer is part of the title, not a category) and is offered with no condition. [cursor only]
2. Running it flips the sign-in debug flag and then opens the Output panel on the Cursor Auth Debug channel.
3. When the flag turns on, the channel gets a first line saying "Auth debugging enabled" with the account's auth id (or none) and whether HTTP/2 is disabled, and later sign-in activity is written there.
4. Running it again turns the logging off and still shows the channel.

### WB-8.22
1. Five entries named Copy Merge Editor State as JSON, Save Merge Editor State to Folder, Load Merge Editor State from Folder, Open Merge Editor State from JSON and Open Selection In Temporary Merge Editor appear in the Command Palette under the category "Merge Editor (Dev)".
2. Copy Merge Editor State as JSON puts the base, both inputs, the result, the initial result and the language id on the clipboard as JSON and shows the notice "Successfully copied merge editor state".
3. Save Merge Editor State to Folder opens a folder picker titled "Select folder to save to" and writes the five texts as files named base, input1, input2, result and initialResult with the language's extension, then shows "Successfully saved merge editor state to folder".
4. The copy and save entries are offered only while a merge editor is the active editor, and if it is gone by the time they run they show the notice "No active merge editor".
5. Load Merge Editor State from Folder opens the same folder picker, asks whether to open "result" or "initial result", and opens a new merge editor whose inputs are labelled "(from file)".
6. Open Merge Editor State from JSON asks "Enter JSON" with the clipboard text filled in, writes the texts to temporary files, and opens a merge editor whose inputs are labelled "(from JSON)"; an empty answer opens an empty plain text merge. This entry and the next exist in the desktop app only.
7. Open Selection In Temporary Merge Editor takes the text selected in the base pane, finds the matching text in both inputs and the result, and opens just that excerpt as a temporary merge editor, and does nothing when nothing is selected.

### WB-8.23
1. On Windows the Command Palette lists Developer: Apply Update from File..., and the command is not registered on macOS or Linux.
2. Running it opens a file picker titled "Apply Update from File" that shows only setup `.exe` files and has an Update button.
3. Closing the picker without choosing a file does nothing.
4. Choosing a setup file makes the updater treat that file as an update that has already been downloaded, with no network download.
5. When `update.enableWindowsBackgroundUpdates` is on, which is the default, and the app is a per-user install, the update starts installing at once.
6. Otherwise the update becomes ready, and the person finishes it from the update control or the Restart to Update row (see TB-5).
7. The command is available only while the updater is idle, so it cannot be run while an update is downloading, downloaded or ready.
8. In Cursor the entry reads Developer: Apply Update... and the picker is titled Apply Update.

### WB-8.24
1. The Command Palette lists Developer: Inspect Key Mappings and Developer: Inspect Key Mappings (JSON), and neither has a key or a menu row.
2. Inspect Key Mappings opens a pinned untitled editor whose text starts with "Layout info:" and the current keyboard layout, then lists the resolved keybindings and the keyboard mapper details, and ends with "Raw mapping:" and the raw key mapping.
3. Inspect Key Mappings (JSON) opens a pinned untitled editor that holds one JSON object with the two entries `layout` and `rawMapping`.
4. Both open as untitled files, so nothing is written to disk until the person saves the tab.

### WB-8.25
1. The Command Palette lists Developer: Open URL, with no key and no menu row.
2. Running it shows an input box with the prompt "URL to open".
3. Pressing Enter with text in the box reads the text as an address and passes it to the URL service, which gives it to the handler registered for that address, and the command shows nothing of its own.
4. Pressing Escape, or accepting an empty box, does nothing.
5. In stock the box starts filled with the last address entered in this workspace, and each accepted address is remembered for the next run. [stock only]
6. In Cursor the box always starts empty and nothing is remembered. [cursor only]

## Unclear
none
