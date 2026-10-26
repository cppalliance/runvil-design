Group: PN-3 Terminal tooling. Pass: 3. Date: 2026-10-25.

### PN-3.1
1. Pressing `Ctrl+F` (`Cmd+F` on macOS) with a terminal focused opens a find box over the terminal, filled with the selected text when the selection is on one line.
2. Typing in the find box highlights the matches in the visible output and the scrollback, and the box shows a match count next to previous match, next match and close buttons.
3. `Enter` in the find input steps to the next match and `Shift+Enter` to the previous one, and `F3` and `Shift+F3` (`Cmd+G` and `Cmd+Shift+G` on macOS) step the same way through `workbench.action.terminal.findNext` and `workbench.action.terminal.findPrevious`.
4. The match case, whole word and regular expression toggles in the find box change which text matches, and `Alt+C`, `Alt+W` and `Alt+R` (`Cmd+Alt+C`, `Cmd+Alt+W` and `Cmd+Alt+R` on macOS) flip them from the keyboard.
5. Pressing `Esc` or `Shift+Esc` while the find box is visible closes it.
6. Dragging the left edge of the find box resizes it from its starting width.

### PN-3.2
1. With text selected in a terminal, pressing `Ctrl+Shift+F` (`Cmd+Shift+F` on macOS) opens the Search view with the selected text already in the search box.
2. The `workbench.action.terminal.searchWorkspace` shortcut only applies while the terminal has focus, a process is supported and text is selected.

### PN-3.3
1. Hovering a file path, folder, URL or plain word in terminal output underlines it as a link.
2. Following a file path link with `Ctrl+click` (`Cmd+click` on macOS) opens that file in the editor, at the line and column when the path has them.
3. Following a folder link focuses it in the Explorer when it is inside the workspace, and opens it in a new window when it is outside.
4. Following a URL link opens the URL, and following a plain word link runs a workspace search for that word.
5. A path that wraps across several terminal lines is still found as one link.
6. Following a URL whose scheme is not on the allowed list first shows a warning notification that asks whether to allow opening links with that scheme, with an `Allow <scheme>` button.
7. Setting `editor.multiCursorModifier` changes the click modifier, so the link click becomes `Alt+click` when the multi-cursor modifier is Ctrl or Cmd.
8. Setting `terminal.integrated.enableFileLinks` chooses whether file and folder links are detected, with values `off`, `on` and `notRemote`.
9. Setting `terminal.integrated.allowedLinkSchemes` lists the URL schemes that open without the prompt, by default `file`, `http`, `https`, `mailto`, `vscode` and `vscode-insiders`.

### PN-3.4
1. Pointing at a detected link shows a hover that names the action and the click modifier, such as `Follow link (ctrl + click)`.
2. The modifier named in the hover follows the platform and `editor.multiCursorModifier`, reading `ctrl + click`, `cmd + click`, `alt + click` or `option + click`.
3. The hover wording changes with the link type, reading `Open file in editor`, `Focus folder in explorer`, `Open folder in new window` or `Search workspace`.
4. A link that goes through a forwarded port shows `Follow link using forwarded port` in its hover.
5. Setting `terminal.integrated.showLinkHover` turns the link hover on or off.

### PN-3.5
1. Pressing `Ctrl+Shift+O` (`Cmd+Shift+O` on macOS) with a terminal focused opens a quick pick with the placeholder `Select the link to open, type to filter all links`.
2. The quick pick groups links under separators named Url, File, Folder and Workspace Search.
3. The list starts with links from the visible area, and typing a filter searches links from the whole buffer.
4. Choosing a row opens that link the same way a click would.
5. `Open Last URL Link` and `Open Last Local File Link` in the Command Palette open the newest URL or file link in the terminal without showing a pick.
6. Pressing `Ctrl+Shift+G` (`Cmd+Shift+G` on macOS) in the accessible view that shows the terminal opens the same quick pick.

### PN-3.6
1. With shell integration active, each command gets a dot in the terminal's left gutter that shows whether it succeeded or failed.
2. Each command also gets a mark in the overview ruler on the right of the terminal.
3. A command that has not finished or has no known exit code shows a neutral default dot instead of a success or failure one.
4. Hovering a dot shows when the command ran, how long it took, and its exit code on failure, for example `Command executed 5 seconds ago, took 2 seconds and failed (Exit Code 1)`.
5. Right-clicking a dot shows a `Toggle Visibility` entry that opens a pick with two checkboxes, one for gutter decorations and one for overview ruler decorations.
6. Setting `terminal.integrated.shellIntegration.decorationsEnabled` chooses where decorations show, with values `both`, `gutter`, `overviewRuler` and `never`.

### PN-3.7
1. Clicking a command's dot opens a menu of actions for that command.
2. Choosing `Rerun Command` runs the command again, and when shell integration did not mark the command as trusted it first asks `Do you want to run the command`.
3. `Copy Command`, `Copy Command and Output` and `Copy Output` copy those parts of that command to the clipboard, and `Copy Output as HTML` copies the output with its colors and styles.
4. The menu also has `Run Recent Command` and `Go To Recent Directory`, which open their quick picks, and `Learn About Shell Integration`, which opens help about it.
5. `Copy Last Command`, `Copy Last Command Output` and `Copy Last Command and Output` in the Command Palette copy the text, the output or both for the newest command.

### PN-3.8
1. Pressing `Ctrl+Up` (`Cmd+Up` on macOS) scrolls the terminal to the previous command, and `Ctrl+Down` scrolls to the next, when the terminal has focus and screen reader mode is off.
2. The command that was jumped to is briefly highlighted with an outline so it is easy to spot.
3. `Scroll To Previous Command` and `Scroll To Next Command` can be shown as up and down arrow buttons on the terminal view toolbar, where they are hidden by default.
4. The same two arrow buttons can be shown on the toolbar of a terminal editor tab, where they are hidden by default.

### PN-3.9
1. Pressing `Ctrl+Shift+Up` (`Cmd+Shift+Up` on macOS) extends the selection back to the previous command mark, and `Ctrl+Shift+Down` extends it forward to the next one.
2. Pressing the same key again keeps extending the selection across more commands and their output.
3. `Select To Previous Line` and `Select To Next Line` in the Command Palette extend the selection by one line, and they have no default key.

### PN-3.10
1. With shell integration active, hovering over the text of a command draws a thin vertical guide bar along that command's prompt and output.
2. Moving the pointer off the command removes the guide bar.
3. Setting `terminal.integrated.shellIntegration.showCommandGuide` turns the command guide on or off, on by default.

### PN-3.11
1. With shell integration active, scrolling up through a long command's output keeps that command's prompt line pinned at the top of the terminal.
2. Hovering the pinned line shows `Navigate to Command` with the keys for scrolling to the previous and next command.
3. Clicking the pinned line jumps the terminal back to that command.
4. Right-clicking the pinned line shows a menu with `Toggle Sticky Scroll`, and the same command in the Command Palette turns sticky scroll off and on.
5. Setting `terminal.integrated.stickyScroll.enabled` turns sticky scroll on or off, on by default.
6. Setting `terminal.integrated.stickyScroll.maxLineCount` limits how many lines are pinned, from 1 to 10 with a default of 5, and the pinned area never takes more than 40 percent of the terminal height.
7. Setting `terminal.integrated.stickyScroll.ignoredCommands` lists commands that do not pin their prompt, by default `clear`, `cls`, `clear-host` and several coding agent command names.

### PN-3.12
1. Pressing `Ctrl+Alt+R` (`Ctrl+Option+R` on macOS), or `Ctrl+R` when screen reader mode is on, opens a quick pick of recent commands with the placeholder `Select a command to run (hold Alt-key to edit the command)`.
2. The rows sit under separators for the current session, the previous session and the shell's own history file.
3. Moving through the rows scrolls the terminal to that command when it is in the current session.
4. Pressing `Enter` on a row runs that command, and holding `Alt` (`Option` on macOS) while choosing it types the command at the prompt without running it.
5. Each row has a `View Command Output` button that opens its output in an editor and a `Remove from Command History` button, and the shell history group has an `Open File` button.
6. A row can be pinned, and pinned commands are remembered for that shell type.
7. A fuzzy search toggle in the input switches the typed filter between contiguous and fuzzy matching.
8. `Clear Previous Session History` in the Command Palette wipes the stored command history, so older commands no longer appear in the pick.
9. `Run Recent Command` can be shown as a button on the terminal view toolbar or a terminal editor toolbar, where it is hidden by default.
10. Setting `terminal.integrated.shellIntegration.history` sets how many recent commands are kept, with a default of 100 and 0 turning the history off.

### PN-3.13
1. Pressing `Ctrl+G` (`Cmd+G` on macOS) with a terminal focused opens a quick pick of recent folders with the placeholder `Select a directory to go to (hold Alt-key to edit the command)`.
2. The folders sit under separators for the current session and the previous session.
3. Choosing a folder runs a `cd` command for it in the terminal.
4. Holding `Alt` (`Option` on macOS) while choosing a folder types the `cd` command at the prompt without running it.
5. The pick has the same remove button on each row and the same fuzzy search toggle as `Run Recent Command`.
6. `Go to Recent Directory` can be shown as a button on the terminal view toolbar or a terminal editor toolbar, where it is hidden by default.
7. Setting `terminal.integrated.shellIntegration.history` also decides how many recent entries are kept for this list, and 0 turns the history off.

### PN-3.14
1. After a command finishes with a result that has a known fix, a lightbulb or sparkle icon appears to the left of the prompt.
2. Clicking the icon or pressing `Ctrl+.` (`Cmd+.` on macOS) lists the available fixes.
3. Each fix is labeled `Run: <command>` when it runs a command or `Open: <link>` when it opens a link.
4. Git fixes cover a mistyped subcommand, a fast-forward pull, push with set-upstream, creating a pull request, and retrying with two dashes.
5. Other fixes cover freeing a busy port, PowerShell's most similar commands and command-not-found suggestions, and the npm `Did you mean` fix.
6. The same fixes are also listed in the command's dot menu.
7. Setting `terminal.integrated.shellIntegration.quickFixEnabled` turns the quick fix icon on or off, on by default, and this setting is not found in Cursor. [stock only]

### PN-3.15
1. Pressing `Ctrl+Shift+C` (`Cmd+C` on macOS) with text selected and the terminal focused copies the selection to the clipboard.
2. On Windows, `Ctrl+C` copies the selection and clears it when text is selected, and otherwise goes to the shell.
3. `Copy as HTML` copies the selection with its colors and styles.
4. The terminal right-click menu has `Copy` and `Copy as HTML` entries, in the panel and in a terminal editor tab.
5. Setting `terminal.integrated.copyOnSelection` copies text to the clipboard as soon as it is selected, off by default.

### PN-3.16
1. Pressing `Ctrl+V` (`Cmd+V` on macOS) with a terminal focused pastes the clipboard text at the prompt, and the right-click menu has a `Paste` entry.
2. `Ctrl+Shift+V` also pastes on Windows and is the paste key on Linux.
3. On Linux, `Shift+Insert` pastes the selection clipboard through `workbench.action.terminal.pasteSelection`.
4. Middle-clicking the terminal pastes when `terminal.integrated.middleClickBehavior` is `paste`.
5. When the clipboard holds a copied file and no text, pasting inserts the file's path.
6. On Windows in PowerShell with screen reader mode off, `Ctrl+V` is sent straight to the shell so PSReadLine handles the paste.
7. In PowerShell on Windows, `Ctrl+V` with a copied file and no text on the clipboard still pastes the file's path through `workbench.action.terminal.pastePwsh`. [stock only]

### PN-3.17
1. Pasting text with more than one line opens a dialog that asks whether to paste that many lines and previews the first three.
2. The dialog offers `Paste`, `Paste as one line` and `Cancel`.
3. A `Do not ask me again` checkbox in the dialog sets the setting to `never`.
4. In the default `auto` mode the dialog is skipped when the shell is in bracketed paste mode, and a single command with a trailing newline is pasted without the newline.
5. Setting `terminal.integrated.enableMultiLinePasteWarning` chooses when the dialog shows, with values `auto`, `always` and `never`.
6. Setting `terminal.integrated.ignoreBracketedPasteMode` makes the terminal ignore bracketed paste mode so the bracket sequences are left out of pasted text, off by default.

### PN-3.18
1. Right-clicking the terminal shows a menu with `Copy`, `Copy as HTML`, `Paste`, `Select All`, `Clear`, `Size to Content Width` and `Kill Terminal`.
2. In a task terminal the menu also has `Rerun Task`.
3. Holding `Shift` while right-clicking always shows the menu, whatever the right-click setting says.
4. Setting `terminal.integrated.rightClickBehavior` chooses what right-click does, with values `default` for the menu, `copyPaste`, `paste`, `selectWord` and `nothing`.
5. Setting `terminal.integrated.middleClickBehavior` chooses what middle-click does, with values `default` for the platform default and `paste`.
6. Setting `terminal.integrated.altClickMovesCursor` lets `Alt+click` move the prompt cursor to the click, on by default and only when `editor.multiCursorModifier` is `alt`.
7. Setting `terminal.integrated.macOptionClickForcesSelection` makes `Option+click` on macOS force a regular line selection, off by default.
8. In Cursor, the terminal right-click menu also has `Split` and `New Terminal` rows in a create group. [cursor only]

### PN-3.19
1. `Select All` in the right-click menu, or `Cmd+A` on macOS with any terminal focused, selects the whole buffer.
2. Pressing `Esc` clears the selection when text is selected and the find box is not visible.
3. Double-clicking a word selects it up to the nearest word separator character.
4. Setting `terminal.integrated.wordSeparators` sets the separator characters, by default spaces, brackets, quotes, comma and a few others, and it also controls plain word link detection.

### PN-3.20
1. When a program rings the terminal bell and the visual bell is on, the terminal's tab shows a warning bell icon with the tooltip `Bell`.
2. Setting `terminal.integrated.enableVisualBell` turns the tab bell icon on or off, off by default.
3. Setting `terminal.integrated.bellDuration` sets how many milliseconds the bell icon stays on the tab, 1000 by default.
4. Setting `accessibility.signals.terminalBell` chooses a sound with `auto`, `on` or `off` and an announcement with `auto` or `off` when the bell rings.
5. The old setting `terminal.integrated.enableBell` is deprecated and its settings entry points to the two newer settings.

### PN-3.21
1. When a program in the terminal sends the OSC 99 escape sequence, a workbench notification appears with the source Terminal and the program's title and body.
2. The notification shows any buttons the program asked for, plus `Dismiss` and `Disable Terminal Notifications` buttons.
3. Choosing `Disable Terminal Notifications` turns the notification setting off, so later sequences are ignored.
4. A notification can focus the terminal when clicked, if the program asked for that.
5. A program can limit its notification so it only shows while the terminal is unfocused or hidden.
6. Setting `terminal.integrated.enableNotifications` turns OSC 99 notifications on or off, on by default.

### PN-3.22
1. With screen reader mode on, pressing `Alt+F2` (`Alt+Shift+F2` on Linux) in the terminal opens a read-only text view of the terminal buffer.
2. In that view `Alt+Up` and `Alt+Down` jump to the previous and next command.
3. In that view `Ctrl+Home` and `Ctrl+End` (`Shift+Home` and `Shift+End` on Linux) jump to the top or bottom of the buffer.
4. Pressing `Alt+F1` (`Alt+Shift+F1` on Linux) in the terminal opens an accessibility help page for the terminal.
5. Pressing `Alt+F1` while the terminal find box has focus opens a separate accessibility help page for terminal find. [stock only]
6. Setting `terminal.integrated.accessibleViewPreserveCursorPosition` keeps the cursor position when the view is reopened instead of moving it to the bottom of the buffer, off by default.
7. Setting `terminal.integrated.accessibleViewFocusOnCommandExecution` moves focus to this view when a command runs, off by default.
8. Setting `accessibility.verbosity.terminal` chooses whether the terminal announces how to open its accessibility help.

### PN-3.23
1. With shell integration active, a command that finishes with a zero exit code can play a sound or an announcement for success.
2. A command that finishes with a non-zero exit code can play a sound or an announcement for failure, and moving to such a command in the accessible view does the same.
3. A separate sound or announcement fires when terminal quick fixes become available.
4. Settings `accessibility.signals.terminalCommandSucceeded`, `accessibility.signals.terminalCommandFailed` and `accessibility.signals.terminalQuickFix` each choose a sound with `auto`, `on` or `off` and an announcement with `auto` or `off`.

### PN-3.24
1. When a screen reader is detected or `editor.accessibilitySupport` is on, the terminal switches to screen reader mode, mirrors the prompt text into a hidden text area, and tells the shell process a screen reader is in use.
2. In PowerShell on Windows under a screen reader, `Ctrl+V` uses the normal paste instead of being sent straight to the shell.
3. `Ctrl+Up` and `Ctrl+Down` no longer jump between commands in screen reader mode.
4. `Ctrl+R` opens `Run Recent Command` in screen reader mode, and `Ctrl+Alt+R` sends `Ctrl+R` to the shell instead.
5. The terminal's input box is labeled with the terminal number, the title and the accessibility help key, or with a hint to turn screen reader mode on when it is off.
6. Setting `editor.accessibilitySupport` chooses whether screen reader mode is used, with values `auto`, `on` and `off`.
7. Setting `accessibility.verbosity.terminal` chooses whether the help key hint is included in the input box label.

### PN-3.25
1. Setting `terminal.integrated.gpuAcceleration` chooses whether the terminal draws with the GPU, with values `auto`, `on` and `off`, where `off` renders much slower but works on all systems.
2. Setting `terminal.integrated.customGlyphs` draws box, block and similar characters directly instead of with the font, on by default and only when GPU acceleration is on.
3. Setting `terminal.integrated.rescaleOverlappingGlyphs` squeezes wide single-cell glyphs back into their cell, on by default.

### PN-3.26
1. Setting `terminal.integrated.cursorStyle` chooses the cursor shape while the terminal is focused, with values `block`, `line` and `underline`.
2. Setting `terminal.integrated.cursorStyleInactive` chooses the cursor shape while the terminal is not focused, with values `outline`, `block`, `line`, `underline` and `none`.
3. Setting `terminal.integrated.cursorWidth` sets the width of the cursor when the style is `line`.
4. Setting `terminal.integrated.cursorBlinking` makes the cursor blink, off by default.
5. Setting `terminal.integrated.textBlinking` lets blinking text blink, off by default.

### PN-3.27
1. Settings `terminal.integrated.fontFamily` and `terminal.integrated.fontSize` set the font and size the terminal draws with.
2. Settings `terminal.integrated.fontWeight` and `terminal.integrated.fontWeightBold` set normal and bold text weight, as `normal`, `bold` or a number from 1 to 1000.
3. Settings `terminal.integrated.lineHeight` and `terminal.integrated.letterSpacing` change the spacing between lines and between characters.
4. Setting `terminal.integrated.tabStopWidth` sets how many cells a tab stop spans.
5. Setting `terminal.integrated.unicodeVersion` chooses the Unicode width table, with values `6` and `11`.
6. Setting `terminal.integrated.minimumContrastRatio` nudges text colors to meet a contrast ratio, with a default of 4.5 and a value of 1 turning it off.
7. Setting `terminal.integrated.drawBoldTextInBrightColors` chooses whether bold text always uses the bright color variants.

### PN-3.28
1. With ligatures turned on and a font that supports them, character sequences such as arrows draw as one glyph in the terminal.
2. Setting `terminal.integrated.fontLigatures.enabled` turns ligatures on or off, off by default.
3. Setting `terminal.integrated.fontLigatures.featureSettings` takes CSS `font-feature-settings` text for the ligatures, `"calt" on` by default.
4. Setting `terminal.integrated.fontLigatures.fallbackLigatures` lists character sequences that are always drawn together when the font cannot be parsed and GPU acceleration is on.

### PN-3.29
1. With the setting on and GPU rendering on, images a program sends with the Sixel protocol are drawn in the terminal.
2. Images sent with the iTerm inline image protocol are drawn on Linux and macOS, and images sent with the kitty graphics protocol are drawn on all platforms.
3. Turning the setting on also turns on transparency mode in the terminal.
4. Setting `terminal.integrated.enableImages` turns image support on or off, off by default.
5. Inline images are not restored after a window reload or reconnect.

### PN-3.30
1. `Increase Font Size` in the Command Palette makes the terminal font one step larger, up to 100.
2. `Decrease Font Size` in the Command Palette makes the terminal font one step smaller, down to 6.
3. `Reset Font Size` in the Command Palette puts the terminal font back to its default size.
4. Setting `terminal.integrated.mouseWheelZoom` lets `Ctrl` (`Cmd` on macOS) plus the mouse wheel over a terminal zoom its font, off by default.
5. Each zoom step is saved into setting `terminal.integrated.fontSize`.

### PN-3.31
1. On macOS, setting `terminal.integrated.fontRendering` set to `inherit` makes terminal text follow the workbench font aliasing setting. [stock only]
2. Setting `terminal.integrated.fontRendering` set to `grayscale` draws terminal text with grayscale antialiasing on high-DPI displays, which can look sharper. [stock only]
3. With GPU rendering on, changing this setting to `grayscale` may need `Developer: Reload Window` before it takes effect. [stock only]
4. The setting is tagged experimental and advanced and is listed only on macOS. [stock only]

### PN-3.32
1. While the terminal is being resized, a small box over it shows the new size in columns and rows, for example `80 x 24`.
2. The box fades out half a second after the last size change.
3. The box is announced politely to screen readers each time its text changes.

### PN-3.33
1. Pressing `Ctrl+Alt+PageDown` or `Ctrl+Alt+PageUp` (`Ctrl+Shift+Down` or `Ctrl+Shift+Up` on Linux) scrolls the terminal down or up one line.
2. Pressing `Shift+PageDown` or `Shift+PageUp` (plain `PageDown` or `PageUp` on macOS) scrolls the terminal down or up one page.
3. Pressing `Ctrl+End` or `Ctrl+Home` (`Shift+End` or `Shift+Home` on Linux) scrolls the terminal to the bottom or the top.
4. These scroll keys only apply while the terminal shows its normal buffer and not a program's alternate screen.
5. Setting `terminal.integrated.smoothScrolling` makes the terminal scroll with an animation, off by default.
6. Setting `terminal.integrated.mouseWheelScrollSensitivity` multiplies the mouse wheel scroll distance, 1 by default.
7. Setting `terminal.integrated.fastScrollSensitivity` multiplies the scroll speed while `Alt` is held, 5 by default.

### PN-3.34
1. `Clear` in the Command Palette empties the terminal output buffer.
2. The `Clear` row in the terminal right-click menu does the same, in the panel and in a terminal editor tab.
3. Pressing `Cmd+K` on macOS clears the terminal when it has focus with screen reader mode off, or when the accessible view shows the terminal, and Windows and Linux have no default key.

### PN-3.35
1. `Send Sequence` in the Command Palette asks `Enter sequence to send to the terminal` and types the answer into the active terminal.
2. The typed text understands `\n`, `\r` and `\xAB` escapes and resolves variables before it is sent.
3. A keybinding can give `Send Sequence` a text argument, so one key types fixed text into the terminal without asking.
4. `Send Signal` in the Command Palette, on Linux and macOS only, lists SIGINT, SIGTERM, SIGKILL, SIGSTOP, SIGCONT, SIGHUP, SIGQUIT, SIGUSR1 and SIGUSR2 with a short description each and sends the chosen signal to the terminal's process.
5. Typing a signal name that is not in the list into the `Send Signal` pick sends that signal.

### PN-3.36
1. Most keys pressed in a focused terminal go to the shell instead of triggering workbench keybindings.
2. Setting `terminal.integrated.commandsToSkipShell` lists command ids whose keys always stay with the workbench, and an id with a leading `-` hands a default skipped command back to the shell.
3. Setting `terminal.integrated.sendKeybindingsToShell` sends nearly every key to the shell and overrides the skip list.
4. Settings `terminal.integrated.allowChords` and `terminal.integrated.allowMnemonics` choose whether keybinding chords and `Alt` menu mnemonics still work while the terminal has focus.
5. Setting `terminal.integrated.macOptionIsMeta` makes `Option` act as the Meta key on macOS.
6. Settings `terminal.integrated.enableKittyKeyboardProtocol` and `terminal.integrated.enableWin32InputMode` turn on the kitty keyboard protocol and win32 input mode, so programs can read more detailed key input.
7. `Alt+Up` and `Alt+Down` send `Ctrl+Up` and `Ctrl+Down`, `Alt+Left` and `Alt+Right` send word movement, and `Ctrl+Backspace` and `Ctrl+Delete` (`Option+Backspace` and `Option+Delete` on macOS) delete a word.
8. On macOS, `Cmd+Left` and `Cmd+Right` move to the start and end of the line and `Cmd+Backspace` deletes to the start of the line.
9. `Ctrl+Shift+2` sends NUL, `Ctrl+Shift+6` sends RS and `Ctrl+Alt+G` sends `Ctrl+G` to the shell.
10. In PowerShell with shell integration and win32 input mode on, `Ctrl+Space`, `Alt+Space` and `Shift+Enter` are mapped to keys that its line editor handles.

### PN-3.37
1. When extensions change a terminal's environment, the terminal's tab gets a status whose detail lists the extensions that contributed.
2. The status has a `Show Environment Contributions` link that opens a read-only `Terminal Environment Changes` document grouped by extension and workspace folder.
3. When a change needs a restart, the status becomes a warning that lists the extensions and offers `Relaunch Terminal`.
4. A terminal you have not typed into relaunches by itself when setting `terminal.integrated.environmentChangesRelaunch` is on.
5. The terminal's input box label also says the environment is stale when a relaunch is needed.
6. In Cursor, setting `terminal.integrated.environmentChangesIndicator` chooses when the tab shows the environment status, with values `off`, `on` and `warnonly`, and `warnonly` by default. [cursor only]

### PN-3.38
1. On a slow connection, typed characters can be echoed locally in a dim style before the shell echoes them. [stock only]
2. Setting `terminal.integrated.localEchoEnabled` chooses when local echo is on, with values `on`, `off` and `auto` for remote workspaces only, and `off` by default. [stock only]
3. Setting `terminal.integrated.localEchoLatencyThreshold` sets the delay in milliseconds after which local echo starts, 30 by default, where 0 means always on and -1 means disabled. [stock only]
4. Setting `terminal.integrated.localEchoExcludePrograms` lists program names that turn local echo off when they appear in the terminal title. [stock only]
5. Setting `terminal.integrated.localEchoStyle` chooses the echo style as `bold`, `dim`, `italic`, `underlined`, `inverted` or a hex color, `dim` by default. [stock only]

### PN-3.39
1. Dragging a file or folder from the Explorer or from the operating system over a terminal shows a drop overlay on it.
2. Dropping the item types its path at the prompt without running it.
3. The typed path is quoted for the shell the terminal is running.

### PN-3.40
1. `Developer: Show Terminal Texture Atlas` in the Command Palette saves `textureAtlas.png` in the first workspace folder and opens it.
2. `Developer: Write Data to Terminal` asks for data and prints it directly in the terminal without sending it to the process.
3. `Developer: Record Terminal Session` shows `Recording terminal session...` in the status bar and copies the recorded events to the clipboard as JSON when recording ends.
4. `Developer: Restart Pty Host` restarts the process host that runs the terminals.
5. Setting `terminal.integrated.developer.devMode` adds prompt-input debugging to each terminal.

### PN-3.41
1. Theme color `terminal.background` sets the terminal's own background, and without a value it falls back to the editor background in an editor tab, the panel background in the panel and the side bar background in the side bar.
2. Theme color `terminal.foreground` sets the default text color.
3. Theme colors `terminalCursor.foreground` and `terminalCursor.background` set the cursor color, which falls back to the text color, and the color of the character under a block cursor.
4. Theme color `terminal.selectionBackground` sets the selection background and follows the editor selection color by default.
5. Theme color `terminal.inactiveSelectionBackground` sets a dimmer selection background for when the terminal is not focused.
6. Theme color `terminal.selectionForeground` sets an optional color for selected text.
7. Setting `workbench.colorCustomizations` overrides these colors, and there is no settings page for them.

### PN-3.42
1. Theme colors `terminal.ansiBlack`, `terminal.ansiRed`, `terminal.ansiGreen`, `terminal.ansiYellow`, `terminal.ansiBlue`, `terminal.ansiMagenta`, `terminal.ansiCyan` and `terminal.ansiWhite` color the normal output colors programs use.
2. Theme colors `terminal.ansiBrightBlack` through `terminal.ansiBrightWhite` color the bright version of each of the eight colors.
3. Each of the 16 colors has its own default for light, dark and high contrast themes.
4. Setting `workbench.colorCustomizations` overrides any of the 16 colors.
5. The minimum contrast ratio setting can still nudge a color for readability.

### PN-3.43
1. Theme colors `terminal.findMatchBackground` and `terminal.findMatchBorder` color the current find match.
2. Theme colors `terminal.findMatchHighlightBackground` and `terminal.findMatchHighlightBorder` color the other find matches.
3. Theme color `terminalOverviewRuler.findMatchForeground` colors the find match markers in the overview ruler.
4. Theme color `terminalOverviewRuler.cursorForeground` colors the ruler marker for the active match or for the line you just jumped to.
5. Theme color `terminalOverviewRuler.border` colors the left border of the overview ruler.
6. Theme color `terminal.hoverHighlightBackground` colors the highlight under a word whose link hover is showing.
7. Theme colors `terminalCommandDecoration.defaultBackground`, `terminalCommandDecoration.successBackground` and `terminalCommandDecoration.errorBackground` color the three kinds of command dot.
8. Theme color `terminal.dropBackground` colors the fill shown over the terminal while something is dragged onto it.
9. Setting `workbench.colorCustomizations` overrides any of these colors.

### PN-3.44
1. When the terminal output contains a message listed in setting `terminal.integrated.autoReplies`, the terminal types the configured reply, for example answering `Terminate batch job (Y/N)` with `Y`.
2. A reply written with `\r` presses Enter after the text.
3. A reply fires at most once a second.
4. The match runs on the raw output, so text that is styled or split by escape codes may not match.
5. Setting a message to `null` switches off a default reply for it.
6. The setting is empty by default and is restricted, so a workspace in Restricted Mode cannot use it.

### PN-3.45
1. In a focused terminal, `Tab` and `Shift+Tab` go to the shell, so shell completion works and focus does not jump away.
2. Pressing `Ctrl+M` (`Ctrl+Shift+M` on macOS) toggles Tab Moves Focus and still works inside the terminal because it is on the list of keys that skip the shell.
3. Once Tab Moves Focus is on, `Tab` leaves the terminal and moves focus to the next control.

### PN-3.46
1. Following an http or https link in the terminal that points at this machine, such as localhost, a loopback address or a `.localhost` name, opens it in a Cursor Browser tab. [cursor only]
2. No `Opening ... in Cursor Browser` toast is shown when such a link opens. [cursor only]
3. Holding `Shift` while following such a link sends it to the system browser instead. [cursor only]
4. The hover on such a link reads like `Follow link (ctrl + click, hold shift for your browser)`. [cursor only]
5. In a remote window, or with the Cursor Settings Browser toggle `Open Local Links in Cursor Browser` off, local links go to the system browser as in stock VS Code. [cursor only]
6. The Cursor Settings Browser toggle `Open Local Links in Cursor Browser` is on by default and is stored as `autoOpenLocalhostUrls`, not as a settings.json setting. [cursor only]

### PN-3.47
1. On Windows with the bundled conpty.dll option on, setting `terminal.integrated.windowsReparentConsoleWindow` makes the terminal's window the owner of its console window, so dialogs from console programs such as sign-in prompts appear in front of the editor instead of behind it. [stock only]
2. The setting is tagged advanced and is on by default in 1.144.0. [stock only]
3. The setting is restricted, so it has no effect in untrusted workspaces. [stock only]
4. The setting is skipped in remote windows. [stock only]

## Unclear

- PN-3.31 and PN-3.47 are tagged `[stock only]` because Cursor has no match for their settings, but both settings exist only in VS Code 1.144.0 (newer-than-1.128), not in the 1.128.0 base that Cursor is built on, so the pieces describe behavior no 1.128.0 build has.
- PN-3.14 piece 7 and PN-3.22 piece 5 are tagged `[stock only]` from the feature's Cursor field (no bundle hit for `quickFixEnabled` and for the terminal find help provider); I did not open Cursor to confirm the setting or the help page is really absent on screen.
- PN-3.16 piece 7 is tagged `[stock only]` from the feature's Cursor field (no hit for `workbench.action.terminal.pastePwsh`); in Cursor a copied file with no text is probably sent to PowerShell as plain `Ctrl+V` and not pasted as a path, but that was not run.
- PN-3.36 is at the 10-piece limit and mixes two areas, settings that decide which keys reach the shell and the built-in key rewrites for the shell, with several settings grouped; it should be split.
- PN-3.12 is also at the 10-piece limit; it fits, but the quick pick row behavior (pinning, buttons, grouping) could be split out if tracking needs finer counts.
- Some pieces rest on the feature block alone and were not re-read in V: the OSC 99 button and focus options in PN-3.21, the shell quoting of dropped paths in PN-3.39, the Relaunch label on the input box in PN-3.37, and the `null` and once-a-second rules in PN-3.44.
