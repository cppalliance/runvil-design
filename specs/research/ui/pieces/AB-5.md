Group: AB-5 Placement, size and visibility. Pass: 3. Date: 2026-10-25.

### AB-5.1
1. Setting `workbench.activityBar.location` chooses where the activity bar sits, with values `default` (a rail beside the Primary Side Bar and a strip on top of the Secondary Side Bar), `top`, `bottom` and `hidden`, and it can be changed in the Settings editor or in settings.json.
2. A saved `workbench.activityBar.visible` of false is migrated to `workbench.activityBar.location` set to `hidden`, and a saved location of `side` is migrated to `default`.
3. In Cursor the setting keeps the same four values and descriptions, but the side bar and rail that read it load only when `workbench.activityBar.orientation` is `vertical`, so in the default horizontal layout it has almost no visible effect. [cursor only]

### AB-5.2
1. Running "Move Activity Bar to Side", "Move Activity Bar to Top", "Move Activity Bar to Bottom" or "Hide Activity Bar" from the Command Palette sets `workbench.activityBar.location` to `default`, `top`, `bottom` or `hidden`. [stock only]
2. Each of those four palette commands is offered only when the bar is not already in that place. [stock only]
3. View > Appearance has an "Activity Bar Position" submenu with Default, Top, Bottom and Hidden, and the current choice shows a check. [stock only]
4. The same "Activity Bar Position" submenu also appears in the rail icon right-click menu, in the view container context menu and in the Secondary Side Bar title menu. [stock only]

### AB-5.3
1. With location `top`, the rail is not drawn and the same container icons show as a horizontal strip in the title of the Primary Side Bar.
2. With location `bottom`, the same strip runs along the bottom edge of the Primary Side Bar.
3. The strip's icons are 16 px with a 30 px overflow slot, and the strip is colored with the `activityBarTop.*` theme colors.
4. Right-clicking the strip adds a Views submenu to its context menu.
5. In Cursor the strip is drawn only by the older side bar that loads when orientation is `vertical`, the default horizontal layout never draws it, and Accounts and Manage do not move to the title bar. [cursor only]

### AB-5.4
1. Setting `workbench.activityBar.location` to `hidden` removes the rail or strip for both the Primary and Secondary Side Bars, and the Accounts and Manage icons go with it.
2. Running "Hide Activity Bar" from the Command Palette, or picking Hidden in the Activity Bar Position submenu, hides the bar the same way. [stock only]
3. When the bar is hidden by any route other than editing the setting, `workbench.activityBar.location` is updated to `hidden`.
4. In Cursor, hiding by the setting works only in the vertical layout, and the default horizontal layout has no rail to hide. [cursor only]

### AB-5.5
1. Running "Toggle Activity Bar Visibility" (`workbench.action.toggleActivityBarVisibility`) hides the activity bar.
2. Running it again brings the bar back where it was last visible, side, top or bottom, which is remembered in profile storage.
3. The Customize Layout picker has an Activity Bar eye toggle that runs the same command. [stock only]
4. The command can be run only from the Customize Layout picker or from a key you assign, because it has no Command Palette entry, menu item or default key.
5. In Cursor the command keeps its id and title, the Customize Layout picker has no Activity Bar item, and in the default horizontal layout the command clears `workbench.activityBar.location` and shows an info dialog reading "Please quit and reopen for the activity bar to be displayed vertically.". [cursor only]

### AB-5.6
1. Setting `workbench.activityBar.compact`, off by default, switches the rail to a narrower layout, 36 px wide with 32 px items and 16 px icons instead of 48 px, 48 px and 24 px. [stock only]
2. The "Activity Bar Size" submenu in the rail right-click menu offers Default and Compact, with a check on the current size. [stock only]
3. The "Activity Bar Size" submenu shows only when location is `default`. [stock only]
4. The rail is rebuilt at once when the size changes. [stock only]

### AB-5.7
1. With `workbench.activityBar.autoHide` on, off by default, and location set to `top` or `bottom`, the strip disappears when the side bar has only one visible view container. [stock only]
2. When a second visible view container appears, the strip comes back. [stock only]
3. The hiding applies to both the Primary and Secondary Side Bars. [stock only]

### AB-5.8
1. Running `workbench.action.toggleZenMode` to enter Zen Mode hides the activity bar while `zenMode.hideActivityBar` is on.
2. Setting `zenMode.hideActivityBar`, on by default, chooses whether Zen Mode hides the activity bar, and with it off the bar stays visible.
3. Changing the setting while in Zen Mode shows or hides the activity bar at once.
4. Leaving Zen Mode brings the activity bar back, unless it was already hidden before.
5. In Cursor the setting keeps its default of true, but its change handler is an older copy that does not keep the strip hidden for `top` or `bottom`, and with the default horizontal layout there is no rail, so it has no visible effect. [cursor only]

### AB-5.9
1. On a compact rail the number pill is 13 px tall instead of 16 px. [stock only]
2. On a compact rail the profile letters on the gear use an 8 px font instead of 9 px. [stock only]
3. On the top or bottom strip the number pill is drawn in a smaller style, 13 px tall with a 9 px font.
4. On the strip, a number of three or more characters, such as `100` or `1K+`, drops to an 8 px font.
5. On the strip, the progress glyph in a progress badge is drawn at 13 px.
6. In Cursor the strip badge styles are kept for the vertical layout, the compact rail styles are missing along with the setting, and the default horizontal row draws no badges. [cursor only]

## Unclear
- Cursor's `workbench.activityBar.location` and every strip piece above depend on `workbench.activityBar.orientation` being `vertical`, which is not Cursor's default. Cursor results are read from the bundle and were not run.
- Tag convention used here: a piece that Cursor lacks ends with `[stock only]`. Where the Cursor feature field says `changed`, one extra piece states what Cursor does instead and ends with `[cursor only]`.
- AB-5.2 piece 4 also lists the Secondary Side Bar title menu, which pass 2 handed to AX-4 for its own submenu. It is kept here only as an entry point of the same feature.
- AB-5.5 piece 4 comes from the feature block saying the command has no Command Palette entry, menu item or default key in stock. Cursor has no picker item, so there a key you assign is the only route found, and that was not checked further.
- AB-5.9 piece 3 reads the strip pill size (13 px tall, 9 px font) from `paneCompositePart.css`. Pixel sizes come from CSS, not from a screenshot.
- AB-5.3 piece 3 takes the 16 px icon size and 30 px overflow slot from the feature block without a second reading of the source.
