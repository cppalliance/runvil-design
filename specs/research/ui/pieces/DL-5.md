Group: DL-5 Workspace Trust and link safety prompts. Pass: 3. Date: 2026-10-25.

### DL-5.1
1. When Workspace Trust is on and an untrusted folder opens, a modal with a shield icon asks "Do you trust the authors of the files in this folder?" and the person must answer before going on.
2. The modal body says "{product} provides features that may automatically execute files in this folder.", then "If you don't trust the authors of these files, we recommend to continue in restricted mode as the files may be malicious." with a docs link, then the folder path.
3. For a multi-root workspace the question, the body line and the button sub-lines use the word "workspace" in place of "folder".
4. The first button reads "Yes, I trust the authors" with the sub-line "Trust folder and enable all features", and picking it marks the folder trusted.
5. The second button reads "No, I don't trust the authors", and picking it keeps the window in Restricted Mode and shows the Restricted Mode notice and status item.
6. The X and Escape are disabled, so one of the two buttons has to be picked.
7. The modal is skipped for folders that are already trusted, for empty windows, for virtual workspaces and when trust cannot be changed.
8. The second button's sub-line reads "Open folder in restricted mode". [stock only]
9. The second button's sub-line reads "Browse folder in restricted mode". [cursor only]

### DL-5.2
1. The startup modal can show a checkbox "Trust the authors of all files in the parent folder '{name}'" under the question.
2. The checkbox shows only for a single folder on a local or remote file system whose folder has a parent, and not for AI-generated workspaces.
3. Ticking the checkbox and picking "Yes, I trust the authors" marks the parent folder trusted, so folders next to it under the same parent are trusted too.
4. Picking "Yes, I trust the authors" with the checkbox clear trusts only the one folder.

### DL-5.3
1. Setting `security.workspace.trust.startupPrompt` chooses when the startup modal shows, with values `always`, `once` and `never`.
2. The value `always` asks every time an untrusted workspace opens, `once` asks only the first time and remembers per workspace, and `never` does not ask.
3. The setting defaults to `never`. [stock only]
4. The setting defaults to `once`. [cursor only]

### DL-5.4
1. In a trusted workspace, opening a file from outside the trusted folders with File > Open File shows a shield box asking "Do you want to allow untrusted files in this workspace?" (reading "in this window" in a trusted empty window).
2. The same box shows when an untrusted file is dropped on the editor area or a tab strip, or opened from the OS or the command line.
3. The box body says "You are trying to open untrusted files in a workspace which is trusted." and recommends opening them in Restricted Mode in a new window, with a docs link.
4. The buttons are "Open", "Open in Restricted Mode" (which opens the file in a new window) and Cancel.
5. The checkbox "Remember my decision for all workspaces" writes `open` or `newWindow` to `security.workspace.trust.untrustedFiles` when a button is picked.
6. After an "Open" answer the box does not ask again for that workspace.
7. Setting `security.workspace.trust.untrustedFiles` chooses what happens, with values `prompt` (the default), `open` and `newWindow`, and the last two skip the box.

### DL-5.5
1. When something that runs workspace code starts in an untrusted folder, a shield box asks "Do you trust the authors of the files in this folder?" (workspace wording for multi-root) with a sentence from the caller, then "If you don't trust the authors of these files, we do not recommend continuing as the files may be malicious." and a docs link.
2. The buttons are "Trust Folder & Continue" ("Trust Workspace & Continue" for multi-root), "Manage" and Cancel.
3. Pressing "Trust Folder & Continue" marks the folder trusted and the action goes on.
4. Pressing "Manage" closes the box and opens the Workspace Trust editor.
5. Pressing Cancel closes the box and the action does not run.
6. When the caller gives no sentence, the box says "A feature you are trying to use may be a security risk if you do not trust the source of the files or folders you currently have open."
7. Starting a debug session shows the sentence "Debugging executes build tasks and program code from your workspace."
8. Listing or running tasks, running tests, running a notebook cell, creating a terminal and saving a file as admin each show this box with their own sentence.
9. The box never shows while `security.workspace.trust.enabled` is off.

### DL-5.6
1. With trust on, adding a folder that is not trusted to a trusted workspace, such as with File > Add Folder to Workspace..., shows a shield box asking "Do you trust the authors of the files in this folder?" before the folder is added.
2. The box says "You are adding files that are not currently trusted to a trusted workspace. Do you trust the authors of these new files?"
3. The buttons are Yes and No, where Yes marks the added folders trusted and No marks them not trusted, and the folders are added either way.
4. The box does not show when every added folder is already trusted, when the workspace itself is not trusted, or when Workspace Trust is off.

### DL-5.7
1. Setting `security.workspace.trust.enabled` switches Workspace Trust on or off for the whole app.
2. With it off every folder counts as trusted, so none of the trust prompts, the Restricted Mode notice or the Restricted Mode status item appear.
3. With it off, the Command Palette entries "Workspaces: Manage Workspace Trust" and "Workspaces: Configure Workspace Trust Settings" are hidden.
4. A hidden command line option `disable-workspace-trust` forces Workspace Trust off.
5. The setting ships on. [stock only]
6. The setting ships off. [cursor only]
7. A team admin flag set to true writes the setting true at user level, and it never turns the setting off again. [cursor only]

### DL-5.8
1. Setting `security.workspace.trust.emptyWindow` chooses whether a window with no folder open counts as trusted without asking, and it is on by default.
2. With the setting on and `security.workspace.trust.untrustedFiles` set to `open`, any file opens in an empty window with no untrusted files box.
3. With the setting off, an empty window starts in Restricted Mode.

### DL-5.9
1. Running "Workspaces: Manage Workspace Trust" (`workbench.trust.manage`) from the Command Palette opens the Workspace Trust editor as a pinned tab.
2. The command is hidden from the Command Palette while `security.workspace.trust.enabled` is off.
3. The "Manage" button in the trust request box, the Manage link in the Restricted Mode notice and the "Restricted Mode" status item each open the same editor.
4. A file in an untrusted folder shows a "Workspace Trust Required" placeholder with a "Manage Workspace Trust" button that opens the editor.
5. A setting that needs trust gets a "Manage Workspace Trust" quick fix in the settings JSON, and the hover on its "Requires workspace trust" label in the Settings editor links to the editor.
6. The empty Source Control view has a link that opens the editor.
7. The Beginner walkthrough has an "enable trust" link that opens the Workspace Trust editor to the side. [stock only]

### DL-5.10
1. Running "Workspaces: Configure Workspace Trust Settings" (`workbench.trust.configure`) from the Command Palette opens User Settings filtered to the trust settings tag, which lists the five `security.workspace.trust.*` settings.
2. The "Configure your settings" link in the Workspace Trust editor header runs the same command.
3. The command is hidden from the Command Palette while `security.workspace.trust.enabled` is off.

### DL-5.11
1. Opening an http or https link whose domain is not trusted shows a box asking "Do you want {product} to open the external website?" with the link as the detail.
2. A long link has its query or fragment text shortened in the middle so the detail stays near 60 characters.
3. Pressing "Open" opens the link, and pressing Cancel closes the box without opening it.
4. Pressing "Copy" copies the link to the clipboard and does not open it.
5. Pressing "Configure Trusted Domains" shows the domain picker, and the link opens afterward if the picked rule covers it.
6. Links that come from workspace content, such as editor links, webviews and notebook outputs, open with no box in a trusted workspace.
7. Localhost links and links to the product's default trusted domains open with no box.
8. The box names the product "Cursor", and the default trusted domains are Cursor's own list, including cursor.com and github.com. [cursor only]

### DL-5.12
1. After "Configure Trusted Domains", a quick pick opens with its first entry "Trust {scheme://host}" already selected.
2. The next entry reads "Trust {domain} and all its subdomains" for a host name, or "Trust {ip} on all ports" for an IP address that has a port, and an IP address with no port gets no such entry.
3. The entry "Trust all domains (disables link protection)" follows.
4. The last entry, "Manage Trusted Domains", opens the Trusted Domains file for that link and does not open the link.
5. Picking a trust entry saves it to the person's trusted domain list right away, and the link opens if the new rule covers it.

### DL-5.13
1. Running "Manage Trusted Domains" (`workbench.action.manageTrustedDomain`) from the Command Palette opens an editor tab named "Trusted Domains" holding a JSON with comments file.
2. The comments at the top explain the rule forms (exact host, port, wildcard subdomain, path prefix, IP prefix and `*`) and list the product's default trusted domains.
3. Below the comments a JSON array holds the person's own entries.
4. Saving the file applies the rules it holds.
5. When the file is opened from the link picker, it adds "Currently configuring trust for {link}" and puts the cursor in the list.
6. The default trusted domains listed in the comments are Cursor's own list. [cursor only]

### DL-5.14
1. Setting `workbench.trustedDomains.promptInTrustedWorkspace` turned on makes the link box also show for links from workspace content, such as editor links, webviews and notebook outputs, in a trusted workspace.
2. The setting is off by default, so those links open with no box.
3. Trust is off by default, so every workspace counts as trusted and such links skip the box unless this setting is turned on. [cursor only]

### DL-5.15
1. When another application opens a link that points at a local file, folder or workspace file, a warning box says "An external application wants to open '{path}' in {product}" and asks whether to open that file, folder or workspace file, with Yes and No buttons.
2. The box detail warns "If you did not initiate this request, it may represent an attempted attack on your system", and No stops the open.
3. Choosing Yes with the box "Allow opening local paths without asking" ticked turns `security.promptForLocalFileProtocolHandling` off in the user settings.
4. Links to a remote path show the same box with the checkbox "Allow opening remote paths without asking", which turns `security.promptForRemoteFileProtocolHandling` off.
5. Both settings are on by default and apply to the whole application, and with one turned off the matching kind of link opens with no question.
6. Only local and remote links are asked about in this box, because links for an extension are confirmed separately inside the window.

### DL-5.16
1. On Windows, opening a path whose network share host is not allowed shows a warning box "The host '{host}' was not found in the list of allowed hosts. Do you want to allow it anyway?" with Allow, Cancel and Learn More buttons.
2. The box detail names the path and says "Unless you trust the host, you should press 'Cancel'", and Cancel stops the open.
3. Allow lets the open go ahead, and ticking "Permanently allow host '{host}'" also adds the host to `security.allowedUNCHosts` in the user settings.
4. Learn More opens a web page about the setting and then shows the box again until a choice is made.
5. Setting `security.restrictUNCAccess` (on by default, listed only on Windows) limits access to hosts on the allowed list or confirmed in the box, and turning it off removes the check.
6. Setting `security.allowedUNCHosts` is a list of host names without backslashes (for example `my-server` or `192.168.0.1`), the Settings editor flags a name with a backslash, and a restart is needed after changing it.
7. A file operation against a host that is not allowed fails with an error that ends "Please update the 'security.allowedUNCHosts' setting if you want to allow this host."
8. Changing `security.restrictUNCAccess` raises the restart prompt described in DL-1.38.

### DL-5.17
1. Setting `workbench.externalBrowser` is empty by default, and then http and https links that leave the app open in the system default browser.
2. Setting it to a browser name such as `edge`, `chrome` or `firefox` opens those links in that browser.
3. Setting it to the absolute path of a browser program opens those links with that program.
4. If a path value points at nothing, the problem is only written to the log and the system default browser opens the link.
5. If the chosen browser fails to start, the link opens in the system default browser instead.
6. Only http and https links use the setting, and other kinds of links go to the system's own handler for them.
7. The setting is marked restricted, so a value set by an untrusted workspace is not applied, and it is offered only in the desktop app.

## Unclear

- DL-5.1: stock can swap the title, docs line and button labels for product-supplied text when a workspace is marked AI generated, and Cursor does not use that text. I did not make a piece for it because the text comes from the product file and not from the app.
- DL-5.11: the third button "Always Open This Domain" exists in the Cursor code but its switch is fixed off, so there is no piece for it.
- DL-5.7: how a team admin sets the flag in Cursor was not traced, so piece 7 only says what the code writes.
- DL-5.9: the "Requires workspace trust" hover link is from the feature block and I did not re-read `settingsEditorSettingIndicators.ts`.
