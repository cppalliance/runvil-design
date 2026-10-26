Bucket: I. Date: 2026-10-25. Pass: 4.

### NEW-I-1 Pick a problem matcher before a task runs
- Group: WB-2
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

### NEW-I-2 Choose what happens when a running task is started again
- Group: WB-2
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

### NEW-I-3 Run, debug or cover the test at the cursor
- Group: SB-6
- Tag: both
- Evidence: source
- Boundary: no
- What: With the text cursor in a test file, run, debug or run with coverage the test that contains the cursor, from the Command Palette, the editor right-click menu or a key chord.
- Commands: testing.runAtCursor; testing.debugAtCursor; testing.coverageAtCursor
- Menus: CommandPalette (Test category, when a test provider exists); EditorContext (group testing, Run Test at Cursor first and Debug Test at Cursor and the coverage entry in the next slot, when the active editor has tests and a profile of that kind exists)
- Keys: Ctrl+; then C (run); Ctrl+; then Ctrl+C (debug); Ctrl+; then Ctrl+Shift+C (coverage), all when the editor has text focus (Cmd on macOS)
- Settings: testing.saveBeforeTest
- Cursor: present - the three ids, the labels (nls 11097 to 11099), the run and debug chords and "No tests found here" are in the bundle. ED-11 handed these commands to SB-6 and SB-6 has no feature for them; the parent may merge this with NEW-B-23, its current-file sibling.
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (ExecuteTestAtCursor lines 1048-1131, RunAtCursor, DebugAtCursor and CoverageAtCursor lines 1133-1176); V\src\vs\workbench\contrib\testing\common\constants.ts (lines 80 and 105); C bundle (classes with ids testing.runAtCursor, testing.debugAtCursor and testing.coverageAtCursor); C out\nls.messages.json (labels, "No tests found here")
- Pieces:
  1. The Command Palette lists Test: Run Test at Cursor, Test: Debug Test at Cursor and Test: Run Test at Cursor with Coverage whenever a test provider is registered.
  2. With text focus in the editor, `Ctrl+;` then `C` runs the test at the cursor, `Ctrl+;` then `Ctrl+C` debugs it, and `Ctrl+;` then `Ctrl+Shift+C` runs it with coverage.
  3. The editor right-click menu shows the matching entries in the testing group only when the active file has tests and a run, debug or coverage profile exists.
  4. The tests picked are the most specific ones whose range contains the cursor, and when none contains it, the closest test above the cursor; tests with an equal range all run, and only tests that have a profile of the chosen kind are used.
  5. When no test is found at the cursor but tests are known to relate to the code there, those related tests run instead.
  6. When nothing is found at all, a small message "No tests found here" appears at the cursor.
  7. A progress indicator shows while the tests of the file are being discovered.
  8. Setting `testing.saveBeforeTest` on makes the editor save the file and refresh the test list before the run starts.

### NEW-I-4 Save, load and open the merge editor state (developer)
- Group: WB-8
- Tag: both
- Evidence: source
- Boundary: no
- What: Developer commands in the Command Palette under "Merge Editor (Dev)" copy the state of the open merge editor as JSON, save it to a folder, or reopen a saved state in a new merge editor.
- Commands: merge.dev.copyContentsJson; merge.dev.saveContentsToFolder; merge.dev.loadContentsFromFolder; merge.dev.openContentsJson; merge.dev.openSelectionInTemporaryMergeEditor
- Menus: CommandPalette (category Merge Editor (Dev); copy and save only while a merge editor is active)
- Keys: none found
- Settings: none found
- Cursor: present - all five ids, the category and the labels (including "Select folder to save to") are in the bundle and in nls. ED-6 handed these commands to the developer group, so they are collected here.
- Source: V\src\vs\workbench\contrib\mergeEditor\browser\commands\devCommands.ts (lines 24-193); V\src\vs\workbench\contrib\mergeEditor\electron-browser\devCommands.ts (lines 20-187); V\src\vs\workbench\contrib\mergeEditor\browser\mergeEditor.contribution.ts (lines 97-99); C bundle (merge.dev.* ids); C out\nls.messages.json (labels and notices)
- Pieces:
  1. Five entries named Copy Merge Editor State as JSON, Save Merge Editor State to Folder, Load Merge Editor State from Folder, Open Merge Editor State from JSON and Open Selection In Temporary Merge Editor appear in the Command Palette under the category "Merge Editor (Dev)".
  2. Copy Merge Editor State as JSON puts the base, both inputs, the result, the initial result and the language id on the clipboard as JSON and shows the notice "Successfully copied merge editor state".
  3. Save Merge Editor State to Folder opens a folder picker titled "Select folder to save to" and writes the five texts as files named base, input1, input2, result and initialResult with the language's extension, then shows "Successfully saved merge editor state to folder".
  4. The copy and save entries are offered only while a merge editor is the active editor, and if it is gone by the time they run they show the notice "No active merge editor".
  5. Load Merge Editor State from Folder opens the same folder picker, asks whether to open "result" or "initial result", and opens a new merge editor whose inputs are labelled "(from file)".
  6. Open Merge Editor State from JSON asks "Enter JSON" with the clipboard text filled in, writes the texts to temporary files, and opens a merge editor whose inputs are labelled "(from JSON)"; an empty answer opens an empty plain text merge. This entry and the next exist in the desktop app only.
  7. Open Selection In Temporary Merge Editor takes the text selected in the base pane, finds the matching text in both inputs and the result, and opens just that excerpt as a temporary merge editor, and does nothing when nothing is selected.
