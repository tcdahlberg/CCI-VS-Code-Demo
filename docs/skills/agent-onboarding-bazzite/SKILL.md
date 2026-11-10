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

## Skills Are Symlinks Into a Maintained Repo — Don't Re-Investigate This

`~/.kiro/skills/<skill-name>` (and the equivalent `~/.copilot/skills/`, `~/.agents/skills/` paths) are **symlinks**, not real directories. Most point into `/var/home/tcdahlberg/IdeaProjects/CCI-VS-Code-Demo/docs/skills/` (this maintained skills repo); some point into `~/.agents/skills/` instead. When editing a skill file (e.g. via a write/edit tool), the edit lands at the **symlink target**, not the path you passed in — this is expected behavior, not a bug.

**If a write/edit tool reports a different resolved path than the one you requested, that's the symlink resolving correctly — don't stop to investigate it as an anomaly.** Just confirm the target with `readlink -f <path>` or `ls -la ~/.kiro/skills/` once if needed, then continue. This has already been root-caused; no need to rediscover it each session.

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

### Variant: stale Apex response *shape* (new field silently missing), not just stale JS bundle

The same staleness can affect an `@AuraEnabled` **Apex controller method's response payload**, not just the LWC's own HTML/JS. Symptom: you add a new property to an Apex DTO class (e.g. a new `@AuraEnabled public String` field), redeploy, and the LWC that calls that method silently receives objects **missing the new key entirely** (not `null` — `JSON.stringify` drops it, so it just isn't there), even though:
- A direct `sf apex run` (anonymous Apex) call to the *exact same public static method* correctly returns the new field — proving the Apex logic and deployed class are correct.
- The method's `@AuraEnabled(Cacheable=true)` was changed to `Cacheable=false` and redeployed.
- The page was retested in a **brand-new isolated browser context** (fresh login, never hit this org before) — ruling out both normal browser cache and per-user LDS client cache.

This points to the same `_lrmc`-style server-side recompile-marker staleness described above, just scoped to the Apex-side action descriptor/schema cache instead of the LWC bundle cache — a plain redeploy of the Apex class (or even the LWC bundle) doesn't reliably bump it. Confirmed unresolved as of this writing; the working theory (untested) is that it needs the same fix as the JS-bundle case: a genuine content change to the Aura app that declares the component's dependency, redeployed alongside the Apex change. If you hit this, try that first before assuming your Apex or LWC code is wrong — don't burn many cycles re-verifying logic that direct anonymous-Apex testing already proved correct.

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

## `lightning-record-edit-form` Fires `load` Again After Every Edit That Changes Its Field List (Silent State Reset)

If a custom LWC wraps `lightning-record-edit-form` and copies the record into component state inside its `onload` handler, **do that copy once per record**. The form fires `load` again whenever its field list changes, and each repeat reports the *saved* values, not the user's edits. The classic trigger: a "type" picklist that reveals extra `lightning-input-field`s. Choosing a new type adds fields, the form re-fetches, `load` fires 2 to 3 more times, and the handler overwrites the component's state with the old type. The picklist still *shows* the new choice but the sections it controls snap back, with no error anywhere.

Fix: remember which record was initialised (`_initialisedRecordId`) and return early on repeat loads; keep "saved" copies up to date in the success handler so Reset still works (include the type field itself in Reset).

**Why tests missed it:** Jest fires one synthetic `load`, and the first browser check opened a record that was already the new type. **Always test the change path in a real browser** (open a record of type A, switch to B, watch the sections), not just the load path, and add a Jest test that fires `load` again after the change. To see what the form actually emits, add listeners from `page.evaluate` (pierce shadow roots) and log `load` and `change` with timestamps.

---

## A Passing Test Proves Nothing Until It Fails Without the Fix

Before calling a bug fix or an ordering/layout rule done, **break the fix on purpose and run the test**: remove the guard, swap the two blocks, drop the grant. The new test must fail with a clear message, then restore the file and watch it pass. Tests that "pass either way" (guards on behaviour the fix must preserve) are fine, but say so. This caught nothing wrong this time only because it was done; twice it showed a test was not testing what its name claimed. Save the real file first (`cp file ai-logs/x.bak`) and restore it in the same command so a mutation never gets left behind.

Companion habit: **when a Salesforce error is vague, measure instead of guessing.** Write a throwaway `@IsTest` probe (name it `ZZ_Probe_TEST`, delete it afterwards, and delete it from the org too) that returns the evidence in an assertion message: `UserRecordAccess`, `Security.stripInaccessible` removed fields, `Database.update(rec, false)` error codes. It found a root cause in two runs that would have taken a dozen guesses.

---

## Experience Cloud Portal Users: What a Permission Set Can and Cannot Grant

- **A permission set grants object, field, Apex-class, and record-type access. It never grants access to a specific record.** If a signed-in user can read a record but a save fails with `INSUFFICIENT_ACCESS_ON_CROSS_REFERENCE_ENTITY`, the cause is record sharing, not FLS, and `without sharing` on the controller does not help (DML still needs edit access). Confirm with `SELECT HasReadAccess, HasEditAccess FROM UserRecordAccess WHERE UserId = ... AND RecordId = ...`.
- **To let a portal user edit records tied to their own Contact, use a Sharing Set** (`SharingSet` metadata: `accessLevel Edit`, `object`, `objectField` such as `Contact__c`, `userField Contact`, and `profiles`). It is attached to **profiles, not permission sets**, and deploys as raw MDAPI (`sharingSets/<Name>.sharingSet`, no `-meta.xml`), after the site exists.
- **"No such column X on entity Y" inside a dynamic `Database.query` can mean the user has no field-level access, not that the field is missing.** Check FLS first. Dynamic queries name fields in several string pieces (SELECT and every WHERE branch), so extract them all at once instead of fixing one per test run.
- **A dynamic `Database.query` with no access clause can fail with "No such column" for a portal user even when the code is meant to be public.** Add `WITH SYSTEM_MODE` to the query string (or pass `AccessLevel.SYSTEM_MODE`); with it the same query works, measured with a probe. And **deploy the controller you are testing**: a test that fails against stale code in the org looks exactly like a real bug (the org still had the old `USER_MODE` class and the failure pointed at the wrong cause).
- **Never grant "all fields" on a directory-style object.** Formula fields often expose email, birthdate, phone, or street address. Derive the field list from the queries the controllers actually run (`WITH USER_MODE` ones need grants; `without sharing` ones with no USER_MODE only need class access) and leave required fields out (Salesforce rejects them).
- **Record types are visible per user, even in `SYSTEM_MODE`.** `getRecordTypeInfosByDeveloperName()` returns only record types the running user can see, so a public (system-mode) controller still returns nothing to a guest unless a permission set makes the record type visible. A System Administrator may not see a record type that is granted only by a permission set: give the test's setup user that permission set, or inserts fail with `INVALID_CROSS_REFERENCE_KEY`.
- **`System.runAs` in a test enforces object/field/record access, but NOT Apex class access.** Prove class grants by reading `SetupEntityAccess` for the permission set, and prove the rest by calling the controllers inside `runAs(portalUser)`.
- **A guest sharing rule needs the site's guest user to exist** ("Specify a guest user's nickname for the guestUser field" otherwise). See the CumulusCI skill for where to deploy it.

---

## Jira Work: See the `jira-crm-workflow` Skill

Jira process (inactive-sprint workflow), `acli` usage patterns, and the MCP markdown-comment bug below are all consolidated in the dedicated `jira-crm-workflow` skill — load that instead of duplicating it here.

## Jira MCP `addCommentToJiraIssue`: Markdown Conversion Bug (Common Gotcha)

The Atlassian MCP server's `addCommentToJiraIssue` tool, called with `contentFormat: "markdown"`, does **not** reliably convert markdown newlines into real ADF paragraph breaks. A `commentBody` string containing `\n\n` between paragraphs can get stored as the **literal two-character sequence** `\n` (backslash + `n`) inside the comment body — visible as raw `\n` text in the rendered Jira comment, not an actual line break. This is a distinct bug from any escaping mistake in the tool call itself: sending correctly-escaped JSON (a real `\n` that decodes to one newline character) still produced a comment with literal backslash-n text when using markdown format.

**`editJiraIssue`'s `description` field with the same `contentFormat: "markdown"` did not exhibit this bug** — multi-paragraph markdown (including tables, bullet lists, bold) converted to ADF correctly there. The bug appears specific to the comment-creation code path.

**Fix:** for `addCommentToJiraIssue`, skip markdown entirely and pass `contentFormat: "adf"` with a hand-built Atlassian Document Format JSON object (`{"type": "doc", "version": 1, "content": [...]}`), using `paragraph`, `bulletList`/`orderedList`, and `text` nodes with `marks` (`strong`, `code`) as needed. If a comment was already posted with the markdown bug, fix it in place by re-calling `addCommentToJiraIssue` with the same `commentId` and a proper ADF body — it updates the existing comment rather than creating a new one.

**Separately:** some Jira custom fields of type "textarea" (e.g. a "Deploy Notes" field) reject ADF `text` nodes that combine two marks on one run (e.g. `strong` + `code` together) or contain an em dash (`—`), failing with a generic `INVALID_INPUT` error with no field-level detail. If a `customfield_*` edit fails with `INVALID_INPUT` and no useful diagnostic, bisect the ADF content down to a single-paragraph, single-mark test payload first rather than assuming the field rejects rich text entirely — plain paragraphs with one mark per text run, plain hyphens instead of em dashes, generally work.

---

## Apex Callout-After-DML Restriction (Common Gotcha — Silent, Hard to Diagnose)

Apex forbids making an HTTP callout (`Http.send()`, any `@RestResource`/external service call) in the same transaction as **uncommitted prior DML** — fails with `System.CalloutException: You have uncommitted work pending. Please commit or rollback before calling out`. This is easy to trigger without realizing it: any controller method that does `insert`/`update`/`delete` and THEN tries a callout later in the same method (or in a chain of calls within the same transaction) will hit this — including from inside a trigger that fired off DML that's still uncommitted in the calling transaction.

**Why this is dangerous:** if the callout is wrapped in a graceful-degradation try/catch (a good practice for feature flags, optional enrichment, third-party APIs that might be down, etc.), this exception gets silently swallowed and downgraded to "unavailable"/"skipped" — the feature will look intermittently or *permanently* broken with zero visible errors anywhere. This can look identical to other root causes (missing Remote Site Setting/CSP Trusted Site, bad API key, endpoint down) that also get caught by the same try/catch. **Add a diagnostic detail field/log line inside the catch block during development** so the actual exception message is visible somewhere (custom object field, debug log, etc.) — don't rely on "it's Unavailable" to tell you *why*.

**Fix:** defer the callout-dependent work to `@future(callout=true)` (or Queueable), which runs in a brand-new transaction after the current one commits — the standard, documented Salesforce pattern for "I need to do DML now AND a callout for the same logical operation."

**Critical follow-on gotcha:** `@future` methods only accept **primitive types and collections of primitives** as parameters — no sObjects, no custom Apex types. Restructure the call site to pass `Id`/`String`/`Decimal` etc. rather than a wrapper object.

**Second critical follow-on gotcha:** request context (`ApexPages.currentPage()`, `RestContext.request`, etc.) is **NOT available inside the future method itself** — the future transaction runs later, outside any HTTP request. If the future method needs header/request data (IP, User-Agent, Referer, etc.), capture it **synchronously in the original context** (the controller method, or the trigger, before dispatching) and pass it in as primitive parameters — do not try to re-read it from inside the `@future` method. This is easy to get backwards on a first attempt; if a trigger dispatches to a future method, capture request data in the trigger, not the future method.

---

## Apex Inner Classes Cannot Have Static Factory Methods

`public static MyInner build(...)` on a class nested inside another class fails to compile with `"static can only be used on methods of a top level type"`. Use a public constructor instead (`new MyInner(...)`) and update all call sites accordingly — there is no static-factory workaround for inner classes in Apex.

---

## Required Lookup Fields Need `deleteConstraint: Restrict` or `Cascade`, Never `SetNull`

A `<required>true</required>` Lookup field must specify `<deleteConstraint>Restrict</deleteConstraint>` or `Cascade` in its field metadata — `SetNull` is rejected at deploy time for any lookup marked required, since setting it to null on parent delete would violate the required constraint. This is a common mistake when quickly scaffolding a new object's lookup fields from a template that defaults to `SetNull`.

---

## PermissionSet Metadata: Two More Structural Rules Beyond FLS Basics

- **Salesforce rejects `fieldPermissions` entries for required fields entirely** (not just ignores them) — deploy fails with `"You cannot deploy to a required field."` Before writing permission set XML for a new object, scan all its `<required>true</required>` fields and exclude them from `fieldPermissions` — they're implicitly always accessible to anyone who can create/edit the record at all.
- **All `fieldPermissions` elements must be grouped together, and all `objectPermissions` elements must be grouped together** — interspersing them (e.g. `fieldPermissions, objectPermissions, fieldPermissions`) causes an XML schema validation error (`"Element X is duplicated at this location"`), since the schema expects each element type in one contiguous block.
- **`<description>` is limited to 255 characters** and the deploy fails with "data value too large". Put the explanation in the project docs, not the metadata.
- A picklist's **blank "--None--" choice stores an empty value**. If code quietly turns blank into a default, users who pick "--None--" get the default and think the control is broken. Decide deliberately what blank means, make every layer (Apex, component, editor preview) agree, and say it in the field's help text. New records still get the picklist's `default="true"` value; **an `@IsTest` record built without setting the field also gets that default**, so to test "blank" set it to `null` explicitly and re-query.

---

## Sparse `update new SObject(Id = ..., field = ...)` DML Means Trigger Context Is Missing Other Fields

If application code updates a record via `update new Application__c(Id = someId, Status__c = 'X')` (a very common pattern for narrow, single-field status updates), **`Trigger.new`/`Trigger.oldMap` in any trigger on that object will only have `Id` and `Status__c` populated** — every other field, including lookups needed for trigger logic, will be `null` even if they have real values in the database. A trigger that needs those other fields (e.g. to look up a related record's configuration) **must re-query** the affected Ids itself rather than trusting `Trigger.new`. Don't assume a trigger's `Trigger.new` reflects the full current state of the record — it only reflects what the calling DML statement actually included.

---

## `sf project deploy start` / `cci task run deploy` "Succeeded" Doesn't Always Mean It Landed

Beyond the 0-components case (see the CumulusCI "Success" doesn't mean deployed section above), a deploy can report success and genuinely include the component in its file list, yet a subsequent `describe`/Tooling API query can show a stale/incomplete result (e.g. a custom object reporting fewer fields than were actually deployed) — this appears to be an API-side caching/propagation lag distinct from the wildcard/MDAPI-structure issue. When in doubt after a "successful" deploy: verify via a direct Tooling API query (`SELECT Name FROM ApexClass`/`CustomField` etc.), but if that *also* disagrees with what a human can see in the org's own Setup UI, trust the human's direct visual confirmation over an ambiguous API describe result — the describe cache can lag behind reality in both directions.

---

## Flow Execution Order Between Independent Record-Triggered Flows Is NOT Guaranteed

If two separate record-triggered Flows fire on the same object and trigger event (e.g. both on `after-save Create` for the same object), Salesforce does **not** guarantee which one runs first unless you explicitly set an order via **Flow Trigger Explorer** in Setup. This is a UI/record-managed setting on the Flow itself in current API versions, not a simple XML element — attempting to hand-write `<triggerOrder>` inside a Flow's `<start>` metadata fails deploy with `Element ... triggerOrder invalid at this location in type FlowStart`.

**Practical implication:** if Flow B needs to read a field that Flow A sets (both firing on the same trigger event), do NOT assume Flow A ran first just because it was built/deployed first. Either:
- Explicitly configure the order via Flow Trigger Explorer (Setup UI) before relying on it, or
- Make the dependent Flow self-sufficient by recomputing the needed value itself (accepting some logic duplication) rather than depending on a sibling Flow's side effect within the same transaction.

## Flow Formula Gotchas: Picklists and Booleans

- **A picklist field used directly in a Boolean/string formula expression** (e.g. `{!$Record.SomeStatus__c} = "Active"`) fails deploy with `Field $Record is a picklist field. Picklist fields are only supported in certain functions.` Wrap it in `TEXT(...)` first: `TEXT({!$Record.SomeStatus__c}) = "Active"`.
- **`TEXT()` does not accept a Boolean argument** in Flow formulas — fails with `Incorrect parameter type for function 'TEXT()'. Expected Number, Date, Date/Time, Picklist, received Boolean.` Use `IF(booleanValue, "Yes", "No")` (or similar) instead of `TEXT(booleanValue)` when rendering a Boolean as text (e.g. inside an email body formula).

---

## Flow Decision Elements Silently Ending an Interview When `defaultConnector` Is Missing

A Flow `<decisions>` element can be deployed and pass validation with a `<defaultConnectorLabel>` present but **no actual `<defaultConnector>` element** — this is not caught by the deploy process. The practical effect: whenever none of the decision's explicit `<rules>` match (the "default"/"else" path), the Flow interview simply **ends at that point** with no error, no fault, and no indication anything is wrong. If that decision sits partway through a multi-step Flow, everything downstream of it silently never executes for that branch — and since this produces no error of any kind, it can go undetected for a long time, especially if the default/else path is the numerically-common case (e.g., "below a threshold," "no match found") rather than the rare/interesting case, since spot-testing during development often happens to exercise the "positive"/interesting branch and never notices the default path is broken.

**When reviewing or debugging a Flow with unexpected "some records get processed further, but not others" behavior**, check every `<decisions>` element for a genuine `<defaultConnector>` (not just a `<defaultConnectorLabel>`) — this can hide silently for a long time and invalidate previously-declared-working behavior for an entire class of records.

## Flow `faultConnector` on `emailSimple` Is Necessary But Not Sufficient

A `faultConnector` on an `emailSimple` action call in a record-triggered Flow can fail to catch certain classes of email-sending failure — the failure surfaces instead as an unhandled `CANNOT_EXECUTE_FLOW_TRIGGER` / "an unhandled fault has occurred in this flow" exception on the triggering DML, exactly as if no fault connector were present at all. Confirmed in the wild with the message `"Probably Limit Exceeded or 0 recipients"` (commonly caused by exhausting a scratch org's low daily email-recipient limit during heavy testing), and documented by multiple other practitioners hitting the identical error under similar conditions. This is a known Salesforce platform behavior, not a Flow-design mistake — correctly wiring `<faultConnector>` in the XML does not guarantee it will always be honored for this specific action type/failure class.

**Practical implication:** never rely on a Flow's own fault connector as the *only* safety net for a record-triggered Flow whose triggering DML matters to an end user. Instead, ensure the **calling Apex code's DML insert/update that actually triggers the Flow** is wrapped in its own `try/catch` — this is what actually stops the exception from propagating to whatever code path triggered the record change in the first place, independent of whether the Flow's internal fault handling catches it. Both layers are worth having: the Flow's own fault connector maximizes the chance a human is notified when something breaks inside the Flow; the calling code's `try/catch` is the actual guarantee that a real user-facing action is never blocked by a downstream Flow failure, regardless of which failure class it is or whether the Flow's fault path catches it.

**When testing Flow fault paths:** testing via a *synchronous, direct* insert/update in anonymous Apex can produce different (and less representative) results than testing through the *real* production code path, especially when the record-triggering DML is normally dispatched from an `@future`/Queueable context. A synchronous test can surface an uncaught exception that would never actually propagate to a real end user in production, because the real triggering DML happens in an isolated async transaction with its own `try/catch` around it. Always test fault-handling both ways: (1) directly, to characterize the Flow's own behavior in isolation, and (2) through the actual real-world call path, to confirm what a real user would or wouldn't experience.

---

## LWC `getRelatedListRecords` Requires the `__r` Suffix on Custom Child Relationship Names

`lightning/uiRelatedListApi`'s `getRelatedListRecords({ parentRecordId, relatedListId, ... })` needs the full relationship name **including the `__r` suffix** for a custom object's child relationship — e.g. `relatedListId: 'My_Children__r'`, not `'My_Children'`. This matches SOQL subquery syntax exactly (`SELECT ... FROM My_Children__r`), which is the fastest way to confirm the correct value: run the equivalent SOQL subquery via `sf data query` first (`Didn't understand relationship 'X' in FROM part of query call` means you're missing `__r`), then use that exact string as `relatedListId`. Passing the bare relationship name (without `__r`) doesn't throw a clear error in the wire adapter — it just silently returns no data, which is easy to misdiagnose as a permissions or field-visibility issue rather than a wrong-relationship-name typo.

---

- **Line endings are LF (`\n`) natively** — no CRLF concerns when editing locally. If files originated on Windows, run `dos2unix <file>` (installed via brew) to strip stray `\r`; `sed -i 's/\r$//' <file>` also works.
- **Never assert on file size or line count** as a proxy for "did the deploy change anything" — diff content directly instead.
- **Bazzite uses an immutable base (`rpm-ostree`)** — system-level package installs require `rpm-ostree install <pkg>` followed by a reboot, or use `toolbox`/`distrobox` for a mutable environment. Prefer user-space installs (`pip`, `npm`, `flatpak`) when possible to avoid reboots.
- **Environment variables** use standard bash syntax: `export MY_VAR=value` to set, `$MY_VAR` or `${MY_VAR}` to reference.
- **Paths use forward slashes** (`/home/user/project`), not backslashes.
- **Permissions:** use `chmod`/`chown` for file permissions; scripts that need to be executable need `chmod +x <script>`.

---

## LWC / Lightning Page Gotchas Learned On UST-EASY-LWC

- **`slds-card` on a plain `div` can lose your border and shadow inside a Lightning page tab.** The card looked fine in a
  `lightning-card`, then vanished once the component sat in a `flexipage:tabset`. Debug trick that found it in minutes:
  remove the class in the browser and re-read `getComputedStyle`; if your rule suddenly applies, that class is the culprit.
  Give the element your own class and put the border, radius and shadow in your CSS.
- **Synthetic shadow DOM:** `shadowRoot.styleSheets` throws; read `document.styleSheets` (component CSS is placed in
  `<head>` with an `lwc-xxxx` attribute selector). Use a recursive `deep(root, selector)` helper to pierce shadow roots.
- **A tab set on an App page is metadata too:** `flexipage:tabset` points at `Facet` regions that hold `flexipage:tab`
  items, which point at more `Facet` regions holding components. People edit pages in the UI, so the repo copy goes
  stale; retrieve the page before touching it.
- **Playwright against Lightning after a deploy:** clear IndexedDB, localStorage, sessionStorage and Cache storage,
  load the page, wait about 25 seconds, and reload once (Lightning re-bootstraps after storage is cleared). Odd
  "state resets by itself" results right after clearing storage are usually that re-bootstrap, not your code. Measure
  before guessing: bounding boxes and computed styles settled two alignment questions in one call each.
- **Multi-select picklists:** Salesforce allows only one default value (the deploy fails with "Multiple default values"),
  and a default never backfills existing records. Design "blank means all" and say so in the field help text. A new field
  is invisible without field-level security, so ship a permission set; the admin also has to add it to the page layout.
  Read admin-set configuration in Apex with `WITH SYSTEM_MODE` (applicant data stays in user mode) so reviewers do not
  need field access. Apex that describes a new field will not compile until the field is deployed first.
- **Two components that need the same record on selection** (for example a read-only card and an edit form): share one
  in-flight request in a utility module (a `Map` of promises keyed by id, cleared in `finally`) so they cannot both
  insert it. Also ignore a slow response if a newer selection has replaced it.
- **`sf project deploy start` `SourceConflictError` is not always a real conflict.** Retrieve the org copy to a folder
  inside the project (`--output-dir` outside the project is rejected), diff it against what you last deployed, and only
  then use `--ignore-conflicts`. If the org copy differs in ways you did not make (a UI-edited flexipage), stop and
  retrieve it instead.

---

## How To Work: Editing, Checking, Tools (Lessons From An RFI-Form Session)

### Editing files
- **Use the `write` tool's `strReplace` for edits.** One call per change; send independent changes in parallel. Do not reach for a Python heredoc just to replace text. A heredoc is only for a mechanical change across many files, and then it must `assert s.count(old) == 1` before writing.
- **Everything in `oldStr` must be in `newStr`, or be meant to go.** A replace that quoted a neighbouring line (`<template if:true={x}>`) and did not repeat it silently deleted that opening tag. After any edit to a template or nested markup, read the region back.
- **Do not put the test run in the same parallel batch as the edit it tests.** The tests ran against the old files and reported a misleading failure.
- **Match the file's naming, not your habit.** Check neighbouring constants and helpers first. In the RFI Apex classes constants are camelCase (`splitAddressQuestions`), and the style table above applies; `UPPER_CASE` constants and `(a, b) =>` lambda names break it.

### The IDE (IDEA / Illuminated Cloud) MCP: use it, it is cheap
Call it through `execute_tool` with `--paramName value` arguments. Confirmed to work:
- `get_file_problems --filePath <path>`: runs Illuminated Cloud's inspections on one file (Javadoc tags, unused code, type errors). **Run it on every Apex file you changed before reporting done.** Known noise: "Constructor is not visible" for a private inner class with an implicit constructor, and `Cacheable != cacheable`; ignore those, fix anything else.
- `get_symbol_info --filePath <p> --line <n> --column <n>`: the doc comment and signature of a symbol, without opening the file.
- `search_text --q <text>`: project-wide text search with line and column. It also indexes `ai-logs/`, so expect noise there.
- `get_run_configurations`, `get_project_modules`: what the IDE can run.
- `reformat_file --files ...`: needs a `files` argument; check the IDE's code style before using it on files you did not write.
Prefer these over `grep`/`sed` pipelines when you want references, problems or documentation. Prefer the `salesforcedx` MCP tools (`run_apex_test`, `run_code_analyzer`, `run_soql_query`, `deploy_metadata`) when you want structured results rather than a `tee` + `grep` of CLI text.

### Checking a Salesforce change before saying "done"
1. **Apex**: deploy (`cci task run deploy --path force-app/main/default/classes --org dev`, and `--path force-app/test/default/classes`), then run *all* test classes, not only the new one (`ls force-app/test/default/classes/*.cls` into `ai-logs/classes.txt`, then `sf apex run test --class-names "$(cat ai-logs/classes.txt)" ...`). Then `get_file_problems` on each changed class.
2. **LWC**: `npx jest force-app/main/default/lwc` for the whole folder. If a suite fails, run it alone and read the error; check whether it also fails without your change before reporting it.
3. **Break the test on purpose** (see "A Passing Test Proves Nothing" above) for every new behaviour test, copying the file to `ai-logs/` first.
4. **Run Code Analyzer** (`run_code_analyzer` on the changed files) and, for UI work, the matching review skills: `experience-accessibility-validate`, `design-systems-slds-validate` (it flags hard-coded colours such as an overlay's `rgba(...)`), `experience-lwc-security-validate`, `experience-lwc-rtl-validate`. These were available and not used; a UI change is not finished without them.
5. **Look at it in a browser** for any user-visible change (Playwright or chrome-devtools MCP, against the scratch org via `sf org open --url-only`). Jest cannot prove visual or validation behaviour (next section). If you cannot look, say so plainly.

### What Jest stubs for `lightning-*` do NOT prove
- Stubs hold the properties you set (`pattern`, `maxlength`, `min`, `step`) and do nothing with them. They do not validate, and `checkValidity()` returns nothing, so tests must define it (see the helpers in `rfiForm.submit.test.js`). A test that "the pattern is set" is not a test that the pattern works.
- Test a `pattern` string with a real `RegExp` using the **`v` flag** (browsers compile the `pattern` attribute with `v`, which is stricter about `( ) [ ] { } - \` inside a class), anchored as `^(?:pattern)$`.
- Properties on a stub are not reflected as attributes, so `querySelector('lightning-combobox[name="x"]')` finds nothing. Select by tag and read the property, or find by `label`.
- Custom elements do not show real layout, so sticky positioning, overlays and gray-outs need a browser.

### Deriving form behaviour from the Salesforce field, not from memory
- Read limits from `Schema.DescribeFieldResult`: `getType()` (number, date, picklist), `getLength()` (text), `getPrecision()` and `getScale()` (numbers). That one source then covers every question mapped to the field, including custom questions, with no per-question code. Keep a short explicit list only for what the type cannot say (a Text field that holds a four-digit year).
- Send values in the type the server expects: a number box returns text, so convert to a number; a Date field needs `YYYY-MM-DD`; a Text field that holds digits stays text.
- A restricted picklist behind an admin tool (the Question Manager reads `Fields_to_Display__c` values) cannot be reshaped from a scratch org without a metadata change that admins own. Filter and add in the component and ship the list from Apex instead of editing the picklist.
- `lightning-input-address` with `show-compact-address` draws the street as an 80-character one-line input plus a separate `subpremise` line. If the target is a TextArea (such as `Mailing_Street__c`), join the two with `\n`; the change event carries `subpremise`.
- `developer.salesforce.com` component pages return 403 to `web_fetch`. Use `explore_lbc_components` for base component APIs and the `platform-docs-get` skill for other Salesforce docs.

### Terminal tools: installed, missing, and traps
- **Installed and worth using more:** `rg`, `fd`, `jq`, `bat`, `delta`, `just`, `xmllint` (read metadata XML with `xmllint --xpath`), `sqlite3`, `gh`, plus (installed via brew, 2026-11) `yq` v4 (YAML, JSON and XML: `yq -p xml -o xml` for metadata, plain `yq` for `cumulusci.yml` and Snowfakery recipes), `sd` (find and replace with plain syntax; use it instead of a Python script for simple replacements), `dos2unix`, and `prettier` 3 (global).
- **`sg` is NOT ast-grep on this machine.** `/usr/bin/sg` is `newgrp`. Nothing is needed from the user: use the `code` tool's `pattern_search` / `pattern_rewrite` (ast-grep underneath) for structural changes.
- **`prettier` Apex setup (UST-Salesforce-RFI):** `prettier` and `prettier-plugin-apex` are devDependencies, `.prettierrc` applies the Apex parser only to `*.cls` and `*.trigger` (4 spaces, width 140), and `npm run format:apex:check` / `format:apex` run it. The plugin always puts `@AuraEnabled` and other annotations on their own line and cannot be told otherwise, so it would reformat about a third of `rfiFormController.cls`. Never run `format:apex` on existing files unless the user asks for a dedicated formatting pass (its own commit); use `--check` on a single file or format a brand-new file instead. Illuminated Cloud's own formatter may disagree with it.
- **Still not installed, only if needed (ask before installing; user-space only, no `rpm-ostree`):** `difftastic` (structural diffs of Apex and JS), `shellcheck` (for `scripts/` shell), `hyperfine` (only for timing).
- **Install route:** the user installs with `brew install <tool>`, or `pipx`/`npm` for the Node ones. Do not install anything yourself and do not use `dnf`.

### Know the limits of what you checked
State in the final message what was verified by what: Apex tests against the scratch org, Jest against stubs, a regex against a real `RegExp`, and what was only reasoned about (real browser behaviour, `lightning-input` validation, guest-user access). Name the ones that were only reasoned about; those are where the next bug is.
