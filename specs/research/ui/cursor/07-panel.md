# Panel (PN) - Cursor 3.27.12 findings

Section: Panel (PN). Date: 2026-10-25. Pass: 1.

- Scope of this file: what Cursor 3.27.12 (VS Code base 1.128.0) has for the Panel section, compared with stock VS Code 1.128.0. The stock findings and the group definitions are in the sibling file under `vscode\07-panel.md`. Group ids PN-1 to PN-7 are the same.
- Level: minor groups only. No full feature lists, no pieces.
- Short names: C = `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`. Bundle = `C\out\vs\workbench\workbench.desktop.main.js`. V = `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0`. N = `C:\Users\Vinnie\src\vscode` (1.144.0, only used for the newer-than-1.128 list in the sibling file).
- Evidence here is `bundle` (strings and compiled code in the main bundle) unless a line says otherwise. Character offsets written as `@N` are positions in the Bundle file as read. They can shift if the file is rebuilt, so search the quoted string instead.
- `workbench.glass.main.js` (Agents window) was not read, per the brief.

## How I searched

I searched the Bundle for: every command id that V registers for the panel, terminal, Problems, Output, Debug Console, Test Results, Comments, Ports, Refactor Preview and tasks (about 150 ids); every `registerViewContainer` call, to see which containers go to the panel (location argument `1`); `Pe.PanelTitle`, `Pe.TerminalInstanceContext`, `Pe.TerminalTabContext`, `Pe.TerminalNewDropdownContext`, `Pe.ProblemsPanelContext`, `Pe.DebugConsoleContext`, `Pe.TestPeekTitle`, `Pe.TunnelContext` and the Terminal menu; all `terminal.integrated.*` setting names; and every `createOutputChannel` call in `C\extensions\*\dist\*.js`. I also read `package.json` of all 117 folders in `C\extensions` for `contributes.views`, `viewsContainers` and `terminal*` keys. I did a separate comparison of `workbench.action.terminal.*` ids between V (138 ids from the enums) and the Bundle (149 ids).

## What the panel hosts in Cursor

Containers registered with location `1` (panel) in the Bundle. Each was found with its title, order and `hideIfEmpty`:

- Problems: id `workbench.panel.markers`, order 0. `registerViewContainer({id:xm.MARKERS_CONTAINER_ID,title:yf.MARKERS_PANEL_TITLE_PROBLEMS,...order:0...},1,...)`.
- Output: id `workbench.panel.output`, order 1. `title:De(8534,"Output")`.
- Debug Console: id `workbench.panel.repl`, order 2. `title:De(5657,"Debug Console")`.
- Terminal: id `terminal`, order 3, `isDefault:!0`. `title:De(10272,"Terminal")`.
- Test Results: id `workbench.panel.testResults`, order 3. `title:De(11123,"Test Results")`.
- Ports: order 5. `title:De(8965,"Ports")`, `hideIfEmpty:!0`.
- Comments: id `workbench.panel.comments`, order 10. Title from a constant, same shape as V.
- Refactor Preview: `title:De(4925,"Refactor Preview")`, no order (same as V).
- Orders match V (V list in the sibling file).

Not in the panel: the Cursor chat. Its container is registered with location `2` (Auxiliary Bar), `registerViewContainer({id:e,title:{original:t??dno,value:t??dno},...,rejectAddedViews:!0,...},2,{isDefault:i})` with `dno="Chat"` (@13864666) and storage id prefix `workbench.panel.composerChatViewPane`. The id strings `workbench.panel.aichat` and `workbench.panel.composerChatViewPane` have "panel" in the name only. Chat belongs to the Auxiliary Bar section and is out of scope.

No Browser panel. The Cursor Browser opens as an editor tab: `workbench.action.openBrowserEditor` calls `openEditor(...)` with the browser editor input, and the toolbar entry is added to `Pe.EditorTitle` (`group:"0_global"`, globe icon). Other browser command ids found: `workbench.action.newBrowserTab`, `workbench.action.reloadBrowserTab`, `workbench.action.focusBrowserLocationBar`. So the Browser is an Editor (ED) item.

No Cursor-added panel container was found. The only other registered containers are Sidebar (location `0`) or Auxiliary Bar (location `2`) ones, plus a generic container for extension-contributed views.

## Groups, with Cursor evidence

### PN-1 Panel frame and layout

- Tag: both
- Evidence: bundle
- Boundary: no
- Found: `workbench.action.togglePanel`, `closePanel`, `focusPanel`, `positionPanelLeft`, `positionPanelRight`, `positionPanelTop`, `positionPanelBottom`, `alignPanelLeft`, `alignPanelRight`, `alignPanelCenter`, `alignPanelJustify`, `toggleMaximizedPanel`, `previousPanelView`, `nextPanelView`, `movePanelToSecondarySideBar`, `moveSecondarySideBarToPanel`, `workbench.action.panel.position`, `workbench.action.panel.align`, `workbench.action.panel.toggleShowLabels`. Settings found: `workbench.panel.showLabels`, `workbench.panel.defaultLocation`, `workbench.panel.opensMaximized`. The Panel Position and Align Panel submenus (`Pe.PanelPositionMenu`, `Pe.PanelAlignmentMenu`) are read in the tab bar context menu code, same as V.
- Differences from V, all seen in the Bundle:
  - Toggle Panel Visibility (class `hva`, run body at @27652067, class ends at @27652346) has extra logic. It reads the context key `agentChatMaximized` (declared as `Eme=new Dt("agentChatMaximized",!1,...)`, @21417409). If the panel is visible and the chat is maximized and the editor area is hidden, the command calls `setUnifiedMaximizeState(!0,{skipHideSidebar:!0})` instead of hiding the panel. If the panel is hidden and the chat is maximized, it calls `setUnifiedMaximizeState(!1,...)` and then shows the panel. Otherwise it un-maximizes the panel first (`isPanelMaximized()&&toggleMaximizedPanel()`) and then hides it. V just hides it (`panelActions.ts` lines 66-69). The "unified maximize" layout belongs to the Agent chat sections, so this is only a hand-off point.
  - The command is disabled when focus is in an auxiliary window: `precondition:FT.toNegated()` and the same `when` on the keybinding, where `FT` is `isAuxiliaryWindowFocusedContext` (@21415389). V has no precondition. The default key is still Ctrl+J (`primary:Me.CtrlCmd|ve.KeyJ`).
  - The two panel layout icons `panel-layout-icon` and `panel-layout-icon-off` are both registered with the same codicon `Oe.panelFrame` (`EhS=Bi("panel-layout-icon",Oe.panelFrame,...),IhS=Bi("panel-layout-icon-off",Oe.panelFrame,...)`, right before the `hva` class). V uses `layoutPanel` and `layoutPanelOff` (`panelActions.ts` lines 25-28).
  - Panel part class (@30823201): constructor options are `{hasTitle:!0}` only, while V passes `{ hasTitle: true, trailingSeparator: true }` (`V\src\vs\workbench\browser\parts\panel\panelPart.ts` lines 86-92). Cursor adds `createTitleArea` and `getLayoutActionsContainer` overrides tied to `isNoTitlebarLayoutActive()` (context key `cursor.noTitlebarLayout.enabled`, "Whether the no-titlebar layout is currently active (title bar hidden)", @17405359). When that layout is on, the panel title does not host the layout actions container. This is title-bar layout work, so it belongs to the Title Bar (TB) section too.
  - Panel toggle button also sits in the Cursor title bar right toolbar (`{id:"workbench.action.togglePanel",toolbar:"right",order:...}`), owned by TB.
  - `workbench.experimental.modernUI` (V only) was not found in the Bundle. See the "In V, not found in Cursor" list.

### PN-2 Terminal sessions

- Tag: both
- Evidence: bundle
- Boundary: no
- Found ids: `terminal.toggleTerminal`, `new`, `newInActiveWorkspace`, `newWithProfile`, `split`, `splitActiveTab`, `unsplit`, `join`, `joinActiveTab`, `kill`, `killAll`, `rename`, `changeIcon`, `changeColor`, `moveToEditor`, `moveIntoNewWindow`, `moveToTerminalPanel`, `createTerminalEditor`, `focusTabs`, `focusPreviousPane`, `resizePaneLeft`, `selectDefaultShell`, `setDimensions` (Set Fixed Dimensions), `sizeToContentWidth`, `relaunch`, `switchTerminal`, `detachSession`, `attachToSession`, `runActiveFile`, `runSelectedText` (all prefixed `workbench.action.terminal.`). Settings found: `terminal.integrated.tabs.enabled`, `tabs.hideCondition`, `tabs.showActions`, `tabs.location` (default `right`), `defaultLocation` (`editor` or `view`, default `view`), `enablePersistentSessions`, `hideOnStartup`, `hideOnLastClosed`, `confirmOnKill`, `splitCwd`, `showExitAlert`, `persistentSessionReviveProcess`.
- Menus found: `Pe.TerminalInstanceContext` (Split, New, Copy, Copy as HTML, Clear, Size to Content Width seen in the menu code; the Paste and Select All ids exist but I did not see their menu entries), `Pe.TerminalTabContext` (Split, Move to Editor, Move into New Window, Rename, Change Icon, Change Color, Size to Content Width, Join, Unsplit), `Pe.TerminalNewDropdownContext` (Select Default Profile, Configure Terminal Settings, Run Task, Configure Tasks), `Pe.MenubarTerminalMenu` (New Terminal, Split Terminal, Run Active File, Run Selected Text, Run Task, Run Build Task, Show Running Tasks, Terminate Task, Configure Default Build Task).
- Tasks: `workbench.action.tasks.runTask`, `showTasks`, `terminate`, `restartTask`, `reRunTask`, `rerunForActiveTerminal`, `build`, `configureDefaultBuildTask` and settings `task.reconnection`, `task.saveBeforeRun` all found in the Bundle, so the task commands and settings exist as in V. The terminal right-click menu also holds "Rerun Task" (`Pe.TerminalInstanceContext`, key Ctrl+Shift+R), same as V's `rerunForActiveTerminal` in `V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts` lines 584-597.
- In V, not found in Cursor: `workbench.action.terminal.newInNewWindow` (menu item "New Terminal Window" in V `terminalMenus.ts` lines 56-66). The tabs-list row for hidden chat terminals (`workbench.action.terminal.chat.viewHiddenChatTerminals`, class `terminal-tabs-chat-entry`) was also not found. That row is chat anyway.
- Cursor extras that touch the terminal view toolbar and are agent related are listed under left out below.
- Built-in extension added a terminal profile: `ms-vscode.js-debug` adds the profile "JavaScript Debug Terminal" (`extension.js-debug.debugTerminal`), see the extension section.

### PN-3 Terminal tooling

- Tag: both
- Evidence: bundle
- Boundary: no
- Found ids: find (`focusFind`, `hideFind`), scrolling and command marks (`scrollToPreviousCommand`, `selectToPreviousCommand`, `scrollToBottom`), links (`openDetectedLink`, `openFileLink`, `openUrlLink`), clipboard (`copySelection`, `copyLastCommand`, `copyLastCommandOutput`), history (`runRecentCommand`, `goToRecentDirectory`, `clearPreviousSessionHistory`), quick fix (`showQuickFixes`), sticky scroll (`toggleStickyScroll`), zoom (`fontZoomIn`), accessibility (`focusAccessibleBuffer`), `sendSequence`, `sendSignal`, `showEnvironmentContributions`, developer (`showTextureAtlas`, `writeDataToTerminal`). Settings found: `terminal.integrated.shellIntegration.enabled`, `shellIntegration.decorationsEnabled`, `shellIntegration.showCommandGuide`, `stickyScroll.enabled`, `stickyScroll.maxLineCount`, `autoReplies`, `copyOnSelection`, `enableMultiLinePasteWarning`, `rightClickBehavior`, `scrollback`, `cursorStyle`, `gpuAcceleration`, `enableImages`, `environmentChangesIndicator`, `accessibleViewPreserveCursorPosition`.
- Terminal suggest (IntelliSense) is present but older than V. Found: `acceptSelectedSuggestion`, `acceptSelectedSuggestionEnter`, `hideSuggestWidget`, `hideSuggestWidgetAndNavigateHistory`, `selectNextSuggestion`, `resetSuggestWidgetSize`, `suggestToggleDetails`, `suggestToggleDetailsFocus`, and 10 settings: `terminal.integrated.suggest.enabled`, `quickSuggestions`, `suggestOnTriggerCharacters`, `runOnEnter`, `windowsExecutableExtensions`, `providers`, `showStatusBar`, `cdPath`, `inlineSuggestion`, `upArrowNavigatesHistory`.
- In V, not found in Cursor (string searched, zero hits): `terminal.integrated.suggest.selectionMode`, `terminal.integrated.suggest.insertTrailingSpace`, `workbench.action.terminal.triggerSuggest`, `workbench.action.terminal.changeSelectionMode*`, `workbench.action.terminal.doNotShowSuggestOnType`, `workbench.action.terminal.showSuggestOnType`, `workbench.action.terminal.suggestLearnMore`, and the terminal dictation commands (`workbench.action.terminal.startVoice`, `stopVoice`, and the label "Start Dictation in Terminal"). The editor dictation command `workbench.action.editorDictation.start` ("Start Dictation in Editor") does exist in the Bundle, so only the terminal side is missing.
- In Cursor, not in V: `workbench.action.terminal.requestCompletions`, listed in the same id list as the suggest commands. I did not trace what triggers it.
- Terminal quick fixes: the npm quick fix ships from `C\extensions\npm\package.json` (`terminalQuickFixes`), same as V.
- The Fig-spec completion extension `terminal-suggest` is in `C\extensions\terminal-suggest` (version 1.0.1, same as V) but its `package.json` has no `contributes` block. V's copy declares `contributes.terminal.completionProviders` and the command `terminal.integrated.suggest.clearCachedGlobals` (`V\extensions\terminal-suggest\package.json`). Neither string appears in the Bundle. So Cursor shows the suggest widget but without the V command to clear cached globals.

### PN-4 Problems

- Tag: both
- Evidence: bundle
- Boundary: yes - two Cursor-added menu entries hand the selected problem to the agent. Everything else in the group is stock.
- Found: container `workbench.panel.markers` (view `workbench.panel.markers.view`), `workbench.actions.view.problems`, `workbench.action.problems.focus`, settings `problems.autoReveal`, `problems.defaultViewMode`, `problems.showCurrentInStatus`, `problems.sortOrder`, `problems.visibility`, label "View as Tree", `Pe.ProblemsPanelContext` with Copy and Copy Message. Filter actions (Show Errors and so on) are built from a template id, so I did not find the literal strings. The labels for View as Table and the filter items are in a separate string table, not in the Bundle text.
- Cursor additions in the Problems context menu (`Pe.ProblemsPanelContext`, group `z_chat`, @36627790 and @36628702):
  - `workbench.action.problems.addToChat`, label "Add to Chat" (order 1). It opens a composer with the text "I'm getting the following error in my code: ... Can you help me understand and fix this issue?" and attaches the file as a file selection.
  - `workbench.action.problems.fixInChat`, label "Fix with Agent" (order 2).
  - Both use `when: SR.MarkerFocusContextKey`, so they show on a problem row, not on related-information rows.

### PN-5 Output and logs

- Tag: both
- Evidence: bundle
- Boundary: no
- Found: `workbench.action.output.toggleOutput`, `workbench.output.action.clearOutput`, `workbench.output.action.toggleAutoScroll`, `workbench.action.showOutputChannels`, `workbench.action.output.addCompoundLog`, `workbench.action.output.remove`, `workbench.action.openActiveLogOutputFile`, `workbench.action.openActiveLogOutputFileInNewWindow`, `workbench.action.saveActiveLogOutputAs`, `workbench.action.showLogs`, `workbench.action.exportLogs`, `workbench.action.importLog`, `workbench.action.output.activeOutputLogLevelDefault`, setting `output.smartScroll.enabled`, setting `workbench.view.showQuietly`. Labels found in text: "Add Compound Log", "Set Log Level", "Toggle Auto Scrolling".
- Cursor addition: many extra Output channels. The Output picker and the Show Logs list get entries from Cursor's own built-in extensions. Names and owners, all from `createOutputChannel(...)` calls in `C\extensions\<ext>\dist\*.js`:
  - `cursor-always-local`: "Cursor Tab", "Cursor Always Local", "Cursor Structured Logs", "Cursor-always-local debug logs", "Filesync"
  - `cursor-retrieval`: "Cursor Indexing & Retrieval", "Cursor Git Graph", "Cursor Grep Service", "Cursor Snapshot Service", "Cursor-retrieval debug logs", "Cursor Agent Review" (shared with `cursor-commits`)
  - `cursor-agent-host`: "Cursor Agent Host", "Cursor Agent Host Network", "Cursor Agent Host Storage", "Cursor Agent Host Structured Logs", "Cursor Plugins", "Mcp FileSystem Writer"
  - `cursor-agent-exec` ("Cursor Agent Exec"), `cursor-agent-worker` ("Cursor Agent Worker"), `cursor-local-agent-runtime` ("Local Agent Runtime"), `cursor-mcp` ("MCP Logs", plus a per server `MCP: <name>` channel), `cursor-computer-use` ("Cursor Computer Use"), `cursor-browser-automation` ("Cursor IDE Browser Automation"), `cursor-shadow-workspace` ("Shadow Workspace"), `cursor-resolver` and `cursor-resolver-helper`, `cursor-socket` ("Cursor Socket"), `cursor-commits` ("Cursor-commits debug logs"), `cursor-ndjson-ingest` ("NDJSON Ingest")
  - Channel content is plumbing and logs from the agent, indexing and tab features. Only the Cursor Tab channel is about an in-scope feature. I did not catalog the channels beyond their names. Boundary for these channels: no for the Output UI, they are only extra rows.
  - Channels from stock built-in extensions (not Cursor additions, extension folders exist in `V\extensions`): "Git" (git), "GitHub" (github), "Markdown" (markdown-language-features), "TypeScript" and "TypeScript Server Log" (typescript-language-features), "Microsoft Authentication" (microsoft-authentication), and Grunt, Gulp and Jake auto detection (grunt, gulp, jake). I matched these by name only.

### PN-6 Debug Console and Test Results

- Tag: both
- Evidence: bundle
- Boundary: no
- Found: Debug Console container `workbench.panel.repl` with view `workbench.panel.repl.view` and `workbench.panel.repl.view.focus`, `workbench.debug.action.toggleRepl`, `workbench.debug.panel.action.clearReplAction`, `repl.action.filter`, `repl.action.find`, `repl.action.copyAll`, `debug.collapseRepl`, `debug.replPaste`, `debug.replCopy`, `Pe.DebugConsoleContext`, settings `debug.console.collapseIdenticalLines`, `debug.console.historySuggestions`, `debug.console.wordWrap`. Test Results container `workbench.panel.testResults` with view `workbench.panel.testResults.view` (`when: hasAnyResults`), `testing.showMostRecentOutput`, `testing.clearTestResults`, `testing.openOutputPeek`, `testing.toggleInlineTestOutput`, `testing.goToNextMessage`, `testing.collapsePeekStack`, `Pe.TestPeekTitle`, settings `testing.automaticallyOpenTestResults`, `testing.automaticallyOpenPeekView`, `testing.openTesting`.
- No Cursor additions found in the Debug Console or Test Results menus. The `Pe.DebugConsoleContext` and `Pe.TestPeekTitle` registrations I saw match V's.
- Not found in the Bundle as text: the labels "Debug Console: Focus Filter" and "Show Most Recent Output" (they live in the separate string table). The ids are present, so I treat the features as present.

### PN-7 Context-only panel views

- Tag: both
- Evidence: bundle
- Boundary: no
- Found: Comments (`workbench.panel.comments`, `workbench.action.focusCommentsPanel`, `comments.collapse`, `comments.reply`, `comments.openView`, `comments.useRelativeTime`, `commentsClearFilterText`, `Pe.CommentsViewThreadActions`); Ports (container title "Ports" order 5, `remote.tunnel.forward`, `remote.tunnel.open`, `remote.tunnel.openPreview`, `remote.tunnel.label`, `remote.tunnel.changeLocalPort`, `remote.tunnel.copyAddressInline`, `Pe.TunnelContext` with "Port Visibility" and "Change Port Protocol" submenus, settings `remote.autoForwardPorts` and `remote.portsAttributes`; label "Forward a Port" found in text); Refactor Preview (title found, `refactorPreview.apply`, `refactorPreview.discard`, `refactorPreview.toggleCheckedState`, `refactorPreview.groupByFile`).
- No Cursor change to these three views found.

## Cursor additions (summary)

1. Toggle Panel Visibility behaves differently with the chat maximized, is disabled in auxiliary windows, and un-maximizes the panel before hiding it (PN-1, Bundle class `hva`).
2. Panel on and off layout icons both use the codicon `panelFrame` (PN-1).
3. Panel title does not host the layout actions container under the no-titlebar layout (PN-1, shared with TB).
4. "Add to Chat" and "Fix with Agent" in the Problems row context menu (PN-4, `boundary`).
5. About 30 extra Output channels from Cursor's built-in extensions (PN-5, names above).
6. JavaScript Debug Terminal profile from the bundled `ms-vscode.js-debug` (PN-2). Not in the V source tree because V downloads that extension at build, so I cannot say from V whether it is Cursor only. I did not check a stock release, so its tag is unverified.
7. Terminal ids found only in the Bundle: `workbench.action.terminal.requestCompletions` (PN-3, suggest id list), `workbench.action.terminal.addTooltip` (internal, shows a hint box at the top right of the terminal, used by the Cursor terminal hint contribution below) and `workbench.action.terminal.debugCheckExecutionServiceVersion` (title "Debug: Check Terminal Execution Service Version", visible in the Command Palette, dev only).

## Built-in extension contributions (C\extensions)

- No built-in extension adds a panel view container, and none declares a `views.panel` key. View keys across all 117 folders: `debug` (`ms-vscode.js-debug`), `explorer` (`npm`), `references-view` (`references-view`). Source: each `C\extensions\<name>\package.json`.
- `ms-vscode.js-debug` 1.93.0: `contributes.terminal.profiles` with one profile, id `extension.js-debug.debugTerminal`, title from `%debug.terminal.label%` ("JavaScript Debug Terminal"), icon `$(debug)`, and the command `extension.js-debug.createDebuggerTerminal`. This feeds PN-2. Source: `C\extensions\ms-vscode.js-debug\package.json`.
- `npm`: one terminal quick fix, `ms-vscode.npm-command`. Feeds PN-3. Same as V.
- `terminal-suggest`: no `contributes` block in Cursor, unlike V. See PN-3.
- No `copilot` extension folder in Cursor, so none of the V copilot terminal profile or terminal quick fixes exist here.
- The `cursor-*` extensions add Output channels only (PN-5). None of them declares panel views or terminal contributions in its `package.json`.

## Left out, with reasons

- Terminal chat and agent tool UI (out of scope per the brief). Seen in the Bundle:
  - `workbench.action.terminal.chat.*` ids (start, close, discard, makeRequest, cancel, feedback, runCommand, insertCommand and more), listed inside the skip-shell id list near @21301474.
  - The terminal Cmd+K "generate in terminal" bar: `cursorai.action.generateInTerminal`, `hideGenerateInTerminal`, `cancelGenerateInTerminal`, `acceptGenerateInTerminal`, `acceptAndRunGenerateInTerminal`, `rejectGenerateInTerminal` and the settings `cursor.terminal.usePreviewBox` and `cursor.terminal.enableAiChecks` (@484999 to @485700).
  - The terminal hint contribution `terminal.cursorDebugHint`: selecting a command block in the terminal shows an "Add to Chat" button (`_insertSelectionIntoComposer`, label "Add to Chat", keybinding label from the same command as the editor Add to Chat) (@37497061 to @37498414).
  - External agent actions on the terminal view toolbar: `workbench.action.terminal.externalAgentActions` (title "External Agent Actions"), `externalAgentAsk` ("Side Chat"), `externalAgentSecondOpinion` ("Second Opinion"), shown on the terminal view title when `hasContributionActions` is true (@37526716).
  - Agent-created terminals: `composerTerminalService`, `registerPersistentAITerminal`, `composer.openTerminalInWorktree`, `composer.cancelTerminalToolCall`, terminal selections and terminal files as chat context (`terminalSelections`, `terminalFiles`).
- The Cursor chat container (`workbench.panel.composerChatViewPane` storage prefix): lives in the Auxiliary Bar, not the panel.
- Cursor Browser: an editor, not a panel tab. Owned by Editor (ED).
- Terminal Execution Service plumbing (`debugCheckExecutionServiceVersion`): agent tool plumbing. Mentioned above only because the command is visible.
- Extension marketplace and extension host: not read.
- Task pickers and the Terminal menu in the menu bar: owned by Quick Access (QA) and Title Bar (TB) as in the stock file.

## In V, not found in Cursor

String searches in the Bundle, zero hits. Could be removed, renamed or never merged. I did not check which.

- `workbench.experimental.modernUI` and `workbench.experimental.floatingPanels` (PN-1 notes in the stock file).
- `workbench.action.terminal.newInNewWindow` (New Terminal Window menu item).
- Terminal dictation: `workbench.action.terminal.startVoice`, `stopVoice`, "Start Dictation in Terminal".
- Terminal suggest: `terminal.integrated.suggest.selectionMode`, `insertTrailingSpace`, `workbench.action.terminal.triggerSuggest`, `changeSelectionMode*`, `doNotShowSuggestOnType`, `showSuggestOnType`, `suggestLearnMore`.
- `terminal.integrated.suggest.clearCachedGlobals` command and the `terminal.completionProviders` declaration (the extension contribution).
- Others from the V enum list with no hit: `workbench.action.terminal.focusInstance`, `openWordLink`, `pastePwsh`, `profile.choice`, `quickKill`, `resetDiscoverability`, `revealCommand`, `sizeToContentWidthActiveTab`. Several of these may be internal ids only, so treat as unconfirmed.

## Newer than 1.128

Nothing new from N applies to Cursor, because Cursor is on 1.128.0. The newer-than-1.128 list (terminal font rendering, Windows console reparenting, terminal context-menu dictation, install-shell action variant) is in the sibling file. I searched the Bundle for `terminal.integrated.fontRendering`, `fontRendering`, `terminal.integrated.windowsReparentConsoleWindow` and `workbench.action.terminal.startVoiceContext`: zero hits for all four, as expected.

## Could not verify

- Run-time behavior. I read the Bundle text and extension files only. I did not open Cursor to see which tabs, channels or toolbar entries actually show up on a fresh window.
- Whether each Cursor Output channel is visible by default. Channels created with `{log:true}` appear in the log list, but I did not check which show in the Output picker or whether the owning extension activates on startup.
- Exact differences in the compiled Panel part beyond the lines quoted above. The Bundle is minified and I did not diff whole classes, so other small differences from V's `panelPart.ts` may exist.
- Why the Cursor terminal suggest code looks older than V (fewer settings and ids, no `contributes` on the extension). I only report what is missing or extra.
- What triggers `workbench.action.terminal.requestCompletions` and `workbench.action.terminal.addTooltip`.
- Strings that exist only in the separate NLS table (many labels). I confirmed those features by command id instead of label.

## Recommendation and confidence

- Keep the same 7 groups for Cursor. No Cursor-only group is needed, because Cursor's panel additions fall inside PN-1 (layout hand-off with the chat), PN-4 (two agent entries) and PN-5 (extra channels). Confidence: high - the Bundle shows the same containers, the same orders and the same command ids as V, with no new panel container.
- Treat the Problems "Add to Chat" and "Fix with Agent" entries as the only boundary items in this section. Confidence: medium - I found no other agent entry points in the panel menus I searched, but I did not search every menu id.