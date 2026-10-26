# VS Code top-level UI

- Date: 2026-10-25
- Source: C:\Users\Vinnie\src\vscode

The workbench shell is split into part folders under src\vs\workbench\browser\parts: titlebar, banner, activitybar, sidebar, editor, panel, auxiliarybar, statusbar, notifications, and dialogs. Quick access is a separate contribution under src\vs\workbench\contrib\quickaccess. The autocomplete widgets are editor contributions under src\vs\editor\contrib\suggest, src\vs\editor\contrib\inlineCompletions, and src\vs\editor\contrib\parameterHints.

Side bar views are contents of the side bar. Those views are files, search, scm, debug, testing, outline, and timeline. Panel tools are contents of the panel: terminal, markers (Problems), and output. The editor family also includes notebooks, the merge editor, multi-diff, preferences, and welcome pages.

Left out of this top-level pass: contrib\extensions, contrib\chat, inlineChat, welcomeAgentSessions, agentsVoice, canvases, remoteCodingAgents, terminal chat, and contrib\mcp. Providers that produce suggestions stay out of the Autocomplete section.
