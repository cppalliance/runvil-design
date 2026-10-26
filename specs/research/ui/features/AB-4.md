Group: AB-4 Accounts and Manage entries. Section: Activity bar (AB). Date: 2026-10-25. Pass: 2.
Features: 25.

### AB-4.1 Open the Accounts menu and see signed-in accounts
- Tag: both
- Evidence: source
- Boundary: no
- What: Click the Accounts icon (a person glyph under the rail, hover text "Accounts") to open a menu with one submenu per signed-in account, named `{account} ({provider})`, then a separator and the rows that other features add to the Accounts menu. Click, Enter or Space opens it, beside the rail (to the right when the side bar is on the left). A disabled "Loading..." row shows until accounts resolve, and a provider that fails shows a disabled "{0} is currently unavailable" and is retried in the background. The action id `workbench.actions.accounts` is a view item id, not a registered command.
- Commands: workbench.actions.accounts
- Menus: AccountsContext
- Keys: none found
- Settings: none found
- Cursor: changed - drawn only in the vertical rail, and it runs an older copy of the menu code: a disabled "You are not signed in to any accounts" row when providers exist but no account does, and a disabled "No accounts requested yet..." placeholder (`noAuthenticationProviders`) when no provider exists. V has neither row.
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (lines 47-151, 258-500); V\src\vs\workbench\common\activity.ts (line 7); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3838-3847); C bundle: vs/workbench/contrib/authentication/browser/authentication.contribution (NLS 4887-4888)

### AB-4.2 Sign out of an account
- Tag: both
- Evidence: source
- Boundary: no
- What: Open an account's submenu in the Accounts menu and pick Sign Out. A confirm box asks "Sign out of '{0}'?", or if extensions have used the account it lists them first ("The account '{0}' has been used by: ... Sign out from these extensions?"), with a Sign Out button. On confirm it removes the account's sessions and its saved extension usage and allowed-extension records. The Sign Out row is left out for an account the host says cannot sign out (an embedder session). The same action is reachable from the Command Palette entry "Accounts: Manage Accounts", which picks an account and then offers Manage Trusted Extensions, Manage Trusted MCP Servers and Sign Out. The submenu rows are built in code, not registered on a menu.
- Commands: _signOutOfAccount; workbench.action.manageAccounts; authentication.command.anysphere.signOut
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - the Sign Out row, `_signOutOfAccount` and the confirm strings are present (vertical rail only). The Command Palette route `workbench.action.manageAccounts` is missing (0 hits for the id and its strings).
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (389-420, 470-482); V\src\vs\workbench\contrib\authentication\browser\actions\signOutOfAccountAction.ts; V\src\vs\workbench\contrib\authentication\browser\actions\manageAccountsAction.ts; V\src\vs\workbench\contrib\authentication\browser\authentication.contribution.ts (92-98); C bundle: vs/workbench/contrib/authentication/browser/actions/signOutOfAccountAction (NLS 4880-4883); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3846)

### AB-4.3 Choose which extensions can use an account
- Tag: both
- Evidence: source
- Boundary: no
- What: In an account's submenu pick Manage Trusted Extensions to open a checklist of extensions that have used or been allowed on that account, newest use first, each with "Last used this account {time}". Untick to revoke or tick to allow, then accept. Extensions trusted by Microsoft sit under a separator and are locked on. Each row has buttons to view the extension and to open its account preferences. This is account access control in the authentication code, not the Extensions view (see Unclear). Also reachable from the Command Palette, which first asks which account.
- Commands: _manageTrustedExtensionsForAccount; workbench.action.manageAccounts
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (vertical rail only) - an older copy of the action with the same command id, title and "Choose which extensions can access this account" text. The `workbench.action.manageAccounts` route is missing.
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (389-400); V\src\vs\workbench\contrib\authentication\browser\actions\manageTrustedExtensionsForAccountAction.ts (21-220); C bundle: vs/workbench/contrib/authentication/browser/actions/manageTrustedExtensionsForAccountAction (NLS 4876-4879); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3845)

### AB-4.4 Choose which account an extension uses
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick "Manage Extension Account Preferences..." at the bottom of the Accounts menu, choose an extension that has used an account (then a provider if it used several), and pick the account it should use for this workspace from the provider's list. Providers that allow several accounts also offer "Use a new account...". If the extension has used no account it says "This extension has not used any accounts yet."
- Commands: _manageAccountPreferencesForExtension
- Menus: AccountsContext
- Keys: none found
- Settings: none found
- Cursor: changed - the command and its strings are present, but it has no Accounts menu row and is hidden from the Command Palette (`f1:!1`, no `menu`). Only the Extensions view context menu calls it, so AB-4 cannot reach it in Cursor.
- Source: V\src\vs\workbench\contrib\authentication\browser\actions\manageAccountPreferencesForExtensionAction.ts (19-135); V\src\vs\workbench\contrib\extensions\browser\extensions.contribution.ts (1784); C bundle: vs/workbench/contrib/authentication/browser/actions/manageAccountPreferencesForExtensionAction (NLS 4859-4867)

### AB-4.5 Answer an extension's sign-in or access request
- Tag: both
- Evidence: source
- Boundary: no
- What: When an extension asks to use an account, the Accounts menu gains a row "Sign in with {provider} to use {extension} (1)" or "Grant access to {provider} for {extension}... (1)", and the Accounts icon shows a number badge (hover "Sign in requested") counting open requests. A sign-in row starts the provider's sign-in. An access row opens either an Allow or Deny box ("The extension '{0}' wants to access the {1} account '{2}'.") or, when several accounts exist, an account picker with "Sign in to another account". The row and the badge clear once answered. Stock also feeds the Accounts badge from MCP server requests, Settings Sync sign-in, Cloud Changes and Marketplace sign-in (see AB-4.7 and AB-4.9).
- Commands: {providerId}:{extensionId}:signIn{n} (dynamic); {providerId}{extensionId}Access (dynamic)
- Menus: AccountsContext (groups 2_signInRequests and 3_accessRequests)
- Keys: none found
- Settings: none found
- Cursor: present (vertical rail only) - extension sign-in and access rows and the number badge are kept. Stock's MCP variants, Settings Sync badge, Cloud Changes badge and Marketplace sign-in badge are not in the bundle (only one `showAccountsActivity` call site found).
- Source: V\src\vs\workbench\services\authentication\browser\authenticationExtensionsService.ts (131-150, 253-283, 393-505); V\src\vs\workbench\services\authentication\browser\authenticationMcpService.ts (206, 453, 513); C bundle: vs/workbench/services/authentication/browser/authenticationExtensionsService (NLS 12298-12306)

### AB-4.6 Turn on or manage Remote Tunnel Access
- Tag: both
- Evidence: source
- Boundary: no
- What: The Accounts menu shows "Turn on Remote Tunnel Access..." when the tunnel is off. While connecting it shows "Remote Tunnel Access is Connecting", and once on it shows "Remote Tunnel Access is On". The last two open a list: Learn More, Copy Browser URL, Show Log, Change Tunnel Name, and Turn Off. Desktop build only.
- Commands: workbench.remoteTunnel.actions.turnOn; workbench.remoteTunnel.actions.manage; workbench.remoteTunnel.actions.connecting
- Menus: AccountsContext (group 2_remoteTunnel)
- Keys: none found
- Settings: none found
- Cursor: present (vertical rail only) - the three registrations and ids are in the bundle. Whether the feature activates depends on product config that was not checked (see Unclear).
- Source: V\src\vs\workbench\contrib\remoteTunnel\electron-browser\remoteTunnel.contribution.ts (489-630, 756-790); C bundle: vs/workbench/contrib/remoteTunnel/electron-sandbox/remoteTunnel.contribution (NLS 9164-9165)

### AB-4.7 Turn on and manage Settings Sync
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Both menus offer "Backup and Sync Settings..." to start Settings Sync (Manage group 3_configuration, Accounts group 1_settings). While it starts a disabled "Turning on Settings Sync..." row shows. Once on, a ticked "Settings Sync is On" row opens a list of sync commands. If the sync account has signed out, Manage shows "Sign in to Sync Settings" and Accounts shows "Sign in to Sync Settings (1)" with a count badge on the Accounts icon. Cursor drops all of these.
- Commands: workbench.userDataSync.actions.turnOn; workbench.userData.actions.turningOn; workbench.userData.actions.signin; workbench.userDataSync.actions.manage
- Menus: AccountsContext; GlobalActivity
- Keys: none found
- Settings: settingsSync.ignoredSettings; settingsSync.ignoredExtensions; settingsSync.keybindingsPerPlatform
- Cursor: missing - 0 hits for `workbench.userDataSync.actions.*`, `workbench.userData.actions.*` and the labels "Backup and Sync Settings", "Settings Sync is On" and "Sign in to Sync Settings". The three `settingsSync.*` setting ids are still registered in the bundle.
- Source: V\src\vs\workbench\contrib\userDataSync\browser\userDataSync.ts (765-875, 912-960, 467-483); V\src\vs\platform\userDataSync\common\userDataSync.ts (79-115)

### AB-4.8 Resolve Settings Sync conflicts from Manage
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When sync finds conflicts the Manage gear gets a number badge (hover "{0}: Conflicts Detected") and the Manage menu shows "Show Conflicts ({n})", which opens the conflict resolver. While Settings Sync is turning on, the gear shows a progress badge instead.
- Commands: workbench.userDataSync.actions.showConflicts
- Menus: GlobalActivity (group 3_configuration)
- Keys: none found
- Settings: none found
- Cursor: missing - 0 hits for `workbench.userDataSync.actions.showConflicts` and the "Show Conflicts" label.
- Source: V\src\vs\workbench\contrib\userDataSync\browser\userDataSync.ts (68, 454-470, 877-910)

### AB-4.9 Turn on or off Cloud Changes
- Tag: vscode
- Evidence: source
- Boundary: no
- What: The Accounts menu offers "Turn on Cloud Changes..." (shown as "Turn on Cloud Changes... (1)" with a badge while a request is pending) to sign in for storing edit sessions in the cloud. Once signed in, "Turn off Cloud Changes..." signs out. Only these two menu rows are listed here, not the edit sessions feature itself (catalog decision 9).
- Commands: workbench.editSessions.actions.signIn; workbench.editSessions.actions.resetAuth
- Menus: AccountsContext
- Keys: none found
- Settings: none found
- Cursor: missing - 0 hits for both command ids and the "Turn on Cloud Changes" label.
- Source: V\src\vs\workbench\contrib\editSessions\browser\editSessionsStorageService.ts (462-515); V\src\vs\workbench\contrib\editSessions\browser\editSessions.contribution.ts (256)

### AB-4.10 Hide or show the Accounts icon
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click the Accounts icon and pick Hide Accounts, or untick "Accounts" in the rail's right-click menu, to remove the icon. Tick it again to bring it back. The Manage gear stays and cannot be hidden. The choice is stored per profile in the storage key `workbench.activity.showAccounts` (default shown), not as a setting, and the same key drives both the rail and the title bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (vertical rail only) - Hide Accounts, the "Accounts" toggle and the storage key are all kept.
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (89-153, 258-260, 729-750); V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts (339-343); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3839, 3841, 3850-3852)

### AB-4.11 Show the account picture on the Accounts icon
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. The Accounts icon can show the signed-in account's profile picture in place of the person glyph. `workbench.accounts.showAvatar` (default on) controls it, and the glyph returns if the image fails to load. Evidence is from N only.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.accounts.showAvatar
- Cursor: missing - 0 hits for `workbench.accounts.showAvatar`.
- Source: N\src\vs\workbench\browser\workbench.contribution.ts (669-673); N\src\vs\workbench\browser\parts\globalCompositeBar.ts (284, 350, 398-443); N\src\vs\workbench\services\authentication\common\authentication.ts (22)

### AB-4.12 Open the Manage menu from the gear
- Tag: both
- Evidence: source
- Boundary: no
- What: Click the gear under the rail (hover text "Manage") to open a menu in four stacked groups: Command Palette; then Profile, Settings, Extensions, Keyboard Shortcuts, Snippets, Tasks, Themes; then Settings Sync rows; then update rows. The rows use their short titles. It opens the same way as the Accounts menu (click, Enter, Space). The gear is always drawn. The Extensions row is the extension system and is not listed as a feature. Stock web builds also add "Switch to Insiders Version..." (web only, not listed). The action id `workbench.actions.manage` is a view item id, not a registered command.
- Commands: workbench.actions.manage
- Menus: GlobalActivity
- Keys: none found
- Settings: none found
- Cursor: changed - same gear and groups in the vertical rail, without the Settings Sync rows. No Cursor-only row was found. Cursor's own settings button is a title bar control (TB-5).
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (47-151, 583-660); V\src\vs\workbench\common\activity.ts (line 6); V\src\vs\workbench\contrib\update\browser\update.ts (371-376); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3840, 3848); C bundle: `Pe.GlobalActivity` registrations (18 matches, including the view item class)

### AB-4.13 See the current profile on the gear
- Tag: both
- Evidence: source
- Boundary: no
- What: When a non-default profile is active the gear's hover text becomes "Manage {name} (Profile)". If the profile has its own icon, that icon replaces the gear. If it has no icon, the gear gets a two-letter uppercase badge made from the profile name, hidden while a count or progress badge (updates, sync) is showing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (vertical rail only) - the `profile-text-overlay` badge code and the "Manage {0} (Profile)" string are in the bundle.
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (583-660); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3849)

### AB-4.14 Open the Command Palette from Manage
- Tag: both
- Evidence: source
- Boundary: no
- What: The first row of the Manage menu, "Command Palette...", opens the Command Palette. Only this menu row is listed here. The palette itself is QA-1.
- Commands: workbench.action.showCommands
- Menus: GlobalActivity (group 1_command, order 1)
- Keys: F1; Ctrl+Shift+P
- Settings: none found
- Cursor: present (vertical rail only) - the `GlobalActivity` registration with group `1_command` and order 1 is in the bundle.
- Source: V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (90-98); V\src\vs\workbench\contrib\quickaccess\browser\commandsQuickAccess.ts (273-289); C bundle: vs/workbench/contrib/quickaccess/browser/quickAccess.contribution (NLS 8908)

### AB-4.15 Open Settings from Manage
- Tag: both
- Evidence: source
- Boundary: no
- What: The "Settings" row opens the Settings editor. Only this menu row is listed here. The editor is ED-8.
- Commands: workbench.action.openSettings
- Menus: GlobalActivity (group 2_configuration, order 2)
- Keys: Ctrl+,
- Settings: none found
- Cursor: present (vertical rail only) - the Manage row has no condition. Cursor's Preferences menu row for the same command carries a condition that Manage lacks (see the menu bar notes for TB-1), so Manage is where plain Settings still shows in the IDE window.
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts (229-250); C bundle: `Pe.GlobalActivity` registration with `Ctrl+Comma` and order 2

### AB-4.16 Open Keyboard Shortcuts from Manage
- Tag: both
- Evidence: source
- Boundary: no
- What: The "Keyboard Shortcuts" row (the command is titled "Open Keyboard Shortcuts", the Manage menu shows its short title) opens the Keyboard Shortcuts editor. Only this menu row is listed here. The editor is ED-8.
- Commands: workbench.action.openGlobalKeybindings
- Menus: GlobalActivity (group 2_configuration, order 4)
- Keys: Ctrl+K Ctrl+S
- Settings: none found
- Cursor: present (vertical rail only) - the command and the `GlobalActivity` registration at order 4 are in the bundle.
- Source: V\src\vs\workbench\contrib\preferences\browser\preferences.contribution.ts (880-915); C bundle: `Pe.GlobalActivity` registration with `openGlobalKeybindingSettings` and order 4

### AB-4.17 Switch or manage profiles from Manage
- Tag: both
- Evidence: source
- Boundary: no
- What: With more than one profile, Manage shows "Profile ({current name})", a submenu that lists every profile with a tick on the active one (pick to switch) and a "Profiles" row that opens the Profiles editor. With only the default profile, Manage shows a plain "Profiles" row that opens the Profiles editor. The editor and the Command Palette routes (Switch Profile..., New Window with Profile...) are not cataloged here (see Unclear).
- Commands: workbench.profiles.actions.manageProfiles; workbench.profiles.actions.switchProfile; workbench.profiles.actions.profileEntry.{profile id} (dynamic)
- Menus: GlobalActivity (group 2_configuration, order 1); ProfilesMenu
- Keys: none found
- Settings: none found
- Cursor: present (vertical rail only) - both Manage registrations (submenu when `hasProfiles`, plain row when not), the manage command and `switchProfile` are in the bundle.
- Source: V\src\vs\workbench\contrib\userDataProfile\browser\userDataProfile.ts (73-75, 176-198, 221-243, 322-392); V\src\vs\workbench\services\userDataProfile\common\userDataProfile.ts (155); C bundle: vs/workbench/contrib/userDataProfile/browser/userDataProfile (NLS 11535)

### AB-4.18 Configure snippets from Manage
- Tag: both
- Evidence: source
- Boundary: no
- What: The "Snippets" row (the command is titled "Configure Snippets", the Manage menu shows its short title) opens the snippet file picker. Only this menu row is listed here. The picker belongs with AC-4.
- Commands: workbench.action.openSnippets
- Menus: GlobalActivity (group 2_configuration, order 5)
- Keys: none found
- Settings: none found
- Cursor: present (vertical rail only) - the command and the `GlobalActivity` registration at order 5 are in the bundle.
- Source: V\src\vs\workbench\contrib\snippets\browser\commands\configureSnippets.ts (244-262); C bundle: `Pe.GlobalActivity` registration with order 5 beside `Pe.MenubarPreferencesMenu`

### AB-4.19 Open user tasks from Manage
- Tag: both
- Evidence: source
- Boundary: no
- What: The "Tasks" row opens the user-level `tasks.json` for editing. It shows only where task execution is supported.
- Commands: workbench.action.tasks.openUserTasks
- Menus: GlobalActivity (group 2_configuration, order 6)
- Keys: none found
- Settings: none found
- Cursor: present (vertical rail only) - the command and the `GlobalActivity` registration at order 6 are in the bundle.
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (343-375); V\src\vs\workbench\contrib\tasks\common\taskService.ts (26)

### AB-4.20 Pick a theme from Manage
- Tag: both
- Evidence: source
- Boundary: no
- What: The "Themes" row opens a submenu with Color Theme, File Icon Theme and Product Icon Theme. Each opens its own picker. Only the submenu is listed here. The three pickers have no owning group (see Unclear).
- Commands: workbench.action.selectTheme; workbench.action.selectIconTheme; workbench.action.selectProductIconTheme
- Menus: GlobalActivity (group 2_configuration, order 7); ThemesSubMenu
- Keys: Ctrl+K Ctrl+T (Color Theme only)
- Settings: workbench.colorTheme; workbench.iconTheme; workbench.productIconTheme
- Cursor: present (vertical rail only) - same three rows. The submenu order values are 1, 3 and 4 in the bundle against 1, 2 and 3 in V, so the visible order is unchanged.
- Source: V\src\vs\workbench\contrib\themes\browser\themes.contribution.ts (405-418, 490, 527, 902-937); V\src\vs\workbench\services\themes\common\workbenchThemeService.ts (24-26); C bundle: vs/workbench/contrib/themes/browser/themes.contribution (NLS 11362-11366)

### AB-4.21 Check for, download and install an update from Manage
- Tag: both
- Evidence: source
- Boundary: no
- What: The bottom group of Manage changes with the updater state: "Check for Updates..." when idle, a disabled "Checking for Updates...", "Download Update (1)", a disabled "Downloading Update...", "Install Update... (1)", a disabled "Installing Update...", and "Restart to Update (1)". The gear shows a count badge of 1 when an update is available or ready ("New {0} update available.") and a progress badge while checking, downloading or installing. A check that finds nothing opens an info box, "There are currently no updates available." Stock also has a disabled "Cancelling Update..." row. The title bar Update button is TB-5 and the update cards are NT-6.
- Commands: update.check; update.checking; update.downloadNow; update.downloading; update.install; update.updating; update.cancelling; update.restart
- Menus: GlobalActivity (group 7_update)
- Keys: none found
- Settings: update.mode; update.enableWindowsBackgroundUpdates
- Cursor: changed - the same rows and the gear badge call are in the bundle (vertical rail only). The "Cancelling Update..." row and the "no updates available" info box are not (0 hits).
- Source: V\src\vs\workbench\contrib\update\browser\update.ts (80-157, 260-318); V\src\vs\platform\update\common\update.config.contribution.ts (19-75); C bundle: vs/workbench/contrib/update/browser/update (NLS 11511-11514)

### AB-4.22 Read the update release notes from Manage
- Tag: both
- Evidence: source
- Boundary: no
- What: When an update is ready, it is a major or minor version step and the build is stable, Manage shows "Show Update Release Notes" above "Restart to Update". It opens the release notes for the new version. The release notes page itself is not cataloged here (see Unclear).
- Commands: update.showUpdateReleaseNotes
- Menus: GlobalActivity (group 7_update, order 1)
- Keys: none found
- Settings: update.showReleaseNotes
- Cursor: present (vertical rail only) - the command and the row (condition: state is ready and a major or minor update) are in the bundle. `update.showReleaseNotes` is not.
- Source: V\src\vs\workbench\contrib\update\browser\update.ts (270-280, 320-341); V\src\vs\platform\update\common\update.config.contribution.ts (74-80); C bundle: vs/workbench/contrib/update/browser/update (NLS 11513)

### AB-4.23 Reach Accounts and Manage with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: In the rail, Down or Right arrow on the view container icons moves focus into the Accounts and Manage icons, and Up or Left from them moves back to the last view container icon. Enter or Space on a focused icon opens its menu. Focus does not wrap round (`preventLoopNavigation`), and the two icons are a vertical action bar labelled "Manage" for screen readers. `workbench.action.focusActivityBar` focuses the rail and has no default key.
- Commands: workbench.action.focusActivityBar
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (vertical rail only) - the arrow key listeners that call `globalCompositeBar.focus()` are in the bundle. The focus command is enabled only when `workbench.activityBar.orientation` is `vertical`. The default horizontal tab row has no keyboard handling at all (AB-6).
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts (373-408, 625-640); V\src\vs\workbench\browser\parts\globalCompositeBar.ts (95-103, 218-228); C bundle: the activity bar part's `registerKeyboardNavigationListeners` code (`this.globalCompositeBar?.focus()`)

### AB-4.24 Draw Accounts and Manage in the title bar when the activity bar is top or bottom
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When `workbench.activityBar.location` is `top` or `bottom`, the rail is not drawn, so the title bar's right toolbar ends with the Accounts icon (if shown) and the Manage gear, after the layout and global buttons. They never fold into the toolbar's "..." overflow (exempt from its 12-item limit), the hover opens below them, and they use the same menus, profile badge and update badge as the rail versions. They are not drawn in auxiliary windows or compact title bars. With location `default` they live in the rail. With `hidden` stock draws them nowhere.
- Commands: workbench.actions.accounts; workbench.actions.manage
- Menus: AccountsContext; GlobalActivity
- Keys: none found
- Settings: workbench.activityBar.location
- Cursor: changed - the code, strings and right-toolbar entries (Accounts at order 900, Manage at order 910) are in the bundle, but the title bar's `activityActionsEnabled` getter always returns false, so they are never drawn there. With the vertical rail and location `top` or `bottom` they appear nowhere (derived from code, not run).
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts (37, 631-640, 671-678, 740-751, 800-805, 884-887); V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts (277-292); V\src\vs\workbench\browser\workbench.contribution.ts (650-661); C bundle: vs/workbench/browser/parts/titlebar/titlebarActions (NLS 4022-4025)

### AB-4.25 Right-click the title bar Accounts or Manage icon
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Right-clicking the title bar Accounts icon shows "Hide Accounts", a separator, an "Accounts" check item, and a "Manage" check item that is ticked and disabled (Manage cannot be hidden, and the action throws if forced). Right-clicking the Manage gear shows only the last two. This replaces the rail's right-click menu, which would show the activity bar position and size items. Hide Accounts removes the icon from the title bar and the rail, because both read one storage key.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - the menu code (ids `hideAccounts`, `toggle.hideAccounts`, `toggle.hideManage`) is in the bundle, but it is never reached because the title bar never draws these icons.
- Source: V\src\vs\workbench\browser\parts\globalCompositeBar.ts (662-750); V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts (631-640); C bundle: vs/workbench/browser/parts/globalCompositeBar (NLS 3850-3852)

## Handed off

- Command Palette itself (the `>` list, history, key hints): QA-1. AB-4.14 lists only the Manage row.
- Settings editor and Keyboard Shortcuts editor: ED-8. AB-4.15 and AB-4.16 list only the Manage rows.
- Snippet file picker behind Configure Snippets: AC-4. AB-4.18 lists only the Manage row.
- The rest of the rail right-click menu (Menu and Hide Menu, Activity Bar Position, Activity Bar Size, Move Primary Side Bar, Hide Primary Side Bar, Hide Badge, Move To): AB-5, AB-2 and AB-3. Only the "Accounts" toggle in it is covered here (AB-4.10).
- The title bar Update button and the Cursor Settings button: TB-5. Update cards and what's-new cards: NT-6.
- The Settings Sync and Cloud Changes surfaces beyond these menu rows (sync views, conflict editor, edit sessions): left out by catalog decision 9.

## Unclear

- Title bar group TB-6 was merged into AB-4 (catalog decision 2). AB-4.24 and AB-4.25 cover its placement features.
- AB-4.3, AB-4.4 and AB-4.5 are kept as account access control (code under the authentication contribution), not the Extensions view. If you read them as extension system, drop them. Confidence medium, they are about accounts but the subjects are extensions.
- Skipped on purpose as out of scope: Manage Trusted MCP Servers, Manage Dynamic Authentication Providers..., the MCP forms of the access and sign-in rows, Manage Language Model Access..., Sign in to use GitHub Copilot..., Sign in to access Extensions Marketplace, and the Extensions row in Manage.
- Pass 1 for Cursor said `_manageAccountPreferencesForExtension` was in the Accounts menu. The bundle shows the command with `f1:!1` and no menu entry, so it is recorded here as changed (AB-4.4).
- Remote Tunnel rows (AB-4.6) are registered in Cursor. Whether they ever show depends on product settings that were not checked. Catalog decision 9 left Remote Explorer out, and only the Accounts menu rows are kept here.
- The Profiles editor, Switch Profile... and New Window with Profile... (AB-4.17), the three theme pickers (AB-4.20), and the Release Notes page (AB-4.22) have no owning group in the catalog. They need an owner or a decision to leave them out.
- Newer than 1.128 and not listed as a feature: a "Layout Density" submenu in Manage (`workbench.action.setLayoutDensity.default` and `.compact`, setting `window.density.layout`), shown only when `workbench.experimental.modernUI` is on. Seen in `N\src\vs\workbench\contrib\modernUI\browser\modernUI.contribution.ts` (66-72) and `N\src\vs\workbench\browser\actions\layoutDensityActions.ts`. Cursor has 0 hits. Pass 1 left Modern UI out, so it stays out unless you add it. Also newer: a menu open animation (`menuMotion`) and Codex account rows (agent related) in `N\src\vs\workbench\browser\parts\globalCompositeBar.ts`.
- When `workbench.activityBar.location` is `hidden`, or the custom title bar is hidden, no Accounts or Manage button exists in stock. Derived from the code in `titlebarPart.ts`, not run.
- AB-4.11 is evidence from N only (a newer source tree), not V. The evidence field says `source` because it was read in source.
- Nothing was run. Every Cursor result comes from bundle strings, NLS tables and minified code fragments.
