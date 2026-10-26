Group: PN-7 Context-only panel views. Section: Panel. Date: 2026-10-25. Pass: 2.
Features: 27.

### PN-7.1 See all comment threads in the Comments view
- Tag: both
- Evidence: source
- Boundary: no
- What: The Comments tab lists every comment thread in the workspace under its file. Each row shows a state icon (comment, unresolved or draft), the author, the body, the line range, the time, the reply count, and an "Outdated" tag when it applies. The tab only exists once an extension supplies comments, and an empty list says "There are no comments in this workspace yet."
- Commands: workbench.action.focusCommentsPanel
- Menus: none found
- Keys: none found
- Settings: comments.useRelativeTime
- Cursor: present
- Source: V\src\vs\workbench\contrib\comments\browser\commentsView.ts; V\src\vs\workbench\contrib\comments\browser\commentsTreeViewer.ts; V\src\vs\workbench\contrib\comments\browser\commentsModel.ts; V\src\vs\workbench\api\browser\mainThreadComments.ts; C bundle (workbench.action.focusCommentsPanel, comments.useRelativeTime, commentsView.hasComments)

### PN-7.2 Open a comment from the list
- Tag: both
- Evidence: source
- Boundary: no
- What: Click or press Enter on a file row or a thread row to open that file and reveal the thread in the editor. The usual open-to-the-side and keep-preview gestures apply.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\comments\browser\commentsView.ts (openFile); V\src\vs\workbench\contrib\comments\browser\commentsController.ts (revealCommentThread); C bundle (CommentsViewThreadActions, workbench.panel.comments)

### PN-7.3 Filter comments by text or author
- Tag: both
- Evidence: source
- Boundary: no
- What: A filter box in the Comments header hides threads that do not match. It matches the file name, comment text, author and replies, and a leading ! excludes matches. A badge says "Showing X of Y", and the filter text and history are kept per workspace.
- Commands: commentsFocusFilter; commentsFocusViewFromFilter; commentsClearFilterText
- Menus: none found
- Keys: Ctrl+F (when the Comments view is focused); Ctrl+Down Arrow (when the filter has focus, moves to the list); Escape (when the filter has focus, clears it)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\comments\browser\commentsView.ts; V\src\vs\workbench\contrib\comments\browser\commentsViewActions.ts; V\src\vs\workbench\contrib\comments\browser\commentsFilterOptions.ts; V\src\vs\workbench\contrib\comments\browser\commentsTreeViewer.ts (Filter); C bundle (commentsFocusFilter, commentsFocusViewFromFilter, commentsClearFilterText)

### PN-7.4 Show or hide resolved and unresolved threads
- Tag: both
- Evidence: source
- Boundary: no
- What: The filter drop-down next to the box has two checked items, Show Resolved and Show Unresolved, to hide either kind of thread. The choice is saved per workspace.
- Commands: workbench.actions.workbench.panel.comments.toggleResolvedComments; workbench.actions.workbench.panel.comments.toggleUnResolvedComments
- Menus: submenu.view.filter
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\comments\browser\commentsViewActions.ts; V\src\vs\workbench\browser\parts\views\viewFilter.ts; C bundle (toggleResolvedComments, toggleUnResolvedComments, commentsView.showResolvedFilter, commentsView.showUnResolvedFilter)

### PN-7.5 Sort comments by file position or last update
- Tag: both
- Evidence: source
- Boundary: no
- What: A Sort By submenu in the filter drop-down orders the list by position in the file (default) or by most recently updated thread.
- Commands: workbench.actions.workbench.panel.comments.toggleSortByResource; workbench.actions.workbench.panel.comments.toggleSortByUpdatedAt
- Menus: submenu.filter.commentSort (inside submenu.view.filter)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\comments\browser\commentsViewActions.ts; V\src\vs\workbench\contrib\comments\browser\commentsView.ts; C bundle (toggleSortByResource, toggleSortByUpdatedAt, submenu.filter.commentSort, commentsView.sortBy)

### PN-7.6 Expand or collapse every thread
- Tag: both
- Evidence: source
- Boundary: no
- What: One header icon expands all threads, or collapses them when any is open. Only one of the two shows at a time.
- Commands: comments.expand; comments.collapse
- Menus: ViewTitle
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\comments\browser\comments.contribution.ts; V\src\vs\workbench\contrib\comments\browser\commentsView.ts; C bundle (comments.expand, comments.collapse)

### PN-7.7 Reply to a thread from the list
- Tag: both
- Evidence: source
- Boundary: no
- What: A Reply button on a thread row opens the thread in the editor with the reply box focused. It is only enabled when the thread allows replies.
- Commands: comments.reply
- Menus: CommentsViewThreadActions; AccessibleView
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\comments\browser\comments.contribution.ts; V\src\vs\workbench\contrib\comments\browser\commentsTreeViewer.ts (CommentsMenus); C bundle (comments.reply, CommentsViewThreadActions)

### PN-7.8 See the unresolved count on the Comments tab
- Tag: both
- Evidence: source
- Boundary: no
- What: The Comments tab carries a number badge with how many threads are unresolved, with the tooltip "N Unresolved Comments".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\comments\browser\comments.contribution.ts (UnresolvedCommentsBadge); C bundle (showViewActivity with the comments view id and a number badge)

### PN-7.9 Open the Comments view automatically
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose whether the Comments view opens by itself when you open a file that has comments: never, every such file, the first such file in a session, or the first with an unresolved thread. An older setting with similar choices is deprecated.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: comments.openView; comments.openPanel
- Cursor: present
- Source: V\src\vs\workbench\contrib\comments\browser\comments.contribution.ts; V\src\vs\workbench\contrib\comments\browser\commentsController.ts (openCommentsView); C bundle (comments.openView, comments.openPanel, firstFileUnresolved)

### PN-7.10 See forwarded ports in the Ports view
- Tag: both
- Evidence: source
- Boundary: no
- What: The Ports tab lists forwarded and detected ports in a table with Port, Forwarded Address, Running Process and Origin columns, plus a Visibility column when the tunnel provider can change visibility. When nothing is forwarded it shows "No forwarded ports" text with a Forward a Port link. The tab appears only when port forwarding is switched on (a remote window, or after a local port is forwarded). This is a remote feature.
- Commands: ~remote.forwardedPorts.focus
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts; V\src\vs\workbench\contrib\remote\browser\remoteExplorer.ts; V\src\vs\workbench\services\remote\common\remoteExplorerService.ts; C bundle (~remote.forwardedPortsContainer, ~remote.forwardedPorts, tunnelViewFocus, forwardedPortsViewEnabled)

### PN-7.11 Forward a port
- Tag: both
- Evidence: source
- Boundary: no
- What: Forward a port by number or address (for example 3000 or 10.10.10.10:2000) from the add row, the empty-state link, a "Forward Port" button on a detected port, the right-click menu, or the Command Palette.
- Commands: remote.tunnel.forwardInline; remote.tunnel.forwardCommandPalette
- Menus: TunnelContext; TunnelPortInline; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (ForwardPortAction); V\src\vs\workbench\contrib\remote\browser\remoteExplorer.ts; C bundle (remote.tunnel.forwardInline, remote.tunnel.forwardCommandPalette, Forward a Port)

### PN-7.12 Stop forwarding a port
- Tag: both
- Evidence: source
- Boundary: no
- What: Remove a forwarded port from its row button, the right-click menu, the Delete key, or a pick list from the Command Palette.
- Commands: remote.tunnel.closeInline; remote.tunnel.closeCommandPalette
- Menus: TunnelContext; TunnelPortInline; CommandPalette
- Keys: Delete (Cmd+Backspace on macOS; when the Ports view is focused and the port can be closed)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (ClosePortAction); C bundle (remote.tunnel.closeInline, remote.tunnel.closeCommandPalette, Stop Forwarding Port)

### PN-7.13 Open a forwarded port in the browser or in an editor tab
- Tag: both
- Evidence: source
- Boundary: no
- What: Open the forwarded address in your browser, or choose Preview in Editor to show it in an editor tab. Both are inline buttons on the address, entries in the right-click menu, and Command Palette lists.
- Commands: remote.tunnel.open; remote.tunnel.openPreview; remote.tunnel.openCommandPalette
- Menus: TunnelContext; TunnelLocalAddressInline; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (OpenPortInBrowserAction, OpenPortInPreviewAction); C bundle (remote.tunnel.open, remote.tunnel.openPreview, remote.tunnel.openCommandPalette, openPreviewEnabled)

### PN-7.14 Copy a port's local address
- Tag: both
- Evidence: source
- Boundary: no
- What: Copy the forwarded address from an inline button, the right-click menu, Ctrl+C on the row, or a pick list in the Command Palette.
- Commands: remote.tunnel.copyAddressInline; remote.tunnel.copyAddressCommandPalette
- Menus: TunnelContext; TunnelLocalAddressInline; CommandPalette
- Keys: Ctrl+C (when the Ports view is focused and one forwarded or detected port is selected)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (CopyAddressAction); C bundle (remote.tunnel.copyAddressInline, remote.tunnel.copyAddressCommandPalette)

### PN-7.15 Label a port
- Tag: both
- Evidence: source
- Boundary: no
- What: Give a forwarded port a name that shows in the Port column, from an inline button, the right-click menu or F2.
- Commands: remote.tunnel.label
- Menus: TunnelContext; TunnelPortInline
- Keys: F2 (Enter on macOS; when a single forwarded port is focused)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (LabelTunnelAction); C bundle (remote.tunnel.label)

### PN-7.16 Change the local port number
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick a different local port for a forwarded port from the right-click menu. It warns when the port is taken and another was used.
- Commands: remote.tunnel.changeLocalPort
- Menus: TunnelContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (ChangeLocalPortAction); C bundle (remote.tunnel.changeLocalPort)

### PN-7.17 Set a port's visibility
- Tag: both
- Evidence: source
- Boundary: no
- What: A Port Visibility submenu in the right-click menu switches a forwarded port between the options the tunnel provider supports, such as Private and Public, with a check next to the current one. It only shows when the provider allows it. The command ids are built per option at run time.
- Commands: remote.tunnel.privacy<id> (one per visibility option)
- Menus: TunnelContext; TunnelPrivacy
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (registerPrivacyActions); C bundle (remote.tunnel.privacy, TunnelPrivacy)

### PN-7.18 Change a port's protocol
- Tag: both
- Evidence: source
- Boundary: no
- What: A Change Port Protocol submenu in the right-click menu sets a forwarded port to HTTP or HTTPS.
- Commands: remote.tunnel.setProtocolHttp; remote.tunnel.setProtocolHttps
- Menus: TunnelContext; TunnelProtocol
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\tunnelView.ts (SetTunnelProtocolAction); C bundle (remote.tunnel.setProtocolHttp, remote.tunnel.setProtocolHttps, TunnelProtocol)

### PN-7.19 Forward ports automatically
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn on automatic forwarding of ports that running programs open, choose how they are detected (process watching, terminal and debug output, or a hybrid of the two), set when it falls back to the hybrid mode, forward local URLs opened from the terminal, restore forwarded ports in the workspace, and pick the local host name used.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: remote.autoForwardPorts; remote.autoForwardPortsSource; remote.autoForwardPortsFallback; remote.forwardOnOpen; remote.restoreForwardedPorts; remote.localPortHost
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\common\remote.contribution.ts; V\src\vs\workbench\contrib\remote\browser\remoteExplorer.ts; V\src\vs\workbench\contrib\remote\browser\urlFinder.ts; C bundle (remote.autoForwardPorts, remote.autoForwardPortsSource, remote.autoForwardPortsFallback, remote.forwardOnOpen, remote.restoreForwardedPorts, remote.localPortHost)

### PN-7.20 Set rules for specific ports
- Tag: both
- Evidence: source
- Boundary: no
- What: Write per-port rules in settings, by number, range, host and port, or process pattern: a label, what happens on automatic forwarding (notify, open browser, open preview, stay silent, ignore), whether to ask for elevation, whether a modal dialog shows when the local port differs, and the protocol. A second setting gives defaults for all other ports.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: remote.portsAttributes; remote.otherPortsAttributes
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\common\remote.contribution.ts; C bundle (remote.portsAttributes, remote.otherPortsAttributes)

### PN-7.21 See the forwarded port count on the Ports tab
- Tag: both
- Evidence: source
- Boundary: no
- What: The Ports tab carries a number badge with how many ports are forwarded, with the tooltip "1 forwarded port" or "N forwarded ports".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\remoteExplorer.ts (updateActivityBadge); C bundle (showViewActivity with the tunnel view container and tunnelModel.forwarded.size)

### PN-7.22 Review pending edits in Refactor Preview
- Tag: both
- Evidence: source
- Boundary: no
- What: When a rename or another action asks for a preview, the Refactor Preview tab opens with a tree of the pending edits: files (with renames, creations and deletions marked), each edit with the removed text struck and the added text highlighted, and categories when edits come from several sources. When you finish, the panel goes back to the tab you had before. Idle, it says "Invoke a code action, like rename, to see a preview of its changes here."
- Commands: refactorPreview.focus
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditTree.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts; C bundle (refactorPreview.enabled, workbench.contrib.bulkEditPreview, Refactor Preview, setPreviewHandler)

### PN-7.23 Check or uncheck edits
- Tag: both
- Evidence: source
- Boundary: no
- What: Each file, edit and category has a checkbox so you can leave out parts of the refactoring. Space toggles the focused row. With nothing checked, the Apply Refactoring command is unavailable.
- Commands: refactorPreview.toggleCheckedState
- Menus: BulkEditContext
- Keys: Space (when a list has focus and a preview is active)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts; C bundle (refactorPreview.toggleCheckedState, refactorPreview.hasCategories)

### PN-7.24 Apply the refactoring
- Tag: both
- Evidence: source
- Boundary: no
- What: Apply the checked edits from the Apply button, the right-click menu or Ctrl+Enter. If a file changed in the meantime a warning says the refactoring cannot be applied.
- Commands: refactorPreview.apply
- Menus: BulkEditContext
- Keys: Ctrl+Enter (when a preview is active and the Refactor Preview view is focused)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts; C bundle (refactorPreview.apply)

### PN-7.25 Discard the refactoring
- Tag: both
- Evidence: source
- Boundary: no
- What: Throw the preview away without changing any file, from the Discard button or the right-click menu.
- Commands: refactorPreview.discard
- Menus: BulkEditContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts; C bundle (refactorPreview.discard)

### PN-7.26 Group changes by file or by type
- Tag: both
- Evidence: source
- Boundary: no
- What: When edits come from more than one category, a header icon or the right-click menu switches the tree between grouping by file and grouping by type. The choice is remembered across sessions.
- Commands: refactorPreview.groupByFile; refactorPreview.groupByType; refactorPreview.toggleGrouping
- Menus: BulkEditTitle; BulkEditContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEdit.contribution.ts; V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts; C bundle (refactorPreview.groupByFile, refactorPreview.groupByType, refactorPreview.toggleGrouping, BulkEditTitle)

### PN-7.27 Open a change in the preview diff editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Opening a file or edit row shows a multi-file diff tab titled "Refactor Preview" with before and after for every file in the refactoring, scrolled to the edit you picked. It is a temporary tab that closes when the preview ends.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPane.ts (_openElementInMultiDiffEditor); V\src\vs\workbench\contrib\bulkEdit\browser\preview\bulkEditPreview.ts; C bundle (vscode-bulkeditpreview-multieditor)

## Handed off
- Comment thread widgets in the editor, comment gutter icons, Toggle Editor Commenting, Add Comment, next and previous thread, and the settings comments.visible, comments.maxHeight, comments.collapseOnResolve and comments.thread.confirmOnCollapse: ED-11
- The "Forwarded Ports" status bar item (status.forwardedPorts): SS-4
- The auto-forward notification ("Your application running on port N is available", with its Open in Browser, Preview, Make Public, Use Port as Sudo and "See all forwarded ports" buttons): NT-2
- The Simple Browser tab that Preview in Editor opens: ED-10
- Starting a preview from the editor: Rename with preview (Shift+Enter in the rename box) and Refactor with Preview (editor.action.refactor.preview, which has zero hits in the Cursor bundle): ED-4
- The confirm box "Another refactoring is being previewed" and the warning "Cannot apply refactoring because ... has changed": DL-1
- The multi-file diff editor widget itself that the preview tab uses: ED-6

## Unclear
- Ports is a remote feature. Per the catalog note it is listed here with boundary no, for the user's call. I did not run VS Code, so I cannot say whether the tab ever shows in a plain local window. The code builds it only when the forwarded ports context keys are on.
- The Comments tab only exists once something supplies comments. Searching createCommentController found only the copilot extension in V (its review service, an agent feature) and nothing in Cursor's built-in extension folders. So in Cursor with no extra extensions the tab most likely never shows.
- Pass 1 listed `remote.tunnel.forward` as a Ports command. That is a localization key. The registered ids are `remote.tunnel.forwardInline` and `remote.tunnel.forwardCommandPalette`.
- The Comments filter and sort ids are built from the view id at run time (`workbench.actions.${view id}.toggle...`). I wrote them with the real view id `workbench.panel.comments`, and found the `toggleResolvedComments`, `toggleUnResolvedComments`, `toggleSortByResource` and `toggleSortByUpdatedAt` tails in the Cursor bundle.
- No other fed-only panel view exists in V. I checked every registerViewContainer call: the panel containers are Terminal, Problems, Output, Debug Console, Test Results, Comments, Ports and Refactor Preview. The voice transcript and voice event stream containers go to the side bar and are out of scope. Containers that extensions add to the panel belong to the extension system.
- Refactor Preview is also fed by extension edits to file renames (a Review or Skip choice box in mainThreadFileSystemEventService.ts). I did not list that as a feature.
- PN-7 has 27 features because it covers three views: 1 to 9 Comments, 10 to 21 Ports, 22 to 27 Refactor Preview.
- The accessible views and help for Comments (commentsAccessibleView.ts, commentsAccessibility.ts) were not listed. The catalog says nobody owns the accessible view.
- Compared V and N source for these views: no new commands, settings or views after 1.128.0. Only inner code changed (a notification link wording in the Ports code, dependency clean-ups).
