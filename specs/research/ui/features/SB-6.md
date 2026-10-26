Group: SB-6 Testing. Section: Side bar. Date: 2026-10-25. Pass: 2.
Features: 30.

### SB-6.1 Open the Testing view
- Tag: both
- Evidence: source
- Boundary: no
- What: Open the Testing side bar from View > Testing or the "Show Testing" command. The container stays hidden until a test extension registers a provider, then it shows the Test Explorer, normally as its only view (so the container title "Testing" doubles as the view header; the Test Coverage view joins it during a coverage run), with a preferred width of 400 px.
- Commands: workbench.view.extension.test; workbench.view.testing.focus (built from the view id at run time)
- Menus: MenubarViewMenu (group 3_sidebar, label "T&&esting")
- Keys: none found (a Ctrl+Shift+; binding is commented out in the source)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts (container 62-79, Test Explorer view 112-131); V\src\vs\workbench\contrib\testing\browser\testingViewPaneContainer.ts; V\src\vs\workbench\services\views\browser\viewsService.ts (open action 401-462, focus action 557-562); C bundle (container id workbench.view.extension.test, hideIfEmpty, view id workbench.view.testing); C out\nls.messages.json ("Testing", "T&&esting", "Test Explorer")

### SB-6.2 Switch between tree and list, and collapse all
- Tag: both
- Evidence: source
- Boundary: no
- What: Show the tests as a nested tree (controller, file, suite, test) or as a flat list of tests, and collapse every open node with one action. The mode is remembered per workspace and defaults to tree.
- Commands: testing.viewAsTree; testing.viewAsList; testing.collapseAll
- Menus: ViewTitle (group viewAs, only for view workbench.view.testing); ViewTitle (group displayAction, same view)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (751-798, 903-924); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (view mode 719-731, projection switch 1130-1148); V\src\vs\workbench\contrib\testing\browser\explorerProjections\treeProjection.ts; V\src\vs\workbench\contrib\testing\browser\explorerProjections\listProjection.ts; C bundle (ids testing.viewAsTree, testing.viewAsList, testing.collapseAll; storage key testing.viewMode); C out\nls.messages.json ("View as Tree", "View as List", "Collapse All Tests")

### SB-6.3 Sort tests
- Tag: both
- Evidence: source
- Boundary: no
- What: Order the tests by Status, by Location, or by Duration. The choice is a checked row in the view's menu, is remembered per workspace, and defaults to Location.
- Commands: testing.sortByStatus; testing.sortByLocation; testing.sortByDuration
- Menus: ViewTitle (group sortBy, only for view workbench.view.testing)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (800-870); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (sorting 733-747, TreeSorter 1311-1349); V\src\vs\workbench\contrib\testing\common\constants.ts (30-34); C bundle (ids testing.sortByStatus, testing.sortByLocation, testing.sortByDuration; storage key testing.viewSorting); C out\nls.messages.json ("Sort by Status", "Sort by Location", "Sort by Duration")

### SB-6.4 Filter tests by text and tag
- Tag: both
- Evidence: source
- Boundary: no
- What: Type in the filter box above the tree to show only matching tests. A comma separates terms, a leading ! excludes, and @controller:tag filters by a test tag. Typing @ opens a suggestion list of the filter terms and known tags, the box keeps a history, and typing a character while the tree is focused jumps into the box. Placeholder text is "Filter (e.g. text, !exclude, @tag)".
- Commands: workbench.actions.treeView.testExplorer.filter
- Menus: none found (the box is a view action item, not a menu entry)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerFilter.ts (35-160); V\src\vs\workbench\contrib\testing\common\testExplorerFilterState.ts (95-210); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (836-855, 880-886, 1202-1308); C bundle (id workbench.actions.treeView.testExplorer.filter, element id testing.explorer.filter, storage key testing.filterHistory2); C out\nls.messages.json ("Filter (e.g. text, !exclude, @tag)", "Filter text for tests in the explorer")

### SB-6.5 Narrow the tree with the More Filters menu
- Tag: both
- Evidence: source
- Boundary: no
- What: The funnel button beside the filter box opens a menu with Show Only Failed Tests, Show Only Executed Tests, Show in Active File Only, Show in Opened Files Only, Fuzzy Match, Show Hidden Tests and Unhide All Tests. Each toggle writes its term (@failed, @executed, @doc, @openedFiles, @hidden) into the filter box, and the button looks checked while any test is hidden. Fuzzy Match changes how the text in the box matches and is remembered across workspaces.
- Commands: none found (the menu is built in code from plain actions)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerFilter.ts (27-33, 175-250); V\src\vs\workbench\contrib\testing\common\testExplorerFilterState.ts (110-120, 227-241); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (TestsFilter 1202-1309); C bundle (storage key testHistoryFuzzy); C out\nls.messages.json ("More Filters...", "Fuzzy Match", "Show Hidden Tests", "Show Only Failed Tests", "Show Only Executed Tests", "Show in Active File Only", "Show in Opened Files Only", "Unhide All Tests")

### SB-6.6 Run all tests
- Tag: both
- Evidence: source
- Boundary: no
- What: Run every test with the default Run profile from the Command Palette or the key chord, or press the Run Tests button in the view title to run just the tests currently visible after filtering. While a run is going the button is replaced by Cancel (see SB-6.13). If no tests exist, an info message says a test provider extension may be needed. Dirty editors are saved first when the setting is on.
- Commands: testing.runAll; testing.runSelected
- Menus: ViewTitle (group navigation, hidden while running); CommandPalette
- Keys: Ctrl/Cmd+; then A (testing.runAll)
- Settings: testing.saveBeforeTest
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (513-543, 586-593, 626-669); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (getTreeIncludeExclude 156-268, title button 311-329); V\src\vs\workbench\contrib\testing\common\testServiceImpl.ts (save before run 443-444); C bundle (ids testing.runAll, testing.runSelected, keybinding chord Ctrl/Cmd+; A); C out\nls.messages.json ("Run All Tests", "Run Tests")

### SB-6.7 Run one test or a group from its row
- Tag: both
- Evidence: source
- Boundary: no
- What: Hover a test, suite or file row to get a Run Test button, or pick Run Test from its right-click menu. Pressing Enter in the tree runs the focused row, or all selected rows when the focused row is part of the selection. Selecting several rows and using a row action runs them all.
- Commands: testing.run
- Menus: TestItem (group inline, and group builtin@1)
- Keys: Enter in the focused tree (hard-coded key handler, not a keybinding rule)
- Settings: testing.saveBeforeTest
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (150-183, 243-252); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (880-886, 1092-1111, action runner 1372-1388, row actions 1572-1580); C bundle (id testing.run, TestItem menu rows); C out\nls.messages.json ("Run Test")

### SB-6.8 Debug tests
- Tag: both
- Evidence: source
- Boundary: no
- What: Start a debug session instead of a plain run. Use the Debug Tests button in the view title for the visible tests, the Debug Test button on a row or in its right-click menu for one test, or Debug All Tests from the Command Palette. These only appear when a test extension has registered a debug profile.
- Commands: testing.debugAll; testing.debugSelected; testing.debug
- Menus: ViewTitle (group navigation, hidden while running); TestItem (group inline, and group builtin@1); CommandPalette
- Keys: Ctrl/Cmd+; then Ctrl/Cmd+A (testing.debugAll)
- Settings: testing.saveBeforeTest
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (185-194, 596-603, 626-632, 671-687); V\src\vs\workbench\contrib\testing\common\testingContextKeys.ts (hasDebuggableTests); C bundle (ids testing.debugAll, testing.debugSelected, testing.debug); C out\nls.messages.json ("Debug All Tests", "Debug Tests", "Debug Test")

### SB-6.9 Run tests with coverage
- Tag: both
- Evidence: source
- Boundary: no
- What: Run tests and collect coverage. Use the Run Tests with Coverage button in the view title for the visible tests, Run Test with Coverage on a row or in its menu for one test, or Run All Tests with Coverage from the Command Palette. A finished run that produced coverage opens the Test Coverage view (SB-6.24).
- Commands: testing.coverageAll; testing.coverageSelected; testing.coverage
- Menus: ViewTitle (group navigation, hidden while running); TestItem (group inline, and group builtin@1); CommandPalette
- Keys: Ctrl/Cmd+; then Ctrl/Cmd+Shift+A (testing.coverageAll)
- Settings: testing.saveBeforeTest
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (196-205, 605-615, 689-705); V\src\vs\workbench\contrib\testing\common\testCoverageService.ts (100-145); C bundle (ids testing.coverageAll, testing.coverageSelected, testing.coverage); C out\nls.messages.json ("Run All Tests with Coverage", "Run Tests with Coverage", "Run Test with Coverage")

### SB-6.10 Run with a chosen profile
- Tag: both
- Evidence: source
- Boundary: no
- What: When a test extension offers more than one profile for Run, Debug or Coverage, the title button becomes a split button. Its drop-down lists each controller's profiles (the default is marked "(Default)") and runs the visible tests with the one you pick. A row's right-click menu has Execute Using Profile... to pick a profile for that test. Extensions can add rows to the drop-down through the testing/profiles/context menu.
- Commands: testing.runUsing; vscode.pickTestProfile
- Menus: TestItem (group builtin@2, when testing.hasNonDefaultProfile); TestProfilesContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (title drop-down 311-451); V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (207-241); V\src\vs\workbench\contrib\testing\browser\testingConfigurationUi.ts (33-90, 133-160); V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts (361); C bundle (ids testing.runUsing, vscode.pickTestProfile, menu TestProfilesContext, context key testing.profile.context.group); C out\nls.messages.json ("Execute Using Profile...", "Select Configuration...", "{0} (Default)", "Pick a test profile to use")

### SB-6.11 Set the default profile and configure profiles
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose which profiles count as the default for Run, Debug and Coverage (a multi-select picker), and open a profile's own configuration step. Both are rows at the bottom of the title-button drop-down (Select Default Profile when more than one profile exists, Configure Test Profiles when one can be configured). Configure Test Profiles is also in the Command Palette. Select Default Profile has no palette entry of its own.
- Commands: testing.selectDefaultTestProfiles; testing.configureProfile; vscode.pickMultipleTestProfiles
- Menus: CommandPalette (testing.configureProfile, when testing.hasConfigurableProfile)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (254-277, 349-376); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (390-410); V\src\vs\workbench\contrib\testing\browser\testingConfigurationUi.ts (98-132); C bundle (ids testing.selectDefaultTestProfiles, testing.configureProfile, vscode.pickMultipleTestProfiles); C out\nls.messages.json ("Select Default Profile", "Configure Test Profiles", "Update Test Configuration", "Select a profile to update")

### SB-6.12 Turn continuous run on or off
- Tag: both
- Evidence: source
- Boundary: no
- What: Make tests re-run by themselves when files change. A row has a Turn on Continuous Run toggle (it shows Turn off Continuous Run when active, and is disabled when a parent is already running continuously), and the row's menu has Start Continous Run Using... to pick which profiles watch. The view title has Start Continuous Run and Stop Continuous Run buttons, which become a split button with a check per profile when several profiles support it. A row that is continuous, or has a continuous child, keeps its continuous-run icon visible even when not hovered.
- Commands: testing.toggleContinuousRunForTest; testing.continuousRunUsingForTest; testing.startContinuousRun; testing.stopContinuousRun
- Menus: TestItem (group inline, group builtin@1, and group builtin@2 for the "Using" row); ViewTitle (group navigation); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (279-347, 380-411, 487-511); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (continuous drop-down 453-511, row style 1572-1580); V\src\vs\workbench\contrib\testing\browser\media\testing.css (94-112); V\src\vs\workbench\contrib\testing\common\testingContinuousRunService.ts; C bundle (ids testing.toggleContinuousRunForTest, testing.continuousRunUsingForTest, testing.startContinuousRun, testing.stopContinuousRun); C out\nls.messages.json ("Turn on Continuous Run", "Turn off Continuous Run", "Start Continous Run Using...", "Start Continuous Run", "Stop Continuous Run")

### SB-6.13 Cancel a running test run
- Tag: both
- Evidence: source
- Boundary: no
- What: While tests are running, the Run, Debug and Coverage buttons in the view title are replaced by a Cancel Test Run button. The command also stops every unfinished run when called without a target.
- Commands: testing.cancelRun
- Menus: ViewTitle (group navigation, only while testing.isRunning); CommandPalette (while running)
- Keys: Ctrl/Cmd+; then Ctrl/Cmd+X
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (707-749); C bundle (id testing.cancelRun, keybinding chord Ctrl/Cmd+; X); C out\nls.messages.json ("Cancel Test Run")

### SB-6.14 Refresh tests
- Tag: both
- Evidence: source
- Boundary: no
- What: Ask test extensions to re-discover tests, from the Refresh Tests button in the view title, the inline button on a root row, or the Command Palette. While a refresh runs the button turns into Cancel Test Refresh, and the view shows a discovery progress bar. It only appears when a test controller supports refresh.
- Commands: testing.refreshTests; testing.cancelTestRefresh
- Menus: ViewTitle (group navigation, when testing.canRefresh); TestItem (group inline, root rows only); CommandPalette
- Keys: Ctrl/Cmd+; then Ctrl/Cmd+R (when testing.canRefresh)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (1722-1795); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (progress 523-531, root-row refresh context 1656-1659); C bundle (ids testing.refreshTests, testing.cancelTestRefresh, keybinding chord Ctrl/Cmd+; R); C out\nls.messages.json ("Refresh Tests", "Cancel Test Refresh", "Discovering Tests")

### SB-6.15 Right-click a test for its menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-clicking a test, suite or file row opens its context menu: Run Test, Debug Test, Run Test with Coverage, Execute Using Profile..., Start Continous Run Using..., Turn on or off Continuous Run, Go to Test, and Hide Test or Unhide Test (Refresh Tests is an inline button only, not a menu row). If the clicked row is part of the selection, the action applies to all selected rows. Extensions can add rows through the testing/item/context menu.
- Commands: testing.run; testing.debug; testing.coverage; testing.runUsing; testing.continuousRunUsingForTest; testing.toggleContinuousRunForTest; testing.editFocusedTest; testing.hideTest; testing.unhideTest
- Menus: TestItem (testing/item/context; the menu shows every group except inline)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (onContextMenu 1076-1090, getActionableElementActions 1642-1687); V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (89-162, 207-241, 279-347); V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts (351); C bundle (TestItem menu rows for the same ids with the same groups and orders; no Cursor-added rows found)

### SB-6.16 Open a test's source
- Tag: both
- Evidence: source
- Boundary: no
- What: Opening a test that has no child rows (a single click by default, a double click if the list open mode says so) opens its file at the test's line, or peeks the first error instead when the test has failed. Open to the side puts it beside the current editor. Go to Test in the row's menu, or Alt+Enter on the focused row, opens the file without the failure peek.
- Commands: testing.editFocusedTest; vscode.revealTest
- Menus: TestItem (group builtin@1, when testing.testItemHasUri)
- Keys: Alt+Enter (when the focused view is workbench.view.testing)
- Settings: workbench.list.openMode
- Cursor: present
- Source: V\src\vs\platform\list\browser\listService.ts (openMode 171, 693-701); V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (961-991); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (open handler 857-872, peek first error 1070-1076); V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts (vscode.revealTest 210-258); C bundle (ids testing.editFocusedTest, vscode.revealTest, keybinding Alt+Enter for view workbench.view.testing); C out\nls.messages.json ("Go to Test")

### SB-6.17 Hide and unhide tests
- Tag: both
- Evidence: source
- Boundary: no
- What: Hide a test, suite or file from the tree with Hide Test in its menu. Hidden rows disappear until you turn on Show Hidden Tests (they then show dimmed with a hidden icon and an Unhide Test entry), or clear everything with Unhide All Tests. Revealing a hidden test from elsewhere turns Show Hidden Tests on automatically.
- Commands: testing.hideTest; testing.unhideTest; testing.unhideAllTests
- Menus: TestItem (group builtin@2 for Hide Test; Unhide Test with order 21)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (89-148); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (revealById 988-1057, hidden row style 1591-1597, hidden filter 1222-1228); V\src\vs\workbench\contrib\testing\browser\testingExplorerFilter.ts (228-243); C bundle (ids testing.hideTest, testing.unhideTest, testing.unhideAllTests); C out\nls.messages.json ("Hide Test", "Unhide Test", "Unhide All Tests", "Show Hidden Tests")

### SB-6.18 Follow the running test in the tree
- Tag: both
- Evidence: source
- Boundary: no
- What: With follow turned on, the tree scrolls to and selects the test that is currently running (it does nothing while several rows are selected). With "always reveal" off, only tests that fail are revealed.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: testing.followRunningTest; testing.alwaysRevealTestOnStateChange
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (895-933); V\src\vs\workbench\contrib\testing\common\configuration.ts (114-117, 162-166); C bundle (settings testing.followRunningTest, testing.alwaysRevealTestOnStateChange); C out\nls.messages.json ("Controls whether the running test should be followed in the Test Explorer view.")

### SB-6.19 Read a test's status in its row
- Tag: both
- Evidence: source
- Boundary: no
- What: Each row shows a state icon (passed, failed, errored, queued, running with a spinner, skipped, not yet run), a lighter icon color (70 percent opacity, theme colors testing.icon*.retired) when the result is outdated, an optional description, and the last duration (for example 12ms or 1.2s). Hovering reads the full label such as "Test name (Passed), in 12ms" or ", outdated result".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (label text 1391-1409, TestItemRenderer 1500-1626, duration format 1628-1638); V\src\vs\workbench\contrib\testing\common\constants.ts (36-48); V\src\vs\workbench\contrib\testing\browser\icons.ts; V\src\vs\workbench\contrib\testing\browser\theme.ts (167-173); V\src\vs\workbench\contrib\testing\browser\media\testing.css (94-117, 156); C out\nls.messages.json ("{0}, in {1}", "{0}, outdated result", "Not yet run", "Passed", "Failed", "Errored", "Queued", "Running", "Skipped")

### SB-6.20 See the run summary and Testing icon badge
- Tag: both
- Evidence: source
- Boundary: no
- What: A row above the tree shows the last run as a status icon, passed over total count (hover gives the detail), and the duration, with Rerun Last Run and Debug Last Run buttons once the run is done. With no runs it says "No test results yet." The same code sends a badge for the Testing icon: the number of failed, passed or skipped tests (or none), and a watch icon while continuous run is on. Stock also shows a spinner badge while tests run. The icon itself is drawn by the rail (AB-3).
- Commands: testing.reRunLastRun; testing.debugLastRun
- Menus: none found
- Keys: none found
- Settings: testing.countBadge
- Cursor: changed (no spinner badge while tests run, the rest is present)
- Source: V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (ResultSummaryView 545-681); V\src\vs\workbench\contrib\testing\common\configuration.ts (100-113); C bundle (renderActivityBadge has no running branch, only count and continuous badges); C out\nls.messages.json ("No test results yet.", "{0} failed tests", "{0} passed tests", "{0} skipped tests", "Tests are being watched for changes", "Controls the count badge on the Testing icon on the Activity Bar."; the string "Tests are running" is not present)

### SB-6.21 Re-run failed tests or the last run
- Tag: both
- Evidence: source
- Boundary: no
- What: Command Palette commands that repeat earlier work: Rerun Failed Tests and Debug Failed Tests (a test counts as failed until a later run passes it), Rerun Last Run, Debug Last Run and Rerun Last Run with Coverage (same profiles as before when they still exist), and Rerun or Debug Failed Tests from Last Run (only the failures of the newest run). They show up when a test provider exists, and the last-run ones also need results. Rerun Last Run and Debug Last Run are also the two buttons in the summary row (SB-6.20).
- Commands: testing.reRunFailTests; testing.debugFailTests; testing.reRunLastRun; testing.debugLastRun; testing.coverageLastRun; testing.reRunFailedFromLastRun; testing.debugFailedFromLastRun
- Menus: CommandPalette
- Keys: Ctrl/Cmd+; then E (reRunFailTests); Ctrl/Cmd+; then Ctrl/Cmd+E (debugFailTests); Ctrl/Cmd+; then L (reRunLastRun); Ctrl/Cmd+; then Ctrl/Cmd+L (debugLastRun); Ctrl/Cmd+; then Ctrl/Cmd+Shift+L (coverageLastRun)
- Settings: none found
- Cursor: changed (the two "from last run" commands and their labels are not found, the other five are present)
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (1386-1663); C bundle (ids testing.reRunFailTests, testing.debugFailTests, testing.reRunLastRun, testing.debugLastRun, testing.coverageLastRun; ids testing.reRunFailedFromLastRun and testing.debugFailedFromLastRun not found); C out\nls.messages.json ("Rerun Failed Tests", "Debug Failed Tests", "Rerun Last Run", "Debug Last Run", "Rerun Last Run with Coverage")

### SB-6.22 See the empty states
- Tag: both
- Evidence: source
- Boundary: no
- What: Once discovery has finished and no tests exist, the view shows "No tests have been found in this workspace yet." with an "Install Additional Test Extensions..." link (it opens an Extensions view search for the testing category, and that view is out of scope here). If the filter is set to the active file and that file has no tests, the tree is replaced by "No tests were found in this file." with a Show Workspace Tests button that switches the file filter off.
- Commands: testing.searchForTestExtension
- Menus: none found (the text is view welcome content)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts (welcome content 103-110); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts (welcome state 146-148, 1113-1123, NoTestsForDocumentWidget 1351-1370); V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (1665-1676); C bundle (welcome content registered for workbench.view.testing with link command:testing.searchForTestExtension); C out\nls.messages.json ("No tests have been found in this workspace yet.", "Install Additional Test Extensions...", "No tests were found in this file.", "Show Workspace Tests", "Search for Test Extension")

### SB-6.23 Open the Testing views automatically
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose whether starting a run opens a view for you: never, open Test Results when tests start (the default), open Test Results on the first failure, or open the Test Explorer when tests start. A run that asks to keep focus where it is never opens anything. The same choice also governs the Test Results view (PN-6). It replaces the older setting testing.openTesting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: testing.automaticallyOpenTestResults
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testingProgressUiService.ts (17-83); V\src\vs\workbench\contrib\testing\common\configuration.ts (33-38, 145-161, 222-232); C bundle (setting testing.automaticallyOpenTestResults, migration from testing.openTesting and testing.automaticallyOpenResults); C out\nls.messages.json ("Controls when the testing view should open.")

### SB-6.24 Browse the Test Coverage view
- Tag: both
- Evidence: source
- Boundary: no
- What: While a coverage report is open, a Test Coverage view joins the Testing container. It is a tree of folders and files (single-child folder chains are merged), each row showing the overall percent and compact bars for statements, functions and branches, with a hover that gives covered over total counts. Expanding a file loads its declarations with their own bars. Opening a row opens the file, at the function for a declaration, and open to the side is supported. A Collapse All Coverage button sits in the view title. Settings pick which percent is the headline (total, statement or minimum) and the red, yellow and green cut-offs.
- Commands: testing.coverageViewCollapseAll
- Menus: ViewTitle (only for view workbench.view.testCoverage)
- Keys: none found
- Settings: testing.displayedCoveragePercent; testing.coverageBarThresholds
- Cursor: changed (view, tree, bars and both settings are present; the Collapse All Coverage command and label are not found)
- Source: V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts (view 122-131); V\src\vs\workbench\contrib\testing\browser\testCoverageView.ts (view 62-120, tree 232-335, renderers 440-700, collapse all 805-822); V\src\vs\workbench\contrib\testing\browser\testCoverageBars.ts (43-135); V\src\vs\workbench\contrib\testing\common\testCoverageService.ts (100-145); V\src\vs\workbench\contrib\testing\common\configuration.ts (168-193); C bundle (view id workbench.view.testCoverage, settings testing.displayedCoveragePercent and testing.coverageBarThresholds; id testing.coverageViewCollapseAll not found); C out\nls.messages.json ("Test Coverage", "Test Coverage Explorer"; "Collapse All Coverage" not found)

### SB-6.25 Sort the coverage view
- Tag: both
- Evidence: source
- Boundary: no
- What: The Change Sort Order button in the Test Coverage title opens a picker with Sort by Location (files alphabetical, declarations by position), Sort by Coverage (best covered first), and Sort by Name. Stock remembers the choice per workspace.
- Commands: testing.coverageViewChangeSorting
- Menus: ViewTitle (group navigation, order 1, only for view workbench.view.testCoverage)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testCoverageView.ts (62-100, 764-803); C bundle (id testing.coverageViewChangeSorting, view id workbench.view.testCoverage); C out\nls.messages.json ("Change Sort Order", "Sort by Coverage", "Sort by Name", "Sort the Test Coverage view...")

### SB-6.26 Filter coverage to one test
- Tag: both
- Evidence: source
- Boundary: no
- What: When the coverage has per-test data, Filter Coverage by Test opens a picker with All Tests and each test, and the view previews the pick as you move through it. The view then shows a first row "Showing coverage for ..." with a filter button that reopens the picker. The button in the view title looks filled while a filter is active.
- Commands: testing.coverageFilterToTest
- Menus: ViewTitle (group navigation, when testing.hasPerTestCoverage, only for view workbench.view.testCoverage); CommandPalette (when testing.hasPerTestCoverage); TestCoverageFilterItem (group inline)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testCoverageView.ts (707-761, filter row 202-216 and 501-538); V\src\vs\workbench\contrib\testing\common\testCoverageService.ts; C bundle (id testing.coverageFilterToTest, menu TestCoverageFilterItem); C out\nls.messages.json ("Filter Coverage by Test")

### SB-6.27 Close or reopen the coverage report
- Tag: both
- Evidence: source
- Boundary: no
- What: Clear Coverage (the X in the Test Coverage title, or the Command Palette while coverage is open) closes the report and hides the Test Coverage view. Open Coverage in the Command Palette reopens the report from the newest run, or says "No coverage information available on the last test run." A new run that finishes without coverage closes the old report, and one that has coverage opens it without taking focus.
- Commands: testing.coverage.close; testing.openCoverage
- Menus: ViewTitle (group navigation, only for view workbench.view.testCoverage); CommandPalette (testing.coverage.close when testing.isTestCoverageOpen; testing.openCoverage when testing.hasAnyResults)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (1796-1843); V\src\vs\workbench\contrib\testing\common\testCoverageService.ts (100-145); C bundle (ids testing.coverage.close, testing.openCoverage); C out\nls.messages.json ("Clear Coverage", "Open Coverage", "No coverage information available on the last test run.")

### SB-6.28 Show coverage bars in the file Explorer
- Tag: both
- Evidence: source
- Boundary: no
- What: While a coverage report is open, files and folders in the Explorer file tree (SB-2) show compact coverage bars, with a hover giving the counts. Toggle Coverage in Explorer in the Command Palette (only while coverage is open) flips the setting, and its menu title changes to Hide Coverage in Explorer when on.
- Commands: testing.toggleCoverageInExplorer
- Menus: CommandPalette (when testing.isTestCoverageOpen)
- Keys: none found
- Settings: testing.showCoverageInExplorer
- Cursor: changed (the setting and the Explorer bar code are present; the toggle command and its labels are not found)
- Source: V\src\vs\workbench\contrib\testing\browser\codeCoverageDecorations.ts (997-1019); V\src\vs\workbench\contrib\testing\browser\testCoverageBars.ts (183-236); V\src\vs\workbench\contrib\testing\browser\testingProgressUiService.ts (34-45); V\src\vs\workbench\contrib\testing\common\configuration.ts (168-171); C bundle (setting testing.showCoverageInExplorer and the Explorer bar class that reads it; id testing.toggleCoverageInExplorer not found); C out\nls.messages.json ("Whether test coverage should be down in the File Explorer view."; "Toggle Coverage in Explorer" and "Hide Coverage in Explorer" not found)

### SB-6.29 Run, debug or cover the tests in the current file
- Tag: both
- Evidence: source
- Boundary: no
- What: Run all tests of the file you are editing, debug them, or run them with coverage, from the Command Palette, the editor right-click menu or a key chord.
- Commands: testing.runCurrentFile; testing.debugCurrentFile; testing.coverageCurrentFile
- Menus: CommandPalette (Test category, when the matching run profile exists); EditorContext (group testing, when the active editor has tests and the profile exists)
- Keys: Ctrl+; then F (run); Ctrl+; then Ctrl+F (debug); Ctrl+; then Ctrl+Shift+F (coverage), all when the editor has text focus
- Settings: testing.saveBeforeTest
- Cursor: present - ids, labels (nls 11100 to 11102) and the chords are in the bundle
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts:1239-1372; C bundle (classes with ids testing.runCurrentFile, testing.debugCurrentFile and testing.coverageCurrentFile and their key chords)

### SB-6.30 Run, debug or cover the test at the cursor
- Tag: both
- Evidence: source
- Boundary: no
- What: With the text cursor in a test file, run, debug or run with coverage the test that contains the cursor, from the Command Palette, the editor right-click menu or a key chord.
- Commands: testing.runAtCursor; testing.debugAtCursor; testing.coverageAtCursor
- Menus: CommandPalette (Test category, when a test provider exists); EditorContext (group testing, Run Test at Cursor first and Debug Test at Cursor and the coverage entry in the next slot, when the active editor has tests and a profile of that kind exists)
- Keys: Ctrl+; then C (run); Ctrl+; then Ctrl+C (debug); Ctrl+; then Ctrl+Shift+C (coverage), all when the editor has text focus (Cmd on macOS)
- Settings: testing.saveBeforeTest
- Cursor: present - the three ids, the labels (nls 11097 to 11099), the run and debug chords and "No tests found here" are in the bundle. ED-11 handed these commands to SB-6 and SB-6 has no feature for them; the parent may merge this with SB-6.29, its current-file sibling.
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (ExecuteTestAtCursor lines 1048-1131, RunAtCursor, DebugAtCursor and CoverageAtCursor lines 1133-1176); V\src\vs\workbench\contrib\testing\common\constants.ts (lines 80 and 105); C bundle (classes with ids testing.runAtCursor, testing.debugAtCursor and testing.coverageAtCursor); C out\nls.messages.json (labels, "No tests found here")

## Handed off
- Test gutter buttons in the editor (run, debug or coverage glyph, the click action, the right-click gutter menu, settings testing.gutterEnabled and testing.defaultGutterClickAction, menu testing/item/gutter, file testingDecorations.ts): ED-11.
- Run, debug or cover tests at the cursor or in the current file (editor context menu group testing, Command Palette, chords Ctrl/Cmd+; C and Ctrl/Cmd+; F with their Ctrl/Cmd and Shift variants): ED-11.
- Peek test failure output in the editor, inline error messages, next and previous message, Toggle Inline Test Output, Peek Output (testing.openOutputPeek, testing.toggleInlineTestOutput), and settings testing.automaticallyOpenPeekView, testing.automaticallyOpenPeekViewDuringAutoRun, testing.showAllMessages: ED-11.
- Coverage in the editor (line decorations, Toggle Inline Coverage, the coverage toolbar, next and previous uncovered line, Filter Coverage to Test in the editor title, settings testing.coverageToolbarEnabled and testing.coverageMinimapEnabled): ED-11. The setting testing.coverageMinimapEnabled is not found in the Cursor bundle, and the commands testing.coverage.goToNextMissedLine and testing.coverage.goToPreviousMissedLine are not found either.
- Go to Related Test, Peek Related Test, Go to Related Code, Peek Related Code (editor context menu group testing, ids testing.goToRelatedTest, testing.peekRelatedTest, testing.goToRelatedCode, testing.peekRelatedCode): ED-4 (they reuse go to and peek). All four ids are present in the Cursor bundle.
- Run, Debug and Run with Coverage for the tests under a file or folder from the Explorer right-click menu (testing.run.uri, testing.debug.uri, testing.coverage.uri, menu ExplorerContext group 6.5_testing): SB-2.
- The Test Results view in the Panel and its commands: Show Output (testing.showMostRecentOutput, the button in the Test Explorer title, Ctrl/Cmd+; then Ctrl/Cmd+O), Clear All Results (testing.clearTestResults, in the Test Explorer overflow menu and in the Results title), Toggle Tree Position (testing.toggleResultsViewLayout) and setting testing.resultsView.layout: PN-6. The command testing.toggleResultsViewLayout and the setting testing.resultsView.layout are not found in the Cursor bundle.
- Drawing the Testing count badge on the rail icon: AB-3 (SB-6.20 owns what the badge says and its setting).
- The View menu row "Testing" that opens the view: TB-1 (SB-6.1 records its command id).

## Unclear
- The group has 28 features, above the usual 20. It covers two views (Test Explorer and Test Coverage) with about 50 commands, and I kept run, debug and coverage as separate features because each has its own button, menu rows and key chord. I did not merge them to hit a count.
- ED-11 is marked proposed in the catalog. If it is cut, the test gutter, run at cursor, output peek and editor coverage items listed above have no owner.
- Pass 1 for this section listed testing.automaticallyOpenResults as a stock setting. In 1.128.0 it only exists as a migration key. The real setting is testing.automaticallyOpenTestResults (SB-6.23).
- workbench.view.testing.focus is built from the view id in viewsService.ts, so it appears as no literal in V or in the Cursor bundle. I list it from the code, not from a search hit.
- Stock saves the coverage sort choice per workspace under the storage key testing.coverageSortOrder. That key was not found in the Cursor bundle, so I could not confirm Cursor remembers the sort across sessions.
- In Cursor's default tab row layout the Testing container should sit in the overflow list (AB-6 default pins only Explorer, Search, Source Control and Extensions). That comes from pass 1 strings and was not checked on screen. All Cursor results here are strings in the bundle and in its message file, nothing was run.
- API-only commands with no UI of their own were left out: testing.getSelectedProfiles, _testing.getExplorerSelection, vscode.runTestsById, vscode.testing.getTestsInFile, vscode.testing.getControllersWithTests, vscode.peekTestError, _revealTestInExplorer, testing.startContinuousRunFromExtension, testing.stopContinuousRunFromExtension.
- Left out as agent or extension surfaces: the Testing chat agent tool (testingChatAgentTool.ts) and the copilot extension row on the test item menu in V\extensions\copilot\package.json (when testing.testItemHasUri). Neither exists in the Cursor install.
- No Cursor-only Testing items were found: the TestItem menu rows, view ids, container and settings in the bundle match stock apart from the gaps marked as changed. No built-in extension in the Cursor install contributes to testing menus.
- Nothing newer than 1.128 was found. The testing browser and common folders in N have the same files, the same command ids and settings, and the eight main files compared (actions, explorer view, filter, coverage view, contribution, coverage decorations, coverage bars, progress) are line for line identical.
