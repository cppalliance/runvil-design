Group: NT-6 Cursor update and what's-new cards. Pass: 3. Date: 2026-10-25.

### NT-6.1
1. A floating card shows the small heading "Update to v{major.minor}", using the version of the ready update. [cursor only]
2. Under the heading the card shows a one-line blurb about that version.
3. The card footer has a Changelog link button and a pulsing Install Now button.
4. The card has a close X.
5. The card shows only when the target version has a built-in blurb or the force flag is set, and otherwise the compact "New update available" card shows instead.

### NT-6.2
1. The compact card shows a gift icon with the text "New update available". [cursor only]
2. The compact card has a Later button and an Install Now button.
3. The compact card shows instead of the "Update to v" card when the version has no built-in blurb.

### NT-6.3
1. After a major or minor version change, a small card titled "New in {major.minor}" appears with a one-line blurb. [cursor only]
2. The card has a View changelog button and a close X.
3. The card shows only when the new version has a built-in blurb, and otherwise nothing appears.
4. A version change that is not a major or minor change shows no card.

### NT-6.4
1. Clicking Install Now on either update card runs Attempt Update (`cursor.doupdate`) with the source set to notification, and hides the card. [cursor only]
2. Attempt Update checks for updates (`update.checkForUpdate`), downloads (`update.downloadUpdate`), installs (`update.installUpdate`) or restarts to update (`update.restartToUpdate`), whichever fits the current update state.
3. When an update step is already running, Attempt Update says "attemptUpdate: update already in progress".
4. Attempt Update is listed in the Command Palette and runs the same flow without a card.

### NT-6.5
1. Clicking Later on the compact card hides the card and snoozes it, so it does not come back until the snooze ends. [cursor only]
2. Clicking the close X on the update card hides the card and snoozes it in the same way.
3. The snooze length in hours comes from the experiment value `snooze_duration_hours`, is limited to between 1 and 240 hours, and falls back to 24 when the value is missing.
4. Clicking the close X on the changelog card hides that card for good.

### NT-6.6
1. Clicking Changelog on the update card opens https://changelog.cursor.com in the browser and leaves the card showing. [cursor only]
2. Clicking View changelog on the changelog card opens the same page and hides the card.
3. On nightly builds both buttons open the page with `?nightly=true` added.

### NT-6.7
1. Every 20 seconds Cursor checks the update state in the background and shows an update card only when an update is ready to install. [cursor only]
2. Setting `update.enableWindowsBackgroundUpdates` turns Windows background download and install on or off, and with it on and a system-scope install the card shows once the update is downloaded instead of ready.
3. Only a main window shows an update card.
4. Cards are rate limited, and the fallback config allows at least 48 hours between cards, 3 cards per version and 1 card per day.
5. The first time a new version runs, the snooze and prompt timing from the older version are cleared.
6. When the update is no longer ready, a card that was showing goes away.
7. Setting `update.mode` chooses how updates are fetched, with values `none`, `manual`, `start`, `default` and `silentlyApplyOnQuit`.
8. With `update.mode` set to `silentlyApplyOnQuit`, no card shows and the update installs without a prompt.
9. The hidden command `cursor.checkonupdate` runs one such check on demand and is not listed in the Command Palette.

### NT-6.8
1. The update card and the compact card float 8 px in from the window edge, above other UI. [cursor only]
2. The cards sit 30 px above the bottom edge when the status bar is visible and 16 px above it when the status bar is hidden.
3. The update card and the compact card are 342 px wide with a glass look.
4. The changelog card is 340 px wide.
5. Setting `workbench.sideBar.location` chooses which edge the cards sit on, `left` or `right`, following the side bar.

### NT-6.9
1. When an update is available for manual download, a notification "There is an available update." appears. [cursor only]
2. That notification has the buttons Download Update (`update.downloadUpdate`), Later and Release Notes.
3. On Linux package installs the notification says "A newer {0} package ({1}) is available via {2}." with the buttons Update, Later and Release Notes.
4. Clicking Release Notes (`update.showCurrentReleaseNotes`) opens the release notes web page in the browser.
5. Clicking Later dismisses the notification without starting a download.

### NT-6.10
1. After a restart that did not finish an update, a warning says "{0} couldn't finish installing the update to version {1}." with a Try Again button. [cursor only]
2. Clicking Try Again starts the update again.
3. When a retry is already running, an info notice says the update is downloading again, and the warning is not shown.

### NT-6.11
1. On Windows, running the user-scope install as Administrator shows "Updates are disabled because you are running the user-scope installation of {0} as Administrator." [cursor only]
2. The notice has a Learn More button that opens the downloads page in the browser.
3. The notice has a Don't Show Again choice that stops the notice from coming back.

### NT-6.12
1. When the updater ends in an error, an error notification from the source "Update Service" shows the error text. [cursor only]
2. Request timeouts and lost network connections show no error notification.
3. A Squirrel error on macOS is replaced by a note to quit Cursor, move it to Applications and open it again.

### NT-6.13
1. After a major or minor update, a sticky card titled "New in {version}" appears near the top right of the window. [stock only]
2. The card shows a banner at the top, a gradient by default or an image from the update info, and a close button.
3. The card can show a small badge above the title.
4. The card body shows a feature list, or markdown text when there is no feature list, and links in the text open when clicked.
5. The card has the buttons from the update info, and by default a single Release Notes button that runs `update.showCurrentReleaseNotes`.
6. Pressing `Escape` closes the card, and keyboard focus stays inside it while it is open.
7. Running "Show Update Info" (`update.showUpdateInfo`) from the Command Palette previews the card.
8. Setting `update.showPostInstallInfo`, off by default, shows this card after an update instead of opening the release notes editor.
9. When the update info cannot be fetched, no card appears.

### NT-6.14
1. After a major or minor update, VS Code opens the release notes editor, and if it cannot, an info prompt asks "Welcome to {0} v{1}! Would you like to read the Release Notes?" [stock only]
2. The prompt has a Release Notes button that opens the release notes web page in the browser. [stock only]
3. Setting `update.showReleaseNotes`, on by default, turns the release notes after an update on or off, and off shows neither the editor nor the prompt. [stock only]
4. With `update.showPostInstallInfo` on, neither the release notes editor nor this prompt shows after an update, because the post-update card takes over. [stock only]

### NT-6.15
1. A sticky warning notification says "Cursor is using a lot of memory ({N} MB). We can capture a snapshot for debugging, and attempt to free up some memory in the process." with a Capture Snapshot button. [cursor only]
2. The warning shows when the JS heap passes 80 percent of the V8 limit, or 50 percent for the live heap.
3. The warning shows at most once per session.
4. The warning is urgent, so it shows even when notifications are filtered to errors only.
5. Memory is checked every 10 seconds, and the warning only shows while the experiment flag `show_user_toast` is true, which it is in the fallback config.
6. Clicking Capture Snapshot starts the snapshot capture described in NT-6.16.

### NT-6.16
1. Capture Snapshot shows a progress notification "Capturing snapshot. The window may briefly freeze." with no Cancel button. [cursor only]
2. When the capture ends, a confirm box "Snapshot saved." offers Send to Cursor and No thanks.
3. In privacy mode the confirm box has only an OK button.
4. Clicking Send to Cursor shows a progress notification "Uploading snapshot to Cursor..." that cannot be cancelled, followed by a notification with the upload result.
5. When the snapshot stays on disk instead of being uploaded, a prompt says "Heap snapshot saved to disk." with a Reveal in File Explorer button that opens the file's folder.
6. The palette command "Developer: Capture and Upload Renderer Heap Snapshot" (`developer.captureAndUploadRendererHeapSnapshot`) runs the same capture and upload directly.

## Unclear
- The snooze length is read from a server experiment value. The code falls back to 24 hours and clamps to 1 to 240, while the bundled config fallback is 72, so the real value on a given account is unknown.
- Which update card appears is inferred. The built-in blurb table has no entry for version 3.27, so the compact card is the likely one, and the "Update to v" card and the changelog card need a blurb or the force flag. The pieces describe all three as the source shows them.
- NT-6.5 piece 4 says the changelog card X hides that card for good, as the feature block does, but no storage key for it was confirmed, so how long "for good" lasts is unknown.
- The side the changelog card sits on is not confirmed, since it has no class rule and only an inline bottom offset. NT-6.8 piece 5 follows the side bar side for the update and compact cards only.
- NT-6.7 piece 8 follows the code, which starts the install as soon as the update is ready. The "installs on quit" timing in the feature block was not confirmed.
- NT-6.4 piece 2 does not name which update state runs which step, because the feature block only says "depending on the update state".
- The Private Inference download prompt that Install Now can open is inference related and left out.
- Nothing was run on screen. All pieces come from source and bundle strings.
