Group: SS-3 Current editor and language details. Pass: 3. Date: 2026-10-25.

### SS-3.1
1. While a text editor is active the right side of the status bar shows "Ln {n}, Col {n}" for the cursor, and the numbers change as the cursor moves.
2. A tab character counts by its display width in the column number, so with a tab size of 4 a cursor just after one leading tab reads Col 5.
3. Hovering the item shows the tooltip "Go to Line/Column".
4. The item is hidden while a notebook cell editor is the active editor.

### SS-3.2
1. With text selected under a single cursor the item reads "Ln {n}, Col {n} ({n} selected)", counting the selected characters.
2. With several cursors and nothing selected the item reads "{n} selections".
3. With several cursors and some text selected the item reads "{n} selections ({n} characters selected)".
4. Clearing the selection or dropping back to one cursor returns the item to plain "Ln {n}, Col {n}".

### SS-3.3
1. Clicking the line and column item opens the Go to Line quick pick for the active editor, where typing a line number, and optionally a column after a colon, moves the cursor there.
2. Pressing `Ctrl+G` opens the same Go to Line quick pick (the key is `Ctrl+G` on macOS too).
3. The Go menu row "Go to Line/Column..." and the command `workbench.action.gotoLine` from the Command Palette open the same quick pick.

### SS-3.4
1. For a file indented with spaces where the indent size equals the tab size the item reads "Spaces: {n}".
2. For a file indented with spaces where the indent size differs from the tab size the item reads "Spaces: {n} (Tab Size: {m})".
3. For a file indented with tabs the item reads "Tab Size: {n}".
4. Hovering the item shows the tooltip "Select Indentation", and the item is hidden while a notebook cell editor is active.
5. Setting `editor.insertSpaces` chooses whether Tab inserts spaces or a tab character in files that have no detected style, with values on and off.
6. Settings `editor.tabSize` (a number from 1 to 100) and `editor.indentSize` (a number or `tabSize`) choose the starting tab width and indent width that the item shows.
7. Setting `editor.detectIndentation` chooses whether the style and size are taken from the file content when it opens, with values on and off, and the item shows what was detected.

### SS-3.5
1. Clicking the indentation item opens a quick pick with the placeholder "Select Action" for the active editor.
2. The first group is labelled "change view" and holds Indent Using Spaces, Indent Using Tabs and Change Tab Display Size.
3. The second group is labelled "convert file" and holds Detect Indentation from Content, Convert Indentation to Spaces, Convert Indentation to Tabs and Trim Trailing Whitespace.
4. Picking a row closes the quick pick, puts focus back in the editor and runs that action.
5. When the active editor is read-only the quick pick shows one row, "The active code editor is read-only."
6. When no text editor is active the quick pick shows one row, "No text editor active at this time".

### SS-3.6
1. Picking Indent Using Spaces opens a size list with the placeholder "Select Tab Size for Current File" and the rows 1 to 8.
2. Picking Indent Using Tabs opens the same size list of 1 to 8.
3. A size row is tagged "Configured Tab Size" when it matches both the configured and the current size, "Default Tab Size" when it matches only the configured one, and "Current Tab Size" when it matches only the current one.
4. The list opens with the file's current tab size highlighted.
5. Picking a size changes the tab size and indent size of that file and switches it to spaces or tabs as chosen, and the status bar indentation item updates at once.
6. The change applies to that file only, and the `editor.insertSpaces`, `editor.tabSize` and `editor.indentSize` settings are left as they were.
7. Running `editor.action.indentUsingSpaces` or `editor.action.indentUsingTabs` from the Command Palette opens the same size list.

### SS-3.7
1. Picking Change Tab Display Size opens the same size list of 1 to 8 with the placeholder "Select Tab Size for Current File".
2. Picking a size changes how many spaces wide a tab character is drawn in that file and nothing else.
3. The indent size and the spaces or tabs style of the file stay as they were, and the status bar item shows the new tab size.
4. Running `editor.action.changeTabDisplaySize` from the Command Palette opens the same list.

### SS-3.8
1. Picking Convert Indentation to Spaces rewrites the leading whitespace of every line in the file as spaces and switches the file's style to spaces.
2. Picking Convert Indentation to Tabs rewrites the leading whitespace of every line in the file as tabs and switches the file's style to tabs.
3. The selection stays in place after either conversion.
4. Both rows need a writable editor, and nothing happens in a read-only editor.
5. Running `editor.action.indentationToSpaces` or `editor.action.indentationToTabs` from the Command Palette does the same conversion.

### SS-3.9
1. Picking Detect Indentation from Content sets the file's indent style and size to match the indentation found in its lines, and the status bar item updates.
2. When the content gives no clear answer the file takes the configured defaults for spaces or tabs and for size.
3. Running `editor.action.detectIndentation` from the Command Palette does the same thing.
4. Setting `editor.detectIndentation` chooses whether this detection also runs on its own when a file opens, with values on and off.

### SS-3.10
1. While a file editor with a loaded model is active the item shows the file's encoding as a short label such as "UTF-8".
2. A file that starts with a byte order mark shows "UTF-8 with BOM" in the item.
3. Hovering the item shows the tooltip "Select Encoding".
4. The item does not appear for editors that have no file encoding, such as a non-file editor, and it updates when the encoding changes.
5. Setting `files.encoding` chooses the default encoding used to read and write files, default `utf8`, and it can be set per language.

### SS-3.11
1. Clicking the encoding item opens a quick pick with the placeholder "Select Action" offering Reopen with Encoding and Save with Encoding.
2. Running Change File Encoding (`workbench.action.editor.changeEncoding`) from the Command Palette opens the same quick pick.
3. In an untitled file the click skips the action quick pick and goes straight to the Save with Encoding list.
4. In a read-only file the click skips the action quick pick and goes straight to the Reopen with Encoding list.
5. In an editor that is not a file the quick pick shows one row, "No file active at this time", and with no text editor it shows "No text editor active at this time".

### SS-3.12
1. Picking Reopen with Encoding opens a list with the placeholder "Select File Encoding to Reopen File" showing each encoding the file can be read with, with the encoding code beside its name.
2. The encoding set in `files.encoding` is the first row and the file's current encoding starts highlighted.
3. When content detection finds an encoding other than the configured one, a row for it tagged "Guessed from content" sits at the very top above a separator.
4. Picking an encoding rereads the file with it and the status bar encoding item changes to match.
5. Setting `files.autoGuessEncoding` chooses whether files try to guess their encoding when they open, with values on and off and a default of off.
6. Setting `files.candidateGuessEncodings` lists, in order, the encodings the guess may choose from, and it starts empty.
7. Stock VS Code asks "Do you want to revert the active text editor and reopen with a different encoding?" with the detail "This will discard any unsaved changes." and a "Discard Changes and Reopen" button when the file has unsaved changes. [stock only]
8. Cursor skips that question and reopens the file with the picked encoding straight away, even with unsaved changes. [cursor only]

### SS-3.13
1. Picking Save with Encoding opens a list with the placeholder "Select File Encoding to Save with" and the file's current encoding highlighted.
2. The list also holds the encodings that can only be written, which the Reopen list hides.
3. Picking an encoding saves the current text in that encoding and the status bar encoding item changes to match.
4. The encoding set in `files.encoding` is the first row of the list.

### SS-3.14
1. While a text editor is active the item shows the line ending as "LF" or "CRLF".
2. Hovering the item shows the tooltip "Select End of Line Sequence".
3. The item is hidden for a read-only editor.
4. The item changes right after the line ending of the file is converted.
5. Setting `files.eol` chooses the line ending for new files, with values `\n`, `\r\n` and `auto` (the system's own), default `auto`.

### SS-3.15
1. Clicking the end of line item opens a quick pick with the placeholder "Select End of Line Sequence" and the rows LF and CRLF, with the file's current one highlighted.
2. Running Change End of Line Sequence (`workbench.action.editor.changeEOL`) from the Command Palette opens the same quick pick.
3. Picking a row converts every line ending in the file at once, and one undo puts them all back.
4. In a read-only editor the quick pick shows one row, "The active code editor is read-only."
5. With no text editor active the quick pick shows one row, "No text editor active at this time".

### SS-3.16
1. While a text editor is active the item shows the name of the file's language, such as "TypeScript" or "Plain Text".
2. Hovering the item shows the tooltip "Select Language Mode".
3. The item does not appear for binary files or other editors that have no language mode, and it changes when the language changes.
4. Setting `files.associations` maps file name patterns to languages, so files matching a pattern show that language in the item.

### SS-3.17
1. Clicking the language item opens a quick pick with the placeholder "Select Language Mode" listing every registered language, sorted by name, with its identifier in brackets under a "languages (identifier)" heading.
2. The row of the file's current language is marked "Configured Language".
3. Picking a language switches the active editor to it, the status item shows the new name, and focus returns to the editor.
4. Pressing `Ctrl+K` then `M` opens the same quick pick, except while a notebook editor has focus.
5. Running Change Language Mode (`workbench.action.editor.changeLanguageMode`) from the Command Palette opens the same quick pick.
6. The "select a language" link in the hint shown in an empty editor opens the same quick pick.
7. Running the command with a language name as its argument switches to that language without showing the quick pick.
8. With no text editor active the quick pick shows one row, "No text editor active at this time".
9. Cursor registers this chord with its own separate macOS keybinding entry that stock VS Code does not have. [cursor only]

### SS-3.18
1. When the file has any content the language quick pick starts with an "Auto Detect" row, and in an empty file the row is missing.
2. Picking Auto Detect first sets the language from the file name or the first line when that identifies one.
3. When the file name and first line identify nothing, Auto Detect sets the language from a guess made on the file content.
4. When no guess comes back Auto Detect leaves the language as it was.
5. Setting `workbench.editor.languageDetection` chooses whether the language of a text editor is detected on its own unless it was set from the picker, with values on and off, default on, and it can be set per language.
6. Setting `workbench.editor.historyBasedLanguageDetection` chooses whether recently opened languages are favored when detecting, default on.
7. Setting `workbench.editor.preferHistoryBasedLanguageDetection` chooses whether the history-aware detection takes precedence, default off.

### SS-3.19
1. Running Detect Language from Content (`editor.detectLanguage`) from the Command Palette sets the active text editor's language to the detected one and the language item updates.
2. Pressing `Alt+Shift+D` with focus in the editor text does the same.
3. When nothing can be detected a warning notification reads "Unable to detect editor language" and the language stays as it was.
4. The command does not run in an editable notebook.

### SS-3.20
1. In an untitled editor whose content looks like a different language than the current one, a lightbulb item appears right next to the language item.
2. Hovering the lightbulb shows "Accept Detected Language: {name}" with the `Alt+Shift+D` key added.
3. Clicking the lightbulb switches the editor to the detected language and the lightbulb goes away.
4. The lightbulb is not shown in saved files, and it goes away when the detected language matches the current one.
5. Setting `workbench.editor.languageDetectionHints` chooses where the lightbulb can show, with the values `untitledEditors` and `notebookEditors` each on or off and both on by default.

### SS-3.21
1. For a file that has a path the language quick pick offers a row "Configure File Association for '{ext}'..." near the top, and an untitled file with no saved path does not get it.
2. Picking that row opens a second list with the placeholder "Select Language Mode to Associate with '{ext}'" holding every language, and the language the extension already maps to is tagged "Current Association".
3. Picking a language writes `*.{ext}` into `files.associations`, so every file with that extension now opens in the chosen language.
4. For dotfiles and for names with no extension, such as `.gitignore` or `Dockerfile`, the whole file name is written instead of `*.{ext}`.
5. The association is saved to user settings, unless the workspace settings already set that same pattern, in which case it is saved to workspace settings.
6. Setting `files.associations` holds the saved patterns, and the language item of matching files shows the chosen language.

### SS-3.22
1. For a file that has a path the language quick pick offers a row "Configure '{language}' language based settings..." naming the file's current language.
2. Picking that row opens the user settings file as JSON, scrolled to the `[language]` block for that language and ready to edit.
3. An untitled file with no saved path does not get this row.

### SS-3.23
1. While overtype mode is on a prominent "OVR" item appears in the status bar and typed characters replace the characters after the cursor instead of pushing them along.
2. Hovering the item shows the tooltip "Enable Insert Mode", and clicking it switches back to insert mode and makes the item disappear.
3. Pressing `Insert` (on macOS `Alt+Cmd+O`) runs Toggle Overtype/Insert Mode, which switches between the two modes and shows or hides the item.
4. Running Toggle Overtype/Insert Mode (`editor.action.toggleOvertypeInsertMode`) from the Command Palette does the same switch.
5. Setting `editor.overtypeCursorStyle` chooses the cursor shape in overtype mode, with values `line`, `block`, `underline`, `line-thin`, `block-outline` and `underline-thin`, default `block`.
6. Setting `editor.overtypeOnPaste` chooses whether pasting overwrites the characters after the cursor in overtype mode, with values on and off, default on.

### SS-3.24
1. While the Tab key is set to move focus instead of typing a tab, a prominent "Tab Moves Focus" item appears in the status bar.
2. Hovering the item shows the tooltip "Disable Accessibility Mode", and clicking it makes Tab type a tab character again and removes the item.
3. Pressing `Ctrl+M` (on macOS `Ctrl+Shift+M`) runs Toggle Tab Key Moves Focus, which turns the mode on or off and shows or hides the item.
4. Running Toggle Tab Key Moves Focus (`editor.action.toggleTabFocusMode`) from the Command Palette does the same switch.
5. A screen reader announces "Pressing Tab will now move focus to the next focusable element" when the mode turns on and "Pressing Tab will now insert the tab character" when it turns off.
6. Setting `editor.tabFocusMode` chooses whether the editor keeps Tab keystrokes or hands them to the workbench for moving focus, with values on and off, default off.

### SS-3.25
1. While column selection mode is on a prominent "Column Selection" item appears in the status bar and selecting with the mouse and keys makes rectangular selections.
2. Hovering the item shows the tooltip "Disable Column Selection Mode", and clicking it turns the mode off and removes the item.
3. The Selection menu row "Column Selection Mode" shows a check while the mode is on, and clicking it turns the mode on or off.
4. Running Toggle Column Selection Mode (`editor.action.toggleColumnSelection`) from the Command Palette does the same switch.
5. Setting `editor.columnSelection` chooses whether selecting with the mouse and keys does column selection, with values on and off, default off.

### SS-3.26
1. While the editor runs in screen reader optimized mode a prominent "Screen Reader Optimized" item appears in the status bar of every window.
2. Clicking the item opens the "Screen reader usage detected" prompt with the buttons Yes, No and Learn More.
3. Choosing Yes writes `on` to `editor.accessibilitySupport` and choosing No writes `off`.
4. Choosing Learn More opens the screen reader section of the accessibility documentation page.
5. Setting `editor.accessibilitySupport` chooses the mode, with values `auto` (detect a screen reader from the platform), `on` (optimize for a screen reader) and `off` (assume none is attached), default `auto`.

### SS-3.27
1. While scrolling is locked across editors a prominent "Scrolling Locked" item appears in the status bar of every window, and scrolling one editor scrolls the others along with it.
2. Hovering the item shows the tooltip "Lock Scrolling Enabled", and clicking it unlocks scrolling and removes the item.
3. Running Toggle Locked Scrolling Across Editors (`workbench.action.toggleLockedScrolling`) from the Command Palette locks scrolling, and running it again unlocks it.
4. Running Hold Locked Scrolling Across Editors (`workbench.action.holdLockedScrolling`) from a key keeps scrolling locked only while that key is held down and unlocks it on release.

### SS-3.28
1. While a binary file is open in the binary file viewer the status bar shows its size as text such as "1.2 KB".
2. Hovering the item shows the tooltip "File Information".
3. The item also shows when the binary file is one side of a side by side diff.
4. The item does not show while a text editor is active.

### SS-3.29
1. In a file that has language status rows a single bracket icon item sits left of the language item and is joined to it as a pair, and it is absent when the file has no rows.
2. The icon is a plain bracket normally, a bracket with a dot when the top row is a warning, a bracket with an error mark when the top row is an error, and a spinner while any row that is not pinned is busy.
3. Hovering the icon opens a list of the rows, and clicking it runs `statusBar.entry.showTooltip` to open the same list.
4. Each row shows a label, detail text that can hold links, an optional command link on the right and a pin button, and its severity icon appears only when the top row is a warning or error.
5. The icon and the list change when the active editor or its language changes, listing only the rows of the file now in front.
6. A screen reader reads the item as "Editor Language Status:" followed by each row's label and detail.

### SS-3.30
1. Hovering a row's pin button shows "Add to Status Bar", and clicking it makes that row its own status bar item right of the language item.
2. The pinned item shows the row's short label, its tooltip is the row's command tooltip or its detail text, and clicking it runs the row's command.
3. A pinned warning row is drawn in the warning color and a pinned error row in the error color.
4. After pinning, the row's button reads "Remove from Status Bar" with a filled pin, and clicking it puts the row back into the bracket item only.
5. Hiding the pinned item from the status bar's right-click list unpins the row, and that list names it "{name} (Language Status)".
6. Pinned rows are remembered per profile and come back when the editor is reopened.

### SS-3.31
1. When a language has several formatters and no default, or the chosen default cannot format the file, the list gets an error row "Formatting" with a detail explaining the conflict and a "Configure..." link.
2. Clicking "Configure..." on that row opens a pick "Select a default formatter for '{language}' files" and the choice is saved as `editor.defaultFormatter`.
3. When a file has more foldable regions than the limit the list gets a warning row "Folding ranges" saying "only {n} shown for performance reasons" with a "Configure" link that opens Settings on `editor.foldingMaximumRegions`.
4. When a file has more color decorators than the limit the list gets a warning row "Color decorators" saying "only {n} shown for performance reasons" with a "Configure" link that opens Settings on `editor.colorDecoratorsLimit`.
5. Each limit row disappears once the file is no longer over its limit.

### SS-3.32
1. The bracket icon wiggles each time the language status changes, and keeps doing so until the person has opened its hover three times.
2. When the top row is a warning or error the icon wiggles and the whole item flashes on every change, even after the person has learned the hover.
3. While the icon shows a spinner it neither wiggles nor flashes.
4. Running Reset Language Status Interaction Counter (`editor.inlayHints.Reset`, in the View category) from the Command Palette makes the icon wiggle again until the hover has been opened three more times.

### SS-3.33
1. In a notebook with a selected kernel the right side shows a kernel icon and the kernel's name, hovering shows the kernel's description, detail or name, and clicking it opens the kernel picker (`_notebook.selectKernel`).
2. In a notebook with no selected kernel but exactly one suggested kernel the item shows that kernel's name, and its tooltip ends with "(suggestion)".
3. A suggested kernel becomes the selected kernel once the person edits a cell, moves cells, changes a cell's language or hovers over cell text.
4. In a notebook with several kernels and none chosen a prominent "Select Kernel" item appears, and clicking it opens the kernel picker.
5. In a notebook that has no kernel at all no kernel item shows.
6. While the notebook toolbar already has its own kernel button no kernel item shows in the status bar.
7. Setting `notebook.globalToolbar` chooses whether the notebook toolbar is shown, with values on and off and a default of on, so the status bar kernel item appears only when it is off.

### SS-3.34
1. While a notebook cell is focused the item shows "Cell {n} of {total}".
2. When several cells are selected the item shows "Cell {n} ({m} selected)".
3. Clicking the item scrolls the active cell to the middle of the notebook editor.
4. The item does not show when no notebook editor is active or when no cell is active.

### SS-3.35
1. In a notebook the item shows the active cell's indentation as "Spaces: {n}" or "Tab Size: {n}", hovering shows "Select Indentation", and it changes as the active cell changes.
2. Values set in `notebook.editorOptionsCustomizations` for `editor.insertSpaces`, `editor.tabSize` and `editor.indentSize` take precedence over the cell's own values in what the item shows.
3. Settings `editor.insertSpaces`, `editor.tabSize` and `editor.indentSize` give the starting indentation that cells show when no notebook override is set.
4. Clicking the item opens a quick pick with the placeholder "Select Action" with a "change view" group (Indent Using Tabs, Indent Using Spaces, Change Tab Display Size) and a "convert file" group (Convert Indentation to Tabs, Convert Indentation to Spaces).
5. Picking an Indent Using row or Change Tab Display Size opens the size list of 1 to 8 with the placeholder "Select Tab Size for Current File" and applies the pick to the notebook's cells.
6. Picking a Convert Indentation row rewrites the leading whitespace in the notebook's cells in one edit labelled "Convert Indentation".
7. In a read-only notebook the quick pick shows one row, "The active notebook editor is read-only.", and with no notebook active it shows "No notebook editor active at this time".
8. Running Select Indentation (`notebook.selectIndentation`) from the Command Palette opens the same quick pick, and it needs an editable notebook with an editable cell.

### SS-3.36
1. Running Developer: Start Tracing (`workbench.action.startTracing`) from the Command Palette starts a performance trace and puts a red "Recording trace (click to stop)" item with a record icon at the far left end of the left side of the status bar, in the desktop app only. [stock only]
2. Hovering the red item shows the tooltip "Click to stop recording". [stock only]
3. Running Developer: Start Heap Tracing (`workbench.action.startHeapTracing`) starts a trace that also records memory and shows the same red item. [stock only]
4. Clicking the red item runs Stop Tracing, which shows a progress dialog "Creating trace file..." with the detail "This can take up to one minute to complete", writes the trace and removes the item. [stock only]
5. Running Developer: Stop Tracing (`workbench.action.stopTracing`) when no trace is running asks whether to relaunch with the trace launch argument, with a "Relaunch and Enable Tracing" button in stock VS Code.
6. In Cursor there is no red recording item and no Start Tracing or Start Heap Tracing command, and Developer: Stop Tracing is the only tracing command left, in an older form that offers to relaunch with the trace launch argument. [cursor only]

### SS-3.37
1. When the network is treated as metered an icon-only radio tower item appears at the far right end of the right side of the status bar in every window. [stock only]
2. Hovering the item shows "Metered connection enabled. Some automatic features like extension updates, Settings Sync, and automatic Git operations are paused to reduce data usage." [stock only]
3. Clicking the item opens a quick pick with the placeholder "Select Metered Connection Mode" holding Auto (detect automatically), On (always treat as metered) and Off (never treat as metered). [stock only]
4. Running Configure Metered Connection (`workbench.action.configureMeteredConnection`) from the Command Palette opens the same quick pick. [stock only]
5. The item disappears when the connection stops being metered. [stock only]
6. Setting `network.meteredConnection` chooses how the connection is judged, with values `auto`, `on` and `off`, default `auto`. [stock only]

### SS-3.38
1. In the web build only, an item "Layout: {name}" shows the detected keyboard layout on the right side of the status bar. [stock only]
2. Clicking the item, or running Change Keyboard Layout (`workbench.action.openKeyboardLayoutPicker`) from the Command Palette, opens a quick pick with the placeholder "Select Keyboard Layout". [stock only]
3. The quick pick starts with an Auto Detect row and a Configure Keyboard Layout row, then lists the layouts sorted by name under a "Keyboard Layouts ({platform})" heading, with the current one marked "(Current layout)". [stock only]
4. Picking a layout switches to it and the item text changes, and picking Auto Detect goes back to detecting the layout on its own. [stock only]
5. Picking Configure Keyboard Layout opens the keyboard layout file as JSON with comments, creating it with a short instruction comment first when it does not exist. [stock only]

### SS-3.39
1. While a rendered Markdown diff preview is the active editor a warning-colored item with a warning icon and the text "Rendered Diff" appears on the right of the status bar. [stock only]
2. Hovering the item shows "Rendered Markdown diffs may hide important changes. Click for details." [stock only]
3. Clicking the item shows a warning message saying rendered diffs may hide changes such as formatting, whitespace, links or HTML and to switch to the text diff to review them, with a "Don't Show Again" button. [stock only]
4. The same warning message shows once on its own the first time a rendered diff preview becomes active in that workspace. [stock only]
5. Choosing "Don't Show Again" removes the item and stops the warning for that workspace. [stock only]
6. The item goes away when another editor becomes active. [stock only]

### SS-3.40
1. While an image is the active Image Preview editor a right-side item shows the zoom level as a percent such as "100%", or "Whole Image" when the picture fits the window.
2. Clicking the item opens a quick pick with the placeholder "Select zoom level" and the rows 1000%, 500%, 200%, 100%, 50%, 20% and Whole Image.
3. Picking a row rescales the image in the preview and the item shows the new level.
4. The item sits left of the Image Size item and hides when another editor is active.

### SS-3.41
1. While an image is the active Image Preview editor a right-side item shows the picture's own pixel size as "{width}x{height}", for example "1920x1080".
2. The item stays empty until the image has loaded.
3. The item has no click action.
4. The item sits just right of the Image Zoom item and just left of the file size item, and hides when the preview is not the active editor.

### SS-3.42
1. While an image preview is the active editor a right-side item shows the size of the file on disk, such as "512B", "12.34KB", "1.20MB", "1.20GB" or "1.20TB".
2. The same item shows in the audio preview and the video preview.
3. The item updates when the file changes on disk.
4. The item has no click action and hides when the preview is not the active editor.

### SS-3.43
1. In a JavaScript, JavaScript React, TypeScript or TypeScript React file the language status list gets a row whose text is the display name of the TypeScript version the server started with, with the detail "TypeScript version".
2. The row has a link "Select Version" whose tooltip is the path of that version, and clicking it opens the version picker.
3. The row appears only after the TypeScript server has started, and it does not appear in files of other languages.
4. Pinning the row to the status bar shows the version as its own item that opens the version picker when clicked.

### SS-3.44
1. Clicking the "Select Version" link opens a quick pick titled "Select the TypeScript version used for JavaScript and TypeScript language features".
2. Running TypeScript: Select TypeScript Version (`typescript.selectTypeScriptVersion`) from the Command Palette opens the same quick pick, and the command is listed only while the active file is one the TypeScript extension manages.
3. The first row is "Use VS Code's Version" with its version number, followed by one "Use Workspace Version" row for each local TypeScript install the extension finds.
4. A "Use TypeScript Native Preview (Experimental)" row appears only when the TypeScript Native Preview extension is installed.
5. The last row, "Learn more about managing TypeScript versions", opens the documentation page for it.
6. The row of the version currently in use is marked with a bullet.
7. Picking a workspace version asks for Workspace Trust, remembers the choice for this workspace, writes the folder path into the workspace settings and restarts the TypeScript server on that version.
8. Setting `js/ts.tsdk.path`, and its older deprecated name `typescript.tsdk`, holds the folder path of the TypeScript install to use.
9. Setting `typescript.enablePromptUseWorkspaceTsdk` chooses whether the person is prompted to use the TypeScript version set in the workspace, with values on and off, default off.

### SS-3.45
1. While the TypeScript server looks up the project of a JavaScript or TypeScript file, the language status list shows an IntelliSense row reading "Loading IntelliSense status" with a spinner.
2. Once the project is known the row shows the workspace-relative path of the tsconfig or jsconfig that owns the file, with a link "Open Config File" that opens it.
3. When no config file owns the file the row reads "No tsconfig" for a TypeScript file or "No jsconfig" for a JavaScript file, with a link "Configure TSConfig" or "Configure JSConfig".
4. Clicking that link opens `tsconfig.json` or `jsconfig.json` in the workspace root, and when the file does not exist an untitled copy opens filled with a starter config built from the implicit project settings plus an exclude of `node_modules`.
5. With no folder open the row reads "No tsconfig" or "No jsconfig" with the detail "No opened folders", and for a file outside the open folders the detail is "File is not part opened folders", with no link in either case.
6. Settings `js/ts.implicitProjectConfig.module` (default `ESNext`) and `js/ts.implicitProjectConfig.target` (default `ES2024`) choose the module and target written into the starter config and used for files outside a project.
7. Settings `js/ts.implicitProjectConfig.checkJs`, `js/ts.implicitProjectConfig.experimentalDecorators`, `js/ts.implicitProjectConfig.strictNullChecks`, `js/ts.implicitProjectConfig.strictFunctionTypes` and `js/ts.implicitProjectConfig.strict` each turn the matching compiler option on or off in the starter config and for files outside a project.

### SS-3.46
1. When project-wide IntelliSense is not available for the open JavaScript or TypeScript file the IntelliSense row reads "Partial mode" with the detail "Project wide IntelliSense not available".
2. The row has a link "Learn More" that opens the partial mode help page.
3. Because the row is a warning, the bracket icon changes to the warning bracket and the item flashes as described in SS-3.32.
4. On desktop the row shows for files whose scheme is not a local file, for untitled files and for notebook cells.
5. Setting `js/ts.tsserver.useSyntaxServer` (and the older `typescript.tsserver.useSyntaxServer`) chooses whether the syntax-only server is used, with values `always`, `never` and `auto`, default `auto`, and `always` puts every file in Partial mode.

### SS-3.47
1. When the TypeScript server turns off project-wide language features because a project holds too many files, a green "Configure Excludes" item appears on the right side of the status bar.
2. Hovering the item shows a tooltip telling the person to exclude large folders with source files they do not work on to enable project-wide JavaScript and TypeScript language features.
3. Clicking the item opens the project's tsconfig or jsconfig, or creates one in the workspace root when the project has none, and shows the same hint as an information message.
4. The same event raises a warning message with the same hint and a "Configure Excludes" button that opens or creates the config file the same way.
5. The item disappears once the server reports that the language service is enabled again.
6. A config file created from this item is filled from the `js/ts.implicitProjectConfig.*` settings listed in SS-3.45.

### SS-3.48
1. In a JSON file with no schema the language status list shows a validation row "No schema validation" with the detail "no JSON schema configured".
2. With one schema the row reads "Schema validated" with the detail "JSON schema configured", and with several schemas the detail is "multiple JSON schemas configured".
3. While the row works it shows a spinner and the detail "Loading JSON info".
4. When the lookup fails the row reads "Unable to compute used schemas" followed by the reason and has no link.
5. The row has a link "Show Schemas" that opens the schema list described in SS-3.49.
6. Setting `json.validate.enable` chooses whether JSON files are validated, with values on and off, default on.
7. Setting `json.schemas` lists the schema associations that decide which schemas the row counts.

### SS-3.49
1. Clicking "Show Schemas" opens a quick pick with the placeholder "Select the schema to use for {file}" listing each schema that applies to the file.
2. A schema that comes from an extension is marked "Configured by extension: {id}" and has an Open Extension button.
3. A schema set in user or workspace settings is marked "Configured in user settings" or "Configured in workspace settings" and has an Open Settings button that opens the right settings file at `json.schemas`.
4. A schema that matches neither case shows only its URL.
5. When no schema applies the quick pick shows one row, "No schema configured for this file", with an Open Settings button that opens the user settings JSON at `json.schemas`.
6. A last row after a separator, "Learn more about JSON schema configuration...", opens the documentation page for it.
7. Picking a schema row opens that schema file.

### SS-3.50
1. When a JSON file has more symbols than the limit the language status list shows a warning row "Outline" with the detail "only {n} document symbols shown for performance reasons".
2. The row has a link "Configure" that opens Settings on `json.maxItemsComputed`.
3. The row shows only while that file is the active editor, and it goes away when the file drops under the limit.
4. Setting `json.maxItemsComputed` chooses the most outline symbols and folding regions computed for a JSON file, a number with a default of 5000.

### SS-3.51
1. When the schema of the active JSON file cannot be loaded the language status list shows an error row "Schema download issue", and the bracket icon switches to the error bracket and flashes as described in SS-3.32. [stock only]
2. With the workspace untrusted the row detail reads "Workspace untrusted" and its link opens Manage Workspace Trust. [stock only]
3. With schema downloads off the row detail reads "Download disabled" and its link "Configure" opens Settings on `json.schemaDownload.enable`. [stock only]
4. When the schema location is not trusted the row detail reads "Location untrusted" and its link opens the trusted domains configuration. [stock only]
5. For any other failure the row detail reads "Unable to resolve schema" and its link "Retry" tries to load the schema again. [stock only]
6. Setting `json.schemaDownload.enable` chooses whether JSON schemas can be downloaded, with values on and off, default on. [stock only]
7. Setting `json.schemaDownload.trustedDomains` chooses which schema locations count as trusted. [stock only]

### SS-3.52
1. In Cursor, when the active JSON file has a schema that cannot be loaded, an icon-only alert item appears on the right side, to the right of the editor items and left of the notifications bell. [cursor only]
2. With schema downloads on, hovering the item shows "Unable to resolve schema. Click to retry." and clicking it retries the validation, showing a watch icon while it runs and hiding the item if the schema now loads. [cursor only]
3. With schema downloads off, hovering the item shows "Downloading schemas is disabled. Click to configure." and clicking it opens Settings on `json.schemaDownload.enable`. [cursor only]
4. The item shows only while the JSON file with the schema error is the active editor and hides when another editor becomes active. [cursor only]

### SS-3.53
1. In a Markdown file with link validation on, the language status list shows the row "Markdown link validation enabled" with a link "Disable" whose tooltip is "Disable validation of Markdown links".
2. In a Markdown file with link validation off, the row reads "Markdown link validation disabled" with a link "Enable" whose tooltip is "Enable validation of Markdown links".
3. Clicking the link flips `markdown.validate.enabled`, and the row text and link change at once.
4. Setting `markdown.validate.enabled` chooses whether Markdown links are validated, with values on and off, default off, and changing it in Settings also updates the row.

## Unclear

- SS-3.17: Cursor gives `workbench.action.editor.changeLanguageMode` its own macOS keybinding entry, and I could not decode the key chord from the bundle, so piece 9 only says the entry exists and cannot be checked against a specific key.
- SS-3.19 and SS-3.20: the feature blocks list `workbench.editor.languageDetection`, but the Detect Language command and the lightbulb in V do not read it, so I left that setting out of their pieces. It appears in SS-3.18 and in no other feature, and it is read by the automatic detection of the language.
- SS-3.36: stock also asks to relaunch with the trace launch argument when Stop Tracing runs with no trace in progress, so only the missing red item and the missing Start commands are Cursor differences. Piece 5 is untagged and piece 6 is tagged `[cursor only]`.
- SS-3.38: the keyboard layout item is web build only, so its pieces cannot be checked in the desktop app in either Cursor or stock VS Code.
- SS-3.6 to SS-3.9: these overlap with the editor commands in ED-2.18 and ED-2.19, as the feature file already notes, so the commands may be counted twice by a tracker.
- The language picker row "Search Marketplace Extensions for '{ext}'..." is not covered here because the feature file leaves it out as out of scope.
- Default visibility on a fresh Cursor profile was not checked at runtime, and several pieces name defaults that were read from V only (for example SS-3.45 default values for the implicit project settings). Cursor package files were only checked for the presence of the settings.
