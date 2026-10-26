Group: WB-2 Tasks. Pass: 3. Date: 2026-10-25.

### WB-2.1
1. Running Tasks: Manage Automatic Tasks shows a picker with Allow Automatic Tasks and Disallow Automatic Tasks, and choosing one writes `task.allowAutomaticTasks` as `on` or `off` in the user settings; Escape changes nothing.
2. Tasks whose run options say `folderOpen` start by themselves after the folder opens, only when the workspace is trusted and the setting allows it.
3. With the setting at its default, the first time such tasks are found a notification asks "This workspace has tasks (...) defined (...) that can launch processes automatically when you open this workspace. Do you want to allow automatic tasks to run in all trusted workspaces?" with Allow, Disallow and Open File buttons, and it is asked once per workspace. [stock only]
4. Allow starts the tasks and sets the setting to on, Disallow sets it to off, and Open File opens the tasks file or files and starts nothing. [stock only]
5. Setting `task.allowAutomaticTasks` takes `on` (Always) or `off` (Never); stock defaults to `off` with application scope. [stock only]
6. Cursor defaults the setting to `on` and has no prompt, so folder-open tasks start with no question unless the setting is `off`. [cursor only]

### WB-2.2
1. The Command Palette shows Tasks: Open Workspace Tasks only when the window has a saved workspace file and task execution is supported.
2. When the workspace file already has a `tasks` section, the command opens that workspace file in an editor tab.
3. When it has none, a picker titled "Select a Task Template" appears, and choosing a template writes it into the workspace file's `tasks` setting and opens the file as a pinned tab; Escape does nothing.
4. In a window without a workspace file the command does nothing.

### WB-2.3
1. Running Tasks: Rerun Last Task from the Command Palette runs the task that ran most recently once more.
2. Before it runs, every unsaved editor is saved with an explicit save.
3. When no task has run yet and none is running, it opens the Run Task list so you can pick one.
4. In a workspace that is not trusted, a Workspace Trust request saying that listing and running tasks needs files to be executed as code comes first, and the task runs only if you agree.
5. A key bound to this command keeps working while a terminal has focus, because the terminal passes it through.

### WB-2.4
1. Running Tasks: Show Task Log from the Command Palette opens the Output panel and selects the Tasks channel, leaving keyboard focus where it was.
2. The Tasks channel is also in the channel list of the Output panel (PN-5.2).
3. The warning "There are task errors. See the output for details." has a Show Output button that opens the same channel.
4. In a virtual workspace the command does nothing.

### WB-2.5
1. Choosing a task from Run Task, from the `task ` quick access, or from the build or test picker opens a picker with the placeholder "Select for which kind of errors and warnings to scan the task output" when the task has no problem matcher, is in the build group or in no group, and can be edited in tasks.json.
2. The top rows read "Continue without scanning the task output", "Never scan the task output for this task", "Never scan the task output for {task type} tasks" and "Learn more about scanning the task output", followed by an "associate" separator and every known problem matcher sorted by name, with its `$id` shown beside names that differ from it.
3. Picking a problem matcher runs the task and writes that matcher into the task's entry in tasks.json, and a matcher that watches in the background also marks the task as a background task.
4. "Continue without scanning the task output" runs the task as it is and writes nothing, so the same picker appears again the next time.
5. "Never scan the task output for this task" writes an empty problem matcher list into the task's entry and runs it, so the picker never appears for that task again.
6. "Never scan the task output for {task type} tasks" adds the task type to the object in `task.problemMatchers.neverPrompt` and runs the task, so no task of that type asks again.
7. "Learn more about scanning the task output" opens the problem matcher page of the online task documentation and does not run the task.
8. Closing the picker without choosing runs the task unchanged.
9. Setting `task.problemMatchers.neverPrompt` set to `true` turns the picker off for every task, an object of task type to true or false turns it off for chosen types only, and the default is `false`.

### WB-2.6
1. `runOptions.instanceLimit` in a task's tasks.json entry sets how many copies of the task may run at once, with a default and a minimum of 1.
2. Starting the task again while all allowed copies are running does not start another copy, and the task's terminal is brought forward if it is not the visible one before the policy is applied. [stock only]
3. `instancePolicy` set to `prompt`, the default, opens a picker titled "Select an instance to terminate" that lists the running copies of that task, and picking one ends it and starts the task again. [stock only]
4. When the prompt picker has no running copy to list, its single row reads "No instance is currently running" and picking it does nothing. [stock only]
5. `terminateNewest` ends the most recently started copy and starts the task again, and `terminateOldest` does the same with the oldest copy. [stock only]
6. `warn` shows a warning notification "The instance limit for this task has been reached." and changes nothing else. [stock only]
7. `silent` does nothing at all when the limit is reached. [stock only]
8. If the chosen copy cannot be ended, a warning notification says "Failed to terminate and restart task {name}". [stock only]
9. With `task.quickOpen.skip` on and exactly one running copy, the prompt picker is skipped and that copy is ended and restarted at once. [stock only]

### WB-2.7
1. The Command Palette lists Tasks: Rerun All Running Tasks whenever the app can run tasks, and the command has no default key. [stock only]
2. Running it restarts every running task in one go, ending each one and then starting it again. [stock only]
3. Before each restart the task is looked up again in the task configuration, so edits made to its definition since it started are used. [stock only]
4. A task that cannot be ended gets the warning "Failed to terminate and restart task {name}" and is not started again, while the other tasks are still restarted. [stock only]
5. With no task running, an information notification says "No running tasks to restart". [stock only]
6. The command first checks Workspace Trust, as the other task commands do. [stock only]

## Unclear
none
