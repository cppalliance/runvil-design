# Panel (PN) - VS Code 1.128.0 findings

Section: Panel (PN). Date: 2026-10-25. Pass: 1.

- Scope of this file: stock VS Code 1.128.0 only (root V). Cursor findings are in the sibling file under `cursor\`.
- Level: minor groups only. No full feature lists, no pieces. Up to 4 example features per group.
- Short names: V = `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0`. N = `C:\Users\Vinnie\src\vscode` (1.144.0, used only to spot newer things). C = the Cursor install `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`. All proof paths below start under `V\src\vs\workbench\` unless they say otherwise.
- Line numbers are from the files as read. They can drift by a line or two, so search the quoted id or label if one does not match.
- Tags in this file say `both` when I also searched the Cursor bundle for a command id or label of that item. The Cursor evidence itself is in the sibling file.

## What this section is

The pane that normally sits under the editor area. It has a title row with tabs (or icons), a toolbar for the active view, and a body that shows one view at a time. It can also sit on the left, right or top.

The part itself is small. The real content is the set of views that register into it. In V the panel hosts these view containers (each one registers with `ViewContainerLocation.Panel`):

- Terminal (default tab): id `terminal`. Registered in `contrib\terminal\browser\terminal.contribution.ts` lines 106-135 with `isDefault: true`. Id from `contrib\terminal\common\terminal.ts` line 24.
- Problems: id `workbench.panel.markers`. Registered in `contrib\markers\browser\markers.contribution.ts` lines 132-141.
- Output: id `workbench.panel.output`. Registered in `contrib\output\browser\output.contribution.ts` lines 73-82.
- Debug Console: id `workbench.panel.repl`. Registered in `contrib\debug\browser\debug.contribution.ts` lines 426-450.
- Test Results: id `workbench.panel.testResults`. Registered in `contrib\testing\browser\testing.contribution.ts` lines 81-100.
- Comments: id `workbench.panel.comments`. Registered in `api\browser\mainThreadComments.ts` lines 709-732, only built when an extension provides comments.
- Ports: id `~remote.forwardedPortsContainer`. Registered in `contrib\remote\browser\remoteExplorer.ts` lines 72-80, id from `services\remote\common\remoteExplorerService.ts` line 24, only built when forwarded ports features are on.
- Refactor Preview: id `refactorPreview`. Registered in `contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts` lines 328-338.

Rule used in this pass: any view whose container is in the Side Bar or Auxiliary Bar can be dragged into the panel (the terminal view sets `canMoveView: true`, `terminal.contribution.ts` line 121). The panel hosts it, but its features belong to its own section. I did not list those features here.

Tab order comes from each container's `order` value. In V: Problems 0 (`markers.contribution.ts` line 137), Output 1 (`output.contribution.ts` line 77), Debug Console 2 (`debug.contribution.ts` line 433), Terminal 3 (`terminal.contribution.ts` line 113), Test Results 3 (`testing.contribution.ts` line 87), Ports 5 (`remoteExplorer.ts` line 79), Comments 10 (`mainThreadComments.ts` line 719). Refactor Preview sets no order.

Most panel views hide themselves when empty (`hideIfEmpty: true` on Terminal, Test Results, Ports, Comments). Debug Console only shows when a debugger is available (`when: CONTEXT_DEBUGGERS_AVAILABLE`, `debug.contribution.ts` line 442). Test Results only shows once there are results (`when: TestingContextKeys.hasAnyResults`, `testing.contribution.ts` line 98).

## Proposed minor groups

Count: 7. Reason: the panel is one frame plus a set of independent views, and the terminal alone is bigger than the rest combined. The brief says aim for 3 to 6 and up to 8 if needed. I need 7 because (a) the terminal splits into two groups, (b) Problems and Output are separate tabs that people think of separately, (c) Debug Console and Test Results fit together as "results of running things", and (d) Comments, Ports and Refactor Preview have no other home and only show up when something feeds them. Fewer than 7 would merge Problems with Output (different tools to a user) or fold the terminal into one very large group.

Terminal decision: two groups, not one. PN-2 is about managing terminal sessions (create, profiles, tabs, split, move, kill). PN-3 is about what happens inside one terminal (find, links, shell integration, suggestions, copy and paste). The split follows the code: session management lives in `contrib\terminal\browser\` (`terminalActions.ts`, `terminalTabsList.ts`, `terminalGroup.ts`, `terminalView.ts`) and the tooling lives in 23 separate `contrib\terminalContrib\*` folders (25 exist, minus `chat` and `chatAgentTools`) that each register a self-contained feature. Confidence: medium - a reasonable person could keep it as one group and split later at the feature level.

### PN-1 Panel frame and layout

- Plain name: Panel frame and layout
- One sentence: The container itself - show or hide it, put it at the bottom, top, left or right, align it, maximize it, and switch between its tabs.
- Tag: both (cursor: searched, command ids found)
- Evidence: source
- Boundary: no
- Example features:
  - Show or hide the panel (`workbench.action.togglePanel`, Ctrl+J, View menu Appearance, title bar layout toggle)
  - Panel position and alignment (Move Panel Left, Right, Top, Bottom; Align Left, Right, Center, Justify)
  - Maximize and restore the panel (`workbench.action.toggleMaximizedPanel`, only when center aligned or left or right), with `workbench.panel.opensMaximized`
  - Panel tab bar: tabs as labels or icons, overflow, next and previous panel view, pin and unpin from the tab bar menu
- Proof paths:
  - `browser\parts\panel\panelActions.ts` (toggle lines 31-72, position and align lines 127-250, previous and next view lines 252-273, maximize lines 275-318, layout control lines 320-342, move panel views to and from secondary side bar lines 344-424)
  - `browser\parts\panel\panelPart.ts` (composite bar options lines 135-163, tab bar context menu lines 167-195, size limits lines 40-65)
  - `browser\parts\panel\media\panelpart.css`
  - `browser\workbench.contribution.ts` lines 573-593 (`workbench.panel.showLabels`, `defaultLocation`, `opensMaximized`)
- Notes: the panel title row shows the active view's own toolbar on the right plus Maximize (order 1) and Hide Panel (order 2) from `MenuId.PanelTitle`. Panel settings keys stored: `workbench.panel.pinnedPanels`, `workbench.panelpart.activepanelid` (`panelPart.ts` lines 70 and 139-141).

### PN-2 Terminal sessions

- Plain name: Terminal sessions
- One sentence: Creating, naming, arranging and closing terminals - profiles, the tabs list, splits and groups, and moving a terminal to the editor area or a new window.
- Tag: both (cursor: searched, command ids found)
- Evidence: source
- Boundary: no
- Example features:
  - New terminal and profile picker (`workbench.action.terminal.new`, `newWithProfile`, the New Terminal dropdown with Select Default Profile, Configure Terminal Settings and Run Task)
  - Terminal tabs list on the side (rename, change icon, change color, status icons, hover details, hide when single, left or right)
  - Split terminals and terminal groups (split, unsplit, join, resize pane, focus next pane)
  - Move a terminal to the editor area, a new window, or back to the panel
- Proof paths:
  - `contrib\terminal\browser\terminal.contribution.ts` (container and view, lines 106-135; terminal editor registration lines 71-104)
  - `contrib\terminal\browser\terminalView.ts` (view pane, new-terminal dropdown line 293, welcome view lines 350-356, single-tab action item line 405)
  - `contrib\terminal\browser\terminalActions.ts` (ids such as `TerminalCommandId.New`, `Split`, `Unsplit`, `Join`, `Kill`, `KillAll`, `Rename`, `ChangeIcon`, `ChangeColor`, `MoveToEditor`, `MoveIntoNewWindow`, `MoveToTerminalPanel`, `Relaunch`, `FocusTabs`)
  - `contrib\terminal\browser\terminalMenus.ts` (Terminal menu lines 41-105, instance context 108-185, new dropdown 332-380, view title toolbar 381-550, tab context 556 onward)
  - `contrib\terminal\browser\terminalTabbedView.ts`, `terminalTabsList.ts`, `terminalGroup.ts`, `terminalGroupService.ts`, `terminalEditor.ts`, `terminalEditorService.ts`, `terminalStatusList.ts`, `terminalTooltip.ts`, `terminalProfileQuickpick.ts`
  - Settings: `contrib\terminal\common\terminalConfiguration.ts` lines 63-143 (`terminal.integrated.tabs.enabled`, `tabs.hideCondition`, `tabs.showActions`, `tabs.location`, `defaultLocation`, `tabs.focusMode`), plus `enablePersistentSessions`, `hideOnStartup`, `hideOnLastClosed`, `confirmOnKill`, `splitCwd`
- Notes:
  - Task terminals are part of this group. A task run reuses the terminal view and shows its state on the tab: spinner while running, then check, warning or error icons with tooltips (`contrib\tasks\browser\taskTerminalStatus.ts` lines 29-37). The exit line says "Terminal will be reused by tasks, press any key to close it." (`contrib\tasks\browser\terminalTaskSystem.ts` line 2014). The task commands themselves (Run Task, Run Build Task, Show Running Tasks, Terminate Task, Restart Running Task) are in the Terminal menu and are registered in `contrib\tasks\browser\task.contribution.ts`. The pickers are quick picks, so they belong to the Quick Access (QA) section.
  - A terminal can live in the editor area as an editor tab (`terminalEditor.ts`, `terminalEditorInput.ts`). The editor-tab side belongs to the Editor (ED) section. The terminal itself is described here.
  - Hidden chat terminals show a row at the bottom of the tabs list (`terminalTabsChatEntry.ts`). That is chat, so it is left out below.

### PN-3 Terminal tooling

- Plain name: Terminal tooling
- One sentence: Everything you use inside one terminal once it is open - find, links, command marks from shell integration, sticky scroll, quick fixes, command suggestions, history, copy and paste, zoom and accessibility.
- Tag: both (cursor: searched, command ids and settings found, with differences noted in the sibling file)
- Evidence: source
- Boundary: no
- Example features:
  - Find in terminal (find widget with regex, whole word, case; Search Workspace)
  - Shell integration marks: command decorations in the gutter, jump to previous or next command, select to command, sticky scroll, command guide
  - Links and quick fixes: open detected file, URL and word links; quick fix lightbulb after a failed command
  - Terminal IntelliSense (suggest widget for commands, arguments and paths) and Run Recent Command or Go to Recent Directory
- Proof paths:
  - `contrib\terminalContrib\find\browser\terminalFindWidget.ts`, `terminal.find.contribution.ts`
  - `contrib\terminalContrib\links\browser\terminalLinkManager.ts`, `terminal.links.contribution.ts`, `terminalLinkQuickpick.ts`
  - `contrib\terminalContrib\quickFix\browser\quickFixAddon.ts`, `terminalQuickFixBuiltinActions.ts`
  - `contrib\terminalContrib\stickyScroll\browser\terminalStickyScrollOverlay.ts`, `commandGuide\browser\terminal.commandGuide.contribution.ts`
  - `contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts`, `terminal.suggest.contribution.ts`, `common\terminalSuggestConfiguration.ts` (12 settings)
  - `contrib\terminalContrib\history\browser\terminalRunRecentQuickPick.ts`
  - `contrib\terminal\browser\xterm\decorationAddon.ts` and `markNavigationAddon.ts` (command decorations and mark navigation), `contrib\terminal\common\terminalConfiguration.ts` line 610 onward (`shellIntegration.enabled`, `decorationsEnabled`)
  - Clipboard, zoom, accessibility: `contrib\terminalContrib\clipboard\browser\terminal.clipboard.contribution.ts`, `zoom\browser\terminal.zoom.contribution.ts`, `accessibility\browser\terminal.accessibility.contribution.ts`
- Other `terminalContrib` folders in this group (not examples, listed so the next pass sees them): `autoReplies`, `environmentChanges` (Show Environment Contributions), `notification`, `resizeDimensionsOverlay`, `sendSequence`, `sendSignal`, `typeAhead`, `developer` (dev only commands such as Show Terminal Texture Atlas, Write Data to Terminal, Restart Pty Host), `wslRecommendation`, `quickAccess` (the `term ` quick pick, owned by QA), `telemetry` (no UI), `voice` (Start Dictation in Terminal). All under `contrib\terminalContrib\`.
- Other terminal-wide controls that belong here: appearance and behavior settings (font, cursor, scrollback, GPU acceleration, copy on select, right click behavior, bell) in `contrib\terminal\common\terminalConfiguration.ts`; right-click menu with Copy, Copy as HTML, Paste, Select All, Clear, Size to Content Width (`terminalMenus.ts` lines 108-185).

### PN-4 Problems

- Plain name: Problems
- One sentence: The list of errors, warnings and infos from all files, as a tree or a table, with filters and quick fixes.
- Tag: both (cursor: searched, command ids found; Cursor adds two chat entries, see sibling file)
- Evidence: source
- Boundary: no in stock VS Code. The Cursor file flags yes for two added menu entries.
- Example features:
  - Problems tree and table views (View as Tree, View as Table, `problems.defaultViewMode`)
  - Filter box and filter menu (text filter, Show Errors, Show Warnings, Show Infos, Show Active File Only, Show Excluded Files)
  - Row actions: open, open to the side, quick fix, copy, copy message, show message on one line or many
  - Problems status in the status bar and the count badge on the panel tab (`problems.showCurrentInStatus`, `problems.visibility`; badge at `markers.contribution.ts` line 714)
- Proof paths:
  - `contrib\markers\browser\markers.contribution.ts` (container lines 132-141, actions from line 143, settings lines 100-125)
  - `contrib\markers\browser\markersView.ts`, `markersTable.ts`, `markersTreeViewer.ts`, `markersFilterOptions.ts`, `markersViewActions.ts`, `messages.ts`, `markersAccessibilityHelp.ts`
  - Context menu id `MenuId.ProblemsPanelContext`; command ids `workbench.actions.view.problems`, `workbench.action.problems.focus`, `workbench.actions.table.workbench.panel.markers.view.viewAsTable`
- Notes: `markersChatContext.ts` attaches problems to chat. That is chat, so left out. `markersFileDecorations.ts` puts error marks on Explorer rows and belongs to the Side Bar section.

### PN-5 Output and logs

- Plain name: Output and logs
- One sentence: A read-only text view that shows one named channel at a time - extension output, tool output and log files - with a channel picker and log tools.
- Tag: both (cursor: searched, command ids found; Cursor adds many channels, see sibling file)
- Evidence: source
- Boundary: no
- Example features:
  - Channel picker (Switch Output drop-down, Show Output Channels)
  - Log level and filter (Set Log Level, trace, debug, info, warning, error toggles, filter text)
  - View tools (Clear Output, Toggle Auto Scrolling, `output.smartScroll.enabled`; text always wraps, `outputView.ts` line 258)
  - Open or move out: Open Output in Editor, Open Output in New Window, Save Output As, Export Logs, Import Log, Add Compound Log
- Proof paths:
  - `contrib\output\browser\output.contribution.ts` (container lines 73-82; actions, log level menu, filters, import and export further down)
  - `contrib\output\browser\outputView.ts`, `outputServices.ts`, `outputLinkProvider.ts` (clickable file links in output), `outputAccessibilityHelp.ts`
  - `contrib\output\common\outputChannelModel.ts`
  - Setting `workbench.view.showQuietly` with the `workbench.panel.output` property: when on, an extension asking to show Output gets a clickable status bar indicator instead (`browser\workbench.contribution.ts` lines 691-700)
- Notes: `workbench.action.showLogs` and `workbench.action.openLogFile` are Command Palette only and open pickers (QA section). Their output lands here.

### PN-6 Debug Console and Test Results

- Plain name: Debug Console and Test Results
- One sentence: The two panel views that show the result of running your code - the Debug Console (a REPL attached to a debug session) and Test Results (the last test runs with their messages and output).
- Tag: both (cursor: searched, command ids found)
- Evidence: source
- Boundary: no
- Example features:
  - Debug Console REPL (evaluate expressions with autocomplete and history, select which session's console to show)
  - Debug Console view tools (filter, find, clear, collapse all, word wrap, collapse identical lines)
  - Test Results tree (runs, suites and tests with status icons)
  - Test Results message and output pane (diff, terminal-style output, stack, previous and next message, layout toggle, clear results, show most recent output)
- Proof paths:
  - Debug Console: `contrib\debug\browser\repl.ts` (ids `repl.action.filter`, `repl.action.find`, `workbench.debug.panel.action.clearReplAction`, `debug.collapseRepl`, `debug.replPaste`, `repl.action.copyAll`), `contrib\debug\browser\debug.contribution.ts` lines 426-450 and settings `debug.console.fontSize`, `wordWrap`, `collapseIdenticalLines`, `historySuggestions`, `acceptSuggestionOnEnter`, `maximumLines`, `closeOnEnd`, `lineHeight`, `fontFamily`
  - Test Results: `contrib\testing\browser\testingOutputPeek.ts` (`TestResultsView`), `testResultsView\testResultsTree.ts`, `testResultsViewContent.ts`, `testResultsOutput.ts`, `testResultsSubject.ts`, `testExplorerActions.ts` (`ShowMostRecentOutputAction`, `ClearTestResultsAction`, `ToggleResultsViewLayoutAction`), `contrib\testing\common\configuration.ts` (`testing.automaticallyOpenTestResults`, `testing.openTesting`)
- Notes: the Test Explorer (Side Bar), test gutter icons and coverage decorations (Editor), and the inline test peek in the editor are not in the panel. They belong to the Side Bar and Editor sections. The test peek in the editor (`testingOutputPeek.ts`) shares code with Test Results, so the next pass should decide where the shared parts are described.

### PN-7 Context-only panel views

- Plain name: Context-only panel views
- One sentence: Panel tabs that only appear when something feeds them - Comments from a review extension, Ports when forwarding is on, and Refactor Preview during a bulk edit.
- Tag: both (cursor: searched, command ids and container ids found)
- Evidence: source
- Boundary: no
- Example features:
  - Comments view (tree of comment threads across files, filter, show resolved or unresolved, sort by time or position, collapse and expand, reply)
  - Ports view (Forward a Port, local address, visibility, protocol, open in browser, preview in editor, copy address, set label)
  - Refactor Preview (list of pending edits with checkboxes, Apply Refactoring, Discard Refactoring, group by file or type)
- Proof paths:
  - Comments: `contrib\comments\browser\commentsView.ts` (`CommentsPanel`), `commentsViewActions.ts`, `commentsTreeViewer.ts`, `comments.contribution.ts` (settings `comments.openView`, `comments.useRelativeTime`, `comments.visible`), registration in `api\browser\mainThreadComments.ts` lines 709-732
  - Ports: `contrib\remote\browser\tunnelView.ts` (ids `remote.tunnel.forward`, `remote.tunnel.open`, `remote.tunnel.openPreview`, `remote.tunnel.label`, `remote.tunnel.changeLocalPort`, `remote.tunnel.copyAddressInline`), `remoteExplorer.ts` lines 72-110, settings `remote.autoForwardPorts` and `remote.portsAttributes` in `contrib\remote\common\remote.contribution.ts` lines 227 and 256
  - Refactor Preview: `contrib\bulkEdit\browser\preview\bulkEditPane.ts` (`static ID = 'refactorPreview'`), `bulkEdit.contribution.ts` (ids `refactorPreview.apply`, `refactorPreview.discard`, `refactorPreview.toggleCheckedState`, `refactorPreview.groupByFile`, `refactorPreview.groupByType`)
- Notes: I kept Ports in scope. It has a panel tab and the scope list does not rule out remote UI. It only appears with forwarded ports (in a remote window, or after a local port is forwarded), so on a plain local window you may never see it.

## Built-in extension contributions (V\extensions)

I read every `V\extensions\*\package.json` and checked `contributes.views`, `contributes.viewsContainers`, `contributes.terminal`, `contributes.terminalQuickFixes` and `contributes.terminalProfiles`.

- No built-in extension adds a panel view container. Only `viewsContainers.activitybar` entries exist (copilot, references-view). View keys found: `copilot-chat` and `context-inspector` (copilot), `explorer` (npm), `references-view` (references-view), `remote` (vscode-api-tests). None is a panel key. Source: `V\extensions\copilot\package.json`, `V\extensions\npm\package.json`, `V\extensions\references-view\package.json`, `V\extensions\vscode-api-tests\package.json`.
- `terminal-suggest` adds the Fig-spec completion provider for the terminal suggest widget (`contributes.terminal.completionProviders`) and one command, `terminal.integrated.suggest.clearCachedGlobals`. This feeds PN-3. Source: `V\extensions\terminal-suggest\package.json`.
- `npm` adds one terminal quick fix, `ms-vscode.npm-command` ("Did you mean ..." suggestions after a mistyped npm command). This feeds PN-3. Source: `V\extensions\npm\package.json` (`contributes.terminalQuickFixes`).
- `copilot` adds a terminal profile "GitHub Copilot CLI" and four terminal quick fixes (`copilot-chat.fixWithCopilot`, `generateCommitMessage`, `terminalToDebugging`, `terminalToDebuggingSuccess`). These are agent features, so they are left out. Source: `V\extensions\copilot\package.json`.
- Not in `V\extensions` but note for the Cursor pass: the JavaScript debug extension (`ms-vscode.js-debug`) is downloaded at build time in stock VS Code, so it is not in this source tree. In Cursor it ships in the install and adds the "JavaScript Debug Terminal" terminal profile (see sibling file).

## Left out, with reasons

- Terminal chat and agent tools: everything in `contrib\terminalContrib\chat` and `contrib\terminalContrib\chatAgentTools` (out of scope per the brief). Also the chat-linked terminal files in `contrib\terminal\browser\`: `chatTerminalCommandMirror.ts`, `agentHostPty.ts`, `agentHostTerminalService.ts`, `ahpTerminalCommandSource.ts`, `terminalTabsChatEntry.ts` (hidden chat terminals row in the tabs list).
- `contrib\terminalContrib\inlineHint` (the faint hint on an empty terminal line): its two actions are `terminalInlineChat.hintAction` and `terminalCopilotCli.hintAction` in `terminal.initialHint.contribution.ts`, both chat. Left out whole.
- `contrib\markers\browser\markersChatContext.ts`: attaches problems to a chat request. Chat.
- Extension system: no part of `contrib\extensions` was read. Terminal and panel contributions made by extensions through the contribution points (for example `terminalExtensionPoints.ts`) are the contribution machinery and are out.
- Features of any view dragged into the panel from the Side Bar or Auxiliary Bar (Explorer, Search, Source Control, Chat and so on): owned by their own sections.
- `contrib\notebook\browser\contrib\debug` (breakpoints, cell pausing and debug decorations): these render in the notebook editor, not in the panel. Nothing there is panel UI.
- Debug beyond the Debug Console: the Run and Debug side bar, debug toolbar, breakpoints, call stack and variables views are Side Bar or Editor items, not panel items.
- Task pickers (Run Task list, task quick pick, `tasksQuickAccess.ts`, `taskQuickPick.ts`): quick pick UI, owned by Quick Access (QA). Task terminal status and the exit message are kept in PN-2.
- Terminal menu in the menu bar (`MenuId.MenubarTerminalMenu`, `terminalMenus.ts` lines 41-105, `task.contribution.ts`): owned by the Title Bar (TB) section. Its commands are described in PN-2.
- Terminal as an editor tab, the Terminal editor, and "Create New Terminal in Editor Area": the tab side is the Editor (ED) section.
## Newer than 1.128 (seen only in N, marked `newer-than-1.128`)

- `terminal.integrated.fontRendering` (antialiasing: inherit or grayscale), with `contrib\terminal\browser\xterm\terminalFontRendering.ts` and `.css` in N only. Seen at `N\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts` line 184.
- `terminal.integrated.windowsReparentConsoleWindow` (N only). `N\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts` line 503.
- A "Start Dictation" entry in the terminal right-click menu (`workbench.action.terminal.startVoiceContext`, N `contrib\terminal\browser\terminalMenus.ts` lines 202 and 323). V has Start Dictation only in the view toolbar and editor terminal.
- `N\src\vs\workbench\contrib\terminal\electron-browser\terminalInstallActions.ts`: an install-shell-command action variant that offers Copilot CLI. Agent adjacent, so left out.
- Panel actions in N hide Left and Right alignment inside the Agents window (`IsSessionsWindowContext`, `N\src\vs\workbench\browser\parts\panel\panelActions.ts`). The Agents window is out of scope.
- N panel layout code changes when floating panels are on (`N\src\vs\workbench\browser\parts\panel\panelPart.ts`, `isFloatingPanelsEnabled`). V already has the same experiment under another name, see the next list.
- N-only terminal files with no user-facing panel UI: `agentHostOutputChannel.ts`, `terminalProfileConfigurationTelemetry.ts`, `terminalLinkContribution.ts`.
- No new panel view containers, no new Problems, Output, Comments, Debug Console or Test Results commands in N compared with V (command ids compared across `contrib\markers`, `contrib\output`, `contrib\testing`, `contrib\comments`, `contrib\debug`).

## Experimental item already in V

- `workbench.experimental.modernUI` (default false, tag `experimental`): when on, the side bars and the bottom panel show as floating cards with rounded corners and gaps, matching the Agents window design. Declared at `V\src\vs\workbench\browser\workbench.contribution.ts` lines 810-815, key at `services\layout\browser\layoutService.ts` line 53, styles in `browser\media\floatingPanels.css`. This is a presentation switch for the whole workbench, not a panel feature, so it is only noted here for the Panel group PN-1 and for the layout owners.

## Could not verify

- Whether the Ports tab appears at all in a plain local window. The code builds the container only when `forwardedPortsFeaturesEnabled` or `forwardedPortsViewEnabled` is true (`remoteExplorer.ts` lines 82-110). I did not run VS Code to see when those turn on.
- Whether Comments appears in a stock install with no extension that provides comments. The container is built on demand from `mainThreadComments.ts`, so I expect it to be absent until an extension calls the comments API.
- Behavior and look. This pass read source only. I did not open a running VS Code, so tab order and visible labels come from the registered `order` values and `localize` strings.
- Whether the PN-3 "other folders" such as `typeAhead` and `autoReplies` have any visible UI at all. They are configuration driven (`terminal.integrated.autoReplies`, `terminal.integrated.localEchoStyle` and similar) and I did not trace their rendering.

## Recommendation and confidence

- Recommend 7 minor groups (PN-1 to PN-7) with the terminal as two groups. Confidence: medium - the grouping follows the code and the tabs, but feature-level work may show that PN-3 should split again (shell integration versus input helpers).
- Recommend keeping Debug Console and Test Results together in PN-6. Confidence: medium - they share the idea "results of running things" but have almost no shared code.
- Recommend keeping Ports in scope inside PN-7. Confidence: low - it depends on whether the project counts remote UI as in scope.
