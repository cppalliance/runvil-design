Group: BN-1 Notice strip under the title bar. Section: Banner. Date: 2026-10-25. Pass: 2.
Features: 10.

### BN-1.1 Strip layout and content
- Tag: both
- Evidence: source
- Boundary: no
- What: A 26 px tall strip with 12 px text that shows, left to right, an icon (a codicon, an image URL, or the default product icon), a one-line message (plain text or markdown, cut off with an ellipsis), a row of underlined action links, and a small close button; the part has role `banner` and screen readers get the notice's aria label (or the message text) instead of the visible pieces.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\banner\bannerPart.ts; V\src\vs\workbench\browser\parts\banner\media\bannerpart.css; V\src\vs\workbench\services\banner\browser\bannerService.ts; V\src\vs\workbench\browser\workbench.ts; C bundle (module: banner part class, id `workbench.parts.banner`, its `show` method, and the 12 `.monaco-workbench .part.banner` rules in workbench.desktop.main.css, same as V)

### BN-1.2 Show and hide a notice by id
- Tag: both
- Evidence: source
- Boundary: no
- What: A feature asks the strip to show a notice with an id; only one notice is held at a time, a notice with a different id replaces the current one, showing the same id again only makes the strip visible again, and hiding by id works only when that id is the notice on screen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\banner\browser\bannerService.ts; V\src\vs\workbench\browser\parts\banner\bannerPart.ts; C bundle (module: banner part class, `show(e){if(e.id===this.item?.id)`)

### BN-1.3 Strip makes room only while shown
- Tag: both
- Evidence: source
- Boundary: no
- What: The strip starts hidden at zero height and, when a notice shows, pushes the whole workbench down by 26 px and also moves overlays like the quick input down; on desktop it sits directly under the title bar and on the web it goes above the title bar unless window controls overlay is on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.commandCenter
- Cursor: changed - same hide and show logic, but the offset code adds a check for Cursor's no-title-bar layout (title bar height counts as 0 and the quick input top is not set to 6 px)
- Source: V\src\vs\workbench\browser\layout.ts (computeContainerOffset, shouldShowBannerFirst, setBannerHidden, initial grid with visible false); V\src\vs\workbench\browser\parts\banner\bannerPart.ts (minimumHeight, maximumHeight, setVisibility); C bundle (module: workbench layout class, `computeContainerOffset`, `shouldShowBannerFirst`, `setBannerHidden`, `isNoTitlebarLayoutActive`)

### BN-1.4 Close a notice
- Tag: both
- Evidence: source
- Boundary: no
- What: The person clicks the small X at the right end (or focuses it and presses Enter or Space) and the strip hides, clears itself, and runs the notice's close callback so its owner can remember the dismissal; the tooltip and label read "Close Banner" unless the notice sets its own text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\banner\bannerPart.ts (close, the `banner.close` action built in show); V\src\vs\workbench\services\banner\browser\bannerService.ts (onClose, closeLabel); C bundle (module: banner part class, `close(e){this.setVisibility(!1)`, `e.closeLabel??"Close Banner"`)

### BN-1.5 Use a link in the notice
- Tag: both
- Evidence: source
- Boundary: no
- What: Each action link opens through the opener service when clicked or when Enter or Space is pressed on it, so a web address opens in the browser and a `command:` link runs a command; links are underlined, act as buttons, and are skipped by Tab until the person moves in with the keyboard (BN-1.7).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\opener\browser\link.ts; V\src\vs\workbench\browser\parts\banner\bannerPart.ts (links built with tabIndex -1); V\src\vs\workbench\browser\parts\banner\media\bannerpart.css (`.message-actions-container a`); C bundle (module: banner part class `show`, links built with `tabIndex:-1`)

### BN-1.6 Focus the strip with Focus Banner
- Tag: both
- Evidence: source
- Boundary: no
- What: The person runs Focus Banner from the Command Palette (View category, no default key) to put keyboard focus on the strip; the strip is tabbable, and the `bannerFocused` context key is true while focus is inside it.
- Commands: workbench.action.focusBanner
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\banner\bannerPart.ts (FocusBannerAction, createContentArea); V\src\vs\workbench\browser\layout.ts (focusPart default case); V\src\vs\workbench\common\contextkeys.ts (BannerFocused); C bundle (module: banner part file, `workbench.action.focusBanner`, label "Focus Banner" via message 3320, `category:Yn.View,f1:!0`, context key `bannerFocused` via message 4380)

### BN-1.7 Move around inside the strip by keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: While the strip has focus, Right or Down moves to the next action (each link, then the close button, then back to the first link), Left or Up moves back the other way, and Escape returns focus to the strip itself.
- Commands: workbench.banner.focusBanner; workbench.banner.focusNextAction; workbench.banner.focusPreviousAction
- Menus: none found
- Keys: Escape (bannerFocused); Right Arrow, Down Arrow (bannerFocused); Left Arrow, Up Arrow (bannerFocused)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\banner\bannerPart.ts (three registerCommandAndKeybindingRule blocks, focusNextAction, focusPreviousAction, focusActionLink); C bundle (module: banner part file, ids `workbench.banner.focusBanner`, `workbench.banner.focusNextAction`, `workbench.banner.focusPreviousAction`, weight 200, same keys)

### BN-1.8 Theme the strip colors
- Tag: both
- Evidence: source
- Boundary: no
- What: Three theme colors control the strip - `banner.background`, `banner.foreground`, and `banner.iconForeground` - with defaults taken from the list selection colors (the light theme darkens the background by 0.3) and the editor info color, and a person can override them in a color customization.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Cursor: present
- Source: V\src\vs\workbench\common\theme.ts; V\src\vs\workbench\browser\parts\banner\media\bannerpart.css; C bundle (module: workbench theme colors, `Ft("banner.background",...)`, `banner.foreground`, `banner.iconForeground`, messages 4456 to 4458)

### BN-1.9 Queue notices by priority
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. Several notices can wait at once, the one with the highest `priority` shows (ties go to the most recently shown), hiding or closing the current notice reveals the next, and focus stays in the strip if it was there when the notice swapped.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: N\src\vs\workbench\services\banner\browser\bannerService.ts; N\src\vs\workbench\browser\parts\banner\bannerPart.ts (items map, renderActiveItem); C bundle (searched for `renderActiveItem`, no match, and the banner part still holds a single `item`)

### BN-1.10 Neutral look for low-key notices
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. A notice can ask for a neutral look, which gives the strip the side bar background, the normal foreground color, and a bottom border in the contrast border color instead of the blue selection colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: N\src\vs\workbench\browser\parts\banner\media\bannerpart.css (`.part.banner.neutral` rules); N\src\vs\workbench\services\banner\browser\bannerService.ts (neutral); C bundle (searched workbench.desktop.main.css for `part.banner.neutral`, no match)

## Handed off

- The strip inside a code editor (`div.editor-banner`, used by the Unicode highlighter, reuses the `banner.*` colors) - ED-5 (closest fit, no group clearly owns it)
- The Restricted Mode status bar item that goes with the notice - SS-4 (already named there)

## Unclear

- What Focus Banner does when no notice is showing. The strip has zero height then, and `focusPart` falls back to a plain `focus()` on the container. I read the code and did not run it.
- How the strip sits under Cursor's no-title-bar layout (title bar becomes a transparent fixed 34 px strip, context key `cursor.noTitlebarLayout.enabled`). The offset code knows about that layout, but I could not tell from the bundle where the strip is drawn in it.