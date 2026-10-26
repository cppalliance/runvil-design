Section: Banner (BN). Date: 2026-10-25. Pass: 1.

# Banner in Cursor 3.27.12 (C)

Short answer: the banner strip in Cursor is stock VS Code 1.128.0 code with no visual or behavior changes. Cursor does not use it for updates, announcements, outages, upgrade or plan messages. Only two stock notices still use it, and one of those is off by default. Cursor's own notices live in other surfaces, listed below.

All proof is in the main bundle `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app\out\vs\workbench\workbench.desktop.main.js` (found 41,558,918 bytes on disk, not the 39.6 MB in the brief), its CSS file `workbench.desktop.main.css`, and the message table `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app\out\nls.messages.json`. Cursor stores UI text as numbered message ids (`k(12133,null)`), so searching the bundle for a label finds nothing. I looked labels up in `nls.messages.json` by id. Variable names below (`$Ea`, `uut`) are minified names and will change per build.

## How I searched

- Part and service: searched `workbench.parts.banner`, `bannerService`, `bannerFocused`, `Close Banner`.
- Every caller: the service id is `uut=gn("bannerService")`. I searched every use of `uut` (9 hits: the declaration, the part registration, three keybinding handlers, and four classes that take the service) and every `bannerService.show` and `bannerService.hide` (2 `show` calls and 1 `hide` call). That is the full caller list.
- Outage, announcement, update, upgrade: searched the JS and CSS for `emergencyAlert`, `emergencyAlertUrl`, `outage`, `status.cursor.com`, `announcement`, `upgrade`, `promo`, `trial`, `incident`, `maintenance`, and every CSS class containing `banner`.
- Other bundles: `workbench.anysphere-ui-automations.js` and `workbench.xai-commerce-prepaid.js` have no match for `bannerService`, `outageAlert` or `OutageAlert`. I did not read `workbench.glass.main.js` (Agents window, out of scope).
- Built-in extensions: no "banner" in `package.json` or `package.nls.json` of git, markdown-language-features, merge-conflict, simple-browser, search-result, references-view, cursor-explorer, cursor-commits, cursor-checkout, cursor-worktree-textmate. No theme file in the extensions folder overrides `banner.background`, `banner.foreground` or `banner.iconForeground`. `product.json` has no `emergencyAlertUrl` and no `configurationDefaults`.

## The strip itself (matches V)

- Class `$Ea` is the banner part, id `workbench.parts.banner`, `hasTitle:false`, height 26, registered as the eager service (`cn(uut,$Ea,0)`). Fragment: `super("workbench.parts.banner",{hasTitle:!1},...),...this.height=26`.
- It builds the same four pieces: `div.icon-container`, `div.message-container`, `div.message-actions-container`, `div.action-container`. Close button is `banner.close`, label default "Close Banner" (a plain string in the bundle, V uses a localized string with the same text).
- It is the older single-item version: `show` replaces the current item, `hide(id)` only hides if the id matches. No priority queue and no `neutral` look. Those two are newer-than-1.128 in V (see the V file).
- Same three keybinding rules, weight 200: `workbench.banner.focusBanner` (Escape), `workbench.banner.focusNextAction` (Right, Down), `workbench.banner.focusPreviousAction` (Left, Up), all `when` `bannerFocused`.
- Same command: `workbench.action.focusBanner`, label "Focus Banner" (message id 3320), category View, shown in the Command Palette.
- Same context key: `bannerFocused`, text "Whether the banner has keyboard focus" (id 4380).
- Same layout: banner is a grid leaf next to the title bar, created hidden (`visible:!1`), role `banner`, class `banner`. `shouldShowBannerFirst()` is `isWeb && !isWCOEnabled()`, so on desktop it sits under the title bar.
- Same colors: message table has the three `banner.*` color descriptions (same text as V). No theme override.
- CSS: `workbench.desktop.main.css` has the same 12 `.monaco-workbench .part.banner ...` rules as V, no extra rules. Strip: `display:flex; height:100%; font-size:12px`, with background and text color taken from the `banner-background` and `banner-foreground` theme variables. Message: `line-height:26px`, one line, ellipsis. Links: underlined, 12px left margin, 3px padding. Icon: 10px left, 6px right padding. Close: 6px left, 10px right padding. No `neutral` rule, no Cursor-only banner rule for the part.

## Proposed minor groups

### BN-1 The banner strip

- Name: the notice strip under the title bar.
- One sentence: same single 26 px line as stock VS Code, with an icon, message, links and a close button.
- Tag: both.
- Evidence: bundle (strings, class, CSS), and source in V.
- Boundary: no.
- Examples: strip layout; show and hide by id; Focus Banner command and arrow keys; banner theme colors.
- Proof: main bundle fragments `workbench.parts.banner`, `workbench.action.focusBanner`, `workbench.banner.focusNextAction`, `bannerFocused`; `workbench.desktop.main.css` rules `.monaco-workbench .part.banner`.

### BN-2 Notices that use the strip

- Name: the warnings that still use the strip.
- One sentence: Cursor keeps two of the five stock callers; everything else that VS Code puts in the strip is gone, and Cursor adds none.
- Tag: both (partial, per-feature below).
- Evidence: bundle.
- Boundary: no.
- Per-feature status in Cursor:
  - BN-2.1 Restricted Mode notice: present, tag both. Same id `workbench.banner.restrictedMode`, dismissed key `workbench.banner.restrictedMode.dismissed`, links "Manage" (message 11868, runs `workbench.trust.manage`) and "Learn More" (11869), same three window, folder, workspace messages (11873-11875), same setting `security.workspace.trust.banner` default `untilDismissed` (messages 11825-11828). Shown only when workspace trust is on. Cursor changes the default: `security.workspace.trust.enabled` defaults to false here (true in V), so a fresh Cursor install does not show this banner. It turns on if the user enables trust, or if a team admin turns on `workspaceTrustControls.enabled` (the bundle writes the setting from team admin settings). Fragments: `[h0r]:{type:"boolean",default:!1,...}`, `a?.workspaceTrustControls?.enabled??!1`.
  - BN-2.2 Installation folder warning: present, tag both. Same id `appRootWarning.banner`, warning icon, text message id 12133 ("Files you store within the installation folder ('{0}') may be OVERWRITTEN or DELETED IRREVERSIBLY without warning at update time."). Fragment: `this.bannerService.show({id:"appRootWarning.banner",message:k(12133,null,...),icon:Oe.warning})`.
  - BN-2.3 Emergency alert: absent. No `emergencyAlert` string, no `emergencyAlertUrl`, none in `product.json`. Tag in V: vscode.
  - BN-2.4 Remote server warnings (unsupported OS, 32-bit ARM): banner code removed. The class still exists (`Y7a`, it takes the banner service) and still has the "unsupported OS" confirmation dialog (messages 8946-8949), but `_checkInitialRemoteConnectionHealth` now only calls `getRawEnvironment()` and never shows a banner. No `unsupportedGlibcWarning` or `arm32ServerDeprecation` strings anywhere. Tag in V: vscode.
  - BN-2.5 Welcome banner: absent. No `welcomeBanner` or `welcome.banner` string. In V it is web-only (`workbench.web.main.ts`), and Cursor's desktop bundle does not include it. Tag in V: vscode (web only).

Recommendation: keep BN-2 as a group but mark it "mostly V only" so the catalog does not promise features Cursor lacks. Confidence: high - the two remaining callers and three missing ones are all checked by string search.

## Cursor additions

None that use the banner strip. The service has exactly two callers and both are stock.

Cursor-only things I found that look like banners, with where they actually live:

- Outage and incident alert service. Registered as `desktopOutageAlertService` in the main bundle. It polls `https://status.cursor.com/api/v2/summary.json` (every 15 minutes), watches the components "IDE", "Cloud Agents" and "cursor.com", gives each incident a severity of minor or major, and can also read a remote config named `portal_outage_alert` (enabled, title, description). It is gated by the feature flag `client_outage_banners`, which defaults to off. The service has `snapshot`, `dismiss(identity)` and `openStatusPage()`. Dismissals are stored under `cursor/desktopOutageAlert.dismissals.v1`, and there is a migration from an older key `cursor/glass.outageAlertDismissedIdentity.v1` (the old key name says Glass, the Agents window). I found no code in the main bundle that reads this service (the service id has only 2 hits, the declaration and the registration), no `bannerService` call from it, and no CSS class with "outage" in the main CSS. So I could not see how or where it shows. Not cataloged as a group. See "could not verify".
- Announcement popup (`.announcement-modal-container`). A floating card pinned bottom left, 342 px wide, fixed position, with an eyebrow, title, image, section list and primary and secondary buttons. It is a product announcement surface, not the strip. Belongs to Notifications or Dialogs. Proof: CSS rule `.announcement-modal-container{...position:fixed;left:8px;bottom:8px;width:342px...}` in `workbench.desktop.main.css`.
- Update control: the title-bar Update entry is the update surface. It is in the Title bar section. No update message goes through the banner service.

## Left out and why

- `.usage-limit-policy-banner` (text such as "Upgrade for extended usage & faster responses", plan-based). It is an upgrade and usage message inside the chat composer area (it reads composer state and auto-switches the model). Agent and chat, out of scope.
- `.composer-input-edit-banner*`. Chat input. Out of scope.
- `.cursor-blame-large-commit-banner` ("Large commit (N files). Diffs load when expanded."). Lives in the review panel markup (`smart-review-panel`). In scope as a Cursor feature but it is not this part. Belongs to the Review Changes or Editor work.
- `.cursor-settings-inline-banner*` and `.cursor-settings-pane-content-inner-banner`. Class names show these are banners inside the Cursor Settings pane. Not this part. I did not open that pane's code.
- Skill migration banner. Commands `workbench.action.customize.forceSkillMigrationBanner` and `...clearForcedSkillMigrationBanner` (Developer category, Command Palette) exist for the Customize editor. The message says it stays hidden until an Agent Store is mounted. It is a banner inside the Customize area, not the strip. I did not find where it renders.
- `.editor-banner` (strip inside a code editor, for the Unicode highlighter). Same as V. Belongs to the Editor section.
- macOS end-of-life warning in the window code uses the notification service, not the banner. Cursor lists "macOS Catalina" (major version "19", message id 12134) where V lists "macOS Big Sur" ("20"). That difference belongs to the Notifications section.

## Newer than 1.128

Nothing in Cursor. Cursor is at 1.128.0 and its banner code is the single-item version. The newer priority queue, `neutral` look and managed-settings banner are only in N (VS Code 1.144.0), described in the V file.

## V items missing from Cursor

- Emergency alert banner (BN-2.3).
- Remote server warning banners, unsupported OS and 32-bit ARM (BN-2.4). The dialog stays, the banner is gone.
- Welcome banner (BN-2.5, web only in V anyway).
- Restricted Mode banner is present in code but does not show by default (trust defaults to off).

## Could not verify

- Where and whether the Cursor outage alert renders. It likely renders in the Agents window bundle, which I was told not to read. I cannot say whether the main IDE window ever shows it. To settle it: read `workbench.glass.main.js` for `desktopOutageAlertService`, or run Cursor with `client_outage_banners` on.
- What the strip looks like on screen. I read CSS and code, I did not run Cursor.
- Where the skill migration banner and the Cursor Settings inline banners render. Not read beyond class names and command ids.
- Whether team admin settings are the only route that flips workspace trust on by default. I saw one code path that does it. There could be others.
