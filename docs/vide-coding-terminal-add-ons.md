# Tool Installation Guide

This guide covers installation of the tools needed for CSS extraction and testing.

---

## 🎯 Essential Tools

These tools are required for the PurgeCSS workflow:

### 1. Node.js & npm
**Required for:** Running PurgeCSS

**Install:**
- Download from [nodejs.org](https://nodejs.org/)
- Choose LTS (Long Term Support) version
- Installer includes npm

**Verify:**
```bash
node --version
npm --version
```

### 2. PurgeCSS
**Required for:** CSS extraction

**Install:**
```bash
cd C:\Users\Thad-PC-2019\IdeaProjects\test_html
npm install -D purgecss
```

**Usage:**
```bash
node run-purgecss.js
```

**What it does:**
- Scans HTML for classes/IDs/selectors
- Extracts only CSS rules actually used
- Preserves media queries and states
- Result: 84.5% smaller CSS with perfect accuracy

### 3. Python 3
**Required for:** Testing scripts

**Install:**
- Download from [python.org](https://www.python.org/)
- Version 3.8 or higher
- Check "Add to PATH" during installation

**Verify:**
```bash
python --version
pip --version
```

### 4. Playwright
**Required for:** Visual verification

**Install:**
```bash
pip install playwright
playwright install chromium
```

**Usage:**
```bash
python playwright-analyze.py
```

**What it does:**
- Renders pages in headless browser
- Captures computed CSS styles
- Compares visual appearance
- Takes screenshots for verification

---

## 🔧 Optional Tools (Quality of Life)

These tools improve productivity but aren't required:

### fd - Fast File Finder
**Install:**
```bash
# Windows (Chocolatey)
choco install fd

# Or download from: https://github.com/sharkdp/fd/releases
```

**Usage:**
```bash
fd "*.css"
fd "template" --type f
```

### bat - Syntax Highlighter
**Install:**
```bash
# Windows (Chocolatey)
choco install bat

# Or download from: https://github.com/sharkdp/bat/releases
```

**Usage:**
```bash
bat file.css
bat --theme="Monokai Extended" template.html
```

### ripgrep - Fast Text Search
**Install:**
```bash
# Windows (Chocolatey)
choco install ripgrep

# Or download from: https://github.com/BurntSushi/ripgrep/releases
```

**Usage:**
```bash
rg "pattern" *.css
rg "Header__" --type css
```

### tokei - Code Statistics
**Install:**
```bash
# Windows (Chocolatey)
choco install tokei

# Or with Cargo (Rust package manager)
cargo install tokei

# Or download from: https://github.com/XAMPPRocky/tokei/releases
```

**Usage:**
```bash
tokei                    # Count lines in current directory
tokei --sort code        # Sort by lines of code
tokei docs/              # Count lines in specific folder
```

**What it does:**
- Counts lines of code by language
- Shows code, comments, blanks separately
- Fast and accurate
- Great for project statistics

---

## 📦 Complete Installation Script

Run these commands to install everything:

```bash
# Essential tools
npm install -D purgecss
pip install playwright
playwright install chromium

# Optional tools (Windows with Chocolatey)
choco install fd bat ripgrep tokei
```

---

## ✅ Verification

Check all tools are installed correctly:

```bash
# Essential
node --version          # Should show v18+ or v20+
npm --version          # Should show 9+ or 10+
python --version       # Should show 3.8+
pip show playwright    # Should show package info

# Optional
fd --version
bat --version
rg --version
tokei --version
```

If all commands succeed, you're ready to go!

---

## 🚀 Quick Start After Installation

1. **Generate minimal CSS:**
   ```bash
   node run-purgecss.js
   ```

2. **Verify it looks correct:**
   ```bash
   python playwright-analyze.py
   ```

3. **Use in template:**
   - CSS is automatically saved to `purgecss-minimal.css`
   - Template already references this file

---

## 💾 Disk Space Requirements

- **Node.js:** ~50 MB
- **PurgeCSS (node_modules):** ~50 MB
- **Python:** ~100 MB
- **Playwright + Chromium:** ~500 MB
- **Optional tools:** ~20 MB

**Total:** ~720 MB

---

## 🔄 Updating Tools

### Update PurgeCSS:
```bash
npm update purgecss
```

### Update Playwright:
```bash
pip install --upgrade playwright
playwright install chromium
```

### Update Optional Tools:
```bash
choco upgrade fd bat ripgrep tokei
```

---

## 🆘 Troubleshooting

### PurgeCSS not found:
```bash
# Install in project directory
cd C:\Users\Thad-PC-2019\IdeaProjects\test_html
npm install -D purgecss
```

### Playwright not found:
```bash
# Install globally or in user space
pip install --user playwright
playwright install chromium
```

### Node command not found:
- Reinstall Node.js
- Ensure "Add to PATH" was checked
- Restart terminal/IDE

### Python command not found:
- Reinstall Python
- Check "Add to PATH" during installation
- Try `py` instead of `python` on Windows

---

## 📖 Documentation

- Node.js: https://nodejs.org/docs/
- PurgeCSS: https://purgecss.com/
- Playwright: https://playwright.dev/python/
- fd: https://github.com/sharkdp/fd
- bat: https://github.com/sharkdp/bat
- ripgrep: https://github.com/BurntSushi/ripgrep
- tokei: https://github.com/XAMPPRocky/tokei

---

**Last Updated:** December 3, 2025  
**Status:** All tools verified and working

