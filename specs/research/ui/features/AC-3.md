Group: AC-3 Parameter hints. Section: Autocomplete. Date: 2026-10-25. Pass: 2.
Features: 13.

### AC-3.1 Open parameter hints by hand
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Shift+Space (Cmd+Shift+Space on macOS) with the cursor inside a call to open the signature popup at once, or run Trigger Parameter Hints from the Command Palette; nothing happens when no signature help provider covers the file.
- Commands: editor.action.triggerParameterHints
- Menus: none found
- Keys: Ctrl+Shift+Space (Cmd+Shift+Space on macOS) when editorTextFocus; the command needs editorHasSignatureHelpProvider
- Settings: none found
- Cursor: present (id, label "Trigger Parameter Hints", the key with the same modifiers and the hasSignatureHelpProvider precondition found)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHints.ts (TriggerParameterHintsAction lines 75-96, manual trigger with zero delay lines 70-72); V\src\vs\editor\common\editorContextKeys.ts (line 87); V\src\vs\editor\browser\widget\codeEditor\codeEditorWidget.ts (lines 2335 and 2416); V\src\vs\editor\contrib\quickAccess\browser\commandsQuickAccess.ts (editor actions listed in the palette, lines 42-55); C bundle (id editor.action.triggerParameterHints, NLS message 1453)

### AC-3.2 Open hints by typing a trigger character
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing a character that the language lists as a signature trigger (for TypeScript and JavaScript that is `(`, `,` or `<`) opens the popup about 120 ms later, as long as automatic hints are on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.parameterHints.enabled
- Cursor: present (the model code reads signatureHelpTriggerCharacters and signatureHelpRetriggerCharacters from each provider; the same settings key found)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (DEFAULT_DELAY line 52, trigger characters collected in onModelChanged lines 250-278, onDidType lines 280-294); V\extensions\typescript-language-features\src\languageFeatures\signatureHelp.ts (trigger characters lines 16-17, registration lines 131-135); C bundle (signatureHelpTriggerCharacters in the model code)

### AC-3.3 Hints follow your typing and cursor
- Tag: both
- Evidence: source
- Boundary: no
- What: While the popup is open, typing, deleting or moving the cursor with the keys refreshes it so the bold parameter follows the comma you are in, and a retrigger character (TypeScript lists `)`) refreshes it too; a refresh keeps the overload you were on when the language reports the same signature again.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (retrigger characters, the isRetrigger flag and activeSignatureHelp pass-through found in the bundle model code)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (isTriggered lines 244-248, onCursorChange lines 296-302, onModelContentChange lines 304-308, doTrigger lines 176-233, mergeTriggerContexts lines 324-340); V\extensions\typescript-language-features\src\languageFeatures\signatureHelp.ts (getActiveSignature keeps the previous label, lines 52-63; retrigger reason lines 103-122); C bundle (isRetrigger in the provide call)

### AC-3.4 Signature with the active parameter highlighted
- Tag: both
- Evidence: source
- Boundary: no
- What: The popup shows the whole signature on one line in the editor font with the parameter you are on in bold and the list highlight color (theme color editorHoverWidget.highlightForeground); a signature with no parameters shows its plain label, and the active parameter is the signature's own, falling back to the overall one.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (class "parameter active", the label offset search, hasDocs and the color id found; CSS rule for the bold highlighted span found)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsWidget.ts (render lines 183-211, renderParameters lines 297-311, getParameterLabelOffsets lines 313-329, color line 367); V\src\vs\editor\contrib\parameterHints\browser\parameterHints.css (.code lines 74-76, .parameter.active lines 149-152); C bundle (CSS .parameter-hints-widget .signature .parameter.active)

### AC-3.5 Overload counter and arrows
- Tag: both
- Evidence: source
- Boundary: no
- What: When the language returns more than one signature the popup gains a narrow strip on its left with an up chevron, a zero padded counter such as 2/3 (or 02/12 with twelve), a down chevron and a faint divider; clicking a chevron shows the previous or next signature and puts focus back in the editor.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (controls, overloads and button classes, both icon ids parameter-hints-previous and parameter-hints-next, the padStart counter code and the multiple CSS rules found)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsWidget.ts (icons lines 32-33, DOM and click handlers lines 69-88, multiple flag lines 190-193, counter lines 239-241, next and previous lines 331-339); V\src\vs\editor\contrib\parameterHints\browser\parameterHints.css (.multiple lines 31-43, .controls, .button and .overloads lines 118-147); C bundle (CSS .multiple and .multiple .body:before)

### AC-3.6 Switch overloads with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: With the popup open and more than one signature, Up or Alt+Up shows the previous signature and Down or Alt+Down the next (on macOS also Ctrl+P and Ctrl+N), and the popup takes those keys from the editor only while it is open.
- Commands: showPrevParameterHint; showNextParameterHint
- Menus: none found
- Keys: Up; Alt+Up (mac also Ctrl+P) for the previous signature and Down; Alt+Down (mac also Ctrl+N) for the next, all when editorFocus and parameterHintsVisible and parameterHintsMultipleSignatures
- Settings: none found
- Cursor: present (both ids, the same primary and secondary keys, and the two context keys found)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHints.ts (weight line 101, commands lines 117-141); V\src\vs\editor\contrib\parameterHints\browser\provideSignatureHelp.ts (context keys lines 19-22); C bundle (ids showPrevParameterHint and showNextParameterHint with weight 175)

### AC-3.7 Wrap around or stop at the last overload
- Tag: both
- Evidence: source
- Boundary: no
- What: With wrap around on, Down on the last signature jumps to the first and Up on the first jumps to the last; with it off, going past either end closes the popup instead.
- Commands: showPrevParameterHint; showNextParameterHint
- Menus: none found
- Keys: none found
- Settings: editor.parameterHints.cycle
- Cursor: present (settings key editor.parameterHints.cycle with default true found)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (next lines 129-146, previous lines 148-165); V\src\vs\editor\common\config\editorOptions.ts (EditorParameterHints lines 3647-3705, cycle at lines 3685-3689); C bundle (editor.parameterHints.cycle with default e.cycle and cycle:!0)

### AC-3.8 Documentation under the signature
- Tag: both
- Evidence: source
- Boundary: no
- What: Below the signature the popup shows the documentation of the active parameter first and then the documentation of the whole signature, as plain text or rendered markdown (code spans, code blocks, links), set off from the signature by a faint rule; with no documentation the area is hidden, and the text can be selected with the mouse.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (docs element toggled with the empty class, has-docs on the signature, markdown-docs class and the hasDocs test found; CSS rules for .docs found)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsWidget.ts (docs rendering lines 213-238, hasDocs lines 281-295, markdown renderer lines 270-279, userSelect line 99); V\src\vs\editor\contrib\parameterHints\browser\parameterHints.css (.has-docs rule lines 63-72, .docs rules lines 78-117); C bundle (render code with has-docs and empty toggles; CSS .signature.has-docs:after)

### AC-3.9 Popup position, size and font
- Tag: both
- Evidence: source
- Boundary: no
- What: The popup sits above the cursor line (below when there is no room), moves with the cursor, is at most 440 px wide and at most a quarter of the editor height (never less than 250 px) tall with its own scrollbar for long documentation, and uses the editor's font size with the signature in the editor font.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (position, max height, 440 px width and font code all found; the frame rule found in the CSS declares border and z-index only, where V 1.128 also declares a corner radius and a shadow from theme tokens, the same older-snapshot pattern seen on the suggest widget; a shadow set by another rule was not ruled out)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsWidget.ts (allowEditorOverflow lines 54-55, scrollbar lines 90-97, font lines 111-140, getPosition lines 173-181, updateMaxHeight lines 352-364); V\src\vs\editor\contrib\parameterHints\browser\parameterHints.css (frame lines 6-18, width lines 25-29); C bundle (CSS .parameter-hints-widget frame rule and .phwrapper max-width 440px; updateMaxHeight code)

### AC-3.10 Close the hints
- Tag: both
- Evidence: source
- Boundary: no
- What: Esc or Shift+Esc closes the popup while the editor has focus; it also closes when the editor loses focus, on a mouse click in the editor text, when the language returns no signature, and at the end of the list when wrap around is off.
- Commands: closeParameterHints
- Menus: none found
- Keys: Escape; Shift+Escape when editorFocus and parameterHintsVisible
- Settings: editor.parameterHints.cycle
- Cursor: present (id, both keys and the visible precondition found; the blur and mouse-click cancel paths are in the model code)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHints.ts (lines 101-116); V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (blur line 83, mouse click lines 296-298, empty result lines 216-222, end of list lines 140-143 and 159-162)

### AC-3.11 Turn automatic hints off
- Tag: both
- Evidence: source
- Boundary: no
- What: Switching `editor.parameterHints.enabled` off stops the popup from opening on its own when you type a trigger character and closes one that is open at that moment; Ctrl+Shift+Space still opens it by hand.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.parameterHints.enabled; editor.parameterHints
- Cursor: present (key found with default true)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (triggerOnType and cancel on change lines 310-316, onDidType guard lines 280-283); V\src\vs\editor\common\config\editorOptions.ts (lines 3680-3684); V\src\vs\editor\contrib\parameterHints\browser\parameterHints.ts (manual trigger lines 70-72, 89-95)

### AC-3.12 Hints inside strings and comments
- Tag: both
- Evidence: bundle
- Boundary: no
- What: Typing a trigger character inside a string or a comment does not open the popup for TypeScript and JavaScript, because the language service returns nothing for a typed character there, and that rule is not applied to the Ctrl+Shift+Space trigger; the editor itself has no string or comment rule, so each language decides for itself.
- Commands: editor.action.triggerParameterHints
- Menus: none found
- Keys: Ctrl+Shift+Space (Cmd+Shift+Space on macOS) when editorTextFocus
- Settings: none found
- Cursor: present (the TypeScript library shipped in the Cursor install has the rule; V's own TypeScript library is not in the V tree, so stock was read only through its provider, which passes the same typed-character reason)
- Source: C\extensions\node_modules\typescript\lib\typescript.js (lines 178240-178241 string and comment test for a typed character, lines 178255-178268 syntactic owner test); V\extensions\typescript-language-features\src\languageFeatures\signatureHelp.ts (toTsTriggerReason lines 103-122); V\src\vs\editor\contrib\parameterHints\browser\parameterHintsModel.ts (onDidType lines 280-294, no token type test)

### AC-3.13 Screen reader announcement of the active parameter
- Tag: both
- Evidence: source
- Boundary: no
- What: Screen readers are told the active parameter's label, then its documentation and the signature's documentation, followed by "hint", and only when the active parameter changes so typing does not repeat it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (message "{0}, hint" found in the NLS file and the announcedLabel check in the widget code)
- Source: V\src\vs\editor\contrib\parameterHints\browser\parameterHintsWidget.ts (render lines 242-264, announcedLabel lines 52 and 168); C bundle (NLS "{0}, hint")

## Handed off
none

## Unclear
- Signature help in comments or strings: the editor core has no setting or code for it (the model never looks at the token type). The only rule found is inside the TypeScript language service (AC-3.12). The PHP provider (`V\extensions\php-language-features\src\features\signatureHelpProvider.ts`) and the HTML embedded JavaScript mode (`V\extensions\html-language-features\server\src\modes\javascriptMode.ts`) were not read, and their behavior in strings and comments is not known.
- `editor.parameterHints.cycle` default: the option interface comment says "Defaults to false" (`V\src\vs\editor\common\config\editorOptions.ts` line 3660) but the code default is true (line 3674) and so is the bundle default, so the doc comment is stale.
- Cursor listens to the parameter hints model (`onChangedHints`, `cppTypeService.onChangedParameterHints`, `getRelevantParameterHints`) and uses the hints as context for Cursor Tab, including asking for a Cursor Tab suggestion when hints change. That is model-side wiring with no UI of its own, so it is not a feature here. It was seen in strings only and not run.
- Nothing Cursor-only was found for parameter hints. Every file in the parameter hints folder is identical between V and N, so nothing newer than 1.128 exists in this group.
- The widget adds a `visible` class 100 ms after showing (`parameterHintsWidget.ts` lines 153-155) but no rule for that class was found in `parameterHints.css` or the Cursor CSS, so any fade-in effect, if one exists, lives somewhere not read.
- The popup is also hosted by the Interactive Window input editor (`V\src\vs\workbench\contrib\interactive\browser\interactiveEditor.ts` line 42 imports the controller); no extra UI was found there and it is not listed.
