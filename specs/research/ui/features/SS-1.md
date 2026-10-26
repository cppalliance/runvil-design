Group: SS-1 Bar behavior and per-item control. Section: Status bar. Date: 2026-10-25. Pass: 2.
Features: 16.

### SS-1.1 Show or hide the status bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn the whole bar on or off from View > Appearance > Status Bar (checked while the bar shows), the Command Palette command "Toggle Status Bar Visibility", the "Hide Status Bar" row at the top of the bar's right-click menu, the Status Bar row in the Customize Layout picker (TB-4 owns the picker), or the `workbench.statusBar.visible` setting (default on). Cursor adds a Status Bar switch in the Layout card on the first tab of Cursor Settings (text "Show status bar at the bottom of the window"; ED-8 owns that page). While the bar is hidden, the notification center and toasts move down to the window edge.
- Commands: workbench.action.toggleStatusbarVisibility; workbench.action.customizeLayout
- Menus: MenubarAppearanceMenu (group 2_workbench_layout, order 3); the status bar right-click menu (built in code, not a MenuId)
- Keys: none found
- Settings: workbench.statusBar.visible
- Cursor: changed (command, menu row, Customize Layout row and setting are all present; Cursor's toggle also stores the new state under storage key `cursor/editorLayout.statusBarVisible` and writes the setting twice, first to the in-memory target and then through the normal write that picks the target itself; the Cursor Settings Layout switch is cursor only; the Sessions-window condition that V puts on the menu row and precondition is absent)
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:393-432 (action), :1370 (Customize Layout row), :1596 (Restore Defaults resets the setting); V\src\vs\workbench\browser\workbench.contribution.ts:624-629 (setting); V\src\vs\workbench\browser\layout.ts:1608-1620 (setStatusBarHidden), :2925-2926 and :2942-2943 (setting to layout state and back); V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:620-621 (right-click row); V\src\vs\workbench\browser\parts\notifications\media\notificationsCenter.css:18 and notificationsToasts.css:15 (`nostatusbar` offsets); C bundle (module layoutActions, nls 3169, 3109, 3138, 4221; the Layout card in the Cursor Settings React code, function `oww`, label "Status Bar")

### SS-1.2 Hide one item with Hide '{name}'
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click directly on an item and the last row of the menu reads Hide '{name}' with the item's name; clicking it removes that item from the bar at once. For an item that came from an extension the menu also gets a Manage Extension row above it, which opens the extension system and is out of scope here (see Handed off). The hidden list is stored per profile under storage key `workbench.statusbar.hidden`, and another window on the same profile picks up the change.
- Commands: none found
- Menus: the status bar right-click menu (built in code, not a MenuId)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:617-653 (menu build, item under the mouse); V\src\vs\workbench\browser\parts\statusbar\statusbarActions.ts:38-60 (HideStatusbarEntryAction, ManageExtensionAction); V\src\vs\workbench\browser\parts\statusbar\statusbarModel.ts:21-60, :135-151, :245-251 (hidden set, storage key, profile scope, save); C bundle (module statusbar, class `rSk` with nls 3986 "Hide '{0}'", class `WEa` with id `statusbar.manage.extension` and nls 3987, storage key `workbench.statusbar.hidden`)

### SS-1.3 Turn items on or off from the right-click checklist
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click anywhere on the bar to get "Hide Status Bar", a separator, and then one checkable row for every item the bar knows about, in bar order and labelled with the item's name; a tick means the item is shown. Clicking a row flips that item, which is the only way to bring a hidden item back. If several items share one id (for example from different extensions) only one row is listed for that id.
- Commands: none found
- Menus: the status bar right-click menu (built in code, not a MenuId)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:617-634; V\src\vs\workbench\browser\parts\statusbar\statusbarActions.ts:22-36 (ToggleStatusbarEntryVisibilityAction, checked = not hidden); V\src\vs\workbench\browser\parts\statusbar\statusbarModel.ts:132-151 (isHidden, hide, show); C bundle (module statusbar, `getContextMenuActions` with nls 3989 "Hide Status Bar", class `iSk`)

### SS-1.4 Items sit on the left or right and sort by priority
- Tag: both
- Evidence: source
- Boundary: no
- What: Every item is placed on the left or the right side of the bar and ordered by a number, where a higher number sits further left on either side; ties break by a hash of the item id so the order stays the same from run to run. An item can instead be placed just before or after another named item with its own fallback number, and it falls back to the number if that other item is not on the bar. The person cannot drag or reorder items; only the code that adds an item sets its side and priority, once, when it is created.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the sort code reads the same; Cursor's notifications bell is hard-wired right-most with priority negative infinity, and V's `workbench.notifications.position` setting that can move it is not in the bundle, see SS-5)
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarModel.ts:255-373 (sort); V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:254-273 (priority and secondary hash), :468-509 (append in order, right side reversed because the container is flex row-reverse); V\src\vs\workbench\services\statusbar\browser\statusbar.ts:38-100 (alignment, location, priority contract); C bundle (class `oSk`, sort and `status.notifications` registration with priority -Infinity)

### SS-1.5 Related items sit together as a compact pair
- Tag: both
- Evidence: source
- Boundary: no
- What: When an item is positioned next to another with the compact flag, the two sit closer with less padding between them, and hovering one lights both with the item hover color and the one under the pointer with the stronger compact hover color. Only items that have a command get that hover feedback, and it is skipped in high-contrast themes. In V the Source Control provider picker, the extra repository items and the Language Status and Language Detection items use it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (compact items exist, for example `status.scm.provider` with `compact:!0`; the spacing differs, the compact-left label has margin-right 3px in Cursor's CSS against 5px in V's)
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:511-593 (updateCompactEntries); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (compact-left and compact-right rules); V\src\vs\workbench\contrib\scm\browser\activity.ts:161 and :176; V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts:227; V\src\vs\workbench\contrib\languageDetection\browser\languageDetection.contribution.ts:104; C bundle (updateCompactEntries code, `status.scm.provider` entry with compact true); C\out\vs\workbench\workbench.desktop.main.css (compact-left rules)

### SS-1.6 Long items are cut and the bar stays one row
- Tag: both
- Evidence: source
- Boundary: no
- What: The bar is a fixed 22 px high strip and each item is capped at 40 percent of the window width, with long text cut off by an ellipsis. The right-side group is set to wrap with its first item on the right, and the bar hides anything that spills past its 22 px, so a crowded bar loses items instead of growing a second row. This is read from the CSS only and was not run on screen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the same rules are in the bundle CSS: height 22px, `max-width:40vw`, right items `flex-direction:row-reverse;flex-wrap:wrap`)
- Source: V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (the `.part.statusbar`, `.right-items`, `.statusbar-item` and `.statusbar-item-label` rules); V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:123 (HEIGHT 22); C\out\vs\workbench\workbench.desktop.main.css

### SS-1.7 Focus the status bar from the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: Run "Focus Status Bar" from the Command Palette (View category, no default key), or press F6 and Shift+F6 to cycle focus through the window's parts, which includes the bar. The bar takes focus and draws an outline in the `statusBar.focusBorder` color, and if an item had focus earlier, that item gets it back. While focus is inside the bar the context key `statusBarFocused` is true, which turns on the arrow keys in SS-1.8.
- Commands: workbench.action.focusStatusBar; workbench.action.focusNextPart; workbench.action.focusPreviousPart
- Menus: CommandPalette (f1 true for each)
- Keys: F6 (Focus Next Part); Shift+F6 (Focus Previous Part); Focus Status Bar has no default key
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarActions.ts:126-144; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:420-427 (focus, restore last entry), :430-440 (context key, tabIndex 0); V\src\vs\workbench\common\contextkeys.ts:128; V\src\vs\workbench\browser\layout.ts:1303-1304 (focusPart); V\src\vs\workbench\browser\actions\navigationActions.ts:260-350 (part cycling includes the bar); C bundle (class `sSk` with id `workbench.action.focusStatusBar` and nls 3988, context key `statusBarFocused` nls 4377, F6 commands with nls 3225 and 3226)

### SS-1.8 Move between items with the arrow keys, Home and End
- Tag: both
- Evidence: source
- Boundary: no
- What: With focus in the bar, Left or Up goes to the previous item, Right or Down to the next, Home to the first and End to the last, in the same left to right order you see on screen, skipping hidden items and wrapping round at either end. Escape first drops focus from the item to the bar itself, and the next Escape returns focus to the active editor.
- Commands: workbench.statusBar.focusPrevious; workbench.statusBar.focusNext; workbench.statusBar.focusFirst; workbench.statusBar.focusLast; workbench.statusBar.clearFocus
- Menus: none found
- Keys: Left or Up when statusBarFocused (focusPrevious); Right or Down when statusBarFocused (focusNext); Home when statusBarFocused (focusFirst); End when statusBarFocused (focusLast); Escape when statusBarFocused (clearFocus)
- Settings: none found
- Cursor: present (all five ids registered with the same keys and the same when context)
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarActions.ts:63-124; V\src\vs\workbench\browser\parts\statusbar\statusbarModel.ts:162-208 (focusEntry, hidden items skipped, wrap to the start position); C bundle (five `registerCommandAndKeybindingRule` calls, weight 200, `when` is `statusBarFocused`)

### SS-1.9 Click an item to run its command
- Tag: both
- Evidence: source
- Boundary: no
- What: Click an item, tap it on a touch screen, or press Enter or Space while it has keyboard focus to run its command; an item with no command looks disabled, with a plain pointer and no hover color, and does nothing. If the command throws, the person sees an error notification with the message. Cursor lets an item ask for the left mouse button press instead of the release, and no longer sends the click telemetry that stock does.
- Commands: none found (each item brings its own command)
- Menus: none found
- Keys: Enter or Space on a focused item
- Settings: none found
- Cursor: changed (adds `commandExecutionTrigger` with `click` as default or `mousedown`, which reacts to the left button only; drops the `workbenchActionExecuted` telemetry call; Space, Enter, Escape and arrow handling on the item is the same)
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:144-171 (listeners and disabled class), :226-243 (executeCommand, telemetry, error notification); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (`a.disabled`, hover rules); C bundle (item class with `getCommandExecutionTrigger`, mouse down branch, `status.sessionRecording` entry that uses `mousedown`)

### SS-1.10 Hover an item to read its tooltip
- Tag: both
- Evidence: source
- Boundary: no
- What: Hovering an item opens its tooltip with no delay when it is plain text or markdown, and after 500 ms when it is a custom element or slow-loading markdown. The tooltip closes when a key is pressed, and Escape, Left or Right closes it while the item has keyboard focus. In stock the tooltip is capped at 90 percent of the window height, and a tooltip can add footer links that run commands, which the item code supports but no in-scope item in 1.128 uses.
- Commands: none found
- Menus: none found
- Keys: Escape, Left or Right while an item with a command has focus (closes the hover)
- Settings: none found
- Cursor: changed (the hover setup keeps instant hover, the 500 ms rule and hide on key down but has no 90 percent height cap; Cursor adds `dontHideHoverOnClick`, which Cursor Tab's item sets so its panel stays open after a click)
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:182-209 (hover delegate); V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:120-142 (tooltip and footer actions), :155-166 (key handling); V\src\vs\workbench\services\statusbar\browser\statusbar.ts:117-130, :162-166 (tooltip types); C bundle (hover delegate with `instantHover:!0,dynamicDelay` and `persistence:{hideOnKeyDown:!0,sticky}`, `dontHideHoverOnClick` in the item and the managed hover)

### SS-1.11 Click an item to open its tooltip
- Tag: both
- Evidence: source
- Boundary: no
- What: An item whose command is the special id `statusBar.entry.showTooltip` opens its tooltip on click and moves keyboard focus into it, instead of running a command, so the tooltip stays up until the person moves away or presses a key. It only works if the item has a tooltip. The id has an empty title, so it has no label of its own. Window Zoom and the Language Status item use it in stock, and Cursor Tab's item uses it when a condition holds.
- Commands: statusBar.entry.showTooltip
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\statusbar\browser\statusbar.ts:104-107; V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:152, :228-229; V\src\vs\workbench\electron-browser\window.ts:1244; V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts:223; C bundle (`WXt={id:"statusBar.entry.showTooltip",title:""}`, the Cursor Tab entry sets `command:i?WXt:ayg.ID`)

### SS-1.12 Show a spinner in an item while it works
- Tag: both
- Evidence: source
- Boundary: no
- What: An item can ask for a progress icon in front of its text, either a spinning loader or a sync icon, and the icon keeps spinning without restarting when the text changes. In stock the window progress item, the remote indicator while it connects, and the terminal session recording item use it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (`showProgress` found in the window progress entry, the remote indicator entry and the `recordSession` entry)
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:92-93, :287-342 (StatusBarCodiconLabel); V\src\vs\workbench\services\statusbar\browser\statusbar.ts:192-198; V\src\vs\workbench\services\progress\browser\progressService.ts:184; V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts:563-571; C bundle (`showProgress:t.type||!0` in the window progress entry, `showProgress:i` in the remote indicator entry, `showProgress:!0` in the `recordSession` entry)

### SS-1.13 Beak pointing at the open notifications center
- Tag: both
- Evidence: source
- Boundary: no
- What: A small triangle points up from an item's top edge to show that a popup is open from that item. Only the notifications bell uses it, and only while the notification center is open; the triangle takes the bar's border color, or its background color if there is no border.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:173-181; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:703-706; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (`has-beak` rules); V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts:92; C bundle (`showBeak:this.isNotificationsCenterVisible` in the bell entry, the `has-beak` class in the item class)

### SS-1.14 Item names and screen reader labels
- Tag: both
- Evidence: source
- Boundary: no
- What: Every item has a short name, which names its row in the right-click checklist and its Hide '{name}' row, and a screen reader label that is set on both the item and its inner button so it is not read twice; the inner element has role button unless the item sets another role. The bar itself is a footer with role status and aria-live off, so text changes are not read out on their own, and the bar can take focus with a tab stop. Extension items that give no label get one built from their text, with icon names turned into words, plus the tooltip.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:68-72, :108-118; V\src\vs\workbench\browser\workbench.ts:354, :370-377; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:439 (tabIndex 0); V\src\vs\workbench\api\browser\statusBarExtensionPoint.ts:84-95 (label fallback); V\src\vs\workbench\services\statusbar\browser\statusbar.ts:134-158; C bundle (`setAttribute("aria-label",e.ariaLabel)` on both elements, `role:e.role||"button"`)

### SS-1.15 Items shown in every window and the bar in popped-out windows
- Tag: both
- Evidence: source
- Boundary: no
- What: A popped-out editor window has its own 22 px status bar that follows `workbench.statusBar.visible` and opens with the same right-click menu. It carries only the items marked to show in all windows, which are added to every open window and to new ones; in stock these are Screen Reader Mode, Scrolling Locked and Metered Connection, and Cursor has the first two. PN-1 owns the popped-out window as a whole.
- Commands: none found
- Menus: the status bar right-click menu (built in code, not a MenuId)
- Keys: none found
- Settings: workbench.statusBar.visible
- Cursor: changed (the auxiliary bar part `workbench.parts.auxiliaryStatus` and `showInAllWindows` are present, Screen Reader Mode uses it; the Metered Connection item is missing from the bundle)
- Source: V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:760-835, :841-900 (auxiliary part, all-windows entries); V\src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts:111-150 (visibility and height); V\src\vs\workbench\contrib\accessibility\browser\accessibilityStatus.ts:91; V\src\vs\workbench\contrib\scrollLocking\browser\scrollLocking.ts:176; V\src\vs\workbench\contrib\meteredConnection\browser\meteredConnectionStatus.ts:56; C bundle (`workbench.parts.auxiliaryStatus.${c}`, `showInAllWindows:!0` on the Screen Reader Mode entry, `eDt.STATUS_BAR_VISIBILITY`)

### SS-1.16 Bar pinned to the bottom edge when the title bar is hidden
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: In Cursor's layout without a title bar, the bar is fixed to the bottom of the window across the full width at 22 px with a high stacking order, and the right side bar keeps 22 px of padding at its bottom so the bar does not cover it. The body gets the class `no-titlebar-layout` and the context key `cursor.noTitlebarLayout.enabled` reports it. How the person turns that layout on is not traced here (TB-4 owns the layout presets).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C\out\vs\workbench\workbench.desktop.main.css (rule `body.no-titlebar-layout .monaco-workbench .split-view-view.monaco-statusbar-split-view` with `position:fixed;bottom:0`, and the `padding-bottom:22px` rules for `.part.sidebar.right`); C bundle (context key `cursor.noTitlebarLayout.enabled`, class name `no-titlebar-layout`, `markSplitViewContainer` in the status bar part)

## Handed off

- Manage Extension row in the item menu (`statusbar.manage.extension`, runs `_extensions.manage`): it opens the extension system, which is out of scope, so no group owns it. The menu it sits in is SS-1.2.
- Items an extension adds with `createStatusBarItem`, `setStatusBarMessage` or the `contributes.statusBarItems` point: extension system, out of scope. Their visible results (position, hide menu, tooltip, label) are covered by SS-1.2 to SS-1.14.
- Hiding the bar when Zen Mode starts (`zenMode.hideStatusBar`, default on in stock and in Cursor): ED-1.34 lists the setting and owns Zen Mode.
- The Customize Layout picker, its Restore Defaults button and the Cursor layout presets: TB-4. The Status Bar row inside the picker is covered in SS-1.1 because TB-4 handed it here.
- The Status Bar switch in Cursor Settings: ED-8 owns the page; the switch is listed as an entry point of SS-1.1.
- The Cursor Tab item, its hover panel and its controls: AC-8. Hiding or showing it from the right-click menu works through SS-1.2 and SS-1.3.
- The notifications bell, its place at the far right, and `workbench.notifications.position`: SS-5 for the item, section 10 for the center. Only the beak is in SS-1.13.
- The Window Zoom item (listed by TB-3 with no clean home): SS-5, where the first pass placed it. It uses SS-1.11.
- The item contents (text, icons, tooltips) of every other item: SS-3 to SS-6.

## Unclear

- The Cursor Settings Layout switch: found in the bundle (React function `oww`, inside the card titled Layout, drawn only when the window is not the Agents window). Not seen running, so the tab and card order are as ED-8 describes them.
- First-run behavior: the Cursor bundle has a small onboarding class that writes `workbench.statusBar.visible` to true when the key `workbench.contrib.onboarding.browser.gettingStarted.contribution.ts.firsttime` is `true`. I did not trace when that key is set or cleared.
- Per-layout memory: Cursor stores bar visibility under `cursor/editorLayout.statusBarVisible` and `cursor/agentLayout.statusBarVisible` and restores it when the layout switches between the editor layout and the agent layout. The agent layout is out of scope, and I did not trace the switching code.
- newer-than-1.128: N has a second special command `statusBar.entry.toggleTooltip` that shows a tooltip, pins it, and closes it on the next click, plus an `onDidShow` callback and `has-command` and `visible-background-color-neighbor` classes. None of this is in V or in the Cursor bundle (0 hits for the id). It is not listed as a feature because it extends SS-1.10 and SS-1.11 only.
- newer-than-1.128 and stock only: a bottom padding under the status bar while the experiment `workbench.experimental.modernUI` (floating panels) is on, via `FLOATING_BOTTOM_PADDING` in V's `statusbarPart.ts`. Cursor has no `isFloatingPanelsEnabled` and no such setting (0 hits), so I did not make a feature of it.
- Tooltip footer links (`tooltip.commands`): the code to draw them is in V and in the Cursor bundle, but I found no producer in V's non-test source. N's only producer is a chat promo item, which is out of scope.
- The 90 percent tooltip height cap is read from V's hover delegate and from one fragment of the Cursor bundle that has no such option. I did not check whether the hover service in Cursor applies its own cap.
- Nothing here was run on screen. Wrapping and clipping in SS-1.6, the keyboard wrap in SS-1.8 and the hover timing in SS-1.10 come from code and CSS.

