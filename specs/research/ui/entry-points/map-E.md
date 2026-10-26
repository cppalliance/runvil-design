Bucket: E. Date: 2026-10-25. Pass: 4.
MAP | accessibility.debugWatchVariableAnnouncements | SB-5.17 | setting | Screen reader announces value changes in the Watch view (debugAccessibilityAnnouncer.ts)
MAP | accessibility.replEditor.autoFocusReplExecution | ED-7.39 | setting | Chooses where focus goes after code runs in the REPL editor: none, input or lastExecution
MAP | accessibility.replEditor.readLastExecutionOutput | ED-7.39 | setting | Screen reader reads the output of the last REPL execution
NEW | accessibility.signalOptions.debouncePositionChanges | NEW-E-2 | setting | Turns on waiting before position based signals play; the delay settings only apply when it is on
NEW | accessibility.signalOptions.experimental.delays.errorAtPosition | NEW-E-2 | setting | Sound and announcement delay (1000 and 3000 ms) for an error at the cursor
NEW | accessibility.signalOptions.experimental.delays.general | NEW-E-2 | setting | Sound and announcement delay (400 and 3000 ms) for every other signal
NEW | accessibility.signalOptions.experimental.delays.warningAtPosition | NEW-E-2 | setting | Sound and announcement delay (1000 and 3000 ms) for a warning at the cursor
NEW | accessibility.signalOptions.volume | NEW-E-2 | setting | Loudness of all signal sounds, 0 to 100, default 70
NEW | accessibility.signals.codeActionApplied | NEW-E-1 | setting | Sound when a code action has been applied (sound only)
NEW | accessibility.signals.codeActionTriggered | NEW-E-1 | setting | Sound when a code action is triggered (sound only)
NEW | accessibility.signals.diffLineDeleted | NEW-E-1 | setting | Sound on a deleted line in a diff or quick diff peek; ED-6 hands the diff sounds to WB
NEW | accessibility.signals.diffLineModified | NEW-E-1 | setting | Sound on a modified line in a diff or quick diff peek
EXCL | accessibility.signals.editsKept | setting | agent | Played only from chat editing code (chatEditing session and notebook integration) when edits are kept
EXCL | accessibility.signals.editsUndone | setting | agent | Played only from chat editing code when edits are undone
NEW | accessibility.signals.format | NEW-E-1 | setting | Sound and announcement when a file or notebook is formatted: userGesture, always or never
NEW | accessibility.signals.lineHasBreakpoint | NEW-E-1 | setting | Cursor lands on a line with a breakpoint
NEW | accessibility.signals.lineHasError | NEW-E-1 | setting | Cursor lands on a line with an error
NEW | accessibility.signals.lineHasFoldedArea | NEW-E-1 | setting | Cursor lands on a line with a folded area
NEW | accessibility.signals.lineHasWarning | NEW-E-1 | setting | Cursor lands on a line with a warning
NEW | accessibility.signals.noInlayHints | NEW-E-1 | setting | Reading a line that should have inlay hints but has none
NEW | accessibility.signals.notebookCellCompleted | NEW-E-1 | setting | A notebook cell finishes running
NEW | accessibility.signals.notebookCellFailed | NEW-E-1 | setting | A notebook cell fails
NEW | accessibility.signals.onDebugBreak | NEW-E-1 | setting | The debugger stopped on a breakpoint
NEW | accessibility.signals.positionHasError | NEW-E-1 | setting | Cursor sits on the text of an error (description in stock says warning by mistake)
NEW | accessibility.signals.positionHasWarning | NEW-E-1 | setting | Cursor sits on the text of a warning
NEW | accessibility.signals.save | NEW-E-1 | setting | Sound and announcement when a file is saved: userGesture, always or never
NEW | accessibility.signals.taskCompleted | NEW-E-1 | setting | A task finishes
NEW | accessibility.signals.taskFailed | NEW-E-1 | setting | A task fails with a non-zero exit code
NEW | accessibility.underlineLinks | NEW-E-3 | setting | Underlines links in the workbench, hover cards, walkthroughs and webviews
NEW | application.shellEnvironmentResolutionTimeout | NEW-E-4 | setting | Seconds before giving up on resolving the shell environment; macOS and Linux only, hidden on Windows
MAP | breadcrumbs.showArrays | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showBooleans | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showConstants | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showConstructors | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showEnumMembers | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showEnums | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showEvents | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showFields | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showFunctions | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showInterfaces | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showKeys | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showModules | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showNamespaces | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showNull | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showNumbers | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showObjects | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showOperators | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showPackages | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showProperties | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showStrings | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showStructs | ED-5.8 | setting | One of the symbol kind switches
MAP | breadcrumbs.showTypeParameters | ED-5.8 | setting | One of the symbol kind switches
MAP | debug.closeReadonlyTabsOnEnd | SB-5.31 | setting | Closes the read-only debug source tabs (like loaded scripts) when the session ends; medium confidence on the feature choice
MAP | editor.accessibilityPageSize | SS-3.26 | setting | How many lines a screen reader reads at once; applies in screen reader optimized mode, default 500
EXCL | editor.allowOverflow | setting | not-ui | Embedder API option with no settings schema
MAP | editor.allowVariableFonts | ED-5.38 | setting | Allows custom fonts in decorations; not in the Cursor bundle (stock only)
MAP | editor.allowVariableFontsInAccessibilityMode | ED-5.38 | setting | Same switch while a screen reader is on, default off; not in the Cursor bundle (stock only)
MAP | editor.allowVariableLineHeights | ED-5.38 | setting | Allows per line heights from decorations; not in the Cursor bundle (stock only)
EXCL | editor.ariaLabel | setting | not-ui | Embedder API option, default label "Editor content", no settings schema
EXCL | editor.ariaRequired | setting | not-ui | Embedder API option with no settings schema
EXCL | editor.auto | setting | not-ui | Not a setting; an extraction artifact from nls keys such as editor.autoClosingBrackets.* that are real settings owned by ED-2
EXCL | editor.automaticLayout | setting | not-ui | Embedder API option with no settings schema
MAP | editor.bracketPairColorization | ED-5.25 | setting | Parent option of editor.bracketPairColorization.enabled and independentColorPoolPerBracketType
MAP | editor.comments | ED-2.16 | setting | Parent option of editor.comments.insertSpace and ignoreEmptyLines for Toggle Comment
MAP | editor.cursorHeight | ED-5.31 | setting | Height of the line cursor, sits beside cursorWidth
EXCL | editor.disableLayerHinting | setting | not-ui | Embedder API option with no settings schema
EXCL | editor.disableMonospaceOptimizations | setting | not-ui | Embedder API option with no settings schema
EXCL | editor.domReadOnly | setting | not-ui | Embedder API option with no settings schema
MAP | editor.dropIntoEditor | ED-2.29 | setting | Parent option of editor.dropIntoEditor.enabled and showDropSelector
NEW | editor.experimental.asyncTokenization | NEW-E-5 | setting | Moves syntax tokenization to a web worker, default on, experimental
EXCL | editor.experimental.asyncTokenizationLogging | setting | developer-only | Writes tokenization logs, described as for debugging only
EXCL | editor.experimental.asyncTokenizationVerification | setting | developer-only | Checks async tokens against the old path, for debugging only
NEW | editor.experimental.preferTreeSitter.css | NEW-E-5 | setting | Use tree-sitter coloring for CSS, default off
NEW | editor.experimental.preferTreeSitter.ini | NEW-E-5 | setting | Use tree-sitter coloring for INI, default off
NEW | editor.experimental.preferTreeSitter.regex | NEW-E-5 | setting | Use tree-sitter coloring for regex, default off
NEW | editor.experimental.preferTreeSitter.typescript | NEW-E-5 | setting | Use tree-sitter coloring for TypeScript, default off
EXCL | editor.experimental.treeSitterTelemetry | setting | not-ui | Collects telemetry while tree-sitter parses; no visible effect, and stock 1.128 has no reader for it
EXCL | editor.extraEditorClassName | setting | not-ui | Embedder API option with no settings schema
EXCL | editor.fixedOverflowWidgets | setting | not-ui | Embedder API option with no settings schema
MAP | editor.fontVariations | ED-5.38 | setting | Turns font weight into font-variation-settings, boolean or CSS string
MAP | editor.guides | ED-5.23 | setting | Parent option of the editor.guides.* indent and bracket pair guide settings
MAP | editor.hover | ED-4.1 | setting | Parent option of the editor.hover.* settings
EXCL | editor.inDiffEditor | setting | not-ui | Embedder API option with no settings schema
MAP | editor.inlayHints | ED-4.21 | setting | Parent option of the editor.inlayHints.* settings
MAP | editor.inlineSuggest | AC-2.33 | setting | Parent option of the editor.inlineSuggest.* settings
EXCL | editor.inlineSuggest.experimental.emptyResponseInformation | setting | not-ui | Controls sending request information to the provider; no visible effect and not in the Cursor bundle
MAP | editor.inlineSuggest.experimental.showOnSuggestConflict | AC-2.35 | setting | Shows ghost text while the suggest list is open: always, never or whenSuggestListIsIncomplete; not in the Cursor bundle (stock only)
MAP | editor.inlineSuggest.experimental.suppressInlineSuggestions | AC-2.33 | setting | Hides inline suggestions from named extension ids; not in the Cursor bundle (stock only)
EXCL | editor.inlineSuggest.triggerCommandOnProviderChange | setting | not-ui | Picks an internal refresh path when a provider changes (marked TODO to remove in source); not in the Cursor bundle
MAP | editor.language.brackets | ED-5.24 | setting | Per language bracket pairs that bracket matching and indentation use
MAP | editor.language.colorizedBracketPairs | ED-5.25 | setting | Per language bracket pairs that get nesting colors
