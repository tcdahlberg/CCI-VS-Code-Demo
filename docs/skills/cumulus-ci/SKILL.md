---
name: cumulus-ci
description: "CumulusCI patterns for Salesforce scratch org development. Covers deploy strategy (incremental vs full rebuild), Apex test execution with CCI and SF CLI, code coverage queries, field/object metadata gotchas, Flow and metadata cleanup, and the Salesforce Code Analyzer workflow. Activate for any cci or sf deployment, testing, or metadata task."
metadata:
  author: UST-Agentic-Coding-Skills
  version: "1.0"
allowed-tools: Bash Read Write
---

# CumulusCI

## Key Commands Reference

```powershell
cci org list                                                        # List all orgs (CCI aliases + SF CLI aliases)
cci org info --org dev                                              # Get details for dev org (including sfdx_alias)
cci task run deploy --path force-app/main/default/lwc --org dev    # Deploy LWC only
cci task run deploy --path force-app/main/default/classes --org dev # Deploy Apex only
cci task run run_tests --org dev                                    # Run all Apex tests
cci org scratch_delete dev                                          # Delete the scratch org
cci flow run dev_org --org dev                                      # Full org rebuild (RARE — see below)
```

---

## Deployment Strategy

### ✅ Incremental Deploys (normal workflow)

```powershell
# LWC only
cci task run deploy --path force-app/main/default/lwc --org dev

# Apex only
cci task run deploy --path force-app/main/default/classes --org dev

# Single component
cci task run deploy --path force-app/main/default/lwc/componentName --org dev

# Entire object (fields + object XML)
cci task run deploy --path force-app/main/default/objects/ObjectName__c --org dev
```

### ⚠️ Full Rebuild — Only for Initial Setup

```powershell
cci flow run dev_org --org dev   # Creates a NEW scratch org — do NOT use for incremental changes
```

### SF CLI Alternative (targeted)

```powershell
sf project deploy start --source-dir force-app/main/default/lwc --target-org {ProjectName}__dev
```

---

## Apex Testing

### Org Naming ⚠️

| Tool | Alias to use | Example |
|---|---|---|
| CumulusCI | `--org dev` | `cci task run run_tests --org dev` |
| SF CLI | `--target-org {project.name}__dev` | `sf apex run test --target-org MyProject__dev` |

Get the SF CLI alias: `cci org info --org dev` — look for `sfdx_alias`.

### Running Tests

**SF CLI (preferred for individual test classes):**
```powershell
# Single class
sf apex run test --class-names ClassName_TEST --target-org {ProjectName}__dev --result-format human --code-coverage --wait 20

# Multiple classes
sf apex run test --class-names Test1,Test2,Test3 --target-org {ProjectName}__dev --result-format human --code-coverage --wait 20
```

- **Always add `--wait 20`** — some orgs are slow; lower values cause "test already enqueued" errors.
- **Always add `--code-coverage`** — shows coverage percentages in output.

**CumulusCI (for full validation/CI):**
```powershell
cci task run run_tests --org dev                                    # All tests
cci task run run_tests --org dev --test_name_match ClassName_TEST  # Specific class
```

- Output goes to `test_results.json` and `test_results.xml`.
- Coverage requirement set in `cumulusci.yml` under `tasks.run_tests.options.required_org_code_coverage_percent`.

---

## Code Coverage Queries

`ApexCodeCoverageAggregate` is in the Tooling API — always use `--use-tooling-api`:

```powershell
sf data query `
  --query "SELECT ApexClassOrTrigger.Name, NumLinesCovered, NumLinesUncovered FROM ApexCodeCoverageAggregate WHERE NumLinesCovered + NumLinesUncovered > 0 ORDER BY ApexClassOrTrigger.Name" `
  --target-org {ProjectName}__dev `
  --use-tooling-api `
  --result-format json 2>&1 | Tee-Object ai-logs/cov.json
```

**Coverage formula:** `(NumLinesCovered / (NumLinesCovered + NumLinesUncovered)) * 100 = Coverage %`

**Important timing:** The aggregate is stale until ALL tests run. It resets to 0 while tests are queued. Run `cci task run run_tests --org dev` (no filter) to refresh the org-wide aggregate. Per-class numbers from a single-class run: use `sf apex run test --class-names ... --code-coverage`.

---

## Apex Testing Gotchas ⚠️

- **"Test already enqueued"** — always add `--wait 20` (not just 10).
- **`RestRequest.params` is final** — use `req.addParameter('key', 'value')` in tests, not a new map assignment.
- **Coverage queries fail** — use `--use-tooling-api` flag.
- **Blank table output in PowerShell** — use `--result-format json`.
- **Picklist value errors** — use actual active values from the org schema; never hardcode assumed values.
- **Encryption key length** — must be exactly 32 bytes for AES256.
- **Date parsing** — `Date.parse('02/12/2026')` is locale-dependent; use explicit date construction or catch exceptions.
- **Unicode in Apex source files corrupts the compiler** — box-drawing characters (`--`, `->`, em-dashes) and other non-ASCII chars in Apex comments or strings cause cascading `"Invalid type"` errors. Always use plain ASCII. When seeing mysterious "Invalid type" errors on valid types, scan the file for non-ASCII bytes.
- **Managed package types need a staging org** — test classes referencing managed package types (`hed__Affiliation__c`, `summit__Summit_Events__c`) will fail to compile on a scratch org. Deploy those test classes to the staging org with `sf project deploy start --target-org <staging-alias>`, NOT `cci task run deploy --org dev`.

---

## Apex Test Writing Rules ⚠️

- **NEVER use `SeeAllData=true`** — tests must be fully isolated; always insert all required data.
- **`@AuraEnabled(cacheable=true)` caches results within a test run** — the most important gotcha:
  - Per-test inserts using auto-generated Names will collide across test methods in the same run.
  - **Fix**: use `@TestSetup` with a **unique sentinel field value**, then query by that value:
    ```apex
    // @TestSetup
    insert new MyObject__c(External_Key__c = 'test-sentinel-myfeature', ...);

    // In test method
    String recordName = [SELECT Name FROM MyObject__c WHERE External_Key__c = 'test-sentinel-myfeature' LIMIT 1].Name;
    MyController.getCacheableMethod(recordName);
    ```
- **Non-writable formula/rollup fields** — branches that filter on them will remain uncovered; accept this structural gap.
- **Restricted picklists — always use schema-driven values:**
  ```apex
  private static String firstPicklistValue(Schema.SObjectField field) {
      for (Schema.PicklistEntry entry : field.getDescribe().getPicklistValues()) {
          if (entry.isActive()) { return entry.getValue(); }
      }
      return null;
  }
  ```
- **`System.runAs()` for permission-dependent tests** — create a fresh user and wrap assertions to avoid elevated-permission interference from the test-running user.
- **Use `replace_string_in_file` over `insert_edit_into_file`** for targeted test edits — avoids duplicate method creation.
- **Do not create helper Python scripts** to generate test class content — edit the `.cls` file directly.

---

## Field Deployment Gotcha ⚠️

**Problem:** SF CLI marks new fields as "Unchanged" but the field doesn't exist in the org.  
**Symptoms:** Deploy says "Success" or "Unchanged", but SOQL queries fail with "No such column."  
**Root Cause:** Source tracking cache thinks field exists when it doesn't.

**Solutions (in order):**
1. Deploy the entire object: `cci task run deploy --path force-app/main/default/objects/ObjectName__c --org dev`
2. Clear cache: `Remove-Item -Path ".sf",".sfdx" -Recurse -Force`, then redeploy.
3. **Manual creation** (most reliable): Setup → Object Manager → Fields → New.

**Always verify** after field deploy:
```powershell
sf data query --query "SELECT Id, NewField__c FROM Object__c LIMIT 1" --target-org {ProjectName}__dev
```

---

## Custom Object & App Metadata Gotchas ⚠️

- **AutoNumber max 10 digits** — `displayFormat` supports at most 10 zeros; 11+ fails. Error cascades to ALL dependent fields, classes, triggers, and LWC.
- **Empty object-meta.xml cascades everywhere** — if the XML file is 0 bytes, ALL dependents fail with `"Entity not found"`. Check the object XML first when seeing widespread "Entity not found" errors.
- **Required Lookup needs `deleteConstraint`** — `<required>true</required>` + Lookup MUST include `<deleteConstraint>Restrict</deleteConstraint>` (or `Cascade`).
- **Master-Detail requires `ControlledByParent` sharing** — must set both `<sharingModel>` and `<externalSharingModel>` to `ControlledByParent`.
- **Lightning App needs `<tabs>`** — a `CustomApplication` with `<uiType>Lightning</uiType>` needs `<tabs>TabName</tabs>` elements or the sidebar is empty.
- **FlexiPage tab** — to add an AppPage to navigation, create a `CustomTab` with `<flexiPage>PageApiName</flexiPage>`, then reference that tab in the app's `<tabs>` list.
- **Deploy order for apps:** Objects → Tabs → FlexiPages → Applications. Out-of-order deploys cause "not found" errors.
- **Iterative deploy-fix cycle** — when `cci flow run dev_org` fails, delete the scratch org before re-running to avoid stale state: `cci org scratch_delete dev`.
- **Permission Set field exclusions** — cannot include `fieldPermissions` for required fields, Master-Detail fields, or Rollup Summary fields. Scan ALL `<required>true</required>` fields in object metadata upfront.
- **No `--` inside XML comments** — the XML spec forbids `--` between `<!--` and `-->`. Use `:` or another separator instead of `--` in comment text (e.g. avoid "read-only" written as "read--only", or phrases like "editable -- users can").

---

## Flow & Metadata Cleanup Gotchas ⚠️

- **Cannot delete an Apex class referenced by a flow** until ALL flow versions are removed.
  - **Fix order**: Delete all flow versions first (Setup → Flows → [Name] → View All Versions → delete each), then destructive-deploy the class.
  - Old/inactive versions still block deletion.
- **Valid flow status values** (Metadata API): Only `Active` and `Draft` — `Inactive` is NOT valid. Use `Draft` to deactivate a flow.
- **"Insufficient access rights on cross-reference id"** during flow deletion — means the flow still has active/draft versions; delete them from Setup UI first.

---

## Salesforce Code Analyzer

### Running

```powershell
# Full scan — save JSON + HTML, fail on Moderate or higher (sev <= 3)
sf code-analyzer run --target force-app/main/default --output-file ai-logs/code-analyzer.json --output-file ai-logs/code-analyzer.html --severity-threshold 3 2>&1 | Out-File ai-logs/ca-run.txt

# High/Critical only — faster feedback
sf code-analyzer run --target force-app/main/default --output-file ai-logs/code-analyzer.json --severity-threshold 2 2>&1 | Out-File ai-logs/ca-run.txt

# Read summary
Get-Content ai-logs/ca-run.txt | Select-Object -Last 20
```

Config file: `code-analyzer.yml` in project root (auto-applied when running from root). Set `eslint_config_file: "eslint.config.js"` to use the project ESLint config.

### Fix Priority

1. **Critical (sev1)** — Fix immediately (unless `UninstantiableEngineError` — tooling conflict, not code).
2. **High (sev2)** — Fix all; mostly security issues (`ApexCRUDViolation`, `ApexSOQLInjection`, `no-dupe-class-members`, `no-prototype-builtins`).
3. **Moderate (sev3)** — Fix where practical; skip complexity/naming if refactor is too risky.
4. **Low/Info (sev4–5)** — Optional; `AvoidDebugStatements` and `ApexDoc` are noise for dev orgs.

### Apex Fix Patterns (High severity)

- **`ApexCRUDViolation`** — Add `WITH USER_MODE` to inline SOQL, or check `isCreateable()`/`isUpdateable()` before DML.
  - ⚠️ `WITH USER_MODE` must come **BEFORE** `ORDER BY`, `LIMIT`, and `OFFSET`:
    ```apex
    // ✅
    queryString += ' WITH USER_MODE ORDER BY Name LIMIT 50';
    // ❌ — throws "unexpected token: 'WITH'" if ORDER BY/LIMIT already appended
    Database.query(queryString + ' WITH USER_MODE');
    ```
  - Never add `WITH USER_MODE` to queries inside test classes.
- **`ApexSOQLInjection`** — Replace string concatenation with `String.escapeSingleQuotes(param)`, or use `:localVar` bind variable.

### LWC/JS Fix Patterns (High severity)

- **`no-unused-vars`** — Remove unused parameter or prefix with `_` if required by interface.
- **`no-dupe-class-members`** — Keep the better implementation, delete the duplicate.
- **`no-prototype-builtins`** — Replace `obj.hasOwnProperty(key)` with `Object.prototype.hasOwnProperty.call(obj, key)`.

### Known Acceptable / Non-Fixable Items

- **`UninstantiableEngineError` (eslint, sev1)** — ESLint version mismatch; not a code bug. Disable ESLint in `code-analyzer.yml` if it blocks CI: set `disable_engine: true` under `engines.eslint`.
- **`CyclomaticComplexity` / `CognitiveComplexity`** — Large controllers legitimately exceed thresholds; treat as a separate refactoring task.
- **`AvoidDebugStatements`** — Clean up before production release; suppress in `code-analyzer.yml` for dev.

---

## Agentforce / Einstein LLM (Prompt Builder)

- **`additionalConfig.applicationName` is REQUIRED** — without it, `ConnectApi.EinsteinLLM.generateMessagesForPromptTemplate()` silently fails with `"Failed to generate Einstein LLM generations response"`.
  ```apex
  ptInput.additionalConfig = new ConnectApi.EinsteinLlmAdditionalConfigInput();
  ptInput.additionalConfig.applicationName = 'PromptBuilderPreview';
  ```
- **`isPreview = false` for real calls** — `isPreview = true` returns 0 generations; it's only for the Prompt Builder UI preview mode.
- **Don't build retry loops around AI failures** — persistent "failed to generate" errors almost always mean a missing config parameter, not a transient error.
- **Flex Template input key format**: `'Input:VariableName'` (the `Input:` prefix is required).
  - ✅ `ptInput.inputParams.put('Input:MyContext', wrappedValue);`
  - ❌ `ptInput.inputParams.put('MyContext', wrappedValue);`

