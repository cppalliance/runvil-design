Group: TB-1 Menu bar. Pass: 3. Date: 2026-10-25.

### TB-1.1
1. On Windows, Linux and the web build, the left of the title bar shows the menus File, Edit, Selection, View, Go, Run, Terminal and Help in that order.
2. Only the main window shows the menu row, so secondary windows never get it.
3. On macOS the title bar never shows the row, because the menus live in the OS menu bar at the top of the screen.
4. When `window.menuBarVisibility` is `compact` the row is replaced by a single button in the side bar instead of the eight titles.
5. Setting `window.titleBarStyle` to `native` hands the menus to the OS, and the drawn row is not shown.

### TB-1.2
1. Setting `window.menuStyle` chooses on Windows and Linux whether menus are drawn by the app or by the OS, with values `custom`, `native` and `inherit`. [stock only]
2. With `inherit` the menu style follows `window.titleBarStyle`. [stock only]
3. A `native` value is ignored while the title bar style is custom, so the menus stay drawn. [stock only]
4. Changing `window.menuStyle` needs a full restart before it takes effect. [stock only]
5. On macOS the menus are always the OS menu bar, whatever the setting says.
6. In Cursor `window.menuStyle` is not a registered setting, so Windows and Linux always show the drawn menus and macOS always shows the OS menu bar. [cursor only]

### TB-1.3
1. Setting `window.menuBarVisibility` set to `classic` shows the bar at the top of the window and hides it only in full screen, and it is the default on desktop.
2. The value `visible` keeps the bar shown at all times, including in full screen.
3. The value `toggle` keeps the bar hidden until the user reveals it.
4. The value `hidden` never shows the bar.
5. The value `compact` shows the menu as one button in the side bar instead of a row, and it is the default on the web build.
6. The value `compact` is ignored when `window.titleBarStyle` is `native` and `window.menuStyle` is `native` or `inherit`.
7. The setting applies on Windows, Linux and the web only, and on macOS with the OS menu the bar always acts as `classic`.
8. When the command center, layout buttons and editor actions are all off, the title bar itself shows only while the bar is revealed in `toggle` mode or while `classic` mode is in full screen.

### TB-1.4
1. Running Toggle Menu Bar (`workbench.action.toggleMenuBar`) while `window.menuBarVisibility` is `classic` or `visible` switches it to `toggle` when the OS menu is used, or to `compact` when the menu is drawn.
2. Running it from `toggle`, `hidden` or `compact` switches the setting back to `classic`.
3. View > Appearance > Menu Bar shows a check mark while the mode is `classic` or `visible`.
4. Right-clicking the title bar or the title text shows a Menu Bar row with the same check mark, except in full screen, with a native title bar, or while an auxiliary window has focus.
5. The Command Palette lists the command as View: Toggle Menu Bar.
6. On macOS the command and its rows do not exist.

### TB-1.5
1. Tapping and releasing Alt moves keyboard focus to the first menu title of the drawn bar.
2. In `toggle` mode, tapping Alt also reveals the hidden bar, and in `classic` mode in full screen it reveals the bar.
3. Tapping Alt again or pressing Escape gives focus back to what had it before.
4. Setting `window.customMenuBarAltFocus` turned off stops Alt alone from focusing the bar, and it never affects `toggle` mode.
5. Alt focus never happens on macOS.
6. In Cursor Alt alone does not focus the drawn bar unless the mode is `toggle`, and `window.customMenuBarAltFocus` is not listed in the settings editor. [cursor only]

### TB-1.6
1. Holding Alt underlines one letter in each title: F in File, E in Edit, S in Selection, V in View, G in Go, R in Run, T in Terminal and H in Help.
2. Pressing Alt plus a title's underlined letter opens that menu.
3. Inside an open menu, pressing a row's plain mnemonic letter picks that row, and repeating a letter shared by several rows cycles through them.
4. On Windows the underlines stay visible all the time when the "underline access keys" accessibility option is on.
5. Setting `window.enableMenuBarMnemonics` turned off stops Alt plus letter from opening menus and frees those keys for editor keybindings.
6. In the web build the Alt plus letter shortcuts work only in full screen.
7. In Cursor `window.enableMenuBarMnemonics` is not listed in the settings editor, the drawn bar treats a missing value as on, and the OS menu shows plain labels with no mnemonics. [cursor only]

### TB-1.7
1. Running Focus Title Bar (`workbench.action.focusTitleBar`) from the Command Palette moves keyboard focus into the title bar.
2. When a drawn menu bar exists, Focus Title Bar lands on its first menu title.
3. In the web build, pressing Alt+F10 or running Focus Application Menu (`workbench.actions.menubar.focus`) focuses the menu bar.
4. A desktop window has Focus Title Bar but no Focus Application Menu command and no Alt+F10 binding.

### TB-1.8
1. With focus on the bar, Left and Right step through the titles and wrap around through the More button.
2. On macOS, Tab and Shift+Tab also step through the titles.
3. Pressing Down or Enter on a focused title opens its menu with the first row selected.
4. Pressing Escape with focus on the bar and no menu open clears the focus and returns it to what had it before.
5. With a menu open, Left and Right jump straight to the next or previous menu.

### TB-1.9
1. Pressing the mouse on a title opens its menu straight away, and the release is ignored.
2. Pressing the open title again closes its menu.
3. Hovering another title while a menu is open switches to that title's menu.
4. Hovering a title while the bar is focused but no menu is open moves focus to that title.
5. Clicking outside the bar, or resizing the window, closes everything.
6. A touch tap on a title opens its menu.
7. Right-clicking a title does nothing.

### TB-1.10
1. In an open drop-down, Up and Down step through enabled rows only, and Tab does not move the selection.
2. Enter runs the selected row, and Space runs it too on macOS and Linux.
3. Pressing a row's mnemonic letter selects that row.
4. Right or Enter on a submenu row opens the submenu with its first row selected.
5. Hovering a submenu row opens the submenu after 250 ms.
6. Left or Escape closes a submenu and puts the selection back on its parent row.
7. Escape on a top-level menu closes it and returns focus to its title.
8. On Linux, Home and PageUp jump to the first row and End and PageDown jump to the last row.
9. A submenu with no rows shows an empty placeholder.
10. Submenus open to the right and flip to the left when there is no room.

### TB-1.11
1. Every row shows its current keybinding at the right end.
2. The shown keybinding updates when the user changes keybindings.
3. Toggle rows such as Auto Save, Word Wrap and Menu Bar show a check mark while on.
4. Some rows swap their label to match their state, for example Switch to Alt+Click for Multi-Cursor and Switch to Cmd+Click for Multi-Cursor.
5. Rows hide or grey out according to their context condition.
6. Separators appear between rows whose group names differ.
7. Rows that open a submenu show an arrow.
8. The bar rebuilds when settings, recent entries or the update state change, but not while a menu has focus.
9. In the OS menu the shortcuts appear as native accelerators, and a chord that cannot be an accelerator is appended to the label as `[Ctrl+K Ctrl+S]`.
10. In Cursor the drawn bar's shortcut lookup falls back to another command's shortcut for the two Add Symbol rows. [cursor only]

### TB-1.12
1. When the title bar is too narrow for every title, the titles that do not fit hide and a More button with three dots appears at the end of the row.
2. Opening the More button lists the hidden titles as submenus.
3. When fewer than about a quarter of the titles fit, all titles hide and only the More button shows.
4. Pressing Down or Enter on the focused More button opens it.

### TB-1.13
1. In `compact` mode the whole menu bar becomes one Application Menu hamburger button at the top of the activity bar.
2. Opening the button shows a flyout that lists each top-level menu as a submenu.
3. Setting `workbench.sideBar.location` decides whether the flyout opens to the right or the left, and `workbench.activityBar.location` decides whether it opens above or below.
4. With the button focused, Space, Enter or the arrow key pointing toward the flyout opens it.
5. Down or Right moves from the button into the activity icons, and Up or Left from the icons moves back to the button.
6. Right-clicking the side bar adds a Menu row, checked while the mode is `compact`, whenever the mode is `compact`, `toggle` or `hidden`, and picking it switches between `compact` and `toggle`.
7. Right-clicking the hamburger button adds a Hide Menu row that switches the mode to `toggle`.

### TB-1.14
1. File > Open Recent lists Reopen Closed Editor, then up to 10 recent folders and workspaces, a separator, up to 10 recent files, a separator, and finally More... and Clear Recently Opened....
2. Clicking a recent folder, workspace or file row opens it in the current window.
3. Holding Ctrl or Shift while clicking a recent row opens it in a new window, and on macOS the keys are Cmd or Option.
4. Choosing Reopen Closed Editor reopens the last closed editor, and Ctrl+Shift+T does the same from the keyboard.
5. Choosing More... runs `workbench.action.openRecent`, which Ctrl+R also runs, and the picker it shows is cataloged elsewhere.
6. The recent list refreshes when the window regains focus and is not rebuilt while the bar is hidden.
7. In the OS menu, a recent entry that fails to open is dropped from the list.

### TB-1.15
1. In newer stock VS Code, a recent entry whose label is longer than 120 characters is cut in the middle so the Open Recent menu stays narrow. [stock only]
2. In newer stock VS Code, a recent entry label of 120 characters or fewer is shown whole. [stock only]
3. In Cursor and in stock 1.128.0, every recent entry shows its full label however long it is.

### TB-1.16
1. Choosing Clear Recently Opened... at the bottom of Open Recent asks for confirmation before clearing anything.
2. Confirming with the Clear button wipes the global recent list and the per-workspace history.
3. Cancelling the confirmation leaves the recent lists as they were.
4. The Command Palette also lists Clear Recently Opened.
5. With no window open, the macOS menu runs the same clear and asks for a confirmation first.

### TB-1.17
1. On Windows and Linux the drawn Help menu shows one extra row just above About, and its label follows the update state.
2. While idle the row reads Check for Updates..., and while a check runs it reads Checking for Updates... in grey.
3. When an update can be downloaded the row reads Download Update, and while it downloads it reads Downloading Update... in grey.
4. When an update is ready to install the row reads Install Update... (not on macOS), and while it installs it reads Installing Update... in grey.
5. When the update needs a restart the row reads Restart to Update.
6. While an update is being cancelled the row reads Cancelling Update... in grey. [stock only]
7. The OS menu on macOS builds the same rows in the app menu.
8. The bar refreshes when the update state changes.

### TB-1.18
1. View > Appearance opens a submenu of 25 rows split into five groups by separators.
2. The first group holds Full Screen, Zen Mode and Centered Layout.
3. The second group holds Menu Bar and the part toggles for the Primary Side Bar, Secondary Side Bar, Status Bar and Panel.
4. The third group holds the move and position rows: Move Primary Side Bar Right or Left, Activity Bar Position, Panel Position, Align Panel, Tab Bar and Editor Actions Position.
5. The fourth group holds the editor aids Minimap, Breadcrumbs, Sticky Scroll, Render Whitespace and Render Control Characters.
6. The fifth group holds Zoom In, Zoom Out and Reset Zoom.
7. Six rows open flyouts of their own: Activity Bar Position, Panel Position, Align Panel, Tab Bar, Editor Actions Position and Render Whitespace.
8. While in Zen Mode, the Tab Bar flyout shown is a separate copy made for Zen Mode.
9. In Cursor the submenu has 24 rows instead of 25, and its Breadcrumbs row reads Toggle Breadcrumbs. [cursor only]

### TB-1.19
1. View > Editor Layout opens a flat submenu of 17 rows in five groups.
2. The first group holds Split Up, Split Down, Split Left and Split Right.
3. The second group shows either Split in Group or Join in Group, and only one of them shows depending on the active editor.
4. The third group holds Move Editor into New Window and Copy Editor into New Window.
5. The fourth group holds eight layout presets: Single, Two Columns, Three Columns, Two Rows, Three Rows, Grid (2x2), Two Rows Right and Two Columns Bottom.
6. The fifth group holds Flip Layout.

### TB-1.20
1. On Windows and Linux Preferences is a submenu of File, and on macOS it is the Preferences entry in the app-name menu.
2. The submenu holds Profile, Settings, Extensions, Keyboard Shortcuts, Configure Snippets, Tasks, Themes and Online Services Settings.
3. When profiles exist the Profile row names the current profile and opens a flyout of profiles built at run time, and when none exist it is a plain Profiles row.
4. The Themes row opens a flyout with Color Theme, File Icon Theme and Product Icon Theme.
5. The Online Services Settings row opens the settings editor filtered to online-service settings.
6. Stock Preferences also holds three Settings Sync rows. [stock only]
7. In Cursor the macOS top-level Preferences title has no mnemonic letter. [cursor only]

### TB-1.21
1. Go > Switch Editor opens a submenu of 10 rows in five groups.
2. The first group holds First Side in Editor and Second Side in Editor, shown only while a side-by-side or compare editor is active.
3. The second group holds Next Editor and Previous Editor.
4. The third group holds Next Used Editor and Previous Used Editor.
5. The fourth group holds Next Editor in Group and Previous Editor in Group.
6. The fifth group holds Next Used Editor in Group and Previous Used Editor in Group.

### TB-1.22
1. Go > Switch Group opens a submenu of 11 rows in three groups.
2. The first group holds Group 1 to Group 5 by number.
3. The second group holds Next Group and Previous Group.
4. The third group holds Group Left, Group Right, Group Above and Group Below.

### TB-1.23
1. Run > New Breakpoint opens a submenu of 7 rows: Conditional Breakpoint, Edit Breakpoint, Inline Breakpoint, Function Breakpoint, Data Breakpoint, Logpoint and Triggered Breakpoint.
2. The Conditional Breakpoint row reads Edit Breakpoint when a breakpoint already exists on the line.
3. The Data Breakpoint row shows only when the debugger can set one.
4. The whole New Breakpoint entry shows only when a debugger is available.

### TB-1.24
1. On macOS the OS menu bar starts with a menu named after the app, holding About, the update row, Preferences, Services, Hide, Hide Others, Show All and Quit.
2. Cmd+H hides the app and Cmd+Alt+H hides the other apps.
3. Cmd+Q runs Quit (`workbench.action.quit`).

### TB-1.25
1. On macOS the OS menu bar has a Window menu with Minimize, Zoom, Switch Window... and Bring All to Front.
2. Cmd+M minimizes the window.
3. Minimize and Zoom are greyed out when no window is open.
4. With `window.nativeTabs` on, the Window menu also holds New Tab, Show Previous Tab, Show Next Tab, Move Tab to New Window and Merge All Windows.

### TB-1.26
1. Right-clicking the app in the macOS Dock shows a menu whose first row is New Window.
2. Choosing New Window from the Dock menu opens an empty window even when no window is open.

### TB-1.27
1. When the last window closes, the macOS menu bar keeps only the File, Window and Help menus.
2. With no window open, the File rows New Text File, New Window, Open, Open Folder, Open Workspace and Clear Recently Opened still work.
3. With no window open, the Help rows that open web links still work.
4. With no window open, the other rows are greyed out.
5. The other menus are drawn again once a window takes focus.
6. The last menus are remembered from the previous run, so the macOS menu bar is not empty at start.

### TB-1.28
1. On Windows or Linux, when a screen reader is detected and the OS menu is in use, a one-time info notification says the custom menu style is the most accessible choice.
2. The notification has an Open Settings button that opens the settings editor filtered to `window.menuStyle`.
3. The notice is shown only once, because it is remembered under the storage key `menubar/accessibleMenubarNotified`.
4. The notice is never shown on macOS or the web.
5. In Cursor the notice says "custom title bar style" and its button filters settings to `window.titleBarStyle`, and since Windows and Linux never use the OS menu there it looks like the notice can never show. [cursor only]

### TB-1.29
1. The title that is open, hovered or focused uses the theme colors `menubar.selectionForeground`, `menubar.selectionBackground` and `menubar.selectionBorder`.
2. A drop-down's text, background and border use `menu.foreground`, `menu.background` and `menu.border`.
3. The selected row in a drop-down uses `menu.selectionForeground`, `menu.selectionBackground` and `menu.selectionBorder`.
4. Separators in a drop-down use `menu.separatorBackground`.
5. The titles dim to the inactive title bar color when the window loses focus.
6. The compact menu button uses the activity bar's inactive and active colors.

### TB-1.30
1. Stock Help has the rows Welcome, Open Walkthrough..., Editor Playground and Get Started with Accessibility Features for in-app learning content. [stock only]
2. Stock Help has an Ask @vscode row that is shown only when the built-in AI setup is neither hidden nor turned off for the workspace. [stock only]
3. Stock Help has the rows Documentation, Video Tutorials, Tips and Tricks, Join Us on YouTube, Search Feature Requests and Privacy Statement that open web pages. [stock only]
4. Stock Help has a Keyboard Shortcuts Reference row, also reached with Ctrl+K Ctrl+R. [stock only]
5. Cursor's Help menu keeps Show All Commands, Show Release Notes, Report Issue, View License, Toggle Developer Tools, Open Process Explorer and About, and has 9 rows against 19 in stock.

### TB-1.31
1. Stock View > Appearance has a Custom Title Bar toggle row, shown when the title bar can be toggled. [stock only]
2. Stock View > Appearance has an Activity Bar Position flyout with Default, Top, Bottom and Hidden. [stock only]
3. Choosing a value in the Activity Bar Position flyout moves the activity bar to match `workbench.activityBar.location`. [stock only]
4. In Cursor neither row is in the Appearance submenu, although the setting `workbench.activityBar.location` is still registered. [stock only]

### TB-1.32
1. The Go menu has an Add Symbol to Current Chat row in the symbol group. [cursor only]
2. The Go menu has an Add Symbol to New Chat row in the symbol group, right after the first one. [cursor only]
3. Add Symbol to Current Chat shows the shortcut of the existing new chat command, and Add Symbol to New Chat shows the shortcut of the new agent chat command, because neither has a shortcut of its own. [cursor only]
4. Choosing either row hands the symbol under the cursor to the chat panel. [cursor only]

## Unclear

- TB-1.4: the What line mentions a Menu Bar row in the Customize Layout picker, but the feature file hands it to TB-4, so no piece is written for it here.
- TB-1.14, TB-1.16, TB-1.24: the Open Recent picker, the Clear Recently Opened confirmation box, the Quit confirmation and the About dialog are handed off to other groups, so the pieces name only the menu side.
- TB-1.18: the pieces name the rows as they sit in each group of the submenu, and what each row does is owned by TB-3, TB-4, ED-5, ED-10, PN-1 and SS-1 per the feature's Handed off list.
- TB-1.20: the Settings, Keyboard Shortcuts, Cursor Settings and VS Code Settings rows are handed to ED-8, so there is no Cursor piece for the Cursor Settings and VS Code Settings rows here.
- TB-1.26: only 2 pieces because the Dock menu has one row in this catalog, and the Cursor unread agent chat block is left out as an agent surface.
- TB-1.24, TB-1.25, TB-1.26, TB-1.27: written from source and strings only, never seen on a Mac.
- TB-1.28 and TB-1.2: the Cursor pieces depend on a reading of minified helper code, not a run.
- TB-1.30: piece 2 does not state the full condition for the Ask @vscode row, because the condition was cut off in the feature file.
