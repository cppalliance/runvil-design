# 9. Status bar (SS)

The thin bar along the bottom and every item in it.

6 groups, 131 features. Each feature lists its entry points and the pieces that finish it.

## SS-1 Bar behavior and per-item control

(both). How the bar is shown, ordered left and right, hidden item by item, reached by keyboard, and what hover and click do. Examples: `workbench.statusBar.visible`; the per-item "Hide" menu; Focus Status Bar; item hover.

16 features.

#### SS-1.1 Show or hide the status bar

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (command, menu row, Customize Layout row and setting are all present; Cursor's toggle also stores the new state under storage key `cursor/editorLayout.statusBarVisible` and writes the setting twice, first to the in-memory target and then through the normal write that picks the target itself; the Cursor Settings Layout switch is cursor only; the Sessions-window condition that V puts on the menu row and precondition is absent)
- What: Turn the whole bar on or off from View > Appearance > Status Bar (checked while the bar shows), the Command Palette command "Toggle Status Bar Visibility", the "Hide Status Bar" row at the top of the bar's right-click menu, the Status Bar row in the Customize Layout picker (TB-4 owns the picker), or the `workbench.statusBar.visible` setting (default on). Cursor adds a Status Bar switch in the Layout card on the first tab of Cursor Settings (text "Show status bar at the bottom of the window"; ED-8 owns that page). While the bar is hidden, the notification center and toasts move down to the window edge.
- Commands: workbench.action.toggleStatusbarVisibility; workbench.action.customizeLayout
- Menus: MenubarAppearanceMenu (group 2_workbench_layout, order 3); the status bar right-click menu (built in code, not a MenuId)
- Keys: none found
- Settings: workbench.statusBar.visible
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:393-432 (action), :1370 (Customize Layout row), :1596 (Restore Defaults resets the setting); V\src\vs\workbench\browser\workbench.contribution.ts:624-629 (setting); V\src\vs\workbench\browser\layout.ts:1608-1620 (setStatusBarHidden), :2925-2926 and :2942-2943 (setting to layout state and back); V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:620-621 (right-click row); V\src\vs\workbench\browser\parts\notifications\media\notificationsCenter.css:18 and notificationsToasts.css:15 (`nostatusbar` offsets); C bundle (module layoutActions, nls 3169, 3109, 3138, 4221; the Layout card in the Cursor Settings React code, function `oww`, label "Status Bar")
- Pieces:
  1. Choosing View > Appearance > Status Bar (`workbench.action.toggleStatusbarVisibility`) shows or hides the whole bar, and the row has a check mark while the bar shows.
  2. Running Toggle Status Bar Visibility from the Command Palette does the same toggle.
  3. The Hide Status Bar row at the top of the bar's right-click menu hides the bar.
  4. The Status Bar row in the Customize Layout picker (`workbench.action.customizeLayout`) shows or hides the bar.
  5. Setting `workbench.statusBar.visible` (default on) shows or hides the bar, and stays in step with the other ways of toggling it.
  6. While the bar is hidden, the notification center and toasts move down to the window edge.
  7. Cursor Settings has a Status Bar switch in the Layout card on its first tab, with the text "Show status bar at the bottom of the window". [cursor only]

#### SS-1.2 Hide one item with Hide '{name}'

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click directly on an item and the last row of the menu reads Hide '{name}' with the item's name; clicking it removes that item from the bar at once. For an item that came from an extension the menu also gets a Manage Extension row above it, which opens the extension system and is out of scope here (see Handed off). The hidden list is stored per profile under storage key `workbench.statusbar.hidden`, and another window on the same profile picks up the change.
- Commands: none found
- Menus: the status bar right-click menu (built in code, not a MenuId)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:617-653 (menu build, item under the mouse); V\src\vs\workbench\browser\parts\statusbar\statusbarActions.ts:38-60 (HideStatusbarEntryAction, ManageExtensionAction); V\src\vs\workbench\browser\parts\statusbar\statusbarModel.ts:21-60, :135-151, :245-251 (hidden set, storage key, profile scope, save); C bundle (module statusbar, class `rSk` with nls 3986 "Hide '{0}'", class `WEa` with id `statusbar.manage.extension` and nls 3987, storage key `workbench.statusbar.hidden`)
- Pieces:
  1. Right-clicking directly on an item shows a menu whose last row reads Hide '{name}' with that item's name.
  2. Clicking Hide '{name}' removes that item from the bar at once.
  3. The hidden items are remembered per profile, so they stay hidden after a restart, and another window on the same profile picks up the change.
  4. Right-clicking empty space on the bar shows no Hide '{name}' row.

#### SS-1.3 Turn items on or off from the right-click checklist

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-click anywhere on the bar to get "Hide Status Bar", a separator, and then one checkable row for every item the bar knows about, in bar order and labelled with the item's name; a tick means the item is shown. Clicking a row flips that item, which is the only way to bring a hidden item back. If several items share one id (for example from different extensions) only one row is listed for that id.
- Commands: none found
- Menus: the status bar right-click menu (built in code, not a MenuId)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:617-634; V\src\vs\workbench\browser\parts\statusbar\statusbarActions.ts:22-36 (ToggleStatusbarEntryVisibilityAction, checked = not hidden); V\src\vs\workbench\browser\parts\statusbar\statusbarModel.ts:132-151 (isHidden, hide, show); C bundle (module statusbar, `getContextMenuActions` with nls 3989 "Hide Status Bar", class `iSk`)
- Pieces:
  1. Right-clicking anywhere on the bar opens a menu with Hide Status Bar, a separator, and one checkable row for every item the bar knows about, in bar order and labelled with the item's name.
  2. A check mark means the item is shown, and clicking a row hides a shown item or brings back a hidden one, which is the only way to bring a hidden item back.
  3. When several items share one id, for example from different extensions, the menu lists only one row for that id.

#### SS-1.4 Items sit on the left or right and sort by priority

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the sort code reads the same; Cursor's notifications bell is hard-wired right-most with priority negative infinity, and V's `workbench.notifications.position` setting that can move it is not in the bundle, see SS-5)
- What: Every item is placed on the left or the right side of the bar and ordered by a number, where a higher number sits further left on either side; ties break by a hash of the item id so the order stays the same from run to run. An item can instead be placed just before or after another named item with its own fallback number, and it falls back to the number if that other item is not on the bar. The person cannot drag or reorder items; only the code that adds an item sets its side and priority, once, when it is created.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarModel.ts:255-373 (sort); V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:254-273 (priority and secondary hash), :468-509 (append in order, right side reversed because the container is flex row-reverse); V\src\vs\workbench\services\statusbar\browser\statusbar.ts:38-100 (alignment, location, priority contract); C bundle (class `oSk`, sort and `status.notifications` registration with priority -Infinity)
- Pieces:
  1. Every item sits on either the left side or the right side of the bar.
  2. Within a side, items are ordered by a priority number, and a higher number sits further left on either side.
  3. Items with the same priority are ordered by a hash of the item id, so the order is the same from run to run.
  4. An item can be placed just before or after another named item, and it falls back to its own priority number when that other item is not on the bar.
  5. Dragging an item on the bar does not reorder it.

#### SS-1.5 Related items sit together as a compact pair

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (compact items exist, for example `status.scm.provider` with `compact:!0`; the spacing differs, the compact-left label has margin-right 3px in Cursor's CSS against 5px in V's)
- What: When an item is positioned next to another with the compact flag, the two sit closer with less padding between them, and hovering one lights both with the item hover color and the one under the pointer with the stronger compact hover color. Only items that have a command get that hover feedback, and it is skipped in high-contrast themes. In V the Source Control provider picker, the extra repository items and the Language Status and Language Detection items use it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:511-593 (updateCompactEntries); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (compact-left and compact-right rules); V\src\vs\workbench\contrib\scm\browser\activity.ts:161 and :176; V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts:227; V\src\vs\workbench\contrib\languageDetection\browser\languageDetection.contribution.ts:104; C bundle (updateCompactEntries code, `status.scm.provider` entry with compact true); C\out\vs\workbench\workbench.desktop.main.css (compact-left rules)
- Pieces:
  1. An item positioned next to another with the compact flag sits closer to it, with less padding between the two.
  2. Hovering one half of a compact pair lights both halves with the item hover color and the half under the pointer with the stronger compact hover color.
  3. Only items that have a command get that hover feedback, and the compact effect is skipped in high-contrast themes.
  4. In stock, the Source Control provider picker, the extra repository items, the Language Status item, and the Language Detection item sit as compact pairs.
  5. The compact-left label has a 3 px right margin instead of the stock 5 px. [cursor only]

#### SS-1.6 Long items are cut and the bar stays one row

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the same rules are in the bundle CSS: height 22px, `max-width:40vw`, right items `flex-direction:row-reverse;flex-wrap:wrap`)
- What: The bar is a fixed 22 px high strip and each item is capped at 40 percent of the window width, with long text cut off by an ellipsis. The right-side group is set to wrap with its first item on the right, and the bar hides anything that spills past its 22 px, so a crowded bar loses items instead of growing a second row. This is read from the CSS only and was not run on screen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (the `.part.statusbar`, `.right-items`, `.statusbar-item` and `.statusbar-item-label` rules); V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:123 (HEIGHT 22); C\out\vs\workbench\workbench.desktop.main.css
- Pieces:
  1. The bar is a fixed strip 22 px high.
  2. Each item is capped at 40 percent of the window width, and text that is too long is cut off with an ellipsis.
  3. When the items do not fit, the ones that spill past the 22 px are clipped and the bar never grows a second row.

#### SS-1.7 Focus the status bar from the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run "Focus Status Bar" from the Command Palette (View category, no default key), or press F6 and Shift+F6 to cycle focus through the window's parts, which includes the bar. The bar takes focus and draws an outline in the `statusBar.focusBorder` color, and if an item had focus earlier, that item gets it back. While focus is inside the bar the context key `statusBarFocused` is true, which turns on the arrow keys in SS-1.8.
- Commands: workbench.action.focusStatusBar; workbench.action.focusNextPart; workbench.action.focusPreviousPart
- Menus: CommandPalette (f1 true for each)
- Keys: F6 (Focus Next Part); Shift+F6 (Focus Previous Part); Focus Status Bar has no default key
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarActions.ts:126-144; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:420-427 (focus, restore last entry), :430-440 (context key, tabIndex 0); V\src\vs\workbench\common\contextkeys.ts:128; V\src\vs\workbench\browser\layout.ts:1303-1304 (focusPart); V\src\vs\workbench\browser\actions\navigationActions.ts:260-350 (part cycling includes the bar); C bundle (class `sSk` with id `workbench.action.focusStatusBar` and nls 3988, context key `statusBarFocused` nls 4377, F6 commands with nls 3225 and 3226)
- Pieces:
  1. Running Focus Status Bar (`workbench.action.focusStatusBar`) from the Command Palette moves keyboard focus to the bar, and the command has no default key.
  2. Pressing `F6` (Focus Next Part) and `Shift+F6` (Focus Previous Part) cycles focus through the window's parts, and the bar is one of them.
  3. The focused bar draws a focus outline.
  4. If an item had focus earlier, that item gets focus back when the bar is focused again.
  5. While focus is inside the bar, the arrow keys, Home, End, and Escape of SS-1.8 work because the context key `statusBarFocused` is true.

#### SS-1.8 Move between items with the arrow keys, Home and End

- Tag: both. Evidence: source. Boundary: no. Cursor: present (all five ids registered with the same keys and the same when context)
- What: With focus in the bar, Left or Up goes to the previous item, Right or Down to the next, Home to the first and End to the last, in the same left to right order you see on screen, skipping hidden items and wrapping round at either end. Escape first drops focus from the item to the bar itself, and the next Escape returns focus to the active editor.
- Commands: workbench.statusBar.focusPrevious; workbench.statusBar.focusNext; workbench.statusBar.focusFirst; workbench.statusBar.focusLast; workbench.statusBar.clearFocus
- Menus: none found
- Keys: Left or Up when statusBarFocused (focusPrevious); Right or Down when statusBarFocused (focusNext); Home when statusBarFocused (focusFirst); End when statusBarFocused (focusLast); Escape when statusBarFocused (clearFocus)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarActions.ts:63-124; V\src\vs\workbench\browser\parts\statusbar\statusbarModel.ts:162-208 (focusEntry, hidden items skipped, wrap to the start position); C bundle (five `registerCommandAndKeybindingRule` calls, weight 200, `when` is `statusBarFocused`)
- Pieces:
  1. Pressing Left or Up with focus in the bar moves to the previous item (`workbench.statusBar.focusPrevious`), and Right or Down moves to the next item (`workbench.statusBar.focusNext`), following the left to right order on screen.
  2. Pressing Home goes to the first item (`workbench.statusBar.focusFirst`), and pressing End goes to the last item (`workbench.statusBar.focusLast`).
  3. Moving skips hidden items and wraps around at either end.
  4. Pressing Escape (`workbench.statusBar.clearFocus`) drops focus from the item to the bar itself, and the next Escape returns focus to the active editor.

#### SS-1.9 Click an item to run its command

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (adds `commandExecutionTrigger` with `click` as default or `mousedown`, which reacts to the left button only; drops the `workbenchActionExecuted` telemetry call; Space, Enter, Escape and arrow handling on the item is the same)
- What: Click an item, tap it on a touch screen, or press Enter or Space while it has keyboard focus to run its command; an item with no command looks disabled, with a plain pointer and no hover color, and does nothing. If the command throws, the person sees an error notification with the message. Cursor lets an item ask for the left mouse button press instead of the release, and no longer sends the click telemetry that stock does.
- Commands: none found
- Menus: none found
- Keys: Enter or Space on a focused item
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:144-171 (listeners and disabled class), :226-243 (executeCommand, telemetry, error notification); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (`a.disabled`, hover rules); C bundle (item class with `getCommandExecutionTrigger`, mouse down branch, `status.sessionRecording` entry that uses `mousedown`)
- Pieces:
  1. Clicking an item, or tapping it on a touch screen, runs its command.
  2. Pressing Enter or Space while an item has keyboard focus runs its command.
  3. An item with no command looks disabled, with a plain pointer and no hover color, and does nothing when clicked.
  4. When an item's command throws, an error notification with the message appears.
  5. An item can ask to run its command when the left mouse button goes down instead of when it is released, and other buttons do not run it. [cursor only]

#### SS-1.10 Hover an item to read its tooltip

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the hover setup keeps instant hover, the 500 ms rule and hide on key down but has no 90 percent height cap; Cursor adds `dontHideHoverOnClick`, which Cursor Tab's item sets so its panel stays open after a click)
- What: Hovering an item opens its tooltip with no delay when it is plain text or markdown, and after 500 ms when it is a custom element or slow-loading markdown. The tooltip closes when a key is pressed, and Escape, Left or Right closes it while the item has keyboard focus. In stock the tooltip is capped at 90 percent of the window height, and a tooltip can add footer links that run commands, which the item code supports but no in-scope item in 1.128 uses.
- Commands: none found
- Menus: none found
- Keys: Escape, Left or Right while an item with a command has focus (closes the hover)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:182-209 (hover delegate); V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:120-142 (tooltip and footer actions), :155-166 (key handling); V\src\vs\workbench\services\statusbar\browser\statusbar.ts:117-130, :162-166 (tooltip types); C bundle (hover delegate with `instantHover:!0,dynamicDelay` and `persistence:{hideOnKeyDown:!0,sticky}`, `dontHideHoverOnClick` in the item and the managed hover)
- Pieces:
  1. Hovering an item whose tooltip is plain text or markdown opens the tooltip with no delay.
  2. A tooltip that is a custom element or slow-loading markdown opens after 500 ms.
  3. The tooltip closes when a key is pressed.
  4. Pressing Escape, Left, or Right while an item with a command has keyboard focus closes its tooltip.
  5. The tooltip is capped at 90 percent of the window height. [stock only]
  6. An item can keep its tooltip open after a click, and the Cursor Tab item uses this so its panel stays open. [cursor only]

#### SS-1.11 Click an item to open its tooltip

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: An item whose command is the special id `statusBar.entry.showTooltip` opens its tooltip on click and moves keyboard focus into it, instead of running a command, so the tooltip stays up until the person moves away or presses a key. It only works if the item has a tooltip. The id has an empty title, so it has no label of its own. Window Zoom and the Language Status item use it in stock, and Cursor Tab's item uses it when a condition holds.
- Commands: statusBar.entry.showTooltip
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\statusbar\browser\statusbar.ts:104-107; V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:152, :228-229; V\src\vs\workbench\electron-browser\window.ts:1244; V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts:223; C bundle (`WXt={id:"statusBar.entry.showTooltip",title:""}`, the Cursor Tab entry sets `command:i?WXt:ayg.ID`)
- Pieces:
  1. Clicking an item whose command is `statusBar.entry.showTooltip` opens its tooltip instead of running a command.
  2. Keyboard focus moves into the opened tooltip, so it stays up until the person moves away or presses a key.
  3. The click opens nothing when the item has no tooltip.
  4. In stock, the Window Zoom item and the Language Status item open their tooltips this way.
  5. The Cursor Tab item opens its tooltip this way when a condition holds. [cursor only]

#### SS-1.12 Show a spinner in an item while it works

- Tag: both. Evidence: source. Boundary: no. Cursor: present (`showProgress` found in the window progress entry, the remote indicator entry and the `recordSession` entry)
- What: An item can ask for a progress icon in front of its text, either a spinning loader or a sync icon, and the icon keeps spinning without restarting when the text changes. In stock the window progress item, the remote indicator while it connects, and the terminal session recording item use it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:92-93, :287-342 (StatusBarCodiconLabel); V\src\vs\workbench\services\statusbar\browser\statusbar.ts:192-198; V\src\vs\workbench\services\progress\browser\progressService.ts:184; V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts:563-571; C bundle (`showProgress:t.type||!0` in the window progress entry, `showProgress:i` in the remote indicator entry, `showProgress:!0` in the `recordSession` entry)
- Pieces:
  1. An item can show a progress icon in front of its text, either a spinning loader or a sync icon.
  2. The icon keeps spinning without restarting when the item's text changes.
  3. The window progress item, the remote indicator while it connects, and the terminal session recording item show the progress icon.

#### SS-1.13 Beak pointing at the open notifications center

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A small triangle points up from an item's top edge to show that a popup is open from that item. Only the notifications bell uses it, and only while the notification center is open; the triangle takes the bar's border color, or its background color if there is no border.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:173-181; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:703-706; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (`has-beak` rules); V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts:92; C bundle (`showBeak:this.isNotificationsCenterVisible` in the bell entry, the `has-beak` class in the item class)
- Pieces:
  1. A small triangle points up from the top edge of an item to show that a popup is open from that item.
  2. Only the notifications bell shows it, and only while the notification center is open.
  3. The triangle takes the bar's border color, or the bar's background color when there is no border.

#### SS-1.14 Item names and screen reader labels

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Every item has a short name, which names its row in the right-click checklist and its Hide '{name}' row, and a screen reader label that is set on both the item and its inner button so it is not read twice; the inner element has role button unless the item sets another role. The bar itself is a footer with role status and aria-live off, so text changes are not read out on their own, and the bar can take focus with a tab stop. Extension items that give no label get one built from their text, with icon names turned into words, plus the tooltip.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:68-72, :108-118; V\src\vs\workbench\browser\workbench.ts:354, :370-377; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:439 (tabIndex 0); V\src\vs\workbench\api\browser\statusBarExtensionPoint.ts:84-95 (label fallback); V\src\vs\workbench\services\statusbar\browser\statusbar.ts:134-158; C bundle (`setAttribute("aria-label",e.ariaLabel)` on both elements, `role:e.role||"button"`)
- Pieces:
  1. Each item has a short name, and that name labels its row in the right-click checklist and its Hide '{name}' row.
  2. Each item has a screen reader label that is set on both the item and its inner button, so a screen reader does not read it twice.
  3. The inner element has the role button unless the item sets another role.
  4. The bar is a footer with the role `status` and aria-live off, so a screen reader does not read out text changes on its own.
  5. The bar can take focus as a tab stop.
  6. An extension item that gives no label gets one built from its text, with icon names turned into words, plus its tooltip.

#### SS-1.15 Items shown in every window and the bar in popped-out windows

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the auxiliary bar part `workbench.parts.auxiliaryStatus` and `showInAllWindows` are present, Screen Reader Mode uses it; the Metered Connection item is missing from the bundle)
- What: A popped-out editor window has its own 22 px status bar that follows `workbench.statusBar.visible` and opens with the same right-click menu. It carries only the items marked to show in all windows, which are added to every open window and to new ones; in stock these are Screen Reader Mode, Scrolling Locked and Metered Connection, and Cursor has the first two. PN-1 owns the popped-out window as a whole.
- Commands: none found
- Menus: the status bar right-click menu (built in code, not a MenuId)
- Keys: none found
- Settings: workbench.statusBar.visible
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:760-835, :841-900 (auxiliary part, all-windows entries); V\src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts:111-150 (visibility and height); V\src\vs\workbench\contrib\accessibility\browser\accessibilityStatus.ts:91; V\src\vs\workbench\contrib\scrollLocking\browser\scrollLocking.ts:176; V\src\vs\workbench\contrib\meteredConnection\browser\meteredConnectionStatus.ts:56; C bundle (`workbench.parts.auxiliaryStatus.${c}`, `showInAllWindows:!0` on the Screen Reader Mode entry, `eDt.STATUS_BAR_VISIBILITY`)
- Pieces:
  1. A popped-out editor window has its own status bar, 22 px high.
  2. The popped-out bar follows the `workbench.statusBar.visible` setting.
  3. Right-clicking the popped-out bar opens the same menu as the main bar.
  4. The popped-out bar carries only the items marked to show in all windows, and those items are added to every open window and to new ones.
  5. The Screen Reader Mode item and the Scrolling Locked item show in every window's bar.
  6. The Metered Connection item shows in every window's bar. [stock only]

#### SS-1.16 Bar pinned to the bottom edge when the title bar is hidden

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: In Cursor's layout without a title bar, the bar is fixed to the bottom of the window across the full width at 22 px with a high stacking order, and the right side bar keeps 22 px of padding at its bottom so the bar does not cover it. The body gets the class `no-titlebar-layout` and the context key `cursor.noTitlebarLayout.enabled` reports it. How the person turns that layout on is not traced here (TB-4 owns the layout presets).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C\out\vs\workbench\workbench.desktop.main.css (rule `body.no-titlebar-layout .monaco-workbench .split-view-view.monaco-statusbar-split-view` with `position:fixed;bottom:0`, and the `padding-bottom:22px` rules for `.part.sidebar.right`); C bundle (context key `cursor.noTitlebarLayout.enabled`, class name `no-titlebar-layout`, `markSplitViewContainer` in the status bar part)
- Pieces:
  1. In Cursor's layout without a title bar, the bar is fixed to the bottom of the window across the full width, 22 px high, with a high stacking order. [cursor only]
  2. In that layout the right side bar keeps 22 px of padding at its bottom so the bar does not cover it. [cursor only]


## SS-2 Bar and item colors

(both). Bar colors in normal, no-folder and debugging states, plus item kinds (prominent, warning, error, remote, offline). Cursor's themes make the no-folder bar and remote indicator match the normal color.

15 features.

#### SS-2.1 Bar background, text and top line in a normal workspace

- Tag: both. Evidence: source. Boundary: no. Cursor: present (same tokens, same defaults and the same `updateStyles` code; the shipped theme values differ, see SS-2.13)
- What: In a window with a folder or workspace open, the bar is painted with `statusBar.background` and `statusBar.foreground`, with a 1 px line along its top edge in `statusBar.border`, or in the high-contrast border color when the theme sets no border; the line is hidden while the bar has keyboard focus. The bar fades between background colors over 0.15 s when motion is on. Token defaults are `#007ACC` with white text and no border, which the shipped themes replace (stock Dark 2026 uses `#191A1B` with `#8C8C8C` text, Light 2026 uses `#FAFAFD` with `#606060`, Cursor Dark uses `#141414` with `#F0F0F099`).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts:284-319 (foreground, background, border tokens); V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:656-682 (updateStyles, top line class); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (motion fade, `status-border-top` line); V\extensions\theme-defaults\themes\2026-dark.json:176-189 and 2026-light.json:181-194; V\src\vs\workbench\services\themes\common\workbenchThemeService.ts:42-45 (default themes are Dark 2026 and Light 2026); C bundle (tokens `statusBar.background` `#007ACC`, `statusBar.foreground`, `statusBar.border`, class set in `updateStyles`); C\extensions\theme-cursor\themes\cursor-dark-color-theme.json
- Pieces:
  1. In a window with a folder or workspace open, the bar is painted with `statusBar.background` and its text uses `statusBar.foreground`.
  2. A 1 px line along the top edge of the bar is drawn in `statusBar.border`, or in the high-contrast border color when the theme sets no border.
  3. The bar fades between background colors over 0.15 s when motion is on.
  4. When a theme sets nothing, the bar is `#007ACC` with white text and no top line.
  5. Dark 2026 uses `#191A1B` with `#8C8C8C` text, and Light 2026 uses `#FAFAFD` with `#606060` text. [stock only]
  6. Cursor Dark uses `#141414` with `#F0F0F099` text. [cursor only]

#### SS-2.2 Bar colors when no folder is open

- Tag: both. Evidence: source. Boundary: no. Cursor: present (tokens and the same purple default are in the bundle; Cursor's own themes set no-folder equal to the normal bar, see SS-2.13)
- What: A window with no folder or workspace open uses `statusBar.noFolderBackground`, `statusBar.noFolderForeground` and `statusBar.noFolderBorder` instead, and the bar switches over as soon as a folder is opened or closed. The token default background is purple `#68217A` (foreground and border follow the normal ones). Stock Dark 2026 sets no-folder equal to the normal bar, Dark Modern uses `#1F1F1F` against `#181818`, Light 2026 uses `#F0F0F3` against `#FAFAFD`, and the plain Dark and Light (Visual Studio) themes still show the purple.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts:291, :300-305, :321; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:220, :663-675 (state switch on `WorkbenchState.EMPTY`); V\extensions\theme-defaults\themes\dark_modern.json:97, light_modern.json:113, 2026-dark.json:182-183, 2026-light.json:187-188, dark_vs.json and light_vs.json (no no-folder keys); C bundle (`statusBar.noFolderBackground` registration with `#68217A`, `onDidChangeWorkbenchState` handler)
- Pieces:
  1. A window with no folder or workspace open paints the bar with `statusBar.noFolderBackground`, `statusBar.noFolderForeground`, and `statusBar.noFolderBorder`.
  2. The bar switches between the normal colors and the no-folder colors as soon as a folder is opened or closed.
  3. When a theme sets nothing, the no-folder background is purple `#68217A` and the foreground and border follow the normal bar.
  4. Dark 2026 sets the no-folder colors equal to the normal bar, and Light 2026 uses `#F0F0F3` against `#FAFAFD`. [stock only]
  5. Dark Modern uses `#1F1F1F` against `#181818`, and Dark (Visual Studio) and Light (Visual Studio) still show the purple.

#### SS-2.3 Bar colors while debugging

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the provider and the three tokens are in the bundle; Cursor's own themes tone it down, see SS-2.13)
- What: While a debug session is running the whole bar changes to `statusBar.debuggingBackground`, `statusBar.debuggingForeground` and `statusBar.debuggingBorder`, and goes back when the session ends. It does not apply to sessions started without debugging, to sessions that ask to hide the debug status bar, or while the debugger is still starting. It overrides the no-folder colors too. Token default is orange `#CC6633`; stock Dark Modern uses `#0078D4`, Dark 2026 `#3994BC`, and Light Modern `#FD716C` with black text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\statusbarColorProvider.ts:20-34 (tokens), :44-107 (override with priority 10, state test); V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:714-724 (overrideStyle); V\extensions\theme-defaults\themes\dark_modern.json:93-94, light_modern.json:110-111, 2026-dark.json:180-181; C bundle (`wsg=Ft("statusBar.debuggingBackground"` with `#CC6633`, `overrideStyle({priority:10,...})`, `suppressDebugStatusbar` read in the debug session class)
- Pieces:
  1. While a debug session is running, the whole bar changes to `statusBar.debuggingBackground`, `statusBar.debuggingForeground`, and `statusBar.debuggingBorder`, and goes back when the session ends.
  2. The debugging colors do not apply to sessions started without debugging, to sessions that ask to hide the debug status bar, or while the debugger is still starting.
  3. The debugging colors override the no-folder colors.
  4. When a theme sets nothing, the debugging background is orange `#CC6633`.
  5. Dark Modern uses `#0078D4`, and Light Modern uses `#FD716C` with black text.
  6. Dark 2026 uses `#3994BC`. [stock only]

#### SS-2.4 Turn the debugging color on or off

- Tag: both. Evidence: source. Boundary: no. Cursor: present (setting found with default true)
- What: The setting "Debug: Enable Status Bar Color" (description "Color of the status bar when the debugger is active", default on) decides whether SS-2.3 happens; switching it takes effect at once, even in a running session. It is a plain boolean in the Settings editor under Debug.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.enableStatusBarColor
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:689-693; V\src\vs\workbench\contrib\debug\browser\statusbarColorProvider.ts:75-91; V\src\vs\workbench\contrib\debug\common\debug.ts:826; C bundle (`"debug.enableStatusBarColor":{type:"boolean",description:k(6160,null),default:!0}`)
- Pieces:
  1. Setting `debug.enableStatusBarColor` (Debug: Enable Status Bar Color, default on) decides whether the bar takes the debugging colors.
  2. Switching the setting takes effect at once, even in a running debug session.
  3. The setting shows as a checkbox under Debug in the Settings editor.

#### SS-2.5 Prominent items stand out

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the four tokens, the CSS and nine `kind:"prominent"` entries are in the bundle; Cursor's own themes use a faint tint, for example `#F0F0F011` in Cursor Dark)
- What: An item marked prominent gets its own background and text from `statusBarItem.prominentBackground` and `statusBarItem.prominentForeground`, and its own hover pair `prominentHoverBackground` and `prominentHoverForeground` (the hover pair is skipped when the item sits in a compact pair). Token default is a black 50 percent layer with the bar's text color; stock Dark Modern uses `#6E768166` and Dark 2026 uses `#3994BC`. In stock the items that use it are Window Zoom, Restricted Mode, Screen Reader Optimized, Scrolling Locked, Tab Moves Focus, OVR, Column Selection, the notebook Select Kernel item and the Show Output quiet item.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts:353-359; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (prominent rules and the compact exception); V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:183-194 (kind class); V\src\vs\workbench\electron-browser\window.ts:1245; V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts:686; V\src\vs\workbench\browser\parts\editor\editorStatus.ts:452, :471, :489; V\src\vs\workbench\contrib\accessibility\browser\accessibilityStatus.ts:90; V\src\vs\workbench\contrib\scrollLocking\browser\scrollLocking.ts:175; V\src\vs\workbench\contrib\notebook\browser\contrib\editorStatusBar\editorStatusBar.ts:173; V\src\vs\workbench\api\browser\mainThreadOutputService.ts:101; C bundle (`statusBarItem.prominentBackground` with `Pi.black.transparent(.5)`, `.prominent-kind` CSS)
- Pieces:
  1. An item marked prominent gets its own background and text from `statusBarItem.prominentBackground` and `statusBarItem.prominentForeground`.
  2. A prominent item uses `statusBarItem.prominentHoverBackground` and `statusBarItem.prominentHoverForeground` on hover, except when it sits in a compact pair.
  3. When a theme sets nothing, the prominent background is a 50 percent black layer and the text is the bar's text color.
  4. Dark Modern uses `#6E768166` for the prominent background.
  5. Dark 2026 uses `#3994BC` for the prominent background. [stock only]
  6. Cursor's own themes use a faint tint, for example `#F0F0F011` in Cursor Dark. [cursor only]
  7. In stock, the items marked prominent are Window Zoom, Restricted Mode, Screen Reader Optimized, Scrolling Locked, Tab Moves Focus, OVR, Column Selection, the notebook Select Kernel item, and the Show Output quiet item.

#### SS-2.6 Warning items turn amber

- Tag: both. Evidence: source. Boundary: no. Cursor: present (tokens, CSS and two `kind:"warning"` entries are in the bundle; I did not check which items they belong to)
- What: An item marked warning is drawn with `statusBarItem.warningBackground` and `statusBarItem.warningForeground`, and the hover pair `warningHoverBackground` and `warningHoverForeground`. Background default is the editor warning color darkened by 40 percent with white text. In stock the Problems visibility item, the Pty Host item and a language status entry with warning severity use it, and an extension item that asks for the warning background color is mapped to this kind.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts:374-385; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (warning rules); V\src\vs\workbench\contrib\markers\browser\markers.contribution.ts:638; V\src\vs\workbench\contrib\terminal\browser\baseTerminalBackend.ts:70; V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts:414-418; V\src\vs\workbench\api\browser\statusBarExtensionPoint.ts:97-104 and V\src\vs\workbench\api\common\extHostStatusBar.ts:23-26; C bundle (`statusBarItem.warningBackground` registration, two `kind:"warning"` entries)
- Pieces:
  1. An item marked warning is drawn with `statusBarItem.warningBackground` and `statusBarItem.warningForeground`.
  2. A warning item uses `statusBarItem.warningHoverBackground` and `statusBarItem.warningHoverForeground` on hover.
  3. When a theme sets nothing, the warning background is the editor warning color darkened by 40 percent, with white text.
  4. In stock, the Problems visibility item, the Pty Host item, and a language status entry with warning severity are drawn as warning items.
  5. An extension item that asks for the warning background color is drawn as a warning item.

#### SS-2.7 Error items turn red

- Tag: both. Evidence: source. Boundary: no. Cursor: present (tokens and CSS are in the bundle; the Performance Trace item that stock uses is missing from Cursor, see SS-5)
- What: An item marked error is drawn with `statusBarItem.errorBackground` and `statusBarItem.errorForeground`, and the hover pair `errorHoverBackground` and `errorHoverForeground`. Background default is the editor error color darkened by 40 percent with white text. In stock the Performance Trace recording item and a language status entry with error severity use it, and an extension item that asks for the error background color is mapped to this kind.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts:361-372; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (error rules); V\src\vs\workbench\electron-browser\actions\developerActions.ts:181; V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts:414-418; V\src\vs\workbench\api\browser\statusBarExtensionPoint.ts:97-104; C bundle (`statusBarItem.errorBackground` registration with `X2(Iwn,.4)`, `.error-kind` CSS)
- Pieces:
  1. An item marked error is drawn with `statusBarItem.errorBackground` and `statusBarItem.errorForeground`.
  2. An error item uses `statusBarItem.errorHoverBackground` and `statusBarItem.errorHoverForeground` on hover.
  3. When a theme sets nothing, the error background is the editor error color darkened by 40 percent, with white text.
  4. A language status entry with error severity is drawn as an error item.
  5. An extension item that asks for the error background color is drawn as an error item.
  6. The Performance Trace recording item is drawn as an error item. [stock only]

#### SS-2.8 Remote indicator color

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the bundle sets `kind:this.networkState==="offline"?"offline":"remote"` with no check for a remote connection, and does not have the default-label test that stock uses)
- What: The remote indicator at the far left is drawn with the remote kind colors, `statusBarItem.remoteBackground` and `statusBarItem.remoteForeground` plus the hover pair `remoteHoverBackground` and `remoteHoverForeground`; the defaults copy the activity bar badge colors, and Dark Modern sets `#0078D4` with white text. In stock the kind is applied only when the window is connected to a remote, so a plain local window shows an unstyled item. Cursor applies it all the time unless the machine is offline, so these tokens always show on the indicator.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts:576-587; V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts:563-581 (line 568 picks the kind); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (remote rules); V\extensions\theme-defaults\themes\dark_modern.json:100-101, hc_black.json:12; C bundle (remote indicator entry `kind:...?"offline":"remote"`, token `statusBarItem.remoteBackground` equal to the activity bar badge token)
- Pieces:
  1. The remote indicator at the far left is drawn with `statusBarItem.remoteBackground` and `statusBarItem.remoteForeground`.
  2. The remote indicator uses `statusBarItem.remoteHoverBackground` and `statusBarItem.remoteHoverForeground` on hover.
  3. The default remote colors copy the activity bar badge colors, and Dark Modern sets `#0078D4` with white text.
  4. The remote colors apply only while the window is connected to a remote, so a plain local window shows an unstyled indicator. [stock only]
  5. The remote colors always show on the indicator unless the machine is offline. [cursor only]

#### SS-2.9 Offline indicator color

- Tag: both. Evidence: source. Boundary: no. Cursor: present (tokens, CSS and the `"offline"` choice in the remote indicator entry are in the bundle)
- What: When the system reports that the network is offline, the remote indicator changes to the offline kind, with `statusBarItem.offlineBackground` (default dark red `#6c1717`), `statusBarItem.offlineForeground` (the remote foreground), and the hover pair `offlineHoverBackground` and `offlineHoverForeground`. It returns to the remote kind or the normal look when the network comes back.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts:589-600; V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts:102, :303-304, :433-437, :568; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (offline rules); C bundle (`statusBarItem.offlineBackground` with `#6c1717`, the remote indicator entry)
- Pieces:
  1. When the system reports that the network is offline, the remote indicator switches to the offline colors.
  2. The offline background `statusBarItem.offlineBackground` defaults to dark red `#6c1717`, and `statusBarItem.offlineForeground` defaults to the remote foreground.
  3. The offline indicator uses `statusBarItem.offlineHoverBackground` and `statusBarItem.offlineHoverForeground` on hover.
  4. The indicator returns to the remote or normal look when the network comes back.

#### SS-2.10 Item hover colors

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the hover, hover text and compact hover tokens, the per-kind hover tokens and the hover CSS are in the bundle; the compact hover colors are set by `updateCompactEntries`, whose spacing differs, see SS-1.5)
- What: While the pointer is over an item that has a command, its background becomes `statusBarItem.hoverBackground` and its text `statusBarItem.hoverForeground`; items of the colored kinds use their own hover pair instead (prominent, warning, error, remote, offline). The two halves of a compact pair light up together, and the half under the pointer uses `statusBarItem.compactHoverBackground`. High-contrast themes add a dashed outline in the contrast active border color and skip the compact effect. Defaults are white 12 percent on dark themes and black 12 percent on light themes, and the hover text follows the bar text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts:337-351; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:511-593 (compact hover colors, skipped when `isHighContrast`); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (the `a:hover:not(.disabled)` rules and the per-kind hover rules); C bundle (`statusBarItem.hoverBackground`, `statusBarItem.hoverForeground`, `statusBarItem.compactHoverBackground` found; hover rules in the bundle CSS)
- Pieces:
  1. While the pointer is over an item that has a command, its background becomes `statusBarItem.hoverBackground` and its text becomes `statusBarItem.hoverForeground`.
  2. Items of the prominent, warning, error, remote, and offline kinds use their own hover colors instead.
  3. The two halves of a compact pair light up together, and the half under the pointer uses `statusBarItem.compactHoverBackground`.
  4. High-contrast themes add a dashed outline in the contrast active border color and skip the compact pair effect.
  5. When a theme sets nothing, the hover background is white 12 percent on dark themes and black 12 percent on light themes, and the hover text follows the bar text.

#### SS-2.11 Item pressed color

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While the mouse button is held down on an item that has a command, its background changes to `statusBarItem.activeBackground` (default white 18 percent, black 18 percent in high contrast light), and high-contrast themes draw a solid outline in the contrast active border color instead of the fill.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts:323-328; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (`a:active:not(.disabled)` rules); C bundle (token `statusBarItem.activeBackground`, matching rules in the bundle CSS)
- Pieces:
  1. While the mouse button is held down on an item that has a command, the item's background changes to `statusBarItem.activeBackground`.
  2. When a theme sets nothing, the pressed background is white 18 percent, and black 18 percent in high contrast light.
  3. High-contrast themes draw a solid outline in the contrast active border color instead of the fill.

#### SS-2.12 Keyboard focus outline

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both tokens, the `:focus` and `:focus-visible` stylesheet written by `updateStyles`, and the macOS corner rules are in the bundle)
- What: When the bar itself has keyboard focus it draws an outline in `statusBar.focusBorder`, and the top line is hidden to make room. When an item has focus by keyboard, it gets a 1 px solid outline in the theme's contrast active border color if it has one, otherwise in `statusBarItem.focusBorder`, set 2 px inside the item when the bar has a border and 1 px otherwise. On macOS, when the window is not full screen, the focused bar gets rounded bottom corners (10 px, 16 px on Tahoe) so the outline fits. In normal themes both focus tokens default to the bar's text color; Dark Modern sets `#0078D4`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\theme.ts:314-319, :330-335; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:684-708 (dynamic stylesheet); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (macOS corner rules, top line hidden on focus); V\extensions\theme-defaults\themes\dark_modern.json:95, :98; C bundle (`statusBar.focusBorder`, `statusBarItem.focusBorder`, `updateStyles` stylesheet text); C\out\vs\workbench\workbench.desktop.main.css (mac rules)
- Pieces:
  1. When the bar itself has keyboard focus, it draws an outline in `statusBar.focusBorder` and hides its top line.
  2. When an item has keyboard focus, it gets a 1 px solid outline in the theme's contrast active border color if there is one, and otherwise in `statusBarItem.focusBorder`.
  3. The item outline sits 2 px inside the item when the bar has a border, and 1 px inside when it does not.
  4. On macOS, when the window is not full screen, a focused bar gets rounded bottom corners, 10 px and 16 px on Tahoe, so the outline fits.
  5. When a theme sets nothing, both focus colors are the bar's text color, and Dark Modern sets `#0078D4`.

#### SS-2.13 Cursor themes match the bar in no-folder windows and on the remote indicator

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor's seven themes set the no-folder bar background equal to the normal bar (Cursor Dark `#141414`, Cursor Light `#F3F3F3`, the High Contrast pair `#0A0A0A` and `#FFFFFF`, Midnight `#191c22`, and the two Colorblind themes the same as their parents) and set the remote indicator to the bar color too in Cursor Dark, Cursor Light, the two Colorblind themes and Light High Contrast, so no purple or blue block appears. The debugging look is a faint tint in Cursor Dark and its Colorblind twin (`#F0F0F01C`), a gray-blue shift (`#434C5E`) in Dark High Contrast and Midnight, and no change at all in the three light themes. Dark High Contrast and Midnight do not set the remote colors and fall back to the activity bar badge colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorTheme
- Source: C\extensions\theme-cursor\themes\cursor-dark-color-theme.json; cursor-dark-colorblind-color-theme.json; cursor-dark-hc-color-theme.json; cursor-dark-midnight-color-theme.json; cursor-light-color-theme.json; cursor-light-colorblind-color-theme.json; cursor-light-hc-color-theme.json (keys `statusBar.*` and `statusBarItem.*`); C\extensions\theme-cursor\package.json (theme list); C bundle (default themes `Cursor Dark` and `Cursor Light`, `workbench.colorTheme` found)
- Pieces:
  1. Cursor Dark sets the no-folder bar background to `#141414` and Cursor Light to `#F3F3F3`, equal to their normal bars. [cursor only]
  2. The two High Contrast themes set the no-folder background to `#0A0A0A` and `#FFFFFF`, Midnight sets it to `#191c22`, and the two Colorblind themes match their parents, all equal to their normal bars. [cursor only]
  3. Cursor Dark, Cursor Light, the two Colorblind themes, and Light High Contrast set the remote indicator to the bar color, so no purple or blue block appears. [cursor only]
  4. While debugging, Cursor Dark and its Colorblind twin show a faint tint `#F0F0F01C`. [cursor only]
  5. While debugging, Dark High Contrast and Midnight shift the bar to the gray-blue `#434C5E`, and the three light themes show no change. [cursor only]
  6. Dark High Contrast and Midnight do not set the remote colors, so the remote indicator falls back to the activity bar badge colors. [cursor only]
  7. Setting `workbench.colorTheme` chooses which of the seven Cursor themes colors the bar. [cursor only]

#### SS-2.14 Dimmed bar when the window is inactive

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (0 hits for `statusBar.inactiveBackground` in the bundle)
- What: newer-than-1.128. A new token, `statusBar.inactiveBackground`, repaints the bar while the window has lost focus, in a window with a folder or workspace open and no debug color active. Its default is unset, so nothing changes until a theme sets it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: N\src\vs\workbench\common\theme.ts:310-315; N\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:232-235, :732-734; N\src\vs\workbench\contrib\modernUI\README.md:209, :229; C bundle (searched, no match)
- Pieces:
  1. Newer than 1.128: the token `statusBar.inactiveBackground` repaints the bar while the window has lost focus, in a window with a folder or workspace open and no debug color active. [stock only]
  2. Newer than 1.128: the token has no default value, so nothing changes until a theme sets it. [stock only]

#### SS-2.15 Recolor the bar yourself

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: There is no settings page for status bar colors. A person changes them by adding any of the 35 `statusBar.*` and `statusBarItem.*` keys (32 core tokens plus the three debugging ones) to `workbench.colorCustomizations` in settings.json, or by switching the theme.
- Commands: workbench.action.selectTheme
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations; workbench.colorTheme
- Source: V\src\vs\workbench\common\theme.ts:284-385, :576-600 (core tokens); V\src\vs\workbench\contrib\debug\browser\statusbarColorProvider.ts:20-34; C bundle (`workbench.colorCustomizations`, `workbench.colorTheme` and `workbench.action.selectTheme` found, all 35 token names found)
- Pieces:
  1. Adding any of the 35 `statusBar.*` and `statusBarItem.*` keys, 32 core keys plus the three debugging keys, to `workbench.colorCustomizations` in settings.json recolors the bar.
  2. Switching the theme with Color Theme (`workbench.action.selectTheme`, setting `workbench.colorTheme`) recolors the bar.
  3. There is no settings page for status bar colors.


## SS-3 Current editor and language details

(both). Right-side items for the active editor: line and column, indentation, encoding, line ending, language, mode flags, notebook items, language status. This is the heaviest group and the first to split later.

53 features.

#### SS-3.1 Line and column readout

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The right side of the bar shows the cursor position of the active text editor as "Ln {n}, Col {n}"; the column counts a tab by its display width, and the item is hidden in notebook cell editors (id status.editor.selection, name "Editor Selection", priority 100.5).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts; C bundle (module vs/workbench/browser/parts/editor/editorStatus, id status.editor.selection, label "Ln {0}, Col {1}")
- Pieces:
  1. While a text editor is active the right side of the status bar shows "Ln {n}, Col {n}" for the cursor, and the numbers change as the cursor moves.
  2. A tab character counts by its display width in the column number, so with a tab size of 4 a cursor just after one leading tab reads Col 5.
  3. Hovering the item shows the tooltip "Go to Line/Column".
  4. The item is hidden while a notebook cell editor is the active editor.

#### SS-3.2 Selection size and multi-cursor count

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With text selected the same item reads "Ln {n}, Col {n} ({n} selected)", and with several cursors it reads "{n} selections" or "{n} selections ({n} characters selected)".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (getSelectionLabel); C bundle (editorStatus labels "Ln {0}, Col {1} ({2} selected)", "{0} selections ({1} characters selected)", "{0} selections")
- Pieces:
  1. With text selected under a single cursor the item reads "Ln {n}, Col {n} ({n} selected)", counting the selected characters.
  2. With several cursors and nothing selected the item reads "{n} selections".
  3. With several cursors and some text selected the item reads "{n} selections ({n} characters selected)".
  4. Clearing the selection or dropping back to one cursor returns the item to plain "Ln {n}, Col {n}".

#### SS-3.3 Click the position to go to a line or column

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking the line and column item (tooltip "Go to Line/Column") opens the Go to Line quick pick for the active editor, where the person types a line and optional column.
- Commands: workbench.action.gotoLine
- Menus: MenubarGoMenu
- Keys: Ctrl+G (mac Ctrl+G)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts; V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts; V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (Go menu row "Go to Line/Column..."); C bundle (workbench.action.gotoLine with the same Ctrl+G and mac Ctrl+G keys, label "Go to Line/Column...")
- Pieces:
  1. Clicking the line and column item opens the Go to Line quick pick for the active editor, where typing a line number, and optionally a column after a colon, moves the cursor there.
  2. Pressing `Ctrl+G` opens the same Go to Line quick pick (the key is `Ctrl+G` on macOS too).
  3. The Go menu row "Go to Line/Column..." and the command `workbench.action.gotoLine` from the Command Palette open the same quick pick.

#### SS-3.4 Indentation readout

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The item shows the indent style of the active text editor as "Spaces: {n}", "Spaces: {n} (Tab Size: {m})" or "Tab Size: {n}", with tooltip "Select Indentation"; it is hidden in notebook cell editors (id status.editor.indentation, name "Editor Indentation", priority 100.4).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.insertSpaces; editor.tabSize; editor.indentSize; editor.detectIndentation
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (onIndentationChange); V\src\vs\editor\common\config\editorConfigurationSchema.ts; C bundle (editorStatus labels "Spaces: {0}", "Spaces: {0} (Tab Size: {1})", "Tab Size: {0}", id status.editor.indentation, settings editor.insertSpaces, editor.tabSize, editor.indentSize, editor.detectIndentation)
- Pieces:
  1. For a file indented with spaces where the indent size equals the tab size the item reads "Spaces: {n}".
  2. For a file indented with spaces where the indent size differs from the tab size the item reads "Spaces: {n} (Tab Size: {m})".
  3. For a file indented with tabs the item reads "Tab Size: {n}".
  4. Hovering the item shows the tooltip "Select Indentation", and the item is hidden while a notebook cell editor is active.
  5. Setting `editor.insertSpaces` chooses whether Tab inserts spaces or a tab character in files that have no detected style, with values on and off.
  6. Settings `editor.tabSize` (a number from 1 to 100) and `editor.indentSize` (a number or `tabSize`) choose the starting tab width and indent width that the item shows.
  7. Setting `editor.detectIndentation` chooses whether the style and size are taken from the file content when it opens, with values on and off, and the item shows what was detected.

#### SS-3.5 Pick an indentation action from the status item

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking the indentation item opens a "Select Action" quick pick with a "change view" group (Indent Using Spaces, Indent Using Tabs, Change Tab Display Size) and a "convert file" group (Detect Indentation from Content, Convert Indentation to Spaces, Convert Indentation to Tabs, Trim Trailing Whitespace); it says "The active code editor is read-only." or "No text editor active at this time" when it cannot run.
- Commands: changeEditorIndentation{windowId}
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (showIndentationPicker, command registered per window id); C bundle (command id changeEditorIndentation, labels "change view", "convert file", "Select Action")
- Pieces:
  1. Clicking the indentation item opens a quick pick with the placeholder "Select Action" for the active editor.
  2. The first group is labelled "change view" and holds Indent Using Spaces, Indent Using Tabs and Change Tab Display Size.
  3. The second group is labelled "convert file" and holds Detect Indentation from Content, Convert Indentation to Spaces, Convert Indentation to Tabs and Trim Trailing Whitespace.
  4. Picking a row closes the quick pick, puts focus back in the editor and runs that action.
  5. When the active editor is read-only the quick pick shows one row, "The active code editor is read-only."
  6. When no text editor is active the quick pick shows one row, "No text editor active at this time".

#### SS-3.6 Switch a file to spaces or tabs and pick the size

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: From the indentation picker the person chooses Indent Using Spaces or Indent Using Tabs, then picks a size from 1 to 8 (rows say Configured, Default or Current Tab Size) and the file's indent style and size change for that file only; the same two editor actions are what ED-2.18 lists as commands.
- Commands: editor.action.indentUsingSpaces; editor.action.indentUsingTabs
- Menus: none found
- Keys: none found
- Settings: editor.insertSpaces; editor.tabSize; editor.indentSize
- Source: V\src\vs\editor\contrib\indentation\browser\indentation.ts (ChangeIndentationSizeAction, IndentUsingSpaces, IndentUsingTabs); V\src\vs\workbench\browser\parts\editor\editorStatus.ts; C bundle (editor.action.indentUsingSpaces, editor.action.indentUsingTabs)
- Pieces:
  1. Picking Indent Using Spaces opens a size list with the placeholder "Select Tab Size for Current File" and the rows 1 to 8.
  2. Picking Indent Using Tabs opens the same size list of 1 to 8.
  3. A size row is tagged "Configured Tab Size" when it matches both the configured and the current size, "Default Tab Size" when it matches only the configured one, and "Current Tab Size" when it matches only the current one.
  4. The list opens with the file's current tab size highlighted.
  5. Picking a size changes the tab size and indent size of that file and switches it to spaces or tabs as chosen, and the status bar indentation item updates at once.
  6. The change applies to that file only, and the `editor.insertSpaces`, `editor.tabSize` and `editor.indentSize` settings are left as they were.
  7. Running `editor.action.indentUsingSpaces` or `editor.action.indentUsingTabs` from the Command Palette opens the same size list.

#### SS-3.7 Change how wide a tab looks

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Change Tab Display Size row opens the 1 to 8 size list and changes only how many spaces a tab character is drawn as for that file, without changing the indent size or the indent style.
- Commands: editor.action.changeTabDisplaySize
- Menus: none found
- Keys: none found
- Settings: editor.tabSize
- Source: V\src\vs\editor\contrib\indentation\browser\indentation.ts (ChangeTabDisplaySize, displaySizeOnly); C bundle (editor.action.changeTabDisplaySize)
- Pieces:
  1. Picking Change Tab Display Size opens the same size list of 1 to 8 with the placeholder "Select Tab Size for Current File".
  2. Picking a size changes how many spaces wide a tab character is drawn in that file and nothing else.
  3. The indent size and the spaces or tabs style of the file stay as they were, and the status bar item shows the new tab size.
  4. Running `editor.action.changeTabDisplaySize` from the Command Palette opens the same list.

#### SS-3.8 Convert existing indentation to spaces or tabs

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Convert Indentation to Spaces and Convert Indentation to Tabs rows rewrite the leading whitespace of every line in the file to the other style and then switch the file's style to match, keeping the selection in place; they need a writable editor.
- Commands: editor.action.indentationToSpaces; editor.action.indentationToTabs
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\editor\contrib\indentation\browser\indentation.ts (IndentationToSpacesAction, IndentationToTabsAction, precondition writable); C bundle (editor.action.indentationToSpaces, editor.action.indentationToTabs)
- Pieces:
  1. Picking Convert Indentation to Spaces rewrites the leading whitespace of every line in the file as spaces and switches the file's style to spaces.
  2. Picking Convert Indentation to Tabs rewrites the leading whitespace of every line in the file as tabs and switches the file's style to tabs.
  3. The selection stays in place after either conversion.
  4. Both rows need a writable editor, and nothing happens in a read-only editor.
  5. Running `editor.action.indentationToSpaces` or `editor.action.indentationToTabs` from the Command Palette does the same conversion.

#### SS-3.9 Detect the indentation from the file content

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Detect Indentation from Content row looks at the lines in the file and sets the file's indent style and size to match them, falling back to the configured defaults when it cannot tell.
- Commands: editor.action.detectIndentation
- Menus: none found
- Keys: none found
- Settings: editor.detectIndentation
- Source: V\src\vs\editor\contrib\indentation\browser\indentation.ts (DetectIndentation); C bundle (editor.action.detectIndentation, label "Detect Indentation from Content")
- Pieces:
  1. Picking Detect Indentation from Content sets the file's indent style and size to match the indentation found in its lines, and the status bar item updates.
  2. When the content gives no clear answer the file takes the configured defaults for spaces or tabs and for size.
  3. Running `editor.action.detectIndentation` from the Command Palette does the same thing.
  4. Setting `editor.detectIndentation` chooses whether this detection also runs on its own when a file opens, with values on and off.

#### SS-3.10 Encoding readout

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The item shows the file encoding of the active file editor as a short label such as "UTF-8" or "UTF-8 with BOM" (tooltip "Select Encoding"); it only appears once the editor has a loaded model and an encoding-capable input (id status.editor.encoding, name "Editor Encoding", priority 100.3).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.encoding
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (onEncodingChange); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (id status.editor.encoding, labels "Editor Encoding", "Select Encoding", setting files.encoding)
- Pieces:
  1. While a file editor with a loaded model is active the item shows the file's encoding as a short label such as "UTF-8".
  2. A file that starts with a byte order mark shows "UTF-8 with BOM" in the item.
  3. Hovering the item shows the tooltip "Select Encoding".
  4. The item does not appear for editors that have no file encoding, such as a non-file editor, and it updates when the encoding changes.
  5. Setting `files.encoding` chooses the default encoding used to read and write files, default `utf8`, and it can be set per language.

#### SS-3.11 Open the encoding picker

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking the encoding item (or running Change File Encoding) asks "Select Action" with Reopen with Encoding or Save with Encoding; an untitled file goes straight to Save with Encoding, a read-only file goes straight to Reopen with Encoding, and a non-file editor says "No file active at this time".
- Commands: workbench.action.editor.changeEncoding
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (ChangeEncodingAction); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (registerAction2); C bundle (workbench.action.editor.changeEncoding, label "Change File Encoding", "Select Action")
- Pieces:
  1. Clicking the encoding item opens a quick pick with the placeholder "Select Action" offering Reopen with Encoding and Save with Encoding.
  2. Running Change File Encoding (`workbench.action.editor.changeEncoding`) from the Command Palette opens the same quick pick.
  3. In an untitled file the click skips the action quick pick and goes straight to the Save with Encoding list.
  4. In a read-only file the click skips the action quick pick and goes straight to the Reopen with Encoding list.
  5. In an editor that is not a file the quick pick shows one row, "No file active at this time", and with no text editor it shows "No text editor active at this time".

#### SS-3.12 Reopen a file with a different encoding

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - no discard-changes confirm for an editor with unsaved changes
- What: Reopen with Encoding lists every decodable encoding (the configured one first, plus a "Guessed from content" row at the top when detection finds a different one) and rereads the file with the pick; on a file with unsaved changes stock VS Code first asks "Do you want to revert the active text editor and reopen with a different encoding?" with a "Discard Changes and Reopen" button, and Cursor skips that question and goes straight to the change.
- Commands: workbench.action.editor.changeEncoding
- Menus: none found
- Keys: none found
- Settings: files.encoding; files.autoGuessEncoding; files.candidateGuessEncodings
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (reopenWithEncodingPick, reopenWithEncodingWarning); C bundle (module vs/workbench/browser/parts/editor/editorStatus keys reopenWithEncoding and guessedEncoding present, keys reopenWithEncodingWarning, reopenWithEncodingDetail and reopen absent from the NLS files, and the compiled pick handler calls setEncoding right after the pick)
- Pieces:
  1. Picking Reopen with Encoding opens a list with the placeholder "Select File Encoding to Reopen File" showing each encoding the file can be read with, with the encoding code beside its name.
  2. The encoding set in `files.encoding` is the first row and the file's current encoding starts highlighted.
  3. When content detection finds an encoding other than the configured one, a row for it tagged "Guessed from content" sits at the very top above a separator.
  4. Picking an encoding rereads the file with it and the status bar encoding item changes to match.
  5. Setting `files.autoGuessEncoding` chooses whether files try to guess their encoding when they open, with values on and off and a default of off.
  6. Setting `files.candidateGuessEncodings` lists, in order, the encodings the guess may choose from, and it starts empty.
  7. Stock VS Code asks "Do you want to revert the active text editor and reopen with a different encoding?" with the detail "This will discard any unsaved changes." and a "Discard Changes and Reopen" button when the file has unsaved changes. [stock only]
  8. Cursor skips that question and reopens the file with the picked encoding straight away, even with unsaved changes. [cursor only]

#### SS-3.13 Save a file with a different encoding

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Save with Encoding lists every encoding the file can be written in, including encode-only ones that Reopen hides, and saves the current text using the pick; the placeholder reads "Select File Encoding to Save with".
- Commands: workbench.action.editor.changeEncoding
- Menus: none found
- Keys: none found
- Settings: files.encoding
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (saveWithEncodingPick, EncodingMode.Encode); C bundle (labels "Save with Encoding", "Select File Encoding to Save with")
- Pieces:
  1. Picking Save with Encoding opens a list with the placeholder "Select File Encoding to Save with" and the file's current encoding highlighted.
  2. The list also holds the encodings that can only be written, which the Reopen list hides.
  3. Picking an encoding saves the current text in that encoding and the status bar encoding item changes to match.
  4. The encoding set in `files.encoding` is the first row of the list.

#### SS-3.14 End of line readout

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The item shows the line ending of the active text editor as "LF" or "CRLF" (tooltip "Select End of Line Sequence") and is hidden for read-only editors (id status.editor.eol, name "Editor End of Line", priority 100.2).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.eol
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (onEOLChange); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (id status.editor.eol, labels "LF", "CRLF", "Select End of Line Sequence", setting files.eol)
- Pieces:
  1. While a text editor is active the item shows the line ending as "LF" or "CRLF".
  2. Hovering the item shows the tooltip "Select End of Line Sequence".
  3. The item is hidden for a read-only editor.
  4. The item changes right after the line ending of the file is converted.
  5. Setting `files.eol` chooses the line ending for new files, with values `\n`, `\r\n` and `auto` (the system's own), default `auto`.

#### SS-3.15 Change the end of line sequence

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking the end of line item (or running Change End of Line Sequence) opens a LF and CRLF pick with the current one preselected and converts the whole file as one undo step; read-only editors show "The active code editor is read-only.".
- Commands: workbench.action.editor.changeEOL
- Menus: none found
- Keys: none found
- Settings: files.eol
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (ChangeEOLAction); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts; C bundle (workbench.action.editor.changeEOL, label "Change End of Line Sequence")
- Pieces:
  1. Clicking the end of line item opens a quick pick with the placeholder "Select End of Line Sequence" and the rows LF and CRLF, with the file's current one highlighted.
  2. Running Change End of Line Sequence (`workbench.action.editor.changeEOL`) from the Command Palette opens the same quick pick.
  3. Picking a row converts every line ending in the file at once, and one undo puts them all back.
  4. In a read-only editor the quick pick shows one row, "The active code editor is read-only."
  5. With no text editor active the quick pick shows one row, "No text editor active at this time".

#### SS-3.16 Language readout

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The item shows the language name of the active text editor, such as "TypeScript" or "Plain Text" (tooltip "Select Language Mode"); it appears for editors that support a language mode and not for binary or other non-text editors (id status.editor.mode, name "Editor Language", priority 100.1).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.associations
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (onLanguageChange); V\src\vs\platform\files\common\files.ts (FILES_ASSOCIATIONS_CONFIG); C bundle (id status.editor.mode, labels "Editor Language", "Select Language Mode", setting files.associations)
- Pieces:
  1. While a text editor is active the item shows the name of the file's language, such as "TypeScript" or "Plain Text".
  2. Hovering the item shows the tooltip "Select Language Mode".
  3. The item does not appear for binary files or other editors that have no language mode, and it changes when the language changes.
  4. Setting `files.associations` maps file name patterns to languages, so files matching a pattern show that language in the item.

#### SS-3.17 Change the language mode from a picker

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the bundle gives this chord its own mac keybinding entry that V does not have (value not decoded)
- What: Clicking the language item (or running Change Language Mode) opens "Select Language Mode" with every registered language, the current one marked "Configured Language", and sets the language of the active editor from the pick; the command also takes a language name as an argument, and the empty editor hint offers the same picker.
- Commands: workbench.action.editor.changeLanguageMode
- Menus: none found
- Keys: Ctrl+K M (not while a notebook editor has focus)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (ChangeLanguageAction); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts; V\src\vs\workbench\contrib\codeEditor\browser\emptyTextEditorHint\emptyTextEditorHint.ts; C bundle (workbench.action.editor.changeLanguageMode, label "Change Language Mode", keybinding block with a separate mac entry)
- Pieces:
  1. Clicking the language item opens a quick pick with the placeholder "Select Language Mode" listing every registered language, sorted by name, with its identifier in brackets under a "languages (identifier)" heading.
  2. The row of the file's current language is marked "Configured Language".
  3. Picking a language switches the active editor to it, the status item shows the new name, and focus returns to the editor.
  4. Pressing `Ctrl+K` then `M` opens the same quick pick, except while a notebook editor has focus.
  5. Running Change Language Mode (`workbench.action.editor.changeLanguageMode`) from the Command Palette opens the same quick pick.
  6. The "select a language" link in the hint shown in an empty editor opens the same quick pick.
  7. Running the command with a language name as its argument switches to that language without showing the quick pick.
  8. With no text editor active the quick pick shows one row, "No text editor active at this time".
  9. Cursor registers this chord with its own separate macOS keybinding entry that stock VS Code does not have. [cursor only]

#### SS-3.18 Auto Detect the language from the picker

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the file is not empty the language picker starts with an "Auto Detect" row that first guesses from the file name or first line and, if that gives nothing, asks the language detection model to guess from the content.
- Commands: workbench.action.editor.changeLanguageMode
- Menus: none found
- Keys: none found
- Settings: workbench.editor.languageDetection; workbench.editor.historyBasedLanguageDetection; workbench.editor.preferHistoryBasedLanguageDetection
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (autoDetectLanguage); V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (label "Auto Detect", settings workbench.editor.languageDetection, workbench.editor.historyBasedLanguageDetection, workbench.editor.preferHistoryBasedLanguageDetection)
- Pieces:
  1. When the file has any content the language quick pick starts with an "Auto Detect" row, and in an empty file the row is missing.
  2. Picking Auto Detect first sets the language from the file name or the first line when that identifies one.
  3. When the file name and first line identify nothing, Auto Detect sets the language from a guess made on the file content.
  4. When no guess comes back Auto Detect leaves the language as it was.
  5. Setting `workbench.editor.languageDetection` chooses whether the language of a text editor is detected on its own unless it was set from the picker, with values on and off, default on, and it can be set per language.
  6. Setting `workbench.editor.historyBasedLanguageDetection` chooses whether recently opened languages are favored when detecting, default on.
  7. Setting `workbench.editor.preferHistoryBasedLanguageDetection` chooses whether the history-aware detection takes precedence, default off.

#### SS-3.19 Detect the language from the content with a command

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Detect Language from Content command sets the language of the active text editor to what the detection model guesses, or raises a warning "Unable to detect editor language" when it cannot; it does not run in an editable notebook.
- Commands: editor.detectLanguage
- Menus: none found
- Keys: Alt+Shift+D (editor text focus)
- Settings: workbench.editor.languageDetection
- Source: V\src\vs\workbench\contrib\languageDetection\browser\languageDetection.contribution.ts; C bundle (editor.detectLanguage, label "Detect Language from Content", same Alt+Shift+D key)
- Pieces:
  1. Running Detect Language from Content (`editor.detectLanguage`) from the Command Palette sets the active text editor's language to the detected one and the language item updates.
  2. Pressing `Alt+Shift+D` with focus in the editor text does the same.
  3. When nothing can be detected a warning notification reads "Unable to detect editor language" and the language stays as it was.
  4. The command does not run in an editable notebook.

#### SS-3.20 Accept the detected language with the lightbulb

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In an untitled editor whose detected language differs from the current one, a lightbulb item appears next to the language item (tooltip "Accept Detected Language: {name}"); clicking it switches the editor to the detected language (id status.languageDetectionStatus, name "Language Detection").
- Commands: editor.detectLanguage
- Menus: none found
- Keys: Alt+Shift+D (shown in the tooltip)
- Settings: workbench.editor.languageDetectionHints; workbench.editor.languageDetection
- Source: V\src\vs\workbench\contrib\languageDetection\browser\languageDetection.contribution.ts (LanguageDetectionStatusContribution, compact pair next to status.editor.mode); C bundle (id status.languageDetectionStatus, label "Accept Detected Language: {0}", setting workbench.editor.languageDetectionHints)
- Pieces:
  1. In an untitled editor whose content looks like a different language than the current one, a lightbulb item appears right next to the language item.
  2. Hovering the lightbulb shows "Accept Detected Language: {name}" with the `Alt+Shift+D` key added.
  3. Clicking the lightbulb switches the editor to the detected language and the lightbulb goes away.
  4. The lightbulb is not shown in saved files, and it goes away when the detected language matches the current one.
  5. Setting `workbench.editor.languageDetectionHints` chooses where the lightbulb can show, with the values `untitledEditors` and `notebookEditors` each on or off and both on by default.

#### SS-3.21 Configure a file association for a file type

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: For a file that has a path the language picker offers "Configure File Association for '{ext}'..."; picking a language writes it into files.associations as `*.ext` (or the whole file name for dotfiles and names without an extension), in user settings unless the workspace already sets that association.
- Commands: workbench.action.editor.changeLanguageMode
- Menus: none found
- Keys: none found
- Settings: files.associations
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (configureFileAssociation); C bundle (labels "Configure File Association for '{0}'...", "Select Language Mode to Associate with '{0}'", "Current Association", setting files.associations)
- Pieces:
  1. For a file that has a path the language quick pick offers a row "Configure File Association for '{ext}'..." near the top, and an untitled file with no saved path does not get it.
  2. Picking that row opens a second list with the placeholder "Select Language Mode to Associate with '{ext}'" holding every language, and the language the extension already maps to is tagged "Current Association".
  3. Picking a language writes `*.{ext}` into `files.associations`, so every file with that extension now opens in the chosen language.
  4. For dotfiles and for names with no extension, such as `.gitignore` or `Dockerfile`, the whole file name is written instead of `*.{ext}`.
  5. The association is saved to user settings, unless the workspace settings already set that same pattern, in which case it is saved to workspace settings.
  6. Setting `files.associations` holds the saved patterns, and the language item of matching files shows the chosen language.

#### SS-3.22 Open language based settings from the picker

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: For a file that has a path the language picker offers "Configure '{language}' language based settings..." which opens the user settings file in JSON at the `[language]` block, ready to edit.
- Commands: workbench.action.editor.changeLanguageMode
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (configureLanguageSettings, openUserSettings); C bundle (label "Configure '{0}' language based settings...")
- Pieces:
  1. For a file that has a path the language quick pick offers a row "Configure '{language}' language based settings..." naming the file's current language.
  2. Picking that row opens the user settings file as JSON, scrolled to the `[language]` block for that language and ready to edit.
  3. An untitled file with no saved path does not get this row.

#### SS-3.23 OVR overtype indicator and toggle

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While overtype mode is on a prominent "OVR" item appears (tooltip "Enable Insert Mode") and clicking it, or pressing Insert, switches back to insert mode; the same command switches into overtype (id status.editor.inputMode, priority 100.6).
- Commands: editor.action.toggleOvertypeInsertMode
- Menus: none found
- Keys: Insert (mac Alt+Cmd+O)
- Settings: editor.overtypeCursorStyle; editor.overtypeOnPaste
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (StatusInputMode, updateInputModeElement); V\src\vs\workbench\contrib\codeEditor\browser\toggleOvertype.ts; V\src\vs\editor\common\config\editorOptions.ts; C bundle (id status.editor.inputMode, command editor.action.toggleOvertypeInsertMode with the same Insert and mac Alt+Cmd+O keys, label "OVR", option overtypeOnPaste)
- Pieces:
  1. While overtype mode is on a prominent "OVR" item appears in the status bar and typed characters replace the characters after the cursor instead of pushing them along.
  2. Hovering the item shows the tooltip "Enable Insert Mode", and clicking it switches back to insert mode and makes the item disappear.
  3. Pressing `Insert` (on macOS `Alt+Cmd+O`) runs Toggle Overtype/Insert Mode, which switches between the two modes and shows or hides the item.
  4. Running Toggle Overtype/Insert Mode (`editor.action.toggleOvertypeInsertMode`) from the Command Palette does the same switch.
  5. Setting `editor.overtypeCursorStyle` chooses the cursor shape in overtype mode, with values `line`, `block`, `underline`, `line-thin`, `block-outline` and `underline-thin`, default `block`.
  6. Setting `editor.overtypeOnPaste` chooses whether pasting overwrites the characters after the cursor in overtype mode, with values on and off, default on.

#### SS-3.24 Tab Moves Focus indicator and toggle

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the Tab key is set to move focus instead of typing a tab, a prominent "Tab Moves Focus" item appears (tooltip "Disable Accessibility Mode"); clicking it or running Toggle Tab Key Moves Focus switches back and announces the new state to screen readers (id status.editor.tabFocusMode, name "Accessibility Mode", priority 100.7).
- Commands: editor.action.toggleTabFocusMode
- Menus: none found
- Keys: Ctrl+M (mac Ctrl+Shift+M)
- Settings: editor.tabFocusMode
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (TabFocusMode, updateTabFocusModeElement); V\src\vs\editor\contrib\toggleTabFocusMode\browser\toggleTabFocusMode.ts; V\src\vs\editor\common\config\editorOptions.ts; C bundle (id status.editor.tabFocusMode, command editor.action.toggleTabFocusMode with the same Ctrl+M and mac Ctrl+Shift+M keys, labels "Tab Moves Focus", "Accessibility Mode")
- Pieces:
  1. While the Tab key is set to move focus instead of typing a tab, a prominent "Tab Moves Focus" item appears in the status bar.
  2. Hovering the item shows the tooltip "Disable Accessibility Mode", and clicking it makes Tab type a tab character again and removes the item.
  3. Pressing `Ctrl+M` (on macOS `Ctrl+Shift+M`) runs Toggle Tab Key Moves Focus, which turns the mode on or off and shows or hides the item.
  4. Running Toggle Tab Key Moves Focus (`editor.action.toggleTabFocusMode`) from the Command Palette does the same switch.
  5. A screen reader announces "Pressing Tab will now move focus to the next focusable element" when the mode turns on and "Pressing Tab will now insert the tab character" when it turns off.
  6. Setting `editor.tabFocusMode` chooses whether the editor keeps Tab keystrokes or hands them to the workbench for moving focus, with values on and off, default off.

#### SS-3.25 Column Selection indicator and toggle

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While column selection mode is on a prominent "Column Selection" item appears (tooltip "Disable Column Selection Mode") and clicking it turns the mode off; the Selection menu row "Column Selection Mode" (checked while on) and the command do the same (id status.editor.columnSelectionMode, priority 100.8).
- Commands: editor.action.toggleColumnSelection
- Menus: MenubarSelectionMenu
- Keys: none found
- Settings: editor.columnSelection
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (updateColumnSelectionModeElement); V\src\vs\workbench\contrib\codeEditor\browser\toggleColumnSelection.ts (menu group 4_config, order 2); C bundle (id status.editor.columnSelectionMode, editor.action.toggleColumnSelection with menu MenubarSelectionMenu, labels "Column Selection", "Toggle Column Selection Mode")
- Pieces:
  1. While column selection mode is on a prominent "Column Selection" item appears in the status bar and selecting with the mouse and keys makes rectangular selections.
  2. Hovering the item shows the tooltip "Disable Column Selection Mode", and clicking it turns the mode off and removes the item.
  3. The Selection menu row "Column Selection Mode" shows a check while the mode is on, and clicking it turns the mode on or off.
  4. Running Toggle Column Selection Mode (`editor.action.toggleColumnSelection`) from the Command Palette does the same switch.
  5. Setting `editor.columnSelection` chooses whether selecting with the mouse and keys does column selection, with values on and off, default off.

#### SS-3.26 Screen Reader Optimized indicator

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the editor runs in screen reader optimized mode a prominent "Screen Reader Optimized" item appears in every window; clicking it reopens the "Screen reader usage detected" prompt with Yes, No and Learn More, and Yes or No writes editor.accessibilitySupport to on or off (id status.editor.screenReaderMode, name "Screen Reader Mode", priority 100.6).
- Commands: showEditorScreenReaderNotification
- Menus: none found
- Keys: none found
- Settings: editor.accessibilitySupport; editor.accessibilityPageSize
- Source: V\src\vs\workbench\contrib\accessibility\browser\accessibilityStatus.ts; C bundle (module vs/workbench/contrib/accessibility/browser/accessibilityStatus, id status.editor.screenReaderMode, command showEditorScreenReaderNotification, label "Screen Reader Optimized")
- Pieces:
  1. While the editor runs in screen reader optimized mode a prominent "Screen Reader Optimized" item appears in the status bar of every window.
  2. Clicking the item opens the "Screen reader usage detected" prompt with the buttons Yes, No and Learn More.
  3. Choosing Yes writes `on` to `editor.accessibilitySupport` and choosing No writes `off`.
  4. Choosing Learn More opens the screen reader section of the accessibility documentation page.
  5. Setting `editor.accessibilitySupport` chooses the mode, with values `auto` (detect a screen reader from the platform), `on` (optimize for a screen reader) and `off` (assume none is attached), default `auto`.

#### SS-3.27 Scrolling Locked indicator and toggle

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While scrolling is synchronized across editors a prominent "Scrolling Locked" item appears in every window (tooltip "Lock Scrolling Enabled", priority 102) and clicking it switches the lock off; the commands switch it on, or hold it only while their key is down.
- Commands: workbench.action.toggleLockedScrolling; workbench.action.holdLockedScrolling
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\scrollLocking\browser\scrollLocking.ts (id status.scrollLockingEnabled); C bundle (id status.scrollLockingEnabled, commands workbench.action.toggleLockedScrolling and workbench.action.holdLockedScrolling, labels "Scrolling Locked", "Toggle Locked Scrolling Across Editors")
- Pieces:
  1. While scrolling is locked across editors a prominent "Scrolling Locked" item appears in the status bar of every window, and scrolling one editor scrolls the others along with it.
  2. Hovering the item shows the tooltip "Lock Scrolling Enabled", and clicking it unlocks scrolling and removes the item.
  3. Running Toggle Locked Scrolling Across Editors (`workbench.action.toggleLockedScrolling`) from the Command Palette locks scrolling, and running it again unlocks it.
  4. Running Hold Locked Scrolling Across Editors (`workbench.action.holdLockedScrolling`) from a key keeps scrolling locked only while that key is held down and unlocks it on release.

#### SS-3.28 File Information for binary files

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a binary file is open in the binary file viewer (or its side by side diff), an item shows the file size as text such as "1.2 KB" (tooltip "File Information"); nothing shows for text editors (id status.editor.info, priority 100).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (onMetadataChange); V\src\vs\workbench\browser\parts\editor\binaryEditor.ts (getMetadata from ByteSize.formatSize); C bundle (id status.editor.info, label "File Information")
- Pieces:
  1. While a binary file is open in the binary file viewer the status bar shows its size as text such as "1.2 KB".
  2. Hovering the item shows the tooltip "File Information".
  3. The item also shows when the binary file is one side of a side by side diff.
  4. The item does not show while a text editor is active.

#### SS-3.29 Language status item and its hover list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Left of the language item a single bracket icon (a plain bracket, a bracket with a dot for a warning, a bracket with an error mark for an error, or a spinner while something is busy) stands for every language status row of the active file; hover or click opens a list where each row has a severity icon, label, detail text with links, an optional command link and a pin button, and it sits as a compact pair with the language item (id status.languageStatus, name "Editor Language Status"). Rows come from core (see SS-3.31) and from built-in language extensions such as the TypeScript version and IntelliSense rows, the JSON schema and outline rows and the Markdown link validation row.
- Commands: statusBar.entry.showTooltip
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts; V\src\vs\workbench\services\statusbar\browser\statusbar.ts (ShowTooltipCommand); V\extensions\typescript-language-features\src\ui\versionStatus.ts; V\extensions\json-language-features\client\src\languageStatus.ts; V\extensions\markdown-language-features\src\languageFeatures\diagnostics.ts; C bundle (module vs/workbench/contrib/languageStatus/browser/languageStatus, id status.languageStatus, label "Editor Language Status", language status ids typescript.version, typescript.projectStatus, json.projectStatus, json.documentSymbolsStatus and markdownStatus in the built-in extension bundles)
- Pieces:
  1. In a file that has language status rows a single bracket icon item sits left of the language item and is joined to it as a pair, and it is absent when the file has no rows.
  2. The icon is a plain bracket normally, a bracket with a dot when the top row is a warning, a bracket with an error mark when the top row is an error, and a spinner while any row that is not pinned is busy.
  3. Hovering the icon opens a list of the rows, and clicking it runs `statusBar.entry.showTooltip` to open the same list.
  4. Each row shows a label, detail text that can hold links, an optional command link on the right and a pin button, and its severity icon appears only when the top row is a warning or error.
  5. The icon and the list change when the active editor or its language changes, listing only the rows of the file now in front.
  6. A screen reader reads the item as "Editor Language Status:" followed by each row's label and detail.

#### SS-3.30 Pin a language status row to the status bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The pin button on a row ("Add to Status Bar") makes that row its own item right of the language item with its short label, tooltip and command, colored as warning or error by severity; the button then reads "Remove from Status Bar", and hiding that item from the bar's right-click list also unpins it (remembered per profile).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts (_dedicated, storage key languageStatus.dedicated, _asStatusbarEntry); C bundle (labels "Add to Status Bar", "Remove from Status Bar", "{0} (Language Status)")
- Pieces:
  1. Hovering a row's pin button shows "Add to Status Bar", and clicking it makes that row its own status bar item right of the language item.
  2. The pinned item shows the row's short label, its tooltip is the row's command tooltip or its detail text, and clicking it runs the row's command.
  3. A pinned warning row is drawn in the warning color and a pinned error row in the error color.
  4. After pinning, the row's button reads "Remove from Status Bar" with a filled pin, and clicking it puts the row back into the bracket item only.
  5. Hiding the pinned item from the status bar's right-click list unpins the row, and that list names it "{name} (Language Status)".
  6. Pinned rows are remembered per profile and come back when the editor is reopened.

#### SS-3.31 Core rows in the language status list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Core adds its own rows: "Formatter Conflicts" (error, label "Formatting", with a "Configure..." link) when a language has several formatters and no default or the chosen one cannot format it, and "Folding Status" or "Color Decorator Status" saying "only {n} shown for performance reasons" with a Configure link that opens the setting.
- Commands: workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: editor.defaultFormatter; editor.foldingMaximumRegions; editor.colorDecoratorsLimit
- Source: V\src\vs\workbench\contrib\format\browser\formatActionsMultiple.ts (id formatter.conflict); V\src\vs\workbench\contrib\limitIndicator\browser\limitIndicator.contribution.ts (ids foldingLimitInfo, decoratorsLimitInfo); C bundle (id formatter.conflict, ids foldingLimitInfo and decoratorsLimitInfo, labels "Formatter Conflicts", "Folding Status", "Color Decorator Status", settings editor.foldingMaximumRegions, editor.colorDecoratorsLimit)
- Pieces:
  1. When a language has several formatters and no default, or the chosen default cannot format the file, the list gets an error row "Formatting" with a detail explaining the conflict and a "Configure..." link.
  2. Clicking "Configure..." on that row opens a pick "Select a default formatter for '{language}' files" and the choice is saved as `editor.defaultFormatter`.
  3. When a file has more foldable regions than the limit the list gets a warning row "Folding ranges" saying "only {n} shown for performance reasons" with a "Configure" link that opens Settings on `editor.foldingMaximumRegions`.
  4. When a file has more color decorators than the limit the list gets a warning row "Color decorators" saying "only {n} shown for performance reasons" with a "Configure" link that opens Settings on `editor.colorDecoratorsLimit`.
  5. Each limit row disappears once the file is no longer over its limit.

#### SS-3.32 Language status attention cues and the reset command

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The bracket icon wiggles when the status changes until the person has opened its hover three times, and the whole item flashes when the top row is a warning or error; the command "Reset Language Status Interaction Counter" (View category, id spelled editor.inlayHints.Reset) starts the wiggle teaching over.
- Commands: editor.inlayHints.Reset
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts (StoredCounter, wiggle and flash, ResetAction); C bundle (editor.inlayHints.Reset, label "Reset Language Status Interaction Counter")
- Pieces:
  1. The bracket icon wiggles each time the language status changes, and keeps doing so until the person has opened its hover three times.
  2. When the top row is a warning or error the icon wiggles and the whole item flashes on every change, even after the person has learned the hover.
  3. While the icon shows a spinner it neither wiggles nor flashes.
  4. Running Reset Language Status Interaction Counter (`editor.inlayHints.Reset`, in the View category) from the Command Palette makes the icon wiggle again until the hover has been opened three more times.

#### SS-3.33 Notebook kernel item

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a notebook the right side shows the selected or only suggested kernel as "{kernel name}" with a kernel icon (a suggested one says "(suggestion)" in its tooltip and is selected for real once the person edits or hovers), or a prominent "Select Kernel" when several kernels exist and none is chosen; clicking runs the kernel picker, and nothing shows when no kernel exists or while the notebook toolbar already carries the kernel button (setting notebook.globalToolbar, default on) (ids _notebook.selectKernel, priority 10, names "Notebook Kernel Info" and "Notebook Kernel Selection"). The picker itself is ED-7.19.
- Commands: _notebook.selectKernel
- Menus: none found
- Keys: none found
- Settings: notebook.globalToolbar
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\editorStatusBar\editorStatusBar.ts (KernelStatus, ImplictKernelSelector); V\src\vs\workbench\contrib\notebook\browser\viewParts\notebookKernelView.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts (notebook.globalToolbar default true); C bundle (_notebook.selectKernel, labels "Notebook Kernel Info", "Notebook Kernel Selection", "Select Kernel", "{0} (suggestion)", setting notebook.globalToolbar)
- Pieces:
  1. In a notebook with a selected kernel the right side shows a kernel icon and the kernel's name, hovering shows the kernel's description, detail or name, and clicking it opens the kernel picker (`_notebook.selectKernel`).
  2. In a notebook with no selected kernel but exactly one suggested kernel the item shows that kernel's name, and its tooltip ends with "(suggestion)".
  3. A suggested kernel becomes the selected kernel once the person edits a cell, moves cells, changes a cell's language or hovers over cell text.
  4. In a notebook with several kernels and none chosen a prominent "Select Kernel" item appears, and clicking it opens the kernel picker.
  5. In a notebook that has no kernel at all no kernel item shows.
  6. While the notebook toolbar already has its own kernel button no kernel item shows in the status bar.
  7. Setting `notebook.globalToolbar` chooses whether the notebook toolbar is shown, with values on and off and a default of on, so the status bar kernel item appears only when it is off.

#### SS-3.34 Notebook active cell indicator

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a notebook the item shows "Cell {n} of {total}" for the focused cell, or "Cell {n} ({m} selected)" when several cells are selected, and clicking it scrolls the active cell to the middle of the editor (id notebook.activeCellStatus, name "Notebook Editor Selections", priority 100).
- Commands: notebook.centerActiveCell
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\editorStatusBar\editorStatusBar.ts (ActiveCellStatus); V\src\vs\workbench\contrib\notebook\browser\contrib\navigation\arrow.ts (CENTER_ACTIVE_CELL); C bundle (id notebook.activeCellStatus, command notebook.centerActiveCell, labels "Cell {0} of {1}", "Cell {0} ({1} selected)")
- Pieces:
  1. While a notebook cell is focused the item shows "Cell {n} of {total}".
  2. When several cells are selected the item shows "Cell {n} ({m} selected)".
  3. Clicking the item scrolls the active cell to the middle of the notebook editor.
  4. The item does not show when no notebook editor is active or when no cell is active.

#### SS-3.35 Notebook indentation item

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a notebook the indentation item shows the active cell's "Spaces: {n}" or "Tab Size: {n}" (tooltip "Select Indentation") and clicking it opens a notebook version of the indentation picker with Indent Using Tabs, Indent Using Spaces, Change Tab Display Size and the two Convert Indentation rows, applied to the notebook's cells; it needs an editable notebook and cell (id notebook.status.indentation, name "Notebook Indentation", priority 100.4).
- Commands: notebook.selectIndentation; notebook.action.indentUsingTabs; notebook.action.indentUsingSpaces; notebook.action.changeTabDisplaySize; notebook.action.convertIndentationToSpaces; notebook.action.convertIndentationToTabs
- Menus: none found
- Keys: none found
- Settings: notebook.editorOptionsCustomizations; editor.insertSpaces; editor.tabSize; editor.indentSize
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\editorStatusBar\editorStatusBar.ts (NotebookIndentationStatus); V\src\vs\workbench\contrib\notebook\browser\controller\editActions.ts (SelectNotebookIndentation); V\src\vs\workbench\contrib\notebook\browser\controller\notebookIndentationActions.ts; C bundle (id notebook.status.indentation, commands notebook.selectIndentation and the five notebook.action indentation ids, setting notebook.editorOptionsCustomizations)
- Pieces:
  1. In a notebook the item shows the active cell's indentation as "Spaces: {n}" or "Tab Size: {n}", hovering shows "Select Indentation", and it changes as the active cell changes.
  2. Values set in `notebook.editorOptionsCustomizations` for `editor.insertSpaces`, `editor.tabSize` and `editor.indentSize` take precedence over the cell's own values in what the item shows.
  3. Settings `editor.insertSpaces`, `editor.tabSize` and `editor.indentSize` give the starting indentation that cells show when no notebook override is set.
  4. Clicking the item opens a quick pick with the placeholder "Select Action" with a "change view" group (Indent Using Tabs, Indent Using Spaces, Change Tab Display Size) and a "convert file" group (Convert Indentation to Tabs, Convert Indentation to Spaces).
  5. Picking an Indent Using row or Change Tab Display Size opens the size list of 1 to 8 with the placeholder "Select Tab Size for Current File" and applies the pick to the notebook's cells.
  6. Picking a Convert Indentation row rewrites the leading whitespace in the notebook's cells in one edit labelled "Convert Indentation".
  7. In a read-only notebook the quick pick shows one row, "The active notebook editor is read-only.", and with no notebook active it shows "No notebook editor active at this time".
  8. Running Select Indentation (`notebook.selectIndentation`) from the Command Palette opens the same quick pick, and it needs an editable notebook with an editable cell.

#### SS-3.36 Performance Trace recording item

- Tag: vscode. Evidence: source. Boundary: no. Cursor: changed - no status item and no Start Tracing or Start Heap Tracing; only Stop Tracing remains, in an older form that offers to relaunch with the trace launch argument
- What: After Developer: Start Tracing or Start Heap Tracing a red "Recording trace (click to stop)" item appears at the far left end of the left side (tooltip "Click to stop recording"); clicking it runs Stop Tracing, which shows a "Creating trace file..." progress dialog and writes the trace (id status.tracing, name "Performance Trace", kind error, desktop only).
- Commands: workbench.action.startTracing; workbench.action.startHeapTracing; workbench.action.stopTracing
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts (startTracingSession, StartTracing, StartHeapTracing, StopTracing); C bundle (workbench.action.stopTracing in module vs/workbench/contrib/issue/electron-sandbox/process.contribution with keys stopTracing.message (asks to relaunch with the trace launch argument), stopTracing.button and stopTracing.title; 0 hits for status.tracing, workbench.action.startTracing, workbench.action.startHeapTracing, "Recording trace (click to stop)", "Performance Trace")
- Pieces:
  1. Running Developer: Start Tracing (`workbench.action.startTracing`) from the Command Palette starts a performance trace and puts a red "Recording trace (click to stop)" item with a record icon at the far left end of the left side of the status bar, in the desktop app only. [stock only]
  2. Hovering the red item shows the tooltip "Click to stop recording". [stock only]
  3. Running Developer: Start Heap Tracing (`workbench.action.startHeapTracing`) starts a trace that also records memory and shows the same red item. [stock only]
  4. Clicking the red item runs Stop Tracing, which shows a progress dialog "Creating trace file..." with the detail "This can take up to one minute to complete", writes the trace and removes the item. [stock only]
  5. Running Developer: Stop Tracing (`workbench.action.stopTracing`) when no trace is running asks whether to relaunch with the trace launch argument, with a "Relaunch and Enable Tracing" button in stock VS Code.
  6. In Cursor there is no red recording item and no Start Tracing or Start Heap Tracing command, and Developer: Stop Tracing is the only tracing command left, in an older form that offers to relaunch with the trace launch argument. [cursor only]

#### SS-3.37 Metered Connection item

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: When the network is treated as metered an icon-only radio tower item appears at the far right end of the right side in every window (tooltip says updates, Settings Sync and automatic Git operations are paused); clicking it opens the Configure Metered Connection pick of Auto, On and Off (id workbench.contrib.meteredConnectionStatus, name "Metered Connection").
- Commands: workbench.action.configureMeteredConnection
- Menus: none found
- Keys: none found
- Settings: network.meteredConnection
- Source: V\src\vs\workbench\contrib\meteredConnection\browser\meteredConnectionStatus.ts; V\src\vs\workbench\contrib\meteredConnection\browser\meteredConnection.contribution.ts; V\src\vs\platform\meteredConnection\common\meteredConnection.config.contribution.ts; C bundle (0 hits for workbench.action.configureMeteredConnection, network.meteredConnection and the label "Metered Connection")
- Pieces:
  1. When the network is treated as metered an icon-only radio tower item appears at the far right end of the right side of the status bar in every window. [stock only]
  2. Hovering the item shows "Metered connection enabled. Some automatic features like extension updates, Settings Sync, and automatic Git operations are paused to reduce data usage." [stock only]
  3. Clicking the item opens a quick pick with the placeholder "Select Metered Connection Mode" holding Auto (detect automatically), On (always treat as metered) and Off (never treat as metered). [stock only]
  4. Running Configure Metered Connection (`workbench.action.configureMeteredConnection`) from the Command Palette opens the same quick pick. [stock only]
  5. The item disappears when the connection stops being metered. [stock only]
  6. Setting `network.meteredConnection` chooses how the connection is judged, with values `auto`, `on` and `off`, default `auto`. [stock only]

#### SS-3.38 Keyboard Layout item

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: In the web build only, an item "Layout: {name}" shows the detected keyboard layout on the right side and clicking it opens Change Keyboard Layout, with Auto Detect, a list of layouts and a Configure Keyboard Layout row that opens a keyboard layout file (id status.workbench.keyboardLayout, name "Keyboard Layout"); the desktop build never loads it.
- Commands: workbench.action.openKeyboardLayoutPicker
- Menus: none found
- Keys: none found
- Settings: keyboard.layout
- Source: V\src\vs\workbench\contrib\preferences\browser\keyboardLayoutPicker.ts; V\src\vs\workbench\workbench.web.main.ts (the only import, none in workbench.desktop.main.ts); V\src\vs\workbench\contrib\preferences\common\preferences.ts (KEYBOARD_LAYOUT_OPEN_PICKER); C bundle (0 hits for status.workbench.keyboardLayout, workbench.action.openKeyboardLayoutPicker and the labels "Layout: {0}" and "Change Keyboard Layout")
- Pieces:
  1. In the web build only, an item "Layout: {name}" shows the detected keyboard layout on the right side of the status bar. [stock only]
  2. Clicking the item, or running Change Keyboard Layout (`workbench.action.openKeyboardLayoutPicker`) from the Command Palette, opens a quick pick with the placeholder "Select Keyboard Layout". [stock only]
  3. The quick pick starts with an Auto Detect row and a Configure Keyboard Layout row, then lists the layouts sorted by name under a "Keyboard Layouts ({platform})" heading, with the current one marked "(Current layout)". [stock only]
  4. Picking a layout switches to it and the item text changes, and picking Auto Detect goes back to detecting the layout on its own. [stock only]
  5. Picking Configure Keyboard Layout opens the keyboard layout file as JSON with comments, creating it with a short instruction comment first when it does not exist. [stock only]

#### SS-3.39 Rendered Diff warning item

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: While a rendered Markdown diff preview is the active editor a warning-colored "Rendered Diff" item appears at priority 100 on the right (tooltip says rendered diffs may hide important changes); clicking it shows a warning that offers "Don't Show Again" (remembered per workspace), and the same warning shows once on first use (id markdown.renderedDiffWarning, name "Rendered Markdown Diff Warning").
- Commands: _markdown.preview.showRenderedDiffWarning
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\extensions\markdown-language-features\src\preview\renderedDiffWarning.ts; V\extensions\markdown-language-features\src\preview\previewManager.ts; C bundle (0 hits for markdown.renderedDiffWarning, renderedDiffWarning and "Rendered Diff" in the workbench bundle and in the markdown-language-features dist and package files)
- Pieces:
  1. While a rendered Markdown diff preview is the active editor a warning-colored item with a warning icon and the text "Rendered Diff" appears on the right of the status bar. [stock only]
  2. Hovering the item shows "Rendered Markdown diffs may hide important changes. Click for details." [stock only]
  3. Clicking the item shows a warning message saying rendered diffs may hide changes such as formatting, whitespace, links or HTML and to switch to the text diff to review them, with a "Don't Show Again" button. [stock only]
  4. The same warning message shows once on its own the first time a rendered diff preview becomes active in that workspace. [stock only]
  5. Choosing "Don't Show Again" removes the item and stops the warning for that workspace. [stock only]
  6. The item goes away when another editor becomes active. [stock only]

#### SS-3.40 Image Zoom item in the Image Preview

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While an image is the active Image Preview editor a right-side item shows the zoom level as a percent such as "100%", or "Whole Image" when the picture is fit to the window (id status.imagePreview.zoom, name "Image Zoom", priority 102); clicking it opens "Select zoom level" with 1000%, 500%, 200%, 100%, 50%, 20% and Whole Image and the preview rescales to the pick, and the item hides when another editor is active. ED-10.1 covers the zoom gestures themselves.
- Commands: _imagePreview.selectZoomLevel
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\extensions\media-preview\src\imagePreview\zoomStatusBarEntry.ts; V\extensions\media-preview\src\imagePreview\index.ts (updateState shows and hides the entry); V\extensions\media-preview\src\ownedStatusBarEntry.ts; C\extensions\media-preview\dist\extension.js (id status.imagePreview.zoom at priority 102, command _imagePreview.selectZoomLevel, scale list 10, 5, 2, 1, .5, .2, fit, labels "Select zoom level" and "Whole Image")
- Pieces:
  1. While an image is the active Image Preview editor a right-side item shows the zoom level as a percent such as "100%", or "Whole Image" when the picture fits the window.
  2. Clicking the item opens a quick pick with the placeholder "Select zoom level" and the rows 1000%, 500%, 200%, 100%, 50%, 20% and Whole Image.
  3. Picking a row rescales the image in the preview and the item shows the new level.
  4. The item sits left of the Image Size item and hides when another editor is active.

#### SS-3.41 Image Size item in the Image Preview

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While an image is the active Image Preview editor a right-side item shows the picture's own pixel size as "{width}x{height}", for example "1920x1080" (id status.imagePreview.size, name "Image Size", priority 101, just left of the binary size item); it has no click action, it is empty until the image has loaded, and it hides when the preview is not the active editor.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\extensions\media-preview\src\imagePreview\sizeStatusBarEntry.ts; V\extensions\media-preview\media\imagePreview.js (posts the size message built from naturalWidth and naturalHeight); V\extensions\media-preview\src\imagePreview\index.ts; C\extensions\media-preview\dist\extension.js (id status.imagePreview.size at priority 101, label "Image Size"); C\extensions\media-preview\media\imagePreview.js (same naturalWidth x naturalHeight message)
- Pieces:
  1. While an image is the active Image Preview editor a right-side item shows the picture's own pixel size as "{width}x{height}", for example "1920x1080".
  2. The item stays empty until the image has loaded.
  3. The item has no click action.
  4. The item sits just right of the Image Zoom item and just left of the file size item, and hides when the preview is not the active editor.

#### SS-3.42 File size item in the image, audio and video previews

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While an image, audio or video preview is the active editor a right-side item shows the file size on disk as "{n}B", "{n.nn}KB", "{n.nn}MB", "{n.nn}GB" or "{n.nn}TB", for example "12.34KB" (id status.imagePreview.binarySize, name "Image Binary Size", priority 100); it has no click action, it refreshes when the file changes on disk, and it hides when the preview is not the active editor. SS-3.28 is the different File Information item of the core binary file viewer.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\extensions\media-preview\src\binarySizeStatusBarEntry.ts; V\extensions\media-preview\src\mediaPreview.ts (updateBinarySize, updateState); V\extensions\media-preview\src\extension.ts (one entry shared by the image, audio and video support); C\extensions\media-preview\dist\extension.js (id status.imagePreview.binarySize at priority 100, labels "{0}KB" and "{0}MB", the entry handed to the audio and video providers)
- Pieces:
  1. While an image preview is the active editor a right-side item shows the size of the file on disk, such as "512B", "12.34KB", "1.20MB", "1.20GB" or "1.20TB".
  2. The same item shows in the audio preview and the video preview.
  3. The item updates when the file changes on disk.
  4. The item has no click action and hides when the preview is not the active editor.

#### SS-3.43 TypeScript version row in the language status list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a JavaScript, JavaScript React, TypeScript or TypeScript React file the language status list gets a row whose text is the display name of the TypeScript version the language server started with (the bundled one or a workspace one) and whose link "Select Version" (tooltip is the path of that version) opens the version picker; the row appears only once the server has started (id typescript.version, name "TypeScript Version", detail "TypeScript version"). It sits inside the SS-3.29 bracket item and can be pinned like any row (SS-3.30).
- Commands: typescript.selectTypeScriptVersion
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\extensions\typescript-language-features\src\ui\versionStatus.ts; V\extensions\typescript-language-features\src\configuration\languageIds.ts (jsTsLanguageModes); C\extensions\typescript-language-features\dist\extension.js (createLanguageStatusItem "typescript.version", labels "TypeScript Version" and "TypeScript version", "Select Version")
- Pieces:
  1. In a JavaScript, JavaScript React, TypeScript or TypeScript React file the language status list gets a row whose text is the display name of the TypeScript version the server started with, with the detail "TypeScript version".
  2. The row has a link "Select Version" whose tooltip is the path of that version, and clicking it opens the version picker.
  3. The row appears only after the TypeScript server has started, and it does not appear in files of other languages.
  4. Pinning the row to the status bar shows the version as its own item that opens the version picker when clicked.

#### SS-3.44 Pick the TypeScript version

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The "Select Version" link, or the command TypeScript: Select TypeScript Version in the Command Palette (only for a file the TypeScript extension manages), opens a pick titled "Select the TypeScript version used for JavaScript and TypeScript language features" with "Use VS Code's Version" and its version, one "Use Workspace Version" row for each local TypeScript install it finds, "Use TypeScript Native Preview (Experimental)" only when that extension is installed, and a "Learn more about managing TypeScript versions" link; the active choice has a bullet, and a workspace version asks for Workspace Trust, remembers the choice for this workspace and writes the folder path into workspace settings before the server restarts on it.
- Commands: typescript.selectTypeScriptVersion
- Menus: commandPalette (when typescript.isManagedFile)
- Keys: none found
- Settings: js/ts.tsdk.path; typescript.tsdk; typescript.enablePromptUseWorkspaceTsdk
- Source: V\extensions\typescript-language-features\src\tsServer\versionManager.ts (promptUserForVersion, getBundledPickItem, getLocalPickItems, getNativePreviewPickItem, LearnMorePickItem); V\extensions\typescript-language-features\src\commands\selectTypeScriptVersion.ts; V\extensions\typescript-language-features\package.json (command, commandPalette entry, tsdk settings); C\extensions\typescript-language-features\dist\extension.js (typescript.selectTypeScriptVersion, "Use VS Code's Version", "Use Workspace Version", "Use TypeScript Native Preview (Experimental)", "Learn more about managing TypeScript versions"); C\extensions\typescript-language-features\package.json (js/ts.tsdk.path, typescript.tsdk, typescript.enablePromptUseWorkspaceTsdk)
- Pieces:
  1. Clicking the "Select Version" link opens a quick pick titled "Select the TypeScript version used for JavaScript and TypeScript language features".
  2. Running TypeScript: Select TypeScript Version (`typescript.selectTypeScriptVersion`) from the Command Palette opens the same quick pick, and the command is listed only while the active file is one the TypeScript extension manages.
  3. The first row is "Use VS Code's Version" with its version number, followed by one "Use Workspace Version" row for each local TypeScript install the extension finds.
  4. A "Use TypeScript Native Preview (Experimental)" row appears only when the TypeScript Native Preview extension is installed.
  5. The last row, "Learn more about managing TypeScript versions", opens the documentation page for it.
  6. The row of the version currently in use is marked with a bullet.
  7. Picking a workspace version asks for Workspace Trust, remembers the choice for this workspace, writes the folder path into the workspace settings and restarts the TypeScript server on that version.
  8. Setting `js/ts.tsdk.path`, and its older deprecated name `typescript.tsdk`, holds the folder path of the TypeScript install to use.
  9. Setting `typescript.enablePromptUseWorkspaceTsdk` chooses whether the person is prompted to use the TypeScript version set in the workspace, with values on and off, default off.

#### SS-3.45 JS/TS project config row in the language status list

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a JavaScript or TypeScript file the language status list gets an IntelliSense row (id typescript.projectStatus, name "JS/TS IntelliSense Status") that reads "Loading IntelliSense status" with a spinner while the server looks up the project, then the workspace-relative path of the tsconfig or jsconfig that owns the file with a link "Open Config File"; when no config owns the file it reads "No tsconfig" (or "No jsconfig" for a JavaScript file) with a link "Configure TSConfig" (or "Configure JSConfig") that opens tsconfig.json or jsconfig.json in the workspace root, and if it does not exist an untitled copy is filled with a starter config built from the implicit project settings plus an exclude of node_modules; with no folder open the row says "No opened folders" or "File is not part opened folders" and has no link.
- Commands: _typescript.openConfig; _typescript.createOrOpenConfig
- Menus: none found
- Keys: none found
- Settings: js/ts.implicitProjectConfig.module; js/ts.implicitProjectConfig.target; js/ts.implicitProjectConfig.checkJs; js/ts.implicitProjectConfig.experimentalDecorators; js/ts.implicitProjectConfig.strictNullChecks; js/ts.implicitProjectConfig.strictFunctionTypes; js/ts.implicitProjectConfig.strict
- Source: V\extensions\typescript-language-features\src\ui\intellisenseStatus.ts; V\extensions\typescript-language-features\src\tsconfig.ts (openOrCreateConfig, inferredProjectConfigSnippet, inferredProjectCompilerOptions); V\extensions\typescript-language-features\package.json (js/ts.implicitProjectConfig.* settings); C\extensions\typescript-language-features\dist\extension.js (createLanguageStatusItem "typescript.projectStatus", commands _typescript.openConfig and _typescript.createOrOpenConfig, labels "JS/TS IntelliSense Status", "Loading IntelliSense status", "No tsconfig", "No jsconfig", "Configure TSConfig", "Configure JSConfig", "Open Config File"); C\extensions\typescript-language-features\package.json (the same settings)
- Pieces:
  1. While the TypeScript server looks up the project of a JavaScript or TypeScript file, the language status list shows an IntelliSense row reading "Loading IntelliSense status" with a spinner.
  2. Once the project is known the row shows the workspace-relative path of the tsconfig or jsconfig that owns the file, with a link "Open Config File" that opens it.
  3. When no config file owns the file the row reads "No tsconfig" for a TypeScript file or "No jsconfig" for a JavaScript file, with a link "Configure TSConfig" or "Configure JSConfig".
  4. Clicking that link opens `tsconfig.json` or `jsconfig.json` in the workspace root, and when the file does not exist an untitled copy opens filled with a starter config built from the implicit project settings plus an exclude of `node_modules`.
  5. With no folder open the row reads "No tsconfig" or "No jsconfig" with the detail "No opened folders", and for a file outside the open folders the detail is "File is not part opened folders", with no link in either case.
  6. Settings `js/ts.implicitProjectConfig.module` (default `ESNext`) and `js/ts.implicitProjectConfig.target` (default `ES2024`) choose the module and target written into the starter config and used for files outside a project.
  7. Settings `js/ts.implicitProjectConfig.checkJs`, `js/ts.implicitProjectConfig.experimentalDecorators`, `js/ts.implicitProjectConfig.strictNullChecks`, `js/ts.implicitProjectConfig.strictFunctionTypes` and `js/ts.implicitProjectConfig.strict` each turn the matching compiler option on or off in the starter config and for files outside a project.

#### SS-3.46 Partial mode warning row

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When project-wide IntelliSense is not available for the open JavaScript or TypeScript file the same IntelliSense row turns into a warning that reads "Partial mode" with the detail "Project wide IntelliSense not available" and a link "Learn More" that opens the partial mode help page; on desktop this happens for files whose scheme is not a local file, an untitled file or a notebook cell, or when the syntax-only server is forced on, and being a warning it brings the warning bracket icon and the flash cue of SS-3.32.
- Commands: vscode.open
- Menus: none found
- Keys: none found
- Settings: js/ts.tsserver.useSyntaxServer; typescript.tsserver.useSyntaxServer
- Source: V\extensions\typescript-language-features\src\ui\intellisenseStatus.ts (IntellisenseState.SyntaxOnly); V\extensions\typescript-language-features\src\typescriptServiceClient.ts (capabilities, hasCapabilityForResource); V\extensions\typescript-language-features\src\configuration\fileSchemes.ts (getSemanticSupportedSchemes); C\extensions\typescript-language-features\dist\extension.js (labels "Partial mode" and "Project wide IntelliSense not available"); C\extensions\typescript-language-features\package.json (js/ts.tsserver.useSyntaxServer)
- Pieces:
  1. When project-wide IntelliSense is not available for the open JavaScript or TypeScript file the IntelliSense row reads "Partial mode" with the detail "Project wide IntelliSense not available".
  2. The row has a link "Learn More" that opens the partial mode help page.
  3. Because the row is a warning, the bracket icon changes to the warning bracket and the item flashes as described in SS-3.32.
  4. On desktop the row shows for files whose scheme is not a local file, for untitled files and for notebook cells.
  5. Setting `js/ts.tsserver.useSyntaxServer` (and the older `typescript.tsserver.useSyntaxServer`) chooses whether the syntax-only server is used, with values `always`, `never` and `auto`, default `auto`, and `always` puts every file in Partial mode.

#### SS-3.47 Configure Excludes item for a project that is too large

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the TypeScript server turns off project-wide language features because a project holds too many files, a right-side item "Configure Excludes" appears in a green text color (id status.typescript.exclude, name "TypeScript: Configure Excludes", priority 98) with a tooltip telling the person to exclude large folders they do not work on; clicking it opens the project's tsconfig or jsconfig (or creates one in the workspace root) and shows the same hint as an information message, the same event also raises a warning message with a Configure Excludes button, and the item hides again once the server reports the language service enabled.
- Commands: js.projectStatus.command
- Menus: none found
- Keys: none found
- Settings: js/ts.implicitProjectConfig.module; js/ts.implicitProjectConfig.target; js/ts.implicitProjectConfig.checkJs; js/ts.implicitProjectConfig.experimentalDecorators; js/ts.implicitProjectConfig.strictNullChecks; js/ts.implicitProjectConfig.strictFunctionTypes; js/ts.implicitProjectConfig.strict
- Source: V\extensions\typescript-language-features\src\ui\largeProjectStatus.ts (ExcludeHintItem, onConfigureExcludesSelected); V\extensions\typescript-language-features\src\typescriptServiceClient.ts (projectLanguageServiceState event); C\extensions\typescript-language-features\dist\extension.js (createStatusBarItem "status.typescript.exclude" at priority 98, command js.projectStatus.command, labels "TypeScript: Configure Excludes" and "Configure Excludes")
- Pieces:
  1. When the TypeScript server turns off project-wide language features because a project holds too many files, a green "Configure Excludes" item appears on the right side of the status bar.
  2. Hovering the item shows a tooltip telling the person to exclude large folders with source files they do not work on to enable project-wide JavaScript and TypeScript language features.
  3. Clicking the item opens the project's tsconfig or jsconfig, or creates one in the workspace root when the project has none, and shows the same hint as an information message.
  4. The same event raises a warning message with the same hint and a "Configure Excludes" button that opens or creates the config file the same way.
  5. The item disappears once the server reports that the language service is enabled again.
  6. A config file created from this item is filled from the `js/ts.implicitProjectConfig.*` settings listed in SS-3.45.

#### SS-3.48 JSON schema validation row

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a file of a language the JSON server handles (JSON, JSON with Comments and the like) the language status list gets a validation row (id json.projectStatus, name "JSON Validation Status") that reads "No schema validation" with the detail "no JSON schema configured", or "Schema validated" with the detail "JSON schema configured" for one schema or "multiple JSON schemas configured" for several; it shows a spinner and "Loading JSON info" while it works, "Unable to compute used schemas" with the reason when the lookup fails, and a link "Show Schemas" that opens the list in SS-3.49.
- Commands: _json.showAssociatedSchemaList
- Menus: none found
- Keys: none found
- Settings: json.validate.enable; json.schemas
- Source: V\extensions\json-language-features\client\src\languageStatus.ts (createLanguageStatusItem); V\extensions\json-language-features\client\src\jsonClient.ts (registration, SettingIds); C\extensions\json-language-features\client\dist\node\jsonClientMain.js (json.projectStatus, labels "JSON Validation Status", "No schema validation", "Schema validated", "no JSON schema configured", "multiple JSON schemas configured", "Show Schemas")
- Pieces:
  1. In a JSON file with no schema the language status list shows a validation row "No schema validation" with the detail "no JSON schema configured".
  2. With one schema the row reads "Schema validated" with the detail "JSON schema configured", and with several schemas the detail is "multiple JSON schemas configured".
  3. While the row works it shows a spinner and the detail "Loading JSON info".
  4. When the lookup fails the row reads "Unable to compute used schemas" followed by the reason and has no link.
  5. The row has a link "Show Schemas" that opens the schema list described in SS-3.49.
  6. Setting `json.validate.enable` chooses whether JSON files are validated, with values on and off, default on.
  7. Setting `json.schemas` lists the schema associations that decide which schemas the row counts.

#### SS-3.49 Show the schemas used for a JSON file

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The "Show Schemas" link opens a pick titled "Select the schema to use for {file}" that lists each schema applied to the file, marked "Configured by extension: {id}" with an Open Extension button, or "Configured in user settings" or "Configured in workspace settings" with an Open Settings button that jumps to json.schemas, or just the schema URL; with none it shows "No schema configured for this file", a last row "Learn more about JSON schema configuration..." links to the docs, and picking a schema opens that schema file.
- Commands: _json.showAssociatedSchemaList; workbench.action.openSettingsJson; workbench.action.openWorkspaceSettingsFile; vscode.open
- Menus: none found
- Keys: none found
- Settings: json.schemas
- Source: V\extensions\json-language-features\client\src\languageStatus.ts (showSchemaList, getExtensionSchemaAssociations, getSettingsSchemaAssociations); C\extensions\json-language-features\client\dist\node\jsonClientMain.js (command _json.showAssociatedSchemaList, labels "Select the schema to use for {0}", "No schema configured for this file", "Learn more about JSON schema configuration")
- Pieces:
  1. Clicking "Show Schemas" opens a quick pick with the placeholder "Select the schema to use for {file}" listing each schema that applies to the file.
  2. A schema that comes from an extension is marked "Configured by extension: {id}" and has an Open Extension button.
  3. A schema set in user or workspace settings is marked "Configured in user settings" or "Configured in workspace settings" and has an Open Settings button that opens the right settings file at `json.schemas`.
  4. A schema that matches neither case shows only its URL.
  5. When no schema applies the quick pick shows one row, "No schema configured for this file", with an Open Settings button that opens the user settings JSON at `json.schemas`.
  6. A last row after a separator, "Learn more about JSON schema configuration...", opens the documentation page for it.
  7. Picking a schema row opens that schema file.

#### SS-3.50 JSON outline limit warning row

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a JSON file has more symbols than the limit, a warning row "Outline" appears in the language status list with the detail "only {n} document symbols shown for performance reasons" and a link "Configure" that opens Settings on json.maxItemsComputed (default 5000); it shows only for the active file and goes away when the file drops under the limit (id json.documentSymbolsStatus, name "JSON Outline Status"). SS-3.31 lists the matching rows that core adds for folding and color decorators.
- Commands: workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: json.maxItemsComputed
- Source: V\extensions\json-language-features\client\src\languageStatus.ts (createLimitStatusItem, createDocumentSymbolsLimitItem); V\extensions\json-language-features\client\src\jsonClient.ts (documentSymbolsLimitStatusbarItem); V\extensions\json-language-features\package.json (json.maxItemsComputed default 5000); C\extensions\json-language-features\client\dist\node\jsonClientMain.js (json.documentSymbolsStatus, label "JSON Outline Status", the "only {0} document symbols shown for performance reasons" text, setting json.maxItemsComputed)
- Pieces:
  1. When a JSON file has more symbols than the limit the language status list shows a warning row "Outline" with the detail "only {n} document symbols shown for performance reasons".
  2. The row has a link "Configure" that opens Settings on `json.maxItemsComputed`.
  3. The row shows only while that file is the active editor, and it goes away when the file drops under the limit.
  4. Setting `json.maxItemsComputed` chooses the most outline symbols and folding regions computed for a JSON file, a number with a default of 5000.

#### SS-3.51 JSON schema download issue row

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing - 0 hits in the Cursor JSON extension for "Schema download issue", "Download disabled", "Location untrusted", "Workspace untrusted", _json.configureTrustedDomains and the setting json.schemaDownload.trustedDomains; its older JSON client shows the alert item in SS-3.52 instead
- What: When the schema of the active JSON file cannot be loaded the language status list gets an error row "Schema download issue" (id json.documentSymbolsStatus, shared with the outline row, name "JSON Outline Status") whose detail says why: "Workspace untrusted" with a link to Manage Workspace Trust, "Download disabled" with a link "Configure" that opens Settings on json.schemaDownload.enable, "Location untrusted" with a link to configure trusted domains, or "Unable to resolve schema" with a link "Retry"; being an error it brings the error bracket icon and the flash cue of SS-3.32.
- Commands: _json.retryResolveSchema; workbench.trust.manage; workbench.action.openSettings; _json.configureTrustedDomains
- Menus: none found
- Keys: none found
- Settings: json.schemaDownload.enable; json.schemaDownload.trustedDomains
- Source: V\extensions\json-language-features\client\src\languageStatus.ts (createSchemaLoadStatusItem, createSchemaLoadIssueItem); V\extensions\json-language-features\client\src\jsonClient.ts (schemaLoadStatusItem, CommandIds); V\extensions\json-language-features\package.json (json.schemaDownload.trustedDomains); C\extensions\json-language-features\client\dist\node\jsonClientMain.js and package.json (searched, none of the strings above)
- Pieces:
  1. When the schema of the active JSON file cannot be loaded the language status list shows an error row "Schema download issue", and the bracket icon switches to the error bracket and flashes as described in SS-3.32. [stock only]
  2. With the workspace untrusted the row detail reads "Workspace untrusted" and its link opens Manage Workspace Trust. [stock only]
  3. With schema downloads off the row detail reads "Download disabled" and its link "Configure" opens Settings on `json.schemaDownload.enable`. [stock only]
  4. When the schema location is not trusted the row detail reads "Location untrusted" and its link opens the trusted domains configuration. [stock only]
  5. For any other failure the row detail reads "Unable to resolve schema" and its link "Retry" tries to load the schema again. [stock only]
  6. Setting `json.schemaDownload.enable` chooses whether JSON schemas can be downloaded, with values on and off, default on. [stock only]
  7. Setting `json.schemaDownload.trustedDomains` chooses which schema locations count as trusted. [stock only]

#### SS-3.52 JSON schema resolution error alert item

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only - older JSON client than V (the item is shown there, the language status row is absent)
- What: In Cursor's JSON client, when the active JSON file has a schema that cannot be loaded, an icon-only alert item appears on the right side (id status.json.resolveError, name "JSON: Schema Resolution Error", priority 0, so right of the editor items and left of the notifications bell); with downloads on its tooltip reads "Unable to resolve schema. Click to retry." and clicking retries the validation (the icon shows a watch while it runs and the item hides if the schema now loads), and with downloads off the tooltip reads "Downloading schemas is disabled. Click to configure." and clicking opens Settings on json.schemaDownload.enable. Stock 1.128.0 creates this item but never shows it, because it moved this job to the language status row in SS-3.51.
- Commands: _json.retryResolveSchema; workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: json.schemaDownload.enable
- Source: C\extensions\json-language-features\client\dist\node\jsonClientMain.js (createStatusBarItem "status.json.resolveError" Right 0, show and hide on active editor change, tooltips "Unable to resolve schema. Click to retry." and "Downloading schemas is disabled. Click to configure.", command _json.retryResolveSchema, text alert icon and watch icon); V\extensions\json-language-features\client\src\jsonClient.ts (the item is created at the same id and priority and given text and tooltip only, with no show call anywhere in the client)
- Pieces:
  1. In Cursor, when the active JSON file has a schema that cannot be loaded, an icon-only alert item appears on the right side, to the right of the editor items and left of the notifications bell. [cursor only]
  2. With schema downloads on, hovering the item shows "Unable to resolve schema. Click to retry." and clicking it retries the validation, showing a watch icon while it runs and hiding the item if the schema now loads. [cursor only]
  3. With schema downloads off, hovering the item shows "Downloading schemas is disabled. Click to configure." and clicking it opens Settings on `json.schemaDownload.enable`. [cursor only]
  4. The item shows only while the JSON file with the schema error is the active editor and hides when another editor becomes active. [cursor only]

#### SS-3.53 Markdown link validation row with an on and off link

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a Markdown file the language status list gets a row (id markdownStatus) that reads "Markdown link validation enabled" with a link "Disable" (tooltip "Disable validation of Markdown links"), or "Markdown link validation disabled" with a link "Enable" (tooltip "Enable validation of Markdown links"); clicking the link flips markdown.validate.enabled (off by default) and the row text updates as soon as that setting changes.
- Commands: _markdown.toggleValidation
- Menus: none found
- Keys: none found
- Settings: markdown.validate.enabled
- Source: V\extensions\markdown-language-features\src\languageFeatures\diagnostics.ts (registerMarkdownStatusItem); V\extensions\markdown-language-features\package.json (markdown.validate.enabled, default false); C\extensions\markdown-language-features\dist\extension.js (markdownStatus, command _markdown.toggleValidation, the four labels above); C\extensions\markdown-language-features\package.json (markdown.validate.enabled)
- Pieces:
  1. In a Markdown file with link validation on, the language status list shows the row "Markdown link validation enabled" with a link "Disable" whose tooltip is "Disable validation of Markdown links".
  2. In a Markdown file with link validation off, the row reads "Markdown link validation disabled" with a link "Enable" whose tooltip is "Enable validation of Markdown links".
  3. Clicking the link flips `markdown.validate.enabled`, and the row text and link change at once.
  4. Setting `markdown.validate.enabled` chooses whether Markdown links are validated, with values on and off, default off, and changing it in Settings also updates the row.


## SS-4 Workspace, remote and source control

(both). Left-side items that say where you work: remote host, Restricted Mode, forwarded ports, Git repo, branch and sync, Git blame.

16 features.

#### SS-4.1 Remote indicator at the far left

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The first item on the left side shows a remote icon (tooltip "Open a Remote Window") in a plain local window, and the remote icon plus the host name (tooltip "Editing on {host}", name cut at 40 characters, shown in the remote color) in a remote window or a virtual workspace such as a GitHub repo (id status.host, name "Remote Host", priority infinity so it is always first).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts (updateRemoteStatusIndicator, renderRemoteStatusIndicator); V\src\vs\workbench\contrib\remote\browser\remote.contribution.ts; C bundle (module vs/workbench/contrib/remote/browser/remoteIndicator, id status.host, labels "Open a Remote Window", "Editing on {0}", "Remote Host"; the code in updateRemoteStatusIndicator matches V)
- Pieces:
  1. In a plain local window the first item on the left of the status bar is a remote icon with the tooltip "Open a Remote Window".
  2. In a remote window the item shows the remote icon followed by the host name, drawn in the remote color, and its tooltip reads "Editing on {host}".
  3. A host name longer than 40 characters is cut off in the item text.
  4. In a virtual workspace such as a GitHub repository the item looks the same as in a remote window, with the remote icon and the workspace host name.
  5. The item always stays first on the left side, ahead of every other left-side item.

#### SS-4.2 Remote indicator shows connection and network trouble

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a remote window the item reads "Opening Remote..." with a spinner while connecting, "Reconnecting to {host}..." with a spinner, or an alert icon with "Disconnected from {host}" when the link drops; when the network looks offline the icon turns into an alert, the item takes the offline color and the tooltip says "Network appears to be offline, certain features might be unavailable.", and on slow links the tooltip gives the last and average latency in ms.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts (withNetworkStatus, setConnectionState, setNetworkState); C bundle (NLS keys host.open, host.reconnecting, disconnectedFrom, networkStatusOfflineTooltip, networkStatusHighLatencyTooltip in module vs/workbench/contrib/remote/browser/remoteIndicator)
- Pieces:
  1. While a remote window is still connecting the item reads "Opening Remote..." with a spinner.
  2. While the link to the host is being re-established the item reads "Reconnecting to {host}..." with a spinner.
  3. When the connection drops the item shows an alert icon with the text "Disconnected from {host}".
  4. When the network looks offline the remote icon turns into an alert icon, the item takes the offline color, and the tooltip says "Network appears to be offline, certain features might be unavailable."
  5. On a slow link the tooltip gives the last and the average latency in milliseconds.

#### SS-4.3 Open the Remote menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking the remote item, or running Show Remote Menu from the Command Palette or the shortcut, opens a picker "Select an option to open a Remote Window" that lists the commands extensions add for the current remote first, then an "Install" section of recommended remote extensions with a Learn More button on each, then the Close Remote Connection or Close Remote Workspace row (and Reload Window when disconnected).
- Commands: workbench.action.remote.showMenu
- Menus: StatusBarRemoteIndicatorMenu (contributes.menus key statusBar/remoteIndicator); StatusBarWindowIndicatorMenu; commandPalette
- Keys: ctrl+alt+o (Cmd+Alt+O on macOS)
- Settings: workbench.remoteIndicator.showExtensionRecommendations
- Source: V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts (registerActions, showRemoteMenu); V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts (statusBar/remoteIndicator); V\src\vs\platform\actions\common\actions.ts; C bundle (command id workbench.action.remote.showMenu with keybinding CtrlCmd+Alt+KeyO, menu statusBar/remoteIndicator, label "Show Remote Menu", setting workbench.remoteIndicator.showExtensionRecommendations)
- Pieces:
  1. Clicking the remote item opens a picker whose placeholder reads "Select an option to open a Remote Window".
  2. Running Show Remote Menu (`workbench.action.remote.showMenu`) from the Command Palette, or pressing `Ctrl+Alt+O` (`Cmd+Alt+O` on macOS), opens the same picker.
  3. The picker lists the commands that extensions add through the `statusBar/remoteIndicator` menu first, under their category headings, with the commands for the current remote ahead of the others.
  4. After those commands an "Install" section lists the recommended remote extensions that are not installed yet, each with a Learn More button.
  5. Setting `workbench.remoteIndicator.showExtensionRecommendations` chooses whether the "Install" section is shown, with values on (default) and off.

#### SS-4.4 Close the remote connection or workspace

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: In a remote window, Close Remote Connection (File menu, Command Palette, or the last row of the Remote menu) reopens the folder in a local window; in a virtual workspace the row reads Close Remote Workspace; when the connection has dropped the Remote menu also offers Reload Window. Not offered in the web build.
- Commands: workbench.action.remote.close; workbench.action.reloadWindow
- Menus: MenubarFileMenu (group 6_close); commandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts (CLOSE_REMOTE_COMMAND_ID, showRemoteMenu); C bundle (id workbench.action.remote.close, File menu group 6_close order 3.5, labels "Close Remote Connection", "Close Remote Workspace", "Reload Window")
- Pieces:
  1. In a remote window, choosing File, Close Remote Connection reopens the folder in a local window.
  2. Running Close Remote Connection (`workbench.action.remote.close`) from the Command Palette does the same, and the command is offered only in a remote window or a virtual workspace.
  3. In a remote window the last row of the Remote menu reads "Close Remote Connection" and does the same.
  4. In a virtual workspace the last row of the Remote menu reads "Close Remote Workspace" instead.
  5. When the connection has dropped, the Remote menu also lists Reload Window (`workbench.action.reloadWindow`) after the close row.
  6. In the web build neither the close command nor the close rows are offered.

#### SS-4.5 Restricted Mode item

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - item, label and command are present, but security.workspace.trust.enabled defaults to false in Cursor (true in stock), so the item only shows after trust is switched on
- What: When Workspace Trust is on and the window, folder or workspace is not trusted, a prominent item "$(shield) Restricted Mode" sits just right of the remote indicator; its hover says "Running in Restricted Mode" with the reason and links to the disabled features list and to Manage Workspace Trust, and clicking it opens the Workspace Trust editor; it disappears once trust is granted (id status.workspaceTrust, name "Workspace Trust").
- Commands: workbench.trust.manage
- Menus: commandPalette
- Keys: none found
- Settings: security.workspace.trust.enabled
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (getRestrictedModeStatusbarEntry, updateStatusbarEntry, MANAGE_TRUST_COMMAND_ID, trust setting default true); C bundle (module vs/workbench/contrib/workspace/browser/workspace.view.contribution, id status.workspaceTrust, label "Restricted Mode", command workbench.trust.manage, setting security.workspace.trust.enabled registered with default false)
- Pieces:
  1. When Workspace Trust is on and the window, folder or workspace is not trusted, an item with a shield icon and the text "Restricted Mode" appears just right of the remote indicator.
  2. Hovering the item shows "Running in Restricted Mode" with the reason, a link to the list of disabled features, and a link to Manage Workspace Trust.
  3. Clicking the item, or running Manage Workspace Trust (`workbench.trust.manage`) from the Command Palette, opens the Workspace Trust editor.
  4. The item disappears once trust is granted.
  5. Setting `security.workspace.trust.enabled` chooses whether Workspace Trust is on, with values on and off, and with it off the item never appears; stock defaults it to on.
  6. In Cursor the setting defaults to off, so the item shows only after trust is switched on. [cursor only]

#### SS-4.6 Forwarded Ports item

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A radio tower icon with a count (tooltip "No Ports Forwarded" or "Forwarded Ports: 3000, 8080") appears on the left side only in a remote window or after a port was forwarded in this session; clicking it focuses the Ports view (id status.forwardedPorts, priority 40).
- Commands: ~remote.forwardedPorts.focus
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\remote\browser\remoteExplorer.ts (updateStatusBar, entry); V\src\vs\workbench\services\remote\common\remoteExplorerService.ts (TUNNEL_VIEW_ID); C bundle (id status.forwardedPorts at priority 40, view id ~remote.forwardedPorts)
- Pieces:
  1. In a remote window a radio tower icon with a port count appears on the left side of the bar.
  2. The tooltip reads "No Ports Forwarded" when nothing is forwarded and "Forwarded Ports: 3000, 8080" with the port numbers when some are.
  3. In a local window the item stays hidden until a port has been forwarded in this session.
  4. Clicking the item focuses the Ports view (`~remote.forwardedPorts.focus`).

#### SS-4.7 Repository name item when several repositories are open

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the item and the active-repository logic are present, but Cursor always draws the plain repo icon and never the pinned-repo icon
- What: When more than one Git repository is open, an extra item with a repo icon and the active repository's name sits just left of the branch item (tooltip "{folder} ({provider label})"); the active repository follows the active editor or the focused repository in the Source Control view unless one is pinned, and stock swaps the icon to a pinned-repo icon while pinned (id status.scm.provider, name "Source Control Provider").
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\scm\browser\activity.ts (_updateStatusBar); V\src\vs\workbench\contrib\scm\browser\util.ts (getSCMRepositoryIcon); V\src\vs\workbench\contrib\scm\browser\scmViewService.ts (activeRepository); C bundle (module vs/workbench/contrib/scm/browser/activity, id status.scm.provider, text "$(repo) {name}", condition scmService.repositoryCount > 1, label "Source Control Provider")
- Pieces:
  1. When more than one Git repository is open, an extra item with a repo icon and the name of the active repository appears just left of the branch item.
  2. With only one repository open the item is not shown.
  3. The tooltip reads "{folder} ({provider label})".
  4. The active repository follows the active editor, or the focused repository in the Source Control view, unless one repository is pinned.
  5. While a repository is pinned the item shows a pinned-repo icon in place of the plain repo icon. [stock only]
  6. In Cursor the item always shows the plain repo icon, even while a repository is pinned. [cursor only]

#### SS-4.8 Pick or pin the active repository

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking the repository item opens a picker "Select the active repository, type to filter all repositories" with Auto first (the active repository follows the active editor) and then every repository with its folder path; choosing a repository pins it and choosing Auto unpins it.
- Commands: scm.setActiveProvider
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (scm.setActiveProvider); V\src\vs\workbench\contrib\scm\browser\scmViewService.ts (RepositoryPicker, pinActiveRepository); C bundle (command scm.setActiveProvider, NLS keys scmActiveRepositoryPlaceHolder and scmActiveRepositoryAutoDescription, label "Auto", method pinActiveRepository)
- Pieces:
  1. Clicking the repository item runs `scm.setActiveProvider` and opens a picker whose placeholder reads "Select the active repository, type to filter all repositories".
  2. The first row of the picker is Auto, which makes the active repository follow the active editor.
  3. Below Auto every open repository is listed as a row with its folder path.
  4. Typing in the picker filters the repositories.
  5. Choosing a repository pins it as the active one.
  6. Choosing Auto unpins the repository.

#### SS-4.9 Branch item shows the current branch, tag or commit

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The first source control item shows an icon and the head name: a branch icon with the branch name, a lock icon when the branch is in git.branchProtection, a tag icon on a tag, a commit icon with the short hash on a detached commit, and a spinning icon with the target name while a checkout runs; the tooltip reads "{repo folder} ({provider}) - {name}, Checkout Branch/Tag..." or the running step ("Committing Changes...", "Synchronizing Changes...") (id status.scm.0, name starts with "Source Control", priority 10000 so it is the first Git item).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.branchProtection
- Source: V\extensions\git\src\statusbar.ts (CheckoutStatusBar.command, getIcon, getTooltip); V\extensions\git\src\repository.ts (headLabel); V\src\vs\workbench\contrib\scm\browser\activity.ts (status.scm.{index}); C bundle (extensions\git\dist\main.js CheckoutStatusBar with icons lock, git-branch, tag, git-commit, loading~spin and tooltip "Checkout Branch/Tag..."; module vs/workbench/contrib/scm/browser/activity)
- Pieces:
  1. The first source control item on the bar shows a branch icon and the name of the current branch.
  2. When the branch is listed in `git.branchProtection` the branch icon is replaced by a lock icon.
  3. On a checked-out tag the item shows a tag icon and the tag name.
  4. On a detached commit the item shows a commit icon and the first 8 characters of the commit hash.
  5. While a checkout runs the item shows a spinning icon and the name of the target.
  6. The tooltip reads "{repo folder} ({provider}) - {name}, Checkout Branch/Tag...", or names the running step: "Checking Out Branch/Tag...", "Committing Changes..." or "Synchronizing Changes...".
  7. Setting `git.branchProtection` lists the branch names that get the lock icon, as a list of names that starts empty.
  8. The item sits first among the Git items on the left side, ahead of the sync item.

#### SS-4.10 Branch item marks changes, staged changes and merge or rebase

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the star, plus, exclamation mark and "(Rebasing)" text are present, but the git-branch-changes, git-branch-staged-changes and git-branch-conflicts icons are not (Cursor only draws branch, lock, tag, commit and spinner icons)
- What: The head name gets a star for working tree or untracked changes, a plus for staged changes, and an exclamation mark while a merge or rebase is in progress (plus " (Rebasing)" during a rebase); in stock the branch icon also changes to a changes, staged changes or branch-conflicts icon. This is the only merge conflict marker on the status bar; the conflict list, merge editor and CodeLens live in other groups.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\extensions\git\src\repository.ts (headLabel); V\extensions\git\src\statusbar.ts (getIcon, rebasing label); C bundle (extensions\git\dist\main.js headLabel with the three suffix characters and the "Rebasing" string, getIcon without the three branch state icons; 0 hits for git-branch-conflicts, git-branch-staged-changes, git-branch-changes)
- Pieces:
  1. The head name on the branch item gets a star at its end when the working tree has changes or untracked files.
  2. The head name gets a plus when changes are staged.
  3. The head name gets an exclamation mark while a merge or rebase is in progress.
  4. During a rebase the label also reads " (Rebasing)" after the head name.
  5. In stock the branch icon changes to a changes icon, a staged changes icon or a branch conflicts icon to match those states. [stock only]
  6. In Cursor the branch icon stays the plain branch icon (or the lock, tag, commit or spinner icon) and never changes to the changes, staged changes or branch conflicts icons. [cursor only]

#### SS-4.11 Click the branch item to check out a branch or tag

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking the branch item runs Checkout to... for that repository and opens the picker of local branches, remote branches and tags (tooltip "Checkout Branch/Tag..."); the click does nothing while a checkout, commit or sync is running.
- Commands: git.checkout
- Menus: commandPalette; scm/title; scm/repository
- Keys: none found
- Settings: git.checkoutType; git.branchSortOrder; git.showReferenceDetails; git.pullBeforeCheckout
- Source: V\extensions\git\src\statusbar.ts (CheckoutStatusBar.command); V\extensions\git\package.json (git.checkout command, menus); V\extensions\git\src\commands.ts; C bundle (extensions\git\dist\main.js git.checkout in CheckoutStatusBar; extensions\git\package.json git.checkout)
- Pieces:
  1. Clicking the branch item runs Checkout to... (`git.checkout`) for that repository and opens the picker of branches and tags.
  2. Checkout to... also opens from the Command Palette and from the Source Control view title and repository menus.
  3. While a checkout, commit or sync is running, clicking the branch item does nothing.
  4. Setting `git.checkoutType` chooses which kinds of refs the picker lists, with values `local`, `tags` and `remote`, all three by default.
  5. Setting `git.branchSortOrder` chooses how branches are sorted, with values `committerdate` (default) and `alphabetically`.
  6. Setting `git.showReferenceDetails` chooses whether the last commit details are shown on refs in the checkout, branch and tag pickers, with values on (default) and off.
  7. Setting `git.pullBeforeCheckout` chooses whether a branch with no outgoing commits is fast-forwarded before it is checked out, with values on and off (default).

#### SS-4.12 Sync Changes item with ahead and behind counts

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Next to the branch item a sync icon appears when the branch has an upstream; when it differs from the remote the item adds counts such as "2↓ 1↑" (behind then ahead, only the down count for a read-only remote); the tooltip says "Pull 2 commits from origin/main", "Push 1 commits to origin/main" or "Pull 2 and push 1 commits between origin/main", and clicking it runs Sync (pull then push); while syncing the icon spins and the click is disabled (id status.scm.1, compact pair with the branch item).
- Commands: git.sync
- Menus: commandPalette
- Keys: none found
- Settings: git.enableStatusBarSync; git.confirmSync
- Source: V\extensions\git\src\statusbar.ts (SyncStatusBar.command); V\extensions\git\src\repository.ts (syncLabel, syncTooltip); V\extensions\git\package.json (git.enableStatusBarSync default true); C bundle (extensions\git\dist\main.js SyncStatusBar with sync and sync~spin icons, git.sync command and the three tooltip strings; extensions\git\package.json git.enableStatusBarSync default true)
- Pieces:
  1. When the branch has an upstream, a sync icon item appears next to the branch item.
  2. While the branch matches its remote the item shows only the icon and its tooltip reads "Synchronize Changes".
  3. When the branch differs from its remote the item adds counts such as "2↓ 1↑", behind first and ahead second.
  4. For a read-only remote the item shows only the down count, for example "2↓".
  5. The tooltip reads "Pull 2 commits from origin/main" when only behind, "Push 1 commits to origin/main" when only ahead, and "Pull 2 and push 1 commits between origin/main" when both.
  6. Clicking the item, or running Sync (`git.sync`) from the Command Palette, pulls and then pushes.
  7. While a checkout, commit or sync is running the click does nothing and the tooltip names the step, and during a sync the icon spins.
  8. Setting `git.enableStatusBarSync` chooses whether the Sync item appears in the bar, with values on (default) and off.
  9. Setting `git.confirmSync` chooses whether a confirmation is asked before synchronizing, with values on (default) and off.

#### SS-4.13 Publish Branch item

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the branch has no upstream, the sync slot shows a cloud-upload icon with the tooltip "Publish Branch" and a click runs Publish Branch; when the repository has no remote at all and a publisher such as GitHub is registered, the icon-only item says "Publish to {name}" (or "Publish to..." with several publishers); with no remote and no publisher the slot stays empty.
- Commands: git.publish
- Menus: commandPalette
- Keys: none found
- Settings: git.enableStatusBarSync
- Source: V\extensions\git\src\statusbar.ts (SyncStatusBar.command, hasRemotes and remoteSourcePublishers branches); V\extensions\git\package.json (git.publish); C bundle (extensions\git\dist\main.js SyncStatusBar with cloud-upload icon, git.publish, "Publish Branch", "Publish to {0}", "Publish to...")
- Pieces:
  1. When the branch has no upstream, the sync slot shows a cloud-upload icon with the tooltip "Publish Branch".
  2. Clicking the item, or running Publish Branch (`git.publish`) from the Command Palette, publishes the branch.
  3. When the repository has no remote at all and one publisher such as GitHub is registered, the slot shows an icon-only item with the tooltip "Publish to {name}".
  4. When several publishers are registered the tooltip reads "Publish to...".
  5. With no remote and no publisher the slot stays empty.
  6. While a checkout or commit is running the click does nothing and the tooltip names the step, "Checking Out Changes..." or "Committing Changes...".
  7. Turning `git.enableStatusBarSync` off hides the publish items together with the Sync item.

#### SS-4.14 Git blame item for the current line

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: On the right side a commit icon with the author and age of the cursor line (for example "Ada Lovelace (3 days ago)" from the default template) shows whenever the active text editor is in a Git repository (id git.blame, name "Git Blame Information", priority 200); it reads "Not Committed Yet" when you move onto an edited line and "Not Committed Yet (Staged)" for staged lines, it hides when there is no blame for the line, for resources other than file and git schemes, or when the cursor sits at the very start of a file you have not clicked in, and clicking it opens that commit.
- Commands: git.viewCommit
- Menus: none found
- Keys: none found
- Settings: git.blame.statusBarItem.enabled; git.blame.statusBarItem.template; git.blame.ignoreWhitespace; git.commitShortHashLength
- Source: V\extensions\git\src\blame.ts (GitBlameStatusBarItem, _updateTextEditorBlameInformation); V\extensions\git\package.json (git.blame.* settings, git.viewCommit); C bundle (extensions\git\dist\main.js createStatusBarItem("git.blame", Right, 200), name "Git Blame Information", strings "Not Committed Yet" and "Not Committed Yet (Staged)", command git.viewCommit; extensions\git\package.json settings git.blame.statusBarItem.enabled default true and git.blame.statusBarItem.template)
- Pieces:
  1. On the right side of the bar a commit icon with the author and age of the line under the cursor appears, for example "Ada Lovelace (3 days ago)" with the default template.
  2. The item shows only while the active text editor is in a Git repository, and it is hidden for resources other than file and git schemes.
  3. After you move onto an edited line the text reads "Not Committed Yet", and on a staged line it reads "Not Committed Yet (Staged)".
  4. The item is hidden when there is no blame for the line, and when the cursor sits at the very start of a file you have not clicked in.
  5. Clicking the item opens that commit (`git.viewCommit`).
  6. Setting `git.blame.ignoreWhitespace` chooses whether whitespace-only changes are ignored when finding the commit for a line, with values on and off (default).
  7. Setting `git.commitShortHashLength` chooses how many characters a short commit hash has, a number from 7 to 40 with default 7.

#### SS-4.15 Blame hover card

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the card has avatar, author, date, message, stats, hash, copy and gear, but no co-author lines, and Open Commit passes no file to reveal in the commit diff
- What: Hovering the blame item shows a card with the author avatar (or an account icon), the author name as an email link, the date as "3 days ago (long date)", co-authors, the full commit message, files changed with insertions and deletions, and a row of links: the short hash that opens the commit, a copy hash button, provider links such as view on GitHub for pushed commits, and a gear that opens Settings on git.blame.
- Commands: git.viewCommit; git.copyContentToClipboard; workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: git.commitShortHashLength
- Source: V\extensions\git\src\blame.ts (getBlameInformationHover, GitBlameStatusBarItem._provideTooltip); V\extensions\git\src\hover.ts (getCommitHover, getHoverCommitHashCommands, appendContent, appendShortStats, appendCommands); C bundle (extensions\git\dist\main.js getBlameInformationHover builds the markdown directly, tooltips "Open Commit", "Copy Commit Hash", "Open Settings"; 0 hits for "Co-author"; viewCommit takes two arguments)
- Pieces:
  1. Hovering the blame item shows a card with the author avatar, or an account icon when there is no avatar, and the author name as an email link.
  2. The card shows the date as "3 days ago (long date)".
  3. The card lists the co-authors of the commit. [stock only]
  4. The card shows the full commit message and the files changed with their insertions and deletions.
  5. The card ends with a row of links: the short hash that opens the commit (`git.viewCommit`), a copy hash button (`git.copyContentToClipboard`), and a gear that opens Settings on `git.blame` (`workbench.action.openSettings`).
  6. For commits that are already pushed the row also holds provider links such as view on GitHub.
  7. Setting `git.commitShortHashLength` sets how long the short hash in the card is, a number from 7 to 40 with default 7.
  8. In Cursor the card has no co-author lines, and Open Commit passes no file to reveal in the commit diff. [cursor only]

#### SS-4.16 Turn the blame item on or off and change its text

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Toggle Git Blame Status Bar Item from the Command Palette flips the item on or off (default on), and a template setting rewrites its text from the tokens hash, hashShort, subject, authorName, authorEmail, authorDate and authorDateAgo.
- Commands: git.blame.toggleStatusBarItem
- Menus: commandPalette
- Keys: none found
- Settings: git.blame.statusBarItem.enabled; git.blame.statusBarItem.template
- Source: V\extensions\git\package.json (command, settings); V\extensions\git\src\commands.ts (toggleBlameStatusBarItem, _toggleBlameSetting); V\extensions\git\src\blame.ts (formatBlameInformationMessage); C bundle (extensions\git\package.json command git.blame.toggleStatusBarItem and both settings; extensions\git\dist\main.js toggleBlameStatusBarItem)
- Pieces:
  1. Running Toggle Git Blame Status Bar Item (`git.blame.toggleStatusBarItem`) from the Command Palette switches the blame item on or off.
  2. Setting `git.blame.statusBarItem.enabled` chooses whether the blame item is shown, with values on (default) and off.
  3. Setting `git.blame.statusBarItem.template` rewrites the text of the blame item, with the default `${authorName} (${authorDateAgo})`.
  4. The template accepts the tokens `hash`, `hashShort`, `subject`, `authorName`, `authorEmail`, `authorDate` and `authorDateAgo`.


## SS-5 Alerts and background activity

(both). Items showing something needs attention or is running: the notifications bell, Problems counts, Debug state, running tasks, progress. Cursor's bell is always right-most.

15 features.

#### SS-5.1 Notifications bell with unread dot

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A bell icon sits at the far right end of the bar (id status.notifications, name "Notifications", priority negative infinity); it turns into a bell with a dot when a notification arrived that you have not seen in the center, or when a notification with progress is running while the toasts and center are hidden, and opening the center clears the unread count; the tooltip reads "No Notifications", "No New Notifications", "1 New Notification", "{n} New Notifications", with "({n} in progress)" variants, and "Hide Notifications" while the center is open.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts (updateNotificationsCenterStatusItem, getTooltip, update); C bundle (module vs/workbench/browser/parts/notifications/notificationsStatus, id status.notifications on the right at negative infinity, icons bell and bell-dot, NLS keys noNotifications, oneNotification, notifications, noNotificationsWithProgress and the other progress variants)
- Pieces:
  1. A bell icon sits at the far right end of the status bar.
  2. The bell becomes a bell with a dot when a notification arrived that you have not seen in the center.
  3. The bell also gets the dot when a notification with progress is running while the toasts and the center are hidden.
  4. Opening the notification center clears the unread state and the dot.
  5. The tooltip reads "No Notifications", "No New Notifications", "1 New Notification" or "{n} New Notifications" to match what is in the center.
  6. While notifications with progress are running the tooltip adds an "({n} in progress)" variant of those texts.
  7. While the notification center is open the tooltip reads "Hide Notifications".

#### SS-5.2 Click the bell to show or hide the notification center

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking the bell opens the notification center above it, with a beak on the item pointing at the center, and clicking again hides it; Show Notifications also runs from the Command Palette or the key chord, and Escape hides the center while it is open.
- Commands: notifications.showList; notifications.hideList
- Menus: commandPalette
- Keys: ctrl+k ctrl+shift+n (notifications.showList); escape when NotificationsCenterVisibleContext (notifications.hideList)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts (command and showBeak); V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts (SHOW_NOTIFICATIONS_CENTER, HIDE_NOTIFICATIONS_CENTER, keybindings, palette items); C bundle (commands notifications.showList with CtrlCmd+K then CtrlCmd+Shift+N, notifications.hideList on Escape, labels "Show Notifications" and "Hide Notifications")
- Pieces:
  1. Clicking the bell opens the notification center above it, and clicking the bell again hides it.
  2. While the center is open the bell item shows a beak pointing at the center.
  3. Show Notifications (`notifications.showList`) opens the center from the Command Palette or from the chord `Ctrl+K Ctrl+Shift+N`.
  4. Pressing `Escape` while the center is open hides it (`notifications.hideList`), and Hide Notifications is also in the Command Palette.

#### SS-5.4 Status message text

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Workbench code can show a short text message in the bar (id status.message, name "Status Message", last item on the left), after an optional delay and for an optional time, replacing any earlier message; examples are "The extension host terminated unexpectedly. Restarting..." for 5 seconds and the message shown when an extension changes workspace folders for 10 seconds. It has no icon and no click action.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts (doSetStatusMessage); V\src\vs\workbench\services\notification\common\notificationService.ts (status); V\src\vs\workbench\services\extensions\electron-browser\nativeExtensionService.ts; V\src\vs\workbench\api\browser\mainThreadWorkspace.ts; C bundle (id status.message, NLS key status.message "Status Message" in module vs/workbench/browser/parts/notifications/notificationsStatus)
- Pieces:
  1. A short text message can appear as the last item on the left side of the bar, with no icon.
  2. A new message replaces any earlier message.
  3. A message can wait for a delay before it shows and can disappear after a set time, for example "The extension host terminated unexpectedly. Restarting..." for 5 seconds.
  4. The message shown when an extension changes the workspace folders stays for 10 seconds.
  5. The message item has no click action.

#### SS-5.5 Problem counts

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A left item (id status.problems, name "Problems", priority 50) shows an error icon with its count and a warning icon with its count, plus an info icon and count only when there are infos; counts above 999 shorten to "1K" to "9K" and above 9999 to "10K+", and the hover reads "Errors: 2, Warnings: 1" or "No Problems".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\markers\browser\markers.contribution.ts (MarkersStatusBarContributions.getMarkersItem, getMarkersText, getMarkersTooltip, packNumber); C bundle (module vs/workbench/contrib/markers/browser/markers.contribution, id status.problems, NLS keys totalErrors, totalWarnings, totalInfos, noProblems, manyProblems)
- Pieces:
  1. The Problems item on the left shows an error icon with the error count and a warning icon with the warning count.
  2. An info icon with its count appears only when there are infos.
  3. Counts above 999 shorten to "1K" through "9K", and counts above 9999 show as "10K+".
  4. The hover reads "Errors: 2, Warnings: 1" with the current counts, or "No Problems" when there are none.

#### SS-5.6 Open or close the Problems view from the item

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking the Problems item opens and focuses the Problems panel, or closes it when it is already visible.
- Commands: workbench.actions.view.toggleProblems
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\markers\browser\markers.contribution.ts (TOGGLE_MARKERS_VIEW_ACTION_ID action, item command); V\src\vs\workbench\contrib\markers\common\markers.ts; C bundle (command workbench.actions.view.toggleProblems on the status.problems item)
- Pieces:
  1. Clicking the Problems item runs Toggle Problems (`workbench.actions.view.toggleProblems`) and opens and focuses the Problems panel.
  2. Clicking the Problems item while the Problems panel is already visible closes the panel.

#### SS-5.7 Problems visibility off warning item

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When problems are turned off, a warning-colored whole-word icon item (id status.problemsVisibility, name "Problems Visibility", priority 49) appears right of the counts with the tooltip "Problems are turned off. Click to open settings." and a click opens Settings filtered to problems.visibility; it disappears when the setting is turned back on.
- Commands: workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: problems.visibility
- Source: V\src\vs\workbench\contrib\markers\browser\markers.contribution.ts (getMarkersItemTurnedOff, addStatusBarEntry); V\src\vs\workbench\browser\workbench.contribution.ts (problems.visibility); C bundle (id status.problemsVisibility, icon whole-word, kind warning, NLS keys status.problemsVisibility and status.problemsVisibilityOff)
- Pieces:
  1. When problems are turned off, a warning-colored item with a whole-word icon appears right of the Problems counts.
  2. The tooltip reads "Problems are turned off. Click to open settings."
  3. Clicking the item opens Settings filtered to `problems.visibility` (`workbench.action.openSettings`).
  4. The item disappears when the setting is turned back on.
  5. Setting `problems.visibility` chooses whether problems are visible throughout the workbench, with values on (default) and off.

#### SS-5.8 Current problem text

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With problems.showCurrentInStatus on (off by default) the left side shows the severity icon and first line of the problem under the cursor (id statusbar.currentProblem, name "Current Problem"); it disappears when the cursor leaves the problem and has no click action.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: problems.showCurrentInStatus
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (ShowCurrentMarkerInStatusbarContribution); V\src\vs\workbench\contrib\markers\browser\markers.contribution.ts (setting, default false); C bundle (id statusbar.currentProblem, setting problems.showCurrentInStatus default false, severity icons error, warning, info)
- Pieces:
  1. While the cursor is on a problem and `problems.showCurrentInStatus` is on, the left side shows the first line of that problem with an icon.
  2. The icon matches the severity of the problem: error, warning or info.
  3. Moving the cursor off the problem removes the item.
  4. The item has no click action.
  5. Setting `problems.showCurrentInStatus` chooses whether this item can appear, with values on and off (default).

#### SS-5.9 Debug item shows the selected launch configuration

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A left item with a debug icon and the name of the selected launch configuration (with the launch file name added when there is more than one) appears with the tooltip "Select and Start Debug Configuration" (id status.debug, name "Debug", priority 30); by default it shows only after the first debug session starts, and a setting makes it always or never show.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.showInStatusBar
- Source: V\src\vs\workbench\contrib\debug\browser\debugStatus.ts (DebugStatusContribution); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (debug.showInStatusBar, default onFirstSessionStart); C bundle (id status.debug at priority 30, setting debug.showInStatusBar with never, always, onFirstSessionStart)
- Pieces:
  1. A left item with a debug icon and the name of the selected launch configuration appears, with the launch file name added when there is more than one launch file.
  2. The tooltip reads "Select and Start Debug Configuration".
  3. By default the item shows only after the first debug session has started.
  4. Setting `debug.showInStatusBar` chooses when the item shows, with values `never`, `always` and `onFirstSessionStart` (default).
  5. With `debug.showInStatusBar` set to `always` the item shows before any debug session has started, and with `never` it does not show at all.

#### SS-5.10 Click the debug item to pick and start a configuration

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking the Debug item runs Select and Start Debugging, a picker of the launch configurations and dynamic debug providers; choosing one selects it and starts it.
- Commands: workbench.action.debug.selectandstart
- Menus: commandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\debug\browser\debugStatus.ts (command); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (SELECT_AND_START_ID); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (registerDebugCommandPaletteItem); C bundle (command workbench.action.debug.selectandstart on the status.debug item)
- Pieces:
  1. Clicking the Debug item opens a picker of the launch configurations and the dynamic debug providers.
  2. Choosing a row in the picker selects that configuration and starts it.
  3. Running Select and Start Debugging (`workbench.action.debug.selectandstart`) from the Command Palette opens the same picker.

#### SS-5.11 Running Tasks count

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While tasks are running a tools icon with the count appears right of the Problems item (id status.runningTasks, name "Running Tasks", tooltip "Show Running Tasks"); clicking it opens the Show Running Tasks picker, and the item disappears when no task is running.
- Commands: workbench.action.tasks.showTasks
- Menus: MenubarTerminalMenu (group Manage, "Show Running Tasks...")
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (TaskStatusBarContributions._updateRunningTasksStatus, menu items); C bundle (id status.runningTasks placed next to status.problems at priority 50, command workbench.action.tasks.showTasks)
- Pieces:
  1. While tasks are running, a tools icon with the number of running tasks appears on the left, right of the Problems item.
  2. The tooltip reads "Show Running Tasks".
  3. Clicking the item opens the Show Running Tasks picker (`workbench.action.tasks.showTasks`).
  4. The item disappears when no task is running.
  5. The Terminal menu row "Show Running Tasks..." in the Manage group opens the same picker.

#### SS-5.12 Progress message with a spinner

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Work that reports window progress, such as a running build task ("Building...") or a language feature, puts a spinning item at the far end of the left side (id status.progress, name "Progress Message") with "title: message" text and a tooltip that adds the source in brackets; it appears after 150 ms, stays at least 150 ms, and a click runs the command the work gave (the build item opens Show Running Tasks).
- Commands: workbench.action.tasks.showTasks
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (withWindowProgress, updateWindowProgress); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (Building... progress); C bundle (id status.progress at negative max value, window progress location 10, "Building..." message with command workbench.action.tasks.showTasks)
- Pieces:
  1. Work that reports window progress puts a spinning item at the far end of the left side, with the text "title: message".
  2. The tooltip repeats the text and adds the source in brackets.
  3. The item appears only after 150 ms of progress and stays on screen for at least 150 ms, so very short work does not flash.
  4. Clicking the item runs the command that the work supplied.
  5. While a build task runs the item reads "Building..." and clicking it opens the Show Running Tasks picker (`workbench.action.tasks.showTasks`).

#### SS-5.13 Window zoom indicator and mini toolbar

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: On the right side a prominent zoom-out or zoom-in icon appears whenever this window's zoom differs from the configured level (id status.windowZoom, name "Window Zoom", priority 102); its hover shows a mini toolbar with Zoom Out, the zoom level number (tooltip "Zoom Level: 2 (144%)"), Zoom In, Reset and a Settings gear that opens window.zoom, and every window has its own item. Desktop only.
- Commands: workbench.action.zoomIn; workbench.action.zoomOut; workbench.action.zoomReset; workbench.action.openSettings
- Menus: none found
- Keys: ctrl+= (also ctrl+shift+= and ctrl+numpad+) for zoom in; ctrl+- (also ctrl+shift+- and ctrl+numpad-) for zoom out; ctrl+numpad0 for reset
- Settings: window.zoomLevel; window.zoomPerWindow
- Source: V\src\vs\workbench\electron-browser\window.ts (ZoomStatusEntry, updateWindowZoomStatusEntry); V\src\vs\workbench\electron-browser\actions\windowActions.ts (zoom commands and keys); V\src\vs\workbench\electron-browser\desktop.contribution.ts (settings); C bundle (module vs/workbench/electron-sandbox/window, id status.windowZoom kind prominent priority 102, icons zoom-in and zoom-out, NLS key zoomNumber, commands and settings)
- Pieces:
  1. On the desktop a prominent zoom-out icon appears on the right side when this window is zoomed out from the configured level, and a zoom-in icon when it is zoomed in.
  2. When the window zoom matches the configured level the item is hidden.
  3. Hovering the item opens a mini toolbar with Zoom Out, the zoom level number, Zoom In, Reset and a Settings gear.
  4. The zoom level number has the tooltip "Zoom Level: 2 (144%)" with the current level and percentage.
  5. The Zoom Out, Zoom In and Reset buttons run `workbench.action.zoomOut`, `workbench.action.zoomIn` and `workbench.action.zoomReset` and the item updates at once.
  6. Pressing `Ctrl+=` (also `Ctrl+Shift+=` and `Ctrl+Numpad+`) zooms in, `Ctrl+-` (also `Ctrl+Shift+-` and `Ctrl+Numpad-`) zooms out, and `Ctrl+Numpad0` resets the zoom.
  7. The Settings gear opens Settings on `window.zoom` (`workbench.action.openSettings`).
  8. Setting `window.zoomPerWindow` chooses whether Zoom In and Zoom Out change all windows or only the active one, with values on (default) and off, and each window has its own item.
  9. Setting `window.zoomLevel` chooses the default zoom level for all windows, a number with default 0 where each step is 20 percent larger or smaller.

#### SS-5.14 Show Output quietly indicator

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When the Output panel is marked in workbench.view.showQuietly and an extension asks to show an output channel while the panel is hidden, a prominent output icon appears on the right next to the bell (id status.view.showQuietly, name "Show Output", tooltip "Show {channel} Output Channel") and a click opens that channel; nothing is shown by default because the setting starts empty.
- Commands: workbench.action.output.show.{channelId}
- Menus: none found
- Keys: none found
- Settings: workbench.view.showQuietly
- Source: V\src\vs\workbench\api\browser\mainThreadOutputService.ts (_showChannelQuietly); V\src\vs\workbench\browser\workbench.contribution.ts (workbench.view.showQuietly); C bundle (id status.view.showQuietly placed left of status.notifications, setting workbench.view.showQuietly with workbench.panel.output)
- Pieces:
  1. When an extension asks to show an output channel while the Output panel is hidden and the panel is marked in `workbench.view.showQuietly`, a prominent output icon appears on the right next to the bell instead.
  2. The tooltip reads "Show {channel} Output Channel".
  3. Clicking the item opens that channel (`workbench.action.output.show.{channelId}`).
  4. Setting `workbench.view.showQuietly` is an object with one key, `workbench.panel.output`, which turns the quiet indicator on or off for the Output panel.
  5. The setting starts empty, so no item is shown by default.

#### SS-5.15 Pty Host unresponsive warning

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: If the terminal's pty host process stops answering, an amber warning item with a disconnect icon and "Pty Host" appears on the left (id ptyHostStatus, name "Pty Host Status"); the tooltip says terminals may stop working and a click restarts the pty host, and the item goes away when the host answers again.
- Commands: workbench.action.terminal.restartPtyHost
- Menus: commandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminal\browser\baseTerminalBackend.ts (onPtyHostUnresponsive entry); V\src\vs\workbench\contrib\terminalContrib\developer\browser\terminal.developer.contribution.ts (Restart Pty Host); C bundle (id ptyHostStatus, kind warning, command workbench.action.terminal.restartPtyHost, title "Restart Pty Host")
- Pieces:
  1. When the terminal pty host stops answering, an amber warning item with a disconnect icon and the text "Pty Host" appears on the left.
  2. The tooltip says that terminals may stop working.
  3. Clicking the item restarts the pty host, and Restart Pty Host (`workbench.action.terminal.restartPtyHost`) does the same from the Command Palette.
  4. The item goes away when the pty host answers again.

#### SS-5.16 Terminal session recording item

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Developer: Record Terminal Session opens a terminal and shows a spinning "Recording terminal session..." item on the left (id recordSession) until the terminal has been idle for 5 seconds, then copies the recording as JSON to the clipboard; the item has no click action.
- Commands: workbench.action.terminal.recordSession
- Menus: commandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\contrib\terminalContrib\developer\browser\terminal.developer.contribution.ts (RecordSession); V\src\vs\workbench\contrib\terminalContrib\developer\common\terminal.developer.ts; C bundle (command workbench.action.terminal.recordSession, title "Record Terminal Session", entry id recordSession)
- Pieces:
  1. Running Developer: Record Terminal Session (`workbench.action.terminal.recordSession`) from the Command Palette opens a terminal.
  2. While it records, a spinning item reading "Recording terminal session..." shows on the left.
  3. When the terminal has been idle for 5 seconds the recording ends and the item goes away.
  4. At the end the recording is copied to the clipboard as JSON.
  5. The recording item has no click action.


## SS-6 AI suggestions and AI stats

(both) [boundary]. Stock has the inline suggestions entry and an AI stats gauge. Cursor swaps in its own: Cursor Tab, Tab Stats, Agent Stats, a recent-commit AI score, a workspace name item, and a memory warning. The Tab controls are AC-8.

16 features.

#### SS-6.1 Inline suggestions row in the Language Status popup

- Tag: both. Evidence: source. Boundary: yes - the row says whether the suggestion provider has a suggestion right now, which is provider output. Only the row is listed.. Cursor: changed (same labels; class id `vs.editor.contrib.inlineCompletionLanguageStatusBarContribution`, focused editor only, no AI-hidden check found; `chat.disableAIFeatures` has 0 hits)
- What: A row named "Inline Suggestions" (detail text "Inline suggestions") inside the Language Status popup, the bracket icon on the right of the bar that SS-3 owns. Its label follows the state of the editor: "Loading..." with a spinner, "Inline completion available", "Inline edit available", or "No inline suggestion available", the last three with a Copilot icon in front. The row has no command link. Like every language status row it has an "Add to Status Bar" pin, and once pinned it gets its own bar item that shows only an icon: a lightbulb, a lightbulb with a sparkle, or a circle with a slash (the busy spinner while loading). It is added only for editors that are not read-only. Stock hides it when AI features are turned off by the setting below. In Cursor the same labels exist, but the code is an editor contribution that works per editor and only for the focused one, and I found no AI-hidden check in it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: chat.disableAIFeatures
- Source: V\src\vs\workbench\contrib\inlineCompletions\browser\inlineCompletionLanguageStatusBarContribution.ts; V\src\vs\workbench\contrib\inlineCompletions\browser\inlineCompletions.contribution.ts; V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts; V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts; V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts; V\src\vs\workbench\services\chat\common\chatEntitlementService.ts; C bundle (NLS module `contrib/inlineCompletions/browser/inlineCompletionLanguageStatusBarContribution`, messages 7443 to 7448, `addStatus({...id:"inlineSuggestions"...})`)
- Pieces:
  1. The Language Status popup has a row named "Inline Suggestions" with the detail text "Inline suggestions", shown only for editors that are not read-only.
  2. While the editor is waiting, the row label reads "Loading..." with a spinner.
  3. When an inline completion is available the label reads "Inline completion available" with a Copilot icon in front, and when an inline edit is available it reads "Inline edit available" with the same icon.
  4. When nothing is available the label reads "No inline suggestion available" with the Copilot icon in front.
  5. The row has no command link.
  6. Like every language status row it can be pinned with "Add to Status Bar", and once pinned its bar item shows only an icon: a lightbulb for a completion, a lightbulb with a sparkle for an edit, a circle with a slash for none, and the spinner while loading.
  7. Stock hides the row when AI features are turned off with `chat.disableAIFeatures`. [stock only]
  8. In Cursor the row has the same labels but follows only the focused editor, and nothing hides it when AI features are turned off, because `chat.disableAIFeatures` does not exist there. [cursor only]

#### SS-6.2 AI usage gauge in the status bar

- Tag: vscode. Evidence: source. Boundary: yes - the gauge shows how much of the work came from AI.. Cursor: missing (0 hits for `editor.aiStats.enabled`, `aiStatsStatusBar` and the label "AI Usage Statistics")
- What: A right-side status bar item (priority 100, name "Inline Suggestions", aria label "Inline suggestions status bar") that has no text. It draws a thin bar, 50 by 6 pixels with rounded ends, that fills from the left to show the current AI share. It exists only when the setting `editor.aiStats.enabled` is on (default off, marked experimental) and AI features are not hidden. It has no click command. Hovering it opens the panel described in SS-6.3. It shares priority 100 with File Information, so their order comes from the id hash.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.aiStats.enabled; chat.disableAIFeatures
- Source: V\src\vs\workbench\contrib\editTelemetry\browser\editStats\aiStatsStatusBar.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editStats\aiStatsFeature.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editTelemetry.contribution.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editTelemetryContribution.ts; V\src\vs\workbench\contrib\editTelemetry\browser\settingIds.ts
- Pieces:
  1. With `editor.aiStats.enabled` on, an item with no text appears on the right side of the bar and draws a thin bar, 50 by 6 pixels with rounded ends, that fills from the left to show the current AI share. [stock only]
  2. The item has the name "Inline Suggestions" and the screen reader label "Inline suggestions status bar". [stock only]
  3. The item has no click action, and hovering it opens the AI Usage Statistics panel. [stock only]
  4. Setting `editor.aiStats.enabled` chooses whether the item exists, with values on and off (default), and it is marked experimental. [stock only]
  5. The item is hidden when AI features are turned off with `chat.disableAIFeatures`. [stock only]

#### SS-6.3 Read the AI usage chart in the gauge hover

- Tag: vscode. Evidence: source. Boundary: yes - the panel shows AI usage numbers.. Cursor: missing (0 hits for `AI Usage Statistics`, "AI vs Typing Average" and `aiStatsStatusBar`)
- What: Hovering the gauge shows a panel headed "AI Usage Statistics" with two lines, "AI vs Typing Average: {n}%" and "Accepted inline suggestions today: {n}". Below a divider is a chart with a header that reads "AI Rate by Day" or "AI Rate by Session". Two small icon buttons at the right of that header, Days (calendar icon) and Sessions (flat list icon), switch the chart, and the active one is highlighted. An empty chart says "No data yet". The panel is built as an element tooltip, so it is rich content in the bar's hover rules (SS-1).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.aiStats.enabled
- Source: V\src\vs\workbench\contrib\editTelemetry\browser\editStats\aiStatsStatusBar.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editStats\aiStatsChart.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editStats\media.css
- Pieces:
  1. Hovering the gauge shows a panel headed "AI Usage Statistics". [stock only]
  2. Two lines under the header read "AI vs Typing Average: {n}%" and "Accepted inline suggestions today: {n}". [stock only]
  3. Below a divider a bar chart shows one bar per day or per session, with 0%, 50% and 100% marks on its vertical axis. [stock only]
  4. The chart header reads "AI Rate by Day" in the Days view, which is the view it opens in, and "AI Rate by Session" in the Sessions view. [stock only]
  5. Two small icon buttons at the right of the chart header, Days with a calendar icon and Sessions with a flat list icon, switch the chart, and the active one is highlighted. [stock only]
  6. In the Days view the bars have date labels along the bottom edge. [stock only]
  7. When there is nothing to chart the chart area reads "No data yet". [stock only]

#### SS-6.4 Open the AI stats setting from the gauge hover

- Tag: vscode. Evidence: source. Boundary: yes - it sits in the AI usage panel and only opens a setting.. Cursor: missing (the panel and `editor.aiStats.enabled` have 0 hits; the command id `workbench.action.openSettings` appears in the bundle for other callers)
- What: The header of the AI Usage Statistics panel has a gear icon button with the tooltip "Configure". Clicking it opens the Settings editor filtered to the `editor.aiStats.enabled` setting (the Settings editor itself is ED-8).
- Commands: workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: editor.aiStats.enabled
- Source: V\src\vs\workbench\contrib\editTelemetry\browser\editStats\aiStatsStatusBar.ts
- Pieces:
  1. The header of the AI Usage Statistics panel has a gear icon button at its right with the tooltip "Configure". [stock only]
  2. Clicking the gear opens the Settings editor filtered to `editor.aiStats.enabled` (`workbench.action.openSettings`). [stock only]

#### SS-6.5 Developer: Edit Source Tracking item

- Tag: vscode. Evidence: source. Boundary: yes - the hover lists where edits came from, including AI edits.. Cursor: missing (0 hits for `telemetry.editStats` and "Edit Source Tracking")
- What: Developer item, off by default. When `telemetry.editStats.showStatusBar` is on, and `telemetry.editStats.enabled` is on (default on) and usage telemetry is allowed, a right-side item (id `editTelemetry`, priority 100, tooltip "Edit Source Tracking") reads "{n} chars inserted" with an edit icon. Its hover shows one JSON block per changed file (the last three), a "View Details" link, a color legend of edit sources, and a "Toggle decorations: Click here" link. The item click and "View Details" run `editTelemetry.showDebugDetails`, which I could not find registered anywhere (see Unclear). "Toggle decorations" runs `editTelemetry.toggleDebugDecorations`, which flips `telemetry.editStats.showDecorations`.
- Commands: editTelemetry.showDebugDetails; editTelemetry.toggleDebugDecorations
- Menus: none found
- Keys: none found
- Settings: telemetry.editStats.showStatusBar; telemetry.editStats.enabled; telemetry.editStats.showDecorations
- Source: V\src\vs\workbench\contrib\editTelemetry\browser\telemetry\editSourceTrackingFeature.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editTelemetry.contribution.ts; V\src\vs\workbench\contrib\editTelemetry\browser\settings.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editTelemetryContribution.ts
- Pieces:
  1. With `telemetry.editStats.showStatusBar` on, a right-side item with an edit icon and the text "{n} chars inserted" appears, with the tooltip "Edit Source Tracking". [stock only]
  2. Hovering the item shows one JSON block for each of the last three changed files. [stock only]
  3. The hover also holds a "View Details" link and a color legend of the edit sources. [stock only]
  4. Clicking the item or the "View Details" link runs `editTelemetry.showDebugDetails`. [stock only]
  5. The hover link "Toggle decorations: Click here" runs `editTelemetry.toggleDebugDecorations`, which flips `telemetry.editStats.showDecorations`. [stock only]
  6. Setting `telemetry.editStats.showStatusBar` chooses whether the item is shown, with values on and off (default), and it is marked experimental. [stock only]
  7. The item also needs `telemetry.editStats.enabled` on, which is its default, and usage telemetry to be allowed. [stock only]
  8. Setting `telemetry.editStats.showDecorations` chooses whether colored backgrounds for each edit source are drawn in the editor, with values on and off (default). [stock only]

#### SS-6.6 Cursor Tab status item (cross-reference only)

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cross-reference only. Cursor Tab shows in the bar as a right-side item named "Cursor Tab" with priority 1000, the highest on the bar, so it sits left-most of the right-side items. Its text is "Cursor Tab", "Cursor Tab (Snoozed)" or "Cursor Tab (Ignored)", and it is dimmed when Tab is off for the open file. Hovering opens a panel and clicking it while signed in keeps the panel open, while signed out the click runs Log In to Cursor. Nothing about the item is described again here. The controls are in AC-8 - AC-8.1 text states and dimmed look, AC-8.2 the hover panel, AC-8.3 turn Tab on or off, AC-8.4 turn Tab off for a language, AC-8.5 pick the Tab model, AC-8.6 snooze, AC-8.7 unsnooze and time left, AC-8.8 why Tab is not available, AC-8.9 log in. Hiding the item from the bar's right-click menu is SS-1. Seen running by the pass 1 status bar check as "Cursor Tab".
- Commands: statusBar.entry.showTooltip; editor.cpp.login
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (`addEntry(this.statusBarElementProps(),"Cursor Tab",1,1e3)`; entry fields `name:"Cursor Tab"`, `cssClass`, `tooltip`; class `copilot-plus-plus-status-off`); V has no entry for it
- Pieces:
  1. The right side of the bar has an item named "Cursor Tab" with the highest priority on the bar, so it sits left-most of the right-side items, and its text states and dimmed look are detailed in AC-8.1. [cursor only]
  2. Hovering the item opens a panel, which is detailed in AC-8.2. [cursor only]
  3. Clicking the item while signed in keeps the panel open (`statusBar.entry.showTooltip`), and while signed out it runs Log In to Cursor (`editor.cpp.login`), as detailed in AC-8.9. [cursor only]

#### SS-6.7 Tab Stats item

- Tag: cursor. Evidence: bundle. Boundary: yes - the numbers show Cursor Tab use.. Cursor: cursor only
- What: A right-side item named "AI Code Tracking Stats - Tab" (priority 100) with a tab icon. The text reads "Tab Stats: {accepted}/{suggested} ({percent}%)", for example "Tab Stats: 0/0 (0%)", and the hover reads "Tab AI Stats (Today): {accepted}/{suggested} lines ({percent}%)". The aria label says the same in words. It has no click command. The tooltip names today, and there is a command to zero it (SS-6.10). The three AI Code Tracking items come from one class that adds them at startup in normal windows, not in local mode and not under the smoke test driver. I found no setting that turns the item off, so hiding it is the bar's right-click menu (SS-1). It shares priority 100 with File Information, so their order comes from the id hash.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (NLS module `contrib/analytics/browser/aiCodeTrackingStatusBar`; `addEntry(this.tabEntry(),"aiCodeTracking.stats.tab",1,100)`; `tabEntry()` fields)
- Pieces:
  1. A right-side item with a tab icon reads "Tab Stats: {accepted}/{suggested} ({percent}%)", for example "Tab Stats: 0/0 (0%)". [cursor only]
  2. The hover reads "Tab AI Stats (Today): {accepted}/{suggested} lines ({percent}%)". [cursor only]
  3. The item is named "AI Code Tracking Stats - Tab" and its screen reader label says the same as the hover in words. [cursor only]
  4. The item has no click action. [cursor only]
  5. The item is added when a normal window starts, not in local mode, and no setting turns it off, so it can only be hidden from the right-click menu of the bar. [cursor only]

#### SS-6.8 Agent Stats item

- Tag: cursor. Evidence: bundle. Boundary: yes - the numbers show agent use.. Cursor: cursor only
- What: A right-side item named "AI Code Tracking Stats - Agent" (priority 99) with a speech bubble icon. The text reads "Agent Stats: {accepted}/{suggested} ({percent}%)" and the hover reads "Agent AI Stats (Today): {accepted}/{suggested} lines ({percent}%)". It sits just right of Tab Stats. No click command. Same creation rules and same lack of an off switch as SS-6.7. Seen running by the pass 1 status bar check.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (NLS module `contrib/analytics/browser/aiCodeTrackingStatusBar`; `addEntry(this.composerEntry(),"aiCodeTracking.stats.composer",1,99)`; `composerEntry()` fields)
- Pieces:
  1. A right-side item with a speech bubble icon reads "Agent Stats: {accepted}/{suggested} ({percent}%)". [cursor only]
  2. The hover reads "Agent AI Stats (Today): {accepted}/{suggested} lines ({percent}%)". [cursor only]
  3. The item is named "AI Code Tracking Stats - Agent" and sits just right of the Tab Stats item. [cursor only]
  4. The item has no click action. [cursor only]
  5. The item is added when a normal window starts and no setting turns it off, so it can only be hidden from the right-click menu of the bar. [cursor only]

#### SS-6.9 Recent commit AI score item

- Tag: cursor. Evidence: bundle. Boundary: yes - the item reports how much of a commit came from Tab and the agent.. Cursor: cursor only
- What: A right-side item named "AI Code Tracking - Recent Commit" (priority 98) with a commit icon, the right-most of the three AI Code Tracking items. Until a commit is scored it reads "No commit scored", and the hover and aria label say "No commit has been scored yet". After that it reads the 7 character commit hash and "{n}% AI" (seen running as "b90411c 100.00% AI"). The hover is a block of lines: "Most Recent Commit Scored:", the hash, the commit message cut to 40 characters with "...", a "Repo:" line and a "Branch:" line when known, a rule line, "AI-Generated: {n}% ({lines} lines)", a "Tab:" line and a "Composer:" line each with added and deleted counts, and "Total Changes: {a} added, {d} deleted". The hover still uses the older word Composer where the bar says Agent. No click command. Same creation rules as SS-6.7.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (NLS module `contrib/analytics/browser/aiCodeTrackingStatusBar`; `addEntry(this.recentCommitEntry(),"aiCodeTracking.stats.recentCommit",1,98)`; `recentCommitEntry()` fields)
- Pieces:
  1. A right-side item with a commit icon sits right-most of the three AI Code Tracking items and reads "No commit scored" until a commit has been scored, with the hover and screen reader label "No commit has been scored yet". [cursor only]
  2. After a commit is scored the item reads the 7 character commit hash and "{n}% AI", for example "b90411c 100.00% AI". [cursor only]
  3. The hover starts with "Most Recent Commit Scored:", the commit hash, and the commit message cut to 40 characters with "...". [cursor only]
  4. The hover then shows a "Repo:" line and a "Branch:" line when they are known. [cursor only]
  5. The hover ends with a rule line, "AI-Generated: {n}% ({lines} lines)", a "Tab:" line and a "Composer:" line each with added and deleted counts, and "Total Changes: {a} added, {d} deleted". [cursor only]
  6. The item is named "AI Code Tracking - Recent Commit" and has no click action. [cursor only]
  7. The item is added when a normal window starts and no setting turns it off. [cursor only]

#### SS-6.10 Reset AI Code Tracking Stats

- Tag: cursor. Evidence: bundle. Boundary: yes - it clears the counters of Tab and agent use.. Cursor: cursor only
- What: Command Palette command "AI Code Tracking: Reset AI Code Tracking Stats" (category "AI Code Tracking"). It sets today's numbers for Tab and Agent back to zero in the current window, and the Tab Stats and Agent Stats items refresh to "0/0 (0%)" at once. In the code I read it does not touch the recent commit item. It shows in the palette for everyone, with no precondition and no default key.
- Commands: aiCodeTracking.resetDailyStats
- Menus: CommandPalette (f1 true)
- Keys: none found
- Settings: none found
- Source: C bundle (NLS module `contrib/analytics/browser/aiCodeTrackingStatusBar`, messages 4831 and 4832; class with `id:"aiCodeTracking.resetDailyStats"`, `f1:!0`; `resetDailyStats()`)
- Pieces:
  1. The Command Palette lists "AI Code Tracking: Reset AI Code Tracking Stats" (`aiCodeTracking.resetDailyStats`) for everyone, with no precondition and no default key. [cursor only]
  2. Running it sets today's Tab and Agent numbers back to zero in the current window. [cursor only]
  3. The Tab Stats and Agent Stats items refresh to "0/0 (0%)" at once. [cursor only]
  4. The recent commit item stays as it was. [cursor only]

#### SS-6.11 Workspace name item

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A left-side item named "Workspace Name" (priority 50) that shows the name of what you have open as plain text, with the hover "Current workspace: {name}" and the aria label "Workspace: {name}". It has no click command. When the first folder has a source control repository, the text is the folder name if the repository root is that folder, otherwise the name of the repository root folder. With no repository it is the workspace label. The item is removed when there is no name, and it updates when the workspace is renamed, folders change, a repository is added, removed or updated, or label formats change. Seen running as "cursor".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (NLS module `contrib/workspace/browser/workspace.view.contribution`, messages 11884 to 11886; class with `entryId="status.workspaceName"`, `addEntry(t,this.entryId,0,50)`); V has no hit for `status.workspaceName`; C bundle (module vs/workbench/contrib/workspace/browser/workspace.view.contribution, class that registers id status.workspaceName at priority 50, NLS keys status.workspaceName, status.workspaceNameAria, status.workspaceNameTooltip; searched V and N and found no such id)
- Pieces:
  1. A left-side item named "Workspace Name" shows the name of what you have open as plain text. [cursor only]
  2. When the first folder has a source control repository, the text is the folder name if the repository root is that folder, and otherwise the name of the repository root folder. [cursor only]
  3. With no repository the text is the workspace label. [cursor only]
  4. The hover reads "Current workspace: {name}" and the screen reader label reads "Workspace: {name}". [cursor only]
  5. The item has no click action. [cursor only]
  6. The item is removed when there is no name. [cursor only]
  7. The item updates when the workspace is renamed, when folders change, when a repository is added, removed or updated, and when label formats change. [cursor only]
  8. A plain text item on the left side shows the repository folder name of the first workspace folder. [cursor only]
  9. The item refreshes when workspace folders or repositories change. [cursor only]

#### SS-6.12 High memory warning item

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A left-side item that shows a row of siren and warning emoji around the words "HIGH MEMORY USAGE". Its name is "Memory WARNING Indicator" or "Memory CRITICAL Indicator", and its hover and aria label read "{WARNING or CRITICAL} Process Memory Usage (JS: {n}MB) exceeds threshold ({m}MB). Consider taking a heap snapshot." It has no click command. It appears only when the memory monitor config flag `show_status_entry` is on and the memory use is over the warning level, and it goes away when use drops. The built-in fallback for that flag is off (fallback base threshold 1536 MB, critical offset 512 MB), and the flag is part of the app's server-sent config, not a user setting, so a normal user may never see it. Not seen running.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (NLS module `contrib/memoryMonitor/electron-sandbox/memoryMonitor.contribution`; `addEntry(p,"workbench.memoryMonitor.indicator",0)`; `memory_monitor_config` with fallback values `show_status_entry:!1`); C bundle (module vs/workbench/contrib/memoryMonitor/electron-sandbox/memoryMonitor.contribution, id workbench.memoryMonitor.indicator, config flag show_status_entry, tooltip "Process Memory Usage (JS: {n}MB) exceeds threshold ({m}MB). Consider taking a heap snapshot."; searched V and N and found no such id)
- Pieces:
  1. A left-side item shows a row of siren and warning emoji around the words "HIGH MEMORY USAGE". [cursor only]
  2. The item is named "Memory WARNING Indicator" or "Memory CRITICAL Indicator" to match the level. [cursor only]
  3. The hover and the screen reader label read "{WARNING or CRITICAL} Process Memory Usage (JS: {n}MB) exceeds threshold ({m}MB). Consider taking a heap snapshot." [cursor only]
  4. The item has no click action. [cursor only]
  5. The item appears only while the memory monitor flag `show_status_entry` is on and the memory use is over the warning level, and it goes away when use drops. [cursor only]
  6. The flag comes from the app's server config and not from a user setting, and its built-in fallback is off, so a normal user may never see the item. [cursor only]
  7. When the memory use of the window passes the warning threshold and a remote config flag allows it, a left item shows siren and warning emoji around the words "HIGH MEMORY USAGE". [cursor only]
  8. The hover gives the memory used in MB, the threshold in MB, and a hint to take a heap snapshot. [cursor only]
  9. The memory monitor flag `show_status_entry` comes from the app's server config and is not a user setting, so there is nothing in Settings to turn the item on or off. [cursor only]

#### SS-6.13 Developer: see and switch the backend server

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Developer only. In a development build, or for a developer user, a right-side item (name "currentServer", priority 200, prominent text color) reads "Server: {name}" with the hover "Click to switch backend server". Clicking runs `cursor.selectBackend` ("Select Backend Server" in the palette), which opens a pick list with the placeholder "Select backend server to use" and the current row marked "Current". Choosing a row switches the server and may ask you to sign in again. In a production build for a non-developer the command only warns "Backend switching is not available in production builds." Nine more palette commands titled "Switch to {name} Backend" do the same switch without the list, each with a chord. The item ties with the Git blame item at priority 200, so their order comes from the id hash.
- Commands: cursor.selectBackend; debug.switchToprodBackend; debug.switchTostagingBackend; debug.switchTostagingLocalWebsiteBackend; debug.switchTolocalBackend; debug.switchTofullLocalBackend; debug.switchTofullLocalincludingcppBackend; debug.switchTolocalExceptEmbeddingsBackend; debug.switchTodevStagingEverythingBackend; debug.switchTolocalStagingBackend
- Menus: CommandPalette (f1 true, precondition isDevelopment or isDevUser)
- Keys: WinCtrl+Shift+Alt+1 (debug.switchToprodBackend); WinCtrl+Shift+Alt+2 (staging); WinCtrl+Shift+Alt+3 (stagingLocalWebsite); WinCtrl+Shift+Alt+4 (local); WinCtrl+Shift+Alt+5 (fullLocal); WinCtrl+Shift+Alt+6 (fullLocalincludingcpp); WinCtrl+Shift+Alt+7 (localExceptEmbeddings); WinCtrl+Shift+Alt+8 (devStagingEverything); WinCtrl+Shift+Alt+9 (localStaging); WinCtrl is the Windows key on Windows and Ctrl on macOS; no `when`, weight 200
- Settings: none found
- Source: C bundle (`updateServerStatusBarOnlyRunsOnLocal`, `addEntry(t,"status.currentServer",1,200)`; class with `id:"cursor.selectBackend"`; factory `fQ_()` that builds `debug.switchTo${name}Backend`; key table with `Digit1` to `Digit9`; module name `cursorCredsService.js`)
- Pieces:
  1. In a development build, or for a developer user, a right-side item in a prominent text color reads "Server: {name}" with the hover "Click to switch backend server". [cursor only]
  2. Clicking the item runs Select Backend Server (`cursor.selectBackend`), which opens a pick list with the placeholder "Select backend server to use". [cursor only]
  3. The row of the server in use is marked "Current". [cursor only]
  4. Choosing a row switches the server and may ask you to sign in again. [cursor only]
  5. Select Backend Server is listed in the Command Palette only when `isDevelopment` or `isDevUser` is true. [cursor only]
  6. In a production build for a non-developer the command only warns "Backend switching is not available in production builds." [cursor only]
  7. Nine more Command Palette commands titled "Switch to {name} Backend" switch the server without showing the list. [cursor only]
  8. The nine commands have the chords `WinCtrl+Shift+Alt+1` to `WinCtrl+Shift+Alt+9`, for prod, staging, stagingLocalWebsite, local, fullLocal, fullLocalincludingcpp, localExceptEmbeddings, devStagingEverything and localStaging in that order, where WinCtrl is the Windows key on Windows and Ctrl on macOS. [cursor only]

#### SS-6.14 Developer: report a bug or lag from the status bar

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Internal only. A right-side item "[Internal] Report" (priority 50) with a bug icon and the hover "Report a bug or lag with automatic session context". It shows only for internal users with the feature gate `internal_session_recording_status_bar`, and it acts on mouse down, not on click. It runs `workbench.action.sessionRecording.reportMenu` ("Report Bug or Lag"), which opens a pick list titled "[Internal] Report" with the placeholder "What do you want to report?" and two rows: "Report Bug" (detail "Flag this session and add context.") and "Report Lag" (detail "Send a one-click lag report."). Each row runs its own command: `workbench.action.sessionRecording.flagAndAddContext` ("Report Bug", opens a report dialog that is not described here) and `workbench.action.sessionRecording.reportLag` ("Report Lag"). All three commands are in the palette under the Developer category when the context key `isDevUser` is true.
- Commands: workbench.action.sessionRecording.reportMenu; workbench.action.sessionRecording.flagAndAddContext; workbench.action.sessionRecording.reportLag
- Menus: CommandPalette (f1 true, precondition isDevUser, category Developer)
- Keys: none found
- Settings: none found
- Source: C bundle (NLS messages 11403 to 11417; class `workbench.contrib.sessionRecordingStatusBar` with `addEntry(e,"status.sessionRecording",1,50)` and `commandExecutionTrigger:"mousedown"`; `shouldShow()` gate)
- Pieces:
  1. For internal users with the feature gate `internal_session_recording_status_bar`, a right-side item "[Internal] Report" with a bug icon appears, with the hover "Report a bug or lag with automatic session context". [cursor only]
  2. The item acts when the mouse button goes down, not on a click. [cursor only]
  3. Pressing the item runs Report Bug or Lag (`workbench.action.sessionRecording.reportMenu`), which opens a pick list titled "[Internal] Report" with the placeholder "What do you want to report?". [cursor only]
  4. The list has two rows: "Report Bug" with the detail "Flag this session and add context." and "Report Lag" with the detail "Send a one-click lag report." [cursor only]
  5. The "Report Bug" row runs `workbench.action.sessionRecording.flagAndAddContext`, which opens a report dialog. [cursor only]
  6. The "Report Lag" row runs `workbench.action.sessionRecording.reportLag`. [cursor only]
  7. All three commands are listed in the Command Palette under the Developer category when `isDevUser` is true. [cursor only]

#### SS-6.15 Developer: score a commit for AI content

- Tag: cursor. Evidence: bundle. Boundary: yes - the results report how much of a commit came from Tab and the agent.. Cursor: cursor only
- What: Developer commands that work only in a development build (context key `isDevelopment`). "AI Tracking: Score Commit SHA for AI Content" opens an input box (title "Score Commit for AI Content", placeholder "Enter commit hash (e.g., abc123...)"), asks which repository when there is more than one, asks "Report metrics to analytics?" with No and Yes, shows "Scoring commit ..." and then a notice "Commit scored! AI: {n}% ({a}/{b} lines). Check output channel for details.", and writes the details to the "aiCodeTracking" Output channel. "AI Tracking: Score Recent Commits" scores several recent commits (its screens were not read). "[Debug] Score Commit for AI Content" is not in the palette and instead appears as a row in the right-click menu of a Source Control history item. "[Debug] Score Commit with Details" is a hidden command that other code calls.
- Commands: developer.aiCodeTracking.scoreCommit; developer.aiCodeTracking.scoreMultipleCommits; developer.aiCodeTracking.scoreCommitInternal; developer.aiCodeTracking.scoreCommitWithDetails
- Menus: CommandPalette (f1 true, precondition isDevelopment); SCMHistoryItemContext (group 1_ai, order 1, when isDevelopment)
- Keys: none found
- Settings: none found
- Source: C bundle (NLS module `contrib/analytics/browser/aiCodeTrackingActions.contribution`, messages 4829 and 4830; classes with the four ids; context key `isDevelopment`); V has no hit
- Pieces:
  1. In a development build, running "AI Tracking: Score Commit SHA for AI Content" (`developer.aiCodeTracking.scoreCommit`) opens an input box titled "Score Commit for AI Content" with the placeholder "Enter commit hash (e.g., abc123...)". [cursor only]
  2. When more than one repository is open the command asks which repository to use. [cursor only]
  3. The command then asks "Report metrics to analytics?" with the choices No and Yes. [cursor only]
  4. The command shows "Scoring commit ..." and then the notice "Commit scored! AI: {n}% ({a}/{b} lines). Check output channel for details." and writes the details to the "aiCodeTracking" Output channel. [cursor only]
  5. "AI Tracking: Score Recent Commits" (`developer.aiCodeTracking.scoreMultipleCommits`) scores several recent commits from the Command Palette. [cursor only]
  6. "[Debug] Score Commit for AI Content" (`developer.aiCodeTracking.scoreCommitInternal`) is not in the Command Palette and shows as a row in the right-click menu of a Source Control history item, in group `1_ai`. [cursor only]
  7. "[Debug] Score Commit with Details" (`developer.aiCodeTracking.scoreCommitWithDetails`) is a hidden command that other code calls. [cursor only]
  8. All four commands and the menu row exist only in a development build (`isDevelopment`). [cursor only]

#### SS-6.16 Developer: reveal AI Code Tracking paths

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Developer command "AI Tracking: Reveal AI Code Tracking Paths" (development build only, context key `isDevelopment`). It writes the location of the tracking database file and the names of its key groups to the "aiCodeTracking" Output channel, shows that channel, and posts a notice "AI Code Tracking paths have been written to the output channel." If the channel is missing it posts a warning instead.
- Commands: developer.aiCodeTracking.revealPaths
- Menus: CommandPalette (f1 true, precondition isDevelopment)
- Keys: none found
- Settings: none found
- Source: C bundle (class with `id:"developer.aiCodeTracking.revealPaths"`, category "AI Tracking", module `contrib/analytics/browser/aiCodeTrackingActions.contribution`); V has no hit
- Pieces:
  1. In a development build, running "AI Tracking: Reveal AI Code Tracking Paths" (`developer.aiCodeTracking.revealPaths`) writes the location of the tracking database file and the names of its key groups to the "aiCodeTracking" Output channel and shows that channel. [cursor only]
  2. The command then posts the notice "AI Code Tracking paths have been written to the output channel." [cursor only]
  3. If the Output channel is missing the command posts a warning instead. [cursor only]
  4. The command is listed in the Command Palette only in a development build (`isDevelopment`). [cursor only]
