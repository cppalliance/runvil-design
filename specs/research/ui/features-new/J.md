Bucket: J. Date: 2026-10-25. Pass: 5.

### NEW-J-1 Git status colors
- Group: SB-4
- Tag: both
- Evidence: source
- Boundary: no
- What: Git paints each changed file in a color for its status (modified, added, deleted, renamed or copied, untracked, ignored, conflict, submodule, and the staged forms of modified and deleted), and a theme or the person can change every one of these colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: git.decorations.enabled; explorer.decorations.colors; workbench.editor.decorations.colors; workbench.colorCustomizations
- Cursor: present - all ten color ids (gitDecoration.addedResourceForeground, modifiedResourceForeground, deletedResourceForeground, renamedResourceForeground, untrackedResourceForeground, ignoredResourceForeground, stageModifiedResourceForeground, stageDeletedResourceForeground, conflictingResourceForeground, submoduleResourceForeground) and their ten labels were found in the Cursor copy of the Git extension manifest and its nls file; how the Source Control list draws them was not checked in the Cursor bundle
- Source: V\extensions\git\package.json (contributes.colors lines 4149-4260; setting git.decorations.enabled lines 3578-3582); V\extensions\git\package.nls.json (colors.* lines 338-347); V\extensions\git\src\repository.ts (getStatusLetter line 58, getStatusColor lines 118-151, resourceDecoration line 308, useIcons line 3002); V\extensions\git\src\decorationProvider.ts (ignore provider line 27, submodule badge lines 103-106, enable and disable lines 276-300); V\src\vs\workbench\services\decorations\browser\decorationsService.ts (lines 79-120); V\src\vs\workbench\contrib\scm\browser\scmViewPane.ts (lines 591 and 636-642); V\src\vs\workbench\browser\workbench.contribution.ts (lines 109-113); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (lines 551-555); C\extensions\git\package.json and package.nls.json
- Pieces:
  1. A modified or type-changed file is drawn in `gitDecoration.modifiedResourceForeground` (dark #E2C08D, light #895503), and a file modified in the index uses `gitDecoration.stageModifiedResourceForeground`, which starts with the same values so the two can be recolored apart.
  2. An added file (or one marked intent to add) uses `gitDecoration.addedResourceForeground` (dark #81b88b, light #587c0c), while a renamed or copied file uses `gitDecoration.renamedResourceForeground` and an untracked file uses `gitDecoration.untrackedResourceForeground`, and those last two both start at dark #73C991 and light #007100.
  3. A file deleted in the working tree uses `gitDecoration.deletedResourceForeground` (dark #c74e39, light #ad0707), and a file deleted in the index uses `gitDecoration.stageDeletedResourceForeground` with the same starting values.
  4. Every conflict state (both deleted, added or deleted by us or by them, both added, both modified) uses `gitDecoration.conflictingResourceForeground` (dark #e4676b, light #ad0707) and carries a `!` badge.
  5. A file that .gitignore covers is greyed with `gitDecoration.ignoredResourceForeground` (dark #8C8C8C, light #8E8E90), and this grey is still applied when `git.decorations.enabled` is off.
  6. A submodule folder gets an `S` badge with the tooltip Submodule and the color `gitDecoration.submoduleResourceForeground` (dark #8db9e2, light #1258a7).
  7. The color shows on the name and the badge letter of rows in the Explorer and the Open Editors view and on the title of editor tabs, and a changed file also colors the folders above it and puts a small dot on them, except when the change is a deletion.
  8. Setting `git.decorations.enabled` (default on) removes Git colors and badges from the Explorer and the Open Editors view when off, and the settings `explorer.decorations.colors` and `workbench.editor.decorations.colors` (both default on) turn off name colors in those views and in editor tab titles.
  9. In the Source Control list each file shows its status letter in the status color while the file name keeps the normal text color, and with `git.decorations.enabled` off the list shows status icons instead of letters.
  10. There is no settings page for these colors, so a person changes them with `workbench.colorCustomizations` or a theme, and each token has its own defaults for light, dark, high contrast and high contrast light themes.
