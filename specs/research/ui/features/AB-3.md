Group: AB-3 Badges on rail icons. Section: Activity bar. Date: 2026-10-25. Pass: 2.
Features: 8.

Reading note: the Cursor field records the stock vertical rail, which Cursor builds only when `workbench.activityBar.orientation` is `vertical`. Cursor's default is `horizontal`, which hides the stock rail and draws its own tab row (AB-6, owned elsewhere). That row draws no badges at all: its `showActivity` does nothing and its tab template has no badge element, so every feature below is invisible in Cursor's default layout. Only the pieces a feature needs to run are recorded per feature.

### AB-3.1 Number badge with 1K+ shortening
- Tag: both
- Evidence: source
- Boundary: no
- What: A small pill with a count sits at the lower right of an icon (Source Control pending changes, unsaved files on Explorer, active debug sessions on Run and Debug, test counts on Testing, extension updates or restarts on Extensions), it adds up every count reported for that container, hides at zero, and shows 1000 as 1K and 1500 as 1K+; Cursor's horizontal row never shows it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\workbench\browser\parts\activitybar\media\activityaction.css; V\src\vs\workbench\services\activity\common\activity.ts; V\src\vs\workbench\contrib\scm\browser\activity.ts; V\src\vs\workbench\contrib\files\common\dirtyFilesIndicator.ts; V\src\vs\workbench\contrib\debug\browser\debugService.ts; V\src\vs\workbench\contrib\extensions\browser\extensionsViewlet.ts; C bundle (badge code with `${a}K+`, count strings NLS 9199, 7388, 7389, 5835, 5836, 6743 to 6746, 11164 to 11166)

### AB-3.2 Progress badge
- Tag: both
- Evidence: source
- Boundary: no
- What: While a long task tied to a container runs (file work on Explorer, repository work on Source Control, installs on Extensions), the icon shows a small progress glyph instead of a count; it appears after 300 ms, stays at least 300 ms, has no hover text of its own, and wins over icon and number badges; Cursor's horizontal row never shows it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts; V\src\vs\workbench\services\progress\browser\media\progressService.css; V\src\vs\workbench\browser\parts\compositeBarActions.ts; C bundle (`showOnActivityBar` code with the 300 ms delay and minimum, `progress-badge` CSS rule)

### AB-3.3 Icon badge
- Tag: both
- Evidence: source
- Boundary: no
- What: An icon glyph can replace the count in the pill, such as a warning triangle on Extensions when there is an extensions warning, a watch glyph on Testing when tests re-run on change, or a spinner while tests run; it beats the number badge, and warning and error glyphs use their own colors; Cursor lacks the Tests are running spinner, and its horizontal row never shows any badge.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (Extensions warning and Testing watch glyphs exist, the Tests are running spinner is missing)
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\workbench\services\activity\common\activity.ts (`IconBadge`, `WarningBadge`, `ErrorBadge`); V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts; V\src\vs\workbench\contrib\extensions\browser\extensionsViewlet.ts; C bundle (`icon-badge` and `icon-overlay` classes, Testing watch string NLS 11163, key `testingRunningBadge` absent from `nls.keys.json`)

### AB-3.4 Badge description in the hover and screen reader label
- Tag: both
- Evidence: source
- Boundary: no
- What: The badge's own description is added after the icon name in the hover and the screen reader label, as in Source Control (Ctrl+Shift+G) - 3 pending changes, and by code reading this text stays even after Hide Badge has hidden the pill; Cursor's horizontal row has no badge text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts (`computeTitle`, `updateTitle`); C bundle (`computeTitle` reads `getVisibleBadges` and adds each description)

### AB-3.5 Hide or show a badge per icon
- Tag: both
- Evidence: source
- Boundary: no
- What: An icon's right-click menu has Hide Badge, which removes every kind of badge from that icon, and the entry then reads Show Badge to bring it back; the choice is saved per profile only while badges are off, and Cursor's horizontal tab menu has no such entry.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts (`ToggleCompositeBadgeAction`, `showContextMenu`); V\src\vs\workbench\browser\parts\compositeBar.ts (`areBadgesEnabled`, `toggleBadgeEnablement`); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (storage key `views.customizations`, `viewContainerBadgeEnablementStates`); C bundle (action id `show.toggleCompositeBadge`, labels NLS 3327 and 3328, `viewContainerBadgeEnablementStates`)

### AB-3.6 A view's badge follows the view to its new container
- Tag: both
- Evidence: source
- Boundary: no
- What: Badges that a view reports, such as the Source Control change count, are drawn on whichever container currently holds that view, so if you drag the view into another container or give it its own icon, the badge moves with it; Cursor's horizontal row never shows the result.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\activity\browser\activityService.ts (`ViewContainerActivityByView`); V\src\vs\workbench\contrib\scm\browser\activity.ts; C bundle (class tracking `viewId` with `onDidChangeContainer` and `onDidChangeLocation`, `showViewActivity` calls)

### AB-3.7 Badge colors
- Tag: both
- Evidence: source
- Boundary: no
- What: The count pill is drawn with the activityBarBadge.background and activityBarBadge.foreground theme colors, warning and error icon badges use the activityWarningBadge.* and activityErrorBadge.* colors, and themes with a contrast border get a thin border around the pill.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Cursor: present
- Source: V\src\vs\workbench\common\theme.ts; V\src\vs\platform\theme\common\colors\miscColors.ts; V\src\vs\workbench\browser\parts\compositeBarActions.ts (`updateStyles`); C bundle (color ids `activityBarBadge.background`, `activityWarningBadge.background`, `activityErrorBadge.background`, NLS 4491 and 4492)

### AB-3.8 Source Control and Testing count badge settings
- Tag: both
- Evidence: source
- Boundary: no
- What: `scm.countBadge` sets the Source Control count to the sum of all repositories, the focused repository only, or off, and `testing.countBadge` sets the Testing count to failed (default), passed, skipped tests, or off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: scm.countBadge; testing.countBadge; git.countBadge
- Cursor: present
- Source: V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts; V\src\vs\workbench\contrib\scm\browser\activity.ts; V\src\vs\workbench\contrib\testing\common\configuration.ts; V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts; C bundle (`scm.countBadge` definition with values all, focused, off; `testing.countBadge` with NLS 11256 to 11259)

## Handed off

- Badges on the Accounts and Manage icons (update available, sign-in requests, Settings Sync, cloud changes, update progress): AB-4.
- Badges on Panel and Secondary Side Bar tabs (Problems count, Comments count, Ports count, Hide Badge in those tab menus): PN-1 and AX-4. They reuse the same badge code.
- The contents of each source (what Source Control counts, which tests count, which extension needs an update): SB-4, SB-6, and the left-out Extensions view.
- The compact rail badge look (smaller pill, `compact-content` class for three or more characters): AB-5, because it only shows when `workbench.activityBar.compact` is on. Cursor lacks that setting.

## Unclear

- The Additional Views menu code tries to add the count after a name, for example Source Control (3), using label format NLS 3324. Its badge lookup reads `activity[0]` from the composite bar model item, and that list is created empty and never written in either stock 1.128.0 or the Cursor bundle (`activity:[]`, no push). By code reading the suffix never shows. Not run, so it is not listed as a feature.
- Cursor's CSS has an extra rule for `.composite-bar .monaco-action-bar.vertical .badge .badge-content` that scales the pill to 0.8. Which view it applies to (the stock rail or another vertical bar) was not checked.
- Whether the Explorer unsaved files badge shows under Cursor's default `files.autoSave`: stock skips files that auto save after a short delay (`hasShortAutoSaveDelay`). The Cursor default for `files.autoSave` was not read here.
- AB-3.8 could also be claimed by SB-4 (Source Control) and SB-6 (Testing) because the two settings belong to those views. It is kept here because both setting descriptions speak of the Activity Bar icon.
- Stock 1.144.0 `compositeBar.ts` and `compositeBarActions.ts` are identical to 1.128.0, so nothing in this group is newer than 1.128.
