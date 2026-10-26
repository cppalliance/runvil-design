Group: WB-2 Tasks. Section: Workbench-wide. Date: 2026-10-25. Pass: 4.
Features: 7.

### WB-2.1 Allow or disallow automatic tasks
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

### WB-2.2 Open the tasks of the workspace file
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

### WB-2.3 Rerun the last task
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

### WB-2.4 Show the task log
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

### WB-2.5 Pick a problem matcher before a task runs
- Tag: both
- Evidence: source
- Boundary: no
- What: When a task that has no problem matcher is started from the Run Task list, the `task ` quick access, or the build and test pickers, a picker asks what kind of errors and warnings to scan its output for, and offers to skip scanning, to never ask again, or to read about it.
- Commands: workbench.action.tasks.runTask; workbench.action.tasks.build; workbench.action.tasks.test (the pickers that start tasks with the prompt turned on; the prompt has no command id of its own)
- Menus: none found
- Keys: none found
- Settings: task.problemMatchers.neverPrompt
- Cursor: present - the four row labels, the placeholder "Select for which kind of errors and warnings to scan the task output", the separator "associate", the setting and its description are all in the bundle and in nls
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (run lines 1443-1466, _isProblemMatcherPromptEnabled and _shouldAttachProblemMatcher lines 1473-1517, _updateNeverProblemMatcherSetting lines 1520-1533, _attachProblemMatcher lines 1535-1610, _openDocumentation line 1378, callers at lines 3145, 3190 and 3308); V\src\vs\workbench\contrib\tasks\browser\tasksQuickAccess.ts (lines 87-90); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (setting lines 470-495); C bundle (task.problemMatchers.neverPrompt) and C out\nls.messages.json (labels and placeholder)

### WB-2.6 Choose what happens when a running task is started again
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When a task is started again while as many copies of it are running as its instance limit allows, its `runOptions.instancePolicy` decides whether to ask which copy to end, end the newest or the oldest, warn, or do nothing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: task.quickOpen.skip (also the tasks.json properties `runOptions.instanceLimit` and `runOptions.instancePolicy`)
- Cursor: missing - `instancePolicy` and the strings "Select an instance to terminate", "No instance is currently running" and "The instance limit for this task has been reached." were not found in the bundle or in nls; `runOptions.instanceLimit` is present
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (_handleExecuteResult lines 2093-2116, _handleInstancePolicy lines 2118-2156, _restart from line 2158); V\src\vs\workbench\contrib\tasks\browser\terminalTaskSystem.ts (run lines 260-270); V\src\vs\workbench\contrib\tasks\common\jsonSchema_v2.ts (lines 391-407); V\src\vs\workbench\contrib\tasks\common\taskConfiguration.ts (lines 729-735); C bundle (instanceLimit found, instancePolicy not found)

### WB-2.7 Rerun All Running Tasks
- Tag: vscode
- Evidence: source
- Boundary: no
- What: The person runs Tasks: Rerun All Running Tasks from the Command Palette and every task that is running is ended and started again at once.
- Commands: workbench.action.tasks.rerunAllRunningTasks
- Menus: CommandPalette (Tasks category, when tasks can run)
- Keys: none found
- Settings: none found
- Cursor: missing - neither the id nor the label "Rerun All Running Tasks" is in the main bundle
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (palette entry, lines 289-296); V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (registration, lines 634-638; _runRerunAllRunningTasksCommand, lines 3535-3546; _restart, lines 2158-2185); V\src\vs\workbench\contrib\tasks\common\tasks.ts (line 1442); C bundle (no hit for the id or the label)

## Handed off
- none

## Unclear
- Created in pass 4 from the coverage check.
