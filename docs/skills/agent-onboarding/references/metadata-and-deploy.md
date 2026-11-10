# Metadata and Deploy Gotchas

Read this when: a deploy reports success but nothing changed, a deploy hits `SourceConflictError`, or before writing field, permission set, picklist or multi-select metadata.

## When "Succeeded" is enough and when to verify

- **Trust it** for a normal source-format deploy of code from `force-app` (Apex, LWC, Aura). Do not add queries after `[Success]: Succeeded`.
- **Verify directly** for anything else: a folder with its own `package.xml` (`unpackaged/config/*`), a task that wraps `cumulusci.tasks.salesforce.Deploy` with a manifest, settings, Experience Cloud metadata, or any change that "doesn't seem to take effect". A deploy of zero real components still reports `"status": "Succeeded"`.

```bash
sf project deploy start --metadata-dir <folder> --target-org <alias> --json | jq '.result.numberComponentsDeployed, .result.numberComponentsTotal, .result.files'
```

Then confirm the record exists with a direct SOQL or Tooling query.

### Causes of a "successful" deploy of zero components
- **Wildcard members do not reliably deploy.** `<members>*</members>` is fine for retrieve, but for some types (confirmed with `CspTrustedSite`) the Metadata API resolves it to zero components on deploy, with no error. Use explicit `<members>Name</members>` entries.
- **Raw MDAPI folder structure.** A folder with its own `package.xml` is raw MDAPI format: files must **not** use the SFDX `-meta.xml` suffix, and they must sit in a subfolder named for the type's folder (`cspTrustedSites/Name.cspTrustedSite`), one level below `package.xml`, not beside it. A file with the right extension and a sensible name still silently fails to deploy if the nesting is wrong.

### A real deploy that a later describe does not show
A deploy can list the component and still show a stale describe or Tooling result (for example a custom object reporting fewer fields than were deployed). This looks like API-side caching or propagation lag. Check with a direct Tooling query (`SELECT Name FROM ApexClass` or `CustomField`). If that also disagrees with what you see in Setup, trust Setup; the describe cache can lag in both directions.

### `SourceConflictError` is not always a real conflict
Retrieve the org copy into a folder inside the project (`--output-dir` outside the project is rejected), diff it against what you last deployed, and only then use `--ignore-conflicts`. If the org copy differs in ways you did not make (a UI-edited flexipage, for example), stop and retrieve it instead.

## Required Lookup: `deleteConstraint` must be `Restrict` or `Cascade`

A `<required>true</required>` Lookup must specify `<deleteConstraint>Restrict</deleteConstraint>` or `Cascade`. `SetNull` is rejected at deploy time because nulling the field on parent delete would break the required constraint. Templates often default to `SetNull`.

## PermissionSet metadata rules

- Salesforce **rejects `fieldPermissions` for required fields** (`You cannot deploy to a required field.`). Before writing permission set XML for a new object, scan its `<required>true</required>` fields and leave them out. They are always accessible to anyone who can create or edit the record.
- **Group all `fieldPermissions` together and all `objectPermissions` together.** Interspersing them (`fieldPermissions, objectPermissions, fieldPermissions`) fails schema validation with `Element X is duplicated at this location`.
- `<description>` is limited to 255 characters (`data value too large`). Put longer explanations in the project docs.
- A new field is invisible without field-level security, so ship a permission set with it. The admin still has to add it to the page layout.

## Picklist metadata

- A picklist's blank "--None--" choice stores an empty value. If code quietly turns blank into a default, users who pick "--None--" get the default and think the control is broken. Decide what blank means, make every layer (Apex, component, editor preview) agree, and say it in the field's help text.
- New records get the picklist's `default="true"` value, and **an `@IsTest` record built without setting the field gets that default too**. To test "blank", set the field to `null` explicitly and re-query.
- **Multi-select picklists allow only one default value** (`Multiple default values` fails the deploy), and a default never backfills existing records. Design "blank means all" and say so in the help text.
- Read admin-set configuration in Apex with `WITH SYSTEM_MODE` (applicant data stays in user mode) so reviewers do not need field access.
- Apex that describes a new field will not compile until the field has been deployed. Deploy fields first.
