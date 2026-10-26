Section: all (step zero). Date: 2026-10-25. Pass: 0.

# Version check and how UI parts are declared

## Version gap found and closed

- Cursor 3.27.12 is built on VS Code 1.128.0 (`C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app\product.json`, field `vscodeVersion`).
- The clone at `C:\Users\Vinnie\src\vscode` is a one-commit shallow clone of `main`, package version 1.144.0 (commit `b208bc25`, 2026-10-25). It has no tags and no 1.128 history. It is 16 minor releases newer than Cursor's base.
- The tag `1.128.0` exists on `microsoft/vscode`. It was cloned with `--depth 1 --branch 1.128.0` into `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0` (about 260 MB, inside the gitignored scratch folder). `git describe` there prints `1.128.0`.
- The original clone was not touched.
- Rule for every pass: V (the 1.128.0 clone) is the primary source. N (the 1.144.0 clone) is only used to spot things added after Cursor's base, marked `newer-than-1.128`.

## Size of each source

| Source | Facts |
|---|---|
| V core workbench | 99 folders under `src\vs\workbench\contrib`. 334 files call `registerAction2`. 258 menu ids in `src\vs\platform\actions\common\actions.ts` |
| N core workbench | 102 contrib folders. 382 files call `registerAction2`. Used only for the delta |
| Cursor main bundle | `out\vs\workbench\workbench.desktop.main.js` is 39.6 MB, with a 1.2 MB stylesheet. Also `workbench.anysphere-ui-automations.js` (8.7 MB) and `workbench.xai-commerce-prepaid.js` (1.4 MB). `workbench.glass.main.js` (47.4 MB) is the Agents window and is out of scope |
| Cursor extensions folder | 117 built-in extension folders. 21 are named `cursor-*` and are mostly agent plumbing: agent-exec, agent-host, agent-worker, always-local, browser-automation, checkout, commits, computer-use, deeplink, explorer, file-service, local-agent-runtime, mcp, ndjson-ingest, polyfills-remote, resolver, resolver-helper, retrieval, shadow-workspace, socket, worktree-textmate |

## In-scope built-in extensions, counted from each `package.json`

Counts are entries under `contributes` in V (1.128.0) and N (1.144.0). Menu items are summed across every menu id.

| Extension | Version | Commands | Menu items | Views | Keybindings | Settings |
|---|---|---|---|---|---|---|
| git (Source Control) | V | 183 | 437 | 0 | 3 | 104 |
| git | N | 183 | 437 | 0 | 3 | 105 |
| markdown-language-features (preview) | V | 16 | 32 | 0 | 2 | 41 |
| markdown-language-features | N | 64 | 80 | 0 | 81 | 44 |
| merge-conflict | V and N | 10 | 4 | 0 | 1 | 4 |
| simple-browser | V and N | 1 | 1 | 0 | 1 | 1 |
| search-result | V and N | 1 | 0 | 0 | 1 | 0 |
| references-view | V and N | 21 | 40 | 1 | 4 | 1 |

The git extension declares no views in its own manifest. The Source Control view container is core workbench code under `src\vs\workbench\contrib\scm`, and the git extension fills it. The Markdown extension grew from 16 to 64 commands and from 2 to 81 keybindings between 1.128.0 and 1.144.0. Those additions are `newer-than-1.128` and should not be assumed to exist in Cursor until the bundle or Cursor's own `extensions\markdown-language-features` folder shows them.

## Where each kind of entry point is declared

Core workbench, in code under `src\vs`:
- Commands: `registerAction2(class extends Action2 { ... })`. The same class can attach menu items and a keybinding.
- Menu items: `MenuRegistry.appendMenuItem(MenuId.Xxx, ...)`, or the `menu` field of an `Action2`. Menu ids live in `src\vs\platform\actions\common\actions.ts`, for example `MenubarFileMenu`, `MenubarEditMenu`, `MenubarSelectionMenu`, `MenubarViewMenu`, `MenubarGoMenu`, `MenubarTerminalMenu`, `MenubarHelpMenu`, `CommandPalette`, `EditorContext`, `EditorTitle`, `ExplorerContext`, `SCMTitle`, `TerminalInstanceContext`.
- Keybindings: `KeybindingsRegistry.registerCommandAndKeybindingRule` and the `keybinding` field of an `Action2`.
- Settings: the configuration registry, `Registry.as(Extensions.Configuration).registerConfiguration`.
- Views and containers: `ViewContainersRegistry` and `ViewsRegistry`.

Built-in extensions: `extensions\<name>\package.json` under `contributes.commands`, `menus`, `views`, `viewsContainers`, `keybindings`, and `configuration`. Labels are in `package.nls.json` next to it.

Cursor: no source. The command ids, labels, and pane ids are strings in the bundle, so Cursor features carry the `bundle` evidence level.

## Consequence for the later passes

- The coverage script in pass 4 has two collectors: one for `registerAction2` and menu registrations in code, one for `contributes` in the in-scope manifests.
- A Cursor feature is only tagged `both` when its command id or label was found in the bundle. A feature seen only in N is `newer-than-1.128`.
