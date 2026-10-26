Group: WB-5 Profiles and appearance pickers. Pass: 3. Date: 2026-10-25.

### WB-5.1
1. After you install a language pack, a sticky notification asks "Would you like to change <product>'s display language to <language> and restart?" with a Change Language and Restart button.
2. That notification has a Don't Show Again choice that is remembered for the whole app.
3. At startup, when the operating system language differs from the display language and no pack for it is installed, a notification offers "Install and Restart" when one matching pack is found, or "Search Marketplace" when several are, with the text in the pack's language and the English text in brackets when a translation exists.
4. Don't Show Again on that startup notification remembers the locale, so it is not offered again for it.
5. Removing the pack of the active language switches the display language back to English.
6. "Configure Display Language" picks an installed language, and "Clear Display Language Preference" resets it.

### WB-5.2
1. On macOS the Settings editor lists Font Aliasing (`workbench.fontAliasing`) under Workbench > Appearance, and it is not listed on Windows or Linux.
2. Value `default` uses sub-pixel font smoothing, which gives the sharpest text on most non-retina displays.
3. Value `antialiased` smooths text at the pixel level, which can make it look lighter overall.
4. Value `none` turns font smoothing off, so text shows with jagged edges.
5. Value `auto` uses antialiased smoothing on high-density displays (2x or more) and the default smoothing otherwise.
6. A change applies to all workbench text at once, without a restart.

## Unclear
none
