Bucket: G. Date: 2026-10-25. Pass: 4.

### NEW-G-1 New text file in the default language
- Group: WB-1
- Tag: both
- Evidence: source
- Boundary: no
- What: The person starts a blank untitled text file from the File menu, Ctrl+N or the Command Palette, and it opens in the language set by `files.defaultLanguage`.
- Commands: workbench.action.files.newUntitledFile; openEditors.newUntitledFile
- Menus: MenubarFileMenu (group 1_new, order 1, row New Text File); EmptyEditorGroupContext (group 1_file, order 10); EditorTabsBarContext (group 1_file, order 10); ViewTitle (Open Editors view, group navigation, order 5); CommandPalette
- Keys: Ctrl+N (Cmd+N on macOS) on workbench.action.files.newUntitledFile
- Settings: files.defaultLanguage
- Cursor: present - the command id, its label "New Untitled Text File", openEditors.newUntitledFile and files.defaultLanguage were found in the bundle and in nls; the 1_file rows of the empty-group and tab-bar menus and the key were not checked
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (lines 661-698); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (lines 281-285 and 670-684); V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (lines 930-948); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (lines 322-325); V\src\vs\workbench\services\untitled\common\untitledTextEditorService.ts (lines 247-253); V\src\vs\workbench\services\untitled\common\untitledTextEditorModel.ts (lines 85-90); C bundle (ids workbench.action.files.newUntitledFile and openEditors.newUntitledFile, setting files.defaultLanguage, nls 7186 and 7166)
- Pieces:
  1. File > New Text File opens a new empty untitled text editor tab in the active group and keeps it open as a pinned tab.
  2. Pressing Ctrl+N (Cmd+N on macOS) does the same, and the Command Palette lists it as File: New Untitled Text File.
  3. Right-clicking the empty editor area or the tab strip offers a New Text File row, and the Open Editors view title has a New Untitled Text File button.
  4. Setting `files.defaultLanguage` names the language id that a new untitled file starts with, and an empty value leaves it unset.
  5. Setting `files.defaultLanguage` to `${activeEditorLanguage}` makes the new file start in the language of the text editor that was active before it opened, and the person can still change the language later by hand or by auto-detection.
  6. The default language is not applied when a file is created with an associated path or with an explicit language.

### NEW-G-2 Proxy and certificate settings
- Group: ED-8
- Tag: both
- Evidence: source
- Boundary: no
- What: The person sets how the app's own web requests reach the internet - a proxy address, hosts that skip it, proxy credentials and certificate checks - in the Settings editor under Application > Proxy.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: http.proxy; http.noProxy; http.proxyAuthorization; http.proxyKerberosServicePrincipal; http.proxyStrictSSL; http.systemCertificates; http.systemCertificatesNode
- Cursor: changed - http.proxy, http.noProxy, http.proxyAuthorization, http.proxyKerberosServicePrincipal, http.proxyStrictSSL and http.systemCertificates are registered in the bundle; http.systemCertificatesNode was not found; Cursor also has its own Browser & Network tab (ED-8.29) for HTTP Compatibility Mode
- Source: V\src\vs\platform\request\common\request.ts (lines 236-361); V\src\vs\platform\request\node\requestService.ts (lines 85-143); V\src\vs\platform\windows\electron-main\windowImpl.ts (lines 1114-1132); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (lines 389-390); C bundle (the http configuration node with the six ids above, 0 hits for http.systemCertificatesNode)
- Pieces:
  1. The Settings editor table of contents has an Application > Proxy section that lists the `http.*` settings that apply to the app.
  2. Setting `http.proxy` takes a proxy URL starting with http, https, socks, socks4, socks4a, socks5 or socks5h (an empty value is allowed), and when it is empty the app uses the `http_proxy` and `https_proxy` environment variables.
  3. Setting `http.noProxy` lists domain names whose HTTP and HTTPS requests skip the proxy.
  4. Setting `http.proxyAuthorization` is sent as the `Proxy-Authorization` header on every request, and it is empty by default.
  5. Setting `http.proxyKerberosServicePrincipal` overrides the Kerberos service name used to sign in to the proxy, and when it is empty a name based on the proxy host is used.
  6. Setting `http.proxyStrictSSL` (on by default) checks the proxy server certificate against the trusted certificate authorities.
  7. Setting `http.systemCertificates` (on by default) loads trusted certificate authorities from the operating system, and on Windows and macOS turning it off needs a window reload.
  8. Setting `http.systemCertificatesNode` (experimental) loads the system certificates through Node.js built-in support and needs a window reload. [stock only]
  9. When the proxy asks for a username and password, the Proxy Authentication Required box described in DL-1.8 opens.

### NEW-G-3 Issue Reporter wizard
- Group: WB-6
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Report Issue can open a three-step wizard in an editor tab instead of the classic reporter window, where the person adds screenshots or videos, describes the problem and then reviews and submits it.
- Commands: workbench.action.openIssueReporter; vscode.openIssueReporter; workbench.action.reportPerformanceIssueUsingReporter; workbench.action.issueReporter.captureScreenshot; workbench.action.issueReporter.toggleRecording
- Menus: MenubarHelpMenu (row Report Issue); CommandPalette
- Keys: Ctrl+Shift+S (workbench.action.issueReporter.captureScreenshot) and Ctrl+Shift+R (workbench.action.issueReporter.toggleRecording), Cmd on macOS, both only while an Issue Reporter tab is open
- Settings: issueReporter.wizard.enabled; issueReporter.wizard.fullWorkspaceScan
- Cursor: missing - the wizard strings, the editor pane and both settings were not found in the bundle or in nls; the command id workbench.action.openIssueReporter and the Report Issue label exist, so Cursor keeps a classic reporter. The commands listed here other than the first also appear in the bucket B leftovers, so the parent should merge the two features.
- Source: V\src\vs\workbench\contrib\issue\electron-browser\issue.contribution.ts (lines 47-76); V\src\vs\workbench\contrib\issue\electron-browser\nativeIssueFormService.ts (lines 59-85); V\src\vs\workbench\contrib\issue\electron-browser\issueReporterKeybindings.ts (lines 21-112); V\src\vs\workbench\contrib\issue\browser\issueReporterOverlay.ts (step titles, field labels and the attachment limit of 5); V\src\vs\workbench\contrib\issue\common\issue.contribution.ts (lines 70-130); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (lines 376-380, the Issue Reporter section with hide set to not web); C bundle (0 hits for IssueReporterEditor, IssueReporterOverlay and issueReporter.wizard)
- Pieces:
  1. With `issueReporter.wizard.enabled` on, Help > Report Issue opens an Issue Reporter editor tab, and with it off (the default) the classic reporter opens in its own window. [stock only]
  2. The wizard has three steps named Attachments, Describe and Review, a Back and a Next button, and a "Step N of 3" label. [stock only]
  3. The Attachments step takes up to 5 screenshots or videos through a floating capture bar with a Screenshot button, a Capture options menu (including Hide Toolbar in Screenshots) and a Record video button. [stock only]
  4. While an Issue Reporter tab is open, Ctrl+Shift+S takes a screenshot and Ctrl+Shift+R starts or stops a recording, even when another widget has focus. [stock only]
  5. The Describe step asks for a Target, an Extension when the target is an extension, a Category (Bug, Feature Request or Performance Issue), a Title with a Generate from description button, and a Markdown Description, and it lists Similar Issues found from the title. [stock only]
  6. For a Performance Issue the Next button shows "Loading diagnostics..." until the performance diagnostics are ready. [stock only]
  7. Setting `issueReporter.wizard.fullWorkspaceScan` (on by default) makes the diagnostics scan walk the whole workspace instead of stopping at 20,000 files. [stock only]
  8. The Review step shows a Review and submit page, and submitting opens the prepared issue in the browser while the wizard stays open with a Close button. [stock only]
  9. The Settings editor table of contents has an Issue Reporter section that is hidden on desktop, so the two wizard settings are found through search or the JSON file. [stock only]
### NEW-G-4 Key matching settings
- Group: ED-8
- Tag: both
- Evidence: source
- Boundary: no
- What: The person chooses how key presses are matched to shortcuts and whether AltGraph counts as Ctrl+Alt, in the Settings editor under Application > Keyboard.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: keyboard.dispatch; keyboard.mapAltGrToCtrlAlt
- Cursor: present - both ids were found in the bundle and nls 2118 holds the dispatch description; keyboard.layout (the web-only layout setting, mapped to SS-3.38) was not found
- Source: V\src\vs\platform\keyboardLayout\common\keyboardConfig.ts (lines 21-52); V\src\vs\workbench\services\keybinding\electron-browser\nativeKeyboardLayout.ts (lines 40-70); V\src\vs\workbench\services\keybinding\common\windowsKeyboardMapper.ts (lines 414-415); V\src\vs\workbench\services\keybinding\common\macLinuxKeyboardMapper.ts (lines 992-993); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (line 395); C bundle (ids keyboard.dispatch and keyboard.mapAltGrToCtrlAlt, nls 2118)
- Pieces:
  1. Setting `keyboard.dispatch` is listed only on macOS and Linux and chooses `code` (the default and the recommended value) or `keyCode` for how a key press is matched to a shortcut.
  2. With `keyCode` the app uses a plain key code mapper in place of the one built from the current keyboard layout.
  3. Changing the keyboard settings takes effect right away, without a restart.
  4. Setting `keyboard.mapAltGrToCtrlAlt` is listed only on Windows, is off by default, and makes AltGraph combinations count as Ctrl+Alt combinations when matching shortcuts.

### NEW-G-5 macOS Touch Bar buttons
- Group: WB-10
- Tag: both
- Evidence: source
- Boundary: no
- What: On a Mac that has a Touch Bar, the strip above the keyboard shows buttons for going back and forward and for running and stepping a debug session, and two settings turn it off or hide single buttons.
- Commands: workbench.action.navigateBack; workbench.action.navigateForward; workbench.action.debug.run; workbench.action.debug.start; workbench.action.debug.continue; workbench.action.debug.pause; workbench.action.debug.stepOver; workbench.action.debug.stepInto; workbench.action.debug.stepOut; workbench.action.debug.restart; workbench.action.debug.stop
- Menus: TouchBarContext (group navigation for Go Back and Go Forward, group 9_debug for the debug buttons)
- Keys: none found
- Settings: keyboard.touchbar.enabled; keyboard.touchbar.ignored
- Cursor: present - TouchBarContext, the back, forward and debug rows and both settings were found in the bundle, and nls 12071 holds the enable description
- Source: V\src\vs\workbench\electron-browser\window.ts (lines 158-170, 377-381 and 905-966); V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 360-380); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (lines 338-350); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (lines 229-253); V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts (touchBar key); C bundle (TouchBarContext rows with the back, forward and debug entries, both settings)
- Pieces:
  1. The Touch Bar shows a Go Back button and a Go Forward button that move through the editor navigation history.
  2. When a debugger is available and no session is running, the Touch Bar shows Start Without Debugging and Start Debugging buttons.
  3. During a debug session the Touch Bar shows Step Over, Step Into, Step Out, Restart and Stop, plus Continue while stopped and Pause while running.
  4. Pressing a button runs its command with the active file as the argument, because the buttons depend on the active editor.
  5. The buttons update a short moment after the active editor, the focus or the debug state changes.
  6. Setting `keyboard.touchbar.enabled` (on by default, listed only on macOS) turns all the buttons off when it is off, and the change applies at once.
  7. Setting `keyboard.touchbar.ignored` takes a list of command ids that must not show on the Touch Bar, for example `workbench.action.navigateBack`.

### NEW-G-6 Screencast Mode
- Group: WB-9
- Tag: both
- Evidence: source
- Boundary: no
- What: The person turns on a demo overlay that rings every mouse click and shows large key presses with the command they run, tuned by six settings.
- Commands: workbench.action.toggleScreencastMode
- Menus: CommandPalette (category Developer)
- Keys: none found
- Settings: screencastMode.fontSize; screencastMode.keyboardOptions; screencastMode.keyboardOverlayTimeout; screencastMode.mouseIndicatorColor; screencastMode.mouseIndicatorSize; screencastMode.verticalOffset
- Cursor: present - the command (nls 3042) and the six setting ids were found in the bundle
- Source: V\src\vs\workbench\browser\actions\developerActions.ts (lines 120-395 and 1236-1316); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (lines 157-158); C bundle (id workbench.action.toggleScreencastMode and the six settings)
- Pieces:
  1. Running Developer: Toggle Screencast Mode from the Command Palette turns the overlay on, and running it again turns it off.
  2. While a mouse button is held down a ring follows the pointer, it shrinks a little while the pointer moves, and it disappears when the button is released.
  3. Setting `screencastMode.mouseIndicatorColor` (a hex color, default `#FF0000`) and `screencastMode.mouseIndicatorSize` (20 to 100 pixels, default 20) set the look of the ring.
  4. Each key press adds a large key label near the bottom of the window with arrow keys drawn as arrows, and a key press with a modifier, Backspace, Escape, an arrow key or a run of more than 20 keys clears the older text first.
  5. When a key press runs a command, the overlay also shows the command name and its shortcut.
  6. Setting `screencastMode.keyboardOptions` picks what shows, with `showKeys`, `showKeybindings` and `showCommands` on by default, `showCommandGroups` off by default to prefix the command category, and `showSingleEditorCursorMoves` on by default (off hides single arrow-key cursor moves).
  7. Settings `screencastMode.fontSize` (20 to 100, default 56), `screencastMode.verticalOffset` (0 to 90 percent from the bottom, default 20) and `screencastMode.keyboardOverlayTimeout` (500 to 5000 milliseconds, default 800) set the size, the height and how long the text stays.
  8. Text typed through an input method shows in the overlay while it is being composed.
  9. Changing any of these settings while the mode is on updates the overlay at once, and the overlay follows whichever window is active.
  10. The Settings editor lists a Screencast Mode section under Workbench in its table of contents.
### NEW-G-7 Confirm when another app opens a path in the app
- Group: DL-5
- Tag: both
- Evidence: source
- Boundary: no
- What: When another application asks the app to open a local or remote file, folder or workspace through its link handler, a warning box asks first, and a checkbox on it turns that question off for good.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.promptForLocalFileProtocolHandling; security.promptForRemoteFileProtocolHandling
- Cursor: present - both settings, the handler table in out\main.js and the channel that writes the settings back were found, and nls 146 to 153 hold the box texts with the product name Cursor filled in
- Source: V\src\vs\code\electron-main\app.ts (lines 156-157 and 835-893); V\src\vs\workbench\electron-browser\window.ts (lines 366-373); V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 382-397); C out\main.js (SECURITY_PROTOCOL_HANDLING_CONFIRMATION_SETTING_KEY with both ids, channel vscode:disablePromptForProtocolHandling); C bundle (the window listener for that channel and both settings); NLS 146 to 153
- Pieces:
  1. When another application opens a link that points at a local file, folder or workspace file, a warning box says "An external application wants to open '{path}' in {product}" and asks whether to open that file, folder or workspace file, with Yes and No buttons.
  2. The box detail warns "If you did not initiate this request, it may represent an attempted attack on your system", and No stops the open.
  3. Choosing Yes with the box "Allow opening local paths without asking" ticked turns `security.promptForLocalFileProtocolHandling` off in the user settings.
  4. Links to a remote path show the same box with the checkbox "Allow opening remote paths without asking", which turns `security.promptForRemoteFileProtocolHandling` off.
  5. Both settings are on by default and apply to the whole application, and with one turned off the matching kind of link opens with no question.
  6. Only local and remote links are asked about in this box, because links for an extension are confirmed separately inside the window.

### NEW-G-8 Windows network share host prompt
- Group: DL-5
- Tag: both
- Evidence: source
- Boundary: no
- What: On Windows, opening a path on a network share whose host is not on the allowed list stops with a warning box that lets the person allow the host once or for good.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.restrictUNCAccess; security.allowedUNCHosts
- Cursor: present - security.restrictUNCAccess and security.allowedUNCHosts are registered, out\main.js has onUNCHostNotAllowed and the not-allowed error, and nls 2736 to 2741 hold the box texts; security.allowedUNCHosts is not in the leftover list and was added here as a related id
- Source: V\src\vs\platform\windows\electron-main\windowsMainService.ts (lines 1239-1300); V\src\vs\workbench\electron-browser\window.ts (lines 342-366); V\src\vs\workbench\common\configuration.ts (lines 180-235); V\src\vs\base\common\errorMessage.ts (line 31); V\src\vs\code\electron-main\app.ts (lines 447-450); C out\main.js (onUNCHostNotAllowed, vscode:configureAllowedUNCHost, ERR_UNC_HOST_NOT_ALLOWED); C bundle (security.allowedUNCHosts and the restrictUNCAccess registration); NLS 2736 to 2741
- Pieces:
  1. On Windows, opening a path whose network share host is not allowed shows a warning box "The host '{host}' was not found in the list of allowed hosts. Do you want to allow it anyway?" with Allow, Cancel and Learn More buttons.
  2. The box detail names the path and says "Unless you trust the host, you should press 'Cancel'", and Cancel stops the open.
  3. Allow lets the open go ahead, and ticking "Permanently allow host '{host}'" also adds the host to `security.allowedUNCHosts` in the user settings.
  4. Learn More opens a web page about the setting and then shows the box again until a choice is made.
  5. Setting `security.restrictUNCAccess` (on by default, listed only on Windows) limits access to hosts on the allowed list or confirmed in the box, and turning it off removes the check.
  6. Setting `security.allowedUNCHosts` is a list of host names without backslashes (for example `my-server` or `192.168.0.1`), the Settings editor flags a name with a backslash, and a restart is needed after changing it.
  7. A file operation against a host that is not allowed fails with an error that ends "Please update the 'security.allowedUNCHosts' setting if you want to allow this host."
  8. Changing `security.restrictUNCAccess` raises the restart prompt described in NEW-G-9.

### NEW-G-9 Restart prompt after changing a restart-only setting
- Group: DL-1
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
- Pieces:
  1. After a restart-only setting changes, a confirm box says "A setting has changed that requires a restart to take effect." with the detail "Press the restart button to restart {product} and enable the setting." and a Restart button next to Cancel.
  2. Choosing Restart restarts the app, and cancelling keeps the new value but it only takes effect at the next start.
  3. The box appears only while the window has focus, only when the value really differs from the one in use, and not when just a default changed or while settings sync is being turned on.
  4. In the desktop app stock watches `window.titleBarStyle`, `window.menuStyle` (Windows and Linux), `window.nativeTabs`, `window.nativeFullScreen`, `window.clickThroughInactive` (macOS), `window.controlsStyle` (Windows and Linux), `editor.accessibilitySupport` (Linux, when turned on), `security.workspace.trust.enabled`, `security.restrictUNCAccess`, `accessibility.verbosity.debug`, `telemetry.feedback.enabled`, `workbench.enableExperiments` and, on builds that are not stable, `_extensionsGallery.enablePPE`.
  5. Setting `workbench.enableExperiments` (on by default, tagged as using an online service) raises the box when it is switched, and that prompt is its only visible effect in the app.
  6. Cursor watches `update.mode`, `workbench.activityBar.orientation` and `application.experimental.extensionMonitor.enabled` as well and does not watch `window.menuStyle` or `telemetry.feedback.enabled`. [cursor only]

### NEW-G-10 Font smoothing on macOS
- Group: WB-5
- Tag: both
- Evidence: source
- Boundary: no
- What: On macOS the person picks how text is smoothed across the whole workbench: sub-pixel, pixel level, none, or chosen by display density.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.fontAliasing
- Cursor: present - the setting is registered with the same four values and nls 4234 holds its description; Cursor's bundle also names a general setting cursor.general.fontSmoothingAntialiased_v2 whose screen was not read
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (lines 703-716); V\src\vs\workbench\browser\workbench.ts (lines 270-295); V\src\vs\workbench\browser\media\style.css (lines 178-193); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (line 133); C bundle and C css (setting registration and the monaco-font-aliasing rules)
- Pieces:
  1. On macOS the Settings editor lists Font Aliasing (`workbench.fontAliasing`) under Workbench > Appearance, and it is not listed on Windows or Linux.
  2. Value `default` uses sub-pixel font smoothing, which gives the sharpest text on most non-retina displays.
  3. Value `antialiased` smooths text at the pixel level, which can make it look lighter overall.
  4. Value `none` turns font smoothing off, so text shows with jagged edges.
  5. Value `auto` uses antialiased smoothing on high-density displays (2x or more) and the default smoothing otherwise.
  6. A change applies to all workbench text at once, without a restart.
### NEW-G-11 Paste or drop files into a Markdown file
- Group: ED-2
- Tag: both
- Evidence: source
- Boundary: no
- What: Pasting a file or an image, or Shift+dropping a file, into a Markdown document inserts a link, image or media tag for it and can first copy the file from outside the workspace into the workspace.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: markdown.editor.drop.enabled; markdown.editor.drop.copyIntoWorkspace; markdown.editor.filePaste.enabled; markdown.editor.filePaste.copyIntoWorkspace; markdown.editor.filePaste.audioSnippet; markdown.editor.filePaste.videoSnippet; markdown.copyFiles.destination; markdown.copyFiles.overwriteBehavior
- Cursor: present - all eight settings and the edit labels "Insert Markdown Media" and "Insert Markdown Link" are in Cursor's copy of the extension (manifest and dist)
- Source: V\extensions\markdown-language-features\package.json; V\extensions\markdown-language-features\package.nls.json; V\extensions\markdown-language-features\src\languageFeatures\copyFiles\dropOrPasteResource.ts (lines 195-270); V\extensions\markdown-language-features\src\languageFeatures\copyFiles\copyFiles.ts (lines 15-80); V\extensions\markdown-language-features\src\languageFeatures\copyFiles\newFilePathGenerator.ts; V\extensions\markdown-language-features\src\languageFeatures\copyFiles\shared.ts (lines 26-90); C\extensions\markdown-language-features\package.json and dist\extension.js; scope note - the earlier ED-10 and ED-2 passes left Markdown source editing out because the scope says Markdown preview, so drop this block if that stands
- Pieces:
  1. With `editor.dropIntoEditor.enabled` on, holding Shift while dropping a file onto an open Markdown document inserts a Markdown link or image to it instead of opening the file, and `markdown.editor.drop.enabled` chooses `always`, `smart` (the default) or `never`.
  2. With `editor.pasteAs.enabled` on, pasting a copied file or a clipboard image inserts a link or image, and `markdown.editor.filePaste.enabled` chooses `always`, `smart` (the default) or `never`.
  3. The paste or drop widget names its choice by what goes in, such as "Insert Markdown Image", "Insert Markdown Link", "Insert Markdown Audio", "Insert Markdown Video" or "Insert Markdown Media", with plural and combined labels when several files are dropped.
  4. Settings `markdown.editor.drop.copyIntoWorkspace` and `markdown.editor.filePaste.copyIntoWorkspace` (`mediaFiles` by default, or `never`) copy image, audio and video files from outside the workspace into it before the link is inserted, while files already in the workspace are linked in place and an untitled document gets no copy.
  5. Setting `markdown.copyFiles.destination` maps globs that match the Markdown document path to a destination path that can use variables such as `${documentDirName}`, `${documentWorkspaceFolder}`, `${fileName}`, `${fileExtName}`, `${unixTime}` and `${isoTime}`, and a document that matches no glob gets the file next to it under its own name.
  6. Setting `markdown.copyFiles.overwriteBehavior` chooses `nameIncrementally` (the default, which adds -1, -2 and so on to the name) or `overwrite` when the target file already exists.
  7. Settings `markdown.editor.filePaste.videoSnippet` and `markdown.editor.filePaste.audioSnippet` set the text inserted for video and audio files, using `${src}` and `${title}`, with a default of a `video` or `audio` tag with controls.

### NEW-G-12 Update Markdown links when a file moves
- Group: SB-2
- Tag: both
- Evidence: source
- Boundary: no
- What: After a file or folder is renamed or moved in the workspace, the Markdown extension can rewrite the links that point at it, after asking or automatically.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: markdown.updateLinksOnFileMove.enabled; markdown.updateLinksOnFileMove.include; markdown.updateLinksOnFileMove.enableForDirectories
- Cursor: present - the three settings and the texts "Update Markdown links for" and "Checking for Markdown links to update" are in Cursor's copy of the extension
- Source: V\extensions\markdown-language-features\src\languageFeatures\linkUpdater.ts (lines 20-210); V\extensions\markdown-language-features\package.json; V\extensions\markdown-language-features\package.nls.json; C\extensions\markdown-language-features\package.json and dist\extension.js; scope note - the earlier ED-10 and ED-2 passes left Markdown source editing out because the scope says Markdown preview, so drop this block if that stands
- Pieces:
  1. After a file that matches the include globs is renamed or moved, a progress item "Checking for Markdown links to update" shows in the window while the extension looks for links that point at it.
  2. Setting `markdown.updateLinksOnFileMove.enabled` chooses `never` (the default), `prompt` or `always` for whether the links that were found are rewritten.
  3. With `prompt` a modal box asks "Update Markdown links for '{name}'?", or lists up to 10 files followed by "...N additional files not shown", with No, Yes, Always and Never buttons.
  4. Always and Never write the choice back into `markdown.updateLinksOnFileMove.enabled`, while Yes and No only decide that one time.
  5. Setting `markdown.updateLinksOnFileMove.include` lists the globs of moved files that start the check, by default Markdown files and common image and video files.
  6. Setting `markdown.updateLinksOnFileMove.enableForDirectories` (on by default) also checks links when a folder is moved or renamed.
  7. When the answer is yes, the links are changed in the Markdown files that contain them.

### NEW-G-13 Markdown link problems
- Group: ED-4
- Tag: both
- Evidence: source
- Boundary: no
- What: When link validation is on, Markdown files get squiggles and Problems entries for broken reference, header and file links and for unused or duplicate link definitions, and a quick fix can exclude a link.
- Commands: _markdown.addToIgnoreLinks; _markdown.toggleValidation
- Menus: none found
- Keys: none found
- Settings: markdown.validate.enabled; markdown.validate.referenceLinks.enabled; markdown.validate.fragmentLinks.enabled; markdown.validate.fileLinks.enabled; markdown.validate.fileLinks.markdownFragmentLinks; markdown.validate.unusedLinkDefinitions.enabled; markdown.validate.duplicateLinkDefinitions.enabled; markdown.validate.ignoredLinks
- Cursor: present - the eight settings, the quick fix text "Exclude '{0}' from link validation", the command id _markdown.addToIgnoreLinks and the code link.no-such-file are in Cursor's copy of the extension
- Source: V\extensions\markdown-language-features\package.json; V\extensions\markdown-language-features\package.nls.json; V\extensions\markdown-language-features\src\languageFeatures\diagnostics.ts (lines 12-125); C\extensions\markdown-language-features\package.json, dist\extension.js and dist\serverWorkerMain.js; scope note - the earlier ED-10 and ED-2 passes left Markdown source editing out because the scope says Markdown preview, so drop this block if that stands
- Pieces:
  1. Setting `markdown.validate.enabled` (off by default) turns link checking on for Markdown files, and the Markdown row in the language status list flips it (SS-3.53).
  2. A reference link such as `[link][ref]` with no definition is reported, with its severity set by `markdown.validate.referenceLinks.enabled` (`ignore`, `warning` by default, or `error`).
  3. A header link inside the same file such as `[link](#header)` with no matching header is reported, with its severity set by `markdown.validate.fragmentLinks.enabled` (`ignore`, `warning` by default, or `error`).
  4. A link to another file that does not exist is reported, with its severity set by `markdown.validate.fileLinks.enabled` (`ignore`, `warning` by default, or `error`).
  5. The header part of a link to another Markdown file such as `file.md#header` is checked at the level of `markdown.validate.fileLinks.markdownFragmentLinks` (`inherit` by default, which reuses the same-file header level, or `ignore`, `warning` or `error`).
  6. Link definitions that nothing uses are reported at the level of `markdown.validate.unusedLinkDefinitions.enabled` (`ignore`, `hint` by default, `warning` or `error`), and definitions made twice at the level of `markdown.validate.duplicateLinkDefinitions.enabled` (`ignore`, `warning` by default or `error`).
  7. Setting `markdown.validate.ignoredLinks` lists links or globs, for example `/about` or `/assets/**/*.svg`, that are never checked.
  8. On a link problem the quick fix list offers "Exclude '{link}' from link validation", which adds that link to `markdown.validate.ignoredLinks` in the workspace folder settings.
