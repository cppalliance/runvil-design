# Status bar (SS) - Cursor 3.27.12 findings

Section: Status bar (SS). Date: 2026-10-25. Pass: 1.

- Scope of this file: what Cursor 3.27.12 (VS Code base 1.128.0) has for the Status bar section, compared with stock VS Code 1.128.0. The stock findings and the group definitions are in the sibling file under `vscode\09-status-bar.md`. Group ids SS-1 to SS-6 are the same.
- Level: minor groups only. No full feature lists, no pieces.
- Short names: C = `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`. Bundle = `C\out\vs\workbench\workbench.desktop.main.js`. NLS = `C\out\nls.keys.json` and `C\out\nls.messages.json` (the Bundle stores labels as numbered strings, for example `k(3774,null)`; the two NLS files map each number to a module, key and English text). V = `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0`.
- Evidence here is `bundle` (strings and compiled code in the Bundle, the NLS files, the extension folders, the CSS) unless a line says otherwise. Where I saw something on the running install it says "seen running".
- `workbench.glass.main.js` (Agents window) was not read, per the brief. The Automations bundle and the xai-commerce-prepaid bundle were searched for status bar calls and have none.

## How I searched

- Every status bar entry goes through one call, so I listed every `.addEntry(` call in the Bundle (about 48 hits, including a few that are unrelated working-copy history calls). That covers every item added from core code, including items an extension adds through the extension API (they go through `MainThreadStatusBar`, which calls `addEntry`).
- For each entry I read the surrounding compiled code and, where the label is a number, looked the number up in the NLS files. I also built a number-to-text table of all 13,625 NLS messages and filtered it for status bar modules.
- Compared the Bundle module list in `nls.keys.json` against the V files that call `addEntry` (about 30) to find items V has and Cursor lacks.
- Searched every `package.json` and every built-in extension bundle in `C\extensions` (117 folders) for `createStatusBarItem`, `setStatusBarMessage` and `statusBarItems`.
- Searched the CSS (`workbench.desktop.main.css`) for every `statusbar` selector and the Cursor theme files in `C\extensions\theme-cursor\themes` for `statusBar` colors.
- Searched the Bundle for all setting, command and color names that contain `statusBar`.
- Took one screenshot of the running Cursor 3.27.12 window on 2026-10-25 and hovered one item, to check which entries show by default and what the unlabeled one is. This is spot evidence only: the window is a real working window, so hidden-item state and open editors are the owner's.

## What the Cursor status bar is, in one paragraph

The status bar part is the stock VS Code 1.128 part with small changes: the same two item containers, the same priority sort, the same right-click hide menu, the same hide state key `workbench.statusbar.hidden`, the same focus keys, the same 22 px height. Cursor adds its own items on both sides (workspace name on the left; Cursor Tab and three AI stats items on the right), drops a few stock items, and the stock items it keeps look like an older VS Code snapshot in several places (module names still use `electron-sandbox`, V uses `electron-browser`; there is no floating-panels padding; the notifications position setting is missing).

Seen running (Cursor 3.27.12 window, 2026-10-25), left to right. Left: remote indicator icon, repository name with a repo icon, branch name with a star and a sync icon, workspace name "cursor", problems "0 0". Right: "Cursor Tab", a zoom icon, "Tab Stats: 0/0 (0%)", "Agent Stats: 34413/897 (3836%)", a commit item "b90411c 100.00% AI", then the notifications bell. No editor items (selection, encoding, language) were visible because no text editor had focus at that moment. I do not claim anything about their default visibility from this.

## Groups, with Cursor evidence

Same six groups as the sibling file. Confidence in the six-way split: medium (same reason as there). Tags below are for the group in Cursor.

### SS-1 Bar behavior and per-item control

- Tag: both
- Evidence: bundle
- Boundary: no
- Found, same as V: setting `workbench.statusBar.visible` (many hits, also used by `config.workbench.statusBar.visible` toggled checks); command `workbench.action.toggleStatusbarVisibility` with NLS "Toggle Status Bar Visibility", the View menu mnemonic "S&&tatus Bar" and the Customize Layout label "Status Bar"; "Focus Status Bar" (`workbench.action.focusStatusBar`); the five key commands `workbench.statusBar.focusPrevious`, `focusNext`, `focusFirst`, `focusLast`, `clearFocus`; context key `statusBarFocused`; storage key `workbench.statusbar.hidden`; the special command `statusBar.entry.showTooltip`; auxiliary window status bar part (`workbench.parts.auxiliaryStatus`). The right-click menu code (`getContextMenuActions`) is the same as V: "Hide Status Bar", one checkable row per item, "Manage Extension", "Hide '{0}'".
- Differences from V:
  - The item class differs. Cursor's entries accept three extra fields: `cssClass` (extra class on the label), `commandExecutionTrigger` (`click` by default, or `mousedown`, which stops the event and only reacts to the left button) and `dontHideHoverOnClick` (hover stays open after a click). V has none of the three.
  - Cursor's item has no `content` field (V lets an entry attach its own HTML element) and no `workbenchActionExecuted` telemetry call when an item is clicked (V sends `from: 'status bar'`).
  - The status bar service has an extra method `getViewModel()`. Cursor Tab uses it to read the Vim extension's status entry (id `vscodevim.vim.primary`) and decide whether Vim is in insert mode. That is a behavior hook, not a visible item.
  - The notifications bell is always on the right, last (`addEntry(...,"status.notifications",RIGHT,-Infinity)`). The setting `workbench.notifications.position` is not in the Bundle (0 hits), so V's options to put the bell on the left (bottom-left position) or move it out of the bar (top-right) do not exist.
  - No floating-panels layout padding for the bar (`isFloatingPanelsEnabled` has 0 hits; `HEIGHT=22` is present).
  - CSS: a Cursor rule makes the bar fixed to the bottom at 22 px (`body.no-titlebar-layout .monaco-workbench .split-view-view.monaco-statusbar-split-view` with `position:fixed` and `bottom:0`, and `.part.statusbar{height:22px!important}`). This is for the no-title-bar layout. Also `.statusbar-item .codicon.codicon-repo{padding-top:1px}`. Rules for `.monaco-workbench.nostatusbar` move the notification center and toasts down when the bar is hidden (I did not compare these with V).
  - The Bundle has a key named `glassChatStatusBarVisible`. By its name it belongs to the Agents window, which I did not read.

### SS-2 Bar and item colors

- Tag: both
- Evidence: bundle
- Boundary: no
- Found, same as V: the `statusBar.*` and `statusBarItem.*` color tokens (foreground, background, border, focusBorder, noFolder trio, debugging trio, item active, hover, compact hover, focus border, and the prominent, warning, error, remote and offline sets with hover variants); setting `debug.enableStatusBarColor` (default true) in the debug configuration; CSS classes `prominent-kind`, `warning-kind`, `error-kind`, `remote-kind`, `offline-kind`, `has-background-color`, `compact-left`, `compact-right`, `has-beak`.
- Not found: `statusBar.inactiveBackground` (newer than 1.128, only in N).
- Cursor's own default colors, from the shipped theme `C\extensions\theme-cursor\themes\cursor-dark-color-theme.json` and `cursor-light-color-theme.json` (the other Cursor theme files were not read for status colors):
  - Dark: background `#141414`, foreground `#F0F0F099`, border `#F0F0F013`. No folder background `#141414` (same as normal) and foreground `#F0F0F0`. Debugging background `#F0F0F01C`, foreground `#F0F0F0`. Remote background `#141414`, foreground `#F0F0F0BD`. Item hover `#F0F0F011`, active `#F0F0F01E`, prominent background `#F0F0F011`, prominent hover `#F0F0F01E`.
  - Light: background `#F3F3F3`, foreground `#14141499`, border `#14141414`. No folder and debugging backgrounds are both `#F3F3F3` (same as normal), debugging foreground `#14141499`. Remote background `#F3F3F3`, foreground `#141414BD`. Hover `#14141414`, active `#14141424`, prominent `#14141414` and hover `#14141424`.
  - So in Cursor's own themes the no-folder bar and the remote indicator look the same as the normal bar (the stock V token defaults are purple `#68217A` for no folder and the activity bar badge color for the remote indicator), and debugging is a faint tint in dark and no change in light.
- Seen running: the bar is dark gray with muted text and no colored remote badge.

### SS-3 Current editor and language details

- Tag: both
- Evidence: bundle
- Boundary: no
- Found, same ids and labels as V (NLS module `browser/parts/editor/editorStatus`, ids and commands confirmed in the Bundle): `status.editor.selection` ("Ln {0}, Col {1}", `workbench.action.gotoLine`), `status.editor.indentation` ("Spaces: {0}", "Tab Size: {0}"), `status.editor.encoding` (`workbench.action.editor.changeEncoding`), `status.editor.eol` (`changeEOL`, LF and CRLF), `status.editor.mode` (`changeLanguageMode`), `status.editor.info` ("File Information"), `status.editor.tabFocusMode` (name "Accessibility Mode", text "Tab Moves Focus", 100.7), `status.editor.inputMode` ("OVR", 100.6), `status.editor.columnSelectionMode` ("Column Selection", 100.8), `statusbar.currentProblem` with setting `problems.showCurrentInStatus`, Screen Reader Mode (`status.editor.screenReaderMode`, "Screen Reader Optimized", command `showEditorScreenReaderNotification`), Scrolling Locked (`status.scrollLockingEnabled`, 102), notebook items (`notebook.activeCellStatus` "Notebook Editor Selections", `notebook.status.indentation` "Notebook Indentation", and the kernel entries "Notebook Kernel Info" and "Notebook Kernel Selection" with text "Select Kernel"), language status (`status.languageStatus`, "Editor Language Status", "Add to Status Bar", "Remove from Status Bar"), language detection (`status.languageDetectionStatus`, setting `workbench.editor.languageDetection`, `workbench.editor.preferHistoryBasedLanguageDetection`).
- Differences from V: none found in ids, labels or priorities. I did not compare pixel or tooltip detail.
- Not verified at runtime: the default visibility of these items in Cursor on a fresh profile.

### SS-4 Workspace, remote and source control

- Tag: both
- Evidence: bundle
- Boundary: no
- Found, same as V: remote indicator (`status.host`, NLS module `contrib/remote/browser/remoteIndicator`, command `workbench.action.remote.showMenu`, menu `StatusBarRemoteIndicatorMenu` and menu contribution point `statusBar/remoteIndicator`; labels "Open a Remote Window", "Opening Remote...", "Reconnecting to {0}...", "Disconnected from {0}", "Editing on {0}"); Restricted Mode entry (label "Restricted Mode", id `status.workspaceTrust` with the trust manage command; NLS module is `contrib/workspace/browser/workspace.view.contribution` here and `workspace.contribution` in V); Forwarded Ports (`status.forwardedPorts`, priority 40); SCM entries (`status.scm.${index}` chain and `status.scm.provider`, command `scm.setActiveProvider`); git extension items (see "Built-in extension contributions").
- Cursor addition in this group: Workspace Name, `status.workspaceName`, left, priority 50. See "Cursor additions".
- Missing from Cursor: Metered Connection (see "V items missing from Cursor").
- Seen running: remote icon, repo name with a repo icon (the repository selector `status.scm.provider`, shown because the window has several repositories), branch with star, sync icon, workspace name.

### SS-5 Alerts and background activity

- Tag: both
- Evidence: bundle
- Boundary: no
- Found, same as V: notifications bell (`status.notifications`, "Notifications", "Do Not Disturb", "Do Not Disturb Mode is Enabled", right side, last) and status message (`status.message`, "Status Message"); Problems (`status.problems`, 50, command `workbench.actions.view.toggleProblems`) and Problems visibility (`status.problemsVisibility`); Debug (`status.debug`, 30, command `workbench.action.debug.selectandstart`, settings `debug.showInStatusBar` and `debug.enableStatusBarColor`); Running Tasks (`status.runningTasks`, command `workbench.action.tasks.showTasks`); Progress Message (`status.progress`); Window Zoom (`status.windowZoom`, module `electron-sandbox/window`; seen running as a zoom icon); Output show quietly (`status.view.showQuietly`, command `workbench.action.output.show.{id}`); Extension Profiler (`status.profiler`, command `workbench.action.extensionHostProfiler.stop`, text "Profiling {0}" and "Profiling {0} ({1} sec)", which is the older wording; V says "Profiling Extension Host"); Pty Host (`ptyHostStatus`, kind warning, "Pty Host Status"); terminal session recording (`recordSession`, "Recording terminal session...").
- Cursor additions in this group: a high-memory warning entry (`workbench.memoryMonitor.indicator`, left) and two internal-only items, `status.currentServer` and `status.sessionRecording` ("[Internal] Report"). See "Cursor additions".
- Missing from Cursor: Performance Trace recording entry (see "V items missing from Cursor").

### SS-6 AI suggestions and AI stats

- Tag: both (the group exists in both products with different contents)
- Evidence: bundle
- Boundary: yes - the numbers and states shown come from the suggestion model and the agent. Only the bar items and their hovers are in scope.
- Cursor Tab status item: cross-reference only. Name "Cursor Tab", right side, priority 1000, text "Cursor Tab", "Cursor Tab (Snoozed)" or "Cursor Tab (Ignored)", with a hover panel. All its controls are cataloged under section 06 Autocomplete, group AC-8, and are not repeated here. I only re-confirmed the registration: `addEntry(this.statusBarElementProps(),"Cursor Tab",RIGHT,1000)`.
- Cursor stats items (three right-side entries, details under "Cursor additions"): Tab Stats, Agent Stats, and the recent commit AI score. These replace nothing in V; V's own stats gauge is a different feature (see missing list).
- Inline suggestions language status entry: present in Cursor (NLS module `contrib/inlineCompletions/browser/inlineCompletionLanguageStatusBarContribution`, labels "Loading...", "Inline completion available", "Inline edit available", "No inline suggestion available", "Inline suggestions"). The string `inlineSuggestionStatusBarItemLabel` also appears in the Bundle and I did not examine it. I did not check whether this entry is visible in Cursor at runtime.
- Missing from Cursor: Copilot Status entry (`chat.statusBarEntry`), the AI Usage Statistics gauge (`aiStatsStatusBar`, `editor.aiStats.enabled`) and the Edit Source Tracking developer item. All three have 0 hits for id and label.

## Cursor additions

Status bar entries that exist in the Cursor Bundle and not in V 1.128.0 or N 1.144.0 (I searched both source trees for each id and label and found none).

User-visible:

- Workspace Name (group SS-4). Entry `status.workspaceName`, name "Workspace Name", left side, priority 50. Text is the workspace name. Tooltip "Current workspace: {name}", aria label "Workspace: {name}". No click command. Module `contrib/workspace/browser/workspace.view.contribution` (NLS keys `status.workspaceName`, `status.workspaceNameAria`, `status.workspaceNameTooltip`). The name logic: if the first workspace folder has a source control repository, use the folder name when the repository root is that folder, otherwise the repository root's folder name; if there is no repository, use the workspace label. It refreshes when the workspace name or folders change and when a repository is added or removed. Seen running: text "cursor", tooltip "Current workspace: cursor".
- Cursor Tab (group SS-6). Right side, priority 1000, text "Cursor Tab", "Cursor Tab (Snoozed)" or "Cursor Tab (Ignored)", hover panel. Cross-reference only: its controls are cataloged under section 06, group AC-8, and are not repeated here. Seen running as "Cursor Tab".
- Tab Stats (group SS-6). Entry `aiCodeTracking.stats.tab`, name "AI Code Tracking Stats - Tab", right side, priority 100. Text `$(tab) Tab Stats: {accepted}/{suggested} ({percent}%)`, tooltip "Tab AI Stats (Today): {accepted}/{suggested} lines ({percent}%)". No click command found. Seen running as "Tab Stats: 0/0 (0%)".
- Agent Stats (group SS-6). Entry `aiCodeTracking.stats.composer`, name "AI Code Tracking Stats - Agent", right side, priority 99. Text `$(comment-discussion) Agent Stats: {accepted}/{suggested} ({percent}%)`, same tooltip shape. Seen running. Boundary: the numbers come from agent output.
- Recent commit AI score (group SS-6). Entry `aiCodeTracking.stats.recentCommit`, name "AI Code Tracking - Recent Commit", right side, priority 98. Text `$(git-commit) No commit scored` until a commit is scored, then the short hash and an AI percentage (seen running as "b90411c 100.00% AI"); the tooltip includes a "Repo: {name}" line. Boundary: yes, same reason.
  - These three items come from one class (module `contrib/analytics/browser/aiCodeTrackingStatusBar`) that creates them at startup unless the window is in local mode or running the smoke-test driver. I found no setting that turns them off; hiding is through the status bar right-click menu. Daily numbers are stored under storage key `aiCodeTracking.dailyStats.v1.5.{date}`. A Command Palette command "Reset AI Code Tracking Stats" (`aiCodeTracking.resetDailyStats`, category "AI Code Tracking") zeroes the day's numbers.
- High memory warning (group SS-5). Entry `workbench.memoryMonitor.indicator`, left side, default priority. Name "Memory WARNING Indicator" or "Memory CRITICAL Indicator". Text is a row of siren and warning emoji around "HIGH MEMORY USAGE". Tooltip "{WARNING or CRITICAL} Process Memory Usage (JS: {n}MB) exceeds threshold ({m}MB). Consider taking a heap snapshot." It only appears when a config flag `show_status_entry` is on and the JS heap passes a threshold (I did not trace where that flag comes from). Module `contrib/memoryMonitor/electron-sandbox/memoryMonitor.contribution`. The same monitor also raises a warning notification with a "Capture Snapshot" button (notification side belongs to section 10). Not seen running.

Internal only (not visible to normal users, but real status bar entries):

- Current server (group SS-5). Entry `status.currentServer`, right side, priority 200. Text "Server: {credentials name}", tooltip "Click to switch backend server", command `cursor.selectBackend`, prominent foreground color. The code returns early unless the user or build is a developer build (`isDevUserOrDevBuild`).
- Session report (group SS-5). Entry `status.sessionRecording`, right side, priority 50. Text "$(bug) [Internal] Report", tooltip "Report a bug or lag with automatic session context". Shown only for internal users with feature gate `internal_session_recording_status_bar`. It runs its command on mouse down, using the `commandExecutionTrigger: "mousedown"` field described under SS-1.

Not status bar entries, but checked and worth knowing:

- Cursor Blame and File Blame (in scope for the editor section) add no status bar entry. `cursor.blame` and `cursor.fileblame` strings exist in the Bundle, none is passed to `addEntry`. The only blame item on the bar is the git extension's `git.blame` item.
- Indexing status, account, upgrade, update, git worktree, browser: I looked for an entry for each and found none. No `addEntry` call, no `createStatusBarItem` in any of the 117 extension folders besides the ones listed below, and none in the Automations or xai-commerce-prepaid bundles. The title bar Update control, account and Settings entries belong to section 01. The Browser opens as an editor.

## Built-in extension contributions (C\extensions)

- git: present and the same as V. Branch and sync come from the extension's status bar commands (strings "Checkout Branch/Tag...", "Synchronize Changes...", "Publish Branch", "Publish to..." in `extensions\git\dist\main.js`) and are shown by the core SCM contribution. The blame item `window.createStatusBarItem("git.blame", Right, 200)` is in `extensions\git\dist\main.js` (name "Git Blame Information", command `git.viewCommit`). Settings `git.enableStatusBarSync`, `git.blame.statusBarItem.enabled`, `git.blame.statusBarItem.template` are in `extensions\git\package.json`.
- markdown-language-features: no status bar item. The "Rendered Diff" warning item that V has (`markdown.renderedDiffWarning`) is not in the Cursor copy (0 hits for the id and label). The language status item for Markdown (`createLanguageStatusItem`) is present.
- merge-conflict, simple-browser, search-result, references-view: none, same as V.
- Seen outside the brief's list: `media-preview` (three entries: image zoom, image size, binary size, through `createStatusBarItem` in `dist\extension.js`), `typescript-language-features`, `json-language-features`, `debug-auto-launch` (`status.debug.autoAttach`, left, "Debug Auto Attach") and `ms-vscode.js-debug` (a right-side item at priority 500 shown while profiling, command `extension.js-debug.stopProfile`). These are language, media or debug support, listed as seen only. Cursor does not ship the `copilot` or `vscode-test-resolver` extensions that V has.
- None of the `cursor-*` extension folders (including `cursor-explorer`, `cursor-commits`, `cursor-checkout`, `cursor-worktree-textmate`) call `createStatusBarItem`, `setStatusBarMessage` or declare `statusBarItems`.

## V items missing from Cursor

Each was searched in the Bundle by entry id and by English label (through the NLS files). A "0 hits" means none for either.

- Performance Trace recording entry (`status.tracing`, name "Performance Trace", text "Recording trace (click to stop)", kind error): 0 hits. The module `electron-sandbox/actions/developerActions` exists in Cursor but has no tracing strings.
- Metered Connection entry (`$(radio-tower)`, command `workbench.action.configureMeteredConnection`): 0 hits; the module `meteredConnection` is not in the module list.
- Keyboard Layout entry (`status.workbench.keyboardLayout`, "Layout: {0}", "Select Keyboard Layout"): 0 hits; module `keyboardLayoutPicker` is not in the module list.
- AI Usage Statistics gauge (`aiStatsStatusBar`, setting `editor.aiStats.enabled`) and Edit Source Tracking developer item (`editTelemetry`): 0 hits, modules absent.
- Copilot Status entry (`chat.statusBarEntry`), voice keyword activation (`status.voiceKeywordActivation`), Agent Host Profiler (`status.agentHostProfiler`): 0 hits each. Chat and agent items, left out of the catalog anyway.
- Markdown "Rendered Diff" warning item: absent (see above).
- Notification position setting `workbench.notifications.position` and the status bar behavior that goes with it: absent.
- Floating-panels bottom padding for the bar (`isFloatingPanelsEnabled`): absent.
- Entry fields `content` and the click telemetry call in the status bar item: absent.
- Wording differences, not missing items: the extension profiler entry says "Profiling {0}" where V says "Profiling Extension Host".

## Left out and why

- Extension system machinery: the extension host status bar bridge (`MainThreadStatusBar`, `ExtHostStatusBar`, `IExtensionStatusBarItemService`, the `contribStatusBarItems` extension point) is plumbing. Only visible results count here.
- Agent and chat items: anything in `workbench.glass.main.js` (Agents window, not read), `glassChatStatusBarVisible`, Copilot, voice and agent host entries.
- Cell status bar in notebooks (`notebook.showCellStatusBar`, `cell-statusbar-container`): not the window status bar; it is part of the notebook editor.
- Editor hover and suggest widget footers (`contentHoverStatusBar`, `editor.suggest.showStatusBar`): widget footers, covered by sections 05 and 06.
- Internal-only entries (`status.currentServer`, `status.sessionRecording`) are recorded under additions as real entries but should not be treated as user features.

## Newer than 1.128

Nothing in Cursor is newer than 1.128.0 in the status bar. The N-only items from the sibling file are all absent from the Bundle: `statusBar.inactiveBackground` (0 hits), `statusBar.entry.toggleTooltip` (0 hits), and the core media preview entries (Cursor still ships the `media-preview` extension). Cursor looks older than V in places (module paths with `electron-sandbox`, missing notifications position, missing floating-panels padding).

## Unverified

- Default visibility of editor items (selection, encoding, EOL, language) and of the Cursor AI stats items on a fresh Cursor profile. The screenshot is of the owner's working window and hidden-item state was not inspected.
- The condition and remote config that decides when the high memory entry appears (`show_status_entry`), and whether `status.sessionRecording` or `status.currentServer` ever show for a normal user. Read in code, not seen running.
- The command id behind the session report item (it is a minified constant in the Bundle).
- Whether the Inline Suggestions language status entry is visible in Cursor at runtime.
- `workbench.statusBar.feedback.visible` and a feedback item: not found in the Bundle, V or N. I treat it as not existing at these versions.
- The status color values for the five Cursor theme files other than `cursor-dark` and `cursor-light` (colorblind, high contrast, midnight).
- Whether Cursor changed the hover tooltip content of stock items such as Window Zoom or the remote indicator. I compared ids, labels and priorities only.


