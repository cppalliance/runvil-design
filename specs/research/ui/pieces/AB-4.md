Group: AB-4 Accounts and Manage entries. Pass: 3. Date: 2026-10-25.

### AB-4.1
1. Clicking the Accounts icon, a person glyph under the rail with the hover text "Accounts", opens the Accounts menu beside the rail, to the right when the side bar is on the left.
2. Pressing Enter or Space on the focused Accounts icon opens the same menu.
3. The menu lists one submenu per signed-in account, named `{account} ({provider})`.
4. A separator follows the account submenus, and below it come the rows that other features add to the Accounts menu.
5. A disabled "Loading..." row shows until the accounts have resolved.
6. A provider that fails shows a disabled "{0} is currently unavailable" row and is retried in the background.
7. In Cursor the menu shows a disabled "You are not signed in to any accounts" row when providers exist but no account is signed in, and a disabled "No accounts requested yet..." row when there is no provider. [cursor only]

### AB-4.2
1. Picking Sign Out in an account's submenu opens a confirm box reading "Sign out of '{0}'?" with a Sign Out button.
2. When extensions have used the account, the confirm box lists them first and reads "The account '{0}' has been used by: ... Sign out from these extensions?".
3. Confirming removes the account's sessions, its saved extension usage and its allowed-extension records.
4. The Sign Out row is left out for an account that the host reports as unable to sign out.
5. Running "Accounts: Manage Accounts" from the Command Palette asks you to pick an account, then lists actions for it including Manage Trusted Extensions and Sign Out. [stock only]
6. With no signed-in account, "Accounts: Manage Accounts" shows a single row reading "There are no active accounts.". [stock only]

### AB-4.3
1. Picking Manage Trusted Extensions in an account's submenu opens a checklist titled "Manage Trusted Extensions" with the placeholder "Choose which extensions can access this account".
2. The checklist holds the extensions that have used or been allowed on that account, newest use first, each with "Last used this account {time}" or "Has not used this account" when it never has.
3. Unticking an extension revokes its access and ticking it allows access, and the change is applied when you accept.
4. Extensions trusted by Microsoft sit under a "Trusted by Microsoft" separator, are locked on, and show a tooltip saying they always have access to the account.
5. Each row has two buttons, one to view the extension's details and one to manage account preferences for that extension.
6. When no extension has used the account, an info box says "This account has not been used by any extensions.".
7. Choosing Manage Trusted Extensions after picking an account in "Accounts: Manage Accounts" opens the same checklist for that account. [stock only]

### AB-4.4
1. Picking "Manage Extension Account Preferences..." at the bottom of the Accounts menu opens a picker of the extensions that have used an account. [stock only]
2. When the chosen extension has used more than one provider, a second picker asks which provider to manage.
3. The provider's account list is titled "'{extension}' Account Preferences For This Workspace", marks the account in use as "Current account", and picking another account sets it for this workspace.
4. For a provider that allows several accounts, the list also offers "Use a new account...".
5. When the extension has used no account, an info box says "This extension has not used any accounts yet.".
6. In Cursor the Accounts menu has no row for this picker and the Command Palette does not list it. [cursor only]

### AB-4.5
1. When an extension asks to sign in, the Accounts menu gains a row reading "Sign in with {provider} to use {extension} (1)", and picking it starts the provider's sign-in.
2. When an extension asks for access to a signed-in account, the Accounts menu gains a row reading "Grant access to {provider} for {extension}... (1)".
3. Picking an access row opens an Allow or Deny box reading "The extension '{0}' wants to access the {1} account '{2}'.".
4. When several accounts exist, the access row opens an account picker instead, which includes "Sign in to another account".
5. The Accounts icon shows a number badge with the hover text "Sign in requested" that counts the open requests.
6. Once a request is answered, its row disappears from the menu and its share of the badge clears.

### AB-4.6
1. When the tunnel is off, the Accounts menu shows "Turn on Remote Tunnel Access...", and picking it starts turning the tunnel on.
2. While the tunnel is connecting, the menu shows "Remote Tunnel Access is Connecting".
3. Once the tunnel is running, the menu shows "Remote Tunnel Access is On".
4. Picking the connecting or on row opens a list with Learn More, Copy Browser URL, Show Log, Change Tunnel Name and Turn Off.
5. These rows are offered only in the desktop build.

### AB-4.7
1. Picking "Backup and Sync Settings..." in the Accounts menu or the Manage menu starts Settings Sync. [stock only]
2. While Settings Sync is starting, both menus show a disabled "Turning on Settings Sync..." row. [stock only]
3. Once Settings Sync is on, a ticked "Settings Sync is On" row opens a list with Configure..., Show Settings, Show Synced Data, Sync Now and Turn Off. [stock only]
4. In that list, Sync Now reads "syncing" or "synced {time}" beside it, and Turn Off shows the account name and provider. [stock only]
5. When the sync account has signed out, Manage shows "Sign in to Sync Settings" and Accounts shows "Sign in to Sync Settings (1)" with a count badge on the Accounts icon. [stock only]
6. Setting `settingsSync.ignoredSettings` lists the settings that Settings Sync leaves out.
7. Setting `settingsSync.ignoredExtensions` lists the extensions, as `{publisher}.{name}`, that Settings Sync leaves out.
8. Setting `settingsSync.keybindingsPerPlatform`, on by default, chooses whether keybindings sync separately for each platform.

### AB-4.8
1. When Settings Sync finds conflicts, the Manage gear shows a number badge with the conflict count and the hover text "{0}: Conflicts Detected". [stock only]
2. The Manage menu then shows "Show Conflicts ({n})", and picking it opens the conflict resolver. [stock only]
3. While Settings Sync is turning on, the gear shows a progress badge instead. [stock only]

### AB-4.9
1. While signed out, the Accounts menu shows "Turn on Cloud Changes...", and picking it signs in to store working changes in the cloud. [stock only]
2. While a sign-in request is pending, the row reads "Turn on Cloud Changes... (1)" and the Accounts icon shows a badge. [stock only]
3. Once signed in, the row becomes "Turn off Cloud Changes...", and picking it asks "Do you want to disable storing working changes in the cloud?" with a checkbox "Delete all stored data from the cloud." and signs out on confirm. [stock only]
4. Both actions are also registered for the Command Palette, so "Turn on Cloud Changes..." and "Turn off Cloud Changes..." can be run from it. [stock only]

### AB-4.10
1. Right-clicking the Accounts icon shows a Hide Accounts entry, and picking it removes the Accounts icon from the rail.
2. The rail's right-click menu has an "Accounts" check item, which is ticked while the icon shows, and unticking it hides the icon and ticking it again brings the icon back.
3. The Manage gear stays in place when Accounts is hidden, and nothing hides the gear.
4. The choice is stored per profile under the storage key `workbench.activity.showAccounts`, shown by default, and is not a setting.

### AB-4.11
1. The Accounts icon shows the signed-in account's profile picture in place of the person glyph, which VS Code adds after 1.128 (newer-than-1.128). [stock only]
2. If the picture fails to load, the person glyph is shown again. [stock only]
3. Setting `workbench.accounts.showAvatar`, on by default, chooses whether the picture is shown, and changing it updates the icon (newer-than-1.128). [stock only]

### AB-4.12
1. Clicking the gear under the rail, with the hover text "Manage", opens the Manage menu, and pressing Enter or Space on the focused gear opens it too.
2. The menu stacks its rows in groups: Command Palette first, then Profile, Settings, Extensions, Keyboard Shortcuts, Snippets, Tasks and Themes, then the update rows.
3. A group of Settings Sync rows sits between the configuration rows and the update rows. [stock only]
4. The rows use their short titles, so the menu reads "Keyboard Shortcuts" and "Snippets" rather than the full command titles.

### AB-4.13
1. While a non-default profile is active, hovering the gear shows "Manage {name} (Profile)".
2. When that profile has its own icon, the icon replaces the gear glyph.
3. When the profile has no icon, the gear gets a two-letter uppercase badge made from the profile name.
4. The two-letter badge is hidden while a count or progress badge, such as an update or sync badge, is showing on the gear.

### AB-4.14
1. The first row of the Manage menu, "Command Palette...", opens the Command Palette.
2. The row sits alone in the first group of the menu, above the configuration rows.

### AB-4.15
1. The "Settings" row in the Manage menu opens the Settings editor.
2. The row sits in the configuration group right after Profile and before Extensions.
3. Pressing `Ctrl+,` opens the same Settings editor.

### AB-4.16
1. The "Keyboard Shortcuts" row in the Manage menu opens the Keyboard Shortcuts editor.
2. The menu row shows the short title "Keyboard Shortcuts" while the command itself is titled "Open Keyboard Shortcuts".
3. Pressing `Ctrl+K Ctrl+S` opens the same editor.

### AB-4.17
1. When only the default profile exists, the Manage menu shows a plain "Profiles" row that opens the Profiles editor.
2. When more than one profile exists, the Manage menu shows a "Profile ({current name})" submenu instead.
3. The submenu lists every profile with a tick on the active one, and picking another profile switches to it.
4. The submenu also has a "Profiles" row that opens the Profiles editor.

### AB-4.18
1. The "Snippets" row in the Manage menu opens the snippet file picker.
2. The menu row shows the short title "Snippets" while the command itself is titled "Configure Snippets".

### AB-4.19
1. The "Tasks" row in the Manage menu opens the user-level `tasks.json` for editing.
2. The row shows only where task execution is supported.

### AB-4.20
1. The "Themes" row in the Manage menu opens a submenu with Color Theme, File Icon Theme and Product Icon Theme.
2. Color Theme opens the color theme picker, and pressing `Ctrl+K Ctrl+T` opens the same picker.
3. File Icon Theme opens the file icon theme picker.
4. Product Icon Theme opens the product icon theme picker.

### AB-4.21
1. When no update work is under way, the last group of the Manage menu shows "Check for Updates...", and picking it checks for an update.
2. When an update can be downloaded, the row reads "Download Update (1)", and picking it starts the download.
3. When an update has been downloaded and waits for install, the row reads "Install Update... (1)", and picking it starts the install.
4. When an update is ready, the row reads "Restart to Update (1)", and picking it quits and installs the update.
5. While the updater is busy, the last group shows one disabled row that names the step: "Checking for Updates...", "Downloading Update..." or "Installing Update...".
6. The gear shows a count badge of 1 with the hover text "New {0} update available." when an update is available, downloaded or ready, and a progress badge while the updater is checking, downloading or installing.
7. A check that finds no update opens an info box reading "There are currently no updates available.". [stock only]
8. While an update is being cancelled, a disabled "Cancelling Update..." row shows and the gear shows a progress badge. [stock only]
9. Setting `update.mode` chooses how updates are found, with values `none`, `manual`, `start` and `default`.
10. In Cursor, `update.mode` also accepts the value `silentlyApplyOnQuit`. [cursor only]

### AB-4.22
1. When an update is ready, it is a major or minor version step and the build is stable, the Manage menu shows "Show Update Release Notes" above "Restart to Update".
2. Picking "Show Update Release Notes" opens the release notes for the new version.
3. For a patch-level update, or on a build that is not stable, the row is not shown.
4. Setting `update.showReleaseNotes`, on by default, chooses whether release notes are shown after an update. [stock only]

### AB-4.23
1. In the rail, pressing Down or Right on the view container icons moves focus into the Accounts and Manage icons.
2. Pressing Up or Left on those two icons moves focus back to the last view container icon.
3. Focus does not wrap round when it reaches either end of the two icons.
4. A screen reader announces the Accounts and Manage icons as one vertical action bar labelled "Manage".
5. Running `workbench.action.focusActivityBar` puts keyboard focus on the rail, and it has no default key.
6. In Cursor, `workbench.action.focusActivityBar` is enabled only when `workbench.activityBar.orientation` is `vertical`, and the default horizontal tab row has no keyboard handling at all. [cursor only]

### AB-4.24
1. With `workbench.activityBar.location` set to `top` or `bottom`, the rail is not drawn and the right end of the title bar toolbar shows the Accounts icon, if it is shown, and the Manage gear, after the layout and global buttons. [stock only]
2. The two icons never fold into the toolbar's "..." overflow menu, even when the toolbar is over its 12 item limit. [stock only]
3. Clicking either icon opens the same Accounts or Manage menu as on the rail, with the menu opening below the icon, and the gear keeps its profile badge and update badge. [stock only]
4. The two icons are not drawn in auxiliary windows or in compact title bars. [stock only]
5. With location `default` the two icons live in the rail, and with `hidden` they are drawn nowhere.
6. In Cursor, with location `top` or `bottom` the Accounts icon and the Manage gear appear nowhere, because the title bar never draws them. [cursor only]

### AB-4.25
1. Right-clicking the title bar Accounts icon shows "Hide Accounts", a separator, an "Accounts" check item and a "Manage" check item. [stock only]
2. The "Manage" check item is ticked and disabled, because Manage cannot be hidden. [stock only]
3. Right-clicking the title bar Manage gear shows only the "Accounts" and "Manage" check items. [stock only]
4. Picking Hide Accounts in this menu removes the Accounts icon from both the title bar and the rail, because both read the same storage key. [stock only]
5. In Cursor this menu is never reached, because the title bar does not draw the Accounts or Manage icons. [cursor only]

## Unclear
- Cursor draws the Accounts icon and the Manage gear only on its vertical rail, which is not the default layout. Pieces with no tag describe that vertical rail, and in Cursor's default horizontal tab row (AB-6) none of them is visible.
- Tag convention used here: a piece that Cursor lacks ends with `[stock only]`. Where the Cursor feature field says `changed`, one extra piece states what Cursor does instead and ends with `[cursor only]`, even when that is an absence such as an icon never drawn.
- AB-4.21 has more than 10 behaviors and should be split. The setting `update.enableWindowsBackgroundUpdates` (Windows only, background download and install) was left out of the 10 pieces.
- AB-4.21 piece 10: the extra `update.mode` value `silentlyApplyOnQuit` was read from the Cursor bundle's setting definition. What it does was not researched.
- AB-4.11 is evidence from the newer source tree only (newer-than-1.128), so its pieces are `[stock only]`.
- AB-4.5 leaves out stock's MCP server request variants (out of scope) and its Settings Sync, Cloud Changes and Marketplace badge sources, which are covered by AB-4.7 and AB-4.9 or were skipped on purpose by pass 2.
- AB-4.2 piece 6, AB-4.3 pieces 1 to 6, AB-4.4 pieces 2 to 5, AB-4.7 pieces 3 and 4, AB-4.9 pieces 3 and 4 and AB-4.21 piece 5 use labels and states read from the stock source files, which are more detailed than the feature blocks. The Cursor copies of these strings were not each searched for.
- AB-4.9 piece 4 comes from the stock source registering both commands for the Command Palette. Cursor has neither command, so it is tagged `[stock only]`.
- AB-4.3 piece 4 mentions a tooltip and a "Trusted by Microsoft" separator read from stock source. Whether Cursor's older copy has the same wording was not checked.
- Nothing was run. Pieces come from source and bundle strings.
