Group: PN-3 Terminal tooling. Section: Panel. Date: 2026-10-25. Pass: 2.
Features: 47.

### PN-3.1 Find in the terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+F in a terminal to open a find box over it, type to highlight matches in the output and scrollback, step with Enter and Shift+Enter, flip match case, whole word and regular expression, read the match count, and close it with Esc.
- Commands: workbench.action.terminal.focusFind; workbench.action.terminal.hideFind; workbench.action.terminal.findNext; workbench.action.terminal.findPrevious; workbench.action.terminal.toggleFindRegex; workbench.action.terminal.toggleFindWholeWord; workbench.action.terminal.toggleFindCaseSensitive
- Menus: none found
- Keys: Ctrl+F (Cmd+F on macOS) on focusFind when a terminal or the find box has focus; Esc on hideFind while the find box is visible; Alt+R (Cmd+Alt+R on macOS) on toggleFindRegex; Alt+W (Cmd+Alt+W on macOS) on toggleFindWholeWord; Alt+C (Cmd+Alt+C on macOS) on toggleFindCaseSensitive; F3 (Cmd+G on macOS) on findNext; Shift+F3 (Cmd+Shift+G on macOS) on findPrevious; Enter and Shift+Enter inside the find input
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\find\browser\terminal.find.contribution.ts (lines 104-240); V\src\vs\workbench\contrib\terminalContrib\find\browser\terminalFindWidget.ts; V\src\vs\workbench\contrib\terminalContrib\find\common\terminal.find.ts; C bundle (all seven command ids; module id terminal.find)

### PN-3.2 Search the workspace for selected terminal text
- Tag: both
- Evidence: source
- Boundary: no
- What: With text selected in the terminal, press Ctrl+Shift+F to open the Search view filled with that text, so a path or error string can be searched across the workspace.
- Commands: workbench.action.terminal.searchWorkspace
- Menus: none found
- Keys: Ctrl+Shift+F (Cmd+Shift+F on macOS) when the terminal has focus, a process is supported and text is selected
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\find\browser\terminal.find.contribution.ts (lines 241-257); C bundle (id searchWorkspace)

### PN-3.3 Click a link in terminal output
- Tag: both
- Evidence: source
- Boundary: no
- What: The terminal finds file paths (with line and column), folders, URLs and plain words in its output and underlines them on hover; Ctrl+click (Cmd+click on macOS, Alt+click when the multi-cursor modifier is Ctrl or Cmd) opens a file in the editor, focuses a workspace folder in the Explorer, opens an outside folder in a new window, opens a URL, or runs a workspace search for a word; a URL with a scheme that is not on the allowed list first asks "Allow <scheme>?" in a warning notification.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enableFileLinks; terminal.integrated.allowedLinkSchemes; editor.multiCursorModifier
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkManager.ts (lines 115-155, 455-480); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkDetectorAdapter.ts (lines 128-140); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLocalLinkDetector.ts; terminalUriLinkDetector.ts; terminalWordLinkDetector.ts; terminalMultiLineLinkDetector.ts; terminalLinkOpeners.ts; V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 507-531); C bundle (settings enableFileLinks and allowedLinkSchemes; module id terminal.link)

### PN-3.4 Hover a terminal link
- Tag: both
- Evidence: source
- Boundary: no
- What: Pointing at a detected link shows a hover that names the action and the click modifier, such as "Follow link (ctrl + click)", "Open file in editor", "Focus folder in explorer", "Open folder in new window", "Search workspace", or "Follow link using forwarded port"; the hover can be turned off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.showLinkHover
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\widgets\terminalHoverWidget.ts (lines 27-60); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkManager.ts (lines 460-510); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkDetectorAdapter.ts (lines 128-140); C bundle (setting showLinkHover)

### PN-3.5 Open a link from the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Shift+O to get a quick pick of the links in the terminal, grouped as Url, File, Folder and Workspace Search, with type-to-filter over all links (first the visible area, then the whole buffer); the Command Palette also has Open Last URL Link and Open Last Local File Link, and the same quick pick opens from the accessible terminal view with Ctrl+Shift+G.
- Commands: workbench.action.terminal.openDetectedLink; workbench.action.terminal.openUrlLink; workbench.action.terminal.openFileLink
- Menus: none found
- Keys: Ctrl+Shift+O (Cmd+Shift+O on macOS) on openDetectedLink when the terminal has focus; Ctrl+Shift+G (Cmd+Shift+G on macOS) on openDetectedLink when the accessible view shows the terminal
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\links\browser\terminal.links.contribution.ts (lines 117-160); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkQuickpick.ts (lines 55-125); V\src\vs\workbench\contrib\terminalContrib\links\common\terminal.links.ts; C bundle (ids openDetectedLink, openUrlLink, openFileLink)

### PN-3.6 Command decorations
- Tag: both
- Evidence: source
- Boundary: no
- What: When shell integration is active, each command gets a dot in the left gutter and a mark in the right overview ruler that shows success or failure; hovering a dot says when the command ran, how long it took, and its exit code; right-clicking a dot shows Toggle Visibility, which opens a two-checkbox pick for gutter decorations and overview ruler decorations.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.shellIntegration.decorationsEnabled
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\xterm\decorationAddon.ts (lines 128-140, 396-424, 565-606); V\src\vs\workbench\contrib\terminal\browser\xterm\decorationStyles.ts (lines 32-135); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 616-628); C bundle (setting decorationsEnabled; action id terminal.toggleVisibility)

### PN-3.7 Rerun and copy a command or its output
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking a command's dot opens a menu with Rerun Command (it asks "Do you want to run the command" first when shell integration did not mark the command as trusted), Copy Command, Copy Command and Output, Copy Output, Copy Output as HTML, Run Recent Command, Go To Recent Directory and Learn About Shell Integration; the Command Palette has Copy Last Command, Copy Last Command Output and Copy Last Command and Output for the newest command.
- Commands: workbench.action.terminal.copyLastCommand; workbench.action.terminal.copyLastCommandOutput; workbench.action.terminal.copyLastCommandAndLastCommandOutput
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\xterm\decorationAddon.ts (lines 396-520, menu actions built in code with ids terminal.rerunCommand, terminal.copyCommand, terminal.copyCommandAndOutput, terminal.copyOutput, terminal.copyOutputAsHtml, terminal.learnShellIntegration); V\src\vs\workbench\contrib\terminalContrib\clipboard\browser\terminal.clipboard.contribution.ts (lines 188-246); C bundle (the three copyLast ids; action ids terminal.rerunCommand, terminal.copyCommandAndOutput, terminal.copyOutputAsHtml and terminal.learnShellIntegration)

### PN-3.8 Jump between commands
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+Up and Ctrl+Down scroll the terminal to the previous or next command mark so old commands can be skimmed; the same two actions can be added as arrow buttons on the terminal view toolbar or a terminal editor toolbar (hidden by default).
- Commands: workbench.action.terminal.scrollToPreviousCommand; workbench.action.terminal.scrollToNextCommand
- Menus: ViewTitle; EditorTitle; CompactWindowEditorTitle
- Keys: Ctrl+Up on scrollToPreviousCommand and Ctrl+Down on scrollToNextCommand (Cmd on macOS), when the terminal has focus and screen reader mode is off
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 868-930); V\src\vs\workbench\contrib\terminal\browser\xterm\markNavigationAddon.ts; C bundle (ids scrollToPreviousCommand and scrollToNextCommand)

### PN-3.9 Select command output
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+Shift+Up and Ctrl+Shift+Down extend a selection from the current spot to the previous or next command mark, which selects whole commands and their output; Select to Previous Line and Select to Next Line in the Command Palette extend the selection one line at a time.
- Commands: workbench.action.terminal.selectToPreviousCommand; workbench.action.terminal.selectToNextCommand; workbench.action.terminal.selectToPreviousLine; workbench.action.terminal.selectToNextLine
- Menus: none found
- Keys: Ctrl+Shift+Up on selectToPreviousCommand and Ctrl+Shift+Down on selectToNextCommand (Cmd+Shift on macOS), when the terminal has focus; none for the two line commands
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 932-982); V\src\vs\workbench\contrib\terminal\browser\xterm\markNavigationAddon.ts (lines 470-520); C bundle (all four ids)

### PN-3.10 Command guide
- Tag: both
- Evidence: source
- Boundary: no
- What: Hovering over the text of a command draws a thin vertical guide bar along that command's prompt and output, so you can see where one command starts and ends.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.shellIntegration.showCommandGuide
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\commandGuide\browser\terminal.commandGuide.contribution.ts (lines 20-90); V\src\vs\workbench\contrib\terminalContrib\commandGuide\common\terminalCommandGuideConfiguration.ts; V\src\vs\workbench\contrib\terminal\browser\xterm\markNavigationAddon.ts (showCommandGuide, line 290); C bundle (setting showCommandGuide; css class terminal-command-guide)

### PN-3.11 Sticky scroll
- Tag: both
- Evidence: source
- Boundary: no
- What: While you scroll up through a long command's output, that command's prompt line stays pinned at the top of the terminal (up to a set number of lines); hovering it says "Navigate to Command" with the jump keys, clicking it jumps back to the command, right-clicking it offers Toggle Sticky Scroll, and some commands can be excluded; Toggle Sticky Scroll is also in the Command Palette.
- Commands: workbench.action.terminal.toggleStickyScroll
- Menus: TerminalStickyScrollContext
- Keys: none found
- Settings: terminal.integrated.stickyScroll.enabled; terminal.integrated.stickyScroll.maxLineCount; terminal.integrated.stickyScroll.ignoredCommands
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\stickyScroll\browser\terminal.stickyScroll.contribution.ts (lines 20-45); V\src\vs\workbench\contrib\terminalContrib\stickyScroll\browser\terminalStickyScrollOverlay.ts (lines 86, 399-462); V\src\vs\workbench\contrib\terminalContrib\stickyScroll\common\terminalStickyScrollConfiguration.ts; V\src\vs\platform\actions\common\actions.ts (line 249); C bundle (id toggleStickyScroll; menu id TerminalStickyScrollContext; three settings)

### PN-3.12 Run recent command
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Alt+R to open a quick pick of recent commands from this terminal session and the shell's history file; moving through the rows scrolls the terminal to that command, Enter runs the chosen one, holding Alt types it on the prompt without running so it can be edited, each row has buttons to view its output in an editor or remove it from history, rows can be pinned, and a fuzzy search toggle sits in the input; Clear Previous Session History in the Command Palette wipes the stored history.
- Commands: workbench.action.terminal.runRecentCommand; workbench.action.terminal.clearPreviousSessionHistory
- Menus: ViewTitle; EditorTitle; CompactWindowEditorTitle
- Keys: Ctrl+Alt+R (Ctrl+Option+R on macOS) on runRecentCommand when the terminal has focus and screen reader mode is off; Ctrl+R on runRecentCommand when screen reader mode is on and the terminal or accessible terminal view has focus
- Settings: terminal.integrated.shellIntegration.history
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\history\browser\terminal.history.contribution.ts (lines 100-215); V\src\vs\workbench\contrib\terminalContrib\history\browser\terminalRunRecentQuickPick.ts (lines 54-275); V\src\vs\workbench\contrib\terminalContrib\history\common\terminal.history.ts; C bundle (ids runRecentCommand and clearPreviousSessionHistory; setting shellIntegration.history)

### PN-3.13 Go to recent directory
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+G to open a quick pick of folders visited in this terminal and listed in the shell's history; choosing one runs a cd command for that folder (hold Alt to type it on the prompt without running), and the pick has the same remove button and fuzzy search toggle as Run Recent Command.
- Commands: workbench.action.terminal.goToRecentDirectory
- Menus: ViewTitle; EditorTitle; CompactWindowEditorTitle
- Keys: Ctrl+G (Cmd+G on macOS) when the terminal has focus
- Settings: terminal.integrated.shellIntegration.history
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\history\browser\terminal.history.contribution.ts (lines 114-156); V\src\vs\workbench\contrib\terminalContrib\history\browser\terminalRunRecentQuickPick.ts (lines 196-250); C bundle (id goToRecentDirectory)

### PN-3.14 Quick fixes after a command
- Tag: both
- Evidence: source
- Boundary: no
- What: After a command finishes with a matching result, a lightbulb or sparkle icon appears to the left of the prompt; clicking it or pressing Ctrl+. lists fixes that either run a command ("Run: ...") or open a link ("Open: ..."), such as git suggestions for a mistyped subcommand, a fast-forward pull, push with set-upstream, create a pull request, retry with two dashes, free a busy port, PowerShell's "most similar commands" and command-not-found suggestions, and the npm "Did you mean" fix; the same fixes are listed in the command's dot menu.
- Commands: workbench.action.terminal.showQuickFixes
- Menus: none found
- Keys: Ctrl+. (Cmd+. on macOS) on showQuickFixes when the terminal has focus
- Settings: terminal.integrated.shellIntegration.quickFixEnabled
- Cursor: changed (the command and the built-in fix ids are present; the quickFixEnabled setting has no hit in the bundle)
- Source: V\src\vs\workbench\contrib\terminalContrib\quickFix\browser\terminal.quickFix.contribution.ts (lines 30-100); V\src\vs\workbench\contrib\terminalContrib\quickFix\browser\terminalQuickFixBuiltinActions.ts; V\src\vs\workbench\contrib\terminalContrib\quickFix\browser\quickFixAddon.ts (lines 300-430); V\extensions\npm\package.json (contributes.terminalQuickFixes, id ms-vscode.npm-command); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 637-641); C\extensions\npm\package.json; C bundle (id showQuickFixes; fix ids Git Similar, Git Fast Forward Pull, Git Two Dashes, Free Port, Git Push Set Upstream, Git Create Pr, Pwsh General Error, Unix Command Not Found)

### PN-3.15 Copy selected text
- Tag: both
- Evidence: source
- Boundary: no
- What: Copy the selection with Ctrl+Shift+C (Cmd+C on macOS); on Windows Ctrl+C copies and clears the selection when text is selected and otherwise goes to the shell; Copy as HTML keeps colors and styles; a setting copies automatically as soon as text is selected.
- Commands: workbench.action.terminal.copySelection; workbench.action.terminal.copyAndClearSelection; workbench.action.terminal.copySelectionAsHtml
- Menus: TerminalInstanceContext; TerminalEditorInstanceContext
- Keys: Ctrl+Shift+C (Cmd+C on macOS) on copySelection and Ctrl+C on Windows on copyAndClearSelection, when text is selected and the terminal has focus
- Settings: terminal.integrated.copyOnSelection
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\clipboard\browser\terminal.clipboard.contribution.ts (lines 60-72, 248-292); V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (lines 120-140); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 159-163); C bundle (the three ids; setting copyOnSelection)

### PN-3.16 Paste into the terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: Paste the clipboard with Ctrl+V (Cmd+V on macOS; Ctrl+Shift+V also works on Windows and is the key on Linux), paste the Linux selection clipboard with Shift+Insert, or paste on middle click when that is chosen; if the clipboard holds a copied file and no text, its path is pasted; on Windows with PowerShell Ctrl+V is passed straight to the shell so PSReadLine handles it unless screen reader mode is on.
- Commands: workbench.action.terminal.paste; workbench.action.terminal.pasteSelection; workbench.action.terminal.pastePwsh
- Menus: TerminalInstanceContext; TerminalEditorInstanceContext
- Keys: Ctrl+V (Cmd+V on macOS) on paste; Ctrl+Shift+V on paste (secondary on Windows, primary on Linux); Shift+Insert on pasteSelection (Linux only); Ctrl+V on pastePwsh (Windows, PowerShell, screen reader mode off)
- Settings: none found
- Cursor: changed (paste and pasteSelection are present; the pastePwsh id has no hit, and the PowerShell Ctrl+V rule is a plain send-sequence rule with no file-clipboard fallback)
- Source: V\src\vs\workbench\contrib\terminalContrib\clipboard\browser\terminal.clipboard.contribution.ts (lines 80-124, 294-322); V\src\vs\workbench\contrib\terminalContrib\sendSequence\browser\terminal.sendSequence.contribution.ts (lines 124-151); C bundle (ids paste and pasteSelection; a send-sequence rule for Ctrl+V on terminalShellType pwsh)

### PN-3.17 Multi-line paste warning
- Tag: both
- Evidence: source
- Boundary: no
- What: Pasting text with more than one line opens a warning dialog that asks whether to paste N lines, shows a preview of the first three lines, and offers Paste, Paste as one line and Cancel plus a "Do not ask me again" checkbox that sets the setting to never; in the default auto mode it is skipped when the shell is in bracketed paste mode and a single command with a trailing newline is pasted without the newline; a second setting makes the terminal ignore bracketed paste mode.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enableMultiLinePasteWarning; terminal.integrated.ignoreBracketedPasteMode
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\clipboard\browser\terminalClipboard.ts (lines 30-110); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 164-175, 653-657); C bundle (setting enableMultiLinePasteWarning with values auto, always and never; setting ignoreBracketedPasteMode)

### PN-3.18 Right-click menu and mouse click behavior
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-clicking the terminal shows a menu with Copy, Copy as HTML, Paste, Select All, Clear, Size to Content Width and Kill Terminal (plus Rerun Task in a task terminal); a setting changes right-click to copy-or-paste, paste, select the word under the pointer, or do nothing (Shift+right-click always shows the menu), middle-click can paste, Alt+click moves the prompt cursor to the click, and on macOS Option+click can force a normal line selection.
- Commands: workbench.action.terminal.copySelection; workbench.action.terminal.copySelectionAsHtml; workbench.action.terminal.paste; workbench.action.terminal.selectAll; workbench.action.terminal.clear; workbench.action.terminal.sizeToContentWidth; workbench.action.terminal.killViewOrEditor
- Menus: TerminalInstanceContext; TerminalEditorInstanceContext
- Keys: none found
- Settings: terminal.integrated.rightClickBehavior; terminal.integrated.middleClickBehavior; terminal.integrated.altClickMovesCursor; terminal.integrated.macOptionClickForcesSelection
- Cursor: changed (the menu and all four settings are present; Cursor's TerminalInstanceContext also holds Split and New Terminal rows in group 1_create, which V lists only in TerminalEditorInstanceContext)
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (lines 108-185 and 205-300); V\src\vs\workbench\contrib\terminalContrib\clipboard\browser\terminal.clipboard.contribution.ts (lines 124-163); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 149-158, 360-381); V\src\vs\workbench\contrib\terminal\browser\terminalTabbedView.ts (line 100); C bundle (TerminalInstanceContext menu items for split, new, killViewOrEditor, copySelection, copySelectionAsHtml, paste, clear, sizeToContentWidth and selectAll; the four settings)

### PN-3.19 Select text and word separators
- Tag: both
- Evidence: source
- Boundary: no
- What: Select All selects the whole buffer (Cmd+A on macOS, or Select All in the right-click menu), Esc clears the selection, and double-clicking a word selects it up to the nearest word separator character (spaces, brackets, quotes, comma and similar by default).
- Commands: workbench.action.terminal.selectAll; workbench.action.terminal.clearSelection
- Menus: TerminalInstanceContext; TerminalEditorInstanceContext
- Keys: Cmd+A on selectAll (macOS only, when any terminal has focus); Esc on clearSelection when text is selected and the find box is not visible
- Settings: terminal.integrated.wordSeparators
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 695-705, 1195-1210); V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (lines 172-182); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 501-506); C bundle (ids selectAll and clearSelection; setting wordSeparators)

### PN-3.20 Bell
- Tag: both
- Evidence: source
- Boundary: no
- What: When a program rings the terminal bell, the tab shows a warning bell icon with the tooltip "Bell" for a set time and the terminal bell signal can play a sound or announcement; the visual bell is off by default and the old enableBell setting is deprecated and migrated to the two newer settings.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enableVisualBell; terminal.integrated.bellDuration; terminal.integrated.enableBell; accessibility.signals.terminalBell
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 855-870); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 250-254, 413-421, 780-795); V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (setting accessibility.signals.terminalBell); C bundle (settings enableBell, enableVisualBell, bellDuration; signal setting accessibility.signals.terminalBell with its legacy key accessibility.alert.terminalBell)

### PN-3.21 Terminal notifications
- Tag: both
- Evidence: source
- Boundary: no
- What: A program in the terminal can send the OSC 99 escape sequence to raise a notification; it shows as a workbench notification from the source Terminal with the program's title, body and any buttons it asked for, plus Dismiss and Disable Terminal Notifications buttons, can focus the terminal when clicked, and can be limited to when the terminal is unfocused or hidden; turning the setting off ignores the sequence.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enableNotifications
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\notification\browser\terminalNotificationHandler.ts (lines 60-110, 300-370); V\src\vs\workbench\contrib\terminalContrib\notification\browser\terminal.notification.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\notification\common\terminalNotificationConfiguration.ts; C bundle (setting enableNotifications; handler action ids terminal.osc99.dismiss and terminal.osc99.disable)

### PN-3.22 Accessible terminal view and help
- Tag: both
- Evidence: source
- Boundary: no
- What: With screen reader mode on, Alt+F2 in the terminal opens a read-only text view of the terminal buffer where Alt+Up and Alt+Down jump between commands and Ctrl+Home and Ctrl+End jump to the top or bottom; Alt+F1 opens an accessibility help page for the terminal (and a separate one while the find box has focus); two settings control whether the cursor position is kept and whether this view takes focus when a command runs.
- Commands: workbench.action.terminal.focusAccessibleBuffer; workbench.action.terminal.accessibleBufferGoToNextCommand; workbench.action.terminal.accessibleBufferGoToPreviousCommand; workbench.action.terminal.scrollToBottomAccessibleView; workbench.action.terminal.scrollToTopAccessibleView; editor.action.accessibilityHelp
- Menus: none found
- Keys: Alt+F2 (Alt+Shift+F2 on Linux) on focusAccessibleBuffer when screen reader mode is on and the terminal has focus; Alt+Down and Alt+Up on the two go-to-command commands while the accessible view shows the terminal; Ctrl+End and Ctrl+Home (Shift+End and Shift+Home on Linux) on the scroll commands in that view; Alt+F1 (Alt+Shift+F1 on Linux) on editor.action.accessibilityHelp
- Settings: terminal.integrated.accessibleViewPreserveCursorPosition; terminal.integrated.accessibleViewFocusOnCommandExecution; accessibility.verbosity.terminal
- Cursor: changed (the five terminal commands and the terminal-help provider are present; the terminal find help provider, id terminalFindHelp, has no hit)
- Source: V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminal.accessibility.contribution.ts; V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminalAccessibilityHelp.ts; V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminalAccessibleBufferProvider.ts; V\src\vs\workbench\contrib\terminalContrib\find\browser\terminalFindAccessibilityHelp.ts; V\src\vs\workbench\contrib\terminalContrib\accessibility\common\terminalAccessibilityConfiguration.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 160-200); C bundle (the five ids; provider id terminal-help; settings accessibleViewPreserveCursorPosition and accessibleViewFocusOnCommandExecution; setting accessibility.verbosity.terminal)

### PN-3.23 Command result sounds and announcements
- Tag: both
- Evidence: source
- Boundary: no
- What: With shell integration active, a finished command can play a sound or screen reader announcement for success or for failure (also when you move to such a command in the accessible view), and a separate signal fires when terminal quick fixes become available.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: accessibility.signals.terminalCommandFailed; accessibility.signals.terminalCommandSucceeded; accessibility.signals.terminalQuickFix
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\xterm\decorationAddon.ts (lines 248-256); V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminal.accessibility.contribution.ts (lines 190-198); V\src\vs\workbench\contrib\terminalContrib\quickFix\browser\quickFixAddon.ts (line 310); V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (lines 475-515); C bundle (the three signal settings)

### PN-3.24 Screen reader mode in the terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: When a screen reader is detected or accessibility support is on, the terminal switches xterm to screen reader mode, mirrors the prompt text into a hidden text area, tells the shell process about it (PowerShell does not run PSReadLine under a screen reader, so the PowerShell-only Ctrl+V rule is switched off), turns off the Ctrl+Up and Ctrl+Down command jumps, uses Ctrl+R for Run Recent Command, and labels its input box with the terminal number, the title and the accessibility help key (or a hint to turn screen reader mode on when it is off).
- Commands: none found
- Menus: none found
- Keys: Ctrl+R on runRecentCommand in screen reader mode; Ctrl+Alt+R sends Ctrl+R to the shell in screen reader mode
- Settings: editor.accessibilitySupport; accessibility.verbosity.terminal
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 1985-1988, 2098-2114); V\src\vs\workbench\contrib\terminal\browser\terminalProcessManager.ts (lines 313, 525); V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\textAreaSyncAddon.ts (lines 15-70); V\src\vs\workbench\contrib\terminalContrib\sendSequence\browser\terminal.sendSequence.contribution.ts (lines 124-136, 200-205); V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 872-909); C bundle (xterm option screenReaderMode set from the accessibility service; module id terminal.textAreaSync)

### PN-3.25 GPU acceleration
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose whether the terminal draws with the GPU (auto, on or off; off renders much slower but works everywhere), plus two related options that draw box and block characters directly and that squeeze wide glyphs back into their cell.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.gpuAcceleration; terminal.integrated.customGlyphs; terminal.integrated.rescaleOverlappingGlyphs
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 334-343, 575-592); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts; C bundle (the three settings)

### PN-3.26 Cursor style
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick the terminal cursor shape (block, line or underline), the shape it takes when the terminal is not focused (outline, block, line, underline or none), the width of the line cursor, and whether the cursor and blinking text blink.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.cursorStyle; terminal.integrated.cursorStyleInactive; terminal.integrated.cursorWidth; terminal.integrated.cursorBlinking; terminal.integrated.textBlinking
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 293-318); C bundle (all five settings)

### PN-3.27 Font and text appearance
- Tag: both
- Evidence: source
- Boundary: no
- What: Set the terminal font family, size, weight and bold weight, line height, letter spacing, tab stop width and Unicode width table, set a minimum contrast ratio (default 4.5) that nudges text colors, and choose whether bold text uses the bright color variants.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.fontFamily; terminal.integrated.fontSize; terminal.integrated.fontWeight; terminal.integrated.fontWeightBold; terminal.integrated.lineHeight; terminal.integrated.letterSpacing; terminal.integrated.tabStopWidth; terminal.integrated.unicodeVersion; terminal.integrated.minimumContrastRatio; terminal.integrated.drawBoldTextInBrightColors
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 172-182, 228-238, 255-292, 533-541); C bundle (all ten settings)

### PN-3.28 Font ligatures
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn on font ligatures (off by default) so the configured terminal font draws sequences such as arrows as one glyph, tune the CSS font-feature-settings used with them, and give a fallback list of character sequences to join when the font cannot be parsed.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.fontLigatures.enabled; terminal.integrated.fontLigatures.featureSettings; terminal.integrated.fontLigatures.fallbackLigatures
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 184-206); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts; C bundle (the three settings; the xterm ligatures addon is loaded)

### PN-3.29 Inline images
- Tag: both
- Evidence: source
- Boundary: no
- What: When turned on (off by default) and GPU rendering is on, images that a program sends with Sixel, the iTerm inline image protocol (Linux and macOS) or the kitty graphics protocol (all platforms) are drawn in the terminal, and transparency mode is switched on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enableImages
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 658-664); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts; C bundle (setting enableImages; the xterm image addon is loaded and refreshed)

### PN-3.30 Zoom the terminal font
- Tag: both
- Evidence: source
- Boundary: no
- What: Increase Font Size, Decrease Font Size and Reset Font Size in the Command Palette change the terminal font size one step at a time (6 to 100), and holding Ctrl (Cmd on macOS) while scrolling the mouse wheel over a terminal zooms it when that setting is on.
- Commands: workbench.action.terminal.fontZoomIn; workbench.action.terminal.fontZoomOut; workbench.action.terminal.fontZoomReset
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.mouseWheelZoom; terminal.integrated.fontSize
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\zoom\browser\terminal.zoom.contribution.ts (lines 40-170); V\src\vs\workbench\contrib\terminalContrib\zoom\common\terminal.zoom.ts; C bundle (the three ids; setting mouseWheelZoom)

### PN-3.31 Text antialiasing
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. An experimental, advanced setting that is listed only on macOS lets the terminal follow the workbench font aliasing or force grayscale antialiasing on high-DPI displays; with GPU rendering it may need a window reload.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.fontRendering
- Cursor: missing
- Source: N\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 183-194); N\src\vs\workbench\contrib\terminal\browser\xterm\terminalFontRendering.ts; C bundle (zero hits for fontRendering)

### PN-3.32 Size overlay while resizing
- Tag: both
- Evidence: source
- Boundary: no
- What: While the terminal is being resized, a small box over it shows the new size in columns and rows (for example "80 x 24") and fades out half a second after the last change; it is announced politely to screen readers.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\resizeDimensionsOverlay\browser\terminalResizeDimensionsOverlay.ts (lines 14-62); V\src\vs\workbench\contrib\terminal\terminal.all.ts (line 33); C bundle (module id terminal.resizeDimensionsOverlay)

### PN-3.33 Scroll the terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: Scroll by one line, by one page, or to the top or bottom from the keyboard while a program is not using the alternate screen, and tune smooth scrolling, the mouse wheel speed and the Alt-key fast scroll multiplier; the size of the scrollback buffer is described in PN-2.39.
- Commands: workbench.action.terminal.scrollDown; workbench.action.terminal.scrollUp; workbench.action.terminal.scrollDownPage; workbench.action.terminal.scrollUpPage; workbench.action.terminal.scrollToBottom; workbench.action.terminal.scrollToTop
- Menus: none found
- Keys: Ctrl+Alt+PageDown and Ctrl+Alt+PageUp on the line commands (Ctrl+Shift+Down and Ctrl+Shift+Up on Linux); Shift+PageDown and Shift+PageUp on the page commands (plain PageDown and PageUp on macOS); Ctrl+End and Ctrl+Home on bottom and top (Shift+End and Shift+Home on Linux); all with a terminal in focus in the normal buffer
- Settings: terminal.integrated.smoothScrolling; terminal.integrated.mouseWheelScrollSensitivity; terminal.integrated.fastScrollSensitivity
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 612-693); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 240-249, 648-652); C bundle (all six ids; the three settings)

### PN-3.34 Clear the terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: Clear empties the terminal output buffer (Cmd+K on macOS, the Clear row in the right-click menu, or the Command Palette) and also works from the accessible terminal view; no separate Reset Terminal command exists in 1.128.0.
- Commands: workbench.action.terminal.clear
- Menus: TerminalInstanceContext; TerminalEditorInstanceContext
- Keys: Cmd+K on macOS when the terminal has focus and screen reader mode is off, or when the accessible view shows the terminal; none on Windows and Linux
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 1354-1370); V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (lines 152-162); C bundle (id clear)

### PN-3.35 Send a sequence or a signal
- Tag: both
- Evidence: source
- Boundary: no
- What: Send Sequence asks for text (it understands \n, \r and \xAB escapes, and resolves variables) or takes a text argument from a keybinding and types it into the active terminal; Send Signal, in the Command Palette on Linux and macOS only, lists SIGINT, SIGTERM, SIGKILL, SIGSTOP, SIGCONT, SIGHUP, SIGQUIT, SIGUSR1 and SIGUSR2 or lets you type a signal name and sends it to the terminal's process.
- Commands: workbench.action.terminal.sendSequence; workbench.action.terminal.sendSignal
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\sendSequence\browser\terminal.sendSequence.contribution.ts (lines 30-120); V\src\vs\workbench\contrib\terminalContrib\sendSignal\browser\terminal.sendSignal.contribution.ts (lines 1-95); C bundle (ids sendSequence and sendSignal)

### PN-3.36 Keys that go to the shell
- Tag: both
- Evidence: source
- Boundary: no
- What: Most keys in a focused terminal go to the shell, and settings decide which workbench keybindings still win (a list of commands that always skip the shell, a switch to send nearly everything to the shell, whether chords and Alt menu mnemonics are allowed, Option as Meta on macOS, the kitty keyboard protocol and win32 input mode); built-in rules also rewrite some keys for the shell, such as Alt+arrows to word movement, Ctrl+Backspace and Ctrl+Delete to delete a word, Cmd+Left, Cmd+Right and Cmd+Backspace on macOS to line start, line end and delete to line start, and a set of PowerShell-only shortcuts.
- Commands: workbench.action.terminal.sendSequence
- Menus: none found
- Keys: Alt+Up and Alt+Down send Ctrl+Up and Ctrl+Down; Alt+Left and Alt+Right send word movement; Ctrl+Backspace (Option+Backspace on macOS) and Ctrl+Delete (Option+Delete on macOS) delete a word; Cmd+Left, Cmd+Right and Cmd+Backspace on macOS; Ctrl+Shift+2 sends NUL; Ctrl+Shift+6 sends RS; Ctrl+Alt+G sends Ctrl+G; Ctrl+Space, Alt+Space and Shift+Enter in PowerShell with shell integration and win32 input mode; all with a terminal in focus
- Settings: terminal.integrated.commandsToSkipShell; terminal.integrated.sendKeybindingsToShell; terminal.integrated.allowChords; terminal.integrated.allowMnemonics; terminal.integrated.macOptionIsMeta; terminal.integrated.enableKittyKeyboardProtocol; terminal.integrated.enableWin32InputMode
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 1148-1192); V\src\vs\workbench\contrib\terminal\browser\terminalConfigurationService.ts (lines 25-75); V\src\vs\workbench\contrib\terminal\common\terminal.ts (line 502, default skip list); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 47-52, 144-148, 423-446, 593-606); V\src\vs\workbench\contrib\terminalContrib\sendSequence\browser\terminal.sendSequence.contribution.ts (lines 108-260); C bundle (the seven settings; send-sequence key rules such as Alt+Up with the Ctrl+Up sequence)

### PN-3.37 Environment variable collection indicator
- Tag: both
- Evidence: source
- Boundary: no
- What: When extensions change a terminal's environment, its tab gets a status whose detail lists which extensions contributed, with a Show Environment Contributions link that opens a read-only "Terminal Environment Changes" document grouped by extension and workspace folder; if the change needs a restart the status is a warning that lists the extensions and offers Relaunch Terminal, and a terminal you have not typed into relaunches by itself when the relaunch setting is on; the input box label also says the environment is stale.
- Commands: workbench.action.terminal.showEnvironmentContributions; workbench.action.terminal.relaunch
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.environmentChangesRelaunch
- Cursor: changed (the commands, the statuses and the relaunch setting are present; Cursor also keeps a terminal.integrated.environmentChangesIndicator setting with off, on and warnonly, default warnonly, which V 1.128.0 does not define)
- Source: V\src\vs\workbench\contrib\terminal\browser\environmentVariableInfo.ts (lines 28-95); V\src\vs\workbench\contrib\terminalContrib\environmentChanges\browser\terminal.environmentChanges.contribution.ts (lines 20-95); V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 2304-2345); V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 474-478); C bundle (ids showEnvironmentContributions and relaunch; status ids relaunch-needed and env-var-info-changes-active; setting environmentChangesIndicator at its definition with enum off, on and warnonly)

### PN-3.38 Local echo
- Tag: vscode
- Evidence: source
- Boundary: no
- What: On a slow connection, typed characters can be echoed locally in a dim style before the shell echoes them; it can be on, off or on only for remote workspaces (off by default), starts after a latency threshold, skips listed programs, and the echo style can be a font style or a color.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.localEchoEnabled; terminal.integrated.localEchoLatencyThreshold; terminal.integrated.localEchoExcludePrograms; terminal.integrated.localEchoStyle
- Cursor: missing
- Source: V\src\vs\workbench\contrib\terminalContrib\typeAhead\common\terminalTypeAheadConfiguration.ts; V\src\vs\workbench\contrib\terminalContrib\typeAhead\browser\terminalTypeAheadAddon.ts; V\src\vs\workbench\contrib\terminal\terminal.all.ts (line 32); C bundle (zero hits for localEcho, typeAhead and TypeAhead)

### PN-3.39 Drop a file to type its path
- Tag: both
- Evidence: source
- Boundary: no
- What: Dragging a file or folder from the Explorer or the operating system onto a terminal shows a drop overlay and, on drop, types the item's path at the prompt, quoted for the shell, without running it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 1259-1268, 1405-1415, 2549-2652); C bundle (css class terminal-drop-overlay and the drop controller code that reads the drop side)

### PN-3.40 Terminal developer diagnostics
- Tag: both
- Evidence: source
- Boundary: no
- What: The Command Palette has Developer entries for the terminal: Show Terminal Texture Atlas (saves textureAtlas.png in the first workspace folder and opens it), Write Data to Terminal (prints raw data to the terminal bypassing the process), Record Terminal Session (shows "Recording terminal session..." in the status bar and copies the recorded events to the clipboard as JSON when it ends), and Restart Pty Host; a developer mode setting adds prompt-input debugging to each terminal.
- Commands: workbench.action.terminal.showTextureAtlas; workbench.action.terminal.writeDataToTerminal; workbench.action.terminal.recordSession; workbench.action.terminal.restartPtyHost
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.developer.devMode
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\developer\browser\terminal.developer.contribution.ts (lines 31-215 and 217-362); V\src\vs\workbench\contrib\terminalContrib\developer\common\terminal.developer.ts; C bundle (the four ids; setting developer.devMode)

### PN-3.41 Terminal text, cursor and selection colors
- Tag: both
- Evidence: source
- Boundary: no
- What: A color theme or `workbench.colorCustomizations` sets the terminal's own background (terminal.background, which falls back to the editor background for a terminal in an editor tab, the panel background in the panel, or the side bar background in the side bar), the default text color (terminal.foreground), the cursor color (terminalCursor.foreground, which falls back to the text color), the color of the character under a block cursor (terminalCursor.background), the selection background (terminal.selectionBackground, which follows the editor selection color), a dimmer selection background for when the terminal is not focused (terminal.inactiveSelectionBackground), and an optional selection text color (terminal.selectionForeground); there is no settings page for them.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\common\terminalColorRegistry.ts (lines 18-38); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts (getXtermTheme, lines 1009-1035); V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (getBackgroundColor, lines 2901-2914); C bundle (all seven color ids found, with the same registry text and defaults)

### PN-3.42 ANSI color palette
- Tag: both
- Evidence: source
- Boundary: no
- What: A theme or `workbench.colorCustomizations` sets the 16 colors that programs use for colored output - black, red, green, yellow, blue, magenta, cyan and white, plus a bright version of each - as terminal.ansiBlack through terminal.ansiBrightWhite; each has its own default for light, dark and high contrast themes, and the minimum contrast ratio in PN-3.27 can still nudge a color for readability.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\common\terminalColorRegistry.ts (ansiColorMap and its registration loop, lines 102-253); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts (lines 1037-1052); C bundle (all 16 ids found; the registry copy has the same defaults, for example ansiBlack index 0 with #000000 in light and dark)

### PN-3.43 Highlight and marker colors
- Tag: both
- Evidence: source
- Boundary: no
- What: A theme or `workbench.colorCustomizations` also colors the terminal's helper marks: the current find match and the other matches (fill and border, terminal.findMatchBackground, terminal.findMatchBorder, terminal.findMatchHighlightBackground, terminal.findMatchHighlightBorder), their marker in the overview ruler (terminalOverviewRuler.findMatchForeground), the ruler marker for the active match or for the line you just jumped to (terminalOverviewRuler.cursorForeground), the left border of the ruler (terminalOverviewRuler.border), the highlight under a word whose link hover is showing (terminal.hoverHighlightBackground), the three command dots (terminalCommandDecoration.defaultBackground, successBackground and errorBackground), and the fill shown over the terminal while something is dragged onto it (terminal.dropBackground).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\common\terminalColorRegistry.ts (lines 40-93); V\src\vs\workbench\contrib\terminal\browser\xterm\xtermTerminal.ts (_updateFindColors, lines 621-642); V\src\vs\workbench\contrib\terminal\browser\xterm\decorationAddon.ts (line 612); V\src\vs\workbench\contrib\terminal\browser\xterm\markNavigationAddon.ts (line 389); V\src\vs\workbench\contrib\terminal\browser\media\terminal.css (lines 44 and 616); C bundle (all 12 ids found)

### PN-3.44 Auto replies
- Tag: both
- Evidence: source
- Boundary: no
- What: A setting holds a list of messages the terminal should watch for and the reply to type when one shows up, for example "Terminate batch job (Y/N)" answered with Y and Enter (written as \r), so routine prompts are answered without you; a reply can fire at most once a second, the match runs on the raw output so styled text may not match, a null value switches off a default, the setting is empty by default, and it is restricted so a workspace in Restricted Mode cannot use it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.autoReplies
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\autoReplies\common\terminalAutoRepliesConfiguration.ts (lines 7-33); V\src\vs\workbench\contrib\terminalContrib\autoReplies\browser\terminal.autoReplies.contribution.ts (lines 14-55); C bundle (module terminalAutoRepliesConfiguration.js with default {} and restricted true; the config-change listener that calls uninstallAllAutoReplies)

### PN-3.45 Tab key in the terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: In a focused terminal the Tab key and Shift+Tab go to the shell, so shell completion works and focus does not jump away; Toggle Tab Key Moves Focus (Ctrl+M, or Ctrl+Shift+M on macOS) is on the terminal's list of keys that skip the shell, so it still works there, and once Tab Moves Focus is on, Tab leaves the terminal and moves focus to the next control instead; the command and its status bar item are described in ED-5 and SS-3.
- Commands: editor.action.toggleTabFocusMode
- Menus: none found
- Keys: Ctrl+M (Ctrl+Shift+M on macOS) on editor.action.toggleTabFocusMode
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (lines 1168-1180); V\src\vs\workbench\contrib\terminal\common\terminal.ts (line 560, default skip-shell list); V\src\vs\editor\contrib\toggleTabFocusMode\browser\toggleTabFocusMode.ts (lines 15-25); C bundle (the id is in the skip-shell list; the toggle command class carries the same id)

### PN-3.46 Localhost links open in the Cursor Browser
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: In Cursor, following an http or https link in the terminal that points at this machine (localhost, a loopback address or a .localhost name) opens it in a Cursor Browser tab, with no "Opening ... in Cursor Browser" toast, and holding Shift while following it sends it to the system browser instead; the link hover then reads like "Follow link (ctrl + click, hold shift for your browser)"; it does not apply in a remote window or when the Cursor Settings Browser toggle "Open Local Links in Cursor Browser" is off, where the link goes to the system browser as in stock VS Code. Boundary reason: the link lands in the Cursor Browser, which is an agent-driven surface.
- Commands: workbench.action.openBrowserEditor
- Menus: none found
- Keys: Shift held while following the link forces the system browser; the follow modifier itself is the one in PN-3.3
- Settings: autoOpenLocalhostUrls (a Cursor application storage key shown on the Cursor Settings Browser page, default on; it is not a settings.json id)
- Cursor: cursor only
- Source: C bundle (the terminal URL link opener near the string "Tried to open a url without a resolved URI", which sets openExternal from the localhost check and Shift key; the link hover builder that adds NLS message 10847 ", hold shift for your browser"; the opener service branch that runs workbench.action.openBrowserEditor; the storage default autoOpenLocalhostUrls true; the Cursor Settings entry labelled "Open Local Links in Cursor Browser"); V\src\vs\workbench\contrib\terminalContrib\links\browser\terminalLinkOpeners.ts (TerminalUrlLinkOpener, which always passes openExternal true in stock). I did not open Cursor Settings or click a link on screen.

### PN-3.47 Console dialogs stay in front on Windows
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. A Windows-only advanced setting that needs the bundled conpty.dll option (PN-2.47) makes the terminal's window the owner of its console window, so dialogs that console programs open, such as sign-in prompts, appear in front of the editor instead of behind it; it is on by default in 1.144.0, restricted in untrusted workspaces, and skipped in remote windows.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.windowsReparentConsoleWindow
- Cursor: missing
- Source: N\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 501-511); N\src\vs\workbench\contrib\terminal\browser\terminalProcessManager.ts (lines 373 and 782-832); N\src\vs\platform\terminal\common\terminal.ts (line 102); C bundle (zero hits for windowsReparentConsoleWindow)

## Handed off

- Terminal suggest (the suggest popup, its commands, its `terminal.integrated.suggest.*` settings, `terminal.integrated.shellIntegration.environmentReporting`, the terminal-suggest extension, and Cursor's `workbench.action.terminal.requestCompletions`): AC-5.
- Shell integration on or off, its wait time, the Rich, Basic and No level, and the launch-failure warning with Learn more and Open user settings: already PN-2.38 and PN-2.13, so not repeated here.
- Status icons on the tab, including the bell icon, the relaunch warning and the environment info (the tab side of PN-3.20 and PN-3.37): PN-2.13.
- Hover card on a terminal tab and Focus Hover (Ctrl+K Ctrl+I): PN-2.14.
- Scrollback size and the restored scrollback size: PN-2.39.
- Size to Content Width (Alt+Z) and Set Fixed Dimensions, which also sit in the right-click menu: PN-2.40.
- Run Selected Text and Run Active File in a terminal: PN-2.10.
- Kill Terminal and Rerun Task rows in the right-click menu: PN-2.28 and PN-2.37.
- The `term ` quick access list that switches terminals: QA-4.
- Theme colors terminal.border (the divider between split panes) and terminal.tab.activeBorder (the border on the active row in the tabs list): PN-2. The theme color terminal.initialHintForeground belongs with the first-prompt tip in PN-2.49.

## Unclear

- Dictation in the terminal (`workbench.action.terminal.startVoice` and `stopVoice`, labels Start Dictation in Terminal and Stop Dictation in Terminal, a mic button on the terminal view toolbar and editor toolbar, Esc to stop) is not listed. It needs the Microsoft speech extension and the command only offers to install it when it is missing, so it is reachable only through an extension. Cursor has none of its ids or labels. V files: `V\src\vs\workbench\contrib\terminalContrib\voice\browser\terminalVoiceActions.ts`, `V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts` (lines 531 and 761).
- Auto replies (`terminal.integrated.autoReplies`) has a setting and no screen of its own: a message in the output gets an automatic answer. It is not listed as a feature for that reason. Cursor has the setting. V files: `V\src\vs\workbench\contrib\terminalContrib\autoReplies\browser\terminal.autoReplies.contribution.ts`, `V\src\vs\workbench\contrib\terminalContrib\autoReplies\common\terminalAutoRepliesConfiguration.ts`.
- The WSL recommendation (`terminalContrib\wslRecommendation`) recommends and installs an extension, so it is left out with the extension system. The Welcome hint line on an empty prompt (`terminalContrib\inlineHint`) and the Attach To Chat row in V's command menu are chat and are left out.
- N (VS Code 1.144.0) changes that touch this group and are newer than 1.128: a `terminal.integrated.resizeDimensionsOverlay.enabled` setting for PN-3.32 (`N\src\vs\workbench\contrib\terminalContrib\resizeDimensionsOverlay\common\terminalResizeDimensionsOverlayConfiguration.ts`), `terminal.integrated.accessibleViewPreserveCursorPosition` becoming a choice of true, false and always for PN-3.22 (`N\src\vs\workbench\contrib\terminalContrib\accessibility\common\terminalAccessibilityConfiguration.ts`), and a Start Dictation row in the right-click menu (`workbench.action.terminal.startVoiceContext`, in N only). Cursor has none of them.
- PN-2.13 describes the same tab statuses as PN-3.20 and PN-3.37. This file owns the program-side behavior (what rings the bell, what the environment indicator lists and does, the Show Environment Contributions document), and PN-2.13 owns how the icon looks on the tab.
- The Cursor menu check for PN-3.18 is from the bundle only. I did not open the menu on screen, so the Split and New Terminal rows are known from the registered items and not from a running window.
- No Reset Terminal command was found in V, N or the Cursor bundle. The user's list names clear and reset, so I recorded Clear only.
- I did not check how Cursor's keybinding default for `workbench.action.terminal.clear` (Cmd+K on macOS) compares, because keybinding defaults in the bundle are minified. Presence is by command id only for every Keys entry.
- Overflow check on 2026-10-25 added 7 features (PN-3.41 to PN-3.47), including auto replies in PN-3.44, which the second Unclear note above had left out.


