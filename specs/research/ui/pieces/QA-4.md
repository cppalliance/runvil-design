Group: QA-4 Switch and open lists. Pass: 3. Date: 2026-10-25.

### QA-4.1
1. Running Open View (or typing `view ` in the quick access box) lists the views, panels, terminals and output channels you can open.
2. Choosing View > Open View... opens the same list, and the Command Palette lists Open View but not Quick Open View.
3. Pressing `Ctrl+Q` (`Control+Q` on Mac, with no default key on Linux) opens the list through `workbench.action.quickOpenView`.
4. Rows sit under heading rows for "Side Bar", "Panel" and "Secondary Side Bar", then one heading per container such as "Side Bar / Explorer", and then "Terminal", "Debug Console" and "Output".
5. Picking a row opens that container, that view, that terminal, that debug console or that Output channel.
6. Typing fuzzy-matches the view name or any part of its container name.
7. Containers marked hide-if-empty are left out of the list.
8. With no match the list shows "No matching views".

### QA-4.2
1. Running "Show Editors in Active Group By Most Recently Used" from the Command Palette (or typing `edt active `) lists only the editors of the active group, most recently used first.
2. Choosing Show Opened Editors from the "..." menu of the tab strip opens the same list.
3. Typing filters the rows on the editor name and its description.
4. Picking a row opens that editor.
5. `Ctrl+Tab` opens this list with the previous editor active and `Ctrl+Shift+Tab` with the least recently used one active (`Control+Tab` and `Control+Shift+Tab` on Mac), as long as the active group is not empty.
6. In Cursor the `Ctrl+Tab` command has extra conditions that look at agents pane focus and composer focus before it opens the list. [cursor only]

### QA-4.3
1. Running "Show All Editors By Appearance" from the Command Palette (or typing `edt `) lists every open editor in all groups, left to right and group by group.
2. Pressing `Ctrl+K Ctrl+P` (`Cmd+Alt+Tab` on Mac) opens the same list.
3. A heading row names each group when more than one group is open.
4. An editor with unsaved changes is marked in its row, and screen readers read it with "unsaved changes" added.
5. Editors in preview mode are drawn in italics.
6. In any of the three editor lists, a filter with no hits shows "No matching editors".

### QA-4.4
1. Running "Show All Editors By Most Recently Used" from the Command Palette (or typing `edt mru `) lists every open editor from all groups, newest first.
2. When more than one group is open, a heading with the group name appears each time the next row belongs to a different group.
3. Running "Quick Open Previous Recently Used Editor" from the Command Palette opens the same list with the previously used editor active.
4. Running "Quick Open Least Recently Used Editor" from the Command Palette opens the same list with the least recently used editor active.

### QA-4.5
1. Each row in the three editor lists has a "Close Editor" button, drawn as an x, that shows on hover.
2. For an editor with unsaved changes the button is drawn as a filled dot and stays visible.
3. Clicking the button closes that editor and removes its row while the list stays open.
4. Focus stays in the input box after the editor is closed.
5. If the editor is still open after the click, for example because closing was cancelled, its row stays in the list.

### QA-4.6
1. Running "Terminal: Switch Active Terminal" from the Command Palette (or typing `term `) lists the open terminals.
2. Terminals in the panel sit under a "panel" heading and terminals in editor tabs sit under an "editor" heading.
3. Each row shows the terminal icon, its color, its number and title (for example "1.2: bash"), and its description.
4. Picking a row shows that terminal and moves focus into it.
5. Pressing Right Arrow on a row opens that terminal without leaving the list.
6. Two rows at the bottom, "Create New Terminal" and "Create New Terminal With Profile...", start a new terminal.
7. The next and previous commands for this list step the active row down and up, and they have no default keys.

### QA-4.7
1. Each terminal row in the Switch Active Terminal list shows a pencil "Rename Terminal" button on hover.
2. Clicking Rename Terminal asks for a new name and leaves the list open.
3. Each terminal row also shows a trash "Kill Terminal" button on hover.
4. Clicking Kill Terminal ends that terminal at once and removes its row.

### QA-4.8
1. Choosing Terminal > Run Task... (or "Tasks: Run Task" in the Command Palette) opens a list with the placeholder "Select the task to run".
2. The list has a "recently used" section, a "configured" section, a "contributed" section with one folder row per task provider (for example npm), and a "Show All Tasks..." row.
3. When no task exists, a "Configure a Task" row opens the task setup.
4. Picking a task runs it with its problem matcher.
5. Picking a provider row opens a second step that lists that provider's tasks.
6. Each top level row has a pin button that moves the task into a "pinned" section at the top of the list.
7. Setting `task.quickOpen.history` (0 to 30, default 30) sets how many recently used tasks are kept for the "recently used" section.
8. Setting `task.quickOpen.detail` (on by default) chooses whether a task's detail line shows under its name.
9. Setting `task.quickOpen.skip` (off by default) skips the list when there is only one task to pick from.
10. Setting `task.quickOpen.showAll` (off by default) makes Run Task use the slower list of all tasks in place of the two-step list grouped by provider.

### QA-4.9
1. Typing `task ` in the quick access box shows the same top level task rows as Run Task, with the placeholder "Type the name of a task to run."
2. Typing filters the rows by task name as you type.
3. A filter with no hits shows "No matching tasks".
4. Picking a task runs it, and picking a provider folder row opens the second step.
5. The help list names this prefix "Run Task", and the command center empty-box shortcut list has a Run Task row at order 60, just before More.
6. Settings `task.quickOpen.history` and `task.quickOpen.detail` apply to this list the same way as to Run Task, limiting the recently used rows and showing or hiding the detail line.

### QA-4.10
1. Task rows in the Run Task list and the `task ` list show a gear "Configure Task" button that opens or creates the task in tasks.json and closes the list.
2. Rows in the "recently used" section also show an x "Remove Recently Used Task" button.
3. Clicking the x drops that row from the history and removes it from the list while the list stays open.
4. Setting `task.quickOpen.history` limits how many recently used rows, and so how many x buttons, can appear.

### QA-4.11
1. Choosing Terminal > Run Build Task..., or pressing `Ctrl+Shift+B` (`Cmd+Shift+B` on Mac), runs the default build task at once.
2. Running "Tasks: Run Test Task" from the Command Palette runs the default test task at once, and it has no default key.
3. When several tasks qualify, for example several defaults or several glob matches for the open file, a list "Select the build task to run" or "Select the test task to run" appears.
4. When no task qualifies, a row offers to configure one, such as "Configure Build Task...".

### QA-4.12
1. Choosing Terminal > Show Running Tasks... (or "Tasks: Show Running Tasks" in the Command Palette) shows the output of a running task.
2. The menu item is greyed out when no task is running.
3. With one running task, or several in one terminal group, its terminal is revealed at once without a list.
4. Otherwise the list "Select the task to show its output" appears.
5. When there is nothing to list, "No task is running" shows.

### QA-4.13
1. Choosing Terminal > Terminate Task... lists the running tasks under "Select a task to terminate", with an extra "All Running Tasks" row.
2. Choosing Terminal > Restart Running Task... lists the running tasks under "Select the task to restart".
3. When there is only one choice, it acts at once without showing a list.
4. An empty Terminate Task list shows "No task is currently running", and an empty Restart Running Task list shows "No task to restart".
5. Both menu items are greyed out when no task is running, and both are also in the Command Palette as "Tasks: Terminate Task" and "Tasks: Restart Running Task".

### QA-4.14
1. Choosing Terminal > Configure Tasks... opens a list "Select a task to configure" and opens the chosen task in tasks.json.
2. Choosing Terminal > Configure Default Build Task... lists tasks under "Select the task to be used as the default build task" and marks the chosen one as the default build task.
3. "Tasks: Configure Default Test Task" in the Command Palette does the same with a test task, and it has no Terminal menu item.

### QA-4.15
1. Running "Debug: Select and Start Debugging" from the Command Palette, typing `debug `, or clicking the Debug status bar item opens a list of launch configurations.
2. Launch configurations sit under a "launch.json" heading, with the last used one first under "Most Recent".
3. Entries from debug providers sit under "contributed", with recent dynamic configurations and one "Provider..." row per provider.
4. Picking a row selects that configuration and starts debugging.
5. A start that fails shows an error notification.
6. With no debugger available the list is empty.
7. While debugging with a docked toolbar, "Start Additional Session" in the menu of the Run and Debug view opens the same list.
8. Setting `debug.showInStatusBar` chooses when the Debug status bar item shows, with values `never`, `always` and `onFirstSessionStart` (the default).

### QA-4.16
1. In the Start Debugging list, a gear "Configure Launch Configuration" button on each launch.json row opens that launch file and closes the list.
2. Recent dynamic configurations show an x "Remove Launch Configuration" button that forgets them and closes the list.
3. A "configure" section at the bottom has "Add Configuration..." rows, one per launch file, that run the add configuration flow (`debug.addConfiguration`).
4. In a multi-root workspace the add rows are named "Add Config (name)...".

### QA-4.17
1. Running "Debug: Select Debug Console" from the Command Palette (or typing `debug consoles `) lists the debug sessions that have their own console.
2. Picking a session focuses it and opens the Debug Console view.
3. Pressing Right Arrow on a row does the same without leaving the list.
4. A last row, "Start a New Debug Session", opens the Start Debugging list.

### QA-4.18
1. Running "Debug: Select Debug Session" from the Command Palette opens a list with the placeholder "Search debug sessions by name".
2. Sessions that were started from a parent show the parent's name as a description.
3. The focused session is highlighted when the list opens.
4. Picking a session focuses it and opens the Debug Console.
5. A last row, "Start a New Debug Session", opens the Start Debugging list.
6. The list opens only from the command and has no quick access prefix to type.

### QA-4.19
1. Typing `issue ` in the quick access box lists the product name and "Extension Marketplace" first, then every installed non-built-in extension and extension-contributed issue entry, sorted by name. [stock only]
2. Picking a row opens the Issue Reporter for it. [stock only]
3. An info button on a row opens the extension page. [stock only]
4. Pressing Right Arrow on a row picks it without closing the list. [stock only]
5. The box placeholder reads "Type the name of an extension to report on." and the help list calls the prefix "Open Issue Reporter". [stock only]
6. The list exists only in the desktop build, and only while `telemetry.feedback.enabled` and `extensions.experimental.issueQuickAccess` (on by default) are both on. [stock only]

## Unclear

- QA-4.2 and QA-4.5: Ctrl+Tab stepping and the held-key release behavior are handed to QA-5, so the cycle commands for the editor list are not turned into pieces here.
- QA-4.4: the feature text says the list is one list, but the shared editor list code adds a group heading whenever the group changes, so piece 2 follows the source and should be checked on screen.
- QA-4.8: the pin button is described here only as a behavior of the Run Task rows, and its look is left to QA-6.24.
- QA-4.19: the feature is a judgment call because the rows come from the extension list, which is out of scope, and Cursor has no matching list, so every piece is tagged stock only.
- QA-4.2: how the Cursor agents pane and composer focus conditions change `Ctrl+Tab` was not traced, so piece 6 only states that the extra conditions exist.
- Behavior on screen was not watched. Every piece comes from source or bundle strings.
