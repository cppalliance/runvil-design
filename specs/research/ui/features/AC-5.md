Group: AC-5 Terminal suggestions. Section: Autocomplete (AC). Date: 2026-10-25. Pass: 2.
Features: 25.

### AC-5.1 Turn terminal suggestions on or off
- Tag: both
- Evidence: source
- Boundary: no
- What: Switch the terminal suggest popup on or off with one setting; it needs shell integration and only works in supported shells (Windows PowerShell, PowerShell 7+, zsh, bash, fish).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.enabled; terminal.integrated.shellIntegration.enabled
- Cursor: changed (setting present but default false and tagged preview; text lists PowerShell v7+, zsh, bash, fish and says shell integration must be true)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; C bundle (module terminalSuggestConfiguration.js, nls 10951)

### AC-5.2 See the suggest popup at the prompt cursor
- Tag: both
- Evidence: source
- Boundary: no
- What: A list opens next to the prompt cursor (below it, or above when there is no room, and flipped left near the right edge), 430 px wide and up to 12 rows tall by default; each row has a kind icon, the label with matched letters in bold, a faded detail text, and a right side description; it also shows in terminals in the editor area, and shows "Loading..." and "No suggestions." messages when opened by hand.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.fontFamily; terminal.integrated.fontSize; terminal.integrated.lineHeight; editor.fontWeight
- Cursor: changed (widget with the same 430 px layout and the two message strings is present; the first row is always focused on show, and no explicit-invoke context key or loading trigger was found)
- Source: V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts; V\src\vs\workbench\services\suggest\browser\simpleSuggestWidgetRenderer.ts; V\src\vs\workbench\services\suggest\browser\media\suggest.css; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts; C bundle (SimpleSuggestWidget class, nls 13099 to 13102)

### AC-5.3 Resize the popup and reset its size
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag the popup edges to resize it, the size is remembered across sessions, and a palette command puts it back to the default.
- Commands: workbench.action.terminal.resetSuggestWidgetSize
- Menus: CommandPalette (f1 defaults to true, title shows as Terminal: Reset Suggest Widget Size)
- Keys: none found
- Settings: none found
- Cursor: present (command id and title "Reset Suggest Widget Size" found; storage key terminal.integrated.suggestSize found)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (PersistedWidgetSize); V\src\vs\workbench\contrib\terminal\common\terminalStorageKeys.ts; C bundle (module terminal.suggest.js region, nls 10911)

### AC-5.4 Kind icons on every suggestion
- Tag: both
- Evidence: source
- Boundary: no
- What: Each row gets a colored symbol icon for its kind (file, folder, symbolic link file and folder, method for commands, alias, argument, option, option value, flag, commit, branch, tag, stash, remote, pull request, completed pull request, inline suggestion star, and a key icon for plain text); file and folder rows switch to file-icon-theme icons when the active theme has them; turning icons off hides them; each icon has a `terminalSymbolIcon.*Foreground` theme color.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.showIcons
- Cursor: changed (9 icons found: alias, argument, file, flag, folder, inline-suggestion, method, option, option-value, with matching colors; no symbolic link, commit, branch, tag, stash, remote, pull request or plain text icons; the showIcons setting is read by the terminal widget)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSymbolIcons.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\media\terminalSymbolIcons.css; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (kind to icon map); V\src\vs\workbench\services\suggest\browser\simpleSuggestWidgetRenderer.ts; C bundle (CSS codicon-terminal-symbol-*, nls 10933 to 10950)

### AC-5.5 Trigger suggestions by hand
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Space in the terminal to open the popup on demand; a manual open selects the first row, can fall back to files and folders, and shows "No suggestions." when empty; the same key toggles the details pane once the popup is open.
- Commands: workbench.action.terminal.triggerSuggest
- Menus: none found
- Keys: Ctrl+Space (mac: Ctrl+Space) when terminalFocus and terminalSuggestWidgetVisible is false and config.terminal.integrated.suggest.enabled is true
- Settings: terminal.integrated.suggest.enabled
- Cursor: changed (the id workbench.action.terminal.requestCompletions, title "Request Completions", is registered with the same Ctrl+Space key and the same when clause; triggerSuggest and "Trigger Suggest" for the terminal not found; in PowerShell with shell integration the Ctrl+Space key also sends the sequence ESC [24~e to the shell)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminal.suggest.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (requestCompletions); C bundle (module terminal.suggest.js region, nls 10910)

### AC-5.6 Show suggestions while typing
- Tag: both
- Evidence: source
- Boundary: no
- What: The popup opens by itself as you type, for the first word (commands), later words (arguments) or unclear spots (unknown, with files and folders as a fallback), and on trigger characters (a dash after a space, a slash after a folder, and any provider trigger character).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.quickSuggestions; terminal.integrated.suggest.suggestOnTriggerCharacters
- Cursor: changed (both settings present; defaults are commands on, arguments on, unknown off, and trigger characters true, where V defaults all off)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (_sync); C bundle (module terminalSuggestConfiguration.js, nls 10953 to 10957)

### AC-5.7 Move through the list
- Tag: both
- Evidence: source
- Boundary: no
- What: Down and Up move one row and wrap around at the ends, PageDown and PageUp move a page; Up on the first row, before you have pressed Down, closes the popup and sends Up to the shell so you walk command history (unless the history setting is off, then Up goes to the last row).
- Commands: workbench.action.terminal.selectNextSuggestion; workbench.action.terminal.selectPrevSuggestion; workbench.action.terminal.selectNextPageSuggestion; workbench.action.terminal.selectPrevPageSuggestion; workbench.action.terminal.hideSuggestWidgetAndNavigateHistory
- Menus: none found
- Keys: Down; PageDown; PageUp; Up when simpleSuggestWidgetHasNavigated or upArrowNavigatesHistory is false (previous row); Up when simpleSuggestWidgetHasNavigated is false and upArrowNavigatesHistory is true (history); all need terminalFocus and terminalSuggestWidgetVisible
- Settings: terminal.integrated.suggest.upArrowNavigatesHistory
- Cursor: present (all five ids found with the same keys and Up conditions; setting present)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts (selectNext, selectPrevious); C bundle (module terminal.suggest.js region, nls 10912 to 10922)
### AC-5.8 Accept a suggestion with Tab or a click
- Tag: both
- Evidence: source
- Boundary: no
- What: Tab inserts the focused row, replacing the typed piece of the word (and the rest of the word to the right), then closes the popup; clicking a row does the same without taking focus from the terminal.
- Commands: workbench.action.terminal.acceptSelectedSuggestion
- Menus: none found
- Keys: Tab when simpleSuggestWidgetHasFocusedSuggestion, terminalFocus and terminalSuggestWidgetVisible
- Settings: none found
- Cursor: changed (id and title "Insert" found with Tab and no focused-row condition; also placed on the popup status row as the left item "Insert (Tab)" through menu MenubarTerminalSuggestStatusMenu)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (acceptSelectedSuggestion); V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts (_onListMouseDownOrTap); C bundle (module terminal.suggest.js region, nls 10919)

### AC-5.9 Accept with Enter and optionally run the command
- Tag: both
- Evidence: source
- Boundary: no
- What: Enter inserts the focused row when it is allowed to (see selection mode), and the run-on-Enter setting decides whether Enter is also sent so the command runs (never, only when typed in full, same ignoring file extension, or always).
- Commands: workbench.action.terminal.acceptSelectedSuggestion; workbench.action.terminal.acceptSelectedSuggestionEnter
- Menus: none found
- Keys: Enter (acceptSelectedSuggestion) when simpleSuggestWidgetHasFocusedSuggestion and (simpleSuggestWidgetExplicitlyInvoked, or selectionMode is not partial, or the first row is not focused, or simpleSuggestWidgetHasNavigated); Enter (acceptSelectedSuggestionEnter) when config.terminal.integrated.suggest.runOnEnter is not never
- Settings: terminal.integrated.suggest.runOnEnter
- Cursor: changed (acceptSelectedSuggestionEnter and runOnEnter present with the same four run modes; the setting has an extra value "ignore", which is the default, and the Enter key rule is "runOnEnter is not ignore", so by default Enter goes straight to the shell; no Enter key on acceptSelectedSuggestion)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (runOnEnter switch); V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; C bundle (module terminalSuggestConfiguration.js, nls 10958 to 10963, 10920)

### AC-5.10 Choose the selection mode
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Pick how an auto-opened popup selects a row: partial (first row only outlined, Tab accepts, Enter accepts only after you press Down), always (first row filled, Tab or Enter accepts) or never (nothing selected until Down); a manual open always selects the first row; the status row button cycles none, partial, always and shows the current mode with a tooltip.
- Commands: workbench.action.terminal.changeSelectionModeNever; workbench.action.terminal.changeSelectionModePartial; workbench.action.terminal.changeSelectionModeAlways
- Menus: MenubarTerminalSuggestStatusMenu (group left, order 1; only the item matching the current mode shows, and only when quickSuggestions or suggestOnTriggerCharacters equals true as written)
- Keys: none found
- Settings: terminal.integrated.suggest.selectionMode
- Cursor: missing (searched the three ids, the setting id and the labels "Selection Mode: None", "Partial (Tab)" and "Always (Tab or Enter)"; none found; the popup always focuses the first row)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts (SuggestSelectionMode, partial-selection class); V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts

### AC-5.11 Add a trailing space after accepting
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When on, accepting a suggestion also types a space and opens suggestions again for the next word; folders and symbolic link folders never get the space.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.insertTrailingSpace
- Cursor: missing (searched the setting id and its text; none found; the accept code in the bundle has no space step)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (acceptSelectedSuggestion)

### AC-5.12 Close the popup
- Tag: both
- Evidence: source
- Boundary: no
- What: Esc closes the popup; it also closes on its own when the terminal scrolls, when you click in the terminal or focus leaves it (except into the details pane), when you type a space, when the typed text no longer matches anything, and when the command line is run.
- Commands: workbench.action.terminal.hideSuggestWidget
- Menus: none found
- Keys: Escape when terminalFocus and terminalSuggestWidgetVisible
- Settings: none found
- Cursor: present (id, title "Hide Suggest Widget" and Escape key found)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (hideSuggestWidget callers); C bundle (module terminal.suggest.js region, nls 10921)

### AC-5.13 Show and focus the details pane
- Tag: both
- Evidence: source
- Boundary: no
- What: Open a side pane with the focused row's detail text and docs (markdown), with a Close button and a "Loading..." state; it sits east, south or north of the list but never west, only opens for rows with docs or a detail that differs from the label, and its open state is remembered; a second key moves focus into the pane to scroll and copy text.
- Commands: workbench.action.terminal.suggestToggleDetails; workbench.action.terminal.suggestToggleDetailsFocus
- Menus: none found
- Keys: Ctrl+Space or Ctrl+I (mac: Ctrl+Space or Cmd+I) when terminalFocus, terminalIsOpen, terminalSuggestWidgetVisible and simpleSuggestWidgetHasFocusedSuggestion; Ctrl+Alt+Space (mac: Ctrl+Alt+Space) when not editorTextInputFocus
- Settings: none found
- Cursor: changed (both ids, titles and keys found; the details overlay has no placement restriction, so west is allowed)
- Source: V\src\vs\workbench\services\suggest\browser\simpleSuggestWidgetDetails.ts; V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts (toggleDetails, toggleDetailsFocus); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; C bundle (module terminal.suggest.js region, nls 10916 and 10917)

### AC-5.14 Explain mode
- Tag: both
- Evidence: source
- Boundary: no
- What: A developer aid that fills the details pane with the focused row's score, prefix, replacement range, index and provider name.
- Commands: terminalSuggestToggleExplainMode
- Menus: none found
- Keys: Ctrl+/ (mac: Cmd+/) when terminalFocus, terminalIsOpen and terminalSuggestWidgetVisible
- Settings: none found
- Cursor: present (id, title "Suggest Toggle Explain Modes" and Ctrl+/ key found)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\services\suggest\browser\simpleSuggestWidgetDetails.ts (renderItem explainMode); C bundle (module terminal.suggest.js region, nls 10915)
### AC-5.15 Status row at the bottom of the popup
- Tag: both
- Evidence: source
- Boundary: no
- What: A thin row of small buttons under the list, split into a left group and a right group, that can be hidden; in V it holds the selection mode switch on the left and the show-while-typing switch, Learn More and Configure on the right.
- Commands: none found
- Menus: MenubarTerminalSuggestStatusMenu
- Keys: none found
- Settings: terminal.integrated.suggest.showStatusBar
- Cursor: changed (row and setting present, default true; items are text buttons drawn as "Label (key)", with only two entries, Insert on the left and Configure on the right)
- Source: V\src\vs\workbench\services\suggest\browser\simpleSuggestWidget.ts; V\src\vs\editor\contrib\suggest\browser\suggestWidgetStatus.ts; V\src\vs\platform\actions\common\actions.ts (MenubarTerminalSuggestStatusMenu); V\src\vs\platform\actions\browser\menuEntryActionViewItem.ts; C bundle (status bar class Mqn, nls 10965)

### AC-5.16 Switch show-while-typing from the popup
- Tag: vscode
- Evidence: source
- Boundary: no
- What: An eye-closed icon button on the status row turns typing suggestions on (commands, arguments and trigger characters), and an eye icon button turns them off; the tooltip is a full sentence saying what changes.
- Commands: workbench.action.terminal.showSuggestOnType; workbench.action.terminal.doNotShowSuggestOnType
- Menus: MenubarTerminalSuggestStatusMenu (group right, order 1)
- Keys: none found
- Settings: terminal.integrated.suggest.quickSuggestions; terminal.integrated.suggest.suggestOnTriggerCharacters
- Cursor: missing (searched both ids and both sentences; none found)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminal.suggest.ts

### AC-5.17 Learn More and Configure buttons
- Tag: both
- Evidence: source
- Boundary: no
- What: A question mark button opens the terminal IntelliSense docs page, and a gear button opens Settings filtered to terminal.integrated.suggest.
- Commands: workbench.action.terminal.suggestLearnMore; workbench.action.terminal.configureSuggestSettings
- Menus: MenubarTerminalSuggestStatusMenu (group right, order 2 for Learn More, order 3 for Configure)
- Keys: none found
- Settings: none found
- Cursor: changed (Configure found with title "Configure", menu group right order 1, and a key Ctrl+Shift+, when the popup is visible; suggestLearnMore not found)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminal.suggest.ts; C bundle (module terminal.suggest.js region, nls 10909)

### AC-5.18 First-use highlight on the status buttons
- Tag: vscode
- Evidence: source
- Boundary: no
- What: For the first few uses (up to 10 shows, or 10 seconds) the first right-hand status button is drawn in the primary button colors to draw the eye to it; the state is stored for the whole app.
- Commands: workbench.action.terminal.resetDiscoverability
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing (searched the class increased-discoverability, the storage key and the command id; none found)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestShownTracker.ts; V\src\vs\workbench\services\suggest\browser\media\suggest.css; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminal.suggest.ts

### AC-5.19 Command and alias suggestions
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing the first word lists commands you can run, with shell built-ins and functions and programs on PATH shown as "Method" rows and shell aliases as "Alias" rows; on Windows the listed executable file types can be changed.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.windowsExecutableExtensions; terminal.integrated.suggest.providers
- Cursor: present (kinds Method and Alias, labels "Method" and "Alias", icons and the windowsExecutableExtensions setting found; the bundled terminal-suggest extension emits Method and Alias)
- Source: V\extensions\terminal-suggest\src\terminalSuggestMain.ts; V\extensions\terminal-suggest\src\shell\common.ts; V\extensions\terminal-suggest\src\shell\pwsh.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionItem.ts; C bundle (extension terminal-suggest dist\terminalSuggestMain.js, nls 10925 and 10926)

### AC-5.20 Argument, option and flag suggestions
- Tag: both
- Evidence: source
- Boundary: no
- What: After a known command, the popup lists its subcommands and arguments, options, option values and flags, each with its own kind label and icon and a description in the details pane.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.quickSuggestions; terminal.integrated.suggest.providers
- Cursor: present (kinds Argument, Option, OptionValue and Flag, labels, icons and colors found; the bundled extension emits all four)
- Source: V\extensions\terminal-suggest\src\terminalSuggestMain.ts; V\extensions\terminal-suggest\src\fig\figInterface.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (kind labels); C bundle (extension terminal-suggest dist\terminalSuggestMain.js, nls 10927 to 10930)

### AC-5.21 File and folder suggestions
- Tag: both
- Evidence: source
- Boundary: no
- What: When a path is expected, the popup lists files and folders (and symbolic links) from the working directory or the typed path, plus `../` and `~` entries, and with the cd command it can list children of the $CDPATH folders.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.cdPath; terminal.integrated.suggest.quickSuggestions
- Cursor: changed (File and Folder kinds, labels, icons and the cdPath setting found; symbolic link file and folder kinds not found)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionService.ts (resolveResources); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionItem.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; C bundle (nls 10923, 10924, 10966 to 10969)

### AC-5.22 Git suggestions
- Tag: both
- Evidence: source
- Boundary: no
- What: Git and GitHub CLI commands list branches, tags, commits, stashes, remotes and pull requests as their own kinds, each with a git icon (branch, tag, commit, stash, remote, pull request, completed pull request) and a color.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.providers
- Cursor: changed (a git command spec with branch lookups is in the bundled extension, drawn with the fig://icon?type=git icon; the branch, tag, commit, stash, remote and pull request kinds, icons and colors, and any gh spec, were not found)
- Source: V\extensions\terminal-suggest\src\completions\git.ts; V\extensions\terminal-suggest\src\completions\gh.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionItem.ts (kind enum); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSymbolIcons.ts; C bundle (extension terminal-suggest dist\terminalSuggestMain.js)

### AC-5.23 Shell inline suggestion row
- Tag: both
- Evidence: source
- Boundary: no
- What: When the shell already shows its own gray inline suggestion (zsh, fish, bash, PowerShell, Git Bash) the popup adds a row labelled "Inline Suggestion" with a star icon, keeps it on top or just below an exact match, and the Right arrow key accepts it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.inlineSuggestion
- Cursor: present (kind, label "Inline Suggestion", star icon, color and setting with the same three values found)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalSuggestAddon.ts (_inlineCompletion, _refreshInlineCompletion); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionModel.ts; V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts; C bundle (nls 10931, 10932, 10970 to 10973)

### AC-5.24 Turn suggestion providers on or off
- Tag: both
- Evidence: source
- Boundary: no
- What: One settings object lists every suggestion provider by id with an on or off value, so a whole category can be dropped from the popup; the language server provider (Python terminals only) is off by default and the others are on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.suggest.providers
- Cursor: changed (setting present; its default lists terminal-suggest and pwsh-shell-integration, both true, and no lsp entry was found)
- Source: V\src\vs\workbench\contrib\terminalContrib\suggest\common\terminalSuggestConfiguration.ts (registerTerminalSuggestProvidersConfiguration); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminal.suggest.contribution.ts (TerminalSuggestProvidersConfigurationManager); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\terminalCompletionService.ts (_getEnabledProviders); V\src\vs\workbench\contrib\terminalContrib\suggest\browser\lspTerminalUtil.ts; C bundle (module terminalSuggestConfiguration.js, nls 10952)

### AC-5.25 Clear cached shell globals
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A palette command, Terminal: Clear Suggest Cached Globals, throws away the saved list of shell built-ins and aliases so the next popup reads them fresh.
- Commands: terminal.integrated.suggest.clearCachedGlobals
- Menus: CommandPalette (extension command with no menus entry, so it is listed by default)
- Keys: none found
- Settings: none found
- Cursor: missing (searched the id in the bundle and in the bundled terminal-suggest extension, and its package.json has no contributes block; none found)
- Source: V\extensions\terminal-suggest\package.json; V\extensions\terminal-suggest\package.nls.json; V\extensions\terminal-suggest\src\terminalSuggestMain.ts

## Handed off

- The empty terminal line hint "Show suggestions {key}." (clickable, runs the trigger command): PN-3 (V\src\vs\workbench\contrib\terminalContrib\inlineHint\browser\terminal.initialHint.contribution.ts).
- The terminal accessibility help lines that list the suggest keys: PN-3 (V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminalAccessibilityHelp.ts; the Cursor bundle has the same help, with the request completions id).
- PowerShell Ctrl+Space and related send-sequence key bindings that reach the shell script (MenuComplete and the F12 mappings): PN-3 (V\src\vs\workbench\contrib\terminalContrib\sendSequence\browser\terminal.sendSequence.contribution.ts).
- The setting terminal.integrated.shellIntegration.environmentReporting, whose text says it feeds terminal suggest: PN-3 settings.

## Unclear

- Pass 1 said every terminal suggest command id carries the prefix `workbench.action.terminal.`. The explain mode id is `terminalSuggestToggleExplainMode` with no prefix, in V and in the Cursor bundle.
- Pass 1 guessed `workbench.action.terminal.requestCompletions` was an older name of the trigger command. The bundle shows it is registered with title "Request Completions", the Ctrl+Space key (mac Ctrl+Space) and the same when clause as V triggerSuggest, so it is the Cursor-side twin of AC-5.5. Nothing else in the Cursor bundle uses the V id triggerSuggest for the terminal.
- The Cursor terminal suggest code looks like an older VS Code snapshot: shell kinds limited to 9 (File, Folder, Method, Alias, Argument, Option, OptionValue, Flag, Inline Suggestion), no explicit-invoke context key, no selection mode, no discoverability, and a status row with text buttons. All read from minified code and strings, not run.
- `workbench.action.terminal.resetDiscoverability` is declared in V and the addon has a resetDiscoverability method, but no registered command or caller was found in V src.
- The status row selection mode items use `config.terminal.integrated.suggest.quickSuggestions equals true` in their when clause, but the setting is an object by default, so those items may never show unless the legacy boolean is set. Not run to confirm.
- There is no separate history provider. The only history related UI is the shell inline suggestion row (AC-5.23) and the Up arrow hand-off to shell history (AC-5.7). Whether the shell's gray text comes from history was not checked.
- Provider ids for the providers setting are registered at run time from extension identifiers; the ids in a live window were not read. Cursor's static default lists `terminal-suggest` and `pwsh-shell-integration`.
- N (1.144.0) was compared file by file for the terminal suggest folder, the simple suggest widget folder, the symbol icons, the css and the terminal-suggest extension manifest: no UI difference found, apart from language server cleanup code in the contribution file. Nothing is marked newer-than-1.128.
- Cursor Tab and the terminal popup: no link between them was searched for, so none is claimed.
