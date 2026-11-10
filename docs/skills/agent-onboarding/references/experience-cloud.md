# Experience Cloud Gotchas

Read this before giving portal or guest users access to records, or when a portal user can read a record but cannot save it.

## What a permission set can and cannot grant a portal user

- **A permission set grants object, field, Apex-class and record-type access. It never grants access to a specific record.** If a signed-in user can read a record but a save fails with `INSUFFICIENT_ACCESS_ON_CROSS_REFERENCE_ENTITY`, the cause is record sharing, not FLS, and `without sharing` on the controller does not help (DML still needs edit access). Confirm with `SELECT HasReadAccess, HasEditAccess FROM UserRecordAccess WHERE UserId = ... AND RecordId = ...`.
- **To let a portal user edit records tied to their own Contact, use a Sharing Set** (`SharingSet` metadata: `accessLevel Edit`, `object`, `objectField` such as `Contact__c`, `userField Contact`, and `profiles`). It attaches to **profiles, not permission sets**, and deploys as raw MDAPI (`sharingSets/<Name>.sharingSet`, no `-meta.xml`) after the site exists.
- **A guest sharing rule needs the site's guest user to exist** (`Specify a guest user's nickname for the guestUser field` otherwise). See the `cumulus-ci` skill for where to deploy it.

## Field and record-type visibility

- **`No such column X on entity Y` in a dynamic `Database.query` can mean the user lacks field-level access, not that the field is missing.** Check FLS first. Dynamic queries name fields in several string pieces (SELECT and every WHERE branch), so extract them all at once instead of fixing one per test run.
- **A dynamic `Database.query` with no access clause can fail with "No such column" for a portal user even when the code is meant to be public.** Add `WITH SYSTEM_MODE` to the query string (or pass `AccessLevel.SYSTEM_MODE`); a probe showed the same query then works. **Deploy the controller you are testing.** A test failing against stale code in the org looks like a real bug (the org still had the old `USER_MODE` class and the failure pointed at the wrong cause).
- **Never grant "all fields" on a directory-style object.** Formula fields often expose email, birthdate, phone or street address. Derive the field list from the queries the controllers run (`WITH USER_MODE` queries need grants; `without sharing` queries with no `USER_MODE` only need class access) and leave required fields out (Salesforce rejects them).
- **Record types are visible per user, even in `SYSTEM_MODE`.** `getRecordTypeInfosByDeveloperName()` returns only the record types the running user can see, so a public system-mode controller returns nothing to a guest unless a permission set makes the record type visible. A System Administrator may not see a record type granted only by a permission set: give the test's setup user that permission set, or inserts fail with `INVALID_CROSS_REFERENCE_KEY`.

## Testing

`System.runAs` in a test enforces object, field and record access, but **not Apex class access**. Prove class grants by reading `SetupEntityAccess` for the permission set, and prove the rest by calling the controllers inside `runAs(portalUser)`.
