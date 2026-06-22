---
name: backstopjs-visual-testing
description: "Visual regression testing with BackstopJS for Salesforce page migrations (e.g. Visualforce to LWC). Covers setup, backstop.json config template, authentication via engine scripts, scratch org test data guidance, and the full reference-test-approve workflow."
metadata:
  author: UST-Agentic-Coding-Skills
  version: "1.0"
allowed-tools: Bash Read Write
---

# BackstopJS Visual Regression Testing

BackstopJS (`backstopjs` — globally installed) uses Playwright under the hood to capture screenshots and generate side-by-side pixel-diff HTML reports. Use this to verify visual parity between two versions of a page (e.g. a Visualforce original vs its LWC replacement).

---

## Key Commands

```powershell
backstop init           # Generate backstop.json in project root (first time only)
backstop reference      # Capture reference screenshots (ground truth)
backstop test           # Capture test screenshots and diff against reference
backstop approve        # Promote test screenshots to new reference (when diff is intentional)
backstop openReport     # Open the HTML diff report in browser
```

---

## Config Location

- Config file: `backstop.json` in project root.
- Output goes to `ai-logs/backstop/` (gitignored):
  - `ai-logs/backstop/reference/` — ground-truth screenshots
  - `ai-logs/backstop/test/` — current build screenshots
  - `ai-logs/backstop/html_report/` — diff report
  - `ai-logs/backstop/engine_scripts/` — auth and interaction scripts

---

## backstop.json Template

```json
{
  "id": "reference-vs-new",
  "viewports": [
    { "label": "desktop", "width": 1280, "height": 900 },
    { "label": "mobile",  "width": 375,  "height": 812 }
  ],
  "scenarios": [
    {
      "label": "Page Name",
      "referenceUrl": "https://<host>/reference-page-url",
      "url":          "https://<host>/new-page-url",
      "delay": 1500,
      "misMatchThreshold": 5.0
    }
  ],
  "paths": {
    "bitmaps_reference": "ai-logs/backstop/reference",
    "bitmaps_test":      "ai-logs/backstop/test",
    "html_report":       "ai-logs/backstop/html_report",
    "engine_scripts":    "ai-logs/backstop/engine_scripts"
  },
  "engine": "playwright",
  "engineOptions": { "browser": "chromium" },
  "report": ["browser"]
}
```

---

## Authentication

Whether a login script is needed depends on the project. Check `docs/AI-TOOLS-CONFIG.md` for the project's access model — some pages are public/guest-accessible; others require authentication.

If authentication is required, use an `onBefore.js` engine script:

```javascript
// ai-logs/backstop/engine_scripts/onBefore.js
module.exports = async (page, scenario, vp) => {
    await page.goto('https://<host>/login');
    await page.fill('#username', process.env.SF_USERNAME);
    await page.fill('#password', process.env.SF_PASSWORD);
    await page.click('#Login');
    await page.waitForNavigation();
};
```

Reference in each scenario that needs it:
```json
"onBeforeScript": "onBefore.js"
```

**Security note:** credentials are read from environment variables (`process.env.SF_USERNAME`, `process.env.SF_PASSWORD`) — never hardcode them in `backstop.json` or the script file.

---

## Scratch Org Test Data

Scratch orgs may contain pre-defined sample data at varying configurations. Not all records will activate all pages/features. Check `docs/AI-TOOLS-CONFIG.md` for project-specific guidance on which records to use for full coverage.

---

## Workflow for Page-to-Page Migration

1. Deploy or access the reference version.
2. Run `backstop reference` to capture ground truth screenshots.
3. Build and deploy the new version.
4. Run `backstop test` — review the HTML report (`backstop openReport`) for layout/content parity.
5. Fix gaps, redeploy, repeat until diff % is at or near 0.
6. Run `backstop approve` when intentional design improvements are accepted as the new baseline.

---

## Troubleshooting

- **Screenshots are blank or show a login page** — authentication is required; add the `onBefore.js` engine script and reference it in your scenario.
- **Diff % is high on identical pages** — increase `misMatchThreshold` (e.g. `5.0`) to ignore anti-aliasing noise, or increase `delay` to allow dynamic content to settle.
- **`backstop test` passes locally but fails in CI** — font rendering differences between OS/browser versions; use a consistent Docker image or accept a higher threshold.
- **Output files accumulate** — `ai-logs/backstop/` is gitignored; clean it manually when needed with `Remove-Item -Recurse -Force ai-logs/backstop/`.

