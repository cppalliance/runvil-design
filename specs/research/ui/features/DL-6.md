Group: DL-6 Cursor plan and account modals. Section: Dialogs. Date: 2026-10-25. Pass: 2.
Features: 15.

### DL-6.1 Subscription tiers modal
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A centered modal (900 px wide, up to 90 percent of the window) titled "Upgrade Your Plan" with the line "Choose the plan that works best for you". It shows plan cards side by side: Pro ($20/month, "Great for individual developers"), Pro+ ($60/month, "More power for demanding projects") and Ultra ($200/month, "Maximum power and flexibility"), each with a short tick list of features. Which cards show depends on the current plan (a Free account sees all three, a Pro account sees Pro+ and Ultra). One card is marked "Recommended", and trial accounts see "N Days Left" chips. Clicking a card selects it, and the primary button reads "Upgrade to {plan}" (or "Start {plan} now" for a trial of that plan) with Cancel beside it. Pressing the button runs the upgrade flow in DL-6.2 (confirmation if a card is on file, otherwise the web checkout in the browser) and closes the modal. The code also has a "Free N Day Trial" chip and a "Start free trial" label, but the switch that turns them on is fixed off.
- Commands: cursor.showSubscriptionTiersModal
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only - command title "Show Subscription Tiers Modal" (not localized, listed in F1); also opened by the Upgrade link on the empty editor landing screen (ED-9)
- Source: C bundle (module uiOverlayService, method showSubscriptionTiersModal and observable subscriptionTiersModal; component KVw with the plan list vBr; command registration with f1 true; landing screen click handler runs the command)

### DL-6.2 Confirm a plan upgrade
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the Free plan "Upgrade to Pro" row in Cursor Settings General opens it; it is also reached from the title bar Upgrade button, the tiers modal and chat error buttons
- What: When an upgrade starts and the account has a saved payment method, a 400 px modal titled "Confirm Upgrade to Pro" (or Pro+, Ultra) shows a one-line pitch for the plan and a warning-colored charge line: "Your saved payment method will be charged $20/month." (or $60, $200), or on a paid plan "You will be charged $60/month, and receive a prorated refund for the remaining days in your current plan." Buttons are Cancel and a rocket "Upgrade to {plan}". It cannot be closed while the upgrade runs. Confirming asks the server to upgrade at once and opens the invoice page in the browser when one comes back. With no saved payment method, or on any error, the web checkout page opens instead and no modal shows. Cancel, the X and a failed instant upgrade all count as "not done", and the flow code then opens the web checkout page too (read from code, not run). A second form takes its title, message and button label from the server after a dashboard action asks for confirmation.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module upgradeUtils, helper eAt; module uiOverlayService, methods showUpgradeConfirmationModal and showServerUpgradeConfirmationModal; component vqw with the per-plan text; helper tAt for the server form; callers: the Free upgrade row component Ikw in Cursor Settings General, the title bar Upgrade click handler, KVw)

### DL-6.3 Usage pricing modal
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it opens from the "Skip the wait." link in the chat queue line ("Slow request ... (n) Skip the wait."), and the rows inside it save account billing settings
- What: A small centered modal (300 px) titled "Configure Usage-Based Pricing" with the line "Skip the wait. Pay only for what you use." and a close X opens from the Command Palette as "Show Usage Pricing Modal" with the controls in DL-6.4 and DL-6.5 and a closing line "You can see your usage and change your hard limit in the account settings" whose last words open the web account settings page, and when it opens from a link outside the editor workbench it also shows one extra sentence from a server price check.
- Commands: cursorai.action.showUsagePricingModal
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only - command title "Show Usage Pricing Modal" (F1)
- Source: C bundle (module uiOverlayService, method showUsageBasedPricingModal with the argument justshow or usage details; component Nqw; detail component Pqw; queue line template with "Skip the wait."; command registration with f1 true)

### DL-6.4 Turn usage-based pricing on or off
- Tag: cursor
- Evidence: bundle
- Boundary: yes - lives in the usage pricing modal, which opens from the chat queue link
- What: Inside the usage pricing modal a switch "Enable usage-based pricing" is on when a spend limit exists. Turning it on sets the limit to $50, and turning it off sets it to $0 (no usage-based). When it is on, a second switch "Enable usage-based pricing for premium models" appears. Both dim and show a loading state while they save, and a server error shows the server's title and detail in red.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component Lqw, switch component LSg, calls setHardLimit and setUsageBasedPremiumRequests on the dashboard client)

### DL-6.5 Set the usage hard limit
- Tag: cursor
- Evidence: bundle
- Boundary: yes - lives in the usage pricing modal, which opens from the chat queue link
- What: A "Hard limit" text box (placeholder "Enter usage-based spend limit") takes a dollar amount, with the dollar sign added for the person. It saves on Enter, when the box loses focus with a changed value, or with the small button beside it, which reads "Save", "Saving..." and then "Saved" with a check. An empty box becomes $0 (usage-based off), a limit at or above the cap shows "Unlimited", and a bad entry shows "Please enter a valid number". The box is dimmed and locked while usage-based pricing is off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component Lqw, input handlers for keydown, blur and input, template Cqw with the Hard limit label, calls getHardLimit and setHardLimit)

### DL-6.6 Free trial ended modal
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: When a subscription refresh shows the account is on a trial plan with no days left, a 500 px modal appears with the heading "Your Free Trial Has Ended", the line "Thank you for trying Cursor! To continue enjoying premium features like:", a list (Unlimited Tab completions, Extended Agent usage, Cloud Agents, And more...) and two buttons: a primary "Upgrade to Pro" (or "Upgrade to Pro+" after a Pro+ trial), which opens the web checkout in the browser, and a "Maybe Later" button that closes it. It also closes by itself if the account moves off the Free plan.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component mqw in the UI overlay list Bqw; subscription changed listener with getDaysRemainingOnTrial; button calls cursorAuthenticationService.pay)

### DL-6.7 Free autocomplete used up popup
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: When Cursor Tab is on but the server says Tab needs a paid plan and the account is not on one, a dimmed full-window backing shows a card with a red X icon, "You've used up your free autocomplete requests." and a paragraph that unlimited Cursor Tab needs a paid plan and that Cursor Pro also has much higher premium model limits. Buttons are "Turn off autocomplete" (closes the popup; I saw no setting change in its handler) and a rocket "Upgrade to keep" (opens the web checkout in the browser). A small X in the corner also closes it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component Fqw with template Aqw; cppService.showingCppUpsell set by the server config update when cppEnabled is true and shouldLetUserEnableCppEvenIfNotPro is false; the Upgrade button calls pay)

### DL-6.8 Open the Invite Team Members modal
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the only way in is the Invite button on the General tab of Cursor Settings
- What: On the General tab of Cursor Settings, a team account whose person is allowed to invite (membership type Enterprise, which covers team plans) sees an "Invite Team Members" row ("Accelerate development, shared knowledge, and context across your team") with an "Invite" button and a link icon. It opens a modal over a dimmed backdrop, and a click on the backdrop closes it. The modal is not drawn in the Agents window and has no command or menu entry.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (account section component Yww with the row condition for membership ENTERPRISE and viewerCanInviteMembers; the landing screen plan label shows "Enterprise" or "Team" for that same membership type, which is why I say it covers team plans; module uiOverlayService, methods showInviteModal and closeInviteModal; overlay component Oqw)

### DL-6.9 Copy the team invite link
- Tag: cursor
- Evidence: bundle
- Boundary: yes - lives in the invite modal, which opens from Cursor Settings
- What: In the invite modal, the title reads "Invite team members" with a close icon. Under "Invite link" a read-only box shows the team's invite link ("Loading..." until it arrives) and a "Copy link" button copies it, with the icon changing to a check for about two seconds. The button stays disabled until the link has loaded.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component HVw, template $Vw, calls getTeamInviteLink on the dashboard client, clipboardService.writeText)

### DL-6.10 Invite teammates by email
- Tag: cursor
- Evidence: bundle
- Boundary: yes - lives in the invite modal, which opens from Cursor Settings
- What: Under "Invite by email" an input (placeholder "Email, comma separated") takes one or more addresses. The "Send invite" button (arrow icon, loading icon while it works) stays disabled until the text looks like an email address, and Enter sends too. Each valid address gets a team invite with the Member role, and the modal closes when the sending is done.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (component HVw, sendTeamInvite on the dashboard client with role MEMBER, email check `/\S+@\S+\.\S+/`)

### DL-6.11 Choose data sharing or Privacy Mode
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the control sits in the Privacy section of the Cursor Settings General tab, and Privacy Mode also limits cloud agent features
- What: A signed-in person sees a "Privacy" section on the General tab. Its row reads "Data Sharing Enabled" (check icon) or "Privacy Mode" or "Privacy Mode (Legacy)" (lock icon) with a sentence on what that means, and a "Configure" dropdown with "Share Data" ("Improve Cursor for everyone"), "Privacy Mode" ("No training. Code may be stored for Background Agent and other features.") and, only for someone already on it, "Privacy Mode (Legacy)". A small ring shows while the change saves. On the legacy mode a note says "Privacy Mode (Legacy) is enabled. Background Agent and some features not available." with a "Switch to Privacy Mode" link. On team accounts the dropdown is hidden and the text says "Data Sharing is controlled by your team admin." with a link. A first-day note ("Data Sharing is paused for your first day of usage. Sharing will activate in N hours.") has a "Share Now" link that is switched off in code, and the command it calls is not registered anywhere I found.
- Commands: cursorAuth.skipPrivacyModeGracePeriod
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only - the command id appears only as the call from the dead "Share Now" link
- Source: C bundle (component Hww inside the General tab component, with helper Www; options list for Share Data, Privacy Mode, Privacy Mode (Legacy); updateUserPrivacyMode and setServerRecordedPrivacyMode; switch S1h fixed false)

### DL-6.12 Switch Privacy Mode confirm
- Tag: cursor
- Evidence: bundle
- Boundary: yes - opens from the Privacy section of Cursor Settings
- What: When a person on Privacy Mode (Legacy) picks another mode, either from the dropdown or the "Switch to Privacy Mode" link, a dialog titled "Switch Privacy Mode?" shows a warning callout, "You won't be able to switch back to Privacy Mode (Legacy)." with buttons "Switch" and "Cancel". Cancel leaves the mode as it was. No box shows for any other change.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only - drawn by Cursor's own pretty dialog engine (DL-1), so `window.dialogStyle` does not change it
- Source: C bundle (component Hww, openDialog call with title "Switch Privacy Mode?", callout message, primaryButton Switch, cancelButton Cancel, shown only when the current mode is NO_STORAGE)

### DL-6.13 Privacy update failed box
- Tag: cursor
- Evidence: bundle
- Boundary: yes - opens from the Privacy section of Cursor Settings
- What: If saving a privacy change fails, a dialog titled "Failed to update privacy settings" says "Please try again. Request ID: {id}" with one OK button, and the mode shown stays as it was.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only - Cursor's pretty dialog engine
- Source: C bundle (component Hww, openDialog call with title "Failed to update privacy settings", primaryButton OK, request id from qn())

### DL-6.14 Import settings from VS Code confirm
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A sticky info notification "Bring your VS Code setup to Cursor. Import your settings and extensions for a seamless start." has an Import button. It appears some time after onboarding when the profile has not imported yet and a .vscode folder with content exists in the person's home folder (not in the Agents window). Import opens a dialog titled "Import settings from VS Code" with the text "Settings, extensions, and other profile preferences will be replaced with current settings from VS Code" and three buttons: Import (primary), "Import without extensions" and Cancel. Either import button runs the import command with the matching option.
- Commands: cursorai.action.importvscode
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only - the command is "Import VS Code Extensions and Settings" (F1); the notification text is messages 8445 and 8446; the dialog is Cursor's pretty dialog engine (DL-1)
- Source: C bundle (function hHw with the openDialog call and the dialog icon for import; class wKa with schedulePostOnboardingVSCodeImportToast and maybeShowPostOnboardingVSCodeImportToast; helper kbg checks the .vscode folder; wbg registers the command with LABEL "Import VS Code Extensions and Settings")

### DL-6.15 Import from VS Code in Cursor Settings with confirm popup
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the row sits on the General tab of Cursor Settings
- What: In the Preferences group of the General tab, the "Import Settings from VS Code" row ("Import settings, extensions, and keybindings from VS Code") has an Import button. It opens a small popup, "Confirm VS Code import", that says "Are you sure? This replaces your settings and extensions with your vscode ones. This cannot be undone." with Cancel and Confirm. After Confirm the button reads "Importing" and then "Success!" for about five seconds, and a notification says "VS Code import completed!" or "Error importing extensions, please try again."
- Commands: cursorai.action.importvscode
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only - a popover, not a dialog; the Command Palette entry for the same command asks nothing in the code I read
- Source: C bundle (component gww in the Preferences section component mww, text constant uww, Popup with aria-label "Confirm VS Code import", executeCommand with the import command id)

## Handed off

- The "Bring your VS Code setup to Cursor" notification that opens DL-6.14 - NT-2
- The "Update required" or "Update recommended" modal (Close and Update buttons, opens the downloads page; driven by the server's client version status) - NT-6
- The "Reset Don't Ask Again Dialogs" row in Cursor Settings General, which clears saved don't-ask-again choices for Cursor's own dialogs - DL-1
- The Free "Upgrade" row, "Cursor Account" row, Sign In row and Log Out button on the General tab, which have no modal of their own beyond DL-6.2 - ED-8.20
- The title bar Upgrade button and the landing screen plan label with its Upgrade link, which open DL-6.2 and DL-6.1 - TB-5.7 and ED-9
- The "Upgrade here." link in the Cursor Tab panel, whose target I did not trace - AC-8

## Unclear

- No sign-in or log-in modal outside the agent surfaces: I found the Cursor account sign-in only as the Sign In row in Cursor Settings (opens the browser), the Accounts menu rows, the onboarding Welcome step (ED-9) and the AI error popup "Please login to continue using Cursor." (left out as chat). I list no feature for it
- Cancel and the X on the upgrade confirmation resolve as "not done", and the shared flow code then opens the web checkout page. This is read from the code, not run, so it may behave differently
- I did not trace how the title bar and Settings Upgrade buttons choose between the confirmation modal and a server-worded confirmation. The helper takes the next upgrade from usage data and only runs the confirmation flow for the plain Pro upgrade
- "Free N Day Trial" and "Start free trial" in the tiers modal are in the code but their switch is fixed off, so I did not count them as live
- The Privacy section also has a header link to the web dashboard for accounts on the unified dashboard. I saw it in code only and did not map its condition
- The chat and agent upgrade buttons ("upgrade", "upgradeChoice", "configureSpendLimit" in AI error popups) reuse DL-6.2 and the usage pricing flow but belong to chat and are not listed
- The in-app ad modal (component UVw, driven by the ad service for things like Bugbot and commit review) and the Private Inference download prompt were seen in the same overlay list. Both are agent or inference promotions and are left out
- The data sharing step of onboarding ("Help Improve Cursor" and "Privacy Mode" cards) is part of the onboarding overlay (ED-9), so it is not listed here
- Whether the sign-in related Cursor Settings rows show in the Agents window only, and whether the General tab Privacy section looks the same there, was not checked
- I did not run Cursor, so modal sizes and wording come from the bundle only
