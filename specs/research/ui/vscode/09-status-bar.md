# Status bar (SS) - VS Code 1.128.0 findings

Section: Status bar (SS). Date: 2026-10-25. Pass: 1.

- Scope of this file: stock VS Code 1.128.0 only (root V). Cursor findings are in the sibling file under `cursor\`. Group ids SS-1 to SS-6 are the same in both files.
- Level: minor groups only. No full feature lists, no pieces. Up to 4 example items per group.
- Short names: V = `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0`. N = `C:\Users\Vinnie\src\vscode` (1.144.0, used only to spot newer things). C = the Cursor install `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`. All proof paths below start under V.
- Tags say `both` when I also searched the Cursor bundle for a command id, entry id or label of that item. The Cursor evidence itself is in the sibling file.
- Evidence is `source` for everything in this file (I read it in V).

## How I searched

- Read the four status bar part files in full: `src\vs\workbench\browser\parts\statusbar\statusbarPart.ts`, `statusbarModel.ts`, `statusbarItem.ts`, `statusbarActions.ts`, plus `media\statusbarpart.css` selectors and the service contract `src\vs\workbench\services\statusbar\browser\statusbar.ts`.
- Found every caller: searched `src` for `IStatusbarService`, `.addEntry(` and `StatusbarAlignment` (about 40 files outside tests), and `extensions` for `createStatusBarItem`, `createLanguageStatusItem` and `statusBarItems`. Read the entry definition (name, text, command, kind, priority) in each file.
- Read the layout side: `src\vs\workbench\browser\actions\layoutActions.ts` (toggle action and Customize Layout item), `src\vs\workbench\browser\workbench.contribution.ts` (setting), `src\vs\workbench\browser\layout.ts` (hidden state), `src\vs\workbench\common\theme.ts` (color tokens).
- Compared status bar files and the entry list with N to find newer changes (listed at the end).
- Brief rule applied: items that only exist for chat or agents are listed under "Left out".

## What this section is

The thin bar along the bottom of the window. It is 22 px high (`StatusbarPart.HEIGHT = 22` in `statusbarPart.ts`) and has two item containers: left items and right items. Every item is a small label (an `a` element with role button) that can show text and codicons, optionally a spinner, can have a hover tooltip, and can run a command on click.

The part itself is generic. Almost all of the user-visible content is entries that other features register through one service call: `IStatusbarService.addEntry(entry, id, alignment, priority)`. The service contract is in `src\vs\workbench\services\statusbar\browser\statusbar.ts`.

How the mechanics work in V (this is what SS-1 and SS-2 cover):

- Alignment and order: each entry has a side (`StatusbarAlignment.LEFT` or `RIGHT`) and a priority. Higher priority moves toward the left inside its side. A priority can be a number or a reference to another entry (`{ location: { id, priority }, alignment, compact }`), which keeps the two entries next to each other. `compact: true` pulls them together and shares one hover highlight (`statusbarPart.ts`, `updateCompactEntries`). Ties are broken by a hash of the id (`secondary: hash(id)`). Sorting code is in `statusbarModel.ts` (`sort`).
- Entry fields (`IStatusbarEntry`): `name`, `text` (with `$(icon)` syntax), `ariaLabel`, `role`, `tooltip` (string, markdown, HTML element, or content plus footer commands), `command`, `kind`, `showProgress` (true, `loading` or `syncing`), `showBeak`, `showInAllWindows`, `extensionId`, `content`, and the deprecated `color` and `backgroundColor`.
- Kinds: `standard`, `warning`, `error`, `prominent`, `remote`, `offline` (`StatusbarEntryKinds` in `statusbar.ts`). Each kind adds a CSS class `<kind>-kind` and has its own theme colors.
- Hide or show: the whole bar by setting `workbench.statusBar.visible` (default true, `workbench.contribution.ts`) and command `workbench.action.toggleStatusbarVisibility` ("Toggle Status Bar Visibility", `layoutActions.ts`). Single items by the right-click menu on the bar (`getContextMenuActions` in `statusbarPart.ts`): "Hide Status Bar", then one checkable row per known entry name, then for the item under the mouse "Manage Extension" (only when the entry has an `extensionId`) and "Hide '{name}'". Hidden ids are stored per profile under storage key `workbench.statusbar.hidden` (`statusbarModel.ts`).
- Keyboard: command "Focus Status Bar" (`workbench.action.focusStatusBar`, Command Palette only, no default key) and, while the bar has focus (context key `statusBarFocused`), Left or Up (`workbench.statusBar.focusPrevious`), Right or Down (`focusNext`), Home (`focusFirst`), End (`focusLast`), Escape (`clearFocus`). All in `statusbarActions.ts`.
- Hover: instant hover; rich content (HTML or long-running markdown) gets a 500 ms delay override, max height 90 percent, hides on key down, sticky when opened from the keyboard (`statusbarPart.ts`, constructor). A command value of `statusBar.entry.showTooltip` ("Show Hover") opens the tooltip on click. Inside an item, Escape, Left or Right hides the hover (`statusbarItem.ts`).
- Colors: the bar picks `statusBar.background` when a folder or workspace is open and `statusBar.noFolderBackground` when empty, with matching foreground and border. A style override (used by debugging) can replace all three. Details in SS-2.
- Other windows: auxiliary windows get their own status bar (`AuxiliaryStatusbarPart`), honoring `workbench.statusBar.visible` (`auxiliaryEditorPart.ts`). Entries opt in with `showInAllWindows`.
- Menu and layout entry points: View > Appearance > Status Bar (`MenubarAppearanceMenu`, mnemonic title "S&&tatus Bar"), a "Status Bar" toggle in Customize Layout (`CreateToggleLayoutItem` in `layoutActions.ts`), and the reset action that resets `workbench.statusBar.visible`.

Not in V 1.128.0: a setting named `workbench.statusBar.feedback.visible` and a feedback entry. I searched V, N and the Cursor bundle and found neither (see "Unverified").

## Proposed minor groups

Count: 6. Reason: the bar is one frame plus a set of independent items. A person thinks of it as (1) the bar itself, (2) its colors, (3) the editor and language info on the right, (4) where I am working and the Git state on the left, (5) things asking for attention or running in the background, and (6) the AI items. The brief says aim for 3 to 6. SS-3 is the heaviest group and is the first candidate to split later (editor details versus language status). Confidence: medium - the split between SS-3 and SS-5 follows the side of the bar and the purpose of the item, and a different reviewer could place Window Zoom or the Output item differently.

### SS-1 Bar behavior and per-item control

- Plain name: Bar behavior and per-item control
- One sentence: How the bar is shown or hidden, how items are ordered on the left and right, how you hide a single item, move across items with the keyboard, and what hovering or clicking an item does.
- Tag: both
- Evidence: source
- Boundary: no
- Example items:
  - Show or hide the bar: `workbench.action.toggleStatusbarVisibility`, setting `workbench.statusBar.visible`, View > Appearance > Status Bar, right-click "Hide Status Bar"
  - Per-item menu: right-click the bar, checkable row per item, "Hide '{name}'", "Manage Extension"
  - Keyboard: `workbench.action.focusStatusBar` then arrows, Home, End, Escape (`workbench.statusBar.focusNext` and friends)
  - Item hover and click: tooltip with footer commands, "Show Hover" (`statusBar.entry.showTooltip`), spinner (`showProgress`), compact pairs
- Proof paths:
  - `src\vs\workbench\browser\parts\statusbar\statusbarPart.ts` (add and remove entries, context menu, compact entries, hover delegate)
  - `src\vs\workbench\browser\parts\statusbar\statusbarModel.ts` (priority sort, hidden ids, focus order, first and last visible item marking)
  - `src\vs\workbench\browser\parts\statusbar\statusbarItem.ts` (label, hover, click, keyboard, progress codicon)
  - `src\vs\workbench\browser\parts\statusbar\statusbarActions.ts` (hide actions, key bindings, Focus Status Bar)
  - `src\vs\workbench\services\statusbar\browser\statusbar.ts` (entry contract, `ShowTooltipCommand`)
  - `src\vs\workbench\browser\actions\layoutActions.ts` (`ToggleStatusbarVisibilityAction`, Customize Layout item)
  - `src\vs\workbench\browser\workbench.contribution.ts` (setting `workbench.statusBar.visible`)
- Other things that belong here: the item order table below, the Notification beak (`showBeak`), the extension API route (`window.createStatusBarItem`, `window.setStatusBarMessage`) as seen by the user, auxiliary window status bars.

Order on the bar in V (higher priority is further left), for later passes:

- Left: remote host (`status.host`, infinity), Restricted Mode (placed to the right of the host), Source Control items (`status.scm.0`, `status.scm.1`, `status.scm.provider`, priority 10000 chain), Problems (50), Problems visibility (49), Running Tasks (right of Problems), Forwarded Ports (40), Debug (30), then low-priority items such as Status Message, Progress Message, Performance Trace.
- Right: Window Zoom and Scrolling Locked (102), Column Selection (100.8), Tab Moves Focus (100.7), OVR and Screen Reader Mode (both 100.6), Selection (100.5), Indentation (100.4), Encoding (100.3), End of Line (100.2), Language (100.1), File Information and AI stats (100), Language Status and Language Detection (placed next to Language), Git blame (200 from the git extension), Notifications bell (last, negative infinity).
- Source: priorities in the `addEntry` calls of `editorStatus.ts`, `window.ts` (under `electron-browser`), `scrollLocking.ts`, `accessibilityStatus.ts`, `languageStatus.ts`, `notificationsStatus.ts`, `remoteIndicator.ts`, `activity.ts` (scm), `markers.contribution.ts`, `task.contribution.ts`, `remoteExplorer.ts`, `debugStatus.ts`, and `extensions\git\src\blame.ts`.

### SS-2 Bar and item colors

- Plain name: Bar and item colors
- One sentence: The bar's background, text and border in each state (normal, no folder open, debugging) and the colored item kinds (prominent, warning, error, remote, offline) with their hover, active and focus colors.
- Tag: both
- Evidence: source
- Boundary: no
- Example items:
  - No folder open: `statusBar.noFolderBackground`, `noFolderForeground`, `noFolderBorder` (V default dark and light background `#68217A`)
  - Debugging: `statusBar.debuggingBackground`, `debuggingForeground`, `debuggingBorder`, applied only while a session is active (not for no-debug runs) and only when setting `debug.enableStatusBarColor` is on (default true); also `commandCenter.debuggingBackground`
  - Remote and offline indicator colors: `statusBarItem.remoteBackground`, `remoteForeground`, `offlineBackground` (remote default comes from the activity bar badge color)
  - Item hover, active and focus: `statusBarItem.hoverBackground`, `activeBackground`, `compactHoverBackground`, `focusBorder`, `statusBar.focusBorder`
- Proof paths:
  - `src\vs\workbench\common\theme.ts` (all `statusBar.*` and `statusBarItem.*` tokens, 32 registered colors)
  - `src\vs\workbench\browser\parts\statusbar\statusbarPart.ts` (`updateStyles`, `overrideStyle`, workbench state switch, border, focus outline stylesheet)
  - `src\vs\workbench\contrib\debug\browser\statusbarColorProvider.ts` (debugging override and its three color tokens)
  - `src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css` (kind classes, hover and active rules)
  - `extensions\theme-defaults\themes\dark_modern.json` (shipped values, for example background `#181818`, debugging `#0078D4`)
- Other things that belong here: prominent items (Window Zoom, OVR, Column Selection, Screen Reader Mode, Scrolling Locked use `kind: 'prominent'`), warning kind (Problems visibility off, Pty Host), error kind (Performance Trace recording), extension items limited to error or warning background (`extHostStatusBar.ts`). There is no settings UI for these colors in V; they are theme tokens set through themes or `workbench.colorCustomizations`.

### SS-3 Current editor and language details

- Plain name: Current editor and language details
- One sentence: The right-side items that describe the active editor - line and column, indentation, encoding, line ending, language, file information, editor mode flags, notebook items - plus the combined language status item with its pinned entries.
- Tag: both
- Evidence: source
- Boundary: no
- Example items:
  - Line and column: `status.editor.selection`, text "Ln {0}, Col {1}" with selected count, click runs `workbench.action.gotoLine`
  - Indentation, encoding, end of line: `status.editor.indentation` ("Spaces: 4", "Tab Size: 4"), `status.editor.encoding` (`workbench.action.editor.changeEncoding`), `status.editor.eol` (`workbench.action.editor.changeEOL`, LF or CRLF)
  - Language mode: `status.editor.mode` (`workbench.action.editor.changeLanguageMode`), plus language status `status.languageStatus` with "Add to Status Bar" and "Remove from Status Bar" pinning
  - Mode flags and notebook: "Tab Moves Focus" (`editor.action.toggleTabFocusMode`), "OVR" (`editor.action.toggleOvertypeInsertMode`), "Column Selection", "Select Kernel" and "Cell {0} of {1}" in notebooks
- Proof paths:
  - `src\vs\workbench\browser\parts\editor\editorStatus.ts` (selection, indentation, encoding, EOL, language, file info, tab focus, OVR, column selection, current problem, and the three pickers)
  - `src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts` (combined item, dedicated pinned items, severity icons, flash, reset counter)
  - `src\vs\workbench\contrib\languageDetection\browser\languageDetection.contribution.ts` (lightbulb "Accept Detected Language")
  - `src\vs\workbench\contrib\notebook\browser\contrib\editorStatusBar\editorStatusBar.ts` (kernel info, kernel select, active cell, notebook indentation)
  - `src\vs\workbench\contrib\accessibility\browser\accessibilityStatus.ts` ("Screen Reader Optimized")
  - `src\vs\workbench\contrib\scrollLocking\browser\scrollLocking.ts` ("Scrolling Locked")
- Settings that feed this group: `problems.showCurrentInStatus` (off by default, adds a Current Problem entry), `workbench.editor.languageDetection`, `workbench.editor.preferHistoryBasedLanguageDetection`, `editor.tabFocusMode`, `editor.accessibilitySupport`. Pickers opened from the bar are quick pick dialogs and belong to section 11 Quick access.
- Boundary note: none. The language status item also hosts the inline suggestions entry, which is covered under SS-6.

### SS-4 Workspace, remote and source control

- Plain name: Workspace, remote and source control
- One sentence: The left-side items that say where you are working and what state it is in - remote connection, Restricted Mode, forwarded ports, and the Git repository, branch and sync items.
- Tag: both
- Evidence: source
- Boundary: no
- Example items:
  - Remote indicator: `status.host`, icon `$(remote)`, click opens the Remote menu (`workbench.action.remote.showMenu`), menu id `StatusBarRemoteIndicatorMenu`; changes color kind when connected or offline
  - Branch and sync (git extension through the SCM service): checkout item (`git.checkout`, branch name with an icon that changes for a protected branch or a merge or rebase in progress) and sync or publish item (`git.sync`, `git.publish`, icons `$(sync)`, `$(sync~spin)`, `$(cloud-upload)`)
  - Restricted Mode: `status.workspaceTrust`, text "$(shield) Restricted Mode", kind prominent, command `workbench.trust.manage`
  - Git blame for the current line: `git.blame`, right side, priority 200, setting `git.blame.statusBarItem.enabled` (default true), command `git.viewCommit`
- Proof paths:
  - `src\vs\workbench\contrib\remote\browser\remoteIndicator.ts` and `remoteExplorer.ts` (host indicator, forwarded ports at priority 40)
  - `src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts` (Restricted Mode entry)
  - `src\vs\workbench\contrib\scm\browser\activity.ts` (turns `statusBarCommands` into entries, repository selector `status.scm.provider` when more than one repository)
  - `extensions\git\src\statusbar.ts` (`CheckoutStatusBar`, `SyncStatusBar`) and `extensions\git\src\blame.ts`
  - `extensions\git\package.json` (settings `git.enableStatusBarSync` default true, `git.blame.statusBarItem.enabled`, `git.blame.statusBarItem.template`)
- Other things that belong here: Forwarded Ports (`status.forwardedPorts`, `$(radio-tower) 3`), Source Control Provider selector, Metered Connection (`$(radio-tower)`, `workbench.action.configureMeteredConnection`).

### SS-5 Alerts and background activity

- Plain name: Alerts and background activity
- One sentence: Items that tell you something needs attention or is running - the notifications bell, problem counts, debug state, running tasks, progress and status messages, and diagnostic indicators.
- Tag: both
- Evidence: source
- Boundary: no
- Example items:
  - Notifications bell: `status.notifications`, `$(bell)` or `$(bell-dot)`, Do Not Disturb variant `$(bell-slash)`, click toggles the notifications center (the center itself is section 10)
  - Problems: `status.problems`, `$(error) 0 $(warning) 0` (info shown only if above zero), click toggles the Problems panel; "Problems visibility" warning item when `problems.visibility` is off
  - Debug: `status.debug`, text "$(debug-alt-small) {config}", setting `debug.showInStatusBar` (`never`, `always`, `onFirstSessionStart`; default `onFirstSessionStart`), click `workbench.action.debug.selectandstart`
  - Running Tasks and progress: `status.runningTasks` (`$(tools) 2`, `workbench.action.tasks.showTasks`), `status.progress` (window progress with spinner), `status.message` (status message text)
- Proof paths:
  - `src\vs\workbench\browser\parts\notifications\notificationsStatus.ts` (bell, status message, alignment by `workbench.notifications.position`)
  - `src\vs\workbench\contrib\markers\browser\markers.contribution.ts` (Problems and Problems visibility entries)
  - `src\vs\workbench\contrib\debug\browser\debugStatus.ts`
  - `src\vs\workbench\contrib\tasks\browser\task.contribution.ts` and `src\vs\workbench\services\progress\browser\progressService.ts`
  - `src\vs\workbench\api\browser\mainThreadOutputService.ts` (Show Output quiet indicator `status.view.showQuietly`)
  - `src\vs\workbench\electron-browser\window.ts` (Window Zoom `status.windowZoom`, prominent, hover with zoom buttons)
- Other things that belong here: Current Problem text (`statusbar.currentProblem`, left), Window Zoom, Extension Profiler (`status.profiler`), Pty Host (`ptyHostStatus`, warning), Performance Trace (`status.tracing`, error kind), Terminal session recording (`recordSession`).
- Boundary note: the bell and its list belong to section 10 Notifications. Only the bar item is counted here.

### SS-6 AI suggestions and AI stats

- Plain name: AI suggestions and AI stats
- One sentence: Items tied to AI suggestions - the inline suggestions entry in the language status popup, the AI usage stats gauge, and the Copilot status entry in stock VS Code; Cursor swaps these for its own items (see the sibling file).
- Tag: both (the group exists in both products with different contents)
- Evidence: source for V, bundle for Cursor
- Boundary: yes - what these items show comes from the suggestion model or the agent. Only the bar item and its hover are in scope.
- Example items in V:
  - Inline Suggestions language status entry: id `inlineSuggestions`, labels "Loading...", "Inline completion available", "Inline edit available", "No inline suggestion available" (lives in the language status popup, short forms are `$(lightbulb)`, `$(lightbulb-sparkle)`, `$(circle-slash)`)
  - AI Usage Statistics gauge: `aiStatsStatusBar`, name "Inline Suggestions", right side priority 100, setting `editor.aiStats.enabled`, hover with a chart and a Configure link
  - Copilot Status: `chat.statusBarEntry`, name "Copilot Status", `$(copilot)` icons (chat and agent entry, so left out of the catalog)
  - Edit Source Tracking (developer): id `editTelemetry`, text "$(edit) {n} chars inserted"
- Proof paths:
  - `src\vs\workbench\contrib\inlineCompletions\browser\inlineCompletionLanguageStatusBarContribution.ts` (uses `ILanguageStatusService.addStatus`, no direct `addEntry`; registered in `inlineCompletions.contribution.ts`)
  - `src\vs\workbench\contrib\editTelemetry\browser\editStats\aiStatsStatusBar.ts` and `aiStatsFeature.ts`
  - `src\vs\workbench\contrib\editTelemetry\browser\telemetry\editSourceTrackingFeature.ts`
  - `src\vs\workbench\contrib\chat\browser\chatStatus\chatStatusEntry.ts` (out of scope, seen only)

## Built-in extension contributions (V\extensions)

The brief's in-scope extensions, checked for status bar items:

- git (`extensions\git`): yes, two things. (a) Branch and sync entries: the extension does not call `createStatusBarItem` for these. `extensions\git\src\statusbar.ts` builds `Command` objects (`CheckoutStatusBar`, `SyncStatusBar`) that the repository exposes as `sourceControl.statusBarCommands`, and the core file `src\vs\workbench\contrib\scm\browser\activity.ts` turns them into entries `status.scm.0`, `status.scm.1` and, with more than one repository, `status.scm.provider`. Setting `git.enableStatusBarSync` (default true) controls the sync item. (b) Git blame item: `window.createStatusBarItem('git.blame', Right, 200)` in `extensions\git\src\blame.ts`, name "Git Blame Information", text `$(git-commit) {template}`, command `git.viewCommit`, controlled by `git.blame.statusBarItem.enabled` (default true) and `git.blame.statusBarItem.template` (default `${authorName} (${authorDateAgo})`).
- markdown-language-features: one item. "Rendered Diff" warning, `window.createStatusBarItem('markdown.renderedDiffWarning', Right, 100)`, text "$(warning) Rendered Diff", tooltip "Rendered Markdown diffs may hide important changes. Click for details." (`extensions\markdown-language-features\src\preview\renderedDiffWarning.ts`). It also adds a language status item `markdownStatus` (`src\languageFeatures\diagnostics.ts`), which shows in the language status popup (SS-3).
- merge-conflict: none. No `createStatusBarItem`, no `StatusBarItem` anywhere in `extensions\merge-conflict`. Its UI is editor CodeLens and commands.
- simple-browser: none. No status bar code in `extensions\simple-browser`.
- search-result and references-view: none found.

Other built-in extensions that add status bar items, seen but outside the brief's extension list (they are language, debug or media support): `media-preview` (Image Zoom 102, Image Size 101, Image Binary Size 100; `ownedStatusBarEntry.ts`), `typescript-language-features` ("Configure Excludes" `status.typescript.exclude` at 98, plus language status items for version and project), `json-language-features` (`status.json.resolveError`, plus language status items), `debug-auto-launch` (`status.debug.autoAttach`, left), `vscode-test-resolver` (test only). The `extensions\copilot` folder in V adds several entries (logging, inline edit, workspace recorder, router decision) - chat and agent, left out.

Extension API route (the way third-party items get in): `vscode.window.createStatusBarItem(id, alignment, priority)` goes through `src\vs\workbench\api\browser\mainThreadStatusBar.ts` and `statusBarExtensionPoint.ts`. `window.setStatusBarMessage` makes the entry `status.extensionMessage` named "Extension Status" (`extHostStatusBar.ts`). There is also a declarative contribution point `contributes.statusBarItems` in `statusBarExtensionPoint.ts`, gated as proposed API in 1.128.0. These belong to the extension system and are only noted here.

## Left out and why

- Extension system machinery (extension host protocol, contribution point plumbing, Extensions view). Only the visible results (the "Manage Extension" row, extension item tooltips) are counted in SS-1.
- Chat and agent items: Copilot Status `chat.statusBarEntry` (`chatStatusEntry.ts`), voice keyword activation `status.voiceKeywordActivation` (`voiceChatActions.ts`), Agent Host Profiler `status.agentHostProfiler` (`profileAgentHostAction.ts`), and everything under `extensions\copilot`. Reason: chat and agent surfaces are out of scope.
- Notebook cell status bar (`contrib\notebook\browser\contrib\cellStatusBar`, `cellDiagnostics`, `view\cellParts\cellStatusPart.ts`): a different bar that sits under each cell, not the window status bar. It is part of the notebook editor (section 05).
- Terminal suggest "status bar" menu and suggest widget status bar: widget footers, covered by section 06.
- Language support extensions' own entries (TypeScript, JSON, media preview, debug auto attach): language or media support, listed above as seen only.
- Developer and test-only entries: `status.tracing` (Performance Trace) is a user-reachable developer command, kept as an example in SS-5; `editTelemetry` and `vscode-test-resolver` are developer only, seen only.
- Sessions window (Agents window) status bar behavior (`IsSessionsWindowContext`): out of scope.

## Newer than 1.128

Found by comparing V with N (1.144.0). All marked `newer-than-1.128`:

- New color `statusBar.inactiveBackground` and a status bar repaint when the window loses focus (`N\src\vs\workbench\common\theme.ts`, `N\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts`). The no-folder and debugging colors stay visible when inactive.
- New special command `statusBar.entry.toggleTooltip` (`ToggleTooltipCommand`) and a tooltip callback `onDidShow` in `N\src\vs\workbench\services\statusbar\browser\statusbar.ts` and `statusbarItem.ts`; new CSS classes `has-command` and `visible-background-color-neighbor` for neighboring colored items.
- Compact density padding for the floating-panels layout (`COMPACT_DENSITY_FLOATING_BOTTOM_PADDING`, layout setting constant `LayoutSettings.MODERN_UI_DENSITY`); V only has a fixed bottom padding in the same experiment.
- Media preview status items moved into core: `status.imagePreview.zoom` (102), `status.imagePreview.dimensions` (101), `status.mediaPreview.fileSize` (100) in `N\src\vs\workbench\contrib\mediaPreview\browser\mediaPreviewStatusBar.ts`. In V they come from the `media-preview` extension.
- New chat-side entry `codexStatusEntry.ts` ("Codex Status", `N\src\vs\workbench\contrib\chat\browser\chatStatus\`): chat, left out.
- No new settings under `workbench.statusBar.*` in N: still only `workbench.statusBar.visible`.

## Unverified in this file

- `workbench.statusBar.feedback.visible` and a feedback entry: not found in V, N or the Cursor bundle, so I treat it as not existing at these versions. If the owner remembers it from an older VS Code, that is a different version.
- Runtime behavior (which items show on a fresh profile, exact hover contents, when the language status item changes severity) was read from code, not run in stock VS Code.
- Whether the Keyboard Layout entry (`status.workbench.keyboardLayout`, text "Layout: {0}", file `contrib\preferences\browser\keyboardLayoutPicker.ts`) ever shows on desktop. I listed it as seen; I did not trace its show condition.
- Priorities for extension items depend on the extension; I did not catalog them.


