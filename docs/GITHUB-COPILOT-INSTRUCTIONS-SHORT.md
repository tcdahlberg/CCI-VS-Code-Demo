# Concise AI Instructions for IntelliJ

## Environment & Tools
- Windows + PowerShell (use PowerShell syntax, NOT cmd)
- IntelliJ IDEA + Illuminated Cloud plugin
- Tools installed: Node.js, Python 3.13, PurgeCSS, Playwright, fd, bat, ripgrep, tokei

## Key Commands
- Use `Remove-Item -Force` NOT `del /Q`
- File search: `fd "pattern"`, `rg "text"`
- Code stats: `tokei`
- CSS: `node run-purgecss.js`

## Project Preferences
- Keep root clean (essential files only)
- Docs in `/docs` folder
- Check `docs/AI-TOOLS-CONFIG.md` for project context

## Proven Solutions
- CSS extraction: Use PurgeCSS (84.5% reduction, 0 differences)
- Avoid: Manual regex extraction (failed with 58 differences)
- Browser testing: Use Playwright
- File operations: PowerShell-native commands only

## When Starting
1. Check `/docs/AI-TOOLS-CONFIG.md` for context
2. Use existing tools (fd, rg, PurgeCSS) before suggesting new ones
3. Verify commands work before declaring success
4. Clean up failed attempts

## Remember
- Professional tools > custom scripts for complex tasks
- Test file operations before moving on
- PowerShell syntax on Windows
- Keep directories organized

