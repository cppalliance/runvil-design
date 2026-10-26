Group: SB-6 Testing. Pass: 3. Date: 2026-10-25.

### SB-6.1
1. The Testing container stays hidden in the side bar until a test extension registers a provider, and then it shows the Test Explorer view.
2. Choosing Testing in the View menu opens the Testing side bar.
3. The Show Testing command in the Command Palette (`workbench.view.extension.test`) opens the Testing side bar.
4. The command `workbench.view.testing.focus` moves keyboard focus into the Test Explorer view.
5. While the Test Explorer is the only view in the container, the container title "Testing" doubles as the view header.
6. The Testing side bar opens at a preferred width of 400 px.

### SB-6.2
1. Choosing View as Tree shows the tests as a nested tree of controller, file, suite and test.
2. Choosing View as List shows the tests as a flat list of tests.
3. Collapse All Tests collapses every open node in the tree with one action.
4. The View as Tree, View as List and Collapse All Tests rows are in the view's title menu, the tree mode is the default, and the chosen mode is remembered per workspace.

### SB-6.3
1. Choosing Sort by Status orders the tests by their result status.
2. Choosing Sort by Location orders the tests by where they sit in the files.
3. Choosing Sort by Duration orders the tests by how long they took to run.
4. The three sort choices are rows in the view's title menu with a check on the active one, and Sort by Location is the default.
5. The chosen sort order is remembered per workspace.

### SB-6.4
1. Typing in the filter box above the tree shows only the tests that match the text.
2. A comma separates filter terms, and a leading `!` on a term excludes matching tests.
3. A term written as `@controller:tag` shows only tests that carry that test tag.
4. Typing `@` in the filter box opens a suggestion list of the filter terms and the known tags.
5. The filter box keeps a history of earlier filters that can be recalled.
6. Typing a character while the tree is focused jumps into the filter box.
7. The empty filter box shows the placeholder "Filter (e.g. text, !exclude, @tag)".

### SB-6.5
1. A funnel button beside the filter box opens a More Filters menu with Show Only Failed Tests, Show Only Executed Tests, Show in Active File Only, Show in Opened Files Only, Fuzzy Match, Show Hidden Tests and Unhide All Tests.
2. Show Only Failed Tests and Show Only Executed Tests are toggles that write `@failed` and `@executed` into the filter box.
3. Show in Active File Only and Show in Opened Files Only are toggles that write `@doc` and `@openedFiles` into the filter box.
4. Fuzzy Match changes how the text in the filter box matches test names, and the choice is remembered across workspaces.
5. Show Hidden Tests is a toggle that writes `@hidden` into the filter box, and Unhide All Tests makes every hidden test visible again.
6. The funnel button looks checked while any test is hidden.

### SB-6.6
1. Choosing Run All Tests in the Command Palette, or pressing `Ctrl/Cmd+;` then `A`, runs every test with the default Run profiles.
2. The Run Tests button in the view title runs only the tests that are currently visible after filtering.
3. When no tests exist, running all tests shows an info message that says a test provider extension may need to be installed.
4. Setting `testing.saveBeforeTest` saves all dirty editors before a run starts, and it is on by default.

### SB-6.7
1. Hovering a test, suite or file row shows a Run Test button that runs that test, or every test under that suite or file.
2. Choosing Run Test in a row's right-click menu runs the same tests.
3. Pressing `Enter` in the tree runs the focused row, or every selected row when the focused row is part of the selection.
4. Selecting several rows and using a row action runs all of the selected rows.
5. Setting `testing.saveBeforeTest` saves all dirty editors before a row run starts.

### SB-6.8
1. The Debug Tests button in the view title starts a debug session for the tests that are currently visible.
2. The Debug Test button on a row, or Debug Test in its right-click menu, debugs that one test.
3. Choosing Debug All Tests in the Command Palette, or pressing `Ctrl/Cmd+;` then `Ctrl/Cmd+A`, debugs every test.
4. The Debug buttons and menu rows appear only when a test extension has registered a debug profile.
5. Setting `testing.saveBeforeTest` saves all dirty editors before a debug run starts.

### SB-6.9
1. The Run Tests with Coverage button in the view title runs the visible tests and collects coverage.
2. The Run Test with Coverage button on a row, or the same entry in its right-click menu, runs one test with coverage.
3. Choosing Run All Tests with Coverage in the Command Palette, or pressing `Ctrl/Cmd+;` then `Ctrl/Cmd+Shift+A`, runs every test with coverage.
4. When a coverage run finishes with coverage data, the Test Coverage view opens in the Testing container.
5. Setting `testing.saveBeforeTest` saves all dirty editors before a coverage run starts.

### SB-6.10
1. When a test extension offers more than one profile for Run, Debug or Coverage, the title button becomes a split button with a drop-down arrow.
2. The drop-down lists each controller's profiles, and the default profile is marked "(Default)".
3. Picking a profile in the drop-down runs the visible tests with that profile.
4. Choosing Execute Using Profile... in a row's right-click menu opens a "Pick a test profile to use" list and runs that test with the pick.
5. Extensions can add their own rows to the drop-down through the `testing/profiles/context` menu.

### SB-6.11
1. Select Default Profile, at the bottom of the title-button drop-down when more than one profile exists, opens a multi-select pick of the default profiles for Run, Debug and Coverage.
2. Configure Test Profiles, at the bottom of the drop-down when a profile can be configured, opens a pick titled "Select a profile to update" and then that profile's own configuration step.
3. Configure Test Profiles is also listed in the Command Palette while a configurable profile exists.
4. Select Default Profile has no Command Palette entry and is reached only from the drop-down.

### SB-6.12
1. A test row has a Turn on Continuous Run toggle that makes the test re-run by itself when files change, and it reads Turn off Continuous Run while it is active.
2. The Turn on Continuous Run toggle is disabled on a row whose parent already runs continuously.
3. Choosing Start Continous Run Using... in a row's right-click menu picks which profiles watch that test.
4. The view title shows a Start Continuous Run button and a Stop Continuous Run button.
5. When several profiles support continuous run, the title button becomes a split button whose drop-down has a check beside each profile.
6. A row that is running continuously, or has a continuous child, keeps its continuous-run icon visible even when it is not hovered.
7. Start Continuous Run and Stop Continuous Run are also listed in the Command Palette.

### SB-6.13
1. While tests are running, the Run, Debug and Coverage buttons in the view title are replaced by a Cancel Test Run button.
2. Clicking Cancel Test Run stops the tests that are running.
3. Cancel Test Run appears in the Command Palette only while tests are running, and using it without a target stops every unfinished run.
4. Pressing `Ctrl/Cmd+;` then `Ctrl/Cmd+X` cancels the test run.

### SB-6.14
1. The Refresh Tests button in the view title asks the test extensions to find tests again.
2. An inline Refresh Tests button on a root row refreshes the tests of that controller.
3. Choosing Refresh Tests in the Command Palette, or pressing `Ctrl/Cmd+;` then `Ctrl/Cmd+R`, refreshes the tests.
4. While a refresh is running, the Refresh Tests button turns into Cancel Test Refresh.
5. While a refresh is running, the view shows a "Discovering Tests" progress bar.
6. The refresh buttons and the key chord appear only when a test controller supports refresh.

### SB-6.15
1. Right-clicking a test, suite or file row opens a menu with Run Test, Debug Test and Run Test with Coverage.
2. The same menu has Execute Using Profile..., Start Continous Run Using... and Turn on Continuous Run or Turn off Continuous Run.
3. The same menu has Go to Test and Hide Test, which reads Unhide Test on a hidden row.
4. Refresh Tests is an inline button only and is not a row in the right-click menu.
5. When the right-clicked row is part of the selection, the chosen action applies to every selected row.
6. Extensions can add their own rows to the menu through the `testing/item/context` menu.

### SB-6.16
1. Opening a test that has no child rows opens its file at the test's line.
2. Opening a test that has failed shows a peek of its first error instead of just opening the file.
3. Setting `workbench.list.openMode` chooses whether a single click or a double click opens a test, with values `singleClick` (default) and `doubleClick`.
4. Opening a test to the side puts its file in an editor beside the current one.
5. Choosing Go to Test in a row's right-click menu opens the test's file without the failure peek.
6. Pressing `Alt+Enter` on a focused row in the Testing view does the same as Go to Test.

### SB-6.17
1. Choosing Hide Test in a row's right-click menu removes that test, suite or file from the tree.
2. With Show Hidden Tests on, hidden rows show again, dimmed and with a hidden icon.
3. A hidden row's right-click menu has Unhide Test, which brings that row back to normal.
4. Unhide All Tests makes every hidden test visible again.
5. Revealing a hidden test from somewhere else, such as from the editor, turns Show Hidden Tests on by itself.

### SB-6.18
1. With `testing.followRunningTest` on (off by default), the tree scrolls to and selects the test that is currently running.
2. The tree does not follow the running test while several rows are selected.
3. Setting `testing.alwaysRevealTestOnStateChange` chooses whether every executed test is revealed when following is on, and with it off (default) only tests that fail are revealed.

### SB-6.19
1. Each test row shows a state icon for passed, failed, errored, queued, running with a spinner, skipped or not yet run.
2. When a result is outdated, the state icon shows in a lighter color through the `testing.icon*.retired` theme colors at 70 percent opacity.
3. A row can show a short description beside its name.
4. A row shows how long the last run took, for example `12ms` or `1.2s`.
5. Hovering a row reads out the full state, such as "Test name (Passed), in 12ms", with ", outdated result" added when the result is outdated.

### SB-6.20
1. A summary row above the tree shows the last run as a status icon, the passed count over the total count, and the duration.
2. Hovering the passed over total count gives the detail of the run.
3. Rerun Last Run and Debug Last Run buttons appear in the summary row once the run is done.
4. With no runs yet, the summary row says "No test results yet."
5. Setting `testing.countBadge` chooses what the badge on the Testing icon counts, with values `failed` (default), `passed`, `skipped` and `off`.
6. The badge on the Testing icon shows a watch icon while continuous run is on.
7. A spinner badge shows on the Testing icon while tests are running. [stock only]

### SB-6.21
1. Rerun Failed Tests and Debug Failed Tests in the Command Palette run again every test that has failed, and a test counts as failed until a later run passes it.
2. Rerun Last Run, Debug Last Run and Rerun Last Run with Coverage repeat the previous run with the same profiles when those profiles still exist.
3. Rerun Failed Tests from Last Run and Debug Failed Tests from Last Run repeat only the failures of the newest run. [stock only]
4. These commands show in the Command Palette only when a test provider exists, and the last-run commands also need earlier results.
5. The chords are `Ctrl/Cmd+;` then `E` (rerun failed), `Ctrl/Cmd+E` (debug failed), `L` (rerun last run), `Ctrl/Cmd+L` (debug last run) and `Ctrl/Cmd+Shift+L` (rerun last run with coverage).

### SB-6.22
1. When discovery has finished and no tests exist, the view shows "No tests have been found in this workspace yet."
2. The "Install Additional Test Extensions..." link under that text opens an Extensions view search for the testing category.
3. When the filter is set to the active file and that file has no tests, the tree is replaced by "No tests were found in this file."
4. The Show Workspace Tests button under that text switches the active-file filter off and shows the whole workspace again.

### SB-6.23
1. Setting `testing.automaticallyOpenTestResults` chooses whether starting a run opens a testing view, with values `neverOpen`, `openOnTestStart` (default), `openOnTestFailure` and `openExplorerOnTestStart`.
2. With `openOnTestStart`, the Test Results view opens when tests start.
3. With `openOnTestFailure`, the Test Results view opens at the first test failure.
4. With `openExplorerOnTestStart`, the Test Explorer opens when tests start.
5. A run that asks to keep focus where it is never opens any view.

### SB-6.24
1. While a coverage report is open, a Test Coverage view joins the Testing container.
2. The Test Coverage view is a tree of folders and files, and a chain of folders with a single child is merged into one row.
3. Each row shows its overall percent and compact bars for statements, functions and branches.
4. Hovering a row's bars shows covered over total counts.
5. Expanding a file loads its declarations, each with its own bars.
6. Opening a row opens the file, jumping to the function for a declaration, and opening to the side is supported.
7. A Collapse All Coverage button in the Test Coverage title collapses every open node. [stock only]
8. Setting `testing.displayedCoveragePercent` chooses which percent is the headline, with values `totalCoverage` (default), `statement` and `minimum`.
9. Setting `testing.coverageBarThresholds` sets the red, yellow and green cut-offs for coverage percents, with defaults of 0, 60 and 90.

### SB-6.25
1. The Change Sort Order button in the Test Coverage title opens a picker of sort orders.
2. Sort by Location lists files alphabetically and declarations by their position in the file.
3. Sort by Coverage lists the best covered rows first.
4. Sort by Name lists rows by name.
5. The chosen coverage sort order is remembered per workspace.

### SB-6.26
1. When the coverage has per-test data, the Filter Coverage by Test button in the Test Coverage title opens a picker with All Tests and each test.
2. Filter Coverage by Test is also in the Command Palette while the coverage has per-test data.
3. Moving through the picker previews the coverage of the highlighted test in the view before it is chosen.
4. After a test is chosen, the view's first row reads "Showing coverage for" and the test, with a filter button that reopens the picker.
5. The filter button in the view title looks filled while a test filter is active.

### SB-6.27
1. The X (Clear Coverage) in the Test Coverage title closes the report and hides the Test Coverage view.
2. Clear Coverage is also in the Command Palette while a coverage report is open.
3. Open Coverage in the Command Palette reopens the report from the newest run when any results exist.
4. When the newest run has no coverage, Open Coverage says "No coverage information available on the last test run."
5. A new run that finishes without coverage closes the old coverage report.
6. A new run that finishes with coverage opens its report without taking focus.

### SB-6.28
1. While a coverage report is open, files and folders in the Explorer file tree show compact coverage bars.
2. Hovering the bars on an Explorer row gives the covered counts.
3. Setting `testing.showCoverageInExplorer` chooses whether those bars show in the Explorer, and it is on by default.
4. Toggle Coverage in Explorer in the Command Palette, listed only while a coverage report is open, flips that setting, and its title reads Hide Coverage in Explorer when the bars are on. [stock only]

### SB-6.29
1. "Test: Run Tests in Current File" runs every test found in the active editor's file with the default run profile, "Test: Debug Tests in Current File" starts them as a debug session, and "Test: Run Tests with Coverage in Current File" runs them with coverage.
2. Each entry shows in the palette, and in the editor's right-click menu under the testing group, only when a test extension has registered a matching run profile, and the editor menu also needs the file to contain tests.
3. With text focus in the editor, `Ctrl+;` then `F` runs, `Ctrl+;` then `Ctrl+F` debugs, and `Ctrl+;` then `Ctrl+Shift+F` runs with coverage.
4. When the file has no tests, the message "No tests found in this file" appears at the cursor in the editor.
5. Setting `testing.saveBeforeTest` (default on) saves unsaved files before the run starts.

### SB-6.30
1. The Command Palette lists Test: Run Test at Cursor, Test: Debug Test at Cursor and Test: Run Test at Cursor with Coverage whenever a test provider is registered.
2. With text focus in the editor, `Ctrl+;` then `C` runs the test at the cursor, `Ctrl+;` then `Ctrl+C` debugs it, and `Ctrl+;` then `Ctrl+Shift+C` runs it with coverage.
3. The editor right-click menu shows the matching entries in the testing group only when the active file has tests and a run, debug or coverage profile exists.
4. The tests picked are the most specific ones whose range contains the cursor, and when none contains it, the closest test above the cursor; tests with an equal range all run, and only tests that have a profile of the chosen kind are used.
5. When no test is found at the cursor but tests are known to relate to the code there, those related tests run instead.
6. When nothing is found at all, a small message "No tests found here" appears at the cursor.
7. A progress indicator shows while the tests of the file are being discovered.
8. Setting `testing.saveBeforeTest` on makes the editor save the file and refresh the test list before the run starts.

## Unclear
- SB-6.25: the feature says stock remembers the coverage sort per workspace, and Cursor does not show the storage key, so the last piece is stock behavior that is not confirmed for Cursor.
- SB-6.2 and SB-6.3: the feature blocks do not say whether the view and sort rows have Command Palette entries, so only the title menu is described.
- SB-6.1: `workbench.view.testing.focus` is built from the view id at run time, so piece 4 rests on the code and not on a search hit.
- SB-6.16: the feature block does not name the entry point for opening to the side, so piece 4 states the behavior only.
- SB-6.9: the open-on-finish behavior in piece 4 is the same event that SB-6.27 pieces 5 and 6 describe from the other side, and it is kept in both because each feature lists it.
