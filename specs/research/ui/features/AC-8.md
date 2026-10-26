Group: AC-8 Cursor Tab controls. Section: Autocomplete. Date: 2026-10-25. Pass: 2.
Features: 19.

### AC-8.1 Cursor Tab status item and its text states
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A right-side status bar item named "Cursor Tab" (priority 1000) that shows "Cursor Tab", "Cursor Tab (Snoozed)" while a snooze runs, or "Cursor Tab (Ignored)" when the open file matches the cursor ignore rules (snoozed wins if both are true). The text is also the aria label. When Tab is not active for the open file the item is drawn dimmed (opacity .5) with a line through the text (class `copilot-plus-plus-status-off`). That look applies when Tab is off globally, not allowed for the account, off for the file language, the file matches an Ignored Files pattern, the file is blocked by cursor ignore, the file is over 5,000,000 characters, or it is a `.env` file in plaintext. The snoozed state uses its own class and is not struck through. Cross-listed in SS-6 (listed there as a cross-reference only).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: cursor.cpp.disabledLanguages
- Cursor: cursor only
- Source: C bundle (status entry builder `statusBarElementProps` at byte offset ~40664627, registered as `addEntry(...,"Cursor Tab",1,1e3)`; `isCppEnabledForEditor`; `shouldSkipCppBecauseFileIsTooLarge`; CSS `.copilot-plus-plus-status-off`); V has no hit for `Cursor Tab`, `editor.cpp.` or `cppEnabled`

### AC-8.2 Open the Cursor Tab panel from the status item
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Hovering the status item shows a small panel, and clicking it while signed in keeps the panel open (Cursor adds a "do not hide the hover on click" flag that V lacks, and uses the stock `statusBar.entry.showTooltip` command so the click shows the hover). When signed out the click runs Log In to Cursor instead. Panel rows, top to bottom: the "Disable globally" checkbox row, the "Disable for {language}" checkbox row (only when a text editor is active), the "Model" row with a select, and the Snooze row. Each row is a flex line with 4px padding and the label on the left, the control on the right. The panel text changes when Tab is unavailable, see AC-8.8.
- Commands: statusBar.entry.showTooltip; editor.cpp.login
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (the item and click command exist in V's status bar; the hover-stays-open flag `dontHideHoverOnClick` is not in V)
- Source: V\src\vs\workbench\services\statusbar\browser\statusbar.ts (`ShowTooltipCommand`); V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts; C bundle (entry fields `tooltip`, `dontHideHoverOnClick:!0`, `command: signedIn ? statusBar.entry.showTooltip : editor.cpp.login`; CSS `.cpp-status-bar-hover-element`, `.cpp-status-bar-toggle-checkbox`)

### AC-8.3 Turn Cursor Tab on or off
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Three Command Palette commands flip one app-wide on or off flag: "Enable Cursor Tab", "Disable Cursor Tab" and "Toggle Cursor Tab". The same flag is set by the "Disable globally" checkbox in the status panel (checked means off) and by the "Cursor Tab" switch in Cursor Settings (AC-8.10). Turning Tab on, by any of these, also ends a running snooze. The flag is stored in app storage, not in settings.json, so there is no setting id. Using any of them updates the context key `cppEnabled`.
- Commands: editor.action.enableCppGlobally; editor.cpp.disableenabled; editor.cpp.toggle
- Menus: CommandPalette (f1 true, no `when`)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (labels "Enable Cursor Tab", "Disable Cursor Tab", "Toggle Cursor Tab" at byte offset ~40857926 to ~40859500; checkbox code `_renderStatusCheckBox`; the effect that sets `cppEnabledContextKey` and calls `unsnoozeCpp` when the flag turns true); V has no hit

### AC-8.4 Turn Cursor Tab off for a language
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The panel's second checkbox, "Disable for {language}", adds or removes the language of the active editor (the word "untitled" for untitled files) in the user setting `cursor.cpp.disabledLanguages`. Checked means the language is in the list. The same list can be edited in the Settings editor or settings.json, where it sits in a group titled "Cursor Tab" (configuration id `cpp`): array of strings, default `["markdown","plaintext"]`, description "Disable Cursor Tab for these languages". Changing the editor language rebuilds the row. Tab is also off for `.env` style files opened as plaintext without any setting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: cursor.cpp.disabledLanguages
- Cursor: cursor only
- Source: C bundle (`registerConfiguration({id:"cpp",title:"Cursor Tab",...})` at byte offset ~38005058; `setCppDisabledForLanguage` writes target 2 (user); `isCppDisabledForLanguage`; `hD_` with the `.env` list)

### AC-8.5 Pick the Cursor Tab model from the panel
- Tag: cursor
- Evidence: bundle
- Boundary: no - the control only. The names behind it and what each one does are out of scope.
- What: The panel's "Model" row holds a bordered drop-down (an HTML select with a small triangle). The first option reads "auto (default)", followed by the names the app keeps refreshed. Picking one stores it (picking the default clears it) and reloads the Cursor Tab setup. The Cursor Settings page has no Model row (see Unclear).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`_renderCppModelSelect` at byte offset ~40662617; label "Model"; stored flag `cppModel`; default label `auto (default)`)

### AC-8.6 Snooze Cursor Tab for a set time
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: "Snooze Cursor Tab" opens a quick pick with the placeholder "Select Cursor Tab snooze duration" and six rows, each with the description "Snooze for {label}": "1 minute", "5 minutes", "15 minutes", "30 minutes", "1 hour", "3 hours" (the 3 hours row is coded as 4 hours, see Unclear). The command shows in the palette only while Tab is on and not snoozed, and does nothing if Tab is off. The panel's "Snooze" link runs the same command, and an info icon beside it carries the tooltip "Temporarily disable Cursor Tab suggestions for a specified duration. You can unsnooze at any time." The Snooze row is dimmed and ignores clicks when Tab is off and not snoozed. A snooze turns the on or off flag off, remembers the earlier state, and the status text becomes "Cursor Tab (Snoozed)". If no length is stored the default is 5 minutes.
- Commands: editor.cpp.snooze
- Menus: CommandPalette (f1 true, `when` not cppSnoozed and cppEnabled)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class for `editor.cpp.snooze` at byte offset ~40858871; duration table; `snoozeCpp`; `_renderSnoozeButtons` at ~40743559; `getSnoozeDuration` default `300*1e3`)

### AC-8.7 Unsnooze Cursor Tab and watch the time left
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: "Unsnooze Cursor Tab" ends a snooze at once and puts the on or off flag back to what it was before. It shows in the palette only while snoozed. In the panel the Snooze link turns into "Unsnooze" with a grey "({time} remaining)" count that ticks every second, written "Xh Ym" above an hour or "Xm Ys" below it. When the time runs out Tab turns itself back on, and a snooze that ended while the app was closed is cleared the next time the status item is built. Unticking "Disable globally" also ends a snooze.
- Commands: editor.cpp.unsnooze
- Menus: CommandPalette (f1 true, `when` cppSnoozed)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class for `editor.cpp.unsnooze`; `unsnoozeCpp`; `formatSnoozedTimeRemaining`; `maybeReenableCppIfSnoozed`; stored flags `cppSnoozed`, `cppSnoozeDuration`, `cppEnabledBeforeSnooze`)

### AC-8.8 See why Cursor Tab is not available
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: When Tab cannot be switched in the panel it shows one line instead of the rows. "Please log in" when signed out. "Requires pro (custom model). " followed by an "Upgrade here." link when signed in but Tab is not allowed for the account (the link starts the upgrade flow, not traced). "Cursor Tab is unavailable in Cursor Private Inference" in local mode, and "Cursor Tab: Off under private inference" when private inference turns the Tab feature off. The status text stays "Cursor Tab" and the item is dimmed.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (strings "Please log in", "Requires pro (custom model). ", "Upgrade here.", "Cursor Tab is unavailable in Cursor Private Inference", "Off under private inference" in `statusBarElementProps`)

### AC-8.9 Log in to Cursor from Cursor Tab
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: "Log In to Cursor" runs the sign-in flow. It is in the Command Palette at all times (no `when`), and it is what a click on the status item runs when you are signed out.
- Commands: editor.cpp.login
- Menus: CommandPalette (f1 true)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class with `ID="editor.cpp.login"`, `LABEL="Log In to Cursor"` at byte offset ~40644045; the status entry `command` field)

### AC-8.10 Cursor Tab section in Cursor Settings
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Inside the Cursor Settings "Code Intelligence" tab (id `indexing`) there is a group named "Cursor Tab" (anchor `cursor-settings-cursor-tab`). Its first row is a switch labelled "Cursor Tab" with the description "Context-aware, multi-line suggestions around your cursor based on recent edits". It is the same on or off flag as AC-8.3. The six tuning rows (AC-8.11 to AC-8.16) show only while the switch is on. Under the group a muted line can appear: "CPP and auto-imports are temporarily disabled", "CPP is temporarily disabled" or "Auto-imports are temporarily disabled", when the server side turns those off. The old hidden tab id `tab` redirects to `indexing`, and the Cursor Settings search has an entry "Cursor Tab" (the category also answers to "cursor tab" and "tab"). The tab frame itself is ED-8.30 and the open command is ED-8.19.
- Commands: aiSettings.action.open
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (settings page component with `accessibleLabel:"Cursor Tab"` at byte offset ~38364061; search entry `Kl("indexing","cursor-tab",...)` at ~38093771; redirect map `{...,tab:"indexing"}` at ~33333321; category title map `indexing:"Code Intelligence"`)

### AC-8.11 Turn partial accepts on or off
- Tag: cursor
- Evidence: bundle
- Boundary: no - the switch only; the partial accept action itself is AC-6.
- What: The "Partial Accepts" row, with the description "Accept the next word of a suggestion via {keybinding}" and a small info button beside the label whose tooltip reads "Keybinding: {keybinding}" (aria label "Partial Accepts keybinding: {keybinding}"). The keybinding text is looked up for `editor.action.inlineSuggest.acceptNextWord` and falls back to the string "Alt+]" when none is bound. The switch writes the user setting `cursor.cpp.enablePartialAccepts` (boolean, default false, description "Enable partial accepts for Cursor Tab, using the editor.action.inlineSuggest.acceptNextWord keybinding"). With it on, the context key `cppSuggestion` is true while a Cursor Tab suggestion exists, so the word-accept key also works for Cursor Tab.
- Commands: editor.action.inlineSuggest.acceptNextWord
- Menus: InlineSuggestionToolbar (group primary, order 2, the stock "Accept Word" item that Cursor redefined)
- Keys: CtrlCmd+Right (editor.action.inlineSuggest.acceptNextWord, when editor.writable and (inlineSuggestionVisible or cppSuggestion))
- Settings: cursor.cpp.enablePartialAccepts
- Cursor: cursor only
- Source: C bundle (row code beside `label:"Partial Accepts"` at byte offset ~38364061; configuration registration at ~38005058; redefined command class at ~24657132; `_bindCppSuggestionKey`)

### AC-8.12 Allow Cursor Tab while typing in comments
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The "Suggestions While Commenting" switch, with the description "Allow Tab to trigger while in a comment region". Off by default. It is stored as an app flag (`cppTriggerInComments`), not a settings.json key.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`label:"Suggestions While Commenting"`, stored flag `cppTriggerInComments`)

### AC-8.13 Allow whitespace-only suggestions
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The "Whitespace-Only Suggestions" switch, with the description "Suggest edits like new lines and indentation that modify whitespace only". Off by default. Stored as an app flag (`cppShowWhitespaceOnlyChanges`).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`label:"Whitespace-Only Suggestions"`, stored flag `cppShowWhitespaceOnlyChanges`)

### AC-8.14 Turn Tab auto imports on or off
- Tag: cursor
- Evidence: bundle
- Boundary: no - the switch only; the import hint it controls is AC-7.
- What: The "Imports" switch, with the description "Automatically import necessary modules for TypeScript", or "...for TypeScript and Python" when the server config already includes Python. The row is hidden when the server has turned auto import off and you never set it. Off by default. Switching it off also dismisses an import hint that is on screen. Stored as an app flag (`cppAutoImportEnabled`).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`label:"Imports"`, `hideShownImport`, stored flags `cppAutoImportEnabled` and `backendHasDisabledCppAutoImport`)

### AC-8.15 Turn on Auto Import for Python (BETA)
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The "Auto Import for Python" switch, with a small "BETA" badge beside the label and the description "Enable auto import for Python. This is a beta feature." It shows only when the Imports row is shown and on, and the server config does not already include Python. Off by default. Stored as an app flag (`userHasEnabledImportPredictionForPython`).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`label:"Auto Import for Python"` with `children:"BETA"`, stored flag `userHasEnabledImportPredictionForPython`)

### AC-8.16 Set ignored file patterns for Cursor Tab
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The "Ignored Files" row, with the description "Glob patterns for files where Cursor Tab will not suggest" and an editable pattern list field (aria label "Ignored Files", placeholder "e.g., *.md, **/generated/**"), aligned to the top of the row. A file that matches is skipped, and the status item shows its dimmed, struck-through look with the text still "Cursor Tab". This is separate from the cursor ignore files, which give the text "Cursor Tab (Ignored)" (ED-8.30 holds the ignore settings). Stored as an app flag (`cppIgnorePatterns`).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`label:"Ignored Files"`, `placeholder:"e.g., *.md, **/generated/**"`, `isCppEnabledForEditor` reading `cppIgnorePatterns`)

### AC-8.17 Use Cursor Tab state in key rules
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Four context keys can be used in `when` clauses in keybindings: `cppEnabled` ("Whether Cursor Tab is currently enabled", default true), `cppSnoozed` ("Whether Cursor Tab is currently snoozed"), `cppSuggestion` (a Cursor Tab suggestion exists and partial accepts is on) and `cpp.shouldAcceptTab` (whether the Tab key should accept; always true when the accept command is rebound away from Tab; the Search view and Problems view also read it to catch Tab). Snooze uses `cppEnabled` and `cppSnoozed` as preconditions and Unsnooze uses `cppSnoozed`. No default key is bound to Enable, Disable, Toggle, Snooze, Unsnooze or Log In. The Tab accept key and the word-accept key are listed here as entry points only, the actions are AC-6. In the Keys list below, entries starting with "when" are context keys, not key chords.
- Commands: editor.cpp.snooze; editor.cpp.unsnooze; editor.action.acceptCursorTabSuggestion; editor.action.inlineSuggest.acceptNextWord
- Menus: none found
- Keys: Tab (editor.action.acceptCursorTabSuggestion, when cpp.shouldAcceptTab is true, weight 500); CtrlCmd+Right (editor.action.inlineSuggest.acceptNextWord, when editor.writable and (inlineSuggestionVisible or cppSuggestion)); when cppEnabled; when cppSnoozed; when cppSuggestion; when cpp.shouldAcceptTab
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`new Dt("cppSnoozed",...)` and `new Dt("cppEnabled",...)` at byte offset ~40643238; `cppSuggestion` key; accept command class with `kbOpts:{primary:ve.Tab,weight:500}` at ~40745786; `VKa="cpp.shouldAcceptTab"`; Search view and Problems view `onKeyDown` handlers)

### AC-8.18 Developer: Copy Recent Cursor Tab Actions
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the copied text is made of recorded Tab requests and their diffs, which are model output. Only the command is cataloged.
- What: Developer command "Developer: Copy Recent Cursor Tab Actions" (original English "Copy Recent Cursor Tab Actions"). It copies the newest ten Tab requests (request id and recorded diff) to the clipboard and shows "Copied N Cursor Tab request IDs + diffs to clipboard". When there are none it shows the error "No Cursor Tab actions found". It is in the Command Palette for everyone, no `when`.
- Commands: workbench.action.copyRecentCppActions
- Menus: CommandPalette (f1 true)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class with `ID="workbench.action.copyRecentCppActions"` at byte offset ~32980061; strings "No Cursor Tab actions found", "Copied ... Cursor Tab request IDs + diffs to clipboard")

### AC-8.19 Developer: Report Cursor Tab Action
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the first list shows recorded Tab output as row descriptions. Only the pickers are cataloged.
- What: Developer command "Developer: Report Cursor Tab Action" (the registered original text is "See Recent Cursor Tab Actions"). It opens a quick pick with the placeholder "Select a Cursor Tab request (last 20 requests)", rows named "{request id} - {time}". Picking a row opens a second quick pick with the placeholder "Choose an action": "Report Good", "Report Bad", "Report with Comment" (asks for text in an input box, placeholder "Type your comment here..."), "Copy Request ID" (greyed with "Disabled in privacy mode" when privacy mode is on) and, for developer users only (context key `isDevUser`), "Open in Prompt Quality". Fails with "No Cursor Tab actions found" when the list is empty. The row preview box next to these rows is QA-6.
- Commands: workbench.action.reportCppAction
- Menus: CommandPalette (f1 true)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class with `ID="workbench.action.reportCppAction"` at byte offset ~32980497; picker function with the two placeholders; `isDevUser` context key)

## Handed off
- Accept Cursor Tab Suggestion on Tab, the partial accept action (the Ctrl+Right behavior), Esc to reject, and the Tab handlers in the Search and Problems views: AC-6.
- Context keys `tabPredictionGoToFileVisibleKey` and `tabPredictionCodePreviewVisibleKey`, and the import hint that the Imports switch controls: AC-7.
- Hiding or showing the "Cursor Tab" item from the status bar right-click menu: SS-1 (the item itself is cross-listed in SS-6).
- The Tab Stats status item next to it: SS-6.
- The Cursor Settings editor, its open command, and the Code Intelligence tab frame with the ignore rows: ED-8 (ED-8.19 and ED-8.30).
- The preview box that appears beside the rows of the Report Cursor Tab Action picker: QA-6.
- The upgrade flow that the "Upgrade here." link starts: DL-6 (not traced).
- Domain rows "Cursor Tab" and "Cursor Tab (regional)" in a list of required domains with purposes: ED-8 Browser and Network tab (likely, the list was not traced to its screen).

## Unclear
- The Cursor Settings Tab section has no Model row. The Model select exists only in the status panel (AC-8.5), so the ask for a Model row in Cursor Settings found nothing to list.
- No first-run onboarding step for Tab was found. The onboarding pages in the main bundle are "welcome", "marketplace" and "data-sharing". The only trace is a hidden command `cursorai.action.cppAfterOnboarding` (title "Cpp After Onboarding", not in the palette) that refreshes the Tab config, and no caller of it was found. Not listed as a feature. If the Agents window onboarding has a Tab step, it is out of scope there. ED-9 owns first-run screens.
- The snooze row labelled "3 hours" is coded as one hour times 4, so it snoozes for 4 hours. Seen in code only, not run.
- `editor.cpp.openPro` ("Open Cursor Tab Pro Pricing", opens the pricing page) is hidden from the palette (f1 false) and no caller was found, so it is not listed.
- `workbench.action.disableAutocompletes` is a constant with no command, label or caller. A table maps the Snooze, Unsnooze, Toggle, Disable and Enable commands to a group named `tab_controls`; what reads the table was not found.
- In AC-8.17 the Keys list mixes key chords with `when <context key>` entries, because the brief has no field for context keys. A tool parsing this field should treat entries starting with "when" as context keys.
- The bundle has no module name marker near most of the Cursor Tab code (none was found after byte offset 40,000,000, where the status item, snooze and login code sit), so Source cites byte offsets in `workbench.desktop.main.js` of Cursor 3.27.12. They move with the build.
- Nothing was run on screen. The panel row order, the check-mark look of the checkboxes and the exact hover timing come from code only. The item was seen running in the status bar by the status bar pass, as the plain text "Cursor Tab".