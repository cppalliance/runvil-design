Group: DL-1 Message, confirm and input boxes, and About. Pass: 3. Date: 2026-10-25.

### DL-1.1
1. A confirm box shows a message, an optional smaller detail line, a primary button and a Cancel button, and the feature that raised it learns whether you confirmed.
2. The primary button says "Yes" unless the caller gives its own label, such as "Clear".
3. Pressing Enter presses the primary button and pressing Escape cancels, while the in-page box is open.
4. Choosing Cancel counts as not confirmed, so the action that raised the box does not go ahead.
5. Setting `window.dialogStyle` chooses whether the confirm box is an OS message box or the in-page box, with values `native` and `custom`.

### DL-1.2
1. A confirm, prompt or input box can show one checkbox under the message, and the answer tells the caller whether it was ticked.
2. Clicking the label text next to the checkbox toggles it, the same as clicking the box itself.
3. Pressing Tab moves focus to the checkbox and pressing Space toggles it.
4. The quit and close confirm shows a "Do not ask me again" checkbox, and ticking it while confirming sets `window.confirmBeforeClose` to `never`.
5. The untitled workspace save prompt shows an "Always discard untitled workspaces without asking" checkbox, and ticking it turns off `window.confirmSaveUntitledWorkspace`.
6. Other boxes carry their own checkboxes, such as "Remember my credentials" on the proxy login, "Remember my choice in user settings" on the pre-launch task prompt, and "Don't restore editors" on the window crash box.

### DL-1.3
1. A prompt box shows a message, an optional detail and two or more custom buttons, and each button hands its own value back, so one box can offer Save, Don't Save or Cancel.
2. The first button is the default, and pressing Enter presses it.
3. An optional Cancel button comes after the custom buttons, and pressing Escape presses it.
4. When the caller gives no buttons the box shows a single OK button.
5. Pressing Left or Right arrow, or Tab, moves focus between the buttons.
6. Setting `window.dialogStyle` chooses whether the prompt is an OS message box or the in-page box, with values `native` and `custom`.

### DL-1.4
1. An info box shows an info icon, a bold message, an optional detail and a single OK button.
2. A warning box has the same layout with a warning icon.
3. An error box has the same layout with an error icon.
4. Pressing Enter or Escape closes the box.
5. Setting `window.dialogStyle` chooses whether these boxes are OS message boxes or the in-page box, with values `native` and `custom`.
6. In Cursor an extra `customError` call shows an error box in the in-page style whatever `window.dialogStyle` says, though no caller of it was found in the editor. [cursor only]

### DL-1.5
1. Any box can show a smaller detail paragraph under the bold message, for example "Your changes will be lost if you don't save them."
2. In the OS box the detail shows as the native detail text under the message.
3. In the in-page box the detail sits under the icon and message.
4. The detail is plain text only, so Markdown marks show exactly as typed.

### DL-1.6
1. A caller can ask for the in-page box on purpose, which overrides `window.dialogStyle`, and the Workspace Trust prompts do this.
2. The in-page box can show extra body paragraphs rendered from Markdown, with links you can click.
3. The in-page box can show a custom icon in place of the standard severity icon.
4. The in-page box can show small caption text under each button.
5. A caller can remove the close X and the Escape shortcut, so the box can only be dismissed with its buttons.
6. In Cursor the captions under the buttons are passed to the widget under the older option name `buttonDetails`, and they still show under the buttons. [cursor only]

### DL-1.7
1. An input box shows a message, one or more text fields, an optional checkbox, a primary button and a Cancel button.
2. Each field can have placeholder text and an optional starting value.
3. A field can be a plain text field or a password field that hides what you type.
4. The first field is focused with its text selected when the box opens.
5. Pressing Enter inside a field accepts the box, and pressing Tab moves to the next field.
6. The primary button says "OK" unless the caller gives its own label.
7. The input box is always the in-page box, whatever `window.dialogStyle` says, because the OS has no version of it.

### DL-1.8
1. When a network request meets a proxy that wants a login, a warning box titled "Proxy Authentication Required" opens and says "The proxy host:port requires a username and password."
2. The box has a Username field and a Password field that hides what you type, and any known values are filled in.
3. A "Remember my credentials" checkbox remembers the choice under the application storage key `window.rememberProxyCredentials`, which is not a setting.
4. Pressing the "Log In" button, or Enter, sends the username and password back to the request.
5. Pressing Cancel, or Escape, replies with no credentials.

### DL-1.9
1. Setting `window.dialogStyle` chooses whether confirm, prompt and info boxes are drawn by the OS or by the app, with values `native` (the default) and `custom`.
2. With `custom`, those boxes appear as the in-page box inside the window instead of an OS message box.
3. Input boxes are always in-page, and some callers force the in-page box whatever the setting says.
4. In Cursor the About box ignores `window.dialogStyle` and always opens its own window, and Cursor's second dialog engine (DL-1.22) is not controlled by the setting either. [cursor only]
5. Cursor shows its own description text for `window.dialogStyle` in Settings, without the smoke-test-driver sentence that stock has. [cursor only]

### DL-1.10
1. On Windows the buttons keep Cancel last.
2. On macOS the default button sits on the trailing side with Cancel just to its left.
3. On Linux the buttons use the macOS order and the whole row is flipped.
4. The OS box uses the app name as its title when the caller gives none, and turns off link-style buttons.
5. The in-page box arranges its own buttons in the same per-OS order.

### DL-1.11
1. While a box is open the rest of the window cannot be used.
2. A box raised while another is open waits in a queue, and boxes show one after another, per window.
3. The in-page box covers the window with a dimmed layer.
4. Clicking the dimmed layer sends focus back into the box.
5. On Windows and Linux with drawn window controls (settings `window.titleBarStyle` and `window.controlsStyle`), the minimize, maximize and close buttons are dimmed while a box is up. [stock only]

### DL-1.12
1. Pressing Enter accepts the box while a text field has focus.
2. Pressing Escape closes the box as Cancel.
3. Pressing Tab, Shift+Tab, Left arrow or Right arrow walks focus through the links, text fields, checkbox and buttons in visual order, and wraps around at the ends.
4. On macOS, pressing Cmd+D presses the second button, such as No or Don't Save.
5. Space is left to the focused control, such as toggling the checkbox or pressing the focused button.
6. Almost every other shortcut is blocked while the box is up, and only copy, cut, select all and paste pass through (the pending box also lets quit and reload through).
7. In later stock builds (newer-than-1.128) the arrow keys stop moving focus while you type in a text field. [stock only]
8. In Cursor the box ignores keys when it is not on screen, and Escape closes it on key release without needing a prior key press. [cursor only]
### DL-1.13
1. When the in-page box opens, focus goes to its first text field with the text selected, or to the default button when there are no fields.
2. Focus cannot leave the box, because anything that tries to take it is pulled back.
3. When the box closes, focus returns to the element that had it before.
4. A dirty-editor close prompt first brings the window to the front, so the box is not hidden behind other windows.

### DL-1.14
1. A small X in the top right of the in-page box, named "Close Dialog", closes the box when clicked.
2. Closing with the X counts as Cancel.
3. When the caller turns the close action off, the X is not drawn and pressing Escape does nothing either.

### DL-1.15
1. Screen readers treat the in-page box as a modal dialog.
2. The box is labelled by its icon and message, and described by its message, detail, body and footer text.
3. The icon has a spoken name of Info, Warning, Error or In Progress.
4. Links in the body and footer can be reached with Tab and are underlined, so colour is not the only cue.

### DL-1.16
1. A pending box shows a spinning icon and a message while work runs.
2. The message changes as the running task reports progress.
3. The box has a Cancel or Dismiss button, unless the caller sets it to sticky, and then there is no button.
4. Pressing Escape cancels the box unless it is sticky, and pressing Enter presses the default button.
5. It is the look used for work shown at the dialog progress location, such as the waiting boxes at shutdown.
6. In stock the pending box also dims the window minimize, maximize and close buttons while it is up, and in Cursor it does not. [stock only]

### DL-1.17
1. Choosing Help > About on Windows and Linux opens the About box, and the update row sits just above it in the Help menu.
2. Choosing the "About {name}" row at the top of the macOS app menu opens the About box, and the Help menu on macOS has no About row.
3. Running "Help: About" (`workbench.action.showAboutDialog`) from the Command Palette opens the About box.
4. In stock the About box is an OS message box, or the in-page box when `window.dialogStyle` is `custom`.
5. In Cursor About opens its own small window (DL-1.20), ignores `window.dialogStyle`, and falls back to an OS box only if that window fails to open. [cursor only]

### DL-1.18
1. The About box has a Copy button that puts every version line on the clipboard, and an OK button that closes it.
2. The copied text is plain, without the "x days ago" note shown on screen.
3. In the OS box on Windows and Linux the Copy button has a C mnemonic.
4. In the Cursor About window the main button reads "Copy version info", copies the version lines and closes the window, and pressing Enter does the same. [cursor only]
5. Pressing Escape closes the Cursor About window without copying. [cursor only]
6. The OS fallback box in Cursor keeps the stock Copy and OK buttons.

### DL-1.19
1. The About text lists Version, Commit, Date, Electron, Chromium, Node.js, V8 and OS, one per line.
2. In stock the Version line carries a "user setup", "system setup" or "Universal" note when that is known.
3. The Date line shows "n days ago" on screen only, and the copied text leaves it out.
4. Stock also lists an ElectronBuildId line, and Cursor has none. [stock only]
5. Cursor's copied text adds the lines VS Code Extension API, Layout and xterm.js. [cursor only]
6. Cursor adds a Build Type line (Stable, Nightly or Dev/Debug) and a Release Track line (Default, Early Access, Nightly, Dogfood or Candidate). [cursor only]
7. Cursor adds a "Local Mode: enabled" line when local mode is on. [cursor only]
8. A later stock build (newer-than-1.128) adds two Copilot package lines to the About text. [stock only]

### DL-1.20
1. The Cursor About window is about 380 px wide, with no title, no full screen, a see-through background and a drag strip along the top. [cursor only]
2. It shows the app icon, the product name, "Version x", the build date and a copyright line. [cursor only]
3. Hovering the version or the date shows a copy icon, and clicking it copies that one value. [cursor only]
4. After a copy, the icon flips to a check mark for about 1.5 seconds. [cursor only]
5. The window resizes to fit its content. [cursor only]
6. Pressing Escape closes the window, and the window also closes when the app shuts down. [cursor only]
7. A second About request reuses the open window instead of opening another. [cursor only]

### DL-1.21
1. Next to "Copy version info" the About window has an update button whose label follows the update state. [cursor only]
2. Clicking it runs the action for the current state: `update.checkForUpdate`, `update.downloadUpdate`, `update.installUpdate` or `update.restartToUpdate`. [cursor only]
3. The button shows a spinning ring while the app is checking, downloading or installing. [cursor only]
4. After an explicit check that finds nothing, the button reads "No updates available" and is disabled. [cursor only]
5. When updates are turned off, the button reads "Updates unavailable". [cursor only]

### DL-1.22
1. Cursor draws its own in-page modal with a bold title and a message that is plain text or a list of blocks. [cursor only]
2. A message block can be plain text, a callout with an icon and colour, a copyable value, or Markdown. [cursor only]
3. The modal can show an optional icon. [cursor only]
4. It has a primary button that can be styled as danger, optional extra secondary buttons and a Cancel button. [cursor only]
5. It can show optional text fields and an optional checkbox. [cursor only]
6. It can show a "confirm" checkbox that must be ticked before the primary button works. [cursor only]
7. It fades in below the top edge over a half-dark backdrop. [cursor only]
8. It is separate from the stock box, so `window.dialogStyle` does not apply to it, and it is not created in the Agents window. [cursor only]

### DL-1.23
1. A box that is given a key and asks for it shows a "Don't ask again" checkbox, and the label and hover text can change. [cursor only]
2. On the quit and close boxes the checkbox sits next to the buttons, and elsewhere it sits under the message. [cursor only]
3. Ticking the checkbox and then picking a button saves that button under the key. [cursor only]
4. Once an answer is saved the box never opens again, and the saved answer is used at once. [cursor only]
5. Cancel is saved only when the box allows it. [cursor only]
6. The saved answers live in Cursor's application storage under `dialogDontAskAgainPreferences` and are not settings. [cursor only]

### DL-1.24
1. In the Preferences section of the General page in Cursor Settings, below the Editor Settings and Keyboard Shortcuts rows, a row titled Reset "Don't Ask Again" Dialogs shows the description "See warnings and tips that you've hidden" and a Show or Hide button. [cursor only]
2. Pressing Show lists each hidden dialog with a Restore button, and a Reset All button below the list. [cursor only]
3. Each name is the saved key turned into words, such as Quit Confirmation, plus two agent warnings named About Allowlist and Run Everything Mode Warning. [cursor only]
4. Pressing Restore on a row makes that dialog open again, and Reset All makes every hidden dialog open again. [cursor only]
5. With nothing saved the list says "No Hidden Dialogs Yet" and "You haven't marked any dialogs as "Don't ask again". Any hidden dialogs will appear here to manage." [cursor only]
6. Searching Cursor Settings for dialogs, warnings, reset or don't ask again finds the row. [cursor only]

### DL-1.25
1. Pressing Enter accepts the primary button, unless the key was pressed on a button, link or field, the box turns Enter off, or the primary button is busy or locked. [cursor only]
2. Pressing Shift+Enter presses the extra button marked with that shortcut. [cursor only]
3. Pressing Escape cancels the box, and so does clicking the backdrop, except when the press started inside the box. [cursor only]
4. When the box opens, focus goes to the primary button, or to the first text field when there are fields. [cursor only]
5. The Cancel button shows "(esc)" in its label and the primary button shows an Enter symbol. [cursor only]
6. The Cancel button sits at the left edge on macOS and Linux. [cursor only]
7. Screen readers get an h1 title and aria labels on copyable values, but no dialog role or modal flag. [cursor only]
### DL-1.26
1. Choosing File > Open Recent > Clear Recently Opened..., or running File: Clear Recently Opened (`workbench.action.clearRecentFiles`) from the Command Palette, opens a confirm box.
2. The box asks "Do you want to clear all recently opened files and workspaces?" with the detail "This action is irreversible!", a Clear button and a Cancel button.
3. Pressing Clear wipes both the global and the per-workspace recent lists.
4. Pressing Cancel leaves the recent lists as they were.

### DL-1.27
1. When a file holds hidden Line Separator or Paragraph Separator characters, a box titled "Unusual Line Terminators" says "Detected unusual line terminators" and names the file.
2. The box offers "Remove Unusual Line Terminators", which takes the characters out, and "Ignore".
3. Choosing Ignore silences the box for that file.
4. Setting `editor.unusualLineTerminators` chooses what happens, with values `auto` (remove them without asking), `off` (ignore them) and `prompt` (show the box, the default).

### DL-1.28
1. When a command picked in the Command Palette throws an error, an error box opens that says "Command '{name}' resulted in an error".
2. The box shows the error text as its detail line.
3. Pressing the OK button closes the box.

### DL-1.29
1. Starting a second refactor preview while one is open shows a box that says "Another refactoring is being previewed." with the detail "Press 'Continue' to discard the previous refactoring and continue with the current refactoring."
2. Choosing Continue discards the earlier preview and goes on with the new one, and choosing Cancel keeps the earlier preview.
3. Pressing Apply in the preview (`refactorPreview.apply`, Ctrl+Enter or Cmd+Enter while the preview pane is focused) when one file changed in the meantime shows a warning box "Cannot apply refactoring because '{file}' has changed in the meantime."
4. When several files changed in the meantime, the warning says "Cannot apply refactoring because {n} other files have changed in the meantime."

### DL-1.30
1. After you ask for an update check by hand and nothing new exists, an info box says "There are currently no updates available."
2. In stock the box shows only from the window that last had focus. [stock only]
3. In stock on macOS with every window closed, the main process shows a native box with the same message instead. [stock only]
4. In Cursor the box comes from the older renderer-only flow, with no last-focus check and no windowless macOS box. [cursor only]

### DL-1.31
1. Before connecting to a remote Linux that the app no longer supports, a warning box says "You are about to connect to an OS version that is unsupported by {product}."
2. The box has Allow, Learn More and Cancel buttons.
3. A "Do not show again" checkbox remembers the Allow answer for that remote.
4. Choosing Learn More opens the FAQ page and then shows the box again.
5. Choosing Cancel does not connect.

### DL-1.32
1. When a debug pre-launch task ends with errors and `debug.onTaskErrors` is `prompt` (the default), a warning box asks what to do, with buttons "Debug Anyway", "Show Errors" and "Abort".
2. A "Remember my choice in user settings" checkbox on that box saves your button choice into `debug.onTaskErrors`.
3. Setting `debug.onTaskErrors` chooses what happens without a box, with values `prompt`, `debugAnyway`, `showErrors` and `abort`.
4. A second box covers a pre-launch task that could not run at all, with a Debug Anyway button, a button that opens the task configuration, and Cancel.
5. A "Remember my choice for this task" checkbox on the second box keeps the answer per workspace.

### DL-1.33
1. When a window hangs, the main process shows an OS warning box that says "The window is not responding" with Reopen, Close and Keep Waiting buttons.
2. When a window dies, an OS warning box says "The window terminated unexpectedly", with the reason and exit code when they are known.
3. The crash box offers Reopen (or New Window for an empty window) and Close.
4. When a folder or workspace is open, a "Don't restore editors" checkbox lets the reopened window skip its saved editors.
5. A hang is ignored while developer tools are open or in an extension development host.

### DL-1.34
1. When the main process is asked to open a path that does not exist, for example from the command line, an OS info box titled "Path does not exist" says "The path '{path}' does not exist on this computer."
2. For a bad URI the box is titled "URI can not be opened" instead.
3. Pressing the OK button closes the box.

### DL-1.35
1. In a remote window whose folder or workspace file is gone, a warning box says "Workspace does not exist" with the detail "Please select another workspace to open."
2. The box has an "Open Workspace..." button and a Cancel button.
3. Choosing "Open Workspace..." opens the workspace file picker when a workspace file was open, and the folder picker when a folder was open.
4. Choosing Cancel closes the box and leaves the window as it is.

### DL-1.36
1. In later stock builds the detail line of a box can be Markdown. [stock only]
2. The in-page box renders that Markdown with working links. [stock only]
3. The OS box shows the same detail as plain text. [stock only]

### DL-1.37
1. In later stock builds a caller can ask for a vertical layout, with full-width buttons stacked under the message. [stock only]
2. The service option that reaches this layout from the custom dialog options exists only in later stock, and the 1.128 widget already has the layout but only the Sessions code uses it. [stock only]

### DL-1.38
1. After a restart-only setting changes, a confirm box says "A setting has changed that requires a restart to take effect." with the detail "Press the restart button to restart {product} and enable the setting." and a Restart button next to Cancel.
2. Choosing Restart restarts the app, and cancelling keeps the new value but it only takes effect at the next start.
3. The box appears only while the window has focus, only when the value really differs from the one in use, and not when just a default changed or while settings sync is being turned on.
4. In the desktop app stock watches `window.titleBarStyle`, `window.menuStyle` (Windows and Linux), `window.nativeTabs`, `window.nativeFullScreen`, `window.clickThroughInactive` (macOS), `window.controlsStyle` (Windows and Linux), `editor.accessibilitySupport` (Linux, when turned on), `security.workspace.trust.enabled`, `security.restrictUNCAccess`, `accessibility.verbosity.debug`, `telemetry.feedback.enabled`, `workbench.enableExperiments` and, on builds that are not stable, `_extensionsGallery.enablePPE`.
5. Setting `workbench.enableExperiments` (on by default, tagged as using an online service) raises the box when it is switched, and that prompt is its only visible effect in the app.
6. Cursor watches `update.mode`, `workbench.activityBar.orientation` and `application.experimental.extensionMonitor.enabled` as well and does not watch `window.menuStyle` or `telemetry.feedback.enabled`. [cursor only]

## Unclear
- DL-1.4 piece 6: the `customError` call has no caller in the editor bundle, so a person cannot reach that behavior today. Keep it only if the tracker wants to count the call.
- DL-1.6 piece 6 is a difference note read from a code name, not a look. The `buttonDetails` option name has no visible effect different from stock, so a tracker may want to drop it.
- DL-1.36 and DL-1.37 are newer-than-1.128 and exist only in later stock, so a Cursor build has nothing to check for them.
- DL-1.30 piece 4: whether Cursor's older flow shows the box from the focused window or the window that ran the check was not confirmed on screen.
- DL-1.24 piece 4: the Restore and Reset All behavior is from the button labels and the saved-key list, and nothing was run to confirm it.
- DL-1.1 to DL-1.5 and DL-1.10 to DL-1.16 are shared box behavior with no entry point of their own, so each one is checked through whichever feature raises a box.
