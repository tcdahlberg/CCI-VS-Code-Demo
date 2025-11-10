# Vibe Coding

This is a setup for agentic coding using Salesforce Model Context Protocol (MCP), IntelliJ IDEA with the Illuminated Cloud plugin, GitHub Copilot with intelliJ plugin, and Salesforce CLI.


Install Node.js

Install Salesforce Command Line Interface (CLI)

```bash
npm install --global @salesforce/mcp
```

Install Salesforce Model Context Protocol

```bash
pip install --upgrade setuptools pip
```

Switch install with update to periodically update these package

Update SF CLI plugins periodically

```bash
sf plugins:install @salesforce/plugin-name@latest
```

or

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
