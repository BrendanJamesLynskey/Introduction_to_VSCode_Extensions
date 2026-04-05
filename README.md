# 🧩 Introduction to VSCode Extensions

An interactive Reveal.js presentation covering VSCode extensions — from architecture and the Extension API through to Language Server Protocol, Webviews, testing, packaging, publishing, and CI/CD.

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/Introduction_to_VSCode_Extensions/)

## 📄 [Markdown Version](presentation.md)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Title | VSCode extensions overview |
| 02 | Agenda | Topics at a glance |
| 03 | Why VSCode Won | Extension model, adoption, philosophy |
| 04 | Architecture | Electron, Extension Host, renderer, LSP, DAP |
| 05 | Anatomy of an Extension | File structure, manifest, activation events |
| 06 | Your First Extension | Scaffolding with Yeoman, F5 debugging, entry point |
| 07 | Commands, Menus & Keybindings | Registering commands, menu contributions, when clauses |
| 08 | Language Features | Completion, hover, diagnostics, and provider APIs |
| 09 | Language Server Protocol | JSON-RPC, client/server architecture, message types |
| 10 | TreeViews & Custom Views | Sidebar panels, Activity Bar, TreeDataProvider |
| 11 | Webviews | Sandboxed iframes, message passing, state persistence |
| 12 | Workspace & Configuration | File system API, watchers, settings, storage, secrets |
| 13 | Debugging Extensions | Extension Development Host, output channels, pitfalls |
| 14 | Testing | @vscode/test-electron, integration tests, unit tests |
| 15 | Notebooks, Terminal & SCM | Notebook controllers, terminal API, source control |
| 16 | Packaging with vsce | Bundling, .vscodeignore, esbuild, size optimisation |
| 17 | Publishing | Marketplace setup, PATs, versioning, listing metadata |
| 18 | CI/CD | GitHub Actions, multi-platform testing, release strategy |
| 19 | Performance & Security | Activation speed, profiling, trust model, best practices |
| 20 | Summary & Next Steps | Key takeaways, reading, and project ideas |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## References

Microsoft, *VSCode Extension API* — code.visualstudio.com/api · Microsoft, *Language Server Protocol Specification* — microsoft.github.io/language-server-protocol · Microsoft, *vscode-extension-samples* — github.com/microsoft/vscode-extension-samples · Microsoft, *vsce CLI* — github.com/microsoft/vscode-vsce

## License

Educational use. Code examples provided as-is.
