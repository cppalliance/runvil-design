Group: SB-4 Source Control. Pass: 3. Date: 2026-10-25.

### SB-4.1
1. Opening the Source Control view lists the repository's changed files under group headings, each heading showing a count of its files.
2. Pressing Ctrl+Shift+G (Control+Shift+G on Mac) runs `workbench.view.scm` and opens the Source Control view.
3. Changed files are sorted into Merge Changes, Staged Changes, Changes and Untracked Changes groups, and the Merge Changes group only shows while there are conflicts.
4. Each changed file row shows the file name, its folder path in list view, and a status letter such as M, D or U on the right.
5. Hovering a group header or a file row shows inline buttons, and `scm.alwaysShowActions` keeps them visible all the time.
6. Setting `git.untrackedChanges` chooses where new files go, with values `mixed` (inside Changes), `separate` (their own Untracked Changes group) and `hidden`.
7. Setting `git.alwaysShowStagedChangesResourceGroup` keeps the Staged Changes group on screen even when nothing is staged.
8. Setting `git.statusLimit` caps how many changes are listed, and `git.ignoreLimitWarning` hides the warning shown when the cap is hit.
9. Settings `git.autorefresh` and `git.ignoreSubmodules` decide whether the list updates itself when files change and whether submodule changes are listed.
10. Setting `scm.maxRenderedResources` limits how many files are drawn in each group, with a default of 5000. [cursor only]

### SB-4.2
1. Clicking the plus button on a file row under Changes moves that file into Staged Changes and the group counts update.
2. Clicking the minus button on a file row under Staged Changes moves that file back into Changes.
3. Right-clicking a file row offers Stage Changes or Unstage Changes (`git.stage`, `git.unstage`) with the same result as the inline button.
4. In tree view a folder row has the same plus or minus button and right-click entry, and it stages or unstages every file under that folder.
5. The Changes header has Stage All Changes (`git.stageAll`) and the Staged Changes header has Unstage All Changes (`git.unstageAll`), as inline buttons and right-click entries.
6. The Merge Changes header has Stage All Merge Changes (`git.stageAllMerge`), which stages every conflicted file in one step.
7. Setting `git.untrackedChanges` changes the header buttons: with `mixed` the Changes header stages everything, and otherwise it shows Stage All Tracked Changes (`git.stageAllTracked`) while the Untracked Changes header shows Stage All Untracked Changes (`git.stageAllUntracked`).
8. In a multi-file diff editor each file's title has Stage and Unstage buttons (`git.stageFile`, `git.unstageFile`).

### SB-4.3
1. Clicking the Discard Changes button on a file row, or choosing it from the right-click menu, asks for confirmation and then restores the file to its last committed content (`git.clean`).
2. The confirmation for a tracked file is a modal that names the file and has a Discard File button, and a file that was deleted shows Restore File instead.
3. In tree view a folder row has Discard Changes in its right-click menu and as an inline button, and it applies to every file under that folder.
4. The Changes header has Discard All Changes (`git.cleanAll`) when `git.untrackedChanges` is `mixed` and Discard All Tracked Changes (`git.cleanAllTracked`) otherwise, and the modal states the file count and that the loss is irreversible.
5. The Untracked Changes header has Discard All Untracked Changes (`git.cleanAllUntracked`), which asks before deleting the new files.
6. Discarding untracked files shows a modal saying the files will be deleted, with a Move to Recycle Bin button on Windows (Move to Trash on other systems) and a note that they can be restored from there.
7. Setting `git.discardUntrackedChangesToTrash` set to false changes that modal into a permanent delete warning with a Delete File button.
8. When a selection holds both tracked and untracked files, the modal offers Discard All Tracked Files or Discard All Files.

### SB-4.4
1. Clicking a changed file row opens a diff editor of that file against its Git version.
2. Setting `git.openDiffOnClick` turned off makes a click open the file itself instead of a diff.
3. An inline Open File button on the row opens the file in the regular editor (`git.openFile`), and `git.showInlineOpenFileAction` hides it.
4. The right-click menu has Open File (HEAD), which shows the committed version, and Compare with Workspace (`git.openHEADFile`, `git.compareWithWorkspace`).
5. Each group header has an inline View Changes button that opens every file of that group in one multi-file diff editor (`git.viewChanges`, `git.viewStagedChanges`, `git.viewUntrackedChanges`).
6. Setting `scm.autoReveal` on makes the list scroll to and select a file's row when that file is opened.
7. Setting `git.closeDiffOnOperation` on closes open diff tabs when changes are staged, unstaged, discarded, committed or stashed.
8. The group View Changes buttons show only while `multiDiffEditor.experimental.enabled` is on, and it is on by default. [cursor only]

### SB-4.5
1. The View and Sort submenu in the "..." title menu switches between View as List and View as Tree, with the current choice checked (`workbench.scm.action.setListViewMode`, `workbench.scm.action.setTreeViewMode`).
2. A button in the view title bar flips the view mode in one click, showing a tree icon while the list is shown and a list icon while the tree is shown (`workbench.scm.action.setTreeViewModeNavigation`, `workbench.scm.action.setListViewModeNavigation`).
3. The View and Sort submenu has Sort Changes by Name, Sort Changes by Path and Sort Changes by Status, with the active one checked (`workbench.scm.action.setSortKey.name`, `.path`, `.status`).
4. In tree view a Collapse All button closes every folder (`workbench.scm.action.collapseAll`).
5. Setting `scm.defaultViewMode` chooses the starting view mode, with values `tree` and `list` and a default of `list`.
6. Setting `scm.defaultViewSortKey` chooses the starting sort order, with values `name`, `path` and `status` and a default of `path`.
7. Setting `scm.compactFolders` on, the default, joins folders that hold a single child folder into one combined row.

### SB-4.6
1. Right-clicking a changed file row and choosing Reveal in Explorer View selects that file in the Explorer view (`git.revealInExplorer`).
2. Right-clicking a changed file row and choosing Reveal in File Explorer on Windows, Reveal in Finder on Mac or Open Containing Folder on Linux shows the file in the system file manager (`git.revealFileInOS.windows`, `git.revealFileInOS.mac`, `git.revealFileInOS.linux`).
3. Right-clicking a changed file row and choosing Add to .gitignore adds that file to the repository's `.gitignore` (`git.ignore`).
4. In tree view a folder row has the same reveal and ignore entries in its right-click menu, and they act on the folder.

### SB-4.7
1. The message box at the top of the Changes view, hidden when `git.showCommitInput` is off, shows the placeholder `Message (Ctrl+Enter to commit on "branch")` and grows from `scm.inputMinLineCount` to `scm.inputMaxLineCount` lines using `scm.inputFontFamily` and `scm.inputFontSize`.
2. Pressing Ctrl+Enter in the box (`scm.acceptInput`) or clicking the Commit button in the view title (`git.commit`) commits with the typed message and empties the box.
3. Pressing Escape clears the typed message when no suggestion list or warning is open (`scm.clearInput`).
4. Pressing Escape while a warning shows under the box dismisses the warning (`scm.clearValidation`). [stock only]
5. Setting `git.inputValidation` on shows a warning under the box when the message or its first line runs past `git.inputValidationLength` (72) or `git.inputValidationSubjectLength` (50).
6. Committing with nothing staged asks whether to stage all changes and commit them, with Yes, Always and Never, and `git.enableSmartCommit`, `git.smartCommitChanges` (values `all` and `tracked`) and `git.suggestSmartCommit` decide whether that happens silently or is offered.
7. Committing on a branch listed in `git.branchProtection` asks Commit Anyway or Commit to a New Branch, and `git.branchProtectionPrompt` chooses `alwaysCommit`, `alwaysCommitToNewBranch` or `alwaysPrompt`.
8. Setting `git.promptToSaveFilesBeforeCommit`, with values `always`, `staged` and `never`, decides whether unsaved files raise a Save All & Commit Changes prompt.
9. Setting `git.diagnosticsCommitHook.enabled` on makes a commit with unresolved problems stop and ask Commit Anyway or View Problems, and `git.diagnosticsCommitHook.sources` lists which sources and severities count.
10. In Cursor the two diagnostics hook settings are spelled `git.diagnosticsCommitHook.Enabled` and `git.diagnosticsCommitHook.Sources` with capital letters. [cursor only]

### SB-4.8
1. A wide button under the message box reads Commit with a check icon when there are changes ready to commit (`git.commit`).
2. The wide button reads Continue during a merge or rebase and finishes that operation.
3. The wide button reads Sync Changes with the number of commits to pull and to push when the branch is ahead or behind its remote (`git.sync`).
4. The wide button reads Publish Branch when the local branch has no remote branch yet (`git.publish`).
5. The Commit button has a drop-down arrow listing Commit and Push, Commit and Sync, and Commit (Amend).
6. Setting `git.postCommitCommand` chooses what runs after a commit, with values `none`, `push` and `sync`, and `git.rememberPostCommitCommand` makes the button remember the last choice.
7. Settings `git.showActionButton` (with `commit`, `publish` and `sync` switches) and `scm.showActionButton` show or hide the wide button.

### SB-4.9
1. The Commit submenu in the "..." title menu lists Commit, Commit Staged and Commit All (`git.commit`, `git.commitStaged`, `git.commitAll`).
2. Commit Staged commits only the staged files, and Commit All stages every change and commits them (`git.commitStaged`, `git.commitAll`).
3. The Signed Off entries add a sign-off line to the message (`git.commitSigned`, `git.commitStagedSigned`, `git.commitAllSigned`), `git.alwaysSignOff` does it for every commit, and `git.enableCommitSigning` signs commits with GPG, X.509 or SSH.
4. The No Verify entries skip the commit hooks and only show when `git.allowNoVerifyCommit` is on (`git.commitNoVerify`, `git.commitStagedNoVerify`, `git.commitAllNoVerify`).
5. Setting `git.confirmNoVerifyCommit` makes a No Verify commit ask for confirmation with OK and OK, Don't Ask Again.
6. Commit Empty makes a commit with no changes after asking for confirmation, and `git.confirmEmptyCommits` turns the question off (`git.commitEmpty`).
7. Committing when there are no changes shows "There are no changes to commit." with a Create Empty Commit button.

### SB-4.10
1. Commit (Amend) in the Commit submenu rewrites the last commit with the staged changes and the text in the message box (`git.commitAmend`).
2. Commit Staged (Amend) uses only the staged changes, and Commit All (Amend) stages everything first (`git.commitStagedAmend`, `git.commitAllAmend`).
3. Amending with an empty message box keeps the previous commit message and does not ask for a new one.
4. Commit (Amend) is also in the drop-down of the Commit button under the message box.
5. The Amend No Verify entries skip the commit hooks and only show when `git.allowNoVerifyCommit` is on (`git.commitAmendNoVerify`, `git.commitStagedAmendNoVerify`, `git.commitAllAmendNoVerify`).

### SB-4.11
1. Choosing Undo Last Commit in the Commit submenu removes the newest commit and puts its changes back into the working tree (`git.undoCommit`).
2. The undone commit's message appears in the message box, ready to edit and commit again.
3. When the newest commit is a merge commit, a modal asks "The last commit was a merge commit. Are you sure you want to undo it?" with an Undo merge commit button.
4. In a repository with no commits, Undo Last Commit shows the warning "Can't undo because HEAD doesn't point to any commit."

### SB-4.12
1. With `git.useEditorAsCommitInput` on, the default, committing with an empty message box opens git's commit message in a full editor tab instead of asking in a small input.
2. The commit message tab has Commit and Cancel buttons, where Commit finishes the commit with the text and Cancel drops it (`git.commitMessageAccept`, `git.commitMessageDiscard`).
3. Running Restore Commit Template puts the commit message template back into the open message editor (`git.restoreCommitTemplate`).
4. Setting `git.verboseCommit` on adds the verbose diff of what is being committed to the commit message editor.
5. Setting `git.terminalGitEditor` on makes the app the Git editor for Git commands run in the integrated terminal, after the terminal is restarted.
6. With `git.useEditorAsCommitInput` off, committing with an empty message box shows an input box that asks "Please provide a commit message".
7. In Cursor the Commit and Cancel buttons sit in the editor title bar instead of the editor content area. [cursor only]

### SB-4.13
1. Pressing Up while the cursor is on the first line of the message box replaces the text with the previous message from the history (`scm.viewPreviousCommit`).
2. Pressing Down while the cursor is on the last line steps to the next message in the history (`scm.viewNextCommit`).
3. Pressing Alt+Up or Alt+Down anywhere in the box steps back or forward through the history whatever line the cursor is on (`scm.forceViewPreviousCommit`, `scm.forceViewNextCommit`).
4. While the suggestion list is open, Up and Down move through the suggestions instead of the history.
5. The history is kept separately for each repository and holds up to 100 messages.

### SB-4.14
1. A sparkle button labeled Generate Commit Message appears in the message box toolbar of a Git repository. [cursor only]
2. Clicking the sparkle button fills the message box with a generated commit message. [cursor only]
3. Running Generate Commit Message from the Command Palette fills the message box the same way (`cursor.generateGitCommitMessage`). [cursor only]
4. Setting `scm.showInputActionButton` on, the default, lets the message box toolbar with this button show, and turning it off hides it. [cursor only]

### SB-4.15
1. Choosing Checkout to... from the "..." title menu opens a pick list of branches, remote branches and tags (`git.checkout`).
2. Picking a local branch, a remote branch or a tag in the list switches the repository to that ref.
3. The list includes Create new branch..., Create new branch from... and Checkout detached... entries along with the refs.
4. Checkout to (Detached)... opens a pick list for checking out a commit without a branch (`git.checkoutDetached`).
5. When local changes would be overwritten by a checkout, a modal offers Stash & Checkout, Migrate Changes or Force Checkout.
6. Setting `git.checkoutType` chooses which kinds of refs the list shows, with values `local`, `remote` and `tags`.
7. Setting `git.branchSortOrder` chooses how branches are ordered in the list, with values `committerdate` and `alphabetically`.
8. Settings `git.showReferenceDetails` and `git.pullBeforeCheckout` turn on or off the last commit details beside each ref and the fast-forward of a branch with no outgoing commits before it is checked out.

### SB-4.16
1. Create Branch... in the Branch submenu asks for a name in an input box that says "Please provide a new branch name" and then creates and switches to the new branch (`git.branch`).
2. Create Branch From... first asks which ref to start from ("Select a ref to create the branch from") and then asks for the name (`git.branchFrom`).
3. Typing a name that already exists shows the message `Branch "name" already exists` and the name is not accepted.
4. Rename Branch... asks for a new name for the current branch and shows an error for an invalid name or one that already exists (`git.renameBranch`).
5. Delete Branch... lists local branches to delete, refuses the active branch with "The active branch cannot be deleted.", and for an unmerged branch asks "not fully merged. Delete anyway?" with a Delete Branch button (`git.deleteBranch`).
6. Delete Remote Branch... lists remote branches to delete and refuses the remote branch of the active branch (`git.deleteRemoteBranch`).
7. Setting `git.branchPrefix` adds a prefix to new branch names and `git.branchWhitespaceChar` replaces spaces in typed names, with a default of `-`.
8. Setting `git.branchValidationRegex` rejects a new branch name that does not match with the message "Branch name needs to match regex".
9. Settings `git.branchRandomName.enable` and `git.branchRandomName.dictionary` pre-fill a random name built from `adjectives`, `animals`, `colors` and `numbers`, and the input has a Regenerate Branch Name button.
10. Setting `git.defaultBranchName` names the first branch of a repository made with Initialize Repository.

### SB-4.17
1. Merge... in the Branch submenu opens a pick list that says "Select a branch or tag to merge from" and merges the pick into the current branch (`git.merge`).
2. A merge that stops on conflicts shows an error saying there are merge conflicts to resolve, with a Show Changes button, and the files appear in the Merge Changes group.
3. Abort Merge in the Branch submenu cancels the merge that is in progress (`git.mergeAbort`).

### SB-4.18
1. Rebase Branch... in the Branch submenu opens a pick list that says "Select a branch to rebase onto" and rebases the current branch onto the pick (`git.rebase`).
2. Abort Rebase in the Commit submenu abandons the rebase that is in progress (`git.rebaseAbort`).
3. Running Abort Rebase when no rebase is running shows the message "No rebase in progress."

### SB-4.19
1. Create Tag... asks for a tag name and then an optional message, and a message makes an annotated tag (`git.createTag`).
2. Delete Tag... lists local tags to delete, and a repository with none shows "This repository has no tags." (`git.deleteTag`).
3. Delete Remote Tag... asks which remote and then which remote tag to delete (`git.deleteRemoteTag`).
4. Push Tags sends all local tags to the remote, and Push (Follow Tags) and Push (Follow Tags, Force) push the branch along with its tags (`git.pushTags`, `git.pushWithTags`, `git.pushWithTagsForce`).
5. Settings `git.pullTags`, `git.replaceTagsWhenPull` and `git.followTagsWhenSync` turn on or off fetching tags on pull, replacing conflicting local tags on pull, and pushing annotated tags on sync.
6. A pull blocked by conflicting tags shows a modal asking whether to replace the local tags, with a Replace Local Tag(s) button and an Always Replace Local Tag(s) button.
7. The Tags submenu in the "..." title menu lists Push Tags as a fourth entry. [stock only]

### SB-4.20
1. Stash saves the tracked changes away and cleans the working tree, after asking for an optional message in an input box that says "Stash message" (`git.stash`).
2. Stash (Include Untracked) also stashes new files and Stash Staged stashes only the staged changes (`git.stashIncludeUntracked`, `git.stashStaged`).
3. Stashing with nothing to stash shows "There are no changes to stash." or "There are no staged changes to stash."
4. Apply Stash... and Pop Stash... ask which stash to use, and Apply Latest Stash and Pop Latest Stash use the newest one without asking, where pop also removes the stash (`git.stashApply`, `git.stashPop`, `git.stashApplyLatest`, `git.stashPopLatest`).
5. Drop Stash... asks "Are you sure you want to drop the stash" before removing one, and Drop All Stashes... asks with the count and warns it may be impossible to recover (`git.stashDrop`, `git.stashDropAll`).
6. View Stash... asks which stash to look at and opens its changes in a multi-file diff (`git.stashView`).
7. Applying, popping, dropping or viewing when the repository has no stashes shows "There are no stashes in the repository."
8. Setting `git.promptToSaveFilesBeforeStash`, with values `always`, `staged` and `never`, decides whether unsaved files raise a Save All & Stash or Stash Anyway prompt.
9. Setting `git.useCommitInputAsStashMessage` on uses the text in the commit message box as the default stash message.
10. The View Stash item only shows while `multiDiffEditor.experimental.enabled` is on, and it is on by default. [cursor only]

### SB-4.21
1. Conflicted files appear in the Merge Changes group, each marked with a conflict status letter.
2. Opening a conflicted file shows it in the editor with its conflict markers (`git.openFile`).
3. Right-clicking a conflicted file offers Accept All Current and Accept All Incoming, which resolve every conflict in that file one way (`merge-conflict.accept.all-current`, `merge-conflict.accept.all-incoming`).
4. Stage Changes on a conflicted file, or Stage All Merge Changes on the group header, moves the file into Staged Changes as resolved (`git.stage`, `git.stageAllMerge`).
5. Staging a file that still has conflicts asks whether you are sure you want to stage it with merge conflicts, with a Yes button.
6. A file deleted on one side and changed on the other shows a modal that says who deleted it, with a Keep Our Version or Keep Their Version button and a Delete File button.

### SB-4.22
1. Fetch in the title menu fetches from the default remote (`git.fetch`).
2. Fetch (Prune) fetches and removes remote branches that no longer exist on the remote (`git.fetchPrune`).
3. Fetch From All Remotes fetches every remote, and a repository with no remotes shows "This repository has no remotes configured to fetch from." (`git.fetchAll`).
4. Fetch is also available from the Pull, Push submenu and from the Graph title bar.
5. Setting `git.autofetch` chooses whether Git fetches by itself on a timer, with values `true`, `false` and `all` (all remotes).
6. Setting `git.autofetchPeriod` sets the number of seconds between automatic fetches.
7. Setting `git.pruneOnFetch` on makes every fetch prune.

### SB-4.23
1. Pull in the title menu pulls the current branch from its remote (`git.pull`).
2. Pull (Rebase) pulls and rebases the branch (`git.pullRebase`).
3. Pull from... asks "Pick a remote to pull the branch from" and then "Pick a branch to pull from" (`git.pullFrom`).
4. A repository with no remote shows "Your repository has no remotes configured to pull from."
5. When a pull looks like the branch was rebased, a warning asks first and offers Always Pull, and `git.ignoreRebaseWarning` hides that warning.
6. Setting `git.autoStash` on stashes changes before the pull and restores them after a successful pull.
7. Setting `git.fetchOnPull` on makes a pull fetch all branches instead of only the current one.

### SB-4.24
1. Push in the title menu pushes the current branch (`git.push`).
2. Push to... asks which remote to push to (`git.pushTo`).
3. A repository with no remote shows "Your repository has no remotes configured to push to." with an Add Remote button.
4. A branch with no remote branch asks "Would you like to publish this branch?" with OK and OK, Don't Ask Again buttons.
5. Push (Force) and Push to... (Force) show in the Pull, Push submenu only when `git.allowForcePush` is on (`git.pushForce`, `git.pushToForce`).
6. A force push asks for confirmation with a message that it can overwrite others' changes, and `git.confirmForcePush` turns that question off.
7. Settings `git.useForcePushWithLease` and `git.useForcePushIfIncludes` choose the safer force push variants.
8. Setting `git.showPushSuccessNotification` on shows a notification after a push succeeds.
9. A push the remote rejects shows "Can't push refs to remote. Try running Pull first to integrate your changes."

### SB-4.25
1. The Sync Changes action button, the Sync entry in the Pull, Push submenu and the Command Palette each pull and then push in one step (`git.sync`).
2. Sync (Rebase) pulls with rebase and then pushes (`git.syncRebase`).
3. Sync first asks for confirmation with the message "This action will pull and push commits from and to" the remote branch, with OK and OK, Don't Show Again buttons, and `git.confirmSync` turns it off.
4. Setting `git.rebaseWhenSync` on makes plain Sync rebase instead of merge.
5. Setting `git.supportCancellation` on shows a notification while Sync runs that lets you cancel it.

### SB-4.26
1. Publish Branch... pushes a local branch with no remote branch for the first time and asks which remote when there are several (`git.publish`).
2. With no remote set up, Publish Branch lists providers such as Publish to GitHub (`github.publish`), and shows "Your repository has no remotes configured to publish to." when no provider is available.
3. Add Remote... in the Remote submenu asks for a URL or a provider and then a remote name, and rejects a name that is badly formed or already used (`git.addRemote`).
4. Remove Remote in the Remote submenu asks "Pick a remote to remove", and a repository with no remotes shows "Your repository has no remotes." (`git.removeRemote`).
5. Publish Branch is reachable from the Branch submenu, from the Graph title bar and from the action button under the message box.

### SB-4.27
1. Clone from the title menu, the Command Palette, or the empty Source Control and Explorer views asks for a source, Clone from URL or a provider, and then for the repository URL (`git.clone`).
2. After the URL, a folder picker titled "Choose a folder to clone" with a Select as Repository Destination button asks where to put the clone.
3. A progress notification reads `Cloning git repository "url"...` while the clone runs.
4. Clone (Recursive) also clones the submodules (`git.cloneRecursive`).
5. When the clone finishes, a prompt asks "Would you like to open the repository?" with Open, Open in New Window and Add to Workspace buttons.
6. Setting `git.openAfterClone` chooses what happens after a clone, with values `always`, `alwaysNewWindow`, `whenNoFolderOpen` and `prompt`.
7. Setting `git.defaultCloneDirectory` sets the default folder that clones go into.
8. Cloning a repository that is already cloned offers a pick list titled "Open Existing Repository Clone" with a Clone again choice.
9. Clone Repository Locally and Open on Desktop... clones the current remote repository to this machine (`git.continueInLocalClone`).

### SB-4.28
1. The Repositories view, hidden by default, lists every repository with its name and current branch.
2. A Repositories submenu in the View and Sort menu has a checkable entry for each repository that decides whether the Changes view shows it (`workbench.scm.action.toggleRepositoryVisibility.<provider id>`).
3. The View and Sort menu has Sort by Discovery Time, Sort by Name and Sort by Path for the repository list, with the active one checked (`workbench.scm.action.repositories.setSortKey.discoveryTime`, `.name`, `.path`).
4. Setting `scm.repositories.sortOrder` chooses the starting repository order, with values `discovery time`, `name` and `path`.
5. With several repositories shown, the Changes view has a header for each one, and Collapse All Repositories and Expand All Repositories buttons appear in its title (`workbench.scm.action.collapseAllRepositories`, `workbench.scm.action.expandAllRepositories`).
6. Setting `scm.alwaysShowRepositories` on shows the repository header in the Changes view even when there is only one repository.
7. Setting `scm.providerCountBadge` chooses the count badge on repository headers, with values `hidden`, `auto` and `visible`.
8. Setting `scm.repositories.visible` sets how many repositories the Repositories view shows at once, with a default of 10, and 0 lets you resize it by hand.
9. A pick list of repositories chooses the active repository (`scm.setActiveProvider`).

### SB-4.29
1. Open Repository asks you to pick a folder and adds the repository in it to the list (`git.openRepository`).
2. Right-clicking a repository header offers Close Repository and Close Other Repositories (`git.close`, `git.closeOtherRepositories`).
3. Reopen Closed Repositories... opens a pick list that says "Pick a repository to reopen" with an All Repositories choice (`git.reopenClosedRepositories`).
4. Right-clicking a repository header offers Open in External Terminal or Open in Integrated Terminal, and `terminal.sourceControlRepositoriesKind` chooses which show, with values `external`, `integrated` and `both` (`scm.openInTerminal`, `scm.openInIntegratedTerminal`).
5. Setting `git.autoRepositoryDetection` chooses when repositories are found by themselves, with values `true`, `false`, `subFolders` and `openEditors`.
6. Settings `git.scanRepositories` and `git.ignoredRepositories` add folders to search and repositories to skip.
7. Settings `git.repositoryScanMaxDepth` and `git.repositoryScanIgnoredFolders` set how deep the scan goes and which folders it skips, with `node_modules` skipped by default.
8. Settings `git.detectSubmodules` and `git.detectSubmodulesLimit` choose whether submodules are listed as repositories and how many.
9. Rescan Repositories searches the folders again for repositories, and Reopen Closed Repositories is a button on the repository header title. [cursor only]

### SB-4.30
1. Setting `scm.repositories.explorer` turns on an experimental Repositories view that shows Branches, Stashes, Tags and Worktrees groups under the selected repository. [stock only]
2. Setting `scm.repositories.selectionMode`, with values `multiple` and `single`, must be `single` for those groups to show. [stock only]
3. Each group header has its own create button, such as Create Branch, Create Tag or Create Worktree (`git.repositories.createBranch`, `git.repositories.createTag`, `git.repositories.createWorktree`). [stock only]
4. Right-clicking a branch offers Checkout, Checkout (Detached), Merge, Rebase, Delete Branch and Compare with... (`git.repositories.checkout`, `git.repositories.checkoutDetached`, `git.repositories.merge`, `git.repositories.rebase`, `git.repositories.deleteBranch`, `git.repositories.compareRef`). [stock only]
5. Right-clicking a stash offers Apply, Pop and Drop (`git.repositories.stashApply`, `git.repositories.stashPop`, `git.repositories.stashDrop`). [stock only]
6. Right-clicking a worktree offers Open and Delete (`git.repositories.openWorktree`, `git.repositories.deleteWorktree`). [stock only]

### SB-4.31
1. With `scm.workingSets.enabled` on, switching branches saves the editors that were open on the branch you leave.
2. Switching to a branch that has a saved set brings back the editors saved for it.
3. Setting `scm.workingSets.default` chooses what a branch with no saved set starts with, with values `empty` (no editors) and `current` (keep the editors that are open now), and a default of `current`.

### SB-4.32
1. Create Worktree... asks which branch or tag to start from ("Select a branch or tag to create the new worktree from") and then asks for the worktree path in an input box with a Select Worktree Destination button (`git.createWorktree`).
2. Picking a branch that is already checked out offers a Create New Branch button instead.
3. After a worktree is created, a prompt offers Open Worktree in Current Window or Open Worktree in New Window (`git.openWorktree`, `git.openWorktreeInNewWindow`).
4. Delete Worktree... asks which worktree to delete, and deleting the one you are in shows "You cannot delete the worktree you are currently in" (`git.deleteWorktree`).
5. Migrate Worktree Changes... asks which worktree to take changes from and brings them into the current one (`git.migrateWorktreeChanges`).
6. The worktree commands are reachable from the Worktrees submenu, the repository header right-click menu and the Command Palette.
7. Settings `git.detectWorktrees` and `git.detectWorktreesLimit` choose whether existing worktrees are listed as repositories and how many, with a default of off and 50.
8. Setting `git.worktreeIncludeFiles` lists glob patterns for ignored files and folders that are copied into a new worktree. [stock only]
9. Delete Worktree is also available from the Command Palette (`git.deleteWorktreeFromPalette`). [cursor only]

### SB-4.33
1. Repositories that live under the Cursor worktrees folder in your home folder are hidden from the repository lists unless one of them is the open workspace. [cursor only]
2. Setting `git.showCursorWorktrees` on lists those repositories along with your others. [cursor only]

### SB-4.34
1. The Graph view draws the commit history as a list with graph lines and a row per commit, with branch and tag badges on the right.
2. Title bar buttons pick the repository, when there are several, and the refs shown, which are the current branch, all branches or chosen refs (`workbench.scm.action.graph.pickRepository`, `workbench.scm.action.graph.pickHistoryItemRefs`).
3. A Go to Current History Item button scrolls the Graph to the commit you have checked out (`workbench.scm.action.graph.revealCurrentHistoryItem`).
4. Title bar buttons refresh the Graph, fetch, pull, and push or publish the branch (`workbench.scm.action.graph.refresh`, `git.fetchAll`, `git.pullRef`, `git.pushRef`, `git.publish`).
5. Scrolling to the end of the list loads the next page when `scm.graph.pageOnScroll` is on, and `scm.graph.pageSize` sets the page size, with a default of 50.
6. Setting `scm.graph.badges` chooses which badges show, with values `all` and `filter`, and a default of `filter`.
7. Hovering a commit row shows a hover card with that commit's details.
8. In Cursor the commit hover card also has a quick action row with copy hash and items from the commit hover menu. [cursor only]
9. The Graph title bar has View as List and View as Tree buttons (`workbench.scm.action.graph.setListViewMode`, `workbench.scm.action.graph.setTreeViewMode`). [stock only]
10. In Cursor the Graph view sits in third place in the Source Control container, behind Agent Review. [cursor only]

### SB-4.35
1. Commits you could pull show as an Incoming Changes entry next to the history in the Graph. [stock only]
2. Commits you could push show as an Outgoing Changes entry next to the history in the Graph. [stock only]
3. Settings `scm.graph.showIncomingChanges` and `scm.graph.showOutgoingChanges` hide those entries when turned off, and both default to on. [stock only]

### SB-4.36
1. Right-clicking a commit row in the Graph shows a menu of actions for that commit.
2. Open Changes opens the commit's changed files in one multi-file diff (`workbench.scm.action.graph.viewChanges`).
3. Checkout (Detached) switches to that commit without a branch (`git.graph.checkoutDetached`).
4. Create Branch... and Create Tag... start a new branch or tag at that commit (`git.branch`, `git.createTag`).
5. Cherry Pick applies the commit to the current branch, shows a message when the changes are already present or when conflicts stop it, and Abort Cherry Pick ends a stopped one (`git.graph.cherryPick`, `git.cherryPickAbort`).
6. Copy Commit Hash and Copy Commit Message put the hash or the message on the clipboard (`git.copyCommitId`, `git.copyCommitMessage`).
7. Open on GitHub opens the commit's page on GitHub (`github.graph.openOnGitHub`).
8. Right-clicking a branch or tag badge offers Checkout and Delete Branch or Delete Tag, and delete asks for confirmation first (`git.graph.checkout`, `git.graph.deleteBranch`, `git.graph.deleteTag`).
9. Compare with..., Compare with Remote and Compare with Merge Base open a comparison of the commit with another ref (`git.graph.compareRef`, `git.graph.compareWithRemote`, `git.graph.compareWithMergeBase`). [stock only]
10. In Cursor Open Changes needs `multiDiffEditor.experimental.enabled` on, and the menu adds Open in Cursor Blame when the `cursor_blame` feature gate allows it (`workbench.scm.action.graph.openInCursorBlame`). [cursor only]

### SB-4.37
1. The Changes view title bar has a Commit check button and a Refresh button beside the "..." menu (`git.commit`, `git.refresh`).
2. Refresh reads the repository status again and updates the changed file list.
3. The "..." menu lists Pull, Push, Clone, Checkout to... and Fetch at the top, followed by eight submenus for commit, changes, pull and push, branch, remote, stash, tags and worktrees.
4. Show Git Output at the bottom of the "..." menu opens the Git output channel that lists the Git commands that ran (`git.showOutput`).
5. Setting `git.showProgress` on shows progress while Git actions run.

### SB-4.38
1. A diff icon button named Review Branch appears in the Changes view title bar when a repository exists (`workbench.scm.action.openReviewChangesPane`). [cursor only]
2. Clicking it opens a Review Changes editor tab that shows the branch against the default branch. [cursor only]
3. The wording reads Diff with Default Branch, or Diff with the branch name. [cursor only]

### SB-4.39
1. The Agent Review view in the Source Control container has a split Start Agent Review button. [cursor only]
2. While a review runs the button reads Stop review. [cursor only]
3. A status line under the button tells how the review is going. [cursor only]
4. A list in the view shows the issues the review found. [cursor only]
5. A Fix All Issues button sends the found issues to chat. [cursor only]
6. A gear in the view title opens the Agent Review settings (`workbench.scm.action.openBugbotSettings`). [cursor only]
7. Start Agent Review can also be run from the Command Palette, and `cursor.openBugbotPane` opens the Agent Review view (`workbench.scm.action.triggerAgentReview`, `cursor.runEditorBugbot`). [cursor only]
8. Setting `autoRunBugbotOnCommit` on starts a review by itself after you make a local commit that is not yet pushed. [cursor only]

### SB-4.40
1. With no folder open, the view shows welcome text with Open Folder and Clone Repository buttons (`vscode.openFolder`, `git.cloneRecursive`).
2. With a workspace that has no folders, the welcome text offers Add Folder to Workspace and Clone Repository buttons (`workbench.action.addRootFolder`).
3. With a folder open that is not a repository, the welcome text offers an Initialize Repository button (`git.init`).
4. Until Git has finished starting, the view shows "Scanning folder for Git repositories..." or "Scanning workspace for Git repositories...".
5. When a repository is found in a parent folder, the welcome text offers an Open Repository button, and with several it lets you choose (`git.openRepositoriesInParentFolders`).
6. When Git refuses a repository as unsafe, the welcome text offers a Manage Unsafe Repositories button (`git.manageUnsafeRepositories`).
7. When repositories were closed, the welcome text offers a Reopen Closed Repositories button (`git.reopenClosedRepositories`).
8. When Git is not installed the view shows text for the platform with a Download Git link and reload and troubleshoot links.
9. With setting `git.enabled` off the view asks you to enable Git in your settings.
10. In Restricted Mode the view says none of the source control providers work there, with a Manage Workspace Trust button (`workbench.trust.manage`), and with nothing else to show it reads "No source control providers registered."

### SB-4.41
1. Running Git: Rename from the Command Palette opens an input box holding the active file's path with the file name part selected (`git.rename`).
2. Confirming the new path moves the file with `git mv`, closes the old tab and opens the renamed file.
3. The command only appears in the Command Palette when the active file is inside an open repository.

### SB-4.42
1. Running Git: Delete from the Command Palette removes the file in the active editor with `git rm` when it has no uncommitted changes (`git.delete`). [stock only]
2. A file with uncommitted changes is not removed, and an information message says Git: Delete can only be performed on committed files without uncommitted changes. [stock only]
3. The tab of the deleted file closes when it has no unsaved edits. [stock only]
4. The command only appears in the Command Palette when the active file is inside an open repository and uses the file scheme. [stock only]
5. Setting `git.confirmCommittedDelete` is declared, with a default of on, but nothing reads it, so no confirmation shows. [stock only]

### SB-4.43
1. Running Git: Close All Diff Editors closes every open diff tab that compares a file with its Git version, even when `git.closeDiffOnOperation` is off (`git.closeAllDiffEditors`).
2. Running Git: Close All Unmodified Editors closes every open text or notebook tab whose file has no Git changes and no unsaved edits (`git.closeAllUnmodifiedEditors`).
3. Both commands are listed in the Command Palette.

### SB-4.44
1. A Git command that needs a username opens a prompt at the top of the window with a text box for the username and then a masked box for the password, titled with the host.
2. A command that needs an SSH key passphrase opens a prompt with a masked box.
3. A connection to an unknown SSH host opens a yes or no pick that asks "Are you sure you want to continue connecting?"
4. For github.com with `github.gitAuthentication` on, the signed-in GitHub account supplies the username and token, so no prompt shows.
5. Setting `git.useIntegratedAskPass` on makes Git ask in the window instead of the terminal.
6. Setting `git.terminalAuthentication` on makes the same window prompt serve Git commands run in the integrated terminal, after the terminal is restarted.

### SB-4.45
1. Opening a link of the form `scheme://vscode.git/clone?url=REPO_URL` starts the Clone flow for that URL (`git.clone`).
2. Adding `&ref=BRANCH` to the link clones the repository at that ref.
3. A link with a bad URL, scheme or ref is ignored, and a bad ref shows "Invalid reference."
4. When Git is missing, the link shows a modal that says it could not clone because Git is not installed, with a Download Git button.
5. The destination folder and the prompt after cloning follow `git.defaultCloneDirectory` and `git.openAfterClone`.
6. In Cursor the link scheme is `cursor`, so the link starts with `cursor://`. [cursor only]

### SB-4.46
1. When a Git repository is found in a folder above the workspace or an open file, an information message asks whether to open it, with Yes, Always and Never buttons.
2. Yes runs Open Repositories In Parent Folders, which opens a pick list titled "Pick a repository to open" with an All Repositories choice when there are several (`git.openRepositoriesInParentFolders`).
3. Always or Never saves the choice in setting `git.openRepositoryInParentFolders`, which has values `prompt`, `always` and `never`, with a default of `prompt`.

### SB-4.47
1. When Git cannot be found while a workspace folder is a Git repository, a warning reads "Git not found. Install it or configure it using the git.path setting" with a Download Git button.
2. When the installed Git is older than 2, a warning reads "Code works best with git >= 2" with an Update Git button.
3. On Windows with Git 2.25 or 2.26, a warning says there are known issues and to update to Git 2.27 or later, with an Update Git button.
4. Each warning has a Don't Show Again button that sets its ignore setting (`git.ignoreMissingGitWarning`, `git.ignoreLegacyWarning`, `git.ignoreWindowsGit27Warning`).
5. Setting `git.path` gives the path to the Git program, either as one path or as a list of paths to try.

### SB-4.48
1. When Git refuses a repository because its folder belongs to another user and other repositories are open, an error message says the repository is potentially unsafe, with Manage Unsafe Repositories and Learn More buttons.
2. Manage Unsafe Repositories opens a pick list titled "Pick a repository to mark as safe and open" with an All Repositories choice when there are several, and the pick is marked safe and opened (`git.manageUnsafeRepositories`).
3. The Manage Unsafe Repositories command appears in the Command Palette only while an unsafe repository is known.

### SB-4.49
1. Each commit row in the Graph can be expanded to list the files that commit changed, and rows start collapsed. [stock only]
2. The files of an expanded commit show as a flat list or as a folder tree, following the Graph's View as List or View as Tree choice (`workbench.scm.action.graph.setListViewMode`, `workbench.scm.action.graph.setTreeViewMode`). [stock only]
3. Clicking a file of an expanded commit opens its diff against the parent commit, and an added file opens the file while a deleted file opens the old version. [stock only]
4. An inline Open File button on a file row opens the file as it was at that commit (`workbench.scm.action.graph.openFile`). [stock only]
5. The Incoming Changes and Outgoing Changes entries expand the same way to list their files. [stock only]
6. Setting `scm.compactFolders` on joins single child folders into one row in the expanded folder tree. [stock only]

### SB-4.50
1. Clicking a commit row in the Graph opens that commit's changed files in one multi-file diff editor titled with the commit (`_workbench.openMultiDiffEditor`). [cursor only]
2. When the `cursor_blame` feature gate and the team setting are on, clicking a commit row opens a Cursor Blame editor for that commit instead. [cursor only]

### SB-4.51
1. Dragging a changed file from the Changes view onto an editor area opens that file.
2. Dragging several selected files carries all of them together.
3. The drag label shows the file name for one file and the number of files for several.
4. Group header and folder rows cannot be dragged, only file rows.

### SB-4.52
1. Running Focus Next Input or Focus Previous Input opens the Changes view if needed and puts the cursor in the commit message box of the active repository.
2. When the box already has focus and several repositories are visible, the same command moves to the next or previous repository's box and wraps around at the ends.
3. With one visible repository and focus already in its box, nothing happens.
4. Focus Next Resource Group or Focus Previous Resource Group moves focus into the file list and selects the first visible group, such as Staged Changes or Changes, then steps to the next or previous visible group and wraps around.
5. Groups that are not shown are skipped.
6. None of the four does anything when no repository is visible, and none has a default key or a menu row, so they show up only once a key is assigned.

### SB-4.53
1. A modified or type-changed file is drawn in `gitDecoration.modifiedResourceForeground` (dark #E2C08D, light #895503), and a file modified in the index uses `gitDecoration.stageModifiedResourceForeground`, which starts with the same values so the two can be recolored apart.
2. An added file (or one marked intent to add) uses `gitDecoration.addedResourceForeground` (dark #81b88b, light #587c0c), while a renamed or copied file uses `gitDecoration.renamedResourceForeground` and an untracked file uses `gitDecoration.untrackedResourceForeground`, and those last two both start at dark #73C991 and light #007100.
3. A file deleted in the working tree uses `gitDecoration.deletedResourceForeground` (dark #c74e39, light #ad0707), and a file deleted in the index uses `gitDecoration.stageDeletedResourceForeground` with the same starting values.
4. Every conflict state (both deleted, added or deleted by us or by them, both added, both modified) uses `gitDecoration.conflictingResourceForeground` (dark #e4676b, light #ad0707) and carries a `!` badge.
5. A file that .gitignore covers is greyed with `gitDecoration.ignoredResourceForeground` (dark #8C8C8C, light #8E8E90), and this grey is still applied when `git.decorations.enabled` is off.
6. A submodule folder gets an `S` badge with the tooltip Submodule and the color `gitDecoration.submoduleResourceForeground` (dark #8db9e2, light #1258a7).
7. The color shows on the name and the badge letter of rows in the Explorer and the Open Editors view and on the title of editor tabs, and a changed file also colors the folders above it and puts a small dot on them, except when the change is a deletion.
8. Setting `git.decorations.enabled` (default on) removes Git colors and badges from the Explorer and the Open Editors view when off, and the settings `explorer.decorations.colors` and `workbench.editor.decorations.colors` (both default on) turn off name colors in those views and in editor tab titles.
9. In the Source Control list each file shows its status letter in the status color while the file name keeps the normal text color, and with `git.decorations.enabled` off the list shows status icons instead of letters.
10. There is no settings page for these colors, so a person changes them with `workbench.colorCustomizations` or a theme, and each token has its own defaults for light, dark, high contrast and high contrast light themes.

## Unclear

- SB-4.7 has more behaviors than 10 pieces can hold (also `git.requireGitUserConfig`, `git.promptToSaveFilesBeforeCommit` details and the message box toolbar), so it should be split.
- SB-4.28 piece 5 joins the repository headers and the collapse and expand buttons because they appear together, and the exact conditions for each were not read.
- SB-4.29 and SB-4.32 describe the Cursor-added commands (`git.rescan`, `git.deleteWorktreeFromPalette`) only from the Cursor manifest, not from a run on screen.
- SB-4.34 piece 10 and SB-4.39 rest on bundle strings, so the Agent Review view may not show for every user because of its feature gate.
- SB-4.40 welcome texts were read from stock 1.128.0 wording, and the Cursor manifest has 19 entries against 17, so the extra two were not compared piece by piece.
- Cursor tags on SB-4.7, SB-4.19, SB-4.20, SB-4.34 and SB-4.36 follow the Cursor field of the feature and were not checked on screen.
