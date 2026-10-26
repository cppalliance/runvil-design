Group: AC-5 Terminal suggestions. Pass: 3. Date: 2026-10-25.

### AC-5.1
1. Setting `terminal.integrated.suggest.enabled` turns the terminal suggest popup on or off.
2. The popup needs shell integration turned on and working (`terminal.integrated.shellIntegration.enabled`), and stays closed without it.
3. The popup only works in supported shells: PowerShell 7+, zsh, bash and fish (stock VS Code also lists Windows PowerShell).
4. The setting is on by default, so the popup works without setup. [stock only]
5. The setting is off by default and tagged preview, so it has to be turned on before the popup shows. [cursor only]

### AC-5.2
1. A list opens next to the prompt cursor, below it, or above it when there is no room below.
2. Near the right edge of the terminal the list moves left so it stays in view.
3. The list is 430 px wide and up to 12 rows tall by default.
4. Each row shows a kind icon, the label with the typed letters in bold, a faded detail text, and a description on the right side.
5. The list also shows in terminals that live in the editor area, not only in the panel.
6. When opened by hand and nothing matches, the list shows the message "No suggestions."
7. When opened by hand, the list shows the message "Loading..." until results arrive. [stock only]
8. Settings `terminal.integrated.fontFamily`, `terminal.integrated.fontSize`, `terminal.integrated.lineHeight` and `editor.fontWeight` set the font used in the rows.
9. In Cursor the first row is always focused when the list shows. [cursor only]

### AC-5.3
1. Dragging the edges of the popup resizes it.
2. The size is remembered across sessions, so the next popup opens at the size you left it.
3. The Command Palette entry "Terminal: Reset Suggest Widget Size" (`workbench.action.terminal.resetSuggestWidgetSize`) puts the popup back to its default size.

### AC-5.4
1. Each row has a colored kind icon for the common kinds: file, folder, method (commands), alias, argument, option, option value, flag, and the inline suggestion star.
2. File and folder rows switch to icons from the file icon theme when the active theme has them.
3. Setting `editor.suggest.showIcons` set to false hides the icons on the rows.
4. Each icon has its own theme color named `terminalSymbolIcon.*Foreground`, so a theme can recolor it.
5. Symbolic link file and symbolic link folder rows have their own icons. [stock only]
6. Commit, branch, tag, stash, remote, pull request and completed pull request rows have their own icons. [stock only]
7. Plain text rows get a key icon. [stock only]

### AC-5.5
1. Pressing Ctrl+Space in the terminal opens the suggest popup on demand when `terminal.integrated.suggest.enabled` is on and the popup is not already showing.
2. A manual open focuses the first row.
3. A manual open can fall back to files and folders when nothing else fits.
4. When a manual open finds nothing, the popup shows "No suggestions."
5. Pressing Ctrl+Space again while the popup is open toggles the details pane.
6. In Cursor the command is `workbench.action.terminal.requestCompletions` with the title "Request Completions", bound to the same Ctrl+Space key and condition, in place of `workbench.action.terminal.triggerSuggest`. [cursor only]

### AC-5.6
1. With the `commands` key of `terminal.integrated.suggest.quickSuggestions` set to `on`, typing the first word of a command line opens the popup by itself with commands.
2. With the `arguments` key set to `on`, typing any later word opens the popup by itself with arguments.
3. With the `unknown` key set to `on`, typing where it is unclear what fits opens the popup by itself, with files and folders as a fallback.
4. With `terminal.integrated.suggest.suggestOnTriggerCharacters` on, typing a trigger character opens the popup: a dash after a space, a slash after a folder, or any character a provider asks for.
5. By default none of these open the popup by itself, so typing never shows it until a setting is turned on. [stock only]
6. By default `commands` and `arguments` are `on`, `unknown` is `off` and `suggestOnTriggerCharacters` is true. [cursor only]

### AC-5.7
1. Pressing Down moves to the next row and wraps from the last row to the first.
2. Pressing Up moves to the previous row and wraps from the first row to the last once you have pressed Down.
3. PageDown and PageUp move the focus by one page of rows.
4. Pressing Up on the first row before you have pressed Down closes the popup and sends Up to the shell, so you walk through command history.
5. Setting `terminal.integrated.suggest.upArrowNavigatesHistory` set to false makes Up on the first row move to the last row instead and keeps the popup open, and its default is true.
6. While the popup is closed, Down, Up, PageDown and PageUp go to the shell as usual.

### AC-5.8
1. Pressing Tab inserts the focused row into the command line in place of the part of the word you typed, then closes the popup.
2. The rest of the word to the right of the cursor is replaced too.
3. Clicking a row inserts it the same way and keeps keyboard focus in the terminal.
4. When the popup is open with no row focused, Tab goes to the shell instead of accepting. [stock only]
5. In Cursor the accept command is titled "Insert", and an "Insert (Tab)" button at the left of the popup's status row accepts the focused row. [cursor only]

### AC-5.9
1. Pressing Enter with a row focused inserts that row into the command line.
2. With selection mode `partial`, Enter does not accept the first row until you press Down or open the popup by hand, and until then Enter goes to the shell. [stock only]
3. Setting `terminal.integrated.suggest.runOnEnter` chooses whether Enter also runs the command after inserting, with values `never`, `exactMatch` (when the row is typed in full), `exactMatchIgnoreExtension` (also when a file is typed without its extension) and `always`.
4. With `always`, Enter inserts the row and also runs the command, and with `never` Enter only inserts it.
5. The setting has an extra value `ignore`, which is the default, and with it Enter goes straight to the shell without accepting a row. [cursor only]

### AC-5.10
1. Setting `terminal.integrated.suggest.selectionMode` chooses how an auto-opened popup selects a row, with values `partial`, `always` and `never`, and the default is `partial`. [stock only]
2. In `partial` mode the first row is only outlined, Tab accepts it, and Enter accepts it only after you press Down. [stock only]
3. In `always` mode the first row is filled in, and Tab or Enter accepts it. [stock only]
4. In `never` mode no row is selected until you press Down. [stock only]
5. A manual open always selects the first row, whatever the mode. [stock only]
6. The status row shows one button named for the current mode ("Selection Mode: None", "Partial (Tab)" or "Always (Tab or Enter)"), and clicking it switches to the next mode in the order none, partial, always. [stock only]
7. The selection mode button only shows when `terminal.integrated.suggest.quickSuggestions` or `terminal.integrated.suggest.suggestOnTriggerCharacters` is set to the plain value `true`. [stock only]

### AC-5.11
1. With `terminal.integrated.suggest.insertTrailingSpace` on, accepting a suggestion also types a space after it, and the setting is off by default. [stock only]
2. After the space is typed, the popup opens again for the next word. [stock only]
3. Folder and symbolic link folder rows never get the trailing space. [stock only]

### AC-5.12
1. Pressing Esc closes the popup.
2. The popup closes on its own when the terminal scrolls.
3. The popup closes on its own when you click in the terminal or focus leaves it, except when focus moves into the details pane.
4. The popup closes on its own when you type a space.
5. The popup closes on its own when the typed text no longer matches anything.
6. The popup closes on its own when the command line is run.

### AC-5.13
1. Pressing Ctrl+Space or Ctrl+I (Cmd+I on a Mac) while the popup has a focused row opens a side pane with that row's detail text and docs rendered as markdown.
2. Pressing the same key again closes the pane.
3. The pane shows a Close button, and shows "Loading..." while its docs load.
4. The pane only opens for rows that have docs or a detail text that differs from the label.
5. The pane sits to the east of the list, or south or north when there is no room, and never to the west. [stock only]
6. In Cursor the pane has no placement limit, so it can also open to the west of the list. [cursor only]
7. Whether the pane is open is remembered, so the next popup opens with the pane open or closed as you left it.
8. Pressing Ctrl+Alt+Space moves focus into the pane, where you can scroll and select or copy its text.

### AC-5.14
1. Pressing Ctrl+/ (Cmd+/ on a Mac) while the popup is showing turns explain mode on.
2. In explain mode the details pane shows the focused row's score, prefix, replacement range, index and provider name.
3. Pressing Ctrl+/ again turns explain mode off.

### AC-5.15
1. A thin row of small buttons shows under the list, split into a left group and a right group.
2. Setting `terminal.integrated.suggest.showStatusBar` shows or hides this row, and its default is true.
3. The row holds the selection mode switch on the left, and the show-while-typing switch, Learn More and Configure on the right. [stock only]
4. In Cursor the buttons are text buttons drawn as "Label (key)", with only two entries: Insert on the left and Configure on the right. [cursor only]

### AC-5.16
1. An eye-closed icon button in the right group of the status row, shown while typing suggestions are off, turns them on when clicked by setting commands, arguments and trigger characters on. [stock only]
2. An eye icon button in the same place, shown while typing suggestions are on, turns them off when clicked. [stock only]
3. The button's tooltip is a full sentence saying what clicking it will change. [stock only]
4. The button is the first one in the right group of the status row. [stock only]

### AC-5.17
1. A question mark button on the status row opens the terminal IntelliSense docs page, and it is the second button in the right group. [stock only]
2. A gear button on the status row opens Settings filtered to `terminal.integrated.suggest`, and in stock it is the third button in the right group.
3. In Cursor the Configure button is a text button titled "Configure" and is the first item in the right group. [cursor only]
4. In Cursor, pressing Ctrl+Shift+, while the popup is showing opens the same filtered Settings view. [cursor only]

### AC-5.18
1. For the first few uses, the first button in the right group of the status row is drawn in the primary button colors. [stock only]
2. The highlight ends for good after the popup has shown 10 times, or 10 seconds after its first show, whichever comes first. [stock only]
3. The count and the finished state are stored for the whole app, so they are shared by all windows and terminals and survive a restart. [stock only]
4. The highlight only shows while the status row is visible. [stock only]

### AC-5.19
1. Typing the first word of a command line lists commands you can run: shell built-ins, functions and programs on PATH show as rows of kind "Method".
2. Shell aliases show as rows of kind "Alias" with the alias icon.
3. On Windows, setting `terminal.integrated.suggest.windowsExecutableExtensions` chooses which executable file types are listed: set an extension to false to drop it, or to true to add one that is not in the default list.
4. Setting the command provider to false in `terminal.integrated.suggest.providers` removes the command and alias rows from the popup.

### AC-5.20
1. After a known command, the popup lists that command's subcommands and arguments, options, option values and flags.
2. Each kind has its own label and icon: "Argument", "Option", "OptionValue" and "Flag".
3. The details pane shows a description for the focused row.
4. With the `arguments` key of `terminal.integrated.suggest.quickSuggestions` set to `on`, typing a later word opens these suggestions by itself.
5. Setting the command provider to false in `terminal.integrated.suggest.providers` removes these rows from the popup.

### AC-5.21
1. When a path is expected, the popup lists files and folders from the working directory or from the path you have typed.
2. Symbolic links show as symbolic link file and symbolic link folder rows. [stock only]
3. The list also includes `../` and `~` entries.
4. With the `cd` command, setting `terminal.integrated.suggest.cdPath` lists the children of the folders in `$CDPATH`, with values `off`, `relative` and `absolute`, and the default is `absolute`.
5. Typing a path opens the file and folder suggestions by itself when the matching key of `terminal.integrated.suggest.quickSuggestions` is `on`.

### AC-5.22
1. After `git`, the popup lists git subcommands, options and branch names for commands that take a branch, drawn with a git icon.
2. Branches, tags, commits, stashes and remotes show as their own row kinds, each with its own git icon and color. [stock only]
3. After a GitHub CLI command, pull requests show as "pull request" rows, and merged ones as "completed pull request" rows. [stock only]
4. Setting the command provider to false in `terminal.integrated.suggest.providers` removes the git rows from the popup.
5. In Cursor, git rows are all drawn with one generic git icon from the command spec, since the per-kind icons and colors do not exist there. [cursor only]

### AC-5.23
1. When the shell already shows its own gray inline suggestion, the popup adds a row labelled "Inline Suggestion" with a star icon.
2. The row appears for shells that draw their own gray text: zsh, fish, bash, PowerShell and Git Bash.
3. Setting `terminal.integrated.suggest.inlineSuggestion` chooses how the row is placed, with values `alwaysOnTop` (the default, row always first), `alwaysOnTopExceptExactMatch` (an exact match sorts above it) and `off` (no row).
4. Pressing the Right arrow key accepts the inline suggestion.

### AC-5.24
1. Setting `terminal.integrated.suggest.providers` lists every suggestion provider by id with a true or false value.
2. Setting a provider's id to false removes the rows it supplies from the popup.
3. Providers that are not mentioned stay on.
4. The language server provider shows only in Python terminals, and it is off by default while the other providers are on. [stock only]
5. In Cursor the default lists only `terminal-suggest` and `pwsh-shell-integration`, both true, with no language server entry. [cursor only]

### AC-5.25
1. Running "Terminal: Clear Suggest Cached Globals" (`terminal.integrated.suggest.clearCachedGlobals`) from the Command Palette throws away the saved list of shell built-ins and aliases. [stock only]
2. The next popup reads the shell's built-ins and aliases fresh, so newly defined aliases and functions show up. [stock only]

## Unclear
- AC-5.2 piece 7 is tagged stock only because no loading trigger was found in the Cursor bundle, although the "Loading..." string is there, so this tag is tentative.
- AC-5.4 pieces 5 to 7 and AC-5.22 piece 2 name the same stock-only git and link icons from two sides, so a counting tool may see overlap between the icon list and the git rows.
- AC-5.6 pieces 5 and 6 and AC-5.1 pieces 4 and 5 give the default values from the pass 2 text, and none of them was run in a live window.
- AC-5.10 piece 7 follows the when clause as written in stock, where the setting is an object by default, so the button may never show unless the legacy plain `true` is set; this was not run.
- AC-5.18 has no piece for `workbench.action.terminal.resetDiscoverability`, because pass 2 found it declared but not registered in V, so no visible behavior is grounded.
- AC-5.5 has no piece for the PowerShell Ctrl+Space sequence sent to the shell, because pass 2 handed it to PN-3.
- AC-5.9 piece 1 and the Cursor piece 5 interact: with the Cursor default `ignore`, Enter does not accept a row at all, so piece 1 only applies once `runOnEnter` is set to another value there.
