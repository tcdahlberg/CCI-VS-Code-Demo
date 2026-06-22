---
description: Salesforce Education Cloud data model, SOQL patterns, Apex service design, LWC data access, and integration patterns for the Education Cloud object graph. Use this skill when writing code that queries, displays, or manipulates Education Cloud objects — AcademicTerm, LearnerProfile, ContactProfile, IndividualApplication, LearnerProgram, CourseOffering, ConstituentRole, or any other Education Cloud standard object. Also use when integrating a managed package or custom app (such as Summit Events App) with the native Education Cloud data model.
name: salesforce-education-cloud
---
# Salesforce Education Cloud — Developer Skill

Apply these steps whenever you write Apex, SOQL, or LWC code that touches Education Cloud objects.

---

## Step 1 — Identify the Domain

Before writing any code, map the task to the correct Education Cloud domain. This determines which objects you'll query, which relationships matter, and which permissions must be present.

| Task description | Domain | Key starting object |
|---|---|---|
| Prospective student inquiries, RFIs, or event attendees | **Recruitment & Admissions** | `EducationalInfoRequest`, `IndividualApplication` |
| Application stages, decisions, reviews, recommendations | **Recruitment & Admissions** | `ApplicationDecision`, `ApplicationReview` |
| Enrollment tracking for a term | **Academic Enrollment** | `AcademicTermEnrollment`, `AcademicTerm` |
| Course scheduling, rosters, grades | **Academic Operations** | `CourseOffering`, `CourseOfferingParticipant`, `CourseOfferingPtcpResult` |
| Degree programs, curriculum, credentials | **Learning & Curriculum** | `LearningProgram`, `LearningProgramPlan`, `AcademicCredential` |
| Advising, early alerts, learner support | **Student Success** | `LearnerProfile`, `WatchlistedLearner`, `PulseCheck` |
| Student goals, care plans, benefit distributions | **Benefits & Support** | `Benefit`, `GoalAssignment`, `BenefitAssignment` |
| Person demographics, disability, traits | **Person / Identity** | `ContactProfile`, `PersonDisability`, `ConstituentRole` |
| Majors, minors, student-athlete status, enrollment eligibility | **Educational Characteristics** | `EducationalCharacteristic`, `EducCharacteristicAssignment` |
| Institutional program/offering discovery | **Institution** | `EducInstitutionOffering`, `EducInstSearchableProfile` |
| Tuition, orders, or price books tied to a term | **Academic Financial** | `AcademicOrder`, `PriceBookAcademicInterval` |
| Appointments, advising slots, waitlists | **Scheduling** | `WorkTypeGroupRole`, `ServiceTerritoryMember`, `Waitlist` |
| Surveys, assessments, rubrics | **Assessment** | `Assessment`, `AssessmentQuestion`, `PulseCheck` |

---

## Step 2 — Follow the Key Relationship Chains

Education Cloud objects form deep parent-child trees. Always traverse the full chain when writing queries to avoid `NullPointerException` or missing relationship fields.

### Academic Calendar Chain
```
AcademicYear
  └── AcademicTerm (AcademicYearId)
        └── AcademicSession (AcademicTermId)
        └── AcademicTermEnrollment (AcademicTermId, ContactId)
        └── AcademicTermRegstrnTimeline (AcademicTermId)
```

### Recruitment & Admissions Chain
```
Contact
  └── EducationalInfoRequest (ContactId)      -- RFI / inquiry
  └── IndividualApplication (ContactId)       -- application record
        └── ApplicationDecision (IndividualApplicationId)
        └── ApplicationReview (IndividualApplicationId)
        └── ApplicationRecommendation (IndividualApplicationId)
        └── IndividualApplicationTask (IndividualApplicationId)
```

### Curriculum → Enrollment Chain
```
LearningProgram
  └── LearningProgramPlan (LearningProgramId)
        └── LearningProgramPlanRqmt (LearningProgramPlanId)
        └── LearnerProgram (LearningProgramPlanId, ContactId)  -- per-student
              └── LearnerProgramRequirement (LearnerProgramId)
              └── LearnerProgramRqmtProgress (LearnerProgramRequirementId)
```

### Course Offering → Participation Chain
```
CourseOffering (from standard object with Ed Cloud fields)
  └── CourseOfferingParticipant (CourseOfferingId, ContactId)
        └── CourseOfferingPtcpResult (CourseOfferingParticipantId)
        └── CourseOfrPtcpActvtyGrd (CourseOfferingParticipantId)
  └── CourseOfferingSchedule (CourseOfferingId)
```

### Contact Identity Chain (all hang off Contact)
```
Contact
  └── ContactProfile (ContactId)              -- demographics, ethnicity, citizenship
  └── ConstituentRole (ContactId)             -- student, alumni, donor, faculty
  └── PersonDisability (ContactId)            -- accessibility needs
  └── PersonTrait / PersonAffinity (ContactId)
  └── LearnerProfile (ContactId)              -- advising profile
  └── WatchlistedLearner (ContactId)          -- early alert
```

### Learning → Achievement Chain
```
Learning
  └── LearningAchievement (LearningId)        -- outcome groups
  └── LearningFoundationItem (LearningId)     -- prerequisites
  └── LearningOutcomeItem (LearningId)        -- learning-to-outcome mapping
  └── LearningRule (LearningId)               -- expression-set rules
```

---

## Step 3 — SOQL Patterns (Safe, Bulk-Safe, WITH USER_MODE)

All Education Cloud objects follow standard Salesforce SOQL rules. Apply `WITH USER_MODE` for all queries in `@AuraEnabled` and `@RestResource` methods.

### Pattern: Enrollments for a term
```apex
List<AcademicTermEnrollment> enrollments = [
    SELECT Id, ContactId, Contact.Name, Contact.Email,
           AcademicTermId, AcademicTerm.Name, Status
    FROM AcademicTermEnrollment
    WHERE AcademicTermId = :termId
    WITH USER_MODE
    ORDER BY Contact.LastName
    LIMIT 200
];
```

### Pattern: Active constituent roles for a contact
```apex
List<ConstituentRole> roles = [
    SELECT Id, Role, Status, StartDate, EndDate
    FROM ConstituentRole
    WHERE ContactId = :contactId
    AND Status = 'Active'
    WITH USER_MODE
];
```

### Pattern: Learner's program requirements and progress
```apex
List<LearnerProgramRequirement> requirements = [
    SELECT Id, Name, Status, LearnerProgramId,
           LearnerProgram.LearningProgramPlan.Name,
           (SELECT Id, CompletionStatus, CompletionDate
            FROM LearnerProgramRqmtProgresses)
    FROM LearnerProgramRequirement
    WHERE LearnerProgram.ContactId = :contactId
    WITH USER_MODE
];
```

### Pattern: Application decisions for a contact
```apex
List<ApplicationDecision> decisions = [
    SELECT Id, DecisionType, DecisionDate, Status,
           IndividualApplicationId,
           IndividualApplication.ContactId,
           IndividualApplication.ProgramId
    FROM ApplicationDecision
    WHERE IndividualApplication.ContactId = :contactId
    WITH USER_MODE
    ORDER BY DecisionDate DESC
];
```

### Pattern: Course offerings for a term with roster counts
```apex
List<CourseOffering> offerings = [
    SELECT Id, Name, StartDate, EndDate,
           AcademicTermId, AcademicTerm.Name,
           (SELECT Id FROM CourseOfferingParticipants WHERE Status = 'Active')
    FROM CourseOffering
    WHERE AcademicTermId = :termId
    WITH USER_MODE
    ORDER BY Name
];
```

### Pattern: Educational characteristics assigned to a student
```apex
List<EducCharacteristicAssignment> characteristics = [
    SELECT Id, EducationalCharacteristicId,
           EducationalCharacteristic.Name,
           EducationalCharacteristic.EducCharacteristicTypeId,
           EducationalCharacteristic.EducCharacteristicType.Name,
           AcademicIntervalId, Status
    FROM EducCharacteristicAssignment
    WHERE ContactId = :contactId
    AND Status = 'Active'
    WITH USER_MODE
];
```

---

## Step 4 — Apex Service Design for Education Cloud

### 4.1 Use Selector Classes per Domain

Do not scatter Education Cloud queries throughout controllers. Group queries into domain-specific selector classes:

```apex
// ✅ Good — grouped selector
public with sharing class AcademicTermSelector {

    /**
     * Returns active terms for a given academic year.
     *
     * @param academicYearId The Id of the AcademicYear to query
     * @return List of active AcademicTerm records
     */
    public static List<AcademicTerm> getActiveTermsByYear(Id academicYearId) {
        return [
            SELECT Id, Name, StartDate, EndDate, Status
            FROM AcademicTerm
            WHERE AcademicYearId = :academicYearId
            AND Status = 'Active'
            WITH USER_MODE
            ORDER BY StartDate
        ];
    }
}
```

### 4.2 Sharing Model by Domain

| Domain | Recommended Sharing | Reason |
|---|---|---|
| Recruitment data (applications, decisions) | `with sharing` | Recruiter/advisor access is row-based |
| Academic calendar objects (terms, sessions) | `with sharing` or `inherited sharing` | Usually readable by all institution staff |
| Student Success (WatchlistedLearner, PulseCheck) | `with sharing` | Advisor-specific; strict confidentiality |
| Benefits / care plans | `without sharing` (with explicit CRUD checks) | System-level assignment jobs |
| Scheduling (ServiceTerritoryMember) | `with sharing` | Field Service / Scheduler OWD governs |

### 4.3 Invocable and @AuraEnabled Patterns

```apex
public with sharing class LearnerProfileService {

    /**
     * Returns the active learner profile for a contact.
     * Used by LWC advisor dashboard.
     *
     * @param contactId The Contact Id of the learner
     * @return LearnerProfileWrapper with profile and role summary
     */
    @AuraEnabled(cacheable=true)
    public static LearnerProfileWrapper getProfile(Id contactId) {
        LearnerProfile profile = [
            SELECT Id, ContactId, Status, AcademicStanding,
                   GradePointAverage, ExpectedGraduationDate
            FROM LearnerProfile
            WHERE ContactId = :contactId
            WITH USER_MODE
            LIMIT 1
        ];
        List<ConstituentRole> roles = [
            SELECT Id, Role, Status
            FROM ConstituentRole
            WHERE ContactId = :contactId
            AND Status = 'Active'
            WITH USER_MODE
        ];
        return new LearnerProfileWrapper(profile, roles);
    }

    public class LearnerProfileWrapper {
        @AuraEnabled public LearnerProfile profile;
        @AuraEnabled public List<ConstituentRole> roles;
        public LearnerProfileWrapper(LearnerProfile profile, List<ConstituentRole> roles) {
            this.profile = profile;
            this.roles = roles;
        }
    }
}
```

### 4.4 Bulk Safety for Education Cloud DML

Education Cloud objects follow standard DML governor limits. For enrollment or assignment operations, always bulkify:

```apex
// ✅ Bulk-safe: create term enrollments for a list of contacts
public static void enrollContacts(Id termId, List<Id> contactIds) {
    List<AcademicTermEnrollment> newEnrollments = new List<AcademicTermEnrollment>();
    for (Id contactId : contactIds) {
        newEnrollments.add(new AcademicTermEnrollment(
            AcademicTermId = termId,
            ContactId = contactId,
            Status = 'Active'
        ));
    }
    insert newEnrollments;   // Single DML outside the loop
}
```

---

## Step 5 — LWC Data Access Patterns for Education Cloud

### 5.1 Wire Adapter Choice

| Scenario | Recommended pattern |
|---|---|
| Display learner profile on a record page | `@wire(getRecord, { recordId, fields })` using schema imports |
| Advisor dashboard — cross-object learner data | `@wire(ApexClass.getProfile, { contactId })` with cacheable Apex |
| Search / filtered list of enrollments | Imperative Apex call (result is user-filtered, not cacheable) |
| Related list (e.g. constituent roles on Contact) | `@wire(getRelatedListRecords, ...)` or LDS `getRecord` with relationship fields |

### 5.2 Schema Imports for Education Cloud Fields

Education Cloud adds fields to existing standard objects. Always import using schema tokens — never hardcode API names as strings in LWC:

```javascript
// ✅ Import with schema tokens (prevents breakage during package upgrades)
import CONTACT_PROFILE_ID from '@salesforce/schema/ContactProfile.Id';
import CONTACT_PROFILE_STATUS from '@salesforce/schema/ContactProfile.Status';
import ACADEMIC_STANDING from '@salesforce/schema/LearnerProfile.AcademicStanding';
import CONSTITUENT_ROLE from '@salesforce/schema/ConstituentRole.Role';
```

### 5.3 Example: Advisor Dashboard Component (Wire + Apex)

```javascript
// learnerDashboard.js
import { LightningElement, api, wire } from 'lwc';
import getProfile from '@salesforce/apex/LearnerProfileService.getProfile';

export default class LearnerDashboard extends LightningElement {
    @api recordId;   // Contact record page

    @wire(getProfile, { contactId: '$recordId' })
    learnerData;

    get hasProfile() {
        return this.learnerData?.data?.profile != null;
    }

    get academicStanding() {
        return this.learnerData?.data?.profile?.AcademicStanding ?? 'Unknown';
    }

    get activeRoles() {
        return this.learnerData?.data?.roles ?? [];
    }

    get isLoading() {
        return !this.learnerData?.data && !this.learnerData?.error;
    }
}
```

```html
<!-- learnerDashboard.html -->
<template>
    <lightning-card title="Learner Profile">
        <template lwc:if={isLoading}>
            <div class="my-spinner"></div>
        </template>
        <template lwc:elseif={hasProfile}>
            <p>Academic Standing: {academicStanding}</p>
            <template for:each={activeRoles} for:item="role">
                <lightning-badge key={role.Id} label={role.Role}></lightning-badge>
            </template>
        </template>
        <template lwc:else>
            <p>No learner profile found.</p>
        </template>
    </lightning-card>
</template>
```

---

## Step 6 — Integration: Custom App (e.g. Summit Events App) + Education Cloud

When a managed package or custom app coexists with Education Cloud in the same org, use these patterns to bridge the two models.

### 6.1 Contact as the Integration Point

The `Contact` record is the shared entity. Never create duplicate Contact records to bridge domains — always locate the existing Contact first:

```apex
// Find the Contact from a registration, then look up their Education Cloud data
Contact registrant = [SELECT Id, Email, Name FROM Contact WHERE Id = :registration.ContactId LIMIT 1];

// From that Contact, reach Education Cloud objects
List<ConstituentRole> roles = [
    SELECT Role, Status FROM ConstituentRole
    WHERE ContactId = :registrant.Id AND Status = 'Active'
    WITH USER_MODE
];
```

### 6.2 Creating an RFI from an Event Registration

When a prospective student registers for a recruiting event, optionally create an `EducationalInfoRequest` to track them in the admissions pipeline:

```apex
private static void createInquiryForRegistrant(Id contactId, Id programId) {
    EducationalInfoRequest rfi = new EducationalInfoRequest(
        ContactId = contactId,
        ProgramId = programId,
        Status = 'Open',
        Source = 'Event Registration'
    );
    insert rfi;
}
```

### 6.3 Populating Academic Interests from Event Questions

When a registrant answers program interest questions, upsert `AcademicInterest` records:

```apex
private static void syncAcademicInterests(Id contactId, List<String> selectedPrograms) {
    // Delete stale interests, then re-insert
    delete [SELECT Id FROM AcademicInterest WHERE ContactId = :contactId];
    List<AcademicInterest> newInterests = new List<AcademicInterest>();
    for (String program : selectedPrograms) {
        newInterests.add(new AcademicInterest(
            ContactId = contactId,
            ProgramName = program,
            Status = 'Active'
        ));
    }
    if (!newInterests.isEmpty()) {
        insert newInterests;
    }
}
```

### 6.4 Checking Constituent Role Before Branching Logic

If your code needs to behave differently for current students vs prospective students:

```apex
public static Boolean isCurrentStudent(Id contactId) {
    return ![
        SELECT Id FROM ConstituentRole
        WHERE ContactId = :contactId
        AND Role = 'Student'
        AND Status = 'Active'
        WITH USER_MODE
        LIMIT 1
    ].isEmpty();
}
```

### 6.5 Aligning Event Instances with Academic Terms

Link a custom event record to an `AcademicTerm` to inherit calendar context:

```apex
// Find the current or upcoming academic term
AcademicTerm currentTerm = [
    SELECT Id, Name, StartDate, EndDate
    FROM AcademicTerm
    WHERE StartDate <= TODAY AND EndDate >= TODAY
    AND Status = 'Active'
    WITH USER_MODE
    ORDER BY StartDate DESC
    LIMIT 1
];
```

---

## Step 7 — Permission Requirements

Education Cloud objects use standard Salesforce object/field-level security. The following objects require explicit permission-set grants for non-admin users.

### Core Read Access (commonly needed on Experience Cloud guest profiles)
```
- ContactProfile          (read)
- ConstituentRole         (read)
- EducationalInfoRequest  (create, read)
- AcademicInterest        (create, read, edit)
- IndividualApplication   (create, read, edit)
```

### Admissions Staff (advisor / recruiter permission set)
```
All of the above, plus:
- ApplicationDecision        (read)
- ApplicationReview          (create, read, edit)
- ApplicationRecommendation  (read)
- LearnerProfile             (read)
- WatchlistedLearner         (create, read, edit, delete)
- PulseCheck                 (create, read)
```

### Student Success Staff
```
All above, plus:
- LearnerProgram             (read)
- LearnerProgramRequirement  (read)
- LearnerProgramRqmtProgress (read, edit)
- GoalAssignment             (create, read, edit)
- BenefitAssignment          (read)
```

### Important: Fields You Cannot Include in Permission Sets
These Education Cloud fields are system-controlled and cannot be in a `PermissionSet`:
- Fields on **Master-Detail** relationship fields
- **Formula** fields (read-only by definition)
- Fields on objects where `<required>true</required>` is set in the metadata

---

## Step 8 — Gotchas and Anti-Patterns

### ❌ Anti-Pattern: Querying Education Cloud objects in loops

```apex
// ❌ — SOQL in loop; each contact fires a separate query
for (Contact c : contacts) {
    LearnerProfile profile = [SELECT Id FROM LearnerProfile WHERE ContactId = :c.Id LIMIT 1];
}

// ✅ — Bulk query before the loop
Set<Id> contactIds = new Map<Id, Contact>(contacts).keySet();
Map<Id, LearnerProfile> profilesByContact = new Map<Id, LearnerProfile>();
for (LearnerProfile p : [SELECT Id, ContactId FROM LearnerProfile WHERE ContactId IN :contactIds WITH USER_MODE]) {
    profilesByContact.put(p.ContactId, p);
}
```

### ❌ Anti-Pattern: Hardcoding Education Cloud API version requirements in early API

Some Education Cloud objects require minimum API versions. Using them in metadata targeting earlier versions causes `INVALID_TYPE` errors:

| Object | Minimum API version |
|---|---|
| `AcademicSession`, `AcademicTerm`, `AcademicTermEnrollment`, `AcademicYear` | 57.0 |
| `LearnerProfile` | 63.0 |
| `EducationalCharacteristic`, `EducCharacteristicAssignment`, `EducCharacteristicType` | 66.0 |
| `AcademicOrder`, `PriceBookAcademicInterval`, `PriorLearningEvaluation` | 66.0 |
| `PartnershipAgreement`, `PartnershipPersonPlacement`, `AcademicReviewRequest` | 67.0 |

Always check `sfdx-project.json` `sourceApiVersion` is high enough for the objects you're using.

### ❌ Anti-Pattern: Missing `WITH USER_MODE` on AuraEnabled methods

Education Cloud data is often sensitive (disability records, academic standing, financial aid). Always enforce sharing in `@AuraEnabled` methods. The exception is internal system-level jobs — document them if `without sharing` is used.

### ❌ Anti-Pattern: Duplicate constituent roles on Contact creation

Multiple code paths (event registration, application, portal signup) may all try to create a `ConstituentRole` record. Always check for an existing role before inserting:

```apex
// ✅ Upsert pattern to prevent duplicate ConstituentRole
List<ConstituentRole> existing = [
    SELECT Id FROM ConstituentRole
    WHERE ContactId = :contactId AND Role = 'Student'
    WITH USER_MODE
    LIMIT 1
];
if (existing.isEmpty()) {
    insert new ConstituentRole(ContactId = contactId, Role = 'Student', Status = 'Active');
}
```

### ❌ Anti-Pattern: Using `EducInstitutionOffering` without understanding it is purely a junction

`EducInstitutionOffering` links an Account (institution) to `Program`, `LearningProgram`, or `AcademicTerm`. It does **not** hold enrollment counts or offering details. Always traverse to the joined object for actual data.

### ❌ Anti-Pattern: Deploying test classes that reference Education Cloud objects to a non-Education-Cloud org

Test classes using `AcademicTerm`, `ConstituentRole`, etc. will fail to compile on orgs where `enableEducationCloud = false`. Always deploy Education Cloud test classes to a scratch org with `EducationCloud:10` in the feature list.

---

## Quick Reference — Common Education Cloud API Names

```
-- Admissions Pipeline --            -- Enrollment & Curriculum --
EducationalInfoRequest               AcademicYear
AcademicInterest                     AcademicTerm
IndividualApplication                AcademicSession
ApplicationDecision                  AcademicTermEnrollment
ApplicationReview                    CourseOffering
ApplicationTimeline                  CourseOfferingParticipant
                                     CourseOfferingPtcpResult
-- Person Identity --                LearningProgram
ContactProfile                       LearningProgramPlan
ConstituentRole                      LearnerProgram
PersonDisability                     LearnerProgramRequirement
PersonTrait                          LearnerProgramRqmtProgress
PersonAffinity
PersonPublicProfile                  -- Student Success --
                                     LearnerProfile
-- Credentials & Learning --         WatchlistedLearner
AcademicCredential                   PulseCheck
PersonAcademicCredential             PulseCheckTemplate
Learning                             SuccessTeam
LearningAchievement                  MentoringProfile
LearnerPathway
LearnerPathwayItem                   -- Characteristics --
                                     EducCharacteristicType
-- Benefits --                        EducationalCharacteristic
Benefit                              EducCharacteristicAssignment
BenefitAssignment                    EducCharRequirement
BenefitDisbursement
GoalAssignment                       -- Partnerships --
                                     PartnershipAgreement
-- Financial --                       PartnershipPersonPlacement
AcademicOrder
PriceBookAcademicInterval
```
