# 📚 Docs Index

A guide to what lives in this folder — focused on agentic Salesforce development, tooling setup, and CumulusCI workflows.

---

## 🤖 Agentic Coding

| File | What it covers |
|------|----------------|
| [GITHUB-COPILOT-INSTRUCTIONS.md](GITHUB-COPILOT-INSTRUCTIONS.md) | Master AI instructions for GitHub Copilot in IntelliJ — LWC patterns, Apex gotchas, deployment strategy, testing rules, terminal best practices, and more. The primary reference for AI-assisted development on any Salesforce project. |
| [vibe-coding.md](vibe-coding.md) | How to set up the full agentic coding stack: Git, Node.js, GitHub CLI, Salesforce CLI, Salesforce MCP, IntelliJ IDEA, Illuminated Cloud plugin, and GitHub Copilot plugin. |

---

## 🛠️ Terminal Tooling

| File | What it covers |
|------|----------------|
| [vide-coding-terminal-add-ons.md](vide-coding-terminal-add-ons.md) | Installation and usage guide for all terminal productivity tools: Node.js, PurgeCSS, Python, Playwright, fd, bat, ripgrep, **jq**, BackstopJS, and tokei. Includes a complete install script and verification steps. |

### Quick Tool Reference

| Tool | Purpose | Install |
|------|---------|---------|
| **Node.js / npm** | Runtime for PurgeCSS, BackstopJS | [nodejs.org](https://nodejs.org/) |
| **PurgeCSS** | Strip unused CSS from templates | `npm install -D purgecss` |
| **Python 3** | Test scripts, JSON parsing helpers | [python.org](https://www.python.org/) |
| **Playwright** | Headless browser automation & screenshots | `pip install playwright` |
| **BackstopJS** | Visual regression testing (pixel-diff HTML reports) | `npm install -g backstopjs` |
| **fd** | Fast file finder | `choco install fd` |
| **bat** | Syntax-highlighted `cat` | `choco install bat` |
| **ripgrep (rg)** | Fast text search across files | `choco install ripgrep` |
| **jq** | JSON processor — filter/extract SF CLI output | `choco install jq` |
| **tokei** | Line-of-code stats by language | `choco install tokei` |

---

## ☁️ Salesforce Org Setup & Workflows

| File | What it covers |
|------|----------------|
| [setup.md](setup.md) | Full project setup walkthrough: installing Salesforce CLI, Git, VS Code/IntelliJ, initializing CumulusCI, connecting to GitHub, spinning up scratch orgs, retrieving and deploying changes. |
| [cci-in-codebuilder.md](cci-in-codebuilder.md) | Installing and using CumulusCI inside Salesforce **Code Builder** (the browser-based IDE). Covers pip setup, project init, and Dev Hub authentication. |
| [scratch-org-snapshots.md](scratch-org-snapshots.md) | Creating, listing, deleting, and using SF CLI org **snapshots** to quickly spin up pre-configured scratch orgs. |
| [unlocked-packages-cci.md](unlocked-packages-cci.md) | Building and promoting **unlocked packages** using CumulusCI — beta release flow, dependency resolution, and promoting to production. |

---

## 🗺️ Where Things Live

```
docs/
├── index.md                        ← you are here
├── GITHUB-COPILOT-INSTRUCTIONS.md  ← AI coding rules (global, project-agnostic)
├── vibe-coding.md                  ← agentic stack setup
├── vide-coding-terminal-add-ons.md ← terminal tool installs
├── setup.md                        ← CCI + VS Code project setup
├── cci-in-codebuilder.md           ← CCI inside browser IDE
├── scratch-org-snapshots.md        ← org snapshot commands
└── unlocked-packages-cci.md        ← packaging workflow
```

> **Project-specific context** (org aliases, object names, URLs) belongs in `docs/AI-TOOLS-CONFIG.md` within the relevant project repo — not in the files above.

