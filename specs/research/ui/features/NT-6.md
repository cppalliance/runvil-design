Group: NT-6 Cursor update and what's-new cards. Section: Notifications. Date: 2026-10-25. Pass: 2.
Features: 16.

### NT-6.1 "Update to v" card
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A floating card with the eyebrow "Update to v{major.minor}", a one-line blurb, a Changelog link button, a pulsing Install Now button and a close X, shown when the target version has a built-in blurb or the force flag is set.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component Zyk mounted by k6m from the side bar part, classes update-notification-container, update-notification-eyebrow, update-notification-title, update-notification-footer; blurb table lut; css update-notification-*)

### NT-6.2 "New update available" card
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The default compact card when the version has no blurb: a gift icon, the text "New update available", a Later button and an Install Now button.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component Zyk fallback branch, classes minor-version-notification-container, minor-version-notification-text, minor-version-notification-buttons; css minor-version-notification-*)

### NT-6.3 "New in {version}" changelog card
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: After a major or minor version change a small card says "New in {major.minor}" with a one-line blurb, a View changelog button and a close X, shown only when the version has a blurb.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component Qyk, class changelog-notification, setShouldShowReleaseNotes in the startup class jln that compares releaseNotes/lastVersion with the current version, blurb table lut)

### NT-6.4 Install Now button
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Install Now on either update card runs Attempt Update with the source notification and hides the card. Attempt Update checks, downloads, installs or restarts depending on the update state, and says "attemptUpdate: update already in progress" when it is busy.
- Commands: cursor.doupdate; update.checkForUpdate; update.downloadUpdate; update.installUpdate; update.restartToUpdate
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (executeCommand cursor.doupdate with notification, command cursor.doupdate titled Attempt Update, ids update.restartToUpdate and update.showCurrentReleaseNotes, module vs/workbench/contrib/update/browser/update.contribution)

### NT-6.5 Snooze with Later or the close X
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Later on the compact card and the X on the update card hide the card and snooze it for the configured number of hours (clamped to 1 to 240), and the X on the changelog card hides that card for good.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (handler y in component Zyk, setSnooze and snoozedUntilTime, experiment config update_prompt_config key snooze_duration_hours, storage keys updatePromptSnoozedUntilTime and lastUpdateHiddenVersion)

### NT-6.6 Changelog link buttons
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The Changelog button on the update card and the View changelog button on the changelog card open https://changelog.cursor.com in the browser (nightly builds add ?nightly=true), and only the second also hides its card.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (components Zyk and Qyk, url changelog.cursor.com, the nightly check on productService.version)
### NT-6.7 When a card shows, and the rate limits
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A check runs every 20 seconds and shows a card only when an update is ready to install, the window is a main window, and the prompt limits allow it (fallback config: at least 48 hours apart, 3 per version, 1 per day), while update.mode silentlyApplyOnQuit skips the card and installs on quit.
- Commands: cursor.checkonupdate
- Menus: none found
- Keys: none found
- Settings: update.mode; update.enableWindowsBackgroundUpdates; update.channel
- Cursor: cursor only
- Source: C bundle (command cursor.checkonupdate with f1 false, class UpdateCheckCronContribution with task update-check-task at 20000 ms, class zgr updateNotificationService with shouldShowPrompt and recordPromptShown, fallbackValues of update_prompt_config, setting update.mode, telemetry cursor.update.promptShown)

### NT-6.8 Card placement and look
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The update and compact cards float 8 px from the window edge on the side bar side (left when the side bar is on the left, else right), 30 px above the bottom with the status bar visible or 16 px without, in a glass style 342 px wide, and the changelog card is 340 px wide.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.sideBar.location
- Cursor: cursor only
- Source: C bundle css (update-notification-outer-container with position fixed and z-index 2000, update-notification-outer-container-left, update-notification-outer-container-right, update-notification-container, minor-version-notification-container), components Zyk and Qyk

### NT-6.9 Classic "update available" prompts
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: When an update is available for manual download Cursor shows a notification "There is an available update." with Download Update, Later and Release Notes, or on Linux package installs "A newer {0} package ({1}) is available via {2}." with Update, Later and Release Notes. Stock 1.128 no longer has them.
- Commands: update.downloadUpdate; update.showCurrentReleaseNotes
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/contrib/update/browser/update, methods onUpdateAvailable and onPackageUpdateAvailable, nls 11503 to 11510); V\src\vs\workbench\contrib\update\browser\update.ts (no such prompts in 1.128.0)

### NT-6.10 Update did not install prompt
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: After a restart that did not finish an update, a warning says "{0} couldn't finish installing the update to version {1}." with a Try Again button, or an info note says it is downloading the update again when a retry is already running.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/contrib/update/browser/update, method maybeShowLastInstallFailedNotification, nls 11492, 11493, 11494)

### NT-6.11 Updates disabled as Administrator notice
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: On Windows, running the user-scope install as Administrator shows "Updates are disabled because you are running the user-scope installation of {0} as Administrator." with Learn More (opens the downloads page) and a Don't Show Again choice.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/contrib/update/browser/update, disabled state with reason 5, neverShowAgain id no-updates-running-as-admin, nls 11495, 11496); V\src\vs\workbench\contrib\update\browser\updateTooltip.ts (stock shows this state only in the title bar tooltip)

### NT-6.12 Update error notice
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: When the updater ends in an error, Cursor shows an error notification from the source "Update Service" with the error text, except for request timeouts and lost network connections, and a Mac Squirrel error is replaced by a note to quit, move Cursor to Applications and relaunch.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/contrib/update/browser/update, method onError, nls 11501)
### NT-6.13 Post-update widget (stock)
- Tag: vscode
- Evidence: source
- Boundary: no
- What: After a major or minor VS Code update a sticky card titled "New in {version}" appears near the top right of the window with a banner, a close button, an optional badge, a feature list or markdown, and buttons (by default a single Release Notes button), Escape closes it, and Show Update Info from the palette previews it.
- Commands: _update.showUpdateInfo; update.showUpdateInfo; update.showCurrentReleaseNotes
- Menus: CommandPalette
- Keys: none found
- Settings: update.showPostInstallInfo
- Cursor: missing
- Source: V\src\vs\workbench\contrib\update\browser\postUpdateWidget.ts; V\src\vs\workbench\contrib\update\browser\update.contribution.ts; V\src\vs\platform\update\common\update.config.contribution.ts; V\src\vs\workbench\contrib\update\common\updateInfoParser.ts; V\src\vs\workbench\contrib\update\common\updateUtils.ts; V\src\vs\workbench\contrib\update\browser\media\postUpdateWidget.css; C bundle (searched _update.showUpdateInfo, update.showUpdateInfo, postUpdateWidget, update.showPostInstallInfo: no hits)

### NT-6.14 Release notes fallback prompt (stock)
- Tag: vscode
- Evidence: source
- Boundary: no
- What: After a major or minor update, if the release notes editor cannot open, VS Code shows an info prompt "Welcome to {0} v{1}! Would you like to read the Release Notes?" with a Release Notes button that opens the notes web page.
- Commands: update.showCurrentReleaseNotes
- Menus: none found
- Keys: none found
- Settings: update.showReleaseNotes; update.showPostInstallInfo
- Cursor: missing
- Source: V\src\vs\workbench\contrib\update\browser\update.ts (class ProductContribution); V\src\vs\platform\update\common\update.config.contribution.ts; C bundle (label "Would you like to read the Release Notes" and setting update.showReleaseNotes not found in the label table or the bundle)

### NT-6.15 Memory warning prompt
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A sticky warning notification says "Cursor is using a lot of memory ({N} MB). We can capture a snapshot for debugging, and attempt to free up some memory in the process." with a Capture Snapshot button, shown once per session when the JS heap passes 80 percent of the V8 limit (or 50 percent for live heap).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/contrib/memoryMonitor/electron-sandbox/memoryMonitor.contribution, method _checkMemoryUsage with prompt priority Urgent and sticky, telemetry renderer.memoryMonitor.userToast, experiment config memory_monitor_config with fallback show_user_toast true, user_toast_heap_percent 80, user_toast_live_heap_percent 50, check interval 10 seconds)

### NT-6.16 Capture Snapshot flow
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Capture Snapshot runs a non-cancellable progress notification "Capturing snapshot. The window may briefly freeze.", then a "Snapshot saved." confirm with Send to Cursor or No thanks (or OK in privacy mode), and ends with an upload progress notification and result, or a prompt "Heap snapshot saved to disk." with Reveal in File Explorer, and the Developer palette command runs the same capture and upload directly.
- Commands: developer.captureAndUploadRendererHeapSnapshot; memoryMonitor.triggerManualSnapshot
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/contrib/memoryMonitor/electron-sandbox/memoryMonitor.contribution, methods _captureUserSnapshot, _captureSnapshotToDisk, _uploadSavedSnapshot, _showRevealNotification, command developer.captureAndUploadRendererHeapSnapshot, nls 7770)

## Handed off
- The title bar Update button (class ide-titlebar-update-button, labels Update, Installing, Restart to Update, experiment ide_update_ux_exp): TB-5
- Palette commands Check for Update, Download Update, Install Update and Restart to Update (present in stock and Cursor) and the Help menu rows for them: TB-1
- Show Release Notes (update.showCurrentReleaseNotes, Help menu row): TB-1. Cursor opens the notes web page in the browser, stock opens a release notes editor, which has no owner and could go to ED-10
- The update rows in the Manage menu (Check for Updates..., Download Update (1), Install Update... (1), Restart to Update (1)) and the Manage gear badge for checking, downloading, updating and a ready update: AB-4
- The status bar memory entry (HIGH MEMORY USAGE, id workbench.memoryMonitor.indicator, off by default): SS-6
- The "There are currently no updates available." info box after an explicit check, and the "Snapshot saved." confirm box: DL-1

## Unclear
- The catalog says "snoozed up to 24 hours". The card handler uses the experiment value snooze_duration_hours, falls back to 24 in code if it is missing, and clamps to 1 to 240 hours, but the config fallback in the bundle is 72. The real value comes from the server.
- Which variant appears is inferred. The blurb table lut has keys 0.46, 0.47, 0.48, 0.49, 0.50 and 1.0 only, so for version 3.27 I expect the compact "New update available" card and no changelog card, unless forceUpdate is true. forceUpdate is stored under updatePromptForceUpdate and the main bundle only ever sets it to false, so something else (maybe the Agents window bundle, not read) sets it. The side bar agent reported seeing "Update to v..." cards, which I could not reproduce.
- Pass 1 named the changelog card button Changelog. The code label there is View changelog. Changelog is the label on the update card.
- The changelog card has no class rule in the css and only a bottom offset in inline style, so I could not confirm which side it sits on.
- The cards have test switches (isTestingUpdateButton, isTestingChangelogComponent) in the UI overlay service, but I found no command in the main bundle that calls the toggles.
- Install Now ends in update.restartToUpdate with source notification, which can open a Private Inference download prompt when feature gate cursor_private_inference_download_prompt is on. That prompt is inference related and left out.
- Developer commands cursor.useLocalhostUpdateServer and the internal-only "[DEV ONLY]" memory dialog (shown only when show_internal_warning_popup is on, off by default) were seen and not listed.
- onUpdateDownloaded and onUpdateReady in Cursor's update class have empty bodies, so I treated only the available-for-download prompts as live.
- Newer than 1.128 (seen only in N): the release notes editor gained feature "tryouts" links (releaseNotesTryouts.ts) and an accessibility help provider. Not listed because no group owns the release notes editor.
- Everything here was read from code and strings. I did not see any card or prompt on screen.

