Group: WB-5 Profiles and appearance pickers. Section: Workbench-wide. Date: 2026-10-25. Pass: 4.
Features: 2.

### WB-5.1 Display language prompts
- Tag: both
- Evidence: source
- Boundary: no
- What: Notifications offer to switch the display language after a language pack is installed or when the operating system language differs, and two palette commands change it by hand.
- Commands: langugage.update.donotask (the Don't Show Again id, spelled this way in the product, not a command); workbench.action.configureLocale; workbench.action.clearLocalePreference
- Menus: CommandPalette (Configure Display Language and Clear Display Language Preference)
- Keys: none found
- Settings: none found
- Cursor: present - the id, the strings (nls 7595 to 7604) and both related commands are in the bundle
- Source: V\src\vs\workbench\contrib\localization\electron-browser\localization.contribution.ts:20-217; V\src\vs\workbench\contrib\localization\common\localizationsActions.ts:16,99; C bundle (class with checkAndInstall and onDidInstallExtension, neverShowAgain id langugage.update.donotask)

### WB-5.2 Font smoothing on macOS
- Tag: both
- Evidence: source
- Boundary: no
- What: On macOS the person picks how text is smoothed across the whole workbench: sub-pixel, pixel level, none, or chosen by display density.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.fontAliasing
- Cursor: present - the setting is registered with the same four values and nls 4234 holds its description; Cursor's bundle also names a general setting cursor.general.fontSmoothingAntialiased_v2 whose screen was not read
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (lines 703-716); V\src\vs\workbench\browser\workbench.ts (lines 270-295); V\src\vs\workbench\browser\media\style.css (lines 178-193); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (line 133); C bundle and C css (setting registration and the monaco-font-aliasing rules)

## Handed off
- none

## Unclear
- Created in pass 4 from the coverage check.
