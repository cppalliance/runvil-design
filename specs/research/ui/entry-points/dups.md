Bucket: DUP. Date: 2026-10-25. Pass: 5.
MERGE | TB-2.1 | QA-5.2 | same click on the title bar search box, same command and setting; QA-5.2 itself says it is also TB-2.1; TB-2 owns the box; fold in QA-5.2's hover tip, screen reader name and setting pieces, and fix TB-2.1's stale "help entry" wording to "shortcut rows" (QA-5.3 covers the rows); confidence medium - catalog also lists the command center under QA-5 as a way in
KEEP | ED-5.36 | PN-6.12 | editor word wrap (Alt+Z, editor.wordWrap) vs Debug Console wrapping (debug.console.wordWrap); different views and settings
KEEP | AC-1.26 | AC-5.14 | suggest list explain mode (toggleExplainMode) vs terminal suggest explain mode (terminalSuggestToggleExplainMode); different widgets and commands
KEEP | AC-1.31 | AC-6.14 | stock suggest list Tab inserts a row vs Cursor Tab accept applies a generated edit; different commands and rules
KEEP | AC-1.36 | QA-6.21 | suggest list arrow and page keys vs quick pick list keys; different widgets and command ids
KEEP | AC-2.47 | AC-6.18 | stock ghost text type-through vs Cursor Tab type-along; different code paths and reject rules
MERGE | SS-6.11 | SS-4.17 | same Workspace Name item (status.workspaceName, priority 50, same hover); catalog lists the workspace name item under SS-6; SS-6.11 has the fuller pieces (aria label, removal, update triggers); SS-4.17's pieces note says one should go; confidence high - identical item
MERGE | SS-6.12 | SS-5.17 | same High memory warning item (Memory WARNING and CRITICAL indicators, show_status_entry flag); catalog lists the memory warning under SS-6; SS-6.12 has the fuller pieces (aria label, fallback off); confidence high - identical item
KEEP | SB-3.28 | ED-3.36 | sending Search view results to a Search Editor vs opening a match from inside the Search Editor
KEEP | ED-7.23 | PN-5.9 | clearing notebook cell outputs vs clearing an Output channel
XREF | SB-3.10 | ED-3.32 | same include and exclude boxes in two surfaces (Search view vs Search Editor); SB-3.10 should point to ED-3.32, which also has the open editors switch, ignore files switch and focus commands
XREF | SB-2.17 | ED-6.11 | SB-2.17's three commands and menus are a subset of ED-6.11, which also adds Compare Active File With...; SB-2.17 should point to ED-6.11 for the diff; not merged because the Explorer entry details (not on folders, shows only after marking) are a separate tree feature; closest call in this list, keeper would be ED-6.11
XREF | BN-2.1 | SS-4.5 | same Restricted Mode state shown in two places (banner strip vs status bar item), both end in workbench.trust.manage; BN-2.1 should point to SS-4.5 and back
KEEP | AB-4.16 | ED-8.54 | Manage row that opens the shortcuts table vs the JSON file buttons in the shortcuts editor; only the back-to-table command is shared
XREF | AB-4.14 | QA-1.1 | the Manage row is one of QA-1.1's listed routes into the palette; AB-4.14 should point to QA-1.1; not merged because the place differs (Manage menu)
XREF | ED-8.58 | DL-6.14 | same command cursorai.action.importvscode, two surfaces: the General tab Import button with its own "Are you sure?" box vs the post-onboarding notification and three-button dialog; each should point to the other
XREF | TB-1.16 | DL-1.26 | same Clear Recently Opened command; TB-1.16 is the menu row and DL-1.26 is the confirm box, and both repeat the confirm text; TB-1.16 should point to DL-1.26
MERGE | TB-3.21 | ED-1.37 | same compact mode for a floating editor window (toggle, enable, disable commands, layout control button, hides status bar); TB-3.21 has the fullest pieces; ED-1.37 and TB-4.18 are subsets; confidence high - identical commands, menu and behavior
KEEP | PN-4.20 | PN-5.19 | Problems filter help page vs Output filter help page; different context keys and page text
XREF | ED-6.27 | SS-4.14 | same blame data in the editor (end of line text) vs the status bar item; separate settings and surfaces; each should point to the other
MERGE | ED-13.4 | PN-6.33 | same inline test peek under the failing line, same seven commands and keys; catalog ED-13 names peeking test output; fold in Peek Output (testing.openOutputPeek, Ctrl+; then Ctrl+M) and the two testing.automaticallyOpenPeekView settings from PN-6.33; confidence medium - PN-6.33 frames it from the panel side
MERGE | ED-1.37 | TB-4.18 | both are the same compact mode feature already dropped into TB-3.21 by the TB-3.21 rows; no extra action beyond applying those rows; confidence high
XREF | AB-4.18 | AC-4.17 | the Manage Snippets row is one of AC-4.17's listed routes to the picker; AB-4.18 should point to AC-4.17
KEEP | QA-2.8 | QA-3.9 | Right Arrow in the Go to File list (opens files pinned) vs in Go to Line and Symbol lists (moves the editor); same command, different lists and results
KEEP | ED-2.28 | ED-10.73 | cut, copy and paste in the text editor vs in a webview editor; different targets and menu
XREF | AB-4.15 | ED-8.1 | the Manage Settings row is one of ED-8.1's listed routes to the Settings editor; AB-4.15 should point to ED-8.1
MERGE | SB-3.28 | ED-3.30 | same Open New Search Editor button, Open Results in Editor command and Alt+Enter key from the Search view; catalog SB-3 lists Open in Editor as an example; fold in ED-3.30's piece about defaultNumberOfContextLines rerun; confidence medium - decision 5 says the Search Editor is ED-3 but the controls live in the Search view
KEEP | ED-4.39 | NT-3.12 | hover accessible view vs notification accessible view; same command id, different content and settings
XREF | NT-3.1 | NT-4.5 | same status bar bell shown in its normal and Do Not Disturb states; NT-3.1 should point to NT-4.5; also note NT-3.1 and SS-5.2 overlap
XREF | ED-1.9 | DL-2.3 | ED-1.9 is the close-many commands and DL-2.3 is the unsaved changes flow they trigger; ED-1.9 should point to DL-2.3
KEEP | AC-6.14 | AC-7.23 | accepting a suggestion vs accepting the import hint; same command id and key, different hint and result; AC-6.14 already points to AC-7.23
XREF | SS-5.2 | NT-3.7 | SS-5.2 is the status bar view of the bell click plus show and hide routes, and NT-3.7 owns the many ways to close the center (Hide button, Escape, command, auto close, focus return); SS-5.2 should point to NT-3.7 and NT-3.6; SS-5.2 is largely the union of NT-3.1, NT-3.6 and NT-3.7, so a later pass could fold it
XREF | SB-6.21 | PN-6.27 | both list the Rerun and Debug Failed Tests from Last Run commands; PN-6.27 adds the Test Results row hover buttons; PN-6.27 should point to SB-6.21
MERGE | ED-1.28 | ED-2.39 | same Copy cursor.com link pair (cursorOrigin.copyCodeLink and copyCodeLinkWithoutRange) from the same Share submenus on the tab, editor, gutter and Explorer menus; ED-1.28 has the Command Palette entries and git.enabled rule; fold in ED-2.39's eligible-remote rule; confidence high - identical commands and menus
XREF | PN-7.10 | SS-4.6 | Ports view table vs the status bar Forwarded Ports item that focuses it; SS-4.6 should point to PN-7.10
MERGE | TB-3.21 | TB-4.18 | same compact mode feature (same three commands and LayoutControlMenu button); TB-3.21 has the fuller pieces; confidence high
XREF | TB-4.5 | AX-1.1 | TB-4.5 is the stock title bar button (missing in Cursor) and AX-1.1 is the toggle command, key and menu rows; TB-4.5 should point to AX-1.1
KEEP | SB-2.38 | ED-10.27 | Explorer Open in Browser entry for a file vs opening a blank Browser tab; shared command only
XREF | SB-4.26 | SS-4.13 | Publish Branch from the Git menus and action button vs the status bar publish item; SS-4.13 should point to SB-4.26
XREF | TB-4.9 | SB-1.2 | TB-4.9 is the title bar right-click row and SB-1.2 is the general side bar move feature; TB-4.9 should point to SB-1.2
KEEP | ED-8.19 | AC-8.10 | the Cursor Settings open command and page frame vs the Cursor Tab section inside the Code Intelligence tab; AC-8.10 already points to ED-8.19
XREF | PN-4.1 | SS-5.6 | PN-4.1 piece 5 covers the same status bar click; SS-5.6 should point to PN-4.1; not merged because SS-5.6 is the item's click
KEEP | NT-3.6 | NT-4.8 | opening the center vs silenced notifications still landing in it
MERGE | TB-5.8 | NT-3.5 | same stock title bar bell (notifications.toggleList, showInTitleBar setting and right-click row, position setting), both vscode only; TB-5 lists the stock notification bell and has the fuller pieces; fold in NT-3.5's piece that the status bar bell is removed at top right; confidence high
XREF | AB-4.16 | ED-8.9 | the Manage Keyboard Shortcuts row is one of ED-8.9's listed routes to the editor; AB-4.16 should point to ED-8.9
XREF | SB-4.21 | ED-6.23 | same Accept All Current and Accept All Incoming commands, from the Merge Changes list vs from the editor with CodeLens; each should point to the other
XREF | SS-5.11 | QA-4.12 | status bar Running Tasks item and the Show Running Tasks picker it opens; SS-5.11 should point to QA-4.12
XREF | AB-2.9 | AB-6.11 | same Configure Keybinding entry in the stock rail icon menu vs Cursor's tab menu; AB-2.9 already says the tab menu has it; each should point to the other
MERGE | SB-1.24 | SB-2.28 | same sticky scroll for view trees (tree.toggleStickyScroll, enableStickyScroll, stickyScrollMaxItemCount); SB-1 owns shared view behavior; fold in SB-2.28's pinned row keys (Left, Enter, Space); confidence medium - SB-2.28 frames it for the Explorer only
XREF | SB-6.13 | PN-6.26 | Test Explorer title Cancel button vs the Test Results row hover buttons; both list testing.cancelRun; PN-6.26 should point to SB-6.13 and SB-6.21
KEEP | ED-3.28 | PN-5.19 | find widget accessibility help vs Output filter help page
KEEP | AC-2.38 | NT-3.12 | suggestion accessible view vs notification accessible view; same command id only
XREF | SS-5.2 | NT-3.6 | same bell click and Ctrl+K Ctrl+Shift+N route to the center; SS-5.2 should point to NT-3.6
KEEP | AB-4.15 | ED-8.21 | Manage Settings row vs Cursor's VS Code Settings tab and menu row; shared command only
XREF | SB-2.39 | ED-6.39 | same cursorBlame.openFileBlame command from the Explorer right-click entry vs the editor menu and blame view; SB-2.39 should point to ED-6.39, which already points back
XREF | ED-2.5 | SS-3.25 | the Selection menu row and toggle command appear in both; SS-3.25 is the status bar indicator; ED-2.5 should point to SS-3.25
KEEP | ED-3.28 | PN-4.20 | find widget accessibility help vs Problems filter help page
KEEP | ED-8.18 | ED-10.46 | the Workspace Trust editor vs the Workspace Trust Required placeholder page; the page's button is the only shared route
XREF | TB-1.25 | TB-3.15 | same six macOS window tab commands, as Window menu rows vs the native tabs setting and palette entries; each should point to the other
XREF | TB-2.4 | ED-1.32 | ED-1.32 is the whole navigation history family and says the arrows are owned by TB-2; ED-1.32 should point to TB-2.4
XREF | TB-4.4 | PN-1.2 | same workbench.action.togglePanel from the title bar button vs the Ctrl+J key and palette entry, with the same Cursor chat branch; each should point to the other
XREF | TB-5.6 | ED-8.19 | the gear button vs the Cursor Settings page it opens; ED-8.19 already says the gear is owned by TB-5; TB-5.6 should point to ED-8.19
XREF | ED-8.18 | DL-5.10 | Configure Workspace Trust Settings is a piece of ED-8.18 and its own feature in DL-5.10; ED-8.18 should point to DL-5.10
KEEP | QA-1.3 | QA-5.3 | typing > to switch to commands vs the shortcut rows shown when Quick Open starts from the command center; shared commands only
XREF | TB-5.2 | AB-4.21 | the title bar Update button vs the Update rows in the Manage menu; both call the same update commands; AB-4.21 already points to TB-5 and NT-6; each should point to the other
XREF | SB-2.40 | ED-10.44 | ED-10.44 pieces 6 and 7 repeat SB-2.40's Explorer entry and setting; SB-2.40 should point to ED-10.44 for the viewer itself
KEEP | ED-4.39 | AC-2.38 | hover accessible view vs suggestion accessible view; same command id only
KEEP | PN-2.11 | QA-4.7 | renaming a terminal in the tabs list vs from a row of the Switch Active Terminal picker; different surfaces
XREF | TB-1.24 | DL-2.11 | same macOS Quit row; TB-1.24 is the app menu and DL-2.11 is the quit confirm box; TB-1.24 should point to DL-2.11
KEEP | TB-2.1 | QA-5.4 | command center box vs the workspace name button shown when the box is off; note QA-5.4 duplicates TB-2.9 (QA-5.4 says "Also listed as TB-2.9"), which this work file did not pair, so a later pass should merge QA-5.4 into TB-2.9
KEEP | TB-4.4 | PN-1.17 | title bar panel button vs the panel's own title toolbar buttons (Maximize and Hide Panel); shared command only
KEEP | AB-4.18 | AC-4.18 | Manage Snippets row vs creating a snippets file from the picker; shared command only
XREF | AB-6.10 | SB-1.6 | Cursor's tab right-click menu vs the container title right-click menu it replaces; both mention Hide Primary Side Bar and Move Primary Side Bar; each should point to the other
KEEP | SB-5.3 | QA-4.15 | Start Debugging button and F5 vs the Select and Start Debugging picker; shared command only
XREF | SB-5.16 | ED-7.18 | Notebook Variables view vs the toolbar button and inline values in the notebook; both cover notebook.openVariablesView and notebook.variablesView; each should point to the other
KEEP | ED-3.28 | PN-3.22 | find widget accessibility help vs terminal accessible view and help
XREF | SS-3.3 | QA-3.1 | status bar line and column item that opens Go to Line vs the Go to Line list itself; SS-3.3 should point to QA-3.1
XREF | SS-5.3 | NT-4.1 | SS-5.3 is the bell icon in Do Not Disturb and NT-4.1 is turning it on or off; SS-5.3 should point to NT-4.1; note SS-5.3 is nearly identical to NT-4.5 (bell-slash icon in the status bar), which this work file did not pair, so a later pass could merge it
XREF | AB-4.20 | ED-8.15 | Manage Themes submenu vs the Color Theme picker it opens; AB-4.20 should point to ED-8.15 and list the other two pickers as ownerless
XREF | SB-2.10 | DL-4.13 | same Ctrl+Z and Ctrl+Y undo of Explorer file steps and same explorer.confirmUndo setting; SB-2.10 is the undo feature and DL-4.13 is the confirm box; each should point to the other
KEEP | SB-2.30 | ED-1.8 | Open Editors pane vs closing one editor; shared command only
KEEP | ED-1.8 | ED-10.30 | closing an editor vs the Browser tab close warning for agent-made tabs
XREF | ED-1.30 | QA-4.2 | the Ctrl+Tab picker is described in both, ED-1.30 as the switching commands and QA-4.2 as the list; each should point to the other
XREF | ED-2.18 | SS-3.9 | same Detect Indentation from Content command; SS-3.9 is the status bar indentation menu row; ED-2.18 should point to SS-3.9
KEEP | AB-4.15 | SS-6.4 | Manage Settings row vs the AI stats gauge gear that opens one setting; shared command only
KEEP | SB-1.6 | AX-4.5 | primary side bar container menu vs secondary side bar tab strip menu; different parts
KEEP | ED-8.1 | SS-6.4 | the Settings editor open routes vs the AI stats gauge gear that opens one setting
XREF | ED-10.46 | DL-5.9 | DL-5.9 is the hub of routes to the trust editor and lists the Workspace Trust Required button; ED-10.46 should point to DL-5.9
