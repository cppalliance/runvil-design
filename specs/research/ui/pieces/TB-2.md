Group: TB-2 Command center and window title. Pass: 3. Date: 2026-10-25.

### TB-2.1
1. Clicking the search-style box in the middle of the title bar opens Quick Open with the help entry included.
2. Setting `window.commandCenter` defaults to on in stock, so the box shows without any setup. [stock only]
3. Setting `window.commandCenter` defaults to off in Cursor, so the box is not shown until it is turned on. [cursor only]
4. In Cursor the click always opens classic Quick Open. [cursor only]
5. Clicking the search-style box in the middle of the title bar runs `workbench.action.quickOpenWithModes` and opens Quick Open with the shortcut rows above the recent files.
6. Hovering the box shows a tip that reads "Search <workspace name> (<key>)" followed by the window title, and the key part is left out when the command has no key.
7. The box is exposed to screen readers as a button whose description is that same tip text.
8. Setting `window.commandCenter` shows or hides the box in the title bar, with values true and false, and it defaults to true in stock.
9. In Cursor `window.commandCenter` defaults to false so the box stays hidden until you turn it on, and a click on the box always opens classic Quick Open. [cursor only]

### TB-2.2
1. Setting `window.commandCenter` turned on shows the search-style box in the middle of the title bar, and turned off collapses the middle of the bar to plain window title text.
2. Turning the box off lowers the title bar from 35 px to 30 px.
3. The right-click row Command Center (`toggle.window.commandCenter`) flips the setting, shows a check mark while the box is on, and appears when the title bar is not compact. [stock only]
4. In stock, turning on Navigation Controls or the experimental Share setting turns the box on if it was off.
5. Compact floating editor windows never draw the box.

### TB-2.3
1. The box shows the workspace name.
2. With editor tabs hidden (`workbench.editor.showTabs` set to none) the box shows the active file name instead of the workspace name.
3. With a customized `window.title` or with a screen reader on, the box shows the whole formatted title.
4. When there is nothing to show, the box reads "Search".
5. The extension-development marker is added before the label and the admin marker after it.
6. Line breaks in the label show as a return symbol.
7. Hovering the box shows "Search {workspace} ({key})" followed by a long dash and the title, and the key part appears only when Quick Open has a keybinding.
8. The box colors come from the `commandCenter.*` theme colors.
9. In Cursor the box always shows a search icon before the label.

### TB-2.4
1. Two arrow buttons to the left of the box step back and forward through editor navigation history.
2. Each arrow is disabled when there is nowhere to go, following the context keys `canNavigateBack` and `canNavigateForward`.
3. Back runs from the keyboard with Alt+Left on Windows, Ctrl+- on macOS, Ctrl+Alt+- on Linux, and the browser Back key.
4. Forward runs from the keyboard with Alt+Right on Windows, Ctrl+Shift+- on macOS and Linux, and the browser Forward key.
5. The Go menu also has Back and Forward rows in its history group.
6. In stock the arrows live on the command center menu, so they show only while the box is on and `workbench.navigationControl.enabled` is on.
7. In Cursor, when the box is off, the same two commands show as buttons in Cursor's left title bar toolbar and use the same enabled and disabled rules. [cursor only]

### TB-2.5
1. The right-click row Navigation Controls hides or shows the back and forward arrows.
2. The Navigation Controls row shows a check mark while the arrows are on.
3. The row appears only when the title bar is not compact and the command center is on.
4. Setting `workbench.navigationControl.enabled` does the same from settings, and it defaults to on.
5. In Cursor the same setting also gates the arrows in Cursor's left toolbar when the box is off. [cursor only]

### TB-2.6
1. Right-clicking anywhere on the title bar opens a menu of title bar toggles.
2. Right-clicking on the box or on a toolbar button in the title bar opens the same menu.
3. On Windows with a custom title bar, the OS system-menu request opens this same menu.
4. The toggle rows sit together in one section, and the Hide Custom Title Bar rows sit in a separate section below it.
5. In Cursor the menu has no Command Center, Notifications, Integrated Browser, Hide Custom Title Bar or Hide Custom Title Bar In Full Screen row. [stock only]

### TB-2.7
1. On macOS, right-clicking the title text opens a second menu with the same toggles plus one row for each folder in the active file's path and one row for the file.
2. On macOS, Cmd+click on the title text opens the same second menu.
3. Picking a path row reveals that folder or file in Finder.
4. The path rows appear only with a custom title bar and an active file.
5. Cmd+click on the title text does not open Quick Open.

### TB-2.8
1. With the box off, the middle of the title bar shows the window title string as centered plain text.
2. Title text that is too long is clipped with an ellipsis.
3. The title text updates as the user switches editors.
4. With a native title bar, the custom bar draws no title text because the OS draws it.
5. In Cursor with a workspace open, the plain title text is replaced by a workspace name button (see TB-2.9). [cursor only]

### TB-2.9
1. With the box off and a workspace open, the middle of the title bar is a text button showing the workspace name. [cursor only]
2. When `window.title` is customized, the button shows the full title instead of the workspace name. [cursor only]
3. A left-button press on the button opens Quick Open. [cursor only]
4. Pressing Enter or Space while the button has focus opens Quick Open. [cursor only]
5. The button is excluded from window dragging, so a click on it never starts a drag. [cursor only]
6. With no workspace open the button is not shown, and the plain title text or product name appears instead. [cursor only]
7. A left click on the button opens Quick Open with the shortcut rows, and it fires on mouse down. [cursor only]
8. The button reads the workspace name, or the full window title when `window.title` has been set to a custom format. [cursor only]
9. The button text is also its screen reader name, and pressing on it opens Quick Open instead of starting a window drag. [cursor only]
10. With no workspace open the middle of the title bar shows plain title text that does nothing when clicked. [cursor only]

### TB-2.10
1. Setting `window.title` sets the text in the OS window title, which shows in the taskbar, Alt-Tab and Mission Control.
2. The same template sets the title text in the title bar.
3. The template accepts the editor variables `${activeEditorShort}`, `${activeEditorMedium}`, `${activeEditorLong}` and `${activeEditorLanguageId}`.
4. The template accepts the folder variables `${activeFolderShort}`, `${activeFolderMedium}`, `${activeFolderLong}`, `${folderName}` and `${folderPath}`.
5. The template accepts the workspace variables `${rootName}`, `${rootNameShort}` and `${rootPath}`.
6. The template accepts the state variables `${profileName}`, `${appName}`, `${remoteName}`, `${dirty}`, `${focusedView}`, `${activeEditorState}` and `${separator}`.
7. On Windows and Linux the default template shows dirty, active file, root name, profile name and app name joined by separators.
8. On macOS desktop the default template leaves out the dirty marker and the app name.
9. An empty result falls back to the product name.

### TB-2.11
1. Setting `window.titleSeparator` sets the text that the `${separator}` variable shows in the title.
2. On Windows and Linux the default separator is a hyphen with a space on each side.
3. On macOS the default separator is an em dash with a space on each side.
4. The separator shows only between values that are not empty.

### TB-2.12
1. The variable `${activeRepositoryName}` shows the name of the active Source Control repository.
2. The variable `${activeRepositoryBranchName}` shows the branch name of the active Source Control repository.
3. Adding both variables to `window.title` makes the OS window title and the title bar text show the repository and branch.

### TB-2.13
1. The default title starts with a filled dot when the active editor has unsaved changes.
2. On macOS desktop the default title shows no dot, because macOS shows its own indicator.
3. In an extension development window the title is prefixed with "[Extension Development Host]".
4. When the app runs elevated, the title ends with "[Administrator]" on Windows and "[Superuser]" elsewhere.
5. Non-space whitespace in the title is replaced with spaces.

### TB-2.14
1. With a screen reader detected and `accessibility.windowTitleOptimized` on, the title gets a separator and the active editor state, for example "modified", added at the end.
2. The state is not added again when the `window.title` template already has `${activeEditorState}`.
3. With `accessibility.windowTitleOptimized` off, the title gets no extra state.
4. In this case the command center shows the full title instead of the workspace name.

### TB-2.15
1. When a quick input opens at the top of the window, the box is hidden but keeps its space, so nothing shifts.
2. The box returns when the quick input closes.
3. In stock the box stays visible when the quick input is centered. [stock only]
4. In Cursor the box hides on every quick input in the active window, with no check on where the input appears. [cursor only]

### TB-2.16
1. With `workbench.experimental.share.enabled` on, a share provider available and a folder open, a Share icon appears in the command center. [stock only]
2. Clicking the Share icon builds a link to the active file or selection and copies it. [stock only]
3. The link is shown in a dialog after it is copied. [stock only]
4. Pressing Alt+Ctrl+S, or Alt+Cmd+S on macOS, runs Share from the keyboard.
5. In Cursor the Share command, its shortcut and the setting still exist, but no Share icon appears in the command center. [stock only]

## Unclear

- TB-2.1: the stock branch that opens the unified quick access when `chat.unifiedAgentsBar.enabled` is on is a chat feature and has no piece here, and QA-5 overlaps with piece 1.
- TB-2.2: whether Cursor also turns the box on when Navigation Controls are turned on was not read, so piece 4 has no Cursor tag.
- TB-2.3: stock's compact look of the box, which hides the icon, depends on the chat setting `chat.agentsControl.enabled` and is left out, so piece 9 only says Cursor always shows the icon.
- TB-2.4: Cursor's left toolbar may also be listed by TB-4 or TB-5, and only the two arrows are written here.
- TB-2.6: the exact rows and order of Cursor's right-click menu were not fully decoded, and the Restore Native Title Bar and Hide Custom Title Bar rows belong to TB-3.
- TB-2.1 to TB-2.16: the Cursor side is from strings and minified code in the bundle, and Cursor was not run.
