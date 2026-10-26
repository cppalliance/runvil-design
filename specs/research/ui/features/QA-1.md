Group: QA-1 Command Palette. Section: Quick access. Date: 2026-10-25. Pass: 2.
Features: 21.

### QA-1.1 Open the Command Palette
- Tag: both
- Evidence: source
- Boundary: no
- What: Press F1 or Ctrl+Shift+P, or pick "Command Palette..." or "Show All Commands" from a menu, to open the list where you type a command name (placeholder "Type the name of a command to run.") and run it.
- Commands: workbench.action.showCommands
- Menus: MenubarViewMenu (1_open, "Command Palette..."); MenubarHelpMenu (1_welcome, "Show All Commands"); GlobalActivity (1_command, "Command Palette..."); EditorContext (z_commands, "Command Palette...", hidden in simple input editors); CommandPalette (implicit, f1)
- Keys: Ctrl+Shift+P (Cmd+Shift+P on Mac, not bound on Firefox); F1
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (273-295); V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (42-48, 55-108); C bundle (class `Btg` ShowAllCommandsAction with the same F1 and Ctrl+Shift+P keys, menu rows on MenubarViewMenu, MenubarHelpMenu, GlobalActivity and EditorContext, strings 8898 and 8903-8909)

### QA-1.2 Type to filter commands by name
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing narrows the list to commands whose name (shown as "Category: Title") holds your words or text, or whose id you typed in full, with the matching letters highlighted and a "No matching commands" row when nothing fits.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (49-60, 99-125); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (84-88, 237-243); C bundle (base provider class `Kke` with the same word filter, provider class `aqe`, strings 8892 and 8894)

### QA-1.3 Switch to commands by typing > in the Quick Open box
- Tag: both
- Evidence: source
- Boundary: no
- What: Type > at the start of the Go to File box to turn it into the Command Palette, and delete it to go back; the same list opens straight away when a command or keybinding runs quick access with the argument ">". The other prefixes are QA-5.9.
- Commands: workbench.action.quickOpen (argument prefix ">"); workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (51); V\src\vs\platform\quickinput\browser\quickAccess.ts (210-225); V\src\vs\workbench\browser\actions\quickAccessActions.ts (125-152); C bundle (prefix constant on `Kke`, quick access controller method `doShowOrPick`, Go to File action with a prefix argument, string 3228)

### QA-1.4 See each command's keybinding on its row
- Tag: both
- Evidence: source
- Boundary: no
- What: Each row shows the key combination bound to that command on its right edge, and a screen reader reads "name, keys".
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (272-302); C bundle (method `toCommandPick` on `Kke` with the keybinding lookup, string 2232)

### QA-1.5 Find commands by their English name in another display language
- Tag: both
- Evidence: source
- Boundary: no
- What: When the display language is not English, each row also shows the English name on a second line, and typing the English name still finds the command.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (84-88, 244-252); V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (99-118, 296-300); C bundle (option `showAlias` set from the language variant in `aqe`, alias filter in `Kke`)

### QA-1.6 Order results by recent use, then name, with Developer commands last
- Tag: both
- Evidence: source
- Boundary: no
- What: Results list commands you ran before first, newest first, then (if the experimental list is on) suggested ones, then everything else A to Z, with commands in the "Developer" category pushed below the rest.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (no Developer-last rule because the bundle's command rows have no category field; adds the Open Browser pin, see QA-1.21)
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (152-211); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (254-263); C bundle (sort function inside `Kke._getPicks`, which ends with the Open Browser check and a plain label compare)

### QA-1.7 See a recently used section at the top
- Tag: both
- Evidence: source
- Boundary: no
- What: Commands you ran before appear first under a "recently used" heading, newest first, and they stay across restarts; the rest follow under an "other commands" heading.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (213-247, 304-311, 373-460, 470-486); C bundle (history class `tT` with storage keys `commandPalette.mru.cache` and `commandPalette.mru.counter`, strings 2227 and 2230)

### QA-1.8 Remove a command from the recently used list
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Rows in the recently used section have a close button ("Remove from Recently Used") that drops that command from history without running it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (284-337, 462-469); C bundle (searched for "Remove from Recently Used" and removeFromRecentlyUsed with zero hits; the bundle's history class has no remove method and its row builder adds no buttons)

### QA-1.9 Limit or turn off command history
- Tag: both
- Evidence: source
- Boundary: no
- What: Set how many recent commands are remembered (default 50), or set 0 so history is off and no recently used section shows.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.commandPalette.history
- Cursor: present
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (496-501); V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (373-420, 488-497); C bundle (setting block with default 50 and minimum 0, string 4200)

### QA-1.10 Clear Command History
- Tag: both
- Evidence: source
- Boundary: no
- What: Run "Clear Command History" from the palette, then confirm in a warning box ("Do you want to clear the history of recently used commands?", "This action is irreversible!", button "Clear") to empty the recently used list; it does nothing when history is off.
- Commands: workbench.action.clearCommandHistory
- Menus: CommandPalette (implicit, f1)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (297-328); V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (499-505); C bundle (class `OJk` ClearCommandHistoryAction, strings 8895 to 8897 and 8899)

### QA-1.11 Reopen the palette with your last typed text
- Tag: both
- Evidence: source
- Boundary: no
- What: With the setting on, opening the palette fills in the text from the last command you ran in this window, instead of starting empty.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: workbench.commandPalette.preserveInput
- Cursor: present
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (59-64, 94-104); V\src\vs\platform\quickinput\browser\quickAccess.ts (90-100, 228-233); V\src\vs\workbench\browser\workbench.contribution.ts (502-506); C bundle (`defaultFilterValue` getter in `aqe`, setting block, string 4201)

### QA-1.12 Configure a command's keybinding from its row
- Tag: both
- Evidence: source
- Boundary: no
- What: Each row has a gear button ("Configure Keybinding") that closes the palette and opens the Keyboard Shortcuts editor already filtered to that command and its when condition.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (127-141); V\src\vs\workbench\services\preferences\browser\keybindingsEditorModel.ts (37-40); C bundle (gear button and trigger inside `aqe.getCommandPicks`, string 8893)

### QA-1.13 List only commands that apply in the current context
- Tag: both
- Evidence: source
- Boundary: no
- What: Commands that are disabled or whose when condition is false are left out, so the list changes with the focused editor, open folder, debug state and so on, and the focused text editor adds its own actions to the list.
- Commands: editor.action.* (every action the active text editor supports)
- Menus: CommandPalette; contributes.menus.commandPalette (git, markdown-language-features, references-view and simple-browser manifests)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\actions\common\actions.ts (485-560, 751-754); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (224-266); V\src\vs\editor\contrib\quickAccess\browser\commandsQuickAccess.ts (35-60); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (697-707); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (113-120); V\extensions\git\package.json (contributes.menus.commandPalette, 175 rows, 70 set to false); C bundle (`aqe.getGlobalCommandPicks`, editor picks class `NJk`, extension point key `commandPalette`; the bundled git extension has 146 palette rows)

### QA-1.14 Tell apart commands that share a name
- Tag: both
- Evidence: source
- Boundary: no
- What: When two commands have the same label, both rows show their command id as a description so you can tell them apart.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (139-150); C bundle (duplicate-label map inside `Kke._getPicks`)

### QA-1.15 Hover a row to read the command description
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Hovering a row shows the command's description text when the command has one (few do).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (272-298); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (250-262); C bundle (read the `toCommandPick` code in `Kke`: the row it returns has no tooltip field, only ariaLabel, detail, keybinding and accept)

### QA-1.16 Similar commands from word similarity
- Tag: both
- Evidence: source
- Boundary: no
- What: Once you type three or more characters, commands that do not match by name but share words with their name, English name or description are added under a "similar commands" heading, up to five.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (53-54, 85-97, 122-137, 226-232, 340-351); C bundle (tf-idf calculation inside `Kke._getPicks`, strings 2228 and 2231)

### QA-1.17 Related commands from natural language search
- Tag: both
- Evidence: source
- Boundary: yes - the extra rows come from an extension-supplied related-information service, not from the palette.
- What: When such a service is installed and the setting is on, up to five more related commands are added under the same "similar commands" heading after a 200 ms pause in typing.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: workbench.commandPalette.experimental.enableNaturalLanguageSearch
- Cursor: present
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (51-52, 106-116, 143-156, 200-222); V\src\vs\workbench\browser\workbench.contribution.ts (530-535); C bundle (`aqe.getRelatedInformationPicks` and `useAiRelatedInfo`, setting block, string 4206)

### QA-1.18 Commonly used section (experimental)
- Tag: both
- Evidence: source
- Boundary: no
- What: With the setting on and a list of suggested command ids supplied by the product file, a "commonly used" heading lists those commands after recently used ones.
- Commands: workbench.action.showCommands
- Menus: none found
- Keys: none found
- Settings: workbench.commandPalette.experimental.suggestCommands
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (180-196, 213-240); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (108-116); V\src\vs\workbench\browser\workbench.contribution.ts (507-511); V\src\vs\base\common\product.ts; C bundle (`suggestedCommandIds` option set in `aqe.updateOptions`, setting block, string 4202)

### QA-1.19 Ask in Chat row
- Tag: vscode
- Evidence: source
- Boundary: yes - the row sends your typed text to chat, whose UI is out of scope.
- What: With text typed, a row "Ask in Chat: <text>" under a separator opens the Chat view or Quick Chat with that text, and its gear button ("Configure visibility") opens the setting.
- Commands: workbench.action.chat.open; workbench.action.quickchat.toggle
- Menus: none found
- Keys: none found
- Settings: workbench.commandPalette.showAskInChat; workbench.commandPalette.experimental.askChatLocation
- Cursor: missing
- Source: V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (158-196); V\src\vs\workbench\browser\workbench.contribution.ts (512-529); V\src\vs\workbench\contrib\chat\browser\actions\chatActions.ts (81); V\src\vs\workbench\contrib\chat\browser\actions\chatQuickInputActions.ts (17); C bundle (searched "Ask in Chat" and showAskInChat with zero hits; `askChatLocation` is declared in the settings block but nothing reads it)

### QA-1.20 Hide Chat and Copilot commands from the list
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor removes any command whose English name starts with "Chat:" or "Ask Github Copilot", so the stock chat commands never show in the palette.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (array `uPk` and the filter on the alias at the top of `Kke._getPicks`)

### QA-1.21 Pin Open Browser above other unused commands
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A command named "Open Browser" is sorted ahead of other commands you have not run before, so it sits at the top of the "other commands" part.
- Commands: composer.openBrowserTab
- Menus: CommandPalette (implicit, f1)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (array `gDr` and the sort step in `Kke._getPicks`; command class `a5m` with title "Open Browser" and id `composer.openBrowserTab`)

## Handed off

- Error box "Command '{0}' resulted in an error" shown when a run fails (`dialogService.error`, string 2233 in Cursor): DL-1.
- Keyboard Shortcuts editor that the gear button opens: ED-8.
- Empty editor watermark rows "Show All Commands" and "Go to File", and the Welcome page "Open Command Palette" buttons: ED-9 (ED-9.18 already lists the watermark).
- The Manage menu row "Command Palette...": AB-4 (AB-4.14 lists it; kept here as an entry point of QA-1.1 only).
- Palette entries "Developer: Report Cursor Tab Action" and "Developer: Report AI Action" and the quick picks with the preview box they open: QA-6 for the box, AC-8 for Cursor Tab.
- Palette entries "Cursor Settings" and "Open VS Code Settings" (id `workbench.action.openGlobalSettingsCursorAlias`, shown as an alias of Open User Settings): ED-8.
- Cursor's one-time notification "Want to launch Cursor from the command line?" that links to the Command Palette key: NT-2.
- Quick access keys inside the terminal (`terminal.integrated.commandsToSkipShell` lists `workbench.action.showCommands` and the quick open ids): PN-3.

## Unclear

- Whether Cursor has any provider for the related-information service. The "similar commands" part from QA-1.17 and the "Ask in Chat" row in stock both need `aiRelatedInformationService.isEnabled()` to be true, because `hasAdditionalCommandPicks` returns false otherwise. I did not trace what registers that service.
- Pass 1 said "similar commands" needs an extension. Reading the code shows the word-similarity part (QA-1.16) is always on and only the related-information part (QA-1.17) needs a service. Pass 1 also said the "Ask in Chat" row is on by default, but it only appears when the related-information service is enabled.
- The "commonly used" list (QA-1.18) reads `commandPaletteSuggestedCommandIds` from the product file. That key is in neither the V `product.json` nor the Cursor `product.json`, so as shipped here the section would show nothing. A release build could supply it.
- The editor right-click row "Command Palette..." in Cursor has an extra condition (the isGlass context key is not true), so it is hidden in the Agents window. I read this as Agents window only and did not catalog it.
- Nothing newer than 1.128 was found for this group. A diff of the four palette source files between V and N shows only one telemetry field.
- Nothing was run on screen. Everything here comes from reading source and bundle fragments.
