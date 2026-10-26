Group: BN-1 Notice strip under the title bar. Pass: 3. Date: 2026-10-25.

### BN-1.1
1. When a notice is showing, a 26 px tall strip appears with 12 px text, holding from left to right an icon, a one-line message, a row of action links, and a small close button.
2. The icon is a codicon, an image from a web address, or the default product icon when the notice gives neither.
3. A message given as plain text shows as is, and a message given as markdown shows with its formatting and with any links in it underlined.
4. A message too long for the strip stays on one line and is cut off with an ellipsis instead of wrapping.
5. The action links show as underlined text after the message, and the close button stays at the right end of the strip.
6. A screen reader treats the strip as a `banner` region and reads the notice's aria label, or the message text when there is no aria label, instead of reading the icon, message, and links separately.

### BN-1.2
1. Showing a notice with a new id fills the strip with that notice's icon, message, links, and close button and makes the strip visible.
2. Showing a notice with a different id while another is on screen replaces it, so only one notice is ever visible.
3. Showing the notice that is already held makes the strip visible again without rebuilding its contents.
4. Hiding by id hides the strip only when that id is the notice on screen, and hiding any other id does nothing.

### BN-1.3
1. Before any notice shows, the strip has zero height and takes no space under the title bar.
2. When a notice shows, the whole workbench moves down by 26 px to make room for the strip.
3. When the notice is hidden or closed, the workbench moves back up and the 26 px is given back.
4. On desktop the strip sits directly under the title bar.
5. In the web build the strip sits above the title bar, except when window controls overlay is on.
6. With the title bar visible and `window.commandCenter` not set to false, the quick input opens over the title bar and strip 6 px from the top, and otherwise it opens below the strip and the title bar.
7. In Cursor's no-title-bar layout the title bar counts as zero height when the overlays are placed, and the quick input top is not set to 6 px. [cursor only]

### BN-1.4
1. Clicking the small X at the right end of the strip hides the strip and removes the notice's contents.
2. Focusing the X and pressing Enter or Space closes the notice the same way as a click.
3. Closing runs the notice's own close callback, so an owner that remembers dismissals can keep the notice from coming back.
4. The close button's tooltip and label read "Close Banner" unless the notice sets its own close text, which then shows instead.
5. Showing the same notice again after it was closed rebuilds it in full, because closing makes the strip forget it.

### BN-1.5
1. Clicking an action link opens its target through the opener, so a web address opens in the browser.
2. Clicking a link whose target is a `command:` address runs that command.
3. Pressing Enter or Space on a focused link does the same as clicking it.
4. Action links are underlined and are exposed to screen readers as buttons.
5. Pressing Tab does not stop on the action links, and they get focus only when the person moves in with the arrow keys.

### BN-1.6
1. Running Focus Banner from the Command Palette puts keyboard focus on the strip.
2. The Command Palette lists the command as `View: Focus Banner` (`workbench.action.focusBanner`) and it has no default keybinding.
3. The strip is a tab stop, so pressing Tab can land on the strip itself when a notice is showing.
4. While focus is inside the strip the `bannerFocused` context key is true, so keybindings that depend on it apply only there.

### BN-1.7
1. With focus in the strip, Right Arrow or Down Arrow (`workbench.banner.focusNextAction`) moves focus to the next action, going through each link, then the close button, then back to the first link.
2. With focus in the strip, Left Arrow or Up Arrow (`workbench.banner.focusPreviousAction`) moves focus the other way, and from the first link it wraps to the close button.
3. Pressing Escape (`workbench.banner.focusBanner`) moves focus from an action back to the strip itself.
4. When the person tabs out of the strip and back in, focus returns to the action they were last on instead of the strip itself.

### BN-1.8
1. `banner.background` sets the strip's background, defaulting to the list active selection background, darkened by 0.3 in light themes.
2. `banner.foreground` sets the color of the strip's text, links, and close button, defaulting to the list active selection foreground.
3. `banner.iconForeground` sets the color of a codicon icon in the strip, defaulting to the editor info color.
4. Putting any of the three ids in `workbench.colorCustomizations` overrides the theme's value for that color.

### BN-1.9
1. In VS Code 1.144 and later, when several notices are waiting, the strip shows the one with the highest `priority`, and a tie goes to the notice shown most recently. [stock only]
2. Hiding or closing the notice on screen reveals the next waiting notice instead of leaving the strip empty. [stock only]
3. If keyboard focus is inside the strip when the notice swaps, focus stays in the strip. [stock only]

### BN-1.10
1. In VS Code 1.144 and later, a notice that asks for a neutral look shows the strip with the side bar background and the normal foreground color instead of the blue selection colors. [stock only]
2. A neutral strip draws a bottom border in the contrast border color. [stock only]
3. In a neutral strip the icon, action links, and close button use the normal foreground color. [stock only]

## Unclear

- BN-1.9 and BN-1.10 are newer than 1.128.0, so Cursor's 1.128 base has no queue and no neutral look, and the first piece of each says which VS Code version it comes from.
- BN-1.3 piece 5 (web build) and the web part of piece 6 cannot be reached in the desktop Cursor app, so the catalog may count them but no desktop person can check them.
- BN-1.6 piece 3 depends on what a zero-height strip does when no notice is showing. The feature file lists this as unclear, so the piece says "when a notice is showing".
- BN-1.3 has no piece for how the strip is drawn under Cursor's no-title-bar layout beyond the offset math, because the feature file could not tell from the bundle where it is drawn.
