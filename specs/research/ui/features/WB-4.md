Group: WB-4 Accessibility. Section: Workbench-wide. Date: 2026-10-25. Pass: 4.
Features: 9.

### WB-4.1 Assign keys for commands from Accessibility Help
- Tag: both
- Evidence: source
- Boundary: no
- What: Inside an Accessibility Help page, Alt+K lists commands that have no key so you can give them one, and Alt+A lists commands that already have a key so you can change it.
- Commands: editor.action.accessibilityHelpConfigureKeybindings; editor.action.accessibilityHelpConfigureAssignedKeybindings; workbench.action.openGlobalKeybindings
- Menus: AccessibleView (navigation group, order 3 for the unassigned list and order 4 for the assigned list)
- Keys: Alt+K (accessibilityHelpIsShown and accessibleViewHasUnassignedKeybindings); Alt+A (accessibilityHelpIsShown and accessibleViewHasAssignedKeybindings)
- Settings: none found
- Cursor: present (both ids, both chords and the labels from nls 4760 and 4761 found)
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 231-285); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (configureKeybindings, lines 447-480); V\src\vs\workbench\contrib\accessibility\common\accessibilityCommands.ts; C bundle (classes with both ids, Alt+K and Alt+A, nls 4728 to 4729 for the picker text and 4746 to 4747 for the help lines)

### WB-4.2 Open the help link from Accessibility Help
- Tag: both
- Evidence: source
- Boundary: no
- What: Inside an Accessibility Help page, Alt+H opens the page's "read more" link in the web browser.
- Commands: editor.action.accessibilityHelpOpenHelpLink
- Menus: none found
- Keys: Alt+H (when accessibilityHelpIsShown)
- Settings: none found
- Cursor: present (id, chord and label from nls 4762 found)
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 287-304); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (openHelpLink, lines 440-445); V\src\vs\workbench\contrib\accessibility\browser\editorAccessibilityHelp.ts (readMoreUrl); V\src\vs\workbench\contrib\terminalContrib\accessibility\browser\terminalAccessibilityHelp.ts (readMoreUrl); C bundle (class with the id and Alt+H, hint text nls 4749)

### WB-4.3 Turn off the accessible view hint
- Tag: both
- Evidence: source
- Boundary: no
- What: Alt+F6 in the accessible view or Accessibility Help switches off the setting that adds the how-to-open hint for that page, and announces it.
- Commands: editor.action.accessibleViewDisableHint
- Menus: AccessibleView (navigation group, bell with a slash icon); CommandPalette
- Keys: Alt+F6 (when the accessible view or help is shown and its hint setting is on)
- Settings: accessibility.verbosity.editor; accessibility.verbosity.terminal; accessibility.verbosity.debug; accessibility.verbosity.sourceControl; accessibility.verbosity.find; accessibility.verbosity.comments; accessibility.verbosity.notebook; accessibility.verbosity.hover; accessibility.verbosity.notification; accessibility.verbosity.keybindingsEditor
- Cursor: present (id, chord and label from nls 4759 found; the individual verbosity settings were not checked)
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 200-229); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (disableHint, lines 532-538); V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (AccessibilityVerbositySettingId, lines 48-75, boolean default true); C bundle (class with the id and Alt+F6, nls 4732 for the announcement and 4744 for the help line)

### WB-4.4 Jump to a symbol in the accessible view
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+Shift+O in the accessible view or Accessibility Help opens a picker of the headings, paragraphs, code blocks and list items on the page and jumps to the one you pick.
- Commands: editor.action.accessibleViewGoToSymbol
- Menus: AccessibleView (navigation group, symbol icon); CommandPalette
- Keys: Ctrl+Shift+O and Ctrl+Shift+. (when the accessible view or help is shown and the page supports symbols)
- Settings: none found
- Cursor: present (id, Ctrl+Shift+O chord and label from nls 4756 found)
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts (lines 130-157); V\src\vs\workbench\contrib\accessibility\browser\accessibleView.ts (goToSymbol, getSymbols, showSymbol, AccessibleViewSymbolQuickPick, lines 379-420, 998-1037); C bundle (class with the id and Ctrl+Shift+O, nls 4752 to 4753 for the picker text and 4745 for the help line)

### WB-4.5 Accessibility sounds and announcements for editing and run events
- Tag: both
- Evidence: source
- Boundary: no
- What: Short sounds (audio cues) and screen reader announcements play when the cursor reaches an error, warning, breakpoint or folded line, and when a diff line, code action, save, format, debugger stop, task or notebook cell event happens, each switched by its own setting.
- Commands: signals.sounds.help; accessibility.announcement.help
- Menus: CommandPalette (Help: List Signal Sounds; Help: List Signal Announcements)
- Keys: none found
- Settings: accessibility.signals.codeActionApplied; accessibility.signals.codeActionTriggered; accessibility.signals.diffLineDeleted; accessibility.signals.diffLineInserted; accessibility.signals.diffLineModified; accessibility.signals.format; accessibility.signals.lineHasBreakpoint; accessibility.signals.lineHasError; accessibility.signals.lineHasFoldedArea; accessibility.signals.lineHasWarning; accessibility.signals.noInlayHints; accessibility.signals.notebookCellCompleted; accessibility.signals.notebookCellFailed; accessibility.signals.onDebugBreak; accessibility.signals.positionHasError; accessibility.signals.positionHasWarning; accessibility.signals.save; accessibility.signals.taskCompleted; accessibility.signals.taskFailed
- Cursor: changed (all settings and both help commands are in the bundle; the code action triggered and applied signals reuse the voice recording started and stopped sounds)
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (the accessibility.signals.* block); V\src\vs\workbench\contrib\accessibilitySignals\browser\commands.ts (ShowSignalSoundHelp, ShowAccessibilityAnnouncementHelp); V\src\vs\workbench\contrib\accessibilitySignals\browser\editorTextPropertySignalsContribution.ts (line and position signals); V\src\vs\workbench\contrib\accessibilitySignals\browser\saveAccessibilitySignal.ts; V\src\vs\workbench\contrib\accessibilitySignals\browser\accessibilitySignalDebuggerContribution.ts; V\src\vs\platform\accessibilitySignal\browser\accessibilitySignalService.ts; V\src\vs\editor\contrib\format\browser\format.ts; V\src\vs\editor\contrib\codeAction\browser\codeAction.ts; V\src\vs\editor\browser\widget\diffEditor\diffEditorWidget.ts; V\src\vs\workbench\contrib\scm\browser\quickDiffWidget.ts; V\src\vs\workbench\contrib\tasks\browser\taskTerminalStatus.ts; V\src\vs\workbench\contrib\notebook\browser\services\notebookExecutionStateServiceImpl.ts; V\src\vs\workbench\contrib\inlayHints\browser\inlayHintsAccessibilty.ts; C bundle (settingsKey strings for every id above, ids signals.sounds.help and accessibility.announcement.help, codeActionTriggered registered with sound voiceRecordingStarted and codeActionApplied with voiceRecordingStopped)

### WB-4.6 Tune the volume and timing of accessibility signals
- Tag: both
- Evidence: source
- Boundary: no
- What: Set how loud the signal sounds are and, when debouncing is on, how long each kind of signal waits before it plays.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: accessibility.signalOptions.volume; accessibility.signalOptions.debouncePositionChanges; accessibility.signalOptions.experimental.delays.general; accessibility.signalOptions.experimental.delays.errorAtPosition; accessibility.signalOptions.experimental.delays.warningAtPosition
- Cursor: present (all five ids and the migrations from the older audioCues.volume and audioCues.debouncePositionChanges names are in the bundle)
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (signalOptions block and migrations); V\src\vs\platform\accessibilitySignal\browser\accessibilitySignalService.ts (getDelayMs, getVolumeInPercent); V\src\vs\workbench\contrib\accessibilitySignals\browser\editorTextPropertySignalsContribution.ts; C bundle (settings accessibility.signalOptions.volume, debouncePositionChanges, delays.general, delays.errorAtPosition, delays.warningAtPosition)

### WB-4.7 Underline links in the workbench
- Tag: both
- Evidence: source
- Boundary: no
- What: One setting draws links underlined in workbench text, hover cards, walkthrough pages, the language status popup and webviews instead of plain.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: accessibility.underlineLinks
- Cursor: present (the setting, the class toggling code and the CSS rule for the underline-links class are in the bundle and its stylesheet)
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts (accessibility.underlineLinks); V\src\vs\platform\accessibility\browser\accessibilityService.ts (toggles the underline-links class); V\src\vs\workbench\browser\media\style.css (text-link-decoration); V\src\vs\base\browser\ui\hover\hoverWidget.css; V\src\vs\workbench\contrib\welcomeWalkthrough\browser\media\walkThroughPart.css; V\src\vs\workbench\contrib\welcomeGettingStarted\browser\media\gettingStarted.css; V\src\vs\workbench\contrib\languageStatus\browser\media\languageStatus.css; V\src\vs\workbench\contrib\webview\browser\themeing.ts; C bundle (setting accessibility.underlineLinks, workbench.desktop.main.css rule .monaco-workbench.underline-links)

### WB-4.8 Toggle Screen Reader Accessibility Mode
- Tag: both
- Evidence: source
- Boundary: no
- What: The person runs Toggle Screen Reader Accessibility Mode, or presses Shift+Alt+F1, and the editor switches between screen reader optimized mode and normal mode, with a spoken message saying which one is now on.
- Commands: editor.action.toggleScreenReaderAccessibilityMode
- Menus: CommandPalette (f1, no category)
- Keys: Shift+Alt+F1 (Shift+Alt+F4 on Linux); Ctrl+E (Cmd+E on macOS) while the accessibility help dialog is shown (accessibilityHelpIsShown)
- Settings: editor.accessibilitySupport
- Cursor: present - the id, the label and both key rules are in the bundle; the two messages come through NLS indexes 898 and 899
- Source: V\src\vs\workbench\contrib\codeEditor\browser\accessibility\accessibility.ts (ToggleScreenReaderMode, lines 22-54); V\src\vs\editor\common\config\editorOptions.ts (accessibilitySupport, lines 1452-1465); V\src\vs\editor\common\standaloneStrings.ts (lines 22-23); C bundle (class with id editor.action.toggleScreenReaderAccessibilityMode and the two key rules with weight 210)

### WB-4.9 Announce Cursor Position
- Tag: vscode
- Evidence: source
- Boundary: no
- What: With the editor focused and screen reader mode on, the person presses Ctrl+Alt+Shift+G and the screen reader says the cursor's line and column.
- Commands: editor.action.announceCursorPosition
- Menus: CommandPalette (f1, no category)
- Keys: Ctrl+Alt+Shift+G (Cmd+Alt+Shift+G on macOS) when editorTextFocus and accessibilityModeEnabled
- Settings: none found
- Cursor: missing - neither the id nor the label "Announce Cursor Position" is in the main bundle
- Source: V\src\vs\workbench\contrib\codeEditor\browser\accessibility\accessibility.ts (AnnounceCursorPosition, lines 56-92); V\src\vs\platform\accessibility\common\accessibility.ts (accessibilityModeEnabled, line 40); C bundle (no hit for the id or the label)

## Handed off
- none

## Unclear
- Created in pass 4 from the coverage check.
