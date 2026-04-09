# 🤖 Agentic Salesforce Development

This site is a practical guide to building an efficient, AI-assisted Salesforce development workflow — one where GitHub Copilot acts as a seasoned pair programmer, not just an autocomplete engine.

Start with the philosophy, then work through the setup and reference docs in order.

---

## 🗺️ The Big Picture

> **[agent-coding-best-practices.md](agent-coding-best-practices.md)** — Start here. Covers the full agentic workflow: why to consolidate your codebase, how to write global AI instructions, when to lead vs. let the agent drive, and how to wrap up a feature with code analysis and documentation.

---

## Step 1 — Set Up Your Agentic Stack

Before you write a line of code, get your tools and AI context in place.

| Doc | What it covers |
|-----|----------------|
| [vibe-coding.md](vibe-coding.md) | Install the full agentic coding stack: Git, Node.js, GitHub CLI, Salesforce CLI, Salesforce MCP, IntelliJ IDEA, Illuminated Cloud plugin, and GitHub Copilot plugin. |
| [vide-coding-terminal-add-ons.md](vide-coding-terminal-add-ons.md) | Terminal productivity tools that make the agent's life easier — `fd`, `bat`, `ripgrep`, `jq`, `tokei`, PurgeCSS, Playwright, and BackstopJS. Includes a complete install script. |
| [GITHUB-COPILOT-INSTRUCTIONS.md](GITHUB-COPILOT-INSTRUCTIONS.md) | The master AI instruction set for GitHub Copilot — LWC patterns, Apex gotchas, deployment strategy, testing rules, terminal best practices. The primary global context file for any Salesforce project. |

### Quick Terminal Tool Reference

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

## Step 2 — Consolidate Your Codebase

The less code in your repo, the better your AI suggestions. Use scratch orgs and CumulusCI to keep only what's relevant.

| Doc | What it covers |
|-----|----------------|
| [setup.md](setup.md) | Full project setup: installing Salesforce CLI, Git, VS Code/IntelliJ, initializing CumulusCI, connecting to GitHub, spinning up scratch orgs, retrieving and deploying changes. |
| [cci-in-codebuilder.md](cci-in-codebuilder.md) | Running CumulusCI inside Salesforce **Code Builder** (the browser-based IDE) — pip setup, project init, and Dev Hub auth. |
| [scratch-org-snapshots.md](scratch-org-snapshots.md) | Creating, listing, and using SF CLI org **snapshots** to spin up pre-configured scratch orgs fast. |
| [unlocked-packages-cci.md](unlocked-packages-cci.md) | Building and promoting **unlocked packages** with CumulusCI — beta release flow, dependency resolution, promoting to production. |

---

## Step 3 — Let the Agent Have at It

With your tools and context in place, you're ready to direct the agent. Watch its reasoning, stop it when it drifts, and let it deploy to scratch orgs so errors surface immediately.

Refer back to **[agent-coding-best-practices.md](agent-coding-best-practices.md)** for the full guidance on staying in control while the agent does the heavy lifting.

---

## 🗺️ Where Things Live

```
docs/
├── index.md                          ← you are here
├── agent-coding-best-practices.md    ← workflow philosophy (start here)
├── GITHUB-COPILOT-INSTRUCTIONS.md    ← AI coding rules (global, project-agnostic)
├── vibe-coding.md                    ← agentic stack setup
├── vide-coding-terminal-add-ons.md   ← terminal tool installs
├── setup.md                          ← CCI + VS Code project setup
├── cci-in-codebuilder.md             ← CCI inside browser IDE
├── scratch-org-snapshots.md          ← org snapshot commands
└── unlocked-packages-cci.md          ← packaging workflow
```

> **Project-specific context** (org aliases, object names, URLs) belongs in `docs/AI-TOOLS-CONFIG.md` within the relevant project repo — not in the files above.
