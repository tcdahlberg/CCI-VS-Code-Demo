# Apex Gotchas

Read this before: writing Apex that makes a callout, adding a trigger, nesting classes, or when a trigger sees fewer fields than the record has.

## Callout after DML in the same transaction

Apex forbids an HTTP callout (`Http.send()`, an external service call) in a transaction that has **uncommitted DML**. It fails with `System.CalloutException: You have uncommitted work pending. Please commit or rollback before calling out`. It is easy to hit without noticing: a controller method that does `insert`/`update`/`delete` and later calls out, or a trigger firing from DML that is still uncommitted in the caller.

**Why it hides:** a callout wrapped in a graceful-degradation try/catch (feature flags, optional enrichment, a third-party API that may be down) swallows this exception and reports "unavailable" or "skipped". The feature looks intermittently or permanently broken with no visible error, and it looks the same as a missing Remote Site Setting or CSP Trusted Site, a bad API key, or a down endpoint. **During development, write the exception message to somewhere visible inside the catch block** (a diagnostic field, a debug log line). "Unavailable" never says why.

**Fix:** move the callout work into `@future(callout=true)` or a Queueable, which runs in a new transaction after the current one commits. Two traps:
- `@future` parameters must be primitives or collections of primitives: no sObjects, no custom Apex types. Pass `Id`, `String`, `Decimal` and so on.
- Request context (`ApexPages.currentPage()`, `RestContext.request`) does not exist inside the future method. Capture IP, User-Agent, Referer and the like **synchronously before dispatching** (in the controller method, or in the trigger if a trigger dispatches) and pass them in as primitives.

## Inner classes cannot have static factory methods

`public static MyInner build(...)` on a nested class fails to compile with `static can only be used on methods of a top level type`. Use a public constructor (`new MyInner(...)`) and update the call sites. There is no static-factory workaround for inner classes.

## A sparse update leaves the other fields null in the trigger

`update new Application__c(Id = someId, Status__c = 'X')` is a common way to change one field. In any trigger on that object, `Trigger.new` and `Trigger.oldMap` then hold **only `Id` and `Status__c`**; every other field, including lookups the trigger needs, is `null` even when the database has values. A trigger that needs other fields **must re-query** the Ids itself. `Trigger.new` reflects only what the DML statement included.

## Names

Constants and static final collections use camelCase like every other member (`splitAddressQuestions`, `countryIndexCache`), not `UPPER_CASE`. Check neighbouring declarations before adding one.
