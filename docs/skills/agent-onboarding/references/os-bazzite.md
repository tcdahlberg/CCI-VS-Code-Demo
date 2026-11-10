# Bazzite Linux (bash)

Read this on the Bazzite Linux machine: shell syntax, paths, environment variables, installed tools, and package installs.

## Shell and paths
- **bash/POSIX syntax only.** Paths use forward slashes (`/home/user/project`). Merge stderr with `2>&1`. Details of capturing output, org aliases and long-running commands are in the `terminal-commands` skill.
- **Environment variables:** `export MY_VAR=value` to set, `$MY_VAR` or `${MY_VAR}` to read. For credentials use `$SF_USERNAME`, `$SF_PASSWORD`, `$MY_API_KEY`; never write them to a file.
- **Permissions:** `chmod` and `chown`; a script you run directly needs `chmod +x <script>`.
- **Line endings are LF**, matching Salesforce orgs. To clean files from Windows use `dos2unix <file>` or `sed -i 's/\r$//' <file>`. Never use file size or line count as proof that a deploy changed something; diff content.
- **Do not paste PowerShell** (`Out-File`, `Remove-Item`, `$env:`) into commands here.

## Installed tools
Installed and in use: Node.js, Python 3, CumulusCI (`cci`), Salesforce CLI (`sf`), PurgeCSS, Playwright, BackstopJS, `rg`, `fd`, `jq`, `bat`, `delta`, `fzf`, `gh`, `just`, `lazygit`, `zoxide`, `tokei`, `xmllint`, `sqlite3`, and (via brew) `yq` v4, `sd`, `dos2unix`, `prettier` 3.

Usage notes:
- `yq`: edits YAML such as `cumulusci.yml` and Snowfakery recipes, and reads or edits metadata XML with `yq -p xml -o xml`.
- `sd`: find and replace with plain syntax; use it instead of a script for simple replacements.
- `bat` to view files, `delta` for diffs, `lazygit` for git TUI, `zoxide` (`z <partial-path>`) to jump directories.
- **`sg` is not ast-grep here.** `/usr/bin/sg` is `newgrp`. Use the `code` tool for structural search and rewrite.
- `prettier` has no Apex support until `prettier-plugin-apex` and a `.prettierrc` exist in the project. A project that has them (UST-Salesforce-RFI) exposes `npm run format:apex:check`. The plugin always puts annotations such as `@AuraEnabled` on their own line, so do not run `format:apex` over existing files unless asked to do a dedicated formatting pass.
- Not installed, only if needed: `difftastic`, `shellcheck`, `hyperfine`, `ast-grep`.

## Installing software
- **Bazzite has an immutable base (`rpm-ostree`).** A system package needs `rpm-ostree install <pkg>` and a reboot, or use `toolbox`/`distrobox`. Prefer user-space installs: `brew`, `pip`/`pipx`, `npm`, `flatpak`. Do not use `dnf` directly.
- The user installs tools; do not install anything yourself.
