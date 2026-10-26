Section: Notifications (NT). Date: 2026-10-25. Pass: 1.

# NT Notifications - Cursor 3.27.12 findings

- Source root C: `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`
- Main bundle: `C\out\vs\workbench\workbench.desktop.main.js`. Styles: `C\out\vs\workbench\workbench.desktop.main.css`. Label table: `C\out\nls.messages.json` (message by index) and `C\out\nls.keys.json` (module and key by the same index). Cursor labels written as `k(3886,null)` in the bundle are looked up there.
- Evidence level for everything in this file is `bundle` (strings and minified code only, no source). I did not run Cursor.
- I did not read `workbench.glass.main.js` (Agents window, out of scope) or the automations bundle.
- Search method: tight `rg` patterns and a small scratch scanner in `c:\Users\Vinnie\cursor\runvil-design\specs\temp` that prints fragments only.
- Overall result: the notification engine is the stock 1.128.0 engine from before the notification position feature (see V items missing). Cursor restyles it, adds a Success severity, lengthens timers, moves toasts to the side bar side, and adds its own floating update cards.

## Decision on the floating "Update to v..." cards

- What they are: three Cursor-only floating cards drawn by a Solid component, not by the notification service. Class names `update-notification-container` ("Update to v{version}"), `minor-version-notification-container` ("New update available"), and `changelog-notification` ("New in {version}"). Bundle fragments: `class=update-notification-eyebrow>Update to v`, `minor-version-notification-text>New update available`, `changelog-notification><div><span>New in `. Styles are in the css file (`.update-notification-outer-container{position:fixed;z-index:2000}`).
- Are they notifications (NT)? Closest fit is yes: transient floating card, dismissible, anchored to a window corner, snoozable. But they do not go through `INotificationService`: they never enter the notification model, so they are not in the center, not counted by the bell, and not affected by Do Not Disturb.
- Are they the title-bar Update control (TB-5)? No. The title bar control is a different component: button class `ide-titlebar-update-button` with labels `Update`, `Installing`, action `Restart to Update` (nls 11480-11484, 4034, module `contrib/ui/browser/buttons/IdeTitlebarUpdateButton`), gated by experiment `ide_update_ux_exp`. Both the cards and the title bar button end at the same update service.
- Decision: catalog the three cards here as NT-6, cross-reference TB-5, and tell the TB owner that the cards are not theirs. The same update flow also has an agents sidebar button (`AgentSidebarUpdateButton`, nls 4796-4800) which is agent window UI and out of scope. Confidence: medium - decided from code and css, not from seeing the cards on screen.
- Do not confuse with the label `Update to v{0}` at nls 6503: that is the Extensions view update button (module `contrib/extensions/browser/extensionsActions`), out of scope.

## Proposed minor groups (Cursor view)

### NT-1 Toast popups

- tag: both
- evidence: bundle
- boundary: no
- Cursor differences:
  - Timers are longer: Info 15 s, Warning 18 s, Error 20 s, Success 15 s (V is 10, 12, 15). Fragment: `PURGE_TIMEOUT={[vi.Info]:15e3,[vi.Warning]:18e3,[vi.Error]:2e4,[vi.Success]:15e3}`. Same stack limits as V: `MAX_WIDTH=450`, `MAX_NOTIFICATIONS=3`, spam guard 800 ms.
  - Placement follows the side bar: method `configureToastContainerPosition` sets `left:3px` when the side bar is not on the right, `right:3px` when it is (`getSideBarPosition()===1`). In V the toast always sits bottom right by default. css fallback `.notifications-toasts{bottom:25px;right:3px}`.
  - New command `cursorai.action.hideNotificationsForMs` (no label, not in the palette): hides all toasts now and blocks new ones for N ms (`hideForMs`, `shouldHideUntil`). Callers pass 10 minutes and 30 seconds around onboarding.
  - Toast chrome overrides at the end of the css: `.notification-toast{border:none!important;margin:6px!important}`, `.notifications-list-container{width:340px!important;box-shadow:var(cursor-box-shadow-base)!important}` (the real variable name has a two-dash prefix that I dropped here), rows get a 6 px radius and a 50 percent mixed border.
- same as V: keyboard commands `notifications.hideToasts`, `notifications.focusToasts` (palette label `Focus Notification Toast`, nls 3892), next, previous, first, last toast ids all present.
- proof: bundle fragments above; css rules listed above; nls 3892.

### NT-2 What a notification contains and its buttons

- tag: both
- evidence: bundle
- boundary: no
- Cursor differences:
  - Smaller rows: `ROW_HEIGHT=28`, `LINE_HEIGHT=16`, new `ACTIONS_HEIGHT=36` (V: 42, 22, 42). Message width basis 340 (V 450). Fonts 12 px, icon 12 px, toolbar 16 px (css `.notification-list-item-message{font-size:12px!important;line-height:16px!important}`).
  - New severities: Cursor's `Severity` enum is `Ignore, Info, Warning, Error, AI=4, Success=5` (V stops at Error). Icons: Info uses `info-two`, Warning uses `warning-two`, Error uses `error`, Success uses `pass` (fragment `toSeverityIcon(e){switch(e){case vi.Warning:return Oe.warningTwo;case vi.Error:return Oe.error;case vi.Success:return Oe.pass}return Oe.infoTwo}`). No icon for `AI` found. Success has no color rule I could find in css, only Info, Warning, Error icon colors.
  - Extra optional fields on a notification: `toastDurationMs`, `closeNotificationOnToastDismiss`, `toastTitle`, `toastDescription`, `toastCode` (14 and 15 hits). Only `toastDurationMs` and `closeNotificationOnToastDismiss` are passed by callers in the main bundle; I found no code in the main bundle that reads them to change the toast (the purge timer still uses severity). See Not verified.
- same as V: primary buttons, secondary gear menu `More Actions...` (nls 3872), `Copy Text` (nls 3873, `workbench.action.copyNotificationMessage`), expand and collapse, `Turn On All Notifications from '{0}'` and `Turn Off Info and Warning Notifications from '{0}'` (nls 3917, 3918), `Don't Show Again` (nls 13017), `Source: {0}`.
- proof: nls 3855-3873, 3915-3919, 13017; fragments above.

### NT-3 Bell and notification center

- tag: both
- evidence: bundle
- boundary: no
- Cursor differences:
  - The bell is always right-aligned and rightmost: `addEntry(t,"status.notifications",1,Number.NEGATIVE_INFINITY)` (V can left-align it or hide it). Icons `bell`, `bell-dot`, `bell-slash`, `bell-slash-dot`, and the same tooltips (nls 3900-3911).
  - The center stays bottom right by css (`.notifications-center{bottom:33px;right:11px;z-index:1000}`), even when toasts are on the left. Header toolbar is Clear All, Do Not Disturb dropdown, Hide (nls 3879 `Notification Center Actions`). No position dropdown.
  - Header css overrides: 12 px action icons, `.notifications-center-header .monaco-button.monaco-text-button` gets 3 px radius.
- same as V: commands `notifications.showList`, `hideList`, `toggleList`, `clearAll`, palette entries `Show Notifications`, `Hide Notifications`, `Clear All Notifications`, `Accept Notification Primary Action` (nls 3886-3889), chord `Ctrl+K Ctrl+Shift+N`, context keys `notificationFocus`, `notificationCenterVisible`, `notificationToastsVisible` (nls 4381-4383), center title `Notifications` or `No new notifications` (nls 3877, 3878), `Status Message` status entry (nls 3912).
- proof: nls 3877-3911; bundle `status.notifications` fragment; css `.notifications-center` rules.

### NT-4 Do Not Disturb

- tag: both
- evidence: bundle
- boundary: no
- Cursor differences: none found. Commands `notifications.toggleDoNotDisturbMode` and `notifications.toggleDoNotDisturbModeBySource` are in the palette (nls 3890, 3891), `Configure Do Not Disturb...` (nls 3868) and `Enable Do Not Disturb Mode` or `Disable Do Not Disturb Mode` (nls 3880, 3881), `Select sources to enable all notifications from` (nls 3884), `Do Not Disturb Mode is Enabled` (nls 3903). Storage keys `notifications.doNotDisturbMode` and `notifications.perSourceDoNotDisturbMode` are identical.
- `zenMode.silentNotifications` exists with default true (nls 4319; bundle `"zenMode.silentNotifications":{type:"boolean",default:!0`).
- proof: nls 3866-3868, 3880-3884, 3890-3891, 3902-3903, 4319.

### NT-5 Progress notifications

- tag: both
- evidence: bundle
- boundary: no (shapes outside notifications belong to SS, AB, DL, SB, PN as in the VS Code file)
- Cursor differences: none found. Same service strings: `Progress Message` (nls 13071), `Cancel` (13072), `Dismiss` (13074), `{0}: {1}` (13067), and the status entry id `status.progress` is left aligned (`addEntry(l,"status.progress",0,-Number.MAX_VALUE)`). Window progress with a command opens the center: `command:"notifications.showList"`.
- proof: nls 13067-13074, bundle fragments above.

### NT-6 Cursor update and what's-new cards

- name: Update and what's-new cards
- one sentence: Three floating cards pinned to the window corner on the side bar side that announce a new Cursor version, offer Install Now, Later and Changelog, and snooze for a day.
- tag: cursor
- evidence: bundle
- boundary: no (the Install Now button only runs the update command, no agent output)
- proof paths: `C\out\vs\workbench\workbench.desktop.main.js` (components `Zyk` update card and `Qyk` changelog card, mounted by `k6m`, fragments below), `C\out\vs\workbench\workbench.desktop.main.css` (classes `update-notification-*`, `minor-version-notification-*`)
- example features:
  - Update card, "Update to v{major.minor}": eyebrow plus a one-line blurb, `Changelog` link button (opens `https://changelog.cursor.com`, nightly builds use `?nightly=true`), pulsing `Install Now` button, close X (snoozes). Shown when the version has a blurb in a built-in table or when `forceUpdate` is set.
  - Minor-version card, "New update available": gift icon, `Later` (snoozes) and `Install Now`. This is the default variant when there is no blurb.
  - Changelog card, "New in {major.minor}": blurb, `Changelog` button, close X. Appears after a major or minor version change (`setShouldShowReleaseNotes(!0)` in the release notes startup code) only if the version has a blurb.
  - Rate limits and snooze: experiment config `update_prompt_config` with `min_hours_between_prompts`, `max_prompts_per_version`, `max_prompts_per_day`, and `snooze_duration_hours` (default 24, clamped 1 to 240); telemetry event `cursor.update.promptShown` with `surface:"sidebar"`.
- placement of the update and minor-version cards: `position:fixed`, z-index 2000, 8 px from the left edge when the side bar is on the left (class `update-notification-outer-container-left`) or 8 px from the right otherwise; bottom 30 px when the status bar is visible, else 16 px. Both cards use `cursor-bg-glass` and `cursor-box-shadow-lg`; the minor-version card is 342 px wide.
- placement of the changelog card: inline styles only (340 px wide, blur 40 px, 7 px radius), bottom 30 px with status bar else 8 px; I did not confirm which side it sits on.
- mounting: function `k6m` renders both components into the window container from two side bar part constructors (once the main container has laid out).
- action: `Install Now` runs `cursor.doupdate` with source `notification`. That command (title `Attempt Update`, palette visible) picks the next step from update state: `update.checkForUpdate`, `update.downloadUpdate`, `update.installUpdate`, or `update.restartToUpdate`.
- bundle fragments: `update_prompt_config`, `snooze_duration_hours`, `executeCommand("cursor.doupdate","notification")`, `changelog.cursor.com`, `update-notification-xmark`.
- also in this group (classic notifications, they DO use the notification service): `There is an available update.` with `Download Update`, `Later`, `Release Notes` (nls 11503-11506); `A newer {0} package ({1}) is available via {2}.` (11507); `{0} couldn't finish installing the update to version {1}.` with `Try Again` (11493, 11494) and the retry variant (11492); `Updates are disabled because you are running the user-scope installation of {0} as Administrator.` with `Learn More` (11495, 11496); `This version of {0} does not have release notes online` (11491). Module `contrib/update/browser/update`. V 1.128.0 no longer has the available-update and package-update prompts.

## Cursor additions (summary)

- NT-6 floating update, minor-version and changelog cards, with snooze and rate limits (above)
- `Severity.Success` and `Severity.AI` in the notification severity enum, `pass` icon for Success
- Longer toast timers (15, 18, 20 s), toasts anchored on the side bar side, 340 px compact styling with glass shadow
- `cursorai.action.hideNotificationsForMs` command to suppress toasts for a while (used by onboarding)
- Extra notification fields `toastDurationMs`, `closeNotificationOnToastDismiss`, `toastTitle`, `toastDescription`, `toastCode` (plumbed through the model and `prompt()` options)
- Memory warning prompt: Warning severity, sticky, URGENT priority, text `Cursor is using a lot of memory ({N} MB). We can capture a snapshot for debugging, and attempt to free up some memory in the process.` with button `Capture Snapshot` (module memory monitor, event `renderer.memoryMonitor.userToast`). The same monitor can also add a loud status bar entry (Status bar section, SS).
- Sign-in policy error toast: `Sign-in on this device is restricted by your organization's device policy. Sign in with an allowed account to continue.` (Error severity, method `showSignInPolicyViolationError`)
- Classic update prompts kept from older VS Code (see NT-6)
- Cursor features that reuse the toast channel (they belong to their own sections, not NT): Browser (`Cookies cleared`, `Cache cleared`, `Browsing history cleared`), git (`Failed to commit`, `Switched to branch`), usage and billing (`Usage-based pricing enabled successfully...`, `Spend limit set successfully!`), internal bug report (`Bug report submitted. Thanks for helping us improve Cursor!`, nls 11405)

## V items missing from Cursor

- Notification position setting `workbench.notifications.position` (bottom right, bottom left, top right): not in the bundle, not in the label table. No `workbench.action.setNotificationsPosition.*`, no `workbench.action.configureNotificationsPosition`, no `Configure Notifications Position...` label, no `NotificationsCenterPositionMenu`. The center and toast css has no `top-right` or `bottom-left` rules. Checked with rg on the ids and a scan of the whole label table.
- Title bar bell and `workbench.notifications.showInTitleBar`, plus the `Notifications` title bar toggle: not found.
- Left-aligned bell (the bell is always right-aligned in Cursor).
- V post-update widget (`_update.showUpdateInfo`, setting `update.showPostInstallInfo`, labels `New in {0}`): not found; Cursor's changelog card plays a similar role.
- V title bar update entry (`workbench.actions.updateIndicator`, setting `update.titleBar`): not found; Cursor has its own title bar update button (TB-5).
- V code for this is in stock 1.128.0 source, so Cursor's merge base seems to predate it or Cursor removed it. I cannot tell which.

## Newer than 1.128

- Nothing in the Cursor bundle is newer than 1.128 in this area. The items seen only in N (telemetry, compact row height API, pending toast dedupe, `NotificationText`, `hasActiveProgress`) are listed in the VS Code file. Cursor's own compact rows (28 px) are a separate, earlier change and do not use N's API.

## Left out

- Agent and chat prompts: `composer-warning-popup`, `bugbot-review-popup`, `composer-action-notification` (css classes), `Thanks for your feedback!` agent feedback popup, `Failed to generate branch name for worktree`, `Skills synced...`, attach-file and cloud agent errors, `You cannot summarize while the agent is still working.`
- MCP: `New MCP server detected: {name}` prompt and MCP auth errors
- Managed hooks prompt (`Your team admin has configured managed hooks...`), agent-side update button `AgentSidebarUpdateButton` (nls 4796-4800)
- Agents window toasts (`workbench.glass.main.js`, not read)
- Billing and usage toasts (prepaid top-up `toastDurationMs` calls, spend limit, limit increase): they belong to Cursor Settings or billing UI
- The `Upgrade` button `upgrade-pro-button` (title bar, TB)
- Extension recommendation and extension management prompts (extension system, out of scope)
- Terminal OSC notifications (`terminal.integrated.enableNotifications`, `Disable Terminal Notifications`, nls 10860): present in Cursor, belongs to the terminal in the Panel section

## Not verified

- I did not see the app run. Which update card appears for a given build is inferred from code: the built-in blurb table I could read has 6 entries (`0.46` to `1.0`), none match 3.27, so I expect the default `New update available` variant unless `forceUpdate` is set, and I expect no changelog card. The side bar agent reported seeing `Update to v...` cards, which would mean `forceUpdate` or a table I read incompletely. Not resolved.
- Where `forceUpdate` and the `update_prompt_config` numbers come from (server experiment config) is not visible in the bundle.
- No reader found for `toastDurationMs`, `toastTitle`, `toastDescription`, `toastCode`, or `closeNotificationOnToastDismiss` in the main bundle. They may be consumed in the Agents window bundle (not read) or be dead plumbing.
- `Severity.AI` is defined but I found no icon, style or caller for it in notifications.
- I did not find any indexing prompt or plan upgrade prompt that uses the notification service. My scan of `notify` and `prompt` calls covered about 200 call sites with literal text and the label table, but it is not exhaustive (calls through other variable names or built text could be missed). Sign-in showed up only as the two error toasts above.
- "Show Notifications Log": no command, label or css found. Treated as not existing in Cursor (same result in V and N).
- Why Cursor lacks the position feature that exists in V 1.128.0 source is unknown.
