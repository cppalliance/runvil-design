Group: SB-4 Source Control. Section: Side bar. Date: 2026-10-25. Pass: 2.
Features: 53.

### SB-4.1 View changed files in groups
- Tag: both
- Evidence: source
- Boundary: no
- What: Open the Source Control view (Changes) and see changed files sorted into Merge Changes, Staged Changes, Changes and Untracked Changes groups, each with a count, status letters and inline buttons.
- Commands: workbench.view.scm
- Menus: scm/resourceGroup/context
- Keys: Ctrl+Shift+G (Control+Shift+G on Mac) opens the view
- Settings: git.untrackedChanges; git.alwaysShowStagedChangesResourceGroup; git.statusLimit; git.ignoreLimitWarning; git.ignoreSubmodules; git.autorefresh; scm.alwaysShowActions; scm.maxRenderedResources; git.similarityThreshold
- Cursor: changed (adds scm.maxRenderedResources, default 5000 rendered files per group, not in stock)
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts; V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts; V\extensions\git\src\repository.ts (resource groups merge, index, workingTree, untracked); V\extensions\git\package.json; C bundle (view id workbench.scm, key Ctrl+Shift+G in openCommandActionDescriptor, setting scm.maxRenderedResources with limitRenderedChildren)

### SB-4.2 Stage or unstage changes
- Tag: both
- Evidence: source
- Boundary: no
- What: Move one file, a folder, or a whole group between Changes and Staged Changes with the plus and minus buttons or the right-click menu.
- Commands: git.stage; git.unstage; git.stageAll; git.unstageAll; git.stageAllTracked; git.stageAllUntracked; git.stageAllMerge; git.stageFile; git.unstageFile
- Menus: scm/resourceState/context; scm/resourceFolder/context; scm/resourceGroup/context; git.changes; multiDiffEditor/resource/title; SCMResourceContext; SCMResourceFolderContext
- Keys: none found
- Settings: git.untrackedChanges; git.optimisticUpdate
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json (same 46 scm/resourceState/context items); C\extensions\git\dist\main.js (git.stage, git.unstage, git.stageAll found)

### SB-4.3 Discard changes
- Tag: both
- Evidence: source
- Boundary: no
- What: Throw away edits to a file, a folder or a whole group from the right-click menu or the inline undo button, after a confirmation; untracked files can go to the Recycle Bin.
- Commands: git.clean; git.cleanAll; git.cleanAllTracked; git.cleanAllUntracked
- Menus: scm/resourceState/context; scm/resourceFolder/context; scm/resourceGroup/context; git.changes
- Keys: none found
- Settings: git.discardUntrackedChangesToTrash; git.untrackedChanges
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.clean, git.cleanAll, git.cleanAllUntracked found)

### SB-4.4 Open a diff from the list
- Tag: both
- Evidence: source
- Boundary: no
- What: Click a changed file to open its diff, use the inline Open File button, Open File (HEAD) or Compare with Workspace, or open a whole group in one multi-file diff with the group's View Changes button.
- Commands: git.openChange; git.openFile; git.openFile2; git.openHEADFile; git.compareWithWorkspace; git.openAllChanges; git.viewChanges; git.viewStagedChanges; git.viewUntrackedChanges
- Menus: scm/resourceState/context; scm/resourceGroup/context
- Keys: none found
- Settings: git.openDiffOnClick; git.showInlineOpenFileAction; scm.autoReveal; git.closeDiffOnOperation
- Cursor: changed (group View Changes buttons are gated on config.multiDiffEditor.experimental.enabled, which defaults to true in the C bundle)
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts; C\extensions\git\package.json; C bundle (setting multiDiffEditor.experimental.enabled, default true)

### SB-4.5 Arrange the list: list or tree, and sort
- Tag: both
- Evidence: source
- Boundary: no
- What: Switch the changes list between a flat list and a folder tree, pick the sort order (name, path or status) from the View and Sort menu, and collapse or compact folders.
- Commands: workbench.scm.action.setListViewMode; workbench.scm.action.setTreeViewMode; workbench.scm.action.setListViewModeNavigation; workbench.scm.action.setTreeViewModeNavigation; workbench.scm.action.setSortKey.name; workbench.scm.action.setSortKey.path; workbench.scm.action.setSortKey.status; workbench.scm.action.collapseAll
- Menus: scm/title (View and Sort submenu and the navigation toggle); SCMResourceGroupContext
- Keys: none found
- Settings: scm.defaultViewMode; scm.defaultViewSortKey; scm.compactFolders
- Cursor: present
- Source: V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (lines 953-1360); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts; C bundle (workbench.scm.action.setListViewMode, setTreeViewMode, setSortKey, collapseAll; settings scm.defaultViewMode, scm.defaultViewSortKey, scm.compactFolders)

### SB-4.6 Reveal a changed file or add it to .gitignore
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a changed file to show it in the Explorer view or the operating system file manager, or to add it to .gitignore.
- Commands: git.revealInExplorer; git.revealFileInOS.windows; git.revealFileInOS.mac; git.revealFileInOS.linux; git.ignore
- Menus: scm/resourceState/context; scm/resourceFolder/context
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.revealInExplorer, git.ignore found)

### SB-4.7 Type a message and commit
- Tag: both
- Evidence: source
- Boundary: no
- What: Type a message in the box at the top of the Changes view and press Ctrl+Enter or the Commit button; the box can grow, warn about long lines, ask about smart commit when nothing is staged, and prompt before committing on a protected branch.
- Commands: scm.acceptInput; git.commit; scm.clearValidation; scm.clearInput
- Menus: scm/title; SCMInputBox
- Keys: Ctrl+Enter (scm.acceptInput, when scmRepository); Escape (scm.clearInput, when no suggest widget and no validation message); Escape (scm.clearValidation, when a validation message is shown)
- Settings: git.showCommitInput; git.enableSmartCommit; git.smartCommitChanges; git.suggestSmartCommit; git.requireGitUserConfig; git.promptToSaveFilesBeforeCommit; git.inputValidation; git.inputValidationLength; git.inputValidationSubjectLength; git.branchProtection; git.branchProtectionPrompt; git.diagnosticsCommitHook.enabled; git.diagnosticsCommitHook.sources; scm.inputFontFamily; scm.inputFontSize; scm.inputMaxLineCount; scm.inputMinLineCount
- Cursor: changed (scm.clearValidation not found; the diagnostics hook settings are spelled git.diagnosticsCommitHook.Enabled and .Sources with capitals)
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 420-475); V\src\vs\workbench\contrib\scm\browser\scmInput.ts; V\extensions\git\src\commands.ts (commitWithAnyInput, lines 2372-2576); V\extensions\git\src\repository.ts (acceptInputCommand line 998, placeholder line 3284); V\extensions\git\src\diagnostics.ts; V\extensions\git\src\postCommitCommands.ts; C bundle (scm.acceptInput, scm.clearInput, settings scm.inputFontFamily and the rest); C\extensions\git\dist\main.js (diagnosticsCommitHook.Enabled)

### SB-4.8 Use the action button under the message box
- Tag: both
- Evidence: source
- Boundary: no
- What: A wide button below the message box reads Commit, Continue (during a merge or rebase), Sync Changes with ahead and behind counts, or Publish Branch depending on the state, and the Commit button has a drop-down for Commit and Push, Commit and Sync, or Commit (Amend).
- Commands: git.commit; git.sync; git.publish
- Menus: none found (the git extension supplies it through the SourceControlActionButton API)
- Keys: none found
- Settings: git.showActionButton; scm.showActionButton; git.postCommitCommand; git.rememberPostCommitCommand
- Cursor: present
- Source: V\extensions\git\src\actionButton.ts; V\extensions\git\src\postCommitCommands.ts; V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts; C\extensions\git\dist\main.js (strings Sync Changes and Publish Branch found); C\extensions\git\package.json (git.postCommitCommand, git.showActionButton)

### SB-4.9 Choose how to commit
- Tag: both
- Evidence: source
- Boundary: no
- What: From the Commit submenu in the "..." menu, commit only staged changes, commit all changes, sign off, skip the hooks (No Verify) or make an empty commit.
- Commands: git.commitStaged; git.commitAll; git.commitSigned; git.commitStagedSigned; git.commitAllSigned; git.commitNoVerify; git.commitStagedNoVerify; git.commitAllNoVerify; git.commitEmpty; git.commitAllSignedNoVerify; git.commitEmptyNoVerify; git.commitSignedNoVerify; git.commitStagedSignedNoVerify
- Menus: git.commit; scm/title
- Keys: none found
- Settings: git.alwaysSignOff; git.enableCommitSigning; git.allowNoVerifyCommit; git.confirmNoVerifyCommit; git.confirmEmptyCommits
- Cursor: present
- Source: V\extensions\git\package.json (git.commit submenu, 20 items); V\extensions\git\src\commands.ts; C\extensions\git\package.json (same 20 items); C\extensions\git\dist\main.js (git.commitStaged, git.commitAll, git.commitSigned, git.commitNoVerify, git.commitEmpty found)

### SB-4.10 Amend the last commit
- Tag: both
- Evidence: source
- Boundary: no
- What: Rewrite the previous commit with the current message and changes using Commit (Amend), Commit Staged (Amend) or Commit All (Amend).
- Commands: git.commitAmend; git.commitStagedAmend; git.commitAllAmend; git.commitAmendNoVerify; git.commitStagedAmendNoVerify; git.commitAllAmendNoVerify
- Menus: git.commit; action button drop-down (SourceControlActionButton secondary commands)
- Keys: none found
- Settings: git.allowNoVerifyCommit
- Cursor: present
- Source: V\extensions\git\package.json (git.commit submenu groups 3_amend and 4_amend_noverify); V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.commitAmend found)

### SB-4.11 Undo the last commit
- Tag: both
- Evidence: source
- Boundary: no
- What: Undo Last Commit takes the newest commit back into the working tree and puts its message in the message box.
- Commands: git.undoCommit
- Menus: git.commit
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\git\package.json (git.commit submenu, group 1_commit at 4); V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.undoCommit found)

### SB-4.12 Write the commit message in a full editor
- Tag: both
- Evidence: source
- Boundary: no
- What: With the setting on (the default), committing with an empty message box opens git's commit message in a full editor tab with Accept and Cancel buttons, and the message template can be restored.
- Commands: git.commitMessageAccept; git.commitMessageDiscard; git.restoreCommitTemplate
- Menus: editor/content (stock); editor/title (Cursor)
- Keys: none found
- Settings: git.useEditorAsCommitInput; git.verboseCommit; git.terminalGitEditor
- Cursor: changed (the Accept and Cancel buttons are declared under editor/title, not editor/content)
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts (line 2461); V\extensions\git\src\gitEditor.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.commitMessageAccept, git.restoreCommitTemplate found)

### SB-4.13 Browse earlier commit messages
- Tag: both
- Evidence: source
- Boundary: no
- What: In the message box, press Up or Down at the first or last line, or Alt+Up and Alt+Down anywhere, to step through messages you typed before.
- Commands: scm.viewPreviousCommit; scm.viewNextCommit; scm.forceViewPreviousCommit; scm.forceViewNextCommit
- Menus: none found
- Keys: Up (scm.viewPreviousCommit, when scmRepository and scmInputIsInFirstPosition and no suggest widget); Down (scm.viewNextCommit, when scmInputIsInLastPosition); Alt+Up (scm.forceViewPreviousCommit); Alt+Down (scm.forceViewNextCommit)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 483-537); V\src\vs\workbench\contrib\scm\browser\scmInput.ts (showNextHistoryValue); C bundle (all four ids found, context key scmInputIsInLastPosition found, labels Source Control: View Next Commit and View Previous Commit in the NLS file)

### SB-4.14 Generate a commit message
- Tag: cursor
- Evidence: bundle
- Boundary: yes (a model reads the staged diff and recent commit messages and writes the text, so the result comes from the agent side)
- What: A sparkle button in the message box fills it with a generated commit message, and the same action is in the Command Palette.
- Commands: cursor.generateGitCommitMessage; scm.input.cancelAction
- Menus: scm/inputBox; commandPalette
- Keys: none found
- Settings: scm.showInputActionButton
- Cursor: cursor only
- Source: C\extensions\cursor-always-local\package.json (contributes.menus scm/inputBox with when scmProvider == git, enabled API proposal contribSourceControlInputBoxMenu); C bundle (command class with id cursor.generateGitCommitMessage, title Generate Commit Message, icon sparkle, f1 true; menu id SCMInputBox registered); V\src\vs\workbench\contrib\scm\browser\scmInput.ts (the input toolbar that hosts the button, shown only when scm.showInputActionButton is on)

### SB-4.15 Switch branches
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick Checkout to... from the title bar menu to switch to a local branch, remote branch or tag, or check out a detached commit.
- Commands: git.checkout; git.checkoutDetached
- Menus: scm/title (group 1_header)
- Keys: none found
- Settings: git.checkoutType; git.branchSortOrder; git.showReferenceDetails; git.pullBeforeCheckout
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.checkout, git.checkoutDetached found)

### SB-4.16 Create, rename or delete a branch
- Tag: both
- Evidence: source
- Boundary: no
- What: Make a new branch from the current commit or from a chosen ref, rename the current branch, or delete a local or remote branch, with naming rules from settings.
- Commands: git.branch; git.branchFrom; git.renameBranch; git.deleteBranch; git.deleteRemoteBranch
- Menus: git.branch; scm/title
- Keys: none found
- Settings: git.branchPrefix; git.branchValidationRegex; git.branchWhitespaceChar; git.branchRandomName.enable; git.branchRandomName.dictionary; git.defaultBranchName
- Cursor: present
- Source: V\extensions\git\package.json (git.branch submenu, 8 items); V\extensions\git\src\commands.ts; C\extensions\git\package.json (same 8 items); C\extensions\git\dist\main.js (git.branch, git.branchFrom, git.deleteBranch, git.renameBranch found)

### SB-4.17 Merge a branch
- Tag: both
- Evidence: source
- Boundary: no
- What: Merge another branch into the current one from the Branch submenu, and abort a merge that is in progress.
- Commands: git.merge; git.mergeAbort
- Menus: git.branch (group 1_merge)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.merge, git.mergeAbort found)

### SB-4.18 Rebase a branch
- Tag: both
- Evidence: source
- Boundary: no
- What: Rebase the current branch onto another from the Branch submenu, and abort a rebase that is in progress with Abort Rebase in the Commit submenu.
- Commands: git.rebase; git.rebaseAbort
- Menus: git.branch (group 1_merge); git.commit (group 1_commit)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.rebase, git.rebaseAbort found)

### SB-4.19 Work with tags
- Tag: both
- Evidence: source
- Boundary: no
- What: Create a tag, delete a local or remote tag, and push tags, from the Tags submenu or the Command Palette.
- Commands: git.createTag; git.deleteTag; git.deleteRemoteTag; git.pushTags; git.pushWithTags; git.pushWithTagsForce
- Menus: git.tags; scm/title
- Keys: none found
- Settings: git.pullTags; git.replaceTagsWhenPull; git.followTagsWhenSync
- Cursor: changed (the Tags submenu has 3 items against 4, Push Tags is missing from the menu, but the command id is declared)
- Source: V\extensions\git\package.json (git.tags submenu); V\extensions\git\src\commands.ts; C\extensions\git\package.json (git.tags has 3 items); C\extensions\git\dist\main.js (git.createTag, git.deleteTag, git.pushTags found)

### SB-4.20 Stash changes
- Tag: both
- Evidence: source
- Boundary: no
- What: Stash changes (all, including untracked, or staged only), then apply, pop, view or drop a stash, or drop them all.
- Commands: git.stash; git.stashIncludeUntracked; git.stashStaged; git.stashApply; git.stashApplyLatest; git.stashPop; git.stashPopLatest; git.stashDrop; git.stashDropAll; git.stashView
- Menus: git.stash; scm/title
- Keys: none found
- Settings: git.promptToSaveFilesBeforeStash; git.useCommitInputAsStashMessage; git.closeDiffOnOperation
- Cursor: changed (the View Stash item needs config.multiDiffEditor.experimental.enabled, which defaults to true in the C bundle)
- Source: V\extensions\git\package.json (git.stash submenu, 10 items); V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.stash, git.stashPop, git.stashApply, git.stashDrop, git.stashView found); C bundle (git.stash, git.stashIncludeUntracked found)

### SB-4.21 Resolve merge conflicts from the list
- Tag: both
- Evidence: source
- Boundary: no
- What: Conflicted files appear in a Merge Changes group where you can open the file with its conflict markers, accept all current or all incoming changes from the right-click menu, and stage them once resolved.
- Commands: git.stageAllMerge; git.stage; git.openFile; merge-conflict.accept.all-current; merge-conflict.accept.all-incoming
- Menus: scm/resourceGroup/context; scm/resourceState/context
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\merge-conflict\package.json (scm/resourceState/context items for the merge group); V\extensions\git\src\repository.ts (merge group id merge); C\extensions\git\package.json; C\extensions\merge-conflict\package.json (same two scm/resourceState/context items); C\extensions\git\dist\main.js (git.stageAllMerge found); C\extensions\merge-conflict\dist\mergeConflictMain.js (accept.all-current and accept.all-incoming found)

### SB-4.22 Fetch
- Tag: both
- Evidence: source
- Boundary: no
- What: Fetch from the default remote, fetch with prune, or fetch from all remotes, and optionally let Git fetch on a timer.
- Commands: git.fetch; git.fetchPrune; git.fetchAll
- Menus: scm/title (group 1_header); git.pullpush; scm/history/title
- Keys: none found
- Settings: git.autofetch; git.autofetchPeriod; git.pruneOnFetch; git.fetchOnPull
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; V\extensions\git\src\autofetch.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.fetch, git.fetchPrune, git.fetchAll found)

### SB-4.23 Pull
- Tag: both
- Evidence: source
- Boundary: no
- What: Pull the current branch, pull with rebase, or pull from a chosen remote and branch, optionally stashing changes first.
- Commands: git.pull; git.pullRebase; git.pullFrom
- Menus: scm/title (group 1_header); git.pullpush
- Keys: none found
- Settings: git.autoStash; git.pullTags; git.fetchOnPull; git.ignoreRebaseWarning; git.replaceTagsWhenPull
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.pull, git.pullRebase, git.pullFrom found)

### SB-4.24 Push
- Tag: both
- Evidence: source
- Boundary: no
- What: Push the current branch, push to a chosen remote, or force push (shown only when force push is allowed), with a confirmation for force pushes.
- Commands: git.push; git.pushTo; git.pushForce; git.pushToForce
- Menus: scm/title (group 1_header); git.pullpush
- Keys: none found
- Settings: git.allowForcePush; git.useForcePushWithLease; git.useForcePushIfIncludes; git.confirmForcePush; git.showPushSuccessNotification
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.push, git.pushTo, git.pushForce found); C bundle (git.push found)

### SB-4.25 Sync changes
- Tag: both
- Evidence: source
- Boundary: no
- What: Sync pulls then pushes in one step, from the Sync Changes action button, the Pull, Push submenu, or the Command Palette, with an optional rebase variant and a confirmation.
- Commands: git.sync; git.syncRebase
- Menus: git.pullpush (group 1_sync)
- Keys: none found
- Settings: git.confirmSync; git.rebaseWhenSync; git.followTagsWhenSync; git.supportCancellation
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; V\extensions\git\src\actionButton.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.sync, git.syncRebase found)

### SB-4.26 Publish a branch or manage remotes
- Tag: both
- Evidence: source
- Boundary: no
- What: Publish Branch pushes a local branch for the first time (and offers a provider such as GitHub when there is no remote), and Add Remote and Remove Remote manage the remote list.
- Commands: git.publish; git.addRemote; git.removeRemote; github.publish
- Menus: git.branch (group 4_publish); git.remotes; scm/history/title
- Keys: none found
- Settings: git.showActionButton
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts (publish, line 4349); V\extensions\github\package.json (github.publish and its scm welcome text); C\extensions\git\package.json; C\extensions\github\package.json (github.publish and two scm welcome entries); C\extensions\git\dist\main.js (git.publish, git.addRemote, git.removeRemote found)

### SB-4.27 Clone a repository
- Tag: both
- Evidence: source
- Boundary: no
- What: Clone a repository from a URL from the title bar menu, the Command Palette, or the empty Source Control and Explorer views, including a recursive clone and a prompt to open it afterwards.
- Commands: git.clone; git.cloneRecursive; git.continueInLocalClone
- Menus: scm/title (group 1_header); viewsWelcome (scm and explorer)
- Keys: none found
- Settings: git.defaultCloneDirectory; git.openAfterClone
- Cursor: present
- Source: V\extensions\git\package.json; V\extensions\git\src\cloneManager.ts; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.clone, git.cloneRecursive found); C bundle (git.clone found)

### SB-4.28 See repositories and choose which ones show
- Tag: both
- Evidence: source
- Boundary: no
- What: The Repositories view (hidden by default) lists every repository with its branch, and you tick which ones the Changes view shows, sort them, collapse or expand them, show count badges, and pick the active repository from a list.
- Commands: scm.setActiveProvider; workbench.scm.action.toggleRepositoryVisibility.<provider id>; workbench.scm.action.repositories.setSortKey.discoveryTime; workbench.scm.action.repositories.setSortKey.name; workbench.scm.action.repositories.setSortKey.path; workbench.scm.action.collapseAllRepositories; workbench.scm.action.expandAllRepositories
- Menus: SCMTitle (View and Sort, Repositories submenu); SCMSourceControlTitle; SCMSourceControlInline
- Keys: none found
- Settings: scm.alwaysShowRepositories; scm.repositories.visible; scm.repositories.sortOrder; scm.providerCountBadge
- Cursor: present
- Source: V\src\vs\workbench\contrib\scm\browser\scmRepositoriesViewPane.ts; V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (lines 953-1332); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (view descriptors lines 90-105, settings lines 318-338); C bundle (view id workbench.scm.repositories, the sort, collapse and toggle ids, scm.setActiveProvider, settings found)

### SB-4.29 Open, close and reopen repositories
- Tag: both
- Evidence: source
- Boundary: no
- What: Open a repository by path, close one or all the others from the repository header right-click menu, reopen closed ones, open a repository in a terminal, and tune how Git finds repositories (scan depth, parent folders, ignored paths, submodules); Cursor adds a Rescan Repositories command.
- Commands: git.openRepository; git.close; git.closeOtherRepositories; git.reopenClosedRepositories; git.rescan; git.openRepositoriesInParentFolders; scm.openInTerminal; scm.openInIntegratedTerminal
- Menus: scm/sourceControl; scm/repositories/title (stock); scm/sourceControl/title (Cursor); commandPalette
- Keys: none found
- Settings: git.autoRepositoryDetection; git.scanRepositories; git.ignoredRepositories; git.repositoryScanMaxDepth; git.repositoryScanIgnoredFolders; git.detectSubmodules; git.detectSubmodulesLimit; terminal.sourceControlRepositoriesKind
- Cursor: changed (adds git.rescan; Reopen Closed Repositories sits under scm/sourceControl/title, not scm/repositories/title)
- Source: V\extensions\git\package.json; V\extensions\git\src\model.ts; V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 538-618, terminal items); C\extensions\git\package.json (git.rescan, scm/sourceControl/title); C\extensions\git\dist\main.js (git.openRepository, git.close, git.closeOtherRepositories, git.reopenClosedRepositories, git.rescan found); C bundle (scm.openInTerminal, scm.openInIntegratedTerminal, terminal.sourceControlRepositoriesKind found)

### SB-4.30 Browse branches, tags, stashes and worktrees in the Repositories view
- Tag: vscode
- Evidence: source
- Boundary: no
- What: An experimental mode of the Repositories view shows Branches, Stashes, Tags and Worktrees under the selected repository, each group with its own create button and right-click menu; it works only when repository selection is set to single.
- Commands: git.repositories.checkout; git.repositories.checkoutDetached; git.repositories.createBranch; git.repositories.createTag; git.repositories.merge; git.repositories.rebase; git.repositories.deleteBranch; git.repositories.stashApply; git.repositories.stashPop; git.repositories.stashDrop; git.repositories.createWorktree; git.repositories.openWorktree; git.repositories.deleteWorktree; git.repositories.compareRef; git.repositories.copyBranchName; git.repositories.copyCommitHash; git.repositories.copyStashName; git.repositories.copyTagName; git.repositories.createFrom; git.repositories.deleteTag; git.repositories.openWorktreeInNewWindow; git.repositories.stashCopyBranchName; git.repositories.stashView; git.repositories.worktreeCopyBranchName; git.repositories.worktreeCopyCommitHash; git.repositories.worktreeCopyPath
- Menus: scm/artifactGroup/context; scm/artifact/context; git.repositories.stash; SCMArtifactGroupContext; SCMArtifactContext
- Keys: none found
- Settings: scm.repositories.explorer; scm.repositories.selectionMode
- Cursor: missing
- Source: V\extensions\git\package.json (26 git.repositories.* commands, scm/artifact/context 26 items, scm/artifactGroup/context 4 items); V\extensions\git\src\artifactProvider.ts (groups branches, stashes, tags, worktrees); V\src\vs\workbench\contrib\scm\browser\scmRepositoriesViewPane.ts; V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 339-354); C\extensions\git\package.json (none of these menus or commands); C bundle and C\extensions\git\dist\main.js (scm.repositories.explorer, scm.repositories.selectionMode, scmArtifact, git.repositories. all 0 hits)

### SB-4.31 Remember open editors per branch
- Tag: both
- Evidence: source
- Boundary: no
- What: With the setting on, switching branches saves the editors you had open and brings back the set saved for the branch you switch to; a second setting decides what a branch with no saved set starts with.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: scm.workingSets.enabled; scm.workingSets.default
- Cursor: present
- Source: V\src\vs\workbench\contrib\scm\browser\workingSet.ts; V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 365-379); C bundle (both settings found)

### SB-4.32 Create, open and delete worktrees
- Tag: both
- Evidence: source
- Boundary: no
- What: Create a Git worktree, open one in the current or a new window, delete one, or move changes into a new worktree, from the Worktrees submenu, the repository right-click menu, or the Command Palette.
- Commands: git.createWorktree; git.openWorktree; git.openWorktreeInNewWindow; git.deleteWorktree; git.deleteWorktree2; git.deleteWorktreeFromPalette; git.migrateWorktreeChanges
- Menus: git.worktrees; scm/sourceControl; commandPalette
- Keys: none found
- Settings: git.detectWorktrees; git.detectWorktreesLimit; git.worktreeIncludeFiles
- Cursor: changed (adds git.deleteWorktreeFromPalette; git.deleteWorktree2 and git.worktreeIncludeFiles not found)
- Source: V\extensions\git\package.json (git.worktrees submenu, 4 items); V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js (git.createWorktree, git.openWorktree, git.deleteWorktree, git.migrateWorktreeChanges, deleteWorktreeFromPalette found; git.deleteWorktree2 not found)

### SB-4.33 Show Cursor worktrees as repositories
- Tag: cursor
- Evidence: bundle
- Boundary: yes (the worktrees under the Cursor worktrees folder come from Cursor's agent worktree flow, which is inferred from bundle strings and not confirmed)
- What: By default the git extension hides repositories that live under the Cursor worktrees folder in your home directory unless one is the open workspace, and turning the setting on lists them with your other repositories; the manifest describes it as the Source Control Repositories view.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.showCursorWorktrees
- Cursor: cursor only
- Source: C\extensions\git\package.json (git.showCursorWorktrees, boolean, scope window, default false); C\extensions\git\dist\main.js (shouldHideCursorWorktreePath, isPathUnderCursorWorktrees, path .cursor/worktrees, shouldRepositoryBeIgnored)

### SB-4.34 Browse commit history in the Graph
- Tag: both
- Evidence: source
- Boundary: no
- What: The Graph view draws the commit history of the current branch, all branches or chosen refs with branch and tag badges, loads more as you scroll, has title buttons (repository picker, ref picker, go to current commit, refresh, fetch, pull, push or publish), and shows a hover card for each commit.
- Commands: workbench.scm.action.graph.pickRepository; workbench.scm.action.graph.pickHistoryItemRefs; workbench.scm.action.graph.revealCurrentHistoryItem; workbench.scm.action.graph.refresh; workbench.scm.action.graph.setListViewMode; workbench.scm.action.graph.setTreeViewMode; git.fetchAll; git.pullRef; git.pushRef; git.publish; workbench.scm.action.graph.copyHistoryItemId; workbench.scm.history
- Menus: scm/history/title; scm/historyItem/hover (Cursor); SCMHistoryTitle
- Keys: none found
- Settings: scm.graph.pageOnScroll; scm.graph.pageSize; scm.graph.badges
- Cursor: changed (no list or tree switch; the Graph view sits at order 3 behind Agent Review; the hover gets a quick action row with copy hash plus items from the scm/historyItem/hover menu)
- Source: V\src\vs\workbench\contrib\scm\browser\scmHistoryViewPane.ts (lines 79-290, 483, 1497-1504); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 130-143, 385-405); V\extensions\git\package.json (scm/history/title, 4 items); V\extensions\git\src\historyProvider.ts; C bundle (pickRepository, pickHistoryItemRefs, revealCurrentHistoryItem, refresh found; graph setListViewMode and setTreeViewMode not found; menu id SCMHistoryItemHover and _getHoverActions with copyHistoryItemId; Graph order 3); C\extensions\github\package.json (scm/historyItem/hover)

### SB-4.35 See incoming and outgoing changes in the Graph
- Tag: vscode
- Evidence: source
- Boundary: no
- What: The Graph also shows commits you can pull (Incoming Changes) and commits you can push (Outgoing Changes) next to the history, and two settings hide them.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: scm.graph.showIncomingChanges; scm.graph.showOutgoingChanges
- Cursor: missing
- Source: V\src\vs\workbench\contrib\scm\browser\scmHistoryViewPane.ts (lines 398-400, 2051-2053); V\src\vs\workbench\contrib\scm\common\history.ts (SCMIncomingHistoryItemId); V\src\vs\workbench\contrib\scm\browser\scmViewService.ts (lines 248-249); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 407-416); C bundle (both settings, scm-graph-incoming-changes and the labels Incoming Changes and Outgoing Changes: 0 hits)

### SB-4.36 Right-click a commit or ref in the Graph
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a commit to open its changes, check it out detached, create a branch or tag, cherry-pick it, compare it, open it on GitHub, or copy its hash or message; right-click a branch or tag badge to check it out or delete it.
- Commands: workbench.scm.action.graph.viewChanges; git.graph.checkoutDetached; git.branch; git.createTag; git.graph.cherryPick; git.cherryPick; git.cherryPickAbort; git.graph.compareRef; git.graph.compareWithRemote; git.graph.compareWithMergeBase; git.copyCommitId; git.copyCommitMessage; git.graph.checkout; git.graph.deleteBranch; git.graph.deleteTag; github.graph.openOnGitHub; workbench.scm.action.graph.openInCursorBlame
- Menus: scm/historyItem/context; scm/historyItemRef/context; SCMHistoryItemRefContext
- Keys: none found
- Settings: none found
- Cursor: changed (6 of 9 git commit items, the 3 Compare items are missing; adds Open in Cursor Blame behind the cursor_blame feature gate; Open Changes is gated on config.multiDiffEditor.experimental.enabled)
- Source: V\extensions\git\package.json (scm/historyItem/context 9 items, scm/historyItemRef/context 3 items); V\extensions\github\package.json (github.graph.openOnGitHub); V\src\vs\workbench\contrib\scm\browser\scmHistoryViewPane.ts (lines 288-390); C\extensions\git\package.json (6 and 3 items); C\extensions\git\dist\main.js (git.graph.checkout, git.graph.cherryPick found; git.graph.compareRef, git.graph.compareWithRemote, git.graph.compareWithMergeBase 0 hits); C bundle (workbench.scm.action.graph.viewChanges, workbench.scm.action.graph.openInCursorBlame with checkFeatureGate cursor_blame; NLS labels Open Changes 9351 and Open in Cursor Blame 9353)

### SB-4.37 Refresh the list and read the Git log
- Tag: both
- Evidence: source
- Boundary: no
- What: The title bar of the Changes view has Commit and Refresh buttons and a "..." menu (Pull, Push, Clone, Checkout to, Fetch, eight submenus) whose footer has Show Git Output, which opens the output channel listing the Git commands that ran.
- Commands: git.commit; git.refresh; git.showOutput
- Menus: scm/title
- Keys: none found
- Settings: git.autorefresh; git.showProgress; git.commandsToLog
- Cursor: present
- Source: V\extensions\git\package.json (scm/title, 16 items, and the 8 submenus); V\extensions\git\src\commands.ts; C\extensions\git\package.json (scm/title, same 16 items); C\extensions\git\dist\main.js (git.refresh, git.showOutput found)

### SB-4.38 Review the branch from the title bar
- Tag: cursor
- Evidence: bundle
- Boundary: yes (it opens Cursor's Review Changes editor, which the brief treats as agent adjacent)
- What: A diff icon in the Changes view title bar, named Review Branch, opens a Review Changes editor tab that shows your branch against the default branch (the label reads Diff with Default Branch or Diff with the branch name).
- Commands: workbench.scm.action.openReviewChangesPane
- Menus: SCMTitle (navigation group, order -900, when the view is workbench.scm and a repository exists)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (action class with id workbench.scm.action.openReviewChangesPane, title Review Branch from NLS 9381, icon diff, gitMode diffWithMain, labels from NLS 9382 and 9383)

### SB-4.39 Run an Agent Review
- Tag: cursor
- Evidence: bundle
- Boundary: yes (the review and its found issues come from the agent, and Fix All Issues hands them to chat)
- What: The Agent Review view in the Source Control container has a split Start Agent Review button that turns into Stop review while it runs, a status line, a list of issues, a Fix All Issues button and a gear for Agent Review settings; a review can also start from the Command Palette or automatically after you make a local commit that is not yet pushed.
- Commands: workbench.scm.action.triggerAgentReview; workbench.scm.action.openBugbotSettings; cursor.runEditorBugbot; cursor.openBugbotPane; cursor.enableAutoReviewOnCommit
- Menus: ViewTitle (gear, when the view id starts with bugbot); commandPalette
- Keys: none found
- Settings: autoRunBugbotOnCommit
- Cursor: cursor only
- Source: C bundle (view id bugbot, name Agent Review NLS 9305, order 2, registration class editorBugbotViewGatedRegistration, NLS 9290 Agent Review Settings, NLS 9291 Start Agent Review, CSS classes scm-review-container and scm-review-run-split, autoRunBugbotOnCommit read by name); C\extensions\cursor-commits\dist\main.js (calls cursor.runEditorBugbot after a recent unpushed commit authored by you)

### SB-4.40 Start from an empty Source Control view
- Tag: both
- Evidence: source
- Boundary: no
- What: With no repository the view shows welcome text with buttons: Open Folder, Clone Repository, Initialize Repository, Add Folder to Workspace, Open Repository from a parent folder, Manage Unsafe Repositories and Reopen Closed Repositories, plus messages for scanning, missing Git, disabled Git and Restricted Mode.
- Commands: git.init; git.cloneRecursive; git.openRepositoriesInParentFolders; git.manageUnsafeRepositories; git.reopenClosedRepositories; vscode.openFolder; workbench.action.addRootFolder; workbench.trust.manage
- Menus: viewsWelcome (scm view)
- Keys: none found
- Settings: git.enabled; git.path; git.ignoreMissingGitWarning
- Cursor: present
- Source: V\extensions\git\package.json (viewsWelcome, 17 entries for the scm view); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 70-88); C\extensions\git\package.json (19 viewsWelcome entries, same as stock); C bundle (NLS 9294 No source control providers registered, 9295 Restricted Mode text, workbench.trust.manage); C\extensions\git\dist\main.js (git.init, git.manageUnsafeRepositories, git.openRepositoriesInParentFolders found)

### SB-4.41 Rename a file with Git
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Git: Rename from the Command Palette on the file in the active editor, type the new path in an input box (the file name part is pre-selected), and Git moves the file with git mv, closes the old tab and opens the renamed file.
- Commands: git.rename
- Menus: commandPalette (only when the file is in an open repository)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\git\package.json (command line 323, commandPalette entry line 1349); V\extensions\git\src\commands.ts (rename, lines 1452-1477); V\extensions\git\src\git.ts (args mv, line 2195); C\extensions\git\package.json (git.rename declared); C\extensions\git\dist\main.js (git.rename found)

### SB-4.42 Delete a committed file with Git
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Run Git: Delete from the Command Palette on the file in the active editor to remove it with git rm, which only works when the file has no uncommitted changes (otherwise an information message says so), and the tab closes if it has no unsaved edits; the confirm setting is declared but nothing in the 1.128.0 source reads it, so no confirmation was found.
- Commands: git.delete
- Menus: commandPalette (only when the file is in an open repository and has the file scheme)
- Keys: none found
- Settings: git.confirmCommittedDelete
- Cursor: missing (neither the command nor the setting is in the Cursor git manifest, and dist\main.js has no git.delete string)
- Source: V\extensions\git\package.json (command line 330, commandPalette entry line 1353, setting line 3451); V\extensions\git\src\commands.ts (delete, lines 1479-1512); V\extensions\git\src\repository.ts (rm, line 1303); V\extensions\git\src\git.ts (args rm, line 1979); C\extensions\git\package.json (154 commands, no git.delete, no git.confirmCommittedDelete); C\extensions\git\dist\main.js (0 hits for git.delete and confirmCommittedDelete)

### SB-4.43 Close diff editors or unmodified editors
- Tag: both
- Evidence: source
- Boundary: no
- What: From the Command Palette, Git: Close All Diff Editors closes every open diff tab that compares a file against its Git version (even when git.closeDiffOnOperation is off), and Git: Close All Unmodified Editors closes every open text or notebook tab whose file has no Git changes and no unsaved edits.
- Commands: git.closeAllDiffEditors; git.closeAllUnmodifiedEditors
- Menus: commandPalette
- Keys: none found
- Settings: git.closeDiffOnOperation (the Close All Diff Editors command ignores it)
- Cursor: present
- Source: V\extensions\git\package.json (commands lines 910-918, commandPalette entries line 1705 and after); V\extensions\git\src\commands.ts (closeDiffEditors and closeUnmodifiedEditors, lines 4940-4972); V\extensions\git\src\repository.ts (closeDiffEditors, lines 1667-1690); C\extensions\git\package.json (both ids declared); C\extensions\git\dist\main.js (both ids found)

### SB-4.44 Answer Git credential prompts in the window
- Tag: both
- Evidence: source
- Boundary: no
- What: When a Git command that talks to a remote needs a secret, a prompt opens at the top of the window instead of the terminal: a text box for the username and a masked box for the password (titled Git: host), a masked box for an SSH key passphrase, and a yes or no pick for an unknown SSH host fingerprint; for github.com with the GitHub setting on, the signed-in GitHub account supplies the username and token so no prompt shows (the GitHub sign-in itself is not cataloged here), and the same prompt serves Git run in the integrated terminal.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.useIntegratedAskPass; git.terminalAuthentication; github.gitAuthentication; git.githubAuthentication
- Cursor: present
- Source: V\extensions\git\src\askpass.ts (lines 60-170); V\extensions\git\src\main.ts (Askpass wiring, lines 75-83); V\extensions\git\package.json (settings useIntegratedAskPass and terminalAuthentication); V\extensions\github\src\credentialProvider.ts (github.com only); V\extensions\github\package.json (github.gitAuthentication); C\extensions\git\package.json (both git settings); C\extensions\github\package.json (github.gitAuthentication); C\extensions\git\dist\main.js (strings Passphrase and Are you sure you want to continue connecting? found)

### SB-4.45 Clone from a link
- Tag: both
- Evidence: source
- Boundary: no
- What: Opening a link of the form scheme://vscode.git/clone?url=REPO_URL (optionally with &ref=BRANCH) makes the app start the Clone flow for that URL, ignores a link with a bad URL, scheme or ref, and shows a modal with a Download Git button if Git is missing; the scheme for Cursor is cursor (product.json urlProtocol), and no link was tried on screen.
- Commands: git.clone
- Menus: none found
- Keys: none found
- Settings: git.defaultCloneDirectory; git.openAfterClone
- Cursor: present
- Source: V\extensions\git\src\protocolHandler.ts (whole file, handleUri at line 25); V\extensions\git\src\commands.ts (git.clone, line 1032); C\extensions\git\dist\main.js (registerUriHandler, Could not clone your repository as Git is not installed, Invalid reference. found); C\product.json (urlProtocol cursor)

### SB-4.46 Get asked to open a repository from a parent folder
- Tag: both
- Evidence: source
- Boundary: no
- What: When a Git repository is found in a folder above the workspace or an open file, an information message asks whether to open it with Yes, Always and Never buttons; Yes runs Open Repositories In Parent Folders (a picker with an All Repositories choice when there are several), and Always or Never saves the choice in the setting.
- Commands: git.openRepositoriesInParentFolders
- Menus: none found
- Keys: none found
- Settings: git.openRepositoryInParentFolders
- Cursor: present
- Source: V\extensions\git\src\model.ts (showParentRepositoryNotification, lines 1166-1190; triggers at lines 319-337 and 632-640); V\extensions\git\src\commands.ts (openRepositoriesInParentFolders, line 4976); V\extensions\git\package.json (setting openRepositoryInParentFolders, values prompt, always, never, default prompt); C\extensions\git\package.json (same setting); C\extensions\git\dist\main.js (the parent folders message strings found)

### SB-4.47 Get warned about a missing or old Git
- Tag: both
- Evidence: source
- Boundary: no
- What: A warning message appears when Git cannot be found while a workspace folder is a Git repository ("Git not found. Install it or configure it using the git.path setting", with Download Git), when the installed Git is older than 2 ("Code works best with git >= 2", with Update Git), and on Windows when Git 2.25 or 2.26 is installed ("known issues ... update to Git >= 2.27", with Update Git); each has a Don't Show Again button that sets its ignore setting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.ignoreMissingGitWarning; git.ignoreLegacyWarning; git.ignoreWindowsGit27Warning; git.path
- Cursor: present
- Source: V\extensions\git\src\main.ts (warnAboutMissingGit, lines 159-190; checkGitv1, lines 266-292; checkGitWindows, lines 294-319); V\extensions\git\package.json (the three ignore settings and git.path); C\extensions\git\package.json (same settings); C\extensions\git\dist\main.js (strings Git not found. Install it, Download Git, Update Git, Don't Show Again, Code works best with git >= 2 and known issues with the installed Git found)

### SB-4.48 Get warned about an unsafe repository
- Tag: both
- Evidence: source
- Boundary: no
- What: When Git refuses a repository because its folder is owned by another user and other repositories are already open, an error message says the repository is potentially unsafe, with Manage Unsafe Repositories (a picker, with an All Repositories choice when there are several, that marks the pick as a safe directory and opens it) and Learn More; with no repository open the empty view's welcome text is used instead (SB-4.40).
- Commands: git.manageUnsafeRepositories
- Menus: commandPalette (only while an unsafe repository is known)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\git\src\model.ts (showUnsafeRepositoryNotification, lines 1192-1212; triggers at lines 342 and 657); V\extensions\git\src\commands.ts (manageUnsafeRepositories, lines 5009-5053); V\extensions\git\package.json (palette when git.unsafeRepositoryCount != 0); C\extensions\git\package.json; C\extensions\git\dist\main.js (the potentially unsafe message strings and git.manageUnsafeRepositories found)

### SB-4.49 Expand a commit in the Graph to see its changed files
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Each commit row in the Graph can be expanded (collapsed by default) to list the files it changed, as a flat list or a folder tree following the Graph's View as List or View as Tree choice; clicking a file opens its diff against the parent commit (an added file opens the file, a deleted file opens the old version), and an inline Open File button opens the file as it was at that commit; the Incoming Changes and Outgoing Changes rows expand the same way; the arrow icon is hidden on commit rows (force-no-twistie), so how a person expands a row was inferred from the tree defaults and not observed.
- Commands: workbench.scm.action.graph.openFile; workbench.scm.action.graph.setListViewMode; workbench.scm.action.graph.setTreeViewMode
- Menus: SCMHistoryItemChangeContext (inline and 0_view groups); scm/history/title (9_viewmode group)
- Keys: none found
- Settings: scm.compactFolders
- Cursor: missing (the bundle has no SCMHistoryItemChangeContext menu id and no workbench.scm.action.graph.openFile; it calls provideHistoryItemChanges only from the Open Changes command and the commit click handler, never to fill child rows)
- Source: V\src\vs\workbench\contrib\scm\browser\scmHistoryViewPane.ts (graph.openFile action, lines 368-410; HistoryItemChangeRenderer, lines 649-750; data source getChildren, lines 906-1018; tree options with collapseByDefault and force-no-twistie, lines 1999-2025; _onDidOpen change branch, lines 2044-2086); V\extensions\git\src\historyProvider.ts (provideHistoryItemChanges, line 338); C bundle (SCMHistory menu ids found are Item, ItemCount, ItemHover, ItemRef and Title only; graph.openFile 0 hits)

### SB-4.50 Open a commit's changes by clicking it in the Graph
- Tag: cursor
- Evidence: bundle
- Boundary: no (when the cursor_blame feature gate and the team setting are on, the click opens a Cursor Blame editor instead; whether Cursor Blame output is agent-driven was not decided, as for SB-4.36)
- What: Clicking a commit row in the Graph opens that commit's changed files in one multi-file diff editor titled with the commit, or in a Cursor Blame editor for the commit when the gate and team setting allow it; stock VS Code 1.128.0 has no open handler for commit rows, only for the changed-file rows and the Load More row.
- Commands: _workbench.openMultiDiffEditor
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (the Graph pane's _onDidOpen with a commit branch: checkFeatureGate cursor_blame, isCursorBlameEnabledForTeam, CursorBlameEditorInput, provideHistoryItemChanges, command _workbench.openMultiDiffEditor, scheme scm-history-item); V\src\vs\workbench\contrib\scm\browser\scmHistoryViewPane.ts (_onDidOpen, lines 2044-2092, no commit branch)

### SB-4.51 Drag changed files out of the list
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag one or more changed files from the Changes view and the drag carries them as editor and file data, so dropping onto an editor area opens them; the drag label shows the file name or the count, and only file rows drag, not group or folder rows; other drop targets such as the terminal were not tried.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (SCMTreeDragAndDrop, lines 200-250; dnd option, line 1634); C bundle (the same drag class with getDragURI, onDragStart, getDragLabel and getResourcesFromDragAndDropData found)

### SB-4.52 Move focus between commit boxes and file groups with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: Four commands move keyboard focus around the Changes view, between the commit message boxes of the visible repositories and between the file groups; they have no default key, so a person has to bind them.
- Commands: workbench.scm.action.focusPreviousInput; workbench.scm.action.focusNextInput; workbench.scm.action.focusPreviousResourceGroup; workbench.scm.action.focusNextResourceGroup
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (lines 620-670); V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (focusInput, focusResourceGroup, lines 2060-2145); C bundle (all four ids found, each opening the Source Control view and calling the matching focus method)

### SB-4.53 Git status colors
- Tag: both
- Evidence: source
- Boundary: no
- What: Git paints each changed file in a color for its status (modified, added, deleted, renamed or copied, untracked, ignored, conflict, submodule, and the staged forms of modified and deleted), and a theme or the person can change every one of these colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.decorations.enabled; explorer.decorations.colors; workbench.editor.decorations.colors; workbench.colorCustomizations
- Cursor: present - all ten color ids (gitDecoration.addedResourceForeground, modifiedResourceForeground, deletedResourceForeground, renamedResourceForeground, untrackedResourceForeground, ignoredResourceForeground, stageModifiedResourceForeground, stageDeletedResourceForeground, conflictingResourceForeground, submoduleResourceForeground) and their ten labels were found in the Cursor copy of the Git extension manifest and its nls file; how the Source Control list draws them was not checked in the Cursor bundle
- Source: V\extensions\git\package.json (contributes.colors lines 4149-4260; setting git.decorations.enabled lines 3578-3582); V\extensions\git\package.nls.json (colors.* lines 338-347); V\extensions\git\src\repository.ts (getStatusLetter line 58, getStatusColor lines 118-151, resourceDecoration line 308, useIcons line 3002); V\extensions\git\src\decorationProvider.ts (ignore provider line 27, submodule badge lines 103-106, enable and disable lines 276-300); V\src\vs\workbench\services\decorations\browser\decorationsService.ts (lines 79-120); V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (lines 591 and 636-642); V\src\vs\workbench\browser\workbench.contribution.ts (lines 109-113); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (lines 551-555); C\extensions\git\package.json and package.nls.json

## Handed off

- Count badge on the Source Control icon (`scm.countBadge`, `git.countBadge`) - AB-3
- Status bar branch, sync and publish items (`git.enableStatusBarSync`, `statusbar.ts`) - SS-4
- Git blame status bar item (`git.blame.toggleStatusBarItem`, `git.blame.statusBarItem.*`) - SS-4
- Git blame editor decoration (`git.blame.toggleEditorDecoration`, `git.blame.editorDecoration.*`, Cursor `git.blame.editorDecoration.forceDisabled`) - ED-6
- Line-level staging, unstaging and reverting in the diff editor and gutter (`git.stageSelectedRanges`, `git.unstageSelectedRanges`, `git.revertSelectedRanges`, `git.diff.stageHunk`, `git.diff.stageSelection`, `git.stageChange`, `git.revertChange`, keys Ctrl+K Ctrl+Alt+S, Ctrl+K Ctrl+N, Ctrl+K Ctrl+R) - ED-6
- Editor title buttons for the active file (Stage, Unstage, Open Changes, Open File, stash Apply and Drop in `editor/title`) - ED-6
- Merge editor entry and its Complete Merge button (`git.openMergeEditor`, `git.acceptMerge`, `git.runGitMerge`, `git.runGitMergeDiff3`) and the merge-conflict CodeLens - ED-6
- Cursor's Resolve in Chat button (`git.resolveAllConflictsInChat`, `merge-conflict.resolve-in-chat`) - ED-6 (chat adjacent, boundary)
- Quick diff gutter bars (`scm.diffDecorations*`, `scm/change/title`) - ED-6
- The multi-file diff editor that opens from View Changes, a stash or a Graph commit - ED-6
- The Review Changes editor opened by Review Branch (SB-4.38) - ED-6
- Timeline items from git (`git.timeline.*`, `git.timeline.date`, `git.timeline.showAuthor`, `git.timeline.showUncommitted`) - SB-7
- Git colors and badges in the Explorer and Open Editors (`git.decorations.enabled`) and the Explorer Clone Repository welcome text - SB-2

## Unclear

- Agent Review default visibility: the view registers only behind a feature gate (class editorBugbotViewGatedRegistration) whose default was not read, so it may not show for every user.
- Generate Commit Message placement: the manifest gives only the menu id scm/inputBox, and the input toolbar shows only when `scm.showInputActionButton` is on, so whether the button is inline or in the drop-down was not seen.
- `autoRunBugbotOnCommit` is read by name in the bundle with a default of false and a kill-switch feature gate. Whether it is a settings.json key or a Cursor Settings value was not confirmed.
- Open in Cursor Blame in the Graph needs the `cursor_blame` feature gate and a team setting. Whether Cursor Blame output is agent-driven was not decided, so SB-4.36 is not marked as boundary.
- Cursor's manifest spells the diagnostics commit hook settings `git.diagnosticsCommitHook.Enabled` and `.Sources` (capital letters) and its code reads those spellings, while stock 1.128.0 uses lower case. Whether this is deliberate was not checked.
- Where `git.showCursorWorktrees` shows the extra repositories (Repositories view only, or the Changes view as well) was inferred from the extension code and not observed.
- A debug item in the Graph commit menu, `[Debug] Score Commit for AI Content` (`cursorAiCodeTracking.scoreCommitInternal`, group 1_ai, gated by a debug condition), was seen in the Cursor bundle and left out. SS-6 mentions a recent-commit AI score.
- `newer-than-1.128`: the git setting `git.worktreeSymlinkFolders` exists in N and not in V or the Cursor bundle. The git manifest in N otherwise matches V (183 commands, same menu counts).
- Palette-only git commands without a feature because of the 40 cap: `git.rename` (both), `git.delete` with `git.confirmCommittedDelete` (stock only), `git.closeAllDiffEditors`, `git.closeAllUnmodifiedEditors`, and the three `git.api.*` commands (internal).
- Skipped as Copilot or chat: Graph Add to Chat and Summarize History Item, `scm.editor.triggerSetup` (Resolve Conflicts with AI), `git.addAICoAuthor`.
- The SCM focus commands (`workbench.scm.action.focusNextInput`, `focusPreviousInput`, `focusNextResourceGroup`, `focusPreviousResourceGroup`) have no default key and no menu entry, so they are not a feature. The SCM accessibility help (`scmAccessibilityHelp.ts`) was not cataloged.
- Cursor "present" results rest on the Cursor git extension manifest, its `dist\main.js` and bundle strings. Nothing was run on screen, and menu items gated by `when` clauses were read as written, not observed.
- Overflow check on 2026-10-25 added 11 features (SB-4.41 to SB-4.51).
