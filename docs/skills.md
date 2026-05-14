# Github Co-Pilot Skills

[Agent Skills](https://github.com/github/awesome-copilot/blob/main/docs/README.skills.md) are self-contained folders with instructions and bundled resources that enhance AI capabilities for specialized tasks. Based on the Agent Skills specification, each skill contains a SKILL.md file with detailed instructions that agents load on-demand.

Skills differ from other primitives by supporting bundled assets (scripts, code samples, reference data) that agents can utilize when performing specialized tasks.

## Install pre-built skills

1. Install github command line tool (gh) if you haven't already: https://cli.github.com/manual/installation
2. Select the skill your want to install from the [Agent Skills Catalog](https://github.com/github/awesome-copilot/blob/main/docs/README.skills.md)
3. Run the install command for that skill in your terminal (each skill's README.md includes the install command)
   - You will be asked what target agents to install this skill for. You can select one or more of your installed agents. Select **GitHub Copilot**.
   - You will be ask if you want to install the skill just for the curren project globally. This is up to you, but if the skill is used in most of your projects you might as well install it globally for all repos.

## Skills to consider installing for Salesforce development

- [Salesforce Apex Quality](https://github.com/github/awesome-copilot/blob/main/skills/salesforce-apex-quality/SKILL.md) ```gh skills install github/awesome-copilot salesforce-apex-quality```
- [Salesforce Component Standards](https://github.com/github/awesome-copilot/blob/main/skills/salesforce-component-standards/SKILL.md) ```gh skills install github/awesome-copilot salesforce-component-standards```
- [Salesforce  Flow Design](https://github.com/github/awesome-copilot/blob/main/skills/salesforce-flow-design/SKILL.md) ```gh skills install github/awesome-copilot salesforce-flow-design```
- [Chrome DevTools](https://github.com/github/awesome-copilot/blob/main/skills/chrome-devtools/SKILL.md) ```gh skills install github/awesome-copilot chrome-devtools```
- [AI Ready](https://github.com/github/awesome-copilot/blob/main/skills/ai-ready/SKILL.md) ```gh skills install github/awesome-copilot ai-ready```
- [Web Coder](https://github.com/github/awesome-copilot/blob/main/skills/web-coder/SKILL.md) ```gh skills install github/awesome-copilot web-coder```

***Note:*** I'm still experimenting with using skills in the Salesforce development workflow, but I can already see how useful they will be for giving your AI coding assistants access to best practices, code samples, and reference materials that they can draw on when generating code for you.