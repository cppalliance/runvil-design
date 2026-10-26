Group: PN-2 Terminal sessions. Pass: 3. Date: 2026-10-25.

### PN-2.1
1. Pressing Ctrl+` (Mac Ctrl+`) or picking View > Terminal opens the panel on the Terminal tab and creates a first terminal when none exists yet.
2. Pressing Ctrl+` while the Terminal view is open and the panel has focus closes the panel instead.
3. Running Focus Terminal (`workbench.action.terminal.focus`) moves keyboard focus into the active terminal and creates a terminal when there is none.
4. With accessibility mode on and the terminal's accessible view open on its last line, pressing Ctrl+Down runs Focus Terminal and returns focus to the terminal.

### PN-2.2
1. Clicking the plus button in the Terminal view title opens a new terminal with the default profile and makes it the active tab.
2. Pressing Ctrl+Shift+` or picking Terminal > New Terminal opens a new terminal with the default profile in the default location.
3. Right-clicking the empty area of the tabs list or inside a terminal editor shows a New Terminal row that opens a new terminal.
4. Holding Alt or Ctrl while clicking the plus button splits the active terminal instead of creating a separate one.
5. Setting `terminal.integrated.defaultLocation` chooses where new terminals open, with values `view` (the panel, default) and `editor` (an editor tab).
6. In a secondary window, New Terminal opens the terminal in that window's own editor area.
7. In a remote window, the Command Palette offers `workbench.action.terminal.newInActiveWorkspace` to create a terminal in the active workspace and `workbench.action.terminal.newLocal` to create a local one.
8. In the web build, pressing Ctrl+Shift+C while no terminal has focus creates a new terminal.

### PN-2.3
1. Running Create New Terminal (With Profile) opens a pick list headed "Select the terminal profile to create".
2. The pick list has sections for configured profiles, contributed profiles and detected profiles, with the default profile first.
3. Choosing a profile row creates a terminal with that profile in the current default location.
4. Right-clicking the empty area of the tabs list and picking New Terminal With Profile opens the same pick list.
5. Holding Alt while accepting a profile row splits the active terminal with that profile instead of creating a separate terminal.
6. Clicking the gear button on a profile row asks for a name you type and saves that profile to the per-platform profiles setting.
7. Choosing a profile whose path another user can edit first shows a warning with Yes and Cancel buttons.
8. Settings `terminal.integrated.profiles.windows`, `terminal.integrated.profiles.osx` and `terminal.integrated.profiles.linux` hold the configured profiles shown in the list.
9. Setting `terminal.integrated.useWslProfiles` chooses whether detected WSL distributions appear in the list, as on or off.

### PN-2.4
1. Clicking the small chevron next to the plus button (tooltip "Launch Profile...") opens a list of profiles to create a terminal from.
2. The default profile row is marked "(Default)", and choosing any profile row creates a new terminal with that profile.
3. A submenu in the list splits the active terminal with a chosen profile.
4. The list ends with the rows Select Default Profile, Configure Terminal Settings, Run Task... and Configure Tasks...
5. The same drop-down shows on the toolbar of a terminal editor tab.
6. Settings `terminal.integrated.profiles.*` decide which profiles are listed and `terminal.integrated.defaultProfile.*` decide which one is marked "(Default)".
7. Stock puts New Terminal, New Terminal Window and Split Terminal rows at the top of the list. [stock only]
8. In Cursor the list starts straight at the profile rows with the default profile first, and its split submenu is titled "Split Terminal". [cursor only]

### PN-2.5
1. Running Terminal: Select Default Profile, or picking Select Default Profile at the bottom of the Launch Profile list, opens a pick list headed "Select your default terminal profile".
2. Choosing a profile writes it to the setting `terminal.integrated.defaultProfile.windows`, `terminal.integrated.defaultProfile.osx` or `terminal.integrated.defaultProfile.linux` for the current platform.
3. Choosing a detected profile also adds it to the per-platform `terminal.integrated.profiles.*` setting.
4. After choosing, the plus button, Ctrl+Shift+` and splits create terminals with the new default profile.
5. Setting `terminal.integrated.useWslProfiles` chooses whether detected WSL distributions appear in the pick list, as on or off.

### PN-2.6
1. Picking Terminal > New Terminal Window opens a terminal in a new compact window that holds only that terminal. [stock only]
2. Pressing Ctrl+Shift+Alt+` runs New Terminal Window (`workbench.action.terminal.newInNewWindow`) and opens the same compact window. [stock only]
3. The Launch Profile list has a New Terminal Window row that does the same. [stock only]

### PN-2.7
1. Running Create New Terminal in Editor Area opens a terminal as an editor tab.
2. Running Create New Terminal in Editor Area to the Side opens the terminal as an editor tab in a group beside the current one.
3. Clicking the plus button on a terminal editor's toolbar opens another terminal editor in the same group (`workbench.action.terminal.createTerminalEditorSameGroup`).
4. Right-clicking the editor tab bar or an empty editor group shows a New Terminal row that opens a terminal in the editor area. [stock only]
5. Setting `terminal.integrated.defaultLocation` set to `editor` makes every new terminal open as an editor tab.
6. In stock the New Terminal button also shows on the title menu of a compact window. [stock only]

### PN-2.8
1. In a window with several workspace folders, creating a terminal first shows a pick list "Select current working directory for new terminal".
2. A folder row says "(Overridden)" when that folder sets its own terminal start path. [stock only]
3. In Cursor the same folder row says "(Overriden)" with a spelling mistake. [cursor only]
4. With a single workspace folder, a new terminal starts in that folder without asking.
5. Setting `terminal.integrated.cwd` sets an explicit start folder for new terminals.
6. Setting `terminal.integrated.splitCwd` chooses the start folder of a split, with values `workspaceRoot` (with a picker when there are several roots), `initial` (the parent's initial folder) and `inherited` (the parent's current folder, default).
7. Running `workbench.action.terminal.newWithCwd` with a folder passed as an argument starts a terminal in that folder.

### PN-2.9
1. Starting a terminal in an untrusted workspace asks for trust with the message "Creating a terminal process requires executing code".
2. If trust is refused, the terminal tab shows "Cannot launch a terminal process in an untrusted workspace".
3. Setting `terminal.integrated.allowInUntrustedWorkspace` lets terminals start in an untrusted workspace without asking, as on or off. [stock only]

### PN-2.10
1. Picking Terminal > Run Selected Text sends the selected text to the active terminal and brings that terminal forward.
2. With nothing selected, Run Selected Text sends the current line.
3. Picking Terminal > Run Active File sends the file's path to the active terminal and brings that terminal forward.
4. When no terminal is open, both commands create one first.
5. Run Active File shows the warning "Only files on disk can be run in the terminal" for files that are not on disk.
6. The terminal view toolbar and the editor toolbar carry Run Selected Text and Run Active File buttons that are hidden by default.
7. Setting `terminal.integrated.focusAfterRun` chooses what gets focus after Run Selected Text, with values `terminal`, `accessible-buffer` and `none` (default).

### PN-2.11
1. Pressing F2 (Enter on Mac) on a focused tab, or picking Rename... from its right-click menu, turns the title into a text box in the tabs list.
2. In that box Enter saves the name, Esc cancels, and an empty name resets the title to its default.
3. Running Rename from the Command Palette, or from the right-click menu of a terminal editor tab, asks for the name in an "Enter terminal name" input box.
4. Running `workbench.action.terminal.renameWithArg` with a name passed as an argument renames the active terminal without asking.
5. Setting `terminal.integrated.tabs.title` sets the template for the default title (default `${process}`), and `terminal.integrated.tabs.description` sets the template for the grey description next to it (default `${task}${separator}${local}${separator}${cwdFolder}`).
6. Setting `terminal.integrated.tabs.separator` sets the separator text used by both templates (default " - ").
7. Setting `terminal.integrated.tabs.allowAgentCliTitle` chooses whether agent command line tools may set the tab title, and when off the title template is used instead. [stock only]

### PN-2.12
1. Picking Change Icon... on a tab's right-click menu opens a floating grid of every icon with a filter box.
2. Picking Change Color... lists the terminal ANSI colors and a Reset to default row.
3. With several tabs selected, the chosen icon or color is applied to all of them.
4. Right-clicking a terminal editor tab also shows Change Icon... and Change Color...
5. Setting `terminal.integrated.tabs.defaultIcon` sets the default icon for every terminal tab, as a codicon id.
6. Setting `terminal.integrated.tabs.defaultColor` sets the default icon color for every terminal tab, as a theme color id.

### PN-2.13
1. When the shell rings the bell and the bell or `terminal.integrated.enableVisualBell` setting is on, the tab briefly shows a bell icon.
2. When the connection to the shell is lost, the tab shows a status icon with the text "Lost connection to process".
3. When extensions ask for the terminal to be relaunched, the tab shows a status with a Relaunch Terminal action (`workbench.action.terminal.relaunch`).
4. When extensions have changed the terminal's environment, the tab shows a status with a Show Environment Contributions action (`workbench.action.terminal.showEnvironmentContributions`).
5. The tab shows a status for the shell integration level, with the texts Rich, Basic, No and Injection failed to activate.
6. When shell integration fails to launch, the tab shows a warning with Learn more about shell integration and Open user settings links (`workbench.action.terminal.learnMore`).
7. A status shows as the tab's icon when the tabs list is narrow and as a colored badge letter when it is wide.
8. Setting `terminal.integrated.tabs.enableAnimation` chooses whether animated status icons animate, as on or off.
9. Setting `terminal.integrated.environmentChangesRelaunch` chooses whether the relaunch status shows when extensions change the environment, as on or off.

### PN-2.14
1. Hovering a tab shows a card with the title, the process ID and the command line.
2. The card lists the text of every status on the tab and shows action links from those statuses.
3. The card ends with a Show Details or Hide Details link that adds longer text, and the choice is remembered.
4. Hovering the single active-terminal button in the view title shows the same card.
5. Pressing Ctrl+K Ctrl+I while the tabs list or a terminal has focus moves focus into the card (`workbench.action.terminal.focusHover`).

### PN-2.15
1. Beside the terminal a list shows one row per terminal with its icon, title and description.
2. Panes in one split group show a split mark (┌ ├ └) in front of their rows.
3. When the list is wide, each row has inline Split and Kill buttons.
4. By default the list appears only once there is more than one terminal, and it stays open while the mouse is over it.
5. Setting `terminal.integrated.tabs.hideCondition` chooses when the list hides, with values `never`, `singleTerminal` (default) and `singleGroup`.
6. Right-clicking the empty area of the list shows New Terminal With Profile, New Terminal and Hide Tabs, and Hide Tabs turns the list off.
7. Setting `terminal.integrated.tabs.enabled` off replaces the list with a drop-down of numbered groups plus a Show Tabs entry that turns the list back on.

### PN-2.16
1. Right-clicking the empty area of the tabs list and picking Move Tabs Left or Move Tabs Right puts the list on that side of the terminal.
2. Setting `terminal.integrated.tabs.location` chooses the side of the list, with values `left` and `right` (default).
3. Dragging the divider resizes the list, and it snaps between a narrow icon-only strip (46 px) and a wide text view (80 px to 500 px).
4. Double-clicking the divider fits the list width to the longest title, or collapses it again.
5. The width is remembered separately for a bottom or top panel and for a left or right panel.

### PN-2.17
1. With the tabs list hidden or narrow, the view title shows the active terminal as one button with its icon, title and description in its color.
2. Clicking that button focuses the terminal, dragging it moves the terminal, and hovering it shows the hover card.
3. Next to the plus button the view title shows Split and Kill buttons.
4. Setting `terminal.integrated.tabs.showActiveTerminal` chooses when the active terminal button shows, with values `always`, `singleTerminal`, `singleTerminalOrNarrow` (default) and `never`.
5. Setting `terminal.integrated.tabs.showActions` chooses when the Split and Kill buttons show, with values `always`, `singleTerminal`, `singleTerminalOrNarrow` (default) and `never`.
6. Clear Terminal, Run Active File and Run Selected Text buttons are hidden by default and can be switched on from the view toolbar's menu.

### PN-2.18
1. Clicking a row selects it, and by default a double click moves focus into that terminal.
2. Setting `terminal.integrated.tabs.focusMode` chooses how a row focuses its terminal, with values `singleClick` and `doubleClick` (default).
3. Ctrl click or Shift click selects several rows so one menu action applies to all of them.
4. Alt click on a row splits that terminal.
5. Middle click on a row kills that terminal.
6. Double click on empty space in the list creates a new terminal.
7. Right-clicking a row opens the tab menu for the selected rows.
8. Pressing Ctrl+Shift+\ moves focus into the tabs list (`workbench.action.terminal.focusTabs`) when a terminal or the list has focus.

### PN-2.19
1. Dragging a row onto another row moves the dragged group to that row's position.
2. Dropping a dragged row on empty space in the list sends it to the end.
3. Holding a drag over a row for half a second switches to that terminal.
4. Dropping a file or folder from the Explorer or the desktop onto a row pastes its path into that terminal.
5. Dropping a terminal editor tab onto the list brings that terminal into the panel.
6. Dropping a terminal from another window onto the list brings it into this window's panel.

### PN-2.20
1. Pressing Ctrl+PageDown in a terminal makes the next group active and brings the panel forward (Mac Cmd+Shift+]).
2. Pressing Ctrl+PageUp in a terminal makes the previous group active and brings the panel forward (Mac Cmd+Shift+[).
3. In a terminal editor tab, Ctrl+PageDown and Ctrl+PageUp are not taken by these commands.
4. Nine commands `workbench.action.terminal.focusAtIndex1` to `workbench.action.terminal.focusAtIndex9` have no default key and jump to the first through ninth group once you assign keys.

### PN-2.21
1. Pressing Ctrl+Shift+5 in a terminal (Mac Cmd+\) adds a second terminal next to the active one inside the same group.
2. Clicking the Split button in the view title or picking Terminal > Split Terminal does the same.
3. Picking Split on a tab's right-click menu, or pressing Ctrl+Shift+5 with the tabs list focused, splits that tab's terminal.
4. Alt clicking the plus button splits the active terminal.
5. Panes sit side by side when the panel is at the bottom or top and stacked when it is on a side.
6. In the editor area a split opens another terminal editor to the side.
7. In a remote window, `workbench.action.terminal.splitInActiveWorkspace` splits the terminal using the active workspace.

### PN-2.22
1. With a split group active, Alt+Left or Alt+Up moves focus to the previous terminal in the group (Mac Alt+Cmd with the arrow).
2. With a split group active, Alt+Right or Alt+Down moves focus to the next terminal in the group (Mac Alt+Cmd with the arrow).
3. These commands show the panel when it was hidden.

### PN-2.23
1. Dragging the divider between two panes resizes them.
2. On Linux, Ctrl+Shift+Left and Ctrl+Shift+Right with a terminal focused make the active pane larger toward that side.
3. On Mac, Ctrl+Cmd with the Left, Right, Up or Down arrow makes the active pane larger toward that side.
4. On Windows the four commands `workbench.action.terminal.resizePaneLeft`, `resizePaneRight`, `resizePaneUp` and `resizePaneDown` have no default key and can be given one.

### PN-2.24
1. Picking Unsplit Terminal on a pane's tab right-click menu moves that pane into a group of its own.
2. Selecting two or more tabs and picking Join Terminals on the right-click menu merges their groups into one split group.
3. Running Join Terminals... from the Command Palette asks you to choose a terminal and joins it with the active one.
4. With fewer than two terminals, a warning says "Insufficient terminals for the join action".
5. When all terminals are already in one split group, a warning says "All terminals are joined already".

### PN-2.25
1. Picking Move Terminal into Editor Area on a tab's right-click menu turns the selected terminals into editor tabs.
2. Running Move Terminal into Editor Area from the Command Palette moves the active terminal into an editor tab.
3. Picking Move Terminal into New Window puts the selected terminals in a window of their own.
4. Dragging a tab from the tabs list into the editor area moves that terminal into an editor tab.
5. Dragging the single tab button in the view title into the editor area does the same.

### PN-2.26
1. Picking Move Terminal into Panel on a terminal editor tab's right-click menu returns the terminal to the Terminal view.
2. Running Move Terminal into Panel from the Command Palette does the same for the active terminal editor.
3. Dragging the terminal editor tab onto the tabs list moves it back into the panel.

### PN-2.27
1. A terminal in the editor area is an editor tab showing the terminal's icon, color, status decorations, title and description.
2. The tab's toolbar has a plus button, where Alt click splits to the side, and the Launch Profile drop-down.
3. Right-clicking inside a terminal editor shows Split, New Terminal, Kill Terminal, Copy, Copy as HTML, Paste, Clear, Select All and Toggle Size to Content Width.
4. Pressing Ctrl+W in a focused terminal editor kills it (`workbench.action.terminal.killEditor`), and on Windows Ctrl+F4 is the primary key with Ctrl+W as the second.
5. Terminal editors are saved with the window layout and come back after a reload.
6. In stock the terminal editor toolbar buttons also show on the title bar of a compact window. [stock only]

### PN-2.28
1. Clicking the trash button in the view title kills the active terminal and its shell.
2. Clicking the trash button on a tab row kills that terminal.
3. Pressing Delete on a focused tab kills the terminal (Mac Cmd+Backspace or Delete).
4. Middle-clicking a tab kills its terminal.
5. Picking Kill Terminal from a terminal's right-click menu or a tab's right-click menu kills the terminal.
6. With several tabs selected, killing from a tab action ends all of the selected terminals.
7. Running Kill All Terminals from the Command Palette ends every terminal.

### PN-2.29
1. Killing a panel terminal that has a running child process asks "Do you want to terminate the active terminal session?" with a Terminate button.
2. A terminal editor tab with a running child process is marked as changed and asks "Do you want to terminate running processes?" when closed.
3. Setting `terminal.integrated.confirmOnKill` chooses where the confirmation shows, with values `never`, `editor` (default), `panel` and `always`.
4. Setting `terminal.integrated.ignoreProcessNames` lists process names that do not count as running work.

### PN-2.30
1. With `terminal.integrated.confirmOnExit` on, closing the window or quitting while terminals are running asks "Do you want to terminate the N active terminal sessions?" with a Terminate button.
2. With a single running session the question uses the single-session wording.
3. Setting `terminal.integrated.confirmOnExit` chooses when it asks, with values `never` (default), `always` and `hasChildProcesses`.
4. The question is skipped when the sessions will be kept for a reload.

### PN-2.31
1. Closing the last terminal hides the Terminal view when it is the only view in its panel tab.
2. Setting `terminal.integrated.hideOnLastClosed` off keeps the empty Terminal view open after the last terminal closes.
3. At startup with the default `terminal.integrated.hideOnStartup` value of `never`, the Terminal view stays visible and a first terminal is created when there are no sessions to restore.
4. Setting `terminal.integrated.hideOnStartup` chooses whether the view hides at startup, with values `never` (default), `whenEmpty` (hide only when no sessions were restored) and `always` (hide even when sessions were restored).

### PN-2.32
1. When a shell exits with an error, an error notification says "The terminal process ... terminated with exit code: N".
2. No exit notification shows after you end the shell yourself with Ctrl+D.
3. Setting `terminal.integrated.showExitAlert` chooses whether the exit notification shows, as on (default) or off.
4. A shell that cannot start shows "The terminal process failed to launch" with an Open Help action (`workbench.action.terminal.launchHelp`).
5. A terminal set to wait on exit keeps its tab until you press a key.
6. Running Relaunch Active Terminal restarts the shell in the same tab (`workbench.action.terminal.relaunch`).

### PN-2.33
1. After a window reload the tabs, groups, split layout, titles, icons and colors come back.
2. After a window reload a number of scrollback lines come back and the shells reattach to their running processes.
3. Setting `terminal.integrated.enablePersistentSessions` chooses whether terminal sessions persist across window reloads, as on (default) or off.
4. Setting `terminal.integrated.persistentSessionReviveProcess` chooses when content and processes are recreated after quitting or closing the window, with values `onExit` (default), `onExitAndWindowClose` and `never`.
5. A lost shell connection restarts the terminal with the message "Restarting the terminal because the connection to the shell process was lost...".
6. Setting `task.reconnection` lets background tasks with problem matchers reconnect after a reload, as on or off.

### PN-2.34
1. Running Detach Session closes the active terminal's tab while its shell keeps running in the background terminal host.
2. Running Attach to Session opens a pick list of unattached shells, each shown with its title, folder and process ID.
3. Choosing a shell in that list opens it in a new tab.
4. When there is nothing to attach, a message says "There are no unattached terminals to attach to".

### PN-2.35
1. The tab of a task's terminal shows a spinner while the task runs.
2. The tab shows a check icon when the task succeeds.
3. The tab shows an error, warning or info icon when the problem matcher finds problems.
4. An idle background task shows text ending in "and waiting..." on its tab.
5. Hovering the task state shows text such as "Task is running" or "Task has errors".
6. A marker line in the terminal notes where the detected errors for the current run begin.
7. With `terminal.integrated.tabs.enableAnimation` off, the running spinner does not animate.

### PN-2.36
1. Running a task reuses an idle terminal according to the task's `presentation.panel` option in tasks.json, with values `shared`, `dedicated` and `new`.
2. A finished task terminal stays open and prints "Terminal will be reused by tasks, press any key to close it." when `presentation.showReuseMessage` is on.
3. With panel `new`, a finished task terminal stays open and prints "Press any key to close the terminal."
4. Setting `presentation.close` to true closes the task's terminal when the task exits.
5. Setting `presentation.clear` to true clears the terminal before the task runs, and a reused terminal scrolls to the bottom.
6. Setting `presentation.group` runs the task in a named terminal group using split panes.

### PN-2.37
1. Picking Rerun Task on the right-click menu of a task's terminal runs that task again.
2. Pressing Ctrl+Shift+R with a task's terminal focused runs that task again (`workbench.action.tasks.rerunForActiveTerminal`).
3. The Rerun Task row and key apply only when the focused terminal belongs to a task.

### PN-2.38
1. Each new terminal gets a startup script injected for bash, zsh, fish and PowerShell (and Git Bash on Windows) by default.
2. Setting `terminal.integrated.shellIntegration.enabled` turns the injection on (default) or off, and only affects terminals created afterwards.
3. Setting `terminal.integrated.shellIntegration.timeout` sets how long to wait before shell integration is treated as absent. [stock only]
4. A shell that fails to start with the script is relaunched without it and the tab shows a warning.
5. The warning offers Learn more about shell integration and Open user settings links (`workbench.action.terminal.learnMore`).

### PN-2.39
1. Setting `terminal.integrated.scrollback` sets how many lines each terminal keeps in its buffer (1000 by default), and more lines use more memory.
2. Setting `terminal.integrated.persistentSessionScrollback` sets how many of those lines come back when a session is restored (100 by default).

### PN-2.40
1. Running Set Fixed Dimensions asks for a column count and then a row count in input boxes, and an empty answer means automatic.
2. A terminal with fixed dimensions larger than the panel shows scrollbars.
3. Pressing Alt+Z (Toggle Size to Content Width) widens the terminal to its longest wrapped line, and pressing it again goes back to fitting the panel.
4. Set Fixed Dimensions and Toggle Size to Content Width are rows on the tab, terminal and terminal editor right-click menus.

### PN-2.41
1. Pressing Ctrl+Shift+C (Mac Cmd+Shift+C) while no terminal has focus starts the system terminal program in the workspace folder.
2. Running Open New External Terminal from the Command Palette starts the same system terminal program.
3. With several folders open, a list titled "Select workspace folder" asks which one to start in.
4. In a remote window the folder's local path is used when it has one, and otherwise the folder of the last active file or the home folder is used.
5. Settings `terminal.external.windowsExec`, `terminal.external.osxExec` and `terminal.external.linuxExec` choose which system terminal program starts on each platform.
6. A debug launch request for an external console starts the program chosen by the same per-platform setting.

### PN-2.42
1. Settings `terminal.integrated.env.windows`, `terminal.integrated.env.osx` and `terminal.integrated.env.linux` each take an object that adds environment variables to every new terminal, and a null value deletes a variable.
2. Setting `terminal.integrated.inheritEnv` chooses whether new shells inherit the editor's own environment, which on macOS and Linux may source a login shell, and has no effect on Windows.
3. Setting `terminal.integrated.detectLocale` chooses whether the `LANG` variable is forced to a UTF-8 value, with values `auto` (default), `off` and `on`.
4. A terminal profile can carry its own `env` object.

### PN-2.43
1. Settings `terminal.integrated.automationProfile.windows`, `terminal.integrated.automationProfile.osx` and `terminal.integrated.automationProfile.linux` each take a profile object (shell path, arguments, icon, color and so on) that tasks and debug sessions use for their terminals.
2. The setting is empty by default, so tasks and debug sessions use the default profile.
3. The setting is restricted, so an untrusted workspace ignores it.
4. On Windows the setting is ignored while the old automation shell setting is set.

### PN-2.44
1. When a launch configuration asks for the integrated terminal, the debug session's program runs in a terminal tab with the debug icon.
2. The tab is named after the session title, or "Debug Process" when there is none.
3. The session starts with the per-platform `terminal.integrated.automationProfile.*` profile.
4. The panel shows the tab without taking keyboard focus.
5. The next session with the same shell, folder and title reuses a finished session's tab once its program has ended.
6. Setting `debug.terminal.clearBeforeReusing` clears the screen before a tab is reused, as on or off.

### PN-2.45
1. Setting `presentation.reveal` in a task's tasks.json entry chooses when its terminal comes forward, with values `always` (default), `silent` (only when the task exits non-zero or its problem matcher finds an error) and `never` (stays in the background).
2. Setting `presentation.focus` to true also gives the task terminal keyboard focus (off by default).
3. Setting `presentation.revealProblems` chooses when the Problems view opens, with values `never` (default), `onProblem` and `always`.
4. With `presentation.revealProblems` set to `always`, the Problems view opens instead of the terminal.
5. Setting `presentation.echo` chooses whether the "Executing task: ..." line is printed first, as on (default) or off.

### PN-2.46
1. A task's terminal tab is named after the task.
2. In a multi-root workspace the tab name also shows the folder name in brackets.
3. The tab shows the icon set by `icon.id` in tasks.json, which is a codicon id.
4. The tab shows the color set by `icon.color` in tasks.json, which is one of the terminal ANSI color ids.
5. A task started as a dependency inherits any icon id or color it does not set from the task that asked for it.
6. Each task terminal gets a Rerun Task button on its tab row.
7. The tasks.json schema offers a `presentation.preserveTerminalName` option, though it changes nothing visible. [stock only]

### PN-2.47
1. The Windows-only setting `terminal.integrated.windowsUseConptyDll` makes the terminal use the conpty.dll shipped with the editor instead of the one inside Windows.
2. In stock `terminal.integrated.windowsUseConptyDll` is on by default. [stock only]
3. In Cursor `terminal.integrated.windowsUseConptyDll` is off by default and tagged preview. [cursor only]
4. The Windows-only setting `terminal.integrated.windowsEnableConpty` chooses between ConPTY (default) and the older Winpty. [cursor only]
5. When ConPTY cannot start, the launch failure message tells you to try the conpty.dll setting.

### PN-2.48
1. Running the editor's command line tool with its wait flag from the editor's own terminal keeps that command waiting until you close the opened file.
2. After you close the file, keyboard focus moves back to the active terminal.
3. This applies only to requests from the editor's own terminal (TERM_PROGRAM is vscode), so requests from other terminals do not move focus.

### PN-2.49
1. At the first prompt of an ordinary new terminal a faint italic tip can appear after the cursor. [cursor only]
2. The tip only appears when the cursor is within the first 30 columns. [cursor only]
3. The tip is not shown in extension terminals, task terminals or other feature terminals. [cursor only]
4. The tip is only offered when the machine has the Claude or Codex command line tool but not the Cursor one. [cursor only]
5. A show count and an hours interval limit how often the tip appears. [cursor only]
6. Pressing any key removes the tip. [cursor only]
7. When the tip has an action, pressing Shift+Tab as the very first key types that action into the prompt without running it. [cursor only]

### PN-2.50
1. Install Cursor CLI appears in the Command Palette. [cursor only]
2. Running Install Cursor CLI opens a new terminal in the panel, focuses it and runs the installer line (curl https://cursor.com/install -fsS piped to bash). [cursor only]
3. When the app has already detected the tool, running Install Cursor CLI shows the message "Cursor CLI is already installed." instead. [cursor only]

## Unclear
- PN-2.50: the Command Palette flag for `cursor.installCli` is the negation of a constant the feature block could not resolve (maybe Windows), so piece 1 does not say on which platforms the command is offered.
- PN-2.13: the feature block lists only `terminal.integrated.enableVisualBell` for the bell icon, but V `terminalInstance.ts` line 860 also tests the bell setting `EnableBell`, so piece 1 says "the bell or enableVisualBell setting"; the feature's Settings line may need that setting added.
- PN-2.2: piece 8 (Ctrl+Shift+C creating a terminal in the web build) is not reachable in Cursor desktop, and the feature block does not say whether it counts in the catalog.
- PN-2.36, PN-2.45 and PN-2.46: these pieces name tasks.json `presentation` and `icon` options, not workbench settings, so a tracking tool may want to treat them as task schema behavior.
- PN-2.43 piece 4: the feature block says "the old automation shell setting" without an id, so the piece does not name it.
- All pieces come from the feature blocks plus spot checks of V source for setting values and message texts, and I did not run either app.
