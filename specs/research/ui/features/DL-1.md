Group: DL-1 Message, confirm and input boxes, and About. Section: Dialogs. Date: 2026-10-25. Pass: 2.
Features: 38.

### DL-1.1 Confirm box with a primary button and Cancel
- Tag: both
- Evidence: source
- Boundary: no
- What: A yes-or-cancel box with a message, an optional detail line, a primary button (default label "Yes", or a caller label such as "Clear") and a Cancel button. The answer is confirmed or not confirmed. Other features raise it, so it has no entry point of its own.
- Commands: none found
- Menus: none found
- Keys: Enter (primary button); Escape (cancel) - both only while the in-page box is open, see DL-1.12
- Settings: window.dialogStyle
- Cursor: present
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (IConfirmation, getConfirmationButtons); V\src\vs\workbench\services\dialogs\common\dialogService.ts (confirm); V\src\vs\workbench\electron-browser\parts\dialogs\dialog.contribution.ts (processDialogs); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts; V\src\vs\workbench\electron-browser\parts\dialogs\dialogHandler.ts; C bundle (contribution workbench.contrib.dialogHandler, confirm branch of processDialogs)

### DL-1.2 Checkbox on a box
- Tag: both
- Evidence: source
- Boundary: no
- What: Any confirm, prompt or input box can carry one checkbox under the message, and the answer reports whether it was ticked. Clicking the label text toggles it too. Examples seen: "Do not ask me again" on the quit and close confirm, "Always discard untitled workspaces without asking", "Remember my credentials" on the proxy login, "Remember my choice in user settings" on the pre-launch task prompt, and "Don't restore editors" on the window crash box.
- Commands: none found
- Menus: none found
- Keys: Space (toggles the focused checkbox); Tab (moves to it)
- Settings: window.confirmBeforeClose; window.confirmSaveUntitledWorkspace
- Cursor: present
- Source: V\src\vs\base\browser\ui\dialog\dialog.ts (checkbox row, lines 217-229); V\src\vs\platform\dialogs\common\dialogs.ts (ICheckbox, ICheckboxResult); V\src\vs\workbench\browser\window.ts (confirmOnShutdown); V\src\vs\workbench\services\workspaces\electron-browser\workspaceEditingService.ts; C bundle (class with .dialog-checkbox-row; nls 4073, 13618, 12119, 5914, 2721)

### DL-1.3 Multi-button prompt
- Tag: both
- Evidence: source
- Boundary: no
- What: A box with two or more custom buttons and an optional Cancel. The first button is the default. Each button returns its own value to the caller, so one box can mean Save, Don't Save or Cancel. With no buttons given it falls back to a single OK.
- Commands: none found
- Menus: none found
- Keys: Enter (default button); Escape (cancel button); Left and Right arrows or Tab (move between buttons), see DL-1.12
- Settings: window.dialogStyle
- Cursor: present
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (IPrompt, getPromptButtons, getPromptResult); V\src\vs\workbench\services\dialogs\common\dialogService.ts (prompt); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (prompt); V\src\vs\workbench\electron-browser\parts\dialogs\dialogHandler.ts (prompt); C bundle (processDialogs promptArgs branch)

### DL-1.4 Info, warning and error boxes
- Tag: both
- Evidence: source
- Boundary: no
- What: One-button boxes that just tell you something: an info, warning or error icon, a message, an optional detail and an OK button. They are sent through the same prompt path. Cursor adds a fourth call that forces the in-page box for an error, but I found no caller of it in the editor bundle.
- Commands: none found
- Menus: none found
- Keys: Enter or Escape closes it
- Settings: window.dialogStyle
- Cursor: changed - adds a customError call that forces the in-page box
- Source: V\src\vs\workbench\services\dialogs\common\dialogService.ts (info, warn, error); V\src\vs\platform\dialogs\common\dialogs.ts (getDialogType); C bundle (class with info, warn, error and customError methods next to async about)

### DL-1.5 Detail text under the message
- Tag: both
- Evidence: source
- Boundary: no
- What: Any box can show a smaller detail paragraph under the bold message, for example "Your changes will be lost if you don't save them." In the OS box it is the native detail text, in the in-page box it sits under the icon and message. Plain text only in 1.128.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (IBaseDialogOptions.detail); V\src\vs\base\browser\ui\dialog\dialog.ts (messageDetailElement, lines 170-182); V\src\vs\workbench\electron-browser\parts\dialogs\dialogHandler.ts; C bundle (class with #monaco-dialog-message-detail)

### DL-1.6 Rich in-page body, icon and button captions
- Tag: both
- Evidence: source
- Boundary: no
- What: A caller can ask for the in-page box on purpose and then add its own body: extra CSS classes, rendered Markdown paragraphs with clickable links, a custom icon, small caption text under each button, and a switch that removes the close X and Escape. Asking for it overrides the native style. The Workspace Trust prompts use it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.dialogStyle
- Cursor: changed - button captions use an older option name (buttonDetails) in the widget
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (ICustomDialogOptions, lines 287-300); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (doShow, renderBody); V\src\vs\base\browser\ui\dialog\dialog.ts (buttonOptions, disableCloseAction); C bundle (in-page handler passing buttonDetails, markdownDetails, icon, disableCloseAction to the dialog widget)

### DL-1.7 Input box with text fields
- Tag: both
- Evidence: source
- Boundary: no
- What: A box with one or more text fields (placeholder, optional starting value, plain or password type), an optional checkbox, a primary button (default "OK") and Cancel. The first field is focused and selected on open, and Enter inside a field accepts. It is always the in-page box, because Electron has no native version and the OS handler throws "Unsupported".
- Commands: none found
- Menus: none found
- Keys: Enter (accept while a field has focus); Tab (next field)
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (IInput, IInputElement); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (input); V\src\vs\workbench\electron-browser\parts\dialogs\dialogHandler.ts (input throws); V\src\vs\workbench\electron-browser\parts\dialogs\dialog.contribution.ts (input is custom only); V\src\vs\base\browser\ui\dialog\dialog.ts (inputs, lines 196-213 and 560-566); C bundle (inputArgs branch always uses the in-page handler; class with .dialog-message-input)

### DL-1.8 Proxy authentication login box
- Tag: both
- Evidence: source
- Boundary: no
- What: When a network request hits a proxy that wants a login, a warning box titled "Proxy Authentication Required" opens with the text "The proxy host:port requires a username and password.", a Username field, a Password field, a "Remember my credentials" checkbox, a "Log In" button and Cancel. Cancelling replies to the main process with no credentials. The checkbox is remembered under the application storage key window.rememberProxyCredentials, which is not a setting.
- Commands: none found
- Menus: none found
- Keys: Enter (log in); Escape (cancel)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\electron-browser\window.ts (vscode:openProxyAuthenticationDialog, lines 293-333); V\src\vs\platform\native\electron-main\auth.ts (sends the request); C bundle (listener for vscode:openProxyAuthenticationDialog, nls 12114 to 12119); C out\main.js (sendWhenReady for the same channel)

### DL-1.9 Dialog style: native or custom
- Tag: both
- Evidence: source
- Boundary: no
- What: One setting decides whether confirm, prompt and info boxes are drawn by the OS or by the app. Default is native. Custom draws the in-page box instead. Input boxes are always in-page. Some callers force the in-page box themselves. In Cursor the About box ignores this setting (it always opens its own window), and Cursor's second dialog engine is not controlled by it either.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.dialogStyle
- Cursor: changed - different description text; About always uses the custom window; no smoke-test-driver clause
- Source: V\src\vs\workbench\electron-browser\desktop.contribution.ts (window.dialogStyle, lines 291-297); V\src\vs\workbench\electron-browser\parts\dialogs\dialog.contribution.ts (useCustomDialog, lines 59-110); C bundle (setting window.dialogStyle with nls 12063, contribution Toi with useCustomDialog and an About branch that always calls the native handler)

### DL-1.10 Button order and default button per OS
- Tag: both
- Evidence: source
- Boundary: no
- What: The same box lists its buttons in a different order on each OS. Windows keeps Cancel last. macOS and Linux put the default button on the trailing side and Cancel just left of it, and Linux also flips the whole row. The OS box sets title to the app name when none is given and turns off link-style buttons. The in-page box applies the same arrangement itself.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (massageMessageBoxOptions, lines 633-740); V\src\vs\platform\dialogs\electron-main\dialogMainService.ts (showMessageBox, line 149); V\src\vs\base\browser\ui\dialog\dialog.ts (rearrangeButtons, lines 632-675); C out\main.js (massage code that sets noLink, defaultId and cancelId)

### DL-1.11 Modal block: one box at a time, window dimmed
- Tag: both
- Evidence: source
- Boundary: no
- What: While a box is open the window cannot be used. Boxes wait in a queue and show one after another, per window. The in-page box covers the window with a dimmed layer, and clicking the layer sends focus back into the box. On Windows and Linux with drawn window controls, the minimize, maximize and close buttons are dimmed to match while the box is up.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.titleBarStyle; window.controlsStyle
- Cursor: changed - queue is present, but the window-controls dimming is missing (no setWindowDimmed and no dimmed option in the main process)
- Source: V\src\vs\workbench\services\dialogs\common\dialogService.ts (DialogsModel queue); V\src\vs\workbench\electron-browser\parts\dialogs\dialog.contribution.ts (processDialogs loop); V\src\vs\platform\dialogs\electron-main\dialogMainService.ts (windowDialogQueues, line 51); V\src\vs\workbench\browser\parts\dialogs\dialog.ts (onVisibilityChange); V\src\vs\platform\windows\electron-main\windowImpl.ts (updateWindowControls, dimColor, lines 412-470); V\src\vs\platform\window\common\window.ts (useWindowControlsOverlay); C bundle (class with .monaco-dialog-modal-block.dimmed; zero hits for setWindowDimmed); C out\main.js (queue names present, zero hits for dimmed)

### DL-1.12 Keyboard in the in-page box
- Tag: both
- Evidence: source
- Boundary: no
- What: Enter accepts while a text field has focus. Escape closes the box as Cancel. Tab, Shift+Tab, Left and Right arrows walk through links, text fields, the checkbox and the buttons in visual order and wrap around. On macOS, Cmd+D presses the second button (No or Don't Save). Space is left to the focused control. Almost every other shortcut is blocked while the box is up. Only copy, cut, select all and paste pass through (the pending box also lets quit and reload through). In later stock builds the arrow keys stop moving focus while you type in a field (newer-than-1.128, not in Cursor).
- Commands: none found
- Menus: none found
- Keys: Enter; Escape; Tab; Shift+Tab; Left Arrow; Right Arrow; Cmd+D (macOS only); Space
- Settings: none found
- Cursor: changed - key handling skips when the box is not on screen, and Escape closes on key up without checking for a prior key down
- Source: V\src\vs\base\browser\ui\dialog\dialog.ts (keydown and keyup handlers, lines 336-488); V\src\vs\workbench\browser\parts\dialogs\dialog.ts (defaultDialogAllowableCommands, createWorkbenchDialogOptions); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (ALLOWABLE_COMMANDS); N\src\vs\base\browser\ui\dialog\dialog.ts (isEditableTarget); C bundle (widget class with the Cmd+D branch and isOnScreen guard; list of allowed commands)

### DL-1.13 Focus on open, keep, and return
- Tag: both
- Evidence: source
- Boundary: no
- What: The in-page box focuses its first text field (selected) or its default button when it opens. Focus cannot leave the box: if it tries to, it is pulled back. When the box closes, focus goes back to the element that had it before. A dirty-editor close prompt first brings the window to the front so the box is not hidden.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\base\browser\ui\dialog\dialog.ts (focus at lines 560-571, focusout at 491-505, focus return at 626-628); V\src\vs\workbench\browser\parts\editor\editorGroupView.ts (hostService.focus before the save prompt); C bundle (widget class with focusToReturn and a focusout handler)

### DL-1.14 Close button on the in-page box
- Tag: both
- Evidence: source
- Boundary: no
- What: A small X in the top right of the in-page box, named "Close Dialog", closes it and counts as Cancel. It is not drawn when the caller turns the close action off, and then Escape does nothing either.
- Commands: none found
- Menus: none found
- Keys: Escape (same result, see DL-1.12)
- Settings: none found
- Cursor: present
- Source: V\src\vs\base\browser\ui\dialog\dialog.ts (dialog.close action, lines 537-548); C bundle (action id dialog.close, nls 7 "Close Dialog")

### DL-1.15 Accessibility of the in-page box
- Tag: both
- Evidence: source
- Boundary: no
- What: The box is marked as a modal dialog for screen readers. It is labelled by its icon and message, and described by the message, detail, body and footer. The icon has a spoken name (Info, Warning, Error or In Progress). Links in the body and footer are made keyboard focusable and underlined so colour is not the only cue. Cursor's second dialog engine does not set a dialog role (see DL-1.25).
- Commands: none found
- Menus: none found
- Keys: Tab (reaches links)
- Settings: none found
- Cursor: present
- Source: V\src\vs\base\browser\ui\dialog\dialog.ts (role, aria-modal, aria-labelledby, aria-describedby at lines 136 and 552-554; getIconAriaLabel at 238-257; link styling in applyStyles); C bundle (widget class with the same aria-labelledby id list)

### DL-1.16 Pending box with spinner and Cancel
- Tag: both
- Evidence: source
- Boundary: no
- What: A box with a spinning icon and a changing message while work runs. It has a Cancel or Dismiss button, unless it is set to sticky, and the message updates as the task reports. It is the box look used for work shown at the dialog progress location, such as the waiting boxes at shutdown (see DL-2.8). The progress side of it is NT-5.10.
- Commands: none found
- Menus: none found
- Keys: Escape (cancel unless sticky); Enter (default button)
- Settings: none found
- Cursor: changed - the pending box is built without the host service, so no window-controls dimming (also noted in NT-5)
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (withDialogProgress, lines 549-615); V\src\vs\base\browser\ui\dialog\dialog.ts (pending type, spin class, updateMessage); C bundle (withDialogProgress calling the options factory with two arguments)

### DL-1.17 Open the About box
- Tag: both
- Evidence: source
- Boundary: no
- What: Help > About (Windows and Linux), the "About {name}" row at the top of the macOS app menu, or "Help: About" in the Command Palette opens the About box. On Windows and Linux the update row sits just above it in the Help menu. In stock it is an OS message box, or the in-page box when dialog style is custom. In Cursor it opens its own small window instead (DL-1.20), ignores dialog style, and only falls back to an OS box if that window fails to open.
- Commands: workbench.action.showAboutDialog
- Menus: MenubarHelpMenu (group z_about, order 1, hidden on macOS native menus); CommandPalette (f1, category Help); macOS application menu (built in the main process)
- Keys: none found
- Settings: window.dialogStyle
- Cursor: changed - custom About window with an OS box as fallback
- Source: V\src\vs\workbench\browser\actions\windowActions.ts (ShowAboutDialogAction, lines 387-411); V\src\vs\workbench\services\dialogs\common\dialogService.ts (about); V\src\vs\workbench\electron-browser\parts\dialogs\dialog.contribution.ts (About branch); V\src\vs\platform\menubar\electron-main\menubar.ts (mAbout at 409, insertCheckForUpdatesItems at 521); V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts (insertActionsBefore, line 447); C bundle (command workbench.action.showAboutDialog, nls 3265 and 3257, native handler about with Failed to open supplemental About window); C out\main.js (command id found, 2 hits)

### DL-1.18 Copy version info from About
- Tag: both
- Evidence: source
- Boundary: no
- What: The About box has a Copy button that puts every version line on the clipboard and an OK button that closes it. The copied text is plain, without the "x days ago" note shown on screen. In Cursor the main button is "Copy version info", it copies and closes the window, and Enter triggers it. The OS fallback box keeps the stock Copy and OK buttons.
- Commands: workbench.action.showAboutDialog
- Menus: none found
- Keys: Enter (Cursor About window: copy and close); Escape (Cursor About window: close); C mnemonic on the Copy button (OS box on Windows and Linux)
- Settings: none found
- Cursor: changed - primary "Copy version info" button in a custom window, Copy and OK only in the fallback box
- Source: V\src\vs\workbench\electron-browser\parts\dialogs\dialogHandler.ts (about, lines 67-84); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (about, lines 76-92); C bundle (nls 3338 "Copy version info", 12098 "&&Copy", 12099 "OK"; Enter and Escape handler of the About window component)

### DL-1.19 Version lines in About
- Tag: both
- Evidence: source
- Boundary: no
- What: Stock lists Version (with a "user setup", "system setup" or "Universal" note when known), Commit, Date (plus "n days ago" on screen only), Electron, ElectronBuildId, Chromium, Node.js, V8 and OS. Cursor copies a different list: Version, VS Code Extension API, Commit, Date, Layout, Build Type (Stable, Nightly, Dev/Debug), Release Track (Default, Early Access, Nightly, Dogfood, Candidate), Electron, Chromium, Node.js, V8, xterm.js and OS, plus "Local Mode: enabled" when local mode is on. Cursor has no ElectronBuildId. A later stock build adds two Copilot package lines (newer-than-1.128, not in Cursor).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - different set of lines, see What
- Source: V\src\vs\platform\dialogs\electron-browser\dialog.ts (createNativeAboutDialogDetails, lines 13-45); V\src\vs\workbench\browser\parts\dialogs\dialog.ts (web text, lines 48-67); N\src\vs\platform\dialogs\electron-browser\dialog.ts (Copilot lines); C bundle (native handler about, nls 12100 and 12101, getBuildType, owy release track labels, withLocalModeDetail, nls 12097)

### DL-1.20 Cursor About window
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor's About is a small window about 380 px wide, with no title, no full screen, a see-through background and a drag strip along the top. It shows the app icon, the product name, "Version x", the build date and a copyright line. Hovering the version or the date shows a copy icon that copies that one value and flips to a check mark for about 1.5 seconds. The window resizes to fit, closes on Escape, closes when the app shuts down, and a second About request reuses the open window.
- Commands: workbench.action.showAboutDialog
- Menus: MenubarHelpMenu; CommandPalette
- Keys: Escape (close); Enter (copy version info and close)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (About window components v5w, f5w and the copy helper Tvg, opener _5w, data-component glass-about-dialog-window, about-dialog-drag-region, glass-about-dialog-body; nls 3334 to 3337 and 12102); C out\main.js (revealAboutWindow, fitAboutWindow, updateAboutWindowControls)

### DL-1.21 Update button in the About window
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Next to "Copy version info" the About window has a button whose label and action follow the update state: check for updates, download, install, or restart to update. It shows a spinning ring while checking, downloading or installing, becomes "No updates available" (disabled) after an explicit check that finds nothing, and "Updates unavailable" when updates are off. Restarting is called with the source about_dialog. The title-bar Update control is TB-5.
- Commands: update.checkForUpdate; update.downloadUpdate; update.installUpdate; update.restartToUpdate
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component g5w, state table e5w, labels t5w and n5w, label function lgw, data-component glass-about-dialog-update-button)

### DL-1.22 Cursor's second dialog engine (pretty dialog)
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor draws its own in-page modal for its own boxes. It has a bold title, a message that can be plain text, a list of blocks (plain, callout with icon and colour, copyable value, Markdown), an optional icon, a primary button (can be styled as danger), extra secondary buttons, a Cancel button, optional text fields, an optional checkbox, and an optional "confirm" checkbox that must be ticked before the primary button works. It fades in below the top edge with a half-dark backdrop. It is separate from the stock box: the dialog style setting does not apply to it, and it is not created in the Agents window. Only the non-agent callers belong in this catalog, see DL-2.17 and DL-2.18 for quit and close.
- Commands: none found
- Menus: none found
- Keys: Enter; Shift+Enter; Escape (see DL-1.25)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (service class X7 registered as D0 with openDialog, updateDialog, closeDialog, confirm, prompt, input, info, warn, error, about; view GMk with wrapper NMk; css classes pretty-dialog, pretty-dialog-modal, pretty-dialog-title, pretty-dialog-message-callout, pretty-dialog-copyable-value, pretty-dialog-input, pretty-dialog-button-primary; z-index 2551 backdrop)

### DL-1.23 "Don't ask again" memory in the pretty dialog
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: When a box is given a key and asks for it, it shows a "Don't ask again" checkbox (the label and hover text can change). The checkbox sits next to the buttons on the quit and close boxes and under the message elsewhere. If you tick it and then pick a button, that button is saved under the key and the box never opens again: the saved answer is used at once. Cancel is saved only when the box allows it. The saved answers live in Cursor's application storage under dialogDontAskAgainPreferences and are not settings.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (openDialog reading dialogDontAskAgainPreferences, handler y in view GMk, flags shouldShowDontAskAgain, saveDontAskAgainOnCancel, checkboxAtBottom, dontAskAgainLabel, dontAskAgainTooltip, doNotCallOnAcceptOnDontShowAgain)

### DL-1.24 Reset "Don't Ask Again" dialogs in Cursor Settings
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the row lives in Cursor Settings, which the brief treats as a boundary surface, and two of its entries are agent warnings
- What: In the Preferences section of the General page in Cursor Settings (below the Editor Settings and Keyboard Shortcuts rows), a row titled Reset "Don't Ask Again" Dialogs (description "See warnings and tips that you've hidden") has a Show or Hide button. Shown, it lists each hidden dialog with a Restore button and a Reset All button below. The names come from the saved keys turned into words, such as Quit Confirmation, plus two agent warnings (About Allowlist, Run Everything Mode Warning). With none saved it says "No Hidden Dialogs Yet" and "You haven't marked any dialogs as "Don't ask again". Any hidden dialogs will appear here to manage." The settings search finds it by dialogs, warnings, reset or don't ask again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component fww inside the Preferences section component mww, hosted by the General page and not drawn in the Agents window; label pww, empty-state text hww, key-to-label function www, list Edg, accessibleLabel Hidden dialogs)

### DL-1.25 Keyboard and focus in the pretty dialog
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Enter accepts the primary button, unless the key was pressed on a button, link or field (they act on their own), the box turns off Enter, or the primary button is busy or locked. Shift+Enter presses the extra button marked with that shortcut. Escape cancels, and so does a click on the backdrop (not when the press started inside the box). The primary button is focused on open, or the first text field. The Cancel button shows "(esc)" in its label and the primary button shows an Enter symbol. Cancel sits at the left edge on macOS and Linux. I found no dialog role or modal flag on this box, only an h1 title and aria labels on copyable values.
- Commands: none found
- Menus: none found
- Keys: Enter; Shift+Enter; Escape
- Settings: none found
- Cursor: cursor only
- Source: C bundle (view GMk key handler and button row, wrapper NMk backdrop click handling, shouldPlaceCancelButtonOnLeft)

### DL-1.26 Clear Recently Opened confirm
- Tag: both
- Evidence: source
- Boundary: no
- What: File > Open Recent > Clear Recently Opened... asks "Do you want to clear all recently opened files and workspaces?" with the detail "This action is irreversible!", a Clear button and Cancel. Confirming wipes both the global and the per-workspace recent lists. The menu row itself is TB-1.16.
- Commands: workbench.action.clearRecentFiles
- Menus: MenubarRecentMenu (group z_clear, order 1); CommandPalette (f1, category File)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (ClearRecentFilesAction, lines 1672-1708); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (menu row, line 719); C bundle (command id, nls 3580, 2745, 3649)

### DL-1.27 Unusual line terminators prompt
- Tag: both
- Evidence: source
- Boundary: no
- What: When a file holds hidden Line Separator or Paragraph Separator characters, a box titled "Unusual Line Terminators" says "Detected unusual line terminators", names the file, and offers "Remove Unusual Line Terminators" or "Ignore". Ignore silences it for that file. The setting picks auto-remove, ignore or prompt, and prompt is the default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.unusualLineTerminators
- Cursor: present
- Source: V\src\vs\editor\contrib\unusualLineTerminators\browser\unusualLineTerminators.ts (lines 92-120); V\src\vs\editor\common\config\editorOptions.ts (lines 6730-6745); C bundle (nls 1691 to 1694, option name unusualLineTerminators)

### DL-1.28 Command error box
- Tag: both
- Evidence: source
- Boundary: no
- What: If a command you picked in the Command Palette throws, an error box says "Command '{name}' resulted in an error" with the error text as its detail and an OK button.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (line 320); C bundle (nls 2233)

### DL-1.29 Refactoring preview boxes
- Tag: both
- Evidence: source
- Boundary: no
- What: Starting a second refactor preview while one is open asks "Another refactoring is being previewed." with the detail "Press 'Continue' to discard the previous refactoring and continue with the current refactoring." and Continue or Cancel. Pressing Apply in the preview when files changed meanwhile shows a warning box "Cannot apply refactoring because '{file}' has changed in the meantime." (or "... {n} other files ..."). The preview panel itself is PN-7.
- Commands: refactorPreview.apply; refactorPreview.discard
- Menus: BulkEditContext
- Keys: Ctrl+Enter or Cmd+Enter (refactorPreview.apply, when the preview pane is focused)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts (lines 111-140 and 165-200); V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts (accept, lines 253-272); C bundle (nls 4909, 4910, 4930, 4931; command ids refactorPreview.apply and refactorPreview.discard)

### DL-1.30 No updates available box
- Tag: both
- Evidence: source
- Boundary: no
- What: After you ask for an update check by hand and nothing new exists, an info box says "There are currently no updates available." Stock only shows it from the window that last had focus, and on macOS with every window closed the main process shows a native box instead. Cursor still shows the older version of this flow, with no focus check and no windowless macOS box.
- Commands: update.checkForUpdate
- Menus: MenubarHelpMenu (Check for Updates row, see TB-1)
- Keys: none found
- Settings: none found
- Cursor: changed - older flow, renderer-only, no last-focus check and no macOS windowless box
- Source: V\src\vs\workbench\contrib\update\browser\update.ts (lines 262-270); V\src\vs\platform\update\electron-main\notAvailableUpdateDialog.ts; C bundle (onUpdateNotAvailable calling the info box, nls 11502); C out\main.js (no copy of the main-process box found)

### DL-1.31 Unsupported remote OS confirm
- Tag: both
- Evidence: source
- Boundary: no
- What: Before connecting to a remote Linux that the app no longer supports, a warning box says "You are about to connect to an OS version that is unsupported by {product}." with Allow, Learn More and Cancel, and a "Do not show again" checkbox that remembers Allow for that remote. Learn More opens the FAQ and then asks again. The banner that follows is BN-2.5.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\remoteConnectionHealth.ts (_confirmConnection, lines 44-80); C bundle (nls 8946 to 8949, storage key constant REMOTE_UNSUPPORTED_CONNECTION_CHOICE)

### DL-1.32 Pre-launch task error prompt
- Tag: both
- Evidence: source
- Boundary: no
- What: When a debug pre-launch task ends with errors and debug.onTaskErrors is "prompt" (the default), a warning box asks what to do with buttons "Debug Anyway", "Show Errors" and "Abort", and a "Remember my choice in user settings" checkbox. A second box covers a task that could not run at all: Debug Anyway, a configure-task button and Cancel, with "Remember my choice for this task" kept per workspace. The debug UI itself is SB-5.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.onTaskErrors
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debugTaskRunner.ts (lines 96-190); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (debug.onTaskErrors); C bundle (nls 5907 to 5915, setting debug.onTaskErrors)

### DL-1.33 Window not responding or crashed box
- Tag: both
- Evidence: source
- Boundary: no
- What: The main process, not the page, shows an OS warning box when a window hangs or dies. A hang says "The window is not responding" with Reopen, Close and Keep Waiting. A crash says "The window terminated unexpectedly" (with the reason and exit code when known) and offers Reopen, or New Window for an empty window, and Close. When a folder or workspace is open a "Don't restore editors" checkbox lets the reopened window skip its saved editors. A hang is ignored while dev tools are open or in an extension development host.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\windows\electron-main\windowImpl.ts (onWindowError, lines 935-1010); C out\main.js (nls 2716 to 2729 in the same handler, isDevToolsOpened check)

### DL-1.34 Path does not exist box
- Tag: both
- Evidence: source
- Boundary: no
- What: When the main process is asked to open a path that is not on the machine (for example from the command line), it shows an OS info box "Path does not exist" with "The path '{path}' does not exist on this computer.", or "URI can not be opened" for a bad URI, and an OK button.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\windows\electron-main\windowsMainService.ts (lines 908-925); C out\main.js (nls 2732 in the same flow)

### DL-1.35 Workspace does not exist box
- Tag: both
- Evidence: source
- Boundary: no
- What: In a remote window whose folder or workspace file is gone, a warning box says "Workspace does not exist" with the detail "Please select another workspace to open." and an "Open Workspace..." button (or a Cancel). Confirming opens the workspace or folder picker (DL-3).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\common\remote.contribution.ts (RemoteInvalidWorkspaceDetector, lines 120-146); C bundle (nls 9071 to 9073)

### DL-1.36 Markdown in dialog detail
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. The detail line of a box can be Markdown. The in-page box renders it with working links, and the OS box shows it as plain text. Stock 1.128 and Cursor take plain text only.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: N\src\vs\platform\dialogs\common\dialogs.ts (detail may be an IMarkdownString); N\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts; N\src\vs\base\browser\ui\dialog\dialog.ts (detailElement); C bundle (zero hits for detailElement)

### DL-1.37 Stacked button layout
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. A caller can ask for a vertical layout, with full-width buttons stacked under the message, through the custom dialog options. The 1.128 widget already has the layout, but only the Sessions code under V\src\vs\sessions uses it, and the service option to reach it is newer. Cursor has neither.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: N\src\vs\platform\dialogs\common\dialogs.ts (alignment option, line 305); N\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (line 133); V\src\vs\base\browser\ui\dialog\dialog.ts (DialogContentsAlignment); C bundle (zero hits for align-vertical)

### DL-1.38 Restart prompt after changing a restart-only setting
- Tag: both
- Evidence: source
- Boundary: no
- What: When the person changes a setting that only works after a restart, a confirm box offers to restart the app right away.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.enableExperiments; security.restrictUNCAccess; window.titleBarStyle; window.menuStyle; window.nativeTabs; window.nativeFullScreen; window.clickThroughInactive; window.controlsStyle; editor.accessibilitySupport; security.workspace.trust.enabled; accessibility.verbosity.debug; telemetry.feedback.enabled; update.mode; workbench.activityBar.orientation
- Cursor: changed - the same watcher (class with a SETTINGS list) and the same box exist, with nls 8923 to 8928 holding the texts; the list adds update.mode, workbench.activityBar.orientation and application.experimental.extensionMonitor.enabled and has no window.menuStyle or telemetry.feedback.enabled entry
- Source: V\src\vs\workbench\contrib\relauncher\browser\relauncher.contribution.ts (lines 56-250); V\src\vs\workbench\services\assignment\common\assignmentService.ts (lines 347-350); C bundle (class with fri.SETTINGS, the nls pair 8923 and 8924, setting workbench.enableExperiments registered with default true and the usesOnlineServices tag); NLS 8923 to 8928, 12297

## Handed off
- The "not in the list of allowed hosts" Windows box for network share paths (`windowsMainService.ts` onUNCHostNotAllowed, buttons Allow, Cancel, Learn More, checkbox "Permanently allow host", setting `security.allowedUNCHosts`) - DL-5
- Git confirm and warning boxes raised with `modal: true` from the built-in git extension (discard, force push and similar) - SB-4
- Explorer delete, move and replace confirmations that reuse the confirm box and its checkbox - DL-4
- Debug stop and exit confirms (`debug.confirmOnExit`) and terminal exit confirms - SB-5 and PN-2
- The Quick Pick and Input widget (the floating list and text box), which is a different control from the input box in DL-1.7 - QA-6

## Unclear
- The count is 37, above the usual 20. Ten of them (DL-1.26 to DL-1.35) are named boxes: eight were handed to DL-1 by other groups (TB-1, ED-5, QA-1, PN-7, NT-6, BN-2, SB-5 and DL-3) and two are main-process boxes I found. I kept each because no other group owns them.
- I did not open the Cursor Settings page that holds the reset row (DL-1.24). The page name General comes from the account rows next to it in the same component and from the DL-5 and DL-6 notes.
- Cursor's pretty dialog service has confirm, prompt, input, info, warn, error and about methods, but I found call sites only for openDialog (26 through the service name, 59 in total). Its about() builds the web-style text (Version, Commit, Date, Browser) with OK and Copy, and I found no desktop caller for it.
- Cursor's customError call on the stock dialog service has no caller in the editor bundle. Only its definition was found.
- Cursor passes a showOverHiddenWindowInWindow flag to OS boxes so a box raised from a hidden Browser tab shows over the parent window. Without it the main process throws "Dialogs are skipped in hidden window-in-windows". I did not trace which Browser actions rely on it, so it is not listed as a feature.
- The command cursor.browserView.configureDialogHandling accepts or dismisses a web page's own alert boxes for the agent. It is agent plumbing and left out.
- The macOS "will now prompt with osascript for Administrator privileges" box for installing or removing the shell command exists in stock and in Cursor's main process. No group in the catalog owns the shell command rows, so I did not list it.
- The "Snapshot saved" box that NT-6 mentioned is not in the 1.128.0 source I searched (workbench and platform). The "Report Bug" dialog from SS-6.14 opens the Issue Reporter, which the catalog lists as an item nobody owns, so I did not list it. The "Find {language} extension" box (SB-5 hand-off) ends in the extension marketplace and is out of scope.
- Cursor's main process has a renderer watchdog (class with _onBlocked and RendererPing) that may show something when a window blocks. I did not trace whether it shows a box beyond DL-1.33.
- Dialog colours come from theme tokens through the default dialog styles. I did not catalog them.
- Nothing was run on screen. Cursor findings are strings and minified code in the bundle, so sizes, positions and exact looks are not confirmed.
