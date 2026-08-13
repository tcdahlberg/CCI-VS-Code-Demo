---
name: agent-onboarding
description: "General onboarding rules for AI agents in this workspace. Covers security rules, project preferences, code style standards, proven tool choices, and the startup checklist every agent should follow before doing any work."
metadata:
  author: UST-Agentic-Coding-Skills
  version: "2.0"
allowed-tools: Read
---

# Agent Onboarding

## When to Use This Skill
Activate at the start of any session or task to establish baseline rules before taking action.

---

## ⚠️ Security Rules — Non-Negotiable

- **NEVER store passwords, API keys, tokens, secrets, or credentials** in any log file, output file, or any file tracked by git or written to the project directory.
  - ❌ Writing a credential to `ai-logs/`, project root, `docs/`, or any tracked path
  - ❌ Echoing a credential value to a terminal command that pipes to a file (`| tee`, `> file.txt`)
  - ✅ Reference credentials via environment variables only: `$SF_USERNAME`, `$SF_PASSWORD`, `$MY_API_KEY`
  - ✅ If a credential must be passed to a command, pass it inline in the terminal without saving it anywhere
- **`ai-logs/` is gitignored but NOT safe for secrets** — treat it the same as any tracked path; other tools or accidental `git add -f` can expose it.
- **If a task requires a credential, prompt the user to set it as an environment variable** before running the command; never ask the user to type it into a file or a command that saves output.
- **Never print credential values** in explanations, code comments, or chat responses.

---

## Primary Stack

- **Build Tool:** CumulusCI (`cci`) — Python CLI for Salesforce
- **Org Type:** Scratch orgs (not production)
- **Config Files:** `cumulusci.yml`, `sfdx-project.json`
- **OS / Shell:** Bazzite Linux (Fedora/rpm-ostree base) + bash — use POSIX/bash syntax
- **IDE:** VS Code (or IntelliJ IDEA + Illuminated Cloud plugin)
- **Installed Tools:** Node.js, Python 3.x, CumulusCI, Salesforce CLI (`sf`), PurgeCSS, Playwright, BackstopJS, `fd`, `bat`, `rg` (ripgrep), `tokei`, `jq`, `fzf`, `gh`, `git-delta`, `lazygit`, `zoxide`
- **Package Management:** `rpm-ostree` for system packages (requires reboot); `flatpak` for desktop apps; `pip`/`npm`/`brew` for dev tools — avoid `dnf` directly on Bazzite

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
| File search | `fd "pattern"`, `rg "text"` | `find`/`grep` for content search (prefer `rg`) |
| File operations | bash-native (`rm -f`, `mv`, `cp`) | Windows/PowerShell syntax |
| Code stats | `tokei` | Manual line counting |
| CSS build | `node run-purgecss.js` | — |
| View files | `bat` (syntax-highlighted) | `cat` (plain) |
| Git diffs | `delta` (syntax-highlighted diffs) | Plain `git diff` output |
| Git TUI | `lazygit` | — |
| Directory jumping | `zoxide` (`z <partial-path>`) | `cd` with full paths |
| GitHub operations | `gh` (PRs, issues, releases) | Browser for routine GitHub tasks |

---

## AI Working Files (`ai-logs/`)

- **All AI-generated output goes in `ai-logs/`** — deploy logs, test results, JSON output, jq filters, Python scripts, browser screenshots/traces.
- `ai-logs/` is in `.gitignore` — never tracked by git, but still NOT safe for secrets.
- **If `ai-logs/` doesn't exist yet in a repo, create it** (and confirm it's gitignored) before writing any working files — don't leave output in the project root or in a tool's default output directory.
- **Standard filenames to reuse** (don't create new ones for the same purpose):
  - `ai-logs/deploy.txt` — CCI/SF deploy output
  - `ai-logs/test.txt` — CCI test run output
  - `ai-logs/sftest.txt` — SF CLI test run output
  - `ai-logs/cov.json` — coverage query results
  - `ai-logs/cov_filter.jq` — jq filter for coverage
  - `ai-logs/screenshots/` — Playwright/Chrome DevTools MCP screenshots, console logs, and page snapshots
- Any Python helper scripts go in `ai-logs/` (e.g. `ai-logs/run_tests.py`).
- Browser automation tools (Playwright MCP, chrome-devtools-mcp) default to writing screenshots and `.playwright-mcp/` trace/console output at the project root — explicitly pass `ai-logs/screenshots/<name>.png` as the output filename, and gitignore `.playwright-mcp/` directly in case any output still lands there.
- Do NOT leave temp files in the project root.

---

## Lightning Out Component Caching (Common Gotcha — Seen Across Multiple Projects)

If an LWC is embedded via Lightning Out on a Visualforce/Site page (`$Lightning.use("c:someAuraApp", ...)` → `$Lightning.createComponent`) and the browser keeps showing stale behavior **even after a hard refresh**, don't assume the deploy failed — verify server-side first:

```bash
sf data query --query "SELECT Source FROM LightningComponentResource WHERE LightningComponentBundle.DeveloperName = '<name>' AND FilePath LIKE '%.html'" --use-tooling-api --target-org <alias> --json
```

If the deployed source is already correct but the browser still renders the old version, this is **not a normal browser cache** — `Ctrl+Shift+R`, devtools "disable cache", and even manually clearing `localStorage`/`sessionStorage` do not fix it. The `auraCmpDef` endpoint serves LWC bundles with `Cache-Control: private, max-age=31536000, immutable`, keyed by a server-side "last recompile marker" (`_lrmc` query param, visible on the bootstrap `*.app?...` request). Redeploying the LWC bundle alone does **not** bump that marker, and the browser's own persistent Aura definition storage (IndexedDB-backed, survives normal reloads) keeps serving the old bundle indefinitely — sometimes without even issuing a new network request for it at all.

**Fix:** make a genuine content change (not a no-op redeploy of identical bytes — that won't bump the marker) to the Aura app/component that declares the `<aura:dependency>` for the changed LWC, then redeploy just that bundle:

```bash
# e.g. add/remove a comment line in the .app file — must be an actual byte diff
cci task run deploy --path force-app/main/default/aura/<appName> --org <alias>
```

**Verify the fix landed** by checking browser network requests for a fresh `auraCmpDef` GET during page load — if that request is missing entirely, the browser served the definition from IndexedDB without even hitting the network, confirming it's still stale and the recompile marker still hasn't moved.

---

## CumulusCI "Success" Doesn't Mean Components Deployed (Common Gotcha)

`cci task run deploy` (or any task wrapping `cumulusci.tasks.salesforce.Deploy`) can print `[Success]: Succeeded` while deploying **zero actual components**. Seen most often with `unpackaged/config/*`-style folders that carry their own standalone `package.xml` (not part of `force-app`'s SFDX source tree). Two specific causes confirmed in the wild:

- **Wildcard members don't reliably work for deploy.** `<members>*</members>` in `package.xml` is fine for retrieve, but for at least some metadata types (confirmed with `CspTrustedSite`) the Metadata API silently resolves it to zero components on deploy — no error, just nothing happens. Use explicit `<members>Name</members>` entries per component instead.
- **Raw MDAPI folder structure is easy to get wrong when a repo mixes it with SFDX conventions.** A folder with its own `package.xml` is raw MDAPI format: files must **not** use the SFDX `-meta.xml` suffix, and they must live in a subfolder named after the metadata type's folder name (e.g. `cspTrustedSites/Name.cspTrustedSite`), one level *below* `package.xml` — not flat beside it. Files that look right at a glance (correct extension, sensible name) can silently fail to deploy if this nesting is off.

**If a config change doesn't seem to take effect after a "successful" deploy, don't trust the log line — verify directly:**
```bash
sf project deploy start --metadata-dir <folder> --target-org <alias> --json | jq '.result.numberComponentsDeployed, .result.numberComponentsTotal, .result.files'
```
A deploy of 0 real components still reports overall success (`"status": "Succeeded"`). Also worth a direct SOQL/Tooling check that the record(s) actually exist in the org, rather than trusting the deploy log alone.

---

## CSP Trusted Sites for LWC/Experience Cloud `fetch()` Calls (Common Gotcha)

If an LWC on an Experience Cloud (Digital Experience) site calls `fetch()`/`XMLHttpRequest` to an external API, it will be blocked by the site's Content Security Policy unless a `CspTrustedSite` record exists for that domain **with `isApplicableToConnectSrc` set to `true`**. An existing trusted site added for something else (e.g. an `<img>` tag pulling from GitHub) typically only has `isApplicableToImgSrc` set — that does **not** cover `fetch()`, and the request will fail with a CSP `connect-src` violation even though the domain is technically "trusted" for other purposes.

Visualforce pages are **not** subject to this same enforced CSP model — a script that works fine embedded in a VF page can still get CSP-blocked once ported into an LWC on the same org. Don't assume "it worked on the VF page" means the domain is already cleared for `connect-src`.

**After deploying a new/updated `CspTrustedSite`, the Experience Cloud site's published CSP header does not update until the site is republished** — deploying the metadata alone is not enough:
```bash
cci task run create_community_publish --org <alias>   # or the project's equivalent PublishCommunity task
```
Publishing is asynchronous (tens of seconds typically); give it a moment and use a fresh/isolated browser context or hard-reload before retesting, to rule out client-side caching as a red herring for what's actually a server-side propagation delay.

---

## `lightning-input-address` Supports Dependent Picklists (Don't Assume Plain-Text-Only)

This base component looks like a fixed 5-field, free-text-only composite, but it actually supports:
- `country-options` / `province-options`: arrays of `{label, value}` — supplying them turns those two fields into real dropdowns.
- `show-compact-address`: splits the single street textarea into two real inputs, `street` and `subpremise` (label them via `street-label`/`subpremise-label`).

**Gotcha:** the free-text fallback for `province`/`country` only triggers when the corresponding `-options` attribute is genuinely `undefined` — binding an **empty array** (`[]`) is still treated as "options provided" and leaves the field stuck as an empty, option-less dropdown. A reactive getter feeding this attribute must return `undefined`, not `[]`, for the "no options for this selection" case.

Before assuming any Lightning base component "can't" do something an address/picklist-style form needs, check the official component reference's Example tab first — it often already covers exactly this kind of case, and rewriting it as several individual fields is usually unnecessary extra work.

---

## Verify Live LWC State via the DOM, Not Just the Accessibility Snapshot (chrome-devtools-mcp)

When live-testing an LWC's data binding (not just its visual rendering) via chrome-devtools-mcp, `take_snapshot`'s accessible-name text for a combobox/input can look blank or stale for reasons unrelated to the underlying bound value — don't assume "looks blank" means "just a rendering/focus quirk." Read the actual component property directly instead, piercing shadow DOM as needed:

```js
function deepQueryAll(root, selector, results = []) {
    root.querySelectorAll(selector).forEach(el => results.push(el));
    root.querySelectorAll('*').forEach(el => {
        if (el.shadowRoot) deepQueryAll(el.shadowRoot, selector, results);
    });
    return results;
}
// e.g. deepQueryAll(document, 'lightning-input-address')[0].country
```

If the property value doesn't match anything in the options list backing a combobox, that mismatch is very likely the real bug (e.g. a value in the wrong format — a country code where the picklist expects a full name, or vice versa) — not a UI timing artifact. Confirm against the actual bound property before writing off unusual-looking rendering as cosmetic.

---

## Linux / Bazzite Notes

- **Line endings are LF (`\n`) natively** — no CRLF concerns when editing locally. If files originated on Windows, run `dos2unix <file>` before processing to strip stray `\r` characters.
- **Never assert on file size or line count** as a proxy for "did the deploy change anything" — diff content directly instead.
- **Bazzite uses an immutable base (`rpm-ostree`)** — system-level package installs require `rpm-ostree install <pkg>` followed by a reboot, or use `toolbox`/`distrobox` for a mutable environment. Prefer user-space installs (`pip`, `npm`, `flatpak`) when possible to avoid reboots.
- **Environment variables** use standard bash syntax: `export MY_VAR=value` to set, `$MY_VAR` or `${MY_VAR}` to reference.
- **Paths use forward slashes** (`/home/user/project`), not backslashes.
- **Permissions:** use `chmod`/`chown` for file permissions; scripts that need to be executable need `chmod +x <script>`.
