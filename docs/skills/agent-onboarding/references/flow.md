# Flow Gotchas

Read this before building or debugging record-triggered Flows, Flow formulas, Decision elements or fault paths.

## Order of independent record-triggered Flows is not guaranteed

Two record-triggered Flows on the same object and event (for example both after-save on Create) run in no guaranteed order unless the order is set in **Flow Trigger Explorer** in Setup. That setting is not an XML element in current API versions: hand-writing `<triggerOrder>` in `<start>` fails deploy with `Element ... triggerOrder invalid at this location in type FlowStart`.

If Flow B reads a field that Flow A sets, do not assume A ran first because it was built or deployed first. Either set the order in Flow Trigger Explorer before relying on it, or make B compute the value itself and accept some duplicated logic.

## Formula gotchas: picklists and Booleans

- A picklist field used directly in a formula (`{!$Record.SomeStatus__c} = "Active"`) fails deploy with `Field $Record is a picklist field. Picklist fields are only supported in certain functions.` Wrap it: `TEXT({!$Record.SomeStatus__c}) = "Active"`.
- `TEXT()` does not accept a Boolean (`Incorrect parameter type for function 'TEXT()'. Expected Number, Date, Date/Time, Picklist, received Boolean.`). Use `IF(booleanValue, "Yes", "No")` to render a Boolean as text, for example in an email body formula.

## A Decision with no `<defaultConnector>` silently ends the interview

A `<decisions>` element deploys and validates with a `<defaultConnectorLabel>` but **no `<defaultConnector>`**. When no rule matches (the default path), the interview just **ends** with no error, no fault and no sign anything is wrong. Everything after that decision never runs for those records. It can go unnoticed for a long time because development testing tends to exercise the "interesting" branch, while the default path (below a threshold, no match found) is the common case.

When some records are processed further and others are not, check every `<decisions>` element for a real `<defaultConnector>`, not only a `<defaultConnectorLabel>`.

## `faultConnector` on `emailSimple` is necessary but not sufficient

A `faultConnector` on an `emailSimple` action in a record-triggered Flow can miss some email failures. The failure then surfaces as an unhandled `CANNOT_EXECUTE_FLOW_TRIGGER` ("an unhandled fault has occurred in this flow") on the triggering DML, as if no fault connector existed. Seen with `Probably Limit Exceeded or 0 recipients`, commonly from exhausting a scratch org's low daily email-recipient limit during heavy testing, and reported by other practitioners. It is platform behaviour, not a wiring mistake.

- Never rely on the Flow's fault connector as the only safety net when the triggering DML matters to an end user. Wrap **the calling Apex's insert or update** in its own `try/catch`. Keep both: the fault connector improves the chance a human is told, and the `try/catch` guarantees a user action is never blocked by a downstream Flow failure.
- **Test fault paths both ways.** A synchronous insert in anonymous Apex can show an uncaught exception that would never reach a real user, because the real DML is dispatched from an `@future` or Queueable transaction with its own `try/catch`. Test directly, to see the Flow in isolation, and through the real call path, to see what a user would experience.
