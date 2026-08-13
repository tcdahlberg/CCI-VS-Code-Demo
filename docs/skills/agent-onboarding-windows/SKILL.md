---
name: agent-onboarding
description: "General onboarding rules for AI agents in this workspace. Covers security rules, project preferences, code style standards, proven tool choices, and the startup checklist every agent should follow before doing any work."
metadata:
  author: UST-Agentic-Coding-Skills
  version: "1.0"
allowed-tools: Read
---

# Agent Onboarding

## When to Use This Skill
Activate at the start of any session or task to establish baseline rules before taking action.

---

## ⚠️ Security Rules — Non-Negotiable

- **NEVER store passwords, API keys, tokens, secrets, or credentials** in any log file, output file, or any file tracked by git or written to the project directory.
  - ❌ Writing a credential to `ai-logs/`, project root, `docs/`, or any tracked path
  - ❌ Echoing a credential value to a terminal command that pipes to a file (`| Out-File`, `| Tee-Object`, `> file.txt`)
  - ✅ Reference credentials via environment variables only: `$env:SF_USERNAME`, `$env:SF_PASSWORD`, `$env:MY_API_KEY`
  - ✅ If a credential must be passed to a command, pass it inline in the terminal without saving it anywhere
- **`ai-logs/` is gitignored but NOT safe for secrets** — treat it the same as any tracked path; other tools or accidental `git add -f` can expose it.
- **If a task requires a credential, prompt the user to set it as an environment variable** before running the command; never ask the user to type it into a file or a command that saves output.
- **Never print credential values** in explanations, code comments, or chat responses.

---

## Primary Stack

- **Build Tool:** CumulusCI (`cci`) — Python CLI for Salesforce
- **Org Type:** Scratch orgs (not production)
- **Config Files:** `cumulusci.yml`, `sfdx-project.json`
- **OS / Shell:** Windows + PowerShell (use PowerShell syntax, NOT cmd)
- **IDE:** IntelliJ IDEA + Illuminated Cloud plugin
- **Installed Tools:** Node.js, Python 3.13, CumulusCI, Salesforce CLI (`sf`), PurgeCSS, Playwright, BackstopJS, `fd`, `bat`, `ripgrep`, `tokei`, `jq`

---

## When Starting Any Task

1. Check for `cumulusci.yml` or `sfdx-project.json` (confirms Salesforce project).
2. Check `docs/AI-TOOLS-CONFIG.md` for project-specific context (org aliases, object names, URLs).
3. Use existing tools (`fd`, `rg`, PurgeCSS, `cci`) before suggesting new ones.
4. Verify commands work before declaring success.
5. Clean up failed attempts; do not leave temp files in the project root.

---

## Project Preferences

- **Keep root clean** — essential config files only; all generated output goes in `ai-logs/`.
- **Docs in `/docs`** — check `docs/AI-TOOLS-CONFIG.md` for project context.
- **Documentation discipline** — consolidate, don't proliferate:
  - Archive historical/redundant docs to `docs/archive/`.
  - Maintain 5–10 core guides max.
  - Update existing docs rather than creating new ones.
  - Use clear naming: `MAIN_GUIDE.md`, `TROUBLESHOOTING.md`, etc.

---

## Code Style Standards

### Naming: Full, Unabbreviated camelCase
Variable names, method names, and local parameters must be written in full camelCase — no abbreviations unless listed below.

| ❌ Avoid | ✅ Use instead |
|---|---|
| `getPmtHist()` | `getPaymentHistory()` |
| `calcEngScore()` | `calculateEngagementScore()` |
| `usr`, `cfg`, `evt`, `btn` | `user`, `configuration`, `event`, `button` |
| `idx`, `cnt`, `mgr`, `svc` | `index`, `count`, `manager`, `service` |

**Allowed abbreviations (universally understood or unavoidable):**
- Salesforce API/metadata names (`@api`, `@wire`, `AuraEnabled`, field API names like `Account__c`)
- Framework-required signatures (`connectedCallback`, `lwc:if`)
- Loop counters `i`, `j` in tight indexed loops
- Acronyms that ARE the full word: `url`, `id`, `dto`, `html`, `css`, `json`, `api`
- Apex system types in comments: `SObject`, `DML`, `SOQL`

### Control Flow: Always Use Braces
All `if`, `else`, `for`, `while`, `for...of` must have curly braces, even for single-line bodies. Applies to Apex, JavaScript, and TypeScript.

```apex
// ❌
if (condition) doSomething();
for (Item i : list) process(i);

// ✅
if (condition) { doSomething(); }
for (Item i : list) { process(i); }
```

### Apex Javadoc — Must Be Complete
When a method has a `/** ... */` doc comment, it must include all required tags (Illuminated Cloud linter enforces this):
- Every **non-void** method requires `@return <description>`
- Every **parameter** requires a matching `@param <name> <description>`
- If no Javadoc block exists at all, no tags are required

```apex
// ✅ Complete
/**
 * Loads the donor summary record.
 *
 * @param contactId The Id of the Contact to load
 * @return Map of summary data keyed by section name
 */
public static Map<String, Object> getSummary(Id contactId) { ... }

// ❌ Incomplete — triggers linter warning
/**
 * Loads the donor summary record.
 */
public static Map<String, Object> getSummary(Id contactId) { ... }
```

---

## Proven Tool Choices

| Task | Use | Avoid |
|---|---|---|
| Salesforce org management | CumulusCI (`cci`) | Manual CLI-only workflows |
| CSS extraction/purging | PurgeCSS | Manual regex CSS extraction (error-prone) |
| Browser/visual testing | Playwright or BackstopJS | Ad-hoc screenshot scripts |
| File search | `fd "pattern"`, `rg "text"` | `Get-ChildItem` for content search |
| File operations | PowerShell-native (`Remove-Item -Force`) | `del /Q` (cmd syntax) |
| Code stats | `tokei` | Manual line counting |
| CSS build | `node run-purgecss.js` | — |

---

## AI Working Files (`ai-logs/`)

- **All AI-generated output goes in `ai-logs/`** — deploy logs, test results, JSON output, jq filters, Python scripts.
- `ai-logs/` is in `.gitignore` — never tracked by git, but still NOT safe for secrets.
- **Standard filenames to reuse** (don't create new ones for the same purpose):
  - `ai-logs/deploy.txt` — CCI/SF deploy output
  - `ai-logs/test.txt` — CCI test run output
  - `ai-logs/sftest.txt` — SF CLI test run output
  - `ai-logs/cov.json` — coverage query results
  - `ai-logs/cov_filter.jq` — jq filter for coverage
- Any Python helper scripts go in `ai-logs/` (e.g. `ai-logs/run_tests.py`).
- Do NOT leave temp files in the project root.

---

## Lightning Out Component Caching (Common Gotcha — Seen Across Multiple Projects)

If an LWC is embedded via Lightning Out on a Visualforce/Site page (`$Lightning.use("c:someAuraApp", ...)` → `$Lightning.createComponent`) and the browser keeps showing stale behavior **even after a hard refresh**, don't assume the deploy failed — verify server-side first:

```powershell
sf data query --query "SELECT Source FROM LightningComponentResource WHERE LightningComponentBundle.DeveloperName = '<name>' AND FilePath LIKE '%.html'" --use-tooling-api --target-org <alias> --json
```

If the deployed source is already correct but the browser still renders the old version, this is **not a normal browser cache** — `Ctrl+Shift+R`, devtools "disable cache", and even manually clearing `localStorage`/`sessionStorage` do not fix it. The `auraCmpDef` endpoint serves LWC bundles with `Cache-Control: private, max-age=31536000, immutable`, keyed by a server-side "last recompile marker" (`_lrmc` query param, visible on the bootstrap `*.app?...` request). Redeploying the LWC bundle alone does **not** bump that marker, and the browser's own persistent Aura definition storage (IndexedDB-backed, survives normal reloads) keeps serving the old bundle indefinitely — sometimes without even issuing a new network request for it at all.

**Fix:** make a genuine content change (not a no-op redeploy of identical bytes — that won't bump the marker) to the Aura app/component that declares the `<aura:dependency>` for the changed LWC, then redeploy just that bundle:

```powershell
# e.g. add/remove a comment line in the .app file — must be an actual byte diff
cci task run deploy --path force-app/main/default/aura/<appName> --org <alias>
```

**Verify the fix landed** by checking browser network requests for a fresh `auraCmpDef` GET during page load — if that request is missing entirely, the browser served the definition from IndexedDB without even hitting the network, confirming it's still stale and the recompile marker still hasn't moved.

---

## Windows / Line Ending Notes

- **Windows uses CRLF (`\r\n`), Salesforce orgs use LF (`\n`)** — never compare file byte counts or line counts between local files and org to detect changes; always diff content directly.
- **Never assert on file size or line count** as a proxy for "did the deploy change anything."

