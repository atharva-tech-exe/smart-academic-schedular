# Architecture v3

## Principles

- Scheduler independent from FastAPI/SQLAlchemy
- Backend authoritative for auth, tenancy, RBAC and entitlements
- Published versions immutable
- Solver inputs reproducible
- Generation asynchronous
- Independent validation mandatory
- School MVP before college/exams
- Canonical scheduling semantics are frozen before implementation

## High-level

React + TypeScript → FastAPI → PostgreSQL.

FastAPI modules:
Authentication, Membership/RBAC, Organization/Institution, Academic,
Import, Timetable, Entitlements, Export, Audit.

FastAPI creates GenerationJobs handled by a scheduling worker.
Object storage holds imports/exports; PostgreSQL stores metadata.

No microservices are required for MVP.

## Domain

### SaaS

User, Organization, OrganizationMembership, Role, Permission,
Institution, Plan, Subscription, Entitlement, BillingEvent, AuditLog.

MVP actively implements only the Organization, Institution, Membership,
Role/Permission and trial/entitlement portions required for access
control. Full Subscription/BillingEvent/payment behavior is Phase 9.

### Academic

AcademicYear, AcademicTerm, ScheduleCycle, Calendar, BellSchedule,
ScheduleDay, TimeSlot, LearnerGroup, LearnerGroupPartition,
Class/Grade, Section, Subject, Teacher, TeacherAvailability,
TeacherSchedulingPolicy, Room, RoomAvailability, RoomFeature,
RoomFeatureAssignment.

### Scheduling

TeachingRequirement, TeachingRequirementTeacher,
TeachingRequirementLearnerGroup, TeachingRequirementRoomRequirement,
ConstraintConfiguration, GenerationJob, GenerationInputRevision,
TimetableVersion, TimetableEntry, TimetableLock, Publication.

### Files

ImportJob, ImportValidationResult, ExportJob, StoredObject.

## Organization / Institution / Membership

For MVP:
- Organization is the tenant boundary.
- Each Organization has exactly one active Institution.
- Schema may later support one-to-many institutions.
- `OrganizationMembership` has unique `(organization_id, user_id)`.
- Membership status is `invited`, `active`, or `suspended`.
- Only active membership grants tenant authorization.

Never trust a client-supplied organization ID. Resolve tenant context
from authenticated identity and active membership.

## RBAC

Authorization is permission-based.

Initial permissions:
- `timetable.view`
- `academic.manage`
- `import`
- `export`
- `schedule.generate`
- `schedule.edit`
- `schedule.lock`
- `schedule.publish`
- `users.manage`
- `billing.manage`

Initial MVP roles:
- Organization Admin
- Academic Scheduler
- Teacher/Staff Viewer

Organization Admin has all MVP permissions except no special billing
restriction; Academic Scheduler manages academic data and scheduling
but does not manage billing or users unless explicitly granted;
Teacher/Staff Viewer is read-only for published/authorized timetable
views and exports.

The exact permission matrix is frozen in `tasks.md` and must be enforced
server-side.

## Time

### ScheduleCycle

MVP ScheduleCycle represents one standard recurring week. It defines the
cycle over which `sessions_per_cycle` is fulfilled.

AcademicYear and AcademicTerm define academic context/effective dates;
ScheduleCycle defines the recurring scheduling pattern within that
context.

A/B rotation and arbitrary multi-week cycles are post-MVP.

### BellSchedule and ScheduleDay

Persist an explicit ScheduleDay definition under the BellSchedule.

Model:
`Calendar → BellSchedule → ScheduleDay → TimeSlot`

A ScheduleDay represents one schedulable day pattern. Different days may
have different TimeSlots.

A TimeSlot contains:
- schedule day
- ordinal
- start time
- end time
- type: teaching / break / fixed_activity / non_teaching

Breaks and fixed activities are not valid teaching slots.

### TimeSlot compatibility

The domain computes `CompatibleSlotGroup`s during preflight.

Two teaching TimeSlots may form one multi-slot occurrence only if:
- same ScheduleDay
- directly adjacent in schedule order
- no break/non-teaching/fixed activity between them
- both are teaching-capable
- their time boundaries are contiguous
- their duration pattern satisfies the TeachingRequirement's
  `slots_per_session` policy

CP-SAT consumes these valid placement groups rather than inferring
compatibility from ordinal numbers alone.

Different durations are allowed in the TimeSlot model, but a multi-slot
placement is valid only if the explicit compatibility rule accepts the
group. MVP does not silently convert minutes into slots.

## LearnerGroup

Every Section has exactly one primary LearnerGroup representing all
learners in the section.

Additional LearnerGroups may belong to the Section for scheduling
subgroups.

`LearnerGroupPartition` defines a disjoint subgroup partition.

Rules:
- sibling groups in one disjoint partition may be scheduled
  simultaneously
- a whole-section primary group conflicts with every subgroup in its
  partition
- arbitrary overlaps are rejected in preflight unless explicitly
  represented by a supported relationship
- full student-level membership is not required for MVP

The canonical SchedulingProblem includes learner-group conflict
relationships, not merely group IDs.

## TeachingRequirement

A TeachingRequirement is the atomic requirement from which occurrences
and valid placements are generated.

Required fields/semantics:
- subject
- learner group(s)
- teacher(s)
- `sessions_per_cycle`
- `slots_per_session`
- optional min/max occurrences per ScheduleDay
- allowed/forbidden TimeSlots
- room requirement/policy
- named soft preferences

One occurrence = one delivery of the requirement.

`slots_per_session` is a count of compatible TimeSlots, not minutes.

Example:
`2 occurrences × 2 slots` means two separate double-slot lessons.

For MVP, each occurrence:
- is placed on one ScheduleDay
- uses one compatible contiguous placement group
- cannot cross breaks
- cannot cross ScheduleDays

## Rooms

Room contains capacity, features/type and an explicit availability
schedule.

`RoomAvailability` is persisted separately and becomes part of the
canonical SchedulingProblem.

Room allocation policy belongs to
`TeachingRequirementRoomRequirement`:

- fixed: only the specified room(s)
- preferred: preferred rooms receive soft penalties when not used
- flexible: any room satisfying capacity/features/availability

A Section classroom can be a default/preference, but a requirement can
override it.

## Teacher Scheduling Policy

MVP vocabulary:
- unavailable TimeSlots
- max occurrences per ScheduleDay
- max consecutive teaching slots
- optional max occurrences per ScheduleCycle

No arbitrary policy blob is required for MVP.

## ConstraintConfiguration

Named objectives:
- teacher gaps
- learner-group gaps
- subject spread
- subject adjacency avoidance
- room preference
- teacher preference
- unnecessary consecutive teaching load

Each has:
- stable objective key
- default weight
- enabled/disabled state
- penalty unit definition
- objective configuration version

`GenerationInputRevision` snapshots the full ConstraintConfiguration.

## Canonical SchedulingProblem

The canonical problem contains:
- TimeSlots and compatible placement groups
- learner groups and explicit conflict relationships
- teachers and availability
- rooms and room availability
- TeachingRequirements
- hard constraint configuration
- soft objective configuration
- locks
- generation input revision metadata

The canonical problem is independent of web/database models.

## Generation Jobs

States:
`queued`, `preflight`, `solving`, `validating`, `completed`, `failed`,
`cancelled`.

Every job references exactly one GenerationInputRevision.

Generation requests carry an idempotency key scoped to the organization
and input revision. A duplicate request returns/reuses the existing
active/completed job rather than starting an accidental duplicate solve.

A successful job creates exactly one candidate draft TimetableVersion.
Failed/cancelled jobs create no publishable version.

Concurrency limits are enforced per organization and globally.

## Reproducibility

Record:
- input revision
- solver version
- constraint/objective configuration
- solver settings
- seed where applicable
- generation timestamps
- job status
- diagnostics/metrics

## Timetable Versioning

Published versions are immutable.

A draft has an optimistic `revision` value.

Every Move/Swap/Lock mutation must include the expected draft revision.
The server rejects stale mutations instead of silently overwriting another
administrator's change.

`TimetableLock` is draft/version scoped and associated with the affected
timetable entry/assignment.

Publishing creates a new immutable version.

## Move / Swap API Boundary

Academic data mutation and timetable lifecycle commands remain separate.

Academic mutations:
- TeacherAvailability
- Teacher
- Subject
- Section
- LearnerGroup
- Room
- TeachingRequirement

Scheduler commands:
- generate
- validate

Timetable lifecycle commands:
- move
- swap
- lock
- publish

Move/Swap requests submit intended operations only. The server:
1. authorizes
2. checks draft revision
3. checks locks
4. validates hard constraints
5. applies atomically
6. increments draft revision
7. persists the mutation

## Import

Pipeline:
Upload → Parse → Map → Validate → Preview → Confirm → Atomic Import.

Each import preview references a validation/import revision and the
relevant academic-data revision.

MVP identity/upsert rules:
- Teacher: stable external ID when supplied; otherwise exact normalized
  email is the preferred identity; ambiguous matches are rejected.
- Subject: stable external ID when supplied; otherwise exact normalized
  subject code; ambiguous matches are rejected.
- Class/Grade: stable external ID when supplied; otherwise exact
  normalized code/name within tenant; ambiguous matches are rejected.
- Section: stable external ID when supplied; otherwise exact normalized
  `(class_id, section_code)`; ambiguous matches are rejected.
- Room: stable external ID when supplied; otherwise exact normalized
  room code; ambiguous matches are rejected.
- LearnerGroup: stable external ID when supplied; otherwise exact
  normalized `(section_id, group_code)`; ambiguous matches are rejected.

Confirmation against a stale validation revision is rejected and must be
revalidated. Confirmed imports commit atomically.

## Trial / Entitlements

First eligible organization activation creates one non-renewable 30-day
trial.

EntitlementService evaluates capabilities and limits. Business endpoints
do not inspect plan names.

After expiry, customer data remains intact. Premium mutation/generation
capabilities are blocked according to entitlement policy; explicitly
permitted read/export operations remain available.

## Authentication

Support registration, verification, login/logout, password reset,
invitations, suspension, session revocation and ownership transfer.

Prefer secure HTTP-only browser sessions unless requirements dictate
otherwise.

## Future examination

Academic Data → Exam Definition → Exam Registration → Exam Scheduling →
Room Allocation → Seating → Invigilation. Separate optimization problem.

## Future college

Map departments, programs, semesters, divisions, batches, electives and
practical groups into the generic scheduling problem. Do not build
college-specific schema during MVP.


## v3.1 Clarifications — Academic Context, Learner Groups, and Academic Data Revisions

### Canonical Academic Context Ownership

The MVP academic and scheduling ownership chain is:

`Organization → Institution → AcademicYear → AcademicTerm → ScheduleCycle`

- An Organization owns its Institution.
- MVP permits exactly one active Institution per Organization.
- An Institution owns AcademicYears.
- An AcademicYear owns AcademicTerms.
- An AcademicTerm owns ScheduleCycles.
- MVP uses a standard repeating weekly ScheduleCycle. Rotating A/B weeks are out of MVP.

The calendar/scheduling-time chain is:

`AcademicTerm → Calendar → BellSchedule → ScheduleDay → TimeSlot`

A Calendar belongs to one AcademicTerm. A BellSchedule belongs to one Calendar. A BellSchedule contains ScheduleDays, and each ScheduleDay contains its TimeSlots.

Every TeachingRequirement belongs to exactly one AcademicTerm and exactly one ScheduleCycle. Every GenerationInputRevision and TimetableVersion belongs to exactly one AcademicTerm/ScheduleCycle scheduling context. A generation request cannot combine unrelated academic contexts.

### Cross-Partition LearnerGroup Collision Rule

A Section has one primary whole-section LearnerGroup and may have multiple explicitly disjoint LearnerGroupPartitions.

- Sibling subgroups within the same explicitly disjoint partition are mutually disjoint and may run simultaneously.
- LearnerGroups from different partitions of the same Section are assumed to conflict by default.
- An explicit supported disjoint relationship may prove groups from different partitions disjoint.
- The whole-section LearnerGroup conflicts with every subgroup of that Section.
- Arbitrary student-level overlap is not modeled in the MVP.
- Preflight validation rejects contradictory or ambiguous learner-group relationships.

The canonical SchedulingProblem receives deterministic learner-group conflict relationships; the CP-SAT model does not infer them.

### AcademicDataRevision and GenerationInputRevision

Each schedulable AcademicTerm maintains an authoritative academic-data revision.

The revision increments atomically whenever scheduling-relevant academic data changes, including changes to teachers, teacher availability/policy, subjects, classes, sections, learner groups/partitions, rooms, room availability, teaching requirements, bell schedule/TimeSlot configuration, or scheduling-relevant constraint configuration.

Unrelated changes such as user-profile or billing changes do not increment this revision.

Import Preview records the AcademicDataRevision it validated against. Import Confirmation succeeds only if that revision is unchanged; otherwise the preview is stale and must be revalidated.

GenerationInputRevision is an immutable, complete canonical solver-input snapshot derived from exactly one AcademicDataRevision and records the source revision. A generation cannot silently combine data from different revisions.

A successful GenerationJob produces exactly one candidate draft TimetableVersion referencing exactly one GenerationInputRevision.
