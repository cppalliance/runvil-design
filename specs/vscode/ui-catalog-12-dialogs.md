# 12. Dialogs (DL)

6 groups, 125 features. Each feature lists its entry points and the pieces that finish it.

## DL-1 Message, confirm and input boxes, and About

(both). The generic pop-ups that tell you something, ask yes or no, offer several buttons or ask for text, drawn by the OS or the app, plus About. Cursor has a second dialog engine of its own with a persistent "don't ask again" memory and a custom About window.

38 features.

#### DL-1.1 Confirm box with a primary button and Cancel

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A yes-or-cancel box with a message, an optional detail line, a primary button (default label "Yes", or a caller label such as "Clear") and a Cancel button. The answer is confirmed or not confirmed. Other features raise it, so it has no entry point of its own.
- Commands: none found
- Menus: none found
- Keys: Enter (primary button); Escape (cancel) - both only while the in-page box is open, see DL-1.12
- Settings: window.dialogStyle
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (IConfirmation, getConfirmationButtons); V\src\vs\workbench\services\dialogs\common\dialogService.ts (confirm); V\src\vs\workbench\electron-browser\parts\dialogs\dialog.contribution.ts (processDialogs); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts; V\src\vs\workbench\electron-browser\parts\dialogs\dialogHandler.ts; C bundle (contribution workbench.contrib.dialogHandler, confirm branch of processDialogs)
- Pieces:
  1. A confirm box shows a message, an optional smaller detail line, a primary button and a Cancel button, and the feature that raised it learns whether you confirmed.
  2. The primary button says "Yes" unless the caller gives its own label, such as "Clear".
  3. Pressing Enter presses the primary button and pressing Escape cancels, while the in-page box is open.
  4. Choosing Cancel counts as not confirmed, so the action that raised the box does not go ahead.
  5. Setting `window.dialogStyle` chooses whether the confirm box is an OS message box or the in-page box, with values `native` and `custom`.

#### DL-1.2 Checkbox on a box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Any confirm, prompt or input box can carry one checkbox under the message, and the answer reports whether it was ticked. Clicking the label text toggles it too. Examples seen: "Do not ask me again" on the quit and close confirm, "Always discard untitled workspaces without asking", "Remember my credentials" on the proxy login, "Remember my choice in user settings" on the pre-launch task prompt, and "Don't restore editors" on the window crash box.
- Commands: none found
- Menus: none found
- Keys: Space (toggles the focused checkbox); Tab (moves to it)
- Settings: window.confirmBeforeClose; window.confirmSaveUntitledWorkspace
- Source: V\src\vs\base\browser\ui\dialog\dialog.ts (checkbox row, lines 217-229); V\src\vs\platform\dialogs\common\dialogs.ts (ICheckbox, ICheckboxResult); V\src\vs\workbench\browser\window.ts (confirmOnShutdown); V\src\vs\workbench\services\workspaces\electron-browser\workspaceEditingService.ts; C bundle (class with .dialog-checkbox-row; nls 4073, 13618, 12119, 5914, 2721)
- Pieces:
  1. A confirm, prompt or input box can show one checkbox under the message, and the answer tells the caller whether it was ticked.
  2. Clicking the label text next to the checkbox toggles it, the same as clicking the box itself.
  3. Pressing Tab moves focus to the checkbox and pressing Space toggles it.
  4. The quit and close confirm shows a "Do not ask me again" checkbox, and ticking it while confirming sets `window.confirmBeforeClose` to `never`.
  5. The untitled workspace save prompt shows an "Always discard untitled workspaces without asking" checkbox, and ticking it turns off `window.confirmSaveUntitledWorkspace`.
  6. Other boxes carry their own checkboxes, such as "Remember my credentials" on the proxy login, "Remember my choice in user settings" on the pre-launch task prompt, and "Don't restore editors" on the window crash box.

#### DL-1.3 Multi-button prompt

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A box with two or more custom buttons and an optional Cancel. The first button is the default. Each button returns its own value to the caller, so one box can mean Save, Don't Save or Cancel. With no buttons given it falls back to a single OK.
- Commands: none found
- Menus: none found
- Keys: Enter (default button); Escape (cancel button); Left and Right arrows or Tab (move between buttons), see DL-1.12
- Settings: window.dialogStyle
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (IPrompt, getPromptButtons, getPromptResult); V\src\vs\workbench\services\dialogs\common\dialogService.ts (prompt); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (prompt); V\src\vs\workbench\electron-browser\parts\dialogs\dialogHandler.ts (prompt); C bundle (processDialogs promptArgs branch)
- Pieces:
  1. A prompt box shows a message, an optional detail and two or more custom buttons, and each button hands its own value back, so one box can offer Save, Don't Save or Cancel.
  2. The first button is the default, and pressing Enter presses it.
  3. An optional Cancel button comes after the custom buttons, and pressing Escape presses it.
  4. When the caller gives no buttons the box shows a single OK button.
  5. Pressing Left or Right arrow, or Tab, moves focus between the buttons.
  6. Setting `window.dialogStyle` chooses whether the prompt is an OS message box or the in-page box, with values `native` and `custom`.

#### DL-1.4 Info, warning and error boxes

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - adds a customError call that forces the in-page box
- What: One-button boxes that just tell you something: an info, warning or error icon, a message, an optional detail and an OK button. They are sent through the same prompt path. Cursor adds a fourth call that forces the in-page box for an error, but I found no caller of it in the editor bundle.
- Commands: none found
- Menus: none found
- Keys: Enter or Escape closes it
- Settings: window.dialogStyle
- Source: V\src\vs\workbench\services\dialogs\common\dialogService.ts (info, warn, error); V\src\vs\platform\dialogs\common\dialogs.ts (getDialogType); C bundle (class with info, warn, error and customError methods next to async about)
- Pieces:
  1. An info box shows an info icon, a bold message, an optional detail and a single OK button.
  2. A warning box has the same layout with a warning icon.
  3. An error box has the same layout with an error icon.
  4. Pressing Enter or Escape closes the box.
  5. Setting `window.dialogStyle` chooses whether these boxes are OS message boxes or the in-page box, with values `native` and `custom`.
  6. In Cursor an extra `customError` call shows an error box in the in-page style whatever `window.dialogStyle` says, though no caller of it was found in the editor. [cursor only]

#### DL-1.5 Detail text under the message

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Any box can show a smaller detail paragraph under the bold message, for example "Your changes will be lost if you don't save them." In the OS box it is the native detail text, in the in-page box it sits under the icon and message. Plain text only in 1.128.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (IBaseDialogOptions.detail); V\src\vs\base\browser\ui\dialog\dialog.ts (messageDetailElement, lines 170-182); V\src\vs\workbench\electron-browser\parts\dialogs\dialogHandler.ts; C bundle (class with #monaco-dialog-message-detail)
- Pieces:
  1. Any box can show a smaller detail paragraph under the bold message, for example "Your changes will be lost if you don't save them."
  2. In the OS box the detail shows as the native detail text under the message.
  3. In the in-page box the detail sits under the icon and message.
  4. The detail is plain text only, so Markdown marks show exactly as typed.

#### DL-1.6 Rich in-page body, icon and button captions

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - button captions use an older option name (buttonDetails) in the widget
- What: A caller can ask for the in-page box on purpose and then add its own body: extra CSS classes, rendered Markdown paragraphs with clickable links, a custom icon, small caption text under each button, and a switch that removes the close X and Escape. Asking for it overrides the native style. The Workspace Trust prompts use it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.dialogStyle
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (ICustomDialogOptions, lines 287-300); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (doShow, renderBody); V\src\vs\base\browser\ui\dialog\dialog.ts (buttonOptions, disableCloseAction); C bundle (in-page handler passing buttonDetails, markdownDetails, icon, disableCloseAction to the dialog widget)
- Pieces:
  1. A caller can ask for the in-page box on purpose, which overrides `window.dialogStyle`, and the Workspace Trust prompts do this.
  2. The in-page box can show extra body paragraphs rendered from Markdown, with links you can click.
  3. The in-page box can show a custom icon in place of the standard severity icon.
  4. The in-page box can show small caption text under each button.
  5. A caller can remove the close X and the Escape shortcut, so the box can only be dismissed with its buttons.
  6. In Cursor the captions under the buttons are passed to the widget under the older option name `buttonDetails`, and they still show under the buttons. [cursor only]

#### DL-1.7 Input box with text fields

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A box with one or more text fields (placeholder, optional starting value, plain or password type), an optional checkbox, a primary button (default "OK") and Cancel. The first field is focused and selected on open, and Enter inside a field accepts. It is always the in-page box, because Electron has no native version and the OS handler throws "Unsupported".
- Commands: none found
- Menus: none found
- Keys: Enter (accept while a field has focus); Tab (next field)
- Settings: none found
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (IInput, IInputElement); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (input); V\src\vs\workbench\electron-browser\parts\dialogs\dialogHandler.ts (input throws); V\src\vs\workbench\electron-browser\parts\dialogs\dialog.contribution.ts (input is custom only); V\src\vs\base\browser\ui\dialog\dialog.ts (inputs, lines 196-213 and 560-566); C bundle (inputArgs branch always uses the in-page handler; class with .dialog-message-input)
- Pieces:
  1. An input box shows a message, one or more text fields, an optional checkbox, a primary button and a Cancel button.
  2. Each field can have placeholder text and an optional starting value.
  3. A field can be a plain text field or a password field that hides what you type.
  4. The first field is focused with its text selected when the box opens.
  5. Pressing Enter inside a field accepts the box, and pressing Tab moves to the next field.
  6. The primary button says "OK" unless the caller gives its own label.
  7. The input box is always the in-page box, whatever `window.dialogStyle` says, because the OS has no version of it.

#### DL-1.8 Proxy authentication login box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a network request hits a proxy that wants a login, a warning box titled "Proxy Authentication Required" opens with the text "The proxy host:port requires a username and password.", a Username field, a Password field, a "Remember my credentials" checkbox, a "Log In" button and Cancel. Cancelling replies to the main process with no credentials. The checkbox is remembered under the application storage key window.rememberProxyCredentials, which is not a setting.
- Commands: none found
- Menus: none found
- Keys: Enter (log in); Escape (cancel)
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\window.ts (vscode:openProxyAuthenticationDialog, lines 293-333); V\src\vs\platform\native\electron-main\auth.ts (sends the request); C bundle (listener for vscode:openProxyAuthenticationDialog, nls 12114 to 12119); C out\main.js (sendWhenReady for the same channel)
- Pieces:
  1. When a network request meets a proxy that wants a login, a warning box titled "Proxy Authentication Required" opens and says "The proxy host:port requires a username and password."
  2. The box has a Username field and a Password field that hides what you type, and any known values are filled in.
  3. A "Remember my credentials" checkbox remembers the choice under the application storage key `window.rememberProxyCredentials`, which is not a setting.
  4. Pressing the "Log In" button, or Enter, sends the username and password back to the request.
  5. Pressing Cancel, or Escape, replies with no credentials.

#### DL-1.9 Dialog style: native or custom

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - different description text; About always uses the custom window; no smoke-test-driver clause
- What: One setting decides whether confirm, prompt and info boxes are drawn by the OS or by the app. Default is native. Custom draws the in-page box instead. Input boxes are always in-page. Some callers force the in-page box themselves. In Cursor the About box ignores this setting (it always opens its own window), and Cursor's second dialog engine is not controlled by it either.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.dialogStyle
- Source: V\src\vs\workbench\electron-browser\desktop.contribution.ts (window.dialogStyle, lines 291-297); V\src\vs\workbench\electron-browser\parts\dialogs\dialog.contribution.ts (useCustomDialog, lines 59-110); C bundle (setting window.dialogStyle with nls 12063, contribution Toi with useCustomDialog and an About branch that always calls the native handler)
- Pieces:
  1. Setting `window.dialogStyle` chooses whether confirm, prompt and info boxes are drawn by the OS or by the app, with values `native` (the default) and `custom`.
  2. With `custom`, those boxes appear as the in-page box inside the window instead of an OS message box.
  3. Input boxes are always in-page, and some callers force the in-page box whatever the setting says.
  4. In Cursor the About box ignores `window.dialogStyle` and always opens its own window, and Cursor's second dialog engine (DL-1.22) is not controlled by the setting either. [cursor only]
  5. Cursor shows its own description text for `window.dialogStyle` in Settings, without the smoke-test-driver sentence that stock has. [cursor only]

#### DL-1.10 Button order and default button per OS

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The same box lists its buttons in a different order on each OS. Windows keeps Cancel last. macOS and Linux put the default button on the trailing side and Cancel just left of it, and Linux also flips the whole row. The OS box sets title to the app name when none is given and turns off link-style buttons. The in-page box applies the same arrangement itself.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\dialogs\common\dialogs.ts (massageMessageBoxOptions, lines 633-740); V\src\vs\platform\dialogs\electron-main\dialogMainService.ts (showMessageBox, line 149); V\src\vs\base\browser\ui\dialog\dialog.ts (rearrangeButtons, lines 632-675); C out\main.js (massage code that sets noLink, defaultId and cancelId)
- Pieces:
  1. On Windows the buttons keep Cancel last.
  2. On macOS the default button sits on the trailing side with Cancel just to its left.
  3. On Linux the buttons use the macOS order and the whole row is flipped.
  4. The OS box uses the app name as its title when the caller gives none, and turns off link-style buttons.
  5. The in-page box arranges its own buttons in the same per-OS order.

#### DL-1.11 Modal block: one box at a time, window dimmed

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - queue is present, but the window-controls dimming is missing (no setWindowDimmed and no dimmed option in the main process)
- What: While a box is open the window cannot be used. Boxes wait in a queue and show one after another, per window. The in-page box covers the window with a dimmed layer, and clicking the layer sends focus back into the box. On Windows and Linux with drawn window controls, the minimize, maximize and close buttons are dimmed to match while the box is up.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.titleBarStyle; window.controlsStyle
- Source: V\src\vs\workbench\services\dialogs\common\dialogService.ts (DialogsModel queue); V\src\vs\workbench\electron-browser\parts\dialogs\dialog.contribution.ts (processDialogs loop); V\src\vs\platform\dialogs\electron-main\dialogMainService.ts (windowDialogQueues, line 51); V\src\vs\workbench\browser\parts\dialogs\dialog.ts (onVisibilityChange); V\src\vs\platform\windows\electron-main\windowImpl.ts (updateWindowControls, dimColor, lines 412-470); V\src\vs\platform\window\common\window.ts (useWindowControlsOverlay); C bundle (class with .monaco-dialog-modal-block.dimmed; zero hits for setWindowDimmed); C out\main.js (queue names present, zero hits for dimmed)
- Pieces:
  1. While a box is open the rest of the window cannot be used.
  2. A box raised while another is open waits in a queue, and boxes show one after another, per window.
  3. The in-page box covers the window with a dimmed layer.
  4. Clicking the dimmed layer sends focus back into the box.
  5. On Windows and Linux with drawn window controls (settings `window.titleBarStyle` and `window.controlsStyle`), the minimize, maximize and close buttons are dimmed while a box is up. [stock only]

#### DL-1.12 Keyboard in the in-page box

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - key handling skips when the box is not on screen, and Escape closes on key up without checking for a prior key down
- What: Enter accepts while a text field has focus. Escape closes the box as Cancel. Tab, Shift+Tab, Left and Right arrows walk through links, text fields, the checkbox and the buttons in visual order and wrap around. On macOS, Cmd+D presses the second button (No or Don't Save). Space is left to the focused control. Almost every other shortcut is blocked while the box is up. Only copy, cut, select all and paste pass through (the pending box also lets quit and reload through). In later stock builds the arrow keys stop moving focus while you type in a field (newer-than-1.128, not in Cursor).
- Commands: none found
- Menus: none found
- Keys: Enter; Escape; Tab; Shift+Tab; Left Arrow; Right Arrow; Cmd+D (macOS only); Space
- Settings: none found
- Source: V\src\vs\base\browser\ui\dialog\dialog.ts (keydown and keyup handlers, lines 336-488); V\src\vs\workbench\browser\parts\dialogs\dialog.ts (defaultDialogAllowableCommands, createWorkbenchDialogOptions); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (ALLOWABLE_COMMANDS); N\src\vs\base\browser\ui\dialog\dialog.ts (isEditableTarget); C bundle (widget class with the Cmd+D branch and isOnScreen guard; list of allowed commands)
- Pieces:
  1. Pressing Enter accepts the box while a text field has focus.
  2. Pressing Escape closes the box as Cancel.
  3. Pressing Tab, Shift+Tab, Left arrow or Right arrow walks focus through the links, text fields, checkbox and buttons in visual order, and wraps around at the ends.
  4. On macOS, pressing Cmd+D presses the second button, such as No or Don't Save.
  5. Space is left to the focused control, such as toggling the checkbox or pressing the focused button.
  6. Almost every other shortcut is blocked while the box is up, and only copy, cut, select all and paste pass through (the pending box also lets quit and reload through).
  7. In later stock builds (newer-than-1.128) the arrow keys stop moving focus while you type in a text field. [stock only]
  8. In Cursor the box ignores keys when it is not on screen, and Escape closes it on key release without needing a prior key press. [cursor only]

#### DL-1.13 Focus on open, keep, and return

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The in-page box focuses its first text field (selected) or its default button when it opens. Focus cannot leave the box: if it tries to, it is pulled back. When the box closes, focus goes back to the element that had it before. A dirty-editor close prompt first brings the window to the front so the box is not hidden.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\base\browser\ui\dialog\dialog.ts (focus at lines 560-571, focusout at 491-505, focus return at 626-628); V\src\vs\workbench\browser\parts\editor\editorGroupView.ts (hostService.focus before the save prompt); C bundle (widget class with focusToReturn and a focusout handler)
- Pieces:
  1. When the in-page box opens, focus goes to its first text field with the text selected, or to the default button when there are no fields.
  2. Focus cannot leave the box, because anything that tries to take it is pulled back.
  3. When the box closes, focus returns to the element that had it before.
  4. A dirty-editor close prompt first brings the window to the front, so the box is not hidden behind other windows.

#### DL-1.14 Close button on the in-page box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A small X in the top right of the in-page box, named "Close Dialog", closes it and counts as Cancel. It is not drawn when the caller turns the close action off, and then Escape does nothing either.
- Commands: none found
- Menus: none found
- Keys: Escape (same result, see DL-1.12)
- Settings: none found
- Source: V\src\vs\base\browser\ui\dialog\dialog.ts (dialog.close action, lines 537-548); C bundle (action id dialog.close, nls 7 "Close Dialog")
- Pieces:
  1. A small X in the top right of the in-page box, named "Close Dialog", closes the box when clicked.
  2. Closing with the X counts as Cancel.
  3. When the caller turns the close action off, the X is not drawn and pressing Escape does nothing either.

#### DL-1.15 Accessibility of the in-page box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The box is marked as a modal dialog for screen readers. It is labelled by its icon and message, and described by the message, detail, body and footer. The icon has a spoken name (Info, Warning, Error or In Progress). Links in the body and footer are made keyboard focusable and underlined so colour is not the only cue. Cursor's second dialog engine does not set a dialog role (see DL-1.25).
- Commands: none found
- Menus: none found
- Keys: Tab (reaches links)
- Settings: none found
- Source: V\src\vs\base\browser\ui\dialog\dialog.ts (role, aria-modal, aria-labelledby, aria-describedby at lines 136 and 552-554; getIconAriaLabel at 238-257; link styling in applyStyles); C bundle (widget class with the same aria-labelledby id list)
- Pieces:
  1. Screen readers treat the in-page box as a modal dialog.
  2. The box is labelled by its icon and message, and described by its message, detail, body and footer text.
  3. The icon has a spoken name of Info, Warning, Error or In Progress.
  4. Links in the body and footer can be reached with Tab and are underlined, so colour is not the only cue.

#### DL-1.16 Pending box with spinner and Cancel

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the pending box is built without the host service, so no window-controls dimming (also noted in NT-5)
- What: A box with a spinning icon and a changing message while work runs. It has a Cancel or Dismiss button, unless it is set to sticky, and the message updates as the task reports. It is the box look used for work shown at the dialog progress location, such as the waiting boxes at shutdown (see DL-2.8). The progress side of it is NT-5.10.
- Commands: none found
- Menus: none found
- Keys: Escape (cancel unless sticky); Enter (default button)
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (withDialogProgress, lines 549-615); V\src\vs\base\browser\ui\dialog\dialog.ts (pending type, spin class, updateMessage); C bundle (withDialogProgress calling the options factory with two arguments)
- Pieces:
  1. A pending box shows a spinning icon and a message while work runs.
  2. The message changes as the running task reports progress.
  3. The box has a Cancel or Dismiss button, unless the caller sets it to sticky, and then there is no button.
  4. Pressing Escape cancels the box unless it is sticky, and pressing Enter presses the default button.
  5. It is the look used for work shown at the dialog progress location, such as the waiting boxes at shutdown.
  6. In stock the pending box also dims the window minimize, maximize and close buttons while it is up, and in Cursor it does not. [stock only]

#### DL-1.17 Open the About box

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - custom About window with an OS box as fallback
- What: Help > About (Windows and Linux), the "About {name}" row at the top of the macOS app menu, or "Help: About" in the Command Palette opens the About box. On Windows and Linux the update row sits just above it in the Help menu. In stock it is an OS message box, or the in-page box when dialog style is custom. In Cursor it opens its own small window instead (DL-1.20), ignores dialog style, and only falls back to an OS box if that window fails to open.
- Commands: workbench.action.showAboutDialog
- Menus: MenubarHelpMenu (group z_about, order 1, hidden on macOS native menus); CommandPalette (f1, category Help); macOS application menu (built in the main process)
- Keys: none found
- Settings: window.dialogStyle
- Source: V\src\vs\workbench\browser\actions\windowActions.ts (ShowAboutDialogAction, lines 387-411); V\src\vs\workbench\services\dialogs\common\dialogService.ts (about); V\src\vs\workbench\electron-browser\parts\dialogs\dialog.contribution.ts (About branch); V\src\vs\platform\menubar\electron-main\menubar.ts (mAbout at 409, insertCheckForUpdatesItems at 521); V\src\vs\workbench\browser\parts\titlebar\menubarControl.ts (insertActionsBefore, line 447); C bundle (command workbench.action.showAboutDialog, nls 3265 and 3257, native handler about with Failed to open supplemental About window); C out\main.js (command id found, 2 hits)
- Pieces:
  1. Choosing Help > About on Windows and Linux opens the About box, and the update row sits just above it in the Help menu.
  2. Choosing the "About {name}" row at the top of the macOS app menu opens the About box, and the Help menu on macOS has no About row.
  3. Running "Help: About" (`workbench.action.showAboutDialog`) from the Command Palette opens the About box.
  4. In stock the About box is an OS message box, or the in-page box when `window.dialogStyle` is `custom`.
  5. In Cursor About opens its own small window (DL-1.20), ignores `window.dialogStyle`, and falls back to an OS box only if that window fails to open. [cursor only]

#### DL-1.18 Copy version info from About

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - primary "Copy version info" button in a custom window, Copy and OK only in the fallback box
- What: The About box has a Copy button that puts every version line on the clipboard and an OK button that closes it. The copied text is plain, without the "x days ago" note shown on screen. In Cursor the main button is "Copy version info", it copies and closes the window, and Enter triggers it. The OS fallback box keeps the stock Copy and OK buttons.
- Commands: workbench.action.showAboutDialog
- Menus: none found
- Keys: Enter (Cursor About window: copy and close); Escape (Cursor About window: close); C mnemonic on the Copy button (OS box on Windows and Linux)
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\parts\dialogs\dialogHandler.ts (about, lines 67-84); V\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (about, lines 76-92); C bundle (nls 3338 "Copy version info", 12098 "&&Copy", 12099 "OK"; Enter and Escape handler of the About window component)
- Pieces:
  1. The About box has a Copy button that puts every version line on the clipboard, and an OK button that closes it.
  2. The copied text is plain, without the "x days ago" note shown on screen.
  3. In the OS box on Windows and Linux the Copy button has a C mnemonic.
  4. In the Cursor About window the main button reads "Copy version info", copies the version lines and closes the window, and pressing Enter does the same. [cursor only]
  5. Pressing Escape closes the Cursor About window without copying. [cursor only]
  6. The OS fallback box in Cursor keeps the stock Copy and OK buttons.

#### DL-1.19 Version lines in About

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - different set of lines, see What
- What: Stock lists Version (with a "user setup", "system setup" or "Universal" note when known), Commit, Date (plus "n days ago" on screen only), Electron, ElectronBuildId, Chromium, Node.js, V8 and OS. Cursor copies a different list: Version, VS Code Extension API, Commit, Date, Layout, Build Type (Stable, Nightly, Dev/Debug), Release Track (Default, Early Access, Nightly, Dogfood, Candidate), Electron, Chromium, Node.js, V8, xterm.js and OS, plus "Local Mode: enabled" when local mode is on. Cursor has no ElectronBuildId. A later stock build adds two Copilot package lines (newer-than-1.128, not in Cursor).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\dialogs\electron-browser\dialog.ts (createNativeAboutDialogDetails, lines 13-45); V\src\vs\workbench\browser\parts\dialogs\dialog.ts (web text, lines 48-67); N\src\vs\platform\dialogs\electron-browser\dialog.ts (Copilot lines); C bundle (native handler about, nls 12100 and 12101, getBuildType, owy release track labels, withLocalModeDetail, nls 12097)
- Pieces:
  1. The About text lists Version, Commit, Date, Electron, Chromium, Node.js, V8 and OS, one per line.
  2. In stock the Version line carries a "user setup", "system setup" or "Universal" note when that is known.
  3. The Date line shows "n days ago" on screen only, and the copied text leaves it out.
  4. Stock also lists an ElectronBuildId line, and Cursor has none. [stock only]
  5. Cursor's copied text adds the lines VS Code Extension API, Layout and xterm.js. [cursor only]
  6. Cursor adds a Build Type line (Stable, Nightly or Dev/Debug) and a Release Track line (Default, Early Access, Nightly, Dogfood or Candidate). [cursor only]
  7. Cursor adds a "Local Mode: enabled" line when local mode is on. [cursor only]
  8. A later stock build (newer-than-1.128) adds two Copilot package lines to the About text. [stock only]

#### DL-1.20 Cursor About window

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor's About is a small window about 380 px wide, with no title, no full screen, a see-through background and a drag strip along the top. It shows the app icon, the product name, "Version x", the build date and a copyright line. Hovering the version or the date shows a copy icon that copies that one value and flips to a check mark for about 1.5 seconds. The window resizes to fit, closes on Escape, closes when the app shuts down, and a second About request reuses the open window.
- Commands: workbench.action.showAboutDialog
- Menus: MenubarHelpMenu; CommandPalette
- Keys: Escape (close); Enter (copy version info and close)
- Settings: none found
- Source: C bundle (About window components v5w, f5w and the copy helper Tvg, opener _5w, data-component glass-about-dialog-window, about-dialog-drag-region, glass-about-dialog-body; nls 3334 to 3337 and 12102); C out\main.js (revealAboutWindow, fitAboutWindow, updateAboutWindowControls)
- Pieces:
  1. The Cursor About window is about 380 px wide, with no title, no full screen, a see-through background and a drag strip along the top. [cursor only]
  2. It shows the app icon, the product name, "Version x", the build date and a copyright line. [cursor only]
  3. Hovering the version or the date shows a copy icon, and clicking it copies that one value. [cursor only]
  4. After a copy, the icon flips to a check mark for about 1.5 seconds. [cursor only]
  5. The window resizes to fit its content. [cursor only]
  6. Pressing Escape closes the window, and the window also closes when the app shuts down. [cursor only]
  7. A second About request reuses the open window instead of opening another. [cursor only]

#### DL-1.21 Update button in the About window

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Next to "Copy version info" the About window has a button whose label and action follow the update state: check for updates, download, install, or restart to update. It shows a spinning ring while checking, downloading or installing, becomes "No updates available" (disabled) after an explicit check that finds nothing, and "Updates unavailable" when updates are off. Restarting is called with the source about_dialog. The title-bar Update control is TB-5.
- Commands: update.checkForUpdate; update.downloadUpdate; update.installUpdate; update.restartToUpdate
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component g5w, state table e5w, labels t5w and n5w, label function lgw, data-component glass-about-dialog-update-button)
- Pieces:
  1. Next to "Copy version info" the About window has an update button whose label follows the update state. [cursor only]
  2. Clicking it runs the action for the current state: `update.checkForUpdate`, `update.downloadUpdate`, `update.installUpdate` or `update.restartToUpdate`. [cursor only]
  3. The button shows a spinning ring while the app is checking, downloading or installing. [cursor only]
  4. After an explicit check that finds nothing, the button reads "No updates available" and is disabled. [cursor only]
  5. When updates are turned off, the button reads "Updates unavailable". [cursor only]

#### DL-1.22 Cursor's second dialog engine (pretty dialog)

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor draws its own in-page modal for its own boxes. It has a bold title, a message that can be plain text, a list of blocks (plain, callout with icon and colour, copyable value, Markdown), an optional icon, a primary button (can be styled as danger), extra secondary buttons, a Cancel button, optional text fields, an optional checkbox, and an optional "confirm" checkbox that must be ticked before the primary button works. It fades in below the top edge with a half-dark backdrop. It is separate from the stock box: the dialog style setting does not apply to it, and it is not created in the Agents window. Only the non-agent callers belong in this catalog, see DL-2.17 and DL-2.18 for quit and close.
- Commands: none found
- Menus: none found
- Keys: Enter; Shift+Enter; Escape (see DL-1.25)
- Settings: none found
- Source: C bundle (service class X7 registered as D0 with openDialog, updateDialog, closeDialog, confirm, prompt, input, info, warn, error, about; view GMk with wrapper NMk; css classes pretty-dialog, pretty-dialog-modal, pretty-dialog-title, pretty-dialog-message-callout, pretty-dialog-copyable-value, pretty-dialog-input, pretty-dialog-button-primary; z-index 2551 backdrop)
- Pieces:
  1. Cursor draws its own in-page modal with a bold title and a message that is plain text or a list of blocks. [cursor only]
  2. A message block can be plain text, a callout with an icon and colour, a copyable value, or Markdown. [cursor only]
  3. The modal can show an optional icon. [cursor only]
  4. It has a primary button that can be styled as danger, optional extra secondary buttons and a Cancel button. [cursor only]
  5. It can show optional text fields and an optional checkbox. [cursor only]
  6. It can show a "confirm" checkbox that must be ticked before the primary button works. [cursor only]
  7. It fades in below the top edge over a half-dark backdrop. [cursor only]
  8. It is separate from the stock box, so `window.dialogStyle` does not apply to it, and it is not created in the Agents window. [cursor only]

#### DL-1.23 "Don't ask again" memory in the pretty dialog

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: When a box is given a key and asks for it, it shows a "Don't ask again" checkbox (the label and hover text can change). The checkbox sits next to the buttons on the quit and close boxes and under the message elsewhere. If you tick it and then pick a button, that button is saved under the key and the box never opens again: the saved answer is used at once. Cancel is saved only when the box allows it. The saved answers live in Cursor's application storage under dialogDontAskAgainPreferences and are not settings.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (openDialog reading dialogDontAskAgainPreferences, handler y in view GMk, flags shouldShowDontAskAgain, saveDontAskAgainOnCancel, checkboxAtBottom, dontAskAgainLabel, dontAskAgainTooltip, doNotCallOnAcceptOnDontShowAgain)
- Pieces:
  1. A box that is given a key and asks for it shows a "Don't ask again" checkbox, and the label and hover text can change. [cursor only]
  2. On the quit and close boxes the checkbox sits next to the buttons, and elsewhere it sits under the message. [cursor only]
  3. Ticking the checkbox and then picking a button saves that button under the key. [cursor only]
  4. Once an answer is saved the box never opens again, and the saved answer is used at once. [cursor only]
  5. Cancel is saved only when the box allows it. [cursor only]
  6. The saved answers live in Cursor's application storage under `dialogDontAskAgainPreferences` and are not settings. [cursor only]

#### DL-1.24 Reset "Don't Ask Again" dialogs in Cursor Settings

- Tag: cursor. Evidence: bundle. Boundary: yes - the row lives in Cursor Settings, which the brief treats as a boundary surface, and two of its entries are agent warnings. Cursor: cursor only
- What: In the Preferences section of the General page in Cursor Settings (below the Editor Settings and Keyboard Shortcuts rows), a row titled Reset "Don't Ask Again" Dialogs (description "See warnings and tips that you've hidden") has a Show or Hide button. Shown, it lists each hidden dialog with a Restore button and a Reset All button below. The names come from the saved keys turned into words, such as Quit Confirmation, plus two agent warnings (About Allowlist, Run Everything Mode Warning). With none saved it says "No Hidden Dialogs Yet" and "You haven't marked any dialogs as "Don't ask again". Any hidden dialogs will appear here to manage." The settings search finds it by dialogs, warnings, reset or don't ask again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component fww inside the Preferences section component mww, hosted by the General page and not drawn in the Agents window; label pww, empty-state text hww, key-to-label function www, list Edg, accessibleLabel Hidden dialogs)
- Pieces:
  1. In the Preferences section of the General page in Cursor Settings, below the Editor Settings and Keyboard Shortcuts rows, a row titled Reset "Don't Ask Again" Dialogs shows the description "See warnings and tips that you've hidden" and a Show or Hide button. [cursor only]
  2. Pressing Show lists each hidden dialog with a Restore button, and a Reset All button below the list. [cursor only]
  3. Each name is the saved key turned into words, such as Quit Confirmation, plus two agent warnings named About Allowlist and Run Everything Mode Warning. [cursor only]
  4. Pressing Restore on a row makes that dialog open again, and Reset All makes every hidden dialog open again. [cursor only]
  5. With nothing saved the list says "No Hidden Dialogs Yet" and "You haven't marked any dialogs as "Don't ask again". Any hidden dialogs will appear here to manage." [cursor only]
  6. Searching Cursor Settings for dialogs, warnings, reset or don't ask again finds the row. [cursor only]

#### DL-1.25 Keyboard and focus in the pretty dialog

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Enter accepts the primary button, unless the key was pressed on a button, link or field (they act on their own), the box turns off Enter, or the primary button is busy or locked. Shift+Enter presses the extra button marked with that shortcut. Escape cancels, and so does a click on the backdrop (not when the press started inside the box). The primary button is focused on open, or the first text field. The Cancel button shows "(esc)" in its label and the primary button shows an Enter symbol. Cancel sits at the left edge on macOS and Linux. I found no dialog role or modal flag on this box, only an h1 title and aria labels on copyable values.
- Commands: none found
- Menus: none found
- Keys: Enter; Shift+Enter; Escape
- Settings: none found
- Source: C bundle (view GMk key handler and button row, wrapper NMk backdrop click handling, shouldPlaceCancelButtonOnLeft)
- Pieces:
  1. Pressing Enter accepts the primary button, unless the key was pressed on a button, link or field, the box turns Enter off, or the primary button is busy or locked. [cursor only]
  2. Pressing Shift+Enter presses the extra button marked with that shortcut. [cursor only]
  3. Pressing Escape cancels the box, and so does clicking the backdrop, except when the press started inside the box. [cursor only]
  4. When the box opens, focus goes to the primary button, or to the first text field when there are fields. [cursor only]
  5. The Cancel button shows "(esc)" in its label and the primary button shows an Enter symbol. [cursor only]
  6. The Cancel button sits at the left edge on macOS and Linux. [cursor only]
  7. Screen readers get an h1 title and aria labels on copyable values, but no dialog role or modal flag. [cursor only]

#### DL-1.26 Clear Recently Opened confirm

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: File > Open Recent > Clear Recently Opened... asks "Do you want to clear all recently opened files and workspaces?" with the detail "This action is irreversible!", a Clear button and Cancel. Confirming wipes both the global and the per-workspace recent lists. The menu row itself is TB-1.16.
- Commands: workbench.action.clearRecentFiles
- Menus: MenubarRecentMenu (group z_clear, order 1); CommandPalette (f1, category File)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (ClearRecentFilesAction, lines 1672-1708); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (menu row, line 719); C bundle (command id, nls 3580, 2745, 3649)
- Pieces:
  1. Choosing File > Open Recent > Clear Recently Opened..., or running File: Clear Recently Opened (`workbench.action.clearRecentFiles`) from the Command Palette, opens a confirm box.
  2. The box asks "Do you want to clear all recently opened files and workspaces?" with the detail "This action is irreversible!", a Clear button and a Cancel button.
  3. Pressing Clear wipes both the global and the per-workspace recent lists.
  4. Pressing Cancel leaves the recent lists as they were.

#### DL-1.27 Unusual line terminators prompt

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a file holds hidden Line Separator or Paragraph Separator characters, a box titled "Unusual Line Terminators" says "Detected unusual line terminators", names the file, and offers "Remove Unusual Line Terminators" or "Ignore". Ignore silences it for that file. The setting picks auto-remove, ignore or prompt, and prompt is the default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.unusualLineTerminators
- Source: V\src\vs\editor\contrib\unusualLineTerminators\browser\unusualLineTerminators.ts (lines 92-120); V\src\vs\editor\common\config\editorOptions.ts (lines 6730-6745); C bundle (nls 1691 to 1694, option name unusualLineTerminators)
- Pieces:
  1. When a file holds hidden Line Separator or Paragraph Separator characters, a box titled "Unusual Line Terminators" says "Detected unusual line terminators" and names the file.
  2. The box offers "Remove Unusual Line Terminators", which takes the characters out, and "Ignore".
  3. Choosing Ignore silences the box for that file.
  4. Setting `editor.unusualLineTerminators` chooses what happens, with values `auto` (remove them without asking), `off` (ignore them) and `prompt` (show the box, the default).

#### DL-1.28 Command error box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: If a command you picked in the Command Palette throws, an error box says "Command '{name}' resulted in an error" with the error text as its detail and an OK button.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\quickinput\browser\commandsQuickAccess.ts (line 320); C bundle (nls 2233)
- Pieces:
  1. When a command picked in the Command Palette throws an error, an error box opens that says "Command '{name}' resulted in an error".
  2. The box shows the error text as its detail line.
  3. Pressing the OK button closes the box.

#### DL-1.29 Refactoring preview boxes

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Starting a second refactor preview while one is open asks "Another refactoring is being previewed." with the detail "Press 'Continue' to discard the previous refactoring and continue with the current refactoring." and Continue or Cancel. Pressing Apply in the preview when files changed meanwhile shows a warning box "Cannot apply refactoring because '{file}' has changed in the meantime." (or "... {n} other files ..."). The preview panel itself is PN-7.
- Commands: refactorPreview.apply; refactorPreview.discard
- Menus: BulkEditContext
- Keys: Ctrl+Enter or Cmd+Enter (refactorPreview.apply, when the preview pane is focused)
- Settings: none found
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts (lines 111-140 and 165-200); V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts (accept, lines 253-272); C bundle (nls 4909, 4910, 4930, 4931; command ids refactorPreview.apply and refactorPreview.discard)
- Pieces:
  1. Starting a second refactor preview while one is open shows a box that says "Another refactoring is being previewed." with the detail "Press 'Continue' to discard the previous refactoring and continue with the current refactoring."
  2. Choosing Continue discards the earlier preview and goes on with the new one, and choosing Cancel keeps the earlier preview.
  3. Pressing Apply in the preview (`refactorPreview.apply`, Ctrl+Enter or Cmd+Enter while the preview pane is focused) when one file changed in the meantime shows a warning box "Cannot apply refactoring because '{file}' has changed in the meantime."
  4. When several files changed in the meantime, the warning says "Cannot apply refactoring because {n} other files have changed in the meantime."

#### DL-1.30 No updates available box

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - older flow, renderer-only, no last-focus check and no macOS windowless box
- What: After you ask for an update check by hand and nothing new exists, an info box says "There are currently no updates available." Stock only shows it from the window that last had focus, and on macOS with every window closed the main process shows a native box instead. Cursor still shows the older version of this flow, with no focus check and no windowless macOS box.
- Commands: update.checkForUpdate
- Menus: MenubarHelpMenu (Check for Updates row, see TB-1)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\update\browser\update.ts (lines 262-270); V\src\vs\platform\update\electron-main\notAvailableUpdateDialog.ts; C bundle (onUpdateNotAvailable calling the info box, nls 11502); C out\main.js (no copy of the main-process box found)
- Pieces:
  1. After you ask for an update check by hand and nothing new exists, an info box says "There are currently no updates available."
  2. In stock the box shows only from the window that last had focus. [stock only]
  3. In stock on macOS with every window closed, the main process shows a native box with the same message instead. [stock only]
  4. In Cursor the box comes from the older renderer-only flow, with no last-focus check and no windowless macOS box. [cursor only]

#### DL-1.31 Unsupported remote OS confirm

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Before connecting to a remote Linux that the app no longer supports, a warning box says "You are about to connect to an OS version that is unsupported by {product}." with Allow, Learn More and Cancel, and a "Do not show again" checkbox that remembers Allow for that remote. Learn More opens the FAQ and then asks again. The banner that follows is BN-2.5.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\remoteConnectionHealth.ts (_confirmConnection, lines 44-80); C bundle (nls 8946 to 8949, storage key constant REMOTE_UNSUPPORTED_CONNECTION_CHOICE)
- Pieces:
  1. Before connecting to a remote Linux that the app no longer supports, a warning box says "You are about to connect to an OS version that is unsupported by {product}."
  2. The box has Allow, Learn More and Cancel buttons.
  3. A "Do not show again" checkbox remembers the Allow answer for that remote.
  4. Choosing Learn More opens the FAQ page and then shows the box again.
  5. Choosing Cancel does not connect.

#### DL-1.32 Pre-launch task error prompt

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a debug pre-launch task ends with errors and debug.onTaskErrors is "prompt" (the default), a warning box asks what to do with buttons "Debug Anyway", "Show Errors" and "Abort", and a "Remember my choice in user settings" checkbox. A second box covers a task that could not run at all: Debug Anyway, a configure-task button and Cancel, with "Remember my choice for this task" kept per workspace. The debug UI itself is SB-5.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.onTaskErrors
- Source: V\src\vs\workbench\contrib\debug\browser\debugTaskRunner.ts (lines 96-190); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (debug.onTaskErrors); C bundle (nls 5907 to 5915, setting debug.onTaskErrors)
- Pieces:
  1. When a debug pre-launch task ends with errors and `debug.onTaskErrors` is `prompt` (the default), a warning box asks what to do, with buttons "Debug Anyway", "Show Errors" and "Abort".
  2. A "Remember my choice in user settings" checkbox on that box saves your button choice into `debug.onTaskErrors`.
  3. Setting `debug.onTaskErrors` chooses what happens without a box, with values `prompt`, `debugAnyway`, `showErrors` and `abort`.
  4. A second box covers a pre-launch task that could not run at all, with a Debug Anyway button, a button that opens the task configuration, and Cancel.
  5. A "Remember my choice for this task" checkbox on the second box keeps the answer per workspace.

#### DL-1.33 Window not responding or crashed box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The main process, not the page, shows an OS warning box when a window hangs or dies. A hang says "The window is not responding" with Reopen, Close and Keep Waiting. A crash says "The window terminated unexpectedly" (with the reason and exit code when known) and offers Reopen, or New Window for an empty window, and Close. When a folder or workspace is open a "Don't restore editors" checkbox lets the reopened window skip its saved editors. A hang is ignored while dev tools are open or in an extension development host.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\windows\electron-main\windowImpl.ts (onWindowError, lines 935-1010); C out\main.js (nls 2716 to 2729 in the same handler, isDevToolsOpened check)
- Pieces:
  1. When a window hangs, the main process shows an OS warning box that says "The window is not responding" with Reopen, Close and Keep Waiting buttons.
  2. When a window dies, an OS warning box says "The window terminated unexpectedly", with the reason and exit code when they are known.
  3. The crash box offers Reopen (or New Window for an empty window) and Close.
  4. When a folder or workspace is open, a "Don't restore editors" checkbox lets the reopened window skip its saved editors.
  5. A hang is ignored while developer tools are open or in an extension development host.

#### DL-1.34 Path does not exist box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the main process is asked to open a path that is not on the machine (for example from the command line), it shows an OS info box "Path does not exist" with "The path '{path}' does not exist on this computer.", or "URI can not be opened" for a bad URI, and an OK button.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\windows\electron-main\windowsMainService.ts (lines 908-925); C out\main.js (nls 2732 in the same flow)
- Pieces:
  1. When the main process is asked to open a path that does not exist, for example from the command line, an OS info box titled "Path does not exist" says "The path '{path}' does not exist on this computer."
  2. For a bad URI the box is titled "URI can not be opened" instead.
  3. Pressing the OK button closes the box.

#### DL-1.35 Workspace does not exist box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a remote window whose folder or workspace file is gone, a warning box says "Workspace does not exist" with the detail "Please select another workspace to open." and an "Open Workspace..." button (or a Cancel). Confirming opens the workspace or folder picker (DL-3).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\common\remote.contribution.ts (RemoteInvalidWorkspaceDetector, lines 120-146); C bundle (nls 9071 to 9073)
- Pieces:
  1. In a remote window whose folder or workspace file is gone, a warning box says "Workspace does not exist" with the detail "Please select another workspace to open."
  2. The box has an "Open Workspace..." button and a Cancel button.
  3. Choosing "Open Workspace..." opens the workspace file picker when a workspace file was open, and the folder picker when a folder was open.
  4. Choosing Cancel closes the box and leaves the window as it is.

#### DL-1.36 Markdown in dialog detail

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: newer-than-1.128. The detail line of a box can be Markdown. The in-page box renders it with working links, and the OS box shows it as plain text. Stock 1.128 and Cursor take plain text only.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: N\src\vs\platform\dialogs\common\dialogs.ts (detail may be an IMarkdownString); N\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts; N\src\vs\base\browser\ui\dialog\dialog.ts (detailElement); C bundle (zero hits for detailElement)
- Pieces:
  1. In later stock builds the detail line of a box can be Markdown. [stock only]
  2. The in-page box renders that Markdown with working links. [stock only]
  3. The OS box shows the same detail as plain text. [stock only]

#### DL-1.37 Stacked button layout

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: newer-than-1.128. A caller can ask for a vertical layout, with full-width buttons stacked under the message, through the custom dialog options. The 1.128 widget already has the layout, but only the Sessions code under V\src\vs\sessions uses it, and the service option to reach it is newer. Cursor has neither.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: N\src\vs\platform\dialogs\common\dialogs.ts (alignment option, line 305); N\src\vs\workbench\browser\parts\dialogs\dialogHandler.ts (line 133); V\src\vs\base\browser\ui\dialog\dialog.ts (DialogContentsAlignment); C bundle (zero hits for align-vertical)
- Pieces:
  1. In later stock builds a caller can ask for a vertical layout, with full-width buttons stacked under the message. [stock only]
  2. The service option that reaches this layout from the custom dialog options exists only in later stock, and the 1.128 widget already has the layout but only the Sessions code uses it. [stock only]

#### DL-1.38 Restart prompt after changing a restart-only setting

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the same watcher (class with a SETTINGS list) and the same box exist, with nls 8923 to 8928 holding the texts; the list adds update.mode, workbench.activityBar.orientation and application.experimental.extensionMonitor.enabled and has no window.menuStyle or telemetry.feedback.enabled entry
- What: When the person changes a setting that only works after a restart, a confirm box offers to restart the app right away.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.enableExperiments; security.restrictUNCAccess; window.titleBarStyle; window.menuStyle; window.nativeTabs; window.nativeFullScreen; window.clickThroughInactive; window.controlsStyle; editor.accessibilitySupport; security.workspace.trust.enabled; accessibility.verbosity.debug; telemetry.feedback.enabled; update.mode; workbench.activityBar.orientation
- Source: V\src\vs\workbench\contrib\relauncher\browser\relauncher.contribution.ts (lines 56-250); V\src\vs\workbench\services\assignment\common\assignmentService.ts (lines 347-350); C bundle (class with fri.SETTINGS, the nls pair 8923 and 8924, setting workbench.enableExperiments registered with default true and the usesOnlineServices tag); NLS 8923 to 8928, 12297
- Pieces:
  1. After a restart-only setting changes, a confirm box says "A setting has changed that requires a restart to take effect." with the detail "Press the restart button to restart {product} and enable the setting." and a Restart button next to Cancel.
  2. Choosing Restart restarts the app, and cancelling keeps the new value but it only takes effect at the next start.
  3. The box appears only while the window has focus, only when the value really differs from the one in use, and not when just a default changed or while settings sync is being turned on.
  4. In the desktop app stock watches `window.titleBarStyle`, `window.menuStyle` (Windows and Linux), `window.nativeTabs`, `window.nativeFullScreen`, `window.clickThroughInactive` (macOS), `window.controlsStyle` (Windows and Linux), `editor.accessibilitySupport` (Linux, when turned on), `security.workspace.trust.enabled`, `security.restrictUNCAccess`, `accessibility.verbosity.debug`, `telemetry.feedback.enabled`, `workbench.enableExperiments` and, on builds that are not stable, `_extensionsGallery.enablePPE`.
  5. Setting `workbench.enableExperiments` (on by default, tagged as using an online service) raises the box when it is switched, and that prompt is its only visible effect in the app.
  6. Cursor watches `update.mode`, `workbench.activityBar.orientation` and `application.experimental.extensionMonitor.enabled` as well and does not watch `window.menuStyle` or `telemetry.feedback.enabled`. [cursor only]


## DL-2 Unsaved changes, close and quit prompts

(both). Boxes that stop you losing work when closing an editor or window, quitting, or leaving an untitled workspace. Examples: Save, Don't Save, Cancel; confirm on quit or close; untitled workspace save; the hot-exit backup failed box.

20 features.

#### DL-2.1 Save changes box for one dirty editor

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Closing an editor that has unsaved changes brings it to the front and asks "Do you want to save the changes you made to {file}?" with the detail "Your changes will be lost if you don't save them." and three buttons: Save, Don't Save, Cancel. Save writes the file and closes (an untitled file goes to Save As, see DL-3). Don't Save reverts and closes, even if the revert fails. Cancel keeps the editor open. Group commands that close several editors one by one, such as Close Others, ask once per dirty editor in turn. Editors with their own close prompt (the merge editor, for example) show that one instead.
- Commands: workbench.action.closeActiveEditor; workbench.action.closeEditorsInGroup; workbench.action.closeOtherEditors; workbench.action.closeEditorsToTheRight
- Menus: MenubarFileMenu (Close Editor, group 6_close, order 2); EditorTitleContext (Close, Close Others, Close to the Right, Close All); EditorTitle (Close button)
- Keys: Ctrl+W, or Cmd+W on macOS (closeActiveEditor; Windows also Ctrl+F4); Ctrl+K W (closeEditorsInGroup); Cmd+Alt+T on macOS (closeOtherEditors)
- Settings: window.dialogStyle
- Source: V\src\vs\workbench\browser\parts\editor\editorGroupView.ts (handleCloseConfirmation, lines 1677-1706; doHandleCloseConfirmation, lines 1708-1860); V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts (showSaveConfirm, lines 167-222); V\src\vs\workbench\browser\parts\editor\editorCommands.ts (close commands, lines 828-930); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (menus at lines 394-398 and 742-750); C bundle (doShowSaveConfirm, nls 12452 to 12457, close command ids)
- Pieces:
  1. Closing an editor that has unsaved changes brings it to the front and asks "Do you want to save the changes you made to {file}?" with the detail "Your changes will be lost if you don't save them."
  2. The box has three buttons: Save, Don't Save and Cancel.
  3. Choosing Save writes the file and closes the editor, and an untitled file goes to Save As first.
  4. Choosing Don't Save reverts the editor and closes it, even if the revert fails.
  5. Choosing Cancel keeps the editor open.
  6. The box is reached by pressing Ctrl+W (Cmd+W on macOS, and Ctrl+F4 on Windows) for `workbench.action.closeActiveEditor`, by File > Close Editor, by the Close button on the tab, and by Close in the tab context menu.
  7. Commands that close several editors one by one, such as Close Others (`workbench.action.closeOtherEditors`), Close to the Right (`workbench.action.closeEditorsToTheRight`) and Close Editors in Group (`workbench.action.closeEditorsInGroup`, Ctrl+K W), ask once per dirty editor in turn.
  8. An editor with its own close prompt, such as the merge editor, shows that prompt instead of this box.
  9. Setting `window.dialogStyle` chooses whether the box is an OS message box or the in-page box, with values `native` and `custom`.

#### DL-2.2 Save changes box for several files

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When more than one file is at stake the same box says "Do you want to save the changes to the following {n} files?" and lists up to ten names, then "...n additional files not shown", and the first button becomes "Save All". Don't Save and Cancel stay. It is raised by Close All Editors and Close All Editor Groups (DL-2.3) and by quitting or closing with no backup (DL-2.5).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.dialogStyle
- Source: V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts (doShowSaveConfirm, lines 186-222); V\src\vs\platform\dialogs\common\dialogs.ts (getFileNamesMessage, MAX_CONFIRM_FILES, lines 606-622); C bundle (nls 12454, 12455, 1898, 1899)
- Pieces:
  1. When more than one file is at stake, the box asks "Do you want to save the changes to the following {n} files?" and lists the file names.
  2. The list shows up to ten names and then "...n additional files not shown".
  3. The first button reads "Save All", and Don't Save and Cancel stay as they are.
  4. The box is raised by Close All Editors and Close All Editor Groups (DL-2.3), and by quitting or closing with unsaved files that could not be backed up (DL-2.5).
  5. Setting `window.dialogStyle` chooses whether the box is an OS message box or the in-page box, with values `native` and `custom`.

#### DL-2.3 Close all editors or groups with unsaved changes

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Close All Editors and Close All Editor Groups first sort the open editors. Editors that auto-save on focus change or window change are saved quietly. The rest are revealed so you can see them, then one box (DL-2.2 when there are several) asks for all the ordinary ones, and editors with their own close prompt are asked after. Cancel at any step stops the whole close. Close All Editors skips pinned tabs, Close All Editor Groups does not. A later stock build also skips editors that cannot be closed and removes only empty groups (newer-than-1.128, not in Cursor).
- Commands: workbench.action.closeAllEditors; workbench.action.closeAllGroups
- Menus: CommandPalette (f1, category View)
- Keys: Ctrl+K Ctrl+W (closeAllEditors); Ctrl+K Ctrl+Shift+W (closeAllGroups)
- Settings: files.autoSave
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (AbstractCloseAllAction, lines 555-757; CloseAllEditorsAction, line 757; CloseAllEditorGroupsAction, line 781); N\src\vs\workbench\browser\parts\editor\editorActions.ts (CannotClose check); C bundle (ids workbench.action.closeAllEditors and workbench.action.closeAllGroups, revealEditorsToConfirm)
- Pieces:
  1. Close All Editors (`workbench.action.closeAllEditors`, Ctrl+K Ctrl+W) and Close All Editor Groups (`workbench.action.closeAllGroups`, Ctrl+K Ctrl+Shift+W, both also in the Command Palette under View) check every open editor for unsaved changes before closing.
  2. Dirty editors that auto-save on focus change or window change are saved quietly, under `files.autoSave`, without a box.
  3. The remaining ordinary dirty editors are revealed so you can see them, and then one box asks about all of them, using the DL-2.2 wording when there are several.
  4. Editors that have their own close prompt are asked about after the ordinary ones.
  5. Choosing Cancel at any step stops the whole close.
  6. Close All Editors skips pinned tabs, and Close All Editor Groups does not.
  7. A later stock build (newer-than-1.128) also skips editors that cannot be closed and removes only empty groups. [stock only]

#### DL-2.4 Auto save replaces the prompt on close

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With auto save set to focus change (or window change on Windows and Linux), closing a dirty file saves it instead of asking, because the box itself would take focus and trigger the save anyway. If that save fails, the normal Save, Don't Save, Cancel box appears. Untitled files are never auto-saved, so they still ask. At quit or window close, any file under an auto save mode is saved first and only the remainder is asked about.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.autoSave
- Source: V\src\vs\workbench\browser\parts\editor\editorGroupView.ts (autoSave branch, lines 1738-1770 and 1812-1830); V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (onBeforeShutdownWithModified, lines 88-113); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (files.autoSave, lines 261-275); C bundle (getAutoSaveMode, setting files.autoSave)
- Pieces:
  1. With `files.autoSave` set to `onFocusChange` (or `onWindowChange` on Windows and Linux), closing a dirty file saves it and does not show the box.
  2. If that save fails, the normal Save, Don't Save and Cancel box appears.
  3. Untitled files are never auto-saved, so closing one still shows the box.
  4. At quit or window close, files under an auto save mode are saved first, and the box asks only about what remains.

#### DL-2.5 Save prompt at quit, window close or workspace change

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the app quits, a window closes or reloads, or the folder changes, and unsaved files could not be backed up, a Save All, Don't Save, Cancel box lists them (DL-2.2 wording). Save All saves each (an untitled file opens Save As); if any save fails or is cancelled the quit or close is stopped. Don't Save reverts them. Cancel stops the quit or close. It shows whenever hot exit is off. With hot exit on it still shows when a window closes while the app keeps running (macOS, or another window stays open on Windows and Linux) and when the folder or workspace changes, except that the "on exit and window close" value backs up folder windows instead.
- Commands: workbench.action.quit; workbench.action.closeWindow; workbench.action.closeFolder; workbench.action.reloadWindow
- Menus: MenubarFileMenu (Exit, group z_Exit, order 1, not macOS); MenubarFileMenu (Close Window, group 6_close, order 4); MenubarFileMenu (Close Folder or Close Workspace, group 6_close, order 3)
- Keys: Cmd+Q (macOS and Linux, quit); Cmd+Shift+W on macOS, Alt+F4 on Windows and Linux, or Ctrl+Shift+W (closeWindow); Ctrl+K F (closeFolder); Ctrl+R in development builds only (reloadWindow)
- Settings: files.hotExit
- Source: V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (onBeforeShutdown, handleModifiedBeforeShutdown, shouldBackupBeforeShutdown, confirmBeforeShutdown, lines 60-330); V\src\vs\workbench\electron-browser\desktop.contribution.ts (quit and Exit row, lines 72-92 and 129-139); V\src\vs\workbench\electron-browser\actions\windowActions.ts (CloseWindowAction, lines 36-67); V\src\vs\workbench\browser\actions\workspaceActions.ts (CloseWorkspaceAction, lines 161-185); C bundle (backup tracker class with isHotExitEnabled, nls 13589 to 13600)
- Pieces:
  1. When the app quits, a window closes or reloads, or the folder changes, and unsaved files could not be backed up, a box with Save All, Don't Save and Cancel lists them with the DL-2.2 wording.
  2. The box is reached by File > Exit or Cmd+Q (`workbench.action.quit`), by File > Close Window (`workbench.action.closeWindow`, Cmd+Shift+W, Alt+F4 or Ctrl+Shift+W), by File > Close Folder or Close Workspace (`workbench.action.closeFolder`, Ctrl+K F), and by `workbench.action.reloadWindow` (Ctrl+R in development builds only).
  3. Choosing Save All saves each file, and an untitled file opens Save As.
  4. If any save fails or is cancelled, the quit or close is stopped.
  5. Choosing Don't Save reverts the files and the quit or close goes ahead.
  6. Choosing Cancel stops the quit or close.
  7. With `files.hotExit` set to `off`, the box shows whenever a window closes or the app quits with unsaved files.
  8. With hot exit on, the box still shows when a window closes while the app keeps running and when the folder or workspace changes, and the value `onExitAndWindowClose` backs up folder windows instead of asking.

#### DL-2.6 Hot exit backs up unsaved files instead of asking

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Hot exit remembers unsaved files, including untitled ones, so quitting does not ask. The default (on exit) backs up when you quit or close the last window on Windows and Linux. The second value also backs up any window that has a folder open, even when it is not the last window. Off turns it off and brings back the box in DL-2.5. Windows without a folder come back at the next start (DL-2.7).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.hotExit
- Source: V\src\vs\workbench\contrib\files\browser\files.contribution.ts (hotExitConfiguration, lines 123-146; files.hotExit, line 321); V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (shouldBackupBeforeShutdown, lines 185-230); V\src\vs\platform\files\common\files.ts (HotExitConfiguration); C bundle (setting files.hotExit with nls 7222 to 7228)
- Pieces:
  1. With hot exit on, quitting does not ask about unsaved files, because the app remembers them, untitled files included.
  2. Setting `files.hotExit` chooses when unsaved files are backed up, with values `off`, `onExit` (the default) and `onExitAndWindowClose`.
  3. With `onExit`, unsaved files are backed up when you quit, or close the last window on Windows and Linux.
  4. With `onExitAndWindowClose`, unsaved files are also backed up for any window that has a folder open, even when it is not the last window.
  5. With `off`, nothing is backed up and the Save All, Don't Save, Cancel box in DL-2.5 comes back.

#### DL-2.7 Reopen restores unsaved files

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: After a hot exit, the next start reopens windows as set by the restore setting (default all of them). Each backed-up unsaved file returns as a modified tab, opened in the background so it does not take focus, with its unsaved text back. Windows that had no folder return as empty windows holding those files. Backups of a folder window stay on disk until that folder is opened again, and the folder shows a dirty marker in Open Recent (DL-2.14).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.restoreWindows; files.hotExit
- Source: V\src\vs\workbench\services\workingCopy\common\workingCopyBackupTracker.ts (restoreBackups, lines 335-420); V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (onBeforeShutdownWithoutModified, lines 377-420); V\src\vs\platform\windows\electron-main\windowsMainService.ts (restore setting, lines 1009-1078); V\src\vs\workbench\electron-browser\desktop.contribution.ts (window.restoreWindows, lines 183-196); C bundle (unrestoredBackups, restoreBackups, setting window.restoreWindows)
- Pieces:
  1. After a hot exit, the next start reopens windows as set by `window.restoreWindows`, with values `preserve`, `all` (the default), `folders`, `one` and `none`.
  2. Each backed-up unsaved file returns as a modified tab with its unsaved text back.
  3. The restored tabs open in the background, so they do not take focus.
  4. Windows that had no folder return as empty windows holding those files.
  5. Backups of a folder window stay on disk until that folder is opened again, and the folder shows a dirty marker in Open Recent (DL-2.14).

#### DL-2.8 Waiting box when a shutdown step is slow

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: If backing up, saving, reverting or discarding backups at shutdown takes a while, a pending box with a spinner appears and says "Backing up editors with unsaved changes is taking a bit longer...", "Saving editors...", "Reverting editors..." or "Discarding backups...". The backup box adds "Click 'Cancel' to stop waiting and to save or revert editors with unsaved changes." Cancel stops the wait and falls back to the save prompt. When untitled files are being saved the progress moves to the status bar instead, because a Save As box may need the screen. The box look is DL-1.16 and the progress service side is NT-5.10.
- Commands: none found
- Menus: none found
- Keys: Escape (cancel)
- Settings: none found
- Source: V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (withProgressAndCancellation callers, lines 256-262, 331-359, 361-375, 425-460); V\src\vs\workbench\services\progress\browser\progressService.ts (withDialogProgress); C bundle (nls 13596 to 13600)
- Pieces:
  1. When backing up, saving, reverting or discarding backups at shutdown takes a while, a pending box with a spinner appears.
  2. The box message reads "Backing up editors with unsaved changes is taking a bit longer...", "Saving editors...", "Reverting editors..." or "Discarding backups...", depending on the step.
  3. The backup box adds "Click 'Cancel' to stop waiting and to save or revert editors with unsaved changes."
  4. Pressing Cancel, or Escape, stops the wait and falls back to the save prompt.
  5. While untitled files are being saved, the progress moves to the status bar instead of a box, because a Save As box may need the screen.

#### DL-2.9 Backup or save failed box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: If the backup cannot be written, or the save or revert you chose fails, an error box says "The following editors with unsaved changes could not be saved to the backup location." (or "...could not be saved or reverted."). The detail lists the files and adds "Try saving or reverting the editors with unsaved changes first and then try again." OK keeps the window open. The second button is "Close Anyway", "Quit Anyway" or "Reload Anyway", depending on what you were doing, and goes ahead and loses the changes. It is skipped in an extension development window.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.hotExit
- Source: V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (handleModifiedBeforeShutdown, showErrorDialog, toForceShutdownLabel, lines 142-182 and 232-262); C bundle (nls 13589 to 13595, 4086 to 4088)
- Pieces:
  1. If the backup cannot be written, an error box says "The following editors with unsaved changes could not be saved to the backup location."
  2. If the save or revert you chose fails, the box says "...could not be saved or reverted." instead.
  3. The detail lists the files and adds "Try saving or reverting the editors with unsaved changes first and then try again."
  4. Choosing OK keeps the window open.
  5. The second button reads "Close Anyway", "Quit Anyway" or "Reload Anyway", depending on what you were doing, and goes ahead and loses the changes.
  6. The box is skipped in an extension development window.

#### DL-2.10 Confirm before quit or close

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - Cursor's own quit and close boxes run first; the checkbox is not shown in the Agents window
- What: One setting asks "Are you sure you want to exit?" (Windows and Linux), "Are you sure you want to quit?" (macOS) or "Are you sure you want to close the window?" with an Exit, Quit or Close Window button, Cancel, and a "Do not ask me again" checkbox that switches the setting to never. Values: always, only when a key started the action, or never. The desktop default is never, a browser tab defaults to key-only. Closing the last window on Windows and Linux counts as quit. A File menu toggle called Confirm Before Close exists only in the browser build. In Cursor the extra boxes in DL-2.17 and DL-2.18 come first, and the stock box still runs after them when the setting asks for it. I did not run the case where both appear.
- Commands: workbench.action.quit; workbench.action.toggleConfirmBeforeClose
- Menus: MenubarFileMenu (Confirm Before Close, group z_ConfirmClose, order 1, web only)
- Keys: Cmd+Q (macOS and Linux, quit)
- Settings: window.confirmBeforeClose
- Source: V\src\vs\workbench\browser\window.ts (confirmOnShutdown, lines 183-210); V\src\vs\workbench\electron-browser\window.ts (onBeforeShutdown, lines 468-505); V\src\vs\workbench\electron-browser\desktop.contribution.ts (quit, lines 72-92); V\src\vs\workbench\browser\workbench.contribution.ts (window.confirmBeforeClose, lines 933-952); V\src\vs\workbench\browser\actions\windowActions.ts (toggle and menu row, lines 495-511); C bundle (confirmOnShutdown class with nls 4067 to 4073, setting default expression, veto.confirmBeforeClose)
- Pieces:
  1. When `window.confirmBeforeClose` asks, quitting shows "Are you sure you want to exit?" on Windows and Linux or "Are you sure you want to quit?" on macOS, with an Exit or Quit button and Cancel.
  2. When `window.confirmBeforeClose` asks, closing a window shows "Are you sure you want to close the window?" with a Close Window button and Cancel.
  3. A "Do not ask me again" checkbox on the box switches `window.confirmBeforeClose` to `never`.
  4. Setting `window.confirmBeforeClose` chooses when the box shows, with values `always`, `keyboardOnly` (only when a keybinding started the action) and `never`.
  5. The desktop default is `never`, and a browser tab defaults to `keyboardOnly`.
  6. Closing the last window on Windows and Linux counts as quit.
  7. The File menu has a Confirm Before Close toggle (`workbench.action.toggleConfirmBeforeClose`) in the browser build only. [stock only]
  8. In Cursor the Quit Cursor? and Close this window? boxes (DL-2.17, DL-2.18) come first, and the stock box still shows after them when the setting asks for it. [cursor only]

#### DL-2.11 macOS app menu Quit confirm

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: On macOS the Quit row of the app menu is handled by the main process, not the page. When the setting asks for it (always, or key-only and a key was used) it shows an OS question box "Are you sure you want to quit?" with Quit and Cancel. It only acts when a window has focus, is minimized, or no window is open. Cursor keeps this code and I found no call to its own quit box in it, so this row probably skips the DL-2.17 box. Read from code only, not run on a Mac.
- Commands: workbench.action.quit
- Menus: macOS application menu (built in the main process)
- Keys: Cmd+Q
- Settings: window.confirmBeforeClose
- Source: V\src\vs\platform\menubar\electron-main\menubar.ts (quit row, lines 424-440; confirmBeforeQuit, lines 464-481); C out\main.js (confirmBeforeQuit with nls 2182 to 2184)
- Pieces:
  1. On macOS, choosing Quit in the app menu (or pressing Cmd+Q) shows an OS question box "Are you sure you want to quit?" with Quit and Cancel when the setting asks for it.
  2. Setting `window.confirmBeforeClose` decides whether the box shows, so `always` shows it, `keyboardOnly` shows it only when a key was used, and `never` skips it.
  3. The box only acts when a window has focus, is minimized, or no window is open.

#### DL-2.12 Closing a floating window keeps unsaved editors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When you close a floating editor window, clean editors just close and every editor with unsaved changes (or its own close prompt) is moved back into the main window, so no save box appears. If an editor cannot be moved, the window stays open. When the confirm-before-close setting asks, the same "close the window" box (DL-2.10) comes first. If something vetoes the close, an error box shows the reason with "Try saving or reverting the editors with unsaved changes first and then try again."
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.confirmBeforeClose
- Source: V\src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts (onBeforeUnload, doClose, mergeGroupsToMainPart, lines 265-280 and 425-480); V\src\vs\workbench\services\auxiliaryWindow\browser\auxiliaryWindowService.ts (lines 160-200); V\src\vs\workbench\services\auxiliaryWindow\electron-browser\auxiliaryWindowService.ts (lines 100-125); C bundle (excludeConfirming, mergeGroupsToMainPart, handleVetoBeforeClose, nls 12321 and 12316)
- Pieces:
  1. Closing a floating editor window with only clean editors just closes it.
  2. Every editor with unsaved changes, or with its own close prompt, moves back into the main window, so no save box appears.
  3. If an editor cannot be moved back, the floating window stays open.
  4. When `window.confirmBeforeClose` asks, the "close the window" box (DL-2.10) shows first.
  5. If something vetoes the close, an error box shows the reason and adds "Try saving or reverting the editors with unsaved changes first and then try again."

#### DL-2.13 Untitled workspace save prompt

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Closing a window that holds an untitled multi-root workspace, or switching to another workspace, asks "Do you want to save your workspace configuration as a file?" with the detail "Save your workspace if you plan to open it again." Save opens a Save As picker for a workspace file (cancel the picker and the window stays open). Don't Save deletes the untitled workspace. Cancel stops the close. A checkbox "Always discard untitled workspaces without asking" turns the prompt off. It is not asked when closing the last window on Windows and Linux, because that is a quit. With the setting off the untitled workspace is deleted silently.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.confirmSaveUntitledWorkspace
- Source: V\src\vs\workbench\services\workspaces\electron-browser\workspaceEditingService.ts (saveUntitledBeforeShutdown, lines 74-160); V\src\vs\workbench\electron-browser\desktop.contribution.ts (window.confirmSaveUntitledWorkspace, lines 167-171); C bundle (nls 13614 to 13618, setting window.confirmSaveUntitledWorkspace, 3 hits)
- Pieces:
  1. Closing a window that holds an untitled multi-root workspace, or switching to another workspace, asks "Do you want to save your workspace configuration as a file?" with the detail "Save your workspace if you plan to open it again."
  2. Choosing Save opens a Save As picker for a workspace file, and cancelling the picker keeps the window open.
  3. Choosing Don't Save deletes the untitled workspace and the close goes ahead.
  4. Choosing Cancel stops the close.
  5. An "Always discard untitled workspaces without asking" checkbox on the box turns the prompt off.
  6. With `window.confirmSaveUntitledWorkspace` off, the untitled workspace is deleted silently and no box shows.
  7. The box is not shown when closing the last window on Windows and Linux, because that counts as quit.

#### DL-2.14 Folder or workspace with unsaved files in Open Recent

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In the Open Recent list, a folder or workspace that still holds unsaved backups gets a dirty-dot button with the tooltip "Folder With Unsaved Files" or "Workspace With Unsaved Files". Clicking it asks "Do you want to open the folder to review the unsaved files?" (or workspace) with the detail "Folders with unsaved files cannot be removed until all unsaved files have been saved or reverted." Confirm opens that window. These entries cannot be removed from the list while dirty. Entries that have backups but are not in the history are added to the list too.
- Commands: workbench.action.openRecent; workbench.action.quickOpenRecent
- Menus: MenubarRecentMenu (More..., group y_more, order 1); CommandPalette (f1, category File)
- Keys: Ctrl+R (workbench.action.openRecent; the Control key on macOS too)
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\windowActions.ts (BaseOpenRecentAction, lines 50-260; OpenRecentAction, lines 280-302); C bundle (nls 3240 to 3252, class dirty-workspace, command id workbench.action.openRecent)
- Pieces:
  1. In the Open Recent list, a folder or workspace that still holds unsaved backups gets a dirty-dot button with the tooltip "Folder With Unsaved Files" or "Workspace With Unsaved Files".
  2. Clicking the dirty-dot button asks "Do you want to open the folder to review the unsaved files?" (or workspace) with the detail "Folders with unsaved files cannot be removed until all unsaved files have been saved or reverted."
  3. Confirming opens that folder or workspace in a window.
  4. Entries with unsaved backups cannot be removed from the list until the files are saved or reverted.
  5. Entries that have backups but are not in the history are added to the list too.
  6. The list is reached by File > Open Recent > More..., by Open Recent in the Command Palette (`workbench.action.openRecent`, Ctrl+R), and by `workbench.action.quickOpenRecent`.

#### DL-2.15 Shutdown error box

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: If shutting down throws an unexpected error, an error box says "An unexpected error prevented the window to close" (or "...the application to quit", "...the window to reload", "...to change the workspace") with the detail "Error: {message}".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\window.ts (onBeforeShutdownError and toShutdownLabel, lines 505-580); C bundle (nls 12122, 12124 to 12127)
- Pieces:
  1. If shutting down throws an unexpected error, an error box says "An unexpected error prevented the window to close" with the detail "Error: {message}".
  2. The wording follows what you were doing, as "...the application to quit", "...the window to reload" or "...to change the workspace".

#### DL-2.16 Save a read-only file anyway

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When Save As lands on a path that the read-only settings cover, a warning box says "'{file}' is marked as read-only. Do you want to save anyway?" with the detail "Paths can be configured as read-only via settings." and Save Anyway or Cancel. It is raised from the Save As flow (DL-3.6), and the same text exists in the file working copy manager.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.readonlyInclude; files.readonlyExclude; files.readonlyFromPermissions
- Source: V\src\vs\workbench\services\textfile\browser\textFileService.ts (confirmMakeWriteable, lines 394-404 and 639-648); V\src\vs\workbench\services\workingCopy\common\fileWorkingCopyManager.ts (line 514); C bundle (nls 13121 and 13558, the three settings)
- Pieces:
  1. When Save As lands on a path that the read-only settings cover, a warning box says "'{file}' is marked as read-only. Do you want to save anyway?" with the detail "Paths can be configured as read-only via settings."
  2. Choosing Save Anyway saves the file, and choosing Cancel does not.
  3. Setting `files.readonlyInclude` lists paths or glob patterns to mark as read-only.
  4. Setting `files.readonlyExclude` lists paths or glob patterns that are kept out of `files.readonlyInclude`.
  5. Setting `files.readonlyFromPermissions` marks files as read-only when their file permissions say so.

#### DL-2.17 Quit Cursor? box

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor adds its own box to the Quit command. Unless the app reports running agents or shell tasks (then another flow asks), it opens "Quit Cursor?" with "Are you sure you want to quit?", Quit and Cancel, and a "Don't ask again" checkbox in the button row. It asks at the default setting, where stock never asks. Tick the box and choose Quit and it never asks again. The stock confirm (DL-2.10) runs after it when that setting asks. The macOS menu row may skip it (DL-2.11). The running-agent prompts are agent-only and left out.
- Commands: workbench.action.quit
- Menus: MenubarFileMenu (Exit, group z_Exit, order 1, Windows and Linux)
- Keys: Cmd+Q (macOS and Linux); Enter (Quit); Escape (Cancel)
- Settings: none found
- Source: C bundle (quit command rule with helper HOw and function pvg, dialogKey quitConfirmation, nls 12011, 12013, 12015, 12017)
- Pieces:
  1. The Quit command (File > Exit on Windows and Linux, Cmd+Q on macOS and Linux, `workbench.action.quit`) opens a box titled "Quit Cursor?" with "Are you sure you want to quit?", a Quit button and a Cancel button. [cursor only]
  2. A "Don't ask again" checkbox sits in the button row. [cursor only]
  3. Pressing Enter chooses Quit and pressing Escape chooses Cancel. [cursor only]
  4. The box asks at the default setting, where stock never asks. [cursor only]
  5. Ticking "Don't ask again" and choosing Quit means the box never asks again. [cursor only]

#### DL-2.18 Close this window? box

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor's Close Window command closes at once when no folder or workspace is open. When one is open it first opens "Close this window?" with "Are you sure you want to close this window?", Close and Cancel, and a "Don't ask again" checkbox. It asks at the default setting, where stock never asks. On macOS Cursor adds a Cmd+W command that closes the active editor when there is one (which may ask DL-2.1) and otherwise runs Close Window. I did not confirm whether the window's own close button or the OS-level Alt+F4 reach this box, since the code only calls it from the command.
- Commands: workbench.action.closeWindow; workbench.action.closeWindowOrEditorOnMac
- Menus: MenubarFileMenu (Close Window, group 6_close, order 4); CommandPalette (f1)
- Keys: Cmd+Shift+W (macOS); Alt+F4 (Windows and Linux); Ctrl+Shift+W (Windows and Linux); Cmd+W (macOS, closeWindowOrEditorOnMac); Enter (Close); Escape (Cancel)
- Settings: none found
- Source: C bundle (class with id workbench.action.closeWindow, function pvg, dialogKey closeWindowConfirmation, command workbench.action.closeWindowOrEditorOnMac, nls 12012, 12014, 12016, 12018, 12030)
- Pieces:
  1. The Close Window command (`workbench.action.closeWindow`) closes the window at once when no folder or workspace is open. [cursor only]
  2. When a folder or workspace is open, the command first opens a box titled "Close this window?" with "Are you sure you want to close this window?", a Close button and a Cancel button. [cursor only]
  3. A "Don't ask again" checkbox on the box stops it from asking again. [cursor only]
  4. Pressing Enter chooses Close and pressing Escape chooses Cancel. [cursor only]
  5. The box asks at the default setting, where stock never asks. [cursor only]
  6. The box is reached by File > Close Window, Cmd+Shift+W on macOS, Alt+F4 or Ctrl+Shift+W on Windows and Linux, and the Command Palette. [cursor only]
  7. On macOS Cmd+W (`workbench.action.closeWindowOrEditorOnMac`) closes the active editor when there is one, which may ask DL-2.1, and otherwise runs Close Window. [cursor only]

#### DL-2.19 Slow close, quit or reload dialog with the pending steps

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the dialog branch with nls 12123, the force labels and the step labels are in the bundle, and Cursor adds two steps
- What: When closing, quitting, reloading or switching workspace takes a while, a progress message and then a box that lists the steps still running, with a button to force it, appear.
- Commands: join.closeStorage; join.textFiles; join.workingCopyBackups; join.fileWorkingCopyManager; join.workingCopyHistory; join.stopExtensionHosts; join.disconnectRemote; join.extensionDevelopment; join.browserViews; join.windowAnalytics (all ten are step ids shown in the list, not commands)
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\window.ts:505-590 (progressOnBeforeShutdown, onWillShutdown, labels); V\src\vs\workbench\electron-browser\desktop.main.ts:170; V\src\vs\workbench\services\textfile\electron-browser\nativeTextFileService.ts:64; V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupService.ts:36; V\src\vs\workbench\services\workingCopy\common\workingCopyHistoryService.ts:873; V\src\vs\workbench\services\workingCopy\common\storedFileWorkingCopyManager.ts:216; V\src\vs\workbench\services\extensions\common\abstractExtensionService.ts:214-232; V\src\vs\workbench\services\extensions\electron-browser\localProcessExtensionHost.ts:652; C bundle (steps join.browserViews "Cleaning up browser views", join.windowAnalytics "Flushing analytics events", nls 12095, 12123, 12128 to 12131, 13580, 13586)
- Pieces:
  1. If shutdown is still going after about 0.8 seconds, a progress message shows in the status bar: "Closing the window is taking a bit longer...", with variants for quitting the application, reloading the window and changing the workspace.
  2. If it is still going after about 1.2 seconds, a box with the same title opens and lists "The following operations are still running:" with one line per unfinished step.
  3. The box has one button, "Close Anyway", "Quit Anyway", "Reload Anyway" or "Change Anyway", that forces the shutdown; the box cannot be cancelled or dismissed and disappears when the shutdown finishes.
  4. The step names that can appear are Saving UI state, Saving text files, Backup working copies, Saving working copies, Saving local history, Stopping Extension Hosts, Disconnect Remote Agent (remote windows, last in order) and Terminating extension debug session (extension development windows).
  5. Cursor adds the steps "Cleaning up browser views" and "Flushing analytics events" to that list. [cursor only]
  6. If shutdown fails with an unexpected error, an error box says "An unexpected error prevented the window to close" (with quit, reload and change workspace variants) and shows "Error: <message>" as detail.

#### DL-2.20 Confirm before closing the window with a debug session running

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the setting with values never and always and the default never, and the shutdown check with the veto reason veto.debug, are in the bundle; the box strings come through NLS indexes
- What: With `debug.confirmOnExit` set to always, closing, quitting or reloading the window while a debug session is live shows a warning box that asks whether to stop it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.confirmOnExit
- Source: V\src\vs\workbench\contrib\debug\common\debugLifecycle.ts (lines 14-58); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (lines 663-672); C bundle (setting `debug.confirmOnExit`; class with onBeforeShutdown veto "veto.debug")
- Pieces:
  1. Setting `debug.confirmOnExit` chooses whether the window asks before closing with live debug sessions, with values `never` (the default) and `always`.
  2. With `always` and one debug session running, closing the window shows a warning box "There is an active debug session, are you sure you want to stop it?" with a Stop Debugging button and a Cancel button.
  3. With several sessions running the box reads "There are active debug sessions, are you sure you want to stop them?"
  4. Only top-level sessions are counted, so sessions started by another session do not add to the number.
  5. Cancel keeps the window open and the sessions running, and Stop Debugging lets the window close.
  6. The check does not look at why the window is shutting down, so quitting and reloading ask the same question.
  7. With `never`, or with no session running, no box shows.


## DL-3 File open and save pickers

(both). Windows for browsing to a file or folder to open, add or save, either the OS picker or the app's type-a-path list. Examples: Open File, Folder and Workspace; Save As; `files.simpleDialog.enable`.

20 features.

#### DL-3.1 Open File

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick one or more files in the OS Open File box (or in the in-app list when it is turned on) and open them as editor tabs.
- Commands: workbench.action.files.openFile; workbench.action.files.openFileInNewWindow
- Menus: MenubarFileMenu; CommandPalette
- Keys: Ctrl+O (not macOS native)
- Settings: window.openFilesInNewWindow; files.simpleDialog.enable
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; V\src\vs\platform\dialogs\electron-main\dialogMainService.ts; C bundle (command id workbench.action.files.openFile, messages 1902 and 3268); C main (pickFile, doPick)
- Pieces:
  1. Choosing File > Open File... shows the OS Open File box, and picking one or more files opens each as an editor tab.
  2. Pressing Ctrl+O (not macOS native) or running `workbench.action.files.openFile` from the Command Palette shows the same box.
  3. Setting `window.openFilesInNewWindow` chooses whether the picked files open in a new window or in an existing one, with values `on`, `off` and `default`.
  4. Running `workbench.action.files.openFileInNewWindow` shows the same box and opens the picked files in a new window.
  5. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS Open File box.

#### DL-3.2 Open Folder

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (Windows "Select folder" button label is missing)
- What: Pick a folder and open it as the window's workspace, and on stock Windows the OS box button reads "Select folder" while Cursor leaves the OS default label.
- Commands: workbench.action.files.openFolder; workbench.action.files.openFolderInNewWindow; workbench.action.files.openFolderViaWorkspace
- Menus: MenubarFileMenu; CommandPalette
- Keys: Ctrl+K Ctrl+O (Windows and Linux only)
- Settings: window.openFoldersInNewWindow; files.simpleDialog.enable
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\platform\dialogs\electron-main\dialogMainService.ts; V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; C bundle (command id workbench.action.files.openFolder, message 1901); C main (pickFolder with no buttonLabel)
- Pieces:
  1. Choosing File > Open Folder... shows the OS box for picking one folder, and the picked folder opens as the window's workspace.
  2. Pressing Ctrl+K Ctrl+O (Windows and Linux only) or running `workbench.action.files.openFolder` from the Command Palette shows the same box.
  3. Setting `window.openFoldersInNewWindow` chooses whether the picked folder opens in a new window or replaces the last active window, with values `on`, `off` and `default`.
  4. Running `workbench.action.files.openFolderInNewWindow` shows the same box and opens the picked folder in a new window.
  5. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS box for picking a folder.
  6. On stock Windows the OS box button reads "Select folder". [stock only]

#### DL-3.3 Open file or folder in one box (macOS)

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: On macOS native the Open box lets you choose a file or a folder in the same dialog (macOS `.app` packages are browsed like folders in every OS Open box).
- Commands: workbench.action.files.openFileFolder; workbench.action.files.openFileFolderInNewWindow
- Menus: MenubarFileMenu; CommandPalette
- Keys: Cmd+O (macOS native)
- Settings: window.openFilesInNewWindow; window.openFoldersInNewWindow
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\platform\dialogs\electron-main\dialogMainService.ts; C bundle (command id workbench.action.files.openFileFolder, message 3271); C main (doPick with treatPackageAsDirectory)
- Pieces:
  1. On macOS native, choosing File > Open... shows one OS box where either a file or a folder can be picked, and a file opens as an editor tab while a folder opens as the window's workspace.
  2. Pressing Cmd+O (macOS native) or running `workbench.action.files.openFileFolder` from the Command Palette shows the same box.
  3. Running `workbench.action.files.openFileFolderInNewWindow` shows the same box and opens the pick in a new window.
  4. Settings `window.openFilesInNewWindow` (for a picked file) and `window.openFoldersInNewWindow` (for a picked folder) choose whether the pick opens in a new window, with values `on`, `off` and `default`.
  5. In every OS Open box a macOS `.app` package is browsed like a folder and is not picked as one file.

#### DL-3.4 Open Workspace from File

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick a `.code-workspace` file in a box that filters to the Workspace type and has an Open button, then open it in the window.
- Commands: workbench.action.openWorkspace; workbench.action.openWorkspaceInNewWindow
- Menus: MenubarFileMenu; CommandPalette
- Keys: none found
- Settings: files.simpleDialog.enable
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\platform\dialogs\electron-main\dialogMainService.ts; V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts; C bundle (command id workbench.action.openWorkspace, messages 1903, 1904, 3272, 12461)
- Pieces:
  1. Choosing File > Open Workspace from File... shows an OS box with a Workspace file type filter and an Open button, and picking a `.code-workspace` file opens that workspace in the window.
  2. Running `workbench.action.openWorkspace` from the Command Palette shows the same box.
  3. Running `workbench.action.openWorkspaceInNewWindow` shows the same box and opens the picked workspace in a new window.
  4. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS box for picking a workspace file.

#### DL-3.5 Add Folder to Workspace

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick one or more folders (the button reads Add) to add to the open workspace, which turns a single folder into a multi-root workspace.
- Commands: workbench.action.addRootFolder; addRootFolder
- Menus: MenubarFileMenu; ExplorerContext; CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; C bundle (command id workbench.action.addRootFolder, picker with messages 3289 and 3290)
- Pieces:
  1. Choosing File > Add Folder to Workspace... shows an OS box with an Add button where one or more folders can be picked.
  2. Every picked folder is added to the open workspace, and a window that had a single folder becomes a multi-root workspace.
  3. The same box also opens from the Explorer context menu entry and from `workbench.action.addRootFolder` in the Command Palette.

#### DL-3.6 Save As

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick a new name and folder for the active editor's file, starting from the file's own path, and save a copy there.
- Commands: workbench.action.files.saveAs
- Menus: MenubarFileMenu; CommandPalette
- Keys: Ctrl+Shift+S
- Settings: files.simpleDialog.enable
- Source: V\src\vs\workbench\contrib\files\browser\fileConstants.ts; V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\services\textfile\browser\textFileService.ts; V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; C bundle (command id workbench.action.files.saveAs, messages 7060, 12463, 12464)
- Pieces:
  1. Choosing File > Save As... shows a box titled "Save As" that starts at the active editor file's own folder and name, and saving writes the content to the chosen name and folder.
  2. Pressing Ctrl+Shift+S or running `workbench.action.files.saveAs` from the Command Palette shows the same box.
  3. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS Save box.
  4. In the in-app Save box a name typed without an extension gets the extension of the first file type filter added to it.

#### DL-3.7 Save an untitled file

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Saving a new untitled tab opens the same Save picker with a suggested file name built from the tab name and its language.
- Commands: workbench.action.files.save
- Menus: MenubarFileMenu
- Keys: Ctrl+S
- Settings: files.simpleDialog.enable
- Source: V\src\vs\workbench\services\textfile\browser\textFileService.ts; V\src\vs\workbench\services\workingCopy\common\fileWorkingCopyManager.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; C bundle (command id workbench.action.files.save)
- Pieces:
  1. Pressing Ctrl+S or choosing File > Save on a new untitled tab shows the Save picker.
  2. The suggested file name in the picker is built from the tab name and the extension of its language.
  3. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS Save box for the untitled tab.

#### DL-3.8 Choose a file type when saving

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The OS Save box has a file type list ordered All Files, the type that matches the current extension, every known language, then No Extension.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts; V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; C bundle (messages 12465 All Files and 12466 No Extension)
- Pieces:
  1. The OS Save box has a file type list.
  2. All Files is the first entry in the list.
  3. The entry that matches the current file's extension comes right after All Files, named after its language, or after the extension in capitals when no language matches.
  4. Every other known language follows in sorted order.
  5. No Extension is the last entry in the list.

#### DL-3.9 Save Workspace As

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pick where to save the workspace file in a box titled Save Workspace that filters to the Workspace type, and `.code-workspace` is added if you leave it off.
- Commands: workbench.action.saveWorkspaceAs
- Menus: MenubarFileMenu; CommandPalette
- Keys: none found
- Settings: files.simpleDialog.enable
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\services\workspaces\browser\abstractWorkspaceEditingService.ts; C bundle (command id workbench.action.saveWorkspaceAs, messages 3274, 13609)
- Pieces:
  1. Choosing File > Save Workspace As... or running `workbench.action.saveWorkspaceAs` from the Command Palette shows a Save box titled "Save Workspace" that filters to the Workspace type.
  2. Leaving the `.code-workspace` extension off the typed name makes the saved file end in `.code-workspace`.
  3. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS Save box.

#### DL-3.10 Create File picker

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (command and Create File button are present, but the typed-name entry "Create New File" in the New File list is missing)
- What: The New File command with a file name opens a Save picker whose button reads Create File, then creates the empty file and opens it.
- Commands: workbench.action.files.newFile
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\welcomeViews\common\newFile.contribution.ts; C bundle (command id workbench.action.files.newFile, message 7180, welcome.showNewFileEntries)
- Pieces:
  1. Running `workbench.action.files.newFile` shows a Save box titled "Create File" whose button reads "Create File".
  2. The box suggests the name `Untitled.txt` in the default folder, or the file name the command was given.
  3. Confirming a name creates an empty file at that path and opens it as a pinned editor tab, and cancelling creates nothing.
  4. Typing a file name in the New File list adds an entry "Create New File ({0})" at the top that opens this box with that name. [stock only]

#### DL-3.11 Switch to the in-app simple file dialog

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Turning on `files.simpleDialog.enable` replaces every OS open and save box with the app's own path list, and file systems that are not local files always use it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.simpleDialog.enable
- Source: V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (files.simpleDialog.enable, message 7268, shouldUseSimplified)
- Pieces:
  1. Turning on `files.simpleDialog.enable` makes every open and save command show the app's own path list in place of the OS box.
  2. Setting `files.simpleDialog.enable` is off by default, so the OS box shows until a person turns it on.
  3. A file system that is not local files always shows the in-app path list, whatever the setting says.
  4. The in-app list shows the same title and confirm button label that the OS box would show for that command.

#### DL-3.12 Browse folders in the simple dialog

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The simple dialog lists the current folder in a quick pick with a ".." row, folders first, files with icons, files that miss the type filter hidden, and Enter on a folder steps into it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (SimpleFileDialog class with createBackItem and createItems)
- Pieces:
  1. The simple dialog shows the current folder's entries in a quick pick list, with a ".." row that goes up one folder.
  2. Folders are listed before files, and each file shows its file icon.
  3. Files that do not match the file type filter are left out of the list.
  4. Pressing Enter on a folder row steps into that folder and the list refreshes to show its entries.

#### DL-3.13 Type a path in the simple dialog

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the "Folder path" placeholder text is missing)
- What: The text box holds a full path that starts at the picker's folder, and typing a slash after a folder name, "~" for the home folder or a backslash for the drive root jumps the list there, with a "Folder path" placeholder in stock.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (tryUpdateItems, canTildaEscapeHatch, message 12475, no filePickBox.placeholder assignment)
- Pieces:
  1. The text box in the simple dialog holds a full path that starts at the picker's folder.
  2. Typing a slash after a folder name jumps the list to that folder.
  3. Typing `~` jumps the list to the home folder, and typing a backslash jumps it to the drive root.
  4. Typing a path that does not exist followed by a slash shows "The path does not exist. Use ~ to go to your home directory." under the box.
  5. The empty text box shows the placeholder "Folder path". [stock only]

#### DL-3.14 Autocomplete in the simple dialog

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While you type a name the first matching entry is completed inline with the added letters selected, and Up and Down fill the box with the highlighted entry, but the inline fill is skipped when a screen reader is on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (setAutoComplete with isScreenReaderOptimized check)
- Pieces:
  1. While a person types a name, the first matching entry is completed inline in the text box, with the added letters selected.
  2. Pressing Up or Down fills the text box with the highlighted entry.
  3. When a screen reader is on, the inline completion is skipped.

#### DL-3.15 Show or hide dot files

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: An eye button in the simple dialog toggles files whose names start with a dot, its tooltip reads "Hide dot files" or "Show dot files", and the choice is remembered per workspace under the key `remoteFileDialog.showDotFiles`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (remoteFileDialog.showDotFiles, messages 12473 and 12474)
- Pieces:
  1. Pressing the eye button in the simple dialog hides entries whose names start with a dot, and pressing it again shows them.
  2. While dot files are shown the button is an open eye with the tooltip "Hide dot files", and while they are hidden it is a closed eye with the tooltip "Show dot files".
  3. Dot files are shown by default, and the choice is remembered per workspace under the key `remoteFileDialog.showDotFiles`.

#### DL-3.16 Path messages in the simple dialog

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Accepting a bad path shows a message under the box, such as "Please enter a valid path.", "Please select a file.", "Please select a folder.", "The folder already exists. Please use a new file name.", "Please enter a valid file name." or "Please start the path with a drive letter."
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (messages 12475 to 12487)
- Pieces:
  1. Accepting text that cannot be turned into a path shows "Please enter a valid path." under the box.
  2. Accepting a folder in a file-only Open box shows "Please select a file.", and accepting a file in a folder-only Open box shows "Please select a folder."
  3. Accepting a path that does not exist in an Open box shows "Please enter a path that exists."
  4. Accepting an existing folder's name in a Save box shows "The folder already exists. Please use a new file name.", and an unusable file name shows "Please enter a valid file name."
  5. Accepting a read-only folder in a Save box shows "This folder cannot be used as a save destination. Please choose another folder".
  6. On Windows, accepting the bare root path shows "Please start the path with a drive letter."

#### DL-3.17 Overwrite prompt in the simple Save As

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Saving over a name that exists asks "{0} already exists. Are you sure you want to overwrite it?" as a small prompt with an OK button and a Cancel button, in place of the OS box's own replace question.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.simpleDialog.enable
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (yesNoPrompt, messages 12476 Cancel and 12479)
- Pieces:
  1. Accepting an existing file name in the simple Save As shows a small prompt that asks "{0} already exists. Are you sure you want to overwrite it?", with an OK button and a Cancel button.
  2. Pressing OK overwrites the file, and pressing Cancel closes the prompt and returns to the path list with nothing written.
  3. Pressing Escape or typing in the prompt's box closes the prompt and counts as Cancel.
  4. With the OS Save box the replace question is asked by the OS and not by this prompt.

#### DL-3.18 Offer to create a missing folder

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the save case is present, the folder-only Open case is missing)
- What: Saving into a folder that does not exist asks "The folder {0} does not exist. Would you like to create it?", and in stock the folder-only Open box asks the same when just the last folder is missing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.simpleDialog.enable
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (message 12481 used only in the save branch, open branch ends at message 12484)
- Pieces:
  1. Accepting a Save path in a folder that does not exist shows a prompt asking "The folder {0} does not exist. Would you like to create it?" with the missing folder's name.
  2. Pressing OK in that prompt goes on with the save, and pressing Cancel returns to the path list.
  3. Accepting a path in the folder-only Open box whose last folder is missing, but whose parent exists, asks "The folder {0} does not exist. Would you like to create it?", and OK creates the folder and picks it. [stock only]
  4. If creating the folder fails, "Could not create folder: {0}" shows under the box. [stock only]

#### DL-3.19 Show Local and Open Local commands in a remote window

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a window connected to a remote machine the simple dialog gets a "Show Local" button that closes the list and opens the picker for the local machine's files, backed by the Open Local File, Open Local Folder, Open Local and Save Local File commands.
- Commands: workbench.action.files.openLocalFile; workbench.action.files.openLocalFolder; workbench.action.files.openLocalFileFolder; workbench.action.files.saveLocalFile
- Menus: none found
- Keys: Ctrl+O (not macOS, when remoteFileDialogVisible); Ctrl+K Ctrl+O (not macOS, when remoteFileDialogVisible); Cmd+O (macOS, when remoteFileDialogVisible); Ctrl+Shift+S or Cmd+Shift+S (when remoteFileDialogVisible)
- Settings: none found
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; V\src\vs\workbench\contrib\remote\electron-browser\remote.contribution.ts; C bundle (the four command ids, remoteFileDialogVisible, messages 12467 to 12472)
- Pieces:
  1. In a window connected to a remote machine, the simple dialog shows a "Show Local" button.
  2. Pressing "Show Local" closes the list and opens the matching picker for the local machine's files.
  3. Hovering the "Show Local" button shows the local command's label with its key, such as "Open Local File... (Ctrl+O)".
  4. Running `workbench.action.files.openLocalFile`, `workbench.action.files.openLocalFolder`, `workbench.action.files.openLocalFileFolder` or `workbench.action.files.saveLocalFile` opens the picker for the local machine's files.
  5. While the remote picker is showing, Ctrl+O, Ctrl+K Ctrl+O (not macOS), Cmd+O (macOS) and Ctrl+Shift+S or Cmd+Shift+S open the matching local picker.

#### DL-3.20 Where a picker starts

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pickers start in the last active file's folder (Open Folder starts in the folder that contains the last workspace folder, Open Workspace starts in the current workspace file's folder), then fall back to the folder set in `files.dialog.defaultPath` or the user home.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.dialog.defaultPath
- Source: V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (files.dialog.defaultPath, preferredHome, messages 7266 and 7267)
- Pieces:
  1. A picker starts in the folder of the last active file.
  2. The Open Folder picker starts in the folder that contains the last workspace folder.
  3. The Open Workspace picker starts in the folder of the current workspace file.
  4. With no such folder, a picker starts in the folder set in `files.dialog.defaultPath`, and then in the user home folder.
  5. Setting `files.dialog.defaultPath` takes an absolute folder path, and the Settings editor shows "Default path for file dialogs must be an absolute path (e.g. C:\myFolder or /myFolder)." for a path that is not.


## DL-4 Delete, move and replace confirmations

(both). Explorer boxes that ask before you delete, move, overwrite, paste or drop files. Examples: `explorer.confirmDelete`; drag and drop move; replace existing.

15 features.

#### DL-4.1 Confirm moving a file to the Recycle Bin or Trash

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Deleting in the Explorer asks "Are you sure you want to delete '{0}'?" with a "Move to Recycle Bin" (Windows) or "Move to Trash" button, a note on how to restore, and a "Do not ask me again" checkbox that turns `explorer.confirmDelete` off.
- Commands: moveFileToTrash
- Menus: ExplorerContext
- Keys: Delete (when the Explorer is focused and the item can go to trash); Cmd+Backspace or Delete (macOS, same condition)
- Settings: explorer.confirmDelete; files.enableTrash
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (deleteFiles function, moveFileToTrash, messages 7080, 7081, 7096 to 7100, 7109 to 7113, explorer.confirmDelete)
- Pieces:
  1. Pressing Delete on an Explorer item (Cmd+Backspace or Delete on macOS) asks "Are you sure you want to delete '{0}'?" with a "Move to Recycle Bin" button on Windows and a "Move to Trash" button on other systems.
  2. Choosing Delete in the Explorer context menu shows the same box.
  3. The box adds "You can restore this file from the Recycle Bin." (or "from the Trash"), and a folder or several selected items change the wording to name the folder and its contents or to list the files.
  4. Ticking the "Do not ask me again" checkbox and confirming sets `explorer.confirmDelete` to false.
  5. Setting `explorer.confirmDelete` set to false deletes without this box, and it is on by default.
  6. Setting `files.enableTrash` chooses whether Delete moves items to the OS trash (on by default) or deletes them for good (off), and off shows the permanent delete box instead of this one.

#### DL-4.2 Confirm deleting permanently

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (in Cursor `explorer.confirmDelete` set to false skips only the Trash box and this box always asks, in stock 1.128.0 the setting skips both)
- What: Shift+Delete, the Alt version of the Delete menu item, or Delete when there is no trash asks "Are you sure you want to permanently delete '{0}'?" with a Delete button and "This action is irreversible!" for folders, and this box has no "Do not ask me again" checkbox.
- Commands: deleteFile
- Menus: ExplorerContext
- Keys: Shift+Delete; Cmd+Alt+Backspace (macOS); Delete (when the item cannot go to trash)
- Settings: explorer.confirmDelete; files.enableTrash
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; C bundle (deleteFiles function condition `useTrash && explorer.confirmDelete === false`, messages 7082, 7093 to 7095, 7114 to 7118, setting message 7292)
- Pieces:
  1. Pressing Shift+Delete on an Explorer item (Cmd+Alt+Backspace on macOS) asks "Are you sure you want to permanently delete '{0}'?" with a Delete button.
  2. Holding Alt over the Delete entry in the Explorer context menu turns it into "Delete Permanently", which shows the same box.
  3. When an item cannot be moved to the trash, pressing Delete shows this box and the context menu entry reads "Delete Permanently".
  4. For a folder the box adds "This action is irreversible!", and for a file it adds a note that the file can be restored with the Undo command.
  5. This box has no "Do not ask me again" checkbox.
  6. Setting `explorer.confirmDelete` set to false also skips this box. [stock only]
  7. Setting `explorer.confirmDelete` set to false skips only the Trash box, and this box still asks every time. [cursor only]

#### DL-4.3 Retry after a failed delete

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the Recycle Bin or Trash fails the box asks "Failed to delete using the Recycle Bin. Do you want to permanently delete instead?" with a Delete Permanently button, and when a permanent delete fails it shows the error with a Retry button.
- Commands: moveFileToTrash; deleteFile
- Menus: none found
- Keys: none found
- Settings: files.enableTrash
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; C bundle (deleteFiles catch branch, messages 7105 to 7108)
- Pieces:
  1. When moving to the Recycle Bin or Trash fails, a warning box asks "Failed to delete using the Recycle Bin. Do you want to permanently delete instead?" (reading "Trash" on macOS and Linux) with a "Delete Permanently" button.
  2. Pressing "Delete Permanently" deletes the items for good with no further question, and cancelling leaves them in place.
  3. When a permanent delete fails, a warning box shows the error message with a "Retry" button.
  4. Pressing "Retry" tries the delete again with no further question.
  5. With `files.enableTrash` off every delete is permanent, so only the "Retry" box can show.

#### DL-4.4 Warn when deleting files with unsaved changes

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Deleting a file or folder that has unsaved edits asks "You are deleting {0} with unsaved changes. Do you want to continue?" with "Your changes will be lost if you don't save them.", and it asks even when `explorer.confirmDelete` is off.
- Commands: moveFileToTrash; deleteFile
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; C bundle (messages 7083 to 7087)
- Pieces:
  1. Deleting a file with unsaved changes shows a warning "You are deleting {0} with unsaved changes. Do you want to continue?" with the detail "Your changes will be lost if you don't save them."
  2. For a folder the warning reads "You are deleting a folder {0} with unsaved changes in {1} files. Do you want to continue?" (or "in 1 file"), and for several items it reads "You are deleting files with unsaved changes. Do you want to continue?"
  3. The confirm button reads the same as in the regular delete box, "Move to Recycle Bin" or "Move to Trash", or "Delete" for a permanent delete.
  4. The box still shows when `explorer.confirmDelete` is false.
  5. Confirming this box skips the regular delete box, so the person is not asked twice.

#### DL-4.5 Warn when deleting read-only files

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Deleting a file or folder marked read-only asks "You are deleting a file {0} that is configured to be read-only. Do you want to continue?" with a Continue button, and it asks even when `explorer.confirmDelete` is off.
- Commands: moveFileToTrash; deleteFile
- Menus: none found
- Keys: none found
- Settings: files.readonlyInclude; files.readonlyExclude; files.readonlyFromPermissions
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (messages 7088 to 7092, files.readonlyInclude)
- Pieces:
  1. Deleting a file marked read-only shows a warning "You are deleting a file {0} that is configured to be read-only. Do you want to continue?" with the detail "The read-only protection will be overridden if you continue." and a "Continue" button.
  2. For a folder the warning says "You are deleting a folder {0} that is configured to be read-only...", and for several items it says "You are deleting files that are configured to be read-only..."
  3. The box still shows when `explorer.confirmDelete` is false.
  4. Settings `files.readonlyInclude` and `files.readonlyExclude` take paths or glob patterns that mark files as read-only and take some of them back out.
  5. Setting `files.readonlyFromPermissions` (off by default) also marks files read-only when their file permissions say so.

#### DL-4.6 Confirm moving files by drag and drop

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Dragging items to another folder inside the Explorer asks "Are you sure you want to move '{0}' into '{1}'?" (or lists several files) with a Move button and a "Do not ask me again" checkbox, and holding Ctrl (Alt on macOS) copies instead and skips the box.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.confirmDragAndDrop; explorer.enableDragAndDrop
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (handleExplorerDrop, messages 7366, 7368, 7369, 7370, explorer.confirmDragAndDrop)
- Pieces:
  1. Dragging an item onto another folder in the Explorer asks "Are you sure you want to move '{0}' into '{1}'?" with a "Move" button.
  2. Dragging several items asks "Are you sure you want to move the following {0} files into '{1}'?" and lists their names.
  3. Holding Ctrl (Alt on macOS) while dropping copies the items instead and skips this box.
  4. Ticking "Do not ask me again" and confirming sets `explorer.confirmDragAndDrop` to false.
  5. Setting `explorer.confirmDragAndDrop` set to false moves dropped items with no box, and it is on by default.
  6. Setting `explorer.enableDragAndDrop` chooses whether items can be dragged to move them inside the Explorer at all, and it is on by default.

#### DL-4.7 Confirm reordering workspace root folders

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Dragging a root folder in a multi-root workspace asks "Are you sure you want to change the order of root folder '{0}' in your workspace?" (or the multiple-folders wording) with a Move button and the same "Do not ask me again" checkbox.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.confirmDragAndDrop
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; C bundle (handleExplorerDrop, messages 7365 and 7367)
- Pieces:
  1. Dragging a root folder to a new place in a multi-root workspace asks "Are you sure you want to change the order of root folder '{0}' in your workspace?" with a "Move" button.
  2. Dragging several root folders at once asks "Are you sure you want to change the order of multiple root folders in your workspace?"
  3. Ticking "Do not ask me again" and confirming sets `explorer.confirmDragAndDrop` to false.
  4. With `explorer.confirmDragAndDrop` false the root folders change order with no box.

#### DL-4.8 Replace prompt when a move lands on an existing name

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a drag move would overwrite items, the box asks "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" (or lists "The following {0} files and/or folders already exist") with a Replace button and "This action is irreversible!", and no setting turns it off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; V\src\vs\workbench\contrib\files\browser\fileImportExport.ts; C bundle (doHandleExplorerDropOnMove conflict branch, messages 7119, 7120, 7214 to 7219)
- Pieces:
  1. When a drag move would land on a name that already exists, a box asks "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" with a "Replace" button.
  2. The box adds "This action is irreversible!"
  3. When several names clash, the box asks "The following {0} files and/or folders already exist in the destination folder. Do you want to replace them?" and lists the names.
  4. Pressing "Replace" moves the items over the existing ones, and cancelling leaves the existing items in place.
  5. The box has no "Do not ask me again" checkbox and no setting hides it.

#### DL-4.9 Replace prompt when copying or pasting with incremental naming off

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pasting or Ctrl-dragging a copy asks "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" only when `explorer.incrementalNaming` is set to disabled, and in the default setting the copy is renamed "name copy" with no box.
- Commands: filesExplorer.paste; filesExplorer.copy; filesExplorer.cut
- Menus: ExplorerContext
- Keys: Ctrl+V or Cmd+V (Explorer focused); Ctrl+C or Cmd+C (Explorer focused); Ctrl+X or Cmd+X (Explorer focused)
- Settings: explorer.incrementalNaming
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (findValidPasteFileTarget, askForOverwrite, filesExplorer.paste, explorer.incrementalNaming, messages 7316 and 7119)
- Pieces:
  1. With `explorer.incrementalNaming` set to `disabled`, pasting an Explorer item whose name already exists in the target folder asks "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" with a "Replace" button.
  2. With the same setting, dropping a Ctrl-drag copy (Alt on macOS) onto an existing name asks the same question.
  3. Setting `explorer.incrementalNaming` chooses how a pasted duplicate is named, with values `simple` (adds the word "copy", the default and no box), `smart` (adds a number or raises the one in the name) and `disabled` (keeps the name and asks to replace).
  4. The Explorer context menu entries Copy, Cut and Paste, and the keys Ctrl+C, Ctrl+X and Ctrl+V (Cmd on macOS) with the Explorer focused, start the copy, cut and paste.

#### DL-4.10 Replace prompt when dropping files from the OS

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Dropping files from the OS file manager onto the Explorer asks the same "already exists in the destination folder" replace question once for each name that clashes, and there is no setting for it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.autoOpenDroppedFile
- Source: V\src\vs\workbench\contrib\files\browser\fileImportExport.ts; V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; C bundle (ExternalFileImport class, messages 7119, 7201, 7202, explorer.autoOpenDroppedFile)
- Pieces:
  1. Dropping files from the OS file manager onto the Explorer asks, for each name that clashes, "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" with a "Replace" button.
  2. The box adds "This action is irreversible!" and has no setting that turns it off.
  3. Setting `explorer.autoOpenDroppedFile` chooses whether a file dropped into the Explorer opens in the editor, and it is on by default.

#### DL-4.11 Choose copy or add to workspace when dropping folders

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Dropping folders from the OS onto a workspace root asks "Do you want to copy the folders or add the folders to the workspace?" with Add Folders to Workspace, Copy Folders and Cancel, and it only offers Copy when no workspace folder shares the dropped folder's file system.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\files\browser\fileImportExport.ts; C bundle (ExternalFileImport.doImport, messages 7193 to 7200)
- Pieces:
  1. Dropping folders from the OS onto a workspace root folder asks "Do you want to copy the folders or add the folders to the workspace?" (for one folder "Do you want to copy '{0}' or add '{0}' as a folder to the workspace?") with "Add Folders to Workspace", "Copy Folders" and Cancel buttons.
  2. Pressing "Add Folders to Workspace" adds the dropped folders as workspace folders.
  3. Pressing "Copy Folders" copies the dropped folders into the Explorer.
  4. When no workspace folder shares the dropped folder's file system, the box asks only "Are you sure to want to copy folders?" (for one folder "Are you sure to want to copy '{0}'?") with "Copy Folders" and Cancel, and no Add button.
  5. Dropping folders onto an Explorer item that is not a workspace root copies them in with no box.

#### DL-4.12 Confirm pasting files from outside

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pasting files copied in the OS file manager into the Explorer asks "Are you sure you want to paste '{0}'?" (or "the following {0} items") with a Paste button and a "Do not ask me again" checkbox that turns `explorer.confirmPasteNative` off.
- Commands: filesExplorer.paste
- Menus: ExplorerContext
- Keys: Ctrl+V or Cmd+V (Explorer focused)
- Settings: explorer.confirmPasteNative
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (pasteFileHandler, messages 7140 to 7143, explorer.confirmPasteNative)
- Pieces:
  1. Pasting files copied in the OS file manager into the Explorer asks "Are you sure you want to paste '{0}'?" with a "Paste" button.
  2. Pasting several items asks "Are you sure you want to paste the following {0} items?" and lists their names.
  3. Ticking "Do not ask me again" and confirming sets `explorer.confirmPasteNative` to false.
  4. Setting `explorer.confirmPasteNative` set to false pastes with no box, and it is on by default.
  5. Pressing Ctrl+V or Cmd+V with the Explorer focused, or choosing Paste in the Explorer context menu, starts the paste.

#### DL-4.13 Confirm before undoing a file operation

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pressing undo with the Explorer focused asks "Would you like to undo '{0}'?" with Yes and No for steps that delete something (a copy or a new file), and `explorer.confirmUndo` set to verbose also asks for rename and move while light skips the box except for new files.
- Commands: undo; redo
- Menus: MenubarEditMenu
- Keys: Ctrl+Z (Explorer focused); Ctrl+Y or Ctrl+Shift+Z (Explorer focused, redo); Cmd+Z and Cmd+Shift+Z (macOS)
- Settings: explorer.confirmUndo; explorer.enableUndo
- Source: V\src\vs\platform\undoRedo\common\undoRedoService.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; V\src\vs\editor\browser\editorExtensions.ts; C bundle (explorer.confirmUndo, explorer.enableUndo, messages 2647 to 2649, 7294 to 7297)
- Pieces:
  1. With the Explorer focused, pressing Ctrl+Z (Cmd+Z on macOS) or choosing Edit > Undo asks "Would you like to undo '{0}'?" with Yes and No buttons when the last file step deleted something, such as a paste, a copy or a new file.
  2. Pressing Yes undoes the step and pressing No leaves it in place.
  3. Setting `explorer.confirmUndo` chooses which undo steps ask, with values `verbose` (also rename and move), `default` (steps that delete something) and `light` (no box except when undoing a new file).
  4. With the Explorer focused, Ctrl+Y or Ctrl+Shift+Z (Cmd+Shift+Z on macOS) redoes the undone file step.
  5. Setting `explorer.enableUndo` chooses whether the Explorer supports undoing file and folder steps at all, and it is on by default.

#### DL-4.14 Replace prompt when an untitled file is saved over a file that now exists

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Saving an untitled tab that already has a file path to that path, after a file was created there in the meantime, asks "'{0}' already exists. Do you want to replace it?" with a Replace button, while the OS Save box asks its own replace question for normal Save As.
- Commands: workbench.action.files.save
- Menus: none found
- Keys: Ctrl+S
- Settings: none found
- Source: V\src\vs\workbench\services\textfile\browser\textFileService.ts; V\src\vs\workbench\services\workingCopy\common\fileWorkingCopyManager.ts; C bundle (messages 13118, 13119, 13120, 13555 to 13557)
- Pieces:
  1. Saving an untitled tab that has its own file path with Ctrl+S, after a file was created at that path in the meantime, shows a warning "'{0}' already exists. Do you want to replace it?" with a "Replace" button.
  2. The box adds "A file or folder with the name '{0}' already exists in the folder '{1}'. Replacing it will overwrite its current contents."
  3. Pressing "Replace" saves over the existing file, and cancelling saves nothing.
  4. A normal Save As that picks an existing name is answered by the OS Save box's own replace question and does not show this box.

#### DL-4.15 Warn before opening a very large file

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Opening a file over the size limit (1 GB local, 10 MB remote, 50 MB web by default) shows "The file is not displayed in the text editor because it is very large ({0})." inside the editor with Open Anyway and Configure Limit buttons, and this is a message in the editor pane and not a pop-up.
- Commands: workbench.action.openLargeFile; workbench.action.configureEditorLargeFileConfirmation
- Menus: none found
- Keys: none found
- Settings: workbench.editorLargeFileConfirmation
- Source: V\src\vs\workbench\common\editor.ts; V\src\vs\workbench\common\editor\resourceEditorInput.ts; V\src\vs\workbench\contrib\files\browser\editors\textFileEditor.ts; V\src\vs\workbench\browser\parts\editor\editorConfiguration.ts; V\src\vs\platform\files\common\files.ts; C bundle (workbench.action.openLargeFile, workbench.editorLargeFileConfirmation, messages 7009, 7010, 4407)
- Pieces:
  1. Opening a file over the size limit shows, inside the editor pane, a warning "The file is not displayed in the text editor because it is very large ({0})." with the file size in place of {0}.
  2. Pressing "Open Anyway" opens the file in the editor with no size limit.
  3. Pressing "Configure Limit" opens User Settings filtered to `workbench.editorLargeFileConfirmation`.
  4. Setting `workbench.editorLargeFileConfirmation` sets the size in MB where the message shows, with a minimum of 1 and defaults of 1 GB for local files, 10 MB for remote files and 50 MB on the web.
  5. When the size is not known the message reads "The file is not displayed in the text editor because it is very large." with no size.


## DL-5 Workspace Trust and link safety prompts

(both). Boxes that ask whether you trust a folder, its files or an outside link, plus the entry to the trust editor. Cursor ships this off by default.

17 features.

#### DL-5.1 Startup trust prompt

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - same flow and strings, but trust is off by default so it never appears until trust is switched on; the No sub-line reads "Browse folder in restricted mode" (message 11863), and the product-supplied text for AI-generated workspaces is not used
- What: When Workspace Trust is on and a folder or workspace is not yet trusted, the window has focus and the startup setting allows it, a modal asks "Do you trust the authors of the files in this folder?" (workspace wording for a multi-root workspace) with a shield icon, the line "{product} provides features that may automatically execute files in this folder.", a docs line and the folder path. It has two buttons, "Yes, I trust the authors" (sub-line "Trust folder and enable all features") and "No, I don't trust the authors" (sub-line "Open folder in restricted mode" in stock). The X and Escape are disabled, so the person must pick one. It is skipped for already trusted folders, empty windows, virtual workspaces and when trust cannot be changed. Picking No shows the Restricted Mode notice and status item.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.enabled; security.workspace.trust.startupPrompt
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (WorkspaceTrustUXHandler: onDidInitiateWorkspaceTrustRequestOnStartup, doShowModal with disableCloseAction, showModalOnStart); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (requestWorkspaceTrustOnStartup, canSetWorkspaceTrust); C bundle (module workspaceTrust; class with doShowModal and showModalOnStart, messages 11856 to 11867, storage key workspace.trust.startupPrompt.shown)
- Pieces:
  1. When Workspace Trust is on and an untrusted folder opens, a modal with a shield icon asks "Do you trust the authors of the files in this folder?" and the person must answer before going on.
  2. The modal body says "{product} provides features that may automatically execute files in this folder.", then "If you don't trust the authors of these files, we recommend to continue in restricted mode as the files may be malicious." with a docs link, then the folder path.
  3. For a multi-root workspace the question, the body line and the button sub-lines use the word "workspace" in place of "folder".
  4. The first button reads "Yes, I trust the authors" with the sub-line "Trust folder and enable all features", and picking it marks the folder trusted.
  5. The second button reads "No, I don't trust the authors", and picking it keeps the window in Restricted Mode and shows the Restricted Mode notice and status item.
  6. The X and Escape are disabled, so one of the two buttons has to be picked.
  7. The modal is skipped for folders that are already trusted, for empty windows, for virtual workspaces and when trust cannot be changed.
  8. The second button's sub-line reads "Open folder in restricted mode". [stock only]
  9. The second button's sub-line reads "Browse folder in restricted mode". [cursor only]

#### DL-5.2 Trust the parent folder from the startup prompt

- Tag: both. Evidence: source. Boundary: no. Cursor: present - message 11858, canSetParentFolderTrust and setParentFolderTrust found in the bundle
- What: The startup prompt can carry a checkbox "Trust the authors of all files in the parent folder '{name}'". It shows only for a single folder on a local or remote file system whose folder has a parent, and not for AI-generated workspaces. Ticking it and answering Yes marks the parent folder trusted, so sibling folders under it are trusted too, instead of trusting only the one folder. The Workspace Trust editor has a matching Trust Parent button, which ED-8.18 covers.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (checkboxString, doShowModal run with checkboxChecked); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (canSetParentFolderTrust lines 522-539, setParentFolderTrust line 541); C bundle (module workspaceTrust; doShowModal calls setParentFolderTrust when the box is ticked, message 11858)
- Pieces:
  1. The startup modal can show a checkbox "Trust the authors of all files in the parent folder '{name}'" under the question.
  2. The checkbox shows only for a single folder on a local or remote file system whose folder has a parent, and not for AI-generated workspaces.
  3. Ticking the checkbox and picking "Yes, I trust the authors" marks the parent folder trusted, so folders next to it under the same parent are trusted too.
  4. Picking "Yes, I trust the authors" with the checkbox clear trusts only the one folder.

#### DL-5.3 Choose when the startup trust prompt appears

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - same three values and descriptions (messages 11821 to 11824), but the default is once instead of never
- What: A setting picks whether the startup prompt shows every time an untrusted workspace opens (always), only the first time (once, remembered per workspace), or never. Stock defaults to never as read in the 1.128.0 source; Cursor defaults to once.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.startupPrompt
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (registered setting lines 806-818, showModalOnStart); C bundle (module workspaceTrust; registered setting with `default:"once"`, enum always, once, never)
- Pieces:
  1. Setting `security.workspace.trust.startupPrompt` chooses when the startup modal shows, with values `always`, `once` and `never`.
  2. The value `always` asks every time an untrusted workspace opens, `once` asks only the first time and remembers per workspace, and `never` does not ask.
  3. The setting defaults to `never`. [stock only]
  4. The setting defaults to `once`. [cursor only]

#### DL-5.4 Open untrusted files prompt

- Tag: both. Evidence: source. Boundary: no. Cursor: present - messages 11836 to 11844 and 11829 to 11832, handleWorkspaceTrust and onDidInitiateOpenFilesTrustRequest found, default prompt
- What: In a trusted workspace or trusted empty window, opening a file from outside the trusted folders by File > Open File, dropping it on the editor area or a tab strip, or opening it from the OS or command line shows "Do you want to allow untrusted files in this workspace?" (window wording in an empty window). The shield box says "You are trying to open untrusted files in a workspace which is trusted." and recommends Restricted Mode in a new window. Buttons are Open, "Open in Restricted Mode" (opens the file in a new window) and Cancel, with a checkbox "Remember my decision for all workspaces" that writes open or newWindow to the setting. After an Open answer it does not ask again for that workspace. The setting also lets a person skip the prompt for good.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.untrustedFiles; security.workspace.trust.emptyWindow
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (WorkspaceTrustRequestHandler, onDidInitiateOpenFilesTrustRequest lines 125-165, setting lines 832-843); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (requestOpenFilesTrust, completeOpenFilesTrustRequest lines 700-775); V\src\vs\workbench\services\editor\browser\editorService.ts (handleWorkspaceTrust); callers of validateTrust: V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts, V\src\vs\workbench\electron-browser\window.ts, V\src\vs\workbench\browser\dnd.ts, V\src\vs\workbench\browser\parts\editor\editorDropTarget.ts, V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts; C bundle (module workspaceTrust; handleWorkspaceTrust, extractEditorResources, messages 11836 to 11844)
- Pieces:
  1. In a trusted workspace, opening a file from outside the trusted folders with File > Open File shows a shield box asking "Do you want to allow untrusted files in this workspace?" (reading "in this window" in a trusted empty window).
  2. The same box shows when an untrusted file is dropped on the editor area or a tab strip, or opened from the OS or the command line.
  3. The box body says "You are trying to open untrusted files in a workspace which is trusted." and recommends opening them in Restricted Mode in a new window, with a docs link.
  4. The buttons are "Open", "Open in Restricted Mode" (which opens the file in a new window) and Cancel.
  5. The checkbox "Remember my decision for all workspaces" writes `open` or `newWindow` to `security.workspace.trust.untrustedFiles` when a button is picked.
  6. After an "Open" answer the box does not ask again for that workspace.
  7. Setting `security.workspace.trust.untrustedFiles` chooses what happens, with values `prompt` (the default), `open` and `newWindow`, and the last two skip the box.

#### DL-5.5 Trust request when an action needs trust

- Tag: both. Evidence: source. Boundary: no. Cursor: present - messages 11845 to 11852, and the caller sentences found (5838, 5839, 9892, 11324, 8384, 10366, 12913, 12914, 12915)
- What: When a person starts something that runs workspace code in an untrusted folder, a shield box asks "Do you trust the authors of the files in this folder?" with the caller's own sentence, then "If you don't trust the authors of these files, we do not recommend continuing as the files may be malicious." Buttons are "Trust Folder & Continue" (workspace wording for multi-root), "Manage" (closes the box and opens the Workspace Trust editor) and Cancel. Callers in scope: Run and Debug ("Debugging executes build tasks and program code from your workspace."), listing or running tasks, running tests, running a notebook cell, creating a terminal, and saving a file as admin. The default sentence is "A feature you are trying to use may be a security risk if you do not trust the source of the files or folders you currently have open." If trust is refused the action does not run (the terminal tab shows its own message, covered by PN-2.9). Never shown while trust is off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.enabled
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (onDidInitiateWorkspaceTrustRequest lines 203-278); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (requestWorkspaceTrust line 856); V\src\vs\workbench\contrib\debug\browser\debugService.ts; V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts; V\src\vs\workbench\contrib\testing\common\testServiceImpl.ts; V\src\vs\workbench\contrib\notebook\browser\services\notebookExecutionServiceImpl.ts; V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts; V\src\vs\workbench\services\files\electron-browser\elevatedFileService.ts; C bundle (module workspaceTrust; onDidInitiateWorkspaceTrustRequest handler with ContinueWithTrust, Manage and Cancel)
- Pieces:
  1. When something that runs workspace code starts in an untrusted folder, a shield box asks "Do you trust the authors of the files in this folder?" (workspace wording for multi-root) with a sentence from the caller, then "If you don't trust the authors of these files, we do not recommend continuing as the files may be malicious." and a docs link.
  2. The buttons are "Trust Folder & Continue" ("Trust Workspace & Continue" for multi-root), "Manage" and Cancel.
  3. Pressing "Trust Folder & Continue" marks the folder trusted and the action goes on.
  4. Pressing "Manage" closes the box and opens the Workspace Trust editor.
  5. Pressing Cancel closes the box and the action does not run.
  6. When the caller gives no sentence, the box says "A feature you are trying to use may be a security risk if you do not trust the source of the files or folders you currently have open."
  7. Starting a debug session shows the sentence "Debugging executes build tasks and program code from your workspace."
  8. Listing or running tasks, running tests, running a notebook cell, creating a terminal and saving a file as admin each show this box with their own sentence.
  9. The box never shows while `security.workspace.trust.enabled` is off.

#### DL-5.6 Confirm trust when adding an untrusted folder to a trusted workspace

- Tag: both. Evidence: source. Boundary: no. Cursor: present - messages 11853 to 11855, the onWillChangeWorkspaceFolders handler with setUrisTrust found
- What: When Workspace Trust is on and a folder that is not trusted is added to a trusted workspace (for example Add Folder to Workspace), a shield box asks "Do you trust the authors of the files in this folder?" with the detail "You are adding files that are not currently trusted to a trusted workspace. Do you trust the authors of these new files?" and Yes and No buttons. The answer marks the added folders trusted or not trusted.
- Commands: workbench.action.addRootFolder
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.enabled
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (WorkspaceTrustUXHandler.registerListeners, onWillChangeWorkspaceFolders lines 336-372); V\src\vs\workbench\browser\actions\workspaceActions.ts (addRootFolder, route in only); C bundle (module workspaceTrust; handler calling dialogService.confirm then setUrisTrust, messages 11853 to 11855)
- Pieces:
  1. With trust on, adding a folder that is not trusted to a trusted workspace, such as with File > Add Folder to Workspace..., shows a shield box asking "Do you trust the authors of the files in this folder?" before the folder is added.
  2. The box says "You are adding files that are not currently trusted to a trusted workspace. Do you trust the authors of these new files?"
  3. The buttons are Yes and No, where Yes marks the added folders trusted and No marks them not trusted, and the folders are added either way.
  4. The box does not show when every added folder is already trusted, when the workspace itself is not trusted, or when Workspace Trust is off.

#### DL-5.7 Turn Workspace Trust on or off

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - default false, enterprise policy name WorkspaceTrustEnabled (minimum version 1.99), the description still says "within VS Code" (messages 11819, 11820), and the team admin value workspaceTrustControls.enabled writes the setting
- What: One application setting switches the whole feature. When it is off every folder counts as trusted, so none of the prompts, the Restricted Mode notice or status item, or the two trust commands appear (the palette entries are hidden while it is off). Stock ships it on. Cursor ships it off, and a team admin flag can turn it on: when the admin value is true and no enterprise policy sets it, Cursor writes the setting true at user level (it never turns it off again). A hidden command line flag also forces it off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.enabled
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (setting lines 799-805); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (WorkspaceTrustEnablementService, calculateWorkspaceTrust line 295 returns true when off); V\src\vs\platform\environment\node\argv.ts (hidden option disable-workspace-trust); C bundle (module workspaceTrust; registered setting with `default:!1` and policy WorkspaceTrustEnabled; team admin settings code reading `workspaceTrustControls?.enabled` and calling updateValue with the user target; getter disableWorkspaceTrust)
- Pieces:
  1. Setting `security.workspace.trust.enabled` switches Workspace Trust on or off for the whole app.
  2. With it off every folder counts as trusted, so none of the trust prompts, the Restricted Mode notice or the Restricted Mode status item appear.
  3. With it off, the Command Palette entries "Workspaces: Manage Workspace Trust" and "Workspaces: Configure Workspace Trust Settings" are hidden.
  4. A hidden command line option `disable-workspace-trust` forces Workspace Trust off.
  5. The setting ships on. [stock only]
  6. The setting ships off. [cursor only]
  7. A team admin flag set to true writes the setting true at user level, and it never turns the setting off again. [cursor only]

#### DL-5.8 Trust the empty window by default

- Tag: both. Evidence: source. Boundary: no. Cursor: present - default true, message 11833
- What: A setting decides whether a window with no folder open is trusted without asking (default on). Together with the untrusted files setting it lets a person open any file in an empty window without the DL-5.4 prompt. When it is off the empty window starts in Restricted Mode.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.emptyWindow; security.workspace.trust.untrustedFiles
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (setting lines 845-851); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (line 324, isEmptyWorkspace branch of calculateWorkspaceTrust); C bundle (module workspaceTrust; registered setting `[Cda]:{type:"boolean",default:!0`)
- Pieces:
  1. Setting `security.workspace.trust.emptyWindow` chooses whether a window with no folder open counts as trusted without asking, and it is on by default.
  2. With the setting on and `security.workspace.trust.untrustedFiles` set to `open`, any file opens in an empty window with no untrusted files box.
  3. With the setting off, an empty window starts in Restricted Mode.

#### DL-5.9 Reach Manage Workspace Trust from prompts, links and the palette

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - command and label found (message 11836, category Workspaces 11834), seen in 7 places in the bundle, and the Command Palette entry is hidden by default because trust is off; the walkthrough link text ("enable trust") was not found in the message table
- What: The entry to the Workspace Trust editor (the editor body is ED-8.18). Routes in scope: Command Palette "Workspaces: Manage Workspace Trust"; the Manage button in the trust request box (DL-5.5); the Manage link in the Restricted Mode notice (BN-2.1); the "Restricted Mode" status item (SS-4.5); a "Manage Workspace Trust" button on the "Workspace Trust Required" placeholder shown for a file in an untrusted folder; a "Manage Workspace Trust" quick fix on untrusted settings in the settings JSON; the hover on the "Requires workspace trust" setting label (ED-8.6); and a link in the empty Source Control view. Stock also has an "enable trust" link in the Beginner walkthrough, which is not in Cursor. The command opens the editor pinned and is hidden while trust is off.
- Commands: workbench.trust.manage
- Menus: CommandPalette
- Keys: none found
- Settings: security.workspace.trust.enabled
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (action lines 769-789, precondition isWorkspaceTrustEnabled and the setting, notice links); V\src\vs\workbench\contrib\workspace\common\workspace.ts (MANAGE_TRUST_COMMAND_ID); V\src\vs\platform\actions\common\actions.ts (f1 entry uses the precondition as its when); V\src\vs\workbench\browser\parts\editor\editorPlaceholder.ts (WorkspaceTrustRequiredPlaceholderEditor); V\src\vs\workbench\contrib\preferences\browser\preferencesRenderers.ts (generateUntrustedSettingCodeActions); V\src\vs\workbench\contrib\preferences\browser\settingsEditorSettingIndicators.ts; V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (welcome content); V\src\vs\workbench\contrib\welcomeGettingStarted\common\gettingStartedContent.ts; C bundle (module workspaceTrust; `workbench.trust.manage` 7 hits, messages 3751, 8663, 8664, 8699, 9296, 11836, 11868)
- Pieces:
  1. Running "Workspaces: Manage Workspace Trust" (`workbench.trust.manage`) from the Command Palette opens the Workspace Trust editor as a pinned tab.
  2. The command is hidden from the Command Palette while `security.workspace.trust.enabled` is off.
  3. The "Manage" button in the trust request box, the Manage link in the Restricted Mode notice and the "Restricted Mode" status item each open the same editor.
  4. A file in an untrusted folder shows a "Workspace Trust Required" placeholder with a "Manage Workspace Trust" button that opens the editor.
  5. A setting that needs trust gets a "Manage Workspace Trust" quick fix in the settings JSON, and the hover on its "Requires workspace trust" label in the Settings editor links to the editor.
  6. The empty Source Control view has a link that opens the editor.
  7. The Beginner walkthrough has an "enable trust" link that opens the Workspace Trust editor to the side. [stock only]

#### DL-5.10 Configure Workspace Trust Settings

- Tag: both. Evidence: source. Boundary: no. Cursor: present - message 11835, `workbench.trust.configure` seen twice, opens settings with the workspaceTrust tag query
- What: The Command Palette entry "Workspaces: Configure Workspace Trust Settings" opens User Settings filtered to the trust settings tag, which lists the five security.workspace.trust.* settings. The Workspace Trust editor header also links to it ("Configure your settings"). Hidden while trust is off.
- Commands: workbench.trust.configure
- Menus: CommandPalette
- Keys: none found
- Settings: security.workspace.trust.enabled
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (action lines 748-766); V\src\vs\workbench\contrib\workspace\browser\workspaceTrustEditor.ts (line 875 header link); C bundle (module workspaceTrust; action with openUserSettings and the tag query, message 11835)
- Pieces:
  1. Running "Workspaces: Configure Workspace Trust Settings" (`workbench.trust.configure`) from the Command Palette opens User Settings filtered to the trust settings tag, which lists the five `security.workspace.trust.*` settings.
  2. The "Configure your settings" link in the Workspace Trust editor header runs the same command.
  3. The command is hidden from the Command Palette while `security.workspace.trust.enabled` is off.

#### DL-5.11 External link confirmation

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - same box, strings and buttons (messages 11521 to 11525), and the product name reads "Cursor"; the default trusted list is Cursor's own (cursor.com, github.com, MCP vendor hosts and others in product.json); the box can show over a hidden window when the click came from a person; in the Agents window the Open button can route to its browser (out of scope); the validator is not registered in the Agents window; a third button "Always Open This Domain" (message 11524) exists in code but its switch useSimpleTrustAction is set false with no setter found
- What: Opening an http or https link whose domain is not trusted shows "Do you want {product} to open the external website?" with the link as detail (long query or fragment text is shortened in the middle to about 60 characters). Buttons are Open, Copy (copies the link and does not open it), "Configure Trusted Domains" (runs the DL-5.12 picker, then opens the link if the picked rule covers it) and Cancel. Links that come from workspace content (editor links, webviews, notebook outputs) skip the box in a trusted workspace unless DL-5.14 is on. Localhost and the product's default trusted list never ask.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.trustedDomains.promptInTrustedWorkspace
- Source: V\src\vs\workbench\contrib\url\browser\trustedDomainsValidator.ts (OpenerValidatorContributions.validateLink lines 45-135); V\src\vs\platform\url\common\trustedDomains.ts (isURLDomainTrusted, localhost rule); V\src\vs\workbench\contrib\url\browser\trustedDomainService.ts (isValid); V\src\vs\editor\contrib\links\browser\links.ts (fromWorkspace link opens); C bundle (module trustedDomains; class with validateLink, _buildTrustButton, useSimpleTrustAction; registered with disableInGlass; messages 11521 to 11525); C\product.json (linkProtectionTrustedDomains)
- Pieces:
  1. Opening an http or https link whose domain is not trusted shows a box asking "Do you want {product} to open the external website?" with the link as the detail.
  2. A long link has its query or fragment text shortened in the middle so the detail stays near 60 characters.
  3. Pressing "Open" opens the link, and pressing Cancel closes the box without opening it.
  4. Pressing "Copy" copies the link to the clipboard and does not open it.
  5. Pressing "Configure Trusted Domains" shows the domain picker, and the link opens afterward if the picked rule covers it.
  6. Links that come from workspace content, such as editor links, webviews and notebook outputs, open with no box in a trusted workspace.
  7. Localhost links and links to the product's default trusted domains open with no box.
  8. The box names the product "Cursor", and the default trusted domains are Cursor's own list, including cursor.com and github.com. [cursor only]

#### DL-5.12 Pick a domain to trust from the link prompt

- Tag: both. Evidence: source. Boundary: no. Cursor: present - messages 11515 to 11520
- What: After "Configure Trusted Domains" a quick pick lists "Trust {scheme://host}" (preselected), "Trust {domain} and all its subdomains" (or "Trust {ip} on all ports" for an IP with a port), "Trust all domains (disables link protection)" and "Manage Trusted Domains". A trust choice is saved to the user's trusted domain list right away and the link opens if it is now covered; Manage opens the DL-5.13 file at the link.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\url\browser\trustedDomains.ts (configureOpenerTrustedDomainsHandler lines 72-163); C bundle (module trustedDomains; function that builds the same picks, messages 11515 to 11519)
- Pieces:
  1. After "Configure Trusted Domains", a quick pick opens with its first entry "Trust {scheme://host}" already selected.
  2. The next entry reads "Trust {domain} and all its subdomains" for a host name, or "Trust {ip} on all ports" for an IP address that has a port, and an IP address with no port gets no such entry.
  3. The entry "Trust all domains (disables link protection)" follows.
  4. The last entry, "Manage Trusted Domains", opens the Trusted Domains file for that link and does not open the link.
  5. Picking a trust entry saves it to the person's trusted domain list right away, and the link opens if the new rule covers it.

#### DL-5.13 Manage Trusted Domains

- Tag: both. Evidence: source. Boundary: no. Cursor: present - same command, label (messages 11519, 11520), file URI `trustedDomains:/Trusted Domains` and help comments found; the default trusted list in the comments is Cursor's
- What: The Command Palette entry "Manage Trusted Domains" opens an editor tab named "Trusted Domains", a JSON with comments file (virtual, scheme trustedDomains). The top comments explain the rule forms (exact host, port, wildcard subdomain, path prefix, IP prefix, `*`) and list the product's default trusted domains, then a JSON array holds the person's own entries. Saving the file applies the rules. When opened from the link prompt it adds "Currently configuring trust for {link}" and puts the cursor in the list.
- Commands: workbench.action.manageTrustedDomain
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\url\browser\trustedDomains.ts (manageTrustedDomainSettingsCommand line 62, openInEditor); V\src\vs\workbench\contrib\url\browser\trustedDomainsFileSystemProvider.ts (help text, computeTrustedDomainContent, writeFile); V\src\vs\workbench\contrib\url\browser\url.contribution.ts (CommandPalette item lines 60-68); C bundle (module trustedDomains; command id workbench.action.manageTrustedDomain, `T8o=Re.parse("trustedDomains:/Trusted Domains")`, help comment text)
- Pieces:
  1. Running "Manage Trusted Domains" (`workbench.action.manageTrustedDomain`) from the Command Palette opens an editor tab named "Trusted Domains" holding a JSON with comments file.
  2. The comments at the top explain the rule forms (exact host, port, wildcard subdomain, path prefix, IP prefix and `*`) and list the product's default trusted domains.
  3. Below the comments a JSON array holds the person's own entries.
  4. Saving the file applies the rules it holds.
  5. When the file is opened from the link picker, it adds "Currently configuring trust for {link}" and puts the cursor in the list.
  6. The default trusted domains listed in the comments are Cursor's own list. [cursor only]

#### DL-5.14 Ask about links in trusted workspace content

- Tag: both. Evidence: source. Boundary: no. Cursor: present - setting registered with default false (message 11527), the validator reads it
- What: A setting makes the DL-5.11 box also appear for links that come from workspace content (editor links, webviews, notebook outputs) in a trusted workspace. It is off by default. Because Cursor ships trust off, every workspace counts as trusted there, so such links skip the box unless this setting is turned on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.trustedDomains.promptInTrustedWorkspace
- Source: V\src\vs\workbench\contrib\url\browser\url.contribution.ts (setting lines 88-98); V\src\vs\workbench\contrib\url\browser\trustedDomainsValidator.ts (line 50); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (trusted when the feature is off); C bundle (module trustedDomains; `promptInTrustedWorkspace":{scope:1,type:"boolean",default:!1`)
- Pieces:
  1. Setting `workbench.trustedDomains.promptInTrustedWorkspace` turned on makes the link box also show for links from workspace content, such as editor links, webviews and notebook outputs, in a trusted workspace.
  2. The setting is off by default, so those links open with no box.
  3. Trust is off by default, so every workspace counts as trusted and such links skip the box unless this setting is turned on. [cursor only]

#### DL-5.15 Confirm when another app opens a path in the app

- Tag: both. Evidence: source. Boundary: no. Cursor: present - both settings, the handler table in out\main.js and the channel that writes the settings back were found, and nls 146 to 153 hold the box texts with the product name Cursor filled in
- What: When another application asks the app to open a local or remote file, folder or workspace through its link handler, a warning box asks first, and a checkbox on it turns that question off for good.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.promptForLocalFileProtocolHandling; security.promptForRemoteFileProtocolHandling
- Source: V\src\vs\code\electron-main\app.ts (lines 156-157 and 835-893); V\src\vs\workbench\electron-browser\window.ts (lines 366-373); V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 382-397); C out\main.js (SECURITY_PROTOCOL_HANDLING_CONFIRMATION_SETTING_KEY with both ids, channel vscode:disablePromptForProtocolHandling); C bundle (the window listener for that channel and both settings); NLS 146 to 153
- Pieces:
  1. When another application opens a link that points at a local file, folder or workspace file, a warning box says "An external application wants to open '{path}' in {product}" and asks whether to open that file, folder or workspace file, with Yes and No buttons.
  2. The box detail warns "If you did not initiate this request, it may represent an attempted attack on your system", and No stops the open.
  3. Choosing Yes with the box "Allow opening local paths without asking" ticked turns `security.promptForLocalFileProtocolHandling` off in the user settings.
  4. Links to a remote path show the same box with the checkbox "Allow opening remote paths without asking", which turns `security.promptForRemoteFileProtocolHandling` off.
  5. Both settings are on by default and apply to the whole application, and with one turned off the matching kind of link opens with no question.
  6. Only local and remote links are asked about in this box, because links for an extension are confirmed separately inside the window.

#### DL-5.16 Windows network share host prompt

- Tag: both. Evidence: source. Boundary: no. Cursor: present - security.restrictUNCAccess and security.allowedUNCHosts are registered, out\main.js has onUNCHostNotAllowed and the not-allowed error, and nls 2736 to 2741 hold the box texts; security.allowedUNCHosts is not in the leftover list and was added here as a related id
- What: On Windows, opening a path on a network share whose host is not on the allowed list stops with a warning box that lets the person allow the host once or for good.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.restrictUNCAccess; security.allowedUNCHosts
- Source: V\src\vs\platform\windows\electron-main\windowsMainService.ts (lines 1239-1300); V\src\vs\workbench\electron-browser\window.ts (lines 342-366); V\src\vs\workbench\common\configuration.ts (lines 180-235); V\src\vs\base\common\errorMessage.ts (line 31); V\src\vs\code\electron-main\app.ts (lines 447-450); C out\main.js (onUNCHostNotAllowed, vscode:configureAllowedUNCHost, ERR_UNC_HOST_NOT_ALLOWED); C bundle (security.allowedUNCHosts and the restrictUNCAccess registration); NLS 2736 to 2741
- Pieces:
  1. On Windows, opening a path whose network share host is not allowed shows a warning box "The host '{host}' was not found in the list of allowed hosts. Do you want to allow it anyway?" with Allow, Cancel and Learn More buttons.
  2. The box detail names the path and says "Unless you trust the host, you should press 'Cancel'", and Cancel stops the open.
  3. Allow lets the open go ahead, and ticking "Permanently allow host '{host}'" also adds the host to `security.allowedUNCHosts` in the user settings.
  4. Learn More opens a web page about the setting and then shows the box again until a choice is made.
  5. Setting `security.restrictUNCAccess` (on by default, listed only on Windows) limits access to hosts on the allowed list or confirmed in the box, and turning it off removes the check.
  6. Setting `security.allowedUNCHosts` is a list of host names without backslashes (for example `my-server` or `192.168.0.1`), the Settings editor flags a name with a backslash, and a restart is needed after changing it.
  7. A file operation against a host that is not allowed fails with an error that ends "Please update the 'security.allowedUNCHosts' setting if you want to allow this host."
  8. Changing `security.restrictUNCAccess` raises the restart prompt described in DL-1.38.

#### DL-5.17 Choose the browser that opens external links

- Tag: both. Evidence: source. Boundary: no. Cursor: present - the setting with its restricted flag and the opener code that reads it are in the bundle
- What: The person sets `workbench.externalBrowser` to a browser name or the path of its program, and web links that leave the app open in that browser instead of the system default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.externalBrowser
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (lines 34-39); V\src\vs\platform\native\electron-main\nativeHostMainService.ts (openExternal, lines 658-671; openExternalBrowser, lines 673-712); V\src\vs\workbench\electron-browser\window.ts (setupOpenHandlers, lines 872-886); C bundle (setting `workbench.externalBrowser` with restricted true; setupOpenHandlers reading it)
- Pieces:
  1. Setting `workbench.externalBrowser` is empty by default, and then http and https links that leave the app open in the system default browser.
  2. Setting it to a browser name such as `edge`, `chrome` or `firefox` opens those links in that browser.
  3. Setting it to the absolute path of a browser program opens those links with that program.
  4. If a path value points at nothing, the problem is only written to the log and the system default browser opens the link.
  5. If the chosen browser fails to start, the link opens in the system default browser instead.
  6. Only http and https links use the setting, and other kinds of links go to the system's own handler for them.
  7. The setting is marked restricted, so a value set by an untrusted workspace is not applied, and it is offered only in the desktop app.


## DL-6 Cursor plan and account modals

(cursor) [boundary]. Cursor's own pop-ups for plan, pricing, upgrade, invite and privacy mode. They open from Cursor Settings and the chat queue.

15 features.

#### DL-6.1 Subscription tiers modal

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only - command title "Show Subscription Tiers Modal" (not localized, listed in F1); also opened by the Upgrade link on the empty editor landing screen (ED-9)
- What: A centered modal (900 px wide, up to 90 percent of the window) titled "Upgrade Your Plan" with the line "Choose the plan that works best for you". It shows plan cards side by side: Pro ($20/month, "Great for individual developers"), Pro+ ($60/month, "More power for demanding projects") and Ultra ($200/month, "Maximum power and flexibility"), each with a short tick list of features. Which cards show depends on the current plan (a Free account sees all three, a Pro account sees Pro+ and Ultra). One card is marked "Recommended", and trial accounts see "N Days Left" chips. Clicking a card selects it, and the primary button reads "Upgrade to {plan}" (or "Start {plan} now" for a trial of that plan) with Cancel beside it. Pressing the button runs the upgrade flow in DL-6.2 (confirmation if a card is on file, otherwise the web checkout in the browser) and closes the modal. The code also has a "Free N Day Trial" chip and a "Start free trial" label, but the switch that turns them on is fixed off.
- Commands: cursor.showSubscriptionTiersModal
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (module uiOverlayService, method showSubscriptionTiersModal and observable subscriptionTiersModal; component KVw with the plan list vBr; command registration with f1 true; landing screen click handler runs the command)
- Pieces:
  1. Running "Show Subscription Tiers Modal" (`cursor.showSubscriptionTiersModal`) from the Command Palette, or pressing the Upgrade link on the empty editor landing screen, opens a centered modal titled "Upgrade Your Plan" with the line "Choose the plan that works best for you".
  2. The modal shows plan cards side by side for Pro ($20/month, "Great for individual developers"), Pro+ ($60/month, "More power for demanding projects") and Ultra ($200/month, "Maximum power and flexibility"), each with a short tick list of features.
  3. Which cards show depends on the current plan, so a Free account sees all three and a Pro account sees Pro+ and Ultra.
  4. One card carries a "Recommended" marker, and trial accounts see "N Days Left" chips.
  5. Clicking a card selects it.
  6. The primary button reads "Upgrade to {plan}", or "Start {plan} now" for a trial of that plan, with a Cancel button beside it.
  7. Pressing the primary button runs the upgrade flow (the confirmation modal if a card is on file, otherwise the web checkout in the browser) and closes the modal, while pressing Cancel only closes it.

#### DL-6.2 Confirm a plan upgrade

- Tag: cursor. Evidence: bundle. Boundary: yes - the Free plan "Upgrade to Pro" row in Cursor Settings General opens it; it is also reached from the title bar Upgrade button, the tiers modal and chat error buttons. Cursor: cursor only
- What: When an upgrade starts and the account has a saved payment method, a 400 px modal titled "Confirm Upgrade to Pro" (or Pro+, Ultra) shows a one-line pitch for the plan and a warning-colored charge line: "Your saved payment method will be charged $20/month." (or $60, $200), or on a paid plan "You will be charged $60/month, and receive a prorated refund for the remaining days in your current plan." Buttons are Cancel and a rocket "Upgrade to {plan}". It cannot be closed while the upgrade runs. Confirming asks the server to upgrade at once and opens the invoice page in the browser when one comes back. With no saved payment method, or on any error, the web checkout page opens instead and no modal shows. Cancel, the X and a failed instant upgrade all count as "not done", and the flow code then opens the web checkout page too (read from code, not run). A second form takes its title, message and button label from the server after a dashboard action asks for confirmation.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (module upgradeUtils, helper eAt; module uiOverlayService, methods showUpgradeConfirmationModal and showServerUpgradeConfirmationModal; component vqw with the per-plan text; helper tAt for the server form; callers: the Free upgrade row component Ikw in Cursor Settings General, the title bar Upgrade click handler, KVw)
- Pieces:
  1. When the account has a saved payment method, a modal titled "Confirm Upgrade to Pro" (or Pro+, Ultra) opens with a one-line pitch for the plan.
  2. The confirmation modal opens when an upgrade starts from the "Upgrade to Pro" row in Cursor Settings General, from the title bar Upgrade button, or from the primary button in the subscription tiers modal.
  3. A warning-colored line reads "Your saved payment method will be charged $20/month." (or $60 or $200 for the other plans).
  4. On a paid plan the line reads "You will be charged $60/month, and receive a prorated refund for the remaining days in your current plan."
  5. The buttons are Cancel and a rocket button "Upgrade to {plan}".
  6. Pressing the upgrade button asks the server to upgrade at once, the modal cannot be closed while it runs, and the invoice page opens in the browser when one comes back.
  7. With no saved payment method, or on any error, the web checkout page opens in the browser and no modal shows.
  8. Pressing Cancel or the X, or a failed instant upgrade, counts as not done, and the flow then opens the web checkout page too.
  9. After a dashboard action asks for confirmation, a second form of the modal takes its title, message and button label from the server.

#### DL-6.3 Usage pricing modal

- Tag: cursor. Evidence: bundle. Boundary: yes - it opens from the "Skip the wait." link in the chat queue line ("Slow request ... (n) Skip the wait."), and the rows inside it save account billing settings. Cursor: cursor only - command title "Show Usage Pricing Modal" (F1)
- What: A small centered modal (300 px) titled "Configure Usage-Based Pricing" with the line "Skip the wait. Pay only for what you use." and a close X opens from the Command Palette as "Show Usage Pricing Modal" with the controls in DL-6.4 and DL-6.5 and a closing line "You can see your usage and change your hard limit in the account settings" whose last words open the web account settings page, and when it opens from a link outside the editor workbench it also shows one extra sentence from a server price check.
- Commands: cursorai.action.showUsagePricingModal
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (module uiOverlayService, method showUsageBasedPricingModal with the argument justshow or usage details; component Nqw; detail component Pqw; queue line template with "Skip the wait."; command registration with f1 true)
- Pieces:
  1. A small centered modal titled "Configure Usage-Based Pricing" opens with the line "Skip the wait. Pay only for what you use." and a close X.
  2. When the modal opens from a link outside the editor workbench, it also shows one extra sentence from a server price check.
  3. Running "Show Usage Pricing Modal" (`cursorai.action.showUsagePricingModal`) from the Command Palette shows only the usage controls and a closing line "You can see your usage and change your hard limit in the account settings", whose last words open the web account settings page.

#### DL-6.4 Turn usage-based pricing on or off

- Tag: cursor. Evidence: bundle. Boundary: yes - lives in the usage pricing modal, which opens from the chat queue link. Cursor: cursor only
- What: Inside the usage pricing modal a switch "Enable usage-based pricing" is on when a spend limit exists. Turning it on sets the limit to $50, and turning it off sets it to $0 (no usage-based). When it is on, a second switch "Enable usage-based pricing for premium models" appears. Both dim and show a loading state while they save, and a server error shows the server's title and detail in red.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component Lqw, switch component LSg, calls setHardLimit and setUsageBasedPremiumRequests on the dashboard client)
- Pieces:
  1. Inside the usage pricing modal a switch "Enable usage-based pricing" is on when a spend limit exists.
  2. Turning the switch on sets the limit to $50, and turning it off sets the limit to $0, which means no usage-based pricing.
  3. While it is on, a second switch "Enable usage-based pricing for premium models" appears.
  4. Both switches dim and show a loading state while a change saves.
  5. A server error shows the server's title and detail in red.

#### DL-6.5 Set the usage hard limit

- Tag: cursor. Evidence: bundle. Boundary: yes - lives in the usage pricing modal, which opens from the chat queue link. Cursor: cursor only
- What: A "Hard limit" text box (placeholder "Enter usage-based spend limit") takes a dollar amount, with the dollar sign added for the person. It saves on Enter, when the box loses focus with a changed value, or with the small button beside it, which reads "Save", "Saving..." and then "Saved" with a check. An empty box becomes $0 (usage-based off), a limit at or above the cap shows "Unlimited", and a bad entry shows "Please enter a valid number". The box is dimmed and locked while usage-based pricing is off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component Lqw, input handlers for keydown, blur and input, template Cqw with the Hard limit label, calls getHardLimit and setHardLimit)
- Pieces:
  1. A "Hard limit" text box with the placeholder "Enter usage-based spend limit" takes a dollar amount, and the dollar sign is added for the person.
  2. The value saves when Enter is pressed, when the box loses focus with a changed value, or when the small button beside the box is pressed.
  3. The small button reads "Save", then "Saving..." while it works, then "Saved" with a check.
  4. An empty box becomes $0, which turns usage-based pricing off, and a limit at or above the cap shows "Unlimited".
  5. A bad entry shows "Please enter a valid number".
  6. The box is dimmed and locked while usage-based pricing is off.

#### DL-6.6 Free trial ended modal

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: When a subscription refresh shows the account is on a trial plan with no days left, a 500 px modal appears with the heading "Your Free Trial Has Ended", the line "Thank you for trying Cursor! To continue enjoying premium features like:", a list (Unlimited Tab completions, Extended Agent usage, Cloud Agents, And more...) and two buttons: a primary "Upgrade to Pro" (or "Upgrade to Pro+" after a Pro+ trial), which opens the web checkout in the browser, and a "Maybe Later" button that closes it. It also closes by itself if the account moves off the Free plan.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component mqw in the UI overlay list Bqw; subscription changed listener with getDaysRemainingOnTrial; button calls cursorAuthenticationService.pay)
- Pieces:
  1. When a subscription refresh shows the account is on a trial plan with no days left, a modal appears with the heading "Your Free Trial Has Ended" and the line "Thank you for trying Cursor! To continue enjoying premium features like:".
  2. The modal lists "Unlimited Tab completions", "Extended Agent usage", "Cloud Agents" and "And more...".
  3. The primary button reads "Upgrade to Pro" (or "Upgrade to Pro+" after a Pro+ trial) and opens the web checkout in the browser.
  4. The "Maybe Later" button closes the modal.
  5. The modal closes by itself if the account moves off the Free plan.

#### DL-6.7 Free autocomplete used up popup

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: When Cursor Tab is on but the server says Tab needs a paid plan and the account is not on one, a dimmed full-window backing shows a card with a red X icon, "You've used up your free autocomplete requests." and a paragraph that unlimited Cursor Tab needs a paid plan and that Cursor Pro also has much higher premium model limits. Buttons are "Turn off autocomplete" (closes the popup; I saw no setting change in its handler) and a rocket "Upgrade to keep" (opens the web checkout in the browser). A small X in the corner also closes it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component Fqw with template Aqw; cppService.showingCppUpsell set by the server config update when cppEnabled is true and shouldLetUserEnableCppEvenIfNotPro is false; the Upgrade button calls pay)
- Pieces:
  1. When Cursor Tab is on but the server says Tab needs a paid plan and the account is not on one, a dimmed full-window backing shows a card with a red X icon and the heading "You've used up your free autocomplete requests."
  2. The card's paragraph says that unlimited Cursor Tab needs a paid plan and that Cursor Pro also has much higher premium model limits.
  3. The rocket button "Upgrade to keep" opens the web checkout in the browser.
  4. The "Turn off autocomplete" button closes the popup.
  5. A small X in the corner also closes the popup.

#### DL-6.8 Open the Invite Team Members modal

- Tag: cursor. Evidence: bundle. Boundary: yes - the only way in is the Invite button on the General tab of Cursor Settings. Cursor: cursor only
- What: On the General tab of Cursor Settings, a team account whose person is allowed to invite (membership type Enterprise, which covers team plans) sees an "Invite Team Members" row ("Accelerate development, shared knowledge, and context across your team") with an "Invite" button and a link icon. It opens a modal over a dimmed backdrop, and a click on the backdrop closes it. The modal is not drawn in the Agents window and has no command or menu entry.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (account section component Yww with the row condition for membership ENTERPRISE and viewerCanInviteMembers; the landing screen plan label shows "Enterprise" or "Team" for that same membership type, which is why I say it covers team plans; module uiOverlayService, methods showInviteModal and closeInviteModal; overlay component Oqw)
- Pieces:
  1. On the General tab of Cursor Settings, a team account whose person is allowed to invite sees an "Invite Team Members" row with the line "Accelerate development, shared knowledge, and context across your team", an "Invite" button and a link icon.
  2. Pressing "Invite" opens the invite modal over a dimmed backdrop.
  3. Clicking the backdrop closes the modal.
  4. The modal has no command or menu entry and is not drawn in the Agents window.

#### DL-6.9 Copy the team invite link

- Tag: cursor. Evidence: bundle. Boundary: yes - lives in the invite modal, which opens from Cursor Settings. Cursor: cursor only
- What: In the invite modal, the title reads "Invite team members" with a close icon. Under "Invite link" a read-only box shows the team's invite link ("Loading..." until it arrives) and a "Copy link" button copies it, with the icon changing to a check for about two seconds. The button stays disabled until the link has loaded.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component HVw, template $Vw, calls getTeamInviteLink on the dashboard client, clipboardService.writeText)
- Pieces:
  1. The invite modal has the title "Invite team members" with a close icon.
  2. Under "Invite link" a read-only box shows the team's invite link, and it reads "Loading..." until the link arrives.
  3. Pressing "Copy link" copies the link, and the button icon changes to a check for about two seconds.
  4. The "Copy link" button stays disabled until the link has loaded.

#### DL-6.10 Invite teammates by email

- Tag: cursor. Evidence: bundle. Boundary: yes - lives in the invite modal, which opens from Cursor Settings. Cursor: cursor only
- What: Under "Invite by email" an input (placeholder "Email, comma separated") takes one or more addresses. The "Send invite" button (arrow icon, loading icon while it works) stays disabled until the text looks like an email address, and Enter sends too. Each valid address gets a team invite with the Member role, and the modal closes when the sending is done.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component HVw, sendTeamInvite on the dashboard client with role MEMBER, email check `/\S+@\S+\.\S+/`)
- Pieces:
  1. Under "Invite by email" an input with the placeholder "Email, comma separated" takes one or more addresses.
  2. The "Send invite" button, which has an arrow icon, stays disabled until the text looks like an email address.
  3. Pressing "Send invite" or pressing Enter in the input sends the invites, and a loading icon shows on the button while it works.
  4. Each valid address gets a team invite with the Member role, and the modal closes when the sending is done.

#### DL-6.11 Choose data sharing or Privacy Mode

- Tag: cursor. Evidence: bundle. Boundary: yes - the control sits in the Privacy section of the Cursor Settings General tab, and Privacy Mode also limits cloud agent features. Cursor: cursor only - the command id appears only as the call from the dead "Share Now" link
- What: A signed-in person sees a "Privacy" section on the General tab. Its row reads "Data Sharing Enabled" (check icon) or "Privacy Mode" or "Privacy Mode (Legacy)" (lock icon) with a sentence on what that means, and a "Configure" dropdown with "Share Data" ("Improve Cursor for everyone"), "Privacy Mode" ("No training. Code may be stored for Background Agent and other features.") and, only for someone already on it, "Privacy Mode (Legacy)". A small ring shows while the change saves. On the legacy mode a note says "Privacy Mode (Legacy) is enabled. Background Agent and some features not available." with a "Switch to Privacy Mode" link. On team accounts the dropdown is hidden and the text says "Data Sharing is controlled by your team admin." with a link. A first-day note ("Data Sharing is paused for your first day of usage. Sharing will activate in N hours.") has a "Share Now" link that is switched off in code, and the command it calls is not registered anywhere I found.
- Commands: cursorAuth.skipPrivacyModeGracePeriod
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component Hww inside the General tab component, with helper Www; options list for Share Data, Privacy Mode, Privacy Mode (Legacy); updateUserPrivacyMode and setServerRecordedPrivacyMode; switch S1h fixed false)
- Pieces:
  1. A signed-in person sees a "Privacy" section on the General tab of Cursor Settings with a row that reads "Data Sharing Enabled" (check icon), "Privacy Mode" or "Privacy Mode (Legacy)" (lock icon), plus a sentence on what that means.
  2. A "Configure" dropdown offers "Share Data" ("Improve Cursor for everyone") and "Privacy Mode" ("No training. Code may be stored for Background Agent and other features.").
  3. "Privacy Mode (Legacy)" shows in the dropdown only for someone who is already on it.
  4. A small ring shows while a change saves.
  5. On the legacy mode a note reads "Privacy Mode (Legacy) is enabled. Background Agent and some features not available." with a "Switch to Privacy Mode" link.
  6. On team accounts the dropdown is hidden and the text says "Data Sharing is controlled by your team admin." with a link.

#### DL-6.12 Switch Privacy Mode confirm

- Tag: cursor. Evidence: bundle. Boundary: yes - opens from the Privacy section of Cursor Settings. Cursor: cursor only - drawn by Cursor's own pretty dialog engine (DL-1), so `window.dialogStyle` does not change it
- What: When a person on Privacy Mode (Legacy) picks another mode, either from the dropdown or the "Switch to Privacy Mode" link, a dialog titled "Switch Privacy Mode?" shows a warning callout, "You won't be able to switch back to Privacy Mode (Legacy)." with buttons "Switch" and "Cancel". Cancel leaves the mode as it was. No box shows for any other change.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component Hww, openDialog call with title "Switch Privacy Mode?", callout message, primaryButton Switch, cancelButton Cancel, shown only when the current mode is NO_STORAGE)
- Pieces:
  1. When a person on "Privacy Mode (Legacy)" picks another mode, from the dropdown or from the "Switch to Privacy Mode" link, a dialog titled "Switch Privacy Mode?" opens.
  2. The dialog shows a warning callout "You won't be able to switch back to Privacy Mode (Legacy)."
  3. The buttons are "Switch" and "Cancel", where "Switch" changes the mode and "Cancel" leaves the mode as it was.
  4. No dialog shows for any other change of mode.
  5. The dialog looks the same whatever `window.dialogStyle` is set to, because Cursor draws it with its own dialog engine.

#### DL-6.13 Privacy update failed box

- Tag: cursor. Evidence: bundle. Boundary: yes - opens from the Privacy section of Cursor Settings. Cursor: cursor only - Cursor's pretty dialog engine
- What: If saving a privacy change fails, a dialog titled "Failed to update privacy settings" says "Please try again. Request ID: {id}" with one OK button, and the mode shown stays as it was.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component Hww, openDialog call with title "Failed to update privacy settings", primaryButton OK, request id from qn())
- Pieces:
  1. If saving a privacy change fails, a dialog titled "Failed to update privacy settings" opens.
  2. The dialog says "Please try again. Request ID: {id}" with a single "OK" button.
  3. The mode shown in the Privacy section stays as it was.
  4. The dialog looks the same whatever `window.dialogStyle` is set to, because Cursor draws it with its own dialog engine.

#### DL-6.14 Import settings from VS Code confirm

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only - the command is "Import VS Code Extensions and Settings" (F1); the notification text is messages 8445 and 8446; the dialog is Cursor's pretty dialog engine (DL-1)
- What: A sticky info notification "Bring your VS Code setup to Cursor. Import your settings and extensions for a seamless start." has an Import button. It appears some time after onboarding when the profile has not imported yet and a .vscode folder with content exists in the person's home folder (not in the Agents window). Import opens a dialog titled "Import settings from VS Code" with the text "Settings, extensions, and other profile preferences will be replaced with current settings from VS Code" and three buttons: Import (primary), "Import without extensions" and Cancel. Either import button runs the import command with the matching option.
- Commands: cursorai.action.importvscode
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (function hHw with the openDialog call and the dialog icon for import; class wKa with schedulePostOnboardingVSCodeImportToast and maybeShowPostOnboardingVSCodeImportToast; helper kbg checks the .vscode folder; wbg registers the command with LABEL "Import VS Code Extensions and Settings")
- Pieces:
  1. A sticky info notification "Bring your VS Code setup to Cursor. Import your settings and extensions for a seamless start." with an "Import" button appears some time after onboarding.
  2. The notification shows only when the profile has not imported yet and a `.vscode` folder with content exists in the person's home folder, and never in the Agents window.
  3. Pressing "Import" opens a dialog titled "Import settings from VS Code" with the text "Settings, extensions, and other profile preferences will be replaced with current settings from VS Code".
  4. The dialog's buttons are "Import" (primary), "Import without extensions" and "Cancel".
  5. Either import button runs `cursorai.action.importvscode` with the matching option, which is also in the Command Palette as "Import VS Code Extensions and Settings", and "Cancel" imports nothing.

#### DL-6.15 Import from VS Code in Cursor Settings with confirm popup

- Tag: cursor. Evidence: bundle. Boundary: yes - the row sits on the General tab of Cursor Settings. Cursor: cursor only - a popover, not a dialog; the Command Palette entry for the same command asks nothing in the code I read
- What: In the Preferences group of the General tab, the "Import Settings from VS Code" row ("Import settings, extensions, and keybindings from VS Code") has an Import button. It opens a small popup, "Confirm VS Code import", that says "Are you sure? This replaces your settings and extensions with your vscode ones. This cannot be undone." with Cancel and Confirm. After Confirm the button reads "Importing" and then "Success!" for about five seconds, and a notification says "VS Code import completed!" or "Error importing extensions, please try again."
- Commands: cursorai.action.importvscode
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component gww in the Preferences section component mww, text constant uww, Popup with aria-label "Confirm VS Code import", executeCommand with the import command id)
- Pieces:
  1. In the Preferences group of the General tab of Cursor Settings, the row "Import Settings from VS Code", with the line "Import settings, extensions, and keybindings from VS Code", has an "Import" button.
  2. Pressing "Import" opens a small popup, "Confirm VS Code import", that says "Are you sure? This replaces your settings and extensions with your vscode ones. This cannot be undone." with Cancel and Confirm buttons.
  3. Pressing Cancel closes the popup and imports nothing.
  4. After Confirm the button reads "Importing", and then "Success!" for about five seconds.
  5. A notification says "VS Code import completed!" or "Error importing extensions, please try again."
  6. The confirmation is a popup anchored to the button and not a dialog, and the Command Palette entry for the same command asks nothing.
