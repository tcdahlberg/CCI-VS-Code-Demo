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
- Salesforce: `cci org list`, `cci org info`, `cci flow run dev_org`
- Use `Remove-Item -Force` NOT `del /Q`
- File search: `fd "pattern"`, `rg "text"`
- Code stats: `tokei`
- CSS: `node run-purgecss.js`

## Project Preferences
- Keep root clean (essential files only)
- Docs in `/docs` folder
- Check `docs/AI-TOOLS-CONFIG.md` for project context
- Check `cumulusci.yml` for Salesforce projects

## Proven Solutions
- Salesforce orgs: Use CumulusCI (cci) commands
- CSS extraction: Use PurgeCSS (84.5% reduction, 0 differences)
- Avoid: Manual regex extraction (failed with 58 differences)
- Browser testing: Use Playwright
- File operations: PowerShell-native commands only

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

