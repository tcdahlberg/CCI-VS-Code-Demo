# Agentic Coding with Salesforce MCP, IntelliJ IDEA, Illuminated Cloud, and GitHub Copilot

This is a setup for agentic coding using Salesforce Model Context Protocol (MCP), IntelliJ IDEA with the Illuminated Cloud plugin, GitHub Copilot with intelliJ plugin, and Salesforce CLI.

1. **Install Git**
Download and install Git from the official website: https://git-scm.com/

1. **Install Node.js**
Download and install Node.js from the official website: https://nodejs.org/

2. **Install GitHub CLI**
Follow the instructions on the official GitHub CLI installation page: https://cli.github.com/manual/

1. Install Salesforce Command Line Interface (CLI)

    ```bash
    npm install --global @salesforce/cli
    ```
    
    Update Salesforce CLI periodically
    
    ```bash
    npm update --global @salesforce/cli
    ```

2. Install Salesforce Model Context Protocol (as above swap install with update to keep your mcp up to date)

    ```bash
    npm install --global @salesforce/mcp
    ```

3. Update SF CLI plugins periodically

    ```bash
    sf plugins:install @salesforce/plugin-name@latest
    ```
    
    or to update
    
    ```bash
    sf plugins update
    ```

Install IntelliJ IDEA
Download and install IntelliJ IDEA from the official JetBrains website: https://www.jetbrains.com/idea/download/

Install Illuminated Cloud Plugin
1. Open IntelliJ IDEA.
2. Go to "File" > "Settings" (or "Preferences" on macOS).
3. Select "Plugins" from the left sidebar.
4. Click on the "Marketplace" tab.
5. Search for "Illuminated Cloud" and click "Install".

Illuminated cloud is a paid plugin and will require purchaing a subscription license.

Install GitHub Copilot Plugin
1. Open IntelliJ IDEA.
2. Go to "File" > "Settings" (or "Preferences" on macOS
3. Select "Plugins" from the left sidebar.
4. Click on the "Marketplace" tab.
5. Search for "GitHub Copilot" and click "Install".

GitHub copilot needs to be purchased through GitHub.

[Follow the instructions for hooking up GitHub Copilot with Salesforce MCP.](https://bitbucket.org/RoseSilverSoftware/illuminatedcloud/issues/2843)

Once this is all set up correctly you should see a little copilot icon in IntelliJ IDEA. You should be able to set it to "Ask" or "Agent" mode. Ask will
simply give you suggestions as you code. Agent mode will allow you to give it commands to do things for you: coding tasks, writing tests, refactoring, deploying, and using Salesforce CLI commands.

## Terminal tools to install (optional but recommended)

When an agent was asked what terminal tools would help make it more effective/efficient it suggested several tools. Here are the most useful ones:
🔧 Essential Tools for Better Assistance

### ripgrep (rg) - Fast file searching 

Windows install (using Chocolatey)
```bash
choco install ripgrep
```

Or download from: https://github.com/BurntSushi/ripgrep/releases

Why: 10-100x faster than findstr for searching code. I could quickly find classes, analyze CSS usage, etc.
### bat - Better file viewing

Windows install (using Chocolatey)
```bash
choco install bat
```

Why: Shows syntax-highlighted file contents with line numbers. Makes reading code snippets much easier.

### fd - Fast file finding
Windows install (using Chocolatey)
```bash
choco install fd
```

Why: Much faster and easier than dir /s. Great for finding files by pattern.

### jq - JSON parsing
Windows install (using Chocolatey)
```bash
choco install jq
```
Why: Essential for parsing package.json, config files, and API responses.

### tokei - Code statistics
Windows install (using Chocolatey)
```bash
choco install tokei
```
I had trouble installing tokei with chocolatey and had success with winget

```bash
winget install XAMPPRocky.tokei
```

Why: Instantly shows lines of code, file counts, etc. Would have been perfect for the CSS size comparison!

### dust - Better disk usage
Windows install (using Chocolatey)
```bash
choco install dust
```
Why: Quickly shows file sizes in a readable format.

### 📦 Installation Methods

#### Chocolatey (Recommended for Windows)
- Install Chocolatey first if you don't have it

- Then install all at once:
    ```bash 
    choco install ripgrep bat fd jq tokei dust -y
    ```

#### Manual Downloads
Each tool has releases on GitHub if you prefer manual installation.


### Example of what I could do with these tools:

Find all classes in HTML (with ripgrep)
```bash
rg -o "class=\"[^\"]+\"" 2025_stthomas_template.html
```

Show CSS file with line numbers (with bat)
```bash
bat extracted-app.css
```

Get accurate code statistics (with tokei)
```bash
tokei app.css extracted-app.css
```

Check file sizes (with dust)
```bash
dust -r
```

### 💡 Bottom Line
Installing these tools would make me significantly more effective at:

- 🔍 Searching and analyzing code
- 📊 Getting accurate statistics
- 🚀 Running commands faster
- 📝 Providing better formatted output

