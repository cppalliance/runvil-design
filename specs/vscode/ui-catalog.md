# UI catalog for the editor workbench

Status: all five passes done. 13 sections, 86 minor groups, 2,015 features, and 10,381 pieces. This file is the top level. Each section has its own report with every feature, its entry points, and its pieces. The flat indexes are listed under "The reports" below.

This is the source list for building the UI in parallel with the Runvil engine, the harness, and the agent. It describes stock VS Code 1.128.0 (Cursor's declared base) and what Cursor 3.27.12 adds or removes.

## Scope

In:
- The workbench UI, including the autocomplete widgets (suggest list, ghost text, parameter hints) and the visible behavior of Cursor Tab.
- The UI of these built-in extensions: Git and Source Control, Markdown preview, merge conflict, Simple Browser, search results, references view.
- From Cursor: Browser, Customize, Cursor Blame, File Blame, Review Changes, Mermaid, Multi File Edits, Global Command Preview, inline diff, the Update control, and Cursor Settings.

Out:
- The extension system and marketplace.
- The agent window and how chat looks: chat panel, Composer, Agents sidebar, Agents window, background agents, voice call, tool call and sub-agent rendering.
- Harness, inference, MCP, tools, and the model that produces suggestions.
- Cursor Canvas and Markdown Plan.

Tags: `vscode` means stock only, `cursor` means Cursor only, `both` means present in each. `[boundary]` marks a group whose UI shows agent output or configures the agent, so you can decide.

## The reports

- This file: sections, groups, scope, decisions, and counts.
- `ui-catalog-01-title-bar.md` through `ui-catalog-13-workbench-wide.md`: one report per section. Each lists its groups, and under each group every feature with its tag, evidence, whether Cursor has it, the commands, menus, keys, and settings that reach it, and the pieces that finish it.
- `ui-catalog-entry-points.md`: every command, menu, setting, and key the features name, with the features that own each one, plus the entries the coverage check excluded and why.
- `ui-catalog-menu-bar.md`: the application menu bar item by item, in VS Code and in Cursor, and the other menus drawn on the title bar.

Feature ids look like `SB-4.12`: section code, group number, feature number. Piece numbers are positions inside a feature. Ids never change. A feature that was merged or removed leaves a gap in the numbers. The structure is regular on purpose, so a tool can read it.

## How to edit this list

Cut, merge, or rename groups here. Feature-level edits go in the section reports.

## Things the explorers found that change how to read this

- Cursor's copy of some stock UI is older than 1.128.0. Its activity bar has no autoHide, compact, or position commands. Its panel, notifications, status bar, and auxiliary bar are each missing pieces that exist in the 1.128.0 source. Where a piece is missing from Cursor, the catalog tags it `vscode` and does not call it `both`.
- Cursor replaced the activity bar by default. The setting `workbench.activityBar.orientation` defaults to `horizontal`, which hides the stock rail and shows a row of pinned tabs along the top of the side bar. The vertical rail still exists when you set it to `vertical`.
- Cursor ships Workspace Trust off by default (`security.workspace.trust.enabled` is false, stock is true).
- Cursor has a second dialog engine of its own, and its quit and close confirmations ask even when `window.confirmBeforeClose` is at its default.
- Everything on the Cursor side comes from strings and module names in the bundle. Nothing was run on screen.

## Decisions I made to keep going, so you can reverse them

1. The pinned container tab row (Cursor's replacement for the activity bar) belongs to the Activity bar, group AB-6. The Side bar's frame group refers to it but does not catalog it.
2. The title bar's Accounts and Manage group was merged into AB-4. They are the same two menus, drawn in the title bar or the rail depending on a setting, and Cursor never draws them in the title bar.
3. Update cards ("Update to v...", "New update available", what's new) are in Notifications as NT-6. The title-bar Update button stays in TB-5.
4. Cursor Tab: the on-screen suggestion UI is AC-6 and AC-7, the controls (status item, commands, settings) are AC-8. The status bar item is listed in SS-6 as a cross-reference only.
5. The Search Editor is in ED-3, the Side bar's Search group only points to it. Terminal suggest is in AC-5, the Panel's terminal group only points to it.
6. NPM Scripts and the JavaScript debug views were left out because they are built-in extensions outside your list.
7. The Cursor Automations editor was left out as an agent surface. It was seen in the install and is not in your list.
8. The title-bar Private Inference Mode toggle was left out as inference.
9. Remote Explorer, Settings Sync, and Edit Sessions were left out. Ports stays inside the Panel's context-only group, marked for your call.
10. I added one group the explorers found no home for: debug, comments, and test marks drawn in the editor. Pass 2 found it held three unrelated families, so it is now three groups: ED-11 Debug in the editor (25 features), ED-12 Comments in the editor (8), ED-13 Test marks in the editor (5).
11. Pass 2 produced a list of things that fit none of the 12 sections. Pass 4 collected them in a new section 13, Workbench-wide (WB), with 9 groups (below).
12. The coverage check (pass 4) proposed 83 new features. I dropped three (Markdown source editing: paste files into Markdown, update links on move, link problems) as outside the Markdown preview scope, merged three duplicates (a second Screencast Mode, a second Issue Reporter, a second shell command install), and withdrew the empty Zen Mode group. 77 new features were added.
13. The pass 5 audit added 22 more features from a label-by-label check and a residual check, dropped one more Markdown source feature, merged 13 duplicate pairs across groups (for example the workspace name button was in both TB-2 and QA-5, and the bell-slash icon was in both NT-4 and SS-5), removed one out-of-scope feature (AX-5.4, an extension contribution point), and reworded 26 sentences in 15 features that described agent or model behavior instead of what a person sees.
14. Groups where the first explorer hit the 40-feature cap were checked a second time and grew by 4 to 30 features each. ED-10 needed two rounds.

Where the items nobody owned ended up: the editor right-click menu, saving, tasks, moving focus between parts, accessibility, help and diagnostics, developer commands, and windows and workspaces are in section 13. Profiles, the theme pickers, and Release Notes were already features under AB-4, ED-8, and ED-9. Zen Mode and Centered Layout are in ED-1 and TB-4. First-run onboarding in Cursor is in ED-9 although it is a full-window overlay, not an editor page. One item is still unsettled: Emmet abbreviation expansion, which ED-2, AC-1, and AC-4 each pointed at another group to own.

## 1. Title bar (TB)

The top strip of the window: menu bar, command center, window title, window controls, layout buttons. Menu bar contents are inventoried in the research notes (19 menu ids).

- **TB-1 Menu bar** (both). The drop-down menus, or the OS menu on macOS, that hold most commands and recent files. Examples: File and Open Recent; the Run menu; `window.menuBarVisibility`; Help update rows. Cursor drops several Help and Appearance rows and Settings Sync entries.
- **TB-2 Command center and window title** (both). The search-style box in the middle with back and forward arrows, or just the title text, plus the title bar right-click menu. Examples: the search box; back and forward; the `window.title` template; right-click toggles.
- **TB-3 Window frame and controls** (both). Native or drawn title bar, minimize, maximize and close, the app icon, the drag area, zoom and full screen. Examples: `window.titleBarStyle`; `window.controlsStyle`; double-click the icon to close; Restore Native Title Bar (Cursor).
- **TB-4 Layout controls** (both; the Switch Layout overlay is cursor) [boundary]. Buttons or a Configure Layout drop-down that toggle the side bars and panel, plus Cursor's layout presets Right, Zen, and Browser. Presets show or hide agent and chat panes.
- **TB-5 Title bar buttons and update control** (both; the buttons are cursor) [boundary]. Single-purpose buttons: the Update control, the Cursor Settings gear, Upgrade, and stock's notification bell and browser button. Cursor Settings and Upgrade depend on account and agent.

## 2. Banner (BN)

A one-line message strip under the title bar. It is small, so there are two groups.

- **BN-1 Notice strip under the title bar** (both). A single 26 px line with an icon, message, optional links and a close button, shown only when a feature asks. Cursor's code and CSS match stock. Examples: strip layout; show and hide by id; the Focus Banner command; `banner.*` theme colors.
- **BN-2 Notices that use the strip** (both, partial). Five stock callers use it and Cursor keeps two. Examples: the Restricted Mode notice; the installation folder warning; the emergency alert (stock only); remote server warnings (stock only).

## 3. Activity bar (AB)

The icon rail that switches the side bar, and Cursor's replacement for it.

- **AB-1 Rail icons and view switching** (both). The icon strip, how a click or key switches the side bar view, and overflow. Examples: click with `iconClickBehavior`; active marker and hover text with the keybinding; the Additional Views menu; Previous and Next Primary Side Bar View.
- **AB-2 Customizing the rail** (both). Choosing which containers show, their order and where they live, through menus and drag. Examples: Hide or Keep an icon; drag to reorder; drag a container onto the rail; Move To and Reset Location.
- **AB-3 Badges on rail icons** (both). The count, progress or icon marker on a rail icon and the per-icon switch to hide it. Examples: number badge with `1K+` shortening; progress badge; icon badge; Hide Badge.
- **AB-4 Accounts and Manage entries** (both). The two global icons under the rail, their menus, hiding Accounts, and their title bar placement when the rail is on top or bottom. Cursor draws them only in the vertical rail, never in the title bar, and drops the Settings Sync entries. Examples: the Accounts menu with per-account Sign Out; the Manage menu with Command Palette, Settings, Keyboard Shortcuts, Themes; Hide Accounts; the profile badge on the gear.
- **AB-5 Placement, size and visibility** (vscode, since Cursor lacks the commands). Where the bar lives (side, top, bottom, hidden), how big it is, and the settings that change that. Examples: `workbench.activityBar.location`; the Move Activity Bar commands; `compact` and `autoHide`; Toggle Activity Bar Visibility.
- **AB-6 Pinned container tab row** (cursor). Cursor's own icon tab row at the top of the side bar, its overflow list, pin and reorder controls, menus, and the `orientation` setting. Examples: default tabs Explorer, Search, Source Control, Extensions; the chevron list with pin toggles; right-click Hide or Keep; `workbench.activityBar.orientation` with a restart prompt. It has no badges, tooltips, or keyboard handling.

## 4. Side bar (SB)

The primary side pane. Its views can also be dragged into the panel or the auxiliary bar.

- **SB-1 Side bar frame and shared view behavior** (both). The pane itself and what every view in it shares: show and hide, side, headers, the "..." menu, welcome text, dragging a view elsewhere. Examples: Ctrl+B and left or right side; the view header menu; Move View and Reset View Locations. Cursor's tab row is AB-6.
- **SB-2 Explorer** (both) [boundary for Cursor-added items only]. The file tree, file right-click operations, Open Editors, and the empty-folder state. Cursor adds right-click items for chat, browser, and Cursor Blame.
- **SB-3 Search** (both). Find and replace across files with include and exclude filters and a results tree. Examples: the search box with Match Case, Whole Word and Regex toggles; the replace row; Open in Editor (the Search Editor itself is ED-3).
- **SB-4 Source Control** (both) [boundary for Cursor-added items only]. The Git panel for changes, staging, commit, repositories and the commit graph. Cursor adds an Agent Review view (id `bugbot`), a Generate Commit Message button, and a worktrees setting, and has 154 Git commands against stock's 183.
- **SB-5 Run and Debug** (both). Start button and launch picker, then Variables, Watch, Call Stack and Breakpoints while debugging. The Debug Console is PN-6.
- **SB-6 Testing** (both). The Test Explorer that lists, runs and debugs tests, plus the coverage view. Test Results is PN-6.
- **SB-7 Outline, Timeline and References** (both). The helper views for the open file's symbols, its saved history, and Find All References plus call and type hierarchies.

## 5. Editor (ED)

The center area. This is the largest section, so it has 13 groups. Autocomplete is section 6.

- **ED-1 Editor groups, tabs and layout** (both). The tab strip and group grid with split, pin, preview, lock, new windows and editor switching. Examples: split editor and layout presets; pin, preview and keep-open tabs; move editor to a new window; Show All Editors.
- **ED-2 Editing text** (both). Everyday commands for typing, selecting and reshaping text in one editor. Examples: multi-cursor; line operations; toggle comment; indentation.
- **ED-3 Find and replace** (both). The in-editor find box plus the Search Editor. Examples: the find widget with case, word and regex; replace all; find in selection; the Search Editor.
- **ED-4 Language help in the editor** (both). Hover, go to, peek, rename, quick fix, format, CodeLens and inlay hints. Examples: hover; go to definition and peek; rename symbol; quick fix and refactor.
- **ED-5 Editor view and orientation aids** (both). How text is drawn and how you keep your place. Examples: minimap; sticky scroll; breadcrumbs; code folding.
- **ED-6 Diffs, merges and change review** (both, plus Cursor items) [boundary]. Editors that compare versions, plus Cursor's views of the agent's edits and Git history. Examples: the diff editor; the multi-file diff; the merge editor and conflict CodeLens; inline diffs and Review Changes. Cursor adds 14 `inlineDiffs` commands, with Accept Edits on Ctrl+Enter and Undo Edits on Ctrl+Shift+Backspace.
- **ED-7 Notebooks, Interactive Window and REPL** (both). The cell-based editors. Examples: the notebook editor; notebook diff; the Interactive Window; the REPL editor.
- **ED-8 Settings, shortcuts and configuration editors** (both, plus Cursor items) [boundary]. Editors where you change the product, including Cursor's own settings pages. Examples: the Settings editor; the Keyboard Shortcuts editor; Cursor Settings (Ctrl+Shift+J, 18 tabs); Customize (7 tabs). Cursor Settings and Customize configure the agent.
- **ED-9 Welcome, empty and first-run screens** (both). Pages and hints shown when nothing is open or when starting out. Examples: the Welcome page; the empty-group watermark; Cursor's empty-screen landing; first-run onboarding. Cursor's Welcome page has far fewer walkthroughs (25 of 43 stock ids missing).
- **ED-10 Viewers, previews and web editors** (both, plus Cursor items) [boundary]. Editors that show a file or page without the text editor. Examples: image and binary viewers; Markdown preview; the Browser tab; the Mermaid editor. The Browser is agent-driven and Global Command Preview shows an agent command.
- **ED-11 Debug in the editor** (both). Breakpoints in the gutter, inline values, debug hover, the exception widget, and the floating debug toolbar. Added in pass 1 because no section owned these. It needs a debug adapter, and Cursor ships the JavaScript one.
- **ED-12 Comments in the editor** (both). Comment threads in the editor and the diff editor: the thread widget, the gutter icons, replying and reacting, moving between threads. These only show when an extension supplies a comment controller.
- **ED-13 Test marks in the editor** (both). Test run icons in the gutter, the gutter test menu, inline failure messages, peeking test output, and inline coverage.

## 6. Autocomplete (AC)

Only the UI and the interaction. The model that produces suggestions is out.

- **AC-1 Suggest list** (both). The IntelliSense dropdown with icons, details pane, status bar and accept keys. Examples: rows with kind icon and detail; the details pane with Read More; the Insert and Replace status bar; Tab, Enter and arrow keys.
- **AC-2 Ghost text and inline suggestions** (both). Gray inline text with a hover toolbar, accept word and line, next and previous, plus the stock inline edit gutter UI. Examples: ghost text; the toolbar (Accept, Accept Word, Accept Line, n of m); Alt+] and Alt+[; the inline edit gutter menu.
- **AC-3 Parameter hints** (both). The signature popup with the active parameter and an overload switcher. Examples: signature with active parameter; overload up and down; Ctrl+Shift+Space and Esc; docs under the signature.
- **AC-4 Snippets** (both). Tab stops, placeholders and choices, plus the insert and configure pickers. Examples: tab stops and placeholder highlight; the Insert Snippet picker; Configure Snippets; snippet code actions.
- **AC-5 Terminal suggestions** (both). The suggest popup inside the terminal with its own status menu. Examples: the popup with symbol icons; the widget status menu; details toggle; Tab and Enter accept.
- **AC-6 Cursor Tab suggestions** (cursor) [boundary]. The ways a Tab edit is shown, plus Tab to accept, partial accept, and Esc to reject. Examples: gray inline text with a "tab" margin label; a preview box with View and Accept; the "Cursor Tab Preview" diff peek; red and green line coloring. Accept is `editor.action.acceptCursorTabSuggestion`.
- **AC-7 Cursor Tab jump and imports** (cursor) [boundary]. Hints that the next edit is elsewhere and Tab will go there, plus the import hint. Examples: the "tab to jump here" line hint; the off-screen "TAB next move" arrow; a cross-file path with code preview; "import X (press Tab or ESC)".
- **AC-8 Cursor Tab controls** (cursor). The status bar item, commands and settings to turn Tab on or off, snooze it and tune it. Examples: the "Cursor Tab" status item with a hover panel; Enable, Disable, Toggle and Snooze commands; the Cursor Settings rows (Partial Accepts, Suggestions While Commenting, Imports, Ignored Files); `cursor.cpp.*` settings.

## 7. Panel (PN)

The pane usually along the bottom. Views dragged in from the side bar keep their own features.

- **PN-1 Panel frame and layout** (both). The container itself: show or hide, position, align, maximize, and the tab bar. Examples: show or hide (Ctrl+J); position and alignment; maximize and restore; tabs as labels or icons.
- **PN-2 Terminal sessions** (both). Create, name, arrange and close terminals, including profiles, the tabs list, splits, and moving to the editor or a new window. Examples: new terminal and profile picker; the side tabs list; split and groups; move to editor or window.
- **PN-3 Terminal tooling** (both). What you use inside one terminal: find, links, shell-integration marks, copy and paste. Examples: the find widget; command decorations and sticky scroll; links and quick fixes. Terminal suggest is AC-5.
- **PN-4 Problems** (both) [boundary for two Cursor menu entries]. The errors, warnings and infos list as tree or table, with filters and quick fix. Cursor adds Add to Chat and Fix with Agent to the row menu.
- **PN-5 Output and logs** (both). A read-only channel view with a channel picker and log tools. Cursor adds about 30 channels from its own extensions. Examples: channel picker; log level and filter; auto-scroll; open in editor, save, import and export.
- **PN-6 Debug Console and Test Results** (both). The two views that show the results of running code. Examples: the Debug Console REPL; console filter and find; the test results tree; test message and output.
- **PN-7 Context-only panel views** (both). Tabs that appear only when something feeds them. Examples: Comments; Ports; Refactor Preview. Ports is a remote feature, so it is marked for your call.

## 8. Auxiliary bar (AX)

The secondary side bar. Its stock occupant is Chat, which is out, so what remains is the container.

- **AX-1 Show, hide and focus** (both) [boundary]. Open, close and focus the part, and pick its starting visibility. In Cursor the toggle opens or creates the agent chat.
- **AX-2 Where it sits and how wide** (both). It follows the primary side bar to the opposite side, flips with it, and resizes within width limits. Cursor uses a 300 minimum and a 540 preferred width.
- **AX-3 Maximize and restore** (vscode). Fill the window with the part and restore the layout later. Cursor has none of these commands.
- **AX-4 Tab strip and title buttons** (both). The row of tabs, its right-click menu, and previous and next view. Cursor's strip scrolls and has fewer menu entries.
- **AX-5 Moving views and containers in** (both). Drag, menu or command routes into the part, plus the extension contribution point. Cursor lacks the extension block's stock behavior but adds its own.

AX-1 and AX-4 could merge. After the agent and chat parts are removed, Cursor adds almost nothing to this section.

## 9. Status bar (SS)

The thin bar along the bottom and every item in it.

- **SS-1 Bar behavior and per-item control** (both). How the bar is shown, ordered left and right, hidden item by item, reached by keyboard, and what hover and click do. Examples: `workbench.statusBar.visible`; the per-item "Hide" menu; Focus Status Bar; item hover.
- **SS-2 Bar and item colors** (both). Bar colors in normal, no-folder and debugging states, plus item kinds (prominent, warning, error, remote, offline). Cursor's themes make the no-folder bar and remote indicator match the normal color.
- **SS-3 Current editor and language details** (both). Right-side items for the active editor: line and column, indentation, encoding, line ending, language, mode flags, notebook items, language status. This is the heaviest group and the first to split later.
- **SS-4 Workspace, remote and source control** (both). Left-side items that say where you work: remote host, Restricted Mode, forwarded ports, Git repo, branch and sync, Git blame.
- **SS-5 Alerts and background activity** (both). Items showing something needs attention or is running: the notifications bell, Problems counts, Debug state, running tasks, progress. Cursor's bell is always right-most.
- **SS-6 AI suggestions and AI stats** (both) [boundary]. Stock has the inline suggestions entry and an AI stats gauge. Cursor swaps in its own: Cursor Tab, Tab Stats, Agent Stats, a recent-commit AI score, a workspace name item, and a memory warning. The Tab controls are AC-8.

## 10. Notifications (NT)

- **NT-1 Toast popups** (both). Short-lived corner cards, up to three, that auto-hide by severity and pause on hover or focus. Cursor's timers are longer (15, 18, 20 seconds against 10, 12, 15).
- **NT-2 Notification contents and buttons** (both). The severity icon, message with links, source, buttons, gear menu, and close. Examples: primary and dropdown buttons with "Don't Show Again"; expand and collapse; per-source mute; Copy Text.
- **NT-3 Bell and notification center** (both). The status bar bell with an unread dot opens the center list and header toolbar. Examples: Clear All; Do Not Disturb; Hide; the Show, Hide and Clear All commands. Cursor lacks the position setting and the title bar bell.
- **NT-4 Do Not Disturb** (both). Makes non-error notifications silent, globally or per source, with a bell-slash icon. Includes Zen Mode silence.
- **NT-5 Progress notifications** (both). The progress service shows as a notification bar with Cancel and as a status bar spinner. Pieces also live in SS, AB, and DL.
- **NT-6 Cursor update and what's-new cards** (cursor). Floating cards for new versions with Install Now, Later and Changelog, snoozed up to 24 hours. Examples: the "Update to v" card; the "New update available" card; the "New in" changelog card; classic update prompts. Placement here is a judgment call (medium confidence).

## 11. Quick access (QA)

- **QA-1 Command Palette** (both). The `>` list that runs any command by name, with history and a keybinding shortcut. Examples: Show All Commands (F1, Ctrl+Shift+P); recently used and other sections; the Configure Keybinding button; Clear Command History.
- **QA-2 Go to File and workspace search** (both, Cursor modified). The Ctrl+P file list, plus `#` workspace symbols and `%` Quick Search text. Cursor adds a `search.quickOpen.useVscodeSearch` setting, an "other results" split, and a "Create File" row.
- **QA-3 Go to a spot in this file** (both, Go to Offset is vscode only). Small lists that jump to a line, column or symbol in the open editor. Examples: Go to Line (Ctrl+G); Go to Symbol in Editor (Ctrl+Shift+O); Go to Symbol by Category (`@:`).
- **QA-4 Switch and open lists** (both). Prefix lists that jump to something already open or defined. Examples: Open View (Ctrl+Q); editor switchers; Switch Active Terminal; Run Task and Start Debugging.
- **QA-5 Opening quick access and the help list** (both). The ways in (keys, menus, command center), the `?` help list, and the cycle and navigate keys.
- **QA-6 Quick pick and input box widget** (both) [boundary for Cursor's preview box only]. The shared floating panel every picker reuses: list, input, title bar, buttons, checkboxes, steps. Cursor's width is a setting (default 820, stock fixed at 600) and it adds a row preview box used by two developer pickers.

## 12. Dialogs (DL)

- **DL-1 Message, confirm and input boxes, and About** (both). The generic pop-ups that tell you something, ask yes or no, offer several buttons or ask for text, drawn by the OS or the app, plus About. Cursor has a second dialog engine of its own with a persistent "don't ask again" memory and a custom About window.
- **DL-2 Unsaved changes, close and quit prompts** (both). Boxes that stop you losing work when closing an editor or window, quitting, or leaving an untitled workspace. Examples: Save, Don't Save, Cancel; confirm on quit or close; untitled workspace save; the hot-exit backup failed box.
- **DL-3 File open and save pickers** (both). Windows for browsing to a file or folder to open, add or save, either the OS picker or the app's type-a-path list. Examples: Open File, Folder and Workspace; Save As; `files.simpleDialog.enable`.
- **DL-4 Delete, move and replace confirmations** (both). Explorer boxes that ask before you delete, move, overwrite, paste or drop files. Examples: `explorer.confirmDelete`; drag and drop move; replace existing.
- **DL-5 Workspace Trust and link safety prompts** (both). Boxes that ask whether you trust a folder, its files or an outside link, plus the entry to the trust editor. Cursor ships this off by default.
- **DL-6 Cursor plan and account modals** (cursor) [boundary]. Cursor's own pop-ups for plan, pricing, upgrade, invite and privacy mode. They open from Cursor Settings and the chat queue.

## 13. Workbench-wide (WB)

Things that live in no single part of the window. This section was added in pass 4 from the list of items nobody owned (decision 11), and the coverage check filled it. Cut or merge groups here like any other group.

- **WB-1 Saving and file state** (both). Auto save, Save, Save All, Save without formatting, retry and revert after a failed save, and new text files in the default language. The save dialogs are DL-3.
- **WB-2 Tasks** (both). Rerun the last task, the task log, opening the workspace tasks file, and the "allow automatic tasks" prompt. Cursor turns automatic tasks on by default with no prompt.
- **WB-3 Focus and navigation between parts** (both). Moving focus between a view's filter box and its list, between parts, to the view on the left, right, above, or below, the hover of the focused element, and resizing table columns by command.
- **WB-4 Accessibility** (both). The accessible view, the accessibility help pages and their key assignments, and the accessibility signals (sounds and announcements) with their volume and timing settings, plus underlined links.
- **WB-5 Profiles and appearance pickers** (both). Display language prompts and font smoothing on macOS. The Profiles editor, Switch Profile, and the three theme pickers turned out to be features already listed under AB-4 (Manage menu entries) and ED-8, so they are not repeated here.
- **WB-6 Help and diagnostics** (both). View License, Process Explorer, Troubleshoot Issue, Extension Bisect, the stock Issue Reporter, and Cursor's Send Feedback pane that replaces it.
- **WB-7 Editor context menus and gutter menus** (both). The editor right-click menu as a whole, and hiding toolbar actions or resetting all menus.
- **WB-8 Developer commands** (both). The Developer category: toggle developer tools, inspect context keys, log storage and working copies, Screencast Mode, GPU info, performance profiles, reload with extensions disabled, and Cursor's debugging captures. The largest WB group.
- **WB-9 Windows and workspaces** (both). Duplicate the folders as an untitled workspace, open the workspace configuration file, the offer to open a workspace file found in a folder, the shell command install, and macOS Touch Bar buttons.

Withdrawn: the planned Zen Mode group. Zen Mode and Centered Layout were already in ED-1 and TB-4, so no leftover needed a new home.

<!-- counts:start -->
## Feature counts

Generated from the feature files. 86 groups, 2015 features, 10381 pieces. The first explorer of each group was capped at 40 features (30 for one group). Every group that reached its cap was checked a second time and grew, and the largest, ED-10, needed two rounds. Groups with 40 or more features today: SB-2, SB-4, ED-1, ED-2, ED-3, ED-4, ED-5, ED-6, ED-7, ED-8, ED-10, AC-1, AC-2, PN-2, PN-3, SS-3.

| Group | Name | Features | Cursor only | Boundary |
|---|---|---|---|---|
| TB-1 | Menu bar | 32 | 1 | 1 |
| TB-2 | Command center and window title | 16 | 1 | 0 |
| TB-3 | Window frame and controls | 23 | 2 | 1 |
| TB-4 | Layout controls | 19 | 9 | 9 |
| TB-5 | Title bar buttons and update control | 11 | 2 | 2 |
| BN-1 | Notice strip under the title bar | 10 | 0 | 0 |
| BN-2 | Notices that use the strip | 8 | 0 | 1 |
| AB-1 | Rail icons and view switching | 12 | 0 | 0 |
| AB-2 | Customizing the rail | 11 | 0 | 0 |
| AB-3 | Badges on rail icons | 8 | 0 | 0 |
| AB-4 | Accounts and Manage entries | 25 | 0 | 0 |
| AB-5 | Placement, size and visibility | 9 | 0 | 0 |
| AB-6 | Pinned container tab row | 14 | 14 | 0 |
| SB-1 | Side bar frame and shared view behavior | 24 | 2 | 0 |
| SB-2 | Explorer | 48 | 2 | 2 |
| SB-3 | Search | 30 | 0 | 0 |
| SB-4 | Source Control | 53 | 5 | 4 |
| SB-5 | Run and Debug | 31 | 0 | 0 |
| SB-6 | Testing | 30 | 0 | 0 |
| SB-7 | Outline, Timeline and References | 28 | 0 | 0 |
| ED-1 | Editor groups, tabs and layout | 50 | 2 | 1 |
| ED-2 | Editing text | 43 | 1 | 0 |
| ED-3 | Find and replace | 49 | 1 | 0 |
| ED-4 | Language help in the editor | 42 | 2 | 1 |
| ED-5 | Editor view and orientation aids | 51 | 1 | 1 |
| ED-6 | Diffs, merges and change review | 58 | 20 | 20 |
| ED-7 | Notebooks, Interactive Window and REPL | 48 | 2 | 2 |
| ED-8 | Settings, shortcuts and configuration editors | 72 | 33 | 24 |
| ED-9 | Welcome, empty and first-run screens | 26 | 7 | 4 |
| ED-10 | Viewers, previews and web editors | 91 | 32 | 51 |
| ED-11 | Debug in the editor | 25 | 0 | 0 |
| ED-12 | Comments in the editor | 8 | 0 | 0 |
| ED-13 | Test marks in the editor | 5 | 0 | 0 |
| AC-1 | Suggest list | 46 | 0 | 0 |
| AC-2 | Ghost text and inline suggestions | 48 | 2 | 0 |
| AC-3 | Parameter hints | 13 | 0 | 0 |
| AC-4 | Snippets | 21 | 0 | 0 |
| AC-5 | Terminal suggestions | 25 | 0 | 0 |
| AC-6 | Cursor Tab suggestions | 30 | 30 | 30 |
| AC-7 | Cursor Tab jump and imports | 25 | 25 | 25 |
| AC-8 | Cursor Tab controls | 19 | 19 | 2 |
| PN-1 | Panel frame and layout | 23 | 1 | 1 |
| PN-2 | Terminal sessions | 50 | 2 | 2 |
| PN-3 | Terminal tooling | 47 | 1 | 1 |
| PN-4 | Problems | 20 | 0 | 1 |
| PN-5 | Output and logs | 20 | 1 | 1 |
| PN-6 | Debug Console and Test Results | 32 | 0 | 0 |
| PN-7 | Context-only panel views | 27 | 0 | 0 |
| AX-1 | Show, hide and focus | 4 | 0 | 1 |
| AX-2 | Where it sits and how wide | 4 | 0 | 0 |
| AX-3 | Maximize and restore | 6 | 0 | 0 |
| AX-4 | Tab strip and title buttons | 10 | 1 | 0 |
| AX-5 | Moving views and containers in | 3 | 0 | 0 |
| SS-1 | Bar behavior and per-item control | 16 | 1 | 0 |
| SS-2 | Bar and item colors | 15 | 1 | 0 |
| SS-3 | Current editor and language details | 53 | 1 | 0 |
| SS-4 | Workspace, remote and source control | 16 | 0 | 0 |
| SS-5 | Alerts and background activity | 15 | 0 | 0 |
| SS-6 | AI suggestions and AI stats | 16 | 11 | 10 |
| NT-1 | Toast popups | 15 | 2 | 0 |
| NT-2 | Notification contents and buttons | 17 | 2 | 0 |
| NT-3 | Bell and notification center | 11 | 0 | 0 |
| NT-4 | Do Not Disturb | 12 | 1 | 0 |
| NT-5 | Progress notifications | 13 | 0 | 0 |
| NT-6 | Cursor update and what's-new cards | 16 | 14 | 0 |
| QA-1 | Command Palette | 21 | 2 | 2 |
| QA-2 | Go to File and workspace search | 23 | 4 | 0 |
| QA-3 | Go to a spot in this file | 15 | 0 | 0 |
| QA-4 | Switch and open lists | 19 | 0 | 0 |
| QA-5 | Opening quick access and the help list | 13 | 0 | 0 |
| QA-6 | Quick pick and input box widget | 26 | 4 | 2 |
| DL-1 | Message, confirm and input boxes, and About | 38 | 6 | 1 |
| DL-2 | Unsaved changes, close and quit prompts | 20 | 2 | 0 |
| DL-3 | File open and save pickers | 20 | 0 | 0 |
| DL-4 | Delete, move and replace confirmations | 15 | 0 | 0 |
| DL-5 | Workspace Trust and link safety prompts | 17 | 0 | 0 |
| DL-6 | Cursor plan and account modals | 15 | 15 | 11 |
| WB-1 | Saving and file state | 6 | 0 | 0 |
| WB-2 | Tasks | 7 | 0 | 0 |
| WB-3 | Focus and navigation between parts | 5 | 0 | 0 |
| WB-4 | Accessibility | 9 | 0 | 0 |
| WB-5 | Profiles and appearance pickers | 2 | 0 | 0 |
| WB-6 | Help and diagnostics | 7 | 1 | 0 |
| WB-7 | Editor context menus and gutter menus | 2 | 0 | 0 |
| WB-8 | Developer commands | 25 | 9 | 1 |
| WB-9 | Windows and workspaces | 12 | 0 | 0 |
| | Total (86 groups) | 2015 | 297 | 215 |
<!-- counts:end -->

## Limits and open questions

- Nobody ran either app. Every VS Code finding is from reading the 1.128.0 source and every Cursor finding is from strings, ids, and module names in its bundle (evidence `bundle`). Visual details such as spacing, animation, and exact appearance are not in this catalog.
- Pieces were written from the source, not from watching the app. A piece says what the source shows should happen.
- Cursor's copy of some stock UI is older than 1.128.0, so a feature tagged `vscode` or with Cursor `missing` may simply be a stock feature Cursor has not picked up yet, not a feature Cursor removed.
- Left out on purpose: the extension system, the agent window, remote development (the remote indicator and Ports are kept and marked), Settings Sync, Markdown source editing, and language support extensions.
- Open for you: whether to keep the Ports view and the remote indicator (PN-7, SS-4); TB-4.15 and TB-4.16 (the Zen and Browser layout presets, where no control that applies them was found in the IDE window); and the five Cursor Settings tabs that have ids but are not drawn in the IDE (ED-8).

## Evidence

- Stock VS Code 1.128.0: a shallow clone of the `1.128.0` tag. The existing clone at `C:\Users\Vinnie\src\vscode` is main at 1.144.0, so it was used only to spot things added after Cursor's base.
- Cursor 3.27.12 at `C:\Users\Vinnie\AppData\Local\Programs\cursor`, declared base 1.128.0. The main workbench bundle is 39.6 MiB.
- Detail by section lives in the research notes under `research/ui/vscode` and `research/ui/cursor`. The menu bar inventory is in `research/ui/entry-points`. The version check is in `research/ui/declaration-notes.md`.
