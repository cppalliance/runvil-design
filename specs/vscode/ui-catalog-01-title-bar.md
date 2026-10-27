# 1. Title bar (TB)

The top strip of the window: menu bar, command center, window title, window controls, layout buttons. Menu bar contents are inventoried in the research notes (19 menu ids).

5 groups, 101 features. Each feature lists its entry points and the pieces that finish it.

## TB-1 Menu bar

(both). The drop-down menus, or the OS menu on macOS, that hold most commands and recent files. Examples: File and Open Recent; the Run menu; `window.menuBarVisibility`; Help update rows. Cursor drops several Help and Appearance rows and Settings Sync entries.

32 features.

#### TB-1.1 Menu bar row in the title bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same row and order; installed when the title bar style is not native. In the Agents window Run, Selection and Go are hidden (left out of this catalog).
- What: On Windows, Linux and the web build, the left of the title bar shows a row of drop-down menus - File, Edit, Selection, View, Go, Run, Terminal, Help - in that order; the row is installed only in the main window, only when the OS menu is not used, and not in compact mode, so secondary windows and macOS never get it.
- Commands: none found
- Menus: MenubarMainMenu; MenubarFileMenu; MenubarEditMenu; MenubarSelectionMenu; MenubarViewMenu; MenubarGoMenu; MenubarDebugMenu; MenubarTerminalMenu; MenubarHelpMenu
- Keys: none found
- Settings: window.titleBarStyle
- Source: V\src\vs\workbench\browser\parts\titlebar\menubar.contribution.ts:10-90; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts:375-384,423-445,481-489; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:261; ui\entry-points\01-title-bar.md (inventory, MenubarMainMenu table and the per-menu tables); C bundle (module vs/workbench/browser/parts/titlebar/menubarControl, strings mFile to mHelp, class with `installMenubar` that appends `div.menubar` to the left title bar content)
- Pieces:
  1. On Windows, Linux and the web build, the left of the title bar shows the menus File, Edit, Selection, View, Go, Run, Terminal and Help in that order.
  2. Only the main window shows the menu row, so secondary windows never get it.
  3. On macOS the title bar never shows the row, because the menus live in the OS menu bar at the top of the screen.
  4. When `window.menuBarVisibility` is `compact` the row is replaced by a single button in the side bar instead of the eight titles.
  5. Setting `window.titleBarStyle` to `native` hands the menus to the OS, and the drawn row is not shown.

#### TB-1.2 Choose drawn menus or OS menus

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the setting `window.menuStyle` is not registered (0 hits); the style helpers in both the window bundle and `main.js` return `custom` for any title bar setting unless macOS has native tabs on (or native full screen off), so Windows and Linux always get the drawn bar and macOS always the OS menu
- What: On Windows and Linux the menus can be drawn by the app (custom) or handed to the OS (native); `window.menuStyle` picks custom, native or inherit from the title bar style, native is ignored when the title bar is custom, and a change needs a restart; macOS always uses the OS menu bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.menuStyle; window.titleBarStyle
- Source: V\src\vs\platform\window\common\window.ts:142-182; V\src\vs\workbench\electron-browser\desktop.contribution.ts:270-288; V\src\vs\workbench\contrib\relauncher\browser\relauncher.contribution.ts:60,88,161; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts:163-166; C bundle (helper functions that return the title bar and menu style, the `menuStyle` property read; `window.titleBarStyle` text); C main.js (title bar style helper and the native-menu helper)
- Pieces:
  1. Setting `window.menuStyle` chooses on Windows and Linux whether menus are drawn by the app or by the OS, with values `custom`, `native` and `inherit`. [stock only]
  2. With `inherit` the menu style follows `window.titleBarStyle`. [stock only]
  3. A `native` value is ignored while the title bar style is custom, so the menus stay drawn. [stock only]
  4. Changing `window.menuStyle` needs a full restart before it takes effect. [stock only]
  5. On macOS the menus are always the OS menu bar, whatever the setting says.
  6. In Cursor `window.menuStyle` is not a registered setting, so Windows and Linux always show the drawn menus and macOS always shows the OS menu bar. [cursor only]

#### TB-1.3 Menu bar visibility modes

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same five values and defaults (classic on desktop, compact on web), same description strings
- What: The setting picks how the bar shows - classic (default on desktop, shown but hidden in full screen), visible (always), toggle (hidden until Alt), hidden (never), compact (one button in the side bar; default on the web); it applies to Windows, Linux and the web only, macOS native always acts as classic, and when the title bar has nothing else to show (command center, layout buttons and editor actions all off) the bar itself appears only while the menu is toggled on in toggle mode or in classic mode in full screen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.menuBarVisibility
- Source: V\src\vs\workbench\browser\workbench.contribution.ts:868-888; V\src\vs\platform\window\common\window.ts:184-192; V\src\vs\workbench\services\layout\browser\layoutService.ts:536-615; V\src\vs\workbench\browser\layout.ts:501-521; V\src\vs\base\browser\ui\menu\menubar.ts:793-805; C bundle (`window.menuBarVisibility` block with enum classic, visible, toggle, hidden, compact; visibility helper that forces classic on macOS native; widget method `setUnfocusedState`)
- Pieces:
  1. Setting `window.menuBarVisibility` set to `classic` shows the bar at the top of the window and hides it only in full screen, and it is the default on desktop.
  2. The value `visible` keeps the bar shown at all times, including in full screen.
  3. The value `toggle` keeps the bar hidden until the user reveals it.
  4. The value `hidden` never shows the bar.
  5. The value `compact` shows the menu as one button in the side bar instead of a row, and it is the default on the web build.
  6. The value `compact` is ignored when `window.titleBarStyle` is `native` and `window.menuStyle` is `native` or `inherit`.
  7. The setting applies on Windows, Linux and the web only, and on macOS with the OS menu the bar always acts as `classic`.
  8. When the command center, layout buttons and editor actions are all off, the title bar itself shows only while the bar is revealed in `toggle` mode or while `classic` mode is in full screen.

#### TB-1.4 Toggle Menu Bar command

- Tag: both. Evidence: source. Boundary: no. Cursor: present - command registered with the same toggled rule and rows in the Appearance menu and both title bar menus; Customize Layout picker has the Menu Bar toggle row too
- What: A checked row named Menu Bar flips the bar: from classic or visible it goes to toggle (OS menu) or compact (drawn menu), from anything else back to classic; it appears in View > Appearance, in the title bar right-click menu (not in full screen or with a native title bar), and as a Menu Bar row in the Customize Layout picker; Windows, Linux and web only.
- Commands: workbench.action.toggleMenuBar
- Menus: MenubarAppearanceMenu; TitleBarContext; TitleBarTitleContext
- Keys: none found
- Settings: window.menuBarVisibility
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:730-771,1359-1362; V\src\vs\workbench\browser\layout.ts:2341-2356; ui\entry-points\01-title-bar.md (MenubarAppearanceMenu and TitleBarContext tables); C bundle (`workbench.action.toggleMenuBar` registration with `mnemonicTitle` "Menu &&Bar", title bar rows, picker list)
- Pieces:
  1. Running Toggle Menu Bar (`workbench.action.toggleMenuBar`) while `window.menuBarVisibility` is `classic` or `visible` switches it to `toggle` when the OS menu is used, or to `compact` when the menu is drawn.
  2. Running it from `toggle`, `hidden` or `compact` switches the setting back to `classic`.
  3. View > Appearance > Menu Bar shows a check mark while the mode is `classic` or `visible`.
  4. Right-clicking the title bar or the title text shows a Menu Bar row with the same check mark, except in full screen, with a native title bar, or while an auxiliary window has focus.
  5. The Command Palette lists the command as View: Toggle Menu Bar.
  6. On macOS the command and its rows do not exist.

#### TB-1.5 Press Alt to show or focus the menu bar

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the drawn bar's code hard-wires "disable Alt focus" to true, so Alt alone does not focus the bar unless the mode is toggle; the setting is declared with `included:false` and default false, so it is not listed in the settings editor
- What: Tapping and releasing Alt moves focus to the first menu; in toggle mode this is also what reveals a hidden bar, and in classic mode in full screen it reveals the bar; Alt again or Escape hands focus back, and the setting `window.customMenuBarAltFocus` can turn off Alt focus (it never affects toggle mode); never on macOS.
- Commands: editor.contrib.menuPreventer
- Menus: none found
- Keys: Alt (tap and release, handled by the menu bar widget, not a keybinding rule)
- Settings: window.customMenuBarAltFocus; window.menuBarVisibility
- Source: V\src\vs\base\browser\ui\menu\menubar.ts:888-902,927-969; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:430-439; V\src\vs\workbench\browser\workbench.contribution.ts:896-902; C bundle (widget getter `shouldAltKeyFocus`; menu bar control getter `currentDisableMenuBarAltFocus` returning true; setting `window.customMenuBarAltFocus` with `included:!1`)
- Pieces:
  1. Tapping and releasing Alt moves keyboard focus to the first menu title of the drawn bar.
  2. In `toggle` mode, tapping Alt also reveals the hidden bar, and in `classic` mode in full screen it reveals the bar.
  3. Tapping Alt again or pressing Escape gives focus back to what had it before.
  4. Setting `window.customMenuBarAltFocus` turned off stops Alt alone from focusing the bar, and it never affects `toggle` mode.
  5. Alt focus never happens on macOS.
  6. In Cursor Alt alone does not focus the drawn bar unless the mode is `toggle`, and `window.customMenuBarAltFocus` is not listed in the settings editor. [cursor only]

#### TB-1.6 Alt plus letter menu shortcuts

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the drawn bar still reads the setting and treats a missing value as on, but the setting is declared with `included:false` and default false (not listed in the settings editor); the OS menu code hard-wires mnemonics off in `main.js`, so the native menu shows plain labels
- What: Each title has one letter marked with `&` (File F, Edit E, Selection S, View V, Go G, Run R, Terminal T, Help H); holding Alt underlines those letters, Alt plus the letter opens that menu, and inside an open menu a plain letter picks a row (a repeated letter cycles rows that share it); Windows can force the underline on all the time from its "underline access keys" accessibility option; turning the setting off frees Alt plus letter for editor keybindings; in the web build it only works in full screen.
- Commands: none found
- Menus: none found
- Keys: Alt+F; Alt+E; Alt+S; Alt+V; Alt+G; Alt+R; Alt+T; Alt+H (derived from the `&&` letters in the labels, not rules in the keybinding registry)
- Settings: window.enableMenuBarMnemonics
- Source: V\src\vs\base\browser\ui\menu\menubar.ts:136-138,182-198,566-611,867-886; V\src\vs\base\browser\ui\menu\menu.ts:140-165,460-472; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:463-471,514-517; V\src\vs\workbench\services\accessibility\electron-browser\accessibilityService.ts:45-54; V\src\vs\workbench\browser\workbench.contribution.ts:889-895; V\src\vs\platform\menubar\electron-main\menubar.ts:189-196; C bundle (`window.enableMenuBarMnemonics` with `included:!1`, getter `currentEnableMenuBarMnemonics`, `alwaysUnderlineAccessKeys` registry read); C main.js (getter `currentEnableMenuBarMnemonics` returns false)
- Pieces:
  1. Holding Alt underlines one letter in each title: F in File, E in Edit, S in Selection, V in View, G in Go, R in Run, T in Terminal and H in Help.
  2. Pressing Alt plus a title's underlined letter opens that menu.
  3. Inside an open menu, pressing a row's plain mnemonic letter picks that row, and repeating a letter shared by several rows cycles through them.
  4. On Windows the underlines stay visible all the time when the "underline access keys" accessibility option is on.
  5. Setting `window.enableMenuBarMnemonics` turned off stops Alt plus letter from opening menus and frees those keys for editor keybindings.
  6. In the web build the Alt plus letter shortcuts work only in full screen.
  7. In Cursor `window.enableMenuBarMnemonics` is not listed in the settings editor, the drawn bar treats a missing value as on, and the OS menu shows plain labels with no mnemonics. [cursor only]

#### TB-1.7 Focus the menu bar by command

- Tag: both. Evidence: source. Boundary: no. Cursor: present - both ids found; the Focus Application Menu registration sits in the same web-only branch
- What: Focus Title Bar in the Command Palette moves focus into the title bar and, when a drawn menu bar exists, lands on its first menu; Focus Application Menu (Alt+F10) does the same on the menu bar but V and Cursor register it only inside the web build code path, so a desktop window has just the first command.
- Commands: workbench.action.focusTitleBar; workbench.actions.menubar.focus
- Menus: CommandPalette
- Keys: Alt+F10 (workbench.actions.menubar.focus, when IsWebContext)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts:117-133,960-967; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:294-321,743-751; V\src\vs\base\browser\ui\menu\menubar.ts:464-472; C bundle (strings `focusMenu`, `focusTitleBar`; the web-only block `Xl&&(...cyk().event(...))`)
- Pieces:
  1. Running Focus Title Bar (`workbench.action.focusTitleBar`) from the Command Palette moves keyboard focus into the title bar.
  2. When a drawn menu bar exists, Focus Title Bar lands on its first menu title.
  3. In the web build, pressing Alt+F10 or running Focus Application Menu (`workbench.actions.menubar.focus`) focuses the menu bar.
  4. A desktop window has Focus Title Bar but no Focus Application Menu command and no Alt+F10 binding.

#### TB-1.8 Move between menus with the arrow keys

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same widget code (OVERFLOW_INDEX, `toggleFocus`, `focusPrevious`)
- What: With focus on the bar, Left and Right step through the titles and wrap through the More button (macOS also Tab and Shift+Tab); Down or Enter opens the menu with the first row selected; Escape clears focus and returns it to what had it before; with a menu open, Left and Right jump straight to the next or previous menu.
- Commands: none found
- Menus: none found
- Keys: LeftArrow; RightArrow; DownArrow; Enter; Escape (menu bar focused, no menu open); Tab and Shift+Tab (macOS only)
- Settings: none found
- Source: V\src\vs\base\browser\ui\menu\menubar.ts:120-149,228-243,807-865,462-472; C bundle (menu bar widget class, `toggleFocus` and `setUnfocusedState` bodies)
- Pieces:
  1. With focus on the bar, Left and Right step through the titles and wrap around through the More button.
  2. On macOS, Tab and Shift+Tab also step through the titles.
  3. Pressing Down or Enter on a focused title opens its menu with the first row selected.
  4. Pressing Escape with focus on the bar and no menu open clears the focus and returns it to what had it before.
  5. With a menu open, Left and Right jump straight to the next or previous menu.

#### TB-1.9 Open and switch menus with the mouse

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same widget code (mouse-enter, mouse-down and tap handlers in the bundle's menu bar class)
- What: Pressing the mouse on a title opens its menu straight away (the release is ignored), pressing the open title again closes it, hovering another title while a menu is open switches to it, hovering with the bar focused but closed moves focus along, clicking outside the bar or resizing the window closes everything, a touch tap opens a menu, and right-click does nothing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\base\browser\ui\menu\menubar.ts:150-157,244-310,909-925; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:733-741; C bundle (menu bar widget class with `ignoreNextMouseUp`, `onMenuTriggered`)
- Pieces:
  1. Pressing the mouse on a title opens its menu straight away, and the release is ignored.
  2. Pressing the open title again closes its menu.
  3. Hovering another title while a menu is open switches to that title's menu.
  4. Hovering a title while the bar is focused but no menu is open moves focus to that title.
  5. Clicking outside the bar, or resizing the window, closes everything.
  6. A touch tap on a title opens its menu.
  7. Right-clicking a title does nothing.

#### TB-1.10 Move inside menus and submenus with the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same menu widget (mnemonic map, submenu item class and the 250 ms show timer are in the bundle)
- What: In an open drop-down Up and Down step through enabled rows only, Enter runs a row (Space too on macOS and Linux), a letter picks a row by its mnemonic, Right or Enter on a submenu row opens it with the first row selected (hovering opens it after 250 ms), Left or Escape closes the submenu and returns to the parent row, Escape on a top-level menu returns focus to its title, Tab is blocked, Home, End, PageUp and PageDown jump to the ends on Linux, and a submenu with no rows shows an empty placeholder; submenus open to the right and flip left when there is no room.
- Commands: none found
- Menus: none found
- Keys: UpArrow; DownArrow; Enter; Space (macOS and Linux); RightArrow; LeftArrow; Escape; Home, End, PageUp, PageDown (Linux)
- Settings: none found
- Source: V\src\vs\base\browser\ui\menu\menu.ts:100-181,725-952; V\src\vs\base\browser\ui\menu\menubar.ts:1001-1060; C bundle (menu widget and submenu item classes, `createSubmenu`)
- Pieces:
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

#### TB-1.11 Menu rows show shortcuts, check marks, swapped labels and disabled state

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the drawn bar looks keybindings up through its own helper that falls back to another command's shortcut for the two Add Symbol rows (mapped to `aichat.newchataction` and `composer.newAgentChat`)
- What: Every row shows its current keybinding on the right and keeps it in sync when keybindings change, toggle rows show a check mark (Auto Save, Word Wrap, Menu Bar), some rows swap their label with their state (Switch to Alt+Click for Multi-Cursor), rows hide or grey out by their context condition, group names decide the separators, submenu rows carry an arrow, and the bar rebuilds when settings, recents or update state change but not while a menu has focus; in the OS menu the shortcuts become native accelerators and a chord that cannot be an accelerator is appended to the label as `[Ctrl+K Ctrl+S]`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:104-125,140-150,193-214,371-379,553-597,672-690; V\src\vs\base\browser\ui\menu\menu.ts:410-414,547-552,685-697; V\src\vs\platform\menubar\electron-main\menubar.ts:821-850; C bundle (control method `lookupMenuKeybinding` and its alias map); C main.js (native menu keybinding helper)
- Pieces:
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

#### TB-1.12 Extra menus fold into a More button on narrow windows

- Tag: both. Evidence: source. Boundary: no. Cursor: present - strings `mMore` and `mAppMenu`, the `toolbar-toggle-more` class and the same overflow code are in the bundle
- What: When the title bar is too narrow for every title, the ones that do not fit hide and a More button (three dots) appears at the end listing them as submenus; if fewer than about a quarter of the titles fit, all titles hide and only the button shows.
- Commands: none found
- Menus: none found
- Keys: DownArrow; Enter (on the More button, opens it)
- Settings: none found
- Source: V\src\vs\base\browser\ui\menu\menubar.ts:313-346,473-563; V\src\vs\workbench\browser\parts\titlebar\media\menubarControl.css:29-60; C bundle (module vs/base/browser/ui/menu/menubar strings, widget method `updateOverflowAction`)
- Pieces:
  1. When the title bar is too narrow for every title, the titles that do not fit hide and a More button with three dots appears at the end of the row.
  2. Opening the More button lists the hidden titles as submenus.
  3. When fewer than about a quarter of the titles fit, all titles hide and only the More button shows.
  4. Pressing Down or Enter on the focused More button opens it.

#### TB-1.13 Compact menu button in the side bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the compact install code and the strings Menu and Hide Menu are in the bundle; where the button shows with Cursor's horizontal tab row in place of the rail was not checked
- What: In compact mode the whole menu bar becomes one Application Menu hamburger button at the top of the activity bar, and its flyout lists each top-level menu as a submenu (opening right or left by the side bar position and above or below by the activity bar position); Space, Enter or the arrow toward the flyout open it, Down or Right moves on into the activity icons and Up or Left from the icons comes back; right-clicking the side bar adds a checked Menu row (compact against toggle) whenever the mode is compact, toggle or hidden, and right-clicking the button adds Hide Menu.
- Commands: none found
- Menus: none found
- Keys: Space; Enter; DownArrow; RightArrow; UpArrow; LeftArrow (on the hamburger and the activity icons)
- Settings: window.menuBarVisibility; workbench.sideBar.location; workbench.activityBar.location
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts:311-337,347-415,430-440; V\src\vs\base\browser\ui\menu\menubar.ts:93-99,214-224,315-345,520-537,1007-1030; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:473-486,682-690; V\src\vs\workbench\browser\parts\titlebar\media\menubarControl.css:11-27; C bundle (activity bar part methods `installMenubar` and `fillContextMenuActions` with action ids `toggleMenuVisibility`, `hideCompactMenu`)
- Pieces:
  1. In `compact` mode the whole menu bar becomes one Application Menu hamburger button at the top of the activity bar.
  2. Opening the button shows a flyout that lists each top-level menu as a submenu.
  3. Setting `workbench.sideBar.location` decides whether the flyout opens to the right or the left, and `workbench.activityBar.location` decides whether it opens above or below.
  4. With the button focused, Space, Enter or the arrow key pointing toward the flyout opens it.
  5. Down or Right moves from the button into the activity icons, and Up or Left from the icons moves back to the button.
  6. Right-clicking the side bar adds a Menu row, checked while the mode is `compact`, whenever the mode is `compact`, `toggle` or `hidden`, and picking it switches between `compact` and `toggle`.
  7. Right-clicking the hamburger button adds a Hide Menu row that switches the mode to `toggle`.

#### TB-1.14 Open Recent submenu

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same ids, same limit of 10 (`MAX_MENU_RECENT_ENTRIES`), same Ctrl+R and Ctrl+Shift+T keys
- What: File > Open Recent lists Reopen Closed Editor, then up to 10 recent folders and workspaces, a separator, up to 10 recent files, a separator, More... and Clear Recently Opened...; clicking a row opens it in this window, holding Ctrl or Shift (Cmd or Option on macOS) opens it in a new window, the list refreshes when the window regains focus and is not rebuilt while the bar is hidden, and the OS menu drops an entry that fails to open.
- Commands: openRecentFolder; openRecentWorkspace; openRecentFile; workbench.action.reopenClosedEditor; workbench.action.openRecent
- Menus: MenubarRecentMenu; MenubarFileMenu
- Keys: Ctrl+Shift+T (workbench.action.reopenClosedEditor); Ctrl+R (workbench.action.openRecent, Ctrl+R on macOS too, using the Control key)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:71,164-188,190-226,229-265,441-446; V\src\vs\workbench\browser\actions\windowActions.ts:280-307,515-521; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:709-718; V\src\vs\workbench\browser\parts\editor\editorActions.ts:1648-1670; V\src\vs\platform\menubar\electron-main\menubar.ts:558-583; ui\entry-points\01-title-bar.md (MenubarRecentMenu table and "Runtime rows" note); C bundle (ids `openRecentFolder`, `openRecentWorkspace`, `openRecentFile`, `workbench.action.openRecent` with `KeyR`, `workbench.action.reopenClosedEditor` with `Shift|KeyT`)
- Pieces:
  1. File > Open Recent lists Reopen Closed Editor, then up to 10 recent folders and workspaces, a separator, up to 10 recent files, a separator, and finally More... and Clear Recently Opened....
  2. Clicking a recent folder, workspace or file row opens it in the current window.
  3. Holding Ctrl or Shift while clicking a recent row opens it in a new window, and on macOS the keys are Cmd or Option.
  4. Choosing Reopen Closed Editor reopens the last closed editor, and Ctrl+Shift+T does the same from the keyboard.
  5. Choosing More... runs `workbench.action.openRecent`, which Ctrl+R also runs, and the picker it shows is cataloged elsewhere.
  6. The recent list refreshes when the window regains focus and is not rebuilt while the bar is hidden.
  7. In the OS menu, a recent entry that fails to open is dropped from the list.

#### TB-1.15 Long Open Recent labels shortened in the middle

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - no `MAX_MENU_RECENT_LABEL_LENGTH` in the bundle and the label code has no middle cut
- What: newer-than-1.128. A recent entry whose label is longer than 120 characters is cut in the middle so the menu stays narrow; V 1.128.0 shows the full label.
- Commands: none found
- Menus: MenubarRecentMenu
- Keys: none found
- Settings: none found
- Source: N\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:73,257; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:71,229-265 (the 1.128.0 code without the cut); C bundle (method that builds the recent action with `unmnemonicLabel` and no truncation)
- Pieces:
  1. In newer stock VS Code, a recent entry whose label is longer than 120 characters is cut in the middle so the Open Recent menu stays narrow. [stock only]
  2. In newer stock VS Code, a recent entry label of 120 characters or fewer is shown whole. [stock only]
  3. In Cursor and in stock 1.128.0, every recent entry shows its full label however long it is.

#### TB-1.16 Clear Recently Opened

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same id, group `z_clear` and the warning dialog strings
- What: The last row of Open Recent asks "Do you want to clear all recently opened files and workspaces?" with an irreversible warning and a Clear button, then wipes the global recent list and the per-workspace history; with no window open the macOS menu runs the same clear with a confirmation.
- Commands: workbench.action.clearRecentFiles
- Menus: MenubarRecentMenu; CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts:1672-1708; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:720-727; V\src\vs\platform\menubar\electron-main\menubar.ts:136-137; ui\entry-points\01-title-bar.md (MenubarRecentMenu table); C bundle (class with `ID="workbench.action.clearRecentFiles"`, `clearRecentlyOpened` calls)
- Pieces:
  1. Choosing Clear Recently Opened... at the bottom of Open Recent asks for confirmation before clearing anything.
  2. Confirming with the Clear button wipes the global recent list and the per-workspace history.
  3. Cancelling the confirmation leaves the recent lists as they were.
  4. The Command Palette also lists Clear Recently Opened.
  5. With no window open, the macOS menu runs the same clear and asks for a confirmation first.

#### TB-1.17 Update row in the Help menu

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the same rows except Cancelling Update... (string `cancellingUpdate` is not in the bundle)
- What: On Windows and Linux the drawn Help menu puts one extra row just above About that follows the update state - Check for Updates..., Checking for Updates... (grey), Download Update, Downloading Update... (grey), Install Update... (not on macOS), Installing Update... (grey), Cancelling Update... (grey), Restart to Update; the OS menu on macOS builds the same rows in the app menu, and the bar refreshes when the state changes.
- Commands: update.check; update.checking; update.downloadNow; update.downloading; update.install; update.updating; update.cancelling; update.restart
- Menus: MenubarHelpMenu
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:380-424,441-461; V\src\vs\platform\menubar\electron-main\menubar.ts:550-556,632-682; ui\entry-points\01-title-bar.md ("Runtime rows" note); C bundle (strings 4006-4012 in module menubarControl, native update strings 2194-2200 in main.js)
- Pieces:
  1. On Windows and Linux the drawn Help menu shows one extra row just above About, and its label follows the update state.
  2. While idle the row reads Check for Updates..., and while a check runs it reads Checking for Updates... in grey.
  3. When an update can be downloaded the row reads Download Update, and while it downloads it reads Downloading Update... in grey.
  4. When an update is ready to install the row reads Install Update... (not on macOS), and while it installs it reads Installing Update... in grey.
  5. When the update needs a restart the row reads Restart to Update.
  6. While an update is being cancelled the row reads Cancelling Update... in grey. [stock only]
  7. The OS menu on macOS builds the same rows in the app menu.
  8. The bar refreshes when the update state changes.

#### TB-1.18 View > Appearance submenu

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - 24 rows instead of 25: Custom Title Bar and the Activity Bar Position flyout are gone, Open Browser is added (shown only while the browser tab is enabled), and the Breadcrumbs row reads Toggle Breadcrumbs
- What: View > Appearance is a 25-row submenu in five groups - Full Screen, Zen Mode, Centered Layout; Menu Bar and the part toggles; the move and position rows; editor aids (Minimap, Breadcrumbs, Sticky Scroll, Render Whitespace, Render Control Characters); Zoom In, Out and Reset - and six rows open flyouts of their own: Activity Bar Position, Panel Position, Align Panel, Tab Bar (a separate copy in Zen Mode), Editor Actions Position and Render Whitespace.
- Commands: none found
- Menus: MenubarViewMenu; MenubarAppearanceMenu; ActivityBarPositionMenu; PanelPositionMenu; PanelAlignmentMenu; EditorTabsBarShowTabsSubmenu; EditorTabsBarShowTabsZenModeSubmenu; EditorActionsPositionSubmenu; EditorRenderWhitespaceSubmenu
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:276-283; V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts:492-584; V\src\vs\workbench\browser\parts\panel\panelActions.ts:183-243; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:377-389; V\src\vs\workbench\contrib\codeEditor\browser\toggleRenderWhitespace.ts:25-141; ui\entry-points\01-title-bar.md (MenubarAppearanceMenu table and "Submenus that are not Menubar ids" table); C bundle (`MenubarAppearanceMenu` 7 hits, `ActivityBarPositionMenu` 0 hits, `toggle.toggleCustomTitleBar` 0 hits)
- Pieces:
  1. View > Appearance opens a submenu of 25 rows split into five groups by separators.
  2. The first group holds Full Screen, Zen Mode and Centered Layout.
  3. The second group holds Menu Bar and the part toggles for the Primary Side Bar, Secondary Side Bar, Status Bar and Panel.
  4. The third group holds the move and position rows: Move Primary Side Bar Right or Left, Activity Bar Position, Panel Position, Align Panel, Tab Bar and Editor Actions Position.
  5. The fourth group holds the editor aids Minimap, Breadcrumbs, Sticky Scroll, Render Whitespace and Render Control Characters.
  6. The fifth group holds Zoom In, Zoom Out and Reset Zoom.
  7. Six rows open flyouts of their own: Activity Bar Position, Panel Position, Align Panel, Tab Bar, Editor Actions Position and Render Whitespace.
  8. While in Zen Mode, the Tab Bar flyout shown is a separate copy made for Zen Mode.
  9. In Cursor the submenu has 24 rows instead of 25, and its Breadcrumbs row reads Toggle Breadcrumbs. [cursor only]

#### TB-1.19 View > Editor Layout submenu

- Tag: both. Evidence: source. Boundary: no. Cursor: present - all 17 rows found with the same groups and order (ids such as `workbench.action.editorLayoutTwoByTwoGrid`)
- What: View > Editor Layout is a 17-row flat submenu in five groups - four split directions; Split in Group or Join in Group (only one shows, by the active editor); Move or Copy Editor into New Window; eight layout presets from Single to Two Columns Bottom; and Flip Layout.
- Commands: none found
- Menus: MenubarViewMenu; MenubarLayoutMenu
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:733-940; V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts:871; ui\entry-points\01-title-bar.md (MenubarLayoutMenu table); C bundle (`MenubarLayoutMenu` 3 hits, `workbench.action.editorLayoutTwoByTwoGrid`)
- Pieces:
  1. View > Editor Layout opens a flat submenu of 17 rows in five groups.
  2. The first group holds Split Up, Split Down, Split Left and Split Right.
  3. The second group shows either Split in Group or Join in Group, and only one of them shows depending on the active editor.
  4. The third group holds Move Editor into New Window and Copy Editor into New Window.
  5. The fourth group holds eight layout presets: Single, Two Columns, Three Columns, Two Rows, Three Rows, Grid (2x2), Two Rows Right and Two Columns Bottom.
  6. The fifth group holds Flip Layout.

#### TB-1.20 Preferences submenu

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the three Settings Sync rows are gone; Cursor Settings (Ctrl+Shift+J) and VS Code Settings are added in the IDE window, and the stock Settings row shows only in the Agents window; the macOS top-level title has no mnemonic
- What: On Windows and Linux this is File > Preferences, on macOS the Preferences entry in the app-name menu; it holds Profile (a flyout of profiles built at run time, or a plain Profiles row when there are none), Settings, Extensions, Keyboard Shortcuts, Configure Snippets, Tasks, Themes (a flyout with Color Theme, File Icon Theme and Product Icon Theme), three Settings Sync rows and Online Services Settings.
- Commands: none found
- Menus: MenubarPreferencesMenu; MenubarFileMenu; ThemesSubMenu; ProfilesMenu
- Keys: Ctrl+, (Cmd+, on macOS, workbench.action.openSettings); Ctrl+K Ctrl+S (workbench.action.openGlobalKeybindings)
- Settings: none found
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts:238,245,480,892,922,1532-1538; V\src\vs\workbench\browser\parts\titlebar\menubar.contribution.ts:80-89; V\src\vs\workbench\contrib\themes\browser\themes.contribution.ts:885-935; V\src\vs\workbench\contrib\userDataProfile\browser\userDataProfile.ts:160-200,371-377; ui\entry-points\01-title-bar.md (MenubarPreferencesMenu table); C bundle (`MenubarPreferencesMenu` 7 hits, `workbench.userDataSync.actions` 0 hits, `aiSettings.action.open`)
- Pieces:
  1. On Windows and Linux Preferences is a submenu of File, and on macOS it is the Preferences entry in the app-name menu.
  2. The submenu holds Profile, Settings, Extensions, Keyboard Shortcuts, Configure Snippets, Tasks, Themes and Online Services Settings.
  3. When profiles exist the Profile row names the current profile and opens a flyout of profiles built at run time, and when none exist it is a plain Profiles row.
  4. The Themes row opens a flyout with Color Theme, File Icon Theme and Product Icon Theme.
  5. The Online Services Settings row opens the settings editor filtered to online-service settings.
  6. Stock Preferences also holds three Settings Sync rows. [stock only]
  7. In Cursor the macOS top-level Preferences title has no mnemonic letter. [cursor only]

#### TB-1.21 Go > Switch Editor submenu

- Tag: both. Evidence: source. Boundary: no. Cursor: present - all 10 rows found (`MenubarSwitchEditorMenu` 2 hits, `workbench.action.focusFirstSideEditor`)
- What: Go > Switch Editor is a 10-row submenu in five groups - First and Second Side in Editor (only while a side-by-side or compare editor is active), Next and Previous Editor, Next and Previous Used Editor, then the same two pairs limited to the current group.
- Commands: none found
- Menus: MenubarGoMenu; MenubarSwitchEditorMenu
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:954-1046; ui\entry-points\01-title-bar.md (MenubarSwitchEditorMenu table); C bundle (`MenubarSwitchEditorMenu`, `workbench.action.focusFirstSideEditor`)
- Pieces:
  1. Go > Switch Editor opens a submenu of 10 rows in five groups.
  2. The first group holds First Side in Editor and Second Side in Editor, shown only while a side-by-side or compare editor is active.
  3. The second group holds Next Editor and Previous Editor.
  4. The third group holds Next Used Editor and Previous Used Editor.
  5. The fourth group holds Next Editor in Group and Previous Editor in Group.
  6. The fifth group holds Next Used Editor in Group and Previous Used Editor in Group.

#### TB-1.22 Go > Switch Group submenu

- Tag: both. Evidence: source. Boundary: no. Cursor: present - all 11 rows found (`MenubarSwitchGroupMenu` 2 hits, `workbench.action.focusFirstEditorGroup`)
- What: Go > Switch Group is an 11-row submenu in three groups - Group 1 to Group 5 by number, Next and Previous Group, and Group Left, Right, Above and Below.
- Commands: none found
- Menus: MenubarGoMenu; MenubarSwitchGroupMenu
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editor.contribution.ts:1054-1162; ui\entry-points\01-title-bar.md (MenubarSwitchGroupMenu table); C bundle (`MenubarSwitchGroupMenu`, `workbench.action.focusFirstEditorGroup`)
- Pieces:
  1. Go > Switch Group opens a submenu of 11 rows in three groups.
  2. The first group holds Group 1 to Group 5 by number.
  3. The second group holds Next Group and Previous Group.
  4. The third group holds Group Left, Group Right, Group Above and Group Below.

#### TB-1.23 Run > New Breakpoint submenu

- Tag: both. Evidence: source. Boundary: no. Cursor: present - all 7 rows found (`MenubarNewBreakpointMenu` 4 hits, `editor.debug.action.conditionalBreakpoint`)
- What: Run > New Breakpoint is a 7-row submenu - Conditional Breakpoint (or Edit Breakpoint when one exists on the line), Inline Breakpoint, Function Breakpoint, Data Breakpoint (only when the debugger can set one), Logpoint and Triggered Breakpoint - and the whole entry shows only when a debugger is available.
- Commands: none found
- Menus: MenubarDebugMenu; MenubarNewBreakpointMenu
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:382; V\src\vs\workbench\contrib\debug\browser\debugEditorActions.ts:108-194; V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:1969,2125; ui\entry-points\01-title-bar.md (MenubarNewBreakpointMenu table); C bundle (`MenubarNewBreakpointMenu`, `editor.debug.action.conditionalBreakpoint`)
- Pieces:
  1. Run > New Breakpoint opens a submenu of 7 rows: Conditional Breakpoint, Edit Breakpoint, Inline Breakpoint, Function Breakpoint, Data Breakpoint, Logpoint and Triggered Breakpoint.
  2. The Conditional Breakpoint row reads Edit Breakpoint when a breakpoint already exists on the line.
  3. The Data Breakpoint row shows only when the debugger can set one.
  4. The whole New Breakpoint entry shows only when a debugger is available.

#### TB-1.24 macOS app-name menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the same rows are built in `main.js` (strings 2175-2181 and the app-menu builder); not run on a Mac
- What: On macOS the OS menu bar starts with a menu named after the app holding About, the update row, Preferences, Services, Hide, Hide Others, Show All and Quit; Quit asks for confirmation when `window.confirmBeforeClose` calls for it, and the whole bar is built by the main process from the menus the window sends; I read it in source and strings only, not on a Mac.
- Commands: workbench.action.showAboutDialog; workbench.action.quit
- Menus: MenubarMainMenu; MenubarPreferencesMenu
- Keys: Cmd+H (Hide); Cmd+Alt+H (Hide Others); Cmd+Q (workbench.action.quit)
- Settings: window.confirmBeforeClose
- Source: V\src\vs\platform\menubar\electron-main\menubar.ts:263-300,408-483; V\src\vs\workbench\electron-browser\desktop.contribution.ts:72-91; V\src\vs\workbench\electron-browser\parts\titlebar\menubarControl.ts:30-110; ui\entry-points\01-title-bar.md ("Runtime rows" note on the native menu); C main.js (app-menu builder with `role:"services"`, `role:"hide"`, `role:"hideOthers"`, `role:"unhide"`, strings 2175-2181)
- Pieces:
  1. On macOS the OS menu bar starts with a menu named after the app, holding About, the update row, Preferences, Services, Hide, Hide Others, Show All and Quit.
  2. Cmd+H hides the app and Cmd+Alt+H hides the other apps.
  3. Cmd+Q runs Quit (`workbench.action.quit`).

#### TB-1.25 macOS Window menu and native tabs

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same rows (strings 2185-2193); the tab rows follow a Cursor helper that turns native tabs off in the Agents window
- What: The OS menu bar has a Window menu with Minimize, Zoom, Switch Window..., and Bring All to Front, plus - when `window.nativeTabs` is on - New Tab, Show Previous Tab, Show Next Tab, Move Tab to New Window and Merge All Windows; Minimize and Zoom grey out with no window.
- Commands: workbench.action.switchWindow; workbench.action.newWindowTab; workbench.action.showPreviousWindowTab; workbench.action.showNextWindowTab; workbench.action.moveWindowTabToNewWindow; workbench.action.mergeAllWindowTabs
- Menus: none found
- Keys: Cmd+M (Minimize)
- Settings: window.nativeTabs
- Source: V\src\vs\platform\menubar\electron-main\menubar.ts:196-208,359-364,603-630; V\src\vs\workbench\electron-browser\desktop.contribution.ts:93-110; C main.js (window-menu builder, native tab helper with `isGlassWindow`, strings 2185-2193)
- Pieces:
  1. On macOS the OS menu bar has a Window menu with Minimize, Zoom, Switch Window... and Bring All to Front.
  2. Cmd+M minimizes the window.
  3. Minimize and Zoom are greyed out when no window is open.
  4. With `window.nativeTabs` on, the Window menu also holds New Tab, Show Previous Tab, Show Next Tab, Move Tab to New Window and Merge All Windows.

#### TB-1.26 macOS Dock menu

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the New Window row is kept; Cursor adds a block of unread agent chats under it, which is an agent surface and is not listed here
- What: Right-clicking the app in the Dock shows a menu whose first row is New Window, which opens an empty window even when none are open.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\menubar\electron-main\menubar.ts:290-298; C main.js (Dock menu class with `createMenu`, string 2635 "New &&Window")
- Pieces:
  1. Right-clicking the app in the macOS Dock shows a menu whose first row is New Window.
  2. Choosing New Window from the Dock menu opens an empty window even when no window is open.

#### TB-1.27 macOS menus with no window open

- Tag: both. Evidence: source. Boundary: no. Cursor: present - fallback handler table and `lastKnownMenubarData` storage key are in `main.js`
- What: When the last window closes the macOS menu bar keeps only the File, Window and Help menus, with built-in handlers that work without a window (New Text File, New Window, Open, Open Folder, Open Workspace, Clear Recently Opened, and the Help web links) and the other rows greyed; the other menus are not drawn until a window takes focus, and the last menus are remembered from the previous run so the bar is not empty at start.
- Commands: workbench.action.files.newUntitledFile; workbench.action.newWindow; workbench.action.files.openFileFolder; workbench.action.files.openFolder; workbench.action.openWorkspace; workbench.action.clearRecentFiles
- Menus: MenubarFileMenu; MenubarHelpMenu
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\menubar\electron-main\menubar.ts:93-171,240-261,485-505,522-540; C main.js (`addFallbackHandlers`, `shouldDrawMenu`, `lastKnownMenubarData`)
- Pieces:
  1. When the last window closes, the macOS menu bar keeps only the File, Window and Help menus.
  2. With no window open, the File rows New Text File, New Window, Open, Open Folder, Open Workspace and Clear Recently Opened still work.
  3. With no window open, the Help rows that open web links still work.
  4. With no window open, the other rows are greyed out.
  5. The other menus are drawn again once a window takes focus.
  6. The last menus are remembered from the previous run, so the macOS menu bar is not empty at start.

#### TB-1.28 Screen reader notice about the OS menu

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the text says "custom title bar style" and the button opens settings filtered to `window.titleBarStyle`; because the style helper never returns native on Windows or Linux, the condition for showing it looks like it can never pass (my reading of the helper code, not run)
- What: On Windows or Linux, when a screen reader is detected and the OS menu is in use, the app shows a one-time info notification saying the custom menu style is the most accessible choice, with an Open Settings button that opens the settings editor filtered to `window.menuStyle`; it is never shown on macOS or the web and is remembered by the storage key `menubar/accessibleMenubarNotified`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.menuStyle; editor.accessibilitySupport
- Source: V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:196-206,267-291; C bundle (strings 4004 and 4005 in module menubarControl, method `notifyUserOfCustomMenubarAccessibility` with `query:"window.titleBarStyle"`)
- Pieces:
  1. On Windows or Linux, when a screen reader is detected and the OS menu is in use, a one-time info notification says the custom menu style is the most accessible choice.
  2. The notification has an Open Settings button that opens the settings editor filtered to `window.menuStyle`.
  3. The notice is shown only once, because it is remembered under the storage key `menubar/accessibleMenubarNotified`.
  4. The notice is never shown on macOS or the web.
  5. In Cursor the notice says "custom title bar style" and its button filters settings to `window.titleBarStyle`, and since Windows and Linux never use the OS menu there it looks like the notice can never show. [cursor only]

#### TB-1.29 Menu bar and menu colors

- Tag: both. Evidence: source. Boundary: no. Cursor: present - all ten color ids and the menu bar CSS rules (`menubar-menu-button`, `menubar.compact`, `toolbar-toggle-more`) are in the bundle
- What: The titles and drop-downs follow the color theme - `menubar.selectionForeground`, `menubar.selectionBackground` and `menubar.selectionBorder` mark the title that is open, hovered or focused; `menu.foreground`, `menu.background`, `menu.border`, `menu.selectionForeground`, `menu.selectionBackground`, `menu.selectionBorder` and `menu.separatorBackground` color the drop-down; titles dim to the inactive title bar color when the window loses focus, and the compact button uses the activity bar's inactive and active colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts:695-710; V\src\vs\platform\theme\common\colors\menuColors.ts:17-45; V\src\vs\workbench\browser\parts\titlebar\media\menubarControl.css:11-60; V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts:691-705; C bundle (ten color id strings; rules in workbench.desktop.main.css)
- Pieces:
  1. The title that is open, hovered or focused uses the theme colors `menubar.selectionForeground`, `menubar.selectionBackground` and `menubar.selectionBorder`.
  2. A drop-down's text, background and border use `menu.foreground`, `menu.background` and `menu.border`.
  3. The selected row in a drop-down uses `menu.selectionForeground`, `menu.selectionBackground` and `menu.selectionBorder`.
  4. Separators in a drop-down use `menu.separatorBackground`.
  5. The titles dim to the inactive title bar color when the window loses focus.
  6. The compact menu button uses the activity bar's inactive and active colors.

#### TB-1.30 Help menu learning and reference rows (stock only)

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - 0 hits for ten of the twelve ids; `workbench.action.openWalkthrough` is still registered but with an empty title and no menu row, and `welcome.showAllWalkthroughs` only appears as a Welcome page entry
- What: Stock Help has 12 learning, reference and feedback rows that Cursor removed - Welcome, Documentation, Open Walkthrough..., Editor Playground, Get Started with Accessibility Features, Ask @vscode, Keyboard Shortcuts Reference, Video Tutorials, Tips and Tricks, Join Us on YouTube, Search Feature Requests and Privacy Statement; Cursor's Help keeps Show All Commands, Show Release Notes, Report Issue, View License, Toggle Developer Tools, Open Process Explorer and About, so it has 9 rows against 19.
- Commands: workbench.action.openWalkthrough; welcome.showAllWalkthroughs; workbench.action.openDocumentationUrl; workbench.action.showInteractivePlayground; workbench.action.getStartedWithAccessibilityFeatures; workbench.action.askVScode; workbench.action.keybindingsReference; workbench.action.openVideoTutorialsUrl; workbench.action.openTipsAndTricksUrl; workbench.action.openYouTubeUrl; workbench.action.openRequestFeatureUrl; workbench.action.openPrivacyStatementUrl; workbench.action.openNewsletterSignupUrl
- Menus: MenubarHelpMenu
- Keys: Ctrl+K Ctrl+R (workbench.action.keybindingsReference)
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\helpActions.ts:36-358; V\src\vs\workbench\contrib\welcomeGettingStarted\browser\gettingStarted.contribution.ts:43-48,208; V\src\vs\workbench\contrib\welcomeWalkthrough\browser\walkThrough.contribution.ts:42; ui\entry-points\01-title-bar.md (MenubarHelpMenu table); C bundle (ids searched: 0 hits except `workbench.action.openWalkthrough` with title `{value:"",original:""}` and `welcome.showAllWalkthroughs` inside a start entry)
- Pieces:
  1. Stock Help has the rows Welcome, Open Walkthrough..., Editor Playground and Get Started with Accessibility Features for in-app learning content. [stock only]
  2. Stock Help has an Ask @vscode row that is shown only when the built-in AI setup is neither hidden nor turned off for the workspace. [stock only]
  3. Stock Help has the rows Documentation, Video Tutorials, Tips and Tricks, Join Us on YouTube, Search Feature Requests and Privacy Statement that open web pages. [stock only]
  4. Stock Help has a Keyboard Shortcuts Reference row, also reached with Ctrl+K Ctrl+R. [stock only]
  5. Cursor's Help menu keeps Show All Commands, Show Release Notes, Report Issue, View License, Toggle Developer Tools, Open Process Explorer and About, and has 9 rows against 19 in stock.

#### TB-1.31 Appearance rows for custom title bar and activity bar position (stock only)

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - 0 hits for `toggle.toggleCustomTitleBar`, `ActivityBarPositionMenu` and `activityBarLocation`; the setting `workbench.activityBar.location` itself is still registered
- What: Stock View > Appearance has a Custom Title Bar toggle (shown when the title bar can be toggled) and an Activity Bar Position flyout with Default, Top, Bottom and Hidden; Cursor removed both rows.
- Commands: toggle.toggleCustomTitleBar; workbench.action.activityBarLocation.default; workbench.action.activityBarLocation.top; workbench.action.activityBarLocation.bottom; workbench.action.activityBarLocation.hide
- Menus: MenubarAppearanceMenu; ActivityBarPositionMenu
- Keys: none found
- Settings: window.customTitleBarVisibility; workbench.activityBar.location
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts:126-176; V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts:483-584; ui\entry-points\01-title-bar.md (MenubarAppearanceMenu table and ActivityBarPositionMenu row); C bundle (three searches above, 0 hits each)
- Pieces:
  1. Stock View > Appearance has a Custom Title Bar toggle row, shown when the title bar can be toggled. [stock only]
  2. Stock View > Appearance has an Activity Bar Position flyout with Default, Top, Bottom and Hidden. [stock only]
  3. Choosing a value in the Activity Bar Position flyout moves the activity bar to match `workbench.activityBar.location`. [stock only]
  4. In Cursor neither row is in the Appearance submenu, although the setting `workbench.activityBar.location` is still registered. [stock only]

#### TB-1.32 Extra Go rows for adding a symbol to chat

- Tag: cursor. Evidence: bundle. Boundary: yes - both rows hand the symbol under the cursor to the chat panel, which is an agent surface. Cursor: cursor only
- What: Cursor adds two rows to the Go menu in the symbol group (orders 5 and 6), Add Symbol to Current Chat and Add Symbol to New Chat; in the menu each shows the shortcut of a chat command (the existing new-chat shortcut and the new agent chat shortcut) because it has none of its own.
- Commands: editor.action.addSymbolToChat; editor.action.addSymbolToNewChat
- Menus: MenubarGoMenu
- Keys: none found
- Settings: none found
- Source: C bundle (both ids found; alias map from them to `aichat.newchataction` and `composer.newAgentChat`); ui\entry-points\01-title-bar.md (MenubarGoMenu table, last two rows)
- Pieces:
  1. The Go menu has an Add Symbol to Current Chat row in the symbol group. [cursor only]
  2. The Go menu has an Add Symbol to New Chat row in the symbol group, right after the first one. [cursor only]
  3. Add Symbol to Current Chat shows the shortcut of the existing new chat command, and Add Symbol to New Chat shows the shortcut of the new agent chat command, because neither has a shortcut of its own. [cursor only]
  4. Choosing either row hands the symbol under the cursor to the chat panel. [cursor only]


## TB-2 Command center and window title

(both). The search-style box in the middle with back and forward arrows, or just the title text, plus the title bar right-click menu. Examples: the search box; back and forward; the `window.title` template; right-click toggles.

16 features.

#### TB-2.1 Open Quick Open from the command center box

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - present, but `window.commandCenter` defaults to false in Cursor (true in stock), and the click always opens classic Quick Open
- What: Click the search-style box in the middle of the title bar to open Quick Open with the help entry included. In stock, when the chat setting `chat.unifiedAgentsBar.enabled` is on, the click opens the unified quick access instead (chat feature, not cataloged). The box is on by default in stock and off by default in Cursor.
- Commands: workbench.action.quickOpenWithModes
- Menus: CommandCenterCenter (order 100); CommandCenter (submenu titled "Command Center", order 101); CommandCenterCenter (order 100, search icon); CommandCenter (submenu "Command Center", order 101)
- Keys: none found
- Settings: window.commandCenter
- Source: V\src\vs\workbench\browser\parts\titlebar\commandCenterControl.ts; V\src\vs\workbench\browser\actions\quickAccessActions.ts; C bundle (modules `commandCenterControl` and `quickAccessActions`, registration of `window.commandCenter`); V\src\vs\workbench\browser\parts\titlebar\commandCenterControl.ts (93-110, 252-270); V\src\vs\workbench\browser\actions\quickAccessActions.ts (154-196); V\src\vs\workbench\browser\workbench.contribution.ts (860-867); C bundle (action registered on `CommandCenterCenter` with title string 3227, setting block with default false, hover strings 3993 to 3995)
- Pieces:
  1. Clicking the search-style box in the middle of the title bar opens Quick Open with the help entry included.
  2. Setting `window.commandCenter` defaults to on in stock, so the box shows without any setup. [stock only]
  3. Setting `window.commandCenter` defaults to off in Cursor, so the box is not shown until it is turned on. [cursor only]
  4. In Cursor the click always opens classic Quick Open. [cursor only]
  5. Clicking the search-style box in the middle of the title bar runs `workbench.action.quickOpenWithModes` and opens Quick Open with the shortcut rows above the recent files.
  6. Hovering the box shows a tip that reads "Search <workspace name> (<key>)" followed by the window title, and the key part is left out when the command has no key.
  7. The box is exposed to screen readers as a button whose description is that same tip text.
  8. Setting `window.commandCenter` shows or hides the box in the title bar, with values true and false, and it defaults to true in stock.
  9. In Cursor `window.commandCenter` defaults to false so the box stays hidden until you turn it on, and a click on the box always opens classic Quick Open. [cursor only]

#### TB-2.2 Show or hide the command center

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - setting present with default false; the right-click row and the id `toggle.window.commandCenter` are not in the bundle (the `titlebarActions` module has no Command Center string)
- What: Turn the center box on or off. Off collapses the middle of the bar to plain window title text and lowers the bar from 35 px to 30 px. In stock, turning on Navigation Controls or the experimental Share setting turns the box on if it was off. Compact floating editor windows never draw the box.
- Commands: toggle.window.commandCenter
- Menus: TitleBarContext; TitleBarTitleContext (group 2_config, order 1, shown when the title bar is not compact)
- Keys: none found
- Settings: window.commandCenter
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\browser\layout.ts; V\src\vs\workbench\services\layout\browser\layoutService.ts; C bundle (settings block for `window.commandCenter`, module `vs/workbench/browser/parts/titlebar/titlebarActions`)
- Pieces:
  1. Setting `window.commandCenter` turned on shows the search-style box in the middle of the title bar, and turned off collapses the middle of the bar to plain window title text.
  2. Turning the box off lowers the title bar from 35 px to 30 px.
  3. The right-click row Command Center (`toggle.window.commandCenter`) flips the setting, shows a check mark while the box is on, and appears when the title bar is not compact. [stock only]
  4. In stock, turning on Navigation Controls or the experimental Share setting turns the box on if it was off.
  5. Compact floating editor windows never draw the box.

#### TB-2.3 Command center label and hover text

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same label logic and strings (`label.dfl`, `title`, `title2`); stock's compact look (icon hidden, label left) depends on `chat.agentsControl.enabled`, which is not in the bundle, so Cursor always shows the search icon
- What: The box shows the workspace name. With editor tabs hidden it shows the active file name instead. With a customized `window.title` or a screen reader on, it shows the whole formatted title. If there is nothing to show it says "Search". The extension-development and admin markers are added before and after the label, and line breaks show as a return symbol. Hover reads "Search {workspace} ({key})" then an em dash and the title; the key part only appears when Quick Open has a binding. Colors come from the `commandCenter.*` theme colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.title; window.titleSeparator; workbench.editor.showTabs
- Source: V\src\vs\workbench\browser\parts\titlebar\commandCenterControl.ts; V\src\vs\workbench\browser\parts\titlebar\windowTitle.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (module `commandCenterControl`, strings 3990-3994, label function in the Quick Open view item class)
- Pieces:
  1. The box shows the workspace name.
  2. With editor tabs hidden (`workbench.editor.showTabs` set to none) the box shows the active file name instead of the workspace name.
  3. With a customized `window.title` or with a screen reader on, the box shows the whole formatted title.
  4. When there is nothing to show, the box reads "Search".
  5. The extension-development marker is added before the label and the admin marker after it.
  6. Line breaks in the label show as a return symbol.
  7. Hovering the box shows "Search {workspace} ({key})" followed by a long dash and the title, and the key part appears only when Quick Open has a keybinding.
  8. The box colors come from the `commandCenter.*` theme colors.
  9. In Cursor the box always shows a search icon before the label.

#### TB-2.4 Go back and forward arrows

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - arrows exist on the command center menu; when the box is off, Cursor draws the same two commands as buttons in its own left title bar toolbar (strings 4040-4041, enabled by the same context keys)
- What: Two arrow buttons to the left of the box step back and forward through editor navigation history. Each is disabled when there is nowhere to go (context keys `canNavigateBack` and `canNavigateForward`). In stock they live on the command center menu, so they show only while the box is on.
- Commands: workbench.action.navigateBack; workbench.action.navigateForward
- Menus: CommandCenter (Back order 1, Forward order 2, when `config.workbench.navigationControl.enabled`); MenubarGoMenu (group 1_history_nav)
- Keys: Back - Alt+Left (Windows), Ctrl+- (macOS), Ctrl+Alt+- (Linux), plus the browser Back key; Forward - Alt+Right (Windows), Ctrl+Shift+- (macOS and Linux), plus the browser Forward key
- Settings: workbench.navigationControl.enabled
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts; C bundle (editor actions module with the `CommandCenter` menu rows for `workbench.action.navigateBack`, title bar part `createActionForId`)
- Pieces:
  1. Two arrow buttons to the left of the box step back and forward through editor navigation history.
  2. Each arrow is disabled when there is nowhere to go, following the context keys `canNavigateBack` and `canNavigateForward`.
  3. Back runs from the keyboard with Alt+Left on Windows, Ctrl+- on macOS, Ctrl+Alt+- on Linux, and the browser Back key.
  4. Forward runs from the keyboard with Alt+Right on Windows, Ctrl+Shift+- on macOS and Linux, and the browser Forward key.
  5. The Go menu also has Back and Forward rows in its history group.
  6. In stock the arrows live on the command center menu, so they show only while the box is on and `workbench.navigationControl.enabled` is on.
  7. In Cursor, when the box is off, the same two commands show as buttons in Cursor's left title bar toolbar and use the same enabled and disabled rules. [cursor only]

#### TB-2.5 Show or hide navigation controls

- Tag: both. Evidence: source. Boundary: no. Cursor: present - generic toggle class builds the id as `toggle.` plus the setting name; row text in strings 4014-4015; row shows when `config.window.commandCenter` is on; default of the setting is true in both
- What: Hide or show the back and forward arrows from the title bar right-click menu or the setting. The row appears only when the title bar is not compact and the command center is on. In Cursor the same setting also gates the arrows that Cursor puts in its left toolbar when the box is off.
- Commands: toggle.workbench.navigationControl.enabled
- Menus: TitleBarContext; TitleBarTitleContext (group 2_config, order 2)
- Keys: none found
- Settings: workbench.navigationControl.enabled
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (module `vs/workbench/browser/parts/titlebar/titlebarActions`, setting registration)
- Pieces:
  1. The right-click row Navigation Controls hides or shows the back and forward arrows.
  2. The Navigation Controls row shows a check mark while the arrows are on.
  3. The row appears only when the title bar is not compact and the command center is on.
  4. Setting `workbench.navigationControl.enabled` does the same from settings, and it defaults to on.
  5. In Cursor the same setting also gates the arrows in Cursor's left toolbar when the box is off. [cursor only]

#### TB-2.6 Title bar right-click menu

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - present, with different rows: it adds Agents Window Button (agent, not cataloged) and, on Linux only, Restore Native Title Bar; it has no Command Center, Notifications, Integrated Browser, Hide Custom Title Bar or Hide Custom Title Bar In Full Screen row
- What: Right-click anywhere on the title bar, including on the box and on toolbar buttons, to open a menu of title bar toggles. On Windows with a custom title bar, the OS system-menu request is redirected to this same menu. Each row is its own feature in the group that owns the thing toggled (see Handed off); this feature is the menu itself.
- Commands: none found
- Menus: TitleBarContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\commandCenterControl.ts; C bundle (`TitleBarContext` 13 hits, `onDidTriggerWindowSystemContextMenu`, module `titlebarActions`)
- Pieces:
  1. Right-clicking anywhere on the title bar opens a menu of title bar toggles.
  2. Right-clicking on the box or on a toolbar button in the title bar opens the same menu.
  3. On Windows with a custom title bar, the OS system-menu request opens this same menu.
  4. The toggle rows sit together in one section, and the Hide Custom Title Bar rows sit in a separate section below it.
  5. In Cursor the menu has no Command Center, Notifications, Integrated Browser, Hide Custom Title Bar or Hide Custom Title Bar In Full Screen row. [stock only]

#### TB-2.7 Title right-click menu with folder path (macOS)

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same menu id, same generated command ids and Cmd+click handler found in the bundle
- What: On macOS, right-click on the title text, or Cmd+click on it, opens a second menu with the same toggles plus one row per folder in the active file's path and one for the file. Picking a row reveals it in Finder. The path rows appear only with a custom title bar and an active file. Cmd+click is caught first so it does not open Quick Open.
- Commands: workbench.action.revealPathInFinder<n> (one id generated per path segment, n is the segment index)
- Menus: TitleBarTitleContext (path rows in group 1_file, negative orders)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\electron-browser\window.ts; C bundle (`TitleBarTitleContext` 9 hits, `workbench.action.revealPathInFinder` generator)
- Pieces:
  1. On macOS, right-clicking the title text opens a second menu with the same toggles plus one row for each folder in the active file's path and one row for the file.
  2. On macOS, Cmd+click on the title text opens the same second menu.
  3. Picking a path row reveals that folder or file in Finder.
  4. The path rows appear only with a custom title bar and an active file.
  5. Cmd+click on the title text does not open Quick Open.

#### TB-2.8 Window title as plain text when the command center is off

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - when the box is off and a workspace is open, Cursor shows the workspace name as a button (see TB-2.9); with no workspace it shows the window title text or the product name
- What: With the box off, the middle of the bar shows the window title string as centered plain text, clipped with an ellipsis, and it updates as you switch editors. With a native title bar the custom bar draws no title text because the OS draws it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.commandCenter; window.title
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part `createTitle` override)
- Pieces:
  1. With the box off, the middle of the title bar shows the window title string as centered plain text.
  2. Title text that is too long is clipped with an ellipsis.
  3. The title text updates as the user switches editors.
  4. With a native title bar, the custom bar draws no title text because the OS draws it.
  5. In Cursor with a workspace open, the plain title text is replaced by a workspace name button (see TB-2.9). [cursor only]

#### TB-2.9 Click the workspace name to open Quick Open

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: In Cursor with the command center off and a workspace open, the middle of the bar is a text button showing the workspace name (or the full title when `window.title` is customized). A left-button press, Enter or Space runs Quick Open. The button is excluded from window dragging. Its class name `agent-workspace-quickopen` hints at the agent layout, but the action is stock Quick Open, so it is not marked boundary.
- Commands: workbench.action.quickOpenWithModes
- Menus: none found
- Keys: Enter or Space while the button has focus; Enter or Space (while the button has focus)
- Settings: window.commandCenter; window.title
- Source: C bundle (title bar part `createTitle`, button class `window-title-text agent-workspace-quickopen`); C bundle (title bar part `createTitle`, button class `window-title-text agent-workspace-quickopen`, mouse down and key down handlers); V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts (591-603, stock sets plain title text here)
- Pieces:
  1. With the box off and a workspace open, the middle of the title bar is a text button showing the workspace name. [cursor only]
  2. When `window.title` is customized, the button shows the full title instead of the workspace name. [cursor only]
  3. A left-button press on the button opens Quick Open. [cursor only]
  4. Pressing Enter or Space while the button has focus opens Quick Open. [cursor only]
  5. The button is excluded from window dragging, so a click on it never starts a drag. [cursor only]
  6. With no workspace open the button is not shown, and the plain title text or product name appears instead. [cursor only]
  7. A left click on the button opens Quick Open with the shortcut rows, and it fires on mouse down. [cursor only]
  8. The button reads the workspace name, or the full window title when `window.title` has been set to a custom format. [cursor only]
  9. The button text is also its screen reader name, and pressing on it opens Quick Open instead of starting a window drag. [cursor only]
  10. With no workspace open the middle of the title bar shows plain title text that does nothing when clicked. [cursor only]

#### TB-2.10 Change the window title template

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same variable set and same default template code
- What: The `window.title` template sets the text in the OS window title (taskbar, Alt-Tab, Mission Control) and the title text in the bar. Variables: activeEditorShort, activeEditorMedium, activeEditorLong, activeEditorLanguageId, activeFolderShort, activeFolderMedium, activeFolderLong, folderName, folderPath, rootName, rootNameShort, rootPath, profileName, appName, remoteName, dirty, focusedView, activeEditorState, separator. The default on Windows and Linux is dirty, active file, root name, profile name, app name joined by separators; on macOS desktop it leaves out dirty and app name. An empty result falls back to the product name.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.title
- Source: V\src\vs\workbench\browser\parts\titlebar\windowTitle.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (module `vs/workbench/browser/parts/titlebar/windowTitle`, `getWindowTitle` and the default template)
- Pieces:
  1. Setting `window.title` sets the text in the OS window title, which shows in the taskbar, Alt-Tab and Mission Control.
  2. The same template sets the title text in the title bar.
  3. The template accepts the editor variables `${activeEditorShort}`, `${activeEditorMedium}`, `${activeEditorLong}` and `${activeEditorLanguageId}`.
  4. The template accepts the folder variables `${activeFolderShort}`, `${activeFolderMedium}`, `${activeFolderLong}`, `${folderName}` and `${folderPath}`.
  5. The template accepts the workspace variables `${rootName}`, `${rootNameShort}` and `${rootPath}`.
  6. The template accepts the state variables `${profileName}`, `${appName}`, `${remoteName}`, `${dirty}`, `${focusedView}`, `${activeEditorState}` and `${separator}`.
  7. On Windows and Linux the default template shows dirty, active file, root name, profile name and app name joined by separators.
  8. On macOS desktop the default template leaves out the dirty marker and the app name.
  9. An empty result falls back to the product name.

#### TB-2.11 Change the window title separator

- Tag: both. Evidence: source. Boundary: no. Cursor: present - setting registered with the same default (4 hits)
- What: `window.titleSeparator` sets the text used by the `${separator}` variable. The default is a hyphen with spaces, and an em dash on macOS. The separator only shows between values that are not empty.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.titleSeparator
- Source: V\src\vs\workbench\browser\parts\titlebar\windowTitle.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (settings block, string 4276)
- Pieces:
  1. Setting `window.titleSeparator` sets the text that the `${separator}` variable shows in the title.
  2. On Windows and Linux the default separator is a hyphen with a space on each side.
  3. On macOS the default separator is an em dash with a space on each side.
  4. The separator shows only between values that are not empty.

#### TB-2.12 Show the active repository and branch in the window title

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the SCM activity module registers both variables with the title service
- What: The variables `${activeRepositoryName}` and `${activeRepositoryBranchName}` fill from the active Source Control repository, so the title can show the repo and branch. The SCM contribution registers them with the title service through the same internal command extensions can call (`registerWindowTitleVariable`, extension system, not cataloged).
- Commands: registerWindowTitleVariable
- Menus: none found
- Keys: none found
- Settings: window.title
- Source: V\src\vs\workbench\contrib\scm\browser\activity.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; C bundle (SCM activity module, `registerVariables` call)
- Pieces:
  1. The variable `${activeRepositoryName}` shows the name of the active Source Control repository.
  2. The variable `${activeRepositoryBranchName}` shows the branch name of the active Source Control repository.
  3. Adding both variables to `window.title` makes the OS window title and the title bar text show the repository and branch.

#### TB-2.13 See unsaved changes and run mode in the window title

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the three marker strings are in module `windowTitle` (4043-4045)
- What: The default title starts with a filled dot when the active editor has unsaved changes (not on macOS desktop, which shows its own indicator). The title is prefixed with "[Extension Development Host]" in an extension development window and suffixed with "[Administrator]" on Windows or "[Superuser]" elsewhere when running elevated. Non-space whitespace in the title is replaced with spaces.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.title
- Source: V\src\vs\workbench\browser\parts\titlebar\windowTitle.ts; C bundle (module `vs/workbench/browser/parts/titlebar/windowTitle`)
- Pieces:
  1. The default title starts with a filled dot when the active editor has unsaved changes.
  2. On macOS desktop the default title shows no dot, because macOS shows its own indicator.
  3. In an extension development window the title is prefixed with "[Extension Development Host]".
  4. When the app runs elevated, the title ends with "[Administrator]" on Windows and "[Superuser]" elsewhere.
  5. Non-space whitespace in the title is replaced with spaces.

#### TB-2.14 Screen reader friendly window title

- Tag: both. Evidence: source. Boundary: no. Cursor: present - setting found (4 hits) and the append logic is in the same `getWindowTitle` code
- What: When a screen reader is detected and `accessibility.windowTitleOptimized` is on, the title gets a separator and the active editor state (for example "modified") appended, unless the template already has it. The command center then shows the full title instead of the workspace name.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: accessibility.windowTitleOptimized; window.title
- Source: V\src\vs\workbench\browser\parts\titlebar\windowTitle.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts; C bundle (module `windowTitle`)
- Pieces:
  1. With a screen reader detected and `accessibility.windowTitleOptimized` on, the title gets a separator and the active editor state, for example "modified", added at the end.
  2. The state is not added again when the `window.title` template already has `${activeEditorState}`.
  3. With `accessibility.windowTitleOptimized` off, the title gets no extra state.
  4. In this case the command center shows the full title instead of the workspace name.

#### TB-2.15 Command center hides while Quick Open is at the top

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the box hides on every quick input show in the active window, with no alignment check, and returns on hide
- What: When a quick input opens at the top of the window, the box is hidden (it keeps its space, so nothing shifts) so it does not sit behind the input. It returns when the input closes. In stock it stays visible when the quick input is centered.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\commandCenterControl.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (command center control class with `onShow` and `onHide` listeners)
- Pieces:
  1. When a quick input opens at the top of the window, the box is hidden but keeps its space, so nothing shifts.
  2. The box returns when the quick input closes.
  3. In stock the box stays visible when the quick input is centered. [stock only]
  4. In Cursor the box hides on every quick input in the active window, with no check on where the input appears. [cursor only]

#### TB-2.16 Share button in the command center (experimental)

- Tag: vscode. Evidence: source. Boundary: no. Cursor: changed - the command, the key and the setting (default false) exist, but the action has no `CommandCenter` menu row, so Cursor has no button
- What: With `workbench.experimental.share.enabled` on, a share provider available and a folder open, a Share icon appears in the command center. It builds a link to the active file or selection, copies it, and shows it in a dialog.
- Commands: workbench.action.share
- Menus: CommandCenter (order 3)
- Keys: Alt+Ctrl+S (Alt+Cmd+S on macOS)
- Settings: workbench.experimental.share.enabled
- Source: V\src\vs\workbench\contrib\share\browser\share.contribution.ts; C bundle (share action constructor with `f1` and keybinding but no `menu`, setting registration)
- Pieces:
  1. With `workbench.experimental.share.enabled` on, a share provider available and a folder open, a Share icon appears in the command center. [stock only]
  2. Clicking the Share icon builds a link to the active file or selection and copies it. [stock only]
  3. The link is shown in a dialog after it is copied. [stock only]
  4. Pressing Alt+Ctrl+S, or Alt+Cmd+S on macOS, runs Share from the keyboard.
  5. In Cursor the Share command, its shortcut and the setting still exist, but no Share icon appears in the command center. [stock only]


## TB-3 Window frame and controls

(both). Native or drawn title bar, minimize, maximize and close, the app icon, the drag area, zoom and full screen. Examples: `window.titleBarStyle`; `window.controlsStyle`; double-click the icon to close; Restore Native Title Bar (Cursor).

23 features.

#### TB-3.1 Choose a native or custom title bar

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - present with default custom; the setting text is older (it says the style also changes the menu look) and `window.menuStyle` is not in the bundle
- What: `window.titleBarStyle` picks the OS title bar (native) or one drawn by the workbench (custom, the default) that carries the menu bar, command center, toolbar buttons and window controls. Changes need a full restart. On macOS, native tabs or turning native full screen off force the native style. In stock the style also decides whether menus and context menus are drawn by the app or the OS (through `window.menuStyle`, owned by TB-1).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.titleBarStyle
- Source: V\src\vs\platform\window\common\window.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; C bundle (module `vs/workbench/electron-sandbox/desktop.contribution`, string 12061; settings block default `custom`)
- Pieces:
  1. Setting `window.titleBarStyle` chooses the title bar, with `custom` (default) drawn by the workbench to carry the menu bar, command center, toolbar buttons and window controls, and `native` drawn by the OS.
  2. Changing `window.titleBarStyle` only takes effect after a full restart of the app.
  3. On macOS, turning on `window.nativeTabs` or turning off `window.nativeFullScreen` forces the native title bar no matter what the setting says.
  4. In stock, setting `window.menuStyle` to `inherit` makes menus and context menus follow the title bar style, app-drawn with `custom` and OS-drawn with `native`. [stock only]
  5. In Cursor the `window.titleBarStyle` description still says the style also changes the menu look, and there is no `window.menuStyle` setting. [cursor only]

#### TB-3.2 Choose native, drawn or hidden window controls

- Tag: both. Evidence: source. Boundary: no. Cursor: present - setting registered with default `native`, hidden on macOS; the `updateWindowControls` call is in the bundle
- What: Windows and Linux only, with the custom title bar. `native` (default) lets the OS draw minimize, maximize and close over the bar (the window controls overlay); the bar sends the OS its background color, text color and height so they match the theme and the zoom. `custom` has the workbench draw them (TB-3.3). `hidden` removes them. Changes need a full restart. The setting is not offered on macOS and is ignored with a native title bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.controlsStyle
- Source: V\src\vs\platform\window\common\window.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; C bundle (settings block, title bar part, `updateWindowControls` 6 hits)
- Pieces:
  1. On Windows and Linux with the custom title bar, `window.controlsStyle` set to `native` (default) has the OS draw minimize, maximize and close over the right end of the bar.
  2. Setting `window.controlsStyle` to `custom` has the workbench draw the three window buttons itself (TB-3.3).
  3. Setting `window.controlsStyle` to `hidden` removes the window buttons from the bar.
  4. With `native` controls, the bar passes the OS its background color, text color and height, so the OS-drawn buttons match the theme and follow the window zoom.
  5. Changing `window.controlsStyle` only takes effect after a full restart of the app.
  6. `window.controlsStyle` is not offered on macOS and is ignored when the title bar style is `native`.

#### TB-3.3 Minimize, maximize or restore, and close with the drawn buttons

- Tag: both. Evidence: source. Boundary: no. Cursor: present - classes `window-minimize`, `window-max-restore`, `window-close` and the red close hover rule are in the bundle and the CSS
- What: With `custom` controls, three 46 px buttons sit at the right end of the bar on Windows and Linux: minimize, maximize (which becomes restore while maximized), and close (red on hover). They are hidden in full screen, keep a fixed pixel size when the window is zoomed, and tint on hover to suit light or dark title bars.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.controlsStyle
- Source: V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part, workbench CSS)
- Pieces:
  1. With `window.controlsStyle` set to `custom`, three buttons 46 px wide (minimize, maximize, close) sit at the right end of the title bar on Windows and Linux.
  2. Clicking the minimize button minimizes the window.
  3. Clicking the maximize button maximizes the window and the icon turns into a restore icon, and clicking restore returns the window to its earlier size.
  4. Clicking the close button closes the window, and the close button turns red when the pointer is over it.
  5. The minimize and maximize buttons tint on hover in a way that suits a light or a dark title bar.
  6. The three buttons are hidden while the window is in full screen.
  7. The buttons keep a fixed pixel size when the window is zoomed in or out.

#### TB-3.4 Close the window by keyboard or File menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present - command id found (3 hits), menu title `miCloseWindow` in module `electron-sandbox/actions/windowActions`
- What: Close Window closes the active window (main or floating). On macOS, Cmd+W also closes the window when no editor is open and there is one editor group. The confirm prompt belongs to DL-2.
- Commands: workbench.action.closeWindow
- Menus: MenubarFileMenu (group 6_close, order 4); CommandPalette
- Keys: Alt+F4 and Ctrl+Shift+W (Windows and Linux); Cmd+Shift+W (macOS); Cmd+W (macOS, when no editor is visible and there is a single group)
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; C bundle (module `vs/workbench/electron-sandbox/actions/windowActions`, string 12018)
- Pieces:
  1. Running Close Window (`workbench.action.closeWindow`) closes the active window, whether it is the main window or a floating editor window.
  2. The File menu has a Close Window row (group `6_close`), and the Command Palette lists the same command.
  3. Alt+F4 and Ctrl+Shift+W close the window on Windows and Linux, and Cmd+Shift+W closes it on macOS.
  4. On macOS, Cmd+W also closes the window when no editor is visible and there is a single editor group.

#### TB-3.5 macOS window buttons at the left

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the left or right placement code with the right-to-left check is in the title bar part; the 28 and 32 px heights were not searched
- What: On macOS with the custom title bar, the traffic light buttons stay at the left (the OS draws them) and move to the right in right-to-left locales. The workbench adds no container for them, so dragging works right up to the buttons. Bar height is 28 px, 32 px on macOS Tahoe or newer, or 35 px with the command center.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\platform\window\common\window.ts; C bundle (title bar part, `textInfo` direction check)
- Pieces:
  1. On macOS with the custom title bar, the traffic light buttons stay at the left end of the bar, drawn by the OS.
  2. In right-to-left locales on macOS, the traffic light buttons move to the right end of the bar.
  3. The workbench adds no container around the traffic light buttons, so dragging the bar works right up to them.
  4. On macOS the bar is 28 px high, or 32 px high on macOS Tahoe or newer.
  5. On macOS the bar is 35 px high when the command center is shown.

#### TB-3.6 App icon at the left of the title bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present - `window-appicon` element and CSS found, including the full-screen hide rule
- What: With a custom title bar on Windows and Linux, an app icon 35 px wide sits at the far left, before the menu bar. It is hidden in full screen. The only behavior in code is the double-click in TB-3.7.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part, workbench CSS)
- Pieces:
  1. With the custom title bar on Windows and Linux, an app icon 35 px wide sits at the far left of the bar, before the menu bar.
  2. The app icon is hidden while the window is in full screen.
  3. The app icon only shows on Windows and Linux with the custom title bar, and not on macOS or with the native bar.

#### TB-3.7 Double-click the app icon to close the window

- Tag: both. Evidence: source. Boundary: no. Cursor: present - setting found (3 hits, default false)
- What: With `window.doubleClickIconToClose` on, a double-click on the app icon closes the window and the icon can no longer be used to drag the window. Only works with the custom title bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.doubleClickIconToClose
- Source: V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; C bundle (settings block, title bar part)
- Pieces:
  1. With `window.doubleClickIconToClose` on, double-clicking the app icon closes the window.
  2. With `window.doubleClickIconToClose` on, the app icon can no longer be used to drag the window.
  3. `window.doubleClickIconToClose` is off by default, so double-clicking the icon does nothing and the icon area drags the window like the rest of the bar.

#### TB-3.8 Drag the title bar to move the window

- Tag: both. Evidence: source. Boundary: no. Cursor: present - `titlebar-drag-region` found in the bundle and the CSS
- What: A full-size drag region sits behind the bar, so dragging empty space moves the window, while buttons, the menu bar and the command center are no-drag. The region is hidden briefly while the Alt-toggled menu bar shows or has focus on Windows and Linux (an Electron issue, #52522). The workbench has no double-click handler on the bar itself, so double-click to maximize is left to the OS.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part, workbench CSS)
- Pieces:
  1. Dragging empty space on the title bar moves the window.
  2. Buttons, the menu bar and the command center do not start a window drag, so pressing them clicks them instead.
  3. On Windows and Linux the drag area is turned off for a moment while the Alt-toggled menu bar is showing or has focus.
  4. The workbench has no double-click handler on the bar, so double-clicking empty space does whatever the OS does for a title bar double-click, such as maximize.

#### TB-3.9 Resize the window from the top edge of the custom frame

- Tag: both. Evidence: source. Boundary: no. Cursor: present - `div.resizer` created in the title bar part and `.resizer` rules in the CSS
- What: On Windows and Linux with drawn window controls, a 4 px no-drag strip at the top of the bar lets the pointer grab the window's top edge to resize it. It is hidden while the window is maximized and in full screen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.controlsStyle
- Source: V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part, workbench CSS)
- Pieces:
  1. On Windows and Linux with drawn window controls, a 4 px strip at the very top of the title bar lets the pointer grab the top edge of the window to resize it.
  2. The top resize strip is hidden while the window is maximized.
  3. The top resize strip is hidden while the window is in full screen.
  4. The top resize strip exists only when `window.controlsStyle` is `custom`.

#### TB-3.10 Hide or show the custom title bar

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - no hits for the setting or for `showCustomTitleBar`, `hideCustomTitleBar` or `toggle.toggleCustomTitleBar`
- What: With the native title bar on, `window.customTitleBarVisibility` set to `never` removes the custom bar (menu bar, command center, toolbar buttons) and `auto` shows it. The setting flips back from `never` to `auto` when something that lives in the bar is turned on (command center, layout controls, activity bar at top or bottom, editor actions in the title bar). The right-click row shows only with the native style; the Appearance menu row is owned by TB-1's menu.
- Commands: showCustomTitleBar; hideCustomTitleBar; toggle.toggleCustomTitleBar; toggle.window.customTitleBarVisibility
- Menus: MenubarAppearanceMenu (group 2_workbench_layout, order 6); TitleBarContext; TitleBarTitleContext (group 3_toggle, order 0)
- Keys: none found
- Settings: window.customTitleBarVisibility
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts; V\src\vs\workbench\services\layout\browser\layoutService.ts; V\src\vs\workbench\browser\layout.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; C bundle (searched, none found)
- Pieces:
  1. With the native title bar on, setting `window.customTitleBarVisibility` to `never` removes the custom bar (menu bar, command center, toolbar buttons), and `auto` (default) shows it. [stock only]
  2. Show Custom Title Bar (`showCustomTitleBar`) and Hide Custom Title Bar (`hideCustomTitleBar`) in the Command Palette set the value to `auto` or `never`, and each is available only while the bar is hidden or shown. [stock only]
  3. With the native style, right-clicking the title bar shows a Hide Custom Title Bar row (`toggle.window.customTitleBarVisibility`, group `3_toggle`), and the row is not offered with the custom style. [stock only]
  4. The View, Appearance menu has a Custom Title Bar row (`toggle.toggleCustomTitleBar`) that shows a check while the bar is visible and flips it between shown and hidden. [stock only]
  5. The setting flips back from `never` to `auto` when something that lives in the bar is turned on, such as the command center, layout controls, the activity bar at top or bottom, or editor actions in the title bar. [stock only]

#### TB-3.11 Hide the custom title bar in full screen only

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - no hits for the setting or either id
- What: Setting `window.customTitleBarVisibility` to `windowed` hides the custom bar in full screen and keeps normal behavior otherwise. The command and the right-click row are offered only while in full screen. The visibility check returns early for this value, so the bar stays hidden in full screen even if the menu bar is toggled on.
- Commands: hideCustomTitleBarInFullScreen; toggle.window.customTitleBarVisibility.windowed
- Menus: TitleBarContext; TitleBarTitleContext (group 3_toggle, order 1, when `isFullscreen`)
- Keys: none found
- Settings: window.customTitleBarVisibility
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts; V\src\vs\workbench\services\layout\browser\layoutService.ts; V\src\vs\workbench\browser\layout.ts; C bundle (searched, none found)
- Pieces:
  1. Setting `window.customTitleBarVisibility` to `windowed` hides the custom title bar while in full screen and keeps the normal behavior in a regular window. [stock only]
  2. Hide Custom Title Bar In Full Screen (`hideCustomTitleBarInFullScreen`) in the Command Palette sets the value to `windowed`, and it is available only in full screen while the bar is shown. [stock only]
  3. Right-clicking the title bar while in full screen shows a Hide Custom Title Bar In Full Screen row (`toggle.window.customTitleBarVisibility.windowed`, group `3_toggle`), and the row does not show outside full screen. [stock only]
  4. With `windowed`, the custom bar stays hidden in full screen even if the menu bar is toggled on. [stock only]
  5. With `windowed`, running the Appearance menu Custom Title Bar row in full screen sets `auto` and shows the bar, while running it in a regular window sets `never`. [stock only]

#### TB-3.12 Restore the native title bar (Cursor)

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A row on the title bar right-click menu sets `window.titleBarStyle` back to `native`. It shows only while the style is custom. By the two platform flags that guard it, it is registered on Linux desktop only: they match isLinux and isNative in the same bundle (compare the `included` rule of `window.menuBarVisibility` and the macOS right-to-left check).
- Commands: toggle.window.titleBarStyle
- Menus: TitleBarContext; TitleBarTitleContext (group 4_restore_native_title, order 0, when the `titleBarStyle` context key is `custom`)
- Keys: none found
- Settings: window.titleBarStyle
- Source: C bundle (module `vs/workbench/browser/parts/titlebar/titlebarActions`, string 4021, registration guarded by the two platform flags)
- Pieces:
  1. Right-clicking the title bar shows a row (`toggle.window.titleBarStyle`, group `4_restore_native_title`) that sets `window.titleBarStyle` back to `native`. [cursor only]
  2. The restore row shows only while the title bar style is `custom`. [cursor only]
  3. The restore row is registered only on Linux desktop, so it does not show on Windows or macOS. [cursor only]

#### TB-3.13 Toggle full screen

- Tag: both. Evidence: source. Boundary: no. Cursor: present - command id found (5 hits), title and menu title strings 3256 and 3263, `window.restoreFullscreen` found; `window.customTitleBarVisibility` is missing in Cursor
- What: Switch the window in and out of full screen. In full screen the app icon, window controls and top resize strip are hidden, and with the classic menu bar mode the custom title bar hides until the menu bar is toggled (macOS hides the title bar in full screen). With `window.restoreFullscreen` on, a window that was in full screen when closed reopens in full screen.
- Commands: workbench.action.toggleFullScreen
- Menus: MenubarAppearanceMenu (group 1_toggle_view, order 1); CommandPalette
- Keys: F11; Ctrl+Cmd+F (macOS)
- Settings: window.restoreFullscreen; window.customTitleBarVisibility
- Source: V\src\vs\workbench\browser\actions\windowActions.ts; V\src\vs\workbench\services\layout\browser\layoutService.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; V\src\vs\workbench\electron-browser\desktop.contribution.ts; C bundle (module `vs/workbench/browser/actions/windowActions`)
- Pieces:
  1. Running Toggle Full Screen (`workbench.action.toggleFullScreen`) puts the window into full screen and running it again takes the window back out.
  2. F11, Ctrl+Cmd+F on macOS, the View, Appearance menu Full Screen row (group `1_toggle_view`) and the Command Palette all run Toggle Full Screen.
  3. In full screen the app icon, the window controls and the top resize strip are hidden.
  4. In full screen with the classic menu bar mode, the custom title bar stays hidden until the menu bar is toggled, and on macOS the title bar is hidden in full screen.
  5. With `window.restoreFullscreen` on, a window that was in full screen when it was closed reopens in full screen.
  6. In stock, `window.customTitleBarVisibility` set to `windowed` hides the custom title bar in full screen only (TB-3.11), and Cursor does not have that setting. [stock only]

#### TB-3.14 Use macOS native full screen

- Tag: both. Evidence: source. Boundary: no. Cursor: present - setting found (2 hits, default true, macOS only)
- What: macOS only. `window.nativeFullScreen` (default on) uses the macOS full screen Space. Turning it off uses simple full screen without a new Space, and that forces the native title bar style. Native window tabs force native full screen on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.nativeFullScreen
- Source: V\src\vs\platform\window\common\window.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; C bundle (settings block, string 12065)
- Pieces:
  1. On macOS, `window.nativeFullScreen` on (default) makes full screen use the macOS full screen Space.
  2. On macOS, turning `window.nativeFullScreen` off makes full screen simple full screen, with no new Space.
  3. Turning `window.nativeFullScreen` off forces the native title bar style.
  4. Turning on native window tabs (`window.nativeTabs`) forces native full screen on.

#### TB-3.15 macOS native window tabs

- Tag: both. Evidence: source. Boundary: no. Cursor: present - all six ids and the setting found, titles in strings 12089-12094
- What: macOS only. With `window.nativeTabs` on, windows join a shared native tab bar and the native title bar style is forced. A restart is needed. The six tab commands show in the Command Palette only while the setting is on.
- Commands: workbench.action.newWindowTab; workbench.action.showPreviousWindowTab; workbench.action.showNextWindowTab; workbench.action.moveWindowTabToNewWindow; workbench.action.mergeAllWindowTabs; workbench.action.toggleWindowTabsBar
- Menus: CommandPalette (when `config.window.nativeTabs` is true)
- Keys: none found
- Settings: window.nativeTabs
- Source: V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\platform\window\common\window.ts; C bundle (module `vs/workbench/electron-sandbox/desktop.contribution`)
- Pieces:
  1. On macOS, setting `window.nativeTabs` on makes windows join a shared native tab bar.
  2. Turning on `window.nativeTabs` forces the native title bar style and needs a full restart.
  3. New Window Tab (`workbench.action.newWindowTab`) opens a new window as a tab in the native tab bar.
  4. Show Previous Window Tab (`workbench.action.showPreviousWindowTab`) and Show Next Window Tab (`workbench.action.showNextWindowTab`) move between the window tabs.
  5. Move Window Tab to New Window (`workbench.action.moveWindowTabToNewWindow`) pulls the current tab out into its own window, and Merge All Windows (`workbench.action.mergeAllWindowTabs`) joins all open windows into one tab bar.
  6. Toggle Window Tabs Bar (`workbench.action.toggleWindowTabsBar`) shows or hides the native tab bar.
  7. The six window tab commands show in the Command Palette only while `window.nativeTabs` is on, and running one with the setting off does nothing.

#### TB-3.16 Zoom the window in, out and reset

- Tag: both. Evidence: source. Boundary: no. Cursor: present - all three ids, both settings and the `counter-zoom` class found; Cursor adds a hidden `workbench.action.setZoomLevel` (no menu, no palette entry) that its Window in Window feature calls
- What: Zoom In, Zoom Out and Reset Zoom scale the whole window by one step each (a step is 20 percent). By default only the active window changes. With `window.zoomPerWindow` off the level applies to all windows and is saved in `window.zoomLevel`. Reset returns to the configured default level (or 0 when applying to all windows). The title bar counter-scales so it keeps its pixel size when it holds only non-zoomable content, and it never shrinks below the window controls when zoomed out. The status bar zoom entry is owned by SS (see Handed off).
- Commands: workbench.action.zoomIn; workbench.action.zoomOut; workbench.action.zoomReset
- Menus: MenubarAppearanceMenu (group 5_zoom, orders 1, 2 and 3); CommandPalette
- Keys: Zoom In - Ctrl+= (also Ctrl+Shift+= and Ctrl+Numpad+); Zoom Out - Ctrl+- (also Ctrl+Shift+- and Ctrl+Numpad-; on Linux Ctrl+- and Ctrl+Numpad-); Reset Zoom - Ctrl+Numpad0; Cmd replaces Ctrl on macOS
- Settings: window.zoomLevel; window.zoomPerWindow
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (module `vs/workbench/electron-sandbox/actions/windowActions`, strings 12019-12034)
- Pieces:
  1. Zoom In (`workbench.action.zoomIn`) scales the whole window up by one step of 20 percent.
  2. Zoom Out (`workbench.action.zoomOut`) scales the whole window down by one step of 20 percent.
  3. Reset Zoom (`workbench.action.zoomReset`) returns the window to the configured default zoom level, or to 0 when the level applies to all windows.
  4. The View, Appearance menu has Zoom In, Zoom Out and Reset Zoom rows (group `5_zoom`), and the Command Palette lists all three commands.
  5. Zoom In is Ctrl+= (also Ctrl+Shift+= and Ctrl+Numpad+), Zoom Out is Ctrl+- (also Ctrl+Shift+- and Ctrl+Numpad-, but only Ctrl+- and Ctrl+Numpad- on Linux), Reset Zoom is Ctrl+Numpad0, and Cmd replaces Ctrl on macOS.
  6. By default only the active window changes zoom, and with `window.zoomPerWindow` off the zoom level applies to all windows.
  7. Setting `window.zoomLevel` holds the default zoom level (0 by default, 20 percent per step), is updated with the new level when zoom applies to all windows, and is the level Reset Zoom goes back to when zoom is per window.
  8. When zoomed, the title bar counter-scales so it keeps its pixel size when it holds only non-zoomable content, and it never shrinks below the window controls when zoomed out.

#### TB-3.17 Title bar colors and dimming when the window loses focus

- Tag: both. Evidence: source. Boundary: no. Cursor: present - `titleBar.activeBackground` and its siblings are in the built-in themes and the `part.titlebar.inactive` rules are in the CSS
- What: The bar takes its colors from the `titleBar.*` theme colors (active and inactive background and foreground, and border). When the window loses focus the bar switches to the inactive colors and drops to 60 percent opacity, and the command center and window control icons follow. Light backgrounds add a `light` class that changes the control hover tint.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part, workbench CSS, theme data)
- Pieces:
  1. The title bar takes its background, text and border colors from the `titleBar.*` theme colors, using the active set while the window has focus.
  2. When the window loses focus, the bar switches to the inactive background and text colors (`titleBar.inactiveBackground` and `titleBar.inactiveForeground`).
  3. When the window loses focus, the bar also drops to 60 percent opacity.
  4. The command center and window control icons dim along with the bar when the window loses focus.
  5. A light title bar background gives the window controls a different hover tint than a dark one.

#### TB-3.18 Window border color (Windows)

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - no hits for `window.border` or `updateWindowAccentColor`; only the theme color id `window.activeBorder` appears (1 hit)
- What: On Windows, `window.border` sets the window's border color: `default` uses the theme colors `window.activeBorder` and `window.inactiveBorder` and falls back to the system color, `system` follows Windows only, `off` removes it, or a color value sets it. It is ignored with the native title bar and is refreshed when full screen changes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.border
- Source: V\src\vs\workbench\electron-browser\window.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\workbench\browser\layout.ts; C bundle (searched, only the color id found)
- Pieces:
  1. On Windows, `window.border` set to `default` (default value) uses the theme colors `window.activeBorder` and `window.inactiveBorder` for the window border, and falls back to the Windows system color. [stock only]
  2. On Windows, `window.border` set to `system` follows the Windows settings only, and `off` removes the border. [stock only]
  3. On Windows, `window.border` set to a color value in hex, RGB, RGBA, HSL or HSLA format draws the border in that color. [stock only]
  4. `window.border` is ignored when the title bar style is `native`. [stock only]
  5. The window border is refreshed when the window enters or leaves full screen. [stock only]

#### TB-3.19 Click through an inactive window (macOS)

- Tag: both. Evidence: source. Boundary: no. Cursor: present - setting found (2 hits, default true, macOS only)
- What: macOS only. With `window.clickThroughInactive` on (default), the first click on an inactive window both activates it and presses whatever is under the pointer. Off means the first click only activates. Changing it prompts for a restart.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.clickThroughInactive
- Source: V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\workbench\contrib\relauncher\browser\relauncher.contribution.ts; V\src\vs\platform\windows\electron-main\windows.ts; C bundle (settings block, string 12066)
- Pieces:
  1. On macOS, `window.clickThroughInactive` on (default) makes the first click on an inactive window both activate it and press whatever is under the pointer.
  2. On macOS, `window.clickThroughInactive` off makes the first click on an inactive window only activate it.
  3. Changing `window.clickThroughInactive` prompts for a restart.

#### TB-3.20 Pin a floating editor window on top

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - none of the three ids is in the bundle
- What: In a floating editor window (auxiliary window), a pin button on the layout controls turns Always on Top on, and a pinned icon turns it off. The palette command toggles it. The main window cannot be pinned.
- Commands: workbench.action.toggleWindowAlwaysOnTop; workbench.action.enableWindowAlwaysOnTop; workbench.action.disableWindowAlwaysOnTop
- Menus: LayoutControlMenu (group navigation, order 1, auxiliary windows only); CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\services\auxiliaryWindow\electron-browser\auxiliaryWindowService.ts; C bundle (searched, none found)
- Pieces:
  1. In a floating editor window, a pin button on the layout controls turns Always on Top on for that window. [stock only]
  2. While Always on Top is on, a pinned icon in the same spot of the layout controls turns it off. [stock only]
  3. Toggle Window Always on Top (`workbench.action.toggleWindowAlwaysOnTop`) in the Command Palette flips the state while a floating editor window has focus, and Turn On Always on Top (`workbench.action.enableWindowAlwaysOnTop`) and Turn Off Always on Top (`workbench.action.disableWindowAlwaysOnTop`) set it explicitly. [stock only]
  4. The main window cannot be pinned, so the pin button and the pin commands apply only to floating editor windows. [stock only]

#### TB-3.21 Compact mode for floating editor windows

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - none of the three ids and no `isCompactTitleBar` key is in the bundle
- What: A floating editor window can switch to compact mode, which trims its title bar to a smaller set of buttons: no command center, no global actions, no Accounts or Manage, and a separate editor title menu. A button on the layout controls and a palette command turn it on and off, and the `isCompactTitleBar` context key follows.
- Commands: workbench.action.toggleCompactAuxiliaryWindow; workbench.action.enableCompactAuxiliaryWindow; workbench.action.disableCompactAuxiliaryWindow
- Menus: LayoutControlMenu (group navigation, order 0, auxiliary windows only); CommandPalette; LayoutControlMenu
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\common\contextkeys.ts; C bundle (searched, none found); V\src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts (lines 50 to 105 for the actions, lines 174 to 185 for what compact changes); C bundle (none of the three ids found); V\src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts:50-105; C bundle (searched the three ids and the labels Turn On Compact Mode and Turn Off Compact Mode, no hits)
- Pieces:
  1. Turning on compact mode in a floating editor window trims its title bar, with no command center, no global actions and no Accounts or Manage buttons. [stock only]
  2. In compact mode the title bar shows editor actions from a separate compact window editor title menu in place of the usual editor title menu. [stock only]
  3. Compact mode also hides the status bar of the floating window. [stock only]
  4. A button on the layout controls of a floating editor window (group `navigation`, order 0) shows Turn On Compact Mode while the window is not compact and Turn Off Compact Mode while it is. [stock only]
  5. The Command Palette lists Toggle Window Compact Mode (`workbench.action.toggleCompactAuxiliaryWindow`) and the two explicit commands `workbench.action.enableCompactAuxiliaryWindow` and `workbench.action.disableCompactAuxiliaryWindow`. [stock only]
  6. A floating window leaves compact mode on its own when an editor group is added to it, and the `isCompactTitleBar` context key follows the state. [stock only]
  7. Toggle Window Compact Mode in the Command Palette turns compact mode on or off while an editor window is focused (`workbench.action.toggleCompactAuxiliaryWindow`). [stock only]
  8. A layout control button in the editor window's title bar turns compact mode on, and while compact mode is on it turns into a button that turns it off (`workbench.action.enableCompactAuxiliaryWindow`, `workbench.action.disableCompactAuxiliaryWindow`). [stock only]
  9. Compact mode slims the title bar and the editor part and hides the status bar of that window. [stock only]
  10. In a floating editor window the layout controls hold a button labelled Turn On Compact Mode that switches the window to compact mode, a smaller title bar with fewer buttons. [stock only]

#### TB-3.22 Window controls overlay for installed web apps (web)

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the `wco-enabled` class and the web branch of the container code are in the bundle, but Cursor ships as desktop only
- What: Web only. When the page runs as an installed app with the window controls overlay, the title bar reserves space for the browser's controls on the right, and on the left as well on macOS, using the overlay's size values, and the bar height follows the overlay height. Desktop Cursor never takes this branch.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part)
- Pieces:
  1. When the page runs as an installed web app with the window controls overlay, the title bar reserves space at its right end for the browser's own window controls.
  2. On macOS the title bar also reserves space at its left end for the overlay controls.
  3. The title bar height follows the height of the window controls overlay.
  4. Desktop Cursor never shows this behavior, because it ships as a desktop app only.

#### TB-3.23 No title bar layout (Cursor)

- Tag: cursor. Evidence: bundle. Boundary: yes - the layout lives in Cursor's agent-centered unified layout and the freed title row hosts agent tabs and moves layout buttons into the editor tab strip. Cursor: cursor only
- What: On macOS and Linux the page body gets the class `no-titlebar-layout`. The title bar becomes a transparent 34 px strip fixed at the top that passes clicks through except on its toolbars, the window title is hidden, the command center is forced off, and side bar, panel and auxiliary bar get about 35 px of top padding. The mode is driven by a stored preference `cursor/noTitlebarLayout.visibility` whose default is `hide`. It never applies on Windows, is off when the feature gate `disable_no_title_bar` is on, and is off when editor tabs are set to `none`. The context key `cursor.noTitlebarLayout.enabled` reports it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (workbench CSS rules for `no-titlebar-layout`; layout module with the storage key, the gate and the platform check; title bar part `isCommandCenterVisible`)
- Pieces:
  1. On macOS and Linux with the no title bar layout on, the title bar becomes a transparent 34 px strip fixed at the top of the window. [cursor only]
  2. The transparent strip passes clicks through to what is under it, except on its toolbars. [cursor only]
  3. In this layout the window title is hidden and the command center is forced off. [cursor only]
  4. In this layout the side bar, the panel and the auxiliary bar get about 35 px of top padding so they clear the strip. [cursor only]
  5. The layout is driven by a stored preference `cursor/noTitlebarLayout.visibility` whose default is `hide`, and the context key `cursor.noTitlebarLayout.enabled` reports whether the layout is on. [cursor only]
  6. The layout never applies on Windows. [cursor only]
  7. The layout is off while the feature gate `disable_no_title_bar` is on, and off when editor tabs are set to `none`. [cursor only]


## TB-4 Layout controls

(both; the Switch Layout overlay is cursor) [boundary]. Buttons or a Configure Layout drop-down that toggle the side bars and panel, plus Cursor's layout presets Right, Zen, and Browser. Presets show or hide agent and chat panes.

19 features.

#### TB-4.1 Show or hide the layout controls

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Turn the whole layout control area in the title bar on or off, from the setting or from the "Layout Controls" row in the title bar right-click menu.
- Commands: toggle.workbench.layoutControl.enabled
- Menus: TitleBarContext (group 2_config, order 4); TitleBarTitleContext (group 2_config, order 4)
- Keys: none found
- Settings: workbench.layoutControl.enabled
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts:75-79; V\src\vs\workbench\browser\workbench.contribution.ts:777-784; C bundle (module titlebarActions strings 4016-4017, setting text 4249, getter `layoutControlEnabled` in the title bar part)
- Pieces:
  1. Setting `workbench.layoutControl.enabled` on (default) shows the layout control area in the title bar.
  2. Setting `workbench.layoutControl.enabled` off hides the whole layout control area from the title bar.
  3. Right-clicking the title bar or the title text area shows a Layout Controls row (`toggle.workbench.layoutControl.enabled`, group `2_config`, order 4) that is checked while the controls show and flips the setting when clicked.

#### TB-4.2 Choose the layout control style (menu, toggles, or both)

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick whether the title bar shows one Configure Layout drop-down (`menu`), separate toggle buttons (`toggles`), or both (`both`). Default is `both` in stock and in Cursor.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.layoutControl.type
- Source: V\src\vs\workbench\browser\workbench.contribution.ts:785-796; C bundle (module workbench.contribution, strings 4250-4253, default `both` read from the registration)
- Pieces:
  1. Setting `workbench.layoutControl.type` to `menu` shows a single Configure Layout button that opens a drop-down of layout options.
  2. Setting `workbench.layoutControl.type` to `toggles` shows separate toggle buttons for showing and hiding the side bars and the panel.
  3. Setting `workbench.layoutControl.type` to `both` (default in stock and in Cursor) shows the layout button and the toggle buttons together.

#### TB-4.3 Toggle the primary side bar from a title bar button

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (button placement comes from fixed left and right toolbar lists; key and command are disabled while an auxiliary window has focus; no screen reader announcement on toggle)
- What: Click a panel-shaped button to hide or show the primary side bar. The icon follows the side bar position and its on or off state. In Cursor the button sits in the left toolbar when the Agents side bar is on the right (the left-toolbar copy is hidden during onboarding), otherwise in the right toolbar.
- Commands: workbench.action.toggleSidebarVisibility
- Menus: LayoutControlMenu (V group navigation, order 0 or 2; Cursor group 2_pane_toggles, order 0 or 2); LayoutControlMenuSubmenu (group 0_workbench_layout, order 0)
- Keys: Ctrl+B (Cmd+B on macOS)
- Settings: workbench.sideBar.location; workbench.layoutControl.type
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:282-391; C bundle (class for workbench.action.toggleSidebarVisibility with keybinding and precondition on isAuxiliaryWindowFocusedContext; toolbar lists S6w and y6w; string 4036)
- Pieces:
  1. Clicking the primary side bar button in the title bar (tooltip Toggle Primary Side Bar) hides the primary side bar when it is shown and shows it when it is hidden.
  2. The button icon is a left panel icon when `workbench.sideBar.location` is `left` and a right panel icon when it is `right`, and it switches between an off and an on version as the side bar is hidden or shown.
  3. In stock the button shows only while `workbench.layoutControl.type` is `toggles` or `both`, and never in a floating editor window. [stock only]
  4. Ctrl+B (Cmd+B on macOS) toggles the primary side bar and the button state follows.
  5. In stock, toggling the primary side bar tells screen readers "Primary Side Bar hidden" or "Primary Side Bar shown". [stock only]
  6. In Cursor the button sits in the left toolbar when the Agents side bar is on the right (and is hidden there during onboarding), and in the right toolbar otherwise. [cursor only]
  7. In Cursor, Ctrl+B and the toggle command are disabled while a floating editor window has focus, and toggling makes no screen reader announcement. [cursor only]

#### TB-4.4 Toggle the panel from a title bar button

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (extra maximized-chat handling and same auxiliary-window key guard as the side bar toggle)
- What: Click a panel-shaped button to hide or show the bottom panel; in Cursor the button is always in the right toolbar, and the command first leaves the maximized layout that hides the editor if that is active.
- Commands: workbench.action.togglePanel
- Menus: LayoutControlMenu (V group navigation, order 1; Cursor group 2_pane_toggles, order 1); LayoutControlMenuSubmenu (group 0_workbench_layout, order 4)
- Keys: Ctrl+J (Cmd+J on macOS)
- Settings: workbench.layoutControl.type
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts:34-70,320-342; C bundle (class with keybinding Ctrl+J and unified maximize handling; string 3948)
- Pieces:
  1. Clicking the panel button in the title bar (tooltip Toggle Panel) hides the bottom panel when it is shown and shows it when it is hidden.
  2. The panel button icon switches between an off and an on version as the panel is hidden or shown.
  3. In stock the panel button shows only while `workbench.layoutControl.type` is `toggles` or `both`, never in a floating editor window, and it sits between the two side bar buttons. [stock only]
  4. Ctrl+J (Cmd+J on macOS) toggles the panel and the button state follows.
  5. In Cursor the panel button is always in the right toolbar. [cursor only]
  6. In Cursor, toggling the panel first leaves the maximized layout that hides the editor when that is active. [cursor only]
  7. In Cursor, Ctrl+J and the panel toggle command are disabled while a floating editor window has focus. [cursor only]

#### TB-4.5 Toggle the secondary side bar from a title bar button

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: Stock shows a second panel-shaped button that hides or shows the secondary side bar, and Cursor puts this button neither on the layout menu nor in the right toolbar.
- Commands: workbench.action.toggleAuxiliaryBar
- Menus: LayoutControlMenu (group navigation, order 0 or 2); LayoutControlMenuSubmenu (group 0_workbench_layout, order 1)
- Keys: Ctrl+Alt+B (Cmd+Alt+B on macOS)
- Settings: workbench.layoutControl.type
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts:30-80,140-180; C bundle (LayoutControlMenu appendMenuItems list, toolbar builder T6w)
- Pieces:
  1. A secondary side bar button in the title bar (tooltip Toggle Secondary Side Bar) hides the secondary side bar when it is shown and shows it when it is hidden. [stock only]
  2. The secondary side bar button sits on the opposite side from the primary side bar button, with a left icon when `workbench.sideBar.location` is `right` and a right icon when it is `left`, and the icon switches between an off and an on version. [stock only]
  3. The secondary side bar button shows only while `workbench.layoutControl.type` is `toggles` or `both`, and never in a floating editor window. [stock only]
  4. Ctrl+Alt+B (Cmd+Alt+B on macOS) toggles the secondary side bar. [stock only]
  5. Toggling the secondary side bar tells screen readers "Secondary Side Bar hidden" or "Secondary Side Bar shown". [stock only]
  6. In Cursor no title bar button toggles the secondary side bar. [cursor only]

#### TB-4.6 Open the Configure Layout drop-down

- Tag: both. Evidence: source. Boundary: yes. Cursor: present
- What: When the style is `menu`, a single layout button opens a list with Primary Side Bar, Secondary Side Bar and Panel toggles (checked when shown) and Customize Layout... at the end. Boundary reason: one row of the drop-down (Secondary Side Bar) opens or creates the agent chat pane in Cursor.
- Commands: workbench.action.toggleSidebarVisibility; workbench.action.toggleAuxiliaryBar; workbench.action.togglePanel; workbench.action.customizeLayout
- Menus: LayoutControlMenu (group 1_workbench_layout, submenu); LayoutControlMenuSubmenu
- Keys: none found
- Settings: workbench.layoutControl.type
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:172-182; V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts:50-66; C bundle (submenu item with `config.workbench.layoutControl.type` equals `menu`; toggleAuxiliaryBar class calls ensureChatVisibleOrCreate)
- Pieces:
  1. With `workbench.layoutControl.type` set to `menu`, a single Configure Layout button (layout icon) shows in the title bar and opens a drop-down list of layout options.
  2. The drop-down lists Primary Side Bar, Secondary Side Bar and Panel as toggle rows, each checked while that part is shown, and clicking a row hides or shows the part.
  3. Customize Layout... is the last row of the drop-down and opens the Customize Layout picker (TB-4.7).
  4. The Configure Layout button is not shown in a floating editor window.
  5. In Cursor the Secondary Side Bar row opens or creates the agent chat pane instead of only toggling a side bar. [cursor only]

#### TB-4.7 Open Customize Layout

- Tag: both. Evidence: source. Boundary: yes. Cursor: changed (no standalone button; no Activity Bar row)
- What: A picker titled Customize Layout with sections for Visibility (Menu Bar, Primary Side Bar, Secondary Side Bar, Panel, Status Bar, and in stock also Activity Bar), side bar position, panel alignment, quick input position, and modes (Full Screen, Zen Mode, Centered Layout). In stock it also has its own title bar button when the style is `both`. In Cursor it is only in the Configure Layout drop-down and the Command Palette, and the Activity Bar row is gone. Boundary reason: the Secondary Side Bar row in Cursor shows or hides the agent chat pane.
- Commands: workbench.action.customizeLayout
- Menus: LayoutControlMenu (group navigation, or group 1_layout when editor actions live in the title bar); LayoutControlMenuSubmenu (group z_end); CommandPalette
- Keys: none found
- Settings: workbench.layoutControl.type
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:1315-1375,1417-1450,1517-1560; C bundle (customizeLayout class with only the z_end submenu entry; item list vEr; strings 3134-3140, 3155, 3208)
- Pieces:
  1. Running Customize Layout... (`workbench.action.customizeLayout`) opens a picker titled Customize Layout with no filter box and a Close button, and it stays open when focus moves elsewhere.
  2. Running Customize Layout... again while the picker is open closes it.
  3. The picker has section headings Visibility, Primary Side Bar Position, Panel Alignment, Quick Input Position and Modes, and each row shows its on or off state and its key binding when it has one.
  4. Choosing a row, or its button, runs that row's command, the list refreshes to the new state at once, and the picker stays open.
  5. Customize Layout... is the last row of the Configure Layout drop-down and is also listed in the Command Palette.
  6. In stock, with `workbench.layoutControl.type` set to `both`, a Customize Layout button (layout icon) shows in the title bar layout controls, in group `navigation` or in group `1_layout` when editor actions live in the title bar. [stock only]
  7. In Cursor there is no standalone Customize Layout button in the title bar, only the drop-down row and the Command Palette entry. [cursor only]
  8. In Cursor the Visibility section has no Activity Bar row, and its Secondary Side Bar row shows or hides the agent chat pane. [cursor only]

#### TB-4.8 Restore layout defaults from Customize Layout

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Restore Defaults button at the top of the Customize Layout picker puts the activity bar location, side bar location, status bar, panel location and menu bar visibility back to their defaults, and resets panel alignment to center and quick input to top.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.location; workbench.sideBar.location; workbench.statusBar.visible; workbench.panel.defaultLocation; window.menuBarVisibility
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:1538-1606; C bundle (nls key `restore defaults` in layoutActions; reset helper reads `inspect(id).defaultValue`; exact list of reset settings not decoded)
- Pieces:
  1. A Restore Defaults button (discard icon) is always visible at the top of the Customize Layout picker, next to the Close button.
  2. Clicking Restore Defaults puts `workbench.activityBar.location`, `workbench.sideBar.location`, `workbench.statusBar.visible` and `workbench.panel.defaultLocation` back to their default values.
  3. Clicking Restore Defaults also puts `window.menuBarVisibility` back to its default, except on macOS desktop where the menu bar is left alone.
  4. Clicking Restore Defaults sets the panel alignment back to center and the quick input position back to top.

#### TB-4.9 Move the side bar from the title bar right-click menu

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: Right-click empty space in the left title bar toolbar to get one row, Move Side Bar Right or Move Side Bar Left, that swaps the primary side bar to the other side. Boundary reason: moving the primary side bar also moves the Agents side bar to the opposite side (the command stores an agent location override).
- Commands: workbench.action.toggleSidebarPosition
- Menus: none found
- Keys: none found
- Settings: workbench.sideBar.location
- Source: C bundle (title bar part method `showLeftActionsContextMenu`, strings 4029-4030; command class for toggleSidebarPosition with the `SIDEBAR_LOCATION_AGENT_OVERRIDE` store); V has the same command only in View > Appearance, not in the title bar
- Pieces:
  1. Right-clicking empty space in the left title bar toolbar opens a context menu with a single row. [cursor only]
  2. The row reads Move Side Bar Right while the primary side bar is on the left, and Move Side Bar Left while it is on the right. [cursor only]
  3. Clicking the row swaps the primary side bar to the other side (`workbench.sideBar.location`). [cursor only]
  4. Moving the primary side bar this way also moves the Agents side bar to the opposite side. [cursor only]

#### TB-4.10 Layout buttons move into the editor tab strip

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: In Cursor's no-title-bar layout the side bar and panel buttons can render at the right end of the top-right editor tab strip instead of in the title bar. They go there when the right edge has no visible side bar and the editor area or panel is the right-most part, and they stay in the title bar otherwise.
- Commands: none found
- Menus: LayoutControlMenu
- Keys: none found
- Settings: none found
- Source: C bundle (functions B_a and r_S, method getLayoutActionsPositioningMode, CSS classes `in-titlebar`, `in-editor-tabs`, `layout-actions-container`; inputs include the chat editor group storage key `cursor/chatEditorGroupEnabled`)
- Pieces:
  1. In Cursor's no-title-bar layout, the side bar and panel buttons can show at the right end of the top-right editor tab strip instead of in the title bar. [cursor only]
  2. The buttons go into the editor tab strip when the right edge has no visible side bar and the editor area or the panel is the right-most part. [cursor only]
  3. The buttons stay in the title bar in every other case. [cursor only]
  4. The buttons show in one place at a time, either in the title bar or in the editor tab strip. [cursor only]

#### TB-4.11 Open the Switch Layout overlay

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: Press Ctrl+Alt+Tab to open a full-screen overlay of tiles. Tab, Left and Right move the selection, Enter or a click applies it, Escape closes it, and letting go of the modifier keys applies the selected tile. The tile that matches the current layout starts selected. Boundary reason: the tiles are agent layouts and apply agent and chat pane visibility.
- Commands: workbench.action.openLayoutSwitcher
- Menus: CommandPalette (category View)
- Keys: Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS)
- Settings: none found
- Source: C bundle (module layoutSwitcherOverlay, strings 3213-3217 and layoutActions string 3166; component tpS; CSS classes `layout-switcher-overlay`, `__backdrop`, `__container`, `__tile`, `__icon-wrapper`, `__label`, `__separator`; the `__hint` and `__shortcut` classes are in the CSS only and the component does not create them)
- Pieces:
  1. Pressing Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS), or running `workbench.action.openLayoutSwitcher` from the Command Palette (category View), opens a full-screen overlay with a row of tiles over a backdrop. [cursor only]
  2. Each tile in the overlay shows an icon with a label, and the tile that matches the current layout starts selected. [cursor only]
  3. Tab, Left and Right move the selection between tiles. [cursor only]
  4. Pressing Enter or clicking a tile applies it and closes the overlay. [cursor only]
  5. Pressing Escape closes the overlay without applying a tile. [cursor only]
  6. Letting go of the modifier keys applies the selected tile and closes the overlay. [cursor only]

#### TB-4.12 Apply the Right layout (the Editor tile)

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: Pick the tile named Editor in the overlay (the preset is named Layout - Right elsewhere in the code). It hides the Agents side bar and the panel, shows the chat pane, the editor area and the primary side bar, and puts the Agents side bar location on the right. Boundary reason: the preset hides the Agents side bar and shows the chat pane.
- Commands: workbench.action.openLayoutSwitcher; workbench.action.toggleSidebarVisibility; workbench.action.togglePanel; workbench.action.toggleAuxiliaryBar
- Menus: none found
- Keys: Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS)
- Settings: none found
- Source: C bundle (preset id `default-editor` in lists pEr and hEr with state in mva; apply function dpS; strings 2999 and 3214; storage key `cursor/agentLayout.quickMenu.lastSelectedLayoutId`)
- Pieces:
  1. Picking the tile named Editor in the overlay applies the Editor layout (named Layout - Right elsewhere in the code). [cursor only]
  2. The Editor layout hides the Agents side bar and the panel. [cursor only]
  3. The Editor layout shows the chat pane, the editor area and the primary side bar. [cursor only]
  4. The Editor layout puts the Agents side bar location on the right. [cursor only]

#### TB-4.13 Switch to another open window from the overlay

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Other open Cursor windows (not auxiliary windows, not the current one) show first in the overlay as tiles named after their workspace folder, then a separator, then the layout tiles. Choosing one focuses that window. The number of tiles is capped by the screen width.
- Commands: workbench.action.openLayoutSwitcher
- Menus: none found
- Keys: Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS)
- Settings: none found
- Source: C bundle (component tpS reads nativeHostService.getWindows with includeAuxiliaryWindows false and calls focusWindow; helper ipS names the tile; string 3215-3217 for the Untitled fallback)
- Pieces:
  1. Other open Cursor windows, not counting floating editor windows or the current window, show first in the overlay as tiles. [cursor only]
  2. Each window tile is named after the workspace folder of that window, with an Untitled name when it has none. [cursor only]
  3. A separator divides the window tiles from the layout tiles. [cursor only]
  4. Choosing a window tile brings that window to the front and focuses it. [cursor only]
  5. The number of tiles shown is capped by the screen width. [cursor only]

#### TB-4.14 Apply a saved custom layout from the overlay

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: Layouts saved in storage show as extra tiles after Agent and Editor, with a small preview icon drawn from the saved pane visibility. They are skipped when no folder or workspace is open, and any whose id or name clashes with a default tile is dropped. No control that creates, renames or deletes a layout was found in the IDE window code. Boundary reason: a saved layout stores agent, chat, editor, panel and side bar visibility, and part widths.
- Commands: workbench.action.openLayoutSwitcher
- Menus: none found
- Keys: Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS)
- Settings: none found
- Source: C bundle (functions kbm and rpS read storage key `cursor/agentLayout.quickMenu.customLayouts`; tpS drops customs when the workbench state is empty)
- Pieces:
  1. Layouts saved in storage show as extra tiles in the overlay after the Agent and Editor tiles. [cursor only]
  2. Each saved layout tile has a small preview icon drawn from the pane visibility saved with it. [cursor only]
  3. Saved layout tiles are skipped when no folder or workspace is open. [cursor only]
  4. A saved layout whose id or name clashes with a default tile is dropped and does not show. [cursor only]
  5. Choosing a saved layout tile applies its stored visibility of the agent, chat, editor, panel and side bar parts and its part widths. [cursor only]

#### TB-4.15 Zen layout preset

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: A preset named Layout - Zen that shows only the editor area (Agents side bar, chat pane, panel and primary side bar all hidden). It is defined in the preset list, but no tile or button in the IDE window applies it (see Unclear). Boundary reason: the preset hides the Agents side bar and the chat pane.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (preset id `default-zen` in list hEr, one reference only; string 3000)
- Pieces:
  1. A preset named Layout - Zen shows only the editor area, with the Agents side bar, chat pane, panel and primary side bar all hidden. [cursor only]
  2. No tile or button in the IDE window applies the Zen preset, so a person cannot pick it from the overlay. [cursor only]

#### TB-4.16 Browser layout preset

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: A preset named Browser with the editor area and chat pane shown and the Agents side bar, panel and primary side bar hidden. It is defined in the preset list, but no tile or button in the IDE window applies it (see Unclear). The preset has no field that opens a browser. Boundary reason: the preset shows the chat pane and hides the Agents side bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (preset id `default-browser` in list hEr, one reference only; string 3001)
- Pieces:
  1. A preset named Browser shows the editor area and the chat pane, with the Agents side bar, panel and primary side bar hidden. [cursor only]
  2. No tile or button in the IDE window applies the Browser preset, so a person cannot pick it from the overlay. [cursor only]
  3. The Browser preset only sets which panes show and does not itself open a browser. [cursor only]

#### TB-4.17 New workspaces open in the last chosen layout

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: When a workspace has no saved side bar or panel visibility, Cursor applies the layout last picked in the overlay (the Editor layout if none) as its first-open layout. It always starts with the Agents side bar hidden and the chat pane shown, and the editor area starts hidden only when the auto-hide-editor-when-empty flag is on. Boundary reason: it decides whether the Agents side bar and chat pane start visible.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (method applyUnifiedWorkspaceLayout, functions hbm and LhS, storage keys `cursor/agentLayout.quickMenu.lastSelectedLayoutId`, `workbench.sideBar.hidden`, `workbench.panel.hidden`)
- Pieces:
  1. When a workspace has no saved side bar or panel visibility, Cursor opens it in the layout last picked in the overlay, or in the Editor layout when none was picked. [cursor only]
  2. The first-open layout always starts with the Agents side bar hidden and the chat pane shown. [cursor only]
  3. The editor area starts hidden in the first-open layout only when the auto-hide-editor-when-empty flag is on. [cursor only]
  4. A workspace that already has saved side bar or panel visibility keeps it and does not get the first-open layout. [cursor only]

#### TB-4.19 Keep an auxiliary window always on top

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: In a popped-out editor window the layout area holds a pin button that turns always-on-top on or off for that window.
- Commands: workbench.action.enableWindowAlwaysOnTop; workbench.action.disableWindowAlwaysOnTop; workbench.action.toggleWindowAlwaysOnTop
- Menus: LayoutControlMenu (group navigation, order 1, auxiliary windows only); CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts:490-575; C bundle (searched the three ids and the labels Turn On Always on Top and Turn Off Always on Top, no hits)
- Pieces:
  1. In a floating editor window the layout controls hold a pin button labelled Turn On Always on Top that keeps that window above other windows. [stock only]
  2. While the window is on top, the same spot shows a pinned icon labelled Turn Off Always on Top that releases it. [stock only]
  3. The pin button is in group `navigation` at order 1 of the layout controls and shows only in floating editor windows, not in the main window. [stock only]
  4. The Command Palette lists Toggle Window Always on Top (`workbench.action.toggleWindowAlwaysOnTop`), which works only while a floating editor window has focus, along with Turn On Always on Top (`workbench.action.enableWindowAlwaysOnTop`) and Turn Off Always on Top (`workbench.action.disableWindowAlwaysOnTop`). [stock only]

#### TB-4.20 Choose layout density in Customize Layout

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: newer-than-1.128. When the experimental Modern UI is on, Customize Layout gets a Layout Density section with Default and Compact, and Restore Defaults also resets it.
- Commands: workbench.action.setLayoutDensity.default; workbench.action.setLayoutDensity.compact
- Menus: LayoutDensityMenu
- Keys: none found
- Settings: window.density.layout; workbench.experimental.modernUI
- Source: N\src\vs\workbench\browser\actions\layoutDensityActions.ts; N\src\vs\workbench\browser\actions\layoutActions.ts (LayoutDensityActions); C bundle (searched `window.density.layout` and `setLayoutDensity`, no hits)
- Pieces:
  1. newer-than-1.128: with the experimental Modern UI on (`workbench.experimental.modernUI`), Customize Layout gets a Layout Density section with Default and Compact rows. [stock only]
  2. newer-than-1.128: choosing Default or Compact sets `window.density.layout` through `workbench.action.setLayoutDensity.default` or `workbench.action.setLayoutDensity.compact`, and the chosen row shows a check. [stock only]
  3. newer-than-1.128: Restore Defaults in Customize Layout also resets the layout density. [stock only]


## TB-5 Title bar buttons and update control

(both; the buttons are cursor) [boundary]. Single-purpose buttons: the Update control, the Cursor Settings gear, Upgrade, and stock's notification bell and browser button. Cursor Settings and Upgrade depend on account and agent.

11 features.

#### TB-5.1 Update button in the title bar

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (different command id, own button component IdeTitlebarUpdateButton, shown only when the update is ready and the experiment variant matches)
- What: A text button labelled Update shows next to the search box (stock) or in the left toolbar (Cursor) when an update is waiting. Stock shows it for available, downloaded and ready updates, hides it in editor Zen mode, in debug mode and while a chat request runs, and hides it when the window is too narrow to fit it. Cursor shows it only when the update service state is `ready`, the server experiment `ide_update_ux_exp` has the variant `non_sidebar`, and the window is not the Agents window.
- Commands: workbench.actions.updateIndicator; cursor.titlebar.installUpdate
- Menus: TitleBarAdjacentCenter (order 0, stock only; Cursor builds the button in code at left toolbar order 130)
- Keys: none found
- Settings: update.titleBar
- Source: V\src\vs\workbench\contrib\update\browser\updateTitleBarEntry.ts:57-75,150-217; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts:495-512,918-936; C bundle (module IdeTitlebarUpdateButton strings 11480-11484; titlebarPart string 4034; computeToolbarState fields ideUpdateAvailable and isGlassWindow; toolbar list S6w entry BFr order 130; component c6w)
- Pieces:
  1. When an update is waiting, a text button labelled Update shows in the title bar, next to the search box in stock and in the left toolbar in Cursor.
  2. In stock the Update button shows for the available, downloaded and ready update states, and the plain Update label is highlighted. [stock only]
  3. In stock the Update button is hidden in editor Zen mode, in debug mode and while a chat request is running. [stock only]
  4. In stock the Update button is hidden when the window is too narrow to fit it. [stock only]
  5. In Cursor the Update button shows only when the update state is `ready`, the server experiment `ide_update_ux_exp` has the variant `non_sidebar`, and the window is not the Agents window. [cursor only]

#### TB-5.2 Click Update to download, install or restart

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (one action, restart-to-update, with an installing state and spinner)
- What: Clicking the button acts on the update state. Stock runs download, install or restart depending on state, and when nothing is actionable it opens the tooltip instead. Cursor only restarts to install: the label turns to Installing with a spinner after a 300 ms arm timer, and it goes back to Update if the restart is skipped, the shutdown is vetoed or the command fails.
- Commands: update.downloadNow; update.install; update.restart; update.restartToUpdate
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\update\browser\updateTitleBarEntry.ts:282-300; V\src\vs\workbench\contrib\update\browser\update.ts:84-160; C bundle (component c6w click handler runs `update.restartToUpdate` with source `titlebar`; attributes data-installing and aria-busy; CSS spinner classes `ide-titlebar-update-button-spinner`)
- Pieces:
  1. In stock, clicking Update when an update is available starts the download (`update.downloadNow`), and clicking it when the update is downloaded starts the install (`update.install`). [stock only]
  2. In stock, clicking Update when the update is ready restarts the app to apply it (`update.restart`). [stock only]
  3. In stock, clicking Update when no action is possible opens the update card instead. [stock only]
  4. In Cursor, clicking Update restarts Cursor to install the update (`update.restartToUpdate`), as its only action. [cursor only]
  5. In Cursor, a moment after the click (a 300 ms timer) the label turns to Installing with a spinner and the button is marked busy for screen readers. [cursor only]
  6. In Cursor, the label goes back to Update when the restart is skipped, the shutdown is vetoed or the command fails. [cursor only]

#### TB-5.3 Update button shows check, download and install progress

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: Stock changes the button label and fill as the update moves: Checking..., Downloading... with a percent fill, Installing... with a percent fill, Updating..., Restarting... and Cancelling.... Download, install and overwrite progress only show when the user started the update. The plain Update state is highlighted. Cursor shows none of these states, only Update and its own Installing spinner (TB-5.2).
- Commands: update.checking; update.downloading; update.updating; update.cancelling (the disabled state rows that mirror these labels in the Manage menu)
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\update\browser\updateTitleBarEntry.ts:303-361; V\src\vs\workbench\contrib\update\browser\update.ts:84-150; C bundle (searched `updateIndicator`, `workbench.actions.updateIndicator` and the labels Checking... and Downloading... in the title bar part, no hits; its title bar button only reads the `ready` state)
- Pieces:
  1. While the app checks for updates, the button reads Checking... with an indefinite progress look. [stock only]
  2. While an update downloads, the button reads Downloading... and fills left to right by the percent downloaded. [stock only]
  3. While an update installs, the button reads Installing... and fills by the percent installed. [stock only]
  4. The button reads Updating... while an update is being overwritten, Restarting... while the app restarts to update, and Cancelling... while an update is being cancelled, each with an indefinite progress look. [stock only]
  5. Download, install and overwrite progress show on the button only when the person started the update. [stock only]
  6. The plain Update label for the available, downloaded and ready states is highlighted. [stock only]

#### TB-5.4 Update tooltip card

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (plain hover text instead of the stock card; no updateTooltip strings in the bundle)
- What: Hovering the button opens a card. Stock shows a sticky card with a title for each state (Update Available, Downloading Update with speed and time left, Update is Ready to Install, Restart to Update, Up to Date, Updates Disabled, Update Error), the current and latest version with a Copy control, the release date, a Release Notes button and Download, Install or Restart buttons. It opens by itself on an update error and, at most once every 30 days, when updates are disabled by a bad config or by running as administrator. Cursor shows a small hover below the button with one line: "Restart Cursor to install the pending update", with the version when known, and a different line while installing.
- Commands: update.downloadNow; update.install; update.restart; update.showCurrentReleaseNotes
- Menus: none found
- Keys: none found
- Settings: update.mode
- Source: V\src\vs\workbench\contrib\update\browser\updateTooltip.ts:120-445; V\src\vs\workbench\contrib\update\browser\updateTitleBarEntry.ts:36-37,160-217,255-275; C bundle (module IdeTitlebarUpdateButton strings 11480-11482; hover opened below the button with a pointer; searched nls keys for `updateTooltip`, no hits)
- Pieces:
  1. Hovering the Update button opens a card with a pointer that stays open until it is dismissed. [stock only]
  2. The card title follows the update state: Update Available, Downloading Update, Update is Ready to Install, Restart to Update, Up to Date, Updates Disabled or Update Error. [stock only]
  3. While an update downloads, the card shows the download speed and the time left. [stock only]
  4. The card shows Current Version and Latest Version lines with a Copy control for the version, and a Released date line. [stock only]
  5. The card has a Release Notes button (`update.showCurrentReleaseNotes`) and, depending on the state, a Download (`update.downloadNow`), Install (`update.install`) or Restart (`update.restart`) button. [stock only]
  6. The card opens by itself on an update error, and at most once every 30 days when updates are disabled by an invalid config or by running as administrator. [stock only]
  7. Setting `update.mode` changes the message on the Up to Date card (automatic updates disabled, checked but not installed, applied on restart, or enabled), and the Updates Disabled card tells the person to change `update.mode` when updates were turned off by hand. [stock only]
  8. In Cursor, hovering the Update button shows a small hover below it with the line "Restart Cursor to install the pending update", with the version when it is known, and a different line while installing. [cursor only]

#### TB-5.5 Hide the Update button

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: A setting turns the title bar update indicator off. Desktop only. Cursor has no such setting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: update.titleBar
- Source: V\src\vs\platform\update\common\update.config.contribution.ts:90-96; V\src\vs\workbench\contrib\update\browser\updateTitleBarEntry.ts:39,160-163; C bundle (searched `update.titleBar` in the bundle and the nls keys, no hits)
- Pieces:
  1. Setting `update.titleBar` on (default) shows the update indicator in the title bar when an update is waiting, and off hides it. [stock only]
  2. With `update.titleBar` off, the update card also never opens by itself on an update error. [stock only]
  3. `update.titleBar` is offered on desktop only, and Cursor has no such setting. [stock only]

#### TB-5.6 Cursor Settings gear button

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: A gear button in the right toolbar, always shown in the IDE window, that opens Cursor Settings. Hover text is Open Cursor Settings. Ctrl+Shift+J does the same, and Ctrl+, does too when the window is not the Agents window. Boundary reason: the button opens Cursor Settings, which configures the agent, models and account.
- Commands: workbench.action.openAgentLayoutQuickMenu; aiSettings.action.open
- Menus: none found
- Keys: Ctrl+Shift+J (Cmd+Shift+J on macOS); Ctrl+, (Cmd+, on macOS, when not isGlass)
- Settings: none found
- Source: C bundle (class registered as workbench.action.openAgentLayoutQuickMenu with title string 3165 and icon settingsGear that runs `aiSettings.action.open`; right toolbar list y6w entry order 225 with isVisible true; titlebarPart strings 4038-4039; key objects with weight 401)
- Pieces:
  1. A gear button in the right toolbar of the title bar, always shown in the IDE window, opens Cursor Settings when clicked (`workbench.action.openAgentLayoutQuickMenu` runs `aiSettings.action.open`). [cursor only]
  2. Hovering the gear button shows the text Open Cursor Settings. [cursor only]
  3. Ctrl+Shift+J (Cmd+Shift+J on macOS) opens Cursor Settings. [cursor only]
  4. Ctrl+, (Cmd+, on macOS) also opens Cursor Settings, but only when the window is not the Agents window. [cursor only]

#### TB-5.7 Upgrade button

- Tag: cursor. Evidence: bundle. Boundary: yes. Cursor: cursor only
- What: A pill button labelled Upgrade at the left end of the right toolbar. It shows only for accounts on the Free plan when no membership refresh is pending. Hover says "Upgrade for extended limits and full model access". Clicking starts the upgrade checkout for the next plan (a Stripe Pro checkout or a unified billing flow) and ignores further clicks until it finishes. Boundary reason: it depends on the signed-in account plan and starts a billing flow.
- Commands: cursor.titlebar.upgradeToPro
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (action id `cursor.titlebar.upgradeToPro`; component d6w with CSS classes `upgrade-pro-button` and `upgrade-pro-button-text`; computeToolbarState field shouldPromptUpgrade from membershipType equal to FREE; analytics event `titlebar.upgrade_pro_clicked`; right toolbar list y6w entry order 0)
- Pieces:
  1. A pill button labelled Upgrade shows at the left end of the right toolbar. [cursor only]
  2. The Upgrade button shows only for accounts on the Free plan when no membership refresh is pending. [cursor only]
  3. Hovering the Upgrade button shows "Upgrade for extended limits and full model access". [cursor only]
  4. Clicking the Upgrade button starts the checkout for the next plan, either a Stripe Pro checkout or a unified billing flow (`cursor.titlebar.upgradeToPro`). [cursor only]
  5. Further clicks on the Upgrade button are ignored until the checkout flow finishes. [cursor only]

#### TB-5.8 Notification bell in the title bar

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: When notifications are set to appear at the top right (an experimental setting), a bell button at the far right of the action toolbar opens and closes the notification center. A setting and a right-click row, Notifications, hide it. Cursor has neither the button nor either setting. The id notifications.toggleList does exist in the Cursor bundle for the status bar bell, but there is no title bar row and no toggle row.
- Commands: notifications.toggleList; toggle.workbench.notifications.showInTitleBar
- Menus: TitleBar (group navigation, order 10000); TitleBarContext (group 2_config, order 5); TitleBarTitleContext (group 2_config, order 5); TitleBarContext (toggle row, order 5)
- Keys: none found
- Settings: workbench.notifications.showInTitleBar; workbench.notifications.position
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts:330-343; V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts:80-84; V\src\vs\workbench\browser\workbench.contribution.ts:630-648; N\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts:360 (same row, now registered through a disposable list); C bundle (searched ids and the settings); V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (searched toggle.workbench.notifications, Toggle Notifications, workbench.notifications.showInTitleBar: no hits; toggleList found)
- Pieces:
  1. With `workbench.notifications.position` set to `top-right` (an experimental setting with values `bottom-right`, `bottom-left` and `top-right`), a bell button shows at the far right of the title bar action toolbar (group `navigation`, order 10000). [stock only]
  2. Clicking the bell opens the notification center and clicking it again closes it (`notifications.toggleList`). [stock only]
  3. Setting `workbench.notifications.showInTitleBar` (default on) hides the bell when it is off, and it applies only while notifications are at the top right. [stock only]
  4. Right-clicking the title bar shows a Notifications row (`toggle.workbench.notifications.showInTitleBar`, group `2_config`, order 5) that flips the bell on and off, and the row shows only while notifications are at the top right. [stock only]
  5. With the notification position set to top right, the bell is removed from the status bar. [stock only]
  6. With the position at top right, a bell button with hover text Toggle Notifications appears at the right end of the title bar. [stock only]
  7. Setting `workbench.notifications.showInTitleBar` shows or hides the title bar bell. [stock only]
  8. Right-clicking the title bar shows a Notifications toggle row that shows or hides the title bar bell (`toggle.workbench.notifications.showInTitleBar`). [stock only]

#### TB-5.9 Browser button in the title bar

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: A globe button that opens a new integrated browser tab, or the list of open browser tabs when one is already open. A setting chooses always, never, or only while a browser editor is open (default). A right-click row, Integrated Browser, toggles it. Cursor has no title bar browser button, and its own Browser opens through other commands (see Handed off).
- Commands: workbench.action.browser.openOrList; workbench.action.browser.open; workbench.action.browser.quickOpen; toggle.workbench.browser.showInTitleBar
- Menus: TitleBar (group navigation, order 10); TitleBarContext (group 2_config, order 8); TitleBarTitleContext (group 2_config, order 8)
- Keys: Ctrl+Alt+/ (Cmd+Alt+/ on macOS)
- Settings: workbench.browser.showInTitleBar
- Source: V\src\vs\workbench\contrib\browserView\electron-browser\features\browserTabManagementFeatures.ts:477-526,553-557,905-922; V\src\vs\platform\browserView\common\browserView.ts:12-19; C bundle (searched `workbench.action.browser`, `workbench.browser.` and `showInTitleBar`, no hits)
- Pieces:
  1. A globe button in the title bar (group `navigation`, order 10) opens a new integrated browser tab when none is open, and opens the list of open browser tabs when one is already open (`workbench.action.browser.openOrList`, which calls `workbench.action.browser.open` or `workbench.action.browser.quickOpen`). [stock only]
  2. Setting `workbench.browser.showInTitleBar` chooses when the globe button shows, with `true` always, `false` never, and `whenOpen` (default) only while a browser editor is open. [stock only]
  3. Right-clicking the title bar shows an Integrated Browser row (`toggle.workbench.browser.showInTitleBar`, group `2_config`, order 8) that is checked while the setting is `true` and flips the setting when clicked. [stock only]
  4. Ctrl+Alt+/ (Cmd+Alt+/ on macOS) opens a new browser tab or the list of open browser tabs, the same as the globe button. [stock only]

#### TB-5.10 Title bar button overflow menu

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (limit 10 instead of 12)
- What: When the action toolbar holds more buttons than its limit, the extra ones collapse into a "..." menu. Stock allows 12 and Cursor allows 10. Accounts, Manage and the editor navigation commands are never collapsed. Cursor sets the "..." position to the end of the toolbar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts:663-680; C bundle (method createActionToolBar with overflowBehavior maxItems 10 and moreActionPlacement end)
- Pieces:
  1. When the title bar action toolbar holds more buttons than its limit, the extra buttons collapse into a "..." menu.
  2. In stock the toolbar shows up to 12 buttons before it collapses the rest. [stock only]
  3. In Cursor the toolbar shows up to 10 buttons before it collapses the rest. [cursor only]
  4. In Cursor the "..." menu sits at the end of the toolbar. [cursor only]
  5. Accounts, Manage and the editor navigation commands are never collapsed into the "..." menu.

#### TB-5.11 Hover a title bar button for its name and shortcut

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Hovering a button in the action toolbar shows its name and, when it has one, its key binding. The toolbar also carries an accessible label, Title actions.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts:663-680; C bundle (createActionToolBar passes getKeyBinding and hoverDelegate; string 4026; hover on the left toolbar buttons was not checked)
- Pieces:
  1. Hovering a button in the title bar action toolbar shows its name.
  2. The hover also shows the key binding of the button when it has one.
  3. The title bar action toolbar has the accessible label Title actions.
