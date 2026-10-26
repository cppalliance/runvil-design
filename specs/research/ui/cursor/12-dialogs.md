# Dialogs (DL) - Cursor 3.27.12 findings

Section: Dialogs (DL). Date: 2026-10-25. Pass: 1.

- Scope of this file: what the Cursor install does for the section, and how it differs from stock VS Code 1.128.0. The group list itself lives in the sibling file under `vscode\`. Group ids here match that file.
- Short names: C = `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`. Main bundle = `C\out\vs\workbench\workbench.desktop.main.js`. Electron main = `C\out\main.js`. Message table = `C\out\nls.messages.json` (the bundle calls `k(<number>,null)`, and the number is the index into that array, which is how I read the labels). V = stock 1.128.0 source.
- Evidence word used below: `bundle` means I saw it only as strings or minified code in the Cursor bundle. `source` means I read it in V.
- Cursor code is minified, so names such as `Toi`, `JVa` or `v5w` are the bundle's own short names and can change between builds. The command ids, setting names, label text and message numbers are the stable proof.

## How I checked

- Counted bundle hits for every V command id and setting id this section uses (dialog style, simple dialog, confirm-before-close, explorer confirms, trust, trusted domains, open and save commands). All but `workbench.action.welcomeOnboarding2026` were found.
- Resolved message numbers through the message table for the V dialog texts (save changes, trust, simple dialog, trusted domains, delete and replace, proxy login). All of them exist in Cursor, with the same wording as V.
- Read the Cursor dialog handler contribution, the native handler class (`JVa`) and the About window component (`v5w`, `g5w`, `f5w`, opener `_5w`) in the bundle.
- Searched the bundle for Cursor's own dialog engine (`openDialog`, 59 call sites), listed every literal title it uses, and read the ones that are not about agents.
- Searched the CSS (`workbench.desktop.main.css`) for `dialog` class names.
- Listed the modal opener methods on Cursor's UI overlay service (`show...Modal`, `show...Dialog`).
- Searched Electron main (`out\main.js`) for the native dialog code, the About window methods and the Open Folder label.
- Searched the automations bundle (`workbench.anysphere-ui-automations.js`) for `openDialog` and the dialog service. None.
- Checked the four built-in extensions the brief names (`cursor-explorer`, `cursor-commits`, `cursor-checkout`, `cursor-worktree-textmate`) for `showWarningMessage`, `showInformationMessage`, `showErrorMessage` and `modal: true`. None.
- Did not read `workbench.glass.main.js` (Agents window, out of scope). The Cursor dialog handler contribution is registered with `disableInGlass: true`, so it does not run in that window.

## Summary of how Cursor differs

- All five V groups (DL-1 to DL-5) exist in Cursor. Every V dialog text and setting I looked up is present.
- Cursor has a second, separate dialog engine of its own (called the pretty dialog service in the code). It is an in-page modal with its own look and a persistent "Don't ask again" memory. `window.dialogStyle` does not control it.
- Quit and Close Window confirmations are moved onto that engine. They ask by default, even though `window.confirmBeforeClose` is `never` by default.
- About is a custom small window (an auxiliary window with a translucent background and its own drag area) with a built-in update button, not a message box. A native message box is kept only as a fallback.
- Workspace Trust is off by default (`security.workspace.trust.enabled` default `false`). V has it on. So the trust prompts exist but do not appear unless trust is turned on, by the user or by a team admin.
- Cursor adds a sixth group of modals for plan, pricing, upgrade, invite and privacy mode (DL-6).
- A few V pieces are missing: the Welcome Onboarding 2026 overlay and the Windows "Select folder" button label.

## Groups, with Cursor evidence

### DL-1 Message, confirm and input boxes (and About)

- Tag: both (the V engine is present and Cursor adds its own engine and its own About)
- Evidence: bundle (V source read as well)
- Boundary: no
- Found in bundle, V engine kept:
  - Dialog handler contribution `workbench.contrib.dialogHandler` (bundle name `Toi`), registered at the block-startup phase with `disableInGlass: true`. It sends confirm and prompt boxes to the OS or the in-page handler, and input boxes always to the in-page handler, exactly as in V
  - Setting `window.dialogStyle`: enum `native` and `custom`, default `native`, application scope (message 12063). Description text is "Adjust the appearance of dialog windows." V says "Adjust the appearance of dialogs to be native by the OS or custom." The only code difference I saw: Cursor's `useCustomDialog` reads only this setting. V also forces the in-page box while the smoke test driver runs
  - Native message box wrapper (class `JVa`) sends `type`, `title`, `message`, `detail`, `buttons`, `cancelId`, checkbox, `targetWindowId` as in V, plus a Cursor-only field `showOverHiddenWindowInWindow`. Electron main has two hits for that name (`out\main.js`), so the option is wired through to the OS box
  - "Proxy Authentication Required" input box present (message 12114)
  - "Do not ask me again" checkbox texts present (messages 2818, 4073, 7100, 7142, 7369, 7847, 8069, 10755)
- Cursor-only pieces (details in Cursor additions below):
  - The pretty dialog engine: `openDialog` with 59 call sites. Options seen: `title`, `message` (plain text, a list of text blocks, callout blocks with icon and colour, copyable-value blocks), `primaryButton` (with `type` such as `danger`), `extraButtons` (with `type: secondary`), `cancelButton`, `dialogIcon`, `dialogIconColor`, `width`, `dialogClass`, `dialogParentClass`, `dialogKey`, `shouldShowDontAskAgain`, `saveDontAskAgainOnCancel`, `checkboxAtBottom`, `noEnterToSubmit`. Answer is the clicked button id. CSS classes `.pretty-dialog-*` (about 40 hits in `workbench.desktop.main.css`), for example `.pretty-dialog-modal`, `.pretty-dialog-title`, `.pretty-dialog-message-callout`, `.pretty-dialog-copyable-value`, `.pretty-dialog-input`, `.pretty-dialog-dont-ask-again`, `.pretty-dialog-button-primary`
  - Custom About window (see below)
- About (found in bundle):
  - Command `workbench.action.showAboutDialog`, title "About" (message 3265), mnemonic "&&About" (3257), Help category, Help menu group `z_about` order 1, hidden on macOS native menu. Same as V. Menubar code inserts the update action next to it (`insertActionsBefore` case `workbench.action.showAboutDialog`)
  - `about()` is not the V one. It builds the text itself and opens a small window with the helper `_5w`: Electron main has `revealAboutWindow`, `fitAboutWindow` and `updateAboutWindowControls` (`out\main.js`, 3, 6 and 3 hits). The window is an auxiliary window opened with `glassVibrancy: true`, a transparent page background, an empty title, no full screen, and a drag region (`data-component="about-dialog-drag-region"`, `glass-about-dialog-window`, `glass-about-dialog-body`)
  - What the window shows: product name, "Version {0}", build date, with per-field copy buttons "Copy version" and "Copy build date" (messages 3335-3337, "Copied" 3334), a primary "Copy version info" button (3338, shortcut shown as Enter), a secondary update button (`glass-about-dialog-update-button`) and the copyright line "Copyright © 2026 Anysphere. All rights reserved." (12102). Escape closes the window. Enter copies the version info and closes
  - Update button runs `update.checkForUpdate`, `update.downloadUpdate`, `update.installUpdate` or `update.restartToUpdate` (with the source `about_dialog`) depending on update state. It shows a progress ring while working. It can show "up to date" text (`showUpToDate`)
  - Copied text (message 12101, with 12100 for the alpha build): `Version`, `VS Code Extension API` (the base VS Code number), `Commit`, `Date`, `Layout`, `Build Type` (Stable, Nightly, Dev/Debug), `Release Track`, `Electron`, `Chromium`, `Node.js`, `V8`, `xterm.js`, `OS`. An extra line "Local Mode: enabled" (12097) is appended when local mode is on. V lists Version, Commit, Date, Electron, ElectronBuildId, Chromium, Node.js, V8 and OS
  - Fallback: if the window fails to open, Cursor logs "[About] Failed to open supplemental About window" and shows a native message box with "Copy" and "OK" (messages 12098 and 12099), like V
  - Web build text (message 3339) is V's web About with a `Layout` line. Not relevant on desktop
- Proof: main bundle, search `window.dialogStyle`, `workbench.action.showAboutDialog`, `revealAboutWindow`, `glass-about-dialog-update-button`, `openDialog(`, `showOverHiddenWindowInWindow`, `pretty-dialog`. Messages 3257, 3265, 3334-3338, 12063, 12097-12102, 12114. Electron main `out\main.js` for the three about-window methods.

### DL-2 Unsaved changes, close and quit prompts

- Tag: both (V prompts exist, Cursor moves quit and close confirmations to its own engine)
- Evidence: bundle (V source read as well)
- Boundary: no (the quit flow also checks for running agents. Those agent boxes are left out)
- Found in bundle, same as V:
  - Save changes box texts: "Do you want to save the changes you made to {0}?" (12453), "...to the following {0} files?" (12454), "Your changes will be lost if you don't save them." (12452), "&&Save All" (12455), "Do&&n't Save" (12457)
  - `window.confirmBeforeClose`: enum `always`, `keyboardOnly`, `never`, default `keyboardOnly` in a web browser (not a standalone app) and `never` on desktop (same rule as V, `workbench.contribution.ts` line 947). Appears 11 times, including the File menu toggle `workbench.action.toggleConfirmBeforeClose` (group `z_ConfirmClose`, order 1; messages 4067 "Are you sure you want to quit?", 4068 "Are you sure you want to exit?", 4073 "Do not ask me again")
  - `window.confirmSaveUntitledWorkspace` (3 hits)
  - Backup-failed box texts (13589 and 13591, same as V's `workingCopyBackupTracker`)
  - The merge editor prompt with its own wording (7832-7837, 7838-7840), same as V
- Cursor changes:
  - Close Window (`workbench.action.closeWindow`, label "Close Window", mnemonic "Clos&&e Window" 12018, File menu group `6_close` order 4, keys Cmd+Shift+W on macOS, Alt+F4 on Windows and Linux, plus Ctrl+Shift+W): if the window has no folder or workspace, it closes at once. Otherwise it opens the pretty dialog titled "Close this window?" (12012) with message "Are you sure you want to close this window?" (12014), buttons "Close" (12016) and "Cancel" (12017). Dialog key `closeWindowConfirmation`
  - Quit (`workbench.action.quit`, Cmd+Q on macOS and Linux): pretty dialog "Quit Cursor?" (12011), "Are you sure you want to quit?" (12013), buttons "Quit" (12015) and "Cancel". Dialog key `quitConfirmation`. The box is skipped when a helper (`HOw`) reports running agents or shell tasks, because the shutdown flow then asks its own agent questions. V's own `window.confirmBeforeClose` confirm still runs after it
  - Both boxes show a "don't ask again" checkbox at the bottom (`shouldShowDontAskAgain`, `checkboxAtBottom`). The choice is saved in application storage under `dialogDontAskAgainPreferences`. A saved "yes" returns the primary button without showing the box
  - Net effect read from code: with `window.confirmBeforeClose` at its default `never`, Cursor still asks "Quit Cursor?" and "Close this window?" until you tick the box. With the V setting on `always`, you can see two boxes in a row (not run, see unverified)
  - Save Error box: when a save fails and the save options include `showPrettyDialogOnError`, Cursor shows a pretty dialog titled "Save Error" with the error text. The first save-error action becomes the primary button, the rest become secondary buttons, and there is a Cancel. These are the same actions V puts on the error notification. Without that option Cursor shows the V notification. Who sets the option is unknown
- Left out inside this group: boxes that warn about streaming agents or background agent tasks when you close, quit or reload ("keep streaming", "keep background tasks running"). These are agent-only
- Proof: main bundle, search `workbench.action.closeWindow`, `workbench.action.quit`, `quitConfirmation`, `closeWindowConfirmation`, `window.confirmBeforeClose`, `showPrettyDialogOnError`, `Save Error`. Messages 12011-12018, 12452-12457, 4067-4073, 13589, 13591.

### DL-3 File open and save pickers

- Tag: both
- Evidence: bundle (V source read as well)
- Boundary: no
- Found in bundle, same as V:
  - File dialog service with the same open and save methods (`pickFileFolderAndOpen` 7 hits, `pickFolderAndOpen` 11, `pickWorkspaceAndOpen` 7) and the same `shouldUseSimplified` rule. The only code difference I saw is that Cursor's rule has no smoke-test-driver clause
  - Setting `files.simpleDialog.enable` (boolean, default `false`, message 7268, same text as V)
  - Commands present: `workbench.action.files.openFile` (2 hits), `...openFolder` (6), `...openFileFolder` (3), `...saveAs` (1), `workbench.action.openWorkspace` (1), `workbench.action.addRootFolder` (3), `workbench.action.saveWorkspaceAs` (1), `workbench.action.files.openLocalFolder` (1), `...saveLocalFile` (1), `...openLocalFile` (2). Context key `remoteFileDialogVisible` (1 hit)
  - Texts present: "Open File or Folder" (12458), "Open Workspace from File" (12461 and 1903), "Save As" (12463, 12464), "Open Local File..." (12467), "Save Local File..." (12468), "Open Local Folder..." (12469), "Open Local..." (12470), "Show Local" (12472), "Hide dot files" (12473), "Please enter a valid path." (12477), "{0} already exists. Are you sure you want to overwrite it?" (12479), "The folder {0} does not exist. Would you like to create it?" (12481)
  - Electron main has the open and save dialog code: `showOpenDialog` (6 hits), `showSaveDialog` (5), `showMessageBox` (23), and `doPick` with `treatPackageAsDirectory` for macOS (`out\main.js`)
- Differences from V:
  - V's Windows-only "Select folder" button label on the Open Folder dialog is not in Cursor. Cursor's `pickFolder` only sets the title ("Open Folder", message 1901). Zero hits for the label in `out\main.js` and in the message table. Cursor's main process looks to be slightly behind V 1.128.0 for this file. Tag for that piece: vscode
- Proof: main bundle search `files.simpleDialog.enable`, `remoteFileDialogVisible`, `workbench.action.files.openLocalFile`, `pickFolderAndOpen`. `out\main.js` search `doPick`, `treatPackageAsDirectory`, `pickFolder(`. Messages 1901-1904, 7268, 12458-12481.

### DL-4 Delete, move and replace confirmations

- Tag: both
- Evidence: bundle (V source read as well)
- Boundary: no
- Found in bundle, same as V:
  - `explorer.confirmDelete` (2 hits), `explorer.confirmDragAndDrop` (2), `explorer.confirmPasteNative` (3)
  - Texts: "Are you sure you want to delete '{0}'?" (7113), "&&Move to Recycle Bin" (7080), "This action is irreversible!" (7093 and others), "Are you sure you want to move '{0}' into '{1}'?" (7368), "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" (7119 and 7214), "Do not ask me again" (7100, 7142, 7369)
- Differences from V: none found. I did not search for Cursor changes to the drop-folders choice ("copy or add to workspace") beyond the message table
- Proof: main bundle search `explorer.confirmDelete`, `explorer.confirmDragAndDrop`, `explorer.confirmPasteNative`. Messages 7080, 7093, 7100, 7113, 7119, 7142, 7214, 7368, 7369.

### DL-5 Workspace Trust and link safety prompts

- Tag: both (all prompts exist, but the feature is off by default)
- Evidence: bundle (V source read as well)
- Boundary: no
- Found in bundle, same as V:
  - All four trust prompts and their texts. Open untrusted files: 11836-11844 ("Do you want to allow untrusted files in this workspace?", "&&Open", "Open in &&Restricted Mode", "Remember my decision for all workspaces"). Trust request: 11845-11857 ("Do you trust the authors of the files in this folder?", "&&Trust Folder & Continue", "&&Manage", "Cancel"). Add folder to a trusted workspace: 11854. Startup prompt: 11858-11866 ("&&Yes, I trust the authors", "&&No, I don't trust the authors", "Trust the authors of all files in the parent folder '{0}'", "{0} provides features that may automatically execute files in this folder.")
  - Ids found: `workbench.trust.manage` (7 hits), `workbench.trust.configure` (2), the editor id `workbench.editor.workspaceTrust` (1), `workbench.banner.restrictedMode` (2), `status.workspaceTrust` (1), `workspace.trust.startupPrompt.shown` (1). The five trust setting ids are declared together in one constant list: `security.workspace.trust.enabled`, `.startupPrompt`, `.banner`, `.untrustedFiles`, `.emptyWindow`
  - External link prompt: "Do you want {0} to open the external website?" (11521). `workbench.action.manageTrustedDomain` (1 hit) and `workbench.trustedDomains.promptInTrustedWorkspace` (2 hits). The bundle mentions `trustedDomains` 14 times
  - The label text "Manage Workspace Trust" appears in the message table at 2798, 3751, 8663, 8664, 8699, 9296 and 11934. I did not map each one to its place in the UI
- Differences from V (read in the bundle):
  - `security.workspace.trust.enabled` default is `false` in Cursor (V: `true`). Message 11819 is V's text "Controls whether or not Workspace Trust is enabled within VS Code." and still says VS Code. Cursor also gives the setting an enterprise policy name `WorkspaceTrustEnabled` (min version 1.99). I did not find that policy in V source
  - `security.workspace.trust.startupPrompt` default is `once` in Cursor (V: `never`)
  - A team admin can switch trust on: the team admin settings include `workspaceTrustControls.enabled`. When it is true and no policy value is set, Cursor writes `security.workspace.trust.enabled` = true at user level (code that reads `a?.workspaceTrustControls?.enabled`)
  - V's note "Trusting this folder will also mark it as trusted in {0}." for the VS Code Sessions window has no hits (`getSessionsWindowTrustNote`, zero). That window does not exist in Cursor
- Proof: main bundle search `security.workspace.trust.startupPrompt`, `workbench.trust.manage`, `workspaceTrustControls`, `onDidInitiateOpenFilesTrustRequest`, `workbench.action.manageTrustedDomain`. Messages 11819-11866, 11521.

### DL-6 Cursor plan and account modals

- Tag: cursor (nothing like it in V)
- Evidence: bundle
- Boundary: yes. The modals are opened from Cursor Settings screens and, for the usage pricing box, from the chat queue wait indicator. The brief names Cursor Settings as a boundary item.
- Found in bundle (modal opener methods on Cursor's UI overlay service, bundle name `P1`):
  - Subscription tiers modal: method `showSubscriptionTiersModal`, command `cursor.showSubscriptionTiersModal`, title "Show Subscription Tiers Modal", listed in the Command Palette (F1)
  - Usage pricing modal: method `showUsageBasedPricingModal(...)`, command `cursorai.action.showUsagePricingModal`, title "Show Usage Pricing Modal", F1. Also opened by clicking the wait or queue indicator in chat
  - Upgrade confirmation modal: `showUpgradeConfirmationModal(tier)` and `showServerUpgradeConfirmationModal`. Flow helper `eAt`: if the account has a valid payment method it shows the confirmation modal, otherwise (or on error) it falls back to the Stripe page
  - Invite Team Members modal: `showInviteModal`, opened by the "Invite" button on the General tab of Cursor Settings
  - Switch Privacy Mode confirm (pretty dialog titled "Switch Privacy Mode?", with a warning callout "You won't be able to switch back to Privacy Mode (Legacy).", buttons "Switch" and "Cancel"), and an error box "Failed to update privacy settings". Both are in the privacy section of Cursor Settings
  - Import settings from VS Code confirm (pretty dialog titled "Import settings from VS Code", text "Settings, extensions, and other profile preferences will be replaced...", buttons "Import", "Import without extensions", "Cancel"). It is opened by the "Import" button on a notification: "Bring your VS Code setup to Cursor. Import your settings and extensions for a seamless start." (messages 8445, 8446)
- Modal styling: class family `.ui-dialog` and `.ui-dialog-backdrop` (backdrop colour rgba(0,0,0,.4), z-index 2551) in `workbench.desktop.main.css`
- Not counted here (see left out): GitHub connect, manage sharing, steer from cloud, local agent configuration, report feedback and docs modals, since they serve chat or cloud agents
- Proof: main bundle search `showSubscriptionTiersModal`, `showUsageBasedPricingModal`, `showUpgradeConfirmationModal`, `showInviteModal`, `Switch Privacy Mode?`, `Import settings from`. Command ids `cursor.showSubscriptionTiersModal` and `cursorai.action.showUsagePricingModal`.

## Cursor additions

- Pretty dialog engine (`openDialog`, DL-1): Cursor's own in-page modal with callout blocks, copyable value blocks, extra and danger buttons, icons, and a persistent "don't ask again" memory (`dialogDontAskAgainPreferences`). Seen `dialogKey` values: `quitConfirmation`, `closeWindowConfirmation`, `composer-checkout-to-message`, `submit-from-previous-message-2`. The last two are chat. Not controlled by `window.dialogStyle`
- Custom About window with a built-in update button, per-field copy buttons, and Cursor-only lines in the copied text (`Layout`, `Build Type`, `Release Track`, `xterm.js`, `VS Code Extension API`, Local Mode) (DL-1)
- Native message box option `showOverHiddenWindowInWindow` (DL-1)
- Quit Cursor? and Close this window? confirmations with "don't ask again" (DL-2). They ask by default
- "Save Error" pretty dialog used instead of the error notification when the save options include `showPrettyDialogOnError` (DL-2)
- Workspace Trust default off, startup prompt default `once`, team admin switch `workspaceTrustControls.enabled`, enterprise policy name `WorkspaceTrustEnabled` (DL-5)
- Plan and account modals: subscription tiers, usage pricing, upgrade confirmation, invite, privacy mode switch, import VS Code settings (DL-6)

## Left out (and why)

- Agent, chat and cloud-agent dialogs that use Cursor's pretty dialog engine. Titles seen: "Close Running Tab?", "Commit on Main Branch", "Create Pull Request", "Push local branch to remote", "Incorrect Branch", "Incorrect Repository", "Cloud Agents require a Git repo", "Cloud Agents Not Supported in Privacy Mode (Legacy)", "Applying changes locally...", "Checking out branch...", "Discard all changes up to this checkpoint?", "Revert files to this message?", "Enable Run Everything?", "Leave Ask Every Time?", "Reset Chat Mode to Default", "About Sandbox" and "About Allowlist", "Enable Sandboxing", "Create chat with prompt" (deeplink), "Create Custom Command", "Create Custom Rule", "Override Custom Command", "Override Custom Rule", "Delete Rule", "Delete automation", "Merge Conflicts Detected", "Another agent's changes have been applied", share transcript, "Workspace Not Found" (deeplink for a bug report link), the streaming-agents and background-tasks prompts on quit, close and reload, and the missing bug report files prompt
- MCP prompts: "Move {server} to your Cursor account?" and the remove-server confirm in Customize
- UI overlay modals that serve chat or cloud agents: GitHub connect (`showGithubConnectModal`), manage sharing (`showManageSharingModal`), steer from cloud (`showSteerFromCloudModal`), local agent configuration (`cursor.openLocalAgentConfigurationModal`), report feedback (`showReportFeedbackModal`), docs (`showDocsModal`)
- AI login-required popup ("Please login to continue using Cursor." and "As a safety measure for our users, please login to use AI agents.") - it is an AI error popup, chat flavored
- "Welcome to Cursor" onboarding (`workbench.contrib.onboardingAgentFirst`): it opens an agent chat named "Welcome to Cursor" and plays a scripted greeting. Agent surface. The context key `cursor.onboarding.showing` marks it
- "Import VS Code Configuration" confirm inside the Extensions view header (extension management UI, out of scope). The "Import settings from VS Code" confirm from the notification is counted in DL-6
- Display language restart confirm "Restart {0} to switch to {1}?" (messages 13013-13015). Belongs to display language, not catalogued here
- Extension, authentication and MCP prompts, as in the V file

## Newer than 1.128 (seen only in N)

These are the same `newer-than-1.128` items as in the V file. I checked whether Cursor has them:

- Markdown dialog detail: not in Cursor (zero hits for `detailElement`)
- Vertical dialog alignment: not in Cursor (zero hits for `DialogContentsAlignment` and the option)
- About lines for `@github/copilot` and `@github/copilot-sdk`: not in Cursor (zero hits)
- Simple file dialog `canCreateFolder`: not in Cursor (zero hits)
- Onboarding tryout feature: not in Cursor (zero hits for `onboardingTryout`, `guidedTryout`)

## V items missing from Cursor

- Welcome Onboarding 2026 overlay: `workbench.action.welcomeOnboarding2026` (0 hits), `welcomeOnboarding.state` (0), class `onboarding-a-overlay` (0). Cursor has its own agent-first onboarding instead (left out). Tag: vscode
- Windows "Select folder" button label on the Open Folder dialog (0 hits in `out\main.js` and the message table). Tag: vscode
- The VS Code Sessions window trust note, `getSessionsWindowTrustNote` (0 hits). Window does not exist in Cursor
- V's About text fields `ElectronBuildId` and the V layout of the list. Cursor shows a different list (see DL-1)
- `window.dialogStyle` forcing custom dialogs for the smoke test driver. Test-only, not a UI feature

## Unverified

- I did not run Cursor. Everything above is read from the bundle, so how a box looks (size, position, button order) is not confirmed
- Whether, with `window.confirmBeforeClose` set to `always`, a person really sees two boxes (the Cursor one and then V's "Are you sure you want to exit?"). Read from the quit command code only
- Who sets `showPrettyDialogOnError` on a save. I saw the check but not the caller
- Whether "Don't ask again" choices can be reset from a setting or command. I found the storage key but no reset
- What `workbench.action.quit` does when the quit confirmation is skipped by the `HOw` helper in all cases (I read the condition, not each branch's outcome)
- Whether the About window works on Linux and macOS the same way. The Electron main methods exist, but I read only the renderer side
- I did not map every menu entry that opens each prompt. I traced commands, settings and message numbers
- Cursor's enterprise policy `WorkspaceTrustEnabled` is not in V source that I searched (`src` only, not the `build` policy files)
- The Trust editor body and the Restricted Mode banner and status entry are listed in the V file as boundaries with other sections. Not checked in Cursor beyond hit counts

