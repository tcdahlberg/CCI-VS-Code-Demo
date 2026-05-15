# Education Cloud Data Model

**Source**: [Salesforce Education Cloud Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.edu_cloud_dev_guide.meta/edu_cloud_dev_guide/edu_cloud_data_model_overview.htm)  
**API Version**: Summer '26 (API version 67.0)  
**Last Updated**: May 15, 2026

---

## Overview

The Summit Events App scratch org runs on **Salesforce Education Cloud** (`EducationCloud:10` feature flag), a Purpose-Built CRM layer on top of core Salesforce that adds education-specific objects, fields, and capabilities. Understanding this data model is essential when extending SEA to work alongside (or integrate with) native Education Cloud features.

### Scratch Org Features Enabled

From `orgs/educloud.json`, the following Education Cloud capabilities are active:

| Feature / Setting | Value |
|---|---|
| `EducationCloud:10` (feature) | Enabled |
| `enableAcademicOperations` | `true` |
| `enableAlumniRelations` | `true` |
| `enableBenefitManagementPreference` | `true` |
| `enableCarePlansPreference` | `true` |
| `enableDiscoveryFrameworkMetadata` | `true` |
| `enableEducationCloud` | `true` |
| `enableGroupMembershipPref` | `true` |
| `enableIndustriesAssessment` | `true` |
| `enableStudentSuccess` | `true` |
| `Assessments` (feature) | Enabled |
| `Fundraising` (feature) | Enabled |
| `LightningScheduler` (feature) | Enabled |
| `OmniStudioDesigner` / `OmniStudioRuntime` | Enabled |
| `PersonAccounts` | Enabled |

---

## Entity Relationship Diagrams

For visual ERDs, see the official Salesforce Architects site:

- [Education Cloud Academic Operations Data Model](https://developer.salesforce.com/docs/platform/data-models/guide/academic-operations.html)
- [Education Cloud Recruitment and Admissions Data Model](https://developer.salesforce.com/docs/platform/data-models/guide/recruitment-and-admissions.html)
- [Education Cloud Student Success Data Model](https://developer.salesforce.com/docs/platform/data-models/guide/student-success.html)
- [Education Cloud Appointment Scheduling Data Model](https://developer.salesforce.com/docs/platform/data-models/guide/appointment-scheduling.html)
- [Education Cloud Fundraising Data Model](https://developer.salesforce.com/docs/platform/data-models/guide/fundraising.html)
- [Education Student Financials Data Model](https://developer.salesforce.com/docs/platform/data-models/guide/student-financials.html)

---

## Domain Breakdown

Objects are grouped into functional domains. "New Objects" are Education Cloud-specific; "Enhanced Standard Objects" are existing Salesforce objects that Education Cloud adds fields to.

---

### 1. Academic Calendar

Manages the time-based structure of an institution's academic calendar.

| Object | API Since | Description |
|---|---|---|
| **AcademicYear** | 57.0 | Defines an academic year period. Top-level calendar container. |
| **AcademicTerm** | 57.0 | Defines an academic period (semester, quarter, trimester) within a year. Can hold more granular sub-periods. |
| **AcademicSession** | 57.0 | Records a specific course-offering period within a term (e.g., a 6-week mini-session within a fall semester). |
| **AcademicTermRegstrnTimeline** | 64.0 | Represents the registration time window for an academic term (open/close dates per student tier). |
| **AcademicTermPolicyRule** | 63.0 | Junction between `AcademicTerm` and `ExpressionSet`; applies an expression-set policy rule to a term. |
| **AcadTermEnrlPolicyRuleLog** | 64.0 | Log of policy rule calculation runs for an academic term enrollment. |

**Key Relationship**: `AcademicYear` → `AcademicTerm` → `AcademicSession`

---

### 2. Recruitment and Admissions

Supports the full student recruitment pipeline from initial inquiry through application decision. **This domain is the most relevant to Summit Events App**, which handles event-based recruitment touchpoints.

| Object | API Since | Description |
|---|---|---|
| **EducationalInfoRequest** | 57.0 | Request for Information (RFI) raised by prospective students, parents, or counselors. |
| **AcademicInterest** | 62.0 | Represents a person's academic interest (program area, subject, etc.). |
| **ApplicationTimeline** | 57.0 | Milestone dates in the application process for a program. |
| **ProgramTermApplnTimeline** | 57.0 | Junction: `AcademicTerm` + `ApplicationTimeline` + `LearningProgram`. |
| **ApplicationStageDefinition** | 59.0 | Defines a stage in the application workflow (e.g., "Documents Submitted", "Under Review"). |
| **ApplicationSectionDefinition** | 59.0 | Defines a section within an application form. |
| **ApplnStageSectionDefinition** | 59.0 | Junction: links a stage to its applicable sections. |
| **ApplicationRenderMethod** | 60.0 | Defines how a part of an application is rendered (e.g., OmniScript, standard form). |
| **ApplnRenderMethodAssignment** | 60.0 | Assigns a render method to a specific application component. |
| **ApplicationDecision** | 57.0 | Records the academic standing decision on an applicant (admit, deny, waitlist, etc.). |
| **ApplicationReview** | 57.0 | A review performed against a specific application by a reviewer. |
| **ApplicationRecommendation** | 57.0 | Recommendation details for an individual application. |
| **ApplicationRecommender** | 57.0 | Junction: links an individual application to its recommender. |
| **IndividualApplicationTask** | 59.0 | A task (checklist item) required to complete an application. |
| **IndividualApplicationTaskItem** | 62.0 | Junction: links an application item to an `IndividualApplicationTask`. |
| **AcademicReviewRequest** | 67.0 | Details of a review request for an academic curriculum. |
| **AcademicReviewRequestItem** | 67.0 | Junction: links an `AcademicReviewRequest` to its related items. |

**Enhanced Standard Objects** (existing Salesforce objects with Education Cloud fields added):

| Object | Description |
|---|---|
| **IndividualApplication** | Core application record with Education Cloud-specific fields (preferences, program choice, status). |
| **PreliminaryApplicationRef** | Holds saved/in-progress application data before submission. |
| **PersonEducation** | Academic standing and prior education history of an applicant. |
| **PersonExamination** | Standardized test scores and exam history. |
| **DocumentChecklistItem** | Checklist items for required document uploads. |

---

### 3. Academic Enrollment

Tracks student enrollment in terms and courses once admitted.

| Object | API Since | Description |
|---|---|---|
| **AcademicTermEnrollment** | 57.0 | A student's enrollment record for a specific `AcademicTerm`. |

**Enhanced Standard Objects**:

| Object | Description |
|---|---|
| **CourseOffering** | An instance of a course offered at a specific time and location (fields added for academic metadata). |
| **CourseOfferingRelationship** | Junction: links related course offerings (co-requisites, cross-listed courses). |
| **CourseOfferingParticipant** | A student's enrollment in a specific `CourseOffering`. |
| **CourseOfferingPtcpResult** | Outcome/grade for a student's participation in a course. |
| **CourseOfrPtcpActvtyGrd** | Activity-level grade within a course participating result. (API 65.0+) |
| **CourseOfrgRubricCriterion** | Activity rubric criterion for a `CourseOffering`. (API 65.0+) |
| **CourseOfferingSchedule** | Schedule (day/time) defined for a `CourseOffering`. |
| **CourseOfferingScheduleTmpl** | Reusable schedule template for creating course offerings. |

---

### 4. Learning and Curriculum

Defines the catalog of courses, programs, and competency frameworks.

| Object | API Since | Description |
|---|---|---|
| **Learning** | 57.0 | Base catalog record — defines a training (course, program, or on-site experience). |
| **LearningCourse** | 57.0 | A specific course required in a learning framework. |
| **LearningProgram** | 57.0 | One or more trainings grouped into a program required to obtain a credential. |
| **LearningProgramPlan** | 57.0 | A specific plan to execute a `LearningProgram` (may vary by cohort or term). |
| **LearningProgramPlanRqmt** | 57.0 | Requirements (learning outcomes) included in a program plan. |
| **LearningAchievement** | 57.0 | Outcome of a learning activity; groups related achievements together. |
| **LearningFoundationItem** | 57.0 | Prerequisite, co-requisite, or recommended prior learning for a learning outcome. |
| **LearningOutcomeItem** | 57.0 | Maps a learning to a related outcome. |
| **LearningPathwayTemplate** | 61.0 | A template a learner can apply to create their planned learning path. |
| **LearningPathwayTemplateItem** | 61.0 | A requirement step within a `LearningPathwayTemplate`. |
| **LearningPathwayTmplPgmPlan** | 61.0 | Junction: `LearningProgramPlan` + `LearningPathwayTemplate`. |
| **LearningEquivalency** | 65.0 | Defines the equivalency between external and internal learnings. |
| **LearningEquivalencyLearning** | 66.0 | Junction: `LearningEquivalency` + `Learning`. Enables filtering by learning. |
| **LearningEqvAchvMapping** | 65.0 | Maps a `LearningEquivalency` to internal and external achievement groups. |
| **LearningRule** | 65.0 | Junction: `Learning` + `ExpressionSet`; applies an extensible rule to a learning. |
| **AcademicCredential** | 59.0 | A credential (degree, certificate, badge) that can be earned. |
| **PersonAcademicCredential** | 59.0 | An `AcademicCredential` that a specific person has earned. |
| **ExternalLearning** | 65.0 | Training made available by an external provider (course, program, or on-site experience). |
| **PriorLearningEvaluation** | 66.0 | Evaluation of a learner's prior learning for potential transfer credit. |
| **CourseCreditTransferAppln** | 65.0 | Details of a course credit transfer application. |

---

### 5. Student Success and Advising

Supports early alert, advising, and holistic student support.

| Object | API Since | Description |
|---|---|---|
| **LearnerProfile** | 63.0 | Holistic profile of a learner for advising purposes. |
| **LearnerProgram** | 57.0 | Details of a `LearningProgramPlan` created for a specific learner. |
| **LearnerProgramRequirement** | 57.0 | A requirement a learner must complete in their assigned program plan. |
| **LearnerProgramRqmtProgress** | 57.0 | Progress tracking for a specific program requirement for a learner. |
| **LearnerPathway** | 61.0 | A learner's planned path to completion of their enrolled programs. |
| **LearnerPathwayItem** | 61.0 | A requirement with completion details within a `LearnerPathway`. |
| **WatchlistedLearner** | 62.0 | Flags a learner for monitoring and targeted support. |
| **PulseCheck** | 62.0 | A point-in-time wellbeing snapshot for a learner (wellness surveys, check-ins). |
| **PulseCheckTemplate** | 62.0 | Reusable template for creating `PulseCheck` records. |
| **SuccessTeam** | 57.0 | A success team record in Salesforce Scheduler for coordinating student support. |
| **MentoringProfile** | 61.0 | Profile for a participant in a mentoring program (mentor or mentee). |
| **CaseTeamMemberProgram** | 59.0 | Mapping between `CaseTeamMember` and `Program` for case-based advising. |
| **InvolvementGroup** | 64.0 | A club, organization, or involvement group at an institution. |
| **InvolvementGroupMember** | 64.0 | A member within an `InvolvementGroup`. |

---

### 6. Person / Identity and Demographics

Stores demographic, disability, and personal profile data about contacts.

| Object | API Since | Description |
|---|---|---|
| **ContactProfile** | 57.0 | Demographics (ethnicity, citizenship, birth place, race, etc.) for an individual. |
| **ConstituentRole** | 57.0 | Roles associated with an individual (student, alumni, donor, faculty, staff, etc.). |
| **PersonDisability** | 57.0 | Information about a person's disability and accommodation needs. |
| **PersonAffinity** | 64.0 | Affinity score or connection of a person to a program or institution. |
| **PersonTrait** | 64.0 | Traits (characteristics, attributes) associated with a person. |
| **PersonPublicProfile** | 59.0 | Information shown on a user's public directory profile. |
| **PersonPublicProfilePrefSet** | 59.0 | A user's preferences for what appears in their directory entry. |

**Enhanced Standard Objects**:

| Object | Description |
|---|---|
| **PersonEmployment** | Employment history and current employment information. |
| **PersonLifeEvent** | Life events (marriage, birth, graduation, etc.) for relationship management. |
| **PersonCompetency** | Skills and competencies held by a person. |
| **Award** | Professional awards associated with a person or organization. |

---

### 7. Educational Characteristics

A flexible framework for tagging learners with categorized attributes (major, minor, campus, athlete status, etc.) that can be required for enrollment in courses or programs.

| Object | API Since | Description |
|---|---|---|
| **EducCharacteristicType** | 66.0 | A category of characteristics (e.g., "Major", "Campus", "Student Status"). |
| **EducationalCharacteristic** | 66.0 | A specific characteristic within a type (e.g., "Computer Science" under "Major"). |
| **EducCharacteristicAssignment** | 66.0 | Assigns a characteristic to a student for an academic interval. |
| **EducCharRequirement** | 66.0 | Defines what characteristics are needed to meet an educational requirement. |
| **EducCharReqRelationship** | 66.0 | Logical relationships between characteristics in a requirement (AND/OR logic). |
| **EducCharReqAssignment** | 66.0 | Assigns an `EducCharRequirement` to a `Learning` or `CourseOffering`. |

---

### 8. Institution and Offering Discovery

Supports institutional profiles and searchable discovery of programs and offerings.

| Object | API Since | Description |
|---|---|---|
| **EducInstitutionOffering** | 64.0 | Junction: links an institution's account to objects like `Program`, `LearningProgram`, or `AcademicTerm`. |
| **EducInstSearchableProfile** | 64.0 | Aggregated profile for an educational institution, used for Criteria-Based Search and Filter. |

---

### 9. Benefits and Support Programs

Manages financial aid, scholarships, support benefits, and disbursements.

**Enhanced Standard Objects**:

| Object | API Since | Description |
|---|---|---|
| **Program** | 57.0 | Enrollment and disbursement of benefits in a support program. |
| **Benefit** | 57.0 | A benefit associated with a program (scholarship, food assistance, housing, etc.). |
| **BenefitType** | 60.0 | Type classification for benefits. |
| **BenefitAssignment** | 57.0 | Enrollment of an individual into a specific benefit. |
| **BenefitDisbursement** | 57.0 | Allocation record for a benefit disbursement (monetary or non-monetary). |
| **GoalDefinition** | 57.0 | A reusable care plan goal template (PGI library). |
| **GoalAssignment** | 57.0 | An instantiated goal assigned to a student in a care plan. |
| **ActionPlanTemplate** | 60.0 | Reusable action plan template. |
| **ActionPlanTemplateAssignment** | 60.0 | Assignment of an action plan template version to a target (Care Plan, Benefit, Goal). |
| **ProviderOffering** | 60.0 | People or organizations providing benefits to program participants. |

---

### 10. Assessment

Supports structured assessments, survey-style wellness checks, and evaluation workflows.

**Enhanced Standard Objects**:

| Object | API Since | Description |
|---|---|---|
| **Assessment** | 63.0 | Header record for an assessment (stores metadata about the assessment form). |
| **AssessmentQuestion** | 63.0 | Container for questions in an assessment. |
| **AssessmentQuestionResponse** | 63.0 | Stores submitted responses to assessment questions. |
| **AssessmentEnvelope** | 62.0 | Groups assessments related to a learner. |
| **AssessmentEnvelopeItem** | 62.0 | An item within an assessment envelope. |

---

### 11. Appointments and Scheduling

Supports academic advising appointments and scheduling workflows through Salesforce Scheduler.

**Enhanced Standard Objects**:

| Object | API Since | Description |
|---|---|---|
| **WorkTypeGroupRole** | 57.0 | Groups work types by role (e.g., "Advisor Appointments" vs. "Registration Help"). |
| **ServiceTerritoryMember** | 57.0 | Service resource assigned to a service territory (used for advisor scheduling). |
| **Waitlist** | 61.0 | Queue for walk-in customers without a scheduled appointment. |
| **RecurrenceSchedule** | 62.0 | Recurrence schedule definition. |
| **Location** | 57.0 | Enhanced location (building, room, facility, region) with Education Cloud fields. |

---

### 12. Compliance and Regulatory

Supports regulatory code management, compliance controls, and clause versioning.

| Object | API Since | Description |
|---|---|---|
| **RgltyCodeRegClauseVer** | 63.0 | Junction: `RegulatoryCode` + `RegulationClauseVersion`. |
| **RgltyCodeViolRegClVer** | 63.0 | Junction: `RegulatoryCodeViolation` + `RegulationClauseVersion`. |

**Enhanced Standard Objects**:

| Object | Description |
|---|---|
| **RegulatoryCode** | The regulation code enforced by a regulatory body. |
| **ComplianceControl** | A compliance control record. |
| **BusinessOperationsProcess** | A business operations process (63.0+). |
| **BusinessProfile** | Business details on a license or permit application. |

---

### 13. Academic Financial

Bridges education-specific financial structures with Salesforce Commerce/Orders.

| Object | API Since | Description |
|---|---|---|
| **AcademicOrder** | 66.0 | Junction between an academic interval (e.g., `AcademicTerm`) and an `Order` record. Supports tuition billing. |
| **PriceBookAcademicInterval** | 66.0 | Junction: `Pricebook` + academic interval (term). Links a price book to a specific term. |

---

### 14. Partnerships

Manages formal agreements with external organizations and associated placements.

| Object | API Since | Description |
|---|---|---|
| **PartnershipAgreement** | 67.0 | Formal agreement defining scope, duration, and outcomes of a corporate, research, or workforce partnership. |
| **PartnershipPersonPlacement** | 67.0 | An individual's verified placement delivered through a `PartnershipAgreement` (internship, research, employment, etc.). |

---

### 15. Competency Framework

Tracks competencies associated with learnings, people, and credentials.

| Object | API Since | Description |
|---|---|---|
| **CompetencyRelatedObject** | 64.0 | Junction: links a `Competency` (from Industries Competency) to another object (Learning, LearningProgram, etc.). |

**Enhanced Standard Objects**:

| Object | Description |
|---|---|
| **PersonCompetency** | Skills and competencies held by a person (with Ed Cloud fields). |

---

### 16. Other / Supporting Objects

| Object | API Since | Description |
|---|---|---|
| **GiftCommitmentSchedule** | 64.0 | Schedule for fulfilling a gift commitment (Fundraising domain). |
| **BusinessProfile** | 64.0 | Business details for license/permit applications. |

---

## Relationship to Summit Events App

Summit Events App (`summit__` namespace) is an **event registration platform** that operates alongside—but largely independent of—the native Education Cloud objects. Here is how the two models intersect:

| SEA Object | Education Cloud Connection |
|---|---|
| `Summit_Events__c` | Represents a recruiting event — logically connects to `EducInstitutionOffering`, `LearningProgram`, and `AcademicTerm` |
| `Summit_Events_Registration__c` | A registration is a pre-application touchpoint; connects to `IndividualApplication` and `EducationalInfoRequest` |
| `Summit_Events_Instance__c` | An event instance maps to a specific `AcademicTerm` or admissions cycle |
| `Contact` (registrant) | The registrant Contact is the same person that `ConstituentRole`, `ContactProfile`, `AcademicInterest`, and `LearnerProfile` hang off |

### Integration Opportunities

1. **Recruitment Funnel**: SEA registrations could create or reference `EducationalInfoRequest` records to track the prospective student's first touchpoint.
2. **Academic Interest**: `AcademicInterest` records could be populated from questions asked in the SEA registration flow.
3. **Application Trigger**: A completed SEA registration could initiate an `IndividualApplication` for the relevant `LearningProgram`.
4. **Contact Profile Enrichment**: Registration data (demographics, interests) could populate `ContactProfile`, `PersonTrait`, and `PersonAffinity`.
5. **Term Alignment**: SEA `Summit_Events_Instance__c` dates could be linked to `AcademicTermRegstrnTimeline` to share open/close date logic with the academic calendar.

---

## Quick Reference: Object API Names

All Education Cloud objects use unprefixed API names (no namespace). They are available without a `__c` suffix because they are Salesforce-managed standard objects.

```
AcademicCredential          LearnerPathway
AcademicInterest            LearnerPathwayItem
AcademicOrder               LearnerProfile
AcademicReviewRequest       LearnerProgram
AcademicReviewRequestItem   LearnerProgramRequirement
AcademicSession             LearnerProgramRqmtProgress
AcademicTerm                Learning
AcademicTermEnrollment      LearningAchievement
AcademicTermPolicyRule      LearningCourse
AcademicTermRegstrnTimeline LearningEquivalency
AcademicYear                LearningEquivalencyLearning
AcadTermEnrlPolicyRuleLog   LearningEqvAchvMapping
ApplicationDecision         LearningFoundationItem
ApplicationRecommendation   LearningOutcomeItem
ApplicationRecommender      LearningPathwayTemplate
ApplicationRenderMethod     LearningPathwayTemplateItem
ApplicationReview           LearningPathwayTmplPgmPlan
ApplicationSectionDefinition LearningProgram
ApplicationStageDefinition  LearningProgramPlan
ApplicationTimeline         LearningProgramPlanRqmt
ApplnRenderMethodAssignment LearningRule
ApplnStageSectionDefinition MentoringProfile
CaseTeamMemberProgram       PartnershipAgreement
CompetencyRelatedObject     PartnershipPersonPlacement
ConstituentRole             PersonAcademicCredential
ContactProfile              PersonAffinity
CourseCreditTransferAppln   PersonDisability
CourseOfferingParticipant   PersonPublicProfile
CourseOfrPtcpActvtyGrd      PersonPublicProfilePrefSet
CourseOfferingPtcpResult    PersonTrait
CourseOfrgRubricCriterion   PriceBookAcademicInterval
CourseOfferingSchedule      PriorLearningEvaluation
CourseOfferingScheduleTmpl  ProgramTermApplnTimeline
EducCharacteristicAssignment PulseCheck
EducCharacteristicType      PulseCheckTemplate
EducCharReqAssignment       ProviderOffering
EducCharReqRelationship     RgltyCodeRegClauseVer
EducCharRequirement         RgltyCodeViolRegClVer
EducationalCharacteristic   SuccessTeam
EducationalInfoRequest      WatchlistedLearner
EducInstitutionOffering     WorkTypeGroupRole
EducInstSearchableProfile
ExternalLearning
IndividualApplicationTask
IndividualApplicationTaskItem
InvolvementGroup
InvolvementGroupMember
```

**Enhanced Standard Objects** (existing objects with Education Cloud fields — API names unchanged):
```
ActionPlanTemplate          Location
ActionPlanTemplateAssignment PersonCompetency
Assessment                  PersonEducation
AssessmentEnvelope          PersonEmployment
AssessmentEnvelopeItem      PersonExamination
AssessmentQuestion          PersonLifeEvent
AssessmentQuestionResponse  PreliminaryApplicationRef
Award                       Program
Benefit                     RecurrenceSchedule
BenefitAssignment           RegulatoryCode
BenefitDisbursement         ServiceTerritoryMember
BenefitType                 Waitlist
BusinessOperationsProcess
BusinessProfile
ComplianceControl
CourseOffering
CourseOfferingRelationship
DocumentChecklistItem
GiftCommitmentSchedule
GoalAssignment
GoalDefinition
IndividualApplication
```

---

## Useful SOQL Queries for This Org

```soql
-- Check which academic terms exist
SELECT Id, Name, StartDate, EndDate FROM AcademicTerm LIMIT 20

-- Check any constituent roles on Contacts
SELECT Id, Contact.Name, Role FROM ConstituentRole LIMIT 20

-- Check for any individual applications
SELECT Id, Name, Status, Contact.Name FROM IndividualApplication LIMIT 10

-- Verify Learning Program Plans
SELECT Id, Name, Status FROM LearningProgramPlan LIMIT 10

-- Check academic credentials  
SELECT Id, Name, CredentialType FROM AcademicCredential LIMIT 10
```

---

## References

- [Education Cloud Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.edu_cloud_dev_guide.meta/edu_cloud_dev_guide/edu_cloud_standard_objects.htm)
- [Education Cloud Fields on Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.edu_cloud_dev_guide.meta/edu_cloud_dev_guide/edu_cloud_fields_on_standard_objects.htm)
- [Education Cloud Associated Objects](https://developer.salesforce.com/docs/atlas.en-us.edu_cloud_dev_guide.meta/edu_cloud_dev_guide/edu_cloud_associated_objects.htm)
- [Education Cloud Standard Value Sets](https://developer.salesforce.com/docs/atlas.en-us.edu_cloud_dev_guide.meta/edu_cloud_dev_guide/edu_cloud_standard_valueset_names.htm)
- [Education Cloud Business APIs](https://developer.salesforce.com/docs/atlas.en-us.edu_cloud_dev_guide.meta/edu_cloud_dev_guide/edu_cloud_apis.htm)

