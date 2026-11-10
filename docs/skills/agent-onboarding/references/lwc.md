# LWC and Lightning Gotchas

Read this before: debugging stale behaviour on a deployed component, calling `fetch()` from an LWC, using `lightning-input-address`, `lightning-record-edit-form` or related-list wires, editing Lightning pages, building a form whose boxes follow Salesforce field limits, or testing an LWC with Jest or in a live browser.

## Stale behaviour on a Lightning Out component

A Lightning Out embed (`$Lightning.use("c:someAuraApp", ...)` then `$Lightning.createComponent`) on a Visualforce or Site page can keep showing old behaviour **even after a hard refresh**. Do not assume the deploy failed. Check the server first:

```bash
sf data query --query "SELECT Source FROM LightningComponentResource WHERE LightningComponentBundle.DeveloperName = '<name>' AND FilePath LIKE '%.html'" --use-tooling-api --target-org <alias> --json
```

If the deployed source is correct but the browser renders the old version, this is not a normal browser cache. `Ctrl+Shift+R`, devtools "disable cache" and clearing `localStorage` or `sessionStorage` do not help. The `auraCmpDef` endpoint serves LWC bundles with `Cache-Control: private, max-age=31536000, immutable`, keyed by a server-side "last recompile marker" (the `_lrmc` query parameter on the bootstrap `*.app?...` request). Redeploying the LWC bundle alone does not move that marker, and the browser's persistent Aura definition storage (IndexedDB, survives reloads) keeps serving the old bundle, sometimes without any network request.

**Fix:** make a real content change (a no-op redeploy of identical bytes does not move the marker) to the Aura app or component that declares the `<aura:dependency>` for the changed LWC, then redeploy just that bundle:

```bash
# e.g. add or remove a comment line in the .app file; it must be an actual byte difference
cci task run deploy --path force-app/main/default/aura/<appName> --org <alias>
```

**Verify the fix landed:** in the browser's network requests, look for a fresh `auraCmpDef` GET during page load. If it is missing, the definition came from IndexedDB and is still stale.

**Variant, unverified: a new field missing from an Apex response.** An `@AuraEnabled` method's response can look stale in the same way: you add a property to an Apex DTO, redeploy, and the LWC receives objects without the new key (the key is absent, not `null`). Direct anonymous Apex against the same method returns the field, so the class is correct. Changing `Cacheable=true` to `false` and a brand-new isolated browser context did not help. The suspected cause is the same recompile-marker staleness scoped to the Apex action schema. The suspected fix, **not yet tested**, is the same as above: change and redeploy the Aura app that declares the dependency alongside the Apex change. Try that before re-verifying logic that anonymous Apex already proved.

## CSP Trusted Sites for `fetch()` from an LWC

An LWC on an Experience Cloud site that calls `fetch()` or `XMLHttpRequest` to an external API is blocked by the site's Content Security Policy unless a `CspTrustedSite` exists for that domain **with `isApplicableToConnectSrc` set to `true`**. A trusted site added for something else (for example an `<img>` from GitHub) usually has only `isApplicableToImgSrc`, which does not cover `fetch()`; the request fails with a `connect-src` violation.

Visualforce pages are not subject to this enforced CSP model, so "it worked on the VF page" says nothing about `connect-src`.

**Deploying a `CspTrustedSite` does not change the published CSP header; republish the site:**

```bash
cci task run create_community_publish --org <alias>   # or the project's equivalent PublishCommunity task
```

Publishing is asynchronous (tens of seconds). Retest in a fresh or isolated browser context so client-side caching does not hide a server-side delay.

## `lightning-input-address`

It looks like five free-text fields but supports more:

- `country-options` / `province-options`: arrays of `{label, value}` that turn those two fields into real dropdowns. The free-text fallback applies only when the attribute is genuinely `undefined`. An empty array `[]` counts as "options provided" and leaves an empty, option-less dropdown, so a getter feeding the attribute must return `undefined`, not `[]`, when there are no options.
- `show-compact-address`: draws the street as an 80-character one-line input plus a separate `subpremise` line (label them with `street-label` and `subpremise-label`). The change event carries `street` and `subpremise`. If the target field is a TextArea (such as `Mailing_Street__c`), join the two with `\n`.

Before assuming a Lightning base component cannot do something, check the component reference (the `experience-lwc-base-components-integrate` skill) for the property. Rebuilding it from separate fields is usually unnecessary.

## `lightning-record-edit-form` fires `load` again when its field list changes

If a component copies the record into its own state inside `onload`, **do that copy once per record**. The form fires `load` again whenever its field list changes, and each repeat reports the saved values, not the user's edits. The classic trigger is a "type" picklist that reveals extra `lightning-input-field`s: choosing a type adds fields, `load` fires 2 to 3 more times, and the handler overwrites state with the old type. The picklist shows the new choice but the sections snap back, with no error.

Fix: remember which record was initialised (`_initialisedRecordId`) and return early on repeat loads. Keep the saved copies up to date in the success handler so Reset works, and include the type field in Reset.

Jest fires one synthetic `load`, so tests miss this. Test the change path in a real browser (open a record of type A, switch to B, watch the sections) and add a Jest test that fires `load` again after the change. To see what the form emits, add listeners from `page.evaluate` (piercing shadow roots) and log `load` and `change` with timestamps.

## `getRelatedListRecords` needs the `__r` suffix

`lightning/uiRelatedListApi` `getRelatedListRecords({ parentRecordId, relatedListId, ... })` needs the full relationship name **including `__r`** for a custom child relationship: `relatedListId: 'My_Children__r'`, not `'My_Children'`. It matches SOQL subquery syntax. Confirm the name first with the equivalent subquery through `sf data query` (`Didn't understand relationship 'X' in FROM part of query call` means `__r` is missing). A bare name raises no clear error in the wire adapter and silently returns no data, which looks like a permissions or field-visibility problem.

## Lightning pages, styling and shared state

- **`slds-card` on a plain `div` can lose its border and shadow inside a Lightning page tab.** It looked fine in a `lightning-card`, then vanished inside `flexipage:tabset`. To find it, remove the class in the browser and re-read `getComputedStyle`; if your rule suddenly applies, that class is the cause. Give the element your own class and put the border, radius and shadow in your CSS.
- **Synthetic shadow DOM:** `shadowRoot.styleSheets` throws. Read `document.styleSheets` (component CSS is placed in `<head>` with an `lwc-xxxx` attribute selector). Use a recursive `deep(root, selector)` helper to pierce shadow roots.
- **A tab set on an App page is metadata too.** `flexipage:tabset` points at `Facet` regions that hold `flexipage:tab` items, which point at more `Facet` regions holding components. People edit pages in the UI, so the repo copy goes stale; retrieve the page before touching it.
- **Two components that need the same record on selection** (for example a read-only card and an edit form): share one in-flight request in a utility module (a `Map` of promises keyed by id, cleared in `finally`) so they cannot both insert it. Also ignore a slow response if a newer selection has replaced it.

## Testing an LWC

### What Jest stubs for `lightning-*` do NOT prove
- Stubs hold the properties you set (`pattern`, `maxlength`, `min`, `step`) and do nothing with them. They do not validate, and `checkValidity()` returns nothing, so tests must define it (see the helpers in `rfiForm.submit.test.js` in UST-Salesforce-RFI). A test that "the pattern is set" does not show the pattern works.
- Test a `pattern` string with a real `RegExp` using the **`v` flag**, anchored as `^(?:pattern)$`. Browsers compile the `pattern` attribute with `v`, which is stricter about `( ) [ ] { } - \` inside a class.
- Properties on a stub are not reflected as attributes, so `querySelector('lightning-combobox[name="x"]')` finds nothing. Select by tag and read the property, or find by `label`.
- Stubs show no real layout, so sticky positioning, overlays and gray-outs need a browser.

### Verify live state through the DOM (chrome-devtools-mcp)
When live-testing data binding, `take_snapshot`'s accessible name for a combobox or input can look blank or stale for reasons unrelated to the bound value. Read the component property directly, piercing shadow DOM:

```js
function deepQueryAll(root, selector, results = []) {
    root.querySelectorAll(selector).forEach(el => results.push(el));
    root.querySelectorAll('*').forEach(el => {
        if (el.shadowRoot) deepQueryAll(el.shadowRoot, selector, results);
    });
    return results;
}
// e.g. deepQueryAll(document, 'lightning-input-address')[0].country
```

If the property value matches nothing in the options list behind a combobox, that mismatch is very likely the real bug (a country code where the picklist expects a name, or the reverse), not a timing artifact.

### Playwright against Lightning after a deploy
Clear IndexedDB, localStorage, sessionStorage and Cache storage, load the page, wait about 25 seconds, and reload once (Lightning re-bootstraps after storage is cleared). Odd "state resets by itself" results right after clearing storage are usually that re-bootstrap, not your code. Measure before guessing: bounding boxes and computed styles settled two alignment questions in one call each.

## Form boxes that follow the Salesforce field

- Read limits from `Schema.DescribeFieldResult`: `getType()` (number, date, picklist), `getLength()` (text), `getPrecision()` and `getScale()` (numbers). One source then covers every question mapped to the field, including custom questions. Keep a short explicit list only for what the type cannot say (a Text field that holds a four-digit year).
- Send values in the type the server expects: a number box returns text, so convert it to a number; a Date field needs `YYYY-MM-DD`; a Text field that holds digits stays text.
- A restricted picklist behind an admin tool (for example the RFI Question Manager, which reads the `Fields_to_Display__c` values) cannot be reshaped from a scratch org without a metadata change that admins own. Filter and add in the component and send the list from Apex instead of editing the picklist.
