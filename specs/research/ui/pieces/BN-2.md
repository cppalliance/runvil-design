Group: BN-2 Notices that use the strip. Pass: 3. Date: 2026-10-25.

### BN-2.1
1. When Workspace Trust is on and the window is not trusted, the strip shows a shield icon and the message "Restricted Mode is intended for safe code browsing. Trust this window to enable all features."
2. The message says "folder" or "workspace" in place of "window" when a folder or a workspace is open, and the screen reader label adds "Use navigation keys to access banner actions."
3. The Manage link runs `workbench.trust.manage`, which opens the Workspace Trust editor.
4. The Learn More link opens the Workspace Trust help page in the browser.
5. The strip hides again as soon as the window becomes trusted.
6. Setting `security.workspace.trust.enabled` turns Workspace Trust on or off, and the notice can only show while it is on.
7. Cursor ships `security.workspace.trust.enabled` off by default, so a fresh install never shows this notice until the setting is turned on. [cursor only]

### BN-2.2
1. Setting `security.workspace.trust.banner` to `always` shows the notice every time an untrusted workspace opens.
2. Setting it to `untilDismissed`, the default, shows the notice until the person closes it once for that workspace, and it stays away for that workspace afterward.
3. Setting it to `never` keeps the notice from ever showing.
4. Closing the notice stores a dismissed flag for that workspace only, and the flag only matters in `untilDismissed` mode.

### BN-2.3
1. After the window has restored, if an open folder is the app's install folder or sits inside it, the strip shows a warning icon and a warning message.
2. The message reads "Files you store within the installation folder ('{path}') may be OVERWRITTEN or DELETED IRREVERSIBLY without warning at update time." with the folder's path filled in.
3. The notice has no action links, only the close button.
4. The notice shows only in an installed (built) app, and never in an extension development window or a source checkout run.
5. Nothing remembers a close, so the notice shows again the next time such a folder is opened.

### BN-2.4
1. When the product has an emergency alert address, a desktop window fetches the alert list from it at startup and again every hour. [stock only]
2. The first alert in the list that matches this build's commit, platform, and architecture shows as a warning-icon notice with the alert's message. [stock only]
3. An alert's optional links show as action links in the strip. [stock only]
4. When no alert matches, or the list cannot be read, an alert notice already on screen is hidden. [stock only]
5. When the product has no emergency alert address, nothing is fetched and no notice shows. [stock only]

### BN-2.5
1. When connected to a remote machine whose OS has an old glibc and the person has allowed the connection, the strip shows a warning icon and "You are connected to an OS version that is unsupported by {product}." [stock only]
2. The notice has a Learn More link. [stock only]
3. The close button's tooltip reads "Do not show again in v{version}". [stock only]
4. Closing the notice keeps it hidden until the next major or minor version, and a patch update does not bring it back. [stock only]

### BN-2.6
1. When connected to a 32-bit ARM Linux remote server, the strip shows a warning icon and "Support for 32-bit ARM remote servers is deprecated and will be removed in a future release of {product}." [stock only]
2. The notice has a Learn More link. [stock only]
3. The close button's tooltip reads "Do not show again in v{version}". [stock only]
4. Closing the notice keeps it hidden until the next major or minor version, and a patch update does not bring it back. [stock only]

### BN-2.7
1. In the web build, when the host page passes a `welcomeBanner` option, the strip shows its message, icon, and links at startup. [stock only]
2. The icon is either a codicon name or an image address. [stock only]
3. Closing the notice is remembered in the profile, so it does not show again on later startups. [stock only]
4. A host page that passes no `welcomeBanner` option shows no notice, and the desktop app never shows it. [stock only]

### BN-2.8
1. In VS Code 1.144 and later, when an organization's managed settings need a newer app version, the strip shows a neutral-look notice with an info icon and the requirement message. [stock only]
2. When the update has a status line, it follows the requirement message in the strip. [stock only]
3. The notice has one action link, the update action, when one is available, and no links otherwise. [stock only]
4. The notice has the lowest priority, so any other waiting notice shows ahead of it. [stock only]
5. Closing the notice hides it for the rest of the session only, and it does not change the requirement. [stock only]
6. The notice does not show while `chat.disableAIFeatures` is on. [stock only]
7. A screen reader announces the requirement message when the notice first appears or its message changes. [stock only]

## Unclear

- BN-2.1 has no piece for `security.workspace.trust.startupPrompt`, because it controls the startup dialog and not the strip. The dialog belongs to DL-5 and DL-1.
- BN-2.2 piece 4 and BN-2.1 piece 5 come from reading the code. The feature file says they were not run.
- BN-2.7 may not belong in the catalog, since only a web host page can reach it. Pieces are written anyway so the count matches the feature.
- BN-2.8 is a boundary feature (tied to the chat disable setting and chat read-only mode) and is newer than 1.128.0, so the first piece names the version.
- BN-2.5 and BN-2.6 are both `Unclear` in the feature file about what happens when a remote is both old glibc and 32-bit ARM. No piece covers it.
