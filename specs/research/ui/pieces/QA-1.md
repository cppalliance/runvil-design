Group: QA-1 Command Palette. Pass: 3. Date: 2026-10-25.

### QA-1.1
1. Pressing `F1` or `Ctrl+Shift+P` (`Cmd+Shift+P` on macOS) opens the Command Palette with a list of commands (`workbench.action.showCommands`).
2. The palette input shows the placeholder "Type the name of a command to run."
3. Choosing a command row with `Enter` or a click runs that command and closes the palette.
4. Choosing View, then "Command Palette..." from the menu bar opens the palette, and so does Help, then "Show All Commands".
5. Choosing "Command Palette..." from the Manage gear menu opens the palette.
6. Right-clicking in a text editor and choosing "Command Palette..." opens the palette, and the entry is hidden in simple input editors.

### QA-1.2
1. Typing narrows the list to commands whose name contains your text or matches your words, with the matching letters highlighted.
2. Each row shows its command name as "Category: Title".
3. Typing a command's full id lists that command even when its name does not match.
4. When nothing fits, the list shows a "No matching commands" row.

### QA-1.3
1. Typing `>` at the start of the Go to File box (`workbench.action.quickOpen`) turns the list into the Command Palette.
2. Deleting the `>` turns the box back into the Go to File list.
3. Running quick access with the argument `>` from a command or keybinding opens the command list straight away.

### QA-1.4
1. Each command row shows the key combination bound to that command at its right edge.
2. A command with no key bound shows no key combination.
3. A screen reader reads a row with a key as "name, keys".

### QA-1.5
1. When the display language is not English, each row shows the English command name on a second line.
2. Typing the English name still finds the command, with the match highlighted on the second line.
3. When the display language is English, rows show no second line.

### QA-1.6
1. Commands you ran before come first in the results, newest first.
2. When the experimental suggested list is on, suggested commands come next, in the order given by the list.
3. All other commands follow, sorted A to Z by name.
4. Commands in the "Developer" category are pushed below the other commands that were not run before. [stock only]
5. Cursor sorts the commands that were not run before by name only, so Developer commands sit among the rest. [cursor only]

### QA-1.7
1. Commands you ran before appear first under a "recently used" heading, newest first.
2. The recently used list is kept when you close and reopen the app.
3. The remaining commands follow under an "other commands" heading.
4. The "recently used" heading shows only when the first matching row is a command you ran before.

### QA-1.8
1. Rows in the recently used section have a close button at the right with the tooltip "Remove from Recently Used". [stock only]
2. Clicking the close button removes that command from the history and drops its row from the list without running it. [stock only]
3. The close button sits to the right of other row buttons such as the Configure Keybinding gear. [stock only]

### QA-1.9
1. Setting `workbench.commandPalette.history` sets how many recently used commands are remembered, with a default of 50 and a minimum of 0.
2. A value of 0 turns the history off, so commands you run are not remembered and no "recently used" heading shows.
3. Lowering the number trims the oldest commands from the remembered list.

### QA-1.10
1. Running "Clear Command History" (`workbench.action.clearCommandHistory`) from the Command Palette opens a warning box.
2. The box asks "Do you want to clear the history of recently used commands?", says "This action is irreversible!" and has a Clear button.
3. Clicking Clear empties the recently used list, so the "recently used" heading is gone the next time.
4. Cancelling the box leaves the history unchanged.
5. Running the command does nothing when the history is off (`workbench.commandPalette.history` set to 0).

### QA-1.11
1. With setting `workbench.commandPalette.preserveInput` on, opening the palette fills in the text that was typed when you last ran a command in this window.
2. With the setting off, the default, the palette opens with an empty input.
3. The restored text is remembered only while the app is running, so after a restart the palette opens empty again.

### QA-1.12
1. Each command row has a gear button with the tooltip "Configure Keybinding".
2. Clicking the gear button closes the palette without running the command.
3. After the palette closes, the Keyboard Shortcuts editor opens with its search filled in for that command and its when condition.

### QA-1.13
1. Commands that are disabled, or whose when condition is false, are left out of the list.
2. The list changes with context such as the focused editor, the open folder and the debug state.
3. When a text editor has focus, the actions that editor supports (`editor.action.*`) are added to the list.
4. Commands that the git, markdown-language-features, references-view and simple-browser extensions mark as hidden from the palette never show.
5. The first time the palette opens, it waits up to 800 ms for extensions to register so their commands are included.

### QA-1.14
1. When two or more commands share the same label, each of those rows shows its command id as a description next to the label.
2. A command with a unique label shows no description.
3. A command shows the id only while a command with the same label is also in the filtered list, so typing more text can remove the id.

### QA-1.15
1. Hovering a command row shows a tooltip with the command's description text when the command has one. [stock only]
2. A command without a description shows no tooltip. [stock only]

### QA-1.16
1. After you type 3 or more characters, commands that do not match by name but share words with their name, English name or description are added to the list.
2. Those commands appear under a "similar commands" heading below the name matches.
3. At most 5 similar commands show, with the closest match first.
4. Typing fewer than 3 characters adds no similar commands.

### QA-1.17
1. When a related-information service is installed and enabled, up to 5 more related commands are added under the "similar commands" heading.
2. The related commands are added only after a 200 ms pause in typing.
3. Related commands are ordered by how closely they relate, and a command already in the list is not repeated.
4. Setting `workbench.commandPalette.experimental.enableNaturalLanguageSearch`, on by default, turns the related commands on or off.
5. With no text typed, no related commands are added.

### QA-1.18
1. With setting `workbench.commandPalette.experimental.suggestCommands` on and a list of suggested command ids supplied by the product file, a "commonly used" heading lists those commands.
2. The "commonly used" section comes after the recently used commands and before the "other commands" heading.
3. The setting is off by default.
4. With the setting on but no list supplied, no "commonly used" heading shows.

### QA-1.19
1. With text typed, the last row reads "Ask in Chat: <text>" and sits under a separator below the other results. [stock only]
2. Choosing the row opens the Chat view with the typed text (`workbench.action.chat.open`). [stock only]
3. Setting `workbench.commandPalette.experimental.askChatLocation` chooses where the row opens chat, with values `chatView` (the default) and `quickChat` (`workbench.action.quickchat.toggle`). [stock only]
4. The row has a gear button with the tooltip "Configure visibility" that closes the palette and opens the settings editor on `workbench.commandPalette.showAskInChat`. [stock only]
5. Setting `workbench.commandPalette.showAskInChat`, on by default, shows or hides the row. [stock only]
6. The row appears only when the related-information service is enabled and `enableNaturalLanguageSearch` is on, and never when nothing is typed. [stock only]

### QA-1.20
1. In Cursor, commands whose English name starts with "Chat:" never show in the palette. [cursor only]
2. Commands whose English name starts with "Ask Github Copilot" never show either.
3. The rule uses the English name, so the commands stay hidden in any display language.

### QA-1.21
1. The command "Open Browser" (`composer.openBrowserTab`) sorts ahead of the other commands you have not run before. [cursor only]
2. "Open Browser" sits at the top of the "other commands" part of the list.
3. Commands you ran before, and suggested commands, still come before it.

## Unclear
- QA-1.17 and QA-1.19 only show anything when a related-information service is enabled. Whether Cursor has any provider for that service was not traced, so these pieces may never be visible in Cursor.
- QA-1.19 piece 6 adds a condition that the feature block does not list. The source shows both the natural language setting and an enabled service are needed before the Ask in Chat row can appear.
- The suggested list in QA-1.18 reads `commandPaletteSuggestedCommandIds` from the product file. That key is in neither the stock nor the Cursor product file checked, so as shipped the section would show nothing.
- QA-1.6 piece 5 and the Cursor field both say Cursor has no Developer-last rule because the bundle's rows have no category field. That was read from the bundle and not seen on screen.
- QA-1.11 pieces follow the source, which stores the text in the input box at the moment a command is accepted. The setting text and the feature block word this as the last typed input and the last command you ran.
- QA-1.15 has only two pieces because it is one small behavior.
- Nothing was run on screen. All pieces come from source and bundle strings.
