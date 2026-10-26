Group: AB-3 Badges on rail icons. Pass: 3. Date: 2026-10-25.

### AB-3.1
1. A small count pill appears at the lower right of a rail icon when its container reports a count, for example pending changes on Source Control, unsaved files on Explorer, active sessions on Run and Debug, test counts on Testing, and extension updates or restarts on Extensions.
2. When more than one view in the same container reports a count, the pill shows the sum of all of them.
3. The pill is hidden when the total is zero.
4. A total above 999 is shortened to thousands: an exact multiple such as 1000 shows `1K`, and anything between shows the whole thousands with a plus, so 1500 shows `1K+`.

### AB-3.2
1. While a long task tied to a container is running, such as file work on Explorer, repository work on Source Control or an install on Extensions, the icon shows a small progress glyph in the pill position instead of a count.
2. The glyph appears only after the task has run for 300 ms and, once shown, stays for at least 300 ms, so short tasks cause no flicker.
3. While the glyph is showing it replaces any icon badge or count pill on the same icon, and the pill comes back when the task ends.
4. The progress glyph that shows for a running container task adds no text of its own to the icon's hover or screen reader label.

### AB-3.3
1. An icon glyph can take the place of the count in the pill, and while one is present it is drawn instead of the count on that icon.
2. Extensions shows a warning triangle in the pill when there is an extensions warning.
3. Testing shows a watch glyph in the pill when tests are set to re-run on change.
4. Testing shows a spinner in the pill while tests are running, with the description "Tests are running". [stock only]

### AB-3.4
1. Hovering a rail icon that has a badge shows the icon name, then its key in parentheses when it has one, then a dash and the badge's description, as in `Source Control (Ctrl+Shift+G) - 3 pending changes`.
2. A screen reader reads that same text as the icon's label.
3. When several badges of the shown kind are on one icon, each badge's description is added after the name in turn.
4. By code reading, the badge description stays in the hover and label even after Hide Badge has hidden the pill.

### AB-3.5
1. Right-clicking a rail icon shows a Hide Badge entry, and picking it removes every kind of badge (count, progress and icon glyph) from that icon.
2. After a badge is hidden, the same entry reads Show Badge, and picking it brings the badge back.
3. Hiding a badge affects only the icon you right-clicked, and other icons keep their badges.
4. The choice is saved per profile, so a hidden badge stays hidden after a restart.

### AB-3.6
1. A badge that a view reports, such as the Source Control change count, is drawn on the icon of whichever container currently holds that view.
2. Dragging the view into a different container moves its badge to that container's icon and takes it off the old one.
3. Giving the view its own icon, so it becomes a container of its own, moves its badge to the new icon.

### AB-3.7
1. The count pill is filled with the `activityBarBadge.background` color and its number is drawn in the `activityBarBadge.foreground` color.
2. Warning icon badges use the `activityWarningBadge.*` colors, and error icon badges use the `activityErrorBadge.*` colors.
3. Themes that define a contrast border draw a thin border around the pill.
4. Setting `workbench.colorCustomizations` can override these badge colors, and the pill is redrawn with the new colors when they change.

### AB-3.8
1. Setting `scm.countBadge` chooses what the Source Control icon counts, with values `all` (the default, the sum of every visible repository), `focused` (only the focused repository) and `off` (no count).
2. With `scm.countBadge` set to `focused`, switching to another repository in Source Control changes the pill to that repository's count.
3. Setting `testing.countBadge` chooses which tests the Testing icon counts, with values `failed` (the default), `passed`, `skipped` and `off`.
4. The Testing icon's badge description follows the counted kind, reading "{n} failed tests", "{n} passed tests" or "{n} skipped tests".

## Unclear
- Every piece above describes the vertical rail. Cursor's default horizontal tab row (AB-6) draws no badge at all, so none of these pieces is visible in Cursor's default layout. Cursor is `present` for these features, so only the missing Tests are running spinner is tagged.
- AB-3.4 piece 4 comes from reading `computeTitle`, which does not check whether badges are enabled. It was not run.
- AB-3.8 piece 2 comes from the code observing the active repository. It was not run.
- AB-3.1 piece 1 lists Explorer unsaved files, but whether that badge shows under Cursor's default `files.autoSave` was not checked (stock skips files with a short auto save delay).
