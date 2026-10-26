Bucket: J. Date: 2026-10-25. Pass: 5.
EXCL | Git SCM Integration | label | extension-system | description line of the Git extension, shown only in the Extensions view
MAP | Open All Changes | SB-4.4 | label | git.openAllChanges, a Command Palette row that opens every change in one multi-file diff
MAP | Stage Block | ED-6.8 | label | git.diff.stageHunk, the plus button in the diff gutter and a palette row, for a diff against the index
MAP | Stage Selection | ED-6.8 | label | git.diff.stageSelection, the plus button on a selection in the diff gutter and a palette row, for a diff against the index
MAP | Commit (Signed Off) | SB-4.9 | label | git.commitSigned in the Commit submenu
MAP | Commit Staged (Signed Off) | SB-4.9 | label | git.commitStagedSigned in the Commit submenu
MAP | Commit All (Signed Off) | SB-4.9 | label | git.commitAllSigned in the Commit submenu
MAP | Commit (No Verify) | SB-4.9 | label | git.commitNoVerify in the Commit submenu, only when git.allowNoVerifyCommit is on
MAP | Commit Staged (No Verify) | SB-4.9 | label | git.commitStagedNoVerify in the Commit submenu, only when git.allowNoVerifyCommit is on
MAP | Commit Empty (No Verify) | SB-4.9 | label | git.commitEmptyNoVerify, a Command Palette row shown only when git.allowNoVerifyCommit is on
MAP | Commit Staged (Signed Off, No Verify) | SB-4.9 | label | git.commitStagedSignedNoVerify in the Commit submenu when no-verify commits are allowed
MAP | Commit (Amend, No Verify) | SB-4.10 | label | git.commitAmendNoVerify in the Commit submenu when no-verify commits are allowed
MAP | Commit (Signed Off, No Verify) | SB-4.9 | label | git.commitSignedNoVerify in the Commit submenu when no-verify commits are allowed
MAP | Commit Staged (Amend, No Verify) | SB-4.10 | label | git.commitStagedAmendNoVerify in the Commit submenu when no-verify commits are allowed
MAP | Commit All (No Verify) | SB-4.9 | label | git.commitAllNoVerify in the Commit submenu when no-verify commits are allowed
MAP | Commit All (Signed Off, No Verify) | SB-4.9 | label | git.commitAllSignedNoVerify in the Commit submenu when no-verify commits are allowed
MAP | Commit All (Amend, No Verify) | SB-4.10 | label | git.commitAllAmendNoVerify in the Commit submenu when no-verify commits are allowed
EXCL | Get Repositories | label | internal | git.api.getRepositories, category Git API, hidden from the palette and called by other extensions
EXCL | Get Repository State | label | internal | git.api.getRepositoryState, category Git API, hidden from the palette and called by other extensions
MAP | Open Staged Changes | SB-4.4 | label | git.viewStagedChanges, the inline button on the Staged Changes group and a palette row
MAP | Open Untracked Changes | SB-4.4 | label | git.viewUntrackedChanges, the inline button on the Untracked Changes group, shown only when git.untrackedChanges is separate
MAP | Copy Branch Name | SB-4.30 | label | row in the branch, stash and worktree menus of the experimental Repositories view; [stock only], missing in Cursor
MAP | Copy Worktree Path | SB-4.30 | label | git.repositories.worktreeCopyPath, row in the worktree menu of the Repositories view; [stock only]
MAP | Copy Tag Name | SB-4.30 | label | git.repositories.copyTagName, row in the tag menu of the Repositories view; [stock only]
MAP | Copy Stash Name | SB-4.30 | label | git.repositories.copyStashName, row in the stash menu of the Repositories view; [stock only]
MAP | Continue Working in New Local Clone | SB-4.27 | label | git.continueInLocalClone, row of the Continue Working On picker, shown only in a remote window; SB-4.27 already names the command
NEW | Color for added resources | NEW-J-1 | label | gitDecoration.addedResourceForeground, color of added files and their badge
NEW | Color for modified resources | NEW-J-1 | label | gitDecoration.modifiedResourceForeground, color of modified and type-changed files
NEW | Color for deleted resources | NEW-J-1 | label | gitDecoration.deletedResourceForeground, color of files deleted in the working tree
NEW | Color for renamed or copied resources | NEW-J-1 | label | gitDecoration.renamedResourceForeground, color of renamed and copied files
NEW | Color for untracked resources | NEW-J-1 | label | gitDecoration.untrackedResourceForeground, color of untracked files
NEW | Color for ignored resources | NEW-J-1 | label | gitDecoration.ignoredResourceForeground, color of files that .gitignore covers
NEW | Color for modified resources which have been staged | NEW-J-1 | label | gitDecoration.stageModifiedResourceForeground, color of files modified in the index
NEW | Color for deleted resources which have been staged | NEW-J-1 | label | gitDecoration.stageDeletedResourceForeground, color of files deleted in the index
NEW | Color for resources with conflicts | NEW-J-1 | label | gitDecoration.conflictingResourceForeground, color of every merge conflict state
NEW | Color for submodule resources | NEW-J-1 | label | gitDecoration.submoduleResourceForeground, color of submodule folders with the S badge
MAP | Color for the blame editor decoration | ED-6.27 | label | git.blame.editorDecorationForeground, tint of the faint blame text at the end of the cursor line; default is the inlay hint foreground
EXCL | Markdown Language Features | label | extension-system | display name of the Markdown extension, shown in the Extensions view
EXCL | Provides rich language support for Markdown | label | extension-system | description line of the Markdown extension, shown in the Extensions view
EXCL | Required for loading styles configured in the workspace | label | extension-system | untrusted workspace note of the Markdown extension, shown as the extension status message in the Extensions view and editor
MAP | Controls where links in Markdown files should be opened | ED-10.13 | label | markdown.links.openLocation; in the 1.128 source only the preview uses it, to open a linked file in the active group or beside
MAP | Open links in the active editor group | ED-10.13 | label | value currentGroup of markdown.links.openLocation, the default
MAP | Open links beside the active editor | ED-10.13 | label | value beside of markdown.links.openLocation
MAP | Controls whether path suggestions are shown while writing links in Markdown files | AC-1.46 | label | markdown.suggest.paths.enabled; AC-1.46 is a Markdown source feature that already exists, so it is kept
MAP | Disable workspace header suggestions | AC-1.46 | label | value never of markdown.suggest.paths.includeWorkspaceHeaderCompletions
EXCL | Always insert Markdown links | label | other | value of the markdown.editor.drop.enabled, filePaste.enabled and pasteUrlAsFormattedLink.enabled settings; Markdown source editing, out of scope
EXCL | Never create Markdown links | label | other | value of the markdown.editor.drop.enabled, filePaste.enabled and pasteUrlAsFormattedLink.enabled settings; Markdown source editing, out of scope
EXCL | Try to copy external image and video files into the workspace | label | other | value of markdown.editor.drop.copyIntoWorkspace and filePaste.copyIntoWorkspace; Markdown source editing, out of scope
EXCL | Do not copy external files into the workspace | label | other | value of markdown.editor.drop.copyIntoWorkspace and filePaste.copyIntoWorkspace; Markdown source editing, out of scope
EXCL | Prompt on each file move | label | other | value of markdown.updateLinksOnFileMove.enabled; update links on move was ruled out of scope
EXCL | Always update links automatically | label | other | value of markdown.updateLinksOnFileMove.enabled; update links on move was ruled out of scope
EXCL | Never try to update link and don't prompt | label | other | value of markdown.updateLinksOnFileMove.enabled; update links on move was ruled out of scope
EXCL | The glob pattern to match file paths against. Set to true to enable the pattern | label | other | markdown.updateLinksOnFileMove.include; update links on move was ruled out of scope
EXCL | Enable updating links when a directory is moved or renamed in the workspace | label | other | markdown.updateLinksOnFileMove.enableForDirectories; update links on move was ruled out of scope
EXCL | Controls whether link occurrences in the current document are highlighted | label | other | markdown.occurrencesHighlight.enabled; highlights links in the Markdown source editor, not the preview
EXCL | Controls if files created by drop or paste should overwrite existing files | label | other | markdown.copyFiles.overwriteBehavior; paste and drop of files was ruled out of scope
EXCL | If a file with the same name already exists, overwrite it | label | other | value overwrite of markdown.copyFiles.overwriteBehavior; paste and drop of files was ruled out of scope
EXCL | Controls whether error reporting is enabled in Markdown files | label | other | markdown.validate.enabled; link validation was ruled out of scope, though SS-3.53 still names this setting for its status row
EXCL | Validate link definitions that are unused in the current file | label | other | markdown.validate.unusedLinkDefinitions.enabled; link validation was ruled out of scope
EXCL | Validate duplicated definitions in the current file | label | other | markdown.validate.duplicateLinkDefinitions.enabled; link validation was ruled out of scope
MAP | Convert URL-like text to links in the Markdown preview | ED-10.14 | label | markdown.preview.linkify
MAP | Enable some language-neutral replacement and quotes beautification in the Markdown preview | ED-10.14 | label | markdown.preview.typographer
MAP | Controls the font family used in the Markdown preview | ED-10.14 | label | markdown.preview.fontFamily
MAP | Controls the font size in pixels used in the Markdown preview | ED-10.14 | label | markdown.preview.fontSize
MAP | When a Markdown editor is scrolled, update the view of the preview | ED-10.13 | label | markdown.preview.scrollPreviewWithEditor
MAP | Mark the current editor selection in the Markdown preview | ED-10.13 | label | markdown.preview.markEditorSelection
MAP | When a Markdown preview is scrolled, update the view of the editor | ED-10.13 | label | markdown.preview.scrollEditorWithPreview
MAP | Double-click in the Markdown preview to switch to the editor | ED-10.13 | label | markdown.preview.doubleClickToSwitchToEditor
MAP | Controls how links to other Markdown files in the Markdown preview should be opened | ED-10.13 | label | markdown.preview.openMarkdownLinks
MAP | Try to open links in the Markdown preview | ED-10.13 | label | value inPreview of markdown.preview.openMarkdownLinks, the default
MAP | Try to open links in the editor | ED-10.13 | label | value inEditor of markdown.preview.openMarkdownLinks
MAP | Do not render frontmatter | ED-10.17 | label | value hide of markdown.preview.frontMatter; [stock only], the setting is missing in Cursor
MAP | Render frontmatter as a code block | ED-10.17 | label | value codeBlock of markdown.preview.frontMatter; [stock only], the setting is missing in Cursor
MAP | Render frontmatter as a table of keys and values | ED-10.17 | label | value table of markdown.preview.frontMatter; [stock only], the setting is missing in Cursor
EXCL | Traces the communication between VS Code and the Markdown language server | label | developer-only | markdown.trace.server (off, messages, verbose), a debugging aid for the Markdown language server
EXCL | Controls the logging level of the Markdown language server | label | developer-only | markdown.server.log (off, debug, trace), a debugging aid for the Markdown language server
EXCL | Highlighting and commands for inline merge conflicts | label | extension-system | description line of the Merge Conflict extension, shown in the Extensions view
MAP | Accept Selection | ED-6.23 | label | merge-conflict.accept.selection, palette row that accepts the Current or Incoming side the cursor is in
MAP | Compare Current Conflict | ED-6.23 | label | merge-conflict.compare, palette row that opens the diff of the conflict at the cursor, the same as the Compare Changes CodeLens
EXCL | Reference Search View | label | extension-system | display name of the references-view extension; the view itself is SB-7.20
EXCL | Reference Search results as separate, stable view in the sidebar | label | extension-system | description line of the references-view extension, shown in the Extensions view
EXCL | Provides syntax highlighting and language features for tabbed search results | label | extension-system | description line of the search-result extension; the Search Editor itself is ED-3.29
EXCL | A very basic built-in webview for displaying web content | label | extension-system | description line of the simple-browser extension; its command is ED-10.25
MAP | Enable/disable the floating indicator that shows when focused in the simple browser | ED-10.26 | label | simpleBrowser.focusLockIndicator.enabled; [stock only], Cursor has no Simple Browser panel
