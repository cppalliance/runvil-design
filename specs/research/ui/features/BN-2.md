Group: BN-2 Notices that use the strip. Section: Banner. Date: 2026-10-25. Pass: 2.
Features: 8.

### BN-2.1 Restricted Mode notice
- Tag: both
- Evidence: source
- Boundary: no
- What: When Workspace Trust is on and the window is not trusted, the strip shows a shield icon and "Restricted Mode is intended for safe code browsing. Trust this window (or folder, or workspace) to enable all features." with the links Manage (runs `workbench.trust.manage`) and Learn More, and it hides again once trust is granted.
- Commands: workbench.trust.manage
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.enabled; security.workspace.trust.startupPrompt
- Cursor: changed - the code, ids, text and links are the same, but Cursor ships `security.workspace.trust.enabled` off by default, so a fresh install never shows it
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (updateWorkbenchIndicators, showModalOnStart, getBannerItem, getBannerItemMessages, getBannerItemAriaLabels); V\src\vs\workbench\contrib\workspace\browser\workspaceTrustEditor.ts (shieldIcon, icon id `workspace-trust-banner`); V\src\vs\workbench\contrib\workspace\common\workspace.ts (MANAGE_TRUST_COMMAND_ID); C bundle (module: workspace trust startup contribution, ids `workbench.banner.restrictedMode`, `workbench.trust.manage`, icon `workspace-trust-banner`, messages 11868 to 11875, `[h0r]:{type:"boolean",default:!1`)

### BN-2.2 Choose when the Restricted Mode notice appears
- Tag: both
- Evidence: source
- Boundary: no
- What: A setting picks whether the notice shows every time an untrusted workspace opens, shows until the person closes it once for that workspace (the default), or never shows, and closing it stores a per-workspace dismissed flag that only counts in the "until dismissed" mode.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.banner
- Cursor: present
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (getBannerItem onClose, bannerSetting, the registered setting with always, untilDismissed, never); C bundle (module: workspace trust startup contribution and configuration, `security.workspace.trust.banner` default `untilDismissed`, messages 11825 to 11828, `onClose:()=>{e&&this.storageService.store(cgg,!0,1,1)}`)

### BN-2.3 Installation folder warning
- Tag: both
- Evidence: source
- Boundary: no
- What: After the window has restored, if an open folder is the app's install folder or sits inside it (built app, not an extension development window), the strip shows a warning icon and "Files you store within the installation folder ('{path}') may be OVERWRITTEN or DELETED IRREVERSIBLY without warning at update time." with no links, and nothing remembers a close.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\electron-browser\window.ts (handleWarnings, installation dir block); C bundle (module: native window class, `this.bannerService.show({id:"appRootWarning.banner",message:k(12133,null,...),icon:Oe.warning})`, message 12133)

### BN-2.4 Emergency alert
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A desktop window fetches a JSON list from the `emergencyAlertUrl` in product.json at startup and every hour, shows the first alert that matches this build's commit, platform and architecture as a warning-icon notice with optional links, hides it when no alert matches, and does nothing when the product has no URL.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\workbench\contrib\emergencyAlert\electron-browser\emergencyAlert.contribution.ts; V\src\vs\workbench\workbench.desktop.main.ts; V\src\vs\base\common\product.ts; C bundle (searched for `emergencyAlertUrl` and `emergencyAlert.banner`, no match; Cursor product.json also has no `emergencyAlert` key)

### BN-2.5 Unsupported remote OS notice
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When connected to a remote machine whose OS has an old glibc and the person has allowed the connection, the strip shows a warning icon, "You are connected to an OS version that is unsupported by {product}." and a Learn More link, with a close tooltip "Do not show again in v{version}", and closing it keeps it hidden until the next major or minor version.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\workbench\contrib\remote\browser\remoteConnectionHealth.ts (_checkInitialRemoteConnectionHealth, _showVersionedBanner); C bundle (searched for `unsupportedGlibcWarning` and `unsupportedGlibcWarning.banner`, no match; the class that held it now only calls `getRawEnvironment()` in `_checkInitialRemoteConnectionHealth`)

### BN-2.6 Deprecated 32-bit ARM remote server notice
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When connected to a 32-bit ARM Linux remote server, the strip shows a warning icon, "Support for 32-bit ARM remote servers is deprecated and will be removed in a future release of {product}." and a Learn More link, with the same "Do not show again in v{version}" close tooltip and the same hidden-until-next-major-or-minor memory.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\workbench\contrib\remote\browser\remoteConnectionHealth.ts (isArm32Server, _showVersionedBanner); C bundle (searched for `arm32ServerDeprecation`, no match; same class as BN-2.5 with the banner call removed)

### BN-2.7 Welcome notice for web embedders
- Tag: vscode
- Evidence: source
- Boundary: no
- What: In the web build only, a host page can pass a `welcomeBanner` option (message, icon, links) and the strip shows it at startup until the person closes it, after which the profile remembers the close.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\workbench\contrib\welcomeBanner\browser\welcomeBanner.contribution.ts; V\src\vs\workbench\workbench.web.main.ts; V\src\vs\workbench\browser\web.api.ts (IWelcomeBanner); C bundle (searched for `welcome.banner`, `welcomeBanner` and `workbench.banner.welcome.dismissed`, no match)

### BN-2.8 Managed settings update notice
- Tag: vscode
- Evidence: source
- Boundary: yes
- What: newer-than-1.128. When an organization's managed settings need a newer app version, a neutral-look strip with an info icon shows the requirement message and one optional update action, at the lowest priority, and closing it only hides the strip for the session; boundary because the requirement is tied to the chat setting and puts chat in read-only mode.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: chat.disableAIFeatures
- Cursor: missing
- Source: N\src\vs\workbench\services\policies\browser\managedSettingsUpdate.contribution.ts; N\src\vs\workbench\services\policies\browser\managedSettingsUpdateService.ts; C bundle (searched for `managedSettingsUpdate` and `ManagedSettingsUpdate`, no match)

## Handed off

- The Workspace Trust editor that the Manage link in BN-2.1 opens (`workbench.trust.manage`) - DL-5
- The confirm dialog shown before the unsupported remote OS notice (BN-2.5), with its "Allow", "Learn More" and "Do not show again" choices. Cursor keeps this dialog (messages 8946 to 8949) - DL-1
- The macOS end-of-life and run-as-root warnings in the same window code. They use the notification service, not the strip - NT-2

## Unclear

- Whether BN-2.7 belongs in the catalog at all. It is web only and a host page drives it, so no desktop person can reach it. It is listed because the catalog counts five stock callers.
- If a remote is both old-glibc and 32-bit ARM, the second notice replaces the first at once, so only the ARM notice would show. Read from the code, not run.
- Whether any path other than a team admin setting turns Workspace Trust on in Cursor. I saw one (`workspaceTrustControls.enabled` writes the setting), so BN-2.1 may show for managed teams. Not run.
