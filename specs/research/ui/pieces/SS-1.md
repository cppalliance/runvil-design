Group: SS-1 Bar behavior and per-item control. Pass: 3. Date: 2026-10-25.

### SS-1.1
1. Choosing View > Appearance > Status Bar (`workbench.action.toggleStatusbarVisibility`) shows or hides the whole bar, and the row has a check mark while the bar shows.
2. Running Toggle Status Bar Visibility from the Command Palette does the same toggle.
3. The Hide Status Bar row at the top of the bar's right-click menu hides the bar.
4. The Status Bar row in the Customize Layout picker (`workbench.action.customizeLayout`) shows or hides the bar.
5. Setting `workbench.statusBar.visible` (default on) shows or hides the bar, and stays in step with the other ways of toggling it.
6. While the bar is hidden, the notification center and toasts move down to the window edge.
7. Cursor Settings has a Status Bar switch in the Layout card on its first tab, with the text "Show status bar at the bottom of the window". [cursor only]

### SS-1.2
1. Right-clicking directly on an item shows a menu whose last row reads Hide '{name}' with that item's name.
2. Clicking Hide '{name}' removes that item from the bar at once.
3. The hidden items are remembered per profile, so they stay hidden after a restart, and another window on the same profile picks up the change.
4. Right-clicking empty space on the bar shows no Hide '{name}' row.

### SS-1.3
1. Right-clicking anywhere on the bar opens a menu with Hide Status Bar, a separator, and one checkable row for every item the bar knows about, in bar order and labelled with the item's name.
2. A check mark means the item is shown, and clicking a row hides a shown item or brings back a hidden one, which is the only way to bring a hidden item back.
3. When several items share one id, for example from different extensions, the menu lists only one row for that id.

### SS-1.4
1. Every item sits on either the left side or the right side of the bar.
2. Within a side, items are ordered by a priority number, and a higher number sits further left on either side.
3. Items with the same priority are ordered by a hash of the item id, so the order is the same from run to run.
4. An item can be placed just before or after another named item, and it falls back to its own priority number when that other item is not on the bar.
5. Dragging an item on the bar does not reorder it.

### SS-1.5
1. An item positioned next to another with the compact flag sits closer to it, with less padding between the two.
2. Hovering one half of a compact pair lights both halves with the item hover color and the half under the pointer with the stronger compact hover color.
3. Only items that have a command get that hover feedback, and the compact effect is skipped in high-contrast themes.
4. In stock, the Source Control provider picker, the extra repository items, the Language Status item, and the Language Detection item sit as compact pairs.
5. The compact-left label has a 3 px right margin instead of the stock 5 px. [cursor only]

### SS-1.6
1. The bar is a fixed strip 22 px high.
2. Each item is capped at 40 percent of the window width, and text that is too long is cut off with an ellipsis.
3. When the items do not fit, the ones that spill past the 22 px are clipped and the bar never grows a second row.

### SS-1.7
1. Running Focus Status Bar (`workbench.action.focusStatusBar`) from the Command Palette moves keyboard focus to the bar, and the command has no default key.
2. Pressing `F6` (Focus Next Part) and `Shift+F6` (Focus Previous Part) cycles focus through the window's parts, and the bar is one of them.
3. The focused bar draws a focus outline.
4. If an item had focus earlier, that item gets focus back when the bar is focused again.
5. While focus is inside the bar, the arrow keys, Home, End, and Escape of SS-1.8 work because the context key `statusBarFocused` is true.

### SS-1.8
1. Pressing Left or Up with focus in the bar moves to the previous item (`workbench.statusBar.focusPrevious`), and Right or Down moves to the next item (`workbench.statusBar.focusNext`), following the left to right order on screen.
2. Pressing Home goes to the first item (`workbench.statusBar.focusFirst`), and pressing End goes to the last item (`workbench.statusBar.focusLast`).
3. Moving skips hidden items and wraps around at either end.
4. Pressing Escape (`workbench.statusBar.clearFocus`) drops focus from the item to the bar itself, and the next Escape returns focus to the active editor.

### SS-1.9
1. Clicking an item, or tapping it on a touch screen, runs its command.
2. Pressing Enter or Space while an item has keyboard focus runs its command.
3. An item with no command looks disabled, with a plain pointer and no hover color, and does nothing when clicked.
4. When an item's command throws, an error notification with the message appears.
5. An item can ask to run its command when the left mouse button goes down instead of when it is released, and other buttons do not run it. [cursor only]

### SS-1.10
1. Hovering an item whose tooltip is plain text or markdown opens the tooltip with no delay.
2. A tooltip that is a custom element or slow-loading markdown opens after 500 ms.
3. The tooltip closes when a key is pressed.
4. Pressing Escape, Left, or Right while an item with a command has keyboard focus closes its tooltip.
5. The tooltip is capped at 90 percent of the window height. [stock only]
6. An item can keep its tooltip open after a click, and the Cursor Tab item uses this so its panel stays open. [cursor only]

### SS-1.11
1. Clicking an item whose command is `statusBar.entry.showTooltip` opens its tooltip instead of running a command.
2. Keyboard focus moves into the opened tooltip, so it stays up until the person moves away or presses a key.
3. The click opens nothing when the item has no tooltip.
4. In stock, the Window Zoom item and the Language Status item open their tooltips this way.
5. The Cursor Tab item opens its tooltip this way when a condition holds. [cursor only]

### SS-1.12
1. An item can show a progress icon in front of its text, either a spinning loader or a sync icon.
2. The icon keeps spinning without restarting when the item's text changes.
3. The window progress item, the remote indicator while it connects, and the terminal session recording item show the progress icon.

### SS-1.13
1. A small triangle points up from the top edge of an item to show that a popup is open from that item.
2. Only the notifications bell shows it, and only while the notification center is open.
3. The triangle takes the bar's border color, or the bar's background color when there is no border.

### SS-1.14
1. Each item has a short name, and that name labels its row in the right-click checklist and its Hide '{name}' row.
2. Each item has a screen reader label that is set on both the item and its inner button, so a screen reader does not read it twice.
3. The inner element has the role button unless the item sets another role.
4. The bar is a footer with the role `status` and aria-live off, so a screen reader does not read out text changes on its own.
5. The bar can take focus as a tab stop.
6. An extension item that gives no label gets one built from its text, with icon names turned into words, plus its tooltip.

### SS-1.15
1. A popped-out editor window has its own status bar, 22 px high.
2. The popped-out bar follows the `workbench.statusBar.visible` setting.
3. Right-clicking the popped-out bar opens the same menu as the main bar.
4. The popped-out bar carries only the items marked to show in all windows, and those items are added to every open window and to new ones.
5. The Screen Reader Mode item and the Scrolling Locked item show in every window's bar.
6. The Metered Connection item shows in every window's bar. [stock only]

### SS-1.16
1. In Cursor's layout without a title bar, the bar is fixed to the bottom of the window across the full width, 22 px high, with a high stacking order. [cursor only]
2. In that layout the right side bar keeps 22 px of padding at its bottom so the bar does not cover it. [cursor only]

## Unclear

- SS-1.2 leaves out the Manage Extension row that appears above Hide '{name}' for an extension item, because it opens the extension system, which is out of scope.
- SS-1.10 leaves out tooltip footer links that run commands. The code supports them, but no in-scope item in 1.128 uses them, so a person cannot see them.
- SS-1.11 piece 5 does not name the condition under which the Cursor Tab item uses the click-to-open tooltip. The feature block does not give it.
- SS-1.16 does not say how a person turns on the layout without a title bar. The feature hands that to the layout presets.
