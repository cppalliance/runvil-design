Group: WB-1 Saving and file state. Section: Workbench-wide. Date: 2026-10-25. Pass: 4.
Features: 6.

### WB-1.1 Save failed notification for file working copies
- Tag: both
- Evidence: source
- Boundary: no
- What: When a save fails for a file working copy, an error notification explains why and offers buttons such as Retry, Overwrite, Revert, Save As and a save as administrator; the plain text editor has its own separate save error handler.
- Commands: fileWorkingCopy.overwrite; fileWorkingCopy.revert; fileWorkingCopy.retry; fileWorkingCopy.unlock; fileWorkingCopy.saveElevated; fileWorkingCopy.saveAs (all six are button ids, not commands)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - all six ids and the strings (nls 13563 to 13579) are in the bundle
- Source: V\src\vs\workbench\services\workingCopy\common\storedFileWorkingCopy.ts:1143-1232; C bundle (doHandleSaveError with the six button ids, nls 13563 to 13579)

### WB-1.2 Retry or revert after a failed save
- Tag: both
- Evidence: source
- Boundary: no
- What: When a save fails with an error, an error notification names the files and the error and offers Retry and Revert buttons.
- Commands: workbench.action.files.saveEditors; workbench.action.files.revertEditors
- Menus: none found (the buttons sit on the error notification)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (doSaveEditors, lines 437-462); C bundle (button ids workbench.action.files.saveEditors and workbench.action.files.revertEditors, labels Retry nls 7175, Revert All nls 7176, Revert nls 7177, message nls 7178)

### WB-1.3 Save all files except untitled ones
- Tag: both
- Evidence: source
- Boundary: no
- What: The Command Palette entry File: Save All Files writes every unsaved editor to disk without asking for names of untitled files.
- Commands: workbench.action.files.saveFiles
- Menus: CommandPalette (File category, hidden in the Agents window)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (lines 525-532, saveAll with includeUntitled false); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (palette entry, lines 240-244); C bundle (id workbench.action.files.saveFiles with saveAll and includeUntitled false, label Save All Files from nls 7069)

### WB-1.4 Save without formatting
- Tag: both
- Evidence: source
- Boundary: no
- What: Save the active file while skipping format on save and the other steps that normally run when you save.
- Commands: workbench.action.files.saveWithoutFormatting
- Menus: CommandPalette (File category)
- Keys: Ctrl+K S on Linux and Cmd+K S on macOS; Ctrl+K Ctrl+Shift+S on Windows
- Settings: editor.formatOnSave; editor.codeActionsOnSave; files.trimTrailingWhitespace; files.insertFinalNewline; files.trimFinalNewlines
- Cursor: present (id, label, palette row and the Windows chord found; the bundle adds a macOS binding that was not decoded)
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (lines 477-484, skipSaveParticipants true and force true); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (palette entry, lines 228-232); V\src\vs\workbench\contrib\codeEditor\browser\saveParticipants.ts (the steps that are skipped); V\src\vs\workbench\services\workingCopy\common\storedFileWorkingCopy.ts (line 961); C bundle (id workbench.action.files.saveWithoutFormatting with the chord and skipSaveParticipants true, label from nls 7184)

### WB-1.5 New text file in the default language
- Tag: both
- Evidence: source
- Boundary: no
- What: The person starts a blank untitled text file from the File menu, Ctrl+N or the Command Palette, and it opens in the language set by `files.defaultLanguage`.
- Commands: workbench.action.files.newUntitledFile; openEditors.newUntitledFile
- Menus: MenubarFileMenu (group 1_new, order 1, row New Text File); EmptyEditorGroupContext (group 1_file, order 10); EditorTabsBarContext (group 1_file, order 10); ViewTitle (Open Editors view, group navigation, order 5); CommandPalette
- Keys: Ctrl+N (Cmd+N on macOS) on workbench.action.files.newUntitledFile
- Settings: files.defaultLanguage
- Cursor: present - the command id, its label "New Untitled Text File", openEditors.newUntitledFile and files.defaultLanguage were found in the bundle and in nls; the 1_file rows of the empty-group and tab-bar menus and the key were not checked
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts (lines 661-698); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (lines 281-285 and 670-684); V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (lines 930-948); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (lines 322-325); V\src\vs\workbench\services\untitled\common\untitledTextEditorService.ts (lines 247-253); V\src\vs\workbench\services\untitled\common\untitledTextEditorModel.ts (lines 85-90); C bundle (ids workbench.action.files.newUntitledFile and openEditors.newUntitledFile, setting files.defaultLanguage, nls 7186 and 7166)

### WB-1.6 Auto save
- Tag: both
- Evidence: source
- Boundary: no
- What: The person switches auto save on or off from the File menu or the Command Palette, or picks a mode in settings, so files with unsaved changes save themselves after a delay or when focus moves away.
- Commands: workbench.action.toggleAutoSave
- Menus: MenubarFileMenu (Auto Save, group 5_autosave, order 1, checked while files.autoSave is not off); CommandPalette (File: Toggle Auto Save)
- Keys: none found
- Settings: files.autoSave; files.autoSaveDelay; files.autoSaveWorkspaceFilesOnly; files.autoSaveWhenNoErrors
- Cursor: present - the command id with its description, the File menu row in group 5_autosave (checked while `files.autoSave` is not off) and all four settings were found in the bundle; `files.autoSave` has the same default (off on desktop, after a delay on web) and is in Cursor's list of commonly used settings
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (lines 538-556); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (lines 717-724); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (lines 261-293); V\src\vs\workbench\services\filesConfiguration\common\filesConfigurationService.ts (lines 464-475); V\src\vs\workbench\browser\parts\editor\editorAutoSave.ts; C bundle (id workbench.action.toggleAutoSave, menu group 5_autosave with the toggled condition on config.files.autoSave, the four settings, product.json commonlyUsedSettings)

## Handed off
- none

## Unclear
- Created in pass 4 from the coverage check.
