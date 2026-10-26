# Cursor top-level UI

- Date: 2026-10-25
- Source: C:\Users\Vinnie\AppData\Local\Programs\cursor
- Version: 3.27.12
- VS Code base: 1.128.0 (product.json in the extracted app)
- Bundle: resources\app\out\vs\workbench\workbench.desktop.main.js

The Cursor install at C:\Users\Vinnie\AppData\Local\Programs\cursor is version 3.27.12. product.json in the extracted app says the VS Code base is 1.128.0. The workbench bundle is resources\app\out\vs\workbench\workbench.desktop.main.js.

Extra editor pane ids found in that bundle: browserEditor, canvas, automations, customize, aisettings, cursorblame, cursorfileblame, reviewchanges, markdownPlan, mermaid, inlineMultiDiffEditor, and globalCommandPreview. Automations was in the install and in the raw list. The polished catalog does not promote it yet. This note keeps the id so it is not lost.

Inline diff is vs/editor/contrib/inlineDiffs. Layout menu labels are Layout Left (agent), Layout Right (editor), Layout Zen, and Browser. The title bar has an Update button. The only extra activity bar view was backgroundAgent, which is left out.

Left out: workbench.panel.aichat, composerChatViewPane, the composer input, unifiedsidebar, the Agents Window, the glass workbench, and voiceCallTranscript. Autocomplete widgets are the VS Code ones. simpleSuggestWidget was listed in NLS and was not found replacing the editor suggest widget.
