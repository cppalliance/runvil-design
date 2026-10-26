Bucket: C. Date: 2026-10-25. Pass: 4.

### NEW-C-1 Attach the debugger to a process from the Process Explorer
- Group: WB-6
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a Node-style process in the Process Explorer and choose Debug to start a debug session attached to that process.
- Commands: debug.startFromConfig; workbench.action.openProcessExplorer
- Menus: Process Explorer process row right-click menu (built in code, rows Kill Process, Force Kill Process, Copy, Copy All, Debug)
- Keys: none found
- Settings: none found
- Cursor: changed (the Process Explorer is a separate native window of older code, and its Debug row sends the same command with the same attach rules)
- Source: V\src\vs\workbench\contrib\processExplorer\browser\processExplorerControl.ts (onTreeContextMenu, isDebuggable, attachTo, DEBUG_FLAGS_PATTERN, DEBUG_PORT_PATTERN); V\src\vs\workbench\contrib\processExplorer\browser\processExplorer.contribution.ts (opens the explorer as an editor in an auxiliary window); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (debug.startFromConfig); C\out\vs\code\electron-sandbox\processExplorer\processExplorerMain.js (attachTo sends debug.startFromConfig); C bundle (debug.startFromConfig handler calling startDebugging, workbench.action.openProcessExplorer calling openProcessExplorer on the native host)
- Pieces:
  1. Right-clicking a process row whose command line has an inspect, inspect-brk or inspect-port option with a port other than 0, or that runs `node`, adds a Debug row under Copy and Copy All.
  2. Choosing Debug starts a Node attach debug session named `process <pid>`.
  3. When the command line gives an inspect port, the session attaches to that port, and with no port it attaches by process id, which signals the process to open its inspector.
  4. An inspect-port option with a port number in the command line overrides the port.
  5. Rows for other processes show no Debug row, and the row is never offered in the web version.
  6. The session starts in the main window like any other debug session, so the floating debug toolbar and the Run and Debug view react to it.
  7. Cursor opens the Process Explorer as its own native window, where stock opens it as an editor in an always-on-top auxiliary window, and its Debug row sends the same command. [cursor only]

### NEW-C-2 Show the editor right-click menu
- Group: WB-8
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click in the text of the editor, or press Shift+F10, to open the context menu that lists the editor's commands.
- Commands: editor.action.showContextMenu; editor.contrib.contextmenu (controller id, not a command)
- Menus: EditorContext; SimpleEditorContext (small editors such as the comment box)
- Keys: Shift+F10 (editor.action.showContextMenu, when textInputFocus); the Menu key also opens it
- Settings: editor.contextmenu (an editor option with no description, default true)
- Cursor: present
- Source: V\src\vs\editor\contrib\contextmenu\browser\contextmenu.ts (ContextMenuController, ShowContextMenu); V\src\vs\workbench\contrib\comments\browser\simpleCommentEditor.ts (contextMenuId SimpleEditorContext); V\src\vs\editor\common\config\editorOptions.ts (contextmenu option); C bundle (class with ID editor.contrib.contextmenu, id editor.action.showContextMenu with label Show Editor Context Menu from nls 1046 and Shift+F10)
- Pieces:
  1. Right-clicking text in the editor focuses the editor, moves the cursor to the click point unless the click is inside an existing selection, and opens the context menu at the mouse position.
  2. The menu lists the rows that features add to `EditorContext`, grouped with separators, shows each row's key binding on the right, and leaves out a submenu that has no visible rows.
  3. `Shift+F10` (Show Editor Context Menu) or the Menu key opens the same menu just below the cursor, after scrolling the cursor into view.
  4. Scrolling the mouse wheel while the menu is open closes it.
  5. Right-clicking the find box or another widget over the editor, or injected text such as an inlay hint, shows the native text menu instead.
  6. Right-clicking the scrollbar or minimap opens the minimap menu from ED-5.2 instead of this one.
  7. With `editor.contextmenu` set to false, a right-click only moves the cursor and gives no menu, and `Shift+F10` does nothing.
  8. Small editors such as the comment box use the `SimpleEditorContext` menu, which holds edit rows such as Cut, Copy, Paste, Undo and Select All instead of the full list.

### NEW-C-3 Right-click menu in the Disassembly View
- Group: ED-10
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Right-click an instruction row in the Disassembly View to copy its address or toggle a breakpoint on it.
- Commands: editor.debug.action.copyAddress; editor.debug.action.toggleBreakpoint
- Menus: DebugDisassemblyContext
- Keys: F9 (editor.debug.action.toggleBreakpoint, when the disassembly view is focused)
- Settings: none found
- Cursor: missing (the bundle has no DebugDisassemblyContext menu, no editor.debug.action.copyAddress and no Copy Address label)
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (lines 389-410, menu items); V\src\vs\workbench\contrib\debug\browser\disassemblyView.ts (onContextMenu, copyAddress command, lines 676 and 1060-1075); V\src\vs\workbench\contrib\debug\browser\debugEditorActions.ts (ToggleBreakpointAction handling a disassembly row); C bundle (searched for the menu id, the command id and the label, 0 hits)
- Pieces:
  1. Right-clicking an instruction row in the Disassembly View opens a menu, shown only while a debugger is available. [stock only]
  2. Copy Address puts that instruction's memory address on the clipboard. [stock only]
  3. Toggle Breakpoint adds an instruction breakpoint at that address, or removes the one already there. [stock only]
  4. Copy Address sits in the first group of the menu and Toggle Breakpoint in a separate breakpoints group below it. [stock only]
  5. Cursor has no Disassembly context menu and no Copy Address row. [stock only]

### NEW-C-4 Select all text in a cell output
- Group: ED-7
- Tag: both
- Evidence: source
- Boundary: no
- What: With keyboard focus inside a notebook cell's output, Ctrl+A selects all the text of that output instead of all the cells.
- Commands: notebook.cell.output.selectAll
- Menus: none found
- Keys: Ctrl+A (Cmd+A on macOS), when the notebook editor is focused and a cell output is focused
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\clipboard\notebookClipboard.ts (lines 593-625); C bundle (id notebook.cell.output.selectAll with CtrlCmd+A and the same two context keys)
- Pieces:
  1. With focus in a cell's output (reached with the focus-in-output key from ED-7.9), `Ctrl+A` selects the whole text of that output.
  2. When the focus is on a text box that lives inside the output, such as an input a renderer draws, `Ctrl+A` selects that box's contents instead.
  3. When focus is in the notebook but not inside the output page, the command does nothing.

### NEW-C-5 Retry or revert after a failed save
- Group: WB-1
- Tag: both
- Evidence: source
- Boundary: no
- What: When a save fails with an error, an error notification names the files and the error and offers Retry and Revert buttons.
- Commands: workbench.action.files.saveEditors; workbench.action.files.revertEditors
- Menus: none found (the buttons sit on the error notification)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (doSaveEditors, lines 437-462); C bundle (button ids workbench.action.files.saveEditors and workbench.action.files.revertEditors, labels Retry nls 7175, Revert All nls 7176, Revert nls 7177, message nls 7178)
- Pieces:
  1. When a save command fails with an error that is not a cancel, an error notification reads "Failed to save '<file names>': <error message>".
  2. The notification has a Retry button that runs the same save again.
  3. It also has a Revert button, or Revert All when several files failed, that discards the unsaved changes; the button is left out when every failed editor is an untitled file so that new files are never discarded this way.
  4. The notification id is built from the failed files, so a repeat failure for the same files replaces the earlier notification instead of stacking.

### NEW-C-6 Save all files except untitled ones
- Group: WB-1
- Tag: both
- Evidence: source
- Boundary: no
- What: The Command Palette entry File: Save All Files writes every unsaved editor to disk without asking for names of untitled files.
- Commands: workbench.action.files.saveFiles
- Menus: CommandPalette (File category, hidden in the Agents window)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (lines 525-532, saveAll with includeUntitled false); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (palette entry, lines 240-244); C bundle (id workbench.action.files.saveFiles with saveAll and includeUntitled false, label Save All Files from nls 7069)
- Pieces:
  1. Running File: Save All Files from the Command Palette saves every unsaved file in every editor group at once.
  2. Untitled files are skipped, so no Save As dialog opens and they stay unsaved.
  3. It saves as an explicit save, so save-time steps such as format on save run as they do for Ctrl+S.
  4. Save All in the File menu is a different command that also offers to save untitled files.

### NEW-C-7 Save without formatting
- Group: WB-1
- Tag: both
- Evidence: source
- Boundary: no
- What: Save the active file while skipping format on save and the other steps that normally run when you save.
- Commands: workbench.action.files.saveWithoutFormatting
- Menus: CommandPalette (File category)
- Keys: Ctrl+K S on Linux and Cmd+K S on macOS; Ctrl+K Ctrl+Shift+S on Windows
- Settings: editor.formatOnSave; editor.codeActionsOnSave; files.trimTrailingWhitespace; files.insertFinalNewline; files.trimFinalNewlines
- Cursor: present (id, label, palette row and the Windows chord found; the bundle adds a macOS binding that was not decoded)
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (lines 477-484, skipSaveParticipants true and force true); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (palette entry, lines 228-232); V\src\vs\workbench\contrib\codeEditor\browser\saveParticipants.ts (the steps that are skipped); V\src\vs\workbench\services\workingCopy\common\storedFileWorkingCopy.ts (line 961); C bundle (id workbench.action.files.saveWithoutFormatting with the chord and skipSaveParticipants true, label from nls 7184)
- Pieces:
  1. Pressing the chord or running File: Save without Formatting saves the active editor and skips every save-time step.
  2. The skipped steps are format on save, code actions on save, trimming trailing whitespace, inserting a final newline and trimming final newlines.
  3. The file is written even when it has no unsaved changes.

### NEW-C-8 Allow or disallow automatic tasks
- Group: WB-2
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose whether tasks marked to run when a folder opens may start by themselves; stock asks the first time and Cursor allows them by default.
- Commands: workbench.action.tasks.manageAutomaticRunning
- Menus: CommandPalette (Tasks category)
- Keys: none found
- Settings: task.allowAutomaticTasks
- Cursor: changed (the setting defaults to on and no prompt notification exists; the picker command is present)
- Source: V\src\vs\workbench\contrib\tasks\browser\runAutomaticTasks.ts (RunAutomaticTasks, _showPrompt, ManageAutomaticTaskRunning); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (palette entry lines 51-60, setting lines 535-548); C bundle (class with id workbench.action.tasks.manageAutomaticRunning and labels nls 9926 to 9928; the RunAutomaticTasks class takes no notification, storage or opener service and runs tasks unless the setting is off; setting task.allowAutomaticTasks with default on and no scope)
- Pieces:
  1. Running Tasks: Manage Automatic Tasks shows a picker with Allow Automatic Tasks and Disallow Automatic Tasks, and choosing one writes `task.allowAutomaticTasks` as `on` or `off` in the user settings; Escape changes nothing.
  2. Tasks whose run options say `folderOpen` start by themselves after the folder opens, only when the workspace is trusted and the setting allows it.
  3. With the setting at its default, the first time such tasks are found a notification asks "This workspace has tasks (...) defined (...) that can launch processes automatically when you open this workspace. Do you want to allow automatic tasks to run in all trusted workspaces?" with Allow, Disallow and Open File buttons, and it is asked once per workspace. [stock only]
  4. Allow starts the tasks and sets the setting to on, Disallow sets it to off, and Open File opens the tasks file or files and starts nothing. [stock only]
  5. Setting `task.allowAutomaticTasks` takes `on` (Always) or `off` (Never); stock defaults to `off` with application scope. [stock only]
  6. Cursor defaults the setting to `on` and has no prompt, so folder-open tasks start with no question unless the setting is `off`. [cursor only]

### NEW-C-9 Open the tasks of the workspace file
- Group: WB-2
- Tag: both
- Evidence: source
- Boundary: no
- What: In a saved multi-root workspace, Tasks: Open Workspace Tasks opens the workspace file where its tasks are kept, or helps you start a tasks section there.
- Commands: workbench.action.tasks.openWorkspaceFileTasks
- Menus: CommandPalette (Tasks category, only when the window holds a workspace file and tasks can run)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (palette entry, lines 241-248); V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (command at line 694, _getResourceForKind, _openTaskFile lines 3562-3610); C bundle (id workbench.action.tasks.openWorkspaceFileTasks, label Open Workspace Tasks from nls 9964, picker title Select a Task Template from nls 9909)
- Pieces:
  1. The Command Palette shows Tasks: Open Workspace Tasks only when the window has a saved workspace file and task execution is supported.
  2. When the workspace file already has a `tasks` section, the command opens that workspace file in an editor tab.
  3. When it has none, a picker titled "Select a Task Template" appears, and choosing a template writes it into the workspace file's `tasks` setting and opens the file as a pinned tab; Escape does nothing.
  4. In a window without a workspace file the command does nothing.

### NEW-C-10 Rerun the last task
- Group: WB-2
- Tag: both
- Evidence: source
- Boundary: no
- What: Tasks: Rerun Last Task saves your files and runs the most recent task again.
- Commands: workbench.action.tasks.reRunTask
- Menus: CommandPalette (Tasks category)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (palette entry, lines 273-280); V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (command at line 622, _reRunTaskCommand lines 3234-3250, _trust lines 3096-3110); V\src\vs\workbench\contrib\terminal\common\terminal.ts (line 570, key pass-through list); C bundle (command registration and palette entry with label Rerun Last Task from nls 9967)
- Pieces:
  1. Running Tasks: Rerun Last Task from the Command Palette runs the task that ran most recently once more.
  2. Before it runs, every unsaved editor is saved with an explicit save.
  3. When no task has run yet and none is running, it opens the Run Task list so you can pick one.
  4. In a workspace that is not trusted, a Workspace Trust request saying that listing and running tasks needs files to be executed as code comes first, and the task runs only if you agree.
  5. A key bound to this command keeps working while a terminal has focus, because the terminal passes it through.

### NEW-C-11 Show the task log
- Group: WB-2
- Tag: both
- Evidence: source
- Boundary: no
- What: Tasks: Show Task Log opens the Output panel on the Tasks channel, where task messages and configuration errors are written.
- Commands: workbench.action.tasks.showLog
- Menus: CommandPalette (Tasks category)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (palette entry, lines 258-265); V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (command at line 645, _showOutput lines 795-835, channel id tasks and label Tasks at lines 211-212); V\src\vs\workbench\contrib\terminal\common\terminal.ts (line 571); C bundle (command registration and palette entry with label Show Task Log from nls 9965)
- Pieces:
  1. Running Tasks: Show Task Log from the Command Palette opens the Output panel and selects the Tasks channel, leaving keyboard focus where it was.
  2. The Tasks channel is also in the channel list of the Output panel (PN-5.2).
  3. The warning "There are task errors. See the output for details." has a Show Output button that opens the same channel.
  4. In a virtual workspace the command does nothing.

### NEW-C-12 Move focus between commit boxes and file groups with the keyboard
- Group: SB-4
- Tag: both
- Evidence: source
- Boundary: no
- What: Four commands move keyboard focus around the Changes view, between the commit message boxes of the visible repositories and between the file groups; they have no default key, so a person has to bind them.
- Commands: workbench.scm.action.focusPreviousInput; workbench.scm.action.focusNextInput; workbench.scm.action.focusPreviousResourceGroup; workbench.scm.action.focusNextResourceGroup
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 620-670); V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (focusInput, focusResourceGroup, lines 2060-2145); C bundle (all four ids found, each opening the Source Control view and calling the matching focus method)
- Pieces:
  1. Running Focus Next Input or Focus Previous Input opens the Changes view if needed and puts the cursor in the commit message box of the active repository.
  2. When the box already has focus and several repositories are visible, the same command moves to the next or previous repository's box and wraps around at the ends.
  3. With one visible repository and focus already in its box, nothing happens.
  4. Focus Next Resource Group or Focus Previous Resource Group moves focus into the file list and selects the first visible group, such as Staged Changes or Changes, then steps to the next or previous visible group and wraps around.
  5. Groups that are not shown are skipped.
  6. None of the four does anything when no repository is visible, and none has a default key or a menu row, so they show up only once a key is assigned.

### NEW-C-13 Assign keys for commands from Accessibility Help
- Group: WB-4
- Tag: both
- Evidence: source
- Boundary: no
- What: Inside an Accessibility Help page, Alt+K lists commands that have no key so you can give them one, and Alt+A lists commands that already have a key so you can change it.
- Commands: editor.action.accessibilityHelpConfigureKeybindings; editor.action.accessibilityHelpConfigureAssignedKeybindings; workbench.action.openGlobalKeybindings
- Menus: AccessibleView (navigation group, order 3 for the unassigned list and order 4 for the assigned list)
- Keys: Alt+K (accessibilityHelpIsShown and accessibleViewHasUnassignedKeybindings); Alt+A (accessibilityHelpIsShown and accessibleViewHasAssignedKeybindings)
- Settings: none found
- Cursor: present (both ids, both chords and the labels from nls 4760 and 4761 found)
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 231-285); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (configureKeybindings, lines 447-480); V\src\vs\workbench\contrib\accessibility\common\accessibilityCommands.ts; C bundle (classes with both ids, Alt+K and Alt+A, nls 4728 to 4729 for the picker text and 4746 to 4747 for the help lines)
- Pieces:
  1. In an Accessibility Help page that lists commands without keys, `Alt+K` opens a picker titled "Configure keybindings" with the placeholder "Select a command ID to configure a keybinding for it".
  2. `Alt+A` opens the same picker for the commands on the page that already have a key.
  3. Picking a command opens the Keyboard Shortcuts editor with that command id in its search box, and Escape returns to the help page.
  4. A record-keys button for each list appears in the title toolbar of the accessible view, and only when the page has commands of that kind.
  5. The help text carries the lines "Configure keybindings for commands that lack them" and "Configure keybindings for commands that already have assignments", each followed by its key.

### NEW-C-14 Open the help link from Accessibility Help
- Group: WB-4
- Tag: both
- Evidence: source
- Boundary: no
- What: Inside an Accessibility Help page, Alt+H opens the page's "read more" link in the web browser.
- Commands: editor.action.accessibilityHelpOpenHelpLink
- Menus: none found
- Keys: Alt+H (when accessibilityHelpIsShown)
- Settings: none found
- Cursor: present (id, chord and label from nls 4762 found)
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 287-304); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (openHelpLink, lines 440-445); V\src\vs\workbench\contrib\accessibility\browser\editorAccessibilityHelp.ts (readMoreUrl); V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminalAccessibilityHelp.ts (readMoreUrl); C bundle (class with the id and Alt+H, hint text nls 4749)
- Pieces:
  1. With an Accessibility Help page open, `Alt+H` opens that page's "read more" link in the web browser, for example the editor help and the terminal help.
  2. A page without a link ignores the key, and its text does not mention it.
  3. A page with a link carries the line "Open a browser window with more information related to accessibility" followed by the key.
  4. The key works only in the help page, not in the plain accessible view.

### NEW-C-15 Turn off the accessible view hint
- Group: WB-4
- Tag: both
- Evidence: source
- Boundary: no
- What: Alt+F6 in the accessible view or Accessibility Help switches off the setting that adds the how-to-open hint for that page, and announces it.
- Commands: editor.action.accessibleViewDisableHint
- Menus: AccessibleView (navigation group, bell with a slash icon); CommandPalette
- Keys: Alt+F6 (when the accessible view or help is shown and its hint setting is on)
- Settings: accessibility.verbosity.editor; accessibility.verbosity.terminal; accessibility.verbosity.debug; accessibility.verbosity.sourceControl; accessibility.verbosity.find; accessibility.verbosity.comments; accessibility.verbosity.notebook; accessibility.verbosity.hover; accessibility.verbosity.notification; accessibility.verbosity.keybindingsEditor
- Cursor: present (id, chord and label from nls 4759 found; the individual verbosity settings were not checked)
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 200-229); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (disableHint, lines 532-538); V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (AccessibilityVerbositySettingId, lines 48-75, boolean default true); C bundle (class with the id and Alt+F6, nls 4732 for the announcement and 4744 for the help line)
- Pieces:
  1. Pressing `Alt+F6` while the accessible view or an Accessibility Help page is open sets that page's verbosity setting, such as `accessibility.verbosity.editor`, to false.
  2. A screen reader announces "<setting id> accessibility verbosity is now disabled".
  3. The bell-with-a-slash button in the accessible view title toolbar and the Command Palette row "Disable Accessible View Hint" do the same.
  4. The command is available only while the page's verbosity setting is on.
  5. Each verbosity setting is a true or false value, true by default, and the person can turn it back on in Settings.
  6. The help text lists the action as "Disable accessibility verbosity for this feature" followed by the key.

### NEW-C-16 Jump to a symbol in the accessible view
- Group: WB-4
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+Shift+O in the accessible view or Accessibility Help opens a picker of the headings, paragraphs, code blocks and list items on the page and jumps to the one you pick.
- Commands: editor.action.accessibleViewGoToSymbol
- Menus: AccessibleView (navigation group, symbol icon); CommandPalette
- Keys: Ctrl+Shift+O and Ctrl+Shift+. (when the accessible view or help is shown and the page supports symbols)
- Settings: none found
- Cursor: present (id, Ctrl+Shift+O chord and label from nls 4756 found)
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 130-157); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (goToSymbol, getSymbols, showSymbol, AccessibleViewSymbolQuickPick, lines 379-420, 998-1037); C bundle (class with the id and Ctrl+Shift+O, nls 4752 to 4753 for the picker text and 4745 for the help line)
- Pieces:
  1. Pressing `Ctrl+Shift+O` (or `Ctrl+Shift+.`) opens a picker titled "Go to Symbol Accessible View" with the placeholder "Type to search symbols".
  2. For a text page each row is a heading, paragraph, code block or list item labelled with its kind, such as "(heading) Title".
  3. Choosing a row closes the picker and shows the page scrolled to that line with the cursor in column 1.
  4. Escape closes the picker without choosing and puts focus back on the accessible view.
  5. The same action is on the title toolbar of the accessible view as a symbol button and in the Command Palette, and it is available only when the page has symbols.
  6. The help text lists the action as "Go to a symbol" followed by the key.

