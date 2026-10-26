Group: SS-4 Workspace, remote and source control. Pass: 3. Date: 2026-10-25.

### SS-4.1
1. In a plain local window the first item on the left of the status bar is a remote icon with the tooltip "Open a Remote Window".
2. In a remote window the item shows the remote icon followed by the host name, drawn in the remote color, and its tooltip reads "Editing on {host}".
3. A host name longer than 40 characters is cut off in the item text.
4. In a virtual workspace such as a GitHub repository the item looks the same as in a remote window, with the remote icon and the workspace host name.
5. The item always stays first on the left side, ahead of every other left-side item.

### SS-4.2
1. While a remote window is still connecting the item reads "Opening Remote..." with a spinner.
2. While the link to the host is being re-established the item reads "Reconnecting to {host}..." with a spinner.
3. When the connection drops the item shows an alert icon with the text "Disconnected from {host}".
4. When the network looks offline the remote icon turns into an alert icon, the item takes the offline color, and the tooltip says "Network appears to be offline, certain features might be unavailable."
5. On a slow link the tooltip gives the last and the average latency in milliseconds.

### SS-4.3
1. Clicking the remote item opens a picker whose placeholder reads "Select an option to open a Remote Window".
2. Running Show Remote Menu (`workbench.action.remote.showMenu`) from the Command Palette, or pressing `Ctrl+Alt+O` (`Cmd+Alt+O` on macOS), opens the same picker.
3. The picker lists the commands that extensions add through the `statusBar/remoteIndicator` menu first, under their category headings, with the commands for the current remote ahead of the others.
4. After those commands an "Install" section lists the recommended remote extensions that are not installed yet, each with a Learn More button.
5. Setting `workbench.remoteIndicator.showExtensionRecommendations` chooses whether the "Install" section is shown, with values on (default) and off.

### SS-4.4
1. In a remote window, choosing File, Close Remote Connection reopens the folder in a local window.
2. Running Close Remote Connection (`workbench.action.remote.close`) from the Command Palette does the same, and the command is offered only in a remote window or a virtual workspace.
3. In a remote window the last row of the Remote menu reads "Close Remote Connection" and does the same.
4. In a virtual workspace the last row of the Remote menu reads "Close Remote Workspace" instead.
5. When the connection has dropped, the Remote menu also lists Reload Window (`workbench.action.reloadWindow`) after the close row.
6. In the web build neither the close command nor the close rows are offered.

### SS-4.5
1. When Workspace Trust is on and the window, folder or workspace is not trusted, an item with a shield icon and the text "Restricted Mode" appears just right of the remote indicator.
2. Hovering the item shows "Running in Restricted Mode" with the reason, a link to the list of disabled features, and a link to Manage Workspace Trust.
3. Clicking the item, or running Manage Workspace Trust (`workbench.trust.manage`) from the Command Palette, opens the Workspace Trust editor.
4. The item disappears once trust is granted.
5. Setting `security.workspace.trust.enabled` chooses whether Workspace Trust is on, with values on and off, and with it off the item never appears; stock defaults it to on.
6. In Cursor the setting defaults to off, so the item shows only after trust is switched on. [cursor only]

### SS-4.6
1. In a remote window a radio tower icon with a port count appears on the left side of the bar.
2. The tooltip reads "No Ports Forwarded" when nothing is forwarded and "Forwarded Ports: 3000, 8080" with the port numbers when some are.
3. In a local window the item stays hidden until a port has been forwarded in this session.
4. Clicking the item focuses the Ports view (`~remote.forwardedPorts.focus`).

### SS-4.7
1. When more than one Git repository is open, an extra item with a repo icon and the name of the active repository appears just left of the branch item.
2. With only one repository open the item is not shown.
3. The tooltip reads "{folder} ({provider label})".
4. The active repository follows the active editor, or the focused repository in the Source Control view, unless one repository is pinned.
5. While a repository is pinned the item shows a pinned-repo icon in place of the plain repo icon. [stock only]
6. In Cursor the item always shows the plain repo icon, even while a repository is pinned. [cursor only]

### SS-4.8
1. Clicking the repository item runs `scm.setActiveProvider` and opens a picker whose placeholder reads "Select the active repository, type to filter all repositories".
2. The first row of the picker is Auto, which makes the active repository follow the active editor.
3. Below Auto every open repository is listed as a row with its folder path.
4. Typing in the picker filters the repositories.
5. Choosing a repository pins it as the active one.
6. Choosing Auto unpins the repository.

### SS-4.9
1. The first source control item on the bar shows a branch icon and the name of the current branch.
2. When the branch is listed in `git.branchProtection` the branch icon is replaced by a lock icon.
3. On a checked-out tag the item shows a tag icon and the tag name.
4. On a detached commit the item shows a commit icon and the first 8 characters of the commit hash.
5. While a checkout runs the item shows a spinning icon and the name of the target.
6. The tooltip reads "{repo folder} ({provider}) - {name}, Checkout Branch/Tag...", or names the running step: "Checking Out Branch/Tag...", "Committing Changes..." or "Synchronizing Changes...".
7. Setting `git.branchProtection` lists the branch names that get the lock icon, as a list of names that starts empty.
8. The item sits first among the Git items on the left side, ahead of the sync item.

### SS-4.10
1. The head name on the branch item gets a star at its end when the working tree has changes or untracked files.
2. The head name gets a plus when changes are staged.
3. The head name gets an exclamation mark while a merge or rebase is in progress.
4. During a rebase the label also reads " (Rebasing)" after the head name.
5. In stock the branch icon changes to a changes icon, a staged changes icon or a branch conflicts icon to match those states. [stock only]
6. In Cursor the branch icon stays the plain branch icon (or the lock, tag, commit or spinner icon) and never changes to the changes, staged changes or branch conflicts icons. [cursor only]

### SS-4.11
1. Clicking the branch item runs Checkout to... (`git.checkout`) for that repository and opens the picker of branches and tags.
2. Checkout to... also opens from the Command Palette and from the Source Control view title and repository menus.
3. While a checkout, commit or sync is running, clicking the branch item does nothing.
4. Setting `git.checkoutType` chooses which kinds of refs the picker lists, with values `local`, `tags` and `remote`, all three by default.
5. Setting `git.branchSortOrder` chooses how branches are sorted, with values `committerdate` (default) and `alphabetically`.
6. Setting `git.showReferenceDetails` chooses whether the last commit details are shown on refs in the checkout, branch and tag pickers, with values on (default) and off.
7. Setting `git.pullBeforeCheckout` chooses whether a branch with no outgoing commits is fast-forwarded before it is checked out, with values on and off (default).

### SS-4.12
1. When the branch has an upstream, a sync icon item appears next to the branch item.
2. While the branch matches its remote the item shows only the icon and its tooltip reads "Synchronize Changes".
3. When the branch differs from its remote the item adds counts such as "2↓ 1↑", behind first and ahead second.
4. For a read-only remote the item shows only the down count, for example "2↓".
5. The tooltip reads "Pull 2 commits from origin/main" when only behind, "Push 1 commits to origin/main" when only ahead, and "Pull 2 and push 1 commits between origin/main" when both.
6. Clicking the item, or running Sync (`git.sync`) from the Command Palette, pulls and then pushes.
7. While a checkout, commit or sync is running the click does nothing and the tooltip names the step, and during a sync the icon spins.
8. Setting `git.enableStatusBarSync` chooses whether the Sync item appears in the bar, with values on (default) and off.
9. Setting `git.confirmSync` chooses whether a confirmation is asked before synchronizing, with values on (default) and off.

### SS-4.13
1. When the branch has no upstream, the sync slot shows a cloud-upload icon with the tooltip "Publish Branch".
2. Clicking the item, or running Publish Branch (`git.publish`) from the Command Palette, publishes the branch.
3. When the repository has no remote at all and one publisher such as GitHub is registered, the slot shows an icon-only item with the tooltip "Publish to {name}".
4. When several publishers are registered the tooltip reads "Publish to...".
5. With no remote and no publisher the slot stays empty.
6. While a checkout or commit is running the click does nothing and the tooltip names the step, "Checking Out Changes..." or "Committing Changes...".
7. Turning `git.enableStatusBarSync` off hides the publish items together with the Sync item.

### SS-4.14
1. On the right side of the bar a commit icon with the author and age of the line under the cursor appears, for example "Ada Lovelace (3 days ago)" with the default template.
2. The item shows only while the active text editor is in a Git repository, and it is hidden for resources other than file and git schemes.
3. After you move onto an edited line the text reads "Not Committed Yet", and on a staged line it reads "Not Committed Yet (Staged)".
4. The item is hidden when there is no blame for the line, and when the cursor sits at the very start of a file you have not clicked in.
5. Clicking the item opens that commit (`git.viewCommit`).
6. Setting `git.blame.ignoreWhitespace` chooses whether whitespace-only changes are ignored when finding the commit for a line, with values on and off (default).
7. Setting `git.commitShortHashLength` chooses how many characters a short commit hash has, a number from 7 to 40 with default 7.

### SS-4.15
1. Hovering the blame item shows a card with the author avatar, or an account icon when there is no avatar, and the author name as an email link.
2. The card shows the date as "3 days ago (long date)".
3. The card lists the co-authors of the commit. [stock only]
4. The card shows the full commit message and the files changed with their insertions and deletions.
5. The card ends with a row of links: the short hash that opens the commit (`git.viewCommit`), a copy hash button (`git.copyContentToClipboard`), and a gear that opens Settings on `git.blame` (`workbench.action.openSettings`).
6. For commits that are already pushed the row also holds provider links such as view on GitHub.
7. Setting `git.commitShortHashLength` sets how long the short hash in the card is, a number from 7 to 40 with default 7.
8. In Cursor the card has no co-author lines, and Open Commit passes no file to reveal in the commit diff. [cursor only]

### SS-4.16
1. Running Toggle Git Blame Status Bar Item (`git.blame.toggleStatusBarItem`) from the Command Palette switches the blame item on or off.
2. Setting `git.blame.statusBarItem.enabled` chooses whether the blame item is shown, with values on (default) and off.
3. Setting `git.blame.statusBarItem.template` rewrites the text of the blame item, with the default `${authorName} (${authorDateAgo})`.
4. The template accepts the tokens `hash`, `hashShort`, `subject`, `authorName`, `authorEmail`, `authorDate` and `authorDateAgo`.

## Unclear
- SS-4.17 and SS-6.11 describe the same Workspace Name item, so its pieces appear in both files. One of the two features should be dropped.
- In SS-4.17 I tagged every piece `[cursor only]` because the whole feature is Cursor only.
- SS-4.1 to SS-4.4 and SS-4.6 depend on the remote scope decision recorded in the pass 2 file, so their pieces stay until that is settled.
- SS-4.9 piece 4 says 8 characters because the stock Git extension source cuts a detached commit hash there, and `git.commitShortHashLength` is not used by the branch item. I did not check the same cut in the Cursor bundle.
