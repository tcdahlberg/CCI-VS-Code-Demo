---
name: terminal-commands
description: "PowerShell terminal best practices for Salesforce development on Windows. Covers foreground vs background execution, output capture to ai-logs/, org alias resolution, anonymous Apex queries, Tooling API queries, and encoding gotchas. Activate whenever running cci, sf, jq, or any long-running CLI command."
metadata:
  author: UST-Agentic-Coding-Skills
  version: "1.0"
allowed-tools: Bash Read Write
---

# Terminal Commands

## Core Rules

- **Always use PowerShell syntax** — NOT cmd (`del /Q`, `type`, etc.)
- **Use `2>&1`** to merge stderr into stdout so CCI/SF progress lines appear in output.
- **NEVER use Python subprocess wrappers** to run `cci`/`sf` — output is silently swallowed and return codes are unreliable. Run directly in the terminal.
- **NEVER use `Start-Sleep` + poll** to wait for a background deploy — this blocks the user and is unreliable.
- **If the terminal stops returning output**, open a new terminal — don't waste attempts testing if it's broken.

---

## Foreground vs Background

| Scenario | `isBackground` | Reason |
|---|---|---|
| Deploy commands (`cci task run deploy`, `sf project deploy start`) | **false** | Finish in <30 s; output captured directly |
| Test runs (`sf apex run test`, `cci task run run_tests`) | **false** | Finish in <5 min; output must be inspected |
| Code analyzer runs | **false** | Long but bounded; save to file |
| Servers / watchers that never terminate (`npm run watch`) | **true** | Only case where background is appropriate |

---

## Saving and Reading Output

Most reliable pattern — save to `ai-logs/`, then read the file:

```powershell
# ✅ GOOD — foreground, save to ai-logs/, read back
cci task run deploy --path force-app/main/default/lwc/myComponent --org dev 2>&1 | Out-File ai-logs/deploy.txt
# Then read ai-logs/deploy.txt

# ❌ BAD — background + sleep + poll
cci task run deploy ... --org dev   # isBackground: true
Start-Sleep -Seconds 35
get_terminal_output
```

If PowerShell swallows output, pipe through `| Out-String | Write-Host` to force display:

```powershell
cci task run deploy --path force-app/main/default/lwc/myComponent --org dev 2>&1 | Out-String | Write-Host
```

---

## Resolving the Org Alias ⚠️

CumulusCI and SF CLI use **different org names** for the same scratch org.

| Tool | Org alias format | Example |
|---|---|---|
| CumulusCI | `--org dev` | `cci task run deploy --org dev` |
| SF CLI | `--target-org {project.name}__dev` | `sf apex run test --target-org MyProject__dev` |

**Do NOT guess** the SF CLI alias — always confirm it:

```powershell
# Step 1: See all orgs
cci org list 2>&1 | Tee-Object ai-logs/cci.txt; Get-Content ai-logs/cci.txt

# Step 2: Get SF CLI alias for the dev org
cci org info --org dev 2>&1 | Tee-Object ai-logs/orginfo.txt; Get-Content ai-logs/orginfo.txt
# Look for: sfdx_alias   {ProjectName}__dev
```

---

## Querying the Org with Anonymous Apex (Happy Path)

`sf data query` in PowerShell silently swallows output. Use anonymous Apex instead:

**Step 1** — Write Apex to a file in `ai-logs/` (avoids all quoting issues):
```apex
// ai-logs/check_something.apex
List<MyObject__c> records = [SELECT Id, SomeField__c FROM MyObject__c LIMIT 3];
for (MyObject__c record : records) {
    System.debug('VAL:' + record.SomeField__c);
}
```

**Step 2** — Run with `sf apex run --file`:
```powershell
sf apex run --target-org {ProjectName}__dev --file ai-logs/check_something.apex 2>&1 | Tee-Object ai-logs/anon_out.txt
```

**Step 3** — Read the output; debug lines appear as `USER_DEBUG[N]DEBUG<value>`.

**Do NOT use:**
- Inline `--apex "..."` strings (PowerShell quoting breaks them)
- `sf data query` for non-standard objects
- Python subprocesses

---

## Tooling API Queries

`ApexCodeCoverageAggregate`, picklist metadata, and other Tooling API objects require the `--use-tooling-api` flag:

```powershell
# ✅ Only reliable pattern
sf data query `
  --query "SELECT ApexClassOrTrigger.Name, NumLinesCovered, NumLinesUncovered FROM ApexCodeCoverageAggregate WHERE NumLinesCovered + NumLinesUncovered > 0 ORDER BY ApexClassOrTrigger.Name" `
  --target-org {ProjectName}__dev `
  --use-tooling-api `
  --result-format json 2>&1 | Tee-Object ai-logs/tooling.json
# Then read ai-logs/tooling.json
```

- **Replace `{ProjectName}__dev`** with the `sfdx_alias` from `cci org info --org dev`.
- **Always use `--result-format json`** — table/human formats may produce empty output in PowerShell.

---

## jq for JSON Parsing

```powershell
sf apex run test --class-names MyClass_TEST --target-org MyOrg__dev --result-format json --code-coverage --wait 20 2>&1 | Out-File ai-logs/test.json
jq ".result.summary" ai-logs/test.json
```

---

## Python for Code Analyzer JSON

PowerShell and jq have encoding issues with Code Analyzer JSON. Always use Python:

```powershell
# Reuse/update ai-logs/parse_violations.py:
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

---

## Trust Deploy Success

When CCI outputs `[Success]: Succeeded`, move on. Do not re-verify with extra queries unless something actually fails.

