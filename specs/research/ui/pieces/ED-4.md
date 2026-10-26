Group: ED-4 Language help in the editor. Pass: 3. Date: 2026-10-25.

### ED-4.1
1. Resting the pointer on code opens a floating hover card with the documentation, type information and links the language supplies.
2. Moving the pointer from the code onto the hover card keeps the card open so its links can be used, and moving away from both closes it after a short delay.
3. The hover card shows a plus button and a minus button named Increase Hover Verbosity and Decrease Hover Verbosity that show more or less detail (`editor.action.increaseHoverVerbosityLevel`, `editor.action.decreaseHoverVerbosityLevel`).
4. Code blocks inside the hover card show a Copy button that copies the code and reports "Copied to clipboard". [stock only]
5. Setting `editor.hover.enabled` chooses whether hovers appear, with values `on`, `off` and `onKeyboardModifier` (the hover shows only while holding Ctrl, or Cmd on Mac, or Alt).
6. Setting `editor.hover.delay` chooses how many milliseconds the pointer must rest before the hover shows, from 0 to 10000 with 300 as the default.
7. Turning `editor.hover.sticky` off makes the hover close as soon as the pointer leaves the code instead of staying open when the pointer moves onto it.
8. Setting `editor.hover.hidingDelay` chooses how many milliseconds a sticky hover waits before it closes, with 300 as the default.
9. Setting `editor.hover.above` makes the hover prefer opening above the line when there is room, and it is on by default.
10. Setting `editor.hover.showLongLineWarning` shows a warning hover on very long lines where tokenization is skipped or rendering is paused, and `editor.action.hideLongLineWarningHover` hides it. [stock only]

### ED-4.2
1. Pressing the Show or Focus Hover key with the text cursor in code opens the hover card at the text cursor without using the mouse (`editor.action.showHover`).
2. Pressing the same key again while the card is open moves keyboard focus into the card.
3. With the card focused, Up, Down, Left and Right scroll its content one step in that direction (`editor.action.scrollUpHover`, `editor.action.scrollDownHover`, `editor.action.scrollLeftHover`, `editor.action.scrollRightHover`).
4. With the card focused, PageUp or Alt+Up scrolls up by a page and PageDown or Alt+Down scrolls down by a page (`editor.action.pageUpHover`, `editor.action.pageDownHover`).
5. With the card focused, Home or Ctrl+Up jumps to the top of the content and End or Ctrl+Down jumps to the bottom (`editor.action.goToTopHover`, `editor.action.goToBottomHover`).
6. Running Hide Hover from the Command Palette closes the open hover card, and it has no default key (`editor.action.hideHover`).
7. In Cursor the Show or Focus Hover key is Ctrl+M Ctrl+I on Windows and Linux and Cmd+R Cmd+I on Mac, where stock uses Ctrl+K Ctrl+I. [cursor only]

### ED-4.3
1. Pressing F12 with the cursor on a symbol jumps to where it is defined (`editor.action.revealDefinition`).
2. Go to Definition is also in the editor right-click menu and in the Go menu, and the right-click entry shows only when the language provides definitions.
3. Open Definition to the Side opens the definition in a split beside the current editor (`editor.action.revealDefinitionAside`).
4. In the web build only, Ctrl+F12 also runs Go to Definition and Ctrl+K Ctrl+F12 opens the definition to the side.
5. Setting `editor.gotoLocation.multipleDefinitions` chooses what happens when several definitions are found, with values `peek`, `gotoAndPeek` and `goto`.
6. Setting `editor.gotoLocation.multiple` is marked deprecated and its message points to the separate per-command settings.
7. Setting `editor.gotoLocation.alternativeDefinitionCommand` names the command that runs instead when Go to Definition lands on the spot you are already at, and it defaults to `editor.action.goToReferences`.
8. In Cursor, Open Definition to the Side starts with Ctrl+M on Windows and Linux and Cmd+R on Mac, where stock starts with Ctrl+K. [cursor only]

### ED-4.4
1. Holding Ctrl (Cmd on Mac) while pointing at a symbol that has a definition underlines it as a link.
2. While the link shows, the hover card previews the code at the definition.
3. Clicking the underlined symbol jumps to its definition.
4. Holding Alt as well while clicking (Ctrl+Alt+click, Cmd+Alt+click on Mac) opens the definition in a split beside the current editor.
5. Running Show Definition Preview Hover shows the same definition preview at the text cursor and moves focus into it, so it works from the keyboard (`editor.action.showDefinitionPreviewHover`).
6. Setting `editor.definitionLinkOpensInPeek` makes the click open the definition in the peek window instead of jumping, and it is off by default.
7. Setting `editor.multiCursorModifier` with value `ctrlCmd` swaps the keys so Alt makes the link and Ctrl (Cmd on Mac) opens it to the side, and with value `alt` the Ctrl or Cmd key makes the link.
8. Setting `editor.mouseMiddleClickAction` chooses what a middle click does in the editor, with values `default`, `openLink` and `ctrlLeftClick`.

### ED-4.5
1. Go to Declaration jumps from the symbol at the cursor to the place it is declared, and it has no default key (`editor.action.revealDeclaration`).
2. Go to Declaration is also in the editor right-click menu and in the Go menu, and the right-click entry shows only when the language provides declarations.
3. Setting `editor.gotoLocation.multipleDeclarations` chooses what happens when several declarations are found, with values `peek`, `gotoAndPeek` and `goto`.
4. Setting `editor.gotoLocation.alternativeDeclarationCommand` names the command that runs instead when Go to Declaration lands on the spot you are already at.

### ED-4.6
1. Go to Type Definition jumps from a variable or expression to the definition of its type, and it has no default key (`editor.action.goToTypeDefinition`).
2. Go to Type Definition is also in the editor right-click menu and in the Go menu, and the right-click entry shows only when the language provides type definitions.
3. Setting `editor.gotoLocation.multipleTypeDefinitions` chooses what happens when several type definitions are found, with values `peek`, `gotoAndPeek` and `goto`.
4. Setting `editor.gotoLocation.alternativeTypeDefinitionCommand` names the command that runs instead when Go to Type Definition lands on the spot you are already at.

### ED-4.7
1. Pressing Ctrl+F12 with the cursor on an interface, abstract member or base method jumps to the code that implements it (`editor.action.goToImplementation`).
2. Go to Implementations is also in the editor right-click menu and in the Go menu, and the right-click entry shows only when the language provides implementations.
3. Setting `editor.gotoLocation.multipleImplementations` chooses what happens when several implementations are found, with values `peek`, `gotoAndPeek` and `goto`.
4. Setting `editor.gotoLocation.alternativeImplementationCommand` names the command that runs instead when Go to Implementations lands on the spot you are already at.

### ED-4.8
1. Pressing Shift+F12 with the cursor on a symbol finds every use of it and jumps straight there when there is only one (`editor.action.goToReferences`).
2. When more than one use is found, the uses open together in the peek window by default.
3. Go to References is also in the editor right-click menu and in the Go menu.
4. Setting `editor.gotoLocation.multipleReferences` chooses what happens when several uses are found, with values `peek`, `gotoAndPeek` and `goto`.
5. Setting `editor.gotoLocation.alternativeReferenceCommand` names the command that runs instead when Go to References lands on the spot you are already at.

### ED-4.9
1. The editor right-click menu has a Peek submenu in its navigation group that holds the peek entries.
2. Peek Definition opens the definition in an inline panel under the current line without leaving the file, with Alt+F12 as its key (Ctrl+Shift+F10 on Linux) (`editor.action.peekDefinition`).
3. Peek Declaration and Peek Type Definition open the declaration or the type definition in the same inline panel (`editor.action.peekDeclaration`, `editor.action.peekTypeDefinition`).
4. Peek Implementations opens the implementations in the inline panel, with Ctrl+Shift+F12 as its key (`editor.action.peekImplementation`).
5. Peek References opens every use of the symbol in the inline panel (`editor.action.referenceSearch.trigger`).
6. Each Peek entry shows in the submenu only when the language provides that kind of result.

### ED-4.10
1. The peek window opens inline under the current line with a title bar that shows the file and how many results there are.
2. Inside the window a results tree sits beside an editable preview of the selected result.
3. Pressing Enter on the focused result opens it in the editor (`openReference`).
4. Pressing Ctrl+Enter on the focused result opens it in a split beside the editor, and on Mac it is the Control key with Enter (`openReferenceToSide`).
5. Pressing F4 or F12 moves to the next result and Shift+F4 or Shift+F12 moves to the previous one (`goToNextReference`, `goToPreviousReference`).
6. Pressing the toggle focus key moves keyboard focus between the peek window and the editor, and it is Ctrl+K F2 in stock (`togglePeekWidgetFocus`).
7. Pressing Escape or Shift+Escape closes the peek window (`closeReferenceSearch`).
8. Setting `editor.peekWidgetDefaultFocus` chooses what has focus when the peek window opens, with values `tree` and `editor`.
9. Turning on `editor.stablePeek` keeps the peek window open when you press Escape or double-click its content.
10. In Cursor the toggle focus chord is Ctrl+M F2 on Windows and Linux and Cmd+R F2 on Mac, where stock uses Ctrl+K F2. [cursor only]

### ED-4.11
1. An icon button named Open in Chat sits at the left end of the action area in the peek window title bar. [cursor only]
2. Clicking Open in Chat starts a new chat. [cursor only]
3. The new chat gets every reference listed in the peek window attached, each with a few lines of code around it. [cursor only]

### ED-4.12
1. Pressing F2 with the cursor on a symbol opens a small rename box at the cursor holding the current name (`editor.action.rename`).
2. Pressing Enter in the rename box renames the symbol everywhere it is used (`acceptRenameInput`).
3. Pressing Ctrl+Enter in the rename box shows the proposed changes in a preview before they are applied (`acceptRenameInputWithPreview`).
4. Pressing Escape or Shift+Escape closes the rename box without renaming (`cancelRenameInput`).
5. Rename Symbol is also in the modification group of the editor right-click menu, and it shows only when the language supports renaming.
6. When a provider supplies name suggestions, a button in the rename box shows them, and Down and Up move through the suggestions (`focusNextRenameSuggestion`, `focusPreviousRenameSuggestion`).
7. Setting `editor.rename.enablePreview` chooses whether the preview option is offered in the rename box, and it is on by default.

### ED-4.13
1. A lightbulb icon appears beside the cursor line when fixes or refactorings are available there, and it goes away when none are.
2. When both the lines above and below are taken by other gutter icons, the lightbulb shows in the gutter instead of beside the text.
3. The icon changes with what is offered, using a plain lightbulb, an auto fix lightbulb, a sparkle lightbulb for an AI fix, and a sparkle auto fix lightbulb when both apply.
4. The lightbulb tooltip reads "Show Code Actions" with the Quick Fix key, or "Show Code Actions. Preferred Quick Fix Available" with the auto fix key, or "Run: <action title>" when clicking runs the action straight away.
5. Clicking the lightbulb opens the code actions menu.
6. Setting `editor.lightbulb.enabled` chooses when the lightbulb shows, with values `off`, `onCode` (lines with code, the default) and `on` (lines with code and empty lines).
7. Turning on `editor.codeActions.triggerOnFocusChange` lets code actions on save also run when `files.autoSave` is `afterDelay`, and it is off by default.

### ED-4.14
1. Pressing Ctrl+. with the cursor in code opens a list of fixes and refactors at the cursor, and the same list opens from the Command Palette (`editor.action.quickFix`, `editor.action.codeAction`).
2. Entries in the list sit under headers named Quick Fix, Extract, Inline, Rewrite, Move, Surround With, Source Action and More Actions....
3. Up and Down move the highlight through the list, and Enter or Ctrl+. applies the highlighted entry (`selectPrevCodeAction`, `selectNextCodeAction`, `acceptSelectedCodeAction`).
4. Pressing Escape or Shift+Escape closes the list without applying anything (`hideCodeActionWidget`).
5. Pressing Ctrl+Enter on an entry previews its changes before applying them (`previewSelectedCodeAction`).
6. Typing in the open list narrows it to matching entries, and a clear filter command empties the filter (`clearFilterCodeActionWidget`). [stock only]
7. Left, Right and Space on a section header collapse it, expand it or toggle it (`collapseSectionCodeAction`, `expandSectionCodeAction`, `toggleSectionCodeAction`). [stock only]
8. Setting `editor.codeActionWidget.showHeaders` chooses whether the group headers show in the list, and it is on by default.
9. Setting `editor.codeActionWidget.includeNearbyQuickFixes` chooses whether the nearest quick fix on the line is offered when the cursor is not on a problem, and it is on by default.
10. Quick Fix is also listed in a second editor menu in stock, while in Cursor it is a plain editor action with no menu entry. [stock only]

### ED-4.15
1. Pressing Ctrl+Shift+R with a selection opens a menu of refactorings for it, such as extract, inline, rewrite and move, and on Mac it is the Control key with Shift+R (`editor.action.refactor`).
2. Refactor... is also in the modification group of the editor right-click menu.
3. Choosing an entry in the refactor menu runs it, and Ctrl+Enter on an entry previews it first.
4. Running `editor.action.codeAction` with a refactor kind from a custom key opens the same menu limited to refactorings.

### ED-4.16
1. Source Action... in the modification group of the editor right-click menu opens a menu of file-wide clean-ups (`editor.action.sourceAction`).
2. Pressing Shift+Alt+O runs Organize Imports on the file (`editor.action.organizeImports`).
3. Running Fix All from the Command Palette applies every automatic fix in the file (`editor.action.fixAll`).
4. Pressing Shift+Alt+. applies the preferred auto fix, and on Mac it is Cmd+Alt+. (`editor.action.autoFix`).
5. Setting `editor.codeActionsOnSave` lists the code action kinds to run on save, each set to `explicit`, `always` or `never`, with true and false still accepted.
6. When `files.autoSave` is `afterDelay`, the actions in `editor.codeActionsOnSave` run only on an explicit save.
7. While a save runs these actions, the save progress reads "Quick Fixes" and then "Applying code action '<title>'.".

### ED-4.17
1. Pressing Shift+Alt+F (Ctrl+Shift+I on Linux) reformats the whole file with the language's formatter (`editor.action.formatDocument`).
2. Format Selection reformats only the selected text, and it has its own key (`editor.action.formatSelection`).
3. Format Document and Format Selection are also in the modification group of the editor right-click menu.
4. Format Document With... and Format Selection With... open a picker titled "Select a formatter" that lists the installed formatters, marks the default one with "(default)" and ends with "Configure Default Formatter..." (`editor.action.formatDocument.multiple`, `editor.action.formatSelection.multiple`).
5. When several formatters fit the file and none is the default, a prompt reads "Configure Default Formatter" with a Configure... button that opens a picker titled "Select a default formatter for '<language>' files".
6. When no formatter is installed for the language, a message reads "There is no formatter for '<language>' files installed." with an Install Formatter... button (`editor.action.formatDocument.none`).
7. Formatting a file that is too large shows the warning "This file cannot be formatted because it is too large".
8. Setting `editor.defaultFormatter` names the formatter extension that wins over other formatter settings, and None is one of the choices.
9. In Cursor, Format Selection starts with Ctrl+M Ctrl+F on Windows and Linux and Cmd+R Cmd+F on Mac, where stock uses Ctrl+K Ctrl+F. [cursor only]

### ED-4.18
1. Running Format Modified Lines from the Command Palette formats only the lines that differ from the source control version and leaves the rest of the file alone (`editor.action.formatChanges`).
2. Format Modified Lines is available only in an editable file whose language has a formatter that can format selections.

### ED-4.19
1. Turning on `editor.formatOnSave` formats the file each time it is saved.
2. Setting `editor.formatOnSaveMode` chooses what is formatted on save, with values `file` (the whole file, the default), `modifications` (only changed lines, needs source control) and `modificationsIfAvailable` (changed lines, or the whole file when source control cannot be used).
3. When `files.autoSave` is `afterDelay`, format on save runs only when the file is saved explicitly.
4. Turning on `editor.formatOnPaste` formats pasted text as it lands in the file.
5. Turning on `editor.formatOnType` formats a line when you finish typing it.

### ED-4.20
1. Small gray clickable text appears above a line, such as a reference count or a Run Test link, and clicking it runs the command behind it.
2. Running Show CodeLens Commands for Current Line opens a picker that lists every lens on the cursor line, and choosing one runs it (`codelens.showLensesInCurrentLine`).
3. Setting `editor.codeLens` chooses whether CodeLens shows at all, and it is on by default.
4. Setting `editor.codeLensFontFamily` chooses the font used for CodeLens text.
5. Setting `editor.codeLensFontSize` chooses the CodeLens font size in pixels, and 0 uses 90 percent of `editor.fontSize`.

### ED-4.21
1. Small gray labels appear inside the code, such as parameter names and types, supplied by the language.
2. Hovering a hint opens a hover card with details about it.
3. Ctrl+click (Cmd+click on Mac) on a part of a hint follows it to its location.
4. Double-clicking a hint inserts its text into the code.
5. Right-clicking a hint opens a menu with the go-to entries such as Go to Definition.
6. Running Read Line with Inlay Hints makes a screen reader read the line together with its hints, and Stop Inlay Hints Reading or Escape stops it (`inlayHints.startReadingLineWithHint`, `inlayHints.stopReadingLineWithHint`).
7. Setting `editor.inlayHints.enabled` chooses when hints show, with values `on`, `off`, `onUnlessPressed` and `offUnlessPressed`, where holding Ctrl+Alt (Ctrl+Option on Mac) flips the last two.
8. Settings `editor.inlayHints.fontSize` and `editor.inlayHints.fontFamily` choose the size and font of the hints, and empty or small values fall back to the editor font.
9. Setting `editor.inlayHints.padding` adds space around each hint when on, and it is off by default.
10. Setting `editor.inlayHints.maximumLength` chooses how long the hints on one line may be before they are cut short, with 43 as the default and 0 meaning never cut.

### ED-4.22
1. Putting the cursor on a symbol shades its other uses in the file.
2. Reads and writes of the symbol are shaded in different shades.
3. Marks for the highlighted uses show in the scrollbar.
4. Pressing F7 runs Go to Next Symbol Highlight and Shift+F7 runs Go to Previous Symbol Highlight, but only while highlights exist (`editor.action.wordHighlight.next`, `editor.action.wordHighlight.prev`).
5. Running Trigger Symbol Highlight from the Command Palette shows the highlights for the symbol at the cursor, and it has no default key (`editor.action.wordHighlight.trigger`).
6. Setting `editor.occurrencesHighlight` chooses where uses are highlighted, with values `off`, `singleFile` (the default) and `multiFile` (experimental, across open files).
7. Setting `editor.occurrencesHighlightDelay` chooses how many milliseconds pass before the uses are highlighted.

### ED-4.23
1. For languages that support it, the code gets extra colors based on meaning, such as a parameter versus a local variable or a type versus a class, layered on top of the grammar colors.
2. Setting `editor.semanticHighlighting.enabled` chooses whether the meaning-based colors show, with values `true`, `false` and `configuredByTheme` (the default).
3. With `configuredByTheme`, the current color theme's `semanticHighlighting` setting decides whether the meaning-based colors show, so switching themes can turn them on or off.
4. Setting `editor.semanticTokenColorCustomizations` overrides the color and style of individual token types.

### ED-4.24
1. Code with an error is underlined with a red wavy line.
2. Code with a warning is underlined with a yellow wavy line and code with an info is underlined with a blue wavy line.
3. Code with a hint is marked with faint dots instead of a wavy line.
4. Each problem also puts a mark in the scrollbar ruler, colored by its severity.
5. Each problem also puts a mark in the minimap.
6. The theme colors `editorError.foreground`, `editorWarning.foreground`, `editorInfo.foreground` and `editorHint.foreground` set the colors of the underlines and marks.

### ED-4.25
1. Code the language marks as unused is faded out.
2. Names the language marks as deprecated are drawn with a line through them.
3. Settings `editor.showUnused` and `editor.showDeprecated` turn the fading and the strikethrough on or off, and both are on by default.
4. The theme color `editorUnnecessaryCode.opacity` sets how faded the unused code looks.

### ED-4.26
1. Hovering a squiggle opens a hover card with the problem message followed by its source and code.
2. The hover card ends with a status row that has a View Problem button showing Alt+F8, and clicking it closes the hover and opens the inline problem widget on that problem.
3. When quick fixes exist, the status row also has a Quick Fix... button showing Ctrl+., and clicking it opens the code actions menu at the button.
4. While the editor looks for quick fixes, the status row reads "Checking for quick fixes..." after a short wait.
5. When there are no quick fixes, the status row reads "No quick fixes available".
6. In a read-only editor the status row shows View Problem but no quick fix row.
7. When an AI fix is among the code actions, the status row adds a button with that fix's title that applies it on click.
8. The AI fix button in the status row has a sparkle icon. [stock only]
9. Other actions added to the problem hover menu show as extra buttons in the status row when there is no AI fix button. [stock only]
10. Setting `editor.hover.enabled` set to `off` also stops problem hovers from showing, and `editor.hover.delay` sets how long the pointer must rest on a squiggle before the hover shows.

### ED-4.27
1. Pressing Alt+F8 moves to the next error, warning or info in the current file and opens an inline problem widget under it, and Shift+Alt+F8 moves to the previous one (`editor.action.marker.next`, `editor.action.marker.prev`).
2. The inline problem widget shows the problem message, source and code, and its position as "n of m problems".
3. The widget title has up and down arrow buttons that step to the previous or next problem.
4. Pressing F8 and Shift+F8 does the same stepping across all files, opening another file when the next problem is there (`editor.action.marker.nextInFiles`, `editor.action.marker.prevInFiles`).
5. Pressing Escape or Shift+Escape closes the inline problem widget (`closeMarkersNavigation`).
6. The Go menu has Next Problem and Previous Problem entries in its problem navigation group.

### ED-4.28
1. URLs, file paths and command links in the text become underlined links while Ctrl (Cmd on Mac) is held.
2. Pointing at a link shows a tooltip that says Follow link or Execute command.
3. Ctrl+click (Cmd+click on Mac) on a link opens it, or runs the command for a command link.
4. Running Open Link from the Command Palette opens the link under the text cursor (`editor.action.openLink`).
5. Setting `editor.links` chooses whether the editor detects links and makes them clickable, and it is on by default.

### ED-4.29
1. Colors written as hex, rgb or hsl values show a small color swatch in the text.
2. Clicking or hovering the swatch opens a color picker next to it.
3. The picker has a saturation box, a hue strip and an opacity strip, and dragging in them rewrites the color text in the code as you go.
4. A button in the picker cycles the color between hex, rgb and hsl formats.
5. Setting `editor.colorDecorators` chooses whether swatches and the color picker show at all, and it is on by default.
6. Setting `editor.colorDecoratorsActivatedOn` chooses how the picker opens, with values `clickAndHover` (the default), `hover` and `click`.
7. Setting `editor.colorDecoratorsLimit` chooses the most swatches drawn in one editor at once, with 500 as the default.
8. Setting `editor.defaultColorDecorators` chooses when the built-in color finder is used, with values `auto` (only when no extension supplies colors), `always` and `never`.

### ED-4.30
1. Running Show or Focus Standalone Color Picker from the Command Palette opens a color picker at the text cursor even when no color is written there (`editor.action.showOrFocusStandaloneColorPicker`).
2. Running the command again while the picker is open moves focus into it.
3. Pressing Enter in the picker inserts the chosen color into the code as text (`editor.action.insertColorWithStandaloneColorPicker`).
4. The picker's format button chooses whether the inserted text is hex, rgb or hsl.
5. Pressing Escape hides the picker (`editor.action.hideColorPicker`).

### ED-4.31
1. Peek Call Hierarchy opens an inline panel under the line titled "Callers of '<name>'" that lists the callers of the symbol beside a preview of each call site.
2. Peek Call Hierarchy is in the Peek submenu of the editor right-click menu only when a call hierarchy provider exists, and it is also in the Command Palette.
3. The Show Outgoing Calls title button switches the list to the calls the symbol makes, and the Show Incoming Calls title button switches it back (`editor.showOutgoingCalls`, `editor.showIncomingCalls`).
4. Pressing Shift+Alt+H with the cursor in code opens the panel, and while the panel shows it swaps between incoming and outgoing calls (`editor.showCallHierarchy`).
5. Refocus Call Hierarchy, or Shift+Enter, makes the focused entry the new root of the list (`editor.refocusCallHierarchy`).
6. Pressing Enter on an entry or double-clicking it opens that entry in the editor.
7. Pressing Escape closes the panel unless `editor.stablePeek` is on (`editor.closeCallHierarchy`).
8. The next panel opens with the last direction used and the panel layout you left.

### ED-4.32
1. Peek Type Hierarchy opens an inline panel under the line that lists the subtypes of the type under the cursor beside a preview.
2. Peek Type Hierarchy is in the Peek submenu of the editor right-click menu only when a type hierarchy provider exists, and it is also in the Command Palette, and no key opens it.
3. The Show Supertypes title button switches the list to the supertypes, and the Show Subtypes title button switches it back (`editor.showSupertypes`, `editor.showSubtypes`).
4. While the panel shows, pressing Shift+Alt+H swaps between supertypes and subtypes (`editor.showTypeHierarchy`).
5. Refocus Type Hierarchy, or Shift+Enter, makes the focused entry the new root of the list (`editor.refocusTypeHierarchy`).
6. Pressing Enter on an entry or double-clicking it opens that entry in the editor.
7. Pressing Escape closes the panel unless `editor.stablePeek` is on (`editor.closeTypeHierarchy`).
8. The next panel opens with the last direction used and the panel layout you left.

### ED-4.33
1. When a go-to command finds several places and its setting is `goto`, the editor jumps to the first one and the status bar reads "Symbol 1 of 3, F12 for next".
2. Pressing F12 while stepping moves to the next place and wraps around after the last one (`editor.gotoNextSymbolFromResult`).
3. Pressing Escape ends the stepping (`editor.gotoNextSymbolFromResult.cancel`).
4. Moving the cursor to some other spot ends the stepping.
5. Setting `editor.gotoLocation.multipleDefinitions`, `editor.gotoLocation.multipleDeclarations`, `editor.gotoLocation.multipleTypeDefinitions`, `editor.gotoLocation.multipleImplementations` or `editor.gotoLocation.multipleReferences` to `goto` turns the stepping on for that command, as does the deprecated `editor.gotoLocation.multiple`.

### ED-4.34
1. When a go-to command such as Go to Definition or Go to Implementations finds nothing, a small bubble fades in beside the cursor with a message such as "No definition found for '<name>'" or "No implementation found for '<name>'".
2. The same bubble gives the reason when a rename or a disabled code action cannot run, and "No related tests found." when there are no related tests.
3. The warning shown when you type in a read-only editor uses the same bubble.
4. Pressing Escape closes the bubble (`leaveEditorMessage`).
5. The bubble also closes when the cursor moves or when the editor loses focus.
6. The bubble also closes when the pointer drifts far away from it.

### ED-4.35
1. Right-clicking code and choosing Go to Related Test jumps to a test that covers it, when a test extension supplies the links, and the entry sits in the testing group of the right-click menu (`testing.goToRelatedTest`).
2. Choosing Peek Related Test opens the related tests in an inline peek under the line (`testing.peekRelatedTest`).
3. Right-clicking inside a test and choosing Go to Related Code or Peek Related Code reaches the code the test exercises (`testing.goToRelatedCode`, `testing.peekRelatedCode`).
4. The go variants jump straight to the place when there is one result and show a peek when there are several.
5. When there is nothing related, the message bubble reads "No related tests found." or "No related code found.".

### ED-4.36
1. When some refactorings or source actions cannot run at the cursor, the bar at the bottom of the Quick Fix, Refactor and Source Action menus shows a Show Disabled row (`showMoreActions`).
2. Choosing Show Disabled adds those actions to the menu in gray, each with the reason it cannot run.
3. After that the row reads Hide Disabled, and choosing it takes the gray actions out again (`hideMoreActions`).
4. Documentation links supplied by a language extension sit in the same bottom bar.

### ED-4.37
1. Dragging an edge or corner of the hover card makes it taller or wider.
2. Which edges can be dragged depends on whether the card opened above or below the text.
3. Later hovers in the same window open at the size you chose, until the window reloads.

### ED-4.38
1. Dragging the bottom edge of the peek window changes how many lines tall it is.
2. Dragging the divider between the preview and the results list changes how the width is shared between them.
3. A peek window opens 18 lines tall with the preview taking 70 percent of the width the first time.
4. The next peek opens with the height and width split you left.

### ED-4.39
1. With a screen reader, focusing the hover card (the Show or Focus Hover key twice) and then pressing Alt+F2 opens its content as plain text in a separate accessible view.
2. The accessible view has plus and minus actions that show more or less detail (`editor.action.increaseHoverVerbosityLevelFromAccessibleView`, `editor.action.decreaseHoverVerbosityLevelFromAccessibleView`).
3. Pressing Alt+F1 while a hover shows opens accessibility help that lists the hover actions that exist (`editor.action.accessibilityHelp`).
4. On Linux the primary keys are Alt+Shift+F2 and Alt+Shift+F1, with the Alt-only keys as second choices.
5. Open Accessible View and Open Accessibility Help are in the Command Palette and work for the hover (`editor.action.accessibleView`).
6. Setting `accessibility.verbosity.hover` adds a spoken hint with the key to press when the hover card takes focus.

### ED-4.40
1. Pressing Ctrl+Alt+F8 (Cmd+Alt+F8 on Mac) with the cursor on a squiggled problem opens the inline problem widget right there with its message, source and code, instead of stepping to the next problem. [cursor only]
2. When the problem is off screen, the view scrolls to center it. [cursor only]
3. When the cursor is not on a problem, nothing opens. [cursor only]
4. Go to Problem at Cursor (Error, Warning, Info) is in the Command Palette (`editor.action.marker.this`). [cursor only]

### ED-4.41
1. Pressing Undo (Ctrl+Z, the Edit menu or the Command Palette) after a rename or refactor that changed several files asks "Would you like to undo '<name>' across all files?".
2. The prompt has the buttons Undo in N Files, Undo this File and Cancel.
3. Undo in N Files undoes the change in every file, Undo this File undoes only the current file's part, and Cancel undoes nothing.
4. When some of those files changed since, a warning lists them and nothing is undone.
5. Pressing Redo (Ctrl+Y or Ctrl+Shift+Z, Cmd+Shift+Z on Mac) after such a refactor gives the matching "Could not redo" warnings.
6. When the edit came from a different source, Undo first asks "Would you like to undo '<name>'?" with Yes and No buttons.

### ED-4.42
1. With setting `problems.sortOrder` at `severity`, the default, Go to Next Problem and Go to Previous Problem walk each file's problems errors first, then warnings, then infos.
2. With `problems.sortOrder` at `position`, they walk each file's problems in the order they appear in the text.
3. The same setting sorts the Problems list the same way.

## Unclear
- ED-4.3 piece 4 (web-only Ctrl+F12 keys) was not checked in the Cursor bundle, so it has no Cursor tag and may not apply to the desktop app.
- ED-4.9 lists `editor.gotoLocation.multiple` as a setting, but it is deprecated and is covered only in ED-4.3 piece 6, so ED-4.9 has no piece of its own for it.
- ED-4.12 piece 6 depends on a provider that supplies name suggestions, and none was confirmed in any built-in extension.
- ED-4.17 piece 6 includes an Install Formatter... button that leads to the extension marketplace, which is out of scope, so only the message and the button are counted.
- ED-4.29 piece 3 says dragging in the picker rewrites the color text live, which follows the feature's description and was not re-read in `hoverColorPickerParticipant.ts`.
- ED-4.39 piece 5 covers Command Palette entries shared with every accessible view, so a tracking tool may want to count them under a shared accessibility feature instead.
- ED-4.1 piece 3 treats the plus and minus buttons as one piece, though each has its own tooltip with a key hint.
