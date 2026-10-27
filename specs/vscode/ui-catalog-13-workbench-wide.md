# 13. Workbench-wide (WB)

Things that live in no single part of the window. This section was added in pass 4 from the list of items nobody owned (decision 11), and the coverage check filled it. Cut or merge groups here like any other group. Withdrawn: the planned Zen Mode group. Zen Mode and Centered Layout were already in ED-1 and TB-4, so no leftover needed a new home.

9 groups, 75 features. Each feature lists its entry points and the pieces that finish it.

## WB-1 Saving and file state

(both). Auto save, Save, Save All, Save without formatting, retry and revert after a failed save, and new text files in the default language. The save dialogs are DL-3.

6 features.

#### WB-1.1 Save failed notification for file working copies

- Tag: both. Evidence: source. Boundary: no. Cursor: present - all six ids and the strings (nls 13563 to 13579) are in the bundle
- What: When a save fails for a file working copy, an error notification explains why and offers buttons such as Retry, Overwrite, Revert, Save As and a save as administrator; the plain text editor has its own separate save error handler.
- Commands: fileWorkingCopy.overwrite; fileWorkingCopy.revert; fileWorkingCopy.retry; fileWorkingCopy.unlock; fileWorkingCopy.saveElevated; fileWorkingCopy.saveAs (all six are button ids, not commands)
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\workingCopy\common\storedFileWorkingCopy.ts:1143-1232; C bundle (doHandleSaveError with the six button ids, nls 13563 to 13579)
- Pieces:
  1. If the file on disk is newer than the copy you edited, the notification reads "Failed to save '<name>': The content of the file is newer. Do you want to overwrite the file with your changes?" with the buttons Overwrite and Revert.
  2. If the file is read-only, it reads "Failed to save '<name>': File is read-only. Select 'Overwrite' to attempt to make it writeable." with Overwrite, Save As... and Revert.
  3. On a permission error, or a read-only file the app can overwrite with elevated rights, the first button is Retry as Admin... or Overwrite as Admin... on Windows and Retry as Sudo... or Overwrite as Sudo... elsewhere, with a message that names it.
  4. For any other error it reads "Failed to save '<name>': <error>" with Retry, Save As... and Revert.
  5. Save As... opens the Save As picker, and if that fails or is cancelled the same notification shows again.
  6. The notification closes by itself as soon as the file is saved or reverted.

#### WB-1.2 Retry or revert after a failed save

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a save fails with an error, an error notification names the files and the error and offers Retry and Revert buttons.
- Commands: workbench.action.files.saveEditors; workbench.action.files.revertEditors
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (doSaveEditors, lines 437-462); C bundle (button ids workbench.action.files.saveEditors and workbench.action.files.revertEditors, labels Retry nls 7175, Revert All nls 7176, Revert nls 7177, message nls 7178)
- Pieces:
  1. When a save command fails with an error that is not a cancel, an error notification reads "Failed to save '<file names>': <error message>".
  2. The notification has a Retry button that runs the same save again.
  3. It also has a Revert button, or Revert All when several files failed, that discards the unsaved changes; the button is left out when every failed editor is an untitled file so that new files are never discarded this way.
  4. The notification id is built from the failed files, so a repeat failure for the same files replaces the earlier notification instead of stacking.

#### WB-1.3 Save all files except untitled ones

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Command Palette entry File: Save All Files writes every unsaved editor to disk without asking for names of untitled files.
- Commands: workbench.action.files.saveFiles
- Menus: CommandPalette (File category, hidden in the Agents window)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (lines 525-532, saveAll with includeUntitled false); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (palette entry, lines 240-244); C bundle (id workbench.action.files.saveFiles with saveAll and includeUntitled false, label Save All Files from nls 7069)
- Pieces:
  1. Running File: Save All Files from the Command Palette saves every unsaved file in every editor group at once.
  2. Untitled files are skipped, so no Save As dialog opens and they stay unsaved.
  3. It saves as an explicit save, so save-time steps such as format on save run as they do for Ctrl+S.
  4. Save All in the File menu is a different command that also offers to save untitled files.

#### WB-1.4 Save without formatting

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, label, palette row and the Windows chord found; the bundle adds a macOS binding that was not decoded)
- What: Save the active file while skipping format on save and the other steps that normally run when you save.
- Commands: workbench.action.files.saveWithoutFormatting
- Menus: CommandPalette (File category)
- Keys: Ctrl+K S on Linux and Cmd+K S on macOS; Ctrl+K Ctrl+Shift+S on Windows
- Settings: editor.formatOnSave; editor.codeActionsOnSave; files.trimTrailingWhitespace; files.insertFinalNewline; files.trimFinalNewlines
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (lines 477-484, skipSaveParticipants true and force true); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (palette entry, lines 228-232); V\src\vs\workbench\contrib\codeEditor\browser\saveParticipants.ts (the steps that are skipped); V\src\vs\workbench\services\workingCopy\common\storedFileWorkingCopy.ts (line 961); C bundle (id workbench.action.files.saveWithoutFormatting with the chord and skipSaveParticipants true, label from nls 7184)
- Pieces:
  1. Pressing the chord or running File: Save without Formatting saves the active editor and skips every save-time step.
  2. The skipped steps are format on save, code actions on save, trimming trailing whitespace, inserting a final newline and trimming final newlines.
  3. The file is written even when it has no unsaved changes.

#### WB-1.5 New text file in the default language

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the command id, its label "New Untitled Text File", openEditors.newUntitledFile and files.defaultLanguage were found in the bundle and in nls; the 1_file rows of the empty-group and tab-bar menus and the key were not checked
- What: The person starts a blank untitled text file from the File menu, Ctrl+N or the Command Palette, and it opens in the language set by `files.defaultLanguage`.
- Commands: workbench.action.files.newUntitledFile; openEditors.newUntitledFile
- Menus: MenubarFileMenu (group 1_new, order 1, row New Text File); EmptyEditorGroupContext (group 1_file, order 10); EditorTabsBarContext (group 1_file, order 10); ViewTitle (Open Editors view, group navigation, order 5); CommandPalette
- Keys: Ctrl+N (Cmd+N on macOS) on workbench.action.files.newUntitledFile
- Settings: files.defaultLanguage
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (lines 661-698); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (lines 281-285 and 670-684); V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (lines 930-948); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (lines 322-325); V\src\vs\workbench\services\untitled\common\untitledTextEditorService.ts (lines 247-253); V\src\vs\workbench\services\untitled\common\untitledTextEditorModel.ts (lines 85-90); C bundle (ids workbench.action.files.newUntitledFile and openEditors.newUntitledFile, setting files.defaultLanguage, nls 7186 and 7166)
- Pieces:
  1. File > New Text File opens a new empty untitled text editor tab in the active group and keeps it open as a pinned tab.
  2. Pressing Ctrl+N (Cmd+N on macOS) does the same, and the Command Palette lists it as File: New Untitled Text File.
  3. Right-clicking the empty editor area or the tab strip offers a New Text File row, and the Open Editors view title has a New Untitled Text File button.
  4. Setting `files.defaultLanguage` names the language id that a new untitled file starts with, and an empty value leaves it unset.
  5. Setting `files.defaultLanguage` to `${activeEditorLanguage}` makes the new file start in the language of the text editor that was active before it opened, and the person can still change the language later by hand or by auto-detection.
  6. The default language is not applied when a file is created with an associated path or with an explicit language.

#### WB-1.6 Auto save

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the command id with its description, the File menu row in group 5_autosave (checked while `files.autoSave` is not off) and all four settings were found in the bundle; `files.autoSave` has the same default (off on desktop, after a delay on web) and is in Cursor's list of commonly used settings
- What: The person switches auto save on or off from the File menu or the Command Palette, or picks a mode in settings, so files with unsaved changes save themselves after a delay or when focus moves away.
- Commands: workbench.action.toggleAutoSave
- Menus: MenubarFileMenu (Auto Save, group 5_autosave, order 1, checked while files.autoSave is not off); CommandPalette (File: Toggle Auto Save)
- Keys: none found
- Settings: files.autoSave; files.autoSaveDelay; files.autoSaveWorkspaceFilesOnly; files.autoSaveWhenNoErrors
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (lines 538-556); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (lines 717-724); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (lines 261-293); V\src\vs\workbench\services\filesConfiguration\common\filesConfigurationService.ts (lines 464-475); V\src\vs\workbench\browser\parts\editor\editorAutoSave.ts; C bundle (id workbench.action.toggleAutoSave, menu group 5_autosave with the toggled condition on config.files.autoSave, the four settings, product.json commonlyUsedSettings)
- Pieces:
  1. File > Auto Save is a toggle row that shows a check mark whenever `files.autoSave` is anything other than `off`.
  2. Choosing it while auto save is off sets `files.autoSave` to `afterDelay`, and choosing it while any auto save mode is on sets it to `off`.
  3. File: Toggle Auto Save in the Command Palette does the same, with the description "Toggle the ability to save files automatically after typing".
  4. Setting `files.autoSave` chooses `off` (the default on desktop), `afterDelay`, `onFocusChange` or `onWindowChange`.
  5. With `afterDelay` an editor with unsaved changes is saved once `files.autoSaveDelay` milliseconds have passed, which is 1000 by default, and the setting only applies in that mode.
  6. With `onFocusChange` the editor saves when it loses focus, and with `onWindowChange` it saves when the window loses focus.
  7. Untitled editors and read-only editors are never auto saved.
  8. Setting `files.autoSaveWorkspaceFilesOnly` limits auto save to files inside the open workspace, and it is off by default.
  9. Setting `files.autoSaveWhenNoErrors` holds an auto save back while the file has errors in the Problems view and saves it once they are gone, and it is off by default.


## WB-2 Tasks

(both). Rerun the last task, the task log, opening the workspace tasks file, and the "allow automatic tasks" prompt. Cursor turns automatic tasks on by default with no prompt.

7 features.

#### WB-2.1 Allow or disallow automatic tasks

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the setting defaults to on and no prompt notification exists; the picker command is present)
- What: Choose whether tasks marked to run when a folder opens may start by themselves; stock asks the first time and Cursor allows them by default.
- Commands: workbench.action.tasks.manageAutomaticRunning
- Menus: CommandPalette (Tasks category)
- Keys: none found
- Settings: task.allowAutomaticTasks
- Source: V\src\vs\workbench\contrib\tasks\browser\runAutomaticTasks.ts (RunAutomaticTasks, _showPrompt, ManageAutomaticTaskRunning); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (palette entry lines 51-60, setting lines 535-548); C bundle (class with id workbench.action.tasks.manageAutomaticRunning and labels nls 9926 to 9928; the RunAutomaticTasks class takes no notification, storage or opener service and runs tasks unless the setting is off; setting task.allowAutomaticTasks with default on and no scope)
- Pieces:
  1. Running Tasks: Manage Automatic Tasks shows a picker with Allow Automatic Tasks and Disallow Automatic Tasks, and choosing one writes `task.allowAutomaticTasks` as `on` or `off` in the user settings; Escape changes nothing.
  2. Tasks whose run options say `folderOpen` start by themselves after the folder opens, only when the workspace is trusted and the setting allows it.
  3. With the setting at its default, the first time such tasks are found a notification asks "This workspace has tasks (...) defined (...) that can launch processes automatically when you open this workspace. Do you want to allow automatic tasks to run in all trusted workspaces?" with Allow, Disallow and Open File buttons, and it is asked once per workspace. [stock only]
  4. Allow starts the tasks and sets the setting to on, Disallow sets it to off, and Open File opens the tasks file or files and starts nothing. [stock only]
  5. Setting `task.allowAutomaticTasks` takes `on` (Always) or `off` (Never); stock defaults to `off` with application scope. [stock only]
  6. Cursor defaults the setting to `on` and has no prompt, so folder-open tasks start with no question unless the setting is `off`. [cursor only]

#### WB-2.2 Open the tasks of the workspace file

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a saved multi-root workspace, Tasks: Open Workspace Tasks opens the workspace file where its tasks are kept, or helps you start a tasks section there.
- Commands: workbench.action.tasks.openWorkspaceFileTasks
- Menus: CommandPalette (Tasks category, only when the window holds a workspace file and tasks can run)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (palette entry, lines 241-248); V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (command at line 694, _getResourceForKind, _openTaskFile lines 3562-3610); C bundle (id workbench.action.tasks.openWorkspaceFileTasks, label Open Workspace Tasks from nls 9964, picker title Select a Task Template from nls 9909)
- Pieces:
  1. The Command Palette shows Tasks: Open Workspace Tasks only when the window has a saved workspace file and task execution is supported.
  2. When the workspace file already has a `tasks` section, the command opens that workspace file in an editor tab.
  3. When it has none, a picker titled "Select a Task Template" appears, and choosing a template writes it into the workspace file's `tasks` setting and opens the file as a pinned tab; Escape does nothing.
  4. In a window without a workspace file the command does nothing.

#### WB-2.3 Rerun the last task

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Tasks: Rerun Last Task saves your files and runs the most recent task again.
- Commands: workbench.action.tasks.reRunTask
- Menus: CommandPalette (Tasks category)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (palette entry, lines 273-280); V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (command at line 622, _reRunTaskCommand lines 3234-3250, _trust lines 3096-3110); V\src\vs\workbench\contrib\terminal\common\terminal.ts (line 570, key pass-through list); C bundle (command registration and palette entry with label Rerun Last Task from nls 9967)
- Pieces:
  1. Running Tasks: Rerun Last Task from the Command Palette runs the task that ran most recently once more.
  2. Before it runs, every unsaved editor is saved with an explicit save.
  3. When no task has run yet and none is running, it opens the Run Task list so you can pick one.
  4. In a workspace that is not trusted, a Workspace Trust request saying that listing and running tasks needs files to be executed as code comes first, and the task runs only if you agree.
  5. A key bound to this command keeps working while a terminal has focus, because the terminal passes it through.

#### WB-2.4 Show the task log

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Tasks: Show Task Log opens the Output panel on the Tasks channel, where task messages and configuration errors are written.
- Commands: workbench.action.tasks.showLog
- Menus: CommandPalette (Tasks category)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (palette entry, lines 258-265); V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (command at line 645, _showOutput lines 795-835, channel id tasks and label Tasks at lines 211-212); V\src\vs\workbench\contrib\terminal\common\terminal.ts (line 571); C bundle (command registration and palette entry with label Show Task Log from nls 9965)
- Pieces:
  1. Running Tasks: Show Task Log from the Command Palette opens the Output panel and selects the Tasks channel, leaving keyboard focus where it was.
  2. The Tasks channel is also in the channel list of the Output panel (PN-5.2).
  3. The warning "There are task errors. See the output for details." has a Show Output button that opens the same channel.
  4. In a virtual workspace the command does nothing.

#### WB-2.5 Pick a problem matcher before a task runs

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the four row labels, the placeholder "Select for which kind of errors and warnings to scan the task output", the separator "associate", the setting and its description are all in the bundle and in nls
- What: When a task that has no problem matcher is started from the Run Task list, the `task ` quick access, or the build and test pickers, a picker asks what kind of errors and warnings to scan its output for, and offers to skip scanning, to never ask again, or to read about it.
- Commands: workbench.action.tasks.runTask; workbench.action.tasks.build; workbench.action.tasks.test (the pickers that start tasks with the prompt turned on; the prompt has no command id of its own)
- Menus: none found
- Keys: none found
- Settings: task.problemMatchers.neverPrompt
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (run lines 1443-1466, _isProblemMatcherPromptEnabled and _shouldAttachProblemMatcher lines 1473-1517, _updateNeverProblemMatcherSetting lines 1520-1533, _attachProblemMatcher lines 1535-1610, _openDocumentation line 1378, callers at lines 3145, 3190 and 3308); V\src\vs\workbench\contrib\tasks\browser\tasksQuickAccess.ts (lines 87-90); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (setting lines 470-495); C bundle (task.problemMatchers.neverPrompt) and C out\nls.messages.json (labels and placeholder)
- Pieces:
  1. Choosing a task from Run Task, from the `task ` quick access, or from the build or test picker opens a picker with the placeholder "Select for which kind of errors and warnings to scan the task output" when the task has no problem matcher, is in the build group or in no group, and can be edited in tasks.json.
  2. The top rows read "Continue without scanning the task output", "Never scan the task output for this task", "Never scan the task output for {task type} tasks" and "Learn more about scanning the task output", followed by an "associate" separator and every known problem matcher sorted by name, with its `$id` shown beside names that differ from it.
  3. Picking a problem matcher runs the task and writes that matcher into the task's entry in tasks.json, and a matcher that watches in the background also marks the task as a background task.
  4. "Continue without scanning the task output" runs the task as it is and writes nothing, so the same picker appears again the next time.
  5. "Never scan the task output for this task" writes an empty problem matcher list into the task's entry and runs it, so the picker never appears for that task again.
  6. "Never scan the task output for {task type} tasks" adds the task type to the object in `task.problemMatchers.neverPrompt` and runs the task, so no task of that type asks again.
  7. "Learn more about scanning the task output" opens the problem matcher page of the online task documentation and does not run the task.
  8. Closing the picker without choosing runs the task unchanged.
  9. Setting `task.problemMatchers.neverPrompt` set to `true` turns the picker off for every task, an object of task type to true or false turns it off for chosen types only, and the default is `false`.

#### WB-2.6 Choose what happens when a running task is started again

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - `instancePolicy` and the strings "Select an instance to terminate", "No instance is currently running" and "The instance limit for this task has been reached." were not found in the bundle or in nls; `runOptions.instanceLimit` is present
- What: When a task is started again while as many copies of it are running as its instance limit allows, its `runOptions.instancePolicy` decides whether to ask which copy to end, end the newest or the oldest, warn, or do nothing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: task.quickOpen.skip (also the tasks.json properties `runOptions.instanceLimit` and `runOptions.instancePolicy`)
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (_handleExecuteResult lines 2093-2116, _handleInstancePolicy lines 2118-2156, _restart from line 2158); V\src\vs\workbench\contrib\tasks\browser\terminalTaskSystem.ts (run lines 260-270); V\src\vs\workbench\contrib\tasks\common\jsonSchema_v2.ts (lines 391-407); V\src\vs\workbench\contrib\tasks\common\taskConfiguration.ts (lines 729-735); C bundle (instanceLimit found, instancePolicy not found)
- Pieces:
  1. `runOptions.instanceLimit` in a task's tasks.json entry sets how many copies of the task may run at once, with a default and a minimum of 1.
  2. Starting the task again while all allowed copies are running does not start another copy, and the task's terminal is brought forward if it is not the visible one before the policy is applied. [stock only]
  3. `instancePolicy` set to `prompt`, the default, opens a picker titled "Select an instance to terminate" that lists the running copies of that task, and picking one ends it and starts the task again. [stock only]
  4. When the prompt picker has no running copy to list, its single row reads "No instance is currently running" and picking it does nothing. [stock only]
  5. `terminateNewest` ends the most recently started copy and starts the task again, and `terminateOldest` does the same with the oldest copy. [stock only]
  6. `warn` shows a warning notification "The instance limit for this task has been reached." and changes nothing else. [stock only]
  7. `silent` does nothing at all when the limit is reached. [stock only]
  8. If the chosen copy cannot be ended, a warning notification says "Failed to terminate and restart task {name}". [stock only]
  9. With `task.quickOpen.skip` on and exactly one running copy, the prompt picker is skipped and that copy is ended and restarted at once. [stock only]

#### WB-2.7 Rerun All Running Tasks

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - neither the id nor the label "Rerun All Running Tasks" is in the main bundle
- What: The person runs Tasks: Rerun All Running Tasks from the Command Palette and every task that is running is ended and started again at once.
- Commands: workbench.action.tasks.rerunAllRunningTasks
- Menus: CommandPalette (Tasks category, when tasks can run)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (palette entry, lines 289-296); V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (registration, lines 634-638; _runRerunAllRunningTasksCommand, lines 3535-3546; _restart, lines 2158-2185); V\src\vs\workbench\contrib\tasks\common\tasks.ts (line 1442); C bundle (no hit for the id or the label)
- Pieces:
  1. The Command Palette lists Tasks: Rerun All Running Tasks whenever the app can run tasks, and the command has no default key. [stock only]
  2. Running it restarts every running task in one go, ending each one and then starting it again. [stock only]
  3. Before each restart the task is looked up again in the task configuration, so edits made to its definition since it started are used. [stock only]
  4. A task that cannot be ended gets the warning "Failed to terminate and restart task {name}" and is not started again, while the other tasks are still restarted. [stock only]
  5. With no task running, an information notification says "No running tasks to restart". [stock only]
  6. The command first checks Workspace Trust, as the other task commands do. [stock only]


## WB-3 Focus and navigation between parts

(both). Moving focus between a view's filter box and its list, between parts, to the view on the left, right, above, or below, the hover of the focused element, and resizing table columns by command.

5 features.

#### WB-3.1 Move between a view's filter box and its list with Ctrl+Up and Ctrl+Down

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In views that pair a filter or search box with a list, Ctrl+Down (Cmd+Down on macOS) moves keyboard focus from the box into the list and Ctrl+Up moves it back.
- Commands: widgetNavigation.focusNext; widgetNavigation.focusPrevious
- Menus: none found
- Keys: Ctrl+Down (widgetNavigation.focusNext, when a navigable container has focus and either no list is focused or the list is scrolled to the bottom); Ctrl+Up (widgetNavigation.focusPrevious, same container condition and the list is scrolled to the top); both use Cmd on macOS
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\widgetNavigationCommands.ts:144-175; V\src\vs\workbench\contrib\markers\browser\markersView.ts:204; V\src\vs\workbench\contrib\comments\browser\commentsView.ts:211; V\src\vs\workbench\contrib\comments\browser\commentThreadWidget.ts:101; V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts:273; V\src\vs\workbench\contrib\debug\browser\repl.ts:629; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts:480; V\src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts:155; C bundle (both command ids with the same keys and when clause; registrations named commentThreadWidget, keybindingsEditor, settingsEditor2, repl, testingExplorerView, markersView, commentsView and extensionsView)
- Pieces:
  1. In the Problems view and the Comments view, Ctrl+Down in the filter box moves focus to the list, and Ctrl+Up from the list moves it back to the filter box.
  2. In the Testing view, Ctrl+Down moves focus into the test tree and Ctrl+Up from the tree moves it to the filter box.
  3. In the Settings editor, Ctrl+Down in the search box moves focus to the table of contents on the left, and Ctrl+Up moves it back to the search box.
  4. In the Keyboard Shortcuts editor, Ctrl+Down in the search box moves focus to the shortcuts table, and Ctrl+Up moves it back to the search box.
  5. In the Debug Console, Ctrl+Down moves focus from the filter box to the output tree and from the tree to the input, and Ctrl+Up from the input moves focus to the tree.
  6. In an open comment thread, Ctrl+Down opens the reply box and focuses it, and Ctrl+Up from the reply box moves focus to the comments above it.
  7. When a list in these views has focus, the keys only jump between widgets once the list is scrolled to the bottom (Ctrl+Down) or the top (Ctrl+Up); before that they keep scrolling the list.

#### WB-3.2 Remove keyboard focus from the focused element

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A command that drops keyboard focus from whatever element has it, so typing no longer goes to that element; it is not in the Command Palette and is meant to be bound to a key.
- Commands: workbench.action.blur
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\windowActions.ts:442-460; C bundle (class with id workbench.action.blur, one hit)
- Pieces:
  1. Running "Remove keyboard focus from focused element" makes the focused element lose focus.
  2. Nothing happens when no element has focus or the focused item is not an HTML element.
  3. The command has no default key and no Command Palette entry, so a person reaches it only by assigning a shortcut to it in the Keyboard Shortcuts editor, where it is listed by its title.

#### WB-3.3 Navigate to the view on the left, right, above or below

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Four commands move keyboard focus from the part that has it to the visible part on the chosen side, such as editor to side bar or panel to editor.
- Commands: workbench.action.navigateLeft; workbench.action.navigateRight; workbench.action.navigateUp; workbench.action.navigateDown
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\navigationActions.ts:23-240; C bundle (class holding the neighbor logic for workbench.parts.editor, panel, sidebar and auxiliarybar, and all four ids, 2 hits each). F6 and Shift+F6 (focusNextPart, focusPreviousPart) are a different behavior in the same file and are already named in SS-1.
- Pieces:
  1. "Navigate to the View on the Left", "Navigate to the View on the Right", "Navigate to the View Above" and "Navigate to the View Below" move keyboard focus to the part that lies in that direction from the focused part.
  2. From the editor area, the command first moves to the editor group in that direction, and only leaves the editor area when no group lies that way.
  3. Coming back into the editor area from the side bar or panel focuses the active group when no group lies in between, otherwise the first group when moving right and the last group for the other directions.
  4. Landing on the side bar, panel or secondary side bar focuses the view container that is open there, and a hidden part is never a target.
  5. The commands are in the Command Palette under View and have no default keys and no menu rows.

#### WB-3.4 Resize a table column from the Command Palette

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With a table focused, run List: Resize Column to pick a column and type the width it should have as a percentage, which is the keyboard way to resize columns.
- Commands: list.resizeColumn
- Menus: CommandPalette (List category)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\list\browser\listResizeColumnAction.ts:13-34; V\src\vs\workbench\contrib\list\browser\tableColumnResizeQuickPick.ts; V\src\vs\workbench\contrib\markers\browser\markersTable.ts, V\src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts, V\src\vs\workbench\contrib\userDataProfile\browser\userDataProfilesEditor.ts, V\src\vs\workbench\contrib\workspace\browser\workspaceTrustEditor.ts (tables that use the workbench table widget); C bundle (id list.resizeColumn, nls 7533 and 7534)
- Pieces:
  1. "List: Resize Column" is offered in the palette only while a list has focus, and it does nothing unless the focused list is a table.
  2. It opens a pick list with the placeholder "Select the column to resize, type to filter." that holds the table's column headings.
  3. After you pick a column an input box asks "Please enter a width in percentage for the '<column>' column." with the placeholder "i.e. 20, 60, 100...".
  4. Text that is not a whole number shows "Please enter an integer.", and a number outside 1 to 100 shows "Please enter a number greater than 0 and less than or equal to 100."
  5. The column takes the new width at once, and cancelling either box or entering nothing leaves the table as it was.
  6. It works on every table built with the workbench table widget, such as the Problems table, the Keyboard Shortcuts editor and the Profiles editor, not only one of them.

#### WB-3.5 Show or focus the hover of the focused element

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the command and `workbench.hover.delay` are present with a `when` of not editorTextFocus; `workbench.hover.reducedDelay` has 0 hits
- What: Press Ctrl+K Ctrl+I with focus on a button, tab, status bar item or tree row to show its hover and move keyboard focus into it, without using the mouse.
- Commands: workbench.action.showHover
- Menus: none found
- Keys: Ctrl+K Ctrl+I (stock: any focus, weight EditorCore; Cursor: only when the editor does not have text focus, weight 199)
- Settings: workbench.hover.delay; workbench.hover.reducedDelay
- Source: V\src\vs\platform\hover\browser\hoverService.ts:110-123, 506-560; V\src\vs\workbench\browser\workbench.contribution.ts:733-746; C bundle (registerCommandAndKeybindingRule for workbench.action.showHover with Ctrl+K Ctrl+I and the negated editorTextFocus condition; the setting workbench.hover.delay)
- Pieces:
  1. With focus on a part of the workbench that has a hover, pressing `Ctrl+K Ctrl+I` shows that hover and puts keyboard focus inside it.
  2. The command looks at the focused element and then at each parent until it finds one with a hover, and does nothing visible when none has one.
  3. While a hover is open, holding `Alt` locks it so it stays when the pointer leaves, releasing `Alt` closes it if the pointer is outside, and any other key that is not bound hides it and returns focus to the element.
  4. Setting `workbench.hover.delay` chooses how many milliseconds the pointer rests before a workbench hover shows (500, or 1500 on macOS).
  5. Setting `workbench.hover.reducedDelay` (default 500) is the shorter delay used where faster feedback helps. [stock only]
  6. In Cursor the workbench rule is switched off while a text editor has focus, so the key there belongs to the editor's own Show or Focus Hover (ED-4). [cursor only]


## WB-4 Accessibility

(both). The accessible view, the accessibility help pages and their key assignments, and the accessibility signals (sounds and announcements) with their volume and timing settings, plus underlined links.

9 features.

#### WB-4.1 Assign keys for commands from Accessibility Help

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids, both chords and the labels from nls 4760 and 4761 found)
- What: Inside an Accessibility Help page, Alt+K lists commands that have no key so you can give them one, and Alt+A lists commands that already have a key so you can change it.
- Commands: editor.action.accessibilityHelpConfigureKeybindings; editor.action.accessibilityHelpConfigureAssignedKeybindings; workbench.action.openGlobalKeybindings
- Menus: AccessibleView (navigation group, order 3 for the unassigned list and order 4 for the assigned list)
- Keys: Alt+K (accessibilityHelpIsShown and accessibleViewHasUnassignedKeybindings); Alt+A (accessibilityHelpIsShown and accessibleViewHasAssignedKeybindings)
- Settings: none found
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 231-285); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (configureKeybindings, lines 447-480); V\src\vs\workbench\contrib\accessibility\common\accessibilityCommands.ts; C bundle (classes with both ids, Alt+K and Alt+A, nls 4728 to 4729 for the picker text and 4746 to 4747 for the help lines)
- Pieces:
  1. In an Accessibility Help page that lists commands without keys, `Alt+K` opens a picker titled "Configure keybindings" with the placeholder "Select a command ID to configure a keybinding for it".
  2. `Alt+A` opens the same picker for the commands on the page that already have a key.
  3. Picking a command opens the Keyboard Shortcuts editor with that command id in its search box, and Escape returns to the help page.
  4. A record-keys button for each list appears in the title toolbar of the accessible view, and only when the page has commands of that kind.
  5. The help text carries the lines "Configure keybindings for commands that lack them" and "Configure keybindings for commands that already have assignments", each followed by its key.

#### WB-4.2 Open the help link from Accessibility Help

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, chord and label from nls 4762 found)
- What: Inside an Accessibility Help page, Alt+H opens the page's "read more" link in the web browser.
- Commands: editor.action.accessibilityHelpOpenHelpLink
- Menus: none found
- Keys: Alt+H (when accessibilityHelpIsShown)
- Settings: none found
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 287-304); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (openHelpLink, lines 440-445); V\src\vs\workbench\contrib\accessibility\browser\editorAccessibilityHelp.ts (readMoreUrl); V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminalAccessibilityHelp.ts (readMoreUrl); C bundle (class with the id and Alt+H, hint text nls 4749)
- Pieces:
  1. With an Accessibility Help page open, `Alt+H` opens that page's "read more" link in the web browser, for example the editor help and the terminal help.
  2. A page without a link ignores the key, and its text does not mention it.
  3. A page with a link carries the line "Open a browser window with more information related to accessibility" followed by the key.
  4. The key works only in the help page, not in the plain accessible view.

#### WB-4.3 Turn off the accessible view hint

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, chord and label from nls 4759 found; the individual verbosity settings were not checked)
- What: Alt+F6 in the accessible view or Accessibility Help switches off the setting that adds the how-to-open hint for that page, and announces it.
- Commands: editor.action.accessibleViewDisableHint
- Menus: AccessibleView (navigation group, bell with a slash icon); CommandPalette
- Keys: Alt+F6 (when the accessible view or help is shown and its hint setting is on)
- Settings: accessibility.verbosity.editor; accessibility.verbosity.terminal; accessibility.verbosity.debug; accessibility.verbosity.sourceControl; accessibility.verbosity.find; accessibility.verbosity.comments; accessibility.verbosity.notebook; accessibility.verbosity.hover; accessibility.verbosity.notification; accessibility.verbosity.keybindingsEditor
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 200-229); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (disableHint, lines 532-538); V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (AccessibilityVerbositySettingId, lines 48-75, boolean default true); C bundle (class with the id and Alt+F6, nls 4732 for the announcement and 4744 for the help line)
- Pieces:
  1. Pressing `Alt+F6` while the accessible view or an Accessibility Help page is open sets that page's verbosity setting, such as `accessibility.verbosity.editor`, to false.
  2. A screen reader announces "<setting id> accessibility verbosity is now disabled".
  3. The bell-with-a-slash button in the accessible view title toolbar and the Command Palette row "Disable Accessible View Hint" do the same.
  4. The command is available only while the page's verbosity setting is on.
  5. Each verbosity setting is a true or false value, true by default, and the person can turn it back on in Settings.
  6. The help text lists the action as "Disable accessibility verbosity for this feature" followed by the key.

#### WB-4.4 Jump to a symbol in the accessible view

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, Ctrl+Shift+O chord and label from nls 4756 found)
- What: Ctrl+Shift+O in the accessible view or Accessibility Help opens a picker of the headings, paragraphs, code blocks and list items on the page and jumps to the one you pick.
- Commands: editor.action.accessibleViewGoToSymbol
- Menus: AccessibleView (navigation group, symbol icon); CommandPalette
- Keys: Ctrl+Shift+O and Ctrl+Shift+. (when the accessible view or help is shown and the page supports symbols)
- Settings: none found
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 130-157); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (goToSymbol, getSymbols, showSymbol, AccessibleViewSymbolQuickPick, lines 379-420, 998-1037); C bundle (class with the id and Ctrl+Shift+O, nls 4752 to 4753 for the picker text and 4745 for the help line)
- Pieces:
  1. Pressing `Ctrl+Shift+O` (or `Ctrl+Shift+.`) opens a picker titled "Go to Symbol Accessible View" with the placeholder "Type to search symbols".
  2. For a text page each row is a heading, paragraph, code block or list item labelled with its kind, such as "(heading) Title".
  3. Choosing a row closes the picker and shows the page scrolled to that line with the cursor in column 1.
  4. Escape closes the picker without choosing and puts focus back on the accessible view.
  5. The same action is on the title toolbar of the accessible view as a symbol button and in the Command Palette, and it is available only when the page has symbols.
  6. The help text lists the action as "Go to a symbol" followed by the key.

#### WB-4.5 Accessibility sounds and announcements for editing and run events

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (all settings and both help commands are in the bundle; the code action triggered and applied signals reuse the voice recording started and stopped sounds)
- What: Short sounds (audio cues) and screen reader announcements play when the cursor reaches an error, warning, breakpoint or folded line, and when a diff line, code action, save, format, debugger stop, task or notebook cell event happens, each switched by its own setting.
- Commands: signals.sounds.help; accessibility.announcement.help
- Menus: CommandPalette (Help: List Signal Sounds; Help: List Signal Announcements)
- Keys: none found
- Settings: accessibility.signals.codeActionApplied; accessibility.signals.codeActionTriggered; accessibility.signals.diffLineDeleted; accessibility.signals.diffLineInserted; accessibility.signals.diffLineModified; accessibility.signals.format; accessibility.signals.lineHasBreakpoint; accessibility.signals.lineHasError; accessibility.signals.lineHasFoldedArea; accessibility.signals.lineHasWarning; accessibility.signals.noInlayHints; accessibility.signals.notebookCellCompleted; accessibility.signals.notebookCellFailed; accessibility.signals.onDebugBreak; accessibility.signals.positionHasError; accessibility.signals.positionHasWarning; accessibility.signals.save; accessibility.signals.taskCompleted; accessibility.signals.taskFailed
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (the accessibility.signals.* block); V\src\vs\workbench\contrib\accessibilitySignals\browser\commands.ts (ShowSignalSoundHelp, ShowAccessibilityAnnouncementHelp); V\src\vs\workbench\contrib\accessibilitySignals\browser\editorTextPropertySignalsContribution.ts (line and position signals); V\src\vs\workbench\contrib\accessibilitySignals\browser\saveAccessibilitySignal.ts; V\src\vs\workbench\contrib\accessibilitySignals\browser\accessibilitySignalDebuggerContribution.ts; V\src\vs\platform\accessibilitySignal\browser\accessibilitySignalService.ts; V\src\vs\editor\contrib\format\browser\format.ts; V\src\vs\editor\contrib\codeAction\browser\codeAction.ts; V\src\vs\editor\browser\widget\diffEditor\diffEditorWidget.ts; V\src\vs\workbench\contrib\scm\browser\quickDiffWidget.ts; V\src\vs\workbench\contrib\tasks\browser\taskTerminalStatus.ts; V\src\vs\workbench\contrib\notebook\browser\services\notebookExecutionStateServiceImpl.ts; V\src\vs\workbench\contrib\inlayHints\browser\inlayHintsAccessibilty.ts; C bundle (settingsKey strings for every id above, ids signals.sounds.help and accessibility.announcement.help, codeActionTriggered registered with sound voiceRecordingStarted and codeActionApplied with voiceRecordingStopped)
- Pieces:
  1. Each event is one setting named `accessibility.signals.<event>` that holds a `sound` value of auto (plays when a screen reader is attached), on or off and, for most events, an `announcement` value of auto (speaks only in screen reader optimized mode) or off.
  2. Moving the cursor to a different line that has an error, a warning, a breakpoint or a folded area plays that line's sound or announcement once for the line (`lineHasError`, `lineHasWarning`, `lineHasBreakpoint`, `lineHasFoldedArea`).
  3. Placing the cursor on the text of an error or a warning, or having a marker newly appear under a still cursor, plays a separate signal (`positionHasError`, `positionHasWarning`).
  4. Stepping through changes in a diff, or moving over a changed line in a quick diff peek, plays a sound for a deleted line or a modified line (`diffLineDeleted`, `diffLineModified`, with `diffLineInserted` for added lines), sound only.
  5. Choosing a code action plays one sound when it starts and another when it has been applied (`codeActionTriggered`, `codeActionApplied`), sound only.
  6. Saving a file and formatting a file take `userGesture`, `always` or `never` for sound and for announcement, default `never`, so auto save and format on save can be told apart from the person asking for it (`save`, `format`).
  7. The debugger stopping on a breakpoint (`onDebugBreak`), a task finishing or failing with a non-zero exit code (`taskCompleted`, `taskFailed`), and a notebook cell finishing or failing (`notebookCellCompleted`, `notebookCellFailed`) each play a sound or announcement.
  8. Asking to read the inlay hints of a line that has none plays `noInlayHints`.
  9. Help: List Signal Sounds and Help: List Signal Announcements open a checkable list of every signal where the highlighted sound plays as a preview, Enter applies the on and off choices, a gear button on the save and format rows opens that setting in settings.json, and the announcements list changes nothing unless a screen reader is attached.
  10. In Cursor the code action triggered and applied signals play the voice recording started and stopped sounds instead of their own sound files. [cursor only]

#### WB-4.6 Tune the volume and timing of accessibility signals

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all five ids and the migrations from the older audioCues.volume and audioCues.debouncePositionChanges names are in the bundle)
- What: Set how loud the signal sounds are and, when debouncing is on, how long each kind of signal waits before it plays.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: accessibility.signalOptions.volume; accessibility.signalOptions.debouncePositionChanges; accessibility.signalOptions.experimental.delays.general; accessibility.signalOptions.experimental.delays.errorAtPosition; accessibility.signalOptions.experimental.delays.warningAtPosition
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (signalOptions block and migrations); V\src\vs\platform\accessibilitySignal\browser\accessibilitySignalService.ts (getDelayMs, getVolumeInPercent); V\src\vs\workbench\contrib\accessibilitySignals\browser\editorTextPropertySignalsContribution.ts; C bundle (settings accessibility.signalOptions.volume, debouncePositionChanges, delays.general, delays.errorAtPosition, delays.warningAtPosition)
- Pieces:
  1. `accessibility.signalOptions.volume` sets the loudness of every signal sound from 0 to 100 percent, default 70, and 0 makes them silent.
  2. `accessibility.signalOptions.debouncePositionChanges` (default off) makes line and position signals wait before they play, so quick cursor movement does not play every one; with it off they play at once.
  3. `accessibility.signalOptions.experimental.delays.general` sets a sound delay (default 400 ms) and an announcement delay (default 3000 ms) for every signal except an error or warning at the cursor position.
  4. `accessibility.signalOptions.experimental.delays.errorAtPosition` sets the sound delay (default 1000 ms) and announcement delay (default 3000 ms) for an error under the cursor.
  5. `accessibility.signalOptions.experimental.delays.warningAtPosition` sets the sound delay (default 1000 ms) and announcement delay (default 3000 ms) for a warning under the cursor.
  6. The three delay settings only take effect while `debouncePositionChanges` is on.
  7. Values saved under the older `audioCues.volume`, `audioCues.debouncePositionChanges` and `accessibility.signals.sounds.volume` names move to the new setting names automatically.

#### WB-4.7 Underline links in the workbench

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the setting, the class toggling code and the CSS rule for the underline-links class are in the bundle and its stylesheet)
- What: One setting draws links underlined in workbench text, hover cards, walkthrough pages, the language status popup and webviews instead of plain.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: accessibility.underlineLinks
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (accessibility.underlineLinks); V\src\vs\platform\accessibility\browser\accessibilityService.ts (toggles the underline-links class); V\src\vs\workbench\browser\media\style.css (text-link-decoration); V\src\vs\base\browser\ui\hover\hoverWidget.css; V\src\vs\workbench\contrib\welcomeWalkthrough\browser\media\walkThroughPart.css; V\src\vs\workbench\contrib\welcomeGettingStarted\browser\media\gettingStarted.css; V\src\vs\workbench\contrib\languageStatus\browser\media\languageStatus.css; V\src\vs\workbench\contrib\webview\browser\themeing.ts; C bundle (setting accessibility.underlineLinks, workbench.desktop.main.css rule .monaco-workbench.underline-links)
- Pieces:
  1. With `accessibility.underlineLinks` on, links inside paragraphs of workbench text, such as notification and dialog messages, are drawn underlined.
  2. With the setting off, the default, those links are drawn without an underline.
  3. Links in hover cards, the walkthrough and Welcome pages and the language status popup follow the same switch.
  4. Links in webview pages are given the same underline choice and change when the setting changes.
  5. Changing the setting takes effect at once without reloading the window.
  6. High contrast light and dark themes always underline links in paragraphs, whatever the setting says.

#### WB-4.8 Toggle Screen Reader Accessibility Mode

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the id, the label and both key rules are in the bundle; the two messages come through NLS indexes 898 and 899
- What: The person runs Toggle Screen Reader Accessibility Mode, or presses Shift+Alt+F1, and the editor switches between screen reader optimized mode and normal mode, with a spoken message saying which one is now on.
- Commands: editor.action.toggleScreenReaderAccessibilityMode
- Menus: CommandPalette (f1, no category)
- Keys: Shift+Alt+F1 (Shift+Alt+F4 on Linux); Ctrl+E (Cmd+E on macOS) while the accessibility help dialog is shown (accessibilityHelpIsShown)
- Settings: editor.accessibilitySupport
- Source: V\src\vs\workbench\contrib\codeEditor\browser\accessibility\accessibility.ts (ToggleScreenReaderMode, lines 22-54); V\src\vs\editor\common\config\editorOptions.ts (accessibilitySupport, lines 1452-1465); V\src\vs\editor\common\standaloneStrings.ts (lines 22-23); C bundle (class with id editor.action.toggleScreenReaderAccessibilityMode and the two key rules with weight 210)
- Pieces:
  1. Running Toggle Screen Reader Accessibility Mode from the Command Palette writes `editor.accessibilitySupport` into the User settings, as `on` when screen reader mode was off and as `off` when it was on.
  2. A screen reader announcement says "Screen Reader Optimized Mode enabled." or "Screen Reader Optimized Mode disabled." to match the new state.
  3. Shift+Alt+F1 (Shift+Alt+F4 on Linux) runs the command from anywhere.
  4. While the accessibility help dialog is open, Ctrl+E (Cmd+E on macOS) runs the command, so the mode can be switched without closing the dialog.
  5. When the setting is `auto` and a screen reader is detected, the mode already counts as on, so running the command sets the setting to `off`.
  6. While the mode is on, the status bar shows the Screen Reader Optimized item (see SS-3.26).

#### WB-4.9 Announce Cursor Position

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - neither the id nor the label "Announce Cursor Position" is in the main bundle
- What: With the editor focused and screen reader mode on, the person presses Ctrl+Alt+Shift+G and the screen reader says the cursor's line and column.
- Commands: editor.action.announceCursorPosition
- Menus: CommandPalette (f1, no category)
- Keys: Ctrl+Alt+Shift+G (Cmd+Alt+Shift+G on macOS) when editorTextFocus and accessibilityModeEnabled
- Settings: none found
- Source: V\src\vs\workbench\contrib\codeEditor\browser\accessibility\accessibility.ts (AnnounceCursorPosition, lines 56-92); V\src\vs\platform\accessibility\common\accessibility.ts (accessibilityModeEnabled, line 40); C bundle (no hit for the id or the label)
- Pieces:
  1. Typing Announce Cursor Position in the Command Palette runs the command, and it is listed whether or not screen reader mode is on. [stock only]
  2. Ctrl+Alt+Shift+G (Cmd+Alt+Shift+G on macOS) runs it, but only while an editor has text focus and screen reader mode is on. [stock only]
  3. The screen reader announcement reads "Line {line}, Column {column}" for the primary cursor. [stock only]
  4. The column is the visible column, so a tab counts as its width, which matches the position shown in the status bar. [stock only]
  5. With no focused editor, or an editor with no text model, nothing is announced. [stock only]


## WB-5 Profiles and appearance pickers

(both). Display language prompts and font smoothing on macOS. The Profiles editor, Switch Profile, and the three theme pickers turned out to be features already listed under AB-4 (Manage menu entries) and ED-8, so they are not repeated here.

2 features.

#### WB-5.1 Display language prompts

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the id, the strings (nls 7595 to 7604) and both related commands are in the bundle
- What: Notifications offer to switch the display language after a language pack is installed or when the operating system language differs, and two palette commands change it by hand.
- Commands: langugage.update.donotask (the Don't Show Again id, spelled this way in the product, not a command); workbench.action.configureLocale; workbench.action.clearLocalePreference
- Menus: CommandPalette (Configure Display Language and Clear Display Language Preference)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\localization\electron-browser\localization.contribution.ts:20-217; V\src\vs\workbench\contrib\localization\common\localizationsActions.ts:16,99; C bundle (class with checkAndInstall and onDidInstallExtension, neverShowAgain id langugage.update.donotask)
- Pieces:
  1. After you install a language pack, a sticky notification asks "Would you like to change <product>'s display language to <language> and restart?" with a Change Language and Restart button.
  2. That notification has a Don't Show Again choice that is remembered for the whole app.
  3. At startup, when the operating system language differs from the display language and no pack for it is installed, a notification offers "Install and Restart" when one matching pack is found, or "Search Marketplace" when several are, with the text in the pack's language and the English text in brackets when a translation exists.
  4. Don't Show Again on that startup notification remembers the locale, so it is not offered again for it.
  5. Removing the pack of the active language switches the display language back to English.
  6. "Configure Display Language" picks an installed language, and "Clear Display Language Preference" resets it.

#### WB-5.2 Font smoothing on macOS

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the setting is registered with the same four values and nls 4234 holds its description; Cursor's bundle also names a general setting cursor.general.fontSmoothingAntialiased_v2 whose screen was not read
- What: On macOS the person picks how text is smoothed across the whole workbench: sub-pixel, pixel level, none, or chosen by display density.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.fontAliasing
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (lines 703-716); V\src\vs\workbench\browser\workbench.ts (lines 270-295); V\src\vs\workbench\browser\media\style.css (lines 178-193); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (line 133); C bundle and C css (setting registration and the monaco-font-aliasing rules)
- Pieces:
  1. On macOS the Settings editor lists Font Aliasing (`workbench.fontAliasing`) under Workbench > Appearance, and it is not listed on Windows or Linux.
  2. Value `default` uses sub-pixel font smoothing, which gives the sharpest text on most non-retina displays.
  3. Value `antialiased` smooths text at the pixel level, which can make it look lighter overall.
  4. Value `none` turns font smoothing off, so text shows with jagged edges.
  5. Value `auto` uses antialiased smoothing on high-density displays (2x or more) and the default smoothing otherwise.
  6. A change applies to all workbench text at once, without a restart.


## WB-6 Help and diagnostics

(both). View License, Process Explorer, Troubleshoot Issue, Extension Bisect, the stock Issue Reporter, and Cursor's Send Feedback pane that replaces it.

7 features.

#### WB-6.1 Help > View License

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Help menu has a View License row that opens the product license page in the web browser, and the same command is in the Command Palette.
- Commands: workbench.action.openLicenseUrl
- Menus: MenubarHelpMenu (group 4_legal, order 1); CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\helpActions.ts:239-278, 398-400; C bundle (class with id workbench.action.openLicenseUrl, title View License, mnemonic View &&License, menu MenubarHelpMenu group 4_legal order 1); C\product.json (licenseUrl is https://cursor.com/license.txt)
- Pieces:
  1. Choosing Help > View License opens the license page in the default web browser.
  2. Running "Help: View License" from the Command Palette does the same.
  3. When the window has a display language set, the page address gets a `lang=` query added with that language.
  4. The menu row sits in the legal group of the Help menu and has the mnemonic L.
  5. The command exists only when the product has a license address; Cursor sets it to https://cursor.com/license.txt.

#### WB-6.2 Open Process Explorer

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the command, the Help row and the Developer category are present; the stock editor id `workbench.editor.processExplorer` has 0 hits, because Cursor asks its process service to open a native window styled with the theme colors; Cursor adds Open Workspace Process Explorer
- What: Run Open Process Explorer from the Help menu or the Command Palette to see every process the app runs with its CPU and memory use, and to kill, copy or debug a process.
- Commands: workbench.action.openProcessExplorer; workbench.editor.processExplorer (the editor id, not a command); workbench.action.openGlassProcessExplorer (Cursor only, related)
- Menus: MenubarHelpMenu (group 5_tools, order 2); CommandPalette (Developer category)
- Keys: Alt+E kills the selected rows while the list has focus (a key handler on the list, not a keybinding rule)
- Settings: none found
- Source: V\src\vs\workbench\contrib\processExplorer\browser\processExplorer.contribution.ts:38-171; V\src\vs\workbench\contrib\processExplorer\browser\processExplorerControl.ts (columns, context menu, Alt+E, 1 second refresh); C bundle (class with id workbench.action.openProcessExplorer, nls 7505 and 7494 for the Help row, processMainService.openProcessExplorer with theme styles, id workbench.action.openGlassProcessExplorer with nls 7506 and mode workspaceGroups)
- Pieces:
  1. Help > Open Process Explorer, or the palette entry "Developer: Open Process Explorer", opens a separate window with a tree of all processes of the app.
  2. Each row shows the columns Process Name, CPU (%), PID and Memory (MB), and the list refreshes about once a second without a button.
  3. Right-clicking a process offers Kill Process, Force Kill Process, Copy, Copy All, and Debug for Node style processes that can be attached to.
  4. Pressing `Alt+E` on the selected rows ends those processes with the same signal as Kill Process.
  5. Remote machines get their own heading in the list, and a machine that cannot be reached shows an error row.
  6. The stock window is an editor in an auxiliary window that stays on top of others, starts at 800 by 500 and remembers its size and place for next time. [stock only]
  7. Cursor draws the explorer in a native window styled with the current theme colors, not as an editor tab. [cursor only]
  8. Cursor adds "Developer: Open Workspace Process Explorer", which opens the same window in a mode that groups processes by workspace. [cursor only]

#### WB-6.3 Troubleshoot Issue

- Tag: both. Evidence: source. Boundary: no. Cursor: present - both ids found
- What: Run Troubleshoot Issue from the Command Palette to be walked through finding out whether a problem comes from an extension, from your settings, or from the product, by reloading the window several times and asking each time if you still see it.
- Commands: workbench.action.troubleshootIssue.start; workbench.action.troubleshootIssue.stop
- Menus: CommandPalette (Help category)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\issue\browser\issueTroubleshoot.ts:43-380; C bundle (ids workbench.action.troubleshootIssue.start and stop, category Help, precondition on the troubleshoot context key)
- Pieces:
  1. Running "Help: Troubleshoot Issue..." first shows a confirm box titled "Troubleshoot Issue" that says the window will reload repeatedly and that you must say each time whether you still see the issue, with a Troubleshoot Issue button.
  2. After you confirm, a temporary profile is created, the window reloads with all extensions off, and a sticky notification says "Issue troubleshooting is active and has temporarily disabled all installed extensions" with the buttons I Can't Reproduce, I Can Reproduce and Stop.
  3. Answering I Can't Reproduce at that stage means an extension causes it, so the window returns to your own profile and starts an extension bisect (WB-6.4).
  4. Answering I Can Reproduce moves to a second stage in an empty temporary profile, with a notification that says your configurations were temporarily reset to defaults and the same three buttons.
  5. In the second stage, I Can't Reproduce reports that your configuration is the cause and tells you to share an exported profile, and I Can Reproduce reports that the issue is in the product; both then offer to open the issue reporter.
  6. On a stable build, before the reporter opens, a box asks you to try the Insiders build, with buttons to download it or to report the issue anyway.
  7. Stop, from the notification or from "Help: Stop Troubleshoot Issue" (listed only while a session is active), ends the session and switches back to the profile you started in.
  8. The session survives the reloads because its stage is stored with the profile, so the notification comes back after each reload.
  9. The start command is not offered in a remote window, in the web, or while a session is already active.

#### WB-6.4 Extension Bisect

- Tag: both. Evidence: source. Boundary: no. Cursor: present - all three ids found (the Extensions view menu row was not checked)
- What: Run Start Extension Bisect to find an extension that causes a problem by turning half of your extensions off at a time and asking whether the problem is still there; it touches the extension system only as the thing it turns off.
- Commands: extension.bisect.start; extension.bisect.next; extension.bisect.stop
- Menus: CommandPalette (Help category); ViewContainerTitle (when viewContainer is workbench.view.extensions, group 2_enablement, order 4)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\extensionManagement\browser\extensionBisect.ts:165-370; C bundle (ids extension.bisect.start, next and stop, and the call that opens the issue reporter with the found extension id)
- Pieces:
  1. "Help: Start Extension Bisect" (or the matching row in the Extensions view title menu) shows a confirm box "Extension Bisect" that says a binary search will reload the window about log2(n) plus 2 times, with a Start Extension Bisect button.
  2. After you confirm, the window reloads with half of your enabled extensions off, and a sticky urgent notification says "Extension Bisect is active and has disabled N extensions" with the buttons I can't reproduce, I can reproduce and Stop Bisect.
  3. Each answer narrows the search and reloads again; running "Help: Continue Extension Bisect" from the palette asks the same question in a box with a fourth button, Cancel Bisect.
  4. When an extension is found, a box says the bisect "has identified {id} as the extension causing the problem", with a checked box "Keep this extension disabled" and the buttons Report Issue & Continue and Continue.
  5. When nothing is found, a box says the bisect is done but no extension was identified and that it might be a problem with the product.
  6. "Help: Stop Extension Bisect" (listed only while a bisect is active) or the Stop Bisect button ends the search and reloads with your extensions as they were.
  7. The start command is hidden while a bisect is active, and the continue and stop commands are hidden when none is.

#### WB-6.5 Stock issue reporter

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - the two wizard ids and both wizard settings have 0 hits; the ids workbench.action.openIssueReporter, vscode.openIssueReporter and workbench.action.reportPerformanceIssueUsingReporter exist but open Cursor's own feedback pane (WB-6.6)
- What: Choose Report Issue to describe a bug, a feature request or a performance problem in a form, or in a three step wizard with screenshots and video, that ends in a prefilled GitHub issue.
- Commands: workbench.action.openIssueReporter; vscode.openIssueReporter; workbench.action.reportPerformanceIssueUsingReporter; workbench.action.issueReporter.captureScreenshot; workbench.action.issueReporter.toggleRecording
- Menus: MenubarHelpMenu (group 3_feedback, order 3, Report Issue); CommandPalette (Help: Report Issue..., Help: Report Performance Issue...); MenubarHelpMenu (row Report Issue); CommandPalette
- Keys: Ctrl+Shift+S (capture a screenshot) and Ctrl+Shift+R (start or stop recording), Cmd instead of Ctrl on macOS, only while an issue reporter wizard tab is open; Ctrl+Shift+S (workbench.action.issueReporter.captureScreenshot) and Ctrl+Shift+R (workbench.action.issueReporter.toggleRecording), Cmd on macOS, both only while an Issue Reporter tab is open
- Settings: issueReporter.wizard.enabled; issueReporter.wizard.fullWorkspaceScan; telemetry.feedback.enabled
- Source: V\src\vs\workbench\contrib\issue\common\issue.contribution.ts:60-130; V\src\vs\workbench\contrib\issue\electron-browser\issue.contribution.ts:34-150; V\src\vs\workbench\contrib\issue\electron-browser\issueService.ts:40-70; V\src\vs\workbench\contrib\issue\browser\issueFormService.ts:421-470; V\src\vs\workbench\contrib\issue\browser\issueReporterPage.ts; V\src\vs\workbench\contrib\issue\browser\issueReporterOverlay.ts; V\src\vs\workbench\contrib\issue\electron-browser\issueReporterKeybindings.ts; V\src\vs\workbench\contrib\issue\electron-browser\issueReporterEditorPane.ts; C bundle (searched all five ids and the two settings); V\src\vs\workbench\contrib\issue\electron-browser\issue.contribution.ts (lines 47-76); V\src\vs\workbench\contrib\issue\electron-browser\nativeIssueFormService.ts (lines 59-85); V\src\vs\workbench\contrib\issue\electron-browser\issueReporterKeybindings.ts (lines 21-112); V\src\vs\workbench\contrib\issue\browser\issueReporterOverlay.ts (step titles, field labels and the attachment limit of 5); V\src\vs\workbench\contrib\issue\common\issue.contribution.ts (lines 70-130); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (lines 376-380, the Issue Reporter section with hide set to not web); C bundle (0 hits for IssueReporterEditor, IssueReporterOverlay and issueReporter.wizard)
- Pieces:
  1. Help > Report Issue and the palette entry "Help: Report Issue..." open the reporter, and "Help: Report Performance Issue..." opens it with the Performance Issue type already chosen.
  2. By default the classic reporter opens in its own small window titled Issue Reporter with a "This is a" type list, a source list, an extension picker, a required title, a required description, and checkboxes for what to include (system information, running processes, workspace metadata, enabled extensions, A/B experiment info and extension data).
  3. The classic form tells you to try the problem again after "disabling all extensions and reloading the window", which is a link, and shows errors such as "A title is required." and "Please provide a longer description."
  4. Setting `issueReporter.wizard.enabled` (default off) swaps the form for a wizard in an editor tab with the steps Attachments, Describe and Review, shown as "Step n of 3".
  5. The Attachments step takes up to 5 screenshots or videos, using a draggable floating capture bar with a Screenshot button, a drop-down for the delay (No delay, 3, 5 or 10 seconds) and Hide Toolbar in Screenshots, and a Record video button; thumbnails can be clicked to edit or removed.
  6. While a wizard tab is open, `Ctrl+Shift+S` takes a screenshot and `Ctrl+Shift+R` starts or stops recording, even when a widget that swallows keys has focus.
  7. A recording stops by itself at the 100 MB upload limit with a message, and on macOS a missing Screen Recording permission shows a message with an Open System Settings button.
  8. The Describe step asks for a category (Bug, Feature Request or Performance Issue), a target (the product, an extension, the Extensions Marketplace), a title with a "Generate from description" button, and a description with guidance text for the category, and it lists similar issues found from the title; a missing field shows a message such as "Enter a title to continue."
  9. The Review step shows the title, description and attachments, with expandable system information, extension data, extension list, A/B experiments and performance data (running processes and workspace metadata), and a last button "Preview on GitHub" (or "Open External Issue Reporter" for an extension with its own page) that reads "Uploading..." while attachments upload.
  10. When `telemetry.feedback.enabled` is off the Report Issue command only shows the notification "Feedback is disabled." [stock only]

#### WB-6.6 Cursor Send Feedback pane

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Choose Help > Report Issue or Help > Give Feedback to open a pane where you pick Idea, Bug or Urgent, describe it, attach screenshots and send it to the Cursor team.
- Commands: aiFeedback.action.open; cursor.giveFeedback; workbench.action.openIssueReporter; vscode.openIssueReporter; workbench.action.reportPerformanceIssueUsingReporter
- Menus: MenubarHelpMenu (group 3_feedback; Report Issue order 3, Give Feedback... order 4); CommandPalette
- Keys: Report Issue has a two key chord whose second key is Ctrl+G (Cmd+G on macOS); the first key is the Cursor chord prefix constant `zd` (`Oh` on macOS) that ED-4 decoded as Ctrl+M (Cmd+R), which I did not decode again
- Settings: none found
- Source: C bundle (class with id aiFeedback.action.open, label "Report Issue", nls 5368; class with id cursor.giveFeedback, label "Give Feedback", nls 5369; AIFeedbackOpenService; pane templates feedback-pane-title "Send Feedback", feedback-type-grid, feedback-screenshot-dropzone; command registrations that call the same service for the three stock ids)
- Pieces:
  1. Help > Report Issue, Help > Give Feedback..., their palette entries, and the three stock ids (`workbench.action.openIssueReporter`, `vscode.openIssueReporter`, `workbench.action.reportPerformanceIssueUsingReporter`, which ignore any prefill arguments) all open the same Send Feedback pane over the window. [cursor only]
  2. The pane has the title "Send Feedback", a close button, three type cards - Idea (Feature request), Bug (Something is off, chosen by default) and Urgent (Can't use Cursor) - and a text box that says "Describe your idea..." for Idea and "Describe the issue..." for the others. [cursor only]
  3. An "Attach screenshot" drop area, pasting an image from the clipboard, or choosing an image file adds a thumbnail with a remove button. [cursor only]
  4. The Send button, also `Ctrl+Enter` (`Cmd+Enter` on macOS), reads "Sending..." and is disabled while it works; sending with an empty text shows "Please describe your feedback", and a failure shows "Couldn't send. Try again." [cursor only]
  5. On success the pane switches to a check mark with "Thanks for your feedback!" and "Your feedback has been sent to the Cursor team.", a Done button closes it, and the saved draft is cleared. [cursor only]
  6. Cancel, the close button, `Esc` and a click on the dark backdrop all close the pane, and the chosen type, text and screenshots are kept so reopening the pane shows them again until they are sent. [cursor only]

#### WB-6.7 Attach the debugger to a process from the Process Explorer

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the Process Explorer is a separate native window of older code, and its Debug row sends the same command with the same attach rules)
- What: Right-click a Node-style process in the Process Explorer and choose Debug to start a debug session attached to that process.
- Commands: debug.startFromConfig; workbench.action.openProcessExplorer
- Menus: Process Explorer process row right-click menu (built in code, rows Kill Process, Force Kill Process, Copy, Copy All, Debug)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\processExplorer\browser\processExplorerControl.ts (onTreeContextMenu, isDebuggable, attachTo, DEBUG_FLAGS_PATTERN, DEBUG_PORT_PATTERN); V\src\vs\workbench\contrib\processExplorer\browser\processExplorer.contribution.ts (opens the explorer as an editor in an auxiliary window); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (debug.startFromConfig); C\out\vs\code\electron-sandbox\processExplorer\processExplorerMain.js (attachTo sends debug.startFromConfig); C bundle (debug.startFromConfig handler calling startDebugging, workbench.action.openProcessExplorer calling openProcessExplorer on the native host)
- Pieces:
  1. Right-clicking a process row whose command line has an inspect, inspect-brk or inspect-port option with a port other than 0, or that runs `node`, adds a Debug row under Copy and Copy All.
  2. Choosing Debug starts a Node attach debug session named `process <pid>`.
  3. When the command line gives an inspect port, the session attaches to that port, and with no port it attaches by process id, which signals the process to open its inspector.
  4. An inspect-port option with a port number in the command line overrides the port.
  5. Rows for other processes show no Debug row, and the row is never offered in the web version.
  6. The session starts in the main window like any other debug session, so the floating debug toolbar and the Run and Debug view react to it.
  7. Cursor opens the Process Explorer as its own native window, where stock opens it as an editor in an always-on-top auxiliary window, and its Debug row sends the same command. [cursor only]


## WB-7 Editor context menus and gutter menus

(both). The editor right-click menu as a whole, and hiding toolbar actions or resetting all menus.

2 features.

#### WB-7.1 Hide toolbar actions and reset all menus

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the command (nls 1783), the Hide row and the Reset Menu row (nls 1780 to 1782) are in the bundle
- What: Right-click an action button in a toolbar to hide it, and use Reset Menu or View: Reset All Menus to bring hidden actions back.
- Commands: menu.resetHiddenStates
- Menus: CommandPalette (View category); the right-click menu of menu driven toolbars (rows Hide, a checklist of hideable actions and Reset Menu)
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\actions\common\menuResetAction.ts:12-27; V\src\vs\platform\actions\browser\toolbar.ts:120-290; V\src\vs\workbench\browser\parts\editor\editorTabsControl.ts:265-298; V\src\vs\workbench\browser\parts\views\viewPane.ts:478; V\src\vs\workbench\contrib\scm\browser\scmRepositoryRenderer.ts:97; V\src\vs\workbench\contrib\notebook\browser\viewParts\notebookEditorToolbar.ts:360; C bundle (id menu.resetHiddenStates, class with resetThisMenu)
- Pieces:
  1. Right-clicking an action button in a toolbar built from a menu (the editor title bar, view title bars, Source Control repository rows, the notebook toolbar) opens a menu with Configure Keybinding when the action can have a key, then Hide '<action>' and a checklist of every hideable action in that toolbar.
  2. Choosing Hide, or unchecking an entry in the checklist, removes the button from the toolbar and moves it into the "..." overflow menu, and checking it again puts it back.
  3. When anything in that toolbar is hidden, the menu also ends with Reset Menu, which shows everything again for that toolbar, and the same row is in the menu opened from the "..." button.
  4. An action that cannot be hidden shows a greyed "Hide" label in place of Hide '<action>'.
  5. "View: Reset All Menus" brings back every hidden action in every toolbar at once and writes "did RESET all menu hidden states" to the log.
  6. Toolbars set not to allow hiding, such as the title bar, the Command Center and the side bar and panel tab strips, show no hide rows.

#### WB-7.2 Show the editor right-click menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click in the text of the editor, or press Shift+F10, to open the context menu that lists the editor's commands.
- Commands: editor.action.showContextMenu; editor.contrib.contextmenu (controller id, not a command)
- Menus: EditorContext; SimpleEditorContext (small editors such as the comment box)
- Keys: Shift+F10 (editor.action.showContextMenu, when textInputFocus); the Menu key also opens it
- Settings: editor.contextmenu (an editor option with no description, default true)
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


## WB-8 Developer commands

(both). The Developer category: toggle developer tools, inspect context keys, log storage and working copies, Screencast Mode, GPU info, performance profiles, reload with extensions disabled, and Cursor's debugging captures. The largest WB group.

25 features.

#### WB-8.1 Developer: Inspect Context Keys

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run the command, click any element in the window, and the values of all context keys that apply at that spot are written to the developer tools console.
- Commands: workbench.action.inspectContextKeys
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:56-125; C bundle (class with id workbench.action.inspectContextKeys, 1 hit)
- Pieces:
  1. After the command runs, the mouse pointer becomes a crosshair everywhere and a translucent red box follows the element under the pointer.
  2. Clicking an element writes every context key value at that element to the console of the developer tools.
  3. The click is swallowed, so the clicked element does not react to it.
  4. After one click the crosshair and the red box go away.
  5. The command is in the Command Palette under Developer and has no menu row or key; the result is seen only by opening the developer tools console.

#### WB-8.2 Developer: Log Storage Database Contents

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run the command and the contents of the storage database for the window are written to the developer tools console, then a box tells the person where to look.
- Commands: workbench.action.logStorage
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:420-441; C bundle (class with id workbench.action.logStorage, 1 hit)
- Pieces:
  1. Running the command writes the storage database contents to the developer tools console.
  2. An info box then says "The storage database contents have been logged to the developer tools." with the detail "Open developer tools from the menu and select the Console tab."
  3. The command is in the Command Palette under Developer and has no menu row or key.

#### WB-8.3 Developer: Log Working Copies

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run the command and the list of open working copies and their backups is written to the Window log, and the Output view opens on that log.
- Commands: workbench.action.logWorkingCopies
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:442-483; C bundle (class with id workbench.action.logWorkingCopies, 1 hit)
- Pieces:
  1. The log gets a Working Copies list, one line per working copy with its address and type id, and a dirty one starts with a filled circle.
  2. The log also gets a Backups list in the same line format.
  3. A list with nothing in it shows as "<none>".
  4. The Output view opens on the Window log channel without taking keyboard focus.
  5. The command is in the Command Palette under Developer and has no menu row or key.

#### WB-8.4 Developer: Remove Large Storage Database Entries

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A picker lists the biggest entries in the app's storage database, lets the person tick some, and removes them after a warning box.
- Commands: workbench.action.removeLargeStorageDatabaseEntries
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:480-590; C bundle (class with id workbench.action.removeLargeStorageDatabaseEntries and SIZE_THRESHOLD 1024*16)
- Pieces:
  1. The command opens a multi-select picker with the placeholder "Select large entries to remove from storage", one row per storage key with its size, biggest first.
  2. Each row's detail says "Scope: Global, Profile or Workspace, Target: Machine or User".
  3. In a built app only entries over 16 KB are listed, while a development build lists every key.
  4. For the default profile the Profile scope is skipped so no key shows twice.
  5. With nothing to remove, the picker's description says "There are no large storage entries to remove."
  6. The picker has a custom Remove button instead of OK and no select-all box.
  7. Pressing Remove opens a warning box "Do you want to remove the selected storage entries from the database?" that lists the keys and warns "This action is irreversible and may result in data loss!"
  8. After confirming, the entries are deleted and the storage database is compacted for each affected scope.

#### WB-8.5 Developer: Policy Diagnostics

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: Run the command and a Markdown report on account, managed-settings and policy state opens in an untitled editor tab. Stock only.
- Commands: workbench.action.showPolicyDiagnostics
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:708-1182; C bundle (0 hits for workbench.action.showPolicyDiagnostics)
- Pieces:
  1. Running the command opens a pinned untitled Markdown editor headed "VS Code Policy Diagnostics" with a warning that the file may hold sensitive information. [stock only]
  2. The report has System Information with the time generated, product name and version, and commit. [stock only]
  3. Account Information shows a summary and a property table, with the session id and analytics tracking id hidden as ***. [stock only]
  4. Account Policy Gate shows its state and reason with a legend of the possible states. [stock only]
  5. Managed Settings lists the native MDM, GitHub server, and managed-settings.json sources, how each key resolves, and the effective values. [stock only]
  6. Policy-Controlled Settings is split into Applied Policy and Non-applied Policy. [stock only]
  7. Authentication Information lists providers and session details, and any lookup that fails is written into the report as an error line. [stock only]
  8. The command is in the Command Palette under Developer and has no menu row or key. [stock only]

#### WB-8.6 Developer: Sync Account Policy

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: Run the command to force a refresh of the signed-in account's policy data, then a box says it worked or failed. Stock only.
- Commands: workbench.action.syncAccountPolicy
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:1185-1213; C bundle (0 hits for workbench.action.syncAccountPolicy)
- Pieces:
  1. Running the command forces a refresh of the default account and its policy data. [stock only]
  2. On success an info box says "Account policy has been synced." [stock only]
  3. On failure an error box says "Failed to sync account policy." with the error message as detail. [stock only]
  4. The command is in the Command Palette under Developer and has no menu row or key. [stock only]

#### WB-8.7 Developer: Toggle Screencast Mode

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run the command to show a colored circle where the mouse is pressed and a large overlay of the keys and commands being used, for demos and videos; run it again to stop.
- Commands: workbench.action.toggleScreencastMode
- Menus: CommandPalette; CommandPalette (category Developer)
- Keys: none found
- Settings: screencastMode.verticalOffset; screencastMode.fontSize; screencastMode.keyboardOptions; screencastMode.keyboardOverlayTimeout; screencastMode.mouseIndicatorColor; screencastMode.mouseIndicatorSize
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:121-415, 1230-1315; C bundle (id workbench.action.toggleScreencastMode, the six screencastMode settings with NLS 3029-3039, and screencast-keyboard and screencast-mouse rules in workbench.desktop.main.css); V\src\vs\workbench\browser\actions\developerActions.ts (lines 120-395 and 1236-1316); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (lines 157-158); C bundle (id workbench.action.toggleScreencastMode and the six settings)
- Pieces:
  1. Running the command turns Screencast Mode on, and running it again turns it off and removes the overlays.
  2. While the mouse button is down a colored circle sits over the pointer and follows it, shrinking a little while the mouse is dragged.
  3. Each key press adds a key label to a large overlay near the bottom of the window, with the command name in front of it when the keys match a shortcut.
  4. The overlay clears when a modifier key, Backspace, Escape or an arrow key starts a new entry, when more than about 20 keys are shown, or after the timeout.
  5. It works across all open windows, including separate editor windows.
  6. Setting `screencastMode.verticalOffset` sets the overlay height from the bottom as a percentage (0 to 90, default 20) and `screencastMode.fontSize` sets its font size (20 to 100, default 56).
  7. Setting `screencastMode.keyboardOptions` turns on or off raw keys, keyboard shortcuts, command names, command group names, and single editor cursor moves, with only group names off by default.
  8. Setting `screencastMode.keyboardOverlayTimeout` sets how long the overlay stays (500 to 5000 ms, default 800).
  9. Settings `screencastMode.mouseIndicatorColor` (default #FF0000) and `screencastMode.mouseIndicatorSize` (20 to 100 px, default 20) change the circle, and all six settings apply live while it is on.

#### WB-8.8 Developer: Capture and Send Debugging Data

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: Run the command to pick what to collect, describe what went wrong, and have the app capture a debugging bundle and send it for support. Boundary yes: the picker can attach agent transcripts and chat or prompt data.
- Commands: workbench.action.developer.captureAndSendDebuggingData
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (class with id workbench.action.developer.captureAndSendDebuggingData, title Capture and Send Debugging Data, NLS 11418-11441 in out/nls.messages.json; the picker ids heapSnapshot, userDirectory, logs, featureFlagsAndExtensionSummary, processDiagnostics, agentTranscripts and agentHostHeapSnapshots)
- Pieces:
  1. Running the command opens a multi-select picker titled "Capture and Send Debugging Data" with the placeholder "Select debugging data to collect. Press Enter to continue.", and the picker stays open when focus moves away. [cursor only]
  2. The picker rows are Renderer heap snapshot, User database, Logs, Feature flags and extension summary, Process diagnostics, and Agent transcripts (last 7 days), each with a one-line detail. [cursor only]
  3. Logs, Feature flags and extension summary, and Process diagnostics start ticked, and the rest start unticked. [cursor only]
  4. The details for User database and Agent transcripts warn that chat and prompt data, which may include attached code, is uploaded even in privacy mode. [cursor only]
  5. An extra row, Agent host heap snapshots, shows only when the agent host saved heap snapshots in the last day. [cursor only]
  6. After Enter, an input box asks "What went wrong? (optional)", and closing it stops the capture. [cursor only]
  7. A progress notification reads "Capturing and sending debugging data..." or, when there is no upload address, "Capturing debugging data...". [cursor only]
  8. The end shows one message: "Debugging data was sent successfully.", or "Debugging data saved locally." with a Reveal in File Explorer button, or "Failed to capture or send debugging data." with the error; which of the first two shows was not traced. [cursor only]
  9. The command is in the Command Palette under Developer and has no menu row or key. [cursor only]

#### WB-8.9 Developer: Capture Performance Profile and Show Performance Profiling Status

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: One command captures a CPU and memory profile of the current window after a confirmation, and a second command shows how many profiles are running or scheduled and whether privacy mode forbids profiling.
- Commands: workbench.action.developer.captureProfile; workbench.action.developer.showProfilingStatus
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (classes with ids workbench.action.developer.captureProfile and workbench.action.developer.showProfilingStatus; NLS 5355-5365 in out/nls.messages.json; log prefix ContinuousProfilingContribution)
- Pieces:
  1. Running "Developer: Capture Performance Profile" first opens a confirm box "Capture Performance Profile" saying a CPU and allocations profile of the current window will be created and uploaded securely for analysis, with Continue and Cancel. [cursor only]
  2. After Continue, a progress notification reads "Capturing performance profile...". [cursor only]
  3. A successful start shows an info message "Performance profile started." with a Profile ID. [cursor only]
  4. A failed start shows "Failed to start performance profile." or "Error starting performance profile: {message}". [cursor only]
  5. "Developer: Show Performance Profiling Status" shows one info message: "Performance Profiling Status: Privacy mode: enabled (profiling forbidden) or disabled (profiling allowed), Active profiles: n, Scheduled profiles: n". [cursor only]
  6. Both commands are in the Command Palette under Developer and have no menu rows or keys. [cursor only]

#### WB-8.10 Developer: Toggle Watch for Crashes and Force Renderer OOM Crash

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: One command turns crash watching on or off, and a second command crashes the window on purpose so the watcher can be tested.
- Commands: workbench.action.developer.toggleWatchForCrashes; workbench.action.developer.forceRendererOOM
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (classes with ids workbench.action.developer.toggleWatchForCrashes and workbench.action.developer.forceRendererOOM; NLS 11442-11454 in out/nls.messages.json)
- Pieces:
  1. Turning crash watching on shows a confirm box "Toggle Watch for Crashes" warning that the editor may get slower and debugging data is sent automatically if a crash happens, with Continue and Cancel. [cursor only]
  2. After the toggle, an info message says "Watching for crashes." or "Stopped watching for crashes."; turning it off asks for no confirmation. [cursor only]
  3. "Developer: Force Renderer OOM Crash" opens a confirm box saying it will use memory until the renderer crashes, the window will freeze and then reload, and it is meant for testing the watcher, with Crash Now and Cancel. [cursor only]
  4. After Crash Now the window stops responding, crashes, and reloads. [cursor only]
  5. Both commands are in the Command Palette under Developer and have no menu rows or keys. [cursor only]

#### WB-8.11 Developer: Dump Feature Gates

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Run the command to save the current feature gates to a JSON file in the home folder and get a notification that can show the file in the file manager.
- Commands: workbench.action.dumpFeatureGates
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (class with id workbench.action.dumpFeatureGates; NLS 3027, 3028 and 3053 in out/nls.messages.json)
- Pieces:
  1. Running the command writes a file named feature-gates plus a timestamp with a .json ending into the .cursor/feature-gates folder of the home folder, and creates the folder when it is missing. [cursor only]
  2. An info notification says "Feature gates dumped to" followed by the file path. [cursor only]
  3. The notification has a Reveal in File Explorer button that shows the file in the operating system's file manager. [cursor only]
  4. The command is in the Command Palette under Developer and has no menu row or key. [cursor only]

#### WB-8.12 Developer: Log Data Directories

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Run the command to log every folder the app uses for data and to show the list in a notification with a copy button.
- Commands: workbench.action.logDataDirectories
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (class with id workbench.action.logDataDirectories, title Log Data Directories, NLS 11968)
- Pieces:
  1. Running the command writes a banner and one line for each of 21 folders and files, such as User Data Directory, Logs Home, Extensions Path, Cache Home and Argv Resource, to the log. [cursor only]
  2. An info notification titled "VS Code Data Directories:" lists the same paths and ends with "To reset VS Code data, delete the User Data Directory and restart VS Code." [cursor only]
  3. The notification has a Copy to Clipboard button, and after it runs the message "Data directories copied to clipboard" or "Failed to copy to clipboard" shows. [cursor only]
  4. The command is in the Command Palette under Developer and has no menu row or key. [cursor only]

#### WB-8.13 Developer: Reload With HMR and Restart in Temporary Profile

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Two Developer commands restart the window in a special way: one reloads with hot module replacement turned on, the other relaunches the app on a throwaway user data folder.
- Commands: workbench.action.reloadWindowWithHMR; workbench.action.restartInTemporaryProfile
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (classes with ids workbench.action.reloadWindowWithHMR, calling reload with hmr true, and workbench.action.restartInTemporaryProfile, calling relaunch with a user data folder argument; NLS 11969 and 11973)
- Pieces:
  1. Running "Developer: Reload With HMR" reloads the window with hot module replacement turned on, with no confirmation. [cursor only]
  2. Running "Developer: Restart in Temporary Profile" relaunches the whole app with no confirmation. [cursor only]
  3. The relaunched app uses a new empty user data folder named vscode-temp-profile plus a timestamp in the temp folder, so it starts with default settings and no saved window state. [cursor only]
  4. The normal user data folder is left untouched by the restart. [cursor only]
  5. Both commands are in the Command Palette under Developer and have no menu rows or keys. [cursor only]

#### WB-8.14 Toggle Developer Tools

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the command, the Help row and the key are all present, and the key has no development-build condition
- What: Run Toggle Developer Tools to open or close the Chromium developer tools of the window you are in, from the Help menu, the Command Palette or a key.
- Commands: workbench.action.toggleDevTools
- Menus: MenubarHelpMenu (group 5_tools, order 1); CommandPalette (Developer category)
- Keys: Ctrl+Shift+I (Cmd+Option+I on macOS); stock binds it only when the `isDevelopment` context is true, Cursor binds it with no `when` at weight 250
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:23-50; V\src\vs\workbench\electron-browser\desktop.contribution.ts:116; C bundle (id workbench.action.toggleDevTools, nls 11996, menu MenubarHelpMenu group 5_tools order 1, keybinding Ctrl|Cmd+Shift+I with mac Cmd+Alt+I)
- Pieces:
  1. Choosing Help > Toggle Developer Tools opens the developer tools for the active window, and choosing it again closes them.
  2. The Command Palette entry "Developer: Toggle Developer Tools" does the same thing.
  3. With several windows open the command acts only on the window that has focus.
  4. Pressing `Ctrl+Shift+I` (`Cmd+Option+I` on macOS) toggles the tools in every Cursor build. [cursor only]
  5. In a release build of stock the key does nothing, because it is bound only in a development build, while the Help row and the palette entry still work. [stock only]

#### WB-8.15 Reload with extensions disabled

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run Reload with Extensions Disabled from the Command Palette to reload the window with every extension turned off, to see whether a problem comes from an extension.
- Commands: workbench.action.reloadWindowWithExtensionsDisabled
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:74-87; V\src\vs\platform\native\electron-main\nativeHostMainService.ts:1089-1110 (reload passes the disable-extensions flag for that reload); C bundle (id workbench.action.reloadWindowWithExtensionsDisabled, nls 11998, category Developer)
- Pieces:
  1. Running "Developer: Reload with Extensions Disabled" reloads the window at once, with no question asked first, and the window comes back with all extensions off.
  2. The command exists only in the Command Palette under Developer, with no menu row and no key.
  3. The extensions are off for that reload only, so an ordinary Reload Window afterwards brings them back (the flag is passed for one reload; I did not run it).

#### WB-8.16 Reveal User Data Folder

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run Reveal User Data Folder from the Command Palette to show the folder where the app keeps its user data in the operating system's file manager.
- Commands: workbench.action.revealUserDataFolder
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:90-106; C bundle (id workbench.action.revealUserDataFolder, nls 11999, category Developer)
- Pieces:
  1. Running "Developer: Reveal User Data Folder" opens the file manager (Explorer on Windows, Finder on macOS) with the user data folder selected in its parent folder.
  2. Nothing shows inside the editor window itself, so there is no dialog, notification or editor tab.
  3. The command exists only in the Command Palette under Developer, with no menu row and no key.

#### WB-8.17 Show GPU info

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run Show GPU Info from the Command Palette to open Chromium's GPU status page in its own window.
- Commands: workbench.action.showGPUInfo
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:109-123; V\src\vs\platform\native\electron-main\nativeHostMainService.ts:1214-1234 (child window on chrome://gpu); C bundle (id workbench.action.showGPUInfo, nls 12000, category Developer)
- Pieces:
  1. Running "Developer: Show GPU Info" opens a separate window, a child of the current window, that shows the Chromium `chrome://gpu` page.
  2. Running the command again while that window is open does not make a second one: it restores the window if it is minimized and brings it to the front.
  3. After you close the window, running the command opens a fresh one.
  4. The command exists only in the Command Palette under Developer, with no menu row and no key.

#### WB-8.18 Show content tracing

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - 0 hits for the id in the bundle (Cursor keeps `workbench.action.stopTracing` and has its own Start Electron Trace instead)
- What: Run Show Content Tracing from the Command Palette to open Chromium's tracing recorder page in its own window.
- Commands: workbench.action.showContentTracing
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:126-141; V\src\vs\platform\native\electron-main\nativeHostMainService.ts:1236-1268 (window on chrome://tracing); C bundle (searched id, 0 hits)
- Pieces:
  1. Running "Developer: Show Content Tracing" opens a window with the Chromium `chrome://tracing` page, which stays hidden until the page has loaded and then shows. [stock only]
  2. Unlike the GPU info window, the tracing window is not a child of the editor window that opened it, so it has no parent to follow. [stock only]
  3. Running the command again while the window is open restores it if minimized and brings it to the front instead of opening another. [stock only]
  4. Cursor has no such command in its Command Palette. [stock only]

#### WB-8.19 Force refresh server config

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Run Force Refresh Server Config from the Command Palette to make Cursor fetch its server-side configuration again right now, and see a notification with the result.
- Commands: aiServerConfigService.forceRefresh
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Source: C bundle (action class with id aiServerConfigService.forceRefresh, title "Force Refresh Server Config", category Developer, f1 true; run calls forceRefreshServerConfig with a request id and rethrow)
- Pieces:
  1. Running "Developer: Force Refresh Server Config" asks the server for the configuration again without waiting for the normal refresh. [cursor only]
  2. On success an info notification "Server config refreshed" appears, with the line "Request ID: <id>" under it. [cursor only]
  3. On failure an error notification "Failed to refresh server config" appears with the same Request ID line. [cursor only]
  4. The command exists only in the Command Palette, with no menu row and no key. [cursor only]

#### WB-8.20 Capture and upload a process snapshot

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Run Capture and Upload Process Snapshot from the Command Palette to save a debugging snapshot of the window's memory use to disk and send it to Cursor.
- Commands: developer.captureAndUploadProcessSnapshot; cpuMonitor.triggerManualSnapshot
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Source: C bundle (class with id developer.captureAndUploadProcessSnapshot, nls 5367, which runs the command cpuMonitor.triggerManualSnapshot registered by the memory monitor contribution; methods _captureAndUploadSnapshot and _uploadSavedSnapshot)
- Pieces:
  1. Running "Developer: Capture and Upload Process Snapshot" starts the capture straight away, with no confirmation first, and writes the snapshot file to disk. [cursor only]
  2. While the file uploads, a progress message "Uploading snapshot to Cursor..." shows in the window. [cursor only]
  3. A notification then says "Snapshot uploaded successfully. Thank you!", or "Snapshot upload failed. The snapshot is still saved on disk.", or "Could not upload snapshot: unable to get upload URL." [cursor only]
  4. If the capture itself fails, an error notification reads "An error occurred while capturing debug snapshot: <message>". [cursor only]
  5. Running the command while another check or snapshot is in progress shows nothing and only writes a warning to the log. [cursor only]

#### WB-8.21 Toggle sign-in debugging

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Run Developer: Toggle Auth Debugging from the Command Palette to switch sign-in debug logging on or off and open the Cursor Auth Debug output channel.
- Commands: cursorAuth.toggleAuthDebug
- Menus: commandPalette (no category, no condition)
- Keys: none found
- Settings: none found
- Source: C bundle (action cursorAuth.toggleAuthDebug, title "Developer: Toggle Auth Debugging", f1 true with no precondition; auth service toggleAuthDebug, getAuthDebugOutputChannelId and channel id cursorAuthDebug; first log line "Auth debugging enabled: AuthId ..., http2Disabled=..."). The sibling palette command cursorAuth.triggerTokenRefresh has no visible result and is excluded.
- Pieces:
  1. The palette entry is titled "Developer: Toggle Auth Debugging" (the word Developer is part of the title, not a category) and is offered with no condition. [cursor only]
  2. Running it flips the sign-in debug flag and then opens the Output panel on the Cursor Auth Debug channel.
  3. When the flag turns on, the channel gets a first line saying "Auth debugging enabled" with the account's auth id (or none) and whether HTTP/2 is disabled, and later sign-in activity is written there.
  4. Running it again turns the logging off and still shows the channel.

#### WB-8.22 Save, load and open the merge editor state (developer)

- Tag: both. Evidence: source. Boundary: no. Cursor: present - all five ids, the category and the labels (including "Select folder to save to") are in the bundle and in nls. ED-6 handed these commands to the developer group, so they are collected here.
- What: Developer commands in the Command Palette under "Merge Editor (Dev)" copy the state of the open merge editor as JSON, save it to a folder, or reopen a saved state in a new merge editor.
- Commands: merge.dev.copyContentsJson; merge.dev.saveContentsToFolder; merge.dev.loadContentsFromFolder; merge.dev.openContentsJson; merge.dev.openSelectionInTemporaryMergeEditor
- Menus: CommandPalette (category Merge Editor (Dev); copy and save only while a merge editor is active)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\mergeEditor\browser\commands\devCommands.ts (lines 24-193); V\src\vs\workbench\contrib\mergeEditor\electron-browser\devCommands.ts (lines 20-187); V\src\vs\workbench\contrib\mergeEditor\browser\mergeEditor.contribution.ts (lines 97-99); C bundle (merge.dev.* ids); C out\nls.messages.json (labels and notices)
- Pieces:
  1. Five entries named Copy Merge Editor State as JSON, Save Merge Editor State to Folder, Load Merge Editor State from Folder, Open Merge Editor State from JSON and Open Selection In Temporary Merge Editor appear in the Command Palette under the category "Merge Editor (Dev)".
  2. Copy Merge Editor State as JSON puts the base, both inputs, the result, the initial result and the language id on the clipboard as JSON and shows the notice "Successfully copied merge editor state".
  3. Save Merge Editor State to Folder opens a folder picker titled "Select folder to save to" and writes the five texts as files named base, input1, input2, result and initialResult with the language's extension, then shows "Successfully saved merge editor state to folder".
  4. The copy and save entries are offered only while a merge editor is the active editor, and if it is gone by the time they run they show the notice "No active merge editor".
  5. Load Merge Editor State from Folder opens the same folder picker, asks whether to open "result" or "initial result", and opens a new merge editor whose inputs are labelled "(from file)".
  6. Open Merge Editor State from JSON asks "Enter JSON" with the clipboard text filled in, writes the texts to temporary files, and opens a merge editor whose inputs are labelled "(from JSON)"; an empty answer opens an empty plain text merge. This entry and the next exist in the desktop app only.
  7. Open Selection In Temporary Merge Editor takes the text selected in the base pane, finds the matching text in both inputs and the result, and opens just that excerpt as a temporary merge editor, and does nothing when nothing is selected.

#### WB-8.23 Developer: Apply Update from File

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the command id, its Windows-only registration, the Developer category, the Setup .exe filter and the idle-state condition are in the bundle; Cursor words the entry "Apply Update...", the dialog title "Apply Update" and the button "&&Update", where stock says "Apply Update from File...", "Apply Update from File" and the same button; the main-process updater behind it was not read
- What: On Windows the person runs Developer: Apply Update from File... from the Command Palette, picks a setup file, and the app updates from that file instead of downloading one.
- Commands: _update.applyupdate
- Menus: CommandPalette (category Developer, Windows only)
- Keys: none found
- Settings: update.enableWindowsBackgroundUpdates
- Source: V\src\vs\workbench\contrib\update\browser\update.contribution.ts (lines 215-246); V\src\vs\platform\update\electron-main\updateService.win32.ts (lines 653-671); V\src\vs\platform\update\common\update.config.contribution.ts (lines 66-73); C bundle (class with id _update.applyupdate, category Developer, precondition idle, strings "Apply Update...", "Apply Update", "&&Update")
- Pieces:
  1. On Windows the Command Palette lists Developer: Apply Update from File..., and the command is not registered on macOS or Linux.
  2. Running it opens a file picker titled "Apply Update from File" that shows only setup `.exe` files and has an Update button.
  3. Closing the picker without choosing a file does nothing.
  4. Choosing a setup file makes the updater treat that file as an update that has already been downloaded, with no network download.
  5. When `update.enableWindowsBackgroundUpdates` is on, which is the default, and the app is a per-user install, the update starts installing at once.
  6. Otherwise the update becomes ready, and the person finishes it from the update control or the Restart to Update row (see TB-5).
  7. The command is available only while the updater is idle, so it cannot be run while an update is downloading, downloaded or ready.
  8. In Cursor the entry reads Developer: Apply Update... and the picker is titled Apply Update.

#### WB-8.24 Developer: Inspect Key Mappings

- Tag: both. Evidence: source. Boundary: no. Cursor: present - both class ids and titles are in the bundle, with the Developer category and the same untitled editor call
- What: The person runs Developer: Inspect Key Mappings, or its JSON form, and a new untitled tab opens with a dump of the keyboard layout and key mappings, used to find out why a key does not do what it should.
- Commands: workbench.action.inspectKeyMappings; workbench.action.inspectKeyMappingsJSON
- Menus: CommandPalette (category Developer)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\codeEditor\browser\inspectKeybindings.ts (lines 14-54); V\src\vs\workbench\services\keybinding\browser\keybindingService.ts (_dumpDebugInfo, lines 395-401; _dumpDebugInfoJSON, lines 403-409); C bundle (classes with ids workbench.action.inspectKeyMappings and workbench.action.inspectKeyMappingsJSON, NLS 5029 and 5030)
- Pieces:
  1. The Command Palette lists Developer: Inspect Key Mappings and Developer: Inspect Key Mappings (JSON), and neither has a key or a menu row.
  2. Inspect Key Mappings opens a pinned untitled editor whose text starts with "Layout info:" and the current keyboard layout, then lists the resolved keybindings and the keyboard mapper details, and ends with "Raw mapping:" and the raw key mapping.
  3. Inspect Key Mappings (JSON) opens a pinned untitled editor that holds one JSON object with the two entries `layout` and `rawMapping`.
  4. Both open as untitled files, so nothing is written to disk until the person saves the tab.

#### WB-8.25 Developer: Open URL

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the id, the title, the Developer category and the "URL to open" prompt are in the bundle, but Cursor's box starts empty and keeps no last input
- What: The person runs Developer: Open URL, types an address in the input box, and the app hands it to its URL service as if the link had been opened from outside.
- Commands: workbench.action.url.openUrl
- Menus: CommandPalette (category Developer)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\url\browser\url.contribution.ts (OpenUrlAction, lines 27-57); C bundle (class with id workbench.action.url.openUrl, title "Open URL", category Developer, prompt message 11526, no storage read)
- Pieces:
  1. The Command Palette lists Developer: Open URL, with no key and no menu row.
  2. Running it shows an input box with the prompt "URL to open".
  3. Pressing Enter with text in the box reads the text as an address and passes it to the URL service, which gives it to the handler registered for that address, and the command shows nothing of its own.
  4. Pressing Escape, or accepting an empty box, does nothing.
  5. In stock the box starts filled with the last address entered in this workspace, and each accepted address is remembered for the next run. [stock only]
  6. In Cursor the box always starts empty and nothing is remembered. [cursor only]


## WB-9 Windows and workspaces

(both). Duplicate the folders as an untitled workspace, open the workspace configuration file, the offer to open a workspace file found in a folder, the shell command install, and macOS Touch Bar buttons.

12 features.

#### WB-9.1 Duplicate the folders as an untitled workspace in a new window

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the command, label and multi-folder condition are present; the stock Agents window condition is not)
- What: Run "Workspaces: Duplicate As Workspace in New Window" and a new window opens on an untitled workspace that holds the same folders as the current window.
- Commands: workbench.action.duplicateWorkspaceInNewWindow
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts:292-325; C bundle (class with id workbench.action.duplicateWorkspaceInNewWindow, category Workspaces, precondition without an Agents window check)
- Pieces:
  1. Running the command opens a new window on an untitled multi-folder workspace that lists the same folders as the current window.
  2. The workspace settings of the current window are copied into the new workspace.
  3. The current window stays open and unchanged.
  4. When the current window is remote, the new window uses the same remote connection.
  5. The command is shown only when multi-folder workspaces are supported.
  6. In stock the command is also hidden in the Agents window; the Cursor bundle has no such condition. [stock only]
  7. It is reached only from the Command Palette under the Workspaces category, with no menu row and no key.

#### WB-9.2 Open the workspace configuration file

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the command, label and workspace-only condition are present; the stock Agents window condition is not)
- What: When a workspace file is open, "Workspaces: Open Workspace Configuration File" opens that file in an editor tab.
- Commands: workbench.action.openWorkspaceConfigFile
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts:188-210; C bundle (class with id workbench.action.openWorkspaceConfigFile, category Workspaces, precondition is only the workspace state)
- Pieces:
  1. Running the command opens the workspace file of the current multi-folder workspace as a pinned editor tab.
  2. The command is available only when the window is on a workspace file, so it does not show for a single folder or an empty window.
  3. In stock the command is also hidden in the Agents window; the Cursor bundle has no such condition. [stock only]
  4. It is reached only from the Command Palette under the Workspaces category, with no menu row and no key.

#### WB-9.3 Install or remove the cursor and code shell commands

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (stock has only install and uninstall of its own command, macOS only; Cursor keeps those, reworded, and adds the two code commands and the prompt)
- What: Four Shell Command entries put the cursor or code command on the path or take it off, and Cursor also shows a one-time notification offering to do it.
- Commands: workbench.action.installCommandLine; workbench.action.uninstallCommandLine; workbench.action.replaceCommandLine; workbench.action.restoreCommandLine
- Menus: CommandPalette; CommandPalette (Shell Command category)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\installActions.ts:15-80; V\src\vs\workbench\electron-browser\desktop.contribution.ts:65-69; C bundle (four classes in category Shell Command, NLS 12002-12010; registration Li&&(Xe(y4w),Xe(w4w)),Xe(S4w),Xe(k4w) where Li is the macOS flag; prompt code reading cmdLineHookState with buttons Install 'code', Install 'cursor' and Never show again); V\src\vs\workbench\electron-browser\actions\installActions.ts:19-80; V\src\vs\workbench\electron-browser\desktop.contribution.ts:65-69 (macOS only); C bundle (ids workbench.action.installCommandLine and uninstallCommandLine, category "Shell Command" nls 12007, titles nls 12008 to 12010, success text nls 12002 and 12006, registration behind the macOS check)
- Pieces:
  1. "Shell Command: Install 'cursor' command" puts the cursor command on the path and shows "Shell command 'cursor' successfully installed." or the error text; it exists on macOS only, as in stock where the label is "Install '{0}' command in PATH".
  2. "Shell Command: Uninstall 'cursor' command from PATH" removes it and shows "Shell command 'cursor' successfully uninstalled from PATH."; cancelling shows nothing.
  3. "Shell Command: Install 'code' command" makes the code command open Cursor and shows "Successfully installed Cursor in the 'code' command." [cursor only]
  4. "Shell Command: Uninstall 'code' command from PATH" unlinks Cursor from the code command and shows "Successfully unlinked Cursor from the 'code' command." [cursor only]
  5. The two code commands are registered on every platform, while the two cursor commands are registered on macOS only. [cursor only]
  6. A sticky notification asks "Want to launch Cursor from the command line? You can add the `code` or `cursor` commands now, or later with the Command Palette key and typing 'install code command'." with buttons Install 'code', Install 'cursor' and Never show again. [cursor only]
  7. The notification appears about 5 minutes after start, on Windows and macOS only, at most twice, and not again within 5 days of showing; pressing any of its three buttons stops it for good. [cursor only]
  8. All four entries are in the Command Palette under Shell Command and have no menu rows or keys.

#### WB-9.4 Bring a window to the front by command

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - 0 hits for both ids in the bundle
- What: Run Switch to Main Window from a secondary editor window, or Focus Window from anywhere, to bring a window to the front and give it focus.
- Commands: workbench.action.switchToMainWindow; workbench.action.focusWindow
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts:393-432; V\src\vs\workbench\electron-browser\desktop.contribution.ts:44-45; C bundle (searched both ids, 0 hits)
- Pieces:
  1. "Switch to Main Window" is listed in the Command Palette only while the active window is an auxiliary window (a secondary editor window), and it brings the main window to the front. [stock only]
  2. "Focus Window" brings the current window to the foreground and focuses it even when the app is not the active one, by forcing focus from the operating system. [stock only]
  3. Focus Window has no menu row and no key, and its code comment says it is meant as the first step of a `runCommands` chain that must reveal the window before a command such as Quick Open. [stock only]

#### WB-9.5 Offer to open a workspace file found in the folder

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the notification class, id and strings (nls 11961 to 11963) are in the bundle
- What: When you open a folder that has a .code-workspace file at its top level, a notification offers to open that workspace file instead.
- Commands: workspaces.dontPromptToOpen (the Don't Show Again id, not a command); workbench.action.openWorkspaceFromEditor (related, the Open Workspace button on a .code-workspace file, named in ED-5)
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\workspaces\browser\workspaces.contribution.ts:26-108; C bundle (class with doHandleWorkspaceFiles, neverShowAgain id workspaces.dontPromptToOpen with workspace scope)
- Pieces:
  1. After a single real (non virtual) folder opens, and its top level holds exactly one workspace file, an info notification says "This folder contains a workspace file '<name>'. Do you want to open it?" with a Learn more link and an Open Workspace button.
  2. When the folder holds several workspace files the text says it "contains multiple workspace files", and its Select Workspace button opens a pick list "Select a workspace to open" of the file names.
  3. Pressing Open Workspace, or picking a name from the list, opens that workspace file.
  4. The notification has a Don't Show Again choice that is remembered for this workspace only.
  5. In a workspace that has been opened before, the notification arrives silently, in the notification center without a pop-up; in a brand new workspace it pops up as usual.

#### WB-9.6 macOS Touch Bar buttons

- Tag: both. Evidence: source. Boundary: no. Cursor: present - TouchBarContext, the back, forward and debug rows and both settings were found in the bundle, and nls 12071 holds the enable description
- What: On a Mac that has a Touch Bar, the strip above the keyboard shows buttons for going back and forward and for running and stepping a debug session, and two settings turn it off or hide single buttons.
- Commands: workbench.action.navigateBack; workbench.action.navigateForward; workbench.action.debug.run; workbench.action.debug.start; workbench.action.debug.continue; workbench.action.debug.pause; workbench.action.debug.stepOver; workbench.action.debug.stepInto; workbench.action.debug.stepOut; workbench.action.debug.restart; workbench.action.debug.stop
- Menus: TouchBarContext (group navigation for Go Back and Go Forward, group 9_debug for the debug buttons)
- Keys: none found
- Settings: keyboard.touchbar.enabled; keyboard.touchbar.ignored
- Source: V\src\vs\workbench\electron-browser\window.ts (lines 158-170, 377-381 and 905-966); V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 360-380); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (lines 338-350); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (lines 229-253); V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts (touchBar key); C bundle (TouchBarContext rows with the back, forward and debug entries, both settings)
- Pieces:
  1. The Touch Bar shows a Go Back button and a Go Forward button that move through the editor navigation history.
  2. When a debugger is available and no session is running, the Touch Bar shows Start Without Debugging and Start Debugging buttons.
  3. During a debug session the Touch Bar shows Step Over, Step Into, Step Out, Restart and Stop, plus Continue while stopped and Pause while running.
  4. Pressing a button runs its command with the active file as the argument, because the buttons depend on the active editor.
  5. The buttons update a short moment after the active editor, the focus or the debug state changes.
  6. Setting `keyboard.touchbar.enabled` (on by default, listed only on macOS) turns all the buttons off when it is off, and the change applies at once.
  7. Setting `keyboard.touchbar.ignored` takes a list of command ids that must not show on the Touch Bar, for example `workbench.action.navigateBack`.

#### WB-9.7 Open Recent picker

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - both command ids, the context key inRecentFilesPicker, the headings "folders & workspaces" and "files", the placeholder "Select to open (hold Ctrl-key to force new window or Alt-key for same window)", the remove button and the two unsaved-files tooltips are in the bundle; the four "Opened in a Window" and "Opened in Active Window" tooltips and the code that adds those buttons are not
- What: The person presses Ctrl+R or runs Open Recent from the Command Palette and gets a list of recent folders, workspaces and files to open, with a button on each row to remove it and, in stock only, a marker for folders and workspaces already open in a window.
- Commands: workbench.action.openRecent; workbench.action.quickOpenRecent
- Menus: MenubarRecentMenu (More..., group y_more, order 1); CommandPalette (Open Recent only, category File; Quick Open Recent has f1 false and is not listed)
- Keys: Ctrl+R on workbench.action.openRecent (the Control key on macOS too); none found for workbench.action.quickOpenRecent
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\windowActions.ts (BaseOpenRecentAction lines 47-278, OpenRecentAction lines 280-309, QuickPickRecentAction lines 311-326); C bundle (class with id workbench.action.openRecent and key R, class with id workbench.action.quickOpenRecent and title "Quick Open Recent...", strings "folders & workspaces", "Remove from Recently Opened", "Folder With Unsaved Files"; no hit for "Folder Opened in a Window")
- Pieces:
  1. Pressing Ctrl+R, or running Open Recent... from the Command Palette or File > Open Recent > More..., opens a list with the placeholder "Select to open (hold Ctrl-key to force new window or Alt-key for same window)", which says Cmd-key and Option-key on macOS.
  2. The list has a "folders & workspaces" heading (just "folders" when none of the entries is a workspace) over the recent folders and workspaces, then a "files" heading over the recent files, in most recently used order.
  3. Each row shows the entry name with a file, folder or workspace icon and its parent path as a description, and typing in the box narrows the rows by name or path.
  4. Choosing a row opens it in the current window, holding Ctrl (Cmd on macOS) while choosing opens it in a new window, and holding Alt (Option on macOS) reuses the current window.
  5. When the first recent entry is the workspace you are already in, the highlight starts on the second row.
  6. Each row has a Remove from Recently Opened button, and clicking it takes the entry out of the list.
  7. A folder or workspace that has unsaved backups shows an always-visible dot button in place of the remove button, so it cannot be removed (see DL-2.14).
  8. A folder or workspace that is already open in a window shows an always-visible window icon with the tooltip "Folder Opened in a Window" or "Workspace Opened in a Window", or "Folder Opened in Active Window" or "Workspace Opened in Active Window" for the window you are in, and clicking the first kind removes the entry from the list. [stock only]
  9. Quick Open Recent (`workbench.action.quickOpenRecent`) shows the same list in quick-navigate mode with the input box hidden, moves through it with the keys bound to it, and is not shown in the Command Palette.

#### WB-9.8 Switch Window picker

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - both command ids, the context key inWindowsPicker, the placeholder "Select a window to switch to", the "window group" heading, the "Current Window" description, the "{0}, window with unsaved changes" label and a Close Window button were found in the bundle; the separate Close Active Window button for the window you are in was not, and Cursor's current-window row has no button
- What: The person runs Switch Window... to see every open window in a list, picks one to bring it to the front, or closes a window from its row.
- Commands: workbench.action.switchWindow; workbench.action.quickSwitchWindow
- Menus: CommandPalette (Switch Window... only; Quick Switch Window has f1 false and is not listed); the macOS Window menu row Switch Window... (see TB-1.25)
- Keys: Ctrl+W on macOS (the Control key) on workbench.action.switchWindow; none found on Windows and Linux; none found for workbench.action.quickSwitchWindow
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts (BaseSwitchWindow lines 233-356, SwitchWindowAction lines 358-376, QuickSwitchWindowAction lines 378-391); C bundle (classes with ids workbench.action.switchWindow and workbench.action.quickSwitchWindow, closeWindowAction and closeDirtyWindowAction only, strings 12029 "Select a window to switch to" and "window group")
- Pieces:
  1. Running Switch Window... from the Command Palette opens a list with the placeholder "Select a window to switch to" and one row per open window, each with the window title and a file or folder icon.
  2. Choosing a row brings that window to the front and gives it focus.
  3. The row for the window you are in carries the description "Current Window", and the highlight starts on the next window below it.
  4. The rows of other windows have a Close Window button, and clicking it closes that window and takes its row out of the list.
  5. A window with unsaved changes shows an always-visible dot button, and a screen reader reads its row as "{title}, window with unsaved changes".
  6. When any window has floating windows, the rows are grouped, with a "window group" heading over each main window followed by its floating windows.
  7. The row for the window you are in has an always-visible Close Active Window button. [stock only]
  8. Quick Switch Window (`workbench.action.quickSwitchWindow`) shows the same list in quick-navigate mode with the input box hidden, and it is not shown in the Command Palette.
  9. On macOS Ctrl+W opens the list, and the OS Window menu has a Switch Window... row.

#### WB-9.9 Close Other Windows

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - neither the id nor the label "Close Other Windows" is in the main bundle
- What: The person runs Close Other Windows from the Command Palette and every other app window is asked to close, leaving the window they ran it in.
- Commands: workbench.action.closeOtherWindows
- Menus: CommandPalette (f1, no category)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts (CloseOtherWindowsAction, lines 69-93); V\src\vs\platform\native\electron-main\nativeHostMainService.ts (getWindows, lines 217-235; closeWindow, lines 1113-1116); C bundle (no hit for the id or the label)
- Pieces:
  1. Typing Close Other Windows in the Command Palette lists the command with no category and no default key. [stock only]
  2. Running it closes every other main window of the app and keeps the window it was run from. [stock only]
  3. Each of those windows is closed with the same call its own close button uses, so a window that has unsaved changes follows its normal close flow and may still ask first (see DL-2). [stock only]
  4. Floating editor windows are not closed one by one, because only main windows are listed, so they go away with their parent window. [stock only]
  5. With a single window open, running the command changes nothing. [stock only]

#### WB-9.10 Choose the size and place of a new window

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the setting, its five values and its default are in the bundle
- What: The person picks, through `window.newWindowDimensions`, whether a new window opens centered, like the last active window, offset from it, maximized or full screen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.newWindowDimensions
- Source: V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 218-231); C bundle (setting `window.newWindowDimensions` with enum default, inherit, offset, maximized, fullscreen and default default)
- Pieces:
  1. Setting `window.newWindowDimensions` chooses how a new window is sized and placed, with values `default`, `inherit`, `offset`, `maximized` and `fullscreen`.
  2. `default`, the default, opens each new window in the center of the screen.
  3. `inherit` opens a new window with the same size as the last active window, and `offset` does the same with an offset position.
  4. `maximized` opens each new window maximized and `fullscreen` opens it in full screen mode.
  5. The setting applies only when at least one window is already open, because the first window always restores the size and place it had when it was closed.
  6. The setting is an application setting, so a workspace cannot override it.

#### WB-9.11 Close the window when its last editor closes

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the setting and the close-when-empty code that reads it are in the bundle; the message strings were not compared
- What: With `window.closeWhenEmpty` on, closing the last open editor of a window that shows no folder also closes the window.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.closeWhenEmpty
- Source: V\src\vs\workbench\electron-browser\window.ts (maybeCloseWindow, lines 625-651; listener, line 395); V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 232-236); C bundle (setting `window.closeWhenEmpty` and method maybeCloseWindow)
- Pieces:
  1. Setting `window.closeWhenEmpty` is off by default, and while it is off closing the last editor leaves the empty window open.
  2. With the setting on, closing the last editor in a window that has no folder or workspace closes that window.
  3. A window that has a folder or workspace open stays open when its last editor closes.
  4. The window also stays open while other windows, such as floating editor windows, still have editors open, and while an extension is being developed in it.
  5. With the setting on, a floating editor window closes when its last editor closes, even when a folder is open in the main window.
  6. The check is debounced, so opening another editor right after the last one closes keeps the window open.

#### WB-9.12 Choose what a second launch without files does

- Tag: both. Evidence: source. Boundary: no. Cursor: present - setting, values and the macOS-aware default are in the bundle
- What: The person picks, through `window.openWithoutArgumentsInNewWindow`, whether starting the app again with nothing to open creates a new empty window or only brings the last active window forward.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.openWithoutArgumentsInNewWindow
- Source: V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 172-182); C bundle (setting `window.openWithoutArgumentsInNewWindow` with enum on and off and default tied to the macOS flag)
- Pieces:
  1. Setting `window.openWithoutArgumentsInNewWindow` chooses what happens when a second instance starts with no folder or file, with values `on` and `off`.
  2. `on` opens a new empty window.
  3. `off` focuses the last active running window instead of opening anything.
  4. The default is `on` on Windows and Linux and `off` on macOS.
  5. The setting is an application setting, and the setting text says it can still be ignored when the new-window or reuse-window command line option is used.
