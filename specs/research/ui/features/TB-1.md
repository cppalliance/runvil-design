Group: TB-1 Menu bar. Section: Title bar (TB). Date: 2026-10-25. Pass: 2.
Features: 32.

### TB-1.1 Menu bar row in the title bar
- Tag: both
- Evidence: source
- Boundary: no
- What: On Windows, Linux and the web build, the left of the title bar shows a row of drop-down menus - File, Edit, Selection, View, Go, Run, Terminal, Help - in that order; the row is installed only in the main window, only when the OS menu is not used, and not in compact mode, so secondary windows and macOS never get it.
- Commands: none found
- Menus: MenubarMainMenu; MenubarFileMenu; MenubarEditMenu; MenubarSelectionMenu; MenubarViewMenu; MenubarGoMenu; MenubarDebugMenu; MenubarTerminalMenu; MenubarHelpMenu
- Keys: none found
- Settings: window.titleBarStyle
- Cursor: present - same row and order; installed when the title bar style is not native. In the Agents window Run, Selection and Go are hidden (left out of this catalog).
- Source: V\src\vs\workbench\browser\parts\titlebar\menubar.contribution.ts:10-90; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts:375-384,423-445,481-489; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:261; ui\entry-points\01-title-bar.md (inventory, MenubarMainMenu table and the per-menu tables); C bundle (module vs/workbench/browser/parts/titlebar/menubarControl, strings mFile to mHelp, class with `installMenubar` that appends `div.menubar` to the left title bar content)

### TB-1.2 Choose drawn menus or OS menus
- Tag: both
- Evidence: source
- Boundary: no
- What: On Windows and Linux the menus can be drawn by the app (custom) or handed to the OS (native); `window.menuStyle` picks custom, native or inherit from the title bar style, native is ignored when the title bar is custom, and a change needs a restart; macOS always uses the OS menu bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.menuStyle; window.titleBarStyle
- Cursor: changed - the setting `window.menuStyle` is not registered (0 hits); the style helpers in both the window bundle and `main.js` return `custom` for any title bar setting unless macOS has native tabs on (or native full screen off), so Windows and Linux always get the drawn bar and macOS always the OS menu
- Source: V\src\vs\platform\window\common\window.ts:142-182; V\src\vs\workbench\electron-browser\desktop.contribution.ts:270-288; V\src\vs\workbench\contrib\relauncher\browser\relauncher.contribution.ts:60,88,161; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts:163-166; C bundle (helper functions that return the title bar and menu style, the `menuStyle` property read; `window.titleBarStyle` text); C main.js (title bar style helper and the native-menu helper)

### TB-1.3 Menu bar visibility modes
- Tag: both
- Evidence: source
- Boundary: no
- What: The setting picks how the bar shows - classic (default on desktop, shown but hidden in full screen), visible (always), toggle (hidden until Alt), hidden (never), compact (one button in the side bar; default on the web); it applies to Windows, Linux and the web only, macOS native always acts as classic, and when the title bar has nothing else to show (command center, layout buttons and editor actions all off) the bar itself appears only while the menu is toggled on in toggle mode or in classic mode in full screen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.menuBarVisibility
- Cursor: present - same five values and defaults (classic on desktop, compact on web), same description strings
- Source: V\src\vs\workbench\browser\workbench.contribution.ts:868-888; V\src\vs\platform\window\common\window.ts:184-192; V\src\vs\workbench\services\layout\browser\layoutService.ts:536-615; V\src\vs\workbench\browser\layout.ts:501-521; V\src\vs\base\browser\ui\menu\menubar.ts:793-805; C bundle (`window.menuBarVisibility` block with enum classic, visible, toggle, hidden, compact; visibility helper that forces classic on macOS native; widget method `setUnfocusedState`)

### TB-1.4 Toggle Menu Bar command
- Tag: both
- Evidence: source
- Boundary: no
- What: A checked row named Menu Bar flips the bar: from classic or visible it goes to toggle (OS menu) or compact (drawn menu), from anything else back to classic; it appears in View > Appearance, in the title bar right-click menu (not in full screen or with a native title bar), and as a Menu Bar row in the Customize Layout picker; Windows, Linux and web only.
- Commands: workbench.action.toggleMenuBar
- Menus: MenubarAppearanceMenu; TitleBarContext; TitleBarTitleContext
- Keys: none found
- Settings: window.menuBarVisibility
- Cursor: present - command registered with the same toggled rule and rows in the Appearance menu and both title bar menus; Customize Layout picker has the Menu Bar toggle row too
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:730-771,1359-1362; V\src\vs\workbench\browser\layout.ts:2341-2356; ui\entry-points\01-title-bar.md (MenubarAppearanceMenu and TitleBarContext tables); C bundle (`workbench.action.toggleMenuBar` registration with `mnemonicTitle` "Menu &&Bar", title bar rows, picker list)

### TB-1.5 Press Alt to show or focus the menu bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Tapping and releasing Alt moves focus to the first menu; in toggle mode this is also what reveals a hidden bar, and in classic mode in full screen it reveals the bar; Alt again or Escape hands focus back, and the setting `window.customMenuBarAltFocus` can turn off Alt focus (it never affects toggle mode); never on macOS.
- Commands: editor.contrib.menuPreventer
- Menus: none found
- Keys: Alt (tap and release, handled by the menu bar widget, not a keybinding rule)
- Settings: window.customMenuBarAltFocus; window.menuBarVisibility
- Cursor: changed - the drawn bar's code hard-wires "disable Alt focus" to true, so Alt alone does not focus the bar unless the mode is toggle; the setting is declared with `included:false` and default false, so it is not listed in the settings editor
- Source: V\src\vs\base\browser\ui\menu\menubar.ts:888-902,927-969; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:430-439; V\src\vs\workbench\browser\workbench.contribution.ts:896-902; C bundle (widget getter `shouldAltKeyFocus`; menu bar control getter `currentDisableMenuBarAltFocus` returning true; setting `window.customMenuBarAltFocus` with `included:!1`)

### TB-1.6 Alt plus letter menu shortcuts
- Tag: both
- Evidence: source
- Boundary: no
- What: Each title has one letter marked with `&` (File F, Edit E, Selection S, View V, Go G, Run R, Terminal T, Help H); holding Alt underlines those letters, Alt plus the letter opens that menu, and inside an open menu a plain letter picks a row (a repeated letter cycles rows that share it); Windows can force the underline on all the time from its "underline access keys" accessibility option; turning the setting off frees Alt plus letter for editor keybindings; in the web build it only works in full screen.
- Commands: none found
- Menus: none found
- Keys: Alt+F; Alt+E; Alt+S; Alt+V; Alt+G; Alt+R; Alt+T; Alt+H (derived from the `&&` letters in the labels, not rules in the keybinding registry)
- Settings: window.enableMenuBarMnemonics
- Cursor: changed - the drawn bar still reads the setting and treats a missing value as on, but the setting is declared with `included:false` and default false (not listed in the settings editor); the OS menu code hard-wires mnemonics off in `main.js`, so the native menu shows plain labels
- Source: V\src\vs\base\browser\ui\menu\menubar.ts:136-138,182-198,566-611,867-886; V\src\vs\base\browser\ui\menu\menu.ts:140-165,460-472; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:463-471,514-517; V\src\vs\workbench\services\accessibility\electron-browser\accessibilityService.ts:45-54; V\src\vs\workbench\browser\workbench.contribution.ts:889-895; V\src\vs\platform\menubar\electron-main\menubar.ts:189-196; C bundle (`window.enableMenuBarMnemonics` with `included:!1`, getter `currentEnableMenuBarMnemonics`, `alwaysUnderlineAccessKeys` registry read); C main.js (getter `currentEnableMenuBarMnemonics` returns false)

### TB-1.7 Focus the menu bar by command
- Tag: both
- Evidence: source
- Boundary: no
- What: Focus Title Bar in the Command Palette moves focus into the title bar and, when a drawn menu bar exists, lands on its first menu; Focus Application Menu (Alt+F10) does the same on the menu bar but V and Cursor register it only inside the web build code path, so a desktop window has just the first command.
- Commands: workbench.action.focusTitleBar; workbench.actions.menubar.focus
- Menus: CommandPalette
- Keys: Alt+F10 (workbench.actions.menubar.focus, when IsWebContext)
- Settings: none found
- Cursor: present - both ids found; the Focus Application Menu registration sits in the same web-only branch
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts:117-133,960-967; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:294-321,743-751; V\src\vs\base\browser\ui\menu\menubar.ts:464-472; C bundle (strings `focusMenu`, `focusTitleBar`; the web-only block `Xl&&(...cyk().event(...))`)

### TB-1.8 Move between menus with the arrow keys
- Tag: both
- Evidence: source
- Boundary: no
- What: With focus on the bar, Left and Right step through the titles and wrap through the More button (macOS also Tab and Shift+Tab); Down or Enter opens the menu with the first row selected; Escape clears focus and returns it to what had it before; with a menu open, Left and Right jump straight to the next or previous menu.
- Commands: none found
- Menus: none found
- Keys: LeftArrow; RightArrow; DownArrow; Enter; Escape (menu bar focused, no menu open); Tab and Shift+Tab (macOS only)
- Settings: none found
- Cursor: present - same widget code (OVERFLOW_INDEX, `toggleFocus`, `focusPrevious`)
- Source: V\src\vs\base\browser\ui\menu\menubar.ts:120-149,228-243,807-865,462-472; C bundle (menu bar widget class, `toggleFocus` and `setUnfocusedState` bodies)

### TB-1.9 Open and switch menus with the mouse
- Tag: both
- Evidence: source
- Boundary: no
- What: Pressing the mouse on a title opens its menu straight away (the release is ignored), pressing the open title again closes it, hovering another title while a menu is open switches to it, hovering with the bar focused but closed moves focus along, clicking outside the bar or resizing the window closes everything, a touch tap opens a menu, and right-click does nothing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - same widget code (mouse-enter, mouse-down and tap handlers in the bundle's menu bar class)
- Source: V\src\vs\base\browser\ui\menu\menubar.ts:150-157,244-310,909-925; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:733-741; C bundle (menu bar widget class with `ignoreNextMouseUp`, `onMenuTriggered`)

### TB-1.10 Move inside menus and submenus with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: In an open drop-down Up and Down step through enabled rows only, Enter runs a row (Space too on macOS and Linux), a letter picks a row by its mnemonic, Right or Enter on a submenu row opens it with the first row selected (hovering opens it after 250 ms), Left or Escape closes the submenu and returns to the parent row, Escape on a top-level menu returns focus to its title, Tab is blocked, Home, End, PageUp and PageDown jump to the ends on Linux, and a submenu with no rows shows an empty placeholder; submenus open to the right and flip left when there is no room.
- Commands: none found
- Menus: none found
- Keys: UpArrow; DownArrow; Enter; Space (macOS and Linux); RightArrow; LeftArrow; Escape; Home, End, PageUp, PageDown (Linux)
- Settings: none found
- Cursor: present - same menu widget (mnemonic map, submenu item class and the 250 ms show timer are in the bundle)
- Source: V\src\vs\base\browser\ui\menu\menu.ts:100-181,725-952; V\src\vs\base\browser\ui\menu\menubar.ts:1001-1060; C bundle (menu widget and submenu item classes, `createSubmenu`)

### TB-1.11 Menu rows show shortcuts, check marks, swapped labels and disabled state
- Tag: both
- Evidence: source
- Boundary: no
- What: Every row shows its current keybinding on the right and keeps it in sync when keybindings change, toggle rows show a check mark (Auto Save, Word Wrap, Menu Bar), some rows swap their label with their state (Switch to Alt+Click for Multi-Cursor), rows hide or grey out by their context condition, group names decide the separators, submenu rows carry an arrow, and the bar rebuilds when settings, recents or update state change but not while a menu has focus; in the OS menu the shortcuts become native accelerators and a chord that cannot be an accelerator is appended to the label as `[Ctrl+K Ctrl+S]`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - the drawn bar looks keybindings up through its own helper that falls back to another command's shortcut for the two Add Symbol rows (mapped to `aichat.newchataction` and `composer.newAgentChat`)
- Source: V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:104-125,140-150,193-214,371-379,553-597,672-690; V\src\vs\base\browser\ui\menu\menu.ts:410-414,547-552,685-697; V\src\vs\platform\menubar\electron-main\menubar.ts:821-850; C bundle (control method `lookupMenuKeybinding` and its alias map); C main.js (native menu keybinding helper)

### TB-1.12 Extra menus fold into a More button on narrow windows
- Tag: both
- Evidence: source
- Boundary: no
- What: When the title bar is too narrow for every title, the ones that do not fit hide and a More button (three dots) appears at the end listing them as submenus; if fewer than about a quarter of the titles fit, all titles hide and only the button shows.
- Commands: none found
- Menus: none found
- Keys: DownArrow; Enter (on the More button, opens it)
- Settings: none found
- Cursor: present - strings `mMore` and `mAppMenu`, the `toolbar-toggle-more` class and the same overflow code are in the bundle
- Source: V\src\vs\base\browser\ui\menu\menubar.ts:313-346,473-563; V\src\vs\workbench\browser\parts\titlebar\media\menubarControl.css:29-60; C bundle (module vs/base/browser/ui/menu/menubar strings, widget method `updateOverflowAction`)

### TB-1.13 Compact menu button in the side bar
- Tag: both
- Evidence: source
- Boundary: no
- What: In compact mode the whole menu bar becomes one Application Menu hamburger button at the top of the activity bar, and its flyout lists each top-level menu as a submenu (opening right or left by the side bar position and above or below by the activity bar position); Space, Enter or the arrow toward the flyout open it, Down or Right moves on into the activity icons and Up or Left from the icons comes back; right-clicking the side bar adds a checked Menu row (compact against toggle) whenever the mode is compact, toggle or hidden, and right-clicking the button adds Hide Menu.
- Commands: none found
- Menus: none found
- Keys: Space; Enter; DownArrow; RightArrow; UpArrow; LeftArrow (on the hamburger and the activity icons)
- Settings: window.menuBarVisibility; workbench.sideBar.location; workbench.activityBar.location
- Cursor: present - the compact install code and the strings Menu and Hide Menu are in the bundle; where the button shows with Cursor's horizontal tab row in place of the rail was not checked
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts:311-337,347-415,430-440; V\src\vs\base\browser\ui\menu\menubar.ts:93-99,214-224,315-345,520-537,1007-1030; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:473-486,682-690; V\src\vs\workbench\browser\parts\titlebar\media\menubarControl.css:11-27; C bundle (activity bar part methods `installMenubar` and `fillContextMenuActions` with action ids `toggleMenuVisibility`, `hideCompactMenu`)

### TB-1.14 Open Recent submenu
- Tag: both
- Evidence: source
- Boundary: no
- What: File > Open Recent lists Reopen Closed Editor, then up to 10 recent folders and workspaces, a separator, up to 10 recent files, a separator, More... and Clear Recently Opened...; clicking a row opens it in this window, holding Ctrl or Shift (Cmd or Option on macOS) opens it in a new window, the list refreshes when the window regains focus and is not rebuilt while the bar is hidden, and the OS menu drops an entry that fails to open.
- Commands: openRecentFolder; openRecentWorkspace; openRecentFile; workbench.action.reopenClosedEditor; workbench.action.openRecent
- Menus: MenubarRecentMenu; MenubarFileMenu
- Keys: Ctrl+Shift+T (workbench.action.reopenClosedEditor); Ctrl+R (workbench.action.openRecent, Ctrl+R on macOS too, using the Control key)
- Settings: none found
- Cursor: present - same ids, same limit of 10 (`MAX_MENU_RECENT_ENTRIES`), same Ctrl+R and Ctrl+Shift+T keys
- Source: V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:71,164-188,190-226,229-265,441-446; V\src\vs\workbench\browser\actions\windowActions.ts:280-307,515-521; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:709-718; V\src\vs\workbench\browser\parts\editor\editorActions.ts:1648-1670; V\src\vs\platform\menubar\electron-main\menubar.ts:558-583; ui\entry-points\01-title-bar.md (MenubarRecentMenu table and "Runtime rows" note); C bundle (ids `openRecentFolder`, `openRecentWorkspace`, `openRecentFile`, `workbench.action.openRecent` with `KeyR`, `workbench.action.reopenClosedEditor` with `Shift|KeyT`)

### TB-1.15 Long Open Recent labels shortened in the middle
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. A recent entry whose label is longer than 120 characters is cut in the middle so the menu stays narrow; V 1.128.0 shows the full label.
- Commands: none found
- Menus: MenubarRecentMenu
- Keys: none found
- Settings: none found
- Cursor: missing - no `MAX_MENU_RECENT_LABEL_LENGTH` in the bundle and the label code has no middle cut
- Source: N\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:73,257; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:71,229-265 (the 1.128.0 code without the cut); C bundle (method that builds the recent action with `unmnemonicLabel` and no truncation)

### TB-1.16 Clear Recently Opened
- Tag: both
- Evidence: source
- Boundary: no
- What: The last row of Open Recent asks "Do you want to clear all recently opened files and workspaces?" with an irreversible warning and a Clear button, then wipes the global recent list and the per-workspace history; with no window open the macOS menu runs the same clear with a confirmation.
- Commands: workbench.action.clearRecentFiles
- Menus: MenubarRecentMenu; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present - same id, group `z_clear` and the warning dialog strings
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts:1672-1708; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:720-727; V\src\vs\platform\menubar\electron-main\menubar.ts:136-137; ui\entry-points\01-title-bar.md (MenubarRecentMenu table); C bundle (class with `ID="workbench.action.clearRecentFiles"`, `clearRecentlyOpened` calls)

### TB-1.17 Update row in the Help menu
- Tag: both
- Evidence: source
- Boundary: no
- What: On Windows and Linux the drawn Help menu puts one extra row just above About that follows the update state - Check for Updates..., Checking for Updates... (grey), Download Update, Downloading Update... (grey), Install Update... (not on macOS), Installing Update... (grey), Cancelling Update... (grey), Restart to Update; the OS menu on macOS builds the same rows in the app menu, and the bar refreshes when the state changes.
- Commands: update.check; update.checking; update.downloadNow; update.downloading; update.install; update.updating; update.cancelling; update.restart
- Menus: MenubarHelpMenu
- Keys: none found
- Settings: none found
- Cursor: changed - the same rows except Cancelling Update... (string `cancellingUpdate` is not in the bundle)
- Source: V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:380-424,441-461; V\src\vs\platform\menubar\electron-main\menubar.ts:550-556,632-682; ui\entry-points\01-title-bar.md ("Runtime rows" note); C bundle (strings 4006-4012 in module menubarControl, native update strings 2194-2200 in main.js)

### TB-1.18 View > Appearance submenu
- Tag: both
- Evidence: source
- Boundary: no
- What: View > Appearance is a 25-row submenu in five groups - Full Screen, Zen Mode, Centered Layout; Menu Bar and the part toggles; the move and position rows; editor aids (Minimap, Breadcrumbs, Sticky Scroll, Render Whitespace, Render Control Characters); Zoom In, Out and Reset - and six rows open flyouts of their own: Activity Bar Position, Panel Position, Align Panel, Tab Bar (a separate copy in Zen Mode), Editor Actions Position and Render Whitespace.
- Commands: none found
- Menus: MenubarViewMenu; MenubarAppearanceMenu; ActivityBarPositionMenu; PanelPositionMenu; PanelAlignmentMenu; EditorTabsBarShowTabsSubmenu; EditorTabsBarShowTabsZenModeSubmenu; EditorActionsPositionSubmenu; EditorRenderWhitespaceSubmenu
- Keys: none found
- Settings: none found
- Cursor: changed - 24 rows instead of 25: Custom Title Bar and the Activity Bar Position flyout are gone, Open Browser is added (shown only while the browser tab is enabled), and the Breadcrumbs row reads Toggle Breadcrumbs
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:276-283; V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts:492-584; V\src\vs\workbench\browser\parts\panel\panelActions.ts:183-243; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:377-389; V\src\vs\workbench\contrib\codeEditor\browser\toggleRenderWhitespace.ts:25-141; ui\entry-points\01-title-bar.md (MenubarAppearanceMenu table and "Submenus that are not Menubar ids" table); C bundle (`MenubarAppearanceMenu` 7 hits, `ActivityBarPositionMenu` 0 hits, `toggle.toggleCustomTitleBar` 0 hits)

### TB-1.19 View > Editor Layout submenu
- Tag: both
- Evidence: source
- Boundary: no
- What: View > Editor Layout is a 17-row flat submenu in five groups - four split directions; Split in Group or Join in Group (only one shows, by the active editor); Move or Copy Editor into New Window; eight layout presets from Single to Two Columns Bottom; and Flip Layout.
- Commands: none found
- Menus: MenubarViewMenu; MenubarLayoutMenu
- Keys: none found
- Settings: none found
- Cursor: present - all 17 rows found with the same groups and order (ids such as `workbench.action.editorLayoutTwoByTwoGrid`)
- Source: V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:733-940; V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts:871; ui\entry-points\01-title-bar.md (MenubarLayoutMenu table); C bundle (`MenubarLayoutMenu` 3 hits, `workbench.action.editorLayoutTwoByTwoGrid`)

### TB-1.20 Preferences submenu
- Tag: both
- Evidence: source
- Boundary: no
- What: On Windows and Linux this is File > Preferences, on macOS the Preferences entry in the app-name menu; it holds Profile (a flyout of profiles built at run time, or a plain Profiles row when there are none), Settings, Extensions, Keyboard Shortcuts, Configure Snippets, Tasks, Themes (a flyout with Color Theme, File Icon Theme and Product Icon Theme), three Settings Sync rows and Online Services Settings.
- Commands: none found
- Menus: MenubarPreferencesMenu; MenubarFileMenu; ThemesSubMenu; ProfilesMenu
- Keys: Ctrl+, (Cmd+, on macOS, workbench.action.openSettings); Ctrl+K Ctrl+S (workbench.action.openGlobalKeybindings)
- Settings: none found
- Cursor: changed - the three Settings Sync rows are gone; Cursor Settings (Ctrl+Shift+J) and VS Code Settings are added in the IDE window, and the stock Settings row shows only in the Agents window; the macOS top-level title has no mnemonic
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:238,245,480,892,922,1532-1538; V\src\vs\workbench\browser\parts\titlebar\menubar.contribution.ts:80-89; V\src\vs\workbench\contrib\themes\browser\themes.contribution.ts:885-935; V\src\vs\workbench\contrib\userDataProfile\browser\userDataProfile.ts:160-200,371-377; ui\entry-points\01-title-bar.md (MenubarPreferencesMenu table); C bundle (`MenubarPreferencesMenu` 7 hits, `workbench.userDataSync.actions` 0 hits, `aiSettings.action.open`)

### TB-1.21 Go > Switch Editor submenu
- Tag: both
- Evidence: source
- Boundary: no
- What: Go > Switch Editor is a 10-row submenu in five groups - First and Second Side in Editor (only while a side-by-side or compare editor is active), Next and Previous Editor, Next and Previous Used Editor, then the same two pairs limited to the current group.
- Commands: none found
- Menus: MenubarGoMenu; MenubarSwitchEditorMenu
- Keys: none found
- Settings: none found
- Cursor: present - all 10 rows found (`MenubarSwitchEditorMenu` 2 hits, `workbench.action.focusFirstSideEditor`)
- Source: V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:954-1046; ui\entry-points\01-title-bar.md (MenubarSwitchEditorMenu table); C bundle (`MenubarSwitchEditorMenu`, `workbench.action.focusFirstSideEditor`)

### TB-1.22 Go > Switch Group submenu
- Tag: both
- Evidence: source
- Boundary: no
- What: Go > Switch Group is an 11-row submenu in three groups - Group 1 to Group 5 by number, Next and Previous Group, and Group Left, Right, Above and Below.
- Commands: none found
- Menus: MenubarGoMenu; MenubarSwitchGroupMenu
- Keys: none found
- Settings: none found
- Cursor: present - all 11 rows found (`MenubarSwitchGroupMenu` 2 hits, `workbench.action.focusFirstEditorGroup`)
- Source: V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:1054-1162; ui\entry-points\01-title-bar.md (MenubarSwitchGroupMenu table); C bundle (`MenubarSwitchGroupMenu`, `workbench.action.focusFirstEditorGroup`)

### TB-1.23 Run > New Breakpoint submenu
- Tag: both
- Evidence: source
- Boundary: no
- What: Run > New Breakpoint is a 7-row submenu - Conditional Breakpoint (or Edit Breakpoint when one exists on the line), Inline Breakpoint, Function Breakpoint, Data Breakpoint (only when the debugger can set one), Logpoint and Triggered Breakpoint - and the whole entry shows only when a debugger is available.
- Commands: none found
- Menus: MenubarDebugMenu; MenubarNewBreakpointMenu
- Keys: none found
- Settings: none found
- Cursor: present - all 7 rows found (`MenubarNewBreakpointMenu` 4 hits, `editor.debug.action.conditionalBreakpoint`)
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:382; V\src\vs\workbench\contrib\debug\browser\debugEditorActions.ts:108-194; V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:1969,2125; ui\entry-points\01-title-bar.md (MenubarNewBreakpointMenu table); C bundle (`MenubarNewBreakpointMenu`, `editor.debug.action.conditionalBreakpoint`)

### TB-1.24 macOS app-name menu
- Tag: both
- Evidence: source
- Boundary: no
- What: On macOS the OS menu bar starts with a menu named after the app holding About, the update row, Preferences, Services, Hide, Hide Others, Show All and Quit; Quit asks for confirmation when `window.confirmBeforeClose` calls for it, and the whole bar is built by the main process from the menus the window sends; I read it in source and strings only, not on a Mac.
- Commands: workbench.action.showAboutDialog; workbench.action.quit
- Menus: MenubarMainMenu; MenubarPreferencesMenu
- Keys: Cmd+H (Hide); Cmd+Alt+H (Hide Others); Cmd+Q (workbench.action.quit)
- Settings: window.confirmBeforeClose
- Cursor: present - the same rows are built in `main.js` (strings 2175-2181 and the app-menu builder); not run on a Mac
- Source: V\src\vs\platform\menubar\electron-main\menubar.ts:263-300,408-483; V\src\vs\workbench\electron-browser\desktop.contribution.ts:72-91; V\src\vs\workbench\electron-browser\parts\titlebar\menubarControl.ts:30-110; ui\entry-points\01-title-bar.md ("Runtime rows" note on the native menu); C main.js (app-menu builder with `role:"services"`, `role:"hide"`, `role:"hideOthers"`, `role:"unhide"`, strings 2175-2181)

### TB-1.25 macOS Window menu and native tabs
- Tag: both
- Evidence: source
- Boundary: no
- What: The OS menu bar has a Window menu with Minimize, Zoom, Switch Window..., and Bring All to Front, plus - when `window.nativeTabs` is on - New Tab, Show Previous Tab, Show Next Tab, Move Tab to New Window and Merge All Windows; Minimize and Zoom grey out with no window.
- Commands: workbench.action.switchWindow; workbench.action.newWindowTab; workbench.action.showPreviousWindowTab; workbench.action.showNextWindowTab; workbench.action.moveWindowTabToNewWindow; workbench.action.mergeAllWindowTabs
- Menus: none found
- Keys: Cmd+M (Minimize)
- Settings: window.nativeTabs
- Cursor: present - same rows (strings 2185-2193); the tab rows follow a Cursor helper that turns native tabs off in the Agents window
- Source: V\src\vs\platform\menubar\electron-main\menubar.ts:196-208,359-364,603-630; V\src\vs\workbench\electron-browser\desktop.contribution.ts:93-110; C main.js (window-menu builder, native tab helper with `isGlassWindow`, strings 2185-2193)

### TB-1.26 macOS Dock menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-clicking the app in the Dock shows a menu whose first row is New Window, which opens an empty window even when none are open.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - the New Window row is kept; Cursor adds a block of unread agent chats under it, which is an agent surface and is not listed here
- Source: V\src\vs\platform\menubar\electron-main\menubar.ts:290-298; C main.js (Dock menu class with `createMenu`, string 2635 "New &&Window")

### TB-1.27 macOS menus with no window open
- Tag: both
- Evidence: source
- Boundary: no
- What: When the last window closes the macOS menu bar keeps only the File, Window and Help menus, with built-in handlers that work without a window (New Text File, New Window, Open, Open Folder, Open Workspace, Clear Recently Opened, and the Help web links) and the other rows greyed; the other menus are not drawn until a window takes focus, and the last menus are remembered from the previous run so the bar is not empty at start.
- Commands: workbench.action.files.newUntitledFile; workbench.action.newWindow; workbench.action.files.openFileFolder; workbench.action.files.openFolder; workbench.action.openWorkspace; workbench.action.clearRecentFiles
- Menus: MenubarFileMenu; MenubarHelpMenu
- Keys: none found
- Settings: none found
- Cursor: present - fallback handler table and `lastKnownMenubarData` storage key are in `main.js`
- Source: V\src\vs\platform\menubar\electron-main\menubar.ts:93-171,240-261,485-505,522-540; C main.js (`addFallbackHandlers`, `shouldDrawMenu`, `lastKnownMenubarData`)

### TB-1.28 Screen reader notice about the OS menu
- Tag: both
- Evidence: source
- Boundary: no
- What: On Windows or Linux, when a screen reader is detected and the OS menu is in use, the app shows a one-time info notification saying the custom menu style is the most accessible choice, with an Open Settings button that opens the settings editor filtered to `window.menuStyle`; it is never shown on macOS or the web and is remembered by the storage key `menubar/accessibleMenubarNotified`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.menuStyle; editor.accessibilitySupport
- Cursor: changed - the text says "custom title bar style" and the button opens settings filtered to `window.titleBarStyle`; because the style helper never returns native on Windows or Linux, the condition for showing it looks like it can never pass (my reading of the helper code, not run)
- Source: V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:196-206,267-291; C bundle (strings 4004 and 4005 in module menubarControl, method `notifyUserOfCustomMenubarAccessibility` with `query:"window.titleBarStyle"`)

### TB-1.29 Menu bar and menu colors
- Tag: both
- Evidence: source
- Boundary: no
- What: The titles and drop-downs follow the color theme - `menubar.selectionForeground`, `menubar.selectionBackground` and `menubar.selectionBorder` mark the title that is open, hovered or focused; `menu.foreground`, `menu.background`, `menu.border`, `menu.selectionForeground`, `menu.selectionBackground`, `menu.selectionBorder` and `menu.separatorBackground` color the drop-down; titles dim to the inactive title bar color when the window loses focus, and the compact button uses the activity bar's inactive and active colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - all ten color ids and the menu bar CSS rules (`menubar-menu-button`, `menubar.compact`, `toolbar-toggle-more`) are in the bundle
- Source: V\src\vs\workbench\common\theme.ts:695-710; V\src\vs\platform\theme\common\colors\menuColors.ts:17-45; V\src\vs\workbench\browser\parts\titlebar\media\menubarControl.css:11-60; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:691-705; C bundle (ten color id strings; rules in workbench.desktop.main.css)

### TB-1.30 Help menu learning and reference rows (stock only)
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Stock Help has 12 learning, reference and feedback rows that Cursor removed - Welcome, Documentation, Open Walkthrough..., Editor Playground, Get Started with Accessibility Features, Ask @vscode, Keyboard Shortcuts Reference, Video Tutorials, Tips and Tricks, Join Us on YouTube, Search Feature Requests and Privacy Statement; Cursor's Help keeps Show All Commands, Show Release Notes, Report Issue, View License, Toggle Developer Tools, Open Process Explorer and About, so it has 9 rows against 19.
- Commands: workbench.action.openWalkthrough; welcome.showAllWalkthroughs; workbench.action.openDocumentationUrl; workbench.action.showInteractivePlayground; workbench.action.getStartedWithAccessibilityFeatures; workbench.action.askVScode; workbench.action.keybindingsReference; workbench.action.openVideoTutorialsUrl; workbench.action.openTipsAndTricksUrl; workbench.action.openYouTubeUrl; workbench.action.openRequestFeatureUrl; workbench.action.openPrivacyStatementUrl; workbench.action.openNewsletterSignupUrl
- Menus: MenubarHelpMenu
- Keys: Ctrl+K Ctrl+R (workbench.action.keybindingsReference)
- Settings: none found
- Cursor: missing - 0 hits for ten of the twelve ids; `workbench.action.openWalkthrough` is still registered but with an empty title and no menu row, and `welcome.showAllWalkthroughs` only appears as a Welcome page entry
- Source: V\src\vs\workbench\browser\actions\helpActions.ts:36-358; V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.contribution.ts:43-48,208; V\src\vs\workbench\contrib\welcomeWalkthrough\browser\walkThrough.contribution.ts:42; ui\entry-points\01-title-bar.md (MenubarHelpMenu table); C bundle (ids searched: 0 hits except `workbench.action.openWalkthrough` with title `{value:"",original:""}` and `welcome.showAllWalkthroughs` inside a start entry)

### TB-1.31 Appearance rows for custom title bar and activity bar position (stock only)
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Stock View > Appearance has a Custom Title Bar toggle (shown when the title bar can be toggled) and an Activity Bar Position flyout with Default, Top, Bottom and Hidden; Cursor removed both rows.
- Commands: toggle.toggleCustomTitleBar; workbench.action.activityBarLocation.default; workbench.action.activityBarLocation.top; workbench.action.activityBarLocation.bottom; workbench.action.activityBarLocation.hide
- Menus: MenubarAppearanceMenu; ActivityBarPositionMenu
- Keys: none found
- Settings: window.customTitleBarVisibility; workbench.activityBar.location
- Cursor: missing - 0 hits for `toggle.toggleCustomTitleBar`, `ActivityBarPositionMenu` and `activityBarLocation`; the setting `workbench.activityBar.location` itself is still registered
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts:126-176; V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts:483-584; ui\entry-points\01-title-bar.md (MenubarAppearanceMenu table and ActivityBarPositionMenu row); C bundle (three searches above, 0 hits each)

### TB-1.32 Extra Go rows for adding a symbol to chat
- Tag: cursor
- Evidence: bundle
- Boundary: yes - both rows hand the symbol under the cursor to the chat panel, which is an agent surface
- What: Cursor adds two rows to the Go menu in the symbol group (orders 5 and 6), Add Symbol to Current Chat and Add Symbol to New Chat; in the menu each shows the shortcut of a chat command (the existing new-chat shortcut and the new agent chat shortcut) because it has none of its own.
- Commands: editor.action.addSymbolToChat; editor.action.addSymbolToNewChat
- Menus: MenubarGoMenu
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (both ids found; alias map from them to `aichat.newchataction` and `composer.newAgentChat`); ui\entry-points\01-title-bar.md (MenubarGoMenu table, last two rows)

## Handed off

- Open Recent More... picker (`workbench.action.openRecent`, Ctrl+R) - QA-4
- The Clear Recently Opened confirmation box itself - DL-1 (the menu row stays in TB-1.16)
- Quit confirmation from the macOS Quit row and File > Exit, and `window.confirmBeforeClose` - DL-2
- About dialog opened from Help > About or the macOS app menu - DL-1
- Go > Back, Forward and Last Edit Location rows - TB-2 (back and forward arrows)
- Commands behind Switch Editor, Switch Group, and Editor Layout rows (splits, layout presets, Flip Layout, move or copy into a new window), and the Tab Bar and Editor Actions Position flyouts - ED-1
- Commands behind New Breakpoint rows - ED-11
- Appearance part toggles (Primary Side Bar, Secondary Side Bar, Status Bar, Panel) and the Move Primary Side Bar row - TB-4; Panel Position and Align Panel flyouts - PN-1; Status Bar toggle - SS-1
- Full Screen and Zoom In, Out and Reset - TB-3
- Zen Mode, Centered Layout, Minimap, Breadcrumbs, Sticky Scroll, Render Whitespace, Render Control Characters rows - ED-5
- The Menu Bar toggle inside the Customize Layout picker - TB-4; inside the title bar right-click menu as a whole - TB-2
- Preferences > Settings, Keyboard Shortcuts, Cursor Settings and VS Code Settings rows - ED-8
- View > Appearance > Open Browser row (`workbench.action.openBrowserEditor`) - ED-10
- Help > Give Feedback... (`cursor.giveFeedback`, opens a Cursor feedback popup) - DL-6 (medium, the popup is a Cursor modal but not about plans or accounts)
- What the update row does after a click (download, install, restart) - TB-5
- The other rows of each menu (Go to Definition, Find in Files, Run Task, and so on) - the groups that own their commands, mapped from the inventory in a later pass

## Unclear

- `workbench.actions.menubar.focus` (Focus Application Menu) is registered only in the web build code in V and in the Cursor bundle, yet the macOS text of `window.menuBarVisibility` tells users to run it, so on a desktop window the toggle mode text points at a command that is not registered; I did not run it.
- Cursor declares `window.enableMenuBarMnemonics` and `window.customMenuBarAltFocus` with `included:false` and default false. Excluded settings are dropped from the registry (V\src\vs\platform\configuration\common\configurationRegistry.ts:767-783), so no default applies and the settings editor hides them; the drawn bar treats a missing mnemonics value as on and ignores the Alt focus setting. What happens when a user types the keys into settings.json was not tested.
- Cursor sets a flag `menubarMenuActive` on its overlay service while the drawn bar is focused or open (Windows only). I found no reader of that flag in the window bundle, so its purpose is unknown (it may feed another bundle).
- Cursor's title bar style helper (window bundle and `main.js`) returns custom for every value except macOS with `window.nativeTabs` on outside the Agents window (or `window.nativeFullScreen` off). That may mean Restore Native Title Bar (`toggle.window.titleBarStyle`) does not switch a Windows or Linux window to native; TB-3 should check.
- The inventory file says the four `workbench.action.activityBarLocation.*` commands are still in Cursor; a search for `activityBarLocation` in the bundle gave 0 hits, so TB-1.31 treats them as missing (this agrees with AB-5 being tagged vscode).
- Where the compact menu button sits in Cursor, which by default shows a horizontal tab row instead of the rail, was not checked; the install code is the stock activity bar code.
- The macOS native menu, Dock menu and Window menu were read in source and strings only, not on a Mac.
- Rows with no obvious owner among the 75 groups: Themes flyout, Configure Snippets, Tasks, Profiles and File > New Window with Profile, File > Share (Export Profile; the `github` extension row is out of scope), Report Issue, Process Explorer, Show Release Notes.
- Left out as out of scope: File > New Agents Window and Switch to Agents Window (`cursor.openOrFocusGlassWindow`); Run, Selection and Go being hidden in the Agents window (`isGlass`); the unread agent rows in the Cursor Dock menu; Help > Open Extension Monitor; the View > Chat row; the three Settings Sync rows (catalog decision 9); extension-only menus `menuBar/edit/copy`, `file/share` and the web-only `menuBar/home`; the web-only File > Confirm Before Close and web navigation actions in the compact menu.
- Newer than 1.128 and not listed as separate features because they are not menu-bar specific: the frosted-glass look and open and close animation for every menu (`workbench.experimental.modernUIFrostedGlass`) and a `menubar-focused` class that keeps an inactive title bar opaque while the menu has focus. Both are in N only and have 0 hits in the Cursor bundle.
- Pixel sizes, hover texts and the icon of the More and hamburger button were not captured.

