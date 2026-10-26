Section: Banner (BN). Date: 2026-10-25. Pass: 1.

# Banner in stock VS Code 1.128.0 (V)

The banner is a one-line message strip that sits under the title bar. It is empty and hidden most of the time. A few built-in features put a notice in it, such as a Restricted Mode warning. It shows one notice at a time.

Plain truth about size: this section is small. It is one UI part plus a short list of things that use it. I propose two groups, not more. Splitting further would just cut a single strip into pieces.

## What I read in V

All paths are under `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0\src\vs\workbench` unless they start with `extensions`.

- `browser\parts\banner\bannerPart.ts` (303 lines): the part, the service implementation, three keybinding rules, and the Focus Banner command.
- `browser\parts\banner\media\bannerpart.css`: the styles.
- `services\banner\browser\bannerService.ts`: the `IBannerService` interface and the `IBannerItem` shape (id, icon, message, actions, ariaLabel, onClose, closeLabel).
- `contrib\welcomeBanner\browser\welcomeBanner.contribution.ts`: the welcome banner.
- Every caller of the service. I searched all of `src` and `extensions` for `IBannerService` and `bannerService.show`. Real callers, found in these files only:
  - `contrib\workspace\browser\workspace.contribution.ts` (Restricted Mode)
  - `electron-browser\window.ts` (installation folder warning)
  - `contrib\emergencyAlert\electron-browser\emergencyAlert.contribution.ts` (emergency alert)
  - `contrib\remote\browser\remoteConnectionHealth.ts` (two remote server warnings)
  - `contrib\welcomeBanner\browser\welcomeBanner.contribution.ts` (web only)
  - `browser\layout.ts` only asks for the service so it is created early (line 333). It does not show anything.
  - Two hits in `extensions\copilot\src\extension\tools\test\node\applyPatch\corpus\*.patch` are test text that mention the word. They are not callers.
- Layout, theme and context keys: `browser\layout.ts`, `browser\workbench.ts:348`, `common\theme.ts:269-280`, `common\contextkeys.ts:141-143`, `services\layout\browser\layoutService.ts:23`.
- Built-in extensions: none. I searched `package.json` and `package.nls.json` of the six in-scope extensions (git, markdown-language-features, merge-conflict, simple-browser, search-result, references-view) for "banner" and found nothing. Extensions have no API to show this banner.

## How the part works

- Class `BannerPart` (`bannerPart.ts:35`) is registered as the `IBannerService` singleton, created eagerly (line 242).
- Fixed height 26 px. Height is 0 when nothing is shown (`bannerPart.ts:41`, `minimumHeight` and `maximumHeight` return 0 when hidden).
- It starts hidden in the layout grid (`browser\layout.ts:2638-2640`, `visible: false`).
- Position: on desktop it sits directly under the title bar. On web, when window controls overlay is off, it goes above the title bar (`layout.ts:1372`, `shouldShowBannerFirst` is `isWeb && !isWCOEnabled()`).
- When shown, overlays like the quick input are pushed down by the banner height (`layout.ts:238-260`, `computeContainerOffset`). With the command center visible the quick input goes over both the title bar and the banner.
- The part element gets role `banner` and class `banner` (`browser\workbench.ts:348`).
- `show(item)` builds four pieces left to right (`bannerPart.ts:181-232`): `div.icon-container`, `div.message-container`, `div.message-actions-container`, `div.action-container`.
  - Icon: a codicon, or an image URL, or the default product icon (class `custom-icon`).
  - Message: plain text, or markdown rendered by the markdown renderer. Single line, cut off with an ellipsis.
  - Action links: each is a link built from an `ILinkDescriptor` (label plus href, so it can open a URL or run a `command:` link).
  - Close: a small X button (`banner.close`). The tooltip and label default to "Close Banner" and can be replaced per item with `closeLabel`.
- Only one item at a time in 1.128. A second `show` with a different id replaces the first. `show` with the same id just makes it visible again. `hide(id)` only works if that id is the current one (`bannerPart.ts:173-179`).
- Closing hides the strip, clears it, and calls the item's `onClose` so the caller can remember the dismissal (`bannerPart.ts:94-107`).

## Proposed minor groups

### BN-1 The banner strip

- Name in plain words: the notice strip under the title bar.
- One sentence: a single 26 px line with an icon, a message, optional links, and a close button, shown only when some feature asks for it.
- Tag: both (checked, the Cursor bundle has the same class, same ids, same CSS).
- Evidence: source.
- Boundary: no.
- Example features (up to 4):
  - BN-1.1 Strip layout: icon, one-line message (text or markdown), action links, close button. Proof: `browser\parts\banner\bannerPart.ts:181-232`, `browser\parts\banner\media\bannerpart.css`.
  - BN-1.2 Show and hide by id through the service: one notice at a time, hidden strip takes no space. Proof: `services\banner\browser\bannerService.ts`, `bannerPart.ts:144-179`, `browser\layout.ts:1868`.
  - BN-1.3 Keyboard use: Focus Banner command (`workbench.action.focusBanner`, in the Command Palette under the View category, no default key), then Escape to refocus the strip, Right or Down to the next action, Left or Up to the previous one, only while the banner has focus (context key `bannerFocused`). Proof: `bannerPart.ts:247-303`, `common\contextkeys.ts:143`.
  - BN-1.4 Theme colors: `banner.background`, `banner.foreground`, `banner.iconForeground`. Defaults: dark and high contrast use the list active selection background, light uses that darkened by 0.3, text uses the list active selection foreground, icon uses the editor info color. Proof: `common\theme.ts:271-280`.
- Entry points: the strip has no menu item, no settings of its own, and no default keybinding. Its only command is Focus Banner. I read `bannerPart.ts` end to end for this.

### BN-2 Notices that use the strip

- Name in plain words: the warnings and messages that actually appear in the strip.
- One sentence: a short fixed list of built-in features that call the banner service, mostly safety and environment warnings.
- Tag: both for the group (Cursor keeps two of the five, see the Cursor file). Per-feature tags are below.
- Evidence: source.
- Boundary: no.
- Example features (up to 4 shown in the summary, five exist):
  - BN-2.1 Restricted Mode notice. Text: "Restricted Mode is intended for safe code browsing. Trust this window|folder|workspace to enable all features." Shield icon, links "Manage" (runs `workbench.trust.manage`) and "Learn More". Closing it stores a per-workspace "dismissed" flag, and that flag is honored only when the setting is `untilDismissed`. Setting `security.workspace.trust.banner` with values always, untilDismissed (default), never. Shown only when workspace trust is enabled. Tag: both. Proof: `contrib\workspace\browser\workspace.contribution.ts:432-443, 553-626, 819-831`, `contrib\workspace\browser\workspaceTrustEditor.ts:63`.
  - BN-2.2 Installation folder warning. Text: "Files you store within the installation folder ('{0}') may be OVERWRITTEN or DELETED IRREVERSIBLY without warning at update time." Warning icon, no links, shown when an open folder is inside the app install folder on a built, non-extension-development window. Tag: both. Proof: `electron-browser\window.ts:733-755`.
  - BN-2.3 Emergency alert. Warning icon, message plus optional links, fetched from a URL in `product.json` (`emergencyAlertUrl`), checked once at startup and then every hour, matched on commit, platform and arch. Hides itself when the alert goes away. Does nothing if the product has no URL. Desktop only. Tag: vscode (not found in Cursor). Proof: `contrib\emergencyAlert\electron-browser\emergencyAlert.contribution.ts:34-121`, `workbench.desktop.main.ts:196`, `base\common\product.ts:255`.
  - BN-2.4 Remote server warnings. Two banners, warning icon, each with a "Learn More" link and a close tooltip "Do not show again in v{version}" (dismissal is remembered until the next major or minor version). One for a remote OS that is unsupported (old glibc), one for 32-bit ARM remote servers being deprecated. Tag: vscode (the banner code is removed in Cursor, see the Cursor file). Proof: `contrib\remote\browser\remoteConnectionHealth.ts:86-125, 175-195`.
  - BN-2.5 Welcome banner (not in the four above). Web only, set by the embedder through the web API option `welcomeBanner` (message, icon, actions). Close is remembered in storage key `workbench.banner.welcome.dismissed`. Tag: vscode, web build only. Proof: `contrib\welcomeBanner\browser\welcomeBanner.contribution.ts`, `workbench.web.main.ts:153`, `browser\web.api.ts:348, 647-668`.
- Not in this group: the macOS end-of-life and run-as-root warnings in `electron-browser\window.ts:728-783` use the notification service, not the banner. They belong to the Notifications section.

Recommendation: keep two groups, and let later passes split BN-2 into features. Confidence: high - the banner service has exactly these callers and no more.

## Things I left out

- The editor banner (`div.editor-banner`, from `src\vs\editor\contrib\unicodeHighlighter\browser\bannerController.ts`). It is a strip inside a code editor, not this part. It reuses the `banner.*` colors. It belongs to the Editor section.
- Notifications (toasts, the notification center). They are a different surface. They belong to the Notifications section.
- Extension warnings and anything under `contrib\extensions`. Out of scope per the brief.

## Newer than 1.128 (seen only in N, VS Code main 1.144.0)

Mark: `newer-than-1.128`. I compared the three banner files and the three caller files against V.

- Banner queue with priority. `IBannerItem` gains `priority?: number` (higher shows first, ties go to the most recently shown). The part keeps a map of items instead of one item. Hiding the active item reveals the next one. Proof: `C:\Users\Vinnie\src\vscode\src\vs\workbench\services\banner\browser\bannerService.ts`, `...\browser\parts\banner\bannerPart.ts` (new `items` map and `renderActiveItem`).
- Neutral look. `IBannerItem` gains `neutral?: boolean`. The strip gets class `neutral`: side bar background, normal foreground, a bottom border in the contrast border color. Proof: `...\parts\banner\media\bannerpart.css` (the added `.neutral` rules).
- Focus is restored to the strip when the shown item is swapped while it had focus (`hadFocus` in the new `renderActiveItem`).
- A new banner user: a managed-settings update notice (priority -1, neutral, info icon, one optional action). It is tied to a chat setting and an organization minimum-version rule, so it is agent-adjacent. Proof: `C:\Users\Vinnie\src\vscode\src\vs\workbench\services\policies\browser\managedSettingsUpdate.contribution.ts`.
- Emergency alert, welcome banner and remote health files are identical to V (no diff). `window.ts` and `workspace.contribution.ts` still call the service in N, but I did not diff them.

## Could not verify

- What the strip looks like on screen. I read CSS and code only. I did not run VS Code.
- Real theme colors for each built-in theme. I only found the defaults in `common\theme.ts`. I searched the in-scope extensions and found no theme file that overrides `banner.*`.
- Whether the Restricted Mode banner appears in a default stock install. Trust defaults to on in V (`workspace.contribution.ts:799-801`, default true), so it should, but I did not run it.
