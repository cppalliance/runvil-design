Bucket: D. Date: 2026-10-25. Pass: 4.

### NEW-D-1 Split a Browser tab into a second Browser tab
- Group: ED-10
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the Browser tab is a page agents can open and drive, and the split makes another such tab.
- What: Splitting a Browser tab up, down, left or right, or splitting it inside its group, opens a second Browser tab on the same page instead of showing the same tab twice.
- Commands: browser.splitEditorWithNewBrowserTab; browser.splitEditorWithNewBrowserTabToDirection; workbench.action.splitEditorUp; workbench.action.splitEditorDown; workbench.action.splitEditorLeft; workbench.action.splitEditorRight; workbench.action.splitEditorInGroup
- Menus: none found (reached through the stock split entries described in ED-1.13 and ED-1.24)
- Keys: Ctrl+K Ctrl+Shift+\ (Split Editor in Group); the other split keys are the stock ones listed in ED-1.13
- Settings: none found
- Cursor: cursor only
- Source: C bundle (commands browser.splitEditorWithNewBrowserTab and browser.splitEditorWithNewBrowserTabToDirection registered in module browserEditor.contribution; the four directional split commands, Split Editor in Group and the shared split helper test for editor resource scheme cursor-browser and call them; helper qxp reads the live page address or the saved last address; editor pane workbench.editor.browserEditor)
- Pieces:
  1. Split Editor Up, Down, Left or Right on a Browser tab opens a pinned Browser tab in the group in that direction (a new group is added when none is there) instead of a second copy of the same tab. [cursor only]
  2. The new tab starts on the address the first tab is showing now, or on the last address it remembers when no live page is available, and from then on it is a separate Browser tab that can go to its own addresses.
  3. Split Editor in Group (Ctrl+K Ctrl+Shift+\ or the Command Palette) on a Browser tab turns the tab into a side-by-side pair holding the original and a new Browser tab on the same address. [cursor only]
  4. The new group takes focus unless the split was asked to keep focus where it was, in which case focus stays put.
  5. When one split moves several editors of a group together, each Browser tab among them gets a new Browser tab and every other editor is copied the usual way.
  6. If the new Browser tab cannot be created, the split falls back to the stock behavior of copying the same tab into the new group, and only a console warning records the failure.

### NEW-D-2 Create a rule from the selected text
- Group: ED-8
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the rule it saves is guidance the agent follows, though the command itself does not call a model.
- What: Select text in an editor, right-click and choose Create Rule to save it as a new manual rule file in the workspace's .cursor/rules folder, then use the Open Rule button on the notice to open it.
- Commands: cursor.createRuleFromSelection; cursor.openCreatedRule
- Menus: EditorContext (group navigation, order 20, only when the editor has a non-empty selection)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (action class with id cursor.createRuleFromSelection, title Create Rule, description "Create a cursor rule from the highlighted text", menu EditorContext group navigation order 20, precondition hasNonEmptySelection; rule service createRule(name, text, false); notification action id cursor.openCreatedRule with label Open Rule and tooltip "Open the generated rule file"; name helpers that cut to 12 characters and add a numeric suffix). Pass 1 notes list cursor.createRuleFromSelection as a chat entry point and the coverage script put it in its agentish list, so it is in no work file; it is added here as a related id.
- Pieces:
  1. With text selected, the editor right-click menu shows Create Rule near the top, in the navigation group at order 20, and it is not available when nothing is selected. [cursor only]
  2. Choosing it saves the selected text, trimmed, as a new manual rule file in the workspace's .cursor/rules folder.
  3. The rule name is the first six words of the selection joined by dashes, lower-cased with quotes dropped and other punctuation turned into dashes, cut to 12 characters; if no word is usable it uses the first three parts of the file's relative path, or the word rule, and if a rule of that name already exists it adds -1, -2 and so on.
  4. A notice reads "Created manual rule '<name>' in .cursor/rules. Attach it explicitly when you want it applied." with an Open Rule button that opens the new rule file in a pinned tab.
  5. When the command cannot run it shows a warning instead: "Open a file to create a cursor rule.", "Select the text you want to turn into a rule.", "Highlight some text before creating a rule." or "Open a workspace folder to store cursor rules."; a write failure shows "Failed to create rule: <message>".
  6. The rule is created as a manual rule, so it applies only when the person attaches it, as the notice says. [cursor only]

### NEW-D-3 Remove Vim mode
- Group: ED-2
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Run Cursor Vim: Remove Vim Mode from the Command Palette to uninstall the Vim extension (vscodevim.vim) and reload the window.
- Commands: cursor.removevim
- Menus: commandPalette (no category, no condition)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (action cursor.removevim, title "Cursor Vim: Remove Vim Mode", f1 true with no precondition; it lists installed extensions, uninstalls the one whose id is vscodevim.vim, then reloads the window; the id is in the tracked command list under vim_mode)
- Pieces:
  1. Cursor Vim: Remove Vim Mode shows in the Command Palette for everyone, with no category prefix and no condition. [cursor only]
  2. Running it uninstalls the Vim extension (vscodevim.vim) when it is installed, with no confirmation box.
  3. The window reloads straight after the uninstall is requested, so the Vim mode indicator and Vim key handling are gone once it comes back.
  4. When the Vim extension is not installed nothing is uninstalled, and the window still reloads.

### NEW-D-4 Toggle sign-in debugging
- Group: WB-9
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Run Developer: Toggle Auth Debugging from the Command Palette to switch sign-in debug logging on or off and open the Cursor Auth Debug output channel.
- Commands: cursorAuth.toggleAuthDebug
- Menus: commandPalette (no category, no condition)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (action cursorAuth.toggleAuthDebug, title "Developer: Toggle Auth Debugging", f1 true with no precondition; auth service toggleAuthDebug, getAuthDebugOutputChannelId and channel id cursorAuthDebug; first log line "Auth debugging enabled: AuthId ..., http2Disabled=..."). The sibling palette command cursorAuth.triggerTokenRefresh has no visible result and is excluded.
- Pieces:
  1. The palette entry is titled "Developer: Toggle Auth Debugging" (the word Developer is part of the title, not a category) and is offered with no condition. [cursor only]
  2. Running it flips the sign-in debug flag and then opens the Output panel on the Cursor Auth Debug channel.
  3. When the flag turns on, the channel gets a first line saying "Auth debugging enabled" with the account's auth id (or none) and whether HTTP/2 is disabled, and later sign-in activity is written there.
  4. Running it again turns the logging off and still shows the channel.
