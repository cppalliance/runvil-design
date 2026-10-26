Bucket: E. Date: 2026-10-25. Pass: 4.

### NEW-E-1 Accessibility sounds and announcements for editing and run events
- Group: WB-4
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
- Pieces:
  1. Each event is one setting named `accessibility.signals.<event>` that holds a `sound` value of auto (plays when a screen reader is attached), on or off and, for most events, an `announcement` value of auto (speaks only in screen reader optimized mode) or off.
  2. Moving the cursor to a different line that has an error, a warning, a breakpoint or a folded area plays that line's sound or announcement once for the line (`lineHasError`, `lineHasWarning`, `lineHasBreakpoint`, `lineHasFoldedArea`).
  3. Placing the cursor on the text of an error or a warning, or having a marker newly appear under a still cursor, plays a separate signal (`positionHasError`, `positionHasWarning`).
  4. Stepping through changes in a diff, or moving over a changed line in a quick diff peek, plays a sound for a deleted line or a modified line (`diffLineDeleted`, `diffLineModified`, with `diffLineInserted` for added lines), sound only.
  5. Choosing a code action plays one sound when it starts and another when it has been applied (`codeActionTriggered`, `codeActionApplied`), sound only.
  6. Saving a file and formatting a file take `userGesture`, `always` or `never` for sound and for announcement, default `never`, so auto save and format on save can be told apart from the person asking for it (`save`, `format`).
  7. The debugger stopping on a breakpoint (`onDebugBreak`), a task finishing or failing with a non-zero exit code (`taskCompleted`, `taskFailed`), and a notebook cell finishing or failing (`notebookCellCompleted`, `notebookCellFailed`) each play a sound or announcement.
  8. Asking to read the inlay hints of a line that has none plays `noInlayHints`.
  9. Help: List Signal Sounds and Help: List Signal Announcements open a checkable list of every signal where the highlighted sound plays as a preview, Enter applies the on and off choices, a gear button on the save and format rows opens that setting in settings.json, and the announcements list changes nothing unless a screen reader is attached.
  10. In Cursor the code action triggered and applied signals play the voice recording started and stopped sounds instead of their own sound files. [cursor only]

### NEW-E-2 Tune the volume and timing of accessibility signals
- Group: WB-4
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
- Pieces:
  1. `accessibility.signalOptions.volume` sets the loudness of every signal sound from 0 to 100 percent, default 70, and 0 makes them silent.
  2. `accessibility.signalOptions.debouncePositionChanges` (default off) makes line and position signals wait before they play, so quick cursor movement does not play every one; with it off they play at once.
  3. `accessibility.signalOptions.experimental.delays.general` sets a sound delay (default 400 ms) and an announcement delay (default 3000 ms) for every signal except an error or warning at the cursor position.
  4. `accessibility.signalOptions.experimental.delays.errorAtPosition` sets the sound delay (default 1000 ms) and announcement delay (default 3000 ms) for an error under the cursor.
  5. `accessibility.signalOptions.experimental.delays.warningAtPosition` sets the sound delay (default 1000 ms) and announcement delay (default 3000 ms) for a warning under the cursor.
  6. The three delay settings only take effect while `debouncePositionChanges` is on.
  7. Values saved under the older `audioCues.volume`, `audioCues.debouncePositionChanges` and `accessibility.signals.sounds.volume` names move to the new setting names automatically.

### NEW-E-3 Underline links in the workbench
- Group: WB-4
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
- Pieces:
  1. With `accessibility.underlineLinks` on, links inside paragraphs of workbench text, such as notification and dialog messages, are drawn underlined.
  2. With the setting off, the default, those links are drawn without an underline.
  3. Links in hover cards, the walkthrough and Welcome pages and the language status popup follow the same switch.
  4. Links in webview pages are given the same underline choice and change when the setting changes.
  5. Changing the setting takes effect at once without reloading the window.
  6. High contrast light and dark themes always underline links in paragraphs, whatever the setting says.

### NEW-E-4 Slow shell environment progress and failure notice
- Group: NT-5
- Tag: both
- Evidence: source
- Boundary: no
- What: On macOS and Linux, when the app was not started from a terminal, a status bar item reads "Resolving shell environment..." while your shell variables load, and an error notification with Restart, Configure and Learn More appears if that fails or runs out of time.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: application.shellEnvironmentResolutionTimeout
- Cursor: present (the error handler vscode:showResolveShellEnvError with three buttons, the progress with a 1600 ms delay and a Learn More button, and the setting with the same 1 to 120 range and a flag that leaves it out on Windows are in the bundle)
- Source: V\src\vs\workbench\electron-browser\window.ts (error prompt and the progress call); V\src\vs\workbench\electron-browser\desktop.contribution.ts (the setting); V\src\vs\platform\shell\node\shellEnv.ts (when it runs, timeout, messages); V\src\vs\code\electron-main\app.ts (sends the error to the window); C bundle (handler vscode:showResolveShellEnvError, call to shellEnv with delay 1600, setting application.shellEnvironmentResolutionTimeout)
- Pieces:
  1. When the shell environment is still loading after 1.6 seconds, a window progress item reading "Resolving shell environment..." shows in the status bar with a Learn More button that opens the documentation.
  2. If resolving times out or fails, an error notification shows the reason, for example "Unable to resolve your shell environment in a reasonable time. Please review your shell configuration and restart."
  3. The notification's Restart button relaunches the application.
  4. The notification's Configure button opens the user Settings filtered to `application.shellEnvironmentResolutionTimeout`.
  5. The notification's Learn More button opens the documentation page.
  6. `application.shellEnvironmentResolutionTimeout` sets the number of seconds to wait, from 1 to 120, default 10, and applies to the whole application.
  7. On Windows and when the app was started from a terminal nothing is resolved, so neither the progress item nor the error appears, and the setting is left out of the Settings editor on Windows.

### NEW-E-5 Choose the syntax coloring engine (experimental)
- Group: ED-5
- Tag: both
- Evidence: source
- Boundary: no
- What: Experimental settings switch syntax coloring for TypeScript, CSS, INI and regex files from the TextMate grammars to tree-sitter parsing, and move tokenization onto a background worker.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.experimental.asyncTokenization; editor.experimental.preferTreeSitter.typescript; editor.experimental.preferTreeSitter.css; editor.experimental.preferTreeSitter.ini; editor.experimental.preferTreeSitter.regex
- Cursor: present (all five keys are in the bundle; the four tree-sitter keys carry the tags experimental and onExP)
- Source: V\src\vs\editor\common\config\editorConfigurationSchema.ts; V\src\vs\workbench\services\treeSitter\browser\treeSitterLibraryService.ts (supportsLanguage reads the per-language setting); V\src\vs\editor\common\model\tokens\tokenizationTextModelPart.ts (chooses the tree-sitter or TextMate backend); V\src\vs\workbench\services\textMate\browser\textMateTokenizationFeatureImpl.ts (async tokenization switch); C bundle (keys editor.experimental.asyncTokenization, editor.experimental.preferTreeSitter.css, .typescript, .ini, .regex)
- Pieces:
  1. Turning on `editor.experimental.preferTreeSitter.typescript`, default off, makes TypeScript files take their syntax colors from tree-sitter parsing instead of the TextMate grammar.
  2. The same on or off switch exists for CSS, INI and regex as `editor.experimental.preferTreeSitter.css`, `editor.experimental.preferTreeSitter.ini` and `editor.experimental.preferTreeSitter.regex`, each independent of the others.
  3. `editor.experimental.asyncTokenization`, default on, runs TextMate tokenization on a web worker so syntax coloring does not hold up typing; turning it off colors on the main thread.
  4. In the Settings editor all five settings carry the Experimental tag.
