# Global AI Instructions for IntelliJ + Illuminated Cloud

**Last Updated:** December 3, 2025  
**IDE:** IntelliJ IDEA + Illuminated Cloud  
**OS:** Windows with PowerShell  
**Status:** Tools verified and working

---

## 🏢 Primary Work: Salesforce Development

### Development Environment:
- **IDE:** IntelliJ IDEA with **Illuminated Cloud plugin** (Salesforce development)
- **Org Type:** **Scratch Orgs** (not production/sandboxes)
- **Version Control:** Git + GitHub
- **Build Tool:** **CumulusCI (cci)** - Python CLI wrapper over Salesforce CLI

### CumulusCI (cci):
**What it is:** Python command-line tool that orchestrates Salesforce development
- Lives on top of: Git, GitHub, Salesforce CLI (sf/sfdx)
- Documentation: https://cumulusci.readthedocs.io/en/latest/

**What cci does:**
- Creates and configures scratch orgs with dependencies
- Loads test data into orgs
- Deploys code from repo
- Creates managed and unmanaged packages
- Automates deployment workflows
- Manages org dependencies and metadata

**Common cci commands:**
```bash
cci org list                    # List orgs
cci org default <org_name>      # Set default org
cci org scratch dev <org_name>  # Create scratch org
cci org info                    # Show org details
cci task list                   # List available tasks
cci task run <task_name>        # Run a task
cci flow list                   # List flows
cci flow run <flow_name>        # Run a flow
```

### Salesforce CLI (sf/sfdx):
**CumulusCI uses these under the hood:**
```bash
sf org list                     # List orgs
sf project deploy start         # Deploy metadata
sf project retrieve start       # Retrieve metadata
sf data query                   # Run SOQL queries
sf apex run                     # Execute Apex
```

### When Working on Salesforce Projects:
1. **Check for `cumulusci.yml`** - CumulusCI project configuration
2. **Check for `sfdx-project.json`** - Salesforce project configuration  
3. **Default to cci commands** for org management and deployment
4. **Use scratch orgs** not production orgs
5. **Check org dependencies** in cumulusci.yml before suggesting changes

---

## 🛠️ Available Tools & Environment

### Development Environment:
- **IDE:** IntelliJ IDEA with Illuminated Cloud plugin (Salesforce)
- **OS:** Windows with PowerShell
- **Terminal:** Integrated terminal with full access to CLI tools

### Essential Tools Installed:
- **Node.js** - JavaScript runtime (v18+)
- **npm** - Package manager
- **Python 3.13** - Script execution (required for CumulusCI)
- **CumulusCI (cci)** - Salesforce development workflow tool
- **Salesforce CLI (sf)** - Salesforce command-line interface
- **PurgeCSS** - CSS optimization tool (npm package)
- **Playwright** - Browser automation for testing (Python package with Chromium)

### CLI Tools Available:
- **fd** - Fast file finder (better than find)
- **bat** - Syntax-highlighted file viewer (better than cat)
- **ripgrep (rg)** - Fast text search (better than grep)
- **tokei** - Code statistics and line counting

---

## 📁 Project Structure Preferences

When working on projects, keep root directories clean:
- Only essential files in root (templates, configs, READMEs)
- Move scripts to `/scripts` or subdirectories
- Keep documentation in `/docs` folder
- Use descriptive file names

---

## ✅ Proven Workflows

### Salesforce Development with CumulusCI:
- **Tool to Use:** CumulusCI (cci) commands for org management
- **Workflow:** Create scratch org → deploy dependencies → load data → deploy code
- **Check:** Always verify cumulusci.yml before suggesting changes

### CSS Extraction (St. Thomas Template Project):
- **Tool to Use:** PurgeCSS (professional tool)
- **Don't Use:** Manual regex extraction, pattern matching
- **Script:** `run-purgecss.js` for CSS regeneration
- **Result:** 84.5% file size reduction with perfect accuracy

### Testing & Verification:
- **Tool to Use:** Playwright for visual comparison
- **Command:** Python scripts for browser automation
- **Verify:** Always test changes before considering them complete

---

## 🎯 Working Preferences

### When Suggesting Solutions:
1. **Check for existing tools first** - Use fd, rg, bat, tokei when available
2. **Prefer professional tools** over custom scripts for complex tasks
3. **Test commands before declaring success** - Run verification
4. **Clean up after experiments** - Remove failed attempts

### File Operations:
- Use **PowerShell syntax** for Windows (not bash/cmd hybrid)
- Use `Remove-Item -Force` not `del /Q` (PowerShell native)
- Use `Get-ChildItem` not `dir` when piping
- Test file operations work before moving on

### Documentation:
- Keep docs focused and practical
- Remove confusing failed attempts
- Provide working examples with real commands
- Update docs when tools/processes change

---

## 🚫 What Doesn't Work

### Avoid These Approaches:
- ❌ Manual CSS extraction with regex (too complex, 58 differences)
- ❌ Pattern-based class extraction (misses cascade rules)
- ❌ Mixed PowerShell/CMD syntax (causes errors)
- ❌ Leaving test files cluttering directories

### Windows Command Issues:
- ❌ Don't use `del /Q` - Use PowerShell `Remove-Item -Force`
- ❌ Don't mix cmd.exe and PowerShell syntax
- ❌ Always use `-ErrorAction SilentlyContinue` for safe deletion

---

## 📊 Project Context Files

### Check These Files for Context:
- `cumulusci.yml` - CumulusCI configuration (Salesforce projects)
- `sfdx-project.json` - Salesforce project configuration
- `docs/AI-TOOLS-CONFIG.md` - Tool capabilities and proven workflows
- `docs/TOOLS-SETUP.md` - Installation instructions
- `package.json` - npm dependencies
- `README.md` - Project overview

### When Starting New Sessions:
1. Look for `/docs` folder
2. Check for `cumulusci.yml` or `sfdx-project.json` (Salesforce project)
3. Check for `AI-TOOLS-CONFIG.md` or `.ai-tools-config.json`
4. Review `README.md` for project overview
5. Use existing tools/patterns instead of reinventing

---

## 💡 Helpful Commands

### Salesforce/CumulusCI:
```bash
cci org list                # List orgs
cci org info                # Current org details
cci task list               # Available tasks
cci flow run dev_org        # Setup dev org
```

### File Search:
```bash
fd "pattern"                # Find files
rg "text" --type css        # Search in files
bat file.css                # View with syntax highlighting
tokei                       # Count lines of code
```

### Project Statistics:
```bash
tokei --sort code           # Show code stats sorted
tokei docs/                 # Stats for specific folder
```

### CSS/HTML Work:
```bash
node run-purgecss.js        # Regenerate optimized CSS
```

---

## 🎯 Key Lessons Learned

### CSS Extraction (December 2025):
- **Problem:** Extract minimal CSS from large file (816 KB)
- **Failed Attempts:** Manual regex, pattern matching, custom parsers
- **Success:** PurgeCSS (professional tool)
- **Result:** 84.5% reduction, 0 visual differences
- **Lesson:** Use industry-standard tools for complex tasks

### Directory Organization:
- **Problem:** 100+ files cluttering root directory
- **Solution:** Keep only essential files in root, move rest to subdirectories
- **Result:** 5 files in root, clean and professional

---

## 🔄 When Troubleshooting

### If Commands Fail:
1. Check if tool is actually installed (`--version` commands)
2. Verify PowerShell syntax (not cmd.exe)
3. Use `-ErrorAction SilentlyContinue` for safety
4. Test on small examples before full execution

### If Approach Isn't Working:
1. Stop and reassess (don't keep trying failed methods)
2. Look for professional tools that solve the problem
3. Check documentation for proven solutions
4. Ask user before major changes

---

## 📝 Documentation Style

### When Creating Docs:
- **Focus:** Only what worked (remove failed attempts)
- **Structure:** Clear sections with practical examples
- **Commands:** Real, tested commands users can copy
- **Updates:** Keep docs synchronized when tools change

### Format Preferences:
- Use emojis for section headers (📁 🎯 ✅ ❌)
- Include code blocks with syntax highlighting
- Provide "Quick Start" sections
- Link related documents

---

## 🤖 For AI Assistants

### Quick Reference:
- **Environment:** Windows + PowerShell + IntelliJ
- **Salesforce:** CumulusCI (cci) for org management, scratch orgs
- **CSS Tasks:** Use PurgeCSS
- **File Search:** Use fd, rg, bat, tokei
- **Testing:** Use Playwright
- **Documentation:** Check `/docs` folder first

### Remember:
- Use PowerShell-native commands
- Check for cumulusci.yml (Salesforce projects)
- Check for existing tools before suggesting installations
- Clean up failed attempts
- Verify commands work before declaring success
- Keep root directories organized

---

**Last Updated:** December 3, 2025  
**IDE:** IntelliJ IDEA + Illuminated Cloud  
**OS:** Windows with PowerShell  
**Status:** Tools verified and working
