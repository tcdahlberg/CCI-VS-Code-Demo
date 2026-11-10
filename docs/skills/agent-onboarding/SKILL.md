---
name: agent-onboarding
description: "Baseline rules for AI agents doing Salesforce development in this workspace: security, code style, how to edit and verify work, the ai-logs/ convention, and an index of reference files (Apex, LWC, Flow, metadata/deploy, Experience Cloud, and per-OS notes). Read at the start of every session or task, before taking any action."
metadata:
  author: UST-Agentic-Coding-Skills
  version: "3.0"
allowed-tools: Read
---

# Agent Onboarding

Activate at the start of every session or task. This file holds the rules that apply to all work. Situational knowledge lives in `references/` and is loaded only when the task needs it (see "Reference Index" at the end).

---

## Security Rules (Non-Negotiable)

- **NEVER store passwords, API keys, tokens, secrets or credentials** in any log, output file, or file tracked by git or written to the project directory.
  - ❌ Writing a credential to `ai-logs/`, the project root, `docs/`, or any tracked path.
  - ❌ Echoing a credential to a command whose output goes to a file (`| tee`, `> file.txt`, `| Out-File`).
  - ✅ Reference credentials through environment variables only (syntax per OS in `references/os-bazzite.md` or `references/os-windows.md`).
  - ✅ If a command needs a credential, pass it inline without saving it anywhere.
- **`ai-logs/` is gitignored but NOT safe for secrets.** Treat it like any tracked path; another tool or an accidental `git add -f` can expose it.
- **If a task needs a credential, ask the user to set it as an environment variable first.** Never ask them to type it into a file or a command that saves output.
- **Never print credential values** in explanations, code comments or chat responses.

---

## Environment

- **Build and deploy:** CumulusCI (`cci`) and Salesforce CLI (`sf`). Org type: scratch orgs. Config: `cumulusci.yml`, `sfdx-project.json`.
- **IDE:** IntelliJ IDEA with the Illuminated Cloud plugin (VS Code is secondary). The IDE exposes an MCP server (see "Using the IDE MCP").
- **OS:** the user works mostly on Bazzite Linux (bash) and sometimes on Windows (PowerShell). Detect it (`uname -s`, or `$env:OS` on Windows) and read the matching file once: `references/os-bazzite.md` or `references/os-windows.md`. Never mix the two shells' syntax.
- **Skills are symlinks** (`~/.kiro/skills/<name>`, and the `~/.copilot/skills/` and `~/.agents/skills/` equivalents) into the maintained skills repo, `CCI-VS-Code-Demo/docs/skills/`. An edit to a skill lands at the symlink target. A write tool that reports a different resolved path is that symlink working, not an anomaly; check with `readlink -f` once if needed.

### Tool choices

| Task | Use | Avoid |
|---|---|---|
| Org management | `cci`, `sf` | Manual CLI-only workflows |
| Search file contents or names | `rg`, `fd` | `grep`/`find` loops, shell-specific globbing |
| Replace text in a file | the `write` tool's `strReplace`; `sd` for simple scripted replaces | Python heredocs for plain replacements |
| JSON, YAML, XML | `jq`, `yq` (`yq -p xml` for metadata), `xmllint --xpath` | Hand-rolled parsing scripts |
| CSS purging | PurgeCSS (`node run-purgecss.js` where a project provides it) | Manual regex extraction |
| Browser and visual testing | Playwright / chrome-devtools MCP, BackstopJS | Ad-hoc screenshot scripts |
| Code statistics | `tokei` | Manual line counting |
| GitHub | `gh` | The browser for routine PR and issue work |
| Structural code search/rewrite | the `code` tool (`pattern_search`, `pattern_rewrite`) | Regex over source |
| Apex formatting | `prettier` + `prettier-plugin-apex` only where a project has set it up (see its `package.json`) | Running it over existing files unasked: it rewrites whole files and hides the real change |

Prefer tools already installed. If something not installed would clearly help, say so and let the user install it; never install tools yourself.

---

## Starting a Task

1. Confirm a Salesforce project (`cumulusci.yml` or `sfdx-project.json`).
2. Read `docs/AI-TOOLS-CONFIG.md` and the repo's `AGENTS.md` for project context (org aliases, object names, URLs, permissions on shared orgs).
3. Read the reference files for the kind of work (index at the end).
4. Keep the project root clean: generated output goes in `ai-logs/`; essential config only in the root. Remove failed attempts and temp files.
5. Documentation discipline: update an existing doc instead of adding one; archive stale docs to `docs/archive/`; keep 5 to 10 core guides with clear names (`MAIN_GUIDE.md`, `TROUBLESHOOTING.md`).

---

## Code Style

### Naming: full, unabbreviated camelCase
Variables, methods, parameters and constants are written in full camelCase. Constants and static final collections are camelCase too (`splitAddressQuestions`), not `UPPER_CASE`. Check neighbouring declarations first.

| ❌ Avoid | ✅ Use |
|---|---|
| `getPmtHist()` | `getPaymentHistory()` |
| `calcEngScore()` | `calculateEngagementScore()` |
| `usr`, `cfg`, `evt`, `btn` | `user`, `configuration`, `event`, `button` |
| `idx`, `cnt`, `mgr`, `svc`, `(a, b) =>` | `index`, `count`, `manager`, `service`, `(first, second) =>` |

**Allowed abbreviations:** Salesforce API and metadata names (`@api`, `@wire`, `AuraEnabled`, field API names like `Account__c`); framework-required names (`connectedCallback`, `lwc:if`); loop counters `i` and `j` in tight indexed loops; acronyms that are the word itself (`url`, `id`, `dto`, `html`, `css`, `json`, `api`); `SObject`, `DML`, `SOQL` in comments.

### Control flow: always use braces
Every `if`, `else`, `for`, `while` and `for...of` has curly braces, even for a one-line body, in Apex, JavaScript and TypeScript.

```apex
// ❌
if (condition) doSomething();
// ✅
if (condition) { doSomething(); }
```

### Apex Javadoc must be complete
When a method has a `/** ... */` block, it needs every tag (the Illuminated Cloud linter enforces this): `@param <name> <description>` for each parameter and `@return <description>` for a non-void method. A method with no Javadoc block needs no tags.

```apex
/**
 * Loads the donor summary record.
 *
 * @param contactId The Id of the Contact to load
 * @return Map of summary data keyed by section name
 */
public static Map<String, Object> getSummary(Id contactId) { ... }
```

---

## Working Method

### Editing files
- **Use `write` `strReplace` for edits:** one call per change, independent changes in parallel. A Python heredoc is for a mechanical change across many files only, and then it must `assert s.count(old) == 1` before writing.
- **Everything in `oldStr` must be in `newStr`, or be meant to go.** Quoting a neighbouring line (`<template if:true={x}>`) and not repeating it deletes that line. After editing a template or nested markup, read the region back.
- **Do not run the tests in the same parallel batch as the edit they test.** They run against the old files and report a misleading result.
- **Match the file's existing conventions** (naming, structure, comment style) before using your own.

### Using the IDE MCP
Call it through `execute_tool` with `--paramName value` arguments. Confirmed: `get_file_problems --filePath <path>` (Illuminated Cloud inspections), `get_symbol_info --filePath <p> --line <n> --column <n>` (doc comment and signature), `search_text --q <text>` (project-wide; it also indexes `ai-logs/`, so expect noise), `get_run_configurations`, `get_project_modules`, `reformat_file --files ...` (check the IDE's code style first).
- **Run `get_file_problems` on every Apex file you changed before reporting done.** Known noise: "Constructor is not visible" for a private inner class with an implicit constructor, and `Cacheable != cacheable`. Fix anything else.
- **The `salesforcedx` MCP server is gone.** Salesforce ended support for it on 2026-11-02 and it is uninstalled here; Salesforce tooling now arrives as skills (the user runs `npx skills update -g -y` weekly). Do not depend on tools such as `run_apex_test`, `run_code_analyzer`, `run_soql_query`, `deploy_metadata` or `explore_lbc_components`, even if a stale session still lists them. Use the `sf` and `cci` CLIs, and load the matching skill: `dx-code-analyzer-run`, `platform-apex-test-run`, `platform-soql-query`, `platform-metadata-deploy`, `experience-lwc-base-components-integrate`.

### Definition of done for a Salesforce change
1. **Apex:** deploy (`cci task run deploy --path force-app/main/default/classes --org dev`, plus the test folder), then run **all** test classes, not only the new one. List them with `ls force-app/test/default/classes/*.cls` into `ai-logs/classes.txt`. Then `get_file_problems` on each changed class.
2. **LWC:** run the whole Jest folder (`npx jest force-app/main/default/lwc`). When a suite fails, run it alone, read the error, and check whether it also fails without your change before reporting it.
3. **Break the test on purpose** for every new behaviour test (next section).
4. **Run Code Analyzer** (`sf code-analyzer run`, see the `dx-code-analyzer-run` skill and the Code Analyzer section of `cumulus-ci`) on the changed files. For UI work, also run the review skills: `experience-accessibility-validate`, `design-systems-slds-validate` (it flags hard-coded colours such as `rgba(...)`), `experience-lwc-security-validate`, `experience-lwc-rtl-validate`.
5. **Look at it in a browser** for any user-visible change (Playwright or chrome-devtools MCP against the scratch org via `sf org open --url-only`). Jest stubs cannot prove visual or validation behaviour (`references/lwc.md`). If you could not look, say so.
6. **State what each check verified:** Apex tests against the scratch org, Jest against stubs, a regex against a real `RegExp`, and what was only reasoned about (real browser behaviour, `lightning-input` validation, guest-user access). Name the reasoned-about parts; that is where the next bug is.

### A passing test proves nothing until it fails without the fix
Before calling a fix, ordering rule or layout rule done, **break it on purpose and run the test**: remove the guard, swap the two blocks, drop the grant. The test must fail with a clear message; restore the file and watch it pass. Tests that pass either way (guards on behaviour the fix must preserve) are fine, but say so. This has shown twice that a test did not test what its name claimed. Copy the real file first (`cp file ai-logs/x.bak`) and restore it in the same command so a mutation is never left behind.

When a Salesforce error is vague, **measure instead of guessing.** Write a throwaway `@IsTest` probe named `ZZ_Probe_TEST` that returns evidence in an assertion message (`UserRecordAccess`, `Security.stripInaccessible` removed fields, `Database.update(rec, false)` error codes), then delete it locally and from the org. It has found a root cause in two runs that would have taken a dozen guesses.

---

## AI Working Files (`ai-logs/`)

- **All AI-generated output goes in `ai-logs/`:** deploy logs, test results, JSON, jq filters, helper scripts, backups, browser screenshots and traces. It is gitignored. If it does not exist, create it and confirm it is ignored before writing.
- **Reuse standard filenames** for the same purpose: `deploy.txt` (CCI/SF deploy output), `test.txt` (CCI test run), `sftest.txt` (`sf apex run test`), `cov.json` (coverage query), `cov_filter.jq`, `classes.txt` (test class list), `screenshots/` (browser output).
- Browser automation tools (Playwright MCP, chrome-devtools-mcp) write screenshots and `.playwright-mcp/` traces to the project root by default. Pass `ai-logs/screenshots/<name>.png` explicitly and gitignore `.playwright-mcp/`.
- Never leave temp files in the project root.

---

## Reference Index

Read the file when the task matches. Each reference starts with its own "read this when" line.

| Task involves | Read |
|---|---|
| Apex callouts, triggers, inner classes, sparse DML | `references/apex.md` |
| LWC, Lightning pages, Lightning Out, CSP, `lightning-input-address`, `record-edit-form`, Jest or browser testing, forms that follow field limits | `references/lwc.md` |
| Flow, Flow formulas, Decisions, fault paths | `references/flow.md` |
| A deploy that "succeeded" but did nothing, `SourceConflictError`, field/permission set/picklist metadata | `references/metadata-and-deploy.md` |
| Portal or guest users, Sharing Sets, record access | `references/experience-cloud.md` |
| Shell syntax, paths, environment variables, installed tools, line endings, package installs | `references/os-bazzite.md` or `references/os-windows.md` |
| Jira, `acli`, sprints | the `jira-crm-workflow` skill |
| `cci`/`sf` deploy and test patterns, Code Analyzer | the `cumulus-ci` and `terminal-commands` skills |
