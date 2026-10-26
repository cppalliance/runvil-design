# Dialogs (DL) - VS Code 1.128.0 findings

Section: Dialogs (DL). Date: 2026-10-25. Pass: 1.

- Scope of this file: stock VS Code 1.128.0 only (root V). Cursor findings are in the sibling file under `cursor\`.
- Level: minor groups only. No feature lists, no pieces.
- Short names: V = `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0`. N = `C:\Users\Vinnie\src\vscode` (1.144.0, used only to spot newer things). All proof paths below start under `V\src\vs\` unless they say otherwise.
- Line numbers are from the files as read. They can drift by a line or two, so search the quoted id or label if one does not match.

## What this section is

Modal prompts. A modal blocks the window until you answer. In V there are two separate families, and a person only meets them as "a box that pops up":

- Message-style dialogs. Code asks `IDialogService` for a `confirm`, `prompt`, `input`, `info`, `warn`, `error` or `about` box. One handler class per runtime draws it. On desktop the contribution `DialogHandlerContribution` picks between the OS message box (`NativeDialogHandler`) and an in-page box (`BrowserDialogHandler`) using the setting `window.dialogStyle`. The input box has no OS version, so it is always drawn in-page.
- File dialogs. Code asks `IFileDialogService` to open or save. On desktop it either calls the OS file picker through the main process, or draws the app's own "simple file dialog" (a quick pick where you type a path). The save-changes box (`showSaveConfirm`) lives in the same service but is a message-style prompt.

Key files (all under `V\src\vs\`):

- `platform\dialogs\common\dialogs.ts` - the shapes of confirm, prompt and input (lines 58, 83, 132), custom options such as icon and markdown details (line 287), `IDialogHandler` (line 305), `IDialogService` (line 464), `IFileDialogService` (line 528), `ConfirmResult` (line 601), and `massageMessageBoxOptions` for per-OS button order (line 653)
- `workbench\browser\parts\dialogs\dialogHandler.ts` - the in-page box (class `BrowserDialogHandler`, line 21, with `about()` near line 76)
- `workbench\browser\parts\dialogs\dialog.ts` - shared look and key handling (`createWorkbenchDialogOptions`, line 29) and the web About text (line 48)
- `workbench\browser\parts\dialogs\dialog.web.contribution.ts` - web-only handler (not used on desktop)
- `workbench\electron-browser\parts\dialogs\dialog.contribution.ts` - desktop chooser, `processDialogs` (line 59) and `useCustomDialog` (line 106), registered to block startup
- `workbench\electron-browser\parts\dialogs\dialogHandler.ts` - OS box (`prompt` line 23, `confirm` line 43, `input` throws "Unsupported" line 63, `about` line 67)
- `platform\dialogs\electron-browser\dialog.ts` - About text (`createNativeAboutDialogDetails`, line 13)
- `platform\dialogs\electron-main\dialogMainService.ts` - the main-process side: open pickers (`pickFileFolder` line 59, `pickFolder` line 63, `pickFile` line 83, `pickWorkspace` line 87, `doPick` line 95), `showMessageBox` (line 149), `showSaveDialog` (line 167), `showOpenDialog` (line 209)
- `workbench\services\dialogs\browser\abstractFileDialogService.ts`, `...\electron-browser\fileDialogService.ts`, `...\browser\simpleFileDialog.ts`, `...\common\dialogService.ts`

## Proposed minor groups

Count: 6. Reason: the section splits along what the person is doing when the box appears, not along the code. The code has only two services, but a user sees six different situations: any generic box, protecting unsaved work, picking a file, confirming a file delete or move, deciding about trust, and (in Cursor only) plan and account modals. A seventh group for "About" was rejected because V treats About as the fourth kind of message-style dialog (`IDialogHandler.about`), so it sits with the generic boxes. Fewer than 6 would merge "save my changes" with "pick a file", which people think of separately. Confidence: high - the split follows the existing settings (`window.confirmBeforeClose`, `files.simpleDialog.enable`, `explorer.confirm*`, `security.workspace.trust.*`).

### DL-1 Message, confirm and input boxes (and About)

- Plain-words name: Message, confirm and input boxes
- One sentence: The generic pop-up boxes that tell you something, ask yes or no, offer several buttons, or ask for text, drawn either by the OS or by the app, plus the About box.
- Tag: both (Cursor bundle has the setting and the handler, see the Cursor file)
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Confirm box: primary button, Cancel, optional checkbox such as "Do not ask me again" (`IConfirmation`, `platform\dialogs\common\dialogs.ts` line 58)
  - Prompt box with two or more custom buttons and an optional Cancel (`IPrompt`, line 132). One-button info, warning and error boxes use the same path (`IDialogService.info`, `warn`, `error`)
  - Input box with text fields, for example "Proxy Authentication Required" with Username, Password and a "Remember my credentials" checkbox (`workbench\electron-browser\window.ts` lines 293-335). Always drawn in-page
  - About box, command `workbench.action.showAboutDialog` ("About"), with Copy and OK buttons, listing version, commit, date, Electron, Chromium, Node.js, V8 and OS (`workbench\browser\actions\windowActions.ts` lines 387-412)
- Also seen in this group, not counted as examples:
  - Setting `window.dialogStyle` (`native` or `custom`, default `native`) at `workbench\electron-browser\desktop.contribution.ts` line 291. `custom` forces the in-page box. Some prompts force the in-page box on their own (`custom` option with an icon and markdown text, used by the Workspace Trust prompts)
  - Per-OS button order and default button for the OS box: `massageMessageBoxOptions`, `platform\dialogs\common\dialogs.ts` line 653
  - Dialogs queue: one at a time per window (`windowDialogQueues` in `platform\dialogs\electron-main\dialogMainService.ts` line 51). The window dims while an in-page box is shown (`onVisibilityChange` in `workbench\browser\parts\dialogs\dialog.ts` line 43)
  - Key handling: most shortcuts are blocked while a box is up. Quit, reload and the clipboard commands are allowed (`defaultDialogAllowableCommands`, same file line 18)
- Where it is reachable from: Help menu "About" (hidden on macOS native menus, where `platform\menubar\electron-main\menubar.ts` line 409 puts "About {name}" in the app menu), Command Palette "Help: About". The other boxes have no direct entry. They come from the groups below.
- Proof paths:
  - `platform\dialogs\common\dialogs.ts` lines 58-165 (shapes), 287-300 (custom options), 305-325 (handler contract), 335-455 (button order and result mapping), 464-520 (service contract)
  - `workbench\browser\parts\dialogs\dialogHandler.ts` lines 21-136
  - `workbench\electron-browser\parts\dialogs\dialog.contribution.ts` lines 59-116
  - `workbench\electron-browser\parts\dialogs\dialogHandler.ts` lines 18-84
  - `platform\dialogs\electron-browser\dialog.ts` lines 13-45
  - `workbench\browser\actions\windowActions.ts` lines 387-412 and 468
  - `workbench\services\dialogs\common\dialogService.ts` (service and `DialogsModel`, `about()` at line 91)

### DL-2 Unsaved changes, close and quit prompts

- Plain-words name: Unsaved changes, close and quit prompts
- One sentence: The boxes that stop you from losing work when you close an editor, close a window, quit the app, or leave an untitled workspace.
- Tag: both (Cursor changes the quit and close boxes, see the Cursor file)
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Save changes box: "Do you want to save the changes you made to {0}?" with Save, Don't Save and Cancel. A "Save All" version appears for many files and lists up to 10 names (`showSaveConfirm`, `workbench\services\dialogs\browser\abstractFileDialogService.ts` lines 167-222)
  - Dirty editor on close, from the editor tab, Close All, or closing a group (`workbench\browser\parts\editor\editorGroupView.ts` around line 1802, `editorActions.ts` around line 643)
  - Confirm on quit or window close: "Are you sure you want to exit?" or "Are you sure you want to close the window?" with a "Do not ask me again" checkbox (setting `window.confirmBeforeClose`: `always`, `keyboardOnly`, `never`; `workbench\browser\window.ts` lines 182-210; File menu toggle `workbench.action.toggleConfirmBeforeClose` in `workbench\browser\actions\windowActions.ts` lines 495-510)
  - Untitled workspace: save or discard when switching workspace (setting `window.confirmSaveUntitledWorkspace`, `workbench\services\workspaces\electron-browser\workspaceEditingService.ts` lines 93-160)
- Also seen in this group, not counted as examples:
  - Hot exit off or failed: with `files.hotExit` set to `off`, quitting with dirty files asks to save. If backup fails, an error box lists the files (`workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts` lines 285-330)
  - The merge editor has its own save prompt with "Do you want to save the changes you made to {0} files?" (`workbench\contrib\mergeEditor\browser\mergeEditorInputModel.ts` line 449)
  - Quit command `workbench.action.quit` runs the same confirmation (`workbench\electron-browser\desktop.contribution.ts` lines 75-92)
- Where it is reachable from: closing any dirty tab, File > Close Editor, File > Exit or Quit, window close button, File menu toggle for the confirm setting.
- Proof paths:
  - `workbench\services\dialogs\browser\abstractFileDialogService.ts` lines 167-222
  - `workbench\browser\parts\editor\editorGroupView.ts` lines 1770-1830
  - `workbench\browser\parts\editor\editorActions.ts` lines 625-660
  - `workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts` lines 285-330
  - `workbench\electron-browser\window.ts` lines 470-520 (confirm on shutdown, desktop)
  - `workbench\browser\window.ts` lines 182-210 (the confirm box itself)
  - Settings: `workbench\browser\workbench.contribution.ts` line 933 (`window.confirmBeforeClose`), `workbench\electron-browser\desktop.contribution.ts` line 167 (`window.confirmSaveUntitledWorkspace`), `workbench\contrib\files\browser\files.contribution.ts` line 321 (`files.hotExit`)

### DL-3 File open and save pickers

- Plain-words name: File open and save pickers
- One sentence: The windows where you browse to a file or folder to open, add or save, drawn either by the OS or by the app's own type-a-path list.
- Tag: both (Cursor bundle has the service, the setting and the simple dialog, see the Cursor file)
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Open File, Open Folder and Open (file or folder) pickers (`workbench.action.files.openFile`, `...openFolder`, `...openFileFolder`; ids declared at `workbench\browser\actions\workspaceActions.ts` lines 31, 56, 117). Open Workspace from File (`workbench.action.openWorkspace`, line 143) and Add Folder to Workspace (`workbench.action.addRootFolder`, line 215) use the same service
  - Save As picker (`workbench.action.files.saveAs`, "Save As...", `workbench\contrib\files\browser\fileConstants.ts` lines 21-22; dialog options built at `abstractFileDialogService.ts` line 394). Also Save Workspace As (`workbench.action.saveWorkspaceAs`, `workspaceActions.ts` line 261)
  - Simple file dialog: a quick pick with a path box where you type a folder path, with a "Folder path" placeholder, an OK button, and an eye button to show or hide dot files (`workbench\services\dialogs\browser\simpleFileDialog.ts` lines 357-370 and 530-537). Turned on by the setting `files.simpleDialog.enable` (default `false`, `files.contribution.ts` line 375) and used automatically for non-file remotes
  - "Show Local" button in the simple dialog on a remote window, with the matching commands "Open Local File...", "Open Local Folder...", "Open Local..." and "Save Local File..." (ids `workbench.action.files.openLocalFile`, `...openLocalFolder`, `...openLocalFileFolder`, `...saveLocalFile`, `simpleFileDialog.ts` lines 44-90; keybindings in `workbench\contrib\remote\electron-browser\remote.contribution.ts` lines 218-258, active only while the dialog context key `remoteFileDialogVisible` is true)
- Also seen in this group, not counted as examples:
  - The simple dialog asks follow-up questions with its own in-list prompts: "{0} already exists. Are you sure you want to overwrite it?", "The folder {0} does not exist. Would you like to create it?" (`simpleFileDialog.ts` lines 866-960)
  - OS pickers are made in the main process (`platform\dialogs\electron-main\dialogMainService.ts` lines 59-130). On Windows the Open Folder button gets a mnemonic label "Select folder" (lines 63-81). On macOS packages are opened as folders (line 120)
  - Rule that picks which kind to show: `shouldUseSimplified` in `workbench\services\dialogs\electron-browser\fileDialogService.ts` line 66. The OS picker only handles `file` and `vscode-userdata` paths
  - Extension API entry points `showOpenDialog` and `showSaveDialog` (`IFileDialogService`, `platform\dialogs\common\dialogs.ts` lines 588-598)
- Where it is reachable from: File menu (Open File, Open Folder, Open Workspace from File, Save As, Add Folder to Workspace, Save Workspace As), the welcome page and the empty editor watermark (`workbench\browser\parts\editor\editorGroupWatermark.ts` lines 37-39), Command Palette.
- Proof paths:
  - `workbench\services\dialogs\electron-browser\fileDialogService.ts` lines 60-160
  - `workbench\services\dialogs\browser\abstractFileDialogService.ts` lines 228-335 and 390-450
  - `workbench\services\dialogs\browser\simpleFileDialog.ts` lines 44-110, 340-470, 525-540, 860-980
  - `workbench\browser\actions\workspaceActions.ts` lines 31-270
  - `workbench\contrib\remote\electron-browser\remote.contribution.ts` lines 218-258
  - `platform\dialogs\electron-main\dialogMainService.ts` lines 46-240
  - Setting: `workbench\contrib\files\browser\files.contribution.ts` lines 374-378

### DL-4 Delete, move and replace confirmations

- Plain-words name: Delete, move and replace confirmations
- One sentence: The Explorer boxes that ask before you delete, move, overwrite, paste or drop files and folders.
- Tag: both (every message and setting was found in the Cursor bundle, see the Cursor file)
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Delete confirmation with "Move to Recycle Bin" (Windows) or "Move to Trash", or "Delete" when there is no bin, and a "Do not ask me again" checkbox (setting `explorer.confirmDelete`, default `true`, `files.contribution.ts` line 494; box built at `workbench\contrib\files\browser\fileActions.ts` lines 78-200). Folders add "This action is irreversible!"
  - Drag and drop move: "Are you sure you want to move '{0}' into '{1}'?" (setting `explorer.confirmDragAndDrop`, default `true`, `files.contribution.ts` line 484; `explorerViewer.ts` lines 1572 and 1859-1865). Reordering root folders has its own wording
  - Replace-existing: "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" with a Replace button (`fileActions.ts` lines 488-491; `fileImportExport.ts` lines 167 and 532; multi-file version in `explorerViewer.ts` around line 2015). I found no setting that turns this one off
  - Paste or drop from outside: "Do you want to copy the folders or add the folders to the workspace?" and the paste-native-files confirm with a "Paste" button (setting `explorer.confirmPasteNative`, default `true`, `files.contribution.ts` line 489; `fileActions.ts` lines 1136-1173; `fileImportExport.ts` lines 470-490)
- Also seen in this group, not counted as examples:
  - Extra warnings before delete: "You are deleting {0} with unsaved changes" and "that is configured to be read-only" (`fileActions.ts` lines 106-150), and a retry box "Delete Permanently" when the bin fails (lines 226-240)
  - There is no setting named `files.confirmDelete` or `files.confirmDragAndDrop` in V. The real names are `explorer.confirmDelete` and `explorer.confirmDragAndDrop`
- Where it is reachable from: Explorer right-click Delete, Delete key, dragging inside the Explorer, dropping from the OS, Paste in the Explorer.
- Proof paths:
  - `workbench\contrib\files\browser\fileActions.ts` lines 78-311, 488-491, 1136-1173
  - `workbench\contrib\files\browser\fileImportExport.ts` lines 160-170, 470-540
  - `workbench\contrib\files\browser\views\explorerViewer.ts` lines 1572, 1830-1870, 2000-2020
  - Settings: `workbench\contrib\files\browser\files.contribution.ts` lines 484-496

### DL-5 Workspace Trust and link safety prompts

- Plain-words name: Workspace Trust and link safety prompts
- One sentence: The boxes that ask whether you trust a folder, its files, or an outside link before something risky runs, plus the entry that opens the trust editor.
- Tag: both (Cursor bundle has the prompts and commands but turns the feature off by default, see the Cursor file)
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Startup trust prompt: "Do you trust the authors of the files in this folder?" (or "workspace") with "Yes, I trust the authors" and "No, I don't trust the authors", a shield icon, and an optional checkbox "Trust the authors of all files in the parent folder". Controlled by `security.workspace.trust.startupPrompt` (`always`, `once`, `never`; default `never` in V) (`workbench\contrib\workspace\browser\workspace.contribution.ts` lines 444-520, setting at lines 806-818)
  - Open untrusted files: "Do you want to allow untrusted files in this workspace?" with Open and "Open in Restricted Mode", plus "Remember my decision for all workspaces" (same file lines 125-165, setting `security.workspace.trust.untrustedFiles`)
  - Trust request when a feature needs trust: "Trust Workspace & Continue" or "Trust Folder & Continue", "Manage" and Cancel (same file lines 203-278). Also "You are adding files that are not currently trusted to a trusted workspace" when you add a folder (lines 340-375)
  - Manage Workspace Trust, command `workbench.trust.manage` (opens the Workspace Trust editor pinned) and Configure Workspace Trust Settings, command `workbench.trust.configure` (opens settings filtered to trust). Both are in the Command Palette under the category "Workspaces" (lines 748-790)
- Also seen in this group, not counted as examples:
  - External link prompt: "Do you want {name} to open the external website?" with Open, Copy and "Configure Trusted Domains" (`workbench\contrib\url\browser\trustedDomainsValidator.ts` lines 82-130). Command `workbench.action.manageTrustedDomain` ("Manage Trusted Domains", `trustedDomains.ts` line 62, listed in the Command Palette at `url.contribution.ts` lines 63-67). Setting `workbench.trustedDomains.promptInTrustedWorkspace` (default `false`, `url.contribution.ts` line 91)
  - The editor body behind "Manage Workspace Trust" (Restricted Mode header, feature list, "Trusted Folders & Workspaces" table, Add Folder) is an editor tab, `WorkspaceTrustEditor` (`workbench.editor.workspaceTrust`, `workspaceTrustEditor.ts` line 669). Only the entry is counted here. The editor itself probably belongs in the Editor section. Boundary to settle with that pass.
  - The Restricted Mode banner and the status bar "Restricted Mode" entry are declared in the same file (`BANNER_RESTRICTED_MODE` at line 53, entry id `status.workspaceTrust` at line 289). They belong to the Banner and Status Bar sections, not here
  - A "Manage Workspace Trust" button in the editor placeholder for untrusted files (`workbench\browser\parts\editor\editorPlaceholder.ts` line 215)
  - Other settings: `security.workspace.trust.enabled` (default `true`), `security.workspace.trust.banner`, `security.workspace.trust.emptyWindow` (lines 799-850)
- Where it is reachable from: opening a folder in an untrusted state, opening a file from an untrusted location, adding a folder to a workspace, clicking a web link in an editor or terminal, Command Palette.
- Proof paths:
  - `workbench\contrib\workspace\browser\workspace.contribution.ts` lines 53-70, 101-281, 287-520, 748-855
  - `workbench\contrib\workspace\common\workspace.ts` lines 13-18 (context keys and command id)
  - `workbench\services\workspaces\common\workspaceTrust.ts` lines 30-34 (setting ids)
  - `workbench\contrib\workspace\browser\workspaceTrustEditor.ts` line 669 (editor id only)
  - `workbench\contrib\url\browser\trustedDomainsValidator.ts` lines 25-135, `trustedDomains.ts` lines 62-100, `url.contribution.ts` lines 60-100

### DL-6 Cursor plan and account modals

- Plain-words name: Cursor plan and account modals
- One sentence: Cursor's own pop-ups for choosing a plan, seeing usage pricing, confirming an upgrade, inviting teammates, and switching privacy mode.
- Tag: cursor (nothing like it in V 1.128.0)
- Evidence: bundle (nothing to read in V, so no V source). Full detail is in the Cursor file
- Boundary: yes. These modals open from Cursor Settings and, for the usage pricing box, from the chat queue wait indicator. Cursor Settings is a named boundary item in the brief.
- Examples (up to 4): Subscription tiers modal (`cursor.showSubscriptionTiersModal`); Usage pricing modal (`cursorai.action.showUsagePricingModal`); Upgrade confirmation modal; Invite Team Members modal
- Proof paths: none in V. See the Cursor file.

## Left out (and why)

- Extension and marketplace prompts, such as install, uninstall, "Do you trust the publisher", extension URL handler "Allow extension to open URI", and "Configure Extension Host" (extension system is out of scope). Source folders `workbench\contrib\extensions`, `workbench\services\extensions\browser\extensionUrlHandler.ts`
- Authentication prompts raised by extensions or MCP servers: "Allow extension X to sign in using Y" (`workbench\api\browser\mainThreadAuthentication.ts`, `workbench\contrib\authentication`) - extension system and MCP
- Chat, agent and MCP approvals: tool confirmation (`workbench\contrib\chat\browser\tools\languageModelToolsConfirmationService.ts`), terminal command approval, chat plan review, chat permission warnings, MCP sampling and trust (`workbench\contrib\mcp`), agent plugin install, agent sessions actions, chat setup. Over 30 files under `workbench\contrib\chat` call the dialog service
- Welcome onboarding overlay "Welcome Onboarding 2026" (`workbench\contrib\welcomeOnboarding`, command `workbench.action.welcomeOnboarding2026`). It is a centered modal with a sign-in step for GitHub Copilot, a personalize step and an agent sessions step, but it only auto-shows to brand-new users when the experimental setting `workbench.welcomePage.experimentalOnboarding` is on (`workbench\contrib\welcomeGettingStarted\browser\startupPage.ts` lines 236-262). Two of its three steps are chat and sign-in, so I count it as chat-flavored and left it out. See unverified for the one place it could still be wanted
- Onboarding spotlight overlay (`workbench\contrib\onboarding\browser\spotlight`) - a highlight-a-control tip, not a dialog
- Prompts that belong to another section's own feature, even though they call the dialog service. They are listed here so the later passes do not lose them: terminal "kill terminal" and exit confirms (`terminal.integrated.confirmOnExit`, `confirmOnKill`), debug stop and restart prompts, Git (extension) discard and force-push confirms, local history delete/restore prompts (`workbench\contrib\localHistory\browser\localHistoryCommands.ts`), settings profile prompts (`workbench\contrib\userDataProfile`), Settings Sync prompts (`workbench\contrib\userDataSync`), edit sessions, issue reporter and troubleshoot prompts (`workbench\contrib\issue`), shell command install prompts (`workbench\electron-browser\actions\installActions.ts`), remote tunnel prompts, theme and notebook prompts
- Web-only dialog handler and web About text (`workbench\browser\parts\dialogs\dialog.web.contribution.ts`, `dialog.ts` line 48) - V desktop only matters for Cursor
- Test-only code: `platform\dialogs\test`, `workbench\services\dialogs\test`

## Newer than 1.128 (seen only in N)

All marked `newer-than-1.128`. None of it was proven in Cursor.

- Dialog `detail` can be Markdown (`readonly detail?: string | IMarkdownString`) and the OS box turns it into plain text. N: `src\vs\platform\dialogs\common\dialogs.ts`, `src\vs\workbench\browser\parts\dialogs\dialogHandler.ts`
- Dialog option `alignment: 'horizontal' | 'vertical'` for a stacked layout with full-width buttons. Same two files
- About text adds two lines, `@github/copilot` and `@github/copilot-sdk` versions. N: `src\vs\platform\dialogs\electron-browser\dialog.ts`
- `massageMessageBoxOptions` moved out of `dialogs.ts` into `src\vs\platform\dialogs\electron-main\dialogMainUtils.ts` (refactor, no UI change)
- Open dialog now starts in the default folder path when it only allows folders (N: `src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts`)
- Simple file dialog can offer to create a folder when any ancestor is writable (not just the parent), and keeps the typed path when it matches the current folder (N: `src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts`, new test `...\test\browser\simpleFileDialog.test.ts`)
- A larger onboarding tryout feature (guided tryouts, a tryout window, URL handler) under `src\vs\workbench\contrib\onboarding\browser\` and `...\electron-browser\onboardingTryoutWindow.ts`. I did not read it. It may or may not be a modal
- Workspace Trust editor: delete-row focus handling changed (`workspaceTrustEditor.ts`), no new dialog

## Unverified

- Which callers set the Cursor-only `showPrettyDialogOnError` flag. In V there is nothing like it, so this file cannot say (see the Cursor file)
- The exact set of Command Palette entries and menu entries that reach each prompt. I traced commands and settings, not every menu id
- I did not run V, so the look of each box (icon, width, button order on each OS) is read from code only
- Whether the "Welcome Onboarding 2026" modal should count as a dialog for this section. I left it out as chat-flavored, but the Personalize step (theme picker) is not chat
- The default for `security.workspace.trust.startupPrompt` is `never` in V 1.128.0 per `workspace.contribution.ts` lines 806-812. I did not check whether the shipped product.json overrides it
- `workspaceTrust.ts` enablement interface moved to `src\vs\platform\workspace\common\workspaceTrust.ts` in N. I did not diff the dialogs that use it

## How I checked

- Read every file in `workbench\browser\parts\dialogs`, `workbench\electron-browser\parts\dialogs`, `platform\dialogs\common`, `platform\dialogs\electron-browser`, `platform\dialogs\electron-main` and `workbench\services\dialogs` (browser, electron-browser, common).
- Read `workbench\contrib\workspace` (contribution, common, editor entry) and the trusted-domain prompt under `workbench\contrib\url`.
- Listed every setting in `workbench` and `platform` whose id contains `dialog`, `confirm` or `trust`, and read the ones that trigger a box.
- Searched for every caller of `showSaveConfirm` and counted callers of `dialogService.confirm|prompt|input|info|warn|error` to decide what to leave out.
- Checked `welcomeOnboarding`, `welcomeBanner`, `welcomeAgentSessions` and `onboarding` folders for modal behavior. No `welcomeDialog` folder exists in V 1.128.0.
- Built-in extensions: none read in depth. A modal box raised by any extension (`window.showWarningMessage` with `modal: true`) lands in `workbench\api\browser\mainThreadMessageService.ts` line 141, which calls the dialog service, so it is drawn by this section's handlers in the same style. The git extension is the main built-in user (5 files under `extensions\git\src` contain `modal: true`). Those confirm boxes belong to Source Control, not here.
- Compared the dialog folders against N file by file for the newer-than list.

