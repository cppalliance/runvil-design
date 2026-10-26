Group: QA-6 Quick pick and input box widget. Section: Quick access. Date: 2026-10-25. Pass: 2.
Features: 26.

### QA-6.1 Pick one row from a list
- Tag: both
- Evidence: source
- Boundary: no
- What: Any picker opens the floating box with a filter field, a placeholder, and a list. Typing narrows the rows with highlighted matching letters (label by default, description and detail only when the picker asks for it, and rows marked "always show" never hide), the first row is active on open, and Enter or a click picks the active row and closes the box. Pickers are built through `pick` and `createQuickPick`, so there is no command of its own.
- Commands: none found
- Menus: none found
- Keys: none found (movement and accept keys are in QA-6.21 and QA-6.22)
- Settings: none found
- Cursor: present (controller class `kAe` builds the same parts with the same class names `.quick-input-widget`, `.quick-input-header`, `.quick-input-filter`, `.quick-input-list`; `createQuickPick` found)
- Source: V\src\vs\platform\quickinput\common\quickInput.ts (20-265, 509-735); V\src\vs\platform\quickinput\browser\quickInput.ts (524-1180); V\src\vs\platform\quickinput\browser\quickInputController.ts (134-451, 463-572); V\src\vs\platform\quickinput\browser\quickInputList.ts (700-800); C bundle (kAe, createQuickPick, quick-input-widget class names)

### QA-6.2 Pick several rows with checkboxes
- Tag: both
- Evidence: source
- Boundary: no
- What: When a picker allows many choices every row gets a checkbox. Space toggles the active row's box when focus is in the list (Space types a space when focus is in the filter field), Enter or OK confirms all checked rows, rows can start checked, and a disabled row shows a greyed box that cannot be changed. Rows marked not pickable show no box.
- Commands: workbench.action.quickPickManyToggle; quickInput.toggleCheckbox
- Menus: none found (the quickPickManyToggle command has no title, so it is not in the palette)
- Keys: Space (quickInput.toggleCheckbox, when the box shows a quick pick or tree and the filter field is not focused); workbench.action.quickPickManyToggle has no default key
- Settings: none found
- Cursor: changed (workbench.action.quickPickManyToggle and the checkbox classes are present; `quickInput.toggleCheckbox` is not found as a command id, so Space handling there is not a registered key rule)
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts (270-290); V\src\vs\workbench\browser\actions\quickAccessActions.ts (99-108); V\src\vs\platform\quickinput\browser\quickInputList.ts (434-465, 1494-1506); V\src\vs\platform\quickinput\common\quickInput.ts (60-100); C bundle (workbench.action.quickPickManyToggle found, quickInput.toggleCheckbox searched with zero hits)

### QA-6.3 Check all, selected count, OK and Custom buttons
- Tag: both
- Evidence: source
- Boundary: no
- What: Multi-select pickers show a check-all box at the left of the header (tooltip "Toggle all checkboxes") that checks every visible row, a "{0} Selected" count badge at the right of the filter field, and an OK button (label can be changed) that confirms. A picker can also show a Custom button with its own label and hover text. Pickers can hide the check-all box or the badge, and can force OK on a single-select list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (check-all box, "{0} Selected", "OK", "Custom", okLabel, hideCheckAll and hideCountBadge found; the option that draws the Custom button in a secondary style, `customButtonSecondary`, has zero hits)
- Source: V\src\vs\platform\quickinput\browser\quickInputController.ts (172-226, 760-800); V\src\vs\platform\quickinput\browser\quickInput.ts (558-567, 789-874, 1036-1050); V\src\vs\platform\quickinput\browser\quickInputList.ts (1094-1110); C bundle (nls "Toggle all checkboxes" 2248, "{0} Selected" 2250, "OK" 2251, "Custom" 2252)

### QA-6.4 Type a value in an input box
- Tag: both
- Evidence: source
- Boundary: no
- What: An input prompt shows a title, a text field with placeholder and optional preset text (with a chosen part of it selected), and a line under the field that reads "{prompt} (Press 'Enter' to confirm or 'Escape' to cancel)" or, with no prompt, "Press 'Enter' to confirm your input or 'Escape' to cancel". Enter confirms and Escape cancels. The box has no list.
- Commands: none found
- Menus: none found
- Keys: Enter; Escape (see QA-6.22)
- Settings: none found
- Cursor: present (createInputBox, valueSelection, prompt message strings found)
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (154-170, 1182-1325); V\src\vs\platform\quickinput\browser\quickInputController.ts (593-651, 662-664); C bundle (createInputBox, nls "Press 'Enter' to confirm your input or 'Escape' to cancel", "{0} (Press 'Enter' to confirm or 'Escape' to cancel)")

### QA-6.5 Type a hidden password
- Tag: both
- Evidence: source
- Boundary: no
- What: An input prompt can switch the field to password mode, where typed characters show as dots. The rest of the prompt (placeholder, prompt line, validation) works the same.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the input box sets `password` from the option)
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (1189, 1234-1245, 1306-1308); V\src\vs\platform\quickinput\browser\quickInputController.ts (647); C bundle (input box `password` assignment in the input function)

### QA-6.6 See a validation message under the input box
- Tag: both
- Evidence: source
- Boundary: no
- What: A prompt can check the text and show a message under the field with an info, warning or error look (colored text and background, a border, and a matching border on the field). The check runs about 100 ms after you stop typing and again when you press Enter. An error blocks Enter, while warning and info do not. The message text can hold clickable links (including command links) and inline icons written as `$(name)`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (validateInput, validationMessage found 33 and 46 times)
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (318-333, 451-512); V\src\vs\platform\quickinput\browser\quickInputController.ts (575-651); V\src\vs\platform\quickinput\browser\quickInputUtils.ts (141-180); V\src\vs\platform\quickinput\browser\quickInputBox.ts (165-175); C bundle (validateInput, validationMessage)

### QA-6.7 See a title, description and step counter
- Tag: both
- Evidence: source
- Boundary: no
- What: A title bar appears above the field when the picker has a title, a step number, or title buttons. It shows the title text with "(2/3)" after it for a multi-step flow (just the number when the total is unknown), and an optional description line under the title area. With no title, step or title button the bar is hidden.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (totalSteps found 9 times, nls "{0}/{1}" found, `.quick-input-titlebar` and `.quick-input-title` classes found)
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (153-232, 388-401, 470-496, 1036-1050); V\src\vs\platform\quickinput\browser\quickInputController.ts (154-175); C bundle (nls "{0}/{1}", totalSteps, quick-input-titlebar)

### QA-6.8 Go back a step in a multi-step flow
- Tag: both
- Evidence: source
- Boundary: no
- What: Multi-step pickers show a Back arrow at the left of the title bar (tooltip "Back", with the bound key added in brackets when there is one). Clicking it or pressing the key returns to the previous step.
- Commands: workbench.action.quickInputBack
- Menus: none found
- Keys: Alt+Left (Windows); Control+- (Mac); Ctrl+Alt+- (Linux); all when inQuickOpen
- Settings: none found
- Cursor: present (id found twice; the key rule with the same three platform keys found; nls "Back" and "Back ({0})" found)
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (110-122); V\src\vs\platform\quickinput\browser\quickInput.ts (91-96, 270-316); V\src\vs\platform\quickinput\browser\quickInputController.ts (725-735, 875-885); V\src\vs\workbench\services\quickinput\browser\quickInputService.ts (43); C bundle (workbench.action.quickInputBack rule, quickInputBack icon, nls "Back")

### QA-6.9 Use buttons in the title bar and around the filter field
- Tag: both
- Evidence: source
- Boundary: no
- What: A picker can add icon buttons in three places. In the title bar (right side, optionally toggles, or tucked into an overflow menu with a label when marked secondary), beside the filter field on its right ("inline"), and inside the field at its far end as on and off toggles (the selected-count badge moves left to make room). Examples are the Reset and Close buttons of Customize Layout, "Open in Search View" in Quick Search, and the zero-based toggle in Go to Offset (stock only).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (location values Title, Inline and Input found, `.quick-input-left-action-bar`, `.quick-input-right-action-bar` and `.quick-input-inline-action-bar` found; the overflow for secondary buttons was not traced)
- Source: V\src\vs\platform\quickinput\common\quickInput.ts (800-870); V\src\vs\platform\quickinput\browser\quickInput.ts (270-316, 418-447); V\src\vs\platform\quickinput\browser\quickInputController.ts (154-212); V\src\vs\workbench\browser\actions\layoutActions.ts (1517-1600); V\src\vs\workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts (111); C bundle (button location enum, action bar class names)

### QA-6.10 Use buttons on a row
- Tag: both
- Evidence: source
- Boundary: no
- What: A row can show one or more icon buttons at its right end. By default they appear when you hover the row or move focus to it, and a button marked "always visible" stays on screen (the unsaved dot in the editor lists). Clicking one fires that row's action without picking the row, and the picker can then keep the list as is, drop the row, refresh the list, or close the box. Examples are "Close Editor", "Rename Terminal", "Configure Task" and "Remove from Recently Opened". Buttons can be toggles, or sit in an overflow menu when marked secondary.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (`alwaysVisible` found 16 times, onDidTriggerItemButton found, row action bar class found)
- Source: V\src\vs\platform\quickinput\common\quickInput.ts (821-880); V\src\vs\platform\quickinput\browser\quickInputList.ts (324-420, 548-565); V\src\vs\platform\quickinput\browser\quickInputUtils.ts (60-140); V\src\vs\platform\quickinput\browser\pickerQuickAccess.ts; V\src\vs\platform\quickinput\browser\media\quickInput.css (349-360); C bundle (alwaysVisible, onDidTriggerItemButton)

### QA-6.11 Group rows under separator headings
- Tag: both
- Evidence: source
- Boundary: no
- What: Pickers that use separators show heading rows with a label, an optional description and an optional tooltip, and a thin line marks where each group starts. A heading with buttons becomes its own row that can be focused and shows its buttons (for example a group-level action), while a plain heading is drawn above its first row. Headings whose rows are all filtered out disappear.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (separator elements and `onDidTriggerSeparatorButton` found 8 times, the class `quick-input-list-separator-as-item` found 3 times)
- Source: V\src\vs\platform\quickinput\common\quickInput.ts (103-114, 884-895); V\src\vs\platform\quickinput\browser\quickInputList.ts (256-272, 592-700, 765-780, 1316-1370); C bundle (separator renderer, onDidTriggerSeparatorButton)

### QA-6.12 See section header rows and custom drawn rows
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor's list adds a third row kind besides items and separators: a section header that cannot be clicked, drawn dimmed at 25 px tall (5 px when it is only a thin divider, with an optional line above), and rows or headers that supply their own DOM and are measured off screen. Cursor also spaces list rows 3 px apart. No picker in the main bundle was found that creates a section header or a custom drawn row, so what the person sees depends on pickers outside this section (the Go to File code only passes them through).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class `dPt` with `sectionHeader`, renderer `y8m` with `.quick-input-list-section-header`, delegate `MCk` heights 25 and 5, `customRenderedDom`, `isPureDivider`, `shouldShowDividerAbove`, tree option `itemGap:3`; workbench.desktop.main.css rules for `.quick-input-list-section-header`)

### QA-6.13 Read a row: icon, label, description, detail and shortcut
- Tag: both
- Evidence: source
- Boundary: no
- What: A row can show an icon (a file type icon, a theme icon written `$(name)` in the label, an image, or an icon class), the label with highlighted matches, a dimmer description to its right, a second line of detail text (rows with detail are taller), and a shortcut label at the far right for the command's key. Rows can be italic (preview editors) or struck through, and a long description or detail gets its own small hover with the full text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (all the row parts and the `.quick-input-list-entry-keybinding` class found; the list tree has an extra 3 px gap between rows and can measure custom drawn rows, see QA-6.12)
- Source: V\src\vs\platform\quickinput\common\quickInput.ts (30-100); V\src\vs\platform\quickinput\browser\quickInputList.ts (86-200, 324-420, 418-565, 274-290); C bundle (quick-input-list-entry-keybinding, list tree options)

### QA-6.14 Hover a row to read its tooltip
- Tag: both
- Evidence: source
- Boundary: no
- What: Pausing the pointer over a row, or pressing Ctrl+Space while a row is active, shows a hover to the right of the row with the row's tooltip. Rows with no tooltip show no row hover. Pressing Ctrl+Space again hides it, and while it is on it follows the active row.
- Commands: quickInput.toggleHover
- Menus: none found
- Keys: Ctrl+Space (Control+Space on Mac; when the box shows a quick pick or tree)
- Settings: none found
- Cursor: present (quickInput.toggleHover rule found, same key)
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts (292-305); V\src\vs\platform\quickinput\browser\quickInputList.ts (975-1000, 1513-1535, 1595-1615); C bundle (quickInput.toggleHover with Ctrl or Control plus Space)

### QA-6.15 See a progress bar while a list is loading
- Tag: both
- Evidence: source
- Boundary: no
- What: While a picker is busy (for example waiting on a search) a thin progress bar runs under the filter field. It only appears after the picker has been busy for 800 ms, and it is hidden from screen readers.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (`.quick-input-progress` class found; the 800 ms delay was not traced)
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (402-417); V\src\vs\platform\quickinput\browser\quickInputController.ts (227-229, 716); C bundle (quick-input-progress)

### QA-6.16 Keep the box open
- Tag: both
- Evidence: source
- Boundary: no
- What: Two behaviors keep a picker on screen. Right arrow (or Alt, Ctrl or Cmd with Right arrow) picks the active row without closing the box, when focus is in the list or the caret is at the end of the filter field, for pickers that allow it such as the editor, terminal, debug console and Go to File lists. And clicking elsewhere closes the box unless the picker asks to stay open or the setting is turned off.
- Commands: quickInput.acceptInBackground
- Menus: none found
- Keys: Right arrow (quickInput.acceptInBackground, when the box shows a quick pick and either the filter field is not focused or its caret is at the end); the same with Alt, Ctrl and Cmd held
- Settings: workbench.quickOpen.closeOnFocusLost
- Cursor: present (quickInput.acceptInBackground rule found with the same key and weight; workbench.quickOpen.closeOnFocusLost found)
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts (230-250); V\src\vs\workbench\services\quickinput\browser\quickInputService.ts (42); V\src\vs\platform\quickinput\browser\quickInputController.ts (320-340); V\src\vs\platform\quickinput\browser\quickInput.ts (673-680, 1165-1180); V\src\vs\workbench\browser\workbench.contribution.ts (536-540); C bundle (quickInput.acceptInBackground, canAcceptInBackground, workbench.quickOpen.closeOnFocusLost)

### QA-6.17 Put the box at the top or the center
- Tag: both
- Evidence: source
- Boundary: no
- What: Two commands snap the box to the top of the window under the title bar (the default) or to the vertical center, and Customize Layout shows them under a "Quick Input Position" heading with a mark on the current one (the box is also reset to the top by that dialog's Reset button). Neither command is in the palette.
- Commands: workbench.action.alignQuickInputTop; workbench.action.alignQuickInputCenter
- Menus: none found (both are f1 false and are reached from the Customize Layout list)
- Keys: none found
- Settings: none found
- Cursor: present (both ids, nls "Align Quick Input Top", "Align Quick Input Center" and "Quick Input Position" found, context key `quickInputAlignment` found)
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (55-56, 1280-1310, 1385-1388, 1503-1509, 1604); V\src\vs\platform\quickinput\browser\quickInput.ts (43-44); V\src\vs\platform\quickinput\browser\quickInputController.ts (667-672, 1129-1148); C bundle (alignQuickInputTop 3 hits, alignQuickInputCenter 2 hits)

### QA-6.18 Drag the box and have it remember its place
- Tag: both
- Evidence: source
- Boundary: no
- What: Dragging the title bar, or the empty part of the header, moves the box anywhere inside the window. It snaps to the top, the vertical center and the horizontal center within 20 px, stays clear of the window controls when the title bar is drawn by the app, and the place is saved between sessions. Double-clicking the title bar or header puts it back at the top. The saved place is a fraction of the window, so it adapts to resizing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found (the place is stored in application storage under workbench.quickInput.viewState, not in settings)
- Cursor: present (drag controller with snap threshold 20, snap ratio 0.25 and the same storage key found; its double-click reset does not skip the title bar buttons and does not set the top mark, so the code is a little older than stock)
- Source: V\src\vs\platform\quickinput\browser\quickInputController.ts (45-50, 359-405, 1041-1050, 1055-1275); C bundle (workbench.quickInput.viewState, _snapThreshold=20, _snapLineHorizontalRatio=.25, registerMouseListeners)

### QA-6.19 Set how wide the box is
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Stock draws the box at 62 percent of the window width up to a fixed 600 px. Cursor adds the setting `workbench.quickOpen.width` (default 820, allowed 580 to 1000) that raises that cap for Quick Open and every other picker, and the box re-lays out as soon as the value changes. The width is still limited to 62 percent of the window, and a value you have not set falls back to a built-in default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.quickOpen.width
- Cursor: cursor only
- Source: C bundle (setting `workbench.quickOpen.width` with default 820, minimum 580, maximum 1e3 and message 4209; controller `kAe` getConfiguredWidth, clampConfiguredWidth and updateLayout using Math.min(width*.62, configured); onDidChangeConfiguration re-layout); V\src\vs\platform\quickinput\browser\quickInputController.ts (53, 895-899) for the stock fixed cap

### QA-6.20 See a preview box beside the active row (Cursor)
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the box shows Cursor Tab request details (model name, time and request data) from the developer picker "Developer: Report Cursor Tab Action". Only the widget side is cataloged.
- What: When the active row, or the hovered row for rows that supply their own preview, holds report data, a floating box 420 px wide and 320 px tall with a rounded border and shadow opens to the right of the list showing a heading line with the time and then the details recorded for the row, opens under the row as wide as the row if it would run off the right edge of the window, and closes when the active row changes to one without data or when the picker closes.
- Commands: workbench.action.reportCppAction
- Menus: CommandPalette (Developer: Report Cursor Tab Action, f1)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`quickInputService2.renderPreviewBox`, `showPreview` and `hidePreview` on the controller, `quick-input-preview-box-content`, component `xfw`, constants 420 and 320, row field `cppReportEvent` set by the Report Cursor Tab Action picker `gPk`, row field `customPreview` read but never set in the main bundle)

### QA-6.21 Move through the list with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: Up and Down move one row, PageUp and PageDown move a page, Ctrl+Home and Ctrl+End go to the first and last row, and Alt+Up and Alt+Down jump to the previous or next heading (Ctrl+Alt+Up and Ctrl+Alt+Down always do). The same keys work with Ctrl or Alt added so a held shortcut such as Ctrl+P can keep stepping, and in a list opened by a held key Alt+Up and Alt+Down only step one row.
- Commands: quickInput.next; quickInput.previous; quickInput.pageNext; quickInput.pagePrevious; quickInput.first; quickInput.last; quickInput.nextSeparatorWithQuickAccessFallback; quickInput.previousSeparatorWithQuickAccessFallback; quickInput.nextSeparator; quickInput.previousSeparator
- Menus: none found
- Keys: Down and Up (also Ctrl+Down and Ctrl+Up); PageDown and PageUp; Ctrl+Home and Ctrl+End (Control on Mac); Alt+Down and Alt+Up for the headings with held-key fallback (Cmd+Down and Cmd+Up on Mac); Ctrl+Alt+Down and Ctrl+Alt+Up for the headings (Cmd+Alt+Down and Cmd+Alt+Up on Mac); all when the box shows a quick pick or tree
- Settings: none found
- Cursor: present (all ten ids found, including the Mac and non-Mac separator variants)
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts (14-205); C bundle (the ten quickInput.* ids, focus helper `Mle`)

### QA-6.22 Accept, cancel and focus the box by key
- Tag: both
- Evidence: source
- Boundary: no
- What: Enter picks the active row and Escape cancels. Holding Alt, Ctrl, Cmd or Shift with Enter passes that modifier to the picker (for example to open a file to the side). Escape and Shift+Escape also close any quick access list. Three more commands with no default key let you bind your own keys to accept, accept with the Ctrl modifier, and move focus into the box.
- Commands: quickInput.accept; quickInput.hide; workbench.action.closeQuickOpen; workbench.action.acceptSelectedQuickOpenItem; workbench.action.alternativeAcceptSelectedQuickOpenItem; workbench.action.focusQuickOpen
- Menus: none found
- Keys: Enter (and Alt, Ctrl, Cmd or Shift with Enter; Enter is ignored while an input method is composing); Escape (and Alt, Ctrl or Cmd with Escape); Shift+Escape (workbench.action.closeQuickOpen, when inQuickOpen)
- Settings: none found
- Cursor: changed (`quickInput.accept` and `quickInput.hide` are not found as ids, and the box handles Enter, Escape and Tab in its own key handler: Enter accepts, Escape hides with the gesture reason, Tab cycles focus through the row buttons, filter field, list, message links and custom widget; the four workbench.action.* commands are found)
- Source: V\src\vs\platform\quickinput\browser\quickInputActions.ts (209-265); V\src\vs\workbench\browser\actions\quickAccessActions.ts (30-72); C bundle (the controller key handler with `case ve.Enter`, `case ve.Escape`, `case ve.Tab`; closeQuickOpen, acceptSelectedQuickOpenItem, alternativeAcceptSelectedQuickOpenItem, focusQuickOpen found; quickInput.accept and quickInput.hide searched with no match)

### QA-6.23 Use the box with a screen reader
- Tag: both
- Evidence: source
- Boundary: no
- What: The list is exposed as a listbox named "Quick Input" whose rows are options (checkboxes in multi-select), each row reads its label, description and detail (plus its heading), and the filter field names the picker from its placeholder and title, or "Type to narrow down results." The result count ("{0} Results") and the selected count ("{0} Selected") are announced politely as they change, the active row is tied to the field with aria-activedescendant, and the progress bar is hidden from the reader. In screen reader mode the list refreshes the focused row after each filter so it is read again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (listbox and option roles, the live count regions, aria-activedescendant and screen reader mode checks found; stock's list focus mode on the filter field (`setListFocusMode`, aria-haspopup=listbox) has no match in the bundle)
- Source: V\src\vs\platform\quickinput\browser\quickInputList.ts (86-110, 292-320, 1150-1195); V\src\vs\platform\quickinput\browser\quickInputController.ts (188-205, 236-256, 340-358); V\src\vs\platform\quickinput\browser\quickInputBox.ts (120-160); V\src\vs\platform\quickinput\browser\quickInput.ts (526, 1063-1085); C bundle (aria-live on `.quick-input-visible-count`, messages 2249 and 2250, aria-activedescendant, isScreenReaderOptimized)

### QA-6.24 Pin rows to the top of a list
- Tag: both
- Evidence: source
- Boundary: no
- What: Rows in the Run Task list and the terminal Run Recent Command list show a pin button on hover (tooltip "Pin command"; "Pinned command" once pinned). Pinned rows move into a "pinned" section at the top while the filter field is empty, and pressing the button again unpins. The choice is saved per workspace.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the pin function, the "pinned" heading, "Pin command" and "Pinned command" messages found in use)
- Source: V\src\vs\platform\quickinput\browser\quickPickPin.ts (1-125); V\src\vs\workbench\contrib\tasks\browser\taskQuickPick.ts (311-320); V\src\vs\workbench\contrib\terminalContrib\history\browser\terminalRunRecentQuickPick.ts (374); C bundle (function `iag` with separator label message 2257, messages 2258 and 2259)

### QA-6.25 Open and close the box with a short animation
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. In VS Code 1.144.0 the box fades in while growing from 97 percent size over 250 ms and fades out over 150 ms (it ignores the pointer while closing), but only when the window uses the modern UI style and motion is enabled. The 1.128.0 source and the Cursor bundle have no such animation.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: N\src\vs\platform\quickinput\browser\quickInputController.ts (close animation, `quick-input-widget-closing`, `QUICK_INPUT_CLOSE_ANIMATION_DURATION`); N\src\vs\platform\quickinput\browser\media\quickInput.css (16-45, keyframes open and close); C bundle (quick-input-widget-closing, quick-input-widget-open, modern-ui and monaco-enable-motion searched in the bundle and its CSS with zero hits)

### QA-6.26 Pick a recent AI action to report (developer picker)
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the rows are recent AI generation records and the follow-up step sends feedback about them. Only the pickers are cataloged.
- What: Run "Developer: Report AI Action" (the registered original text is "See Recent AI Actions") to open a quick pick "Select a recent AI action" with up to 30 recent actions, each row showing its type, a shortened description and its id, with the time as the description. Picking a row opens a second pick "Choose an action (ai)" with "Report Good", "Report Bad", "Report with Comment" (opens an input box "Type your comment here..."), "Copy Request ID" (greyed with "Disabled in privacy mode" when privacy mode is on) and, for developer users only, "Open in Prompt Quality". These rows do not set the preview field, so no preview box was found for this picker.
- Commands: workbench.action.reportAIAction
- Menus: CommandPalette (Developer: Report AI Action, f1)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class with `ID="workbench.action.reportAIAction"` and `LABEL="Developer: Report AI Action"`, picker function `mPk`, second step `oGm`, feedback function `aGm`)

## Handed off

- Open in Search View and See More Files buttons of the Quick Search list, which are the inline buttons this widget draws (QA-6.9): QA-2.
- Go to Offset zero-based toggle button, an input-location toggle (vscode only): QA-3.3 and QA-3.4.
- Customize Layout list itself (the list that holds the Quick Input Position rows, with its Reset and Close buttons): TB-4. QA-6.17 only names the two position commands.
- Browser address bar edit overlay, the only stock picker that anchors the box over another element (`anchor` with `anchorPosition` overlay; Cursor's box has no anchor support): ED-10.
- Run Recent Command list itself (it uses the pin feature in QA-6.24 and an input-location toggle): PN-3.12.
- Command Palette rows, the Configure Keybinding button on each row and Remove from Recently Used: QA-1. Those are row buttons drawn by QA-6.10.
- Quick Tree and Quick Widget modes (chat code, skipped by the parent): none.

## Unclear

- The catalog says the preview box is used by two developer pickers. Only "Developer: Report Cursor Tab Action" was found to set the row field `cppReportEvent` (the only creator found in the main bundle). The "Developer: Report AI Action" picker builds plain rows with no preview field, and `customPreview`, the other trigger, is read in the box code and never set anywhere in the main bundle, the automations bundle or the built-in extensions. If a second picker uses the box, it is in the Agents window bundle, which was not read.
- Pass 1 described the stock width as a fixed 600 px; the stock code uses 62 percent of the window up to 600, and Cursor keeps the 62 percent rule on top of its setting. Pass 1 also said the validation message has an icon, and that separator keys were Ctrl+Up and Ctrl+Down. Source shows a colored message with a border and no icon unless the text holds `$(name)`, and the heading keys are Alt+Up and Alt+Down (Cmd on Mac) with Ctrl+Alt+Up and Ctrl+Alt+Down always jumping. Ctrl+Up and Ctrl+Down are an extra pair for single-row moves.
- Cursor's box is an older copy of stock's in places: no `quickInput.accept`, `quickInput.hide` or `quickInput.toggleCheckbox` ids, no `customButtonSecondary`, no anchor support, no `setListFocusMode`, and an explicit Tab trap. Whether any of these changes what a person sees (beyond Tab order in the box) was not checked.
- Overlap with QA-2: QA-2.8 and QA-2.19 describe Right arrow and closeOnFocusLost for the Go to File list. QA-6.16 describes them for the widget as a whole. Same for QA-3.9 (jump without closing). The same ids appear in both places on purpose.
- QA-5 hands the four palette commands with no default key to QA-6: they are in QA-6.22 (accept, alternative accept, focus) and QA-6.2 (quickPickManyToggle), and the Back key is QA-6.8. QA-5.15 also lists closeQuickOpen, which is repeated in QA-6.22.
- A row's `keybinding` shortcut label, `tooltip` markdown with links, and `ariaLabel` are set by each picker, so what a user sees for them depends on the picker. They are described once here, in QA-6.13, QA-6.14 and QA-6.23.
- Cursor ships the `createQuickWidget` mode (custom HTML above the list, class `.quick-input-html-widget`) but not `createQuickTree`. Both are used only by chat code in stock, so neither is cataloged. Cursor's `customRenderedDom` rows in QA-6.12 are a different mechanism.
- The color tokens the box reads (`quickInput.background`, `quickInput.foreground`, `quickInputTitle.background`, `pickerGroup.foreground`, `pickerGroup.border` and the `quickInputList.focus*` set) belong to the theme section and are not features here.
- Newer than 1.128.0 and not cataloged as features: a per-row icon color (`iconColor` with a `data-quick-input-id` attribute on rows), a height setter on the filter field, the `anchorPosition` value `below` with `withQuickInputAnchor`, a custom widget slot that sits above the list (`has-custom-widget`), and a dropped `isComposing` guard on the accept key rule. All are plumbing or style hooks with no new command, setting or menu entry. Only the animation (QA-6.25) is a visible behavior.
- Behavior on screen was not watched. Every statement comes from reading source or bundle strings.
