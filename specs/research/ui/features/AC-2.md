Group: AC-2 Ghost text and inline suggestions. Section: Autocomplete. Date: 2026-10-25. Pass: 2.
Features: 48.

Path note: `V\src\vs\editor\contrib\inlineCompletions\browser\` is the main stock folder and is written in full in each Source line. `C bundle` means the Cursor main bundle `workbench.desktop.main.js` and its NLS file `nls.messages.json`. Cursor 3.27.12 carries an older snapshot of the stock inline completion code, so several stock pieces are missing or shaped differently. Nothing was run on screen.

### AC-2.1 See ghost text at the cursor
- Tag: both
- Evidence: source
- Boundary: no
- What: Gray italic text appears right after the cursor as the proposed completion, drawn as injected text inside the line with the syntax colors dimmed to 70 percent opacity, and the cursor stays in front of it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css; V\src\vs\editor\contrib\inlineCompletions\browser\model\computeGhostText.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts; C bundle (decoration description `ghost-text-decoration`, classes `ghost-text-decoration`, `ghost-text-decoration-preview`, `syntax-highlighted`)

### AC-2.2 See multi-line ghost text under the line
- Tag: both
- Evidence: source
- Boundary: no
- What: When a suggestion has more than one line, the extra lines are drawn in a view zone under the cursor line in the editor font, the rest of the original line to the right of the insert point is hidden while it shows, and the suggest list is moved above the cursor when the ghost text has two or more lines.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts (AdditionalLinesWidget, class `suggest-preview-text`, decoration `ghost-text-hidden`); V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (forceRenderingAbove); C bundle (`suggest-preview-text`, `ghost-text-hidden`, `forceRenderingAbove`)

### AC-2.3 See the text a suggestion will replace or delete
- Tag: both
- Evidence: source
- Boundary: no
- What: For an inline edit shown side by side, as a line replacement, as a deletion, or as an inline insertion, the original text is marked in the editor with line and character highlights for removed text and inserted text. Plain ghost text can also underline the range it replaces (`inline-completion-text-to-replace`), but no code in V constructs that replace type (searched V\src for `new GhostTextReplacement`, no match), so the underline may never show in 1.128.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\originalEditorInlineDiffView.ts (classes `inlineCompletions-line-insert`, `-line-delete`, `-char-insert`, `-char-delete`, `-original-lines`); V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\model\ghostText.ts (GhostTextReplacement); C bundle (`inlineCompletions-char-insert`, `inlineCompletions-char-delete`, `inlineCompletions-line-insert`, `inlineCompletions-line-delete`, `inline-completion-text-to-replace`; `inlineCompletions-original-lines` not found)

### AC-2.4 See very short ghost text with a dotted underline
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Ghost text with fewer than 3 non-space characters, or an inserted part under 5 characters, gets a dotted underline in the ghost text color and skips the syntax-color treatment, so tiny suggestions stay visible.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts (class `short-text`, `highlightShortSuggestions: true` in inlineSuggestionsView.ts); V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts; C bundle (`short-text` searched in JS and CSS, 0 hits)

### AC-2.5 See a warning on a suggestion
- Tag: both
- Evidence: source
- Boundary: no
- What: When a suggestion carries a warning, the ghost text gets a warning squiggle with a double underline in the editor warning color, and the toolbar shows the warning message (plain text or markdown, up to 400 px wide) above its buttons; hovering the ghost text opens it. An orange warning icon widget at the end of the line also exists in code, but a constant (`USE_SQUIGGLES_FOR_WARNING`) switches it off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css; V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\inlineCompletionsHintsWidget.ts (class `warningMessage`); V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\hoverParticipant.ts (getWarningWidgetContext); C bundle (`warningMessage`, `ghost-text-view-warning-widget`, CSS `.ghost-text-decoration.warning`)

### AC-2.6 Style the ghost text
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose whether ghost text keeps syntax colors, pick a font family just for ghost text, and set its foreground, background, and border with the theme colors `editorGhostText.foreground`, `editorGhostText.background`, and `editorGhostText.border`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.syntaxHighlightingEnabled; editor.inlineSuggest.fontFamily
- Cursor: present
- Source: V\src\vs\editor\common\config\editorOptions.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts (font family style sheet); V\src\vs\editor\common\core\editorColorRegistry.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css; C bundle (both setting keys, the three `editorGhostText.*` color ids, NLS "Controls whether to show syntax highlighting for inline suggestions in the editor." and "Controls the font family of the inline suggestions.")

### AC-2.7 Preview the highlighted suggest list item as ghost text
- Tag: both
- Evidence: source
- Boundary: no
- What: With `editor.suggest.preview` on (default off), the item highlighted in the suggest list also shows in the editor as ghost text, with the part beyond the typed prefix in the preview style, and the suggest list moves above the cursor when the preview is multi-line. The owner of the suggest list itself is AC-1, the rendering is the ghost text view here.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.suggest.preview
- Cursor: present
- Source: V\src\vs\editor\common\config\editorOptions.ts (suggest.preview, previewMode default subwordSmart, no settings key for previewMode); V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts; V\src\vs\editor\contrib\inlineCompletions\browser\model\computeGhostText.ts; V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts; C bundle (`editor.suggest.preview`, `forceRenderingAbove`, NLS "Controls whether to preview the suggestion outcome in the editor.")

### AC-2.8 Use the inline suggestion toolbar
- Tag: both
- Evidence: source
- Boundary: no
- What: A small toolbar offers Accept and Accept Word as buttons, with Accept Line and an Always Show Toolbar toggle in its overflow menu, each showing its key label. With the default `onHover` it opens in the editor hover over ghost text, with `always` it is pinned above or below the ghost text line, with `never` it is off. Provider actions can join the overflow menu through `InlineCompletionsActions` (proposed API, extension fed, not cataloged here).
- Commands: editor.action.inlineSuggest.commit; editor.action.inlineSuggest.acceptNextWord; editor.action.inlineSuggest.acceptNextLine; editor.action.inlineSuggest.toggleAlwaysShowToolbar
- Menus: InlineSuggestionToolbar; InlineCompletionsActions (contributes.menus key `editor/inlineCompletions/actions`, marked for deprecation in code)
- Keys: none found
- Settings: editor.inlineSuggest.showToolbar
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\inlineCompletionsHintsWidget.ts (root class `inlineSuggestionsHints`); V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\inlineCompletionsHintsWidget.css; V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\hoverParticipant.ts; V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts; V\src\vs\platform\actions\common\actions.ts (MenuId); V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts; C bundle (class `inlineSuggestionsHints`, menu `InlineSuggestionToolbar`, labels "Accept", "Accept Word", "Accept Line", "Always Show Toolbar", setting `editor.inlineSuggest.showToolbar`)

### AC-2.9 Step through suggestions with n of m and the arrow keys
- Tag: both
- Evidence: source
- Boundary: no
- What: When more than one suggestion is available, the toolbar starts with a previous chevron, an `n/m` count, and a next chevron (buttons are disabled when there is only one), and Alt+] and Alt+[ move to the next and previous suggestion. The three toolbar items are added in code, not as menu entries.
- Commands: editor.action.inlineSuggest.showNext; editor.action.inlineSuggest.showPrevious
- Menus: none found
- Keys: Alt+] (show next, when editor writable and inlineSuggestionVisible); Alt+[ (show previous, same when)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\inlineCompletionsHintsWidget.ts (icons `inline-suggestion-hints-previous` and `-next`, class `availableSuggestionCount`); V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts; C bundle (ids `editor.action.inlineSuggest.showNext` and `.showPrevious` with Alt+] and Alt+[, icons `inline-suggestion-hints-next` and `-previous`, class `availableSuggestionCount`)

### AC-2.10 Accept the suggestion with Tab
- Tag: both
- Evidence: source
- Boundary: no
- What: Tab takes the whole suggestion, unless the suggest list is open, Tab is set to move focus, a hover has focus, text is selected, or the suggestion starts with more whitespace than one Tab would add (then Tab indents as usual). The same accept also runs from the Accept button in the toolbar and from the gutter menu, and it takes an inline edit when Tab is set to accept rather than jump. Cursor has its own Tab accept for Cursor Tab, handed off to AC-6.
- Commands: editor.action.inlineSuggest.commit
- Menus: InlineSuggestionToolbar; InlineEditsActions
- Keys: Tab (when inlineSuggestionVisible, not editorTabMovesFocus, not suggestWidgetVisible, not editorHoverFocused, no selection, inlineSuggestionHasIndentationLessThanTabSize; or inlineEditIsVisible with tabShouldAcceptInlineEdit and the same other guards); Tab also inside the inline edits preview editor
- Settings: none found
- Cursor: changed (the Tab rule has no selection check; Cursor Tab adds its own Tab accept)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (AcceptInlineCompletion); V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionContextKeys.ts; C bundle (id `editor.action.inlineSuggest.commit`, label "Accept Inline Suggestion", Tab rule at weight 200, menus `InlineSuggestionToolbar` and `InlineEditsActions`)

### AC-2.11 Accept the next word with Ctrl+Right
- Tag: both
- Evidence: source
- Boundary: no
- What: Ctrl+Right (Cmd+Right on macOS) takes only the next word of the ghost text when the cursor sits right in front of it and screen reader mode is off. The Accept Word button in the toolbar does the same. A suggestion with a snippet is accepted whole instead.
- Commands: editor.action.inlineSuggest.acceptNextWord
- Menus: InlineSuggestionToolbar
- Keys: Ctrl+Right (Cmd+Right on macOS) when editor writable, inlineSuggestionVisible, cursorBeforeGhostText, and accessibility mode off
- Settings: none found
- Cursor: changed (the key rule also fires for a Cursor Tab suggestion and first tries the Cursor Tab partial accept, with no cursorBeforeGhostText or screen reader guard)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (AcceptNextWordOfInlineCompletion); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (acceptNextWord, _acceptNext); C bundle (id `editor.action.inlineSuggest.acceptNextWord`, label "Accept Next Word Of Inline Suggestion", toolbar title "Accept Word")

### AC-2.12 Accept the next line
- Tag: both
- Evidence: source
- Boundary: no
- What: Takes the ghost text up to and including the next line break. It has no default key and is reached from the toolbar overflow menu (Accept Line) or the command palette.
- Commands: editor.action.inlineSuggest.acceptNextLine
- Menus: InlineSuggestionToolbar; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (AcceptNextLineOfInlineCompletion); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (acceptNextLine); C bundle (id `editor.action.inlineSuggest.acceptNextLine`, label "Accept Next Line Of Inline Suggestion", toolbar title "Accept Line", secondary group)

### AC-2.13 Reject a suggestion with Esc
- Tag: both
- Evidence: source
- Boundary: no
- What: Esc hides the ghost text or inline edit and keeps focus in the editor. Reject also shows in the gutter menu. The registered `InlineEditsActions` entries (Accept, Jump, Reject) have no code in V that draws them as a menu, so only the gutter menu shows Reject.
- Commands: editor.action.inlineSuggest.hide
- Menus: InlineEditsActions
- Keys: Escape (when editorTextFocus, same weight as closing the suggest list); Escape or Shift+Escape inside the inline edits preview editor
- Settings: none found
- Cursor: changed (the Esc rule has no editorTextFocus guard; Esc also rejects a Cursor Tab suggestion, handed off to AC-6)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (HideInlineCompletion); C bundle (id `editor.action.inlineSuggest.hide`, label "Hide Inline Suggestion", Esc at weight 190, menu `InlineEditsActions` title "Reject")

### AC-2.14 Trigger an inline suggestion by hand
- Tag: both
- Evidence: source
- Boundary: no
- What: The command asks for a suggestion at the cursor from the palette or a key you bind. Stock takes optional arguments (showNoResultNotification, providerId, explicit) and can show the notice "No inline suggestion is available." when nothing comes back; it also plays the suggestion sound cue.
- Commands: editor.action.inlineSuggest.trigger
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (command and label "Trigger Inline Suggestion" present; the bundle shows no argument schema and no no-result notice)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (TriggerInlineSuggestionAction); C bundle (id `editor.action.inlineSuggest.trigger`; NLS "No inline suggestion is available." searched, 0 hits)

### AC-2.15 Trigger a next edit suggestion by hand
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The palette command "Trigger Next Edit Suggestion" asks only for an inline edit (not ghost text) and shows the notice "No inline edit is available." when none comes back. A second id, `editor.action.inlineSuggest.triggerInlineEdit`, has no label. Neither id is in V 1.128.0 or N, so these look like older stock code left in the bundle; origin not verified.
- Commands: editor.action.inlineSuggest.triggerInlineEditExplicit; editor.action.inlineSuggest.triggerInlineEdit
- Menus: CommandPalette (only the labeled `...Explicit` command)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (ids `editor.action.inlineSuggest.triggerInlineEditExplicit` and `.triggerInlineEdit`, label "Trigger Next Edit Suggestion", notice "No inline edit is available.")

### AC-2.16 Accept the next part of an inline edit
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: The palette command "Accept Next Inline Edit Part" takes the first piece of a multi-part inline edit. It has no default key. Not in V 1.128.0 or N, so it looks like older stock code left in the bundle; origin not verified.
- Commands: editor.action.inlineSuggest.acceptNextInlineEditPart
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (id `editor.action.inlineSuggest.acceptNextInlineEditPart`, label "Accept Next Inline Edit Part", precondition editor writable and inlineEditIsVisible)

### AC-2.17 See the inline edit gutter indicator
- Tag: both
- Evidence: source
- Boundary: no
- What: A small pill in the left gutter marks an inline edit (or a ghost text that offers the edit menu). Beside the edit it shows a Tab icon when Tab will accept, a Tab-above or Tab-below icon when Tab will jump up or down to the edit, and a check when hovered. When the edit is outside the viewport the pill slides into view and shows an arrow that points up or down. It carries the line number, accepts or jumps on click, and pulses a few times for new users and when you hover the edit.
- Commands: editor.action.inlineSuggest.commit; editor.action.inlineSuggest.jump
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (older gutter code: click accepts when the pill is docked, otherwise jumps; the new-user storage key `inlineEditsGutterIndicatorUserKind` is present)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\components\gutterIndicatorView.ts (class `inline-edits-view-gutter-indicator`, icons keyboard-tab, arrow-right, check, keyboard-tab-above, keyboard-tab-below); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsNewUsers.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts; C bundle (class `inline-edits-view-gutter-indicator`, storage key `inlineEditsGutterIndicatorUserKind`, `triggerAnimation`)

### AC-2.18 Use the gutter indicator menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Hovering the pill opens a small menu with a header (the provider name), Go To / Accept (check icon), Reject (close icon), an optional alternative action, Show Collapsed or Show Expanded, Snooze (bell-slash icon), and Settings (gear icon, opens Settings filtered to the tag `@tag:nextEditSuggestions`). Provider commands and a footer action bar follow. Each row shows its key label.
- Commands: editor.action.inlineSuggest.commit; editor.action.inlineSuggest.hide; editor.action.inlineSuggest.commitAlternativeAction; editor.action.inlineSuggest.toggleShowCollapsed; editor.action.inlineSuggest.snooze; workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.showCollapsed
- Cursor: changed (no Snooze row and no alternative action row; the Go To / Accept row switches between accept and jump with the Tab action; Reject, Show Expanded, Show Collapsed, and Settings are present)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\components\gutterIndicatorMenu.ts; C bundle (row ids `gotoAndAccept`, `reject`, `showExpanded`, `showCollapsed`, `settings`; NLS "Go To", "Accept", "Reject", "Show Expanded", "Show Collapsed", "Settings"; `inline-edit-gutter-indicator-menu-separator`)

### AC-2.19 Jump to the next inline edit with Tab
- Tag: both
- Evidence: source
- Boundary: no
- What: When an inline edit is somewhere else in the file, Tab moves the cursor to it first (the gutter pill shows a Tab-above or Tab-below icon) and a second Tab accepts it. The registered menu entry is titled Jump and is hidden when the cursor is already at the edit, but nothing in V draws that menu (see Unclear).
- Commands: editor.action.inlineSuggest.jump
- Menus: InlineEditsActions
- Keys: Tab (when inlineEditIsVisible, not editorTabMovesFocus, not suggestWidgetVisible, not editorHoverFocused, tabShouldJumpToInlineEdit)
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (JumpToNextInlineEdit); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (jump); C bundle (id `editor.action.inlineSuggest.jump`, label "Jump to next inline edit", Tab at weight 201)

### AC-2.20 Accept the alternative action (Rename) with Shift+Tab
- Tag: vscode
- Evidence: source
- Boundary: no
- What: An inline edit can carry a second action, drawn as a small labeled segment next to the replacement (for example Rename, with a count of occurrences when you hold Shift: "Rename 3 occurrences"). Shift+Tab runs it, and the gutter menu lists it as a row.
- Commands: editor.action.inlineSuggest.commitAlternativeAction; editor.action.inlineSuggest.renameSymbol
- Menus: none found
- Keys: Shift+Tab (when inlineSuggestionAlternativeActionVisible and inlineEditIsVisible); Shift+Tab also inside the inline edits preview editor
- Settings: none found
- Cursor: missing
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (AcceptInlineCompletionAlternativeAction); V\src\vs\editor\contrib\inlineCompletions\browser\controller\commandIds.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsWordReplacementView.ts (class `inline-edit-alternative-action-label`); V\src\vs\editor\contrib\inlineCompletions\browser\model\renameSymbolProcessor.ts (label "Rename"); C bundle (ids, class, and label "Accept Inline Suggestion Alternative Action" searched, 0 hits)

### AC-2.21 See a larger inline edit side by side
- Tag: both
- Evidence: source
- Boundary: no
- What: A change that spans lines shows the new code in a bordered, clickable preview box next to the original lines when the old and new text both fit across the editor width, and the border color follows what Tab will do (accept, jump, or nothing). Clicking the box accepts the edit once it has been on screen for 350 ms. Set `renderSideBySide` to `never` to always use the line replacement view (AC-2.22) instead.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.renderSideBySide
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsSideBySideView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (_determineView); C bundle (`sideBySide` view kind, class `originalCornerCutoutSideBySide`, setting `editor.inlineSuggest.edits.renderSideBySide`, NLS "Larger suggestions will show side by side...")

### AC-2.22 See an inline edit as a line replacement
- Tag: both
- Evidence: source
- Boundary: no
- What: A one-line change, a larger change when side by side is off or does not fit, and a one-line insertion before the cursor all show as a bordered box under the original lines with the new lines inside and the old lines highlighted. Clicking accepts it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.renderSideBySide
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsLineReplacementView.ts (class `line-replacement`); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (_determineView); C bundle (`lineReplacement` view kind, class `modifiedLinesLineReplacement`)

### AC-2.23 See an inline edit as a word replacement
- Tag: both
- Evidence: source
- Boundary: no
- What: A small change inside one line (each side under 100 characters, no tabs) shows as a small box just under the changed word with the new text, while the old word stays highlighted in the line. Clicking accepts it. When the edit has an alternative action it adds the second segment described in AC-2.20.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (word replacement view present; no alternative action segment)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsWordReplacementView.ts (class `word-replacement`, MAX_LENGTH 100); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts; C bundle (class `word-replacement`, view kind `wordReplacements`)

### AC-2.24 See an inline edit inserted in place
- Tag: both
- Evidence: source
- Boundary: no
- What: An edit that only adds text is drawn where it will land. A single-line insertion after the cursor shows inline and shifts the code to make room. A multi-line insertion shows as a bordered block that pushes the lines below down. `allowCodeShifting` set to `always` (default) allows both, `horizontal` allows only the inline shift, and `never` uses a line replacement or side by side view instead.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.allowCodeShifting
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsInsertionView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\originalEditorInlineDiffView.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (_determineView); C bundle (view kinds `insertionInline` and `insertionMultiLine`, class `originalOverlayInsertion`, setting `editor.inlineSuggest.edits.allowCodeShifting`)

### AC-2.25 See an inline edit as a deletion
- Tag: both
- Evidence: source
- Boundary: no
- What: An edit that only removes text outlines the text to delete in place with a bordered overlay in the original lines; no new text appears.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsDeletionView.ts (classes `originalOverlayDeletion`, `originalSeparatorDeletion`); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts; C bundle (class `originalOverlayDeletion`, view kind `deletion`)

### AC-2.26 See an inline edit collapsed
- Tag: both
- Evidence: source
- Boundary: no
- What: With Show Collapsed on, the full preview is replaced by a small downward-pointing triangle at the start of the edit until you jump to it. Hovering the gutter pill shows the full preview again. The gutter menu row and the command flip between Show Collapsed and Show Expanded, and the triangle bobs once for users who have not yet become regular users.
- Commands: editor.action.inlineSuggest.toggleShowCollapsed
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.showCollapsed
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsCollapsedView.ts (classes `inline-edits-collapsed-view`, `collapsedView`, `collapsedViewPath`); V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (ToggleInlineCompletionShowCollapsed); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts; C bundle (id `editor.action.inlineSuggest.toggleShowCollapsed`, label "Toggle Inline Suggestions Show Collapsed", class `inline-edits-collapsed-view`, setting `editor.inlineSuggest.edits.showCollapsed`)

### AC-2.27 See a "to jump" label at the jump target
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When a suggestion is only a jump to a position in the file (no text to insert), a small bordered label with the Tab key chip and the words "to jump" is drawn at the target position (class `inline-edit-jump-to-widget`). A blinking-cursor style of the same widget exists in code.
- Commands: editor.action.inlineSuggest.commit
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\jumpToView.ts (decoration `inline-edit-jump-to-decoration`); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (view kind `jumpTo`); C bundle (`inline-edit-jump-to-widget`, `inline-edit-jump-to-decoration`, `"jumpTo"` searched, 0 hits)

### AC-2.28 See a hint label for an edit shown elsewhere
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When a suggestion supplies a hint (text plus a range in the current file) instead of a diff, a small bordered text label is drawn at the end of that line, or just below or above it when the end of the line has no room. This view is chosen when the edit targets another document, or has a display location and has not been jumped to yet. The border color follows the Tab action, the text can be syntax-colored when the hint style is Code, and clicking accepts.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\inlineEditsCustomView.ts (class `inline-edits-custom-view`); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (view kind `custom`); V\src\vs\editor\common\languages.ts (InlineCompletionHintStyle Code and Label); C bundle (`inline-edits-custom-view` searched, 0 hits)

### AC-2.29 See the long distance hint
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When the next edit is outside the visible part of the file, or in another file, a small preview box of the target code appears near the cursor line, in free space on the right or over the code if there is none. Its bottom bar shows the symbol outline at the target (or the target file icon and name) and the label "Tab to jump" ("Go to suggestion" if Tab is not the key), "Tab to open" or "Go to file" for another file, plus an arrow showing the direction. Clicking it jumps. It exists in 1.128.0, so it is not newer; the bundle has none of it.
- Commands: editor.action.inlineSuggest.jump
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.showLongDistanceHint
- Cursor: missing
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\longDistanceHint\inlineEditsLongDistanceHint.ts (class `inline-edits-long-distance-hint-widget`); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\longDistanceHint\longDistancePreviewEditor.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\longDistanceHint\longDistnaceWidgetPlacement.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsView.ts (_getLongDistanceHintState); V\src\vs\editor\common\config\editorOptions.ts (experimental, tag nextEditSuggestions, default true); C bundle (`long-distance`, `longDistance`, `showLongDistanceHint`, `inline-edits-long-distance-hint-widget` searched, 0 hits)

### AC-2.30 Choose how many context lines the long distance hint shows
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. A setting from 0 to 10 (default 0, experimental) for how many lines above and below the target line the long distance preview shows; 0 shows only the target line.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.edits.longDistanceHintContextLineCount
- Cursor: missing
- Source: N\src\vs\editor\common\config\editorOptions.ts; N\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\inlineEditsViews\longDistanceHint\longDistancePreviewEditor.ts; C bundle (key searched, 0 hits)

### AC-2.31 See an accepted inline edit fade out
- Tag: both
- Evidence: source
- Boundary: no
- What: After you accept an inline edit, the changed ranges flash the modified-text background and fade out over about one second. It is skipped when motion is reduced.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.reduceMotion
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (FadeoutDecoration); V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.css (class `edits-fadeout-decoration`); V\src\vs\platform\accessibility\browser\accessibilityService.ts; C bundle (class `edits-fadeout-decoration`, 1000 ms animation)

### AC-2.32 Recolor the inline edit UI with theme colors
- Tag: both
- Evidence: source
- Boundary: no
- What: Theme colors change how inline edits look: `inlineEdit.originalBackground`, `.modifiedBackground`, `.originalChangedLineBackground`, `.originalChangedTextBackground`, `.modifiedChangedLineBackground`, `.modifiedChangedTextBackground`, `.originalBorder`, `.modifiedBorder`, `.tabWillAcceptModifiedBorder`, `.tabWillAcceptOriginalBorder`, and the gutter pill colors `inlineEdit.gutterIndicator.*` in plain, primary, secondary, and successful variants (foreground, border, background).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\theme.ts; C bundle (ids `inlineEdit.gutterIndicator.primaryBackground`, `.successfulBackground`, `.secondaryBackground`, `inlineEdit.tabWillAcceptModifiedBorder`, `inlineEdit.tabWillAcceptOriginalBorder`, `inlineEdit.modifiedBorder`, `inlineEdit.originalBorder`, `inlineEdit.originalBackground`)

### AC-2.33 Turn inline suggestions on or off
- Tag: both
- Evidence: source
- Boundary: no
- What: One setting (default on) controls whether inline suggestions show automatically as you type.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.enabled; editor.inlineSuggest; editor.inlineSuggest.experimental.suppressInlineSuggestions
- Cursor: present
- Source: V\src\vs\editor\common\config\editorOptions.ts; C bundle (key `editor.inlineSuggest.enabled`, NLS "Controls whether to automatically show inline suggestions in the editor.")

### AC-2.34 Hold suggestions back after typing or in snippet mode
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Two settings delay or silence suggestions: `minShowDelay` waits 0 to 10000 ms (default 0) after typing before a suggestion shows, and `suppressInSnippetMode` (default on) hides inline suggestions while a snippet's tab stops are active.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.minShowDelay; editor.inlineSuggest.suppressInSnippetMode
- Cursor: missing
- Source: V\src\vs\editor\common\config\editorOptions.ts; C bundle (`minShowDelay` and `suppressInSnippetMode` searched, 0 hits; NLS "Controls the minimal delay" and "Controls whether inline suggestions are suppressed when in snippet mode" 0 hits)

### AC-2.35 Keep the suggest list quiet while ghost text shows
- Tag: both
- Evidence: source
- Boundary: no
- What: With `suppressSuggestions` on (default off), the suggest list does not open on its own while an inline suggestion is available. The context key `inlineSuggestionSuppressSuggestions` carries the flag a suggestion sets for itself.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.suppressSuggestions; editor.inlineSuggest.experimental.showOnSuggestConflict
- Cursor: present
- Source: V\src\vs\editor\common\config\editorOptions.ts; V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionContextKeys.ts; V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts; C bundle (key `editor.inlineSuggest.suppressSuggestions`, context key `inlineSuggestionSuppressSuggestions`, NLS "Controls how inline suggestions interact with the suggest widget...")

### AC-2.36 Snooze inline suggestions
- Tag: vscode
- Evidence: source
- Boundary: no
- What: "Snooze Inline Suggestions" opens a quick pick titled "Select snooze duration for Inline Suggestions" with 1, 5, 10, 15, 30 and 60 minutes, "Custom..." (asks "Enter snooze duration in minutes", placeholder "Duration in minutes (e.g. 90)"), and a "{n} minutes (Last used)" row when you used a custom length before. The command also accepts a minutes number as its first argument. "Cancel Snooze Inline Suggestions" ends it and only shows in the palette while snoozed (context key `inlineCompletions.snoozed`). The gutter menu has a Snooze row (AC-2.18). Cursor's own snooze for Cursor Tab is a different command, handed off to AC-8.
- Commands: editor.action.inlineSuggest.snooze; editor.action.inlineSuggest.cancelSnooze
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\editor\browser\services\inlineCompletionsService.ts; V\src\vs\editor\contrib\inlineCompletions\browser\inlineCompletions.contribution.ts; V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineEdits\components\gutterIndicatorMenu.ts; C bundle (both ids, `inlineCompletions.snoozed`, `inlineCompletions.lastSnoozeDuration`, and the labels "Snooze Inline Suggestions", "Cancel Snooze Inline Suggestions", "Select snooze duration for Inline Suggestions" searched in the bundle and NLS, 0 hits; `editor.cpp.snooze` found, owned by AC-8)

### AC-2.37 See inline suggestion status in the language status popup
- Tag: both
- Evidence: source
- Boundary: no
- What: The language status popup (the language indicator on the status bar) gets an "Inline Suggestions" row for the focused file with the detail "Inline suggestions" and one of four states: "Loading..." (busy), "Inline completion available" (short icon lightbulb), "Inline edit available" (lightbulb-sparkle), or "No inline suggestion available" (circle-slash), updated after a 300 ms debounce. Stock hides the row when AI features are turned off. This is the workbench contribution the status bar uses for inline completions.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: chat.disableAIFeatures
- Cursor: changed (an editor contribution `vs.editor.contrib.inlineCompletionLanguageStatusBarContribution` instead of a workbench contribution; no chat entitlement check; shows only while the editor has focus)
- Source: V\src\vs\workbench\contrib\inlineCompletions\browser\inlineCompletions.contribution.ts; V\src\vs\workbench\contrib\inlineCompletions\browser\inlineCompletionLanguageStatusBarContribution.ts (id `vs.contrib.inlineCompletionLanguageStatusBarContribution`, status id `inlineSuggestions`); V\src\vs\workbench\services\chat\common\chatEntitlementService.ts (hidden state follows `chat.disableAIFeatures`); C bundle (status id `inlineSuggestions`, id `vs.editor.contrib.inlineCompletionLanguageStatusBarContribution`, NLS "Loading...", "Inline completion available", "Inline edit available", "No inline suggestion available", "Inline suggestions"; `IChatEntitlementService` and `chat.disableAIFeatures` searched, 0 hits)

### AC-2.38 Inspect a suggestion in the accessible view
- Tag: both
- Evidence: source
- Boundary: no
- What: With a suggestion showing, Alt+F2 opens the accessible view with the line plus ghost text, or a readable patch for an inline edit. Show Next and Show Previous inside the view step through suggestions, Ctrl+/ accepts it (button "Accept Inline Completion" in the view toolbar), and closing the view hides the suggestion. A screen reader hint "Inspect this in the accessible view ({0})" points to it.
- Commands: editor.action.accessibleView; editor.action.accessibleViewNext; editor.action.accessibleViewPrevious; editor.action.accessibleViewAcceptInlineCompletion
- Menus: AccessibleView; CommandPalette
- Keys: Alt+F2 to open (Linux: Alt+Shift+F2, with Alt+F2 as secondary); Ctrl+/ to accept while the view shows an inline completion (Control+/ on macOS)
- Settings: accessibility.verbosity.inlineCompletions
- Cursor: present
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\inlineCompletionsAccessibleView.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts; V\src\vs\workbench\contrib\accessibility\common\accessibilityCommands.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts; C bundle (ids `editor.action.accessibleViewAcceptInlineCompletion`, `editor.action.accessibleViewNext`, `editor.action.accessibleViewPrevious`, setting `accessibility.verbosity.inlineCompletions`, NLS "Accept Inline Completion", "Inspect this in the accessible view ({0})")

### AC-2.39 Hear and read suggestions with a screen reader or audio cue
- Tag: both
- Evidence: source
- Boundary: no
- What: When a suggestion appears, a screen reader reads it aloud (on by default), with the accessible view hint added when the verbosity setting allows. If the announce setting is off, a screen reader user gets the hover text "Suggestion:" plus the code, and the hover reads "There are inline completions here". A sound cue can play when a line has an inline suggestion, and a second cue exists for next edit suggestions.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.screenReaderAnnounceInlineSuggestion; editor.inlineCompletionsAccessibilityVerbose; accessibility.verbosity.inlineCompletions; accessibility.signals.lineHasInlineSuggestion; accessibility.signals.nextEditSuggestion
- Cursor: changed (the nextEditSuggestion signal key is not in the bundle; the announce option, the verbosity option, the line cue key, and the hover texts are)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (_provideScreenReaderUpdate, playAccessibilitySignal); V\src\vs\editor\contrib\inlineCompletions\browser\hintsWidget\hoverParticipant.ts (renderScreenReaderText, getAccessibleContent); V\src\vs\platform\accessibilitySignal\browser\accessibilitySignalService.ts; V\src\vs\editor\common\config\editorOptions.ts; C bundle (`screenReaderAnnounceInlineSuggestion`, `inlineCompletionsAccessibilityVerbose`, `accessibility.signals.lineHasInlineSuggestion`, NLS "Suggestion:", "There are inline completions here"; `accessibility.signals.nextEditSuggestion` searched, 0 hits)

### AC-2.40 Copy a repro of the current suggestion
- Tag: both
- Evidence: source
- Boundary: no
- What: The palette command "Developer: Extract Inline Suggest State" copies the file text plus the current suggestion as commented JSON to the clipboard so a bug can be reproduced.
- Commands: editor.action.inlineSuggest.dev.extractRepro
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (precondition is only inlineEditIsVisible, stock also allows inlineSuggestionVisible)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\commands.ts (DevExtractReproSample); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (extractReproSample); C bundle (id `editor.action.inlineSuggest.dev.extractRepro`, label "Developer: Extract Inline Suggest State")

### AC-2.41 See quick suggestions as ghost text
- Tag: both
- Evidence: source
- Boundary: no
- What: Set `editor.quickSuggestions` to `inline` (for all code, or only for other code, comments or strings) and, at the end of a word or right after a trigger character, the best matching completion shows as ghost text and the suggest list does not open. The provider hands over every matching item, with the one picked last time first and the first six resolved early, so the n/m count and the next and previous keys of AC-2.9 should step through them (not run). This provider gives nothing while the suggest list is open, so a list opened by hand with Ctrl+Space replaces the ghost text. The setting itself is owned by AC-1.3, the ghost text that results is here.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.quickSuggestions
- Cursor: present (the provider class is in the bundle with `_getTriggerCharacterInfo`, a `canBeReused` check and the `valueFor(...)!=="inline"` test; NLS "Quick suggestions show as ghost text" found; the bundle has the values on, inline and off but not offWhenInlineCompletions, see AC-1.3)
- Source: V\src\vs\editor\contrib\suggest\browser\suggestInlineCompletions.ts (SuggestInlineCompletions, registered through V\src\vs\editor\editor.all.ts); V\src\vs\editor\contrib\suggest\browser\suggestModel.ts (quick suggest gate, value `inline` returns early); V\src\vs\editor\contrib\suggest\browser\suggest.ts (QuickSuggestionsOptions); V\src\vs\editor\common\config\editorOptions.ts (EditorQuickSuggestions, enum descriptions "Quick suggestions show as ghost text"); C bundle (provider class around `_getTriggerCharacterInfo`; NLS "Quick suggestions show as ghost text")

### AC-2.42 Use ghost text with several cursors
- Tag: both
- Evidence: source
- Boundary: no
- What: With more than one cursor, ghost text is drawn at every cursor where the text the suggestion would replace is the same as at the main cursor (a plain insertion matches at every cursor), not only at the main one. Tab, Accept Word and Accept Line then type the same text at each of those cursors and leave one cursor at the end of each. Inline edits (AC-2.21 to AC-2.29) are for the main cursor only and keep a single cursor after accept.
- Commands: editor.action.inlineSuggest.commit; editor.action.inlineSuggest.acceptNextWord; editor.action.inlineSuggest.acceptNextLine
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the bundle carries one ghost text per cursor in the model state and a secondary edit helper, and the view loops over those ghost texts; the accept code sets one selection per edit; not run)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (getSecondaryEdits, ghostTexts, accept, _acceptNext); V\src\vs\editor\contrib\inlineCompletions\browser\view\inlineSuggestionsView.ts (_ghostTexts); V\src\vs\editor\contrib\inlineCompletions\test\browser\getSecondaryEdits.test.ts; C bundle (`ghostTexts`, the secondary edit helper next to `inlineCompletionPartialAccept`, view field `_ghostTexts`)

### AC-2.43 Pick the list item that matches the ghost text
- Tag: both
- Evidence: source
- Boundary: no
- What: When the suggest list opens while ghost text is showing, the list starts with the item the ghost text builds on already highlighted: the longest list item whose text is the start of the inline suggestion. If no item fits, the list picks its usual first item. The suggest list itself is AC-1.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (a selector registered on the suggest controller with priority 100 is in the bundle)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts (registerSelector priority 100, singleTextEditAugments); V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (preselector passed to ObservableSuggestWidgetAdapter); V\src\vs\editor\contrib\inlineCompletions\browser\model\singleTextEditHelpers.ts; C bundle (`registerSelector({priority:100,select:...` with the same preselector)

### AC-2.44 See ghost text continue past the highlighted list item
- Tag: both
- Evidence: source
- Boundary: no
- What: When an inline suggestion starts with the same text as the highlighted list item and covers at least the same range, the editor shows the whole inline suggestion as ghost text, and the part beyond the list item is drawn in the preview style. This works even with `editor.suggest.preview` off (AC-2.7); the preview is then used only when no inline suggestion extends the item. Holding Shift while the list is open flips the insert or replace range used for this preview, the same way Shift+Tab does for accepting. The experimental `showOnSuggestConflict` setting can turn this off, it is provider facing and not cataloged.
- Commands: none found
- Menus: none found
- Keys: Shift (held while the suggest list is open, flips insert and replace for the preview)
- Settings: editor.suggest.preview
- Cursor: present (`_computeAugmentation`, the preview-enabled check that returns early only when there is no augmentation, and the Shift key handlers are in the bundle)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (_computeAugmentation, state derivation with fullEditPreviewLength); V\src\vs\editor\contrib\inlineCompletions\browser\model\computeGhostText.ts (previewSuffixLength); V\src\vs\editor\contrib\inlineCompletions\browser\model\suggestWidgetAdapter.ts (isShiftKeyPressed, getOverwriteInfo); V\src\vs\editor\contrib\inlineCompletions\browser\view\ghostText\ghostTextView.ts (class `ghost-text-decoration-preview`); C bundle (`_computeAugmentation`, `isShiftKeyPressed`)

### AC-2.45 Use inline suggestions in the commit message and comment boxes
- Tag: both
- Evidence: source
- Boundary: no
- What: The commit message box in Source Control and the reply box of a review comment thread both host ghost text, with the same Tab, Esc, Accept Word keys and the same toolbar as the main editor. In the commit box stock lets Esc reject a showing suggestion first and only empties the message when no suggestion is showing. Notebook cell editors and the chat input also host the same controller, those are not part of this feature (chat is out of scope).
- Commands: editor.action.inlineSuggest.hide; scm.clearInput
- Menus: none found
- Keys: Escape (scm.clearInput, when scmRepository, suggest list hidden, no inline suggestion visible, no validation message, no non-empty selection)
- Settings: none found
- Cursor: changed (the controller is in both lists, the 18 entry list of the commit box and the 10 entry list of the comment reply box; the bundle's `scm.clearInput` rule has no inline suggestion guard and no validation message guard, and its weight 200 is above the weight 190 of Esc for Hide Inline Suggestion, so Esc in the commit box should empty the message even while ghost text shows; read from the rule weights, not run)
- Source: V\src\vs\workbench\contrib\scm\browser\scmInput.ts (contribution list with InlineCompletionsController.ID); V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts (scm.clearInput, weight WorkbenchContrib); V\src\vs\workbench\contrib\comments\browser\simpleCommentEditor.ts (contribution list with InlineCompletionsController.ID); V\src\vs\workbench\contrib\notebook\browser\notebookEditorWidget.ts (clearActiveCellWidgets); C bundle (`scm.clearInput` rule with weight 200; the two `getSomeEditorContributions` lists that hold the controller class `J1`)

### AC-2.46 See a suggestion step aside
- Tag: both
- Evidence: source
- Boundary: no
- What: Ghost text goes away when you move the cursor yourself (a click or an arrow key, not typing), but an inline edit stays. Any suggestion goes away when the editor loses focus, unless focus is in the gutter indicator menu, the accessible view is open, the toolbar's overflow menu is open, or it is an inline edit you asked for with the trigger command. Focusing another editor ends the suggestion in the first one. Inline edits also hide while a peek view (peek definition or references) is open, and a read-only editor never shows suggestions. The hidden setting `editor.inlineSuggest.keepOnBlur` keeps them (listed in Unclear, not a user setting).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the cursor move rule that stops only ghost text, the blur rule with `accessibleViewIsShown`, `keepOnBlur` and the toolbar `dropDownVisible` checks, the peek counter `openedPeekWidgets` and `_hasVisiblePeekWidgets`, and the read-only guard on the model are all in the bundle)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts (model with isReadonly guard, selections handler, focus handler, _instances); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (_hasVisiblePeekWidgets); V\src\vs\editor\browser\observableCodeEditor.ts (openedPeekWidgets); V\src\vs\editor\contrib\peekView\browser\peekView.ts (counter); C bundle (same checks in the controller and model classes)

### AC-2.47 Type along with a suggestion
- Tag: both
- Evidence: source
- Boundary: no
- What: While you type the same characters as the ghost text, it stays and gets shorter. As soon as you type something that is not the next part of it, it goes away. Leading indentation is ignored in the comparison, and the text to the right of the cursor must still fit the rest of the suggestion. A suggestion that starts on a different line than the cursor is not shown.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the same check is in the bundle: indentation trimmed, then a `startsWith` test and a substring match; module fragment found)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionIsVisible.ts; V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineSuggestionItem.ts (isVisible); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (visibility filter); C bundle (visibility check next to `canBeReused` and `reuse` in the inline completion item class)

### AC-2.48 Accept a suggestion that is a snippet
- Tag: both
- Evidence: source
- Boundary: no
- What: A suggestion that carries a snippet (tab stops and placeholders) shows as plain ghost text. Tab or the Accept button inserts it as a snippet, so the cursor lands on the first tab stop and snippet mode (AC-4) starts. Accept Word and Accept Line do not take a part of it, they accept the whole snippet. While snippet mode is on, other suggestions stay hidden if `suppressInSnippetMode` is on (AC-2.34).
- Commands: editor.action.inlineSuggest.commit; editor.action.inlineSuggest.acceptNextWord; editor.action.inlineSuggest.acceptNextLine
- Menus: none found
- Keys: none found
- Settings: editor.inlineSuggest.suppressInSnippetMode
- Cursor: present (the snippet branch that inserts through the snippet controller, and the accept-whole check in the next word and next line code, are in the bundle; the setting `suppressInSnippetMode` is missing there, see AC-2.34)
- Source: V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts (accept with snippetInfo, _acceptNext); V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineSuggestionItem.ts (snippetInfo); V\src\vs\editor\contrib\snippet\browser\snippetController2.ts (insert); C bundle (`i.snippetInfo` branch calling the snippet controller `insert`, `s.snippetInfo||s.filterText!==s.insertText` then accept)

## Handed off
- Cursor Tab ghost text and its other shapes (class `cursor-ghost-text`, line class `cpp-inline-suggestion`, view zone, preview box, red and green coloring, diff peek): AC-6.
- Cursor Tab accept and reject (`editor.action.acceptCursorTabSuggestion` on Tab, Esc rejecting a Cursor Tab suggestion, the "Tab" key chips): AC-6.
- Cursor Tab partial accept (setting `cursor.cpp.enablePartialAccepts` and the `cppSuggestion` rule that Cursor added to `editor.action.inlineSuggest.acceptNextWord`): AC-6. The stock command itself is AC-2.11.
- Cursor hiding the stock ghost text while a Cursor Tab suggestion shows, and restoring it afterward: AC-6.
- Cursor Tab "tab to jump here", off-screen and cross-file indicators, go-back indicator, import hint: AC-7.
- Cursor Tab status bar item, Enable, Disable, Toggle, Snooze and Unsnooze commands, the `cursor.cpp.*` settings and the Cursor Settings page: AC-8 (status bar item cross-listed in SS-6).
- Terminal inline suggestion setting `terminal.integrated.suggest.inlineSuggestion`: AC-5.
- Emmet abbreviation ghost text setting `emmet.useInlineCompletions`: AC-1 (pass 1 filed the Emmet settings there).
- The language status popup itself (the container for AC-2.37): SS-3. The Inline Suggestions row stays here as AC-2.37, and SS-6 also lists the stock inline suggestions entry.
- Ghost completion inside the Browser address field (classes `url-input-ghost*`): the Browser feature group, not in the Autocomplete section.
- The gray placeholder hint drawn in an empty small editor (`PlaceholderTextContribution`, theme color `editor.placeholder.foreground`, which defaults to the ghost text foreground; the commit message box and the comment reply box both host it): SB-4 for the commit box (confidence low, SB-4 already mentions the commit box placeholder, no group owns the color).
- The IntelliSense walkthrough step text with a "Trigger Inline Suggestion" button and the two accessibility setting names (`gettingStartedContent.ts`): ED-9 (confidence low, it is walkthrough content, not an inline suggestion widget).

## Unclear
- Nothing was run on screen. For Cursor the answer is "present in the bundle", not "reachable". Whether any stock inline completion piece actually renders in Cursor was not tested, and what feeds them was not examined (out of scope).
- Correction to pass 1: the "Suggestion:" text with the code is a screen reader only row. For other users the hover content is the toolbar itself (AC-2.8).
- Correction to pass 1: Cursor does have "Go To / Accept" in the gutter menu. The bundle builds it from two strings ("Go To" and "Accept") joined with a slash, so a search for the whole phrase missed it. Also, some labels are written inline in the JS as `De(id,"text")` (for example "Accept Next Inline Edit Part"), while others are numbered lookups into the NLS file, so searching both finds more.
- The long distance hint exists in V 1.128.0 (so it is not "newer"); it is missing in Cursor. Only its context-lines setting is newer (AC-2.30, N only).
- `MenuId.InlineEditsActions` carries Accept, Jump and Reject entries, but nothing in V, N or the Cursor bundle builds a menu from it. The visible equivalent is the gutter menu (AC-2.18).
- `GhostTextReplacement` (underline of replaced text) is read by the ghost text view in V, but no code in V constructs it (searched V\src for `new GhostTextReplacement`), so that underline may be dead in 1.128.0. The Cursor bundle has the same view code.
- `InlineEditsWordInsertView` (class `word-insert`) exists in V but `inlineEditsView.ts` never creates it. The warning icon widget in the ghost text view is switched off by a constant. Both are recorded here only so a builder does not chase them.
- Ids `editor.action.inlineSuggest.triggerInlineEdit`, `.triggerInlineEditExplicit` and `.acceptNextInlineEditPart` are in the Cursor bundle but not in V 1.128.0 or N. They look like older stock code. Origin not verified.
- Hidden or debug settings were left out: `editor.inlineSuggest.keepOnBlur` (read at run time, not in V's settings schema, registered in the Cursor bundle), `editor.inlineSuggest.edits.resetNewUserExperience` (V only), `logFetch`, `logChangeReason.commandId`, the provider-facing `experimental.*` keys, and `triggerCommandOnProviderChange`.
- Whether the settings tag `nextEditSuggestions` (used by the gutter menu Settings row) marks any setting that Cursor shows was not checked.
- The `showOnSuggestConflict` experimental setting changes whether ghost text shows next to the suggest list. It is provider facing and left out; AC-2.35 covers the visible setting.
- Default keys on macOS (for example Cmd+Right for Accept Word, Control+/ in the accessible view) were read from code, not from the live keybindings editor.
- Overflow check on 2026-10-25 added 8 features (AC-2.41 to AC-2.48).

