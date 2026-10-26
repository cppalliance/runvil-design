Group: SS-4 Workspace, remote and source control. Section: Status bar. Date: 2026-10-25. Pass: 2.
Features: 16.

### SS-4.1 Remote indicator at the far left
- Tag: both
- Evidence: source
- Boundary: no
- What: The first item on the left side shows a remote icon (tooltip "Open a Remote Window") in a plain local window, and the remote icon plus the host name (tooltip "Editing on {host}", name cut at 40 characters, shown in the remote color) in a remote window or a virtual workspace such as a GitHub repo (id status.host, name "Remote Host", priority infinity so it is always first).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts (updateRemoteStatusIndicator, renderRemoteStatusIndicator); V\src\vs\workbench\contrib\remote\browser\remote.contribution.ts; C bundle (module vs/workbench/contrib/remote/browser/remoteIndicator, id status.host, labels "Open a Remote Window", "Editing on {0}", "Remote Host"; the code in updateRemoteStatusIndicator matches V)

### SS-4.2 Remote indicator shows connection and network trouble
- Tag: both
- Evidence: source
- Boundary: no
- What: In a remote window the item reads "Opening Remote..." with a spinner while connecting, "Reconnecting to {host}..." with a spinner, or an alert icon with "Disconnected from {host}" when the link drops; when the network looks offline the icon turns into an alert, the item takes the offline color and the tooltip says "Network appears to be offline, certain features might be unavailable.", and on slow links the tooltip gives the last and average latency in ms.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts (withNetworkStatus, setConnectionState, setNetworkState); C bundle (NLS keys host.open, host.reconnecting, disconnectedFrom, networkStatusOfflineTooltip, networkStatusHighLatencyTooltip in module vs/workbench/contrib/remote/browser/remoteIndicator)

### SS-4.3 Open the Remote menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the remote item, or running Show Remote Menu from the Command Palette or the shortcut, opens a picker "Select an option to open a Remote Window" that lists the commands extensions add for the current remote first, then an "Install" section of recommended remote extensions with a Learn More button on each, then the Close Remote Connection or Close Remote Workspace row (and Reload Window when disconnected).
- Commands: workbench.action.remote.showMenu
- Menus: StatusBarRemoteIndicatorMenu (contributes.menus key statusBar/remoteIndicator); StatusBarWindowIndicatorMenu; commandPalette
- Keys: ctrl+alt+o (Cmd+Alt+O on macOS)
- Settings: workbench.remoteIndicator.showExtensionRecommendations
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts (registerActions, showRemoteMenu); V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts (statusBar/remoteIndicator); V\src\vs\platform\actions\common\actions.ts; C bundle (command id workbench.action.remote.showMenu with keybinding CtrlCmd+Alt+KeyO, menu statusBar/remoteIndicator, label "Show Remote Menu", setting workbench.remoteIndicator.showExtensionRecommendations)

### SS-4.4 Close the remote connection or workspace
- Tag: both
- Evidence: source
- Boundary: no
- What: In a remote window, Close Remote Connection (File menu, Command Palette, or the last row of the Remote menu) reopens the folder in a local window; in a virtual workspace the row reads Close Remote Workspace; when the connection has dropped the Remote menu also offers Reload Window. Not offered in the web build.
- Commands: workbench.action.remote.close; workbench.action.reloadWindow
- Menus: MenubarFileMenu (group 6_close); commandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts (CLOSE_REMOTE_COMMAND_ID, showRemoteMenu); C bundle (id workbench.action.remote.close, File menu group 6_close order 3.5, labels "Close Remote Connection", "Close Remote Workspace", "Reload Window")

### SS-4.5 Restricted Mode item
- Tag: both
- Evidence: source
- Boundary: no
- What: When Workspace Trust is on and the window, folder or workspace is not trusted, a prominent item "$(shield) Restricted Mode" sits just right of the remote indicator; its hover says "Running in Restricted Mode" with the reason and links to the disabled features list and to Manage Workspace Trust, and clicking it opens the Workspace Trust editor; it disappears once trust is granted (id status.workspaceTrust, name "Workspace Trust").
- Commands: workbench.trust.manage
- Menus: commandPalette
- Keys: none found
- Settings: security.workspace.trust.enabled
- Cursor: changed - item, label and command are present, but security.workspace.trust.enabled defaults to false in Cursor (true in stock), so the item only shows after trust is switched on
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (getRestrictedModeStatusbarEntry, updateStatusbarEntry, MANAGE_TRUST_COMMAND_ID, trust setting default true); C bundle (module vs/workbench/contrib/workspace/browser/workspace.view.contribution, id status.workspaceTrust, label "Restricted Mode", command workbench.trust.manage, setting security.workspace.trust.enabled registered with default false)

### SS-4.6 Forwarded Ports item
- Tag: both
- Evidence: source
- Boundary: no
- What: A radio tower icon with a count (tooltip "No Ports Forwarded" or "Forwarded Ports: 3000, 8080") appears on the left side only in a remote window or after a port was forwarded in this session; clicking it focuses the Ports view (id status.forwardedPorts, priority 40).
- Commands: ~remote.forwardedPorts.focus
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\remote\browser\remoteExplorer.ts (updateStatusBar, entry); V\src\vs\workbench\services\remote\common\remoteExplorerService.ts (TUNNEL_VIEW_ID); C bundle (id status.forwardedPorts at priority 40, view id ~remote.forwardedPorts)

### SS-4.7 Repository name item when several repositories are open
- Tag: both
- Evidence: source
- Boundary: no
- What: When more than one Git repository is open, an extra item with a repo icon and the active repository's name sits just left of the branch item (tooltip "{folder} ({provider label})"); the active repository follows the active editor or the focused repository in the Source Control view unless one is pinned, and stock swaps the icon to a pinned-repo icon while pinned (id status.scm.provider, name "Source Control Provider").
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - the item and the active-repository logic are present, but Cursor always draws the plain repo icon and never the pinned-repo icon
- Source: V\src\vs\workbench\contrib\scm\browser\activity.ts (_updateStatusBar); V\src\vs\workbench\contrib\scm\browser\util.ts (getSCMRepositoryIcon); V\src\vs\workbench\contrib\scm\browser\scmViewService.ts (activeRepository); C bundle (module vs/workbench/contrib/scm/browser/activity, id status.scm.provider, text "$(repo) {name}", condition scmService.repositoryCount > 1, label "Source Control Provider")

### SS-4.8 Pick or pin the active repository
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the repository item opens a picker "Select the active repository, type to filter all repositories" with Auto first (the active repository follows the active editor) and then every repository with its folder path; choosing a repository pins it and choosing Auto unpins it.
- Commands: scm.setActiveProvider
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (scm.setActiveProvider); V\src\vs\workbench\contrib\scm\browser\scmViewService.ts (RepositoryPicker, pinActiveRepository); C bundle (command scm.setActiveProvider, NLS keys scmActiveRepositoryPlaceHolder and scmActiveRepositoryAutoDescription, label "Auto", method pinActiveRepository)

### SS-4.9 Branch item shows the current branch, tag or commit
- Tag: both
- Evidence: source
- Boundary: no
- What: The first source control item shows an icon and the head name: a branch icon with the branch name, a lock icon when the branch is in git.branchProtection, a tag icon on a tag, a commit icon with the short hash on a detached commit, and a spinning icon with the target name while a checkout runs; the tooltip reads "{repo folder} ({provider}) - {name}, Checkout Branch/Tag..." or the running step ("Committing Changes...", "Synchronizing Changes...") (id status.scm.0, name starts with "Source Control", priority 10000 so it is the first Git item).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.branchProtection
- Cursor: present
- Source: V\extensions\git\src\statusbar.ts (CheckoutStatusBar.command, getIcon, getTooltip); V\extensions\git\src\repository.ts (headLabel); V\src\vs\workbench\contrib\scm\browser\activity.ts (status.scm.{index}); C bundle (extensions\git\dist\main.js CheckoutStatusBar with icons lock, git-branch, tag, git-commit, loading~spin and tooltip "Checkout Branch/Tag..."; module vs/workbench/contrib/scm/browser/activity)

### SS-4.10 Branch item marks changes, staged changes and merge or rebase
- Tag: both
- Evidence: source
- Boundary: no
- What: The head name gets a star for working tree or untracked changes, a plus for staged changes, and an exclamation mark while a merge or rebase is in progress (plus " (Rebasing)" during a rebase); in stock the branch icon also changes to a changes, staged changes or branch-conflicts icon. This is the only merge conflict marker on the status bar; the conflict list, merge editor and CodeLens live in other groups.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - the star, plus, exclamation mark and "(Rebasing)" text are present, but the git-branch-changes, git-branch-staged-changes and git-branch-conflicts icons are not (Cursor only draws branch, lock, tag, commit and spinner icons)
- Source: V\extensions\git\src\repository.ts (headLabel); V\extensions\git\src\statusbar.ts (getIcon, rebasing label); C bundle (extensions\git\dist\main.js headLabel with the three suffix characters and the "Rebasing" string, getIcon without the three branch state icons; 0 hits for git-branch-conflicts, git-branch-staged-changes, git-branch-changes)

### SS-4.11 Click the branch item to check out a branch or tag
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the branch item runs Checkout to... for that repository and opens the picker of local branches, remote branches and tags (tooltip "Checkout Branch/Tag..."); the click does nothing while a checkout, commit or sync is running.
- Commands: git.checkout
- Menus: commandPalette; scm/title; scm/repository
- Keys: none found
- Settings: git.checkoutType; git.branchSortOrder; git.showReferenceDetails; git.pullBeforeCheckout
- Cursor: present
- Source: V\extensions\git\src\statusbar.ts (CheckoutStatusBar.command); V\extensions\git\package.json (git.checkout command, menus); V\extensions\git\src\commands.ts; C bundle (extensions\git\dist\main.js git.checkout in CheckoutStatusBar; extensions\git\package.json git.checkout)

### SS-4.12 Sync Changes item with ahead and behind counts
- Tag: both
- Evidence: source
- Boundary: no
- What: Next to the branch item a sync icon appears when the branch has an upstream; when it differs from the remote the item adds counts such as "2↓ 1↑" (behind then ahead, only the down count for a read-only remote); the tooltip says "Pull 2 commits from origin/main", "Push 1 commits to origin/main" or "Pull 2 and push 1 commits between origin/main", and clicking it runs Sync (pull then push); while syncing the icon spins and the click is disabled (id status.scm.1, compact pair with the branch item).
- Commands: git.sync
- Menus: commandPalette
- Keys: none found
- Settings: git.enableStatusBarSync; git.confirmSync
- Cursor: present
- Source: V\extensions\git\src\statusbar.ts (SyncStatusBar.command); V\extensions\git\src\repository.ts (syncLabel, syncTooltip); V\extensions\git\package.json (git.enableStatusBarSync default true); C bundle (extensions\git\dist\main.js SyncStatusBar with sync and sync~spin icons, git.sync command and the three tooltip strings; extensions\git\package.json git.enableStatusBarSync default true)

### SS-4.13 Publish Branch item
- Tag: both
- Evidence: source
- Boundary: no
- What: When the branch has no upstream, the sync slot shows a cloud-upload icon with the tooltip "Publish Branch" and a click runs Publish Branch; when the repository has no remote at all and a publisher such as GitHub is registered, the icon-only item says "Publish to {name}" (or "Publish to..." with several publishers); with no remote and no publisher the slot stays empty.
- Commands: git.publish
- Menus: commandPalette
- Keys: none found
- Settings: git.enableStatusBarSync
- Cursor: present
- Source: V\extensions\git\src\statusbar.ts (SyncStatusBar.command, hasRemotes and remoteSourcePublishers branches); V\extensions\git\package.json (git.publish); C bundle (extensions\git\dist\main.js SyncStatusBar with cloud-upload icon, git.publish, "Publish Branch", "Publish to {0}", "Publish to...")

### SS-4.14 Git blame item for the current line
- Tag: both
- Evidence: source
- Boundary: no
- What: On the right side a commit icon with the author and age of the cursor line (for example "Ada Lovelace (3 days ago)" from the default template) shows whenever the active text editor is in a Git repository (id git.blame, name "Git Blame Information", priority 200); it reads "Not Committed Yet" when you move onto an edited line and "Not Committed Yet (Staged)" for staged lines, it hides when there is no blame for the line, for resources other than file and git schemes, or when the cursor sits at the very start of a file you have not clicked in, and clicking it opens that commit.
- Commands: git.viewCommit
- Menus: none found
- Keys: none found
- Settings: git.blame.statusBarItem.enabled; git.blame.statusBarItem.template; git.blame.ignoreWhitespace; git.commitShortHashLength
- Cursor: present
- Source: V\extensions\git\src\blame.ts (GitBlameStatusBarItem, _updateTextEditorBlameInformation); V\extensions\git\package.json (git.blame.* settings, git.viewCommit); C bundle (extensions\git\dist\main.js createStatusBarItem("git.blame", Right, 200), name "Git Blame Information", strings "Not Committed Yet" and "Not Committed Yet (Staged)", command git.viewCommit; extensions\git\package.json settings git.blame.statusBarItem.enabled default true and git.blame.statusBarItem.template)

### SS-4.15 Blame hover card
- Tag: both
- Evidence: source
- Boundary: no
- What: Hovering the blame item shows a card with the author avatar (or an account icon), the author name as an email link, the date as "3 days ago (long date)", co-authors, the full commit message, files changed with insertions and deletions, and a row of links: the short hash that opens the commit, a copy hash button, provider links such as view on GitHub for pushed commits, and a gear that opens Settings on git.blame.
- Commands: git.viewCommit; git.copyContentToClipboard; workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: git.commitShortHashLength
- Cursor: changed - the card has avatar, author, date, message, stats, hash, copy and gear, but no co-author lines, and Open Commit passes no file to reveal in the commit diff
- Source: V\extensions\git\src\blame.ts (getBlameInformationHover, GitBlameStatusBarItem._provideTooltip); V\extensions\git\src\hover.ts (getCommitHover, getHoverCommitHashCommands, appendContent, appendShortStats, appendCommands); C bundle (extensions\git\dist\main.js getBlameInformationHover builds the markdown directly, tooltips "Open Commit", "Copy Commit Hash", "Open Settings"; 0 hits for "Co-author"; viewCommit takes two arguments)

### SS-4.16 Turn the blame item on or off and change its text
- Tag: both
- Evidence: source
- Boundary: no
- What: Toggle Git Blame Status Bar Item from the Command Palette flips the item on or off (default on), and a template setting rewrites its text from the tokens hash, hashShort, subject, authorName, authorEmail, authorDate and authorDateAgo.
- Commands: git.blame.toggleStatusBarItem
- Menus: commandPalette
- Keys: none found
- Settings: git.blame.statusBarItem.enabled; git.blame.statusBarItem.template
- Cursor: present
- Source: V\extensions\git\package.json (command, settings); V\extensions\git\src\commands.ts (toggleBlameStatusBarItem, _toggleBlameSetting); V\extensions\git\src\blame.ts (formatBlameInformationMessage); C bundle (extensions\git\package.json command git.blame.toggleStatusBarItem and both settings; extensions\git\dist\main.js toggleBlameStatusBarItem)

## Handed off

- The branch picker that opens from the branch item (local, remote, tags, create branch, detached checkout) -> SB-4.15, and its shared list widget -> QA-6.
- What Sync and Publish do once clicked, including the confirmation and the GitHub publish flow -> SB-4.25 and SB-4.26.
- The Ports view and everything in it (forward, label, visibility, auto forward rules) -> PN-7.10 to PN-7.21.
- The Workspace Trust editor and the trust prompts that Restricted Mode points to -> DL-5.
- The commit diff that Open Commit opens (multi-file diff editor titled "{hash} - {subject}") -> ED-6.13.
- The end-of-line blame text in the editor and its toggle -> ED-6.27.
- The merge conflict list and Complete Merge -> SB-4.21; the merge editor and conflict CodeLens -> ED-6.17 to ED-6.23.
- Remote indicator and offline colors -> SS-2.8 and SS-2.9; the prominent color of Restricted Mode -> SS-2.5; the compact pairing of the branch and sync items -> SS-1.5.
- Metered Connection item (pass 1 listed it here) -> SS-3.37.

## Unclear

- Remote is outside the stated scope. The remote indicator, its menu and Close Remote Connection are listed as features with boundary no because the user asked for them, but the remote extensions that fill the menu (and Forwarded Ports, which only matters on a remote or tunnel) are not in the scope list. The parent should decide whether SS-4.1 to SS-4.4 and SS-4.6 stay.
- No status bar item exists for merge conflicts in V or in Cursor. The merge-conflict extension has no status bar code (confirmed in pass 1). The only status bar sign of a merge is the "!" suffix and the conflicts icon on the branch item (SS-4.10). Cursor's git extension adds a "Resolve in Chat" command (git.resolveAllConflictsInChat) that is a chat feature and is not listed.
- The Remote menu rows that install recommended remote extensions, and the Install Remote Development Extensions command (workbench.action.remote.extensions), open the Extensions view and extension install flow, so they are out of scope and not listed. The links "features are disabled" (Restricted Mode hover) and "features are not available" (virtual workspace hover) open the unsupported extensions list and are left out for the same reason.
- This machine has the Remote - SSH extension (anysphere.remote-ssh) installed as a user extension, not a built-in. I did not compare the Remote menu contents with it installed.
- Newer than 1.128, details of features already listed: in N (1.144.0) the Forwarded Ports item runs remoteExplorer.toggleForwardedPortsView instead of focusing the Ports view (SS-4.6), and the repository item can take its name from the provider's activeRepositoryName (SS-4.7). Both are newer-than-1.128 and Cursor has neither.
- The exact name of the branch and sync items depends on a suffix taken from the command argument (for example "Source Control Git"); I did not trace the runtime value. Priority 10000 and ids status.scm.0 and status.scm.1 are from the source.
- Nothing was run on screen except what pass 1 saw: remote icon, repository item with a repo icon, branch item with a star and a sync icon, and the workspace name item. The blame item was not visible in that look because no text editor had focus.
- The Restricted Mode item is off in Cursor by default because trust is off, so it was not seen running. The hover text strings are from the Cursor NLS file.
