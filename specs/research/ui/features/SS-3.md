Group: SS-3 Current editor and language details. Section: Status bar. Date: 2026-10-25. Pass: 2.
Features: 53.

### SS-3.1 Line and column readout
- Tag: both
- Evidence: source
- Boundary: no
- What: The right side of the bar shows the cursor position of the active text editor as "Ln {n}, Col {n}"; the column counts a tab by its display width, and the item is hidden in notebook cell editors (id status.editor.selection, name "Editor Selection", priority 100.5).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts; C bundle (module vs/workbench/browser/parts/editor/editorStatus, id status.editor.selection, label "Ln {0}, Col {1}")

### SS-3.2 Selection size and multi-cursor count
- Tag: both
- Evidence: source
- Boundary: no
- What: With text selected the same item reads "Ln {n}, Col {n} ({n} selected)", and with several cursors it reads "{n} selections" or "{n} selections ({n} characters selected)".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (getSelectionLabel); C bundle (editorStatus labels "Ln {0}, Col {1} ({2} selected)", "{0} selections ({1} characters selected)", "{0} selections")

### SS-3.3 Click the position to go to a line or column
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the line and column item (tooltip "Go to Line/Column") opens the Go to Line quick pick for the active editor, where the person types a line and optional column.
- Commands: workbench.action.gotoLine
- Menus: MenubarGoMenu
- Keys: Ctrl+G (mac Ctrl+G)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts; V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts; V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (Go menu row "Go to Line/Column..."); C bundle (workbench.action.gotoLine with the same Ctrl+G and mac Ctrl+G keys, label "Go to Line/Column...")

### SS-3.4 Indentation readout
- Tag: both
- Evidence: source
- Boundary: no
- What: The item shows the indent style of the active text editor as "Spaces: {n}", "Spaces: {n} (Tab Size: {m})" or "Tab Size: {n}", with tooltip "Select Indentation"; it is hidden in notebook cell editors (id status.editor.indentation, name "Editor Indentation", priority 100.4).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.insertSpaces; editor.tabSize; editor.indentSize; editor.detectIndentation
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (onIndentationChange); V\src\vs\editor\common\config\editorConfigurationSchema.ts; C bundle (editorStatus labels "Spaces: {0}", "Spaces: {0} (Tab Size: {1})", "Tab Size: {0}", id status.editor.indentation, settings editor.insertSpaces, editor.tabSize, editor.indentSize, editor.detectIndentation)

### SS-3.5 Pick an indentation action from the status item
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the indentation item opens a "Select Action" quick pick with a "change view" group (Indent Using Spaces, Indent Using Tabs, Change Tab Display Size) and a "convert file" group (Detect Indentation from Content, Convert Indentation to Spaces, Convert Indentation to Tabs, Trim Trailing Whitespace); it says "The active code editor is read-only." or "No text editor active at this time" when it cannot run.
- Commands: changeEditorIndentation{windowId}
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (showIndentationPicker, command registered per window id); C bundle (command id changeEditorIndentation, labels "change view", "convert file", "Select Action")

### SS-3.6 Switch a file to spaces or tabs and pick the size
- Tag: both
- Evidence: source
- Boundary: no
- What: From the indentation picker the person chooses Indent Using Spaces or Indent Using Tabs, then picks a size from 1 to 8 (rows say Configured, Default or Current Tab Size) and the file's indent style and size change for that file only; the same two editor actions are what ED-2.18 lists as commands.
- Commands: editor.action.indentUsingSpaces; editor.action.indentUsingTabs
- Menus: none found
- Keys: none found
- Settings: editor.insertSpaces; editor.tabSize; editor.indentSize
- Cursor: present
- Source: V\src\vs\editor\contrib\indentation\browser\indentation.ts (ChangeIndentationSizeAction, IndentUsingSpaces, IndentUsingTabs); V\src\vs\workbench\browser\parts\editor\editorStatus.ts; C bundle (editor.action.indentUsingSpaces, editor.action.indentUsingTabs)

### SS-3.7 Change how wide a tab looks
- Tag: both
- Evidence: source
- Boundary: no
- What: The Change Tab Display Size row opens the 1 to 8 size list and changes only how many spaces a tab character is drawn as for that file, without changing the indent size or the indent style.
- Commands: editor.action.changeTabDisplaySize
- Menus: none found
- Keys: none found
- Settings: editor.tabSize
- Cursor: present
- Source: V\src\vs\editor\contrib\indentation\browser\indentation.ts (ChangeTabDisplaySize, displaySizeOnly); C bundle (editor.action.changeTabDisplaySize)

### SS-3.8 Convert existing indentation to spaces or tabs
- Tag: both
- Evidence: source
- Boundary: no
- What: The Convert Indentation to Spaces and Convert Indentation to Tabs rows rewrite the leading whitespace of every line in the file to the other style and then switch the file's style to match, keeping the selection in place; they need a writable editor.
- Commands: editor.action.indentationToSpaces; editor.action.indentationToTabs
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\indentation\browser\indentation.ts (IndentationToSpacesAction, IndentationToTabsAction, precondition writable); C bundle (editor.action.indentationToSpaces, editor.action.indentationToTabs)

### SS-3.9 Detect the indentation from the file content
- Tag: both
- Evidence: source
- Boundary: no
- What: The Detect Indentation from Content row looks at the lines in the file and sets the file's indent style and size to match them, falling back to the configured defaults when it cannot tell.
- Commands: editor.action.detectIndentation
- Menus: none found
- Keys: none found
- Settings: editor.detectIndentation
- Cursor: present
- Source: V\src\vs\editor\contrib\indentation\browser\indentation.ts (DetectIndentation); C bundle (editor.action.detectIndentation, label "Detect Indentation from Content")

### SS-3.10 Encoding readout
- Tag: both
- Evidence: source
- Boundary: no
- What: The item shows the file encoding of the active file editor as a short label such as "UTF-8" or "UTF-8 with BOM" (tooltip "Select Encoding"); it only appears once the editor has a loaded model and an encoding-capable input (id status.editor.encoding, name "Editor Encoding", priority 100.3).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.encoding
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (onEncodingChange); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (id status.editor.encoding, labels "Editor Encoding", "Select Encoding", setting files.encoding)

### SS-3.11 Open the encoding picker
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the encoding item (or running Change File Encoding) asks "Select Action" with Reopen with Encoding or Save with Encoding; an untitled file goes straight to Save with Encoding, a read-only file goes straight to Reopen with Encoding, and a non-file editor says "No file active at this time".
- Commands: workbench.action.editor.changeEncoding
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (ChangeEncodingAction); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (registerAction2); C bundle (workbench.action.editor.changeEncoding, label "Change File Encoding", "Select Action")

### SS-3.12 Reopen a file with a different encoding
- Tag: both
- Evidence: source
- Boundary: no
- What: Reopen with Encoding lists every decodable encoding (the configured one first, plus a "Guessed from content" row at the top when detection finds a different one) and rereads the file with the pick; on a file with unsaved changes stock VS Code first asks "Do you want to revert the active text editor and reopen with a different encoding?" with a "Discard Changes and Reopen" button, and Cursor skips that question and goes straight to the change.
- Commands: workbench.action.editor.changeEncoding
- Menus: none found
- Keys: none found
- Settings: files.encoding; files.autoGuessEncoding; files.candidateGuessEncodings
- Cursor: changed - no discard-changes confirm for an editor with unsaved changes
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (reopenWithEncodingPick, reopenWithEncodingWarning); C bundle (module vs/workbench/browser/parts/editor/editorStatus keys reopenWithEncoding and guessedEncoding present, keys reopenWithEncodingWarning, reopenWithEncodingDetail and reopen absent from the NLS files, and the compiled pick handler calls setEncoding right after the pick)

### SS-3.13 Save a file with a different encoding
- Tag: both
- Evidence: source
- Boundary: no
- What: Save with Encoding lists every encoding the file can be written in, including encode-only ones that Reopen hides, and saves the current text using the pick; the placeholder reads "Select File Encoding to Save with".
- Commands: workbench.action.editor.changeEncoding
- Menus: none found
- Keys: none found
- Settings: files.encoding
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (saveWithEncodingPick, EncodingMode.Encode); C bundle (labels "Save with Encoding", "Select File Encoding to Save with")

### SS-3.14 End of line readout
- Tag: both
- Evidence: source
- Boundary: no
- What: The item shows the line ending of the active text editor as "LF" or "CRLF" (tooltip "Select End of Line Sequence") and is hidden for read-only editors (id status.editor.eol, name "Editor End of Line", priority 100.2).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.eol
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (onEOLChange); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (id status.editor.eol, labels "LF", "CRLF", "Select End of Line Sequence", setting files.eol)

### SS-3.15 Change the end of line sequence
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the end of line item (or running Change End of Line Sequence) opens a LF and CRLF pick with the current one preselected and converts the whole file as one undo step; read-only editors show "The active code editor is read-only.".
- Commands: workbench.action.editor.changeEOL
- Menus: none found
- Keys: none found
- Settings: files.eol
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (ChangeEOLAction); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts; C bundle (workbench.action.editor.changeEOL, label "Change End of Line Sequence")

### SS-3.16 Language readout
- Tag: both
- Evidence: source
- Boundary: no
- What: The item shows the language name of the active text editor, such as "TypeScript" or "Plain Text" (tooltip "Select Language Mode"); it appears for editors that support a language mode and not for binary or other non-text editors (id status.editor.mode, name "Editor Language", priority 100.1).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.associations
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (onLanguageChange); V\src\vs\platform\files\common\files.ts (FILES_ASSOCIATIONS_CONFIG); C bundle (id status.editor.mode, labels "Editor Language", "Select Language Mode", setting files.associations)

### SS-3.17 Change the language mode from a picker
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the language item (or running Change Language Mode) opens "Select Language Mode" with every registered language, the current one marked "Configured Language", and sets the language of the active editor from the pick; the command also takes a language name as an argument, and the empty editor hint offers the same picker.
- Commands: workbench.action.editor.changeLanguageMode
- Menus: none found
- Keys: Ctrl+K M (not while a notebook editor has focus)
- Settings: none found
- Cursor: changed - the bundle gives this chord its own mac keybinding entry that V does not have (value not decoded)
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (ChangeLanguageAction); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts; V\src\vs\workbench\contrib\codeEditor\browser\emptyTextEditorHint\emptyTextEditorHint.ts; C bundle (workbench.action.editor.changeLanguageMode, label "Change Language Mode", keybinding block with a separate mac entry)

### SS-3.18 Auto Detect the language from the picker
- Tag: both
- Evidence: source
- Boundary: no
- What: When the file is not empty the language picker starts with an "Auto Detect" row that first guesses from the file name or first line and, if that gives nothing, asks the language detection model to guess from the content.
- Commands: workbench.action.editor.changeLanguageMode
- Menus: none found
- Keys: none found
- Settings: workbench.editor.languageDetection; workbench.editor.historyBasedLanguageDetection; workbench.editor.preferHistoryBasedLanguageDetection
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (autoDetectLanguage); V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (label "Auto Detect", settings workbench.editor.languageDetection, workbench.editor.historyBasedLanguageDetection, workbench.editor.preferHistoryBasedLanguageDetection)

### SS-3.19 Detect the language from the content with a command
- Tag: both
- Evidence: source
- Boundary: no
- What: The Detect Language from Content command sets the language of the active text editor to what the detection model guesses, or raises a warning "Unable to detect editor language" when it cannot; it does not run in an editable notebook.
- Commands: editor.detectLanguage
- Menus: none found
- Keys: Alt+Shift+D (editor text focus)
- Settings: workbench.editor.languageDetection
- Cursor: present
- Source: V\src\vs\workbench\contrib\languageDetection\browser\languageDetection.contribution.ts; C bundle (editor.detectLanguage, label "Detect Language from Content", same Alt+Shift+D key)

### SS-3.20 Accept the detected language with the lightbulb
- Tag: both
- Evidence: source
- Boundary: no
- What: In an untitled editor whose detected language differs from the current one, a lightbulb item appears next to the language item (tooltip "Accept Detected Language: {name}"); clicking it switches the editor to the detected language (id status.languageDetectionStatus, name "Language Detection").
- Commands: editor.detectLanguage
- Menus: none found
- Keys: Alt+Shift+D (shown in the tooltip)
- Settings: workbench.editor.languageDetectionHints; workbench.editor.languageDetection
- Cursor: present
- Source: V\src\vs\workbench\contrib\languageDetection\browser\languageDetection.contribution.ts (LanguageDetectionStatusContribution, compact pair next to status.editor.mode); C bundle (id status.languageDetectionStatus, label "Accept Detected Language: {0}", setting workbench.editor.languageDetectionHints)

### SS-3.21 Configure a file association for a file type
- Tag: both
- Evidence: source
- Boundary: no
- What: For a file that has a path the language picker offers "Configure File Association for '{ext}'..."; picking a language writes it into files.associations as `*.ext` (or the whole file name for dotfiles and names without an extension), in user settings unless the workspace already sets that association.
- Commands: workbench.action.editor.changeLanguageMode
- Menus: none found
- Keys: none found
- Settings: files.associations
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (configureFileAssociation); C bundle (labels "Configure File Association for '{0}'...", "Select Language Mode to Associate with '{0}'", "Current Association", setting files.associations)

### SS-3.22 Open language based settings from the picker
- Tag: both
- Evidence: source
- Boundary: no
- What: For a file that has a path the language picker offers "Configure '{language}' language based settings..." which opens the user settings file in JSON at the `[language]` block, ready to edit.
- Commands: workbench.action.editor.changeLanguageMode
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (configureLanguageSettings, openUserSettings); C bundle (label "Configure '{0}' language based settings...")

### SS-3.23 OVR overtype indicator and toggle
- Tag: both
- Evidence: source
- Boundary: no
- What: While overtype mode is on a prominent "OVR" item appears (tooltip "Enable Insert Mode") and clicking it, or pressing Insert, switches back to insert mode; the same command switches into overtype (id status.editor.inputMode, priority 100.6).
- Commands: editor.action.toggleOvertypeInsertMode
- Menus: none found
- Keys: Insert (mac Alt+Cmd+O)
- Settings: editor.overtypeCursorStyle; editor.overtypeOnPaste
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (StatusInputMode, updateInputModeElement); V\src\vs\workbench\contrib\codeEditor\browser\toggleOvertype.ts; V\src\vs\editor\common\config\editorOptions.ts; C bundle (id status.editor.inputMode, command editor.action.toggleOvertypeInsertMode with the same Insert and mac Alt+Cmd+O keys, label "OVR", option overtypeOnPaste)

### SS-3.24 Tab Moves Focus indicator and toggle
- Tag: both
- Evidence: source
- Boundary: no
- What: When the Tab key is set to move focus instead of typing a tab, a prominent "Tab Moves Focus" item appears (tooltip "Disable Accessibility Mode"); clicking it or running Toggle Tab Key Moves Focus switches back and announces the new state to screen readers (id status.editor.tabFocusMode, name "Accessibility Mode", priority 100.7).
- Commands: editor.action.toggleTabFocusMode
- Menus: none found
- Keys: Ctrl+M (mac Ctrl+Shift+M)
- Settings: editor.tabFocusMode
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (TabFocusMode, updateTabFocusModeElement); V\src\vs\editor\contrib\toggleTabFocusMode\browser\toggleTabFocusMode.ts; V\src\vs\editor\common\config\editorOptions.ts; C bundle (id status.editor.tabFocusMode, command editor.action.toggleTabFocusMode with the same Ctrl+M and mac Ctrl+Shift+M keys, labels "Tab Moves Focus", "Accessibility Mode")

### SS-3.25 Column Selection indicator and toggle
- Tag: both
- Evidence: source
- Boundary: no
- What: While column selection mode is on a prominent "Column Selection" item appears (tooltip "Disable Column Selection Mode") and clicking it turns the mode off; the Selection menu row "Column Selection Mode" (checked while on) and the command do the same (id status.editor.columnSelectionMode, priority 100.8).
- Commands: editor.action.toggleColumnSelection
- Menus: MenubarSelectionMenu
- Keys: none found
- Settings: editor.columnSelection
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (updateColumnSelectionModeElement); V\src\vs\workbench\contrib\codeEditor\browser\toggleColumnSelection.ts (menu group 4_config, order 2); C bundle (id status.editor.columnSelectionMode, editor.action.toggleColumnSelection with menu MenubarSelectionMenu, labels "Column Selection", "Toggle Column Selection Mode")

### SS-3.26 Screen Reader Optimized indicator
- Tag: both
- Evidence: source
- Boundary: no
- What: When the editor runs in screen reader optimized mode a prominent "Screen Reader Optimized" item appears in every window; clicking it reopens the "Screen reader usage detected" prompt with Yes, No and Learn More, and Yes or No writes editor.accessibilitySupport to on or off (id status.editor.screenReaderMode, name "Screen Reader Mode", priority 100.6).
- Commands: showEditorScreenReaderNotification
- Menus: none found
- Keys: none found
- Settings: editor.accessibilitySupport; editor.accessibilityPageSize
- Cursor: present
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibilityStatus.ts; C bundle (module vs/workbench/contrib/accessibility/browser/accessibilityStatus, id status.editor.screenReaderMode, command showEditorScreenReaderNotification, label "Screen Reader Optimized")

### SS-3.27 Scrolling Locked indicator and toggle
- Tag: both
- Evidence: source
- Boundary: no
- What: While scrolling is synchronized across editors a prominent "Scrolling Locked" item appears in every window (tooltip "Lock Scrolling Enabled", priority 102) and clicking it switches the lock off; the commands switch it on, or hold it only while their key is down.
- Commands: workbench.action.toggleLockedScrolling; workbench.action.holdLockedScrolling
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\scrollLocking\browser\scrollLocking.ts (id status.scrollLockingEnabled); C bundle (id status.scrollLockingEnabled, commands workbench.action.toggleLockedScrolling and workbench.action.holdLockedScrolling, labels "Scrolling Locked", "Toggle Locked Scrolling Across Editors")

### SS-3.28 File Information for binary files
- Tag: both
- Evidence: source
- Boundary: no
- What: When a binary file is open in the binary file viewer (or its side by side diff), an item shows the file size as text such as "1.2 KB" (tooltip "File Information"); nothing shows for text editors (id status.editor.info, priority 100).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (onMetadataChange); V\src\vs\workbench\browser\parts\editor\binaryEditor.ts (getMetadata from ByteSize.formatSize); C bundle (id status.editor.info, label "File Information")

### SS-3.29 Language status item and its hover list
- Tag: both
- Evidence: source
- Boundary: no
- What: Left of the language item a single bracket icon (a plain bracket, a bracket with a dot for a warning, a bracket with an error mark for an error, or a spinner while something is busy) stands for every language status row of the active file; hover or click opens a list where each row has a severity icon, label, detail text with links, an optional command link and a pin button, and it sits as a compact pair with the language item (id status.languageStatus, name "Editor Language Status"). Rows come from core (see SS-3.31) and from built-in language extensions such as the TypeScript version and IntelliSense rows, the JSON schema and outline rows and the Markdown link validation row.
- Commands: statusBar.entry.showTooltip
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts; V\src\vs\workbench\services\statusbar\browser\statusbar.ts (ShowTooltipCommand); V\extensions\typescript-language-features\src\ui\versionStatus.ts; V\extensions\json-language-features\client\src\languageStatus.ts; V\extensions\markdown-language-features\src\languageFeatures\diagnostics.ts; C bundle (module vs/workbench/contrib/languageStatus/browser/languageStatus, id status.languageStatus, label "Editor Language Status", language status ids typescript.version, typescript.projectStatus, json.projectStatus, json.documentSymbolsStatus and markdownStatus in the built-in extension bundles)

### SS-3.30 Pin a language status row to the status bar
- Tag: both
- Evidence: source
- Boundary: no
- What: The pin button on a row ("Add to Status Bar") makes that row its own item right of the language item with its short label, tooltip and command, colored as warning or error by severity; the button then reads "Remove from Status Bar", and hiding that item from the bar's right-click list also unpins it (remembered per profile).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts (_dedicated, storage key languageStatus.dedicated, _asStatusbarEntry); C bundle (labels "Add to Status Bar", "Remove from Status Bar", "{0} (Language Status)")

### SS-3.31 Core rows in the language status list
- Tag: both
- Evidence: source
- Boundary: no
- What: Core adds its own rows: "Formatter Conflicts" (error, label "Formatting", with a "Configure..." link) when a language has several formatters and no default or the chosen one cannot format it, and "Folding Status" or "Color Decorator Status" saying "only {n} shown for performance reasons" with a Configure link that opens the setting.
- Commands: workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: editor.defaultFormatter; editor.foldingMaximumRegions; editor.colorDecoratorsLimit
- Cursor: present
- Source: V\src\vs\workbench\contrib\format\browser\formatActionsMultiple.ts (id formatter.conflict); V\src\vs\workbench\contrib\limitIndicator\browser\limitIndicator.contribution.ts (ids foldingLimitInfo, decoratorsLimitInfo); C bundle (id formatter.conflict, ids foldingLimitInfo and decoratorsLimitInfo, labels "Formatter Conflicts", "Folding Status", "Color Decorator Status", settings editor.foldingMaximumRegions, editor.colorDecoratorsLimit)

### SS-3.32 Language status attention cues and the reset command
- Tag: both
- Evidence: source
- Boundary: no
- What: The bracket icon wiggles when the status changes until the person has opened its hover three times, and the whole item flashes when the top row is a warning or error; the command "Reset Language Status Interaction Counter" (View category, id spelled editor.inlayHints.Reset) starts the wiggle teaching over.
- Commands: editor.inlayHints.Reset
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts (StoredCounter, wiggle and flash, ResetAction); C bundle (editor.inlayHints.Reset, label "Reset Language Status Interaction Counter")

### SS-3.33 Notebook kernel item
- Tag: both
- Evidence: source
- Boundary: no
- What: In a notebook the right side shows the selected or only suggested kernel as "{kernel name}" with a kernel icon (a suggested one says "(suggestion)" in its tooltip and is selected for real once the person edits or hovers), or a prominent "Select Kernel" when several kernels exist and none is chosen; clicking runs the kernel picker, and nothing shows when no kernel exists or while the notebook toolbar already carries the kernel button (setting notebook.globalToolbar, default on) (ids _notebook.selectKernel, priority 10, names "Notebook Kernel Info" and "Notebook Kernel Selection"). The picker itself is ED-7.19.
- Commands: _notebook.selectKernel
- Menus: none found
- Keys: none found
- Settings: notebook.globalToolbar
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\editorStatusBar\editorStatusBar.ts (KernelStatus, ImplictKernelSelector); V\src\vs\workbench\contrib\notebook\browser\viewParts\notebookKernelView.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts (notebook.globalToolbar default true); C bundle (_notebook.selectKernel, labels "Notebook Kernel Info", "Notebook Kernel Selection", "Select Kernel", "{0} (suggestion)", setting notebook.globalToolbar)

### SS-3.34 Notebook active cell indicator
- Tag: both
- Evidence: source
- Boundary: no
- What: In a notebook the item shows "Cell {n} of {total}" for the focused cell, or "Cell {n} ({m} selected)" when several cells are selected, and clicking it scrolls the active cell to the middle of the editor (id notebook.activeCellStatus, name "Notebook Editor Selections", priority 100).
- Commands: notebook.centerActiveCell
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\editorStatusBar\editorStatusBar.ts (ActiveCellStatus); V\src\vs\workbench\contrib\notebook\browser\contrib\navigation\arrow.ts (CENTER_ACTIVE_CELL); C bundle (id notebook.activeCellStatus, command notebook.centerActiveCell, labels "Cell {0} of {1}", "Cell {0} ({1} selected)")

### SS-3.35 Notebook indentation item
- Tag: both
- Evidence: source
- Boundary: no
- What: In a notebook the indentation item shows the active cell's "Spaces: {n}" or "Tab Size: {n}" (tooltip "Select Indentation") and clicking it opens a notebook version of the indentation picker with Indent Using Tabs, Indent Using Spaces, Change Tab Display Size and the two Convert Indentation rows, applied to the notebook's cells; it needs an editable notebook and cell (id notebook.status.indentation, name "Notebook Indentation", priority 100.4).
- Commands: notebook.selectIndentation; notebook.action.indentUsingTabs; notebook.action.indentUsingSpaces; notebook.action.changeTabDisplaySize; notebook.action.convertIndentationToSpaces; notebook.action.convertIndentationToTabs
- Menus: none found
- Keys: none found
- Settings: notebook.editorOptionsCustomizations; editor.insertSpaces; editor.tabSize; editor.indentSize
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\editorStatusBar\editorStatusBar.ts (NotebookIndentationStatus); V\src\vs\workbench\contrib\notebook\browser\controller\editActions.ts (SelectNotebookIndentation); V\src\vs\workbench\contrib\notebook\browser\controller\notebookIndentationActions.ts; C bundle (id notebook.status.indentation, commands notebook.selectIndentation and the five notebook.action indentation ids, setting notebook.editorOptionsCustomizations)

### SS-3.36 Performance Trace recording item
- Tag: vscode
- Evidence: source
- Boundary: no
- What: After Developer: Start Tracing or Start Heap Tracing a red "Recording trace (click to stop)" item appears at the far left end of the left side (tooltip "Click to stop recording"); clicking it runs Stop Tracing, which shows a "Creating trace file..." progress dialog and writes the trace (id status.tracing, name "Performance Trace", kind error, desktop only).
- Commands: workbench.action.startTracing; workbench.action.startHeapTracing; workbench.action.stopTracing
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - no status item and no Start Tracing or Start Heap Tracing; only Stop Tracing remains, in an older form that offers to relaunch with the trace launch argument
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts (startTracingSession, StartTracing, StartHeapTracing, StopTracing); C bundle (workbench.action.stopTracing in module vs/workbench/contrib/issue/electron-sandbox/process.contribution with keys stopTracing.message (asks to relaunch with the trace launch argument), stopTracing.button and stopTracing.title; 0 hits for status.tracing, workbench.action.startTracing, workbench.action.startHeapTracing, "Recording trace (click to stop)", "Performance Trace")

### SS-3.37 Metered Connection item
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When the network is treated as metered an icon-only radio tower item appears at the far right end of the right side in every window (tooltip says updates, Settings Sync and automatic Git operations are paused); clicking it opens the Configure Metered Connection pick of Auto, On and Off (id workbench.contrib.meteredConnectionStatus, name "Metered Connection").
- Commands: workbench.action.configureMeteredConnection
- Menus: none found
- Keys: none found
- Settings: network.meteredConnection
- Cursor: missing
- Source: V\src\vs\workbench\contrib\meteredConnection\browser\meteredConnectionStatus.ts; V\src\vs\workbench\contrib\meteredConnection\browser\meteredConnection.contribution.ts; V\src\vs\platform\meteredConnection\common\meteredConnection.config.contribution.ts; C bundle (0 hits for workbench.action.configureMeteredConnection, network.meteredConnection and the label "Metered Connection")

### SS-3.38 Keyboard Layout item
- Tag: vscode
- Evidence: source
- Boundary: no
- What: In the web build only, an item "Layout: {name}" shows the detected keyboard layout on the right side and clicking it opens Change Keyboard Layout, with Auto Detect, a list of layouts and a Configure Keyboard Layout row that opens a keyboard layout file (id status.workbench.keyboardLayout, name "Keyboard Layout"); the desktop build never loads it.
- Commands: workbench.action.openKeyboardLayoutPicker
- Menus: none found
- Keys: none found
- Settings: keyboard.layout
- Cursor: missing
- Source: V\src\vs\workbench\contrib\preferences\browser\keyboardLayoutPicker.ts; V\src\vs\workbench\workbench.web.main.ts (the only import, none in workbench.desktop.main.ts); V\src\vs\workbench\contrib\preferences\common\preferences.ts (KEYBOARD_LAYOUT_OPEN_PICKER); C bundle (0 hits for status.workbench.keyboardLayout, workbench.action.openKeyboardLayoutPicker and the labels "Layout: {0}" and "Change Keyboard Layout")

### SS-3.39 Rendered Diff warning item
- Tag: vscode
- Evidence: source
- Boundary: no
- What: While a rendered Markdown diff preview is the active editor a warning-colored "Rendered Diff" item appears at priority 100 on the right (tooltip says rendered diffs may hide important changes); clicking it shows a warning that offers "Don't Show Again" (remembered per workspace), and the same warning shows once on first use (id markdown.renderedDiffWarning, name "Rendered Markdown Diff Warning").
- Commands: _markdown.preview.showRenderedDiffWarning
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\extensions\markdown-language-features\src\preview\renderedDiffWarning.ts; V\extensions\markdown-language-features\src\preview\previewManager.ts; C bundle (0 hits for markdown.renderedDiffWarning, renderedDiffWarning and "Rendered Diff" in the workbench bundle and in the markdown-language-features dist and package files)

### SS-3.40 Image Zoom item in the Image Preview
- Tag: both
- Evidence: source
- Boundary: no
- What: While an image is the active Image Preview editor a right-side item shows the zoom level as a percent such as "100%", or "Whole Image" when the picture is fit to the window (id status.imagePreview.zoom, name "Image Zoom", priority 102); clicking it opens "Select zoom level" with 1000%, 500%, 200%, 100%, 50%, 20% and Whole Image and the preview rescales to the pick, and the item hides when another editor is active. ED-10.1 covers the zoom gestures themselves.
- Commands: _imagePreview.selectZoomLevel
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\media-preview\src\imagePreview\zoomStatusBarEntry.ts; V\extensions\media-preview\src\imagePreview\index.ts (updateState shows and hides the entry); V\extensions\media-preview\src\ownedStatusBarEntry.ts; C\extensions\media-preview\dist\extension.js (id status.imagePreview.zoom at priority 102, command _imagePreview.selectZoomLevel, scale list 10, 5, 2, 1, .5, .2, fit, labels "Select zoom level" and "Whole Image")

### SS-3.41 Image Size item in the Image Preview
- Tag: both
- Evidence: source
- Boundary: no
- What: While an image is the active Image Preview editor a right-side item shows the picture's own pixel size as "{width}x{height}", for example "1920x1080" (id status.imagePreview.size, name "Image Size", priority 101, just left of the binary size item); it has no click action, it is empty until the image has loaded, and it hides when the preview is not the active editor.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\media-preview\src\imagePreview\sizeStatusBarEntry.ts; V\extensions\media-preview\media\imagePreview.js (posts the size message built from naturalWidth and naturalHeight); V\extensions\media-preview\src\imagePreview\index.ts; C\extensions\media-preview\dist\extension.js (id status.imagePreview.size at priority 101, label "Image Size"); C\extensions\media-preview\media\imagePreview.js (same naturalWidth x naturalHeight message)

### SS-3.42 File size item in the image, audio and video previews
- Tag: both
- Evidence: source
- Boundary: no
- What: While an image, audio or video preview is the active editor a right-side item shows the file size on disk as "{n}B", "{n.nn}KB", "{n.nn}MB", "{n.nn}GB" or "{n.nn}TB", for example "12.34KB" (id status.imagePreview.binarySize, name "Image Binary Size", priority 100); it has no click action, it refreshes when the file changes on disk, and it hides when the preview is not the active editor. SS-3.28 is the different File Information item of the core binary file viewer.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\media-preview\src\binarySizeStatusBarEntry.ts; V\extensions\media-preview\src\mediaPreview.ts (updateBinarySize, updateState); V\extensions\media-preview\src\extension.ts (one entry shared by the image, audio and video support); C\extensions\media-preview\dist\extension.js (id status.imagePreview.binarySize at priority 100, labels "{0}KB" and "{0}MB", the entry handed to the audio and video providers)

### SS-3.43 TypeScript version row in the language status list
- Tag: both
- Evidence: source
- Boundary: no
- What: In a JavaScript, JavaScript React, TypeScript or TypeScript React file the language status list gets a row whose text is the display name of the TypeScript version the language server started with (the bundled one or a workspace one) and whose link "Select Version" (tooltip is the path of that version) opens the version picker; the row appears only once the server has started (id typescript.version, name "TypeScript Version", detail "TypeScript version"). It sits inside the SS-3.29 bracket item and can be pinned like any row (SS-3.30).
- Commands: typescript.selectTypeScriptVersion
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\typescript-language-features\src\ui\versionStatus.ts; V\extensions\typescript-language-features\src\configuration\languageIds.ts (jsTsLanguageModes); C\extensions\typescript-language-features\dist\extension.js (createLanguageStatusItem "typescript.version", labels "TypeScript Version" and "TypeScript version", "Select Version")

### SS-3.44 Pick the TypeScript version
- Tag: both
- Evidence: source
- Boundary: no
- What: The "Select Version" link, or the command TypeScript: Select TypeScript Version in the Command Palette (only for a file the TypeScript extension manages), opens a pick titled "Select the TypeScript version used for JavaScript and TypeScript language features" with "Use VS Code's Version" and its version, one "Use Workspace Version" row for each local TypeScript install it finds, "Use TypeScript Native Preview (Experimental)" only when that extension is installed, and a "Learn more about managing TypeScript versions" link; the active choice has a bullet, and a workspace version asks for Workspace Trust, remembers the choice for this workspace and writes the folder path into workspace settings before the server restarts on it.
- Commands: typescript.selectTypeScriptVersion
- Menus: commandPalette (when typescript.isManagedFile)
- Keys: none found
- Settings: js/ts.tsdk.path; typescript.tsdk; typescript.enablePromptUseWorkspaceTsdk
- Cursor: present
- Source: V\extensions\typescript-language-features\src\tsServer\versionManager.ts (promptUserForVersion, getBundledPickItem, getLocalPickItems, getNativePreviewPickItem, LearnMorePickItem); V\extensions\typescript-language-features\src\commands\selectTypeScriptVersion.ts; V\extensions\typescript-language-features\package.json (command, commandPalette entry, tsdk settings); C\extensions\typescript-language-features\dist\extension.js (typescript.selectTypeScriptVersion, "Use VS Code's Version", "Use Workspace Version", "Use TypeScript Native Preview (Experimental)", "Learn more about managing TypeScript versions"); C\extensions\typescript-language-features\package.json (js/ts.tsdk.path, typescript.tsdk, typescript.enablePromptUseWorkspaceTsdk)

### SS-3.45 JS/TS project config row in the language status list
- Tag: both
- Evidence: source
- Boundary: no
- What: In a JavaScript or TypeScript file the language status list gets an IntelliSense row (id typescript.projectStatus, name "JS/TS IntelliSense Status") that reads "Loading IntelliSense status" with a spinner while the server looks up the project, then the workspace-relative path of the tsconfig or jsconfig that owns the file with a link "Open Config File"; when no config owns the file it reads "No tsconfig" (or "No jsconfig" for a JavaScript file) with a link "Configure TSConfig" (or "Configure JSConfig") that opens tsconfig.json or jsconfig.json in the workspace root, and if it does not exist an untitled copy is filled with a starter config built from the implicit project settings plus an exclude of node_modules; with no folder open the row says "No opened folders" or "File is not part opened folders" and has no link.
- Commands: _typescript.openConfig; _typescript.createOrOpenConfig
- Menus: none found
- Keys: none found
- Settings: js/ts.implicitProjectConfig.module; js/ts.implicitProjectConfig.target; js/ts.implicitProjectConfig.checkJs; js/ts.implicitProjectConfig.experimentalDecorators; js/ts.implicitProjectConfig.strictNullChecks; js/ts.implicitProjectConfig.strictFunctionTypes; js/ts.implicitProjectConfig.strict
- Cursor: present
- Source: V\extensions\typescript-language-features\src\ui\intellisenseStatus.ts; V\extensions\typescript-language-features\src\tsconfig.ts (openOrCreateConfig, inferredProjectConfigSnippet, inferredProjectCompilerOptions); V\extensions\typescript-language-features\package.json (js/ts.implicitProjectConfig.* settings); C\extensions\typescript-language-features\dist\extension.js (createLanguageStatusItem "typescript.projectStatus", commands _typescript.openConfig and _typescript.createOrOpenConfig, labels "JS/TS IntelliSense Status", "Loading IntelliSense status", "No tsconfig", "No jsconfig", "Configure TSConfig", "Configure JSConfig", "Open Config File"); C\extensions\typescript-language-features\package.json (the same settings)

### SS-3.46 Partial mode warning row
- Tag: both
- Evidence: source
- Boundary: no
- What: When project-wide IntelliSense is not available for the open JavaScript or TypeScript file the same IntelliSense row turns into a warning that reads "Partial mode" with the detail "Project wide IntelliSense not available" and a link "Learn More" that opens the partial mode help page; on desktop this happens for files whose scheme is not a local file, an untitled file or a notebook cell, or when the syntax-only server is forced on, and being a warning it brings the warning bracket icon and the flash cue of SS-3.32.
- Commands: vscode.open
- Menus: none found
- Keys: none found
- Settings: js/ts.tsserver.useSyntaxServer; typescript.tsserver.useSyntaxServer
- Cursor: present
- Source: V\extensions\typescript-language-features\src\ui\intellisenseStatus.ts (IntellisenseState.SyntaxOnly); V\extensions\typescript-language-features\src\typescriptServiceClient.ts (capabilities, hasCapabilityForResource); V\extensions\typescript-language-features\src\configuration\fileSchemes.ts (getSemanticSupportedSchemes); C\extensions\typescript-language-features\dist\extension.js (labels "Partial mode" and "Project wide IntelliSense not available"); C\extensions\typescript-language-features\package.json (js/ts.tsserver.useSyntaxServer)

### SS-3.47 Configure Excludes item for a project that is too large
- Tag: both
- Evidence: source
- Boundary: no
- What: When the TypeScript server turns off project-wide language features because a project holds too many files, a right-side item "Configure Excludes" appears in a green text color (id status.typescript.exclude, name "TypeScript: Configure Excludes", priority 98) with a tooltip telling the person to exclude large folders they do not work on; clicking it opens the project's tsconfig or jsconfig (or creates one in the workspace root) and shows the same hint as an information message, the same event also raises a warning message with a Configure Excludes button, and the item hides again once the server reports the language service enabled.
- Commands: js.projectStatus.command
- Menus: none found
- Keys: none found
- Settings: js/ts.implicitProjectConfig.module; js/ts.implicitProjectConfig.target; js/ts.implicitProjectConfig.checkJs; js/ts.implicitProjectConfig.experimentalDecorators; js/ts.implicitProjectConfig.strictNullChecks; js/ts.implicitProjectConfig.strictFunctionTypes; js/ts.implicitProjectConfig.strict
- Cursor: present
- Source: V\extensions\typescript-language-features\src\ui\largeProjectStatus.ts (ExcludeHintItem, onConfigureExcludesSelected); V\extensions\typescript-language-features\src\typescriptServiceClient.ts (projectLanguageServiceState event); C\extensions\typescript-language-features\dist\extension.js (createStatusBarItem "status.typescript.exclude" at priority 98, command js.projectStatus.command, labels "TypeScript: Configure Excludes" and "Configure Excludes")

### SS-3.48 JSON schema validation row
- Tag: both
- Evidence: source
- Boundary: no
- What: In a file of a language the JSON server handles (JSON, JSON with Comments and the like) the language status list gets a validation row (id json.projectStatus, name "JSON Validation Status") that reads "No schema validation" with the detail "no JSON schema configured", or "Schema validated" with the detail "JSON schema configured" for one schema or "multiple JSON schemas configured" for several; it shows a spinner and "Loading JSON info" while it works, "Unable to compute used schemas" with the reason when the lookup fails, and a link "Show Schemas" that opens the list in SS-3.49.
- Commands: _json.showAssociatedSchemaList
- Menus: none found
- Keys: none found
- Settings: json.validate.enable; json.schemas
- Cursor: present
- Source: V\extensions\json-language-features\client\src\languageStatus.ts (createLanguageStatusItem); V\extensions\json-language-features\client\src\jsonClient.ts (registration, SettingIds); C\extensions\json-language-features\client\dist\node\jsonClientMain.js (json.projectStatus, labels "JSON Validation Status", "No schema validation", "Schema validated", "no JSON schema configured", "multiple JSON schemas configured", "Show Schemas")

### SS-3.49 Show the schemas used for a JSON file
- Tag: both
- Evidence: source
- Boundary: no
- What: The "Show Schemas" link opens a pick titled "Select the schema to use for {file}" that lists each schema applied to the file, marked "Configured by extension: {id}" with an Open Extension button, or "Configured in user settings" or "Configured in workspace settings" with an Open Settings button that jumps to json.schemas, or just the schema URL; with none it shows "No schema configured for this file", a last row "Learn more about JSON schema configuration..." links to the docs, and picking a schema opens that schema file.
- Commands: _json.showAssociatedSchemaList; workbench.action.openSettingsJson; workbench.action.openWorkspaceSettingsFile; vscode.open
- Menus: none found
- Keys: none found
- Settings: json.schemas
- Cursor: present
- Source: V\extensions\json-language-features\client\src\languageStatus.ts (showSchemaList, getExtensionSchemaAssociations, getSettingsSchemaAssociations); C\extensions\json-language-features\client\dist\node\jsonClientMain.js (command _json.showAssociatedSchemaList, labels "Select the schema to use for {0}", "No schema configured for this file", "Learn more about JSON schema configuration")

### SS-3.50 JSON outline limit warning row
- Tag: both
- Evidence: source
- Boundary: no
- What: When a JSON file has more symbols than the limit, a warning row "Outline" appears in the language status list with the detail "only {n} document symbols shown for performance reasons" and a link "Configure" that opens Settings on json.maxItemsComputed (default 5000); it shows only for the active file and goes away when the file drops under the limit (id json.documentSymbolsStatus, name "JSON Outline Status"). SS-3.31 lists the matching rows that core adds for folding and color decorators.
- Commands: workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: json.maxItemsComputed
- Cursor: present
- Source: V\extensions\json-language-features\client\src\languageStatus.ts (createLimitStatusItem, createDocumentSymbolsLimitItem); V\extensions\json-language-features\client\src\jsonClient.ts (documentSymbolsLimitStatusbarItem); V\extensions\json-language-features\package.json (json.maxItemsComputed default 5000); C\extensions\json-language-features\client\dist\node\jsonClientMain.js (json.documentSymbolsStatus, label "JSON Outline Status", the "only {0} document symbols shown for performance reasons" text, setting json.maxItemsComputed)

### SS-3.51 JSON schema download issue row
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When the schema of the active JSON file cannot be loaded the language status list gets an error row "Schema download issue" (id json.documentSymbolsStatus, shared with the outline row, name "JSON Outline Status") whose detail says why: "Workspace untrusted" with a link to Manage Workspace Trust, "Download disabled" with a link "Configure" that opens Settings on json.schemaDownload.enable, "Location untrusted" with a link to configure trusted domains, or "Unable to resolve schema" with a link "Retry"; being an error it brings the error bracket icon and the flash cue of SS-3.32.
- Commands: _json.retryResolveSchema; workbench.trust.manage; workbench.action.openSettings; _json.configureTrustedDomains
- Menus: none found
- Keys: none found
- Settings: json.schemaDownload.enable; json.schemaDownload.trustedDomains
- Cursor: missing - 0 hits in the Cursor JSON extension for "Schema download issue", "Download disabled", "Location untrusted", "Workspace untrusted", _json.configureTrustedDomains and the setting json.schemaDownload.trustedDomains; its older JSON client shows the alert item in SS-3.52 instead
- Source: V\extensions\json-language-features\client\src\languageStatus.ts (createSchemaLoadStatusItem, createSchemaLoadIssueItem); V\extensions\json-language-features\client\src\jsonClient.ts (schemaLoadStatusItem, CommandIds); V\extensions\json-language-features\package.json (json.schemaDownload.trustedDomains); C\extensions\json-language-features\client\dist\node\jsonClientMain.js and package.json (searched, none of the strings above)

### SS-3.52 JSON schema resolution error alert item
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: In Cursor's JSON client, when the active JSON file has a schema that cannot be loaded, an icon-only alert item appears on the right side (id status.json.resolveError, name "JSON: Schema Resolution Error", priority 0, so right of the editor items and left of the notifications bell); with downloads on its tooltip reads "Unable to resolve schema. Click to retry." and clicking retries the validation (the icon shows a watch while it runs and the item hides if the schema now loads), and with downloads off the tooltip reads "Downloading schemas is disabled. Click to configure." and clicking opens Settings on json.schemaDownload.enable. Stock 1.128.0 creates this item but never shows it, because it moved this job to the language status row in SS-3.51.
- Commands: _json.retryResolveSchema; workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: json.schemaDownload.enable
- Cursor: cursor only - older JSON client than V (the item is shown there, the language status row is absent)
- Source: C\extensions\json-language-features\client\dist\node\jsonClientMain.js (createStatusBarItem "status.json.resolveError" Right 0, show and hide on active editor change, tooltips "Unable to resolve schema. Click to retry." and "Downloading schemas is disabled. Click to configure.", command _json.retryResolveSchema, text alert icon and watch icon); V\extensions\json-language-features\client\src\jsonClient.ts (the item is created at the same id and priority and given text and tooltip only, with no show call anywhere in the client)

### SS-3.53 Markdown link validation row with an on and off link
- Tag: both
- Evidence: source
- Boundary: no
- What: In a Markdown file the language status list gets a row (id markdownStatus) that reads "Markdown link validation enabled" with a link "Disable" (tooltip "Disable validation of Markdown links"), or "Markdown link validation disabled" with a link "Enable" (tooltip "Enable validation of Markdown links"); clicking the link flips markdown.validate.enabled (off by default) and the row text updates as soon as that setting changes.
- Commands: _markdown.toggleValidation
- Menus: none found
- Keys: none found
- Settings: markdown.validate.enabled
- Cursor: present
- Source: V\extensions\markdown-language-features\src\languageFeatures\diagnostics.ts (registerMarkdownStatusItem); V\extensions\markdown-language-features\package.json (markdown.validate.enabled, default false); C\extensions\markdown-language-features\dist\extension.js (markdownStatus, command _markdown.toggleValidation, the four labels above); C\extensions\markdown-language-features\package.json (markdown.validate.enabled)

## Handed off

- Current Problem text on the left (statusbar.currentProblem, setting problems.showCurrentInStatus, code in editorStatus.ts) -> SS-5.
- Window Zoom (status.windowZoom) -> SS-5.
- Inline Suggestions row in the language status list (id inlineSuggestions) -> SS-6.
- Cursor Tab, Tab Stats, Agent Stats and the recent commit AI score (the Cursor-only items on the right side) -> SS-6. No Cursor-only editor-level item was found for SS-3.
- Git blame item (git.blame, right side, priority 200) -> SS-4.
- The Go to Line picker itself and Go to Offset (Cursor's Go to Line placeholder lacks the `::` offset and negative number help that V has) -> QA-3.
- The shared quick pick widget used by the indentation, encoding, end of line, language and keyboard layout pickers -> QA-6.
- The editor-only commands behind SS-3.6, SS-3.8 and SS-3.9 (Indent Using Spaces and Tabs, Change Tab Display Size, Convert Indentation, Detect Indentation) are the same editor actions ED-2.18 and ED-2.19 list; the Trim Trailing Whitespace row of the indentation picker -> ED-2.
- Select Notebook Kernel as an editor title and notebook toolbar button and the kernel picker -> ED-7.19.
- The Screen reader usage detected prompt (Yes, No, Learn More) that SS-3.26 opens -> NT-2.
- Hiding one item, the compact pairing of Language with its neighbours, and item colors for the prominent, warning and error kinds -> SS-1 and SS-2.
- The Debug Auto Attach item (status.debug.autoAttach, debug-auto-launch extension, left side) and the JavaScript debug profiling item (priority 500, shown while profiling) are debug state, not editor details -> SS-5 (its Unclear section already leaves both out as debug support).
- The Image Preview zoom gestures, copy and save, and the reopen commands stay in ED-10; only the three status items of that viewer were added here as SS-3.40 to SS-3.42 -> ED-10.

## Unclear

- Pass 1 corrections that affect this group: Performance Trace sits on the left side (not the right) and is a desktop-only developer item, and Cursor is not fully missing it because Stop Tracing exists in an older form (SS-3.36). Keyboard Layout is loaded only by the web workbench, so a desktop build lacking it is expected (SS-3.38). Pass 1 placed Metered Connection under SS-4 and Performance Trace under SS-5, and the user assigned both here.
- Default visibility on a fresh Cursor profile and what shows by default was not checked at runtime. Every Cursor value is from the bundle, the NLS files and the built-in extension bundles.
- The extra mac keybinding entry on Change Language Mode in the Cursor bundle (SS-3.17) was not decoded to a key chord, and the mac and non-mac keys of the other commands were compared only by presence of the command id and the visible keybinding block.
- Rows in the language status list that come from extensions (TypeScript version, JS/TS IntelliSense, JSON validation and outline, Markdown link validation) are named in SS-3.29 but are not features of their own because they belong to language support extensions.
- The language picker also has a "Search Marketplace Extensions for '{ext}'..." row (command workbench.action.showLanguageExtensions, present in the Cursor bundle). It opens the Extensions view, which is out of scope, so it is not listed.
- The media-preview built-in extension adds image status items (zoom, size, binary size) in both V and Cursor. It is a media support extension outside the brief's extension list, so not listed. In 1.144 those items move into core (see the pass 1 note).
- The Rendered Diff item (SS-3.39) is a right side item for the active editor that pass 1 did not assign to a group; it could also sit with the Markdown preview in ED-10.
- SS-3.6 to SS-3.9 repeat the commands ED-2.18 and ED-2.19 list. They are here because the person reaches them from the indentation item, so the parent may want to merge or cross-link them.
- The notebook kernel item stays hidden while notebook.globalToolbar is on, and it defaults to on in V. I did not check the Cursor default of that setting.
- In 1.144 (N) the metered connection item also gets a warning kind and a longer tooltip that adds inline completions and telemetry to the paused list. The other source files in this group are identical in V and N. This is a detail on SS-3.37, not a new feature.
- Overflow check on 2026-10-25 added 14 features (SS-3.40 to SS-3.53). It followed the owner's request to read the status bar items of the typescript, json, css, html, markdown, media-preview and git extensions, so the media-preview items and the language status rows of the TypeScript, JSON and Markdown extensions are now listed even though the lines above call them out of scope. css-language-features and html-language-features add no status bar item or language status row, and git adds only the blame item already in SS-4. In V the JSON extension never shows its status.json.resolveError item (SS-3.52 is Cursor only), and in 1.144 (N) the media-preview items move into core (mediaPreview/browser/mediaPreviewStatusBar.ts) with the same ids.
