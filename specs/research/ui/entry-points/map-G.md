Bucket: G. Date: 2026-10-25. Pass: 4.
MAP | editor.lightbulb | ED-4.13 | setting | namespace of editor.lightbulb.enabled, the code action lightbulb
EXCL | editor.lineDecorationsWidth | setting | internal | API-only editor option with no settings schema, set in code for embedded editors
MAP | editor.minimap | ED-5.2 | setting | namespace of the minimap options; the on or off switch is ED-5.1
EXCL | editor.mouseStyle | setting | internal | API-only editor option (text, default, copy) with no settings schema
EXCL | editor.none | setting | not-ui | false hit, only a test string in the configuration resolver tests
EXCL | editor.overviewRulerLanes | setting | internal | API-only editor option with no settings schema; the overview ruler itself is ED-5.29
MAP | editor.padding | ED-5.43 | setting | namespace of editor.padding.top and editor.padding.bottom
MAP | editor.parameterHints | AC-3.11 | setting | namespace of editor.parameterHints.enabled and editor.parameterHints.cycle
MAP | editor.pasteAs | ED-2.29 | setting | namespace of editor.pasteAs.enabled, showPasteSelector and preferences
EXCL | editor.placeholder | setting | internal | API-only editor option read by the placeholder text contribution of embedded editors; not a user setting
MAP | editor.readOnly | ED-2.37 | setting | API-only option the workbench sets from the file read-only state; the editor then refuses typing
MAP | editor.readOnlyMessage | ED-2.37 | setting | API-only option the workbench fills in; it is the text of the bubble shown when you type in a read-only editor
MAP | editor.renderRichScreenReaderContent | ED-2.42 | setting | real setting that only applies when editor.editContext is on; Cursor does not have it
EXCL | editor.renderValidationDecorations | setting | internal | API-only editor option (editable, on, off) with no settings schema
EXCL | editor.revealHorizontalRightPadding | setting | internal | API-only editor option set only by the simple editor options helper
MAP | editor.scrollbar | ED-5.29 | setting | namespace of the editor.scrollbar.* options
EXCL | editor.selectOnLineNumbers | setting | internal | API-only editor option, default true; the visible click on a line number is already in ED-2.6
MAP | editor.simple | ED-2.37 | setting | not a setting, a message key for "Cannot edit in read-only input" shown by small input-style editors
MAP | editor.smartSelect | ED-2.7 | setting | namespace of editor.smartSelect.selectSubwords and selectLeadingAndTrailingWhitespace
MAP | editor.suggest | AC-1.14 | setting | namespace of the editor.suggest.* options; the per-kind show switches are the biggest group in that feature
EXCL | editor.tabIndex | setting | internal | API-only editor option with no settings schema
MAP | editor.unicodeHighlight | ED-5.37 | setting | namespace of the editor.unicodeHighlight.* options
EXCL | editor.useShadowDOM | setting | internal | API-only editor option with no settings schema
EXCL | editor.wordWrapBreakAfterCharacters | setting | internal | API-only editor option with no settings schema, not set anywhere in the workbench
EXCL | editor.wordWrapBreakBeforeCharacters | setting | internal | API-only editor option with no settings schema, not set anywhere in the workbench
MAP | editor.wordWrapOverride1 | ED-6.1 | setting | API-only option the diff editor sets on each side from diffEditor.wordWrap
MAP | editor.wordWrapOverride2 | ED-5.36 | setting | API-only option that Toggle Word Wrap (Alt+Z) sets for one editor
MAP | editor.wrapOnEscapedLineFeeds | ED-5.36 | setting | real setting, wraps a line after a literal backslash n when word wrap is on; ED-2.35 is the twin feature
NEW | files.defaultLanguage | NEW-G-1 | setting | language of a new untitled text file; no feature covers New Text File itself
EXCL | files.participants.timeout | setting | extension-system | only read by the working copy participant that exists for extension onWill file events
EXCL | http.electronFetch | setting | extension-system | only changes the fetch that local extensions get
EXCL | http.experimental.networkInterfaceCheckInterval | setting | extension-system | only read by the extension host proxy resolver; Cursor does not have it
EXCL | http.experimental.systemCertificatesV2 | setting | extension-system | only read by the extension host proxy resolver
EXCL | http.fetchAdditionalSupport | setting | extension-system | only extends the fetch used by extensions
NEW | http.noProxy | NEW-G-2 | setting | proxy bypass list for the app's own requests
NEW | http.proxy | NEW-G-2 | setting | proxy URL for the app's own requests
NEW | http.proxyAuthorization | NEW-G-2 | setting | Proxy-Authorization header value
NEW | http.proxyKerberosServicePrincipal | NEW-G-2 | setting | Kerberos principal for proxy sign-in
NEW | http.proxyStrictSSL | NEW-G-2 | setting | whether the proxy certificate is verified
EXCL | http.proxySupport | setting | extension-system | description says it is the proxy support for extensions
NEW | http.systemCertificates | NEW-G-2 | setting | load OS certificate authorities; window reload needed on Windows and macOS
NEW | http.systemCertificatesNode | NEW-G-2 | setting | experimental Node.js certificate loading; stock only
EXCL | http.useLocalProxyConfiguration | setting | remote | only applies as a remote setting during remote development
EXCL | http.webSocketAdditionalSupport | setting | extension-system | only extends the WebSocket used by extensions; Cursor does not have it
EXCL | issueReporter.experimental.webReporter | setting | web-only | registered only by the web build's issue contribution
NEW | issueReporter.wizard.enabled | NEW-G-3 | setting | switches Report Issue from the classic window to the wizard in an editor tab; stock only
NEW | issueReporter.wizard.fullWorkspaceScan | NEW-G-3 | setting | file cap for the wizard's performance scan; stock only
NEW | keyboard.dispatch | NEW-G-4 | setting | code or keyCode matching of key presses to shortcuts; listed on macOS and Linux only
MAP | keyboard.layout | SS-3.38 | setting | the web-only keyboard layout picker writes this setting; Cursor does not have it
NEW | keyboard.mapAltGrToCtrlAlt | NEW-G-4 | setting | treats AltGraph as Ctrl+Alt; listed on Windows only
NEW | keyboard.touchbar.enabled | NEW-G-5 | setting | turns the macOS Touch Bar buttons on or off
NEW | keyboard.touchbar.ignored | NEW-G-5 | setting | command ids hidden from the Touch Bar
NEW | screencastMode.fontSize | NEW-G-6 | setting | size of the keyboard overlay text
NEW | screencastMode.keyboardOptions | NEW-G-6 | setting | what the keyboard overlay shows
NEW | screencastMode.keyboardOverlayTimeout | NEW-G-6 | setting | how long the keyboard overlay stays
NEW | screencastMode.mouseIndicatorColor | NEW-G-6 | setting | color of the mouse click ring
NEW | screencastMode.mouseIndicatorSize | NEW-G-6 | setting | size of the mouse click ring
NEW | screencastMode.verticalOffset | NEW-G-6 | setting | height of the keyboard overlay above the bottom
NEW | security.promptForLocalFileProtocolHandling | NEW-G-7 | setting | confirm box when another app opens a local path in the app
NEW | security.promptForRemoteFileProtocolHandling | NEW-G-7 | setting | confirm box when another app opens a remote path in the app
NEW | security.restrictUNCAccess | NEW-G-8 | setting | Windows network share host allow list and its warning box; also in the restart prompt list of NEW-G-9
MAP | telemetry.enableCrashReporter | ED-8.42 | setting | deprecated setting tagged usesOnlineServices and telemetry, so it shows in the Telemetry Settings filter
MAP | update.channel | NT-6.7 | setting | deprecated alias of update.mode, which NT-6.7 names; the Settings editor shows a deprecation note
EXCL | workbench.browser.experimentalUserTools.enabled | setting | agent | only adds experimental tools to the Browser's Add to Chat menu; Cursor does not have it
NEW | workbench.enableExperiments | NEW-G-9 | setting | its only visible effect is the restart prompt after a change
NEW | workbench.fontAliasing | NEW-G-10 | setting | macOS font smoothing for the whole workbench
EXCL | workbench.hover.reducedDelay | setting | agent | only used by hovers in the chat status panel and the agent sessions list; Cursor does not have it
EXCL | workbench.reduceTransparency | setting | not-ui | in 1.128.0 it only sets a CSS class that no stylesheet uses; Cursor lacks it and has its own cursor.general.reduceTransparency
MAP | git.commandsToLog | SB-4.37 | setting | Git commands whose output is written to the Git output channel that Show Git Output opens
MAP | git.githubAuthentication | SB-4.44 | setting | deprecated predecessor of github.gitAuthentication, which SB-4.44 names
MAP | git.optimisticUpdate | SB-4.2 | setting | the Changes list updates at once after stage, unstage and discard
MAP | git.similarityThreshold | SB-4.1 | setting | decides which added and deleted file pairs show as a rename in the Changes groups
NEW | markdown.copyFiles.destination | NEW-G-11 | setting | where pasted or dropped files are saved
NEW | markdown.copyFiles.overwriteBehavior | NEW-G-11 | setting | rename or overwrite when the target exists
NEW | markdown.editor.drop.copyIntoWorkspace | NEW-G-11 | setting | copy dropped media from outside the workspace into it
NEW | markdown.editor.drop.enabled | NEW-G-11 | setting | Shift+drop of a file inserts a Markdown link
NEW | markdown.editor.filePaste.audioSnippet | NEW-G-11 | setting | snippet used for pasted audio
NEW | markdown.editor.filePaste.copyIntoWorkspace | NEW-G-11 | setting | copy pasted media from outside the workspace into it
NEW | markdown.editor.filePaste.enabled | NEW-G-11 | setting | pasting a file creates a Markdown link
NEW | markdown.editor.filePaste.videoSnippet | NEW-G-11 | setting | snippet used for pasted video
NEW | markdown.updateLinksOnFileMove.enableForDirectories | NEW-G-12 | setting | also update links when a folder moves
NEW | markdown.updateLinksOnFileMove.enabled | NEW-G-12 | setting | never, prompt or always update links after a move
NEW | markdown.updateLinksOnFileMove.include | NEW-G-12 | setting | globs of moved files that trigger the update
NEW | markdown.validate.duplicateLinkDefinitions.enabled | NEW-G-13 | setting | severity for duplicate link definitions
NEW | markdown.validate.fileLinks.enabled | NEW-G-13 | setting | severity for links to missing files
NEW | markdown.validate.fileLinks.markdownFragmentLinks | NEW-G-13 | setting | severity for header fragments in links to other Markdown files
NEW | markdown.validate.fragmentLinks.enabled | NEW-G-13 | setting | severity for header links inside the same file
NEW | markdown.validate.ignoredLinks | NEW-G-13 | setting | links and globs that are never checked; the quick fix writes here
NEW | markdown.validate.referenceLinks.enabled | NEW-G-13 | setting | severity for reference links with no definition
NEW | markdown.validate.unusedLinkDefinitions.enabled | NEW-G-13 | setting | severity for unused link definitions
