# Concise AI Instructions for IntelliJ

## ⚠️ Maintaining These Instructions
These instructions apply across ALL projects. When adding or updating content here:
- **Keep everything project-agnostic** — no project names, org aliases, object/field names, or URLs specific to any one project
- **Use placeholders** like `{ProjectName}`, `<host>`, `MyObject__c` instead of real values
- **Project-specific context belongs in `docs/AI-TOOLS-CONFIG.md`** within that project's repo — reference it from here, never inline it here
- If you find yourself writing something only true for one project, put it in that project's `AI-TOOLS-CONFIG.md` instead

## Primary Work: Salesforce Development
- **Build Tool:** CumulusCI (cci) - Python CLI for Salesforce
- **Org Type:** Scratch orgs (not production)
- **Config Files:** cumulusci.yml, sfdx-project.json
- **Docs:** https://cumulusci.readthedocs.io/

## Environment & Tools
- Windows + PowerShell (use PowerShell syntax, NOT cmd)
- IntelliJ IDEA + Illuminated Cloud plugin
- Tools installed: Node.js, Python 3.13, CumulusCI, Salesforce CLI (sf), PurgeCSS, Playwright, BackstopJS, fd, bat, ripgrep, tokei, jq

## Key Commands
- Salesforce: `cci org list`, `cci org info`
- **Targeted Deploy**: `cci task run deploy --path force-app/main/default/lwc --org dev`
- **NEVER** use `cci flow run dev_org` for incremental changes (rebuilds entire org!)
- **Code Analysis**: `sf code-analyzer run --target force-app/main/default --output-file ai-logs/code-analyzer.json --output-file ai-logs/code-analyzer.html`
- **Testing (LWC)**: `npm test` (runs Jest tests), `npm test -- --coverage` (with coverage)
- **Testing (Apex)**:
  - **CumulusCI (preferred)**: `cci task run run_tests --org dev` (uses org alias "dev")
  - **SF CLI**: `sf apex run test --target-org <ProjectName>__dev` (different org name!)
  - **Org naming**: CumulusCI uses `dev`, SF CLI uses `{project.name}__dev` from cumulusci.yml
  - **Get project name**: Check `project.name` in cumulusci.yml
- Use `Remove-Item -Force` NOT `del /Q`
- File search: `fd "pattern"`, `rg "text"`
- Code stats: `tokei`
- CSS: `node run-purgecss.js`

## Project Preferences
- Keep root clean (essential files only)
- Docs in `/docs` folder
- Check `docs/AI-TOOLS-CONFIG.md` for project context
- Check `cumulusci.yml` for Salesforce projects
- **Documentation**: Consolidate, don't proliferate
  - Archive historical/redundant docs to `docs/archive/`
  - Maintain 5-10 core guides max
  - Update existing docs rather than creating new ones
  - Use clear naming: MAIN_GUIDE.md, TROUBLESHOOTING.md, etc.

## Proven Solutions
- Salesforce orgs: Use CumulusCI (cci) commands
- CSS extraction: Use PurgeCSS (significant reduction, verified zero differences vs manual regex)
- Avoid: Manual regex CSS extraction (error-prone)
- Browser testing: Use Playwright or BackstopJS
- File operations: PowerShell-native commands only

## BackstopJS — Visual Comparison (e.g. VF vs LWC)
BackstopJS (`backstopjs` — globally installed) uses Playwright under the hood to capture screenshots and generate side-by-side pixel-diff HTML reports. Use this to verify visual parity between two versions of a page (e.g. a Visualforce original vs its LWC replacement).

### Key Commands
```powershell
backstop init              # Generate backstop.json in project root (first time only)
backstop reference         # Capture reference screenshots (ground truth)
backstop test              # Capture test screenshots and diff against reference
backstop approve           # Promote test screenshots to new reference (when diff is intentional)
backstop openReport        # Open the HTML diff report in browser
```

### Config Location
- Config file: `backstop.json` in project root
- Output goes to: `ai-logs/backstop/` (gitignored)
  - `ai-logs/backstop/reference/` — ground-truth screenshots
  - `ai-logs/backstop/test/` — current build screenshots
  - `ai-logs/backstop/html_report/` — diff report

### backstop.json Template
```json
{
  "id": "reference-vs-new",
  "viewports": [
    { "label": "desktop", "width": 1280, "height": 900 },
    { "label": "mobile", "width": 375, "height": 812 }
  ],
  "scenarios": [
    {
      "label": "Page Name",
      "referenceUrl": "https://<host>/reference-page-url",
      "url":          "https://<host>/new-page-url",
      "delay": 1500,
      "misMatchThreshold": 5.0
    }
  ],
  "paths": {
    "bitmaps_reference": "ai-logs/backstop/reference",
    "bitmaps_test":      "ai-logs/backstop/test",
    "html_report":       "ai-logs/backstop/html_report",
    "engine_scripts":    "ai-logs/backstop/engine_scripts"
  },
  "engine": "playwright",
  "engineOptions": { "browser": "chromium" },
  "report": ["browser"]
}
```

### Authentication
Whether a login script is needed depends on the project. Check `docs/AI-TOOLS-CONFIG.md` for the project's access model (some pages are public/guest-accessible; others require authentication).

If authentication is required, use an `onBefore.js` engine script:
```javascript
// ai-logs/backstop/engine_scripts/onBefore.js
module.exports = async (page, scenario, vp) => {
  await page.goto('https://<host>/login');
  await page.fill('#username', process.env.SF_USERNAME);
  await page.fill('#password', process.env.SF_PASSWORD);
  await page.click('#Login');
  await page.waitForNavigation();
};
```
Reference in each scenario that needs it: `"onBeforeScript": "onBefore.js"`

### Scratch Org Test Data
Scratch orgs may contain pre-defined sample data at varying configurations. Not all records will activate all pages/features. Check `docs/AI-TOOLS-CONFIG.md` for project-specific guidance on which records to use for full coverage.

### Workflow for Page-to-Page Migration
1. Deploy or access the reference version, run `backstop reference` to capture ground truth
2. Build the new version, deploy it
3. Run `backstop test` — review HTML report for layout/content parity
4. Fix gaps, repeat until diff % is at or near 0
5. Run `backstop approve` when intentional design improvements are accepted as new baseline

## LWC Development (Proven Patterns)
- **Reactivity**: NEVER mutate nested objects/arrays directly
  - ❌ `this.data.array[0].prop = value` (won't trigger re-render)
  - ✅ `this.data = {...this.data, array: [...newArray]}` (triggers reactivity)
- **Wire Adapters & Updates**: ALWAYS use refreshApex after updateRecord
  - ❌ `await updateRecord(recordInput);` (wire adapter won't refresh)
  - ✅ `await updateRecord(recordInput); await refreshApex(this.wiredResult);`
  - Store wire result: `@wire(getRecord, ...) wiredData(result) { this.wiredResult = result; }`
- **Templates**: NO computed property access
  - ❌ `{answers[question.field]}` (LWC1038 error)
  - ✅ Pass object, use `{question.currentValue}` or getters
- **Templates**: lwc:else must be sibling to lwc:if, NOT nested inside
  - ❌ `<template lwc:if={condition}><template lwc:else>` (LWC1165 error)
  - ✅ `<template lwc:if={condition}>...</template><template lwc:else>...</template>`
- **Boolean attributes**: Can't use expressions like `disabled={item.id === 0}`
  - ❌ `disabled={item.id === 0}` (HTML parse error)
  - ✅ Create getter or handle in onclick: `if (id === 0) return;`
- **Reserved Keywords**: Avoid `Page`, `User`, etc. as class names
- **File Creation**: Create files directly, avoid edits that scramble content
- **Apex-First**: Load all data in one call, minimize server round trips
- **Modern UI**: Use modals for editing, drag-drop for reordering, toast for feedback
  - Modals: Better UX than inline forms
  - Drag-drop: HTML5 Drag API + arrow button fallback for accessibility
  - Drag-drop reorder: After removing item, use target index directly for insert
  - Toast: `ShowToastEvent` for all user feedback (not console.log)

## Deployment Strategy
- **Incremental Changes**:
  - LWC only: `cci task run deploy --path force-app/main/default/lwc --org dev`
  - Apex only: `cci task run deploy --path force-app/main/default/classes --org dev`
  - Single component: `cci task run deploy --path force-app/main/default/lwc/componentName --org dev`
- **Full Rebuild** (rare, initial setup only):
  - `cci flow run dev_org --org dev` (creates new scratch org!)
- **Alternative**: Salesforce CLI for targeted deploys
  - `sf project deploy start --source-dir force-app/main/default/lwc --target-org {ProjectName}__dev`

## Apex Testing Strategy
- **⚠️ CRITICAL ORG NAMING**:
  - CumulusCI commands: Use `--org dev` (alias)
  - SF CLI commands: Use `--target-org {project.name}__dev` (full name from cumulusci.yml)
  - Example: For project `My-Project`, use `--target-org My-Project__dev`
- **Preferred: Use SF CLI** for running individual test classes (more reliable):
  - Run specific test: `sf apex run test --class-names ClassName_TEST --target-org {ProjectName}__dev --result-format human --code-coverage --wait 10`
  - Run multiple tests: `sf apex run test --class-names Test1,Test2,Test3 --target-org {ProjectName}__dev --result-format human --code-coverage --wait 10`
  - **Always add `--wait 10`** to avoid "test already enqueued" errors
  - **Always add `--code-coverage`** to see coverage percentages in output
- **Alternative: Use CumulusCI** for running all tests (validation/CI):
  - Run all tests: `cci task run run_tests --org dev`
  - Run specific test: `cci task run run_tests --org dev --test_name_match ClassName_TEST`
  - Coverage requirement already set in cumulusci.yml (e.g., 75%)
  - **Org alias**: Always use `dev` with CumulusCI
- **Test results**: CumulusCI outputs to `test_results.json` and `test_results.xml`
- **Coverage requirement**: Check `tasks.run_tests.options.required_org_code_coverage_percent` in cumulusci.yml

## Getting Code Coverage Reports
- **IMPORTANT**: Use `--use-tooling-api` flag for coverage queries (ApexCodeCoverageAggregate is in Tooling API, not Data API)
- **After running tests**, query coverage using Tooling API:
  ```powershell
  # Get coverage for all classes (with percentage calculation)
  sf data query --query "SELECT ApexClassOrTrigger.Name, NumLinesCovered, NumLinesUncovered FROM ApexCodeCoverageAggregate WHERE NumLinesCovered + NumLinesUncovered > 0 ORDER BY ApexClassOrTrigger.Name" --target-org {ProjectName}__dev --use-tooling-api --result-format json
  
  # Calculate coverage: (NumLinesCovered / (NumLinesCovered + NumLinesUncovered)) * 100
  ```
- **IMPORTANT**: Replace `{ProjectName}__dev` with the `sfdx_alias` from `cci org info --org dev`
- **Pro tip**: Use `--result-format json` for parseable output (table/human formats may be empty in PowerShell)
- **Coverage formula**: `(NumLinesCovered / (NumLinesCovered + NumLinesUncovered)) * 100 = Coverage %`
- **Example**: 126 covered / (126 + 14 uncovered) = 126/140 = 90% coverage

## Apex Testing Gotchas ⚠️
- **"Test already enqueued" error**: Always add `--wait 10` to `sf apex run test` commands
- **RestRequest.params is final**: Can't assign new map in tests - use `req.addParameter('key', 'value')` instead
- **Coverage queries fail**: Use `--use-tooling-api` flag (ApexCodeCoverageAggregate is in Tooling API, not Data API)
- **Blank table/human output in PowerShell**: Use `--result-format json` instead for reliable output
- **Picklist value errors in tests**:
  - ❌ `MyField__c = 'SomeValue'` fails if value doesn't exist in org
  - ✅ Use actual picklist values from your org or make field non-restricted
  - **Country fields**: Use full country names or 2-letter codes that exist in org (e.g., 'United States' or skip field)
- **Encryption key length**: Must be exactly 32 bytes for AES256 (use in test setup)
  - Example: `Encryption_Key__c = '12345678901234567890123456789012'`
- **Date parsing**: Use locale-aware format in tests or catch exceptions
  - `Date.parse('02/12/2026')` format depends on user locale
- **Test data isolation**: Always use `@TestSetup` for shared test data to improve performance

## Field Deployment Gotcha ⚠️
- **Problem**: SF CLI marks new fields as "Unchanged" but field doesn't exist in org
- **Symptoms**: Deploy says "Success" or "Unchanged", but SOQL queries fail with "No such column"
- **Root Cause**: Source tracking cache thinks field exists when it doesn't
- **Solutions** (in order):
  1. Deploy entire object: `cci task run deploy --path force-app/main/default/objects/ObjectName__c --org dev`
  2. Clear cache: `Remove-Item -Path ".sf",".sfdx" -Recurse -Force`, then redeploy
  3. **Manual creation** (most reliable): Setup → Object Manager → Fields → New
- **Always verify**: After field deploy, test with SOQL query
  - With CumulusCI: `sf data query --query "SELECT Id, NewField__c FROM Object__c LIMIT 1" --target-org {ProjectName}__dev`
  - Replace `{ProjectName}` with `project.name` from cumulusci.yml
- **Document manual steps**: If manual creation required, note it for future deploys

## Salesforce Code Analyzer ⚠️
- **Config file**: `code-analyzer.yml` in project root (auto-applied when running from root)
- **ESLint config**: Set `eslint_config_file: "eslint.config.js"` in `code-analyzer.yml` to use project ESLint config

### Running the Analyzer
```powershell
# Full scan — save JSON + HTML, fail on Moderate or higher
sf code-analyzer run --target force-app/main/default --output-file ai-logs/code-analyzer.json --output-file ai-logs/code-analyzer.html --severity-threshold 3 2>&1 | Out-File ai-logs/ca-run.txt

# High/Critical only — faster feedback loop
sf code-analyzer run --target force-app/main/default --output-file ai-logs/code-analyzer.json --severity-threshold 2 2>&1 | Out-File ai-logs/ca-run.txt

# Read summary line from run log
Get-Content ai-logs/ca-run.txt | Select-Object -Last 20
```

### Parsing Results
PowerShell and jq have encoding issues with Code Analyzer JSON output. **Always use Python**:
```powershell
# Use ai-logs/parse_violations.py (reuse/update this file):
python ai-logs/parse_violations.py > ai-logs/parse-out.txt 2>&1; type ai-logs/parse-out.txt
```
Standard parse script template:
```python
import json
with open('ai-logs/code-analyzer.json', encoding='utf-8') as f:
    data = json.load(f)
high = [v for v in data['violations'] if v['severity'] <= 2]
print(f"High/Critical: {len(high)}")
for v in high:
    loc = v['locations'][0]
    fname = loc.get('file', 'N/A').replace('\\', '/').split('/')[-1]
    print(f"  sev={v['severity']} {v['engine']} {v['rule']} {fname}:{loc.get('startLine','?')}")
    print(f"    {v['message'][:120]}")
```

### Fix Priority
1. **Critical (sev1)** — Fix immediately unless it's `UninstantiableEngineError` (tooling conflict, not code)
2. **High (sev2)** — Fix all; mostly security issues (`ApexCRUDViolation`, `ApexSOQLInjection`, JS `no-dupe-class-members`, `no-prototype-builtins`)
3. **Moderate (sev3)** — Fix where practical; skip complexity/naming if refactor is too risky
4. **Low (sev4) / Info (sev5)** — Optional; `AvoidDebugStatements` and `ApexDoc` are noise for dev orgs

### Apex Fix Patterns (High severity)
- **`ApexCRUDViolation`** — Add `WITH USER_MODE` to inline SOQL, or `WITH SECURITY_ENFORCED`; for DML add `if (!Schema.sObjectType.MyObj__c.isCreateable/isUpdateable()) throw new AuraHandledException(...)`
  - ⚠️ **`WITH USER_MODE` must come BEFORE `ORDER BY`, `LIMIT`, and `OFFSET`** — appending it to the end of a complete query string causes `System.QueryException: unexpected token: 'WITH'`
  - ✅ `query_string += ' WITH USER_MODE ORDER BY Name LIMIT 50'`
  - ❌ `Database.query(query_string + ' WITH USER_MODE')` when query_string already has ORDER BY/LIMIT
- **`ApexSOQLInjection`** — Replace string concatenation with `String.escapeSingleQuotes(param)` before concat, OR extract to local variable and use `:localVar` bind
- **Never** add `WITH USER_MODE` to queries inside `@TestVisible` or test classes

### LWC/JS Fix Patterns (High severity)
- **`no-unused-vars`** — Remove unused parameter or prefix with `_` if required by interface
- **`no-dupe-class-members`** — Keep the better implementation (more error handling / null checks), delete the duplicate
- **`no-prototype-builtins`** — Replace `obj.hasOwnProperty(key)` → `Object.prototype.hasOwnProperty.call(obj, key)`

### Known Non-Fixable / Acceptable Items
- **`UninstantiableEngineError` (eslint, sev1)** — ESLint version mismatch between Code Analyzer and project; not a code bug; disable ESLint in `code-analyzer.yml` if it blocks CI: `disable_engine: true` under `engines.eslint`
- **`CyclomaticComplexity` / `CognitiveComplexity`** — Large controllers legitimately exceed thresholds; refactoring is a separate task, not a quick fix
- **`AvoidDebugStatements`** — Development debug statements; suppress in `code-analyzer.yml` or clean up before production release

## When Starting
1. Check for `cumulusci.yml` or `sfdx-project.json` (Salesforce project)
2. Check `/docs/AI-TOOLS-CONFIG.md` for context
3. Use existing tools (fd, rg, PurgeCSS, cci) before suggesting new ones
4. Verify commands work before declaring success
5. Clean up failed attempts

## Terminal Command Best Practices
- **Always run deploys and one-shot commands as FOREGROUND** (`isBackground: false`) — they finish in <30s and output is captured directly
- **Background** (`isBackground: true`) is ONLY for servers/watchers that never terminate (e.g. `npm run watch`)
- **NEVER use `Start-Sleep` + `get_terminal_output`** to wait for a background deploy — this blocks the user and is unreliable
- **Use `2>&1`** to merge stderr into stdout so CCI/SF progress lines appear in output
- **If PowerShell swallows output**, pipe through `| Out-String | Write-Host` to force display:
  ```powershell
  cci task run deploy --path force-app/main/default/lwc/myComponent --org dev 2>&1 | Out-String | Write-Host
  ```
- **NEVER use Python subprocess wrappers** to run `cci`/`sf` — output is silently swallowed and return codes are unreliable. Run directly in the terminal.
- **Trust deploy success** — when CCI outputs `[Success]: Succeeded`, move on. Do not re-verify with extra queries unless something actually fails.
- **If terminal stops returning output**, open a new terminal — don't waste attempts testing if it's broken.
- **Save output to ai-logs/ then read with read_file** — most reliable way to capture long-running command output:
  ```powershell
  cci task run deploy ... 2>&1 | Out-File ai-logs/deploy.txt
  # Then use read_file to inspect ai-logs/deploy.txt
  ```
- **jq for JSON parsing** — use `jq` to parse `sf` JSON output reliably:
  ```powershell
  sf apex run test --class-names MyClass_TEST --target-org MyOrg__dev --result-format json --code-coverage --wait 10 2>&1 | Out-File ai-logs/test.json
  jq ".result.summary" ai-logs/test.json
  ```
- **Correct pattern:**
  ```powershell
  # ✅ GOOD — foreground, save to ai-logs/, read back
  cci task run deploy --path force-app/main/default/lwc/myComponent --org dev 2>&1 | Out-File ai-logs/deploy.txt
  ```
- **Wrong pattern:**
  ```powershell
  # ❌ BAD — background + sleep + poll
  cci task run deploy ... --org dev   # isBackground: true
  Start-Sleep -Seconds 35             # separate foreground call
  get_terminal_output                 # poll
  ```

## Resolving the Org Alias (Happy Path) ⚠️
When starting work on a project and needing to know the correct SF CLI `--target-org` alias:
1. **Always start here** — `cci org list` to see both CCI aliases and SF CLI aliases:
   ```powershell
   cci org list 2>&1 | Tee-Object ai-logs/cci.txt; Get-Content ai-logs/cci.txt
   ```
2. The SF CLI alias appears in the `sfdx_alias` column of `cci org info --org dev` output:
   ```powershell
   cci org info --org dev 2>&1 | Tee-Object ai-logs/orginfo.txt; Get-Content ai-logs/orginfo.txt
   ```
   Look for: `sfdx_alias   {ProjectName}__dev`
3. **Rule**: CumulusCI always uses `--org dev`; SF CLI always uses the `sfdx_alias` value (e.g. `{ProjectName}__dev`)
4. **Do NOT guess** the SF CLI alias from the project name alone — the format is `{sfdx-project.json "name"}__dev` but confirm via `cci org info`

## Querying the Org with Anonymous Apex (Happy Path) ⚠️
`sf data query` in PowerShell silently swallows output. The reliable pattern for inspecting live org data:

1. **Write the Apex to a file** in `ai-logs/` — avoids ALL quoting issues:
   ```apex
   // ai-logs/check_something.apex
   List<MyObject__c> recs = [SELECT Id, SomeField__c FROM MyObject__c LIMIT 3];
   for (MyObject__c r : recs) System.debug('VAL:' + r.SomeField__c);
   ```
2. **Run it with `sf apex run --file`** and capture via `Tee-Object`:
   ```powershell
   sf apex run --target-org {ProjectName}__dev --file ai-logs/check_something.apex 2>&1 | Tee-Object ai-logs/anon_out.txt
   ```
3. **Read the output file** — debug lines appear as `USER_DEBUG[N]DEBUG<value>`:
   ```
   USER_DEBUG[2]DEBUGVAL:AS_AH_CER|AHCT
   ```
4. **Do NOT use**: inline `--apex "..."` strings (PowerShell quoting breaks them), `sf data query` for non-standard objects or Tooling API, or Python subprocesses.

**For Tooling API queries specifically** (e.g. `ApexCodeCoverageAggregate`, picklist metadata):
```powershell
# ✅ Only reliable pattern — use --use-tooling-api flag:
sf data query --query "SELECT ..." --target-org {ProjectName}__dev --use-tooling-api --result-format json 2>&1 | Tee-Object ai-logs/tooling.json
# Then read_file ai-logs/tooling.json
```

## AI Working Files (ai-logs/)
- **All AI-generated output files go in `ai-logs/`** — deploy logs, test results, JSON output, jq filters, Python scripts
- `ai-logs/` is in `.gitignore` — never tracked by git
- Standard filenames to reuse: `ai-logs/deploy.txt`, `ai-logs/test.txt`, `ai-logs/sftest.txt`, `ai-logs/cov.json`, `ai-logs/cov_filter.jq`
- Any Python helper scripts go in `ai-logs/` (e.g. `ai-logs/run_tests.py`)
- Do NOT leave temp files in the project root

## Windows / Line Ending Notes ⚠️
- **Windows uses CRLF (`\r\n`), Salesforce orgs use LF (`\n`)** — never compare file byte counts or line counts between local files and org to detect changes; always diff content directly
- **Never assert on file size or line count** as a proxy for "did the deploy change anything"

## Apex Test Writing Rules ⚠️
- **NEVER use `SeeAllData=true`** — tests must be fully isolated; always insert all required data
- **`@AuraEnabled(cacheable=true)` caches results WITHIN a test run** — this is the most important gotcha for AuraEnabled controller tests:
  - If test method A calls `getRecord('My Record 0003')` and test method B also resolves to that same auto-generated Name, the cache returns A's result to B
  - **Per-test inserts using auto-generated Names WILL collide** with each other across test methods in the same run
  - **The fix**: Put test data in `@TestSetup` with a **unique sentinel field value** (any field that won't collide — a URL, an external ID, a unique text field), then query by that value to get the Name:
    ```apex
    // In @TestSetup — use a unique sentinel value on a queryable field:
    insert new MyObject__c(External_Key__c = 'test-sentinel-myfeature', ...);

    // In test method — query by sentinel, not by auto-generated Name:
    String recName = [SELECT Name FROM MyObject__c
        WHERE External_Key__c = 'test-sentinel-myfeature' LIMIT 1].Name;
    MyController.getCacheableMethod(recName); // cacheable=true finds this record — no collision
    ```
  - `@TestSetup` records are inserted ONCE before any test method runs, so they're always the first (and only) records cacheable sees for their Name
  - **Do NOT assert on field values** that depend on which record `cacheable=true` returns — only assert the method doesn't throw and returns non-null
- **Non-writable formula/rollup fields can't be set in tests** — branches that filter on them will remain uncovered unless real org data exists. Accept this as a structural gap and move on rather than chasing it.
- **Restricted picklists must use schema-driven values** — never hardcode picklist values; always query the schema to get a valid active value:
  ```apex
  // Helper to get first active picklist value for a field:
  private static String firstPicklistValue(Schema.SObjectField f) {
      for (Schema.PicklistEntry e : f.getDescribe().getPicklistValues()) {
          if (e.isActive()) return e.getValue();
      }
      return null;
  }
  // Usage:
  String val = firstPicklistValue(Schema.SObjectType.MyObj__c.fields.getMap().get('MyField__c'));
  if (String.isNotBlank(val)) record.MyField__c = val;
  ```
- **`ApexCodeCoverageAggregate` is stale until ALL tests run** — running only one test class refreshes per-test coverage but NOT the org-wide aggregate. The aggregate only updates after a full org test run (`cci task run run_tests --org dev` with no filter). Use `sf apex run test --class-names ... --code-coverage` to get per-class numbers from a single-class run.
- **Coverage aggregate shows 0 while tests are queued** — if `sf apex run test` times out, the tests keep running in the org. The aggregate resets to 0 until they finish. Poll with the Tooling API query and wait for `NumLinesCovered > 0` before re-queuing.
- **"Test already enqueued" error**: Always add `--wait 20` to `sf apex run test` commands (not just 10 — some orgs are slow)
- **`@TestSetup` data is visible to all test methods** in the class — use it for shared setup; per-test inserts are fine for branch-specific data
- **Querying by Id after insert** — safe for non-cacheable methods:
  ```apex
  insert record;
  String name = [SELECT Name FROM MyObject__c WHERE Id = :record.Id].Name;
  ```
  **Warning**: for `cacheable=true` methods, another test method may have already cached a result for that same Name. Prefer `@TestSetup` + sentinel value for cacheable controllers.
- **replace_string_in_file over insert_edit_into_file** for targeted test edits — `insert_edit_into_file` can create duplicate methods; use `replace_string_in_file` with enough context lines to be unique
- **Do not create helper Python scripts** to generate test class content — edit the `.cls` file directly
