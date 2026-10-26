Group: PN-7 Context-only panel views. Pass: 3. Date: 2026-10-25.

### PN-7.1
1. The Comments tab lists every comment thread in the workspace as rows nested under a row for the file they belong to.
2. Each thread row starts with a state icon that shows whether the thread is a plain comment, unresolved or a draft.
3. Each thread row shows the author, the comment body, the line range such as `[Ln 3-5]` and the time.
4. A thread with replies shows a second line with the reply count such as "2 replies", "Last reply from" the latest author, and the time of that reply.
5. A thread whose code has changed shows an "Outdated" tag on its row.
6. Setting `comments.useRelativeTime`, on by default, chooses whether comment times read as relative text such as "1 day ago" or as a full local date and time.
7. With no comments in the workspace the view shows "There are no comments in this workspace yet."
8. The Comments tab only appears in the panel once an extension supplies comments.
9. Running `workbench.action.focusCommentsPanel` brings up the Comments view with keyboard focus in it.

### PN-7.2
1. Clicking a thread row opens its file in the editor and reveals the thread there.
2. Pressing Enter on a thread row opens its file and reveals the thread.
3. Clicking or pressing Enter on a file row opens that file in the editor.
4. Using the usual open-to-the-side gesture on a row opens the file in an editor beside the current one.
5. Using the keep-preview gesture on a row makes the opened editor tab permanent instead of a preview tab.

### PN-7.3
1. A filter box in the Comments header with the placeholder "Filter (e.g. text, author)" hides threads that do not match the typed text.
2. The filter matches the file name, the comment text, the author and the replies.
3. Starting the filter text with `!` hides the threads that match instead of keeping them.
4. A badge in the filter box says "Showing X of Y" while the filter hides some threads.
5. The filter text and its history are kept per workspace.
6. Pressing Ctrl+F while the Comments view is focused moves focus into the filter box (`commentsFocusFilter`).
7. Pressing Ctrl+Down Arrow in the filter box moves focus to the list (`commentsFocusViewFromFilter`).
8. Pressing Escape in the filter box clears its text (`commentsClearFilterText`).

### PN-7.4
1. The filter drop-down next to the filter box has Show Resolved and Show Unresolved items, both checked at first.
2. Unchecking Show Resolved (`workbench.actions.workbench.panel.comments.toggleResolvedComments`) hides resolved threads.
3. Unchecking Show Unresolved (`workbench.actions.workbench.panel.comments.toggleUnResolvedComments`) hides unresolved threads.
4. The two choices are saved per workspace.

### PN-7.5
1. A Sort By submenu in the filter drop-down offers Position in File and Updated Time.
2. Position in File, the default, orders threads by where they sit in their files.
3. Updated Time orders threads with the most recently updated first.
4. The active sort shows a check mark in the Sort By submenu.

### PN-7.6
1. Clicking the Expand All icon in the Comments header (`comments.expand`) opens every thread row to show its replies.
2. Clicking the Collapse All icon in the Comments header (`comments.collapse`) closes every thread row.
3. Only one of the two icons shows at a time, with Collapse All showing while any thread is expanded.
4. Neither icon shows while the view has no comments.

### PN-7.7
1. A Reply button on a thread row opens the thread in the editor with the reply box focused (`comments.reply`).
2. The Reply button is enabled only when the thread allows replies.
3. The accessible view of a comment thread also offers the Reply action.

### PN-7.8
1. The Comments tab carries a number badge with how many threads are unresolved.
2. Hovering the badge shows the tooltip "N Unresolved Comments".
3. The badge number changes as threads are added, resolved or deleted.

### PN-7.9
1. Setting `comments.openView` chooses when the Comments view opens by itself, with values `never`, `file`, `firstFile` and `firstFileUnresolved`, default `firstFile`.
2. With `never`, the Comments view is never opened by itself.
3. With `file`, the Comments view opens whenever a file with comments becomes the active file.
4. With `firstFile`, the Comments view opens the first time in a session that a file with comments is active, and not again.
5. With `firstFileUnresolved`, the Comments view opens the first time in a session that a file with an unresolved comment is active.
6. The older setting `comments.openPanel`, with values `neverOpen`, `openOnSessionStart` and `openOnSessionStartWithComments`, is marked deprecated in favor of `comments.openView`.

### PN-7.10
1. The Ports tab lists forwarded and detected ports in a table with the columns Port, Forwarded Address, Running Process and Origin.
2. When the tunnel provider can change visibility, a Visibility column shows between Running Process and Origin.
3. With nothing forwarded in a remote window, the view shows "No forwarded ports. Forward a port to access your running services locally." and a Forward a Port link.
4. With nothing forwarded in a local window, the view shows "No forwarded ports. Forward a port to access your locally running services over the internet." and a Forward a Port link.
5. Hovering a port row shows a tooltip saying which remote port is forwarded to which local address, or that the remote port is not forwarded.
6. Hovering a column header shows a tooltip describing that column.
7. The Ports tab appears only when port forwarding is switched on, which is in a remote window or after a local port has been forwarded.
8. Running `~remote.forwardedPorts.focus` brings up the Ports view with keyboard focus in it.

### PN-7.11
1. An Add Port button in the add row of the Ports table prompts for a port number or address with the hint "Port number or address (eg. 3000 or 10.10.10.10:2000)." and forwards it (`remote.tunnel.forwardInline`).
2. The Forward a Port link in the empty Ports view starts the same prompt.
3. A detected port that is not yet forwarded shows a Forward Port button on its row.
4. The right-click menu in the Ports view has a Forward a Port entry.
5. Running Forward a Port (`remote.tunnel.forwardCommandPalette`) from the Command Palette starts the same prompt.
6. Typing text that is not a number or a host and port shows "Forwarded port should be a number or a host:port.", and typing a port that is already forwarded shows "Port is already forwarded".
7. When forwarding fails, a warning says "Unable to forward" the host and port, naming that the host may not be available or the port may already be forwarded.

### PN-7.12
1. A Stop Forwarding Port button on a forwarded port's row removes that port (`remote.tunnel.closeInline`).
2. The right-click menu on a forwarded port has a Stop Forwarding Port entry.
3. Pressing Delete, or Cmd+Backspace on macOS, while the Ports view is focused removes the selected port when it can be closed.
4. Running Stop Forwarding Port (`remote.tunnel.closeCommandPalette`) from the Command Palette opens a list titled "Choose a port to stop forwarding" and removes the chosen port.
5. With no forwarded ports, the Command Palette list says "No ports currently forwarded. Try running the Forward a Port command".

### PN-7.13
1. An Open in Browser button on a forwarded address opens that address in your default browser (`remote.tunnel.open`).
2. A Preview in Editor button on a forwarded address shows that address in an editor tab (`remote.tunnel.openPreview`).
3. The right-click menu on a forwarded port has Open in Browser and Preview in Editor entries.
4. Running Open Port in Browser (`remote.tunnel.openCommandPalette`) from the Command Palette opens a list titled "Choose the port to open" and opens the chosen port.
5. With no forwarded ports, the Open Port in Browser list says "No ports currently forwarded. Open the Ports view to get started." and offers "Open the Ports view...".
6. The forwarded address is a link whose hover text names the click needed to follow it, such as "ctrl + click".

### PN-7.14
1. A Copy Local Address button on a forwarded address copies that address (`remote.tunnel.copyAddressInline`).
2. The right-click menu on a forwarded port has a Copy Local Address entry.
3. Pressing Ctrl+C while the Ports view is focused and one forwarded or detected port is selected copies its address.
4. Running Copy Forwarded Port Address (`remote.tunnel.copyAddressCommandPalette`) from the Command Palette opens a list titled "Choose a forwarded port" and copies the chosen port's address.

### PN-7.15
1. A Set Port Label button on a port's row opens an inline text box in the Port column with the placeholder "Port label" (`remote.tunnel.label`).
2. The right-click menu on a forwarded port has a Set Port Label entry that opens the same text box.
3. Pressing F2 (Enter on macOS) while a single forwarded port is focused opens the same text box.
4. Pressing Enter in the text box saves the name and shows it in the Port column, and pressing Escape cancels.

### PN-7.16
1. Choosing Change Local Address Port in the right-click menu of a forwarded port opens an inline text box with the placeholder "New local port" (`remote.tunnel.changeLocalPort`).
2. Typing something that is not a number shows "Local port should be a number."
3. When the chosen local port is taken, a warning says that port is not available and names the port number that was used instead.
4. The Change Local Address Port entry only shows for a forwarded port whose local port can be changed.

### PN-7.17
1. A Port Visibility submenu in the right-click menu of a forwarded port lists the visibility options the tunnel provider supports, such as Private and Public.
2. The current visibility option shows a check mark in the submenu.
3. Choosing another option changes the port's visibility (`remote.tunnel.privacy<id>`).
4. The Port Visibility submenu only shows when the tunnel provider allows changing visibility.

### PN-7.18
1. A Change Port Protocol submenu in the right-click menu of a forwarded port offers HTTP and HTTPS.
2. Choosing HTTP (`remote.tunnel.setProtocolHttp`) or HTTPS (`remote.tunnel.setProtocolHttps`) sets the forwarded port to that protocol.
3. The current protocol shows a check mark in the submenu.
4. The Change Port Protocol submenu only shows when the tunnel provider allows changing the protocol.

### PN-7.19
1. Setting `remote.autoForwardPorts`, on by default, chooses whether ports that new running processes listen on are forwarded automatically.
2. Setting `remote.autoForwardPortsSource` chooses how ports are detected, with values `process` (watch started processes), `output` (read terminal and debug output) and `hybrid` (read output and un-forward by watching processes), default `process`.
3. Setting `remote.autoForwardPortsFallback`, default 20, sets how many auto forwarded ports switch detection from `process` to `hybrid`, and 0 turns the switch off.
4. Setting `remote.forwardOnOpen`, on by default, chooses whether local URLs with a port are forwarded when opened from the terminal or the debug console.
5. Setting `remote.restoreForwardedPorts`, on by default, chooses whether ports you forwarded in a workspace come back when it reopens.
6. Setting `remote.localPortHost` chooses the local host name used for forwarding, with values `localhost` and `allInterfaces`, default `localhost`.

### PN-7.20
1. Setting `remote.portsAttributes` holds per-port rules whose keys are a port number, a range such as `40000-55000`, a host and port such as `db:1234`, or a regular expression matched against the process command line.
2. A rule's `label` sets the name shown for that port in the Ports view.
3. A rule's `onAutoForward` chooses what happens on automatic forwarding, with values `notify`, `openBrowser`, `openBrowserOnce`, `openPreview`, `silent` and `ignore`, default `notify`.
4. A rule's `elevateIfNeeded` asks for elevation when the local port needs it, and its `requireLocalPort` shows a modal dialog when the chosen local port is not used, and both are off by default.
5. A rule's `protocol` sets the protocol, with values `http` and `https`.
6. The default value of `remote.portsAttributes` sets ports 443 and 8443 to the `https` protocol.
7. Setting `remote.otherPortsAttributes` gives the same kind of defaults for all ports no rule matches, with `onAutoForward` values `notify`, `openBrowser`, `openPreview`, `silent` and `ignore`.

### PN-7.21
1. The Ports tab carries a number badge with how many ports are forwarded.
2. Hovering the badge shows the tooltip "1 forwarded port" or "N forwarded ports".
3. The badge changes as ports are forwarded or stopped, and is gone while no port is forwarded.

### PN-7.22
1. When a rename or another action asks for a preview, the Refactor Preview tab opens in the panel with a tree of the pending edits.
2. File rows in the tree mark renames as "old path → new path" with "(renaming)", creations with "(creating)" and deletions with "(deleting)".
3. Each edit row shows the removed text struck through and the added text highlighted.
4. When edits come from several sources, the tree also has category rows, and edits with no named source go under a category called "Other".
5. When you apply or discard the preview, the panel goes back to the tab you had open before.
6. With no preview pending, the view says "Invoke a code action, like rename, to see a preview of its changes here."
7. Running `refactorPreview.focus` brings up the Refactor Preview view with keyboard focus in it.

### PN-7.23
1. Each file, edit and category row has a checkbox.
2. Unchecking a row leaves that file, edit or category out of the refactoring.
3. Pressing Space on the focused row toggles its checkbox (`refactorPreview.toggleCheckedState`).
4. The right-click menu in the tree has a Toggle Change entry for the selected row.
5. With nothing checked, the Apply Refactoring command is unavailable.

### PN-7.24
1. Clicking the Apply button in the Refactor Preview view applies the checked edits to the files.
2. Choosing Apply Refactoring (`refactorPreview.apply`) in the right-click menu of the tree applies the checked edits.
3. Pressing Ctrl+Enter while a preview is active and the Refactor Preview view is focused applies the checked edits.
4. If a file changed since the preview was made, a warning says "Cannot apply refactoring because" that file "has changed in the meantime", or names how many other files changed.

### PN-7.25
1. Clicking the Discard button in the Refactor Preview view throws the preview away without changing any file.
2. Choosing Discard Refactoring (`refactorPreview.discard`) in the right-click menu of the tree throws the preview away without changing any file.
3. After the preview is discarded, the view goes back to its idle message.

### PN-7.26
1. Clicking the grouping icon in the Refactor Preview header toggles the tree between grouping by file and grouping by type (`refactorPreview.toggleGrouping`).
2. The right-click menu in the tree has Group Changes By File and Group Changes By Type entries (`refactorPreview.groupByFile`, `refactorPreview.groupByType`).
3. Grouping by file lists files with their edits under them, and grouping by type lists categories with their files under them.
4. The grouping controls only show when the edits come from more than one category.
5. The chosen grouping is remembered across sessions.

### PN-7.27
1. Opening a file or edit row in the Refactor Preview tree shows a multi-file diff tab titled "Refactor Preview".
2. The diff tab shows the before and after of every file in the refactoring.
3. The diff tab is scrolled to the edit you picked.
4. The diff tab is temporary and closes when the preview ends.

## Unclear
- PN-7.10 piece 7 and PN-7.1 piece 8 say the tabs only appear when something enables them (port forwarding, or an extension supplying comments). In Cursor with no extra extensions the Comments tab most likely never shows, so those pieces may never be checkable in a plain Cursor window.
- PN-7.11 piece 7 and PN-7.24 piece 4 are warnings that the Notifications and Dialogs groups may also count (NT-2, DL-1). They are kept here because each feature's own description names them.
- PN-7.7 piece 3 rests only on the AccessibleView menu entry in the feature block, and the accessible view itself has no owner in the catalog.

