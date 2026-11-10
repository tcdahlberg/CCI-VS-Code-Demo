# Windows (PowerShell)

Read this on a Windows machine: shell syntax, paths, environment variables, line endings, and tools.

## Shell and paths
- **PowerShell syntax only. Never cmd** (`del /Q`, `type`, `copy`). Merge stderr with `2>&1`. Capturing output, org aliases and long-running commands are covered by the `terminal-commands` skill (the Windows copy of it).
- **Environment variables:** `$env:MY_VAR = 'value'` to set for the session, `$env:MY_VAR` to read. For credentials use `$env:SF_USERNAME`, `$env:SF_PASSWORD`, `$env:MY_API_KEY`; never write them to a file.
- Windows accepts `/` in most tools, but quote paths with spaces and prefer `Join-Path` in scripts.
- **Skill links:** where Linux uses symlinks, use a junction: `New-Item -ItemType Junction -Path $HOME\.kiro\skills\agent-onboarding -Target <repo>\docs\skills\agent-onboarding`. Linking a skill needs no admin rights with a junction.

## Line endings
- **Windows files can be CRLF (`\r\n`); Salesforce orgs use LF (`\n`).** Never compare byte counts or line counts between a local file and the org, and never use file size or line count as proof that a deploy changed something. Diff the content.
- A stray `\r` can break shell scripts and some parsers. Strip it with `(Get-Content file -Raw) -replace "`r`n","`n" | Set-Content -NoNewline file`, or `dos2unix file` if that is installed.

## Output encoding
PowerShell 5 redirects (`>` and `| Out-File` without `-Encoding`) write UTF-16 with a BOM, which `jq`, Python's default reader and other tools cannot parse. Write with `| Out-File -Encoding utf8 <file>` or `| Tee-Object -FilePath <file>`. Parse Code Analyzer JSON with Python (`encoding='utf-8'`), not `jq`. PowerShell 7 defaults to UTF-8 without a BOM.

## Bash to PowerShell
| Bash | PowerShell |
|---|---|
| `cmd 2>&1 \| tee ai-logs/x.txt` | `cmd 2>&1 \| Tee-Object ai-logs/x.txt` |
| `cmd > file` | `cmd \| Out-File -Encoding utf8 file` |
| `\` at end of a line (continuation) | `` ` `` at end of a line |
| `export NAME=value`, `$NAME` | `$env:NAME = 'value'`, `$env:NAME` |
| `rm -f path`, `rm -rf dir` | `Remove-Item -Force path`, `Remove-Item -Recurse -Force dir` |
| `cat file`, `tail -20 file` | `Get-Content file`, `Get-Content file -Tail 20` |
| `sleep 35` | `Start-Sleep -Seconds 35` (do not sleep and poll for a background deploy; run in the foreground) |
| `cp -r a b`, `mv a b` | `Copy-Item -Recurse a b`, `Move-Item a b` |

## Tools
As last recorded (not rechecked since the Bazzite setup was extended): Node.js, Python 3.13, CumulusCI, Salesforce CLI (`sf`), PurgeCSS, Playwright, BackstopJS, `fd`, `bat`, `rg`, `tokei`, `jq`. Use `rg`/`fd` for searches, not `Get-ChildItem` loops. `yq`, `sd`, `prettier` and `dos2unix` are installed on the Bazzite machine only unless you confirm them here with `Get-Command <tool>`. Do not install anything yourself; ask the user.
