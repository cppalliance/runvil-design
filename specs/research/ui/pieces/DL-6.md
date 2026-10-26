Group: DL-6 Cursor plan and account modals. Pass: 3. Date: 2026-10-25.

### DL-6.1
1. Running "Show Subscription Tiers Modal" (`cursor.showSubscriptionTiersModal`) from the Command Palette, or pressing the Upgrade link on the empty editor landing screen, opens a centered modal titled "Upgrade Your Plan" with the line "Choose the plan that works best for you".
2. The modal shows plan cards side by side for Pro ($20/month, "Great for individual developers"), Pro+ ($60/month, "More power for demanding projects") and Ultra ($200/month, "Maximum power and flexibility"), each with a short tick list of features.
3. Which cards show depends on the current plan, so a Free account sees all three and a Pro account sees Pro+ and Ultra.
4. One card carries a "Recommended" marker, and trial accounts see "N Days Left" chips.
5. Clicking a card selects it.
6. The primary button reads "Upgrade to {plan}", or "Start {plan} now" for a trial of that plan, with a Cancel button beside it.
7. Pressing the primary button runs the upgrade flow (the confirmation modal if a card is on file, otherwise the web checkout in the browser) and closes the modal, while pressing Cancel only closes it.

### DL-6.2
1. When the account has a saved payment method, a modal titled "Confirm Upgrade to Pro" (or Pro+, Ultra) opens with a one-line pitch for the plan.
2. The confirmation modal opens when an upgrade starts from the "Upgrade to Pro" row in Cursor Settings General, from the title bar Upgrade button, or from the primary button in the subscription tiers modal.
3. A warning-colored line reads "Your saved payment method will be charged $20/month." (or $60 or $200 for the other plans).
4. On a paid plan the line reads "You will be charged $60/month, and receive a prorated refund for the remaining days in your current plan."
5. The buttons are Cancel and a rocket button "Upgrade to {plan}".
6. Pressing the upgrade button asks the server to upgrade at once, the modal cannot be closed while it runs, and the invoice page opens in the browser when one comes back.
7. With no saved payment method, or on any error, the web checkout page opens in the browser and no modal shows.
8. Pressing Cancel or the X, or a failed instant upgrade, counts as not done, and the flow then opens the web checkout page too.
9. After a dashboard action asks for confirmation, a second form of the modal takes its title, message and button label from the server.

### DL-6.3
1. A small centered modal titled "Configure Usage-Based Pricing" opens with the line "Skip the wait. Pay only for what you use." and a close X.
2. When the modal opens from a link outside the editor workbench, it also shows one extra sentence from a server price check.
3. Running "Show Usage Pricing Modal" (`cursorai.action.showUsagePricingModal`) from the Command Palette shows only the usage controls and a closing line "You can see your usage and change your hard limit in the account settings", whose last words open the web account settings page.

### DL-6.4
1. Inside the usage pricing modal a switch "Enable usage-based pricing" is on when a spend limit exists.
2. Turning the switch on sets the limit to $50, and turning it off sets the limit to $0, which means no usage-based pricing.
3. While it is on, a second switch "Enable usage-based pricing for premium models" appears.
4. Both switches dim and show a loading state while a change saves.
5. A server error shows the server's title and detail in red.

### DL-6.5
1. A "Hard limit" text box with the placeholder "Enter usage-based spend limit" takes a dollar amount, and the dollar sign is added for the person.
2. The value saves when Enter is pressed, when the box loses focus with a changed value, or when the small button beside the box is pressed.
3. The small button reads "Save", then "Saving..." while it works, then "Saved" with a check.
4. An empty box becomes $0, which turns usage-based pricing off, and a limit at or above the cap shows "Unlimited".
5. A bad entry shows "Please enter a valid number".
6. The box is dimmed and locked while usage-based pricing is off.

### DL-6.6
1. When a subscription refresh shows the account is on a trial plan with no days left, a modal appears with the heading "Your Free Trial Has Ended" and the line "Thank you for trying Cursor! To continue enjoying premium features like:".
2. The modal lists "Unlimited Tab completions", "Extended Agent usage", "Cloud Agents" and "And more...".
3. The primary button reads "Upgrade to Pro" (or "Upgrade to Pro+" after a Pro+ trial) and opens the web checkout in the browser.
4. The "Maybe Later" button closes the modal.
5. The modal closes by itself if the account moves off the Free plan.

### DL-6.7
1. When Cursor Tab is on but the server says Tab needs a paid plan and the account is not on one, a dimmed full-window backing shows a card with a red X icon and the heading "You've used up your free autocomplete requests."
2. The card's paragraph says that unlimited Cursor Tab needs a paid plan and that Cursor Pro also has much higher premium model limits.
3. The rocket button "Upgrade to keep" opens the web checkout in the browser.
4. The "Turn off autocomplete" button closes the popup.
5. A small X in the corner also closes the popup.

### DL-6.8
1. On the General tab of Cursor Settings, a team account whose person is allowed to invite sees an "Invite Team Members" row with the line "Accelerate development, shared knowledge, and context across your team", an "Invite" button and a link icon.
2. Pressing "Invite" opens the invite modal over a dimmed backdrop.
3. Clicking the backdrop closes the modal.
4. The modal has no command or menu entry and is not drawn in the Agents window.

### DL-6.9
1. The invite modal has the title "Invite team members" with a close icon.
2. Under "Invite link" a read-only box shows the team's invite link, and it reads "Loading..." until the link arrives.
3. Pressing "Copy link" copies the link, and the button icon changes to a check for about two seconds.
4. The "Copy link" button stays disabled until the link has loaded.

### DL-6.10
1. Under "Invite by email" an input with the placeholder "Email, comma separated" takes one or more addresses.
2. The "Send invite" button, which has an arrow icon, stays disabled until the text looks like an email address.
3. Pressing "Send invite" or pressing Enter in the input sends the invites, and a loading icon shows on the button while it works.
4. Each valid address gets a team invite with the Member role, and the modal closes when the sending is done.

### DL-6.11
1. A signed-in person sees a "Privacy" section on the General tab of Cursor Settings with a row that reads "Data Sharing Enabled" (check icon), "Privacy Mode" or "Privacy Mode (Legacy)" (lock icon), plus a sentence on what that means.
2. A "Configure" dropdown offers "Share Data" ("Improve Cursor for everyone") and "Privacy Mode" ("No training. Code may be stored for Background Agent and other features.").
3. "Privacy Mode (Legacy)" shows in the dropdown only for someone who is already on it.
4. A small ring shows while a change saves.
5. On the legacy mode a note reads "Privacy Mode (Legacy) is enabled. Background Agent and some features not available." with a "Switch to Privacy Mode" link.
6. On team accounts the dropdown is hidden and the text says "Data Sharing is controlled by your team admin." with a link.

### DL-6.12
1. When a person on "Privacy Mode (Legacy)" picks another mode, from the dropdown or from the "Switch to Privacy Mode" link, a dialog titled "Switch Privacy Mode?" opens.
2. The dialog shows a warning callout "You won't be able to switch back to Privacy Mode (Legacy)."
3. The buttons are "Switch" and "Cancel", where "Switch" changes the mode and "Cancel" leaves the mode as it was.
4. No dialog shows for any other change of mode.
5. The dialog looks the same whatever `window.dialogStyle` is set to, because Cursor draws it with its own dialog engine.

### DL-6.13
1. If saving a privacy change fails, a dialog titled "Failed to update privacy settings" opens.
2. The dialog says "Please try again. Request ID: {id}" with a single "OK" button.
3. The mode shown in the Privacy section stays as it was.
4. The dialog looks the same whatever `window.dialogStyle` is set to, because Cursor draws it with its own dialog engine.

### DL-6.14
1. A sticky info notification "Bring your VS Code setup to Cursor. Import your settings and extensions for a seamless start." with an "Import" button appears some time after onboarding.
2. The notification shows only when the profile has not imported yet and a `.vscode` folder with content exists in the person's home folder, and never in the Agents window.
3. Pressing "Import" opens a dialog titled "Import settings from VS Code" with the text "Settings, extensions, and other profile preferences will be replaced with current settings from VS Code".
4. The dialog's buttons are "Import" (primary), "Import without extensions" and "Cancel".
5. Either import button runs `cursorai.action.importvscode` with the matching option, which is also in the Command Palette as "Import VS Code Extensions and Settings", and "Cancel" imports nothing.

### DL-6.15
1. In the Preferences group of the General tab of Cursor Settings, the row "Import Settings from VS Code", with the line "Import settings, extensions, and keybindings from VS Code", has an "Import" button.
2. Pressing "Import" opens a small popup, "Confirm VS Code import", that says "Are you sure? This replaces your settings and extensions with your vscode ones. This cannot be undone." with Cancel and Confirm buttons.
3. Pressing Cancel closes the popup and imports nothing.
4. After Confirm the button reads "Importing", and then "Success!" for about five seconds.
5. A notification says "VS Code import completed!" or "Error importing extensions, please try again."
6. The confirmation is a popup anchored to the button and not a dialog, and the Command Palette entry for the same command asks nothing.

## Unclear

- I left `[cursor only]` off every piece in this group, as the assignment allows, because every feature here exists only in Cursor.
- DL-6.2 piece 8: Cancel, the X and a failed upgrade opening the web checkout is read from the code and was not run, so it may behave differently.
- DL-6.1: "Free N Day Trial" and "Start free trial" are in the code but their switch is fixed off, so no piece covers them.
- DL-6.7: the "Turn off autocomplete" button only closes the popup in the code I read, with no setting change, so piece 4 says only that.
- DL-6.11: the first-day note "Data Sharing is paused for your first day of usage. Sharing will activate in N hours." and its "Share Now" link (`cursorAuth.skipPrivacyModeGracePeriod`) are switched off in code, so no piece covers them.
- DL-6.11: the header link to the web dashboard for unified dashboard accounts was not mapped, so no piece covers it.
- DL-6.14 and DL-6.15: the feature files disagree on whether the Command Palette entry for `cursorai.action.importvscode` asks anything first, so DL-6.14 only says the buttons run the command and DL-6.15 piece 6 repeats that the palette entry asks nothing.
- DL-6.2 and DL-6.3: the entry points from chat error buttons and the chat queue link are chat features, so I named them only as boundary entries.
- I did not run Cursor, so wording was spot-checked in the bundle for "Upgrade Your Plan", "Maybe Later", "Configure Usage-Based Pricing", "Enter usage-based spend limit", "Email, comma separated", "Switch Privacy Mode?" and "Confirm VS Code import", and the rest comes from the feature file.
