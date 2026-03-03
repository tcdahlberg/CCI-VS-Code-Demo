# Concise AI Instructions for IntelliJ

## Primary Work: Salesforce Development
- **Build Tool:** CumulusCI (cci) - Python CLI for Salesforce
- **Org Type:** Scratch orgs (not production)
- **Config Files:** cumulusci.yml, sfdx-project.json
- **Docs:** https://cumulusci.readthedocs.io/

## Environment & Tools
- Windows + PowerShell (use PowerShell syntax, NOT cmd)
- IntelliJ IDEA + Illuminated Cloud plugin
- Tools installed: Node.js, Python 3.13, CumulusCI, Salesforce CLI (sf), PurgeCSS, Playwright, fd, bat, ripgrep, tokei

## Key Commands
- Salesforce: `cci org list`, `cci org info`
- **Targeted Deploy**: `cci task run deploy --path force-app/main/default/lwc --org dev`
- **NEVER** use `cci flow run dev_org` for incremental changes (rebuilds entire org!)
- **Testing (LWC)**: `npm test` (runs Jest tests), `npm test -- --coverage` (with coverage)
- **Testing (Apex)**:
    - **CumulusCI (preferred)**: `cci task run run_tests --org dev` (uses org alias "dev")
    - **SF CLI**: `sf apex run test --target-org <ProjectName>__dev` (different org name!)
    - **Org naming**: CumulusCI uses `dev`, SF CLI uses `{project.name}__dev` from cumulusci.yml
    - **Get project name**: Check `project.name` in cumulusci.yml (e.g., `UST-Salesforce-RFI__dev`)
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
- CSS extraction: Use PurgeCSS (84.5% reduction, 0 differences)
- Avoid: Manual regex extraction (failed with 58 differences)
- Browser testing: Use Playwright
- File operations: PowerShell-native commands only

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
    - Example: For project `UST-Salesforce-RFI`, use `--target-org UST-Salesforce-RFI__dev`
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
  
  # Get coverage for specific classes (e.g., UPay classes)
  sf data query --query "SELECT ApexClassOrTrigger.Name, NumLinesCovered, NumLinesUncovered FROM ApexCodeCoverageAggregate WHERE ApexClassOrTrigger.Name LIKE 'UPay%'" --target-org {ProjectName}__dev --use-tooling-api --result-format json
  
  # Calculate coverage: (NumLinesCovered / (NumLinesCovered + NumLinesUncovered)) * 100
  ```
- **IMPORTANT**: Replace `{ProjectName}` with `project.name` from cumulusci.yml (e.g., `Summit-Events-App-Touchnet-UPay__dev`)
- **Pro tip**: Use `--result-format json` for parseable output (table/human formats may be empty in PowerShell)
- **Coverage formula**: `(NumLinesCovered / (NumLinesCovered + NumLinesUncovered)) * 100 = Coverage %`
- **Example**: 126 covered / (126 + 14 uncovered) = 126/140 = 90% coverage

## Apex Testing Gotchas ⚠️
- **"Test already enqueued" error**: Always add `--wait 10` to `sf apex run test` commands
- **RestRequest.params is final**: Can't assign new map in tests - use `req.addParameter('key', 'value')` instead
- **Coverage queries fail**: Use `--use-tooling-api` flag (ApexCodeCoverageAggregate is in Tooling API, not Data API)
- **Blank table/human output in PowerShell**: Use `--result-format json` instead for reliable output
- **Picklist value errors in tests**:
    - ❌ `summit__Template__c = 'Standard'` fails if value doesn't exist in org
    - ✅ Use actual picklist values from your org or make field non-restricted
    - **Country fields**: Use full country names or 2-letter codes that exist in org (e.g., 'United States' or skip field)
- **Encryption key length**: Must be exactly 32 bytes for AES256 (use in test setup)
    - Example: `summit__Cookie_Encryption_Key__c = '12345678901234567890123456789012'`
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

## When Starting
1. Check for `cumulusci.yml` or `sfdx-project.json` (Salesforce project)
2. Check `/docs/AI-TOOLS-CONFIG.md` for context
3. Use existing tools (fd, rg, PurgeCSS, cci) before suggesting new ones
4. Verify commands work before declaring success
5. Clean up failed attempts

## Remember
- Professional tools > custom scripts for complex tasks
- Test file operations before moving on
- PowerShell syntax on Windows
- Keep directories organized
- Default to cci for Salesforce work
- **ALWAYS use targeted deploys** - protect existing org setup
- Create new files cleanly - don't let content get scrambled
- LWC reactivity: new objects/arrays, never mutate in place
- Test deployments after file changes to catch errors early

