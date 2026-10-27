# 11. Quick access (QA)

6 groups, 117 features. Each feature lists its entry points and the pieces that finish it.

## QA-1 Command Palette

(both). The `>` list that runs any command by name, with history and a keybinding shortcut. Examples: Show All Commands (F1, Ctrl+Shift+P); recently used and other sections; the Configure Keybinding button; Clear Command History.

21 features.

#### QA-1.1 Open the Command Palette

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press F1 or Ctrl+Shift+P, or pick "Command Palette..." or "Show All Commands" from a menu, to open the list where you type a command name (placeholder "Type the name of a command to run.") and run it.
- Commands: workbench.action.showCommands
- Menus: MenubarViewMenu (1_open, "Command Palette..."); MenubarHelpMenu (1_welcome, "Show All Commands"); GlobalActivity (1_command, "Command Palette..."); EditorContext (z_commands, "Command Palette...", hidden in simple input editors); CommandPalette (implicit, f1)
- Keys: Ctrl+Shift+P (Cmd+Shift+P on Mac, not bound on Firefox); F1
- Settings: none found
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (273-295); V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (42-48, 55-108); C bundle (class `Btg` ShowAllCommandsAction with the same F1 and Ctrl+Shift+P keys, menu rows on MenubarViewMenu, MenubarHelpMenu, GlobalActivity and EditorContext, strings 8898 and 8903-8909)
- Pieces:
  1. Pressing `F1` or `Ctrl+Shift+P` (`Cmd+Shift+P` on macOS) opens the Command Palette with a list of commands (`workbench.action.showCommands`).
  2. The palette input shows the placeholder "Type the name of a command to run."
  3. Choosing a command row with `Enter` or a click runs that command and closes the palette.
  4. Choosing View, then "Command Palette..." from the menu bar opens the palette, and so does Help, then "Show All Commands".
  5. Choosing "Command Palette..." from the Manage gear menu opens the palette.
  6. Right-clicking in a text editor and choosing "Command Palette..." opens the palette, and the entry is hidden in simple input editors.

#### QA-1.2 Type to filter commands by name

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Typing narrows the list to commands whose name (shown as "Category: Title") holds your words or text, or whose id you typed in full, with the matching letters highlighted and a "No matching commands" row when nothing fits.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (49-60, 99-125); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (84-88, 237-243); C bundle (base provider class `Kke` with the same word filter, provider class `aqe`, strings 8892 and 8894)
- Pieces:
  1. Typing narrows the list to commands whose name contains your text or matches your words, with the matching letters highlighted.
  2. Each row shows its command name as "Category: Title".
  3. Typing a command's full id lists that command even when its name does not match.
  4. When nothing fits, the list shows a "No matching commands" row.

#### QA-1.3 Switch to commands by typing > in the Quick Open box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Type > at the start of the Go to File box to turn it into the Command Palette, and delete it to go back; the same list opens straight away when a command or keybinding runs quick access with the argument ">". The other prefixes are QA-5.9.
- Commands: workbench.action.quickOpen (argument prefix ">"); workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (51); V\src\vs\platform\quickinput\browser\quickAccess.ts (210-225); V\src\vs\workbench\browser\actions\quickAccessActions.ts (125-152); C bundle (prefix constant on `Kke`, quick access controller method `doShowOrPick`, Go to File action with a prefix argument, string 3228)
- Pieces:
  1. Typing `>` at the start of the Go to File box (`workbench.action.quickOpen`) turns the list into the Command Palette.
  2. Deleting the `>` turns the box back into the Go to File list.
  3. Running quick access with the argument `>` from a command or keybinding opens the command list straight away.

#### QA-1.4 See each command's keybinding on its row

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Each row shows the key combination bound to that command on its right edge, and a screen reader reads "name, keys".
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (272-302); C bundle (method `toCommandPick` on `Kke` with the keybinding lookup, string 2232)
- Pieces:
  1. Each command row shows the key combination bound to that command at its right edge.
  2. A command with no key bound shows no key combination.
  3. A screen reader reads a row with a key as "name, keys".

#### QA-1.5 Find commands by their English name in another display language

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the display language is not English, each row also shows the English name on a second line, and typing the English name still finds the command.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (84-88, 244-252); V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (99-118, 296-300); C bundle (option `showAlias` set from the language variant in `aqe`, alias filter in `Kke`)
- Pieces:
  1. When the display language is not English, each row shows the English command name on a second line.
  2. Typing the English name still finds the command, with the match highlighted on the second line.
  3. When the display language is English, rows show no second line.

#### QA-1.6 Order results by recent use, then name, with Developer commands last

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (no Developer-last rule because the bundle's command rows have no category field; adds the Open Browser pin, see QA-1.21)
- What: Results list commands you ran before first, newest first, then (if the experimental list is on) suggested ones, then everything else A to Z, with commands in the "Developer" category pushed below the rest.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (152-211); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (254-263); C bundle (sort function inside `Kke._getPicks`, which ends with the Open Browser check and a plain label compare)
- Pieces:
  1. Commands you ran before come first in the results, newest first.
  2. When the experimental suggested list is on, suggested commands come next, in the order given by the list.
  3. All other commands follow, sorted A to Z by name.
  4. Commands in the "Developer" category are pushed below the other commands that were not run before. [stock only]
  5. Cursor sorts the commands that were not run before by name only, so Developer commands sit among the rest. [cursor only]

#### QA-1.7 See a recently used section at the top

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Commands you ran before appear first under a "recently used" heading, newest first, and they stay across restarts; the rest follow under an "other commands" heading.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (213-247, 304-311, 373-460, 470-486); C bundle (history class `tT` with storage keys `commandPalette.mru.cache` and `commandPalette.mru.counter`, strings 2227 and 2230)
- Pieces:
  1. Commands you ran before appear first under a "recently used" heading, newest first.
  2. The recently used list is kept when you close and reopen the app.
  3. The remaining commands follow under an "other commands" heading.
  4. The "recently used" heading shows only when the first matching row is a command you ran before.

#### QA-1.8 Remove a command from the recently used list

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: Rows in the recently used section have a close button ("Remove from Recently Used") that drops that command from history without running it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (284-337, 462-469); C bundle (searched for "Remove from Recently Used" and removeFromRecentlyUsed with zero hits; the bundle's history class has no remove method and its row builder adds no buttons)
- Pieces:
  1. Rows in the recently used section have a close button at the right with the tooltip "Remove from Recently Used". [stock only]
  2. Clicking the close button removes that command from the history and drops its row from the list without running it. [stock only]
  3. The close button sits to the right of other row buttons such as the Configure Keybinding gear. [stock only]

#### QA-1.9 Limit or turn off command history

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Set how many recent commands are remembered (default 50), or set 0 so history is off and no recently used section shows.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.commandPalette.history
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (496-501); V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (373-420, 488-497); C bundle (setting block with default 50 and minimum 0, string 4200)
- Pieces:
  1. Setting `workbench.commandPalette.history` sets how many recently used commands are remembered, with a default of 50 and a minimum of 0.
  2. A value of 0 turns the history off, so commands you run are not remembered and no "recently used" heading shows.
  3. Lowering the number trims the oldest commands from the remembered list.

#### QA-1.10 Clear Command History

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run "Clear Command History" from the palette, then confirm in a warning box ("Do you want to clear the history of recently used commands?", "This action is irreversible!", button "Clear") to empty the recently used list; it does nothing when history is off.
- Commands: workbench.action.clearCommandHistory
- Menus: CommandPalette (implicit, f1)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (297-328); V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (499-505); C bundle (class `OJk` ClearCommandHistoryAction, strings 8895 to 8897 and 8899)
- Pieces:
  1. Running "Clear Command History" (`workbench.action.clearCommandHistory`) from the Command Palette opens a warning box.
  2. The box asks "Do you want to clear the history of recently used commands?", says "This action is irreversible!" and has a Clear button.
  3. Clicking Clear empties the recently used list, so the "recently used" heading is gone the next time.
  4. Cancelling the box leaves the history unchanged.
  5. Running the command does nothing when the history is off (`workbench.commandPalette.history` set to 0).

#### QA-1.11 Reopen the palette with your last typed text

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With the setting on, opening the palette fills in the text from the last command you ran in this window, instead of starting empty.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: workbench.commandPalette.preserveInput
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (59-64, 94-104); V\src\vs\platform\quickinput\browser\quickAccess.ts (90-100, 228-233); V\src\vs\workbench\browser\workbench.contribution.ts (502-506); C bundle (`defaultFilterValue` getter in `aqe`, setting block, string 4201)
- Pieces:
  1. With setting `workbench.commandPalette.preserveInput` on, opening the palette fills in the text that was typed when you last ran a command in this window.
  2. With the setting off, the default, the palette opens with an empty input.
  3. The restored text is remembered only while the app is running, so after a restart the palette opens empty again.

#### QA-1.12 Configure a command's keybinding from its row

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Each row has a gear button ("Configure Keybinding") that closes the palette and opens the Keyboard Shortcuts editor already filtered to that command and its when condition.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (127-141); V\src\vs\workbench\services\preferences\browser\keybindingsEditorModel.ts (37-40); C bundle (gear button and trigger inside `aqe.getCommandPicks`, string 8893)
- Pieces:
  1. Each command row has a gear button with the tooltip "Configure Keybinding".
  2. Clicking the gear button closes the palette without running the command.
  3. After the palette closes, the Keyboard Shortcuts editor opens with its search filled in for that command and its when condition.

#### QA-1.13 List only commands that apply in the current context

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Commands that are disabled or whose when condition is false are left out, so the list changes with the focused editor, open folder, debug state and so on, and the focused text editor adds its own actions to the list.
- Commands: editor.action.* (every action the active text editor supports)
- Menus: CommandPalette; contributes.menus.commandPalette (git, markdown-language-features, references-view and simple-browser manifests)
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\actions\common\actions.ts (485-560, 751-754); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (224-266); V\src\vs\editor\contrib\quickAccess\browser\commandsQuickAccess.ts (35-60); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (697-707); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (113-120); V\extensions\git\package.json (contributes.menus.commandPalette, 175 rows, 70 set to false); C bundle (`aqe.getGlobalCommandPicks`, editor picks class `NJk`, extension point key `commandPalette`; the bundled git extension has 146 palette rows)
- Pieces:
  1. Commands that are disabled, or whose when condition is false, are left out of the list.
  2. The list changes with context such as the focused editor, the open folder and the debug state.
  3. When a text editor has focus, the actions that editor supports (`editor.action.*`) are added to the list.
  4. Commands that the git, markdown-language-features, references-view and simple-browser extensions mark as hidden from the palette never show.
  5. The first time the palette opens, it waits up to 800 ms for extensions to register so their commands are included.

#### QA-1.14 Tell apart commands that share a name

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When two commands have the same label, both rows show their command id as a description so you can tell them apart.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (139-150); C bundle (duplicate-label map inside `Kke._getPicks`)
- Pieces:
  1. When two or more commands share the same label, each of those rows shows its command id as a description next to the label.
  2. A command with a unique label shows no description.
  3. A command shows the id only while a command with the same label is also in the filtered list, so typing more text can remove the id.

#### QA-1.15 Hover a row to read the command description

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: Hovering a row shows the command's description text when the command has one (few do).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (272-298); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (250-262); C bundle (read the `toCommandPick` code in `Kke`: the row it returns has no tooltip field, only ariaLabel, detail, keybinding and accept)
- Pieces:
  1. Hovering a command row shows a tooltip with the command's description text when the command has one. [stock only]
  2. A command without a description shows no tooltip. [stock only]

#### QA-1.16 Similar commands from word similarity

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Once you type three or more characters, commands that do not match by name but share words with their name, English name or description are added under a "similar commands" heading, up to five.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (53-54, 85-97, 122-137, 226-232, 340-351); C bundle (tf-idf calculation inside `Kke._getPicks`, strings 2228 and 2231)
- Pieces:
  1. After you type 3 or more characters, commands that do not match by name but share words with their name, English name or description are added to the list.
  2. Those commands appear under a "similar commands" heading below the name matches.
  3. At most 5 similar commands show, with the closest match first.
  4. Typing fewer than 3 characters adds no similar commands.

#### QA-1.17 Related commands from natural language search

- Tag: both. Evidence: source. Boundary: yes - the extra rows come from an extension-supplied related-information service, not from the palette.. Cursor: present
- What: When such a service is installed and the setting is on, up to five more related commands are added under the same "similar commands" heading after a 200 ms pause in typing.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: workbench.commandPalette.experimental.enableNaturalLanguageSearch
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (51-52, 106-116, 143-156, 200-222); V\src\vs\workbench\browser\workbench.contribution.ts (530-535); C bundle (`aqe.getRelatedInformationPicks` and `useAiRelatedInfo`, setting block, string 4206)
- Pieces:
  1. When a related-information service is installed and enabled, up to 5 more related commands are added under the "similar commands" heading.
  2. The related commands are added only after a 200 ms pause in typing.
  3. Related commands are ordered by how closely they relate, and a command already in the list is not repeated.
  4. Setting `workbench.commandPalette.experimental.enableNaturalLanguageSearch`, on by default, turns the related commands on or off.
  5. With no text typed, no related commands are added.

#### QA-1.18 Commonly used section (experimental)

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With the setting on and a list of suggested command ids supplied by the product file, a "commonly used" heading lists those commands after recently used ones.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: workbench.commandPalette.experimental.suggestCommands
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (180-196, 213-240); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (108-116); V\src\vs\workbench\browser\workbench.contribution.ts (507-511); V\src\vs\base\common\product.ts; C bundle (`suggestedCommandIds` option set in `aqe.updateOptions`, setting block, string 4202)
- Pieces:
  1. With setting `workbench.commandPalette.experimental.suggestCommands` on and a list of suggested command ids supplied by the product file, a "commonly used" heading lists those commands.
  2. The "commonly used" section comes after the recently used commands and before the "other commands" heading.
  3. The setting is off by default.
  4. With the setting on but no list supplied, no "commonly used" heading shows.

#### QA-1.19 Ask in Chat row

- Tag: vscode. Evidence: source. Boundary: yes - the row sends your typed text to chat, whose UI is out of scope.. Cursor: missing
- What: With text typed, a row "Ask in Chat: <text>" under a separator opens the Chat view or Quick Chat with that text, and its gear button ("Configure visibility") opens the setting.
- Commands: workbench.action.chat.open; workbench.action.quickchat.toggle
- Menus: none found
- Keys: none found
- Settings: workbench.commandPalette.showAskInChat; workbench.commandPalette.experimental.askChatLocation
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (158-196); V\src\vs\workbench\browser\workbench.contribution.ts (512-529); V\src\vs\workbench\contrib\chat\browser\actions\chatActions.ts (81); V\src\vs\workbench\contrib\chat\browser\actions\chatQuickInputActions.ts (17); C bundle (searched "Ask in Chat" and showAskInChat with zero hits; `askChatLocation` is declared in the settings block but nothing reads it)
- Pieces:
  1. With text typed, the last row reads "Ask in Chat: <text>" and sits under a separator below the other results. [stock only]
  2. Choosing the row opens the Chat view with the typed text (`workbench.action.chat.open`). [stock only]
  3. Setting `workbench.commandPalette.experimental.askChatLocation` chooses where the row opens chat, with values `chatView` (the default) and `quickChat` (`workbench.action.quickchat.toggle`). [stock only]
  4. The row has a gear button with the tooltip "Configure visibility" that closes the palette and opens the settings editor on `workbench.commandPalette.showAskInChat`. [stock only]
  5. Setting `workbench.commandPalette.showAskInChat`, on by default, shows or hides the row. [stock only]
  6. The row appears only when the related-information service is enabled and `enableNaturalLanguageSearch` is on, and never when nothing is typed. [stock only]

#### QA-1.20 Hide Chat and Copilot commands from the list

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor removes any command whose English name starts with "Chat:" or "Ask Github Copilot", so the stock chat commands never show in the palette.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (array `uPk` and the filter on the alias at the top of `Kke._getPicks`)
- Pieces:
  1. In Cursor, commands whose English name starts with "Chat:" never show in the palette. [cursor only]
  2. Commands whose English name starts with "Ask Github Copilot" never show either.
  3. The rule uses the English name, so the commands stay hidden in any display language.

#### QA-1.21 Pin Open Browser above other unused commands

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A command named "Open Browser" is sorted ahead of other commands you have not run before, so it sits at the top of the "other commands" part.
- Commands: composer.openBrowserTab
- Menus: CommandPalette (implicit, f1)
- Keys: none found
- Settings: none found
- Source: C bundle (array `gDr` and the sort step in `Kke._getPicks`; command class `a5m` with title "Open Browser" and id `composer.openBrowserTab`)
- Pieces:
  1. The command "Open Browser" (`composer.openBrowserTab`) sorts ahead of the other commands you have not run before. [cursor only]
  2. "Open Browser" sits at the top of the "other commands" part of the list.
  3. Commands you ran before, and suggested commands, still come before it.


## QA-2 Go to File and workspace search

(both, Cursor modified). The Ctrl+P file list, plus `#` workspace symbols and `%` Quick Search text. Cursor adds a `search.quickOpen.useVscodeSearch` setting, an "other results" split, and a "Create File" row.

23 features.

#### QA-2.1 Open a file by name with Go to File

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (same command, keys and menu rows; placeholder reads "Search files, content, and symbols (append : to go to line or @ to go to symbol)" and the picks come from Cursor's own function, see QA-2.20)
- What: Press Ctrl+P, type part of a file name, and pick a row to open it. Each row shows a file icon, the name, the folder path, and a filled dot when the file has unsaved changes. With no matches the list shows "No matching results".
- Commands: workbench.action.quickOpen
- Menus: MenubarGoMenu (Go to File..., group 3_global_nav); EmptyEditorGroupContext (Open File...); EditorTabsBarContext (Open File...)
- Keys: Ctrl+P; Ctrl+E (Windows and Linux only); Cmd+P (Mac)
- Settings: search.exclude; files.exclude; search.useIgnoreFiles; search.useGlobalIgnoreFiles; search.useParentIgnoreFiles; search.followSymlinks; workbench.editor.customLabels.enabled
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts; V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\searchQuickAccess.contribution.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\services\search\common\queryBuilder.ts; C bundle (command, Eje key table, Go menu item, class vI, messages 9447 and 9448)
- Pieces:
  1. Pressing `Ctrl+P` (`Cmd+P` on Mac, and also `Ctrl+E` on Windows and Linux), typing part of a file name, and pressing Enter on a row opens that file and closes the list.
  2. Choosing Go to File... in the Go menu, or Open File... in the right-click menu of the empty editor area or of the tab bar, opens the same list.
  3. Each row shows a file icon, the file name, and the folder path next to it in grey.
  4. A row for a file that has unsaved changes is read out by screen readers with the words "unsaved changes" added.
  5. When nothing matches, the list shows "No matching results".
  6. Files that match `files.exclude` or `search.exclude` are left out of the results.
  7. Settings `search.useIgnoreFiles`, `search.useGlobalIgnoreFiles` and `search.useParentIgnoreFiles` each turn on or off hiding the files named in ignore files, and `search.followSymlinks` turns on or off searching inside symlinked folders.
  8. With `workbench.editor.customLabels.enabled` on, a file that has a custom label shows that label as its name and its full relative path as the description.
  9. In Cursor the box placeholder reads "Search files, content, and symbols (append : to go to line or @ to go to symbol)" and the rows come from Cursor's own ranking. [cursor only]

#### QA-2.2 See recently opened files first

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (present, but history rows under /.cursor/worktrees/ are hidden)
- What: With an empty box the list is the editor history under a "recently opened" heading. When you type, matching history rows stay above the "file results" heading and are left out of the file results below.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: search.quickOpen.includeHistory; search.quickOpen.history.filterSortOrder
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts; C bundle (getEditorHistoryPicks, messages 9418 and 9419, 9633 to 9636)
- Pieces:
  1. With an empty box the list shows the editor history under a "recently opened" heading.
  2. Once you type, the history rows that match stay above the "file results" heading, and those same files are left out of the file results below.
  3. Setting `search.quickOpen.includeHistory` (on by default) chooses whether history rows are included in the results while you type, and an empty box still lists history.
  4. Setting `search.quickOpen.history.filterSortOrder` chooses how history rows are ordered while you type, with values `default` (most relevant first) and `recency` (most recently opened first).
  5. History rows only match on the file name unless the typed text contains a slash, in which case they also match on the folder path.
  6. In Cursor, history rows for files under a `/.cursor/worktrees/` folder are left out of the list. [cursor only]

#### QA-2.3 Fuzzy match with highlighted letters

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Typed letters match out of order across the file name and folder path, and the matched letters are drawn highlighted in each row. A space starts a second search word, a double quote asks for an exact match, and a slash makes the match include the folder path.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\base\common\fuzzyScorer.ts; V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; C bundle (prepareQuery function Xz with the same values and exact-match fields, scoreItemFuzzy)
- Pieces:
  1. Typed letters match out of order across the file name and the folder path, and each row draws its matched letters highlighted.
  2. Typing a space starts a second search word, and a row stays in the list only if it matches every word.
  3. Wrapping the typed text in double quotes asks for an exact match instead of a fuzzy one.
  4. Typing a slash makes the match include the folder path and not only the file name.

#### QA-2.4 Open a file by typing its path

- Tag: both. Evidence: source. Boundary: no. Cursor: present (stock also fixes the file name casing against the disk, the bundle copy does not)
- What: Typing a full path, a path starting with ~, or a path relative to a workspace folder shows that exact file as a single result, even when search excludes would normally hide it.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; C bundle (getAbsolutePathFileResult, getRelativePathFileResults)
- Pieces:
  1. Typing a full path to a file shows that exact file as the only result row.
  2. Typing a path that starts with `~` shows the file from the home folder as the only result row.
  3. Typing a path relative to a workspace folder shows that file as the only result row.
  4. The single result shows even when `search.exclude` or `files.exclude` would normally hide the file, and all of its text is drawn highlighted.
  5. When the typed path uses different letter casing than the file on disk, the row shows the file name with the casing from disk. [stock only]

#### QA-2.5 Jump to a line and column with a suffix

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the bundle pattern has no end-of-range part, and the file opens with the cursor collapsed to the start)
- What: After a file name, type :12, :12:5, :12,5, #12 or (12) to open the file at that line and column. Stock also takes a range such as :20-40 or :20:3-40:5 and selects it. Changing only the suffix keeps the current results in place.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\common\search.ts; V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; C bundle (line-colon pattern G4o, openAnything with collapseToStart)
- Pieces:
  1. After a file name, typing `:12`, `#12` or `(12)` opens the file at line 12 when you pick it.
  2. After a file name, typing `:12:5` or `:12,5` opens the file at line 12, column 5.
  3. After a file name, typing a range such as `:20-40` or `:20:3-40:5` opens the file with that range selected. [stock only]
  4. Changing only the suffix after the file name keeps the current result rows in place instead of searching again.
  5. In Cursor the file opens with the cursor collapsed to the start of the target position. [cursor only]

#### QA-2.6 Go to a symbol inside a file you are filtering

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Type @ after a partial file name to list the symbols of the highlighted file. The file opens in a temporary tab, the active symbol is scrolled to the middle and highlighted, and Escape puts the old editor view back.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts; C bundle (getEditorSymbolPicks, doGetEditorSymbolPicks, openTransientEditor)
- Pieces:
  1. Typing `@` after a partial file name lists the symbols of the highlighted file in the same list.
  2. The highlighted file opens in a temporary tab behind the list without taking keyboard focus.
  3. The active symbol row scrolls to the middle of the editor and is highlighted there.
  4. Pressing Escape puts the earlier editor view back.
  5. Picking a symbol row opens the file with the cursor at that symbol.
  6. Deleting the `@` part of the text brings the earlier file list back.

#### QA-2.7 Open to the side or bottom

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Each file row has an "Open to the Side" button (it reads "Open to the Bottom" when the side direction is down). Alt+Enter does the same, and so does Ctrl+Enter while previews from Quick Open are off, which is the default. The same button sits on symbol rows in the # and @ lists.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: Alt+Enter (modifier read by the picker, no keybinding rule); Ctrl+Enter (same)
- Settings: workbench.editor.openSideBySideDirection; workbench.editor.enablePreviewFromQuickOpen
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (createAnythingPick, openAnything, messages 9427 and 9428)
- Pieces:
  1. Each file row has an "Open to the Side" button that opens the file in a split on the side and closes the list.
  2. Setting `workbench.editor.openSideBySideDirection` chooses the side, with values `right` and `down`, and with `down` the button reads "Open to the Bottom" and the file opens below.
  3. Pressing `Alt+Enter` on a row opens that file to the side.
  4. Pressing `Ctrl+Enter` on a row opens that file to the side while `workbench.editor.enablePreviewFromQuickOpen` is off, which is the default.
  5. The same Open to the Side button appears on the symbol rows of the `#` and `@` lists.

#### QA-2.8 Open files without closing the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Press Right Arrow on a row to open that file in the background as a pinned tab while the list stays open, so several files can be opened in a row. The provider turns this on for files, symbols and Quick Search matches.
- Commands: quickInput.acceptInBackground
- Menus: none found
- Keys: Right Arrow (when in quick pick and focus is in the list or the cursor is at the end of the input box)
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts; V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\contrib\welcomeGettingStarted\common\gettingStartedContent.ts; C bundle (quickInput.acceptInBackground registration)
- Pieces:
  1. Pressing Right Arrow on a file row opens that file in the background and the list stays open.
  2. The file opened this way becomes a pinned tab, not a preview tab.
  3. Focus stays in the list, so moving down and pressing Right Arrow again opens several files in a row.
  4. Right Arrow opens the row only when focus is in the list or the text cursor is at the end of the input, and otherwise it moves the text cursor.
  5. Symbol rows and Quick Search match rows also open in the background with Right Arrow.

#### QA-2.9 Open as a preview tab or a pinned tab

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: By default a file picked from the list opens as a pinned tab. With the preview setting on it opens as a preview tab (italic, replaced by the next one) and holding Ctrl while picking pins it instead.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: Ctrl+Enter (pins when the preview setting is on)
- Settings: workbench.editor.enablePreviewFromQuickOpen; workbench.editor.enablePreview
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (openAnything pinned option, setting key in the main bundle)
- Pieces:
  1. By default a file picked from the list opens as a pinned tab.
  2. With `workbench.editor.enablePreviewFromQuickOpen` on, a picked file opens as a preview tab, drawn in italics and replaced by the next preview.
  3. With that setting on, holding Ctrl while picking a file (`Ctrl+Enter`) opens it as a pinned tab instead.
  4. With `workbench.editor.enablePreview` off, preview tabs are off everywhere, so a picked file is pinned even when `workbench.editor.enablePreviewFromQuickOpen` is on.

#### QA-2.10 Remove a file from recently opened

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Rows that come from history have an X button, tooltip "Remove from Recently Opened", that drops the file from the list and from history. A row with unsaved changes shows an always-visible filled dot in the same spot.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; C bundle (createAnythingPick, message 9429 and 9430)
- Pieces:
  1. A row that comes from editor history has an X button with the tooltip "Remove from Recently Opened".
  2. Clicking the X removes that row from the list and removes the file from the editor history.
  3. A history row for a file with unsaved changes shows a filled dot in place of the X, and the dot is always visible instead of only on hover.
  4. Rows that are file results and not history have no X button.

#### QA-2.11 Add workspace symbols to the file results

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With the setting on, symbols from language servers are scored together with files and shown under the "file and symbol results" heading in the same list. Only top-level kinds (classes, enums, interfaces, namespaces, modules, packages, files) or symbols with no container are used, and none are added while a line suffix is typed.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: search.quickOpen.includeSymbols
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\symbolsQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts; C bundle (getWorkspaceSymbolPicks, messages 9424, 9631)
- Pieces:
  1. With `search.quickOpen.includeSymbols` on (it is off by default), symbols from language servers are scored together with files in the same list.
  2. The heading over the combined results reads "file and symbol results" in place of "file results".
  3. Only symbols of top-level kinds (classes, enums, interfaces, namespaces, modules, packages and files) or symbols with no container are added.
  4. No symbols are added while a line suffix such as `:12` is typed.
  5. Picking a symbol row opens its file with the cursor at that symbol.

#### QA-2.12 Go to Symbol in Workspace

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (same command, key, menu item and placeholder; older copy where the kind icon is part of the label text, and the rows have no shift-to-attach callback)
- What: Ctrl+T, or typing #, lists symbols from all language servers. The box starts filled with the selected or nearby word, each row shows a kind icon, the name, and "container, bullet, relative path", deprecated symbols are struck through, and a second search word filters by container. Enter opens the file with the cursor at the symbol start.
- Commands: workbench.action.showAllSymbols
- Menus: MenubarGoMenu (Go to Symbol in Workspace..., group 3_global_nav, order 2)
- Keys: Ctrl+T; Cmd+T (Mac)
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\symbolsQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\searchActionsSymbol.ts; V\src\vs\workbench\contrib\search\browser\searchQuickAccess.contribution.ts; C bundle (class ome, showAllSymbols action, messages 9449 to 9450 and 9491 to 9493)
- Pieces:
  1. Pressing `Ctrl+T` (`Cmd+T` on Mac) or typing `#` in the quick access box lists symbols from all language servers and narrows them as you type.
  2. Choosing Go to Symbol in Workspace... in the Go menu opens the same list.
  3. When opened with the command, the box starts filled with the one-line text selected in the focused editor, or with the word at its cursor when nothing is selected.
  4. Each row shows a kind icon, the symbol name, and a grey description made of the container name, a bullet, and the relative path of its file.
  5. Deprecated symbols are drawn struck through.
  6. With a second search word, the first word matches the symbol name and the rest match the container and path text.
  7. Pressing Enter on a row opens the file with the cursor at the start of the symbol.
  8. A search with no hits shows "No matching workspace symbols".
  9. In Cursor the list is an older copy where the kind icon is written into the label text of each row. [cursor only]

#### QA-2.13 Leave chosen extensions out of # symbol search

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A setting that takes a list of extension ids whose symbol providers are skipped when the # list searches for symbols. It takes effect as soon as it changes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: search.quickAccess.excludeSymbolProviders
- Source: C bundle (class ome, _refreshExcludedProvidersFromConfig, message 9680); not in V or N
- Pieces:
  1. Setting `search.quickAccess.excludeSymbolProviders` takes a list of extension ids, empty by default, and symbols supplied by those extensions do not appear in the `#` list. [cursor only]
  2. Extension ids in the list match without regard to upper or lower case. [cursor only]
  3. Changing the setting takes effect on the next search without reopening the list or reloading the window. [cursor only]

#### QA-2.14 Search text across files with Quick Search

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Typing % lists text matches in the workspace, grouped under one heading per file (file name and folder), with the matched text highlighted in each line. It shows up to 30 files and 10 matches per file, then "More" and "See More Files" rows. Run from the command, it starts with the selected editor text.
- Commands: workbench.action.quickTextSearch
- Menus: CommandPalette (Search: Quick Search, f1 only)
- Keys: none found
- Settings: search.maxResults; search.smartCase; search.sortOrder; editor.find.seedSearchStringFromSelection
- Source: V\src\vs\workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\searchActionsTextQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts; V\src\vs\workbench\contrib\search\common\constants.ts; C bundle (QuickTextSearchActionId, Quick Search class, messages 9437 to 9443, 9451, 9452, 9494)
- Pieces:
  1. Typing `%` in the quick access box lists text matches from the workspace, with the matched text highlighted in each line.
  2. Matches are grouped under one heading row per file that shows the file name and folder, and the file of the active editor is listed first.
  3. The list shows up to 30 files, followed by a "See More Files" row, and up to 10 matches per file, followed by a "More" row.
  4. Pressing Enter on a match row opens its file with the match selected, and `Alt+Enter` opens it to the side.
  5. Running Search: Quick Search from the Command Palette starts with the selected text of the focused editor already typed in when `editor.find.seedSearchStringFromSelection` is on.
  6. With an empty box the list shows "Enter a term to search for across your files."
  7. A search with no hits shows "No matching results".
  8. Setting `search.maxResults` limits how many matches the search collects, and a value of `null` means no limit.
  9. Setting `search.smartCase` makes an all-lowercase search ignore case and a search with capitals match case.
  10. Setting `search.sortOrder` chooses the order of the file groups, with values `default`, `fileNames`, `type`, `modified`, `countDescending` and `countAscending`.

#### QA-2.15 Preview each Quick Search match as you move

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Moving through the match rows opens that file in a temporary tab at the match, without taking focus, after a short pause of 75 ms. Escape puts the earlier editor view back, and accepting clears the match highlights.
- Commands: workbench.action.quickTextSearch
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts; C bundle (Quick Search provide with debounce and openTransientEditor)
- Pieces:
  1. Moving to a match row opens that file in a temporary tab at the match, without taking keyboard focus.
  2. The file opens only after a 75 ms pause in moving, so stepping quickly through rows does not open every file.
  3. Pressing Escape puts the earlier editor view back.
  4. While results are listed, the matching text is highlighted in the open editors, and accepting a match clears those highlights.

#### QA-2.16 Move Quick Search results into the Search view

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (input button behaves differently when the result list is empty)
- What: An "Open in Search View" button at the right of the input, one on each match row, an "Open File" button on each file heading, and the "More" and "See More Files" rows all hand the results to the Search view (SB-3) and close the list. In stock the input button always moves the search over, and in Cursor it just opens the view when there are no results yet.
- Commands: workbench.action.quickTextSearch
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts; C bundle (Quick Search provide, moveToSearchViewlet, messages 9437 to 9441)
- Pieces:
  1. The "Open in Search View" button at the right of the input moves the search into the Search view and closes the list.
  2. The "Open in Search View" button on a match row opens the Search view with that match focused and closes the list.
  3. The "More" and "See More Files" rows open the Search view with the next hidden match or file focused.
  4. The "Open File" button on a file heading opens that file and closes the list.
  5. In Cursor the input button only opens the Search view when the result list is still empty, and does not move a search over. [cursor only]

#### QA-2.17 Remember the last typed text

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Two settings bring back the last text typed in Go to File, and separately in Quick Search, the next time the list opens. Both are off by default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.quickOpen.preserveInput; search.quickAccess.preserveInput
- Source: V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts; V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; V\src\vs\workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts; C bundle (configuration getters in classes vI and the Quick Search class)
- Pieces:
  1. Setting `workbench.quickOpen.preserveInput` (off by default) brings back the last text typed in Go to File when the list opens again.
  2. Setting `search.quickAccess.preserveInput` (off by default) brings back the last text typed in Quick Search when it opens again.
  3. Restored text is shown selected, so typing replaces it.
  4. Go to File and Quick Search each keep their own last text, and text from one does not appear in the other.

#### QA-2.18 Open Go to File with the previous file ready

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A command, with no default key or menu item, that opens Go to File with the second history row active, so Enter reopens the file you were just in.
- Commands: workbench.action.quickOpenPreviousEditor
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts; C bundle (command registration with ItemActivation.SECOND)
- Pieces:
  1. Running `workbench.action.quickOpenPreviousEditor` opens Go to File with an empty box and the editor history listed.
  2. The second history row is the active row, so pressing Enter reopens the file you were just in.

#### QA-2.19 Keep the list open when focus leaves

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: By default every quick list closes when it loses focus. Turning the setting off keeps Go to File, and every other quick pick, open until you press Escape or pick something.
- Commands: workbench.action.closeQuickOpen
- Menus: none found
- Keys: Escape; Shift+Escape (closeQuickOpen, when in a quick pick)
- Settings: workbench.quickOpen.closeOnFocusLost
- Source: V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\services\quickinput\browser\quickInputService.ts; V\src\vs\workbench\browser\actions\quickAccessActions.ts; C bundle (setting in the main bundle, closeQuickOpen key rule)
- Pieces:
  1. By default every quick list, including Go to File, closes when focus moves out of it.
  2. With `workbench.quickOpen.closeOnFocusLost` off, Go to File and every other quick pick stay open until you press Escape or pick something.
  3. Pressing `Escape` or `Shift+Escape` while a quick list is open closes it.

#### QA-2.20 Split results into file results and other results

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor ranks Go to File results in two lists. Up to 6 rows scoring 100 or more go under "file results" (or "file and symbol results"), and every other match goes under "other results", up to 512 rows in all. With several search words, a high-scoring row whose matched runs average 3 letters or fewer is dropped from the results instead of shown.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: search.quickOpen.useVscodeSearch
- Source: C bundle (doGetCursorPicks, getCursorAdditionalPicks, constants HIGH_SCORING_THRESHOLD 100 and HIGH_SCORING_CAP_ITEMS 6, messages 9416 and 9424 and 9425)
- Pieces:
  1. The first group, headed "file results" (or "file and symbol results" when symbols are included), holds at most 6 matches, those that score 100 or more. [cursor only]
  2. Every other match goes under an "other results" heading, with at most 512 rows in the whole list. [cursor only]
  3. With several search words, a high-scoring row whose matched runs of letters average 3 letters or fewer is dropped from the results instead of shown. [cursor only]
  4. The two-group list applies only while `search.quickOpen.useVscodeSearch` is off, which is its default. [cursor only]

#### QA-2.21 Switch Go to File back to VS Code's own search

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A setting, off by default, that makes Go to File use the stock algorithm instead of Cursor's modified one. The list then has a single "file results" group and no Create File row.
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: none found
- Settings: search.quickOpen.useVscodeSearch
- Source: C bundle (configuration getter useVscodeSearch, doGetPicks choosing doGetVscodePicks or doGetCursorPicks, message 9637)
- Pieces:
  1. Setting `search.quickOpen.useVscodeSearch`, off by default, when turned on makes Go to File use VS Code's own search instead of Cursor's modified one. [cursor only]
  2. With the setting on, the list has a single "file results" group in place of the "file results" and "other results" groups. [cursor only]
  3. With the setting on, no Create File row appears when nothing matches. [cursor only]

#### QA-2.22 Create a file from the Go to File box

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: When nothing matches and the typed text looks like a path (absolute, or contains a slash, or ends in an extension such as .ts, and does not end in a slash), the list shows "No matching results" and a row "Create File '<path>'" with the note "Create a new file at this path". Enter makes the empty file (and any missing folder, under the first workspace folder for a relative path) and opens it. Alt or Ctrl opens it to the side. A failure shows "Failed to create file".
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: Enter; Alt+Enter; Ctrl+Enter
- Settings: none found
- Source: C bundle (isValidFilePath, createFilePicksForPath, createNewFile, CREATE_FILE_PICK_ID, messages 9431 to 9433); not in V or N
- Pieces:
  1. When nothing matches and the typed text looks like a path, the list shows "No matching results" and a row "Create File '<path>'" with the note "Create a new file at this path". [cursor only]
  2. Typed text counts as a path when it is absolute, contains a slash, or ends in an extension such as `.ts`, and it does not end in a slash. [cursor only]
  3. Pressing Enter on the Create File row makes the empty file, makes any missing folders, and opens the new file. [cursor only]
  4. A relative path is created under the first workspace folder. [cursor only]
  5. When a file already exists at the typed path, pressing Enter on the Create File row opens it and leaves it unchanged. [cursor only]
  6. Pressing `Alt+Enter` or `Ctrl+Enter` on the Create File row opens the new file to the side. [cursor only]
  7. When the file cannot be made, a "Failed to create file" message shows. [cursor only]

#### QA-2.23 Open a link from Go to File

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (bundle has no "Invalid Link" or "paste a link" text)
- What: newer-than-1.128. Typing or pasting a link such as scheme://... shows a link row with an Open to the Side button, or an "Invalid Link" row with the note "Enter a valid link to open it from Quick Open." The placeholder gains "or paste a link".
- Commands: workbench.action.quickOpen
- Menus: none found
- Keys: Alt+Enter
- Settings: none found
- Source: N\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts; N\src\vs\workbench\contrib\search\browser\searchQuickAccess.contribution.ts; C bundle searched, no hits
- Pieces:
  1. In VS Code 1.144 and later (not in 1.128.0), typing or pasting a link such as `scheme://...` into Go to File shows a link row. [stock only]
  2. The link row has an "Open to the Side" button, and `Alt+Enter` opens the link to the side. [stock only]
  3. A link that is not valid shows an "Invalid Link" row with the note "Enter a valid link to open it from Quick Open." [stock only]
  4. The Go to File placeholder gains the words "or paste a link". [stock only]


## QA-3 Go to a spot in this file

(both, Go to Offset is vscode only). Small lists that jump to a line, column or symbol in the open editor. Examples: Go to Line (Ctrl+G); Go to Symbol in Editor (Ctrl+Shift+O); Go to Symbol by Category (`@:`).

15 features.

#### QA-3.1 Go to Line/Column

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (same command, key, menu item and prefix; older copy with other row text such as "Go to line {0} and character {1}." and "Current Line: {0}, Character: {1}. Type a line number between 1 and {2} to navigate to.", the column is a plain character column, and the no-editor row reads "Open a text editor first to go to a line.")
- What: Ctrl+G, or typing a colon, opens a one-row list that tells you what Enter will do. Type a line number, optionally followed by a column (12:5, 12,5 or 12#5), and Enter moves the cursor there. In stock the column counts visible columns, so a tab counts as its tab width. With no text editor open the row reads "Open a text editor first to go to a line or an offset."
- Commands: workbench.action.gotoLine
- Menus: MenubarGoMenu (Go to Line/Column..., group 5_infile_nav, order 1)
- Keys: Ctrl+G; Control+G (Mac)
- Settings: none found
- Source: V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts; C bundle (class EOr, base class dig, action rlg, messages 1476 to 1480, 5039 to 5041, 8907)
- Pieces:
  1. Pressing `Ctrl+G` (`Control+G` on Mac) or typing `:` in the quick access box opens a one-row list that tells you what Enter will do.
  2. Choosing Go to Line/Column... in the Go menu opens the same list.
  3. Typing a line number and pressing Enter moves the cursor to that line.
  4. Typing a line number followed by a column (`12:5`, `12,5` or `12#5`) and pressing Enter moves the cursor to that line and column.
  5. The single row changes as you type, with prompts such as "Type a line number to go to (from 1 to {max})." and "Press 'Enter' to go to line {n} at column {c}." [stock only]
  6. The column counts visible columns, so a tab counts as its tab width. [stock only]
  7. With no text editor open, the row tells you to open a text editor first.
  8. The list uses an older copy with rows such as "Go to line {0} and character {1}." and "Current Line: {0}, Character: {1}. Type a line number between 1 and {2} to navigate to.", the column is a plain character column, and the no-editor row reads "Open a text editor first to go to a line." [cursor only]

#### QA-3.2 Count back from the end with negative numbers

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (negative line works, negative column and clipping do not)
- What: A negative line number counts from the last line (-1 is the last line), and in stock a negative column counts from the end of the line. Stock also clips a number that is too big to the nearest valid line or column, so Enter always lands somewhere. In Cursor, only the line may be negative, and a number out of range shows a hint row and Enter does nothing.
- Commands: workbench.action.gotoLine
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; C bundle (parsePosition, isValidLineNumber, isValidColumn in class dig)
- Pieces:
  1. Typing a negative line number counts from the last line, so `-1` goes to the last line.
  2. A negative column counts from the end of the line. [stock only]
  3. A number that is too big is clipped to the nearest valid line or column, so pressing Enter always lands somewhere. [stock only]
  4. A number out of range shows a hint row and pressing Enter does nothing. [cursor only]

#### QA-3.3 Go to Offset

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (no command id, no label, no :: prefix in the bundle)
- What: Typing two colons (::1024), or running the command, goes to a character position counted from the start of the file. A negative number counts from the end. The row says which line and column the offset lands on, or asks for a position "from 1 to {max}".
- Commands: workbench.action.gotoOffset
- Menus: CommandPalette (Go to Offset..., f1 only)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; C bundle searched for workbench.action.gotoOffset, no hits
- Pieces:
  1. Typing `::1024` in the quick access box and pressing Enter goes to the character position 1024 counted from the start of the file. [stock only]
  2. Running Go to Offset... from the Command Palette opens the list ready for an offset. [stock only]
  3. A negative number counts from the end of the file. [stock only]
  4. The row says which line and column the offset lands on, or asks for a position "from 1 to {max}". [stock only]

#### QA-3.4 Switch between 1-based and 0-based offsets

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (no gotoLine.useZeroBasedOffset key and no toggle label in the bundle)
- What: While you are in offset mode (the text starts with ::) a toggle button "Toggle Zero-Based Offset" appears at the right of the input. It flips between counting from 1 and from 0, and the choice is remembered for all windows.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts (storage key gotoLine.useZeroBasedOffset, application scope); C bundle searched, no hits
- Pieces:
  1. A toggle button "Toggle Zero-Based Offset" shows at the right of the input only while the text starts with `::`. [stock only]
  2. Clicking the button flips the count between starting at 1 and starting at 0, and the prompt changes between "from 1 to {max}" and "from 0 to {max-1}". [stock only]
  3. The choice is remembered across windows. [stock only]

#### QA-3.5 Preview the target line as you type

- Tag: both. Evidence: source. Boundary: no. Cursor: present (theme colors editor.rangeHighlightBackground and editorOverviewRuler.rangeHighlightForeground and the quick-access-range-highlight decoration are in the bundle)
- What: As the typed number changes, the editor scrolls that line to the middle with a smooth scroll and paints the whole line with a range highlight plus a mark in the overview ruler. Pressing Escape puts back the view you had before. A number that gives no line clears the highlight.
- Commands: workbench.action.gotoLine
- Menus: none found
- Keys: Escape
- Settings: none found
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\editorNavigationQuickAccess.ts; V\src\vs\editor\common\core\editorColorRegistry.ts; C bundle (base class K4o, addDecorations, revealRangeInCenter)
- Pieces:
  1. As the typed number changes, the editor scrolls that line to the middle with a smooth scroll.
  2. The whole target line is painted with a range highlight and a mark in the overview ruler.
  3. A typed number that gives no line clears the highlight.
  4. Pressing Escape puts back the view you had before.
  5. The highlight and the ruler mark take their colors from the theme colors `editor.rangeHighlightBackground` and `editorOverviewRuler.rangeHighlightForeground`.

#### QA-3.6 See absolute line numbers while Go to Line is open

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: If the editor shows relative line numbers, they switch to normal numbers while the Go to Line list is open, so the number you type matches what you see. They go back to relative when the list closes.
- Commands: workbench.action.gotoLine
- Menus: none found
- Keys: none found
- Settings: editor.lineNumbers
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; C bundle (provideWithTextEditor in class dig, lineNumbers on and relative)
- Pieces:
  1. While the Go to Line list is open, line numbers that were drawn relative switch to normal numbers, so the number you type matches what you see.
  2. When the list closes, the editor goes back to relative line numbers.
  3. Only `editor.lineNumbers` set to `relative` is changed while the list is open, and the values `on`, `off` and `interval` stay as they are.

#### QA-3.7 Land on the spot centered, with focus back in the editor

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (same jump, scroll and focus; shorter screen reader message)
- What: Enter selects the target, scrolls it to the middle of the editor with a smooth scroll, and returns focus to the editor. The jump counts as a navigation jump, so Go Back returns to where you were. Stock tells screen readers "Line {0}, column {1}: {text}", and Cursor reads out only the line text.
- Commands: workbench.action.gotoLine; workbench.action.gotoSymbol
- Menus: none found
- Keys: Enter
- Settings: none found
- Source: V\src\vs\editor\contrib\quickAccess\browser\editorNavigationQuickAccess.ts; V\src\vs\workbench\browser\parts\editor\textEditor.ts (JUMP selection source); V\src\vs\workbench\services\history\browser\historyService.ts (JUMP adds a navigation stop); C bundle (gotoLocation with code.jump)
- Pieces:
  1. Pressing Enter in the Go to Line or Go to Symbol list moves the cursor to the target and scrolls it to the middle of the editor with a smooth scroll.
  2. Focus returns to the editor after the jump, so typing goes straight into the file.
  3. The jump counts as a navigation jump, so Go Back returns to where you were before.
  4. Screen readers are told "Line {0}, column {1}: {text}" after the jump. [stock only]
  5. Screen readers hear only the text of the target line after the jump. [cursor only]

#### QA-3.8 Open the jump in a side editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Alt+Enter, or Ctrl+Enter while previews from Quick Open are off (the default), opens the same file in the side group at the target. Symbol rows also have an "Open to the Side" button ("Open to the Bottom" when the side direction is down). The view in the first editor is restored.
- Commands: workbench.action.gotoLine; workbench.action.gotoSymbol
- Menus: none found
- Keys: Alt+Enter (modifier read by the picker, no keybinding rule); Ctrl+Enter (same)
- Settings: workbench.editor.openSideBySideDirection; workbench.editor.enablePreviewFromQuickOpen
- Source: V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts; V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; C bundle (gotoLocation overrides in classes EOr and bce, messages 1485 and 1486)
- Pieces:
  1. Pressing `Alt+Enter` in the Go to Line or Go to Symbol list opens the same file in the side group with the cursor at the target.
  2. Pressing `Ctrl+Enter` does the same while `workbench.editor.enablePreviewFromQuickOpen` is off, which is the default.
  3. Symbol rows have an "Open to the Side" button that does the same, and it reads "Open to the Bottom" when `workbench.editor.openSideBySideDirection` is `down`.
  4. The view in the first editor is restored, so only the side editor moves.

#### QA-3.9 Jump without closing the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right Arrow on a row moves the editor to that line or symbol while the list stays open and focus stays in the list, so you can hop between spots.
- Commands: quickInput.acceptInBackground
- Menus: none found
- Keys: Right Arrow (when in quick pick and focus is in the list or the cursor is at the end of the input box)
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; C bundle (quickInput.acceptInBackground rule, canAcceptInBackground in both providers)
- Pieces:
  1. Pressing Right Arrow on a row moves the editor to that line or symbol while the list stays open.
  2. Focus stays in the list, so you can hop between spots.
  3. Right Arrow acts this way only when focus is in the list or the text cursor is at the end of the input.

#### QA-3.10 Go to Symbol in Editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present (same command, key, menu item and placeholder "Type the name of a symbol to go to."; the rows have no shift-to-attach callback)
- What: Ctrl+Shift+O, or typing @, lists the symbols of the open file under a "symbols (N)" heading. Each row has a kind icon, the name, and its container, with the matched letters highlighted, and deprecated symbols are struck through. With an empty box the symbol that contains the cursor starts as the active row. In a diff editor the list uses the modified side.
- Commands: workbench.action.gotoSymbol
- Menus: MenubarGoMenu (Go to Symbol in Editor..., group 4_symbol_nav, order 1)
- Keys: Ctrl+Shift+O (when the accessible view and accessibility help are not shown); Cmd+Shift+O (Mac)
- Settings: none found
- Source: V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; C bundle (class bce, base class gie, action r5o, messages 5043 to 5047, 1487 to 1500)
- Pieces:
  1. Pressing `Ctrl+Shift+O` (`Cmd+Shift+O` on Mac) or typing `@` lists the symbols of the open file.
  2. Choosing Go to Symbol in Editor... in the Go menu opens the same list.
  3. The list sits under a "symbols (N)" heading that shows how many symbols match.
  4. Each row shows a kind icon, the symbol name and its container, with the matched letters highlighted.
  5. Deprecated symbols are drawn struck through.
  6. With an empty box, the symbol that contains the cursor starts as the active row.
  7. In a diff editor the list uses the symbols of the modified side.
  8. The box placeholder reads "Type the name of a symbol to go to."
  9. `Ctrl+Shift+O` does nothing while the accessible view or accessibility help is shown.
  10. A progress indicator shows on the input while the symbols are being collected.

#### QA-3.11 Group symbols by category

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the by-category help text 5046 and the kind headings 1487 to 1500 are in the table; prefix constant SCOPE_PREFIX in the bundle)
- What: Typing @: (a colon right after the @) groups the symbols by kind and puts a heading with a count over each group, such as "methods (4)", "functions (2)" or "classes (1)". The groups are ordered by the kind's heading text, and inside a group rows are sorted by match score, then by position in the file. Anything after the colon filters as usual.
- Commands: workbench.action.gotoSymbol
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts (help entry "Go to Symbol in Editor by Category"); C bundle (doGetSymbolPicks with SCOPE_PREFIX, messages 5046 and 1487 to 1500)
- Pieces:
  1. Typing `@:` (a colon right after the `@`) groups the symbols by kind, with a heading and a count over each group, such as "methods (4)", "functions (2)" or "classes (1)".
  2. The groups are ordered by the text of the kind heading.
  3. Inside a group, rows are sorted by match score and then by position in the file.
  4. Text typed after the colon filters the symbols as usual, and each group heading shows its filtered count.

#### QA-3.12 Narrow symbols by container with a second word

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With two search words the first matches the symbol name and the rest match its container (for example the class name), so "run Server" finds run inside Server. The container text is highlighted in the grey description.
- Commands: workbench.action.gotoSymbol
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; C bundle (doGetSymbolPicks, container query split)
- Pieces:
  1. With two search words, the first matches the symbol name and the rest match its container, so `run Server` finds `run` inside `Server`.
  2. A symbol with no container is left out when the extra words do not also match its own name.
  3. The container text in the grey description is highlighted where it matched.

#### QA-3.13 Preview the symbol as you move through the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The active row scrolls its symbol to the middle of the editor with a smooth scroll and highlights the symbol's whole range, plus a mark in the overview ruler. Escape puts the old view back, and the cursor does not move until you accept.
- Commands: workbench.action.gotoSymbol
- Menus: none found
- Keys: Up Arrow; Down Arrow; Escape
- Settings: none found
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; V\src\vs\editor\contrib\quickAccess\browser\editorNavigationQuickAccess.ts; C bundle (onDidChangeActive handler with revealRangeInCenter and addDecorations)
- Pieces:
  1. Moving to a row scrolls its symbol to the middle of the editor with a smooth scroll.
  2. The whole range of the symbol is highlighted, with a mark in the overview ruler.
  3. The cursor and selection do not move until you accept a row.
  4. Pressing Escape puts the old view back.
  5. Up Arrow and Down Arrow change the active row, and the preview follows.

#### QA-3.14 Go to Symbol in notebooks and other non-text editors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the active editor is not a plain text editor but can make an outline (a notebook, for example), the same list shows the outline entries with their own icons. Moving through the rows previews the entry, Enter reveals it, and an empty filter result shows "No matching entries". For notebooks a setting chooses whether full code symbols are listed or only Markdown headers.
- Commands: workbench.action.gotoSymbol
- Menus: none found
- Keys: Ctrl+Shift+O
- Settings: notebook.gotoSymbols.showAllSymbols
- Source: V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\outline\notebookOutline.ts; C bundle (doGetOutlinePicks, getQuickPickElements, canCreateOutline, message 5042 and 8034)
- Pieces:
  1. When the active editor is not a plain text editor but can make an outline (a notebook, for example), the list shows the outline entries with their own icons.
  2. Moving through the rows previews the entry in the editor.
  3. Pressing Enter on a row reveals that entry.
  4. A filter with no result shows "No matching entries".
  5. Setting `notebook.gotoSymbols.showAllSymbols` chooses whether notebooks list full code symbols as well as Markdown headers (on, the default) or only Markdown headers (off).

#### QA-3.15 See why the symbol list is empty

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With no editor open the list says "To go to a symbol, first open a text editor with symbol information." With an editor that has no symbol provider it says "The active text editor does not provide symbol information." and switches to the real list on its own if language support arrives late. A search with no hits says "No matching editor symbols" and an empty file says "No editor symbols".
- Commands: workbench.action.gotoSymbol
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts; C bundle (provideWithoutTextEditor, waitForLanguageSymbolRegistry, messages 1481 to 1484)
- Pieces:
  1. With no editor open, the list says "To go to a symbol, first open a text editor with symbol information."
  2. With an editor that has no symbol provider, the list says "The active text editor does not provide symbol information."
  3. When language support arrives late, that message is replaced by the real symbol list without reopening it.
  4. A search with no hits says "No matching editor symbols".
  5. A file with no symbols says "No editor symbols".


## QA-4 Switch and open lists

(both). Prefix lists that jump to something already open or defined. Examples: Open View (Ctrl+Q); editor switchers; Switch Active Terminal; Run Task and Start Debugging.

19 features.

#### QA-4.1 Open a view, panel, terminal or output channel by name

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids, the `inViewsPicker` context key, the placeholder, the "&&Open View..." label, "No matching views" and the Ctrl+Q key rule with Linux set to none were found)
- What: Run Open View (or type `view ` in the box) and pick a row to open a Side Bar, Panel or Secondary Side Bar container, one of its views, a terminal, a debug console, or an Output channel. Rows sit under heading rows ("Side Bar", "Panel", "Secondary Side Bar", then one per container such as "Side Bar / Explorer", then "Terminal", "Debug Console", "Output"). Typing fuzzy-matches the view name or any part of its container name, and containers marked hide-if-empty are left out. With no match the list shows "No matching views".
- Commands: workbench.action.openView; workbench.action.quickOpenView
- Menus: MenubarViewMenu (View > Open View..., group 1_open, order 2); CommandPalette (Open View, f1; Quick Open View is f1 false so it stays out of the palette)
- Keys: Ctrl+Q (workbench.action.quickOpenView; Control+Q on Mac; no default key on Linux)
- Settings: none found
- Source: V\src\vs\workbench\contrib\quickaccess\browser\viewQuickAccess.ts (32-262); V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (33-40, 73-80, 117-145); C bundle (workbench.action.openView, workbench.action.quickOpenView, inViewsPicker, provider `sdt`, message 8910 to 8912, nls "No matching views")
- Pieces:
  1. Running Open View (or typing `view ` in the quick access box) lists the views, panels, terminals and output channels you can open.
  2. Choosing View > Open View... opens the same list, and the Command Palette lists Open View but not Quick Open View.
  3. Pressing `Ctrl+Q` (`Control+Q` on Mac, with no default key on Linux) opens the list through `workbench.action.quickOpenView`.
  4. Rows sit under heading rows for "Side Bar", "Panel" and "Secondary Side Bar", then one heading per container such as "Side Bar / Explorer", and then "Terminal", "Debug Console" and "Output".
  5. Picking a row opens that container, that view, that terminal, that debug console or that Output channel.
  6. Typing fuzzy-matches the view name or any part of its container name.
  7. Containers marked hide-if-empty are left out of the list.
  8. With no match the list shows "No matching views".

#### QA-4.2 Pick an editor from the active group by most recent use

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (all ids, labels and keys found; the Ctrl+Tab in-group command has extra preconditions that test agents pane focus and composer focus)
- What: Run "Show Editors in Active Group By Most Recently Used" (or type `edt active `, or choose Show Opened Editors from the tab strip "..." menu) to list only the editors of the active group, most recently used first. Typing filters on name and description, picking a row opens that editor, and Ctrl+Tab and Ctrl+Shift+Tab open this same list while the key is held.
- Commands: workbench.action.showEditorsInActiveGroup; workbench.action.showEditorsInGroup; workbench.action.quickOpenPreviousRecentlyUsedEditorInGroup; workbench.action.quickOpenLeastRecentlyUsedEditorInGroup; workbench.action.quickOpenNavigateNextInEditorPicker; workbench.action.quickOpenNavigatePreviousInEditorPicker
- Menus: EditorTitle (Show Opened Editors, group 3_open, order 10); CommandPalette (View: Show Editors in Active Group By Most Recently Used)
- Keys: Ctrl+Tab and Ctrl+Shift+Tab (Control+Tab and Control+Shift+Tab on Mac; the two in-group commands need a non-empty active group; the cycle commands apply when inQuickOpen and inEditorsPicker)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorQuickAccess.ts (35-190, 192-212); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (143-152, 312-331, 424); V\src\vs\workbench\browser\parts\editor\editorActions.ts (1710-1727, 1778-1861); V\src\vs\workbench\browser\parts\editor\editorCommands.ts (1430-1446); C bundle (providers `rIe`, `tst`, `sIe`; key rule for quickOpenPreviousRecentlyUsedEditorInGroup with its precondition; ids showEditorsInActiveGroup, showEditorsInGroup; nls "Show Opened Editors", messages 3412 to 3416)
- Pieces:
  1. Running "Show Editors in Active Group By Most Recently Used" from the Command Palette (or typing `edt active `) lists only the editors of the active group, most recently used first.
  2. Choosing Show Opened Editors from the "..." menu of the tab strip opens the same list.
  3. Typing filters the rows on the editor name and its description.
  4. Picking a row opens that editor.
  5. `Ctrl+Tab` opens this list with the previous editor active and `Ctrl+Shift+Tab` with the least recently used one active (`Control+Tab` and `Control+Shift+Tab` on Mac), as long as the active group is not empty.
  6. In Cursor the `Ctrl+Tab` command has extra conditions that look at agents pane focus and composer focus before it opens the list. [cursor only]

#### QA-4.3 Pick from all open editors in order of appearance

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id and title found, help text "Show All Opened Editors By Appearance" found; the key chord was not pulled from the Cursor key table)
- What: Run "Show All Editors By Appearance" (or type `edt `) to list every open editor in all groups, left to right and group by group. A heading row names each group when more than one group is open, unsaved editors are marked, and editors in preview mode are drawn in italics.
- Commands: workbench.action.showAllEditors
- Menus: CommandPalette (File: Show All Editors By Appearance)
- Keys: Ctrl+K Ctrl+P (Cmd+Alt+Tab on Mac)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorQuickAccess.ts (35-190, 219-245); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (154-160); V\src\vs\workbench\browser\parts\editor\editorActions.ts (1730-1755); C bundle (class with id workbench.action.showAllEditors, provider `tst`, messages 3414 and 3415)
- Pieces:
  1. Running "Show All Editors By Appearance" from the Command Palette (or typing `edt `) lists every open editor in all groups, left to right and group by group.
  2. Pressing `Ctrl+K Ctrl+P` (`Cmd+Alt+Tab` on Mac) opens the same list.
  3. A heading row names each group when more than one group is open.
  4. An editor with unsaved changes is marked in its row, and screen readers read it with "unsaved changes" added.
  5. Editors in preview mode are drawn in italics.
  6. In any of the three editor lists, a filter with no hits shows "No matching editors".

#### QA-4.4 Pick from all open editors by most recent use

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all three ids and the help text "Show All Opened Editors By Most Recently Used" found)
- What: Run "Show All Editors By Most Recently Used" (or type `edt mru `) to list every open editor from all groups as one list, newest first. Two more palette commands open the same list for quick stepping, one starting on the previous used editor and one on the least recently used.
- Commands: workbench.action.showAllEditorsByMostRecentlyUsed; workbench.action.quickOpenPreviousRecentlyUsedEditor; workbench.action.quickOpenLeastRecentlyUsedEditor
- Menus: CommandPalette (View: Show All Editors By Most Recently Used; View: Quick Open Previous Recently Used Editor; View: Quick Open Least Recently Used Editor)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorQuickAccess.ts (251-275); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (162-168); V\src\vs\workbench\browser\parts\editor\editorActions.ts (1757-1822); C bundle (ids found, provider `sIe`, message 3416)
- Pieces:
  1. Running "Show All Editors By Most Recently Used" from the Command Palette (or typing `edt mru `) lists every open editor from all groups, newest first.
  2. When more than one group is open, a heading with the group name appears each time the next row belongs to a different group.
  3. Running "Quick Open Previous Recently Used Editor" from the Command Palette opens the same list with the previously used editor active.
  4. Running "Quick Open Least Recently Used Editor" from the Command Palette opens the same list with the least recently used editor active.

#### QA-4.5 Close an editor from its row in the editor lists

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the "Close Editor" label and the three editor providers were found; the button wiring itself was not traced)
- What: Each row in the three editor lists has a "Close Editor" button on hover (an x). For an editor with unsaved changes the button is a dot and stays visible. Clicking it closes that editor and removes the row while the list stays open and focus stays in the box.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorQuickAccess.ts (140-190); C bundle (nls "Close Editor", providers `rIe`, `tst`, `sIe`)
- Pieces:
  1. Each row in the three editor lists has a "Close Editor" button, drawn as an x, that shows on hover.
  2. For an editor with unsaved changes the button is drawn as a filled dot and stays visible.
  3. Clicking the button closes that editor and removes its row while the list stays open.
  4. Focus stays in the input box after the editor is closed.
  5. If the editor is still open after the click, for example because closing was cancelled, its row stays in the list.

#### QA-4.6 Switch Active Terminal from a list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, `inTerminalPicker` context key, provider `jLt`, nls "Show All Opened Terminals", "Create New Terminal", "Create New Terminal With Profile..." and both navigate ids found)
- What: Run "Terminal: Switch Active Terminal" (or type `term `) to list the panel terminals under a "panel" heading and terminal editors under an "editor" heading, each row with its icon, color, number and title (for example "1.2: bash") and its description. Picking a row shows and focuses that terminal, and Right arrow opens it without leaving the list. Two rows at the bottom, "Create New Terminal" and "Create New Terminal With Profile...", start a new one.
- Commands: workbench.action.quickOpenTerm; workbench.action.quickOpenNavigateNextInTerminalPicker; workbench.action.quickOpenNavigatePreviousInTerminalPicker; workbench.action.terminal.new; workbench.action.terminal.newWithProfile
- Menus: CommandPalette (Terminal: Switch Active Terminal)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\quickAccess\browser\terminalQuickAccess.ts (25-143); V\src\vs\workbench\contrib\terminalContrib\quickAccess\browser\terminal.quickAccess.contribution.ts (18-40); C bundle (workbench.action.quickOpenTerm, provider `jLt`, messages 10862, 10863 and 10865)
- Pieces:
  1. Running "Terminal: Switch Active Terminal" from the Command Palette (or typing `term `) lists the open terminals.
  2. Terminals in the panel sit under a "panel" heading and terminals in editor tabs sit under an "editor" heading.
  3. Each row shows the terminal icon, its color, its number and title (for example "1.2: bash"), and its description.
  4. Picking a row shows that terminal and moves focus into it.
  5. Pressing Right Arrow on a row opens that terminal without leaving the list.
  6. Two rows at the bottom, "Create New Terminal" and "Create New Terminal With Profile...", start a new terminal.
  7. The next and previous commands for this list step the active row down and up, and they have no default keys.

#### QA-4.7 Rename or kill a terminal from its row

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the "Rename Terminal" label (message 10867) and the provider were found; the kill button calls the terminal service directly, so it has no command id)
- What: Each terminal row in the Switch Active Terminal list shows a pencil "Rename Terminal" button and a trash "Kill Terminal" button on hover. Rename asks for a new name and leaves the list open. Kill ends that terminal at once and removes the row.
- Commands: workbench.action.terminal.rename
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\quickAccess\browser\terminalQuickAccess.ts (82-143); C bundle (nls "Rename Terminal", provider `jLt`)
- Pieces:
  1. Each terminal row in the Switch Active Terminal list shows a pencil "Rename Terminal" button on hover.
  2. Clicking Rename Terminal asks for a new name and leaves the list open.
  3. Each terminal row also shows a trash "Kill Terminal" button on hover.
  4. Clicking Kill Terminal ends that terminal at once and removes its row.

#### QA-4.8 Run a task from the Run Task list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id found 6 times, all four task.quickOpen settings found, nls "Run Task", "Show All Tasks..." and "recently used" found)
- What: Choose Terminal > Run Task... (or the palette command) to open "Select the task to run". It shows a "recently used" section, a "configured" section, a "contributed" section with one folder row per task provider (for example npm), and a "Show All Tasks..." row. If no task exists, a "Configure a Task" row opens the setup. Picking a task runs it with its problem matcher. Picking a provider row opens a second step listing that provider's tasks. Each top level row also has a pin button that moves the task into a "pinned" section at the top (the pin widget itself is QA-6.24).
- Commands: workbench.action.tasks.runTask
- Menus: MenubarTerminalMenu (Run Task..., group 3_run, order 1, when TaskExecutionSupportedContext); CommandPalette (Tasks: Run Task)
- Keys: none found
- Settings: task.quickOpen.history; task.quickOpen.detail; task.quickOpen.skip; task.quickOpen.showAll
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (585-620, 3110-3221); V\src\vs\workbench\contrib\tasks\browser\taskQuickPick.ts (14-260, 311-320); V\src\vs\platform\quickinput\browser\quickPickPin.ts; V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (163-173, 267-272, 515-536); C bundle (workbench.action.tasks.runTask, task.quickOpen.* settings, messages 9977 and 9983, pin function with message 2257)
- Pieces:
  1. Choosing Terminal > Run Task... (or "Tasks: Run Task" in the Command Palette) opens a list with the placeholder "Select the task to run".
  2. The list has a "recently used" section, a "configured" section, a "contributed" section with one folder row per task provider (for example npm), and a "Show All Tasks..." row.
  3. When no task exists, a "Configure a Task" row opens the task setup.
  4. Picking a task runs it with its problem matcher.
  5. Picking a provider row opens a second step that lists that provider's tasks.
  6. Each top level row has a pin button that moves the task into a "pinned" section at the top of the list.
  7. Setting `task.quickOpen.history` (0 to 30, default 30) sets how many recently used tasks are kept for the "recently used" section.
  8. Setting `task.quickOpen.detail` (on by default) chooses whether a task's detail line shows under its name.
  9. Setting `task.quickOpen.skip` (off by default) skips the list when there is only one task to pick from.
  10. Setting `task.quickOpen.showAll` (off by default) makes Run Task use the slower list of all tasks in place of the two-step list grouped by provider.

#### QA-4.9 Run a task by typing the task prefix

- Tag: both. Evidence: source. Boundary: no. Cursor: present (provider `Wln`, context key `inTasksPicker`, placeholder 9941, help text 9942 and command center order 60 found)
- What: Type `task ` in the quick access box (placeholder "Type the name of a task to run.") to get the same top level task rows as Run Task, filtered by name as you type, with "No matching tasks" when nothing fits. Picking a task runs it, and picking a provider folder row opens the second step. The help list names it "Run Task", and the command center empty-box shortcut list has a Run Task row (order 60, just before More).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: task.quickOpen.history; task.quickOpen.detail
- Source: V\src\vs\workbench\contrib\tasks\browser\tasksQuickAccess.ts (23-97); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (401-410); C bundle (provider `Wln`, messages 9941 and 9942)
- Pieces:
  1. Typing `task ` in the quick access box shows the same top level task rows as Run Task, with the placeholder "Type the name of a task to run."
  2. Typing filters the rows by task name as you type.
  3. A filter with no hits shows "No matching tasks".
  4. Picking a task runs it, and picking a provider folder row opens the second step.
  5. The help list names this prefix "Run Task", and the command center empty-box shortcut list has a Run Task row at order 60, just before More.
  6. Settings `task.quickOpen.history` and `task.quickOpen.detail` apply to this list the same way as to Run Task, limiting the recently used rows and showing or hiding the detail line.

#### QA-4.10 Configure a task or forget a recent task from its row

- Tag: both. Evidence: source. Boundary: no. Cursor: present (nls "Configure Task" and "Remove Recently Used Task" found)
- What: Task rows in the Run Task list and the `task ` list show a gear "Configure Task" button that opens or creates the task in tasks.json and closes the list. Rows in the "recently used" section also show an x "Remove Recently Used Task" button that drops the row from history and refreshes the list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: task.quickOpen.history
- Source: V\src\vs\workbench\contrib\tasks\browser\taskQuickPick.ts (42-48, 102-110, 190-197, 233-262); V\src\vs\workbench\contrib\tasks\browser\tasksQuickAccess.ts (62-80); C bundle (nls "Configure Task", "Remove Recently Used Task")
- Pieces:
  1. Task rows in the Run Task list and the `task ` list show a gear "Configure Task" button that opens or creates the task in tasks.json and closes the list.
  2. Rows in the "recently used" section also show an x "Remove Recently Used Task" button.
  3. Clicking the x drops that row from the history and removes it from the list while the list stays open.
  4. Setting `task.quickOpen.history` limits how many recently used rows, and so how many x buttons, can appear.

#### QA-4.11 Run the build or test task, choosing one when there is no single default

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids, nls "Run Build Task", "Run Test Task", "Select the build task to run" and "Configure Build Task..." found; the Cursor key table was not pulled)
- What: Run Build Task (Ctrl+Shift+B) or Run Test Task runs the default task of that group at once. When several tasks qualify (several defaults, or several glob matches for the open file) a list "Select the build task to run" or "Select the test task to run" appears, and when none qualifies a row offers to configure one.
- Commands: workbench.action.tasks.build; workbench.action.tasks.test
- Menus: MenubarTerminalMenu (Run Build Task..., group 3_run, order 2, when TaskExecutionSupportedContext); CommandPalette (Tasks: Run Build Task; Tasks: Run Test Task)
- Keys: Ctrl+Shift+B (workbench.action.tasks.build; Cmd+Shift+B on Mac; when TaskCommandsRegistered); Run Test Task has no default key
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (3260-3430); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (175-183, 311-327, 387-392); C bundle (workbench.action.tasks.build menu item and palette item, workbench.action.tasks.test)
- Pieces:
  1. Choosing Terminal > Run Build Task..., or pressing `Ctrl+Shift+B` (`Cmd+Shift+B` on Mac), runs the default build task at once.
  2. Running "Tasks: Run Test Task" from the Command Palette runs the default test task at once, and it has no default key.
  3. When several tasks qualify, for example several defaults or several glob matches for the open file, a list "Select the build task to run" or "Select the test task to run" appears.
  4. When no task qualifies, a row offers to configure one, such as "Configure Build Task...".

#### QA-4.12 Show a running task from a list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id found 6 times, the menu item in group 5_manage and nls "Show Running Tasks" found; the status bar items that also run it are in SS-5)
- What: Choose Terminal > Show Running Tasks... (greyed when no task runs). With one running task, or several in one terminal group, its terminal is revealed at once. Otherwise the list "Select the task to show its output" appears, and "No task is running" shows if the list is empty.
- Commands: workbench.action.tasks.showTasks
- Menus: MenubarTerminalMenu (Show Running Tasks..., group 5_manage, order 1, precondition TASK_RUNNING_STATE); CommandPalette (Tasks: Show Running Tasks)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (679-683, 3900-3935); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (186-195, 296-303); C bundle (workbench.action.tasks.showTasks, nls "Show Running Tasks")
- Pieces:
  1. Choosing Terminal > Show Running Tasks... (or "Tasks: Show Running Tasks" in the Command Palette) shows the output of a running task.
  2. The menu item is greyed out when no task is running.
  3. With one running task, or several in one terminal group, its terminal is revealed at once without a list.
  4. Otherwise the list "Select the task to show its output" appears.
  5. When there is nothing to list, "No task is running" shows.

#### QA-4.13 Terminate or restart a running task from a list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids found 4 times each; nls "Terminate Task...", "Restart Running Task...", "Select a task to terminate" and "Select the task to restart" found)
- What: Terminal > Terminate Task... lists the running tasks under "Select a task to terminate" with an extra "All Running Tasks" row, and "No task is currently running" when empty. Terminal > Restart Running Task... lists them under "Select the task to restart" and shows "No task to restart" when empty. A single choice acts at once. Both are greyed when no task runs.
- Commands: workbench.action.tasks.terminate; workbench.action.tasks.restartTask
- Menus: MenubarTerminalMenu (Restart Running Task..., group 5_manage, order 2; Terminate Task..., group 5_manage, order 3; both with precondition TASK_RUNNING_STATE); CommandPalette (Tasks: Restart Running Task; Tasks: Terminate Task)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (625-645, 3425-3475, 3490-3532); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (197-217, 279-289, 303-311); C bundle (ids and the four labels)
- Pieces:
  1. Choosing Terminal > Terminate Task... lists the running tasks under "Select a task to terminate", with an extra "All Running Tasks" row.
  2. Choosing Terminal > Restart Running Task... lists the running tasks under "Select the task to restart".
  3. When there is only one choice, it acts at once without showing a list.
  4. An empty Terminate Task list shows "No task is currently running", and an empty Restart Running Task list shows "No task to restart".
  5. Both menu items are greyed out when no task is running, and both are also in the Command Palette as "Tasks: Terminate Task" and "Tasks: Restart Running Task".

#### QA-4.14 Pick a task to configure or to make the default build or test task

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all three ids and nls "Configure Tasks...", "Select a task to configure" and "Select the task to be used as the default build task" found)
- What: Terminal > Configure Tasks... opens a list "Select a task to configure" and opens the chosen task in tasks.json. Terminal > Configure Default Build Task... and the matching test command list tasks under "Select the task to be used as the default build task" (or test task) and mark the chosen one as the default.
- Commands: workbench.action.tasks.configureTaskRunner; workbench.action.tasks.configureDefaultBuildTask; workbench.action.tasks.configureDefaultTestTask
- Menus: MenubarTerminalMenu (Configure Tasks..., group 7_configure, order 1; Configure Default Build Task..., group 7_configure, order 2); CommandPalette (Tasks: Configure Task; Tasks: Configure Default Build Task; Tasks: Configure Default Test Task)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (661-676, 3700-3890); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (219-239, 249-256, 327-345); C bundle (workbench.action.tasks.configureTaskRunner, configureDefaultBuildTask, configureDefaultTestTask)
- Pieces:
  1. Choosing Terminal > Configure Tasks... opens a list "Select a task to configure" and opens the chosen task in tasks.json.
  2. Choosing Terminal > Configure Default Build Task... lists tasks under "Select the task to be used as the default build task" and marks the chosen one as the default build task.
  3. "Tasks: Configure Default Test Task" in the Command Palette does the same with a test task, and it has no Terminal menu item.

#### QA-4.15 Start debugging from the launch configuration list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id found, `inLaunchConfigurationsPicker` context key, provider `w5a`, placeholder 5614, help text "Start Debugging" 5615 with command center order 50, nls "Most Recent" and "launch.json" and the `status.debug` item found)
- What: Run "Debug: Select and Start Debugging" (or type `debug `, or click the Debug status bar item) to list launch configurations under a "launch.json" heading, with the last used one first under "Most Recent". Entries from debug providers sit under "contributed" (recent dynamic configurations, and one "Provider..." row per provider). Picking a row selects that configuration and starts debugging, and a failed start shows an error notification. With no debugger available the list is empty. The same command is behind "Start Additional Session" in the Run and Debug view menu while debugging with a docked toolbar.
- Commands: workbench.action.debug.selectandstart
- Menus: CommandPalette (Debug: Select and Start Debugging); ViewContainerTitle (Start Additional Session, Run and Debug container, when debugging and the toolbar is docked or the launcher is hidden)
- Keys: none found
- Settings: debug.showInStatusBar
- Source: V\src\vs\workbench\contrib\debug\browser\debugQuickAccess.ts (18-170); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (86-98, 145, 546-551); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (68, 125, 724-746); V\src\vs\workbench\contrib\debug\browser\debugStatus.ts (24-72); V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts (290-303); C bundle (workbench.action.debug.selectandstart, provider `w5a`, status.debug, debug.showInStatusBar)
- Pieces:
  1. Running "Debug: Select and Start Debugging" from the Command Palette, typing `debug `, or clicking the Debug status bar item opens a list of launch configurations.
  2. Launch configurations sit under a "launch.json" heading, with the last used one first under "Most Recent".
  3. Entries from debug providers sit under "contributed", with recent dynamic configurations and one "Provider..." row per provider.
  4. Picking a row selects that configuration and starts debugging.
  5. A start that fails shows an error notification.
  6. With no debugger available the list is empty.
  7. While debugging with a docked toolbar, "Start Additional Session" in the menu of the Run and Debug view opens the same list.
  8. Setting `debug.showInStatusBar` chooses when the Debug status bar item shows, with values `never`, `always` and `onFirstSessionStart` (the default).

#### QA-4.16 Edit, remove or add launch configurations from the list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id debug.addConfiguration, nls "Configure Launch Configuration", "Remove Launch Configuration" and "Add Configuration..." found)
- What: In the Start Debugging list a gear "Configure Launch Configuration" button on each launch.json row opens that launch file and closes the list. Recent dynamic configurations show an x "Remove Launch Configuration" button that forgets them and closes the list. A "configure" section at the bottom has "Add Configuration..." rows (one per launch file, named "Add Config (name)..." in a multi-root workspace) that run the add configuration flow.
- Commands: debug.addConfiguration
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debugQuickAccess.ts (50-66, 105-117, 145-168); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (46); C bundle (debug.addConfiguration, messages 5827 and 5830)
- Pieces:
  1. In the Start Debugging list, a gear "Configure Launch Configuration" button on each launch.json row opens that launch file and closes the list.
  2. Recent dynamic configurations show an x "Remove Launch Configuration" button that forgets them and closes the list.
  3. A "configure" section at the bottom has "Add Configuration..." rows, one per launch file, that run the add configuration flow (`debug.addConfiguration`).
  4. In a multi-root workspace the add rows are named "Add Config (name)...".

#### QA-4.17 Pick a debug console from a list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id, `inDebugConsolePicker` context key, provider `h5a`, messages 5616 and 5617, nls "Select Debug Console" and "Start a New Debug Session" found)
- What: Run "Debug: Select Debug Console" (or type `debug consoles `) to list the debug sessions that have their own console. Picking one focuses that session and opens the Debug Console view, and Right arrow does it without leaving the list. A last row, "Start a New Debug Session", runs the Start Debugging list.
- Commands: workbench.action.debug.selectDebugConsole
- Menus: CommandPalette (Debug: Select Debug Console)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debugConsoleQuickAccess.ts (15-65); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (100-107, 149); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (69, 126, 749-754); C bundle (workbench.action.debug.selectDebugConsole, provider `h5a`)
- Pieces:
  1. Running "Debug: Select Debug Console" from the Command Palette (or typing `debug consoles `) lists the debug sessions that have their own console.
  2. Picking a session focuses it and opens the Debug Console view.
  3. Pressing Right Arrow on a row does the same without leaving the list.
  4. A last row, "Start a New Debug Session", opens the Start Debugging list.

#### QA-4.18 Select a debug session by name

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id and nls "Select Debug Session" and "Search debug sessions by name" found)
- What: Run "Debug: Select Debug Session" for a list with the placeholder "Search debug sessions by name". Sessions started from a parent show the parent's name as a description, the focused session is highlighted on open, and picking one focuses it and opens the Debug Console. A last row, "Start a New Debug Session", opens the Start Debugging list. This list is a plain quick pick, not a quick access prefix.
- Commands: workbench.action.debug.selectDebugSession
- Menus: CommandPalette (Debug: Select Debug Session)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debugSessionPicker.ts (14-122); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (70, 757-761); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (150); C bundle (workbench.action.debug.selectDebugSession, message 5737 or 5901 for "Start a New Debug Session")
- Pieces:
  1. Running "Debug: Select Debug Session" from the Command Palette opens a list with the placeholder "Search debug sessions by name".
  2. Sessions that were started from a parent show the parent's name as a description.
  3. The focused session is highlighted when the list opens.
  4. Picking a session focuses it and opens the Debug Console.
  5. A last row, "Start a New Debug Session", opens the Start Debugging list.
  6. The list opens only from the command and has no quick access prefix to type.

#### QA-4.19 List extensions to report an issue on with the issue prefix

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (no provider, no `inReportIssuePicker` context key, no `extensions.experimental.issueQuickAccess` setting, and none of the labels "Open Issue Reporter", "Type the name of an extension to report on." or "Open Extension Page" in the bundle; the command id workbench.action.openIssueReporter itself exists in the bundle)
- What: Type `issue ` to list the product name and "Extension Marketplace" first, then every installed non-built-in extension and extension-contributed issue entry, sorted by name. Picking a row opens the Issue Reporter for it, an info button opens the extension page, and Right arrow picks without closing. The help list calls it "Open Issue Reporter". It exists only in the desktop build, only while `telemetry.feedback.enabled` and `extensions.experimental.issueQuickAccess` (default on) are both on, and the rows are drawn from the extension list (an out of scope system), which is why it is a judgment call to catalog it here.
- Commands: workbench.action.openIssueReporter
- Menus: IssueReporter (extension-contributed rows)
- Keys: none found
- Settings: extensions.experimental.issueQuickAccess; telemetry.feedback.enabled
- Source: V\src\vs\workbench\contrib\issue\browser\issueQuickAccess.ts (20-150); V\src\vs\workbench\contrib\issue\electron-browser\issue.contribution.ts (78-121); V\src\vs\workbench\contrib\extensions\browser\extensions.contribution.ts (316-320); C bundle (searched ids and labels above, all absent except the command id)
- Pieces:
  1. Typing `issue ` in the quick access box lists the product name and "Extension Marketplace" first, then every installed non-built-in extension and extension-contributed issue entry, sorted by name. [stock only]
  2. Picking a row opens the Issue Reporter for it. [stock only]
  3. An info button on a row opens the extension page. [stock only]
  4. Pressing Right Arrow on a row picks it without closing the list. [stock only]
  5. The box placeholder reads "Type the name of an extension to report on." and the help list calls the prefix "Open Issue Reporter". [stock only]
  6. The list exists only in the desktop build, and only while `telemetry.feedback.enabled` and `extensions.experimental.issueQuickAccess` (on by default) are both on. [stock only]


## QA-5 Opening quick access and the help list

(both). The ways in (keys, menus, command center), the `?` help list, and the cycle and navigate keys.

13 features.

#### QA-5.1 Open quick access with a prefix argument

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run the quick access command with a prefix argument, for example from a keybinding with args, and the box opens with that text already typed and left as is.
- Commands: workbench.action.quickOpen
- Menus: CommandPalette (implicit, f1)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (124-152); C bundle (Go to File action with `metadata.args` prefix and `preserveValue` when a string is given, string 3228)
- Pieces:
  1. Running `workbench.action.quickOpen` with a text argument, for example from a keybinding rule with `args`, opens the quick access box with that text already typed in the field.
  2. The argument text is kept exactly as given, with the caret at its end and nothing selected, so the next key adds to it instead of replacing it.
  3. When the argument starts with a provider prefix such as `>`, `@` or `%`, the box opens on that provider's list and uses the rest of the text as the filter.
  4. Running the command with no argument opens the plain Go to File list, and the command is listed in the Command Palette as "Go to File...".

#### QA-5.3 See shortcut rows in the empty Quick Open box

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the rows and their order numbers 10, 20, 25, 40, 50, 60 and 70 are present, but there is no Open Quick Chat row)
- What: When Quick Open is started from the command center and the box is empty, the list begins with rows for Go to File, Command Palette, Search for Text, Open Quick Chat (only if quick chat is on), Go to Symbol in Editor, Start Debugging, Run Task and More, followed by the recently opened files; typing hides the rows.
- Commands: workbench.action.quickOpenWithModes; workbench.action.quickOpen; workbench.action.showCommands; workbench.action.quickTextSearch; workbench.action.quickchat.toggle; workbench.action.gotoSymbol; workbench.action.debug.selectandstart
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts (400-412, 824-870); V\src\vs\platform\quickinput\common\quickAccess.ts (152-165); V\src\vs\workbench\contrib\search\browser\searchQuickAccess.contribution.ts (18-28); V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (21-31, 42-48); V\src\vs\workbench\contrib\search\browser\search.contribution.ts (92-98); V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts (313-325); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (93-98); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (404-410); C bundle (`getHelpPicks` in class `vI` and the provider registrations with `commandCenterOrder`; searched "Open Quick Chat" with zero hits)
- Pieces:
  1. When Quick Open is opened from the command center and the box is empty, the list begins with shortcut rows in this order: Go to File, Command Palette, Search for Text, Go to Symbol in Editor, Start Debugging, Run Task and More.
  2. Quick Open opened any other way, such as with the Go to File key, shows no shortcut rows and starts with the recent files only.
  3. A "recently opened" heading and the recently opened files follow the shortcut rows.
  4. Each shortcut row shows its label, its prefix as the description, and the key bound to its command at the right edge when there is one.
  5. Picking a shortcut row switches the box to that list with its prefix typed in, for example the Command Palette row types `>` and the More row opens the help list.
  6. Typing any text in the box hides the shortcut rows.
  7. In stock an Open Quick Chat row with its quick chat key sits between Search for Text and Go to Symbol in Editor when quick chat is on, and picking it toggles quick chat, but Cursor has no such row. [stock only]

#### QA-5.5 Show the help list with ?

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Typing ? at the start of the quick access box, or picking the More row in the command center list, shows a list of every quick access prefix with a short description.
- Commands: workbench.action.quickOpen (argument prefix "?")
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\helpQuickAccess.ts (18-55); V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (21-31); C bundle (help provider class `oqe` with prefix "?", strings 8900 to 8902)
- Pieces:
  1. Typing `?` at the start of the quick access box shows the help list, which lists every quick access prefix with a short description.
  2. While the help list is open the field placeholder reads "Type '?' to get help on the actions you can take from here."
  3. Picking the More row in the command center Quick Open list opens the help list.
  4. Running `workbench.action.quickOpen` with the argument `?` opens the help list directly.

#### QA-5.6 Read the provider rows in the help list

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (no rows for Go to Offset, Report Issue, MCP resources or agent sessions; two agent rows added, "Search chat history" and "Switch between recently used agents"; extension rows still present)
- What: Rows are sorted by prefix and each shows the prefix (an ellipsis for the plain Go to File list), a description, and the key bound to the provider's command if there is one; the "?" row itself is left out.
- Commands: workbench.action.quickOpen; workbench.action.showCommands; workbench.action.gotoSymbol; workbench.action.showAllSymbols; workbench.action.quickTextSearch; workbench.action.gotoLine; workbench.action.gotoOffset; workbench.action.openView; workbench.action.showEditorsInActiveGroup; workbench.action.showAllEditors; workbench.action.showAllEditorsByMostRecentlyUsed; workbench.action.quickOpenTerm; workbench.action.debug.selectandstart; workbench.action.debug.selectDebugConsole
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\helpQuickAccess.ts (56-77); V\src\vs\workbench\contrib\search\browser\searchQuickAccess.contribution.ts (18-36); V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts (98-127); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (146-168); V\src\vs\workbench\contrib\terminalContrib\quickAccess\browser\terminal.quickAccess.contribution.ts (21-29); C bundle (provider registrations with `helpEntries`, zero hits for `workbench.action.gotoOffset`, strings 4787 and 4794)
- Pieces:
  1. The help list is sorted by prefix, so the plain Go to File entry, shown as an ellipsis, comes first.
  2. Each row shows the prefix as its label and a short description, for example `>` with "Show and Run Commands", and the key bound to the provider's command at the right edge when there is one.
  3. The `?` row itself is left out of the list.
  4. A provider with several help entries gets one row per entry, for example `@` for Go to Symbol in Editor and `@:` for Go to Symbol in Editor by Category.
  5. Each row is read to a screen reader as the prefix and the description joined by a comma.
  6. Stock also lists rows for Go to Offset, MCP resources and agent sessions, which Cursor leaves out. [stock only]
  7. Cursor adds two agent rows to the list, "Search chat history" and "Switch between recently used agents". [cursor only]

#### QA-5.7 Pick a help row to switch to that list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pressing Enter or clicking a row in the help list replaces the box text with that prefix and opens that list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\helpQuickAccess.ts (30-40); C bundle (accept handler in `oqe.provide`)
- Pieces:
  1. Pressing Enter on the active help row, or clicking a row, replaces the box text with that row's prefix and opens that list.
  2. After the switch the prefix stays as typed with the caret right after it, ready for more typing.
  3. Picking the ellipsis row for Go to File returns the box to the plain file list with no prefix.

#### QA-5.8 Jump to a list by typing its prefix after ?

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Typing "?" followed by a provider's full prefix, for example "?term " with the trailing space, switches straight to that list without picking a row.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\helpQuickAccess.ts (41-48); V\src\vs\platform\quickinput\common\quickAccess.ts (260-265); C bundle (value change handler in `oqe.provide`)
- Pieces:
  1. Typing `?` followed by a provider's full prefix, for example `?term ` with the trailing space, switches straight to that list without picking a row.
  2. The box text is replaced by just that prefix, so `?term ` becomes `term ` and the terminals list opens.
  3. A partial prefix such as `?ter`, or a lone `?`, does not switch lists and the help list stays open.

#### QA-5.9 Switch lists by typing or deleting a prefix

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Any prefix typed at the start of the box (@, :, #, %, "view ", "task " and so on) switches to that list and keeps the rest of the text, the longest matching prefix wins, and deleting the prefix goes back to Go to File. The > case is QA-1.3.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickAccess.ts (193-225, 237-250); V\src\vs\platform\quickinput\common\quickAccess.ts (225-265); C bundle (controller methods `registerPickerListeners` and `getOrInstantiateProvider`)
- Pieces:
  1. Typing a provider prefix at the very start of the box, such as `@`, `:`, `#`, `%`, `view ` or `task `, switches to that provider's list and keeps the rest of the text.
  2. When prefixes overlap, the longest matching prefix wins, for example `@:` is picked over `@`.
  3. Deleting the prefix switches the box back to the plain Go to File list.
  4. Only the text after the prefix filters the rows, so the prefix itself never hides a row.
  5. The field placeholder changes to the new list's own hint text whenever the list switches.

#### QA-5.10 Keep or reselect typed text when opening another list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Opening a different list while one is open moves your typed text under the new prefix (type "foo" in Go to File, press F1, and the box shows ">foo"), and opening the same list again only selects the typed text so the next key replaces it.
- Commands: workbench.action.showCommands; workbench.action.quickOpen
- Menus: none found
- Keys: F1; Ctrl+Shift+P (opens the palette from any list except Go to File, where the same keys move up the list, see QA-5.11)
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickAccess.ts (55-100, 172-191); C bundle (`doShowOrPick` and `adjustValueSelection` on the quick access controller)
- Pieces:
  1. Opening a different list while one is open moves your typed text under the new prefix, so typing `foo` in Go to File and then pressing F1 shows `>foo`.
  2. Opening the same list again, for example pressing F1 while the Command Palette is open, keeps the box as it is and selects the typed text after the prefix so the next key replaces it.
  3. Pressing F1 opens the Command Palette from any list, including Go to File, while Ctrl+Shift+P does so from every list except Go to File, where it moves up the list instead.

#### QA-5.11 Press a list's open key again to move through it

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With a list open, pressing its open key again moves down and adding Shift moves up, for Go to File, Open View, the editor switcher, Open Recent and terminals.
- Commands: workbench.action.quickOpenNavigateNextInFilePicker; workbench.action.quickOpenNavigatePreviousInFilePicker; workbench.action.quickOpenNavigateNextInViewPicker; workbench.action.quickOpenNavigatePreviousInViewPicker; workbench.action.quickOpenNavigateNextInEditorPicker; workbench.action.quickOpenNavigatePreviousInEditorPicker; workbench.action.quickOpenNavigateNextInRecentFilesPicker; workbench.action.quickOpenNavigatePreviousInRecentFilesPicker; workbench.action.quickOpenNavigateNextInTerminalPicker; workbench.action.quickOpenNavigatePreviousInTerminalPicker
- Menus: none found
- Keys: Ctrl+P and Ctrl+E for next, Ctrl+Shift+P and Ctrl+Shift+E for previous (Go to File, when inQuickOpen && inFilesPicker; Mac uses Cmd+P and Cmd+Shift+P only); Ctrl+Q and Ctrl+Shift+Q (Open View, when inQuickOpen && inViewsPicker; Mac uses Control+Q, none on Linux); Ctrl+Tab and Ctrl+Shift+Tab (editor switcher, when inQuickOpen && inEditorsPicker; Mac uses Control+Tab); Ctrl+R and Ctrl+Shift+R (Open Recent, when inQuickOpen && inRecentFilesPicker; Mac uses Control+R); terminal list has no default key
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (74-98); V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (120-146); V\src\vs\workbench\contrib\quickaccess\browser\viewQuickAccess.ts (228-262); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (143-144, 312-331); V\src\vs\workbench\browser\actions\windowActions.ts (470-492); V\src\vs\workbench\contrib\terminalContrib\quickAccess\browser\terminal.quickAccess.contribution.ts (30-33); C bundle (the same ids and keys for file, view, editor, recent files and terminal pickers; the file picker key table `Eje` is Ctrl+P with secondary Ctrl+E)
- Pieces:
  1. With Go to File open, pressing Ctrl+P or Ctrl+E again moves the highlight down, and Ctrl+Shift+P or Ctrl+Shift+E moves it up, with Mac using only Cmd+P and Cmd+Shift+P (`workbench.action.quickOpenNavigateNextInFilePicker` and its Previous pair).
  2. With Open View open, Ctrl+Q moves down and Ctrl+Shift+Q moves up, with Mac using Control+Q and Linux having no default key (`workbench.action.quickOpenNavigateNextInViewPicker` and its Previous pair).
  3. With the editor switcher open, Ctrl+Tab moves down and Ctrl+Shift+Tab moves up, with Mac using Control+Tab (`workbench.action.quickOpenNavigateNextInEditorPicker` and its Previous pair).
  4. With Open Recent open, Ctrl+R moves down and Ctrl+Shift+R moves up, with Mac using Control+R (`workbench.action.quickOpenNavigateNextInRecentFilesPicker` and its Previous pair).
  5. The terminal list has the commands `workbench.action.quickOpenNavigateNextInTerminalPicker` and `workbench.action.quickOpenNavigatePreviousInTerminalPicker` with no default key, so you bind your own.

#### QA-5.12 Navigate Next and Previous in Quick Open commands

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Two palette commands move the highlight down or up the open list; they have no default key, so you bind your own, and they also arm release-to-open (QA-5.13).
- Commands: workbench.action.quickOpenNavigateNext; workbench.action.quickOpenNavigatePrevious
- Menus: CommandPalette (implicit, f1)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (205-240, 280-283); C bundle (base class `SEr` and the two subclasses, strings 3229 and 3230)
- Pieces:
  1. The Command Palette lists the commands "Navigate Next in Quick Open" (`workbench.action.quickOpenNavigateNext`) and "Navigate Previous in Quick Open" (`workbench.action.quickOpenNavigatePrevious`).
  2. With a key of your own bound to them, Next moves the highlight down the open list and Previous moves it up.
  3. Neither command has a default key.
  4. Pressing a key bound to either command with a modifier held makes the list ready to open its highlighted row when the modifier is released (see QA-5.13).

#### QA-5.13 Release the held key to open the highlighted row

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: After moving through a list with a held modifier (Ctrl, Alt, Shift or Cmd, whichever the shortcut uses), letting go of that key opens the highlighted row; lists started by a cycle key such as Ctrl+Tab open with the input line hidden.
- Commands: workbench.action.quickOpenNavigateNext; workbench.action.quickOpenNavigatePrevious
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (974-1025); V\src\vs\platform\quickinput\browser\quickAccess.ts (112-123); V\src\vs\platform\quickinput\browser\quickInputController.ts (860-867); C bundle (`registerQuickNavigation` key up handler and `hideInput` set in `doShowOrPick`)
- Pieces:
  1. After moving through a list with a held modifier key (Ctrl, Alt, Shift or Cmd, whichever the shortcut uses), letting go of that key opens the highlighted row.
  2. A list started by a cycle key such as Ctrl+Tab opens with the input line hidden.
  3. If a list was already open when the cycle key was pressed, the input line stays visible.
  4. The release only works once, so if no row is highlighted when the key comes up the list stays open and later key releases do nothing.
  5. In a list with checkboxes, letting go of the held key opens nothing.

#### QA-5.14 Select Next and Previous in Quick Open

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: On Mac, Control+N and Control+P move the highlight down and up in any open list; on Windows and Linux the commands exist with no default key.
- Commands: workbench.action.quickOpenSelectNext; workbench.action.quickOpenSelectPrevious
- Menus: CommandPalette (implicit, f1)
- Keys: Ctrl+N (Mac, Control key, when inQuickOpen); Ctrl+P (Mac, Control key, when inQuickOpen)
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (243-280); V\src\vs\workbench\browser\quickaccess.ts (20-25); C bundle (the two actions with Mac `WinCtrl` N and P keys, strings 3231 and 3232)
- Pieces:
  1. On Mac, Control+N moves the highlight down and Control+P moves it up in any open list.
  2. On Windows and Linux the commands `workbench.action.quickOpenSelectNext` and `workbench.action.quickOpenSelectPrevious` have no default key and can be bound to your own keys.
  3. The Command Palette lists the commands as "Select Next in Quick Open" and "Select Previous in Quick Open".
  4. Moving with these commands only moves the highlight and never opens a row when a key is released.

#### QA-5.15 Close the open list with Escape

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Escape or Shift+Escape closes any quick access list without picking a row.
- Commands: workbench.action.closeQuickOpen
- Menus: none found
- Keys: Escape; Shift+Escape (when inQuickOpen)
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (30-40); V\src\vs\workbench\browser\quickaccess.ts (20-22); C bundle (key rule for `workbench.action.closeQuickOpen` with Escape and Shift+Escape)
- Pieces:
  1. Pressing Escape or Shift+Escape closes any open quick access list without picking a row.
  2. The close key is the command `workbench.action.closeQuickOpen`, which acts only while a quick access box is open and can be rebound.


## QA-6 Quick pick and input box widget

(both) [boundary for Cursor's preview box only]. The shared floating panel every picker reuses: list, input, title bar, buttons, checkboxes, steps. Cursor's width is a setting (default 820, stock fixed at 600) and it adds a row preview box used by two developer pickers.

26 features.

#### QA-6.1 Pick one row from a list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (controller class `kAe` builds the same parts with the same class names `.quick-input-widget`, `.quick-input-header`, `.quick-input-filter`, `.quick-input-list`; `createQuickPick` found)
- What: Any picker opens the floating box with a filter field, a placeholder, and a list. Typing narrows the rows with highlighted matching letters (label by default, description and detail only when the picker asks for it, and rows marked "always show" never hide), the first row is active on open, and Enter or a click picks the active row and closes the box. Pickers are built through `pick` and `createQuickPick`, so there is no command of its own.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\common\quickInput.ts (20-265, 509-735); V\src\vs\platform\quickinput\browser\quickInput.ts (524-1180); V\src\vs\platform\quickinput\browser\quickInputController.ts (134-451, 463-572); V\src\vs\platform\quickinput\browser\quickInputList.ts (700-800); C bundle (kAe, createQuickPick, quick-input-widget class names)
- Pieces:
  1. A picker opens as a floating box with a filter field showing the picker's placeholder and a list of rows under it, and keyboard focus starts in the filter field.
  2. Typing in the filter field narrows the rows and highlights the matching letters in each row's label.
  3. Rows are matched on the label by default, and on the description and detail only when the picker asks for it, while rows marked "always show" stay visible whatever you type.
  4. The first row is active when the list opens.
  5. Pressing Enter picks the active row and clicking a row picks that row, and either one closes the box.

#### QA-6.2 Pick several rows with checkboxes

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (workbench.action.quickPickManyToggle and the checkbox classes are present; `quickInput.toggleCheckbox` is not found as a command id, so Space handling there is not a registered key rule)
- What: When a picker allows many choices every row gets a checkbox. Space toggles the active row's box when focus is in the list (Space types a space when focus is in the filter field), Enter or OK confirms all checked rows, rows can start checked, and a disabled row shows a greyed box that cannot be changed. Rows marked not pickable show no box.
- Commands: workbench.action.quickPickManyToggle; quickInput.toggleCheckbox
- Menus: none found
- Keys: Space (quickInput.toggleCheckbox, when the box shows a quick pick or tree and the filter field is not focused); workbench.action.quickPickManyToggle has no default key
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts (270-290); V\src\vs\workbench\browser\actions\quickAccessActions.ts (99-108); V\src\vs\platform\quickinput\browser\quickInputList.ts (434-465, 1494-1506); V\src\vs\platform\quickinput\common\quickInput.ts (60-100); C bundle (workbench.action.quickPickManyToggle found, quickInput.toggleCheckbox searched with zero hits)
- Pieces:
  1. When a picker allows many choices, every row shows a checkbox at its left.
  2. Pressing Space with focus in the list toggles the active row's checkbox, while Space in the filter field types a space.
  3. Pressing Enter confirms all checked rows.
  4. A picker can open with some rows already checked.
  5. A disabled row shows a greyed checkbox that cannot be changed.
  6. Rows marked not pickable show no checkbox.
  7. The command `workbench.action.quickPickManyToggle` has no default key and, when bound, toggles the active row's checkbox.
  8. The command `quickInput.toggleCheckbox` that Space runs exists only in stock, and Cursor has no command id of that name. [stock only]

#### QA-6.3 Check all, selected count, OK and Custom buttons

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (check-all box, "{0} Selected", "OK", "Custom", okLabel, hideCheckAll and hideCountBadge found; the option that draws the Custom button in a secondary style, `customButtonSecondary`, has zero hits)
- What: Multi-select pickers show a check-all box at the left of the header (tooltip "Toggle all checkboxes") that checks every visible row, a "{0} Selected" count badge at the right of the filter field, and an OK button (label can be changed) that confirms. A picker can also show a Custom button with its own label and hover text. Pickers can hide the check-all box or the badge, and can force OK on a single-select list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInputController.ts (172-226, 760-800); V\src\vs\platform\quickinput\browser\quickInput.ts (558-567, 789-874, 1036-1050); V\src\vs\platform\quickinput\browser\quickInputList.ts (1094-1110); C bundle (nls "Toggle all checkboxes" 2248, "{0} Selected" 2250, "OK" 2251, "Custom" 2252)
- Pieces:
  1. A multi-select picker shows a check-all box at the left of the header with the tooltip "Toggle all checkboxes", and clicking it checks every visible row and clicking it again unchecks them.
  2. A count badge at the right of the filter field reads "{0} Selected" with the number of checked rows.
  3. An OK button confirms the checked rows, and its label reads "OK" unless the picker sets another label.
  4. A picker can show the OK button on a single-select list too.
  5. A picker can show a Custom button in the header with its own label and its own hover text.
  6. A picker can hide the check-all box or the count badge.
  7. In stock a picker can draw the Custom button in a secondary style, which Cursor does not support. [stock only]

#### QA-6.4 Type a value in an input box

- Tag: both. Evidence: source. Boundary: no. Cursor: present (createInputBox, valueSelection, prompt message strings found)
- What: An input prompt shows a title, a text field with placeholder and optional preset text (with a chosen part of it selected), and a line under the field that reads "{prompt} (Press 'Enter' to confirm or 'Escape' to cancel)" or, with no prompt, "Press 'Enter' to confirm your input or 'Escape' to cancel". Enter confirms and Escape cancels. The box has no list.
- Commands: none found
- Menus: none found
- Keys: Enter; Escape (see QA-6.22)
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (154-170, 1182-1325); V\src\vs\platform\quickinput\browser\quickInputController.ts (593-651, 662-664); C bundle (createInputBox, nls "Press 'Enter' to confirm your input or 'Escape' to cancel", "{0} (Press 'Enter' to confirm or 'Escape' to cancel)")
- Pieces:
  1. An input prompt shows a text field with the prompt's placeholder and no list under it.
  2. The field can open with preset text, with a chosen part of that text already selected.
  3. A line under the field reads "{prompt} (Press 'Enter' to confirm or 'Escape' to cancel)".
  4. When the prompt has no prompt text, the line under the field reads "Press 'Enter' to confirm your input or 'Escape' to cancel".
  5. Pressing Enter confirms the typed text and closes the box, and pressing Escape cancels it.

#### QA-6.5 Type a hidden password

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the input box sets `password` from the option)
- What: An input prompt can switch the field to password mode, where typed characters show as dots. The rest of the prompt (placeholder, prompt line, validation) works the same.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (1189, 1234-1245, 1306-1308); V\src\vs\platform\quickinput\browser\quickInputController.ts (647); C bundle (input box `password` assignment in the input function)
- Pieces:
  1. An input prompt can switch the field to password mode, where typed characters show as dots.
  2. In password mode the placeholder, the prompt line and the validation message work the same as in a normal prompt.
  3. Password mode applies only to that prompt, so the next prompt shown in the same box has plain text again.

#### QA-6.6 See a validation message under the input box

- Tag: both. Evidence: source. Boundary: no. Cursor: present (validateInput, validationMessage found 33 and 46 times)
- What: A prompt can check the text and show a message under the field with an info, warning or error look (colored text and background, a border, and a matching border on the field). The check runs about 100 ms after you stop typing and again when you press Enter. An error blocks Enter, while warning and info do not. The message text can hold clickable links (including command links) and inline icons written as `$(name)`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (318-333, 451-512); V\src\vs\platform\quickinput\browser\quickInputController.ts (575-651); V\src\vs\platform\quickinput\browser\quickInputUtils.ts (141-180); V\src\vs\platform\quickinput\browser\quickInputBox.ts (165-175); C bundle (validateInput, validationMessage)
- Pieces:
  1. A prompt can check the typed text and show a message under the field with an info, warning or error look, which is colored text and background with a border and a matching border on the field.
  2. The check runs about 100 ms after you stop typing and runs again when you press Enter.
  3. An error blocks Enter and keeps the box open, while a warning or an info message does not block it, and a check result given as plain text counts as an error.
  4. The message text can hold clickable links, including command links, and inline icons written as `$(name)`.
  5. When the check finds nothing wrong the message and the field border go away.

#### QA-6.7 See a title, description and step counter

- Tag: both. Evidence: source. Boundary: no. Cursor: present (totalSteps found 9 times, nls "{0}/{1}" found, `.quick-input-titlebar` and `.quick-input-title` classes found)
- What: A title bar appears above the field when the picker has a title, a step number, or title buttons. It shows the title text with "(2/3)" after it for a multi-step flow (just the number when the total is unknown), and an optional description line under the title area. With no title, step or title button the bar is hidden.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (153-232, 388-401, 470-496, 1036-1050); V\src\vs\platform\quickinput\browser\quickInputController.ts (154-175); C bundle (nls "{0}/{1}", totalSteps, quick-input-titlebar)
- Pieces:
  1. A title bar appears above the field when the picker has a title, a step number or title buttons, and it shows the title text.
  2. In a multi-step flow the title is followed by the step as "(2/3)", or by just "(2)" when the total is unknown, and with no title the bar shows only the step.
  3. An optional description line appears under the title area.
  4. With no title, no step and no title button, the title bar is hidden.

#### QA-6.8 Go back a step in a multi-step flow

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id found twice; the key rule with the same three platform keys found; nls "Back" and "Back ({0})" found)
- What: Multi-step pickers show a Back arrow at the left of the title bar (tooltip "Back", with the bound key added in brackets when there is one). Clicking it or pressing the key returns to the previous step.
- Commands: workbench.action.quickInputBack
- Menus: none found
- Keys: Alt+Left (Windows); Control+- (Mac); Ctrl+Alt+- (Linux); all when inQuickOpen
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (110-122); V\src\vs\platform\quickinput\browser\quickInput.ts (91-96, 270-316); V\src\vs\platform\quickinput\browser\quickInputController.ts (725-735, 875-885); V\src\vs\workbench\services\quickinput\browser\quickInputService.ts (43); C bundle (workbench.action.quickInputBack rule, quickInputBack icon, nls "Back")
- Pieces:
  1. A multi-step picker shows a Back arrow at the left of the title bar.
  2. Hovering the Back arrow shows the tooltip "Back", with the bound key added in brackets when there is one.
  3. Clicking the Back arrow returns to the previous step.
  4. Pressing the key bound to `workbench.action.quickInputBack` while the box is open returns to the previous step, with Alt+Left on Windows, Control+- on Mac and Ctrl+Alt+- on Linux.

#### QA-6.9 Use buttons in the title bar and around the filter field

- Tag: both. Evidence: source. Boundary: no. Cursor: present (location values Title, Inline and Input found, `.quick-input-left-action-bar`, `.quick-input-right-action-bar` and `.quick-input-inline-action-bar` found; the overflow for secondary buttons was not traced)
- What: A picker can add icon buttons in three places. In the title bar (right side, optionally toggles, or tucked into an overflow menu with a label when marked secondary), beside the filter field on its right ("inline"), and inside the field at its far end as on and off toggles (the selected-count badge moves left to make room). Examples are the Reset and Close buttons of Customize Layout, "Open in Search View" in Quick Search, and the zero-based toggle in Go to Offset (stock only).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\common\quickInput.ts (800-870); V\src\vs\platform\quickinput\browser\quickInput.ts (270-316, 418-447); V\src\vs\platform\quickinput\browser\quickInputController.ts (154-212); V\src\vs\workbench\browser\actions\layoutActions.ts (1517-1600); V\src\vs\workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts (111); C bundle (button location enum, action bar class names)
- Pieces:
  1. A picker can add icon buttons at the right side of the title bar, as plain buttons or as toggles, for example the Reset and Close buttons of Customize Layout.
  2. Title bar buttons marked secondary are tucked into an overflow menu with a label instead of showing as icons.
  3. A picker can add inline icon buttons beside the right end of the filter field, for example "Open in Search View" in Quick Search.
  4. A picker can add on and off toggle buttons inside the far end of the filter field, for example the zero-based toggle in the stock Go to Offset list.
  5. When toggle buttons sit inside the filter field the selected-count badge moves left to make room for them.

#### QA-6.10 Use buttons on a row

- Tag: both. Evidence: source. Boundary: no. Cursor: present (`alwaysVisible` found 16 times, onDidTriggerItemButton found, row action bar class found)
- What: A row can show one or more icon buttons at its right end. By default they appear when you hover the row or move focus to it, and a button marked "always visible" stays on screen (the unsaved dot in the editor lists). Clicking one fires that row's action without picking the row, and the picker can then keep the list as is, drop the row, refresh the list, or close the box. Examples are "Close Editor", "Rename Terminal", "Configure Task" and "Remove from Recently Opened". Buttons can be toggles, or sit in an overflow menu when marked secondary.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\common\quickInput.ts (821-880); V\src\vs\platform\quickinput\browser\quickInputList.ts (324-420, 548-565); V\src\vs\platform\quickinput\browser\quickInputUtils.ts (60-140); V\src\vs\platform\quickinput\browser\pickerQuickAccess.ts; V\src\vs\platform\quickinput\browser\media\quickInput.css (349-360); C bundle (alwaysVisible, onDidTriggerItemButton)
- Pieces:
  1. A row can show one or more icon buttons at its right end, for example "Close Editor", "Rename Terminal", "Configure Task" and "Remove from Recently Opened".
  2. By default the row buttons appear when you hover the row or move keyboard focus to it, and a button marked "always visible" stays on screen, such as the unsaved dot in the editor lists.
  3. Clicking a row button fires that row's action without picking the row.
  4. After a row button is clicked the picker decides what happens next, which is keeping the list as it is, dropping the row, refreshing the list or closing the box.
  5. A row button can be a toggle that shows an on or off state.
  6. Row buttons marked secondary sit in an overflow menu instead of showing as icons.

#### QA-6.11 Group rows under separator headings

- Tag: both. Evidence: source. Boundary: no. Cursor: present (separator elements and `onDidTriggerSeparatorButton` found 8 times, the class `quick-input-list-separator-as-item` found 3 times)
- What: Pickers that use separators show heading rows with a label, an optional description and an optional tooltip, and a thin line marks where each group starts. A heading with buttons becomes its own row that can be focused and shows its buttons (for example a group-level action), while a plain heading is drawn above its first row. Headings whose rows are all filtered out disappear.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\common\quickInput.ts (103-114, 884-895); V\src\vs\platform\quickinput\browser\quickInputList.ts (256-272, 592-700, 765-780, 1316-1370); C bundle (separator renderer, onDidTriggerSeparatorButton)
- Pieces:
  1. Pickers that use separators show heading rows with a label.
  2. A heading can also show a description and a tooltip.
  3. A thin line marks where each group of rows starts.
  4. A heading with buttons becomes its own row that can be focused and shows its buttons, while a plain heading is drawn above its first row.
  5. A heading whose rows are all filtered out disappears.

#### QA-6.12 See section header rows and custom drawn rows

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor's list adds a third row kind besides items and separators: a section header that cannot be clicked, drawn dimmed at 25 px tall (5 px when it is only a thin divider, with an optional line above), and rows or headers that supply their own DOM and are measured off screen. Cursor also spaces list rows 3 px apart. No picker in the main bundle was found that creates a section header or a custom drawn row, so what the person sees depends on pickers outside this section (the Go to File code only passes them through).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (class `dPt` with `sectionHeader`, renderer `y8m` with `.quick-input-list-section-header`, delegate `MCk` heights 25 and 5, `customRenderedDom`, `isPureDivider`, `shouldShowDividerAbove`, tree option `itemGap:3`; workbench.desktop.main.css rules for `.quick-input-list-section-header`)
- Pieces:
  1. Cursor's list can show a section header row that cannot be clicked, drawn dimmed at 25 px tall. [cursor only]
  2. A section header can be only a thin 5 px divider, with an optional line above it. [cursor only]
  3. Rows and section headers can supply their own custom content, and the list measures its height off screen before drawing it. [cursor only]

#### QA-6.13 Read a row: icon, label, description, detail and shortcut

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (all the row parts and the `.quick-input-list-entry-keybinding` class found; the list tree has an extra 3 px gap between rows and can measure custom drawn rows, see QA-6.12)
- What: A row can show an icon (a file type icon, a theme icon written `$(name)` in the label, an image, or an icon class), the label with highlighted matches, a dimmer description to its right, a second line of detail text (rows with detail are taller), and a shortcut label at the far right for the command's key. Rows can be italic (preview editors) or struck through, and a long description or detail gets its own small hover with the full text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\common\quickInput.ts (30-100); V\src\vs\platform\quickinput\browser\quickInputList.ts (86-200, 324-420, 418-565, 274-290); C bundle (quick-input-list-entry-keybinding, list tree options)
- Pieces:
  1. A row can show an icon before the label, which is a file type icon, a theme icon written `$(name)` in the label, an image or an icon class.
  2. The label shows the matching letters highlighted, with a dimmer description to its right.
  3. A row can have a second line of detail text, and rows with detail are taller.
  4. A shortcut label at the far right of the row shows the key bound to the command when the picker supplies one.
  5. A row can be drawn in italic, as for preview editors, or struck through.
  6. A long description or detail gets its own small hover with the full text.
  7. Cursor leaves a 3 px gap between rows in the list. [cursor only]

#### QA-6.14 Hover a row to read its tooltip

- Tag: both. Evidence: source. Boundary: no. Cursor: present (quickInput.toggleHover rule found, same key)
- What: Pausing the pointer over a row, or pressing Ctrl+Space while a row is active, shows a hover to the right of the row with the row's tooltip. Rows with no tooltip show no row hover. Pressing Ctrl+Space again hides it, and while it is on it follows the active row.
- Commands: quickInput.toggleHover
- Menus: none found
- Keys: Ctrl+Space (Control+Space on Mac; when the box shows a quick pick or tree)
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts (292-305); V\src\vs\platform\quickinput\browser\quickInputList.ts (975-1000, 1513-1535, 1595-1615); C bundle (quickInput.toggleHover with Ctrl or Control plus Space)
- Pieces:
  1. Pausing the pointer over a row shows a hover to the right of the row with that row's tooltip.
  2. Pressing Ctrl+Space, or Control+Space on Mac, while a row is active shows the same hover for that row (`quickInput.toggleHover`).
  3. Pressing Ctrl+Space again hides the hover, and while it is on it follows the active row.
  4. Rows with no tooltip show no row hover.

#### QA-6.15 See a progress bar while a list is loading

- Tag: both. Evidence: source. Boundary: no. Cursor: present (`.quick-input-progress` class found; the 800 ms delay was not traced)
- What: While a picker is busy (for example waiting on a search) a thin progress bar runs under the filter field. It only appears after the picker has been busy for 800 ms, and it is hidden from screen readers.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (402-417); V\src\vs\platform\quickinput\browser\quickInputController.ts (227-229, 716); C bundle (quick-input-progress)
- Pieces:
  1. While a picker is busy, a thin progress bar runs under the filter field.
  2. The bar appears only after the picker has been busy for 800 ms, so quick lists never show it.
  3. The bar goes away as soon as the picker stops being busy.
  4. The progress bar is hidden from screen readers.

#### QA-6.16 Keep the box open

- Tag: both. Evidence: source. Boundary: no. Cursor: present (quickInput.acceptInBackground rule found with the same key and weight; workbench.quickOpen.closeOnFocusLost found)
- What: Two behaviors keep a picker on screen. Right arrow (or Alt, Ctrl or Cmd with Right arrow) picks the active row without closing the box, when focus is in the list or the caret is at the end of the filter field, for pickers that allow it such as the editor, terminal, debug console and Go to File lists. And clicking elsewhere closes the box unless the picker asks to stay open or the setting is turned off.
- Commands: quickInput.acceptInBackground
- Menus: none found
- Keys: Right arrow (quickInput.acceptInBackground, when the box shows a quick pick and either the filter field is not focused or its caret is at the end); the same with Alt, Ctrl and Cmd held
- Settings: workbench.quickOpen.closeOnFocusLost
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts (230-250); V\src\vs\workbench\services\quickinput\browser\quickInputService.ts (42); V\src\vs\platform\quickinput\browser\quickInputController.ts (320-340); V\src\vs\platform\quickinput\browser\quickInput.ts (673-680, 1165-1180); V\src\vs\workbench\browser\workbench.contribution.ts (536-540); C bundle (quickInput.acceptInBackground, canAcceptInBackground, workbench.quickOpen.closeOnFocusLost)
- Pieces:
  1. Pressing Right arrow, or Alt, Ctrl or Cmd with Right arrow, picks the active row without closing the box, for pickers that allow it such as the editor, terminal, debug console and Go to File lists (`quickInput.acceptInBackground`).
  2. That key works when focus is in the list or the caret is at the end of the filter field, and with the caret anywhere else in the field Right arrow just moves the caret.
  3. Clicking elsewhere closes the box unless the picker asks to stay open.
  4. Setting `workbench.quickOpen.closeOnFocusLost` chooses whether the box closes when it loses focus, with values true, the default, and false.

#### QA-6.17 Put the box at the top or the center

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids, nls "Align Quick Input Top", "Align Quick Input Center" and "Quick Input Position" found, context key `quickInputAlignment` found)
- What: Two commands snap the box to the top of the window under the title bar (the default) or to the vertical center, and Customize Layout shows them under a "Quick Input Position" heading with a mark on the current one (the box is also reset to the top by that dialog's Reset button). Neither command is in the palette.
- Commands: workbench.action.alignQuickInputTop; workbench.action.alignQuickInputCenter
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (55-56, 1280-1310, 1385-1388, 1503-1509, 1604); V\src\vs\platform\quickinput\browser\quickInput.ts (43-44); V\src\vs\platform\quickinput\browser\quickInputController.ts (667-672, 1129-1148); C bundle (alignQuickInputTop 3 hits, alignQuickInputCenter 2 hits)
- Pieces:
  1. The command `workbench.action.alignQuickInputTop` snaps the box to the top of the window under the title bar, which is the default place.
  2. The command `workbench.action.alignQuickInputCenter` snaps the box to the vertical center of the window.
  3. Customize Layout shows both commands under a "Quick Input Position" heading with a mark on the current one.
  4. Neither command is in the Command Palette, so you reach them only from the Customize Layout list.
  5. The Reset button of Customize Layout puts the box back at the top.

#### QA-6.18 Drag the box and have it remember its place

- Tag: both. Evidence: source. Boundary: no. Cursor: present (drag controller with snap threshold 20, snap ratio 0.25 and the same storage key found; its double-click reset does not skip the title bar buttons and does not set the top mark, so the code is a little older than stock)
- What: Dragging the title bar, or the empty part of the header, moves the box anywhere inside the window. It snaps to the top, the vertical center and the horizontal center within 20 px, stays clear of the window controls when the title bar is drawn by the app, and the place is saved between sessions. Double-clicking the title bar or header puts it back at the top. The saved place is a fraction of the window, so it adapts to resizing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInputController.ts (45-50, 359-405, 1041-1050, 1055-1275); C bundle (workbench.quickInput.viewState, _snapThreshold=20, _snapLineHorizontalRatio=.25, registerMouseListeners)
- Pieces:
  1. Dragging the title bar, or the empty part of the header, moves the box anywhere inside the window.
  2. While dragging, the box snaps to the top, the vertical center and the horizontal center when it comes within 20 px of them.
  3. When the app draws the title bar itself, the box stays clear of the window controls.
  4. The place is saved between sessions as a fraction of the window, so it adapts when the window is resized.
  5. Double-clicking the title bar or the header puts the box back at the top.
  6. In stock a double-click on a title bar button does not reset the box and the reset also sets the top mark in Customize Layout, which Cursor's older code does not do. [stock only]

#### QA-6.19 Set how wide the box is

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Stock draws the box at 62 percent of the window width up to a fixed 600 px. Cursor adds the setting `workbench.quickOpen.width` (default 820, allowed 580 to 1000) that raises that cap for Quick Open and every other picker, and the box re-lays out as soon as the value changes. The width is still limited to 62 percent of the window, and a value you have not set falls back to a built-in default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.quickOpen.width
- Source: C bundle (setting `workbench.quickOpen.width` with default 820, minimum 580, maximum 1e3 and message 4209; controller `kAe` getConfiguredWidth, clampConfiguredWidth and updateLayout using Math.min(width*.62, configured); onDidChangeConfiguration re-layout); V\src\vs\platform\quickinput\browser\quickInputController.ts (53, 895-899) for the stock fixed cap
- Pieces:
  1. Setting `workbench.quickOpen.width` sets the width cap of the box for Quick Open and every other picker, with a default of 820 and allowed values from 580 to 1000. [cursor only]
  2. The box re-lays out as soon as the value changes. [cursor only]
  3. The width is still limited to 62 percent of the window, so on a narrow window the box is narrower than the setting. [cursor only]
  4. When the setting has no value, the box uses the built-in default width. [cursor only]
  5. Stock has no width setting and draws the box at 62 percent of the window up to a fixed 600 px. [stock only]

#### QA-6.20 See a preview box beside the active row (Cursor)

- Tag: cursor. Evidence: bundle. Boundary: yes - the box shows Cursor Tab request details (model name, time and request data) from the developer picker "Developer: Report Cursor Tab Action". Only the widget side is cataloged.. Cursor: cursor only
- What: When the active row, or the hovered row for rows that supply their own preview, holds report data, a floating box 420 px wide and 320 px tall with a rounded border and shadow opens to the right of the list showing a heading line with the time and then the details recorded for the row, opens under the row as wide as the row if it would run off the right edge of the window, and closes when the active row changes to one without data or when the picker closes.
- Commands: workbench.action.reportCppAction
- Menus: CommandPalette (Developer: Report Cursor Tab Action, f1)
- Keys: none found
- Settings: none found
- Source: C bundle (`quickInputService2.renderPreviewBox`, `showPreview` and `hidePreview` on the controller, `quick-input-preview-box-content`, component `xfw`, constants 420 and 320, row field `cppReportEvent` set by the Report Cursor Tab Action picker `gPk`, row field `customPreview` read but never set in the main bundle)
- Pieces:
  1. When the active row holds Cursor Tab report data, a floating box opens to the right of the list, 420 px wide and 320 px tall, with a rounded border and a shadow. [cursor only]
  2. The box shows a heading line with the time, then the details recorded for the row. [cursor only]
  3. If the box would run off the right edge of the window, it opens under the row instead, as wide as the row. [cursor only]
  4. The box closes when the active row changes to one without data and when the picker closes. [cursor only]
  5. For rows that supply their own preview, hovering the row opens the box too. [cursor only]
  6. The command `workbench.action.reportCppAction`, listed in the Command Palette as "Developer: Report Cursor Tab Action", opens the picker whose rows show this box. [cursor only]

#### QA-6.21 Move through the list with the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all ten ids found, including the Mac and non-Mac separator variants)
- What: Up and Down move one row, PageUp and PageDown move a page, Ctrl+Home and Ctrl+End go to the first and last row, and Alt+Up and Alt+Down jump to the previous or next heading (Ctrl+Alt+Up and Ctrl+Alt+Down always do). The same keys work with Ctrl or Alt added so a held shortcut such as Ctrl+P can keep stepping, and in a list opened by a held key Alt+Up and Alt+Down only step one row.
- Commands: quickInput.next; quickInput.previous; quickInput.pageNext; quickInput.pagePrevious; quickInput.first; quickInput.last; quickInput.nextSeparatorWithQuickAccessFallback; quickInput.previousSeparatorWithQuickAccessFallback; quickInput.nextSeparator; quickInput.previousSeparator
- Menus: none found
- Keys: Down and Up (also Ctrl+Down and Ctrl+Up); PageDown and PageUp; Ctrl+Home and Ctrl+End (Control on Mac); Alt+Down and Alt+Up for the headings with held-key fallback (Cmd+Down and Cmd+Up on Mac); Ctrl+Alt+Down and Ctrl+Alt+Up for the headings (Cmd+Alt+Down and Cmd+Alt+Up on Mac); all when the box shows a quick pick or tree
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts (14-205); C bundle (the ten quickInput.* ids, focus helper `Mle`)
- Pieces:
  1. Down and Up move the active row by one, and so do Ctrl+Down and Ctrl+Up (`quickInput.next` and `quickInput.previous`).
  2. PageDown and PageUp move the active row by a page (`quickInput.pageNext` and `quickInput.pagePrevious`).
  3. Ctrl+Home and Ctrl+End, or Control+Home and Control+End on Mac, go to the first and last row (`quickInput.first` and `quickInput.last`).
  4. Alt+Down and Alt+Up jump to the next or previous heading, with Cmd+Down and Cmd+Up on Mac (`quickInput.nextSeparatorWithQuickAccessFallback` and `quickInput.previousSeparatorWithQuickAccessFallback`).
  5. Ctrl+Alt+Down and Ctrl+Alt+Up always jump to the next or previous heading, with Cmd+Alt+Down and Cmd+Alt+Up on Mac (`quickInput.nextSeparator` and `quickInput.previousSeparator`).
  6. In a list opened by a held key, Alt+Up and Alt+Down only step one row instead of jumping to a heading.
  7. The page and first and last moves also work with Alt, Ctrl or Cmd added, so a held shortcut such as Ctrl+P can keep stepping through the list.

#### QA-6.22 Accept, cancel and focus the box by key

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (`quickInput.accept` and `quickInput.hide` are not found as ids, and the box handles Enter, Escape and Tab in its own key handler: Enter accepts, Escape hides with the gesture reason, Tab cycles focus through the row buttons, filter field, list, message links and custom widget; the four workbench.action.* commands are found)
- What: Enter picks the active row and Escape cancels. Holding Alt, Ctrl, Cmd or Shift with Enter passes that modifier to the picker (for example to open a file to the side). Escape and Shift+Escape also close any quick access list. Three more commands with no default key let you bind your own keys to accept, accept with the Ctrl modifier, and move focus into the box.
- Commands: quickInput.accept; quickInput.hide; workbench.action.closeQuickOpen; workbench.action.acceptSelectedQuickOpenItem; workbench.action.alternativeAcceptSelectedQuickOpenItem; workbench.action.focusQuickOpen
- Menus: none found
- Keys: Enter (and Alt, Ctrl, Cmd or Shift with Enter; Enter is ignored while an input method is composing); Escape (and Alt, Ctrl or Cmd with Escape); Shift+Escape (workbench.action.closeQuickOpen, when inQuickOpen)
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts (209-265); V\src\vs\workbench\browser\actions\quickAccessActions.ts (30-72); C bundle (the controller key handler with `case ve.Enter`, `case ve.Escape`, `case ve.Tab`; closeQuickOpen, acceptSelectedQuickOpenItem, alternativeAcceptSelectedQuickOpenItem, focusQuickOpen found; quickInput.accept and quickInput.hide searched with no match)
- Pieces:
  1. Pressing Enter picks the active row, and holding Alt, Ctrl, Cmd or Shift with Enter passes that modifier to the picker, for example to open a file to the side.
  2. Enter is ignored while an input method is composing text.
  3. Pressing Escape, Shift+Escape, or Alt, Ctrl or Cmd with Escape cancels the box and closes it.
  4. The command `workbench.action.acceptSelectedQuickOpenItem` has no default key and, when bound, picks the active row.
  5. The command `workbench.action.alternativeAcceptSelectedQuickOpenItem` has no default key and, when bound, picks the active row as if Ctrl were held with Enter.
  6. The command `workbench.action.focusQuickOpen` has no default key and, when bound, moves keyboard focus into the box.
  7. In Cursor, pressing Tab in the box cycles focus through the row buttons, the filter field, the list, the message links and a custom widget instead of leaving the box. [cursor only]
  8. In stock, Enter and Escape run the commands `quickInput.accept` and `quickInput.hide`, which Cursor does not have because its box handles those keys in its own key handler. [stock only]

#### QA-6.23 Use the box with a screen reader

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (listbox and option roles, the live count regions, aria-activedescendant and screen reader mode checks found; stock's list focus mode on the filter field (`setListFocusMode`, aria-haspopup=listbox) has no match in the bundle)
- What: The list is exposed as a listbox named "Quick Input" whose rows are options (checkboxes in multi-select), each row reads its label, description and detail (plus its heading), and the filter field names the picker from its placeholder and title, or "Type to narrow down results." The result count ("{0} Results") and the selected count ("{0} Selected") are announced politely as they change, the active row is tied to the field with aria-activedescendant, and the progress bar is hidden from the reader. In screen reader mode the list refreshes the focused row after each filter so it is read again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickInputList.ts (86-110, 292-320, 1150-1195); V\src\vs\platform\quickinput\browser\quickInputController.ts (188-205, 236-256, 340-358); V\src\vs\platform\quickinput\browser\quickInputBox.ts (120-160); V\src\vs\platform\quickinput\browser\quickInput.ts (526, 1063-1085); C bundle (aria-live on `.quick-input-visible-count`, messages 2249 and 2250, aria-activedescendant, isScreenReaderOptimized)
- Pieces:
  1. The list is exposed to a screen reader as a listbox named "Quick Input" whose rows are options, or checkboxes in a multi-select picker.
  2. Each row reads its label, its description and its detail, plus the heading it sits under.
  3. The filter field is named from the picker's placeholder and title, or "Type to narrow down results." when it has neither.
  4. The result count "{0} Results" and the selected count "{0} Selected" are announced politely as they change.
  5. The active row is tied to the filter field with `aria-activedescendant`, so moving the highlight reads the new row.
  6. In screen reader mode the list refreshes the focused row after each filter so it is read again.
  7. Stock's list focus mode on the filter field, which sets `aria-haspopup` to listbox, does not exist in Cursor. [stock only]

#### QA-6.24 Pin rows to the top of a list

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the pin function, the "pinned" heading, "Pin command" and "Pinned command" messages found in use)
- What: Rows in the Run Task list and the terminal Run Recent Command list show a pin button on hover (tooltip "Pin command"; "Pinned command" once pinned). Pinned rows move into a "pinned" section at the top while the filter field is empty, and pressing the button again unpins. The choice is saved per workspace.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\quickPickPin.ts (1-125); V\src\vs\workbench\contrib\tasks\browser\taskQuickPick.ts (311-320); V\src\vs\workbench\contrib\terminalContrib\history\browser\terminalRunRecentQuickPick.ts (374); C bundle (function `iag` with separator label message 2257, messages 2258 and 2259)
- Pieces:
  1. Rows in the Run Task list and the terminal Run Recent Command list show a pin button on hover with the tooltip "Pin command".
  2. Once a row is pinned, its pin button tooltip reads "Pinned command".
  3. Pinned rows move into a "pinned" section at the top of the list while the filter field is empty.
  4. Pressing the pin button again unpins the row.
  5. The pinned choices are saved per workspace.

#### QA-6.25 Open and close the box with a short animation

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: newer-than-1.128. In VS Code 1.144.0 the box fades in while growing from 97 percent size over 250 ms and fades out over 150 ms (it ignores the pointer while closing), but only when the window uses the modern UI style and motion is enabled. The 1.128.0 source and the Cursor bundle have no such animation.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: N\src\vs\platform\quickinput\browser\quickInputController.ts (close animation, `quick-input-widget-closing`, `QUICK_INPUT_CLOSE_ANIMATION_DURATION`); N\src\vs\platform\quickinput\browser\media\quickInput.css (16-45, keyframes open and close); C bundle (quick-input-widget-closing, quick-input-widget-open, modern-ui and monaco-enable-motion searched in the bundle and its CSS with zero hits)
- Pieces:
  1. Newer than 1.128: in VS Code 1.144.0 the box fades in while growing from 97 percent of its size over 250 ms. [stock only]
  2. In VS Code 1.144.0 the box fades out over 150 ms when it closes, and it ignores the pointer while closing. [stock only]
  3. The open and close animation plays only when the window uses the modern UI style and motion is enabled. [stock only]

#### QA-6.26 Pick a recent AI action to report (developer picker)

- Tag: cursor. Evidence: bundle. Boundary: yes - the rows are recent AI generation records and the follow-up step sends feedback about them. Only the pickers are cataloged.. Cursor: cursor only
- What: Run "Developer: Report AI Action" (the registered original text is "See Recent AI Actions") to open a quick pick "Select a recent AI action" with up to 30 recent actions, each row showing its type, a shortened description and its id, with the time as the description. Picking a row opens a second pick "Choose an action (ai)" with "Report Good", "Report Bad", "Report with Comment" (opens an input box "Type your comment here..."), "Copy Request ID" (greyed with "Disabled in privacy mode" when privacy mode is on) and, for developer users only, "Open in Prompt Quality". These rows do not set the preview field, so no preview box was found for this picker.
- Commands: workbench.action.reportAIAction
- Menus: CommandPalette (Developer: Report AI Action, f1)
- Keys: none found
- Settings: none found
- Source: C bundle (class with `ID="workbench.action.reportAIAction"` and `LABEL="Developer: Report AI Action"`, picker function `mPk`, second step `oGm`, feedback function `aGm`)
- Pieces:
  1. Running "Developer: Report AI Action" (`workbench.action.reportAIAction`) from the Command Palette opens a quick pick reading "Select a recent AI action". [cursor only]
  2. The list holds up to 30 recent actions, each row showing its type, a shortened description and its id, with the time as the description. [cursor only]
  3. Picking a row opens a second pick "Choose an action (ai)" with the rows "Report Good", "Report Bad" and "Report with Comment". [cursor only]
  4. Picking "Report with Comment" opens an input box with the prompt "Type your comment here...". [cursor only]
  5. The "Copy Request ID" row is greyed with "Disabled in privacy mode" when privacy mode is on. [cursor only]
  6. The "Open in Prompt Quality" row appears only for developer users. [cursor only]
