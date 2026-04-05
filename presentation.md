# Introduction to VSCode Extensions

**Building, Publishing & Mastering the Extension Ecosystem**

Extension API -- Language Server Protocol -- Webviews -- Marketplace -- Debugging

---

## Table of Contents

1. [Agenda](#slide-02--agenda)
2. [Why VSCode Won](#slide-03--why-vscode-won)
3. [VSCode Architecture](#slide-04--vscode-architecture)
4. [Anatomy of an Extension](#slide-05--anatomy-of-an-extension)
5. [Your First Extension](#slide-06--your-first-extension)
6. [Commands, Menus & Keybindings](#slide-07--commands-menus--keybindings)
7. [Language Features](#slide-08--language-features)
8. [Language Server Protocol (LSP)](#slide-09--language-server-protocol-lsp)
9. [TreeViews & Custom Views](#slide-10--treeviews--custom-views)
10. [Webviews](#slide-11--webviews)
11. [Workspace, FileSystem & Configuration](#slide-12--workspace-filesystem--configuration)
12. [Debugging Your Extension](#slide-13--debugging-your-extension)
13. [Testing Extensions](#slide-14--testing-extensions)
14. [Notebooks, Terminal & Source Control](#slide-15--notebooks-terminal--source-control)
15. [Packaging with vsce](#slide-16--packaging-with-vsce)
16. [Publishing to the Marketplace](#slide-17--publishing-to-the-marketplace)
17. [CI/CD for Extensions](#slide-18--cicd-for-extensions)
18. [Performance & Security](#slide-19--performance--security)
19. [Summary & Next Steps](#slide-20--summary--next-steps)

---

## Slide 02 -- Agenda

### Foundations

- Why VSCode won & the extension model
- Architecture: Electron, processes, isolation
- Extension anatomy: manifest, activation, lifecycle
- Your first extension in 5 minutes

### Core APIs

- Commands, menus, & keybindings
- Language features & Language Server Protocol
- TreeViews, Webviews, & custom editors
- Workspace, file system, & configuration

### Advanced Topics

- Debugging extensions & the Extension Host
- Testing with @vscode/test-electron
- Webview UI Toolkit & message passing
- Notebooks, terminal, & source control APIs

### Ship It

- Packaging with vsce
- Publishing to the Marketplace
- CI/CD for extensions
- Performance, security, & maintenance

---

## Slide 03 -- Why VSCode Won

Visual Studio Code launched in 2015 and quickly became the **most popular code editor** in the world. The Stack Overflow Developer Survey consistently shows 70%+ adoption. The secret is its **extension model**.

Unlike monolithic IDEs, VSCode ships a **lean core** and delegates language support, debugging, themes, and tooling to extensions. This means the community -- not Microsoft -- drives most of the editor's capability.

### By the Numbers

- **50,000+** extensions on the Marketplace
- **Billions** of total installs across all extensions
- **Monthly** release cadence with public roadmap
- **MIT-licensed** core (Code - OSS), proprietary build (Visual Studio Code)

### The Extension Philosophy

- **Contribution points** -- declare what you add (commands, views, languages, themes)
- **Activation events** -- load only when needed
- **Isolation** -- extensions run in a separate process
- **Common APIs** -- one extension model for all features

### What Extensions Can Do

- Add language support (syntax, IntelliSense, linting)
- Provide debuggers for any runtime
- Create custom UI panels, sidebars, and editors
- Integrate with external tools and services
- Add themes, icons, and keybinding schemes
- Extend the terminal, SCM, and notebook experiences

---

## Slide 04 -- VSCode Architecture

VSCode is built on **Electron** (Chromium + Node.js). Extensions run in a separate **Extension Host** process so a misbehaving extension cannot freeze the editor UI.

```
+------------------------------------------------------------------+
|               Main Process (Electron)                            |
+-------------------------------+----------------------------------+
| Renderer Process (UI)         | Extension Host Process           |
| Editor, panels, sidebar,      | Runs all extensions -- separate  |
| terminal -- Chromium           | Node.js process                  |
|          <--- IPC --->        |                                  |
+-------------------------------+----------------------------------+
| Webview iframes               | Language Server  | Debug Adapter |
| (sandboxed Chromium)          | (LSP)            | (DAP)         |
+-------------------------------+------------------+---------------+
```

### Renderer

The UI you see. Monaco editor, tree views, status bar. Communicates with Extension Host over IPC (JSON-RPC).

### Extension Host

A Node.js process that loads and runs extensions. Extensions share this process -- heavy work should be offloaded to a Language Server or worker.

### Language Server Protocol

A JSON-RPC protocol between the Extension Host and a separate language server process. Enables completions, diagnostics, hover, go-to-definition, and more.

---

## Slide 05 -- Anatomy of an Extension

### File Structure

```text
my-extension/
├── .vscode/
│   └── launch.json        # debug config
├── src/
│   └── extension.ts       # entry point
├── package.json           # manifest (critical!)
├── tsconfig.json          # TypeScript config
├── README.md              # Marketplace listing
├── CHANGELOG.md           # version history
└── icon.png               # 128x128 Marketplace icon
```

### Activation Events

Extensions are **lazy-loaded**. They only activate when a matching event fires:

- `onCommand:myExt.run` -- command executed
- `onLanguage:python` -- file of language opened
- `workspaceContains:**/Cargo.toml` -- file exists
- `onStartupFinished` -- after startup (use sparingly)
- `*` -- always active (strongly discouraged)

### package.json -- The Manifest

```json
{
  "name": "my-extension",
  "displayName": "My Extension",
  "version": "1.0.0",
  "engines": { "vscode": "^1.85.0" },
  "activationEvents": [],
  "main": "./out/extension.js",
  "contributes": {
    "commands": [{
      "command": "myExt.helloWorld",
      "title": "Hello World"
    }],
    "keybindings": [{
      "command": "myExt.helloWorld",
      "key": "ctrl+shift+h",
      "when": "editorTextFocus"
    }],
    "configuration": {
      "title": "My Extension",
      "properties": {
        "myExt.greeting": {
          "type": "string",
          "default": "Hello"
        }
      }
    }
  }
}
```

---

## Slide 06 -- Your First Extension

### Step 1: Scaffold

```bash
# Install the generator
npm install -g yo generator-code

# Scaffold a new extension
yo code

# Choose:
#   > New Extension (TypeScript)
#   Name: hello-world
#   Identifier: hello-world
#   Enable strict mode: Yes
```

### Step 2: Run

- Open the project in VSCode
- Press `F5` -- opens Extension Development Host
- Run command: `Ctrl+Shift+P` -> "Hello World"
- See your notification message appear

### Step 3: Iterate

Edit code -> `Ctrl+Shift+F5` to reload the Development Host. Breakpoints work in the main VSCode instance. Console output appears in the Debug Console.

### extension.ts -- Entry Point

```typescript
import * as vscode from 'vscode';

// Called when the extension is activated
export function activate(context: vscode.ExtensionContext) {

  // Register a command
  const disposable = vscode.commands.registerCommand(
    'hello-world.helloWorld',
    () => {
      // Read from configuration
      const config = vscode.workspace.getConfiguration('myExt');
      const greeting = config.get<string>('greeting', 'Hello');

      // Show an information message
      vscode.window.showInformationMessage(
        `${greeting} from Hello World!`
      );
    }
  );

  // Push to subscriptions for cleanup
  context.subscriptions.push(disposable);
}

// Called when the extension is deactivated
export function deactivate() {}
```

---

## Slide 07 -- Commands, Menus & Keybindings

**Commands** are the primary interaction model in VSCode. Every action -- from opening a file to formatting code -- is a command. Extensions register commands and bind them to menus, keybindings, or other triggers.

```typescript
// Register a text editor command
const cmd = vscode.commands.registerTextEditorCommand(
  'myExt.insertDate',
  (editor, edit) => {
    const date = new Date().toISOString().split('T')[0];
    edit.insert(editor.selection.active, date);
  }
);

// Execute another extension's command
await vscode.commands.executeCommand(
  'editor.action.formatDocument'
);
```

### When Clauses

Control visibility and enablement with context conditions:

```json
"when": "editorLangId == typescript && !editorReadonly"
"when": "resourceScheme == file"
"when": "view == myCustomView"
```

### Menu Contribution Points

```json
"contributes": {
  "menus": {
    "editor/context": [{
      "command": "myExt.insertDate",
      "when": "editorTextFocus",
      "group": "1_modification"
    }],
    "explorer/context": [{
      "command": "myExt.processFile",
      "when": "resourceExtname == .csv"
    }],
    "editor/title": [{
      "command": "myExt.togglePreview",
      "group": "navigation"
    }],
    "commandPalette": [{
      "command": "myExt.insertDate",
      "when": "editorIsOpen"
    }]
  }
}
```

### Menu Groups (ordering)

`navigation` (top), `1_modification`, `2_workspace`, `z_commands` (bottom). Items sort alphabetically within groups.

---

## Slide 08 -- Language Features

VSCode provides two approaches: **programmatic language features** (direct API) for simple cases and the **Language Server Protocol** for full-featured language support.

### Direct API -- Quick & Simple

```typescript
// Completion provider
vscode.languages.registerCompletionItemProvider(
  'markdown',
  {
    provideCompletionItems(doc, pos) {
      const items = [
        new vscode.CompletionItem('TODO',
          vscode.CompletionItemKind.Keyword),
        new vscode.CompletionItem('FIXME',
          vscode.CompletionItemKind.Keyword),
      ];
      return items;
    }
  },
  '@'  // trigger character
);

// Hover provider
vscode.languages.registerHoverProvider('json', {
  provideHover(doc, pos) {
    const range = doc.getWordRangeAtPosition(pos);
    const word = doc.getText(range);
    return new vscode.Hover(`Key: **${word}**`);
  }
});
```

### Available Providers

| Provider | Feature |
|----------|---------|
| CompletionItem | IntelliSense suggestions |
| Hover | Tooltip on hover |
| Definition | Go to definition |
| Reference | Find all references |
| DocumentSymbol | Outline & breadcrumbs |
| CodeAction | Quick fixes & refactors |
| CodeLens | Inline actionable info |
| Diagnostic | Errors & warnings |
| Formatter | Document / range formatting |
| SignatureHelp | Parameter hints |

### Diagnostics (Errors/Warnings)

```typescript
const diag = vscode.languages.createDiagnosticCollection('myLint');
diag.set(doc.uri, [
  new vscode.Diagnostic(range, 'Unused variable',
    vscode.DiagnosticSeverity.Warning)
]);
```

---

## Slide 09 -- Language Server Protocol (LSP)

The **Language Server Protocol** is a JSON-RPC protocol between an editor and a language-specific server. Invented by Microsoft for VSCode, it's now used by Vim, Emacs, Sublime, and others.

### Why LSP?

- **Write once, use everywhere** -- one server works in any LSP-capable editor
- **Any language** -- server can be written in Python, Rust, Go, etc.
- **Process isolation** -- crashes don't bring down the editor
- **Performance** -- heavy analysis runs out-of-process

### Client Side Example

```typescript
// Client side (extension.ts)
import { LanguageClient, TransportKind }
  from 'vscode-languageclient/node';

const serverModule = context.asAbsolutePath(
  'server/out/server.js'
);

const client = new LanguageClient(
  'myLangServer', 'My Language Server',
  { module: serverModule, transport: TransportKind.ipc },
  { documentSelector: [{ scheme: 'file', language: 'myLang' }] }
);

client.start();
```

### LSP Message Types

- `textDocument/completion` -- IntelliSense
- `textDocument/hover` -- hover info
- `textDocument/definition` -- go-to-definition
- `textDocument/publishDiagnostics` -- errors
- `textDocument/formatting` -- code formatting
- `textDocument/rename` -- symbol rename

### Popular Language Servers

typescript-language-server, rust-analyzer, pylsp, gopls, clangd, lua-language-server

---

## Slide 10 -- TreeViews & Custom Views

TreeViews let you add custom panels to the **sidebar**, **panel area**, or **Activity Bar**. They display hierarchical data with icons, descriptions, and context menus.

### Contribution in package.json

```json
"contributes": {
  "viewsContainers": {
    "activitybar": [{
      "id": "myExtExplorer",
      "title": "My Extension",
      "icon": "resources/icon.svg"
    }]
  },
  "views": {
    "myExtExplorer": [{
      "id": "myExt.projectsView",
      "name": "Projects"
    }, {
      "id": "myExt.tasksView",
      "name": "Tasks"
    }]
  }
}
```

### TreeDataProvider

```typescript
class ProjectProvider
  implements vscode.TreeDataProvider<ProjectItem> {

  private _onDidChange = new vscode.EventEmitter<
    ProjectItem | undefined
  >();
  readonly onDidChangeTreeData = this._onDidChange.event;

  getTreeItem(el: ProjectItem): vscode.TreeItem {
    return el;
  }

  async getChildren(el?: ProjectItem) {
    if (!el) {
      return this.getProjects();  // root items
    }
    return el.getFiles();          // child items
  }

  refresh(): void {
    this._onDidChange.fire(undefined);
  }
}

// Register the provider
vscode.window.registerTreeDataProvider(
  'myExt.projectsView',
  new ProjectProvider()
);
```

---

## Slide 11 -- Webviews

Webviews are **sandboxed iframes** that render custom HTML/CSS/JS inside VSCode. Use them for rich UIs that go beyond the built-in component model -- dashboards, visual editors, forms, or rendered previews.

### Creating a Webview Panel (Extension Side)

```typescript
// Create a Webview panel
const panel = vscode.window.createWebviewPanel(
  'myPreview',           // internal ID
  'My Preview',          // title
  vscode.ViewColumn.Two, // editor column
  {
    enableScripts: true,           // allow JS
    retainContextWhenHidden: true, // keep state
    localResourceRoots: [          // restrict access
      vscode.Uri.joinPath(context.extensionUri, 'media')
    ]
  }
);

// Set HTML content
panel.webview.html = getHtmlContent(panel.webview);

// Receive messages from webview
panel.webview.onDidReceiveMessage(msg => {
  if (msg.command === 'save') {
    saveData(msg.data);
  }
});
```

### Inside the Webview HTML

```html
<script>
  const vscode = acquireVsCodeApi();

  // Send message to extension
  vscode.postMessage({
    command: 'save',
    data: { name: 'test' }
  });

  // Receive messages from extension
  window.addEventListener('message', event => {
    const msg = event.data;
    if (msg.command === 'update') {
      document.getElementById('content')
        .textContent = msg.text;
    }
  });

  // Persist state across visibility changes
  vscode.setState({ scrollPos: 42 });
  const state = vscode.getState();
</script>
```

### Security

Webviews are sandboxed. Use `Content-Security-Policy` headers, `nonce` attributes on scripts, and `localResourceRoots` to restrict file access. Never inject user content as raw HTML.

---

## Slide 12 -- Workspace, FileSystem & Configuration

### Workspace API

```typescript
// Find files
const files = await vscode.workspace.findFiles(
  '**/*.test.ts',    // include
  '**/node_modules/**' // exclude
);

// Read & write files
const uri = vscode.Uri.file('/path/to/file.txt');
const data = await vscode.workspace.fs.readFile(uri);
await vscode.workspace.fs.writeFile(uri, Buffer.from('new'));

// Watch for changes
const watcher = vscode.workspace.createFileSystemWatcher(
  '**/*.json'
);
watcher.onDidChange(uri => console.log('Changed:', uri));
watcher.onDidCreate(uri => console.log('Created:', uri));
watcher.onDidDelete(uri => console.log('Deleted:', uri));

// Apply a batch of edits
const wsEdit = new vscode.WorkspaceEdit();
wsEdit.replace(uri, range, 'replacement text');
wsEdit.createFile(newUri, { ignoreIfExists: true });
await vscode.workspace.applyEdit(wsEdit);
```

### Configuration API

```typescript
// Read configuration
const config = vscode.workspace.getConfiguration('myExt');
const value = config.get<string>('greeting', 'default');

// Write configuration
await config.update(
  'greeting', 'Bonjour',
  vscode.ConfigurationTarget.Workspace
);

// Watch for config changes
vscode.workspace.onDidChangeConfiguration(e => {
  if (e.affectsConfiguration('myExt.greeting')) {
    // reload
  }
});
```

### Configuration Targets

- **Global** -- user settings (~/.config/Code/...)
- **Workspace** -- .vscode/settings.json
- **WorkspaceFolder** -- per-folder in multi-root

### Storage API

Use `context.globalState` and `context.workspaceState` for persistent key-value storage. Use `context.secrets` for sensitive data (tokens, passwords) -- stored in the OS keychain.

---

## Slide 13 -- Debugging Your Extension

VSCode has first-class support for debugging extensions. Press **F5** to launch an **Extension Development Host** -- a second VSCode window with your extension loaded.

### launch.json

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Extension",
      "type": "extensionHost",
      "request": "launch",
      "args": ["--extensionDevelopmentPath=${workspaceFolder}"],
      "outFiles": ["${workspaceFolder}/out/**/*.js"],
      "preLaunchTask": "npm: watch"
    },
    {
      "name": "Extension Tests",
      "type": "extensionHost",
      "request": "launch",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}",
        "--extensionTestsPath=${workspaceFolder}/out/test"
      ]
    }
  ]
}
```

### Debugging Techniques

- **Breakpoints** -- set in your extension source files (TypeScript or JavaScript)
- **Debug Console** -- evaluate expressions in the Extension Host context
- **Output Channel** -- dedicated logging panel for your extension
- **Developer Tools** -- Ctrl+Shift+I opens Chromium DevTools for the renderer

```typescript
// Create a dedicated output channel
const log = vscode.window.createOutputChannel('My Extension');
log.appendLine('Extension activated');
log.show();  // focus the channel

// Use console.log too — appears in Debug Console
console.log('Debug info:', someObject);
```

### Common Pitfalls

- Forgetting to compile TS before debugging
- Activation event doesn't match -- extension never loads
- Stale Extension Host -- restart with Ctrl+Shift+F5
- Wrong `engines.vscode` version in package.json

---

## Slide 14 -- Testing Extensions

Extension tests run inside a real VSCode instance via **@vscode/test-electron**. This gives tests full access to the VSCode API, real file systems, and installed extensions.

### Test Runner Setup

```typescript
// src/test/runTest.ts
import { runTests } from '@vscode/test-electron';
import path from 'path';

async function main() {
  await runTests({
    extensionDevelopmentPath: path.resolve(__dirname, '../../'),
    extensionTestsPath: path.resolve(__dirname, './suite/index'),
    launchArgs: [
      '--disable-extensions',  // isolate your extension
      path.resolve(__dirname, '../../test-fixtures')
    ]
  });
}
main();
```

### Writing Tests

```typescript
// src/test/suite/extension.test.ts
import * as assert from 'assert';
import * as vscode from 'vscode';

suite('Extension Tests', () => {

  test('Command is registered', async () => {
    const cmds = await vscode.commands.getCommands(true);
    assert.ok(cmds.includes('myExt.helloWorld'));
  });

  test('Inserts date at cursor', async () => {
    const doc = await vscode.workspace.openTextDocument({
      content: '', language: 'plaintext'
    });
    const editor = await vscode.window.showTextDocument(doc);

    await vscode.commands.executeCommand('myExt.insertDate');

    const text = doc.getText();
    assert.match(text, /^\d{4}-\d{2}-\d{2}$/);
  });
});
```

### Unit Tests (No VSCode)

Pure logic (parsers, formatters, utils) can be tested with Jest or Mocha directly -- no need for the Extension Host. Only test VSCode API interactions with @vscode/test-electron.

---

## Slide 15 -- Notebooks, Terminal & Source Control

### Notebook API

Create custom notebook types (like Jupyter) with your own kernel. Register a `NotebookSerializer` to load/save and a `NotebookController` to execute cells.

```typescript
const controller = vscode.notebooks
  .createNotebookController(
    'myKernel', 'my-notebook', 'My Kernel'
  );

controller.executeHandler = async (cells) => {
  for (const cell of cells) {
    const exec = controller.createNotebookCellExecution(cell);
    exec.start(Date.now());
    // Run the cell content
    const result = evaluate(cell.document.getText());
    exec.replaceOutput([
      new vscode.NotebookCellOutput([
        vscode.NotebookCellOutputItem.text(result)
      ])
    ]);
    exec.end(true);
  }
};
```

### Terminal API

Create, write to, and manage integrated terminals programmatically.

```typescript
// Create a terminal
const term = vscode.window.createTerminal({
  name: 'My Build',
  cwd: '/path/to/project',
  env: { NODE_ENV: 'test' }
});

term.show();
term.sendText('npm test');

// Terminal link provider
vscode.window.registerTerminalLinkProvider({
  provideTerminalLinks(ctx) {
    // Detect patterns in terminal output
    const match = ctx.line.match(
      /error in (.+):(\d+)/
    );
    if (match) {
      return [{
        startIndex: match.index,
        length: match[0].length,
        file: match[1], line: match[2]
      }];
    }
  },
  handleTerminalLink(link) {
    // Open the file at the error line
  }
});
```

### Source Control API

Extensions can provide their own SCM (not just Git). Register changes, groups, and quick-diff decorations.

```typescript
const scm = vscode.scm.createSourceControl(
  'myScm', 'My SCM'
);

const changes = scm.createResourceGroup(
  'changes', 'Changes'
);

changes.resourceStates = [{
  resourceUri: fileUri,
  decorations: {
    strikeThrough: false,
    tooltip: 'Modified'
  }
}];

scm.quickDiffProvider = {
  provideOriginalResource(uri) {
    // Return URI of the original file
    return toOriginalUri(uri);
  }
};
```

---

## Slide 16 -- Packaging with vsce

**vsce** (Visual Studio Code Extensions) is the CLI tool for packaging and publishing. It bundles your extension into a `.vsix` file -- a zip containing your compiled code, manifest, README, and assets.

```bash
# Install vsce
npm install -g @vscode/vsce

# Package into a .vsix
vsce package
# Creates: my-extension-1.0.0.vsix

# Install locally for testing
code --install-extension my-extension-1.0.0.vsix

# Check what will be included
vsce ls
```

### .vscodeignore

Like `.gitignore` but for packaging. Exclude dev files to keep the vsix small:

```text
.vscode/**
src/**
node_modules/**
!node_modules/prod-dependency/**
**/*.test.ts
tsconfig.json
.eslintrc.json
```

### Bundling with esbuild

Bundle your extension into a single file for faster activation and smaller size:

```javascript
// esbuild.mjs
import { build } from 'esbuild';

await build({
  entryPoints: ['src/extension.ts'],
  bundle: true,
  outfile: 'out/extension.js',
  external: ['vscode'],  // provided by VSCode
  format: 'cjs',
  platform: 'node',
  minify: true,
  sourcemap: true,
});
```

### Size Matters

- Smaller vsix -> faster install & marketplace load
- Fewer files -> faster activation (less I/O)
- Bundle + minify can shrink 10x
- Tree-shake unused dependencies

### Pre-publish Checks

- README.md renders correctly (used on Marketplace page)
- CHANGELOG.md documents version changes
- icon.png is 128x128 and looks good at small sizes
- `engines.vscode` is the minimum supported version

---

## Slide 17 -- Publishing to the Marketplace

### Setup (one-time)

```bash
# 1. Create a publisher at:
#    https://marketplace.visualstudio.com/manage

# 2. Create a Personal Access Token (PAT)
#    Azure DevOps -> User Settings -> PATs
#    Scope: Marketplace (Manage)

# 3. Login with vsce
vsce login <publisher-name>
# Paste your PAT when prompted
```

### Publish

```bash
# Publish current version
vsce publish

# Bump version and publish
vsce publish minor   # 1.0.0 -> 1.1.0
vsce publish patch   # 1.1.0 -> 1.1.1

# Pre-release versions
vsce publish --pre-release
```

### Unpublish

`vsce unpublish <publisher>.<extension>` removes it entirely. Use **with caution** -- users who depend on it will lose access.

### Marketplace Listing

```json
{
  "publisher": "myPublisher",
  "displayName": "My Extension",
  "description": "One-line summary",
  "icon": "icon.png",
  "galleryBanner": {
    "color": "#0a0a0f",
    "theme": "dark"
  },
  "categories": [
    "Programming Languages",
    "Formatters",
    "Linters"
  ],
  "keywords": ["python", "linting", "formatter"],
  "repository": {
    "type": "git",
    "url": "https://github.com/me/my-ext"
  },
  "badges": [{
    "url": "https://img.shields.io/...",
    "href": "https://...",
    "description": "Build Status"
  }]
}
```

### Discovery Tips

- Clear, searchable `displayName`
- Accurate `categories` and `keywords`
- High-quality README with screenshots/GIFs
- Respond to issues and reviews promptly

---

## Slide 18 -- CI/CD for Extensions

### GitHub Actions Workflow

```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run lint
      - run: npm run compile
      - run: npm test
        env:
          DISPLAY: ':99.0'
      - run: npx @vscode/vsce package
      - uses: actions/upload-artifact@v4
        with:
          name: vsix
          path: '*.vsix'

  publish:
    needs: build
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npx @vscode/vsce publish
        env:
          VSCE_PAT: ${{ secrets.VSCE_PAT }}
```

### The Display Problem

Extension tests launch a real VSCode window and need a display server. On Linux CI, use **xvfb** (X Virtual Frame Buffer):

```yaml
- run: |
    sudo apt-get install -y xvfb
    xvfb-run -a npm test
```

### Release Strategy

- **Tag-based** -- push `v1.2.3` tag to trigger publish
- **Pre-release channel** -- ship beta builds for early feedback
- **Changelog** -- auto-generate from conventional commits
- **Semantic versioning** -- match npm/Marketplace expectations

### Multi-Platform Testing

Test on **ubuntu**, **macos**, and **windows** runners. Path separators, line endings, and shell behaviour differ. Use a matrix strategy:

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest, windows-latest]
runs-on: ${{ matrix.os }}
```

---

## Slide 19 -- Performance & Security

### Activation Performance

- Use **specific activation events** -- never `*`
- Bundle with esbuild/webpack -- one file, no node_modules scan
- Defer heavy work with `setTimeout` or `setImmediate`
- Use **Extension Bisect** (`Help > Start Extension Bisect`) to find slow extensions

### Runtime Performance

- Offload CPU work to Language Servers or Worker Threads
- Debounce file watchers and text change listeners
- Use `CancellationToken` to abort stale requests
- Cache expensive computations
- Profile with `Developer: Show Running Extensions`

### Startup Timeline

`Developer: Startup Performance` shows exactly when your extension activated, how long it took, and what triggered it.

### Security Considerations

- **Extensions run with full user permissions** -- they can read/write any file, make network requests, and execute shell commands
- VSCode does **not sandbox** extension code (only Webview content)
- The Marketplace has **basic malware scanning** but no full audit

### Best Practices for Authors

- Request **minimal permissions** and file access
- Validate all user and file input
- Use `context.secrets` for tokens (OS keychain)
- Pin dependency versions & audit regularly
- Never execute arbitrary code from untrusted sources
- Set CSP headers on all Webviews

### For Users

- Check publisher reputation and install counts
- Review source code if open-source
- Use **Workspace Trust** to restrict extensions in untrusted folders
- Review extension permissions before installing

---

## Slide 20 -- Summary & Next Steps

### Key Takeaways

- VSCode's power comes from its extension model
- Extensions run in a separate Extension Host process
- `package.json` is the manifest -- declares everything
- Activation events keep extensions lazy and fast
- Language Server Protocol enables reusable language tooling
- Webviews provide rich custom UIs (sandboxed iframes)
- Bundle with esbuild for fast activation and small size
- Publish to the Marketplace with vsce
- Test in real VSCode with @vscode/test-electron
- Automate build, test, publish with GitHub Actions

### Recommended Reading

- **code.visualstudio.com/api** -- official Extension API docs
- **github.com/microsoft/vscode-extension-samples** -- official samples
- **Language Server Protocol spec** -- microsoft.github.io/language-server-protocol
- **VSCode Extension Marketplace** -- study popular extensions' source

### Project Ideas

- Snippet pack for your favourite framework
- Custom theme based on your terminal colours
- Linter for a domain-specific config format
- Sidebar panel that integrates with an external API
- Notebook controller for a scripting language
- Language Server for a niche programming language

---

*Thank you! -- Built with Reveal.js -- Single self-contained HTML file*
