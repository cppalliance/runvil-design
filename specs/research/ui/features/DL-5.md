Group: DL-5 Workspace Trust and link safety prompts. Section: Dialogs. Date: 2026-10-25. Pass: 2.
Features: 17.

### DL-5.1 Startup trust prompt
- Tag: both
- Evidence: source
- Boundary: no
- What: When Workspace Trust is on and a folder or workspace is not yet trusted, the window has focus and the startup setting allows it, a modal asks "Do you trust the authors of the files in this folder?" (workspace wording for a multi-root workspace) with a shield icon, the line "{product} provides features that may automatically execute files in this folder.", a docs line and the folder path. It has two buttons, "Yes, I trust the authors" (sub-line "Trust folder and enable all features") and "No, I don't trust the authors" (sub-line "Open folder in restricted mode" in stock). The X and Escape are disabled, so the person must pick one. It is skipped for already trusted folders, empty windows, virtual workspaces and when trust cannot be changed. Picking No shows the Restricted Mode notice and status item.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.enabled; security.workspace.trust.startupPrompt
- Cursor: changed - same flow and strings, but trust is off by default so it never appears until trust is switched on; the No sub-line reads "Browse folder in restricted mode" (message 11863), and the product-supplied text for AI-generated workspaces is not used
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (WorkspaceTrustUXHandler: onDidInitiateWorkspaceTrustRequestOnStartup, doShowModal with disableCloseAction, showModalOnStart); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (requestWorkspaceTrustOnStartup, canSetWorkspaceTrust); C bundle (module workspaceTrust; class with doShowModal and showModalOnStart, messages 11856 to 11867, storage key workspace.trust.startupPrompt.shown)

### DL-5.2 Trust the parent folder from the startup prompt
- Tag: both
- Evidence: source
- Boundary: no
- What: The startup prompt can carry a checkbox "Trust the authors of all files in the parent folder '{name}'". It shows only for a single folder on a local or remote file system whose folder has a parent, and not for AI-generated workspaces. Ticking it and answering Yes marks the parent folder trusted, so sibling folders under it are trusted too, instead of trusting only the one folder. The Workspace Trust editor has a matching Trust Parent button, which ED-8.18 covers.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - message 11858, canSetParentFolderTrust and setParentFolderTrust found in the bundle
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (checkboxString, doShowModal run with checkboxChecked); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (canSetParentFolderTrust lines 522-539, setParentFolderTrust line 541); C bundle (module workspaceTrust; doShowModal calls setParentFolderTrust when the box is ticked, message 11858)

### DL-5.3 Choose when the startup trust prompt appears
- Tag: both
- Evidence: source
- Boundary: no
- What: A setting picks whether the startup prompt shows every time an untrusted workspace opens (always), only the first time (once, remembered per workspace), or never. Stock defaults to never as read in the 1.128.0 source; Cursor defaults to once.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.startupPrompt
- Cursor: changed - same three values and descriptions (messages 11821 to 11824), but the default is once instead of never
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (registered setting lines 806-818, showModalOnStart); C bundle (module workspaceTrust; registered setting with `default:"once"`, enum always, once, never)

### DL-5.4 Open untrusted files prompt
- Tag: both
- Evidence: source
- Boundary: no
- What: In a trusted workspace or trusted empty window, opening a file from outside the trusted folders by File > Open File, dropping it on the editor area or a tab strip, or opening it from the OS or command line shows "Do you want to allow untrusted files in this workspace?" (window wording in an empty window). The shield box says "You are trying to open untrusted files in a workspace which is trusted." and recommends Restricted Mode in a new window. Buttons are Open, "Open in Restricted Mode" (opens the file in a new window) and Cancel, with a checkbox "Remember my decision for all workspaces" that writes open or newWindow to the setting. After an Open answer it does not ask again for that workspace. The setting also lets a person skip the prompt for good.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.untrustedFiles; security.workspace.trust.emptyWindow
- Cursor: present - messages 11836 to 11844 and 11829 to 11832, handleWorkspaceTrust and onDidInitiateOpenFilesTrustRequest found, default prompt
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (WorkspaceTrustRequestHandler, onDidInitiateOpenFilesTrustRequest lines 125-165, setting lines 832-843); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (requestOpenFilesTrust, completeOpenFilesTrustRequest lines 700-775); V\src\vs\workbench\services\editor\browser\editorService.ts (handleWorkspaceTrust); callers of validateTrust: V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts, V\src\vs\workbench\electron-browser\window.ts, V\src\vs\workbench\browser\dnd.ts, V\src\vs\workbench\browser\parts\editor\editorDropTarget.ts, V\src\vs\workbench\browser\parts\editor\multiEditorTabsControl.ts; C bundle (module workspaceTrust; handleWorkspaceTrust, extractEditorResources, messages 11836 to 11844)

### DL-5.5 Trust request when an action needs trust
- Tag: both
- Evidence: source
- Boundary: no
- What: When a person starts something that runs workspace code in an untrusted folder, a shield box asks "Do you trust the authors of the files in this folder?" with the caller's own sentence, then "If you don't trust the authors of these files, we do not recommend continuing as the files may be malicious." Buttons are "Trust Folder & Continue" (workspace wording for multi-root), "Manage" (closes the box and opens the Workspace Trust editor) and Cancel. Callers in scope: Run and Debug ("Debugging executes build tasks and program code from your workspace."), listing or running tasks, running tests, running a notebook cell, creating a terminal, and saving a file as admin. The default sentence is "A feature you are trying to use may be a security risk if you do not trust the source of the files or folders you currently have open." If trust is refused the action does not run (the terminal tab shows its own message, covered by PN-2.9). Never shown while trust is off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.enabled
- Cursor: present - messages 11845 to 11852, and the caller sentences found (5838, 5839, 9892, 11324, 8384, 10366, 12913, 12914, 12915)
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (onDidInitiateWorkspaceTrustRequest lines 203-278); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (requestWorkspaceTrust line 856); V\src\vs\workbench\contrib\debug\browser\debugService.ts; V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts; V\src\vs\workbench\contrib\testing\common\testServiceImpl.ts; V\src\vs\workbench\contrib\notebook\browser\services\notebookExecutionServiceImpl.ts; V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts; V\src\vs\workbench\services\files\electron-browser\elevatedFileService.ts; C bundle (module workspaceTrust; onDidInitiateWorkspaceTrustRequest handler with ContinueWithTrust, Manage and Cancel)

### DL-5.6 Confirm trust when adding an untrusted folder to a trusted workspace
- Tag: both
- Evidence: source
- Boundary: no
- What: When Workspace Trust is on and a folder that is not trusted is added to a trusted workspace (for example Add Folder to Workspace), a shield box asks "Do you trust the authors of the files in this folder?" with the detail "You are adding files that are not currently trusted to a trusted workspace. Do you trust the authors of these new files?" and Yes and No buttons. The answer marks the added folders trusted or not trusted.
- Commands: workbench.action.addRootFolder
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.enabled
- Cursor: present - messages 11853 to 11855, the onWillChangeWorkspaceFolders handler with setUrisTrust found
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (WorkspaceTrustUXHandler.registerListeners, onWillChangeWorkspaceFolders lines 336-372); V\src\vs\workbench\browser\actions\workspaceActions.ts (addRootFolder, route in only); C bundle (module workspaceTrust; handler calling dialogService.confirm then setUrisTrust, messages 11853 to 11855)

### DL-5.7 Turn Workspace Trust on or off
- Tag: both
- Evidence: source
- Boundary: no
- What: One application setting switches the whole feature. When it is off every folder counts as trusted, so none of the prompts, the Restricted Mode notice or status item, or the two trust commands appear (the palette entries are hidden while it is off). Stock ships it on. Cursor ships it off, and a team admin flag can turn it on: when the admin value is true and no enterprise policy sets it, Cursor writes the setting true at user level (it never turns it off again). A hidden command line flag also forces it off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.enabled
- Cursor: changed - default false, enterprise policy name WorkspaceTrustEnabled (minimum version 1.99), the description still says "within VS Code" (messages 11819, 11820), and the team admin value workspaceTrustControls.enabled writes the setting
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (setting lines 799-805); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (WorkspaceTrustEnablementService, calculateWorkspaceTrust line 295 returns true when off); V\src\vs\platform\environment\node\argv.ts (hidden option disable-workspace-trust); C bundle (module workspaceTrust; registered setting with `default:!1` and policy WorkspaceTrustEnabled; team admin settings code reading `workspaceTrustControls?.enabled` and calling updateValue with the user target; getter disableWorkspaceTrust)

### DL-5.8 Trust the empty window by default
- Tag: both
- Evidence: source
- Boundary: no
- What: A setting decides whether a window with no folder open is trusted without asking (default on). Together with the untrusted files setting it lets a person open any file in an empty window without the DL-5.4 prompt. When it is off the empty window starts in Restricted Mode.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.workspace.trust.emptyWindow; security.workspace.trust.untrustedFiles
- Cursor: present - default true, message 11833
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (setting lines 845-851); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (line 324, isEmptyWorkspace branch of calculateWorkspaceTrust); C bundle (module workspaceTrust; registered setting `[Cda]:{type:"boolean",default:!0`)

### DL-5.9 Reach Manage Workspace Trust from prompts, links and the palette
- Tag: both
- Evidence: source
- Boundary: no
- What: The entry to the Workspace Trust editor (the editor body is ED-8.18). Routes in scope: Command Palette "Workspaces: Manage Workspace Trust"; the Manage button in the trust request box (DL-5.5); the Manage link in the Restricted Mode notice (BN-2.1); the "Restricted Mode" status item (SS-4.5); a "Manage Workspace Trust" button on the "Workspace Trust Required" placeholder shown for a file in an untrusted folder; a "Manage Workspace Trust" quick fix on untrusted settings in the settings JSON; the hover on the "Requires workspace trust" setting label (ED-8.6); and a link in the empty Source Control view. Stock also has an "enable trust" link in the Beginner walkthrough, which is not in Cursor. The command opens the editor pinned and is hidden while trust is off.
- Commands: workbench.trust.manage
- Menus: CommandPalette
- Keys: none found
- Settings: security.workspace.trust.enabled
- Cursor: changed - command and label found (message 11836, category Workspaces 11834), seen in 7 places in the bundle, and the Command Palette entry is hidden by default because trust is off; the walkthrough link text ("enable trust") was not found in the message table
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (action lines 769-789, precondition isWorkspaceTrustEnabled and the setting, notice links); V\src\vs\workbench\contrib\workspace\common\workspace.ts (MANAGE_TRUST_COMMAND_ID); V\src\vs\platform\actions\common\actions.ts (f1 entry uses the precondition as its when); V\src\vs\workbench\browser\parts\editor\editorPlaceholder.ts (WorkspaceTrustRequiredPlaceholderEditor); V\src\vs\workbench\contrib\preferences\browser\preferencesRenderers.ts (generateUntrustedSettingCodeActions); V\src\vs\workbench\contrib\preferences\browser\settingsEditorSettingIndicators.ts; V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (welcome content); V\src\vs\workbench\contrib\welcomeGettingStarted\common\gettingStartedContent.ts; C bundle (module workspaceTrust; `workbench.trust.manage` 7 hits, messages 3751, 8663, 8664, 8699, 9296, 11836, 11868)

### DL-5.10 Configure Workspace Trust Settings
- Tag: both
- Evidence: source
- Boundary: no
- What: The Command Palette entry "Workspaces: Configure Workspace Trust Settings" opens User Settings filtered to the trust settings tag, which lists the five security.workspace.trust.* settings. The Workspace Trust editor header also links to it ("Configure your settings"). Hidden while trust is off.
- Commands: workbench.trust.configure
- Menus: CommandPalette
- Keys: none found
- Settings: security.workspace.trust.enabled
- Cursor: present - message 11835, `workbench.trust.configure` seen twice, opens settings with the workspaceTrust tag query
- Source: V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts (action lines 748-766); V\src\vs\workbench\contrib\workspace\browser\workspaceTrustEditor.ts (line 875 header link); C bundle (module workspaceTrust; action with openUserSettings and the tag query, message 11835)

### DL-5.11 External link confirmation
- Tag: both
- Evidence: source
- Boundary: no
- What: Opening an http or https link whose domain is not trusted shows "Do you want {product} to open the external website?" with the link as detail (long query or fragment text is shortened in the middle to about 60 characters). Buttons are Open, Copy (copies the link and does not open it), "Configure Trusted Domains" (runs the DL-5.12 picker, then opens the link if the picked rule covers it) and Cancel. Links that come from workspace content (editor links, webviews, notebook outputs) skip the box in a trusted workspace unless DL-5.14 is on. Localhost and the product's default trusted list never ask.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.trustedDomains.promptInTrustedWorkspace
- Cursor: changed - same box, strings and buttons (messages 11521 to 11525), and the product name reads "Cursor"; the default trusted list is Cursor's own (cursor.com, github.com, MCP vendor hosts and others in product.json); the box can show over a hidden window when the click came from a person; in the Agents window the Open button can route to its browser (out of scope); the validator is not registered in the Agents window; a third button "Always Open This Domain" (message 11524) exists in code but its switch useSimpleTrustAction is set false with no setter found
- Source: V\src\vs\workbench\contrib\url\browser\trustedDomainsValidator.ts (OpenerValidatorContributions.validateLink lines 45-135); V\src\vs\platform\url\common\trustedDomains.ts (isURLDomainTrusted, localhost rule); V\src\vs\workbench\contrib\url\browser\trustedDomainService.ts (isValid); V\src\vs\editor\contrib\links\browser\links.ts (fromWorkspace link opens); C bundle (module trustedDomains; class with validateLink, _buildTrustButton, useSimpleTrustAction; registered with disableInGlass; messages 11521 to 11525); C\product.json (linkProtectionTrustedDomains)

### DL-5.12 Pick a domain to trust from the link prompt
- Tag: both
- Evidence: source
- Boundary: no
- What: After "Configure Trusted Domains" a quick pick lists "Trust {scheme://host}" (preselected), "Trust {domain} and all its subdomains" (or "Trust {ip} on all ports" for an IP with a port), "Trust all domains (disables link protection)" and "Manage Trusted Domains". A trust choice is saved to the user's trusted domain list right away and the link opens if it is now covered; Manage opens the DL-5.13 file at the link.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - messages 11515 to 11520
- Source: V\src\vs\workbench\contrib\url\browser\trustedDomains.ts (configureOpenerTrustedDomainsHandler lines 72-163); C bundle (module trustedDomains; function that builds the same picks, messages 11515 to 11519)

### DL-5.13 Manage Trusted Domains
- Tag: both
- Evidence: source
- Boundary: no
- What: The Command Palette entry "Manage Trusted Domains" opens an editor tab named "Trusted Domains", a JSON with comments file (virtual, scheme trustedDomains). The top comments explain the rule forms (exact host, port, wildcard subdomain, path prefix, IP prefix, `*`) and list the product's default trusted domains, then a JSON array holds the person's own entries. Saving the file applies the rules. When opened from the link prompt it adds "Currently configuring trust for {link}" and puts the cursor in the list.
- Commands: workbench.action.manageTrustedDomain
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present - same command, label (messages 11519, 11520), file URI `trustedDomains:/Trusted Domains` and help comments found; the default trusted list in the comments is Cursor's
- Source: V\src\vs\workbench\contrib\url\browser\trustedDomains.ts (manageTrustedDomainSettingsCommand line 62, openInEditor); V\src\vs\workbench\contrib\url\browser\trustedDomainsFileSystemProvider.ts (help text, computeTrustedDomainContent, writeFile); V\src\vs\workbench\contrib\url\browser\url.contribution.ts (CommandPalette item lines 60-68); C bundle (module trustedDomains; command id workbench.action.manageTrustedDomain, `T8o=Re.parse("trustedDomains:/Trusted Domains")`, help comment text)

### DL-5.14 Ask about links in trusted workspace content
- Tag: both
- Evidence: source
- Boundary: no
- What: A setting makes the DL-5.11 box also appear for links that come from workspace content (editor links, webviews, notebook outputs) in a trusted workspace. It is off by default. Because Cursor ships trust off, every workspace counts as trusted there, so such links skip the box unless this setting is turned on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.trustedDomains.promptInTrustedWorkspace
- Cursor: present - setting registered with default false (message 11527), the validator reads it
- Source: V\src\vs\workbench\contrib\url\browser\url.contribution.ts (setting lines 88-98); V\src\vs\workbench\contrib\url\browser\trustedDomainsValidator.ts (line 50); V\src\vs\workbench\services\workspaces\common\workspaceTrust.ts (trusted when the feature is off); C bundle (module trustedDomains; `promptInTrustedWorkspace":{scope:1,type:"boolean",default:!1`)

### DL-5.15 Confirm when another app opens a path in the app
- Tag: both
- Evidence: source
- Boundary: no
- What: When another application asks the app to open a local or remote file, folder or workspace through its link handler, a warning box asks first, and a checkbox on it turns that question off for good.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.promptForLocalFileProtocolHandling; security.promptForRemoteFileProtocolHandling
- Cursor: present - both settings, the handler table in out\main.js and the channel that writes the settings back were found, and nls 146 to 153 hold the box texts with the product name Cursor filled in
- Source: V\src\vs\code\electron-main\app.ts (lines 156-157 and 835-893); V\src\vs\workbench\electron-browser\window.ts (lines 366-373); V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 382-397); C out\main.js (SECURITY_PROTOCOL_HANDLING_CONFIRMATION_SETTING_KEY with both ids, channel vscode:disablePromptForProtocolHandling); C bundle (the window listener for that channel and both settings); NLS 146 to 153

### DL-5.16 Windows network share host prompt
- Tag: both
- Evidence: source
- Boundary: no
- What: On Windows, opening a path on a network share whose host is not on the allowed list stops with a warning box that lets the person allow the host once or for good.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: security.restrictUNCAccess; security.allowedUNCHosts
- Cursor: present - security.restrictUNCAccess and security.allowedUNCHosts are registered, out\main.js has onUNCHostNotAllowed and the not-allowed error, and nls 2736 to 2741 hold the box texts; security.allowedUNCHosts is not in the leftover list and was added here as a related id
- Source: V\src\vs\platform\windows\electron-main\windowsMainService.ts (lines 1239-1300); V\src\vs\workbench\electron-browser\window.ts (lines 342-366); V\src\vs\workbench\common\configuration.ts (lines 180-235); V\src\vs\base\common\errorMessage.ts (line 31); V\src\vs\code\electron-main\app.ts (lines 447-450); C out\main.js (onUNCHostNotAllowed, vscode:configureAllowedUNCHost, ERR_UNC_HOST_NOT_ALLOWED); C bundle (security.allowedUNCHosts and the restrictUNCAccess registration); NLS 2736 to 2741

### DL-5.17 Choose the browser that opens external links
- Tag: both
- Evidence: source
- Boundary: no
- What: The person sets `workbench.externalBrowser` to a browser name or the path of its program, and web links that leave the app open in that browser instead of the system default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.externalBrowser
- Cursor: present - the setting with its restricted flag and the opener code that reads it are in the bundle
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (lines 34-39); V\src\vs\platform\native\electron-main\nativeHostMainService.ts (openExternal, lines 658-671; openExternalBrowser, lines 673-712); V\src\vs\workbench\electron-browser\window.ts (setupOpenHandlers, lines 872-886); C bundle (setting `workbench.externalBrowser` with restricted true; setupOpenHandlers reading it)

## Handed off

- The Workspace Trust editor body (header with Trust and Don't Trust, features list, Trusted Folders & Workspaces table, Trust Parent, Add Folder) - ED-8.18, already listed there
- The Restricted Mode status bar item - SS-4.5, already listed there
- The Restricted Mode notice strip and the `security.workspace.trust.banner` setting - BN-2.1 and BN-2.2, already listed there
- The terminal trust gate and its refusal message - PN-2.9, already listed there
- The "Requires workspace trust" setting label and the Settings editor trust filter - ED-8.6, already listed there
- The "Workspace Trust Required" editor placeholder with its Manage Workspace Trust button - ED-10 (ED-9 handed it there, but the ED-10 file has no feature for it)
- The "Reset Don't Ask Again Dialogs" row in Cursor Settings General, which clears the saved don't-ask-again memory - DL-1

## Unclear

- The resource trust box ("You are trying to open an untrusted folder. Do you trust the authors of this content?" with only "Trust Folder & Continue" and Cancel) is in stock and in Cursor (messages 11852, 11853), but I found only extension API and agent callers for it, so it is not listed as a feature
- Extension, MCP and agent prompts that call the same trust request (extension enablement, MCP server definitions, chat setup, agent host) are out of scope and not listed
- The automatic tasks prompt ("Allow" and "Disallow" for tasks that run when a folder opens, setting task.allowAutomaticTasks) is a notification with buttons, not a dialog, and no catalog group owns Tasks
- "Developer: Open URL" (`workbench.action.url.openUrl`) sits in the same contribution as the trusted domains code but is an input box that hands a URL to the URL service. It is not a trust prompt, so it is not listed. Cursor has it, but I saw no remembered last input in its code
- Whether a refused link prompt can be re-answered later: Cancel and Copy leave no memory in the code I read
- I did not run Cursor with trust turned on, so what the prompts look like there (size, button order) is read from code only
- How a team admin sees or sets `workspaceTrustControls.enabled` was not traced, only the code that reads it and writes the setting
- The stock default for `security.workspace.trust.startupPrompt` is never in the 1.128.0 source I read. I did not check the shipped product.json for an override
