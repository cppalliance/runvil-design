Section: Notifications (NT). Date: 2026-10-25. Pass: 1.

# NT Notifications - stock VS Code 1.128.0 findings

- Source root V: `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0` (all paths below are relative to V unless they start with N or C)
- Main folder read: `src\vs\workbench\browser\parts\notifications` (center, toasts, list, viewer, actions, commands, status, alerts, accessible view, 4 css files in `media`)
- Also read: `src\vs\platform\notification\common\notification.ts`, `src\vs\workbench\common\notifications.ts`, `src\vs\workbench\services\notification\common\notificationService.ts`, `src\vs\workbench\services\progress\browser\progressService.ts`, `progressIndicator.ts`, `src\vs\platform\progress\common\progress.ts`
- Wiring: `src\vs\workbench\browser\workbench.ts` lines 385-404 create the center, toasts, alerts and status objects, call `registerNotificationCommands`, and register the accessible view
- The prompt named a `notificationsTelemetry` file. It does not exist in V (it only exists in N, see the newer-than-1.128 list)
- The prompt named a "Show Notifications Log" view. I found no such command, label or view in V, in N, or in the Cursor bundle. The closest things are the notification center list (open notifications only) and `Show Logs...` (Output channels, owned by the Panel section). Treated as not existing.

## How notifications work in one paragraph

Any code calls `INotificationService` (`notify`, `info`, `warn`, `error`, `prompt`, `status`). The service puts the item into a model. Three views watch the model: toasts (transient cards), the center (full list), and the status bar bell. Do Not Disturb is a filter on the model that silences non-error items. Progress notifications are normal notifications with a progress bar. Proof: `src\vs\workbench\services\notification\common\notificationService.ts`, `src\vs\workbench\common\notifications.ts` (class `NotificationsModel`, `NotificationViewItem`).

## Proposed minor groups

### NT-1 Toast popups

- name: Toast popups
- one sentence: Short-lived cards that appear in a window corner when something happens, stack up to three high, and hide themselves after a few seconds unless you hover, focus, or the notification is sticky.
- tag: both
- evidence: source
- boundary: no
- proof paths: `src\vs\workbench\browser\parts\notifications\notificationsToasts.ts`, `src\vs\workbench\browser\parts\notifications\media\notificationsToasts.css`, `src\vs\workbench\browser\parts\notifications\notificationsCommands.ts`, `src\vs\workbench\common\notifications.ts` (sticky rules, line 586)
- example features:
  - Toast stack: newest on top, max 3 visible (`MAX_NOTIFICATIONS`), 450 px max width, spam guard drops anything past 3 per 800 ms (`SPAM_PROTECTION`), fade-in animation, skipped while the center is open
  - Auto-hide timers: Info 10 s, Warning 12 s, Error 15 s (`PURGE_TIMEOUT`); timer waits while the window is unfocused, the toast has focus, or the mouse is over it; sticky items never hide (errors with buttons, expanded items, running progress)
  - Keyboard control: `Escape` hides toasts (`notifications.hideToasts`), `Focus Notification Toast` command, Up/Down/PageUp/PageDown/Home/End move between toasts (when focused)
  - Priority gating: `SILENT` priority never makes a toast, `OPTIONAL` priority is skipped if it would cover a focused input in the toast corner, `URGENT` ignores Do Not Disturb (`NotificationPriority`, `isElementInNotificationQuarter`)

### NT-2 What a notification contains and its buttons

- name: Notification contents and buttons
- one sentence: The pieces inside one notification, same in a toast and in the center: severity icon, message with links, source line, buttons, extra actions, and close.
- tag: both
- evidence: source
- boundary: no
- proof paths: `src\vs\workbench\browser\parts\notifications\notificationsViewer.ts`, `src\vs\workbench\browser\parts\notifications\notificationsActions.ts`, `src\vs\workbench\browser\parts\notifications\notificationsList.ts`, `src\vs\workbench\services\notification\common\notificationService.ts`, `src\vs\platform\notification\common\notification.ts`
- example features:
  - Severity icon and message: info, warning, error codicons, `[text](link)` links (commands run through the opener), message cut at 1000 characters, single line when collapsed with a hover for the full text
  - Buttons: primary actions render as buttons under the message (first is primary style, the rest secondary, one can have a dropdown); clicking closes the notification unless `keepOpen`; `Don't Show Again` is added by the service when `neverShowAgain` is set (workspace, profile, or application scope)
  - Row toolbar: expand or collapse (reveals source and buttons), `More Actions...` gear menu (secondary actions plus `Turn Off Info and Warning Notifications from '{source}'`), close (hidden while progress runs)
  - Mouse and clipboard shortcuts: right-click `Copy Text`, middle-click closes, double-click toggles expand

### NT-3 Bell and notification center

- name: Bell and notification center
- one sentence: The status bar bell shows an unread dot and opens the notification center, a list that holds every notification still open, with a header toolbar to clear all, set Do Not Disturb, and hide.
- tag: both
- evidence: source
- boundary: no
- proof paths: `src\vs\workbench\browser\parts\notifications\notificationsStatus.ts`, `src\vs\workbench\browser\parts\notifications\notificationsCenter.ts`, `src\vs\workbench\browser\parts\notifications\media\notificationsCenter.css`, `src\vs\workbench\browser\parts\notifications\notificationsCommands.ts`
- example features:
  - Status bar bell (`status.notifications`): `bell` or `bell-dot` (unread = added while the center was closed, or progress running), tooltips such as `1 New Notification` and `No New Notifications (1 in progress)`, click shows or hides the center; entry sits far right
  - Center panel: title `Notifications` or `No new notifications`, max 450 x 400, auto-hides when the last item goes, `Escape` hides it, focus returns to the editor group
  - Header toolbar: `Clear All Notifications` (skips items with progress), Do Not Disturb dropdown, Hide; V also adds a position dropdown (see V-only list)
  - Commands: `Show Notifications` (`notifications.showList`, `Ctrl+K Ctrl+Shift+N`), `Hide Notifications`, `Clear All Notifications`, per-item `Delete` clears, Left/Right collapse and expand, Space or Enter toggles
- also here, not given its own group: the status bar `Status Message` entry that `INotificationService.status()` writes (`notificationsStatus.ts` lines 205-270), plus ARIA alerts (`notificationsAlerts.ts`) and the accessible view (`notificationAccessibleView.ts`)

### NT-4 Do Not Disturb

- name: Do Not Disturb
- one sentence: A mode that makes every non-error notification silent (no toast, still listed in the center), either for everything or per source, shown as a bell-slash icon in the status bar.
- tag: both
- evidence: source
- boundary: no
- proof paths: `src\vs\workbench\browser\parts\notifications\notificationsCommands.ts` (lines 274-317), `src\vs\workbench\browser\parts\notifications\notificationsCenter.ts` (lines 237-272), `src\vs\workbench\browser\parts\notifications\notificationsStatus.ts` (lines 94-101), `src\vs\workbench\services\notification\common\notificationService.ts`, `src\vs\workbench\browser\workbench.zenMode.contribution.ts` (line 71)
- example features:
  - Toggle Do Not Disturb Mode: command `notifications.toggleDoNotDisturbMode`, header dropdown `Enable Do Not Disturb Mode` or `Disable Do Not Disturb Mode`, status bar shows `bell-slash` and tooltip `Do Not Disturb Mode is Enabled`, center closes when it turns on
  - Do Not Disturb by source: command `notifications.toggleDoNotDisturbModeBySource`, a multi-select quick pick titled `Select sources to enable all notifications from`; the header dropdown lists up to 10 sources then `More…`
  - Per-notification mute: gear menu entry `Turn Off Info and Warning Notifications from '{source}'` or `Turn On All Notifications from '{source}'`
  - Zen Mode: setting `zenMode.silentNotifications` (default true) turns Do Not Disturb on while in Zen Mode
- rules: errors and `URGENT` priority always show; filter state is saved in application storage under `notifications.doNotDisturbMode` and `notifications.perSourceDoNotDisturbMode`

### NT-5 Progress notifications

- name: Progress notifications
- one sentence: Long-running work reported through the progress service shows as a notification with a progress bar and Cancel, drops to a spinner entry in the status bar when hidden, and also feeds progress bars in views, the activity bar, and a waiting dialog.
- tag: both
- evidence: source
- boundary: no (but pieces render in other sections, see cross references)
- proof paths: `src\vs\workbench\services\progress\browser\progressService.ts`, `src\vs\workbench\services\progress\browser\progressIndicator.ts`, `src\vs\platform\progress\common\progress.ts`, `src\vs\workbench\browser\parts\notifications\notificationsViewer.ts` (`renderProgress`, line 547), `src\vs\workbench\browser\parts\notifications\notificationsStatus.ts` (in-progress bell dot)
- example features:
  - Notification progress (`ProgressLocation.Notification`): infinite bar or percent bar, optional `Cancel` (or custom labels via `cancellable` and `buttons`), optional delay, shown at least 800 ms, sticky, no close button while running, never cleared by Clear All
  - Window progress (`ProgressLocation.Window`): a `Progress Message` entry in the status bar left side with a spinner (types `loading` and `syncing`), appears after 150 ms, click runs `notifications.showList`; a hidden progress notification also falls back to this entry
  - View and side bar progress (`Explorer`, `Scm`, `Extensions`, or a view or container id): progress bar inside the view or pane header plus a progress badge on the activity bar icon after 300 ms; editor groups have their own bar (`EditorProgressIndicator`)
  - Dialog progress (`ProgressLocation.Dialog`): a `pending` dialog with `Cancel` or `Dismiss`, optional detail and sticky mode
- cross references: window progress entry and bell belong to the status bar section (SS), the activity bar badge to AB, the pending dialog to DL, view progress bars to SB and PN. This group only claims the progress service as the shared source. Boundary question for the parent: should the three non-notification progress shapes move to their host sections in pass 2? Recommendation: keep the notification and window shapes here, move the other two. Confidence: medium, because the code treats all five as one service.

### NT-6 Update and what's-new cards

- name: Update and what's-new cards
- one sentence: In V this area is covered by a title bar update entry and a post-update "New in {version}" widget; in Cursor it is the floating update and changelog cards (see the Cursor file).
- tag: cursor (V has analogs only, listed below)
- evidence: source for the V analogs
- boundary: no
- V analog proof paths: `src\vs\workbench\contrib\update\browser\postUpdateWidget.ts` (sticky hover card at top right, banner, features list, buttons, setting `update.showPostInstallInfo`), `src\vs\workbench\contrib\update\browser\updateTitleBarEntry.ts` (action id `workbench.actions.updateIndicator`, setting `update.titleBar`), `src\vs\workbench\contrib\update\browser\update.ts` line 196 (prompt `Welcome to {0} v{1}! Would you like to read the Release Notes?` when the release notes editor fails)
- example features (V side only): Post-update widget; Release notes fallback prompt; title bar update entry (owned by the Title bar section, TB-5)
- note: V 1.128.0 no longer has the old `There is an available update.` notification prompts. Cursor still has them (see the Cursor file).

## Commands, keys, menus

Each entry is id, label, keybinding, proof. All proof lines are in `src\vs\workbench\browser\parts\notifications\notificationsCommands.ts` unless stated.

- `notifications.showList` - Show Notifications - `Ctrl+K Ctrl+Shift+N` - id line 27, rule line 98
- `notifications.hideList` - Hide Notifications - `Escape` when the center is visible - id line 28, rule line 109
- `notifications.toggleList` - no label, internal, used by the V title bar bell - line 29
- `notifications.clearAll` - Clear All Notifications - no key - id line 45
- `notifications.hideToasts` - no palette label - `Escape` when toasts are visible (lower weight) and when a toast has focus (higher weight) - id line 32, rules lines 210 and 217
- `notifications.focusToasts` - Focus Notification Toast - no key - id line 33, command line 224
- `notifications.focusNextToast`, `focusPreviousToast`, `focusFirstToast`, `focusLastToast` - no label - Down, Up, PageUp or Home, PageDown or End (a toast has focus) - lines 34-37
- `notification.clear` - Clear Notification (row icon) - `Delete`, `Cmd+Backspace` on macOS - id line 44, rule line 128
- `notification.expand`, `notification.collapse`, `notification.toggle` - Expand Notification, Collapse Notification - Right, Left, Space or Enter - lines 40, 41, 43
- `notification.acceptPrimaryAction` - Accept Notification Primary Action - `Ctrl+Shift+A`, only in screen reader mode - id line 42, rule line 159
- `notifications.toggleDoNotDisturbMode` - Toggle Do Not Disturb Mode - no key - id line 46, command line 276
- `notifications.toggleDoNotDisturbModeBySource` - Toggle Do Not Disturb Mode By Source... - no key - id line 47, command line 283
- `workbench.action.configureDoNotDisturbMode`, `workbench.action.configureNotificationsPosition`, `workbench.action.configureNotification`, `workbench.action.copyNotificationMessage` - Configure Do Not Disturb..., Configure Notifications Position..., More Actions..., Copy Text - center header and row actions, not in the palette - `notificationsActions.ts`
- `workbench.action.setNotificationsPosition.bottomRight`, `.bottomLeft`, `.topRight` - Bottom Right, Bottom Left, Top Right - menu `NotificationsCenterPositionMenu` - lines 348-397, menu id at `src\vs\platform\actions\common\actions.ts` line 135

- Command palette category for the palette entries is `Notifications` (line 321). In the palette (lines 322-328): Show, Hide (center visible), Clear All, Accept Primary Action, Toggle Do Not Disturb Mode, Toggle Do Not Disturb Mode By Source..., Focus Notification Toast (toasts visible).
- Title bar bell: `MenuId.TitleBar` item `Toggle Notifications` with the bell icon, only when position is `top-right` and `workbench.notifications.showInTitleBar` is true (lines 331-344). Title bar right-click toggle `Notifications` in `src\vs\workbench\browser\parts\titlebar\titlebarActions.ts` line 80.
- Context keys: `notificationFocus`, `notificationCenterVisible`, `notificationToastsVisible` (`src\vs\workbench\common\contextkeys.ts` lines 150-152).
- Terminal keybinding skip list includes `notifications.hideList`, `notifications.hideToasts`, `notification.acceptPrimaryAction` (`src\vs\workbench\contrib\terminal\common\terminal.ts` lines 561, 562, 649).

## Settings

- `workbench.notifications.position` - enum `bottom-right` (default), `bottom-left`, `top-right`; tagged experimental with `experiment.mode: auto` (`src\vs\workbench\browser\workbench.contribution.ts` line 630)
- `workbench.notifications.showInTitleBar` - boolean, default true, only used when position is top right (same file line 645)
- `zenMode.silentNotifications` - boolean, default true (`src\vs\workbench\browser\workbench.zenMode.contribution.ts` line 71)
- `accessibility.verbosity.notification` - adds an accessible-view hint to each notification (`src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts` line 173)
- `accessibility.signals.clear` - sound or announcement when notifications are cleared (same file line 685)
- Stored state, not settings: `notifications.doNotDisturbMode`, `notifications.perSourceDoNotDisturbMode` (`notificationService.ts` lines 67-68)

## Theme colors and icons

- Colors: `notifications.foreground`, `notifications.background`, `notifications.border`, `notificationLink.foreground`, `notificationCenter.border`, `notificationToast.border`, `notificationCenterHeader.foreground`, `notificationCenterHeader.background`, `notificationsErrorIcon.foreground`, `notificationsWarningIcon.foreground`, `notificationsInfoIcon.foreground` (`src\vs\workbench\common\theme.ts` lines 766-801)
- Icons: `notifications-clear`, `notifications-clear-all`, `notifications-hide`, `notifications-hide-up`, `notifications-expand`, `notifications-expand-down`, `notifications-collapse`, `notifications-collapse-up`, `notifications-configure`, `notifications-do-not-disturb`, `notifications-position` (`notificationsActions.ts` lines 15-26)

## V-only in this slice (candidates for "V items missing from Cursor")

- Notification position choice (bottom right, bottom left, top right), the position dropdown in the center header, the title bar bell, the left-aligned bell for bottom-left, and the title bar toggle. Proof: `notificationsCommands.ts`, `notificationsStatus.ts` lines 104-150, `notificationsCenter.ts` lines 90-128, `workbench.contribution.ts` line 630.
- Post-update widget and title bar update entry (`src\vs\workbench\contrib\update\browser\postUpdateWidget.ts`, `updateTitleBarEntry.ts`).

## Left out (not cataloged here)

- Extension recommendation prompts and everything under `src\vs\workbench\contrib\extensions` (out of scope by brief)
- Chat input notification service (`src\vs\workbench\contrib\chat\browser\widget\input\chatInputNotificationService.ts`) and other chat or agent prompts (out of scope by brief)
- Terminal OSC 99 notifications (`terminal.integrated.enableNotifications`, `src\vs\workbench\contrib\terminalContrib\notification`): they use the notification service but belong to the Panel section's terminal
- Task completion OS notification (`task.notifyWindowOnTaskCompletion`, `src\vs\workbench\contrib\tasks\browser\task.contribution.ts` line 568): tasks area, uses the OS and a toast
- Modal dialogs and message boxes (Dialogs section, DL)
- Status bar item rendering itself (Status bar section, SS), and the title bar bell hosting (Title bar section, TB)

## Newer than 1.128 (seen only in N, `C:\Users\Vinnie\src\vscode`, version 1.144.0)

- Notification telemetry files `src\vs\platform\notification\common\notificationTelemetry.ts` and `src\vs\workbench\common\notificationTelemetry.ts`; `logNotificationInteraction` hooks in center, toasts, viewer, commands (no visible UI)
- Compact notification rows: `setNotificationRowHeight`, `COMPACT_NOTIFICATION_ROW_HEIGHT` 34 px, default 42 px, used by the Sessions window tests (`N\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts`)
- `pendingNotificationToasts.ts`: dedupes toasts that are still waiting for the next animation frame
- `NotificationText` (`N\src\vs\platform\notification\common\notificationMessage.ts`): structured message with explicit links instead of parsed markdown, `prompt()` now accepts it; `legacyExtensionLinkParsing` flag
- `hasActiveProgress` replaces `hasProgress` (a finished progress no longer blocks close or Clear All)
- Accessible view: live content updates and link actions for notifications (`N\...\notificationAccessibleView.ts` line 157 onward)
- Button `enabled` state follows the action in notification buttons (`notificationsViewer.ts`)

## Not verified

- I did not run VS Code, so toast timing, layout and the experimental position feature are read from source only.
- `workbench.notifications.position` carries `tags: ['experimental']` and `experiment.mode: auto` (workbench.contribution.ts lines 640-643), so whether stock 1.128.0 users saw it by default depends on the experiment service. Not checked.
- The `agentsWindow` keys in `workbench.contribution.ts` mean some settings are tuned for a separate window. Not relevant to NT but noted.
- Exact pixel sizes (450 max width, 42 px row, 22 px line) come from the code constants, not from a rendered check.
