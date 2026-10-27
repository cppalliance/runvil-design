# 2. Banner (BN)

A one-line message strip under the title bar. It is small, so there are two groups.

2 groups, 18 features. Each feature lists its entry points and the pieces that finish it.

## BN-1 Notice strip under the title bar

(both). A single 26 px line with an icon, message, optional links and a close button, shown only when a feature asks. Cursor's code and CSS match stock. Examples: strip layout; show and hide by id; the Focus Banner command; `banner.*` theme colors.

10 features.

#### BN-1.1 Strip layout and content

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A 26 px tall strip with 12 px text that shows, left to right, an icon (a codicon, an image URL, or the default product icon), a one-line message (plain text or markdown, cut off with an ellipsis), a row of underlined action links, and a small close button; the part has role `banner` and screen readers get the notice's aria label (or the message text) instead of the visible pieces.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\banner\bannerPart.ts; V\src\vs\workbench\browser\parts\banner\media\bannerpart.css; V\src\vs\workbench\services\banner\browser\bannerService.ts; V\src\vs\workbench\browser\workbench.ts; C bundle (module: banner part class, id `workbench.parts.banner`, its `show` method, and the 12 `.monaco-workbench .part.banner` rules in workbench.desktop.main.css, same as V)
- Pieces:
  1. When a notice is showing, a 26 px tall strip appears with 12 px text, holding from left to right an icon, a one-line message, a row of action links, and a small close button.
  2. The icon is a codicon, an image from a web address, or the default product icon when the notice gives neither.
  3. A message given as plain text shows as is, and a message given as markdown shows with its formatting and with any links in it underlined.
  4. A message too long for the strip stays on one line and is cut off with an ellipsis instead of wrapping.
  5. The action links show as underlined text after the message, and the close button stays at the right end of the strip.
  6. A screen reader treats the strip as a `banner` region and reads the notice's aria label, or the message text when there is no aria label, instead of reading the icon, message, and links separately.

#### BN-1.2 Show and hide a notice by id

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A feature asks the strip to show a notice with an id; only one notice is held at a time, a notice with a different id replaces the current one, showing the same id again only makes the strip visible again, and hiding by id works only when that id is the notice on screen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\banner\browser\bannerService.ts; V\src\vs\workbench\browser\parts\banner\bannerPart.ts; C bundle (module: banner part class, `show(e){if(e.id===this.item?.id)`)
- Pieces:
  1. Showing a notice with a new id fills the strip with that notice's icon, message, links, and close button and makes the strip visible.
  2. Showing a notice with a different id while another is on screen replaces it, so only one notice is ever visible.
  3. Showing the notice that is already held makes the strip visible again without rebuilding its contents.
  4. Hiding by id hides the strip only when that id is the notice on screen, and hiding any other id does nothing.

#### BN-1.3 Strip makes room only while shown

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - same hide and show logic, but the offset code adds a check for Cursor's no-title-bar layout (title bar height counts as 0 and the quick input top is not set to 6 px)
- What: The strip starts hidden at zero height and, when a notice shows, pushes the whole workbench down by 26 px and also moves overlays like the quick input down; on desktop it sits directly under the title bar and on the web it goes above the title bar unless window controls overlay is on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.commandCenter
- Source: V\src\vs\workbench\browser\layout.ts (computeContainerOffset, shouldShowBannerFirst, setBannerHidden, initial grid with visible false); V\src\vs\workbench\browser\parts\banner\bannerPart.ts (minimumHeight, maximumHeight, setVisibility); C bundle (module: workbench layout class, `computeContainerOffset`, `shouldShowBannerFirst`, `setBannerHidden`, `isNoTitlebarLayoutActive`)
- Pieces:
  1. Before any notice shows, the strip has zero height and takes no space under the title bar.
  2. When a notice shows, the whole workbench moves down by 26 px to make room for the strip.
  3. When the notice is hidden or closed, the workbench moves back up and the 26 px is given back.
  4. On desktop the strip sits directly under the title bar.
  5. In the web build the strip sits above the title bar, except when window controls overlay is on.
  6. With the title bar visible and `window.commandCenter` not set to false, the quick input opens over the title bar and strip 6 px from the top, and otherwise it opens below the strip and the title bar.
  7. In Cursor's no-title-bar layout the title bar counts as zero height when the overlays are placed, and the quick input top is not set to 6 px. [cursor only]

#### BN-1.4 Close a notice

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The person clicks the small X at the right end (or focuses it and presses Enter or Space) and the strip hides, clears itself, and runs the notice's close callback so its owner can remember the dismissal; the tooltip and label read "Close Banner" unless the notice sets its own text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\banner\bannerPart.ts (close, the `banner.close` action built in show); V\src\vs\workbench\services\banner\browser\bannerService.ts (onClose, closeLabel); C bundle (module: banner part class, `close(e){this.setVisibility(!1)`, `e.closeLabel??"Close Banner"`)
- Pieces:
  1. Clicking the small X at the right end of the strip hides the strip and removes the notice's contents.
  2. Focusing the X and pressing Enter or Space closes the notice the same way as a click.
  3. Closing runs the notice's own close callback, so an owner that remembers dismissals can keep the notice from coming back.
  4. The close button's tooltip and label read "Close Banner" unless the notice sets its own close text, which then shows instead.
  5. Showing the same notice again after it was closed rebuilds it in full, because closing makes the strip forget it.

#### BN-1.5 Use a link in the notice

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Each action link opens through the opener service when clicked or when Enter or Space is pressed on it, so a web address opens in the browser and a `command:` link runs a command; links are underlined, act as buttons, and are skipped by Tab until the person moves in with the keyboard (BN-1.7).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\platform\opener\browser\link.ts; V\src\vs\workbench\browser\parts\banner\bannerPart.ts (links built with tabIndex -1); V\src\vs\workbench\browser\parts\banner\media\bannerpart.css (`.message-actions-container a`); C bundle (module: banner part class `show`, links built with `tabIndex:-1`)
- Pieces:
  1. Clicking an action link opens its target through the opener, so a web address opens in the browser.
  2. Clicking a link whose target is a `command:` address runs that command.
  3. Pressing Enter or Space on a focused link does the same as clicking it.
  4. Action links are underlined and are exposed to screen readers as buttons.
  5. Pressing Tab does not stop on the action links, and they get focus only when the person moves in with the arrow keys.

#### BN-1.6 Focus the strip with Focus Banner

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The person runs Focus Banner from the Command Palette (View category, no default key) to put keyboard focus on the strip; the strip is tabbable, and the `bannerFocused` context key is true while focus is inside it.
- Commands: workbench.action.focusBanner
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\banner\bannerPart.ts (FocusBannerAction, createContentArea); V\src\vs\workbench\browser\layout.ts (focusPart default case); V\src\vs\workbench\common\contextkeys.ts (BannerFocused); C bundle (module: banner part file, `workbench.action.focusBanner`, label "Focus Banner" via message 3320, `category:Yn.View,f1:!0`, context key `bannerFocused` via message 4380)
- Pieces:
  1. Running Focus Banner from the Command Palette puts keyboard focus on the strip.
  2. The Command Palette lists the command as `View: Focus Banner` (`workbench.action.focusBanner`) and it has no default keybinding.
  3. The strip is a tab stop, so pressing Tab can land on the strip itself when a notice is showing.
  4. While focus is inside the strip the `bannerFocused` context key is true, so keybindings that depend on it apply only there.

#### BN-1.7 Move around inside the strip by keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While the strip has focus, Right or Down moves to the next action (each link, then the close button, then back to the first link), Left or Up moves back the other way, and Escape returns focus to the strip itself.
- Commands: workbench.banner.focusBanner; workbench.banner.focusNextAction; workbench.banner.focusPreviousAction
- Menus: none found
- Keys: Escape (bannerFocused); Right Arrow, Down Arrow (bannerFocused); Left Arrow, Up Arrow (bannerFocused)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\banner\bannerPart.ts (three registerCommandAndKeybindingRule blocks, focusNextAction, focusPreviousAction, focusActionLink); C bundle (module: banner part file, ids `workbench.banner.focusBanner`, `workbench.banner.focusNextAction`, `workbench.banner.focusPreviousAction`, weight 200, same keys)
- Pieces:
  1. With focus in the strip, Right Arrow or Down Arrow (`workbench.banner.focusNextAction`) moves focus to the next action, going through each link, then the close button, then back to the first link.
  2. With focus in the strip, Left Arrow or Up Arrow (`workbench.banner.focusPreviousAction`) moves focus the other way, and from the first link it wraps to the close button.
  3. Pressing Escape (`workbench.banner.focusBanner`) moves focus from an action back to the strip itself.
  4. When the person tabs out of the strip and back in, focus returns to the action they were last on instead of the strip itself.

#### BN-1.8 Theme the strip colors

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Three theme colors control the strip - `banner.background`, `banner.foreground`, and `banner.iconForeground` - with defaults taken from the list selection colors (the light theme darkens the background by 0.3) and the editor info color, and a person can override them in a color customization.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Source: V\src\vs\workbench\common\theme.ts; V\src\vs\workbench\browser\parts\banner\media\bannerpart.css; C bundle (module: workbench theme colors, `Ft("banner.background",...)`, `banner.foreground`, `banner.iconForeground`, messages 4456 to 4458)
- Pieces:
  1. `banner.background` sets the strip's background, defaulting to the list active selection background, darkened by 0.3 in light themes.
  2. `banner.foreground` sets the color of the strip's text, links, and close button, defaulting to the list active selection foreground.
  3. `banner.iconForeground` sets the color of a codicon icon in the strip, defaulting to the editor info color.
  4. Putting any of the three ids in `workbench.colorCustomizations` overrides the theme's value for that color.

#### BN-1.9 Queue notices by priority

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: newer-than-1.128. Several notices can wait at once, the one with the highest `priority` shows (ties go to the most recently shown), hiding or closing the current notice reveals the next, and focus stays in the strip if it was there when the notice swapped.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: N\src\vs\workbench\services\banner\browser\bannerService.ts; N\src\vs\workbench\browser\parts\banner\bannerPart.ts (items map, renderActiveItem); C bundle (searched for `renderActiveItem`, no match, and the banner part still holds a single `item`)
- Pieces:
  1. In VS Code 1.144 and later, when several notices are waiting, the strip shows the one with the highest `priority`, and a tie goes to the notice shown most recently. [stock only]
  2. Hiding or closing the notice on screen reveals the next waiting notice instead of leaving the strip empty. [stock only]
  3. If keyboard focus is inside the strip when the notice swaps, focus stays in the strip. [stock only]

#### BN-1.10 Neutral look for low-key notices

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: newer-than-1.128. A notice can ask for a neutral look, which gives the strip the side bar background, the normal foreground color, and a bottom border in the contrast border color instead of the blue selection colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: N\src\vs\workbench\browser\parts\banner\media\bannerpart.css (`.part.banner.neutral` rules); N\src\vs\workbench\services\banner\browser\bannerService.ts (neutral); C bundle (searched workbench.desktop.main.css for `part.banner.neutral`, no match)
- Pieces:
  1. In VS Code 1.144 and later, a notice that asks for a neutral look shows the strip with the side bar background and the normal foreground color instead of the blue selection colors. [stock only]
  2. A neutral strip draws a bottom border in the contrast border color. [stock only]
  3. In a neutral strip the icon, action links, and close button use the normal foreground color. [stock only]


## BN-2 Notices that use the strip

(both, partial). Five stock callers use it and Cursor keeps two. Examples: the Restricted Mode notice; the installation folder warning; the emergency alert (stock only); remote server warnings (stock only).

8 features.

#### BN-2.1 Restricted Mode notice

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the code, ids, text and links are the same, but Cursor ships `security.workspace.trust.enabled` off by default, so a fresh install never shows it
- What: When Workspace Trust is on and the window is not trusted, the strip shows a shield icon and "Restricted Mode is intended for safe code browsing. Trust this window (or folder, or workspace) to enable all features." with the links Manage (runs `workbench.trust.manage`) and Learn More, and it hides again once trust is granted.
- Commands: workbench.trust.manage
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.enabled; security.workspace.trust.startupPrompt
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (updateWorkbenchIndicators, showModalOnStart, getBannerItem, getBannerItemMessages, getBannerItemAriaLabels); V\src\vs\workbench\contrib\workspace\browser\workspaceTrustEditor.ts (shieldIcon, icon id `workspace-trust-banner`); V\src\vs\workbench\contrib\workspace\common\workspace.ts (MANAGE_TRUST_COMMAND_ID); C bundle (module: workspace trust startup contribution, ids `workbench.banner.restrictedMode`, `workbench.trust.manage`, icon `workspace-trust-banner`, messages 11868 to 11875, `[h0r]:{type:"boolean",default:!1`)
- Pieces:
  1. When Workspace Trust is on and the window is not trusted, the strip shows a shield icon and the message "Restricted Mode is intended for safe code browsing. Trust this window to enable all features."
  2. The message says "folder" or "workspace" in place of "window" when a folder or a workspace is open, and the screen reader label adds "Use navigation keys to access banner actions."
  3. The Manage link runs `workbench.trust.manage`, which opens the Workspace Trust editor.
  4. The Learn More link opens the Workspace Trust help page in the browser.
  5. The strip hides again as soon as the window becomes trusted.
  6. Setting `security.workspace.trust.enabled` turns Workspace Trust on or off, and the notice can only show while it is on.
  7. Cursor ships `security.workspace.trust.enabled` off by default, so a fresh install never shows this notice until the setting is turned on. [cursor only]

#### BN-2.2 Choose when the Restricted Mode notice appears

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A setting picks whether the notice shows every time an untrusted workspace opens, shows until the person closes it once for that workspace (the default), or never shows, and closing it stores a per-workspace dismissed flag that only counts in the "until dismissed" mode.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.banner
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (getBannerItem onClose, bannerSetting, the registered setting with always, untilDismissed, never); C bundle (module: workspace trust startup contribution and configuration, `security.workspace.trust.banner` default `untilDismissed`, messages 11825 to 11828, `onClose:()=>{e&&this.storageService.store(cgg,!0,1,1)}`)
- Pieces:
  1. Setting `security.workspace.trust.banner` to `always` shows the notice every time an untrusted workspace opens.
  2. Setting it to `untilDismissed`, the default, shows the notice until the person closes it once for that workspace, and it stays away for that workspace afterward.
  3. Setting it to `never` keeps the notice from ever showing.
  4. Closing the notice stores a dismissed flag for that workspace only, and the flag only matters in `untilDismissed` mode.

#### BN-2.3 Installation folder warning

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: After the window has restored, if an open folder is the app's install folder or sits inside it (built app, not an extension development window), the strip shows a warning icon and "Files you store within the installation folder ('{path}') may be OVERWRITTEN or DELETED IRREVERSIBLY without warning at update time." with no links, and nothing remembers a close.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\window.ts (handleWarnings, installation dir block); C bundle (module: native window class, `this.bannerService.show({id:"appRootWarning.banner",message:k(12133,null,...),icon:Oe.warning})`, message 12133)
- Pieces:
  1. After the window has restored, if an open folder is the app's install folder or sits inside it, the strip shows a warning icon and a warning message.
  2. The message reads "Files you store within the installation folder ('{path}') may be OVERWRITTEN or DELETED IRREVERSIBLY without warning at update time." with the folder's path filled in.
  3. The notice has no action links, only the close button.
  4. The notice shows only in an installed (built) app, and never in an extension development window or a source checkout run.
  5. Nothing remembers a close, so the notice shows again the next time such a folder is opened.

#### BN-2.4 Emergency alert

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: A desktop window fetches a JSON list from the `emergencyAlertUrl` in product.json at startup and every hour, shows the first alert that matches this build's commit, platform and architecture as a warning-icon notice with optional links, hides it when no alert matches, and does nothing when the product has no URL.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\emergencyAlert\electron-browser\emergencyAlert.contribution.ts; V\src\vs\workbench\workbench.desktop.main.ts; V\src\vs\base\common\product.ts; C bundle (searched for `emergencyAlertUrl` and `emergencyAlert.banner`, no match; Cursor product.json also has no `emergencyAlert` key)
- Pieces:
  1. When the product has an emergency alert address, a desktop window fetches the alert list from it at startup and again every hour. [stock only]
  2. The first alert in the list that matches this build's commit, platform, and architecture shows as a warning-icon notice with the alert's message. [stock only]
  3. An alert's optional links show as action links in the strip. [stock only]
  4. When no alert matches, or the list cannot be read, an alert notice already on screen is hidden. [stock only]
  5. When the product has no emergency alert address, nothing is fetched and no notice shows. [stock only]

#### BN-2.5 Unsupported remote OS notice

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: When connected to a remote machine whose OS has an old glibc and the person has allowed the connection, the strip shows a warning icon, "You are connected to an OS version that is unsupported by {product}." and a Learn More link, with a close tooltip "Do not show again in v{version}", and closing it keeps it hidden until the next major or minor version.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\remoteConnectionHealth.ts (_checkInitialRemoteConnectionHealth, _showVersionedBanner); C bundle (searched for `unsupportedGlibcWarning` and `unsupportedGlibcWarning.banner`, no match; the class that held it now only calls `getRawEnvironment()` in `_checkInitialRemoteConnectionHealth`)
- Pieces:
  1. When connected to a remote machine whose OS has an old glibc and the person has allowed the connection, the strip shows a warning icon and "You are connected to an OS version that is unsupported by {product}." [stock only]
  2. The notice has a Learn More link. [stock only]
  3. The close button's tooltip reads "Do not show again in v{version}". [stock only]
  4. Closing the notice keeps it hidden until the next major or minor version, and a patch update does not bring it back. [stock only]

#### BN-2.6 Deprecated 32-bit ARM remote server notice

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: When connected to a 32-bit ARM Linux remote server, the strip shows a warning icon, "Support for 32-bit ARM remote servers is deprecated and will be removed in a future release of {product}." and a Learn More link, with the same "Do not show again in v{version}" close tooltip and the same hidden-until-next-major-or-minor memory.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\remoteConnectionHealth.ts (isArm32Server, _showVersionedBanner); C bundle (searched for `arm32ServerDeprecation`, no match; same class as BN-2.5 with the banner call removed)
- Pieces:
  1. When connected to a 32-bit ARM Linux remote server, the strip shows a warning icon and "Support for 32-bit ARM remote servers is deprecated and will be removed in a future release of {product}." [stock only]
  2. The notice has a Learn More link. [stock only]
  3. The close button's tooltip reads "Do not show again in v{version}". [stock only]
  4. Closing the notice keeps it hidden until the next major or minor version, and a patch update does not bring it back. [stock only]

#### BN-2.7 Welcome notice for web embedders

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: In the web build only, a host page can pass a `welcomeBanner` option (message, icon, links) and the strip shows it at startup until the person closes it, after which the profile remembers the close.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\welcomeBanner\browser\welcomeBanner.contribution.ts; V\src\vs\workbench\workbench.web.main.ts; V\src\vs\workbench\browser\web.api.ts (IWelcomeBanner); C bundle (searched for `welcome.banner`, `welcomeBanner` and `workbench.banner.welcome.dismissed`, no match)
- Pieces:
  1. In the web build, when the host page passes a `welcomeBanner` option, the strip shows its message, icon, and links at startup. [stock only]
  2. The icon is either a codicon name or an image address. [stock only]
  3. Closing the notice is remembered in the profile, so it does not show again on later startups. [stock only]
  4. A host page that passes no `welcomeBanner` option shows no notice, and the desktop app never shows it. [stock only]

#### BN-2.8 Managed settings update notice

- Tag: vscode. Evidence: source. Boundary: yes. Cursor: missing
- What: newer-than-1.128. When an organization's managed settings need a newer app version, a neutral-look strip with an info icon shows the requirement message and one optional update action, at the lowest priority, and closing it only hides the strip for the session; boundary because the requirement is tied to the chat setting and puts chat in read-only mode.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: chat.disableAIFeatures
- Source: N\src\vs\workbench\services\policies\browser\managedSettingsUpdate.contribution.ts; N\src\vs\workbench\services\policies\browser\managedSettingsUpdateService.ts; C bundle (searched for `managedSettingsUpdate` and `ManagedSettingsUpdate`, no match)
- Pieces:
  1. In VS Code 1.144 and later, when an organization's managed settings need a newer app version, the strip shows a neutral-look notice with an info icon and the requirement message. [stock only]
  2. When the update has a status line, it follows the requirement message in the strip. [stock only]
  3. The notice has one action link, the update action, when one is available, and no links otherwise. [stock only]
  4. The notice has the lowest priority, so any other waiting notice shows ahead of it. [stock only]
  5. Closing the notice hides it for the rest of the session only, and it does not change the requirement. [stock only]
  6. The notice does not show while `chat.disableAIFeatures` is on. [stock only]
  7. A screen reader announces the requirement message when the notice first appears or its message changes. [stock only]
