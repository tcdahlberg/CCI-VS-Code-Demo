# Global AI Instructions for IntelliJ + Illuminated Cloud
**Status:** Tools verified and working
**OS:** Windows  
**IDE:** IntelliJ IDEA + Illuminated Cloud  
**Last Updated:** December 3, 2025  

---

- Keep root directories organized
- Verify commands work before declaring success
- Clean up failed attempts
- Check for existing tools before suggesting installations
- Use PowerShell-native commands
### Remember:

- **Documentation:** Check `/docs` folder first
- **Testing:** Use Playwright
- **File Search:** Use fd, rg, bat, tokei
- **CSS Tasks:** Use PurgeCSS
- **Environment:** Windows + PowerShell + IntelliJ
### Quick Reference:

## 🤖 For AI Assistants

---

- Link related documents
- Provide "Quick Start" sections
- Include code blocks with syntax highlighting
- Use emojis for section headers (📁 🎯 ✅ ❌)
### Format Preferences:

- **Updates:** Keep docs synchronized when tools change
- **Commands:** Real, tested commands users can copy
- **Structure:** Clear sections with practical examples
- **Focus:** Only what worked (remove failed attempts)
### When Creating Docs:

## 📝 Documentation Style

---

4. Ask user before major changes
3. Check documentation for proven solutions
2. Look for professional tools that solve the problem
1. Stop and reassess (don't keep trying failed methods)
### If Approach Isn't Working:

4. Test on small examples before full execution
3. Use `-ErrorAction SilentlyContinue` for safety
2. Verify PowerShell syntax (not cmd.exe)
1. Check if tool is actually installed (`--version` commands)
### If Commands Fail:

## 🔄 When Troubleshooting

---

- **Result:** 5 files in root, clean and professional
- **Solution:** Keep only essential files in root, move rest to subdirectories
- **Problem:** 100+ files cluttering root directory
### Directory Organization:

- **Lesson:** Use industry-standard tools for complex tasks
- **Result:** 84.5% reduction, 0 visual differences
- **Success:** PurgeCSS (professional tool)
- **Failed Attempts:** Manual regex, pattern matching, custom parsers
- **Problem:** Extract minimal CSS from large file (816 KB)
### CSS Extraction (December 2025):

## 🎯 Key Lessons Learned

---

```
node run-purgecss.js     # Regenerate optimized CSS
```bash
### CSS/HTML Work:

```
tokei docs/              # Stats for specific folder
tokei --sort code        # Show code stats sorted
```bash
### Project Statistics:

```
tokei                    # Count lines of code
bat file.css             # View with syntax highlighting
rg "text" --type css     # Search in files
fd "pattern"              # Find files
```bash
### File Search:

## 💡 Helpful Commands

---

4. Use existing tools/patterns instead of reinventing
3. Review `README.md` for project overview
2. Check for `AI-TOOLS-CONFIG.md` or `.ai-tools-config.json`
1. Look for `/docs` folder
### When Starting New Sessions:

- `README.md` - Project overview
- `package.json` - npm dependencies
- `docs/TOOLS-SETUP.md` - Installation instructions
- `docs/AI-TOOLS-CONFIG.md` - Tool capabilities and proven workflows
### Check These Files for Context:

## 📊 Project Context Files

---

- ❌ Always use `-ErrorAction SilentlyContinue` for safe deletion
- ❌ Don't mix cmd.exe and PowerShell syntax
- ❌ Don't use `del /Q` - Use PowerShell `Remove-Item -Force`
### Windows Command Issues:

- ❌ Leaving test files cluttering directories
- ❌ Mixed PowerShell/CMD syntax (causes errors)
- ❌ Pattern-based class extraction (misses cascade rules)
- ❌ Manual CSS extraction with regex (too complex, 58 differences)
### Avoid These Approaches:

## 🚫 What Doesn't Work

---

- Update docs when tools/processes change
- Provide working examples with real commands
- Remove confusing failed attempts
- Keep docs focused and practical
### Documentation:

- Test file operations work before moving on
- Use `Get-ChildItem` not `dir` when piping
- Use `Remove-Item -Force` not `del /Q` (PowerShell native)
- Use **PowerShell syntax** for Windows (not bash/cmd hybrid)
### File Operations:

4. **Clean up after experiments** - Remove failed attempts
3. **Test commands before declaring success** - Run verification
2. **Prefer professional tools** over custom scripts for complex tasks
1. **Check for existing tools first** - Use fd, rg, bat, tokei when available
### When Suggesting Solutions:

## 🎯 Working Preferences

---

- **Verify:** Always test changes before considering them complete
- **Command:** Python scripts for browser automation
- **Tool to Use:** Playwright for visual comparison
### Testing & Verification:

- **Result:** 84.5% file size reduction with perfect accuracy
- **Script:** `run-purgecss.js` for CSS regeneration
- **Don't Use:** Manual regex extraction, pattern matching
- **Tool to Use:** PurgeCSS (professional tool)
### CSS Extraction (St. Thomas Template Project):

## ✅ Proven Workflows

---

- Use descriptive file names
- Keep documentation in `/docs` folder
- Move scripts to `/scripts` or subdirectories
- Only essential files in root (templates, configs, READMEs)
When working on projects, keep root directories clean:

## 📁 Project Structure Preferences

---

- **tokei** - Code statistics and line counting
- **ripgrep (rg)** - Fast text search (better than grep)
- **bat** - Syntax-highlighted file viewer (better than cat)
- **fd** - Fast file finder (better than find)
### CLI Tools Available:

- **Playwright** - Browser automation for testing (Python package with Chromium)
- **PurgeCSS** - CSS optimization tool (npm package)
- **Python 3.13** - Script execution
- **npm** - Package manager
- **Node.js** - JavaScript runtime (v18+)
### Essential Tools Installed:

- **Terminal:** Integrated terminal with full access to CLI tools
- **OS:** Windows with PowerShell
- **IDE:** IntelliJ IDEA with Illuminated Cloud plugin (Salesforce)
### Development Environment:

## 🛠️ Available Tools & Environment


